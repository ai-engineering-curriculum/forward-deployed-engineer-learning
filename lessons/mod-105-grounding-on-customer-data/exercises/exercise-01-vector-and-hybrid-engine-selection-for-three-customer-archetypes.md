# exercise-01: Vector and Hybrid Engine Selection for Three Customer Archetypes

**Estimated effort:** 2 hours

## Objective

Produce a defensible retrieval-engine selection matrix and one-page ADR for each of the three customer archetypes introduced in Chapter 01 (US healthcare payer, US federal agency, EU retail bank). The output is the kind of artifact an FDE would hand to a customer AppSec reviewer and a customer platform team in week 2–3 of a pilot.

You will choose *among* Pinecone, Weaviate, Qdrant, Chroma, pgvector, Elasticsearch (or OpenSearch), Vespa, Azure AI Search, and Vertex AI Search, scoring each against the six-dimensional budget from Chapter 01.

## Prerequisites

- Read [Chapter 01](../01-grounding-overlay.md) and [Chapter 02](../02-retrieval-engine-selection.md).
- Skim the primary docs for at least three candidate engines you have not previously used — the vendor compliance / trust page is often the most relevant section.
- Have the ADR template from mod-104 Chapter 02 to hand; this exercise's ADR mirrors its shape.
- Optional: read mod-102 Chapter 04 on the pilot-scope ADR to see how customer context flows into decision docs.

## Problem statement

For each of the three archetypes:

- **Archetype A** — Mid-size US healthcare payer. AWS cloud of record. HIPAA + state privacy. ~500k-doc corpus (policies, plan docs, prior auths, call-centre notes). Growth ~10k/week. Query rate 2 QPS p50 / 10 QPS p95. Retrieval slice latency SLO ~300 ms p95. RBAC (3 tiers) + DAC on sensitive items.
- **Archetype B** — US federal agency. Must run in GovCloud. FedRAMP Moderate baseline, path to High. ~200k-doc corpus (policy memos, regulations, FOIA archives). Low growth. Query rate ~1 QPS p95. Latency slice ~500 ms p95. ABAC + classification labels (FOUO, CUI).
- **Archetype C** — EU retail bank. Azure cloud of record under the EU Data Boundary. GDPR + DORA. ~300k-doc corpus (product T&Cs, procedures, prior case notes, regulatory updates). Growth ~20k/week on case notes. Query rate 3 QPS p50 / 15 QPS p95. Latency slice ~300 ms p95. Per-branch / per-region DAC + GDPR right-to-be-forgotten.

Produce the selection and the ADR as if you were the FDE on the pilot.

## Requirements

For each archetype:

1. Populate the scoring matrix from Chapter 02 (nine engines × six dimensions) with 1–5 scores. Mark 0 for any gate failure (residency, compliance, permission fidelity).
2. One-line rationale per non-zero cell, citing a primary source (the engine's own docs, compliance page, pricing page, or the customer's regulatory reference). For every citation, include the URL so the ADR reader can re-verify.
3. Choose weights per dimension for this customer, and defend the weights in 2–4 sentences at the top of the matrix.
4. Pick a primary engine and name one alternative. If the alternative lands close on your scoring, explain what would tip the choice to it.
5. Write a one-page ADR per archetype using the Chapter 02 template.
6. In an appendix, list any claim you could not verify from primary sources — flag each with `<!-- needs-research: ... -->` so a reviewer can backfill.

## Starter guidance

- Residency, compliance, and permission fidelity are gates. Any engine that fails must score 0 on that dimension and be excluded from scoring on the remaining dimensions — do not try to "average through" a gate failure.
- Prefer AWS / Azure / GCP native options when the cloud of record matches; the review path is shorter. That is not a rule, only a prior.
- For permission fidelity: trace the customer's actual permission system (Microsoft Entra / Google Workspace / in-house IdP) into the engine's filter model. If the engine does not pre-filter, downgrade that cell.
- For self-hosted candidates, recognise that the operational cost is not on the engine's pricing page — ask the customer's platform team.
- For hyperscaler-native engines (Azure AI Search, Vertex AI Search), check the specific service's residency and compliance pages (Azure Service Trust Portal, Google Cloud compliance reports) — the hyperscaler's overall compliance is not the service's.
- Do not benchmark-shop. Model-quality benchmarks (MTEB, BEIR) do not determine engine selection; they inform downstream embedder choices.
- Some facts (regional availability, plan-tier scope of HIPAA BAAs, FedRAMP authorization) change often — use `<!-- needs-research: ... -->` liberally rather than guessing.

## Acceptance criteria

- Three complete scoring matrices, one per archetype, dated.
- Three one-page ADRs, each with: context, decision, alternatives considered, consequences, gate checks, sign-off block (unfilled is fine for a learning exercise).
- Every non-zero score cell has a one-line rationale with a primary-source URL.
- Weights are defended per archetype in writing.
- Gate failures are explicit — a 0 on a gate dimension disqualifies the engine; it does not get summed into a total.
- Every unverifiable claim is tagged `<!-- needs-research: ... -->` with enough detail that a reviewer can resolve it.
- The three choices show genuine variation across archetypes — the exercise is wrong if you landed on the same engine for all three without explicit justification.

## Stretch goals

- Add a *second-year* column to each matrix that scores what changes when the pilot scales from pilot-size traffic to the customer's full user base (5–10× queries; 2× corpus).
- Write a one-paragraph note per archetype on *when you would reopen the decision*: what signal (metric, cost, residency change, M&A event) would cause you to re-ADR.
- Explore combining engines: e.g., pgvector for structured-store-adjacent chunks + an inverted-index engine for free-text chunks, with multi-index fusion. Is the complexity worth the gain? Capture in an appendix.
- For Archetype B, trace whether Azure AI Search inside Azure Government or Vertex AI Search inside Assured Workloads would qualify — and what documentation you would need to confirm it.
