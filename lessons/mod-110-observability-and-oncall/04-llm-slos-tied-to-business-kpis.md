# Chapter 04 — LLM-Specific SLOs Tied to the Customer's Business KPI

## Why this chapter exists

Chapters 02 and 03 made every invocation measurable; this chapter decides *which measurements are the ones the pilot promises to hit*. That is the **service-level objective** (SLO) — a target on a service-level indicator (SLI) with an **error budget** and an **error-budget policy** that names who gets paged, who stops releases, and who signs the "the budget is burnt" letter.

The standard discipline comes from the Google SRE book and workbook ([Chapter 4 of the Site Reliability Engineering book](https://sre.google/sre-book/service-level-objectives/) and the [SRE workbook's SLO chapters](https://sre.google/workbook/)). That discipline assumed web-service SLIs — availability, latency, throughput. An LLM pilot has four orthogonal SLI families, not three, and every one of them must trace back to a business KPI the sponsor committed to at scoping ([mod-102](../mod-102-customer-discovery-and-pilot-scoping/)). An SLO that does not route to a KPI is a slide-ware SLO; an LLM-specific regression that does not burn any SLO is a silent regression.

The four families:

1. **Quality** — does the model do the job well? Judge scores, SME feedback acceptance, gold-set-mapped pass-rate.
2. **Latency** — does it answer in time? TTFT, TPOT, p50 / p95 / p99 end-to-end.
3. **Cost** — does it fit the per-task, per-tenant, per-day envelope the sponsor approved?
4. **Safety** — does it refuse when it should and not refuse when it shouldn't? Guardrail trip rate, refusal rate, policy-violation rate.

Each family has a different measurement shape, a different error-budget half-life, and a different burn-rate alert policy. SLO truthfulness is a gate dimension (Chapter 01); the rest of this chapter is how to be truthful.

## SLO fundamentals the FDE already assumes

The FDE at level 35 is assumed to know the standard SRE SLO vocabulary. A compressed refresher so this chapter's additions are legible.

- **SLI (service-level indicator).** A ratio of good events to total events, measured over a window. Example: `good / valid` where `valid` excludes retries, synthetic probes, maintenance. Keep SLIs as proportions when possible — proportions compose, averages do not.
- **SLO (service-level objective).** A target on the SLI: "99.5 % of valid requests over a rolling 30-day window succeed." The number is a decision; higher is not better.
- **SLA (service-level agreement).** Contractual commitments — a customer-visible, legally-meaningful subset of the SLOs with consequences attached (credits, breach language). The handoff SLA from [mod-109 Chapter 06](../mod-109-deployment-rollout-change-management/06-customer-side-handoff-and-sme-training.md) lives here.
- **Error budget.** The remainder: `1 − SLO`. If the SLO is 99.5 %, the budget is 0.5 % — i.e. 3.6 hours per month at a 30-day window. A policy on burn rates decides what happens as the budget depletes.
- **Error-budget policy.** The *organisational* side of the SLO: who gets paged on burn, who stops releases, who signs the "we have burnt the budget" letter. The Google SRE workbook makes the error-budget policy a named artefact; mod-110 keeps the convention.

LLM-specific additions:

- **Judge SLIs are noisy.** A judge-as-LLM call introduces ±3-5 pp of random variance per call (varies by judge and task). SLOs on judge metrics use wider windows and smaller targets than SLOs on hard deterministic measurements.
- **Latency SLIs have a sub-structure.** TTFT (time to first token) and TPOT (time per output token mean) are separate measurements from end-to-end latency; a streaming UI cares about TTFT; a batch workflow cares about end-to-end. Chapter 04's SLO shape distinguishes them.
- **Cost SLIs are usage-pinned.** A fixed USD/day target assumes a constant volume. A pilot ramp changes volume; the cost SLI must be a per-task or per-tenant number plus a budget alert on the total.
- **Safety SLIs are bidirectional.** Refuse-when-should (true positive on guardrail) and don't-refuse-when-shouldn't (false positive) are two distinct SLIs; a refusal-rate graph alone obscures which direction moved.

## Quality SLOs

**Quality** is the hardest family to measure well. The FDE needs three layers: offline eval over a fixed gold set, online eval over a stratified traffic sample via judges, and user-and-SME feedback.

### Offline quality SLO

Base signal: the pilot gold-set pass-rate from [mod-107 Chapter 03](../mod-107-eval-eng-on-customer-data/03-offline-eval-design.md), run on every CI build and on every bundle promotion (per [mod-109 Chapter 05](../mod-109-deployment-rollout-change-management/05-per-tenant-versioning-and-replay.md)).

- **SLI definition.** `good_cases / total_cases` on the pilot gold set, where `good_case` is: judge verdict `pass` AND no gold-set-hard-fail check tripped.
- **SLO.** Example: `≥ 0.85 pass-rate` on the pilot gold set at every promotion. (The specific number is per pilot; 0.85 is illustrative.)
- **Window.** Point-in-time (per build), not rolling. Offline quality is a gate, not a trending metric.
- **Error budget.** Zero on a hard gate; a warning-only threshold (e.g. 0.82) can allow one build before an escalation.
- **Error-budget policy.** On fail: the bundle cannot promote. On warning: the release manager is notified; the next promotion blocks until the trend recovers.

### Online quality SLO

Base signal: the online-eval judge from [mod-107 Chapter 05](../mod-107-eval-eng-on-customer-data/05-online-eval.md) running on a stratified sample of live traffic.

- **SLI definition.** Rolling share of sampled invocations where `judge.score >= judge.gate_threshold`. Stratify the sample by intent class, tenant, and user segment so no single category dominates.
- **SLO.** Example: `≥ 0.80 judge-pass rate` on a 7-day rolling window. (The number tracks offline; online tends to be a few points lower because real traffic is harder than the gold set.)
- **Window.** 7 days rolling; 24-hour sub-window for burn-rate alerting.
- **Error budget.** `1 − SLO = 0.20`. Burn-rate alerting (see below).
- **Error-budget policy.** 25 % burn in 24h: warning, operator review, SME batch; 50 % burn in 24h: bundle freeze (no new deploys to this pilot), on-call analysis, candidate rollback to previous bundle; 100 % burn: automated kill-switch to previous bundle, incident declared.

### User / SME acceptance SLO

Base signal: the user's or SME's explicit accept / reject / edit action on a draft output (per [mod-107 Chapter 05](../mod-107-eval-eng-on-customer-data/05-online-eval.md)'s implicit-feedback discipline).

- **SLI definition.** `acceptances / (acceptances + rejections + major_edits)` per workflow step.
- **SLO.** Example: `≥ 0.70 acceptance rate` on a 7-day rolling window.
- **Window.** 7 days; 24-hour sub-window for burn-rate.
- **Error budget.** `1 − SLO = 0.30`.
- **Error-budget policy.** The acceptance SLO is the most sensitive to prompt / model / retriever changes; a drop here is the earliest signal of a regression that will later show up in judge metrics. Burn-rate alerting routes to the release manager first, not the on-call.

### Quality SLI gotchas

- **Judge drift.** The online judge is itself an LLM; providers roll silent updates (per [mod-108 Chapter 06](../mod-108-enterprise-security-and-compliance/06-ai-supply-chain-provenance.md)). A judge pinned to a specific provider model is instrumented in Chapter 02; a step-change in judge metrics without a bundle change is a provenance alert, not a quality incident.
- **Sample bias.** Online judge sampling that isn't stratified over-represents easy traffic; adding a hard intent class to the workflow drops the measured pass-rate without the pilot regressing.
- **SME-feedback volume.** A 70 % acceptance SLO assumes enough SME actions to compute it. In an air-gapped Archetype B pilot with 20 analysts, 7-day windows may contain few tens of actions; the SLO's confidence interval is wide, and the error-budget policy accounts for it by requiring a sustained burn, not a single-day drop.

## Latency SLOs

LLM latency has sub-structure that generic APM "p95 request latency" hides.

### TTFT SLO (time to first token)

Relevant when the UI streams tokens to the user.

- **SLI definition.** p95 of `time from request receipt to first token emitted`, measured over a 1-hour window, computed server-side at the gateway ([mod-104 Chapter 03](../mod-104-llm-app-in-customer-envs/03-model-gateway-in-customer-env.md)).
- **SLO.** Example: `p95 TTFT ≤ 500 ms` for a chat-style workflow. (The target is per pilot; conversational feel typically needs sub-second TTFT.)
- **Window.** 1 hour rolling; 30-day for the headline.
- **Error budget.** Fraction of 1-hour windows in the 30-day period that breached the target.
- **Error-budget policy.** Moderate — a brief TTFT breach is user-visible but transient. Alert on 5-consecutive-window breach at warning; 10-window at page.

### TPOT SLO (time per output token)

Relevant to streaming and to any workflow where completion length matters.

- **SLI definition.** p95 of `(end_ts − first_token_ts) / output_token_count` across invocations where `output_token_count > 20`, over a 1-hour window.
- **SLO.** Example: `p95 TPOT ≤ 40 ms/token`.
- **Window.** 1 hour rolling.
- **Error budget.** Treated as TTFT.

### End-to-end latency SLO

Relevant to every workflow.

- **SLI definition.** p50 / p95 / p99 of end-to-end span duration, over a 1-hour window, computed from Chapter 02's trace data.
- **SLO.** Three tiers — p50, p95, p99 — each with their own target. Example: `p50 ≤ 1.5 s, p95 ≤ 3.0 s, p99 ≤ 6.0 s`.
- **Window.** 1 hour; 30-day for the headline.
- **Error budget.** `1 − fraction_of_windows_in_SLO`.
- **Error-budget policy.** p99 burn is the "something is pathologically slow for a specific slice" signal; p95 burn is the "something is widely slow" signal. Both page the on-call; the p99 alert links to a runbook that starts with "which tenant / which intent / which model variant?"

Latency targets differ by deployment archetype ([mod-109 Chapter 02](../mod-109-deployment-rollout-change-management/02-deployment-modes-saas-vpc-onprem.md)):

| Archetype | TTFT target | p95 end-to-end | Why |
|---|---|---|---|
| A (VPC install on AWS) | 500 ms | 3.0 s | Same-region Bedrock; realistic |
| B (on-prem self-hosted) | 800 ms | 5.0 s | Self-hosted serving stack; constrained GPU; different envelope |
| C (dedicated SaaS in EU) | 600 ms | 3.5 s | Azure OpenAI same-region |

## Cost SLOs

Cost burns the fastest of the four families when it goes wrong — a prompt change that doubles output length arrives on the bill before it shows up in a quality metric.

### Per-task cost SLO

Relevant to every workflow.

- **SLI definition.** p95 of `pilot.cost.total.usd` per task (an "answer_member_inquiry" or "draft_reply"), computed from Chapter 02's metrics.
- **SLO.** Example: `p95 cost_per_task ≤ $0.025`. (Comes from the cost model in the handoff packet ([mod-109 Chapter 06](../mod-109-deployment-rollout-change-management/06-customer-side-handoff-and-sme-training.md)).)
- **Window.** 1-hour rolling for alerting; daily for reporting.
- **Error budget.** Fraction of windows above target.
- **Error-budget policy.** Any 1-hour window where p95 cost > 1.5× SLO warns; > 2× pages the on-call and freezes new bundle promotions.

### Per-tenant daily cost SLO

Relevant when the pilot serves multiple tenants or departments.

- **SLI definition.** Daily total `pilot.cost.total.usd` per tenant, aggregated from metric labels.
- **SLO.** Per-tenant daily cap, set per pilot (e.g. `≤ $1,000/day` per tenant).
- **Window.** Daily.
- **Error budget.** Binary: either the daily cap holds or it doesn't.
- **Error-budget policy.** 80 % of cap by mid-day: tenant admin notified; 100 %: rate-limit applied at the gateway; 120 %: tenant paused.

### Pilot-level monthly cost SLO

Relevant to the sponsor's budget commitment.

- **SLI definition.** Rolling 30-day total cost against the monthly envelope from the pilot's cost model.
- **SLO.** The monthly envelope.
- **Window.** 30 days rolling; aligned with the billing cycle.
- **Error budget.** The overage.
- **Error-budget policy.** 50 % burn at mid-month: steering review; 80 %: escalation to the sponsor; 100 %: auto-rate-limit at the gateway with the sponsor's pre-signed permission.

### Cost SLI gotchas

- **Cache hits hide savings and costs.** A prompt-cache hit on a Bedrock or OpenAI call is far cheaper than a miss. The cost SLI must be computed from actual provider usage, not estimated from token counts.
- **Judge calls are cost, too.** Running a judge on 10 % of traffic adds a second model call to 10 % of invocations. The judge model's cost belongs in `pilot.cost.total.usd`; failing to include it under-reports.
- **Rate-card changes.** Model providers update prices; the pilot's `pilot.cost.rate_card.version` attribute from Chapter 02 is what lets the dashboard correctly decompose a cost change into "rate-card update" vs "usage change."

## Safety SLOs

Safety has two directions. Guardrails ([mod-106 Chapter 05](../mod-106-agentic-workflows-on-enterprise/05-tool-permissions-and-audit.md)) and the model's own refusal / safety behaviour each produce events. The FDE treats the family as a 2x2: should-block × did-block.

### Guardrail trip rate SLO

- **SLI definition.** `blocked_or_rewritten / total` in the guardrail layer.
- **SLO.** Example: `≤ 2 %` on a 7-day rolling window. The number is not "lower is better" — a pilot with a 0 % trip rate likely has a mis-configured guardrail.
- **Window.** 7 days rolling.
- **Error budget.** On both sides — `> 5 %` is a page (trip storm); `< 0.5 %` sustained is a warning (guardrail is probably broken).
- **Error-budget policy.** Trip storm (> 2× SLO): page the on-call; freeze guardrail config changes; triage against the "false-positive storm" playbook in Chapter 05. Trip collapse (< 0.25× SLO sustained 72 h): warning; verify guardrail integration.

### Refusal-rate SLO

- **SLI definition.** `refusals / total` where a refusal is a model response matching the pilot's refusal taxonomy (polished definition per pilot).
- **SLO.** Target range (not a one-sided target). Example: `0.5 % ≤ refusal_rate ≤ 3 %`.
- **Window.** 7 days rolling.
- **Error budget.** Each side of the band has its own budget.
- **Error-budget policy.** Breach high: user-visible overreach — page; quality SLO review. Breach low: potential safety-floor erosion — warning; red-team review scheduled.

### Guardrail false-positive SLO

- **SLI definition.** `guardrail_blocks_overturned_by_sme / guardrail_blocks` — the share of guardrail blocks the SME marked "actually should have been allowed" on later review.
- **SLO.** Example: `≤ 10 %` on a 30-day window.
- **Window.** 30 days (SME review is slow).
- **Error budget.** Fraction of blocks that are false positives.
- **Error-budget policy.** Burn here is a quality cost paid by the user — blocked answers they should have received. Links to the "false-positive storm" playbook in Chapter 05.

### Safety SLI gotchas

- **Refusal taxonomy drift.** The model provider updates its own classifier; what counted as a refusal last month may now be rewritten. Chapter 02's `gen_ai.response.finish_reasons` captures the provider's label; the pilot's own classifier is a secondary signal.
- **SME-labelling backlog.** The false-positive SLO depends on SME review of blocks; a backlog means the signal lags. Budget SME time per week specifically for block review.

## The error-budget policy

The SLO is numerical; the error-budget policy is organisational. Four questions, written per pilot and signed by the customer operations lead and the vendor FDE:

1. **Who gets paged on which burn rate.** Tier 1 on fast burn, Tier 2 on slow burn, release manager on quality burn, FinOps lead on cost burn.
2. **What stops when the budget burns 50 %, 75 %, 100 %.** New bundle promotions freeze at 50 %; canary expansions pause at 75 %; auto-rollback at 100 %. The specific thresholds are per SLO family.
3. **Who signs the "budget is burnt" letter.** Monthly roll-up: did any SLO's 30-day budget fully burn? If yes, a formal note from the operations lead to the sponsor, co-signed by the FDE, naming the cause and the remediation.
4. **What the quarterly SLO review covers.** Every pilot's SLO targets are reviewed each quarter; targets tighten as the pilot matures. A target that never changes is a target no one reads.

Burn-rate alerting follows [the Google SRE workbook's multi-window multi-burn-rate approach](https://sre.google/workbook/alerting-on-slos/). Two alerts per SLO:

- **Fast-burn alert.** 2 % of the 30-day budget burnt in 1 hour → page. Catches sharp regressions.
- **Slow-burn alert.** 5 % of the 30-day budget burnt in 6 hours → ticket, not a page. Catches slow drifts.

The multi-window form is what separates "page the on-call because a real incident is happening right now" from "assign a ticket to the release manager because the SLO is drifting." Chapter 05 writes the pager routing on top.

## Tying each SLO to a business KPI

The SLO is a technical target; the KPI is what the sponsor paid for. Every SLO in the handoff SLA ([mod-109 Chapter 06](../mod-109-deployment-rollout-change-management/06-customer-side-handoff-and-sme-training.md)) routes to a specific KPI from the scoping agreement ([mod-102](../mod-102-customer-discovery-and-pilot-scoping/)).

A table for Archetype A — member-inquiry resolution:

| SLO | Target | Business KPI it supports | KPI target |
|---|---|---|---|
| Online judge-pass rate | ≥ 0.80 | First-contact resolution rate | +5 pp over control |
| SME acceptance rate | ≥ 0.70 | Agent time-per-inquiry | −15 % over control |
| p95 end-to-end latency | ≤ 3.0 s | SME throughput (inquiries/hour) | +10 % over control |
| p95 cost-per-task | ≤ $0.025 | Cost-per-inquiry (total) | within monthly cost envelope |
| Refusal rate band | 0.5 % – 3 % | Member experience (escalation rate) | no increase over control |
| Guardrail FP rate | ≤ 10 % | SME time waste (over-blocked inquiries) | ≤ 2 hours / week |

A table for Archetype C — customer-support reply + case triage:

| SLO | Target | Business KPI it supports | KPI target |
|---|---|---|---|
| Online judge-pass rate | ≥ 0.80 | Reply draft acceptance rate | ≥ 70 % |
| p95 end-to-end latency | ≤ 3.5 s | Average handle time | −10 % over control |
| p95 cost-per-task | ≤ €0.030 | Cost per case | within envelope |
| Refusal rate band | 0.5 % – 3 % | Compliance-flag rate | no increase |
| Monthly pilot cost | ≤ envelope | FY cost commitment | hold the commitment |

The table is reviewed quarterly; a target that never ties to a KPI the sponsor cares about gets dropped.

## The SLO document

Every SLO exists as a managed document — not a slide. The shape for one SLO:

```yaml
# slo/online-judge-pass-rate.yaml
slo:
  id: pilot-member-assist.quality.online-judge-pass-rate
  name: Online judge pass rate
  owner:
    primary: customer_operations_lead
    secondary: vendor_fde
  business_kpi:
    kpi_id: member-assist.first-contact-resolution
    kpi_target: "+5pp over control"
    scoping_doc: ../../../mod-102-customer-discovery-and-pilot-scoping/pilot-member-assist/scoping-memo.md
  sli:
    description: >-
      Share of stratified online-judge-sampled invocations where
      judge.score >= judge.gate_threshold.
    query: >
      sum(rate(online_eval_judge_pass_total{pilot="member-assist"}[7d]))
      / sum(rate(online_eval_judge_total{pilot="member-assist"}[7d]))
    exclusions:
      - synthetic_probes
      - canary_arm
      - shadow_arm
    judge:
      id: judge-member-inquiry-v4
      version: 4.2.1
      pinned_model: claude-opus-4-8
  target: 0.80
  window: 7d_rolling
  error_budget_policy:
    fast_burn_alert: 2pct_of_30d_budget_in_1h
    slow_burn_alert: 5pct_of_30d_budget_in_6h
    freeze_promotions_at: 50pct_burn
    pause_canary_at: 75pct_burn
    auto_rollback_at: 100pct_burn
    signatory: customer_operations_lead
  review_cadence: quarterly
  change_log:
    - 2026-04-01: created
    - 2026-06-15: target raised from 0.75 to 0.80 after 90d of consistent 0.83 performance
```

The YAML lives in the artefact-bundle manifest (per [mod-109 Chapter 05](../mod-109-deployment-rollout-change-management/05-per-tenant-versioning-and-replay.md)); a change to a target is a bundle change, reviewed and audited.

## SRE SLI / SLO specification standard

The emerging cross-vendor SLO specification worth knowing: [OpenSLO](https://github.com/OpenSLO/OpenSLO). A YAML schema for SLO documents that tools (Nobl9, Sloth, Pyrra) can read directly. For an FDE handing an SLO document to a customer who runs their own SRE platform, OpenSLO is the format that lets the customer's tooling consume the SLOs without re-authoring them. <!-- needs-research: confirm OpenSLO current spec version and major tool support at publication -->

## Running the three archetypes

**Archetype A — US healthcare payer.** Four SLO families; six core SLOs in the handoff SLA; error-budget policy co-signed by the member-services operations lead and the FDE; monthly SLO review; quarterly target recalibration. Burn-rate alerting integrated with the customer's Datadog monitor fleet; fast-burn pages via PagerDuty to the customer's Tier 1 operations rotation.

**Archetype B — US federal agency.** Four SLO families; smaller SLO set (quality and safety most prominent; cost is nearly fixed on dedicated GPUs); error-budget policy aligned with the ATO continuous-monitoring cadence; quarterly review aligned with POA&M updates. Burn-rate alerting integrated with the agency's Splunk ITSI or equivalent; pages via the agency's on-call tool. <!-- needs-research: confirm whether Splunk ITSI or an alternative is standard for federal agency on-call integration -->

**Archetype C — EU retail bank.** Four SLO families; six core SLOs; error-budget policy co-signed by the bank's operations lead, model-risk lead, and the FDE; weekly SLO review; quarterly target recalibration aligned with the EU AI Act Article 72 post-market-monitoring record. Burn-rate alerting via Azure Monitor + Sentinel; pages via the bank's operations centre. SR 11-7-equivalent model-risk function reviews every SLO target change.

## Anti-patterns

**SLOs with no measurement query.** The handoff SLA promises "99.5 % availability" and "high quality"; nobody wrote the query that computes either. Chapter 04 requires every SLO's `sli.query` field to be a working query against the production telemetry.

**One SLO per family, averaged.** A single "quality SLO" is a composite of judge, acceptance, and gold-set metrics that moves slowly and masks regressions in any one component. Each family has multiple SLOs; the burn-rate alerts route each to the right owner.

**Error-budget policy as a Slack thread.** The decision "who stops releases at 75 % burn" is debated on the day the burn happens. The policy is a signed document per Chapter 04's template, version-controlled with the SLO definitions.

**Targets never updated.** The 95 % acceptance target set in week two is still the target in month twelve, even though the pilot has been running at 98 % for six months. SLOs that never tighten are SLOs no one reads. Quarterly recalibration is non-negotiable.

**Burn-rate alerting without multi-window.** Single-window alerts either page too often (window too short) or miss sharp regressions (window too long). The multi-window form from the SRE workbook is the baseline, not a sophistication.

**Cost SLO estimated from token counts.** The cost SLI is derived from `tokens × rate_card`; a prompt-cache hit charges less; the discrepancy accumulates. Chapter 04's cost SLI reads actual provider-reported costs, captured as `pilot.cost.total.usd` at the invocation.

**Safety SLO on refusal rate only.** A low refusal rate is treated as success; the guardrail false-positive signal is not instrumented; the SMEs quietly complain that the pilot over-blocks and then stop using it. Safety is a four-cell matrix; one cell on a dashboard is a half-measurement.

**Business KPI floating free.** The handoff promises "+5 pp first-contact-resolution" with no mapping to an SLO; the SLOs promise technical targets with no mapping to the KPI. In the quarterly review, the sponsor asks "are we hitting the number we committed to?" and the answer is a reconstruction project. Every SLO ties to a KPI; every KPI ties to at least one SLO.

## Summary

- An LLM pilot has four orthogonal SLO families, not three: quality, latency, cost, safety. Each has multiple SLOs; each ties to a business KPI the sponsor committed to at scoping.
- Quality SLOs come from offline gold-set pass-rate (gate), online judge pass-rate (rolling), and user / SME acceptance rate (rolling). Each has its own window and error-budget policy.
- Latency SLOs decompose into TTFT (streaming UIs), TPOT, and end-to-end p50 / p95 / p99. Targets differ by deployment archetype because self-hosted inference and hosted APIs have different envelopes.
- Cost SLOs decompose into per-task, per-tenant daily, and pilot-level monthly. All three read actual provider-reported costs, not token-count estimates.
- Safety SLOs decompose into guardrail trip rate, refusal rate band (not a one-sided target), and guardrail false-positive rate. Over-blocking is a quality cost; the SME-feedback pipe is how it's measured.
- Error-budget policy is a signed, version-controlled document that names who gets paged at which burn rate, what freezes at 50 / 75 / 100 % burn, and who signs the monthly "budget burnt" letter. Burn-rate alerting follows the Google SRE workbook's multi-window multi-burn-rate approach.
- Each SLO lives as a YAML document — name, owner, business KPI, SLI query, target, window, error-budget policy, change log. The format is compatible with OpenSLO for customer-tool portability.
- Archetype A gets six core SLOs with monthly review; Archetype B gets a smaller safety-and-quality-heavy set aligned with ATO continuous monitoring; Archetype C runs a full six-SLO set with SR 11-7-equivalent model-risk oversight.
- Eight anti-patterns: SLOs without queries, averaged families, informal error-budget policy, static targets, single-window alerts, token-count cost estimates, refusal-rate-only safety, KPI-floating-free.
- Chapter 05 is next: the on-call rota that answers these burn-rate pages — joint incident command with the customer's operator, SEV classification, LLM-specific failure-mode playbooks.
