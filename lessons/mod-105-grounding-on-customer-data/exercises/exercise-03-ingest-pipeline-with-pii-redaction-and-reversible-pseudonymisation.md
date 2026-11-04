# exercise-03: Ingest Pipeline with PII Redaction and Reversible Pseudonymisation

**Estimated effort:** 3 hours

## Objective

Build an ingest pipeline that lands a corpus into a retrieval index with a signed PII contract enforced at all four boundaries (ingest, retrieval, output, audit). Deliver the contract document, the pipeline code, and a test suite that proves no raw PII leaks through any boundary — including embeddings.

## Prerequisites

- Read [Chapter 04](../04-pii-safe-retrieval.md).
- Install [Presidio](https://microsoft.github.io/presidio/) (`presidio-analyzer`, `presidio-anonymizer`) and a sentence-transformers model for embeddings.
- Local Postgres + pgvector (or reuse the engine stack from exercise-02).
- A small key-management surface — HashiCorp Vault dev mode, or AWS KMS / Azure Key Vault / GCP Secret Manager if you have free-tier access. A pure-local HMAC with a filesystem-mounted key is acceptable for the exercise.

## Problem statement

You are bootstrapping retrieval for the EU retail bank archetype. The source corpus is a ~2,000-row export of customer support cases (CSV or JSONL) with free-text notes that contain:

- Customer names, emails, phones, addresses, DOBs, IBANs, account numbers, national IDs.
- Occasional embedded credit-card numbers or SSNs (shouldn't be there — treat as security findings).
- References to account states, product codes, and procedure IDs that are **not** PII but may look superficially like identifiers.

Deliver:

1. A signed **PII contract** (one page) covering entity types, treatment per entity type at each boundary, reversibility design, audit shape, RTBF SLO.
2. An **ingest pipeline** that: detects PII with Presidio + at least one custom recogniser for a bank-specific entity type you invent (e.g., `CASE_NUMBER` with a specific pattern); applies the contract's transforms per entity; stores a reversible pseudonym mapping in a vault; emits an audit event per decision; writes chunks with no raw PII into the retrieval index.
3. A **retrieval-time sanitiser** that re-runs detection against returned chunks and enforces the per-user clearance from the token.
4. An **output sanitiser** (post-LLM) that catches any entity that re-materialises in the response.
5. A **rehydration endpoint** that, on authenticated authorised call, returns the real value for a pseudonym and emits an audit event.
6. An **RTBF handler** that, given a customer identifier, removes the subject's data from the source, the index, the embedding store, and the pseudonym vault within an SLO you commit to.
7. A **test suite** that proves: no raw PII reaches the index; no raw PII reaches the embeddings (verify by recomputing embedding of the raw text vs. the redacted text and confirming the index stores the redacted-text embedding); the output sanitiser catches entities the model attempts to echo; the RTBF handler leaves no trace in any store.

## Requirements

- **Never embed raw PII.** The redaction happens before the embedder; write a test that explicitly compares the raw and redacted vectors and proves the redacted one is what's in the index.
- The audit log must never contain a raw PII value. Record entity type, span, confidence, pseudonym, decision. Log to a JSONL file (SIEM in production).
- Pseudonyms must be deterministic for the same real value (coalescing) and non-sequential (no `[PERSON-0001]` → `[PERSON-0002]` ordering that leaks ordering).
- The vault is a separate store with its own access controls; the retrieval path must not have read access to the mapping.
- A credit-card or SSN finding emits a **security alert**, not just a redaction — include this alert channel in the pipeline.
- The RTBF handler's SLO is explicit (e.g., ≤ 30 minutes for the exercise). Measure across at least 5 runs.
- Precision and recall of the PII detection on a labelled sample (provide ~50 hand-labelled lines with ground-truth spans) must be reported.

## Starter guidance

- Start with Presidio's default recognisers. Add a custom recogniser using a regex + context-word boost for your `CASE_NUMBER` entity — see [Presidio custom recogniser docs](https://microsoft.github.io/presidio/analyzer/adding_recognizers/).
- Pick a locale (EU-centric) and tune Presidio for it — EU phone / postal / national-ID shapes differ from US defaults.
- For deterministic pseudonymisation, HMAC-SHA256 with a per-entity-type key derived via HKDF ([RFC 5869](https://datatracker.ietf.org/doc/html/rfc5869)) is sufficient for the exercise. Use a URL-safe short prefix: `[PERSON-7f3a9b]`.
- For format-preserving pseudonymisation (e.g., keeping a 10-digit phone number as 10 digits), [NIST SP 800-38G](https://nvlpubs.nist.gov/nistpubs/specialpublications/NIST.SP.800-38G.pdf) describes FF1; cloud managed offerings have FPE primitives. Optional for the exercise.
- For the output sanitiser, run Presidio on the model's response. The honest trick: if an entity appears in the response that is not in any chunk the user was shown as a pseudonym, treat it as a hallucination and redact.
- For the test comparing raw vs. redacted embeddings, embedding cosine similarity should differ meaningfully for texts with PII; the point is to prove the index stores the redacted-text embedding — compute both, store the redacted, assert equality with the stored vector.
- The gateway in mod-104 Chapter 03 is where the output sanitiser would live in production. For this exercise, a module in the pilot is fine.

## Acceptance criteria

- PII contract document delivered (one page, filled-in sign-off block is fine as placeholder).
- All seven pipeline pieces implemented.
- `pytest` passes the full suite; precision ≥ 0.90, recall ≥ 0.85 on the labelled sample (adjust the labelled sample to make this achievable).
- Audit log contains no raw PII; a grep for any sample PII value across all pipeline outputs returns zero.
- The RTBF test enumerates every store (source CSV, index, embedding store, pseudonym vault, audit log retention policy) and verifies erasure or retention justification.
- Rehydration test: unauthorised caller is denied; authorised caller receives the real value and an audit event is emitted.
- Short write-up (≤ 500 words) answering: *Where would your pipeline fail if the customer added a new entity type mid-pilot? What is your re-ingest / incremental-detect strategy?*

## Stretch goals

- Add a **classification pass** at ingest that assigns each chunk a sensitivity label (`public`, `internal`, `restricted`) based on detected entity density and composes with the Chapter 03 permission filter.
- Implement a **cross-chunk coalescing report**: for each pseudonym, how many chunks across the corpus mention it? Build a small view and use it to demonstrate the value of reversible pseudonymisation vs. scrubbing.
- Add **drift detection**: track the detection-rate per entity type over time; alert when a new type's count spikes (possibly a new source or a bug).
- Re-run the entire pipeline against a 10× larger synthetic corpus and measure latency / cost per 1000 docs. Report the shape of the growth curve.
