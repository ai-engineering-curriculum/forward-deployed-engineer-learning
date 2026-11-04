# Chapter 03 — Propagating the Customer's Permission Model into the Retriever

## Why this chapter exists

The single most common week-10 blocker on a grounding pilot is: *the retriever surfaced a document to a user who should not have been able to see it*. The customer's AppSec discovers this in security review, or an end-user discovers it in UAT, and the pilot stops until the permission story is rebuilt. The cost is weeks, not days.

The fix is cheap only if it is designed in week 2. The discipline is: *never index a document whose permission model is not represented alongside it, and never issue a retrieval query that does not filter against the current user's identity*. This chapter makes that discipline precise.

Four things have to land: (1) a model of the customer's permission system the retriever can speak; (2) an ingestion-time propagation that stamps every chunk with the right filterable metadata; (3) a query-time enforcement that composes the current user's clearance into every retrieval call; (4) an invalidation path for permission changes (joiners, leavers, role changes, document reclassification, right-to-be-forgotten).

## The customer permission surface the FDE will meet

Customers rarely speak one permission model. A real pilot's corpus straddles several systems, each with its own posture. The FDE's first move is enumeration.

**Role-based access control (RBAC).** Users have roles; roles grant access to resources. The simplest and most common shape. "Tier 1 CSR, Tier 2 CSR, Supervisor, Compliance."

**Attribute-based access control (ABAC).** Access is a function of attributes on the user and on the resource: user clearance level, user department, resource classification, resource department-of-record, resource effective date range. NIST [SP 800-162](https://nvlpubs.nist.gov/nistpubs/specialpublications/NIST.SP.800-162.pdf) is the canonical reference. Common in federal, financial services, large enterprise.

**Discretionary access control (DAC).** A document has an explicit list of users / groups who can read it (an ACL). SharePoint, Google Drive, Confluence, Jira all land here. The owner of the document can grant or revoke access.

**Row-level security (RLS).** A SQL-style policy that filters rows a user may read in a relational database. Standard in modern data warehouses — [Postgres RLS](https://www.postgresql.org/docs/current/ddl-rowsecurity.html), [Snowflake row access policies](https://docs.snowflake.com/en/user-guide/security-row-intro), [BigQuery row-level security](https://cloud.google.com/bigquery/docs/managing-row-level-security), [Databricks row filters](https://docs.databricks.com/en/data-governance/unity-catalog/row-and-column-filters.html). The pilot that reads directly from a warehouse often encounters RLS.

**Classification labels.** Document is tagged `Public`, `Internal`, `Confidential`, `Secret`, `TS`; or `FOUO`, `CUI`, `S`, `TS`. A user with clearance level L can read documents with label ≤ L. Federal / defence / regulated verticals.

**Mandatory access control (MAC).** Classic multi-level security. The user and the resource have labels; the system enforces non-bypassable rules. Rare outside government / defence; the FDE identifies it and does not try to invent it in software.

**Right-to-be-forgotten / data-subject rights.** Under GDPR Article 17 ([Right to erasure](https://gdpr-info.eu/art-17-gdpr/)), the customer must be able to remove a data subject's personal data on request. For a retriever, this means a hard-delete path from every index that holds derivative data (embeddings, chunks, surrogate keys), not only the source.

Most pilots see a mix: a corpus from SharePoint with DAC, from Confluence with DAC-plus-groups, from a warehouse with RLS, from a SaaS with its own API-level permissions. The FDE's job is to compose these into one consistent filter the retriever can enforce.

## The three propagation patterns

Three shapes. Pick one per engine; many deployments combine two.

### Pattern A — Permission-aware metadata on every chunk

Each document (and therefore each chunk) carries a filterable permission payload. At ingest, the pipeline fetches the source's permission state and writes it into the chunk metadata; at query time, the retriever applies a filter that admits only chunks whose payload is compatible with the current user's identity.

Shape of a chunk's metadata:

```json
{
  "doc_id": "...",
  "chunk_id": "...",
  "source": "confluence",
  "source_object_id": "page-18422",
  "allow_users":   ["u:12", "u:79"],
  "allow_groups":  ["g:eu-support-tier-2", "g:eu-support-tier-1"],
  "deny_users":    ["u:421"],
  "classification": "internal",
  "department":    "eu-support",
  "effective_from": "2024-11-01",
  "effective_until": null,
  "as_of":          "2026-11-04T07:00:00Z"
}
```

The query composes a filter from the user's identity and clearance:

```
filter:
  (allow_users contains ${user.id}) OR
  (allow_groups intersects ${user.group_ids})
  AND
  NOT (deny_users contains ${user.id})
  AND
  classification <= ${user.max_classification}
  AND
  (department = ${user.department} OR classification = 'public')
  AND
  effective_from <= now() AND (effective_until is null OR effective_until > now())
```

Engine-shape notes:

- **Elasticsearch / OpenSearch** — document-level security filters on the query; the official pattern is a filter DSL applied per request. See [Azure AI Search docs on security filtering](https://learn.microsoft.com/en-us/azure/search/search-security-trimming-for-azure-search) for one explicit pattern, [OpenSearch fine-grained access control](https://opensearch.org/docs/latest/security/access-control/document-level-security/) for another.
- **Weaviate** — `where` filter composed with the vector query. Multi-tenancy at the schema level gives per-tenant isolation separate from per-user filters.
- **Qdrant** — `filter` parameter on search. Secondary payload indexes on filter fields are essential for performance; see [Qdrant filtering docs](https://qdrant.tech/documentation/concepts/filtering/).
- **Pinecone** — metadata filtering on query; see [metadata filtering docs](https://docs.pinecone.io/guides/data/filter-with-metadata).
- **pgvector** — a `WHERE` clause on the SQL query that composes the vector similarity with structured predicates. pgvector is uniquely flexible here because the filter can touch any Postgres column.
- **Azure AI Search** — security filters per the official pattern; integrates with Entra ID group membership for a Microsoft-first customer.
- **Vertex AI Search** — [Identity-aware search](https://cloud.google.com/generative-ai-app-builder/docs/identity-provider-config) propagates ACLs from connected sources automatically.
- **Chroma / Vespa** — filters on query; Vespa offers rich boolean logic and tensor expressions.

**Pre-filter vs. post-filter.** This matters enough to call out. *Pre-filter* means the engine restricts the candidate set *before* running the ANN search: it searches only the subset the user may read, and ranks within that. *Post-filter* means the engine runs the ANN search over the full set, then drops results the user may not read: the ranks, counts, and timings leak information about documents the user never saw. Pre-filter is the correct posture for permissioned retrieval. Elastic, Qdrant, Weaviate, pgvector (SQL predicate) support pre-filter; some engines default to post-filter at some scales for performance reasons — confirm and configure explicitly.

### Pattern B — Per-tenant / per-group index partitioning

Each tenant / group / classification gets its own index (or namespace). At query time, the retriever picks the index(es) the user may read and searches only those.

When to pick this pattern:

- Pinecone's **namespaces** are the canonical primitive; one namespace per tenant is common. See [Pinecone namespaces](https://docs.pinecone.io/guides/indexes/use-namespaces).
- Weaviate's **multi-tenancy** at the schema level; see [Weaviate multi-tenancy](https://weaviate.io/developers/weaviate/manage-data/multi-tenancy).
- Qdrant's **payload partitioning** (one collection per tenant) or filter-based tenant-id approach — see [Qdrant multi-tenancy](https://qdrant.tech/documentation/guides/multiple-partitions/).
- When tenant isolation has to be demonstrable to the customer's AppSec (not only enforced in a filter).

Trade-offs: strong isolation; higher operational cost at tenant count; cross-tenant retrieval becomes a routing problem rather than a filter.

### Pattern C — Permission-at-source on retrieval

The retriever performs a candidate-set recall from the index (dense + hybrid) without strong permission filtering, then round-trips to the source of truth to verify the user's authorisation for each candidate before returning.

When this shows up:

- The source's permission model is too dynamic to mirror cost-effectively (fine-grained per-record ACLs that change per minute).
- The customer's legal / security posture requires that authorisation decisions are made by the source system, not by a derived index.

Trade-offs: slower retrieval (an extra hop per candidate — amortised with batching); correct by construction because the source always decides; limits top-k inflation because many candidates may drop after authorisation.

Most real pilots use Pattern A, often with Pattern B underneath for tenant partitioning, with Pattern C reserved for one or two high-sensitivity surfaces.

## Mapping customer systems to filterable metadata

The FDE's week-2 move: for each source feeding the corpus, write down the mapping from the source's permission concepts to the retriever's filter model.

Examples the FDE will meet.

**SharePoint / Microsoft 365.** Users, Entra ID groups, item-level permissions. Via Microsoft Graph [`drives/{drive-id}/items/{item-id}/permissions`](https://learn.microsoft.com/en-us/graph/api/listitem-list-permissions) the pilot pulls granted identities. For Microsoft-first customers, Entra group IDs come down with the user's token at query time. The mapping is: `allow_groups` ← Entra group IDs on the item; `allow_users` ← granted identities; `inheritance` must be resolved to flat group lists at ingest.

**Confluence.** Spaces, pages, restrictions. The [Confluence Cloud REST API](https://developer.atlassian.com/cloud/confluence/rest/v2/) exposes restrictions on pages; space-level permissions come from the space. Map Confluence groups to the pilot's group model; carry page restrictions as `allow_users`/`allow_groups` on each chunk.

**Jira.** Projects, issue security schemes. Issue security levels give a classification-like model; issue-level restrictions give DAC. Map issue security level to `classification`; map issue-level restrictions to `allow_*`.

**Google Drive / Workspace.** Drive-level permissions, folder inheritance, item-level sharing. Drive API [`permissions.list`](https://developers.google.com/drive/api/reference/rest/v3/permissions) exposes grants; map to `allow_users`/`allow_groups`. Vertex AI Search's identity-aware search automates this if the pilot uses Vertex.

**Salesforce.** Object permissions, sharing rules, field-level security. SOAP-era queryable; expose via the Bulk / REST API. Map sharing to `allow_*`; use `department` or similar for territory-based access.

**ServiceNow, Workday, Zendesk.** Each with its own model; the enumeration is the same.

**Warehouses (Snowflake / BigQuery / Databricks).** Row-level security policies apply to whoever runs the query. For retrieval, either (a) the ingestion pipeline runs as a service principal with broad read, records the user/group clearance on each row's chunk, and the retriever filters; or (b) the retriever delegates back to the warehouse for permission checks (Pattern C).

**Postgres with RLS.** Same shape as warehouses: run ingest as a privileged role, carry per-row predicates into chunk metadata, filter at query. Direct RLS at retrieval time is feasible for pgvector deployments — see below.

**Vertical SaaS (Epic, Veeva, etc.).** Treat as a black-box API with its own permission model. Pattern C often applies here because the system's permission model is the regulator's compliance boundary and cannot be mirrored.

For every source, document three things:

1. **Mapping.** Source concept → filter metadata.
2. **Resolution strategy.** Where does the ingestion pipeline resolve group memberships (ingest-time snapshot, query-time lookup)?
3. **Invalidation hook.** How does a permission change propagate? See the next section.

## The invalidation path for permission changes

The permission model is not static. Six invalidation events the FDE must design for:

**Joiner.** New user; new group memberships. No action on the index — the filter will pick up the new user's groups at query time via their token. The ingestion pipeline does not need to re-run.

**Leaver.** User removed from a group, or departs. The index's chunk metadata does not change; the departed user's token stops containing the group. Query-time filter is correct automatically. *Provided* the query-time filter reads the current token / IdP state, not a cached copy.

**Role change / group rebalance.** Same shape as joiner/leaver — token reflects the change, filter stays correct. The FDE's job is to confirm there is no cache in the critical path that holds a stale group list for more than seconds.

**Document ACL change.** The source's grants change: a page becomes restricted, an issue's security level is raised. The chunk metadata in the index is now stale. The ingestion pipeline must re-ingest the document (or at minimum update the permission metadata) with the new ACL. Lag between source change and index update is a **permission invalidation lag** — a quantity the FDE must name an SLO for.

**Document reclassification.** The classification label changes. Treat as ACL change — re-ingest or metadata-update.

**Right-to-be-forgotten (GDPR).** A data subject requests erasure. The erasure must propagate to: the source system; the ingestion pipeline's staging area; the chunk index; the embedding store (if separate); the vector cache; any derivative artifacts (gold set rows that include the subject's data). The FDE's job is the inventory and the audit trail that proves erasure was complete.

Three mechanisms to implement invalidation.

**Change-data-capture (CDC) from the source.** The source emits change events (Debezium on a database, webhooks on SaaS, Graph change notifications on Microsoft 365, Drive push notifications on Google). The ingestion pipeline listens, filters to events that touch chunk metadata, and updates the index. Owned by `ai-data-engineer` for non-trivial pipelines (mod-103 Chapter 09).

**Scheduled re-crawl with diff.** The pipeline polls the source on a cadence (hourly, daily) and compares to its last snapshot; changed docs are re-ingested. Simpler than CDC; higher lag; higher cost at scale.

**Soft-delete tombstones.** On delete / revoke, write a tombstone row that the retriever's filter checks ( `tombstone = false`). Compact tombstones out of the index on a scheduled job. Useful when hard-delete is expensive but the invalidation must take effect immediately.

The invalidation SLO. Name it in the pilot's documentation. Example: *"a document ACL change in SharePoint is reflected in the retriever's filter within 5 minutes p95, 15 minutes p99; right-to-be-forgotten erasure within 24 hours end-to-end."* The SLO drives the mechanism: 5-minute p95 lag needs CDC or webhooks; 24-hour lag can live on scheduled re-crawl.

## The query-time path

The retriever's call signature changes shape under permission propagation. Instead of `retrieve(query, top_k=20)`, it reads something like:

```python
retrieve(
    query=user_query,
    user=current_user,                # identity + group IDs + clearance
    extra_filters={"locale": "en-GB"},
    top_k=20,
)
```

The internal flow:

1. **Resolve the user's identity envelope.** From the OIDC token the pilot received (mod-103 Chapter 03), extract `sub`, group claims, clearance attributes, `department`, locale. Avoid a database round-trip here if the token carries enough; the gateway and the pilot already paid for identity.
2. **Compose the permission filter.** Mechanically, from the user envelope, build the engine-specific filter expression.
3. **Compose any business filters.** `locale`, `effective_from <= now()`, `status = 'current'`, `source != 'deprecated'`.
4. **Execute the retrieval.** Engine-specific hybrid / dense call with the composed filter.
5. **Log the retrieval.** For audit: the user's `sub`, the query hash, the filter expression, the returned `doc_id`s, the retrieval timings. Not the query text, not the returned chunks, unless the customer's audit regime requires it. Mod-110 covers the observability shape.
6. **Return the chunks.** With `doc_id`, source URL, chunk text, score, `as_of` timestamp, classification. The classification is required for the citation UX in Chapter 05.

Three things to avoid in the query-time path:

- **No application-layer post-filtering of returned results.** If the engine returned a chunk, the FDE's code must assume the user was authorised to see it. Post-filter in application code is a smell — it leaks ranks and counts even if it keeps text out of the response, and it is a hint that the engine's filter did not express the customer's model.
- **No "admin mode" retrieval that bypasses filters.** Some pilots ship a "see-all" mode for debugging or ops. This is the surface the next incident disables in production. If retrieval ever runs without a user filter, audit-log loudly and expire the mode in minutes.
- **No caching of retrieval results across users.** A cache keyed only on the query text cross-contaminates results across identity envelopes. Cache keyed on `(query_hash, filter_hash)` is safe; cache keyed on `query_hash` alone is a leak.

## The audit trail

The customer's AppSec will ask for one. Three shapes to standardise around.

**Retrieval audit log.** One entry per retrieval call: timestamp, user `sub`, pilot session ID, query hash, filter expression (JSON-serialised), returned `doc_id`s, scores, timings. Retained per the customer's regime (90 days common, longer in financial / federal).

**Ingest audit log.** One entry per indexing operation: timestamp, source, source object ID, chunk IDs created / updated / deleted, metadata written, who / what triggered the indexing. The customer's data governance team reads this.

**Invalidation audit log.** One entry per permission or classification change that propagated through the pipeline: timestamp, source object ID, change type (ACL add / revoke / classify / tombstone / RTBF), index update timestamp, invalidation lag. Prove the invalidation SLO.

Pipe audit logs into the customer's SIEM. The customer's AppSec reads from theirs, not yours.

## Three archetypes and their permission posture

### Archetype A — US healthcare payer

Permission shape: RBAC (CSR tier 1, tier 2, supervisor) + DAC on sensitive member records. HIPAA Minimum Necessary Standard applies: a CSR should only retrieve member records necessary for the inquiry. Right-to-be-forgotten does not apply under HIPAA the way it does under GDPR, but [45 CFR §164.526](https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164/subpart-E/section-164.526) (right to amend) and state laws may introduce similar erasure obligations.

Pattern mix: **A (metadata filters) + B (per-plan-type namespace)**. Source systems: policy repository (DAC), claims system (RLS via warehouse), call-centre notes (per-case owner). Invalidation: webhooks from source systems; scheduled re-crawl of policy repo nightly; 5-minute SLO on claims and notes.

### Archetype B — US federal agency

Permission shape: ABAC with classification labels (FOUO / CUI / S), cleared-personnel attributes. MAC applies above a certain classification — treat as a gate, not something to express in retrieval metadata.

Pattern mix: **A (classification + attribute filter) + B (per-classification index)**. Sources: internal policy memo stores, regulation databases, FOIA archives. Invalidation: scheduled re-crawl plus manual reclassification events signed by a records officer.

### Archetype C — EU retail bank

Permission shape: DAC (per-branch / per-region case notes) + classification (public, internal, restricted) + GDPR right-to-be-forgotten.

Pattern mix: **A (branch + classification filter)** on an Azure AI Search index integrated with Entra ID. GDPR adds a right-to-be-forgotten handler: on a data subject request, the pipeline hard-deletes the subject's case data from the source, the index, the embedding store, and any derivative stores, writes an audit record, and acknowledges the request within the GDPR-mandated timeline. [EDPB guidance on the right to erasure](https://www.edpb.europa.eu/our-work-tools/our-documents/guidelines/guidelines-052021-interplay-between-application-article-3-and_en) is one reference; the customer's DPO owns the policy.

## Anti-patterns

**Indexing "to see what comes back" without the permission model wired.** The index now holds documents the pilot users cannot legally be served. Rebuilding the index is not expensive; explaining to the customer's security reviewer why the pilot has an index full of unapproved data is.

**Applying the permission filter in application code after retrieval.** The ranks leaked, the counts leaked, the timings leaked. The engine's native filter is the correct surface.

**Caching retrieval results across users with the same query text.** Cross-identity cache contamination. Keyed on `(query_hash, filter_hash)` is the right move.

**Holding group memberships in a long-lived application cache.** A revoke event in the IdP doesn't propagate; a removed user keeps reading. Trust the token; refresh via standard OIDC flows; cap any cache to seconds.

**No invalidation SLO.** The customer's security reviewer asks "if I revoke access to a document, how fast does the pilot stop surfacing it?" and the pilot team does not have an answer. Pick a number; measure it; publish it.

**No audit log.** The customer's AppSec cannot sign off on a retrieval system they cannot audit. The log is cheap; add it in week 2.

## Summary

- Never index a document whose permission model is not represented alongside it; never retrieve without filtering against the current user's identity.
- Six permission surfaces show up in real customers — RBAC, ABAC, DAC, RLS, classification, MAC — plus GDPR right-to-be-forgotten.
- Three propagation patterns cover the design space: metadata-on-every-chunk, per-tenant index partitioning, permission-at-source retrieval. Most pilots combine two.
- Pre-filter is required for permissioned retrieval; post-filter leaks ranks and counts.
- Six invalidation events — joiner, leaver, role change, ACL change, reclassification, right-to-be-forgotten — each with a propagation mechanism (CDC, scheduled re-crawl, tombstone) and an invalidation-lag SLO.
- The retriever's audit trail has three shapes: retrieval, ingest, invalidation. Pipe into the customer's SIEM.
- The archetypes land on different patterns: healthcare payer on RBAC+DAC metadata, federal agency on ABAC+classification with per-classification indexes, EU bank on DAC+classification with GDPR erasure wired.
- Chapter 04 is next: once the retriever enforces identity, the PII contract — detection, redaction, reversible pseudonymisation, audit — becomes the next gate.
