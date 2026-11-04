# Chapter 05 — Online Eval on Production Customer Traffic — Judge-as-LLM at Sample Rate, Implicit Feedback, and Drift

## Why this chapter exists

The offline gold set is frozen in time. It was labelled by the customer's SMEs on a specific week, against a specific case distribution, under a specific set of policies. Customer workloads drift. New products ship. Compliance rewrites a policy section. A new call-centre cohort shifts the intent mix. A regulator changes a cite format. The gold set does not move, but the world the pilot answers against does.

The offline portfolio (Chapters 02–04) is the *pre-ship gate*: the pilot does not go to the next rollout stage until the portfolio passes. Online eval is the *post-ship watch*: the pilot does not keep its production slot unless live traffic keeps matching the shape the gold set measured. Together they are the two halves of the quality loop. Neither half is sufficient alone.

This chapter walks four components of the online loop: judge-as-LLM at a sample rate; implicit user-feedback capture; the per-tenant quality dashboard; and drift detection across four layers. Then alerting, privacy posture, and application to the three archetypes.

## Judge-as-LLM at sample rate

Running the judge on every production call is expensive and unnecessary. The judge gives a stable signal from a well-designed sample. Sample rate is a function of two things:

- **Workload volume.** A pilot handling 500k calls/day cannot afford a 10% judge pass; a pilot handling 2k/day can comfortably.
- **Judge cost.** Judge-LLM inference is itself a cost line. The FDE picks the rate so the judge's monthly cost is a small fraction of the primary model's.

Typical ranges in the field: **1–10% sample rate**, picked per workload. Below 1% the signal gets too noisy on small slices; above 10% the cost rises without proportional signal gain.

### Stratification

A naive uniform sample under-represents the slices that matter most. Stratify the sample across intent, role, tenant, and any axis the gold set already partitions on. The low-volume slices (rare intents, senior-role sessions, specific tenants) need a *higher* in-slice sample rate so each slice gets statistically useful numbers.

```python
# Per-(tenant, intent) sample rate table maintained alongside the gold set.
SAMPLE_RATES = {
    ("payer-a", "claim_inquiry"): 0.05,
    ("payer-a", "prior_auth_appeal"): 0.25,   # rare + high-stakes → oversample
    ("payer-a", "benefits_lookup"): 0.02,
    # ...
}

def should_judge(trace):
    rate = SAMPLE_RATES.get((trace.tenant, trace.intent), 0.02)
    return deterministic_bucket(trace.id) < rate

if should_judge(trace):
    enqueue_for_judge(trace)   # async; do not block the user response
```

`deterministic_bucket(trace.id)` is a stable hash into `[0, 1)`. Determinism matters for two reasons: the same trace does not get re-judged on retry; and the sample is reproducible when the FDE investigates an incident.

### Judge pipeline shape

Four rules:

1. **Asynchronous.** The judge call is on a queue, not in the user's request path. Online eval must never degrade user latency.
2. **Same judge prompt as offline.** The calibrated judge from Chapter 04 is reused directly. If the offline and online judges diverge, the online number is not comparable to the pre-ship gate.
3. **Same privacy posture as the primary call.** The judge runs inside the customer's trust boundary, on a zero-retention, no-training endpoint. The judge sees the same trace content the primary model did; if the primary model is subject to Bedrock Model Invocation inside a VPC (Archetype A), the judge is too.
4. **Trace + judge-score stored as a pair.** The judge score is written back onto the trace record so a single query against the trace store returns both.

### Sampling granularity and cost ceiling

Per-tenant and per-intent caps avoid runaway cost when one slice spikes. A simple ceiling pattern:

```python
# Daily cap in judge calls, per (tenant, intent).
JUDGE_CAPS = {
    ("payer-a", "claim_inquiry"): 2000,
    ("payer-a", "prior_auth_appeal"): 400,
}

if judge_count_today(trace.tenant, trace.intent) >= JUDGE_CAPS[...]:
    skip_judge(trace)
```

The cap is not the sample rate. The cap is a budget-safety floor underneath it.

## Implicit user feedback capture

Judge-as-LLM measures the pilot against itself. Implicit feedback measures the pilot against the user. Four signals every pilot should emit:

| Signal | What it means | Coverage | Primary use |
|---|---|---|---|
| **Accept** | User used the suggestion unchanged | Varies by workflow; usually 20–60% | Quality top-line |
| **Edit** | User used the suggestion after modification | Usually 30–60% | Quality nuance — how much rework |
| **Discard** | User did not use the suggestion | Usually 10–40% | Quality bottom — the miss rate |
| **Thumbs up/down** | Explicit rating from the UI | Usually < 1% of traffic | Supplementary; qualitative colour |

Thumbs are the one most teams reach for first. They are the least useful. Coverage is tiny; the users who click thumbs are a biased population (the very happy and the very frustrated); the signal is slow. Treat thumbs as commentary, not metric. The accept/edit/discard funnel is the single most actionable online signal the pilot has.

### Edit-distance capture

"Edit" is more informative than "accept" or "discard" because the edit distance tells the FDE *how much* the user changed. Two normalised measures work:

- Python `difflib.SequenceMatcher(a, b).ratio()` — simple, well-known, returns `[0, 1]`.
- Normalised Levenshtein distance — `1 - levenshtein(a, b) / max(len(a), len(b))`.

A distribution of ratios across edits tells you whether users are nudging commas (ratio ≈ 0.95, pilot is fine) or rewriting whole paragraphs (ratio ≈ 0.4, pilot is draft-only in practice).

**Capture the ratio, not the user's rewrite content**, unless the DPA covers it. The rewrite is almost always PII-rich (Archetype A: a CSR's reply embeds member names, claim numbers, clinical fragments). The ratio is a single float; it leaks nothing.

### Feedback event schema

```json
{
  "event_id": "fb_01HN...",
  "trace_id": "trc_01HN...",
  "tenant_id": "payer-a",
  "user_id_hash": "sha256:...",
  "role": "csr_tier1",
  "intent": "claim_inquiry",
  "suggestion_id": "sug_01HN...",
  "outcome": "edit",
  "edit_ratio": 0.82,
  "latency_to_outcome_ms": 11420,
  "explicit_rating": null,
  "ts": "2026-11-04T14:03:21.221Z"
}
```

Three notes on the schema:

- `user_id_hash`, not `user_id` raw. The pilot does not need to know who the user is to compute funnel metrics; it needs to be able to group by a stable identifier. Mod-108 governs the hashing scheme.
- `latency_to_outcome_ms` matters. If the user accepts after 300ms they probably did not read the suggestion. If they discard after 20s they considered it and rejected.
- `outcome` is enumerated, not freeform. Prevents the downstream dashboard from guessing.

The feedback event joins to the trace record by `trace_id`. The joined row is the primary dataset for the per-tenant dashboards below.

## Per-tenant quality dashboards

Every tenant gets its own dashboard view. Tenants do not share dashboards; a shared view hides the tenant-level drift that matters. The view holds:

| Panel | Source | Refresh |
|---|---|---|
| Offline portfolio last-run | CI eval job (Ch 04) | On merge to main |
| Online judge-score, rolling 7d / 30d | Judge pipeline output | Hourly |
| Implicit feedback funnel (accept / edit / discard %) | Feedback events | Hourly |
| Edit-ratio distribution | Feedback events | Daily |
| Drift metrics (PSI / KL on input + output) | Drift job | Daily |
| SLO breaches (latency, error, refusal) | mod-104 Ch 05 + mod-110 | Live |
| Golden-trace replay status | Replay job | Nightly |

The dashboard lives inside the customer's observability stack (mod-110), not the vendor's. Candidate tools:

- **Grafana + Prometheus** (or an in-boundary tsdb) when the customer already runs Grafana. Most VPC installs land here.
- **Datadog** where the customer's DPA explicitly covers shipping telemetry to Datadog SaaS. Not the default for regulated customers.
- **Metabase** against the customer's warehouse (Snowflake, BigQuery, Redshift, Databricks SQL) when the feedback events and judge scores already land in the warehouse.
- **Azure Monitor / Log Analytics** for Azure-native deploys (Archetype C's default).
- **Elastic / Kibana** or **Splunk Enterprise** when the customer's SIEM stack doubles as the observability stack (common in Archetype B).

### Daily rollup SQL shape

A per-tenant daily rollup that backs most of the panels:

```sql
-- Per-tenant daily online-eval rollup.
WITH judged AS (
  SELECT
    t.tenant_id,
    t.intent,
    DATE_TRUNC('day', t.ts) AS day,
    AVG(j.score) AS mean_judge_score,
    COUNT(*) AS judged_n
  FROM traces t
  JOIN judge_scores j USING (trace_id)
  WHERE t.ts >= CURRENT_DATE - INTERVAL '30 days'
  GROUP BY 1, 2, 3
),
feedback AS (
  SELECT
    tenant_id,
    intent,
    DATE_TRUNC('day', ts) AS day,
    SUM(CASE WHEN outcome = 'accept' THEN 1 ELSE 0 END)::FLOAT
      / COUNT(*) AS accept_rate,
    SUM(CASE WHEN outcome = 'edit' THEN 1 ELSE 0 END)::FLOAT
      / COUNT(*) AS edit_rate,
    SUM(CASE WHEN outcome = 'discard' THEN 1 ELSE 0 END)::FLOAT
      / COUNT(*) AS discard_rate,
    AVG(CASE WHEN outcome = 'edit' THEN edit_ratio END)
      AS mean_edit_ratio,
    COUNT(*) AS feedback_n
  FROM feedback_events
  WHERE ts >= CURRENT_DATE - INTERVAL '30 days'
  GROUP BY 1, 2, 3
)
SELECT *
FROM judged
FULL OUTER JOIN feedback USING (tenant_id, intent, day)
ORDER BY tenant_id, intent, day DESC;
```

The query is illustrative, not exhaustive. Add refusal rate, cite-count-per-answer, and latency percentiles alongside.

## Drift detection — four layers

Drift is not a single metric. The pilot drifts in four distinct ways, each with its own detector and its own action.

### 1. Input drift

A shift in the distribution of what users are asking. Detectable without any model output. Features to monitor:

- Intent-classifier distribution over a rolling window.
- Topic / entity distribution from a cheap topic extractor.
- Freshness-bucket distribution (how old are the sources retrieval is finding — if the pilot is suddenly answering more "what did we say in 2019" questions, retrieval load has shifted).
- Query-length distribution.

**PSI (Population Stability Index)** is the standard measure. For a feature discretised into bins, PSI between the current window `C` and the baseline window `B` is:

```
PSI = Σ_i  (C_i - B_i) * ln(C_i / B_i)
```

where `C_i` and `B_i` are the proportion of observations in bin `i`. Common thresholds (as used in credit-risk model monitoring): **PSI < 0.1 stable; 0.1–0.25 moderate drift; > 0.25 significant drift.** These bands are conventional — originally popularised in credit-risk / scorecard monitoring — and FDEs cite them as such. <!-- needs-research: trace the earliest widely-cited publication of the 0.1 / 0.25 PSI thresholds; several model-risk-management handbooks use them without a canonical primary source. -->

Alternative: **KL divergence** `D_KL(C || B) = Σ_i C_i * ln(C_i / B_i)`. KL is more sensitive to small shifts in rare bins; use it when the slices under 1% matter (rare intents, high-stakes workflows).

For continuous features without a natural binning, use the **Kolmogorov-Smirnov** two-sample test. KS compares the empirical CDFs of the baseline and current windows; the test statistic is the maximum absolute difference between the CDFs.

### 2. Output drift

The distribution of the pilot's *responses*, independent of whether they are right. Features:

- Refusal rate (the pilot's rate of "I don't have grounds" responses — Chapter 05 of mod-105).
- Mean / median confidence score the pilot emits (if any).
- Mean answer length in tokens.
- Mean citation count per answer.
- Tool-call frequency per session (for agentic pilots).

Monitor each as a time series. Alert on z-score deviation from the baseline window — a z of 3 over the baseline means the metric is at an extreme tail relative to its own history.

Output drift is almost always *not* caused by workload change alone. It usually means a model version shifted, a prompt shifted, a retrieval index changed, or a downstream tool changed. Output drift without input drift is an infrastructure signal.

### 3. Quality drift

The actionable signal. Two quality drifts:

- **Online judge score** moving relative to the baseline window. The gold-set-calibrated judge now scoring the pilot lower.
- **Accept rate** (from the implicit feedback funnel) moving. Users accepting less often.

Either, independent of input drift, says the pilot got worse at the same job. Either, co-moving with input drift, says the workload shifted into territory the gold set did not cover and the pilot is weaker on. Both branches require action; the branches differ in whether the gold set needs re-sampling (Chapter 06) or the pilot needs re-engineering.

### 4. Embedding drift

A surface-level canary. Compute cosine similarity between a rolling embedding-centroid of recent inputs and the embedding-centroid of the gold set's inputs. Shifts in this cosine are a "something is different" smoke alarm.

Embedding drift is not conclusive on its own — two very different corpora can have similar embedding centroids, and two similar corpora can drift in embedding space because the embedding model itself was re-versioned. Treat it as triage input alongside the other three.

### Partitioning the drift view

A single "is the pilot drifting" boolean is not useful. The dashboard panel shows all four layers side-by-side, per tenant:

| Layer | Feature | This window | Baseline | Measure | State |
|---|---|---|---|---|---|
| Input | intent distribution | — | — | PSI | 0.08 → stable |
| Input | query length | — | — | KS | p=0.03 → moderate |
| Output | refusal rate | 4.1% | 2.7% | z-score | z=3.8 → alert |
| Output | cite count | 2.9 | 3.0 | z-score | z=-0.4 → stable |
| Quality | judge score mean | 0.71 | 0.78 | delta | -0.07 → alert |
| Quality | accept rate | 46% | 53% | delta | -7pp → alert |
| Embedding | input-centroid cosine | 0.91 | 1.0 | cosine delta | -0.09 → attention |

A pattern is only clear with the full table. Refusal spiking + accept rate dropping + judge score dropping, with stable input PSI, is "the pilot got worse" — probably a model or retrieval change. The four layers are a differential.

## Golden-trace replay

A small canonical set of traces (20–50), hand-curated from the gold set and augmented with any production incidents the pilot has resolved, replays nightly through the production pipeline. The same judge from Chapter 04 scores each. The replay result is a pass / fail table.

Replays catch silent regressions that the sampled online judge is too noisy to catch quickly: a model-version bump the vendor pushed; a retrieval index rebuild with a bad embedding; a tool-side schema change that breaks a specific intent.

```yaml
# golden_trace_replay.yaml
schedule: "0 3 * * *"              # 03:00 UTC nightly
dataset: gold_set/replay_v12
dataset_sha: "a9f0c1..."           # pinned; changes are PRs
target:
  pilot_env: prod
  model_version: ${PROD_MODEL}     # tracks the live-deployed version
  retrieval_index_version: ${PROD_INDEX}
judge:
  prompt: judges/customer_service_v3
  prompt_sha: "7b23a0..."
  temperature: 0.0
pass_criteria:
  per_case: judge_score >= 0.7
  overall: pass_rate >= 0.95
on_fail:
  page: oncall_rotation_fde_mod110
  block_rollout_label: "quality-regression"
retention_days: 180
```

The replay runs against the *live production stack* — not a staging environment. The point is to detect that production, as deployed right now, still handles cases the pilot used to handle correctly. When a replay case that passed yesterday fails tonight, something in production changed.

Two disciplines:

- **Pin the dataset.** The replay set changes only by PR. A silently-shifting replay set hides regressions inside itself.
- **Version the judge prompt.** The judge the replay uses must match the judge the offline portfolio uses. Diverging judges mean the online pass/fail is not comparable to the offline portfolio.

## Alerting

Alerting separates noisy from actionable. The question for every alert is: "If this fires at 03:00, is there a specific thing the on-call is supposed to do in the next hour?" If not, it is a ticket, not a page.

| Condition | Severity | Destination |
|---|---|---|
| Tenant-level accept-rate drop > 10pp day-over-day | Page | On-call rotation (mod-110) |
| Refusal-rate spike z > 3 over 1h | Page | On-call rotation |
| Golden-trace replay case that passed yesterday fails tonight | Page | On-call rotation |
| Judge-score mean drift > 0.05 vs 30d baseline, sustained 1h | Page | On-call rotation |
| PSI 0.1–0.25 on any input feature | Ticket | FDE weekly review |
| Edit-ratio distribution left-shift (more heavy edits) over a week | Ticket | FDE weekly review |
| Single-tenant judge score drop without workload change | Ticket | FDE weekly review |
| Thumbs-down spike (small absolute volume) | Ticket | FDE weekly review |

Two non-negotiables:

- **Every alert gets a runbook.** The runbook names the likely causes, the checks the on-call runs, and the mitigation (rollback to prior model version, flip to shadow mode, open a canary pause). An alert without a runbook is decoration — it fires, the on-call acknowledges, nothing happens.
- **Alerts gate rollouts.** The mod-109 rollout machinery reads the alert state. If a drift alert fired in the last hour, the GA promotion is blocked until the alert is cleared or waived with a named approver. Alerts that fire but do not block new rollouts are strictly worse than no alerts.

## Privacy posture

Online eval inherits the pilot's trust boundary. Three rules.

1. **Judge-LLM endpoint inside the customer's boundary.** Same model-gateway posture the primary call uses (mod-104 Ch 03). Zero-retention, no-training. The judge sees real customer content; it cannot leak it.
2. **Trace + judge-score pair stored in the customer's data store.** Not in the vendor's eval warehouse. The retention policy for the pair is governed by the DPA; it is typically *shorter* than the production trace retention (because the judge-score pair carries an additional layer of inference about the content, and the customer's legal team often tightens its lifetime).
3. **Feedback events governed by the DPA.** Edit-ratio alone is a float — low-risk. Edit *content* is PII-rich and requires an explicit DPA clause to capture. Do not ship edit content to the vendor by default.

The online eval pipeline is subject to the same security review as the pilot itself (mod-108). A judge pipeline added later, outside the original review, is a fresh security review — not a reuse.

## Three archetypes

### Archetype A — US healthcare payer

High-volume CSR assistant. Sample rate: **5%** on member-inquiry resolutions; oversample to 20% on prior-auth appeal (rare + high-stakes). Implicit feedback: did the CSR save the drafted reply as-is (accept), edit before saving (edit, ratio captured), or dismiss the draft (discard)? Thumbs are present but < 1% of traffic. Dashboard in **Grafana under the VPC**, backed by Prometheus for metrics and Snowflake-via-Metabase for warehouse-side rollups. First drift alert typically fires on **PSI on the member_history intent proportion** when open-enrolment shifts the call mix. Judge pipeline on Bedrock with zero-retention, same boundary as the primary Claude call.

### Archetype B — US federal agency

Policy analyst drafting. Lower volume, higher sensitivity. Sample rate: **2%** on draft-generation; the judge itself runs on the GovCloud self-hosted inference endpoint (same boundary as the primary model). Implicit feedback: analyst draft-save hook — did the analyst keep the AI-drafted section in the final memo (accept), rewrite it (edit), or delete it (discard)? Edit-ratio captured; rewrite content is not. Dashboard in **in-GovCloud Elastic stack**, the same stack the agency's SOC uses. First alert typically fires on a **drop in cite-count per answer** — a signal that retrieval is returning fewer grounded chunks, which for a federal analyst workflow is a quality regression. The replay set is pinned to a signed artifact; changes go through the agency's change-management board.

### Archetype C — EU retail bank

Case-worker reply assistant. Sample rate: **10%** (judge cost is tolerable at the volume, and EU AI Act Article 14 (human oversight) pushes for dense online monitoring evidence). Implicit feedback: case-worker accepted the draft reply, edited it, or discarded it; edit-ratio captured; content is not captured without a DPA addendum that the bank has not signed. Dashboard in **Azure Monitor and Log Analytics under the EU Data Boundary**. First alert typically fires on **refusal-rate drift** — because Article 14 requires the pilot to defer to a human in specific cases, a sudden drop in refusals is a sign the model is now over-answering, and a sudden rise is a sign the model is now over-deferring. Either direction is a meaningful signal. The DORA-driven incident reporting path is wired into the alerting destination.

## Anti-patterns

**Explicit-feedback-only.** The dashboard shows a glowing thumbs-up ratio from the 0.4% of users who clicked. Leadership celebrates. Meanwhile the implicit funnel — if anyone built it — shows accept = 18%, edit = 22%, discard = 60%. The pilot is being ignored and the dashboard says it is loved. Implicit funnel first; thumbs second.

**Judge-LLM at 100% sample rate.** The judge cost now exceeds the primary model cost. The marginal signal above 5% is small. The customer's finance team eventually finds the line item and the pilot gets a budget cut. Pick a rate; cap the budget.

**Dashboards without an owner.** The dashboard exists. Nobody owns it. The accept rate drops 15 points over three weeks and nobody notices. Every dashboard panel has a named FDE on the runbook; weekly review is on the calendar.

**Drift alerts with no runbook.** The on-call gets paged at 03:00 by "PSI alert on intent distribution." They do not know what PSI is, why it is alerting, or what to do. They acknowledge and go back to sleep. Every alert has a runbook; new alerts without runbooks do not ship.

**Alerts that fire but do not block a rollout.** The drift alert fires at 14:00. The GA promotion rolls out at 15:00 — the rollout pipeline does not consult the alert state. The new version lands in production on top of a known drift. Rollout gates read the alert state; a firing alert is a blocker until cleared or waived with a named approver.

## Where this chapter sits with the rest

- **Chapter 03** named the metric portfolio; the online judge mirrors those metrics, not a separate set.
- **Chapter 04** defined the judge prompt and calibration; the same judge is reused online.
- **Chapter 06** (the exit eval) consumes the online baseline and feeds the gold set with newly-discovered edge cases.
- **mod-109** reads the online eval alert state to gate the shadow → canary → GA transitions.
- **mod-110** owns the on-call rotation that receives the pages and the SIEM destination the alerts route to.

## Summary

- Online eval is the post-ship watch. The offline portfolio is the pre-ship gate. Neither half is sufficient alone.
- Judge-as-LLM runs on a stratified 1–10% sample of production traffic, asynchronously, with the same prompt and calibration as offline, inside the customer's trust boundary, on a zero-retention endpoint.
- Implicit feedback (accept / edit / discard, plus edit-ratio) is the primary user signal. Thumbs are supplementary. Capture ratios, not rewrite content, unless the DPA covers it.
- Per-tenant quality dashboards live in the customer's observability stack. Panels: offline portfolio last-run, online judge score rolling, implicit funnel, edit-ratio distribution, drift, SLO breaches, replay status.
- Drift has four layers: input (PSI / KL / KS), output (z-score on refusal / length / cite count), quality (judge score + accept rate), and embedding (centroid cosine, triage only). The differential across the four is what identifies the cause.
- Golden-trace replay runs nightly against production with a pinned dataset and a pinned judge. Catches silent regressions from model bumps, index rebuilds, tool changes.
- Alerting partitions page from ticket. Every alert has a runbook; alerts gate mod-109 rollouts.
- Privacy posture matches the primary call: judge inside the boundary, pair stored under customer retention, feedback events governed by the DPA.
- Three archetypes land at 5% / 2% / 10% sample rates with tenant-specific dashboards and tenant-specific first-alert patterns.
- Chapter 06 is next: the pilot-to-production exit eval that folds online learnings back into the gold set and signs off on go-live.
