# Chapter 03 — Designing the Offline Eval — the Metric Portfolio and the Business-KPI Mapping

## Why this chapter exists

A single eval metric is manipulable. The team that reports one headline number will optimise for that number, and the pilot will ship with a regression hiding behind a cheerful average. The customer's governance board will discover the regression during the exit eval, and the budget conversation at week 10 goes badly.

The discipline the FDE owns in this chapter is the *portfolio*. Three to six metrics, each pointing at a distinct failure mode, each with a defensible threshold, each traceable to the business KPI the customer signed up for in mod-102 Chapter 03. The pilot ships only when the portfolio agrees; a single-metric headline is treated as advertising, not evidence.

Chapter 02 produced the gold set that populates each metric. Chapter 04 will produce the tooling that runs the portfolio. This chapter chooses *what* to run and *what to count as a pass*. The depth on judge-model calibration and metric methodology belongs to `ai-eval-engineer` (peer, level 30); the FDE picks the portfolio that fits the customer's workflow and defends the choice to the business owner.

## Four families of metric

Every metric the FDE will field in a customer pilot falls into one of four families. Each family catches a different kind of failure; each family misses a different kind. A portfolio uses at least two families, and usually three.

### Family 1 — Task-level correctness (reference-match)

A labelled answer exists; the pilot output is compared to it. For stringy outputs: exact-match, token-F1, BLEU, ROUGE. For classification: accuracy, precision, recall, F1. For structured outputs: JSON-schema validity, field-level exact match, numerical tolerance.

- **When it wins.** The task is closed-enough that a correct answer is well-defined. Intent classification. Named-entity extraction. Refusal-or-answer binary. Structured field extraction. Policy-lookup where the right policy section ID is the answer.
- **When it loses.** Open-ended generation. Two correct answers phrased differently will both score badly on exact-match. BLEU and ROUGE correlate weakly with answer usefulness in grounded QA — they measure n-gram overlap with a reference, not whether the answer helps the CSR.
- **Cost and signal.** Cheap to compute, deterministic, no judge-model variance. When the task fits, this family is the backbone.

### Family 2 — Judge-as-LLM (reference-free rubric scoring)

A judge-model scores an output against an authored rubric. No reference answer is required; the rubric encodes what *good* looks like. Classic shape: "Score 1–5 on correctness, with 5 = fully correct and grounded, 1 = contradicts the source."

Zheng et al., [*Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*](https://arxiv.org/abs/2306.05685) (arXiv:2306.05685), is the reference for the method and for its known failure modes. The biases the FDE must control:

- **Verbosity bias.** Judge-models prefer longer answers. A pilot whose answers grow from 80 to 180 tokens often shows a judge-score increase that is entirely verbosity.
- **Position bias.** In pairwise mode, judges prefer the option presented first. Unmitigated, swapping A and B flips the verdict for a non-trivial fraction of pairs.
- **Self-preference bias.** A judge-model from vendor X tends to rate vendor X's outputs higher. Running the judge on its own family's output is a conflict of interest that shows up in the score distribution.
- **Familiarity bias.** Judges prefer phrasings and formats they have seen often in training. Novel but correct formulations score lower than familiar-but-mediocre ones.

Mitigations the FDE builds into the design, not post-hoc:

- **Random swap for pairwise.** Each pair runs twice, with A and B in each position; the score is averaged.
- **Few-shot anchoring at both scale ends.** The rubric prompt ships with example outputs at the top of the scale and at the bottom. The judge calibrates against both.
- **Chain-of-thought rubric with separate rationale and score fields.** Force the judge to emit a rationale before the numeric score. The rationale becomes an audit record and reduces the "pick-a-number" failure mode.
- **Judge-model diversity.** Two judges from different families. Agreement is the signal; disagreement escalates to a human reviewer. The aggregator is defined upfront, not improvised when the first disagreement lands.

Judge-as-LLM is the most flexible family and the easiest to abuse. The portfolio must assume the judge is biased and design the pass-bar accordingly.

### Family 3 — Pairwise comparison

A vs B preference. Appropriate when the output is subjective — tone, actionability, customer-facing reply quality — and a graded rubric scores poorly. Humans (and judges) are better at "which is better" than at "how good is this one."

Aggregation uses Elo or Bradley-Terry models. [LMArena / Chatbot Arena](https://lmarena.ai/) is the reference framework for the method applied at scale. The FDE's version is smaller — a few hundred pairs on a hold-out slice — but the aggregation discipline is the same.

- **Ties must be a first-class outcome.** Forcing a choice between two outputs that are both acceptable distorts the ratings. The labelling UI offers "A better / B better / tie / both bad." The last two are not noise.
- **Human-friendly format.** Pairwise is the easiest format for SMEs to label at scale. Two minutes per pair on tone judgements is realistic; grading a single output on a five-point rubric is slower and noisier.
- **AI-vs-human pairs for the exit eval.** The pilot's output versus the current-process human output on the same input, scored blind. This is the pair that answers the governance board's question in Chapter 06.

### Family 4 — Factuality / groundedness / citation-accuracy

Reference-free checks on grounded answers. The question is not "does the answer match a reference" but "is each claim in the answer supported by the retrieved context, and is the citation it offers actually the right citation?"

Three concrete reference frames the FDE should know by name:

- **RAGAS triad.** Faithfulness (every claim in the answer is entailed by the retrieved context), answer relevance (the answer actually addresses the question), context precision / recall (the retrieved context is relevant and complete). See [RAGAS](https://github.com/explodinggradients/ragas) for the implementations.
- **TruLens RAG triad.** Context relevance, groundedness, answer relevance. Overlapping with RAGAS; the implementations differ in how the entailment check is composed. See [TruLens](https://www.trulens.org/).
- **Citation-accuracy.** Every factual claim in the output carries a citation; the cited chunk, when read, actually supports the claim. The check is a claim-chunk entailment pass; disagreement means the citation is decorative, not supporting.

For regulated archetypes, citation-accuracy is often the gating metric. A grounded-looking answer that cites the wrong chunk is the hallucination pattern that causes the most downstream damage because it looks correct on inspection.

## The metric portfolio

The portfolio is assembled from the four families, weighted to the workflow. One worked composition:

| Metric | Family | What it catches | What it misses | Typical pilot cost per run | Signal quality |
|---|---|---|---|---|---|
| Exact-match on intent | Task correctness | Mis-classified intents, broken routers | Nothing about the downstream answer | Very low | High when the task fits |
| F1 on structured extraction | Task correctness | Missing or wrong fields in a JSON payload | Field-ordering, prose fields | Very low | High for schema-bounded tasks |
| BLEU / ROUGE vs reference reply | Task correctness | Gross drift from an approved reply | Equivalent paraphrases; semantic correctness | Very low | Weak on open generation |
| Judge-as-LLM "correctness" | Judge-as-LLM | Factually wrong grounded answers against a rubric | Judge biases unless controlled | Medium | Medium-high with mitigations |
| Judge-as-LLM "helpfulness" | Judge-as-LLM | Technically-correct-but-unhelpful answers | Correctness itself | Medium | Medium; pairs well with correctness |
| Judge-as-LLM "policy compliance" | Judge-as-LLM | Tone, mandatory-disclaimer, out-of-scope breaches | Factuality | Medium | High when rubric is customer-authored |
| Pairwise win rate (AI vs human) | Pairwise | Subjective quality the rubric cannot pin | Why the loss happened; absolute bar | Medium-high | High for exit-eval decisions |
| RAGAS faithfulness | Factuality | Claims unsupported by the retrieved context | Claims the retrieval missed entirely | Medium | High on grounded workflows |
| Citation-accuracy | Factuality | Decorative citations pointing at the wrong chunk | Missing citations the pilot should have made | Medium | Critical for regulated archetypes |
| Refusal precision / recall | Task correctness | Over-refusal and missed-refusal cases | What the pilot would have said if it had answered | Low | High; non-negotiable in regulated archetypes |

The portfolio covers at least three failure modes: *factually wrong*, *stylistically wrong*, *refusing wrong*. A portfolio that only measures factuality lets a tone regression ship. A portfolio that only measures helpfulness lets a hallucination ship.

## Eval, metric, and business KPI — the three-column discipline

Metrics without a business-KPI mapping are vanity. The three-column table is the discipline that makes the eval defensible at the pilot-exit review:

| Eval (what we run) | Metric (what it outputs) | Business KPI (how it rolls up) |

One row per metric in the portfolio. The FDE authors the mapping; the business owner (mod-102 Chapter 03) signs it; AppSec counter-signs the factuality and refusal rows. The following three blocks show the mapping applied to each archetype. The numeric thresholds are *illustrative* — real thresholds are calibrated against the SME-labelled hold-out before the pilot signs off, as the next section describes.

<!-- needs-research: the threshold values below are illustrative; calibrate against the customer's SME hold-out before co-signing -->

### Archetype A — Mid-size US healthcare payer

| Eval | Metric | Business KPI rollup |
|---|---|---|
| Judge-as-LLM on 200 labelled member-inquiry outputs | Correctness ≥ 0.85 | Projected CSR handle-time reduction ≈ 45 s / call on policy-lookup steps |
| RAGAS faithfulness on grounded policy answers | Faithfulness ≥ 0.90 | ≤ 2 % hallucinated policy answers in sample |
| Refusal recall on `expected_refusal: true` slice | Recall ≥ 0.90 | Supervisor-escalation volume stays flat or declines |

The payer's pilot ships when all three hit the threshold on the overall set *and* on each labelled slice (see the slice section below). HIPAA posture means the hallucinated-policy-answer rate is the executive-visible number; the business owner and the AppSec reviewer co-sign.

### Archetype B — US federal agency

| Eval | Metric | Business KPI rollup |
|---|---|---|
| Citation-accuracy on policy-analyst outputs | ≥ 0.95 | Zero fabricated regulation citations in the SME-reviewed sample |
| Task correctness F1 on regulation-lookup intent | ≥ 0.80 | Analyst draft-time saved ≈ 20 min / memo |
| Refusal precision on out-of-scope queries | ≥ 0.95 | Records-officer finding rate (false-positive refusals) stays low |

The federal pilot has a stricter citation-accuracy bar because a fabricated regulation citation is a hard-fail finding for the records officer. Refusal precision is the top-priority refusal metric here — the agency can tolerate some missed answers, but a confident-sounding wrong citation is an escalation to the agency CIO.

### Archetype C — EU retail bank

| Eval | Metric | Business KPI rollup |
|---|---|---|
| Pairwise AI-vs-human on 300 held-out case replies | Win-or-tie rate ≥ 0.70 | Case-handling time saved with no quality regression on blind SME review |
| Judge-as-LLM "regulatory tone" on draft replies | ≥ 0.90 | Compliance-reviewer flagged-reply rate stays at or below current baseline |
| Refusal recall on queries requiring human escalation | ≥ 0.90 | DORA incident-trigger rate unchanged |

The bank's exit eval is pairwise-first because GDPR Article 22 and the EU AI Act (Article 14) push the governance conversation toward "is this at least as good as the human for a human-in-the-loop setup," not "is this above an absolute threshold." The pairwise win-or-tie rate becomes the governance board's headline.

## Calibrating the pass bar

Thresholds are never read off a chart in a vacuum. The FDE runs a calibration pass on the SME-labelled hold-out before the pilot kicks off:

1. **ROC-curve analysis for judge-based metrics.** Sweep the threshold across the possible range. At each threshold, measure the judge's agreement with the SME label as true-positive and false-positive rates. Pick the threshold that matches the customer's asymmetric cost of error — if a false-negative (shipping a bad answer) costs 10 × a false-positive (holding a good answer), the threshold sits toward the high-precision side of the curve.
2. **Business-cost-of-error weights.** Reuse the value-per-task figures from mod-102 Chapter 03. A payer pilot with a 5-minute escalation cost per mis-answered policy lookup sets a different threshold than a bank pilot with a regulatory-tone miss that triggers a compliance review. The weighted cost is the objective function.
3. **Hallucination tolerance per customer.** The federal agency's tolerance for fabricated regulation citations is effectively zero — any fabricated citation in the eval sample is a stop-ship finding. The payer tolerates low single-digit percentage hallucination on the draft-only phase, with a path to lower as the pilot matures. The bank's tolerance is typically ≤ 1 % on grounded answers.
4. **Co-sign ritual.** Each threshold is initialled in the eval design doc by the business owner *and* the AppSec reviewer before the pilot runs. The threshold cannot change mid-pilot without re-signing. This is the artifact that keeps the exit-eval conversation from devolving into "well, the number was meant to be ..." in week 10.

The output of the calibration is not just a number; it is a short paragraph per threshold explaining *why* this number. "0.85 because at this threshold the judge's agreement with SME-01 and SME-02 is κ = 0.73, and the false-negative cost (a mis-answered policy lookup) is weighted 8× the false-positive cost (holding a correct answer for review)."

## Slice analysis is not optional

Overall metric values hide slice regressions. The gold set's coverage matrix (mod-105 Chapter 06) defined slices across intent × role × tenant × freshness. The eval reports against those slices.

A metric averaging up with a regression on an important slice is NOT a pass. This is the single discipline that most often flips a go into a no-go, and the one most tempting to skip when the overall number looks good.

The slice table has one row per slice, one column per portfolio metric. The entries are the metric value on that slice plus the delta from the prior pilot version. An example pattern from a payer pilot:

| Slice | Correctness | Faithfulness | Refusal recall |
|---|---|---|---|
| policy_lookup / tier-1 CSR / fresh | 0.89 (+0.02) | 0.93 (+0.01) | 0.95 (±0) |
| policy_lookup / tier-1 CSR / stale | 0.86 (+0.01) | 0.91 (+0.02) | 0.92 (−0.01) |
| member_history / tier-1 CSR / fresh | 0.83 (+0.03) | 0.89 (±0) | 0.91 (±0) |
| escalation_triage / supervisor / fresh | 0.74 (−0.08) | 0.86 (−0.04) | 0.88 (−0.03) |

Overall averages rose in this fictional run. The `escalation_triage / supervisor / fresh` slice regressed on every metric. The pilot does not ship on this result; the FDE walks the regression back to the gold-set labels, the prompt change, or the retrieval index diff that caused it.

The slice-pass rule lives in the portfolio config: `fail_if_any_slice_below(threshold - margin)` is the common shape, where `margin` is the acceptable gap between overall and worst-slice. Zero margin is strict; a 0.1 margin is permissive. Pick with the customer.

## A portfolio config the harness will consume

Chapter 04's eval harness runs against a YAML artifact under version control. One shape that has held up across pilots:

```yaml
portfolio_version: v2.1
gold_set_version: v1.3
metrics:
  - id: correctness_judge
    family: judge_as_llm
    judge_model: claude-sonnet-4-6
    rubric_prompt: rubrics/correctness.md
    aggregator: mean
    threshold: 0.85
    slice_rule: fail_if_any_slice_below(0.75)
    bias_controls:
      random_swap: true
      few_shot_anchors: rubrics/correctness_anchors.yaml
      rationale_before_score: true
      second_judge: gpt-4.1
      disagreement_review: human_queue
  - id: faithfulness
    family: factuality
    implementation: ragas.faithfulness
    threshold: 0.90
    slice_rule: fail_if_any_slice_below(0.85)
  - id: citation_accuracy
    family: factuality
    implementation: custom.claim_chunk_entailment
    entailment_model: claude-sonnet-4-6
    threshold: 0.95
    slice_rule: fail_if_any_slice_below(0.92)
  - id: refusal_recall
    family: task_correctness
    threshold: 0.90
    scope: slice.expected_refusal_true
  - id: refusal_precision
    family: task_correctness
    threshold: 0.95
    scope: slice.model_refused_true
  - id: pairwise_vs_human
    family: pairwise
    reference: current_process_human_output
    aggregator: bradley_terry
    threshold_win_or_tie: 0.70
    tie_as_first_class: true
    scope: holdout.exit_eval_only
slice_axes: [intent, role, tenant, freshness]
sign_off:
  business_owner: pending
  appsec: pending
  ai_eval_engineer_consult: optional
```

The config is the single source of truth. The eval harness reads it; the eval report cites it; the pilot-exit decision document references the version hash. Changing a threshold means bumping `portfolio_version` and re-signing.

## Anti-patterns

**Single-metric headline.** The number goes up; nobody asks why. One metric is a direction of travel at best and a misdirection at worst. Report the portfolio; make the headline a table, not a scalar.

**Judge-LLM without bias controls.** The scores look great until the judge is swapped for a different family and the ranking flips. Verbosity, position, self-preference, and familiarity biases are known; mitigate them upfront.

**Pairwise with no tie handling.** Forced choice distorts the Elo / Bradley-Terry ratings and inflates apparent differences. Ties are not noise; they are the signal that two outputs are both acceptable.

**Ignoring slice regressions.** The average improves. The release ships. Two weeks later the escalation_triage intent silently breaks and the supervisors notice before the dashboards do. Slice analysis is the discipline that prevents this.

**Thresholds authored in a vacuum.** The FDE picks thresholds that look reasonable; the customer never signs them; at the exit eval the business owner disputes the bar. The co-sign ritual before the pilot runs is cheap and resolves the dispute class entirely.

**Treating the portfolio as static.** The pilot learns things in weeks 3–6 that reshape the risk profile. Rev the portfolio version, re-sign, record the delta. A frozen portfolio becomes irrelevant; an unversioned one becomes incoherent.

## Summary

- A single-metric headline is advertising, not evidence. The portfolio of three to six metrics across at least two of the four families is the FDE's defensible shape.
- Four families each catch a different failure: task-level correctness (deterministic outputs), judge-as-LLM (rubric scoring with controlled biases), pairwise comparison (subjective quality, easiest to label), factuality / citation-accuracy (grounded answer verification).
- Judge-as-LLM biases — verbosity, position, self-preference, familiarity — are known and must be mitigated in the design, not post-hoc.
- The three-column mapping (eval → metric → business KPI) is the discipline that makes the portfolio defensible to the governance board at the pilot exit.
- Thresholds are calibrated per customer against the SME-labelled hold-out; the ROC-curve, the business-cost-of-error, and the hallucination tolerance set the bar. The business owner and AppSec co-sign before the pilot runs.
- Slice analysis is the single discipline that most often flips a pilot from a go into a no-go. Overall averages hide slice regressions; the portfolio config encodes the slice-pass rule.
- The portfolio config is a versioned YAML artifact the Chapter 04 harness runs against; the Chapter 06 exit eval references it by version hash.
- Methodology-depth decisions (judge-prompt design, Bradley-Terry parameterisation, bias-control statistics) escalate to `ai-eval-engineer` — Chapter 07 defines the boundary.
- Chapter 02 produced the gold set that fills the portfolio; Chapter 04 is next, with the tooling that executes it under customer-compliance constraints.
