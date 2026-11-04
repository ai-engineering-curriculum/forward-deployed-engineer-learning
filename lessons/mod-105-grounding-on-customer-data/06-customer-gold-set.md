# Chapter 06 — Bootstrapping a Customer-Data Gold Set for Retrieval Eval

## Why this chapter exists

Every retrieval pilot gets asked, in some form: *is it working?* The question has two levels. The lower level is retrieval quality — recall@k, precision@k, MRR, nDCG on the customer's corpus. The higher level is business value — did the AI answer eliminate a ticket, shorten the handle time, prevent an escalation. A pilot that cannot answer either level loses the budget conversation at week 8.

Generic benchmarks (BEIR, MTEB) do not answer either level. BEIR's performance on FiQA tells you nothing about the customer's policy manual. The gold set — a labelled corpus of queries and expected results drawn from the customer's actual workflow — is the only artifact that does.

The FDE's job is the bootstrap: get the first 50–200 labelled queries into a shape mod-107's eval harness can consume. The RAG engineer (Chapter 07) owns the engine-side tuning that chases nDCG; the evaluation engineer (mod-107) owns the harness; the FDE owns *what to measure* against *the customer's workflow*. This chapter walks the bootstrap.

## What a gold set looks like

Minimum shape per row:

```yaml
query_id: Q-0042
query_text: "What's the pre-auth process for outpatient MRI for a Gold plan member?"
intent: policy_lookup | member_history | precedent_cases | troubleshooting | ...
source_of_truth:
  - doc_id: doc-18422
    chunk_ids: ["doc-18422#3", "doc-18422#4"]
    relevance: 3   # 0 = irrelevant, 1 = tangential, 2 = helpful, 3 = essential
    rationale: "§4.1 names the MRI pre-auth form; §4.2 covers Gold plan exceptions."
  - doc_id: doc-18500
    chunk_ids: ["doc-18500#1"]
    relevance: 2
    rationale: "Related to outpatient imaging more broadly; useful context."
user_envelope:
  role: csr_tier_1
  groups: ["g:us-support"]
  clearance: internal
expected_grounded: true
expected_refusal: false
labeled_by: "SME-02"
labeled_at: "2026-11-04"
provenance: "support ticket #TT-55123; week 2 SME interview"
notes: "SME flagged that the §4.2 exception is often missed."
business_kpi_linkage:
  workflow_step: "PA guidance step 3 of 6"
  if_correct: "saves CSR 45s lookup"
  if_wrong: "escalation to supervisor; adds ~5 min to call"
```

Notes on the shape:

- **Graded relevance.** 0–3 (or 0–4) is the common choice. Binary is simpler but discards gradient; nDCG wants graded.
- **Rationale per label.** The SME says why this chunk is relevant. Audit trail for the label; the RAG engineer (Chapter 07) will tune against these rationales when retrieval misses.
- **User envelope.** The same query from a tier-1 CSR and a supervisor may have different expected sources. The gold set captures the permission context (Chapter 03).
- **Expected refusal.** Not every query should return an answer. Include queries whose correct behaviour is refusal.
- **Provenance.** Where the query came from — an actual ticket, an SME interview, a workflow process mining session (mod-102 Chapter 02).
- **Business-KPI linkage.** What this query is worth. Ties evaluation to pilot value.

Store as YAML / JSON / Parquet in the customer's sanctioned sandbox. Version alongside the pilot code. The gold set is a product artifact; treat it with the same review discipline as prompts.

## The core metrics

Three metrics the FDE must be able to interpret and report.

### Precision@k

Of the top-k results, what fraction are graded relevance ≥ 2 (helpful or essential)? Simple, interpretable, aligns with "when the CSR sees the top 3 chunks, how many are actually useful?" Useful for workflows that read top-k verbatim.

### Mean Reciprocal Rank (MRR)

Of the first relevant result, what rank was it at? Reciprocal rank = 1 / (position of first relevant). MRR is the mean over the query set. Appropriate for workflows where the user is looking for *the* answer and will stop at the first good hit.

Reference: [TREC Question Answering overview](https://trec.nist.gov/pubs/trec8/papers/qa_report.pdf) — the historical introduction of MRR for QA-style evaluation.

### Normalised Discounted Cumulative Gain (nDCG@k)

Weighted sum of graded relevance at each rank, discounted by log-rank, normalised by the ideal ordering. The metric to use when the pilot reads multiple chunks and the LLM must weight them. nDCG rewards both "did you find the essential chunk" and "did you put it at the top." Standard reference: [Järvelin & Kekäläinen, 2002](https://dl.acm.org/doi/10.1145/582415.582418).

Formula sketch:

```
DCG@k   = sum over i in 1..k of (2^rel_i - 1) / log2(i + 1)
IDCG@k  = same, ordered by ideal relevance ranking
nDCG@k  = DCG@k / IDCG@k
```

### Which metric when

- **Precision@k** — top-k rendered verbatim to a user, each must be useful.
- **MRR** — "is the first good source in the top few?" Useful for lookup workflows.
- **nDCG@k** — mixed-source summarisation, where chunk weight matters.
- **Recall@k** — for larger k (50, 100). "Did we at least *pass* the right chunk through retrieval for the reranker to pick up?"

Report multiple. One metric alone is manipulable.

## Precision / recall trade and refusal-aware metrics

The gold set includes queries where the correct answer is refusal (Chapter 05). Metrics must extend to measure refusal quality.

- **Refusal precision.** Of the pilot's refusals, how many were on queries labelled `expected_refusal: true`? A high value means refusals are correct; a low value means the pilot refuses when it shouldn't.
- **Refusal recall.** Of the queries labelled `expected_refusal: true`, how many did the pilot refuse on? A high value means the pilot does refuse when the corpus doesn't cover.
- **Hallucination rate.** Of the queries labelled `expected_grounded: false` (or `expected_refusal: true`), on how many did the pilot produce a grounded-looking answer with retrieved citations? This is the dangerous failure mode — a confidently wrong grounded answer.

Hallucination rate must be reported. The customer's AppSec will ask.

## The SME-paired labelling session

The core bootstrap activity. Not a crowdsourced labelling task; not an AI-pre-labelled task the SME confirms. A paired session between the FDE and the customer's SME, structured as follows.

### Before the session

Collect candidate queries. Three sources:

1. **Workflow process-mining output (mod-102 Chapter 02).** Queries the SME actually poses during the workflow, captured in the shadow / sit-with sessions.
2. **Historical ticket / case data.** Pull a representative sample of support tickets, policy-lookup logs, analyst Q&A. Anonymise under the PII contract (Chapter 04).
3. **SME-proposed hard cases.** Ask the SME in advance: "which questions would be a disaster if the AI got them wrong?" These are the eval anchors.

Aim for 80–120 candidate queries before the first session. Deduplicate to 50 representative, covering the pilot's query distribution.

### During the session

Pair the FDE with one SME at a time. 90-minute blocks, no more — labelling fatigue is real. Per query:

1. SME reads query; states their expected answer.
2. SME names the source they would consult (section, doc title). FDE finds the source and the specific chunk(s). SME confirms.
3. FDE grades relevance (SME directs; FDE records the grade and rationale).
4. If the SME would refuse ("not enough information to answer"), record as `expected_refusal: true`.
5. If the pilot can be tested against a running index now, do it — show the SME the top-k and ask "is this what you'd expect?" Record the SME's reaction as `notes`.

The pace: 15–25 queries per 90-minute session for the first session, 25–40 after the SME finds the rhythm. Two to four sessions to reach 50–100 labelled queries. The gold set is coverage-oriented — one additional session when a new workflow slice appears.

### Who the SME is

Not the executive sponsor. Not the IT lead. The actual *doer* — the CSR, the analyst, the branch support, the compliance reviewer. Mod-102 Chapter 01 names this; mod-105 depends on it. If the SME is not the person whose workflow the pilot touches, the gold set labels do not predict pilot success.

### Multi-SME agreement

For queries where SMEs might disagree (policy interpretation, ambiguous intent), label with two SMEs independently and record inter-annotator agreement (Cohen's kappa for binary, Krippendorff's alpha for graded). Low agreement is a signal the pilot's answer should hedge; the gold set makes this explicit as a `uncertain: true` flag.

## Coverage, not just volume

A gold set of 200 queries that all cover one workflow slice is worse than 50 that cover the pilot's five workflow slices. Build the gold set with coverage in mind:

- **Intent coverage.** Each labelled intent (policy_lookup, member_history, etc.) has at least 10 queries.
- **Permission coverage.** Queries from tier-1 users, tier-2 users, supervisors; queries that span classifications.
- **Freshness coverage.** Queries whose correct source is fresh (<30 days); queries whose correct source is stable (>1 year); queries whose correct source has just changed (invalidation-test queries).
- **Refusal coverage.** At least 10% of the gold set should be `expected_refusal: true`.
- **PII coverage.** Queries whose chunks contain PII that must be redacted or pseudonymised; verifies Chapter 04's pipeline.
- **Edge cases.** Typos, acronyms, abbreviations, multi-language (if the pilot is multi-lingual).

A coverage matrix the FDE publishes with the gold set — intent × role × freshness bucket × refusal — with counts in each cell. Gaps are visible; the next labelling session targets them.

## Linking to the business KPI

Mod-102 Chapter 03 committed the pilot to a business-KPI value-per-task. The gold set is the connective tissue between retrieval quality and the KPI.

Three rows to add to each query (already in the shape above):

```yaml
business_kpi_linkage:
  workflow_step: "<which step of the workflow this query serves>"
  if_correct: "<time / money / quality saved if the pilot answers correctly>"
  if_wrong:   "<time / money / quality lost if the pilot answers incorrectly or hallucinates>"
```

The eval report then rolls up: "On the labelled set, retrieval achieves nDCG@10 = 0.78, hallucination rate 2%. Projected to the workflow: on 1,200 queries/week at the labelled distribution, the pilot saves ~X hours and prevents ~Y mis-routed cases."

The specific projection logic is a model, not a measurement — but the labels power the model. Mod-107 owns the aggregation; the FDE provides the labels and the per-query linkage.

## Living with the gold set

Gold sets rot. The customer's corpus evolves; workflows change; new product lines appear. Treat the gold set as a product:

- **Review cadence.** Quarterly review with SMEs; retire queries that no longer match the workflow; add queries covering new areas.
- **Churn metric.** Report the fraction of the gold set that changed each quarter. High churn is signal that the workflow is moving.
- **Hold-out slice.** Reserve ~20% of the gold set as a hold-out the RAG engineer does not see during tuning. Report metrics on both the training slice and the hold-out. [Common ML eval practice — e.g. BEIR benchmark splits](https://github.com/beir-cellar/beir) — applied to the customer's corpus.
- **Version and sign.** Each gold-set version signed by the SMEs who labelled it. Changes pass a lightweight review — the labels affect pilot measurement, so bad labels change what "good" means.

## Three archetypes and their gold-set shapes

### Archetype A — US healthcare payer

Gold set: 120 queries after two SME sessions with tier-1 CSRs and one with a supervisor. Intents: policy_lookup (40), member_history (30), plan_comparison (20), escalation_triage (15), refusal_cases (15). PII coverage: every member_history query exercises the pseudonymisation path. Business KPI linkage: each policy_lookup is worth ~45s saved per call; each mis-answered policy_lookup is ~5 min of escalation.

### Archetype B — US federal agency

Gold set: 80 queries after three sessions with cleared policy analysts. Intents: regulation_lookup (25), precedent_memo (20), definition_lookup (15), out_of_scope_refusal (20). Classification coverage: queries from the FOUO and CUI levels both represented. Business KPI linkage: each regulation_lookup is worth ~20 min saved on the analyst's draft; each hallucinated regulation citation is a correctness failure that the records officer flags.

### Archetype C — EU retail bank

Gold set: 100 queries after two sessions with branch support and one with compliance. Intents: product_tnc_lookup (30), procedure_lookup (25), prior_case_lookup (20), regulatory_lookup (15), refusal_cases (10). Freshness coverage: queries over the most recent regulatory update represented in both the pre-update and post-update index snapshots. Business KPI linkage: each product_tnc_lookup is worth ~2 min on reply drafting; each incorrect answer that cites an outdated T&C is a regulatory risk.

## Anti-patterns

**Pre-labelling with an LLM and asking the SME to confirm.** The SME anchors on the LLM's answer; the labels regress toward whatever the pilot already returns. Pair-labelling from scratch.

**Benchmarking on MTEB / BEIR and calling it eval.** Those are embedding-model benchmarks, not retrieval-system benchmarks on the customer's corpus. Use them for component selection (Chapter 07's rag-engineer boundary); do not substitute for the customer gold set.

**Reporting only nDCG.** nDCG is a strong headline metric; it does not speak to hallucination, refusal correctness, or permission leaks. Report the full dashboard: precision@k, MRR, nDCG@k, recall@k, refusal precision/recall, hallucination rate, permission-leak count (should be 0).

**One metric moves, call the pilot shipped.** Metrics aggregated over the gold set hide slice regressions. Mod-107's slice analysis is the discipline — subsection into intent × role × freshness bucket, and look for slices where quality dropped even if the average rose.

**No hold-out.** The RAG engineer tunes to the labelled set and reports metrics on the same set; the model memorises the test. Carve out a hold-out early.

**Gold set lives in a spreadsheet nobody reviews.** It rots. Put it under version control; review quarterly; retire and refresh.

## Summary

- The customer gold set is the artifact that answers "is it working?" at both the retrieval-quality level and the business-KPI level. Generic benchmarks do not substitute.
- Each row carries query text, intent, graded source-of-truth labels with rationales, user envelope (permission context), refusal expectation, provenance, and business-KPI linkage.
- Three core metrics — precision@k, MRR, nDCG@k — plus refusal precision/recall and hallucination rate make up the dashboard.
- SME-paired labelling in 90-minute sessions; 50–200 queries covering intent × role × freshness × refusal slices.
- Business KPI linkage per row makes the metric roll-up translate to pilot value.
- Gold sets rot; review quarterly, retire and refresh; hold out ~20% from the RAG engineer's tuning.
- The archetypes show different coverage patterns but share the shape.
- Chapter 07 is next: with the gold set in hand, the boundary to `rag-engineer` opens — chunking, embedding, rerankers, multi-index fusion — and the FDE hands the retrieval-tuning depth back to the peer specialist.
