# Chapter 02 — Selecting a Retrieval Engine per Customer Deployment

## Why this chapter exists

The retrieval-engine decision is the first grounding-layer choice an FDE makes on a pilot. Like the provider stack in mod-104 Chapter 02, it constrains everything downstream — the permission-propagation strategy (Chapter 03), what PII handling the engine can do natively (Chapter 04), the achievable retrieval latency slice of the end-to-end SLO (mod-104 Chapter 05), the ingestion pipeline's scale (mod-103 Chapter 09). It is also the decision that the customer's platform team scrutinises hardest — because, in a VPC-install archetype, they are usually the ones who end up operating it.

Nine engines cover the realistic design space. Three are hosted-first vector databases, four are open-source engines that can be self-hosted (two of them with hosted variants), one is a general-purpose search engine with vector support, and two are hyperscaler-native search services. This chapter walks them, scores against the six-dimensional budget from Chapter 01, and runs the three customer archetypes through the matrix.

## The nine candidates

### Hosted-first vector databases

**Pinecone** — [docs.pinecone.io](https://docs.pinecone.io). Fully managed vector DB; serverless and pod-based tiers. Metadata filtering, namespaces for tenant isolation, sparse-dense hybrid. SOC 2 Type II; HIPAA available on Enterprise. AWS, GCP, Azure region choice depending on tier. No self-host option.

**Weaviate** — [weaviate.io/developers/weaviate](https://weaviate.io/developers/weaviate). Open-source core (Apache 2.0) with hosted [Weaviate Cloud](https://weaviate.io/developers/wcs). HNSW index, hybrid dense + BM25, modular vectoriser architecture. Compliance surface published on the Weaviate Cloud [trust page](https://weaviate.io/service/security). Self-hostable on Kubernetes.

**Qdrant** — [qdrant.tech/documentation](https://qdrant.tech/documentation/). Open-source core (Apache 2.0) with [Qdrant Cloud](https://qdrant.tech/cloud/). Rust implementation; HNSW; rich payload filtering with secondary indexes; sparse vectors for hybrid. Self-hostable on Kubernetes or bare metal.

### Open-source engines (self-hosted or embedded)

**Chroma** — [docs.trychroma.com](https://docs.trychroma.com/). Open-source (Apache 2.0). Developer-friendly, embedded by default, small-team hosted service. Excellent for prototyping and pilots at small corpus scale; less mature at production scale and HA than the engines above.

**pgvector** — [github.com/pgvector/pgvector](https://github.com/pgvector/pgvector). Postgres extension adding vector type, HNSW, IVFFlat. Runs wherever Postgres runs — including every hyperscaler's managed Postgres ([AWS RDS / Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_PostgreSQL.html#PostgreSQL.Concepts.General.FeatureSupport.Extensions), [Azure Database for PostgreSQL](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-extensions), [Cloud SQL for PostgreSQL](https://cloud.google.com/sql/docs/postgres/extensions)). The only engine where the vectors sit in the same transactional store as the structured data.

### General-purpose search with vectors

**Elasticsearch** — [elastic.co/guide/en/elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/dense-vector.html). Dense vector field type, HNSW, kNN search, hybrid retrieval combining BM25 and kNN (e.g. [reciprocal rank fusion](https://www.elastic.co/guide/en/elasticsearch/reference/current/rrf.html)). ELSER sparse model for semantic search without external embeddings. Document-level security and field-level security on the platinum tier. Self-hostable; hosted via Elastic Cloud. [OpenSearch](https://opensearch.org/docs/latest/search-plugins/vector-search/) is an Apache-2.0 fork with similar vector capabilities and strong AWS-native support.

**Vespa** — [docs.vespa.ai](https://docs.vespa.ai/). Yahoo-originated, open-source. Combines structured search, vector search, and tensor compute in one engine; production-grade at very large scale. Self-hosted; hosted via [Vespa Cloud](https://cloud.vespa.ai/). The highest-ceiling engine on the list; also the steepest operational curve.

### Hyperscaler-native search services

**Azure AI Search** (formerly Azure Cognitive Search) — [learn.microsoft.com/en-us/azure/search](https://learn.microsoft.com/en-us/azure/search/). Managed by Microsoft; vector + full-text + hybrid retrieval ([semantic ranker](https://learn.microsoft.com/en-us/azure/search/semantic-search-overview) built in). Document-level security integrations with Microsoft Entra / Azure AD. First-party within Azure; EU Data Boundary coverage per the Microsoft commitment.

**Vertex AI Search** (formerly Enterprise Search / Gen App Builder) — [cloud.google.com/enterprise-search](https://cloud.google.com/enterprise-search). Managed by Google; vector + structured + hybrid; built-in connectors for Google Drive, SharePoint, Confluence, Jira, Salesforce, ServiceNow, and others. ACL propagation is a published, first-class feature — see [Identity-aware search](https://cloud.google.com/generative-ai-app-builder/docs/identity-provider-config). Regional availability across GCP.

A note on the list: the FDE does not need to know every engine in depth. The job is to know enough about each to score it correctly on the first three gate dimensions and the three optimisation dimensions, read the primary docs on a short list of two or three, and make the call. The RAG engineer owns the tuning depth.

## The six decision dimensions

Chapter 01 named the six-dimensional budget. This chapter uses those six as the scoring axes.

### Data residency and sovereignty

The question: *where do the source documents, the chunks, the embeddings, the query logs, and the index itself physically process and rest?*

Specific sub-questions per candidate:

- In which region(s) can the engine run?
- Is the engine data plane reachable over PrivateLink / Private Service Connect / Private Endpoint, or only over the public internet?
- Does the engine retain query logs? For how long? Where?
- Does the embedding model (whether called by the engine or by the ingestion pipeline) sit in the same residency boundary?
- Are the backups and snapshots in the same region as the primary?

Residency applies to **vectors as derived personal data** when the source documents contain PII. Some regulators (notably under GDPR interpretation) treat embeddings as personal data subject to the same transfer and retention rules as the source. Chapter 04 gives the PII posture; Chapter 02 names the engine's physical boundary.

This dimension is a **gate**. An engine that cannot run in the customer's required region on the customer's required network path is not a candidate at any score.

### Compliance

The question: *does this engine (or its managed-service offering) sit inside the customer's regulatory regime?*

Same sub-questions as mod-104 Chapter 02: SOC 2 Type II, HIPAA BAA, FedRAMP Moderate / High, PCI DSS, ISO/IEC 27001 / 27701, EU AI Act scope, DORA scope, local AI regulations (NYC LL 144, Colorado AI Act, CCPA, BIPA).

Authoritative references:

- [SOC 2 trust reports](https://www.aicpa-cima.com/) — each vendor publishes under NDA.
- [HIPAA Privacy Rule](https://www.hhs.gov/hipaa/for-professionals/privacy/index.html).
- [FedRAMP Marketplace](https://marketplace.fedramp.gov/) — authoritative list of authorized services.
- [EU Data Boundary](https://learn.microsoft.com/en-us/privacy/eudb/eu-data-boundary-learn) — Microsoft's commitment.
- [GDPR](https://gdpr-info.eu/) — Chapter V on international transfers.

This dimension is also a **gate**.

### Permission fidelity

The question: *can the engine express the customer's actual permission model with enough fidelity that no unauthorised document can surface?*

Specific sub-questions:

- Can the engine filter results by per-document metadata (user ID, group IDs, role tags, attribute key-value pairs, classification labels)?
- Does the engine support **pre-filtered** vector search (filter first, then rank over the filtered set), or only post-filter (rank first, then drop filtered docs)? Post-filter leaks ranks and counts even when documents are not returned.
- Can the engine express boolean composition over filters (user is in group A **AND** has attribute X **AND** document classification ≤ Y)?
- Does the engine support attribute-based access control (ABAC) at scale, or only group membership?
- How does an ACL change propagate — is the filter evaluated at query time against a fresh ACL, or is a stale snapshot baked into the chunk metadata at indexing time?

Chapter 03 makes this decision precise. Chapter 02's job is the gate: engines that cannot pre-filter, or cannot express boolean filter composition, or cannot update filter values without re-indexing, are structurally weaker on permission fidelity.

This dimension is a **gate**. An engine whose permission expressiveness is lower than the customer's permission model means the pilot will leak — or will be wrapped in an application-layer filter that leaks its own information (query times, counts, suggestions).

### Latency

The question: *can the engine meet the retrieval slice of the end-to-end SLO?*

Specific sub-questions:

- Observed p50 / p95 top-k latency at the pilot's corpus size, dimensionality, and query rate.
- Hybrid retrieval overhead (dense + sparse fused) vs. dense-only.
- Rerank latency if the engine runs rerank server-side.
- Cold-start cost for less-queried indexes (particularly serverless vector DBs).
- Multi-tenant noisy-neighbour behaviour on shared tiers.

Published numbers to read before committing:

- [ANN-Benchmarks](http://ann-benchmarks.com/) — ongoing open benchmark of approximate-nearest-neighbour libraries and engines; useful for relative shapes, not absolute production numbers.
- Each vendor's own latency documentation and reference architectures.
- The only authoritative number is one measured on the customer's egress path to the specific region with the pilot's prompt sizes and corpus.

This dimension is an **optimisation**, not a gate — unless the end-to-end latency SLO is tight enough that only one engine can meet it (uncommon at pilot scale).

### Cost

The question: *does per-query and per-document-stored cost fit the value-per-task the pilot promised?*

Specific sub-questions:

- Per-vector storage cost at the pilot's corpus size.
- Per-query cost at the pilot's expected query rate.
- Re-embedding cost at a reasonable drift cadence (every chunking / embedding change rebuilds the index).
- Reserved / committed pricing for predictable workloads.
- For self-hosted: compute, storage, operator time — the all-in cost.

Published pricing pages (always re-read before writing numbers):

- [Pinecone pricing](https://www.pinecone.io/pricing/)
- [Weaviate Cloud pricing](https://weaviate.io/pricing)
- [Qdrant Cloud pricing](https://qdrant.tech/pricing/)
- [Elastic Cloud pricing](https://www.elastic.co/pricing)
- [Azure AI Search pricing](https://azure.microsoft.com/en-us/pricing/details/search/)
- [Vertex AI Search pricing](https://cloud.google.com/generative-ai-app-builder/pricing)
- Self-hosted engines: total cost of ownership depends on hardware and operator time — scope with the customer's platform team.

This dimension is an **optimisation**.

### Hybrid / filter / freshness capability

The question: *does the engine natively support the retrieval shape the pilot needs?*

Three sub-dimensions:

**Hybrid dense + sparse.** BM25 + vector with reciprocal rank fusion or weighted combination. The literature (and field experience) are consistent that hybrid beats dense-only on long-tail queries, acronyms, exact-match legal / policy language, and misspellings. Engines with strong hybrid:

- Elasticsearch / OpenSearch, Vespa — full-text heritage.
- Weaviate (BM25 + vector + RRF), Qdrant (sparse vectors + dense vectors with payload filters).
- Azure AI Search and Vertex AI Search — hybrid is first-class.
- Pinecone — sparse-dense hybrid supported, less first-class than full-text engines.
- pgvector — pair with Postgres `tsvector` for BM25-like behaviour.
- Chroma — recent hybrid support; less mature.

**Rich filters.** See the Permission-fidelity dimension above; same questions, different frame. Filters also serve freshness (e.g. `as_of > ...`), locale (e.g. `locale = 'en-GB'`), lifecycle (e.g. `status = 'current'`).

**Freshness / incremental updates.** The pilot's corpus mutates: documents are added, revised, deleted; permissions change; metadata updates. The engine's upsert / delete latency and the cost of partial re-embedding matter. pgvector and Elasticsearch handle this through standard database / index operations. Vector-first engines vary — Pinecone upserts are fast; Weaviate, Qdrant, and Chroma support upsert; some embedded setups require full rebuilds. Chapter 03 makes the invalidation question precise.

This dimension is an **optimisation**, but a deficiency here can effectively gate the engine for workflows that need hybrid or that mutate frequently.

## A scoring approach

Score each candidate on each dimension 1–5. Weight by customer context — residency / compliance / permission weight higher in regulated verticals, latency weights higher for real-time user-facing retrieval, cost weights higher for large corpora with heavy query loads.

Starter template:

```
Customer: <name>  |  Pilot: <workflow>  |  Date: 2026-11-04
Corpus: <size, docs, growth rate>  |  Query rate: <QPS p50 / p95>  |  Workflow latency SLO: <ms>

Dimension                 Weight   Pinecone    Weaviate    Qdrant     Elastic     Azure AI    Vertex AI    pgvector    Vespa       Chroma
---------------------------------------------------------------------------------------------------------------------------------------
Data residency            W_R      s*W         s*W         s*W        s*W         s*W         s*W          s*W         s*W         s*W
Compliance                W_C      s*W         s*W         s*W        s*W         s*W         s*W          s*W         s*W         s*W
Permission fidelity       W_P      s*W         s*W         s*W        s*W         s*W         s*W          s*W         s*W         s*W
Latency                   W_L      s*W         s*W         s*W        s*W         s*W         s*W          s*W         s*W         s*W
Cost                      W_$      s*W         s*W         s*W        s*W         s*W         s*W          s*W         s*W         s*W
Capability                W_K      s*W         s*W         s*W        s*W         s*W         s*W          s*W         s*W         s*W
---------------------------------------------------------------------------------------------------------------------------------------
                                   TOTAL       TOTAL       TOTAL      TOTAL       TOTAL       TOTAL        TOTAL       TOTAL       TOTAL

Gate check:
  - Any s_R = 0 → engine disqualified on residency
  - Any s_C = 0 → engine disqualified on compliance
  - Any s_P = 0 → engine disqualified on permission fidelity (filter model cannot express customer model)
```

Rules (mirroring mod-104 Chapter 02):

- **Gate before score.** Any engine that fails residency, compliance, or permission fidelity drops out.
- **Rationale per cell.** Every cell has a one-line rationale citing the primary source (the engine's docs, the compliance page, the pricing page). The matrix is a security-review artifact.
- **Weights are defended in the ADR.** Why is permission fidelity weighted 3 and cost weighted 1 for this customer? Point to the data-contract conversation and the business-KPI value-per-task.
- **Date the matrix.** Engine features and pricing change. A matrix from six months ago is a snapshot, not a current recommendation.

## Three customer archetypes and their likely engine

The three archetypes from Chapter 01 run through the matrix. These are shapes, not prescriptions — a specific customer can land anywhere.

### Archetype A — Mid-size US healthcare payer

Constraints:

- HIPAA + state privacy; BAA required.
- SOC 2 Type II onboarding; cloud of record AWS.
- Corpus: ~500k documents (member policies, plan docs, prior auths, call-centre notes). Growth ~10k/week.
- Query rate: pilot projected 2 QPS p50, 10 QPS p95.
- Latency SLO: a few seconds end-to-end; retrieval slice ~300 ms p95.
- Permission model: RBAC (CSR tier 1 / tier 2 / supervisor) + per-document ACLs on sensitive items.

Likely engine: **AWS OpenSearch with vector support** (or Elastic Cloud on AWS) for the first shortlist, with **pgvector on AWS Aurora Postgres** as a serious alternative if the structured data already lives there.

Rationale rows: residency = inside AWS region (5); compliance = AWS BAA covers OpenSearch / Aurora (5); permission fidelity = document-level security on Elastic/OpenSearch is strong (5); hybrid+filter+freshness = native BM25 + kNN + document security + rapid upsert (5); latency = native in-region (4); cost = Aurora ≈ cheaper if small corpus, OpenSearch ≈ linear with vector count (3).

Pinecone and Weaviate Cloud are possible if the customer's residency / BAA posture allows them — specifically, Pinecone's HIPAA offering is on specific plan tiers; verify current scope <!-- needs-research: confirm Pinecone HIPAA plan tier and regional coverage at pilot time -->. Qdrant Cloud similarly. Vertex AI Search is on the wrong cloud. Chroma is too immature for 500k docs in production. Vespa is more engine than this pilot needs.

### Archetype B — US federal agency

Constraints:

- FedRAMP Moderate baseline with path to High.
- Data classified at FOUO.
- Must run inside GovCloud enclave.
- Corpus: ~200k policy memos, regulations, FOIA-released docs. Low growth.
- Query rate: ~1 QPS p95.
- Latency tolerance: several seconds end-to-end; retrieval slice ~500 ms p95.
- Permission model: classification + cleared-personnel attributes (ABAC).

Likely engine: **Self-hosted — OpenSearch or Qdrant in GovCloud**, or **pgvector on GovCloud RDS**, standup owned by the customer's platform team per mod-103 Chapter 09. Vespa is a plausible option if the customer's platform team has existing operational maturity. Pinecone / Weaviate Cloud / Qdrant Cloud are typically off the table because of FedRAMP boundary. Azure AI Search inside [Azure Government](https://azure.microsoft.com/en-us/explore/global-infrastructure/government/) and Vertex AI Search inside [Assured Workloads](https://cloud.google.com/assured-workloads) may be feasible if the agency is on the respective cloud — but check current authorization <!-- needs-research: confirm Azure AI Search / Vertex AI Search FedRAMP High status at pilot time -->.

Rationale rows: residency = GovCloud (5 for in-enclave engines, 0 for commercial-only); compliance = FedRAMP boundary (gate); permission = ABAC via engine filters (4 or 5); hybrid = strong on Elastic/OpenSearch / Vespa (5); latency = in-region (4); cost = higher operational burden (2).

### Archetype C — EU retail bank

Constraints:

- GDPR + DORA.
- Data must remain in the EU.
- Cloud of record Azure, EU Data Boundary active.
- Corpus: ~300k product T&Cs, internal procedures, prior case notes, regulatory updates. Growth ~20k/week on case notes.
- Query rate: 3 QPS p50, 15 QPS p95.
- Latency SLO: a few seconds end-to-end; retrieval slice ~300 ms p95.
- Permission model: branch / region / team filters; GDPR right-to-be-forgotten on customer data.

Likely engine: **Azure AI Search** in an EU region (Sweden Central / West Europe / North Europe) with EU Data Boundary. Document-level security via Entra ID group membership; native hybrid + semantic ranker; native connectors for SharePoint and other Microsoft sources. **pgvector on Azure Database for PostgreSQL** is a strong alternative if the structured data already lives there.

Rationale rows: residency = EU region under EU Data Boundary (5); compliance = SOC 2 + ISO 27001, customer still writes DORA ICT register (4 — standard caveat); permission = Entra-integrated document security (5); hybrid+filter+freshness = native (5); latency = in-region (5); cost = Azure contract (3).

Vertex AI Search is on the wrong cloud under DORA ICT concentration risk. Pinecone is possible if its EU region plan tier meets the customer's requirements <!-- needs-research: confirm Pinecone EU region tier at pilot time -->. Self-hosted Qdrant / Weaviate / Elasticsearch on Azure is plausible if the customer's platform team runs it. Vespa is more engine than needed.

## The engine decision as an ADR

Mirror mod-104's ADR shape, scoped to the retrieval engine decision.

```
ADR-NN: Retrieval engine for <customer> pilot <workflow>

Status: proposed | accepted | superseded (date, by ADR-MM)

Context
  - Customer: <name, vertical>
  - Workflow: <one-line description, from pilot-scope doc>
  - Regulatory regime: HIPAA / FedRAMP / GDPR+DORA / none
  - Cloud of record: AWS / GCP / Azure / multi / on-prem
  - Corpus shape: <doc count, growth rate, mean doc size, mean chunk size>
  - Query rate (projected): <p50 QPS, p95 QPS>
  - Retrieval latency SLO: <p95 ms budget>
  - Permission model summary: <RBAC / ABAC, groups, classifications, right-to-forget>
  - PII contract summary: <see mod-105 Chapter 04 ADR>

Decision
  <Engine + deployment model>
  Example: Elastic Cloud on AWS in us-east-1, hybrid dense+BM25 with reranker
  via ELSER; document-level security against customer's SAML IdP.

Alternatives considered
  - <Engine X> — rejected because <reason tied to the six dimensions>
  - <Engine Y> — rejected because <reason>
  - <Engine Z> — deferred; see ADR-MM for the self-hosted path if pilot graduates.

Consequences
  - Positive: <residency satisfied, hybrid first-class, customer already holds Elastic contract>
  - Negative: <ELSER model licensing / tier required; verify>
  - Risks: <re-embed cost if we revise chunking; mitigation: incremental re-index path>

Gate checks
  - Residency: pass (in customer AWS region, no data egress)
  - Compliance: pass (SOC 2 + AWS BAA path)
  - Permission fidelity: pass (document-level security, boolean filter composition)

Sign-off
  - FDE: <name>
  - Customer AppSec: <name>
  - Customer platform team (if VPC-install): <name>
  - Customer Legal (if required): <name>
  - Date: <YYYY-MM-DD>
```

## Per-engine eyebrow-raises

Facts worth re-checking before committing, from the field.

- **Pinecone** — serverless and pod-based tiers behave differently on cost and latency at scale; the choice matters. HIPAA requires the Enterprise tier; verify the current plan boundary. Namespaces are the primary tenant-isolation primitive.
- **Weaviate** — hosted cluster size and replication topology are explicit choices; the defaults are not production. Modules (text2vec-*) are a nice shortcut but tie the engine to a specific embedding provider; set them explicitly.
- **Qdrant** — payload-index design is explicit; without a secondary index on a filter field, filtered search is slow at scale. Hybrid via sparse vectors is first-class now; set it up at index creation, not retrofitted.
- **Chroma** — fantastic for prototypes; scale and operations maturity are an open question for anything above low-six-figures of documents under production load. Use for discovery, graduate for pilot.
- **pgvector** — IVFFlat vs. HNSW is a parameter choice with real impact. ANN recall vs. speed trade is set per index. Use HNSW for pilot unless you have a reason not to. Partition large tables; think about table size vs. index memory.
- **Elasticsearch / OpenSearch** — document-level security is on the paid tier (Platinum/Enterprise or OpenSearch's fine-grained access control). ELSER (Elastic's sparse encoder) is a strong hybrid option but licensed per Elastic's terms.
- **Vespa** — power / complexity ceiling is high. If the pilot is "search over a known corpus with hybrid and filters," most teams are faster on an engine with narrower scope. Vespa is right when the customer already operates it, when scale genuinely exceeds the competition, or when the workload needs tensor compute at retrieval time.
- **Azure AI Search** — semantic ranker is a first-party reranker; it is an optional add-on with its own cost. Document-level security via security filters integrates with Entra ID; wire this up via the official pattern documented in [Azure AI Search docs on security filtering](https://learn.microsoft.com/en-us/azure/search/search-security-trimming-for-azure-search).
- **Vertex AI Search** — the identity-aware search feature propagates Google Workspace / Microsoft Entra / custom IdP ACLs from connected sources. If the customer runs on Google Workspace or wants managed connectors to common SaaS sources, this is a significant time-saver. Regional availability varies.

## Anti-patterns

**Picking by vendor benchmark deck.** Every vendor publishes benchmarks that favour their engine on their chosen workload. Benchmarks are inputs; the customer's six-dimensional budget is the decision.

**Picking Pinecone (or any hosted SaaS) before clearing residency.** Residency is the first gate. A vendor's "we're in AWS us-east-1" statement is not the same as the customer's "our data cannot leave eu-central-1" requirement. Gate first.

**Picking pgvector because "we already have Postgres." Then discovering at 2M vectors that the HNSW build takes 6 hours and the query latency is 800 ms p95 on the pilot's shape.** pgvector is a strong choice at small-to-medium scale; at large scale and dimensionality it requires discipline (partitioning, index tuning, pre-aggregated embeddings). Co-locate decisions with the RAG engineer (Chapter 07) when the pilot's corpus grows.

**Leaving the engine behind an untouchable "team choice."** The retrieval engine is a *customer deployment* decision, not a product one. Mod-111 (field → product loop) is where cross-customer patterns surface; the pilot does not have to stand on the product's default engine if the customer's budget pushes a different one.

**Not writing the ADR.** Same as mod-104 Chapter 02. A decision that lives only in a slide deck gets re-litigated in week 10 by someone who wasn't in the room in week 2.

## Summary

- Nine engines span the design space: Pinecone, Weaviate, Qdrant, Chroma, pgvector, Elasticsearch (and OpenSearch), Vespa, Azure AI Search, Vertex AI Search.
- Six decision dimensions score each candidate: data residency (gate), compliance (gate), permission fidelity (gate), latency, cost, and hybrid / filter / freshness capability.
- Three customer archetypes point to typical landing spots: Elastic/OpenSearch or pgvector on AWS for a healthcare payer; self-hosted OpenSearch / Qdrant / pgvector in GovCloud for a federal agency; Azure AI Search for an EU bank.
- The engine decision is captured as a one-page ADR signed by the FDE, the customer's AppSec, the customer's platform team (for VPC-install), and legal where required.
- Permission fidelity is a gate dimension specific to retrieval: an engine that cannot express the customer's permission model structurally cannot be made safe at the application layer alone.
- Residency applies to embeddings, not only to source documents — embeddings are derived personal data in some regimes.
- Chapter 03 is next: propagating the customer's permission model into whichever engine the matrix selected.
