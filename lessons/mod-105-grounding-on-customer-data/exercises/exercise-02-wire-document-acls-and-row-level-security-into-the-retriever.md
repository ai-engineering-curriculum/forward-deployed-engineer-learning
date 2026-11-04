# exercise-02: Wire Document ACLs and Row-Level Security into the Retriever

**Estimated effort:** 3 hours

## Objective

Stand up a small, working retrieval pipeline whose results are correctly filtered by document ACLs *and* row-level security, against the identity of the calling user. Include the invalidation path for permission changes, the retrieval audit log shape, and a failing-test that would catch a post-filter leak.

The point is not to build a production retriever. The point is to make the permission-propagation story concrete enough that you can defend it in a customer AppSec review.

## Prerequisites

- Read [Chapter 03](../03-permission-propagation.md).
- Working local environment with: a Python 3.11+ runtime, Postgres with the `pgvector` extension (any docker image works), and your choice of a second engine (one of Elasticsearch / OpenSearch / Qdrant / Weaviate — local docker or a free tier).
- An OIDC issuer you can call — [Keycloak](https://www.keycloak.org/) locally, or a mock OIDC token signer (the exercise does not require a real IdP; signed claims are enough).
- From mod-103 Chapter 03, an identity-token model with groups and attributes.

## Problem statement

You are prototyping a grounded support-assistant pilot for a US healthcare payer. The pilot reads from two sources:

- **Policy repository.** Static docs with DAC — each doc has an `allow_groups` list and a `classification` label (`public`, `internal`, `restricted`).
- **Case notes warehouse.** Postgres table `case_notes(case_id, member_id, branch, text, created_at)` protected by row-level security — a CSR can only read rows where `branch = current_setting('app.user_branch')`.

Three roles with different permission envelopes:

- `csr_tier_1` — group `csr`, clearance `internal`, branch `boston`.
- `csr_tier_2` — groups `csr`, `csr_tier_2`, clearance `internal`, branch `nyc`, additional per-doc grants `['doc-18001', 'doc-18002']`.
- `supervisor` — groups `csr`, `supervisor`, clearance `restricted`, branch `any`, permission to read any branch's case notes.

Build:

1. An **ingest pipeline** that loads a sample policy corpus (provide ~30 markdown docs with varied ACLs and classifications) and a sample `case_notes` table (~500 rows across 3 branches), computes embeddings, and writes the chunks with filterable permission metadata into both pgvector and your second engine.
2. A **retriever** with signature `retrieve(query, user_token, extra_filters=None, top_k=20)` that: parses the OIDC token; composes the permission filter; executes a pre-filtered hybrid (dense + BM25 / sparse) retrieval against both engines; fuses results; and returns top-k with citation metadata.
3. An **invalidation pipeline** that reflects three events — a policy doc's ACL changing, a case note being deleted (right-to-be-forgotten), and a user's group membership changing — into the retriever's filter within the SLO you commit to.
4. A **retrieval audit log** in the shape named in Chapter 03.
5. A **test suite** that covers: a `csr_tier_1` query that must not return `restricted` docs; a cross-branch case-note query that must not leak another branch's rows; a `supervisor` query that must; and an ACL-revocation test that re-runs the same query five seconds after the revocation and verifies the doc is no longer returned.

## Requirements

- The retriever must be pre-filtered — the engine applies the filter before (or as part of) ANN search; no application-layer drop of returned results.
- No cross-user caching of retrieval results keyed on query text alone.
- The invalidation SLO is explicit — commit to a number (e.g., ≤ 30 seconds p95 for the exercise), measure it, report it.
- The audit log must include the filter expression in a form a reviewer can read.
- The `RTBF` event must propagate through: source table, chunk index, embedding store. Write a test that proves no trace remains in either engine.
- No raw PII in the audit log.
- The test suite runs end-to-end (seed data → index → invalidate → query → assert) without manual intervention.

## Starter guidance

- Use Postgres `ROW LEVEL SECURITY` policies and `SET LOCAL app.user_branch = '...'` in a transaction to make RLS enforcement by the database concrete rather than application-layer.
- For your second engine, confirm whether it pre-filters by default and configure accordingly. Qdrant: payload index + filter. Elasticsearch / OpenSearch: filter DSL in the query. Weaviate: `where` clause.
- Do not try to represent MAC. DAC + classification labels + RLS are enough surface for this exercise.
- Keep embedding simple — any small sentence-transformers model is fine; the exercise is about permission wiring, not retrieval quality.
- For the ACL-revocation test, use a dedicated invalidation channel (e.g., a function that re-ingests the doc with updated metadata, then polls until the retriever returns the new state).
- Record the retrieval audit log to a local JSON Lines file for the exercise; in production this would be the customer's SIEM.

## Acceptance criteria

- `pytest` (or equivalent) passes all four scenario tests.
- A short write-up (≤ 500 words) answering: *How would you explain the pre-filter guarantee to a customer AppSec reviewer? What is the one scenario your test suite does not cover, and how would you add it?*
- The invalidation SLO is reported with a measured number across at least 10 revocation runs.
- The RTBF test enumerates every store the data could persist in and verifies absence.
- The audit log format matches the Chapter 03 shape (timestamp, user `sub`, pilot session id, query hash, filter expression, returned `doc_id`s, scores, timings — no raw query text, no raw PII).

## Stretch goals

- Add a **Pattern C** branch (permission-at-source retrieval): instead of carrying ACL metadata into the index, return a candidate set from retrieval and round-trip to the source to confirm authorisation per candidate. Compare latency with Pattern A.
- Add an **ABAC attribute** (`clearance_expires_at`) and a time-based filter (`now() < clearance_expires_at`). Prove a revoked user's query returns nothing without re-ingesting any chunk.
- Add an **admin-mode flag** that bypasses filters for debugging, with a hard 5-minute expiry and a loud audit event. Write a test proving the flag auto-expires.
- Model a **multi-tenancy** overlay: the same policy corpus serves two customer plans with different exceptions; implement per-tenant namespaces or filters and prove no cross-tenant leakage under a `csr_tier_1` token from each.
