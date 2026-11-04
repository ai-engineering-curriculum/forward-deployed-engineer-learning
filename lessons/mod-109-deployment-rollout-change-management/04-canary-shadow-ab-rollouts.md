# Chapter 04 — Canary, Shadow, and A/B Rollouts for LLM Features

## Why this chapter exists

Chapter 03 landed the flag platform — rollout flags, kill-switches, experiment flags, naming conventions, audit stream, CAB ticket linkage. Chapter 04 uses those flag types to run the three LLM-specific rollout primitives: canary, shadow, and A/B. The primitives are what turn a flag flip into a defensible claim that the candidate bundle is better than the control.

The problem a generic blue-green pipeline cannot solve: **LLM regressions do not surface as HTTP 5xx.** They surface as a drop in judge pass-rate, a spike in refusal-rate, a shift in output distribution, a quiet creep in cost-per-call, a new failure mode on one intent class. A release pipeline that watches 5xx and p95 latency and promotes the canary on green metrics is the silent-regression generator named in Chapter 01. The LLM canary gate reads the eval harness and the online judge, not just the load balancer.

This chapter is the heart of the rollout-safety optimisation dimension from Chapter 01. It consumes Chapter 03's flag taxonomy; produces the gate criteria Chapter 05's bundle-version promotion fires against; feeds Chapter 06's admin dashboard the canary state; and feeds Chapter 08's pager protocol the rollback trigger.

## Three rollout primitives, named

Three primitives, often conflated, with distinct mechanics and distinct answers.

**Canary.** A small, scoped percentage of live production traffic is routed through the candidate bundle; the rest stays on the control. The canary runs under explicit, pre-registered success criteria read from the eval harness, the online judge, and the SLO budget. If the canary passes the gate, it expands (1% → 5% → 25% → 100%); if it burns the gate, it rolls back via the kill-switch. The canary is live, the outputs are returned to users, and users bear the risk — which is why the percentage is small and the gate is strict.

**Shadow.** The candidate bundle runs in parallel with the control on the *same* input, but its output is observed, not returned. The user sees only the control's answer. The shadow arm produces an output for comparison, logged with a correlation ID to the control. Shadow lets the team measure a change on real customer traffic without exposing users to it. The gateway from [mod-104 Chapter 03](../mod-104-llm-app-in-customer-envs/03-model-gateway-in-customer-env.md) is where the request-clone is wired.

**A/B.** A statistically designed split between two or more arms under a pre-registered hypothesis, powered sample size, randomisation unit, metric definition, and stopping rule. Where a canary answers "is it safe to expand?" an A/B answers "is the candidate *better* than the control on the metric that matters by at least Δ, at power 0.8 and α 0.05?" An A/B is an experiment in the methodological sense, not a rollout vehicle.

The three compose. A candidate bundle typically moves: shadow (observe) → canary (small live percentage, gated) → A/B (powered comparison on the full feature) → full rollout. A canary expansion is often gated on a shadow-eval pass; an A/B evaluates a change a canary has already shown is not catastrophic. Collapsing the three into one flag flip is how Chapter 01's silent-regression anti-pattern gets committed.

## Canary success criteria for LLM features

A canary without gate criteria is theatre. The gate is a set of pre-registered thresholds, one per dimension, that every expansion step must pass. If any one fails, the canary rolls back automatically.

Six gate criteria must be defined before the canary starts.

1. **Offline eval pass-rate on the pilot gold set does not regress.** The pilot gold set from [mod-107 Chapter 03](../mod-107-eval-eng-on-customer-data/03-offline-eval-harness.md) is re-run against the candidate bundle; the pass-rate on the candidate must be no more than X percentage points below the control (X is per-pilot, typically 0-2pp). A regression here means the candidate failed before touching a user.
2. **Online LLM-judge pairwise preference reaches parity with the control.** The online judge rubric from [mod-107 Chapter 05](../mod-107-eval-eng-on-customer-data/05-online-eval-judges.md) scores candidate-vs-control pairs blind. Preference for the candidate must be ≥ 50% ± the judge's margin of error for the sample size at the current canary percentage.
3. **p95 latency stays inside the SLO from [mod-104 Chapter 05](../mod-104-llm-app-in-customer-envs/05-cost-and-latency-slos.md).** The candidate may not raise end-to-end p95 above the SLO envelope the handoff SLA will commit to in Chapter 07. A model upgrade that takes 1.8s instead of 0.9s fails this gate even if quality improves.
4. **Error-rate and refusal-rate bounded.** Error-rate (API errors, guardrail blocks, tool failures) and refusal-rate (model declined to answer) stay within a defined band above control — typically ≤ 1-2pp above. A spike in refusals is the most common silent regression on a prompt change.
5. **Cost-per-call within the per-call budget.** The per-call cost (model tokens + retrieval compute + judge cost) must be within the per-call envelope from mod-104 Chapter 05. A reranker that doubles retrieval latency and triples cost may pass quality gates and still fail this one.
6. **Human review sampling on canary traffic does not flag regressions above threshold.** A stratified sample of canary invocations is reviewed by SMEs or the customer-side operations lead; the flagged-as-regression rate must be below a per-pilot threshold (typically 2-5%). This gate catches failure modes the judge missed.

A concrete gate table for a hypothetical "generator model upgrade" canary on an Archetype A member-inquiry pilot:

| Gate | Metric | Control value | Threshold | Window |
|---|---|---|---|---|
| Offline eval | Pilot gold pass-rate | 0.87 | ≥ 0.85 | Pre-canary, blocking |
| Online judge | Pairwise preference for candidate | — | ≥ 0.48 | Rolling 24h on canary traffic |
| Latency SLO | p95 end-to-end | 1.2 s | ≤ 1.5 s | Rolling 1h |
| Refusal rate | Refusal / total | 1.4% | ≤ 3.0% | Rolling 24h |
| Cost-per-call | USD / invocation | $0.018 | ≤ $0.025 | Rolling 24h |
| Human review | SME-flagged regressions | — | ≤ 3% of sampled | Daily SME batch |

The thresholds are written in the release note, the CAB ticket, and the flag's audit metadata — before the canary starts. "Success criteria written after launch" is the Chapter 01 anti-pattern.

## Shadow methodology

The plumbing — gateway cloning the request, async dispatch to the candidate bundle, correlation-ID logging — is in [mod-104 Chapter 03](../mod-104-llm-app-in-customer-envs/03-model-gateway-in-customer-env.md). The *analysis* of what the shadow produces is the Chapter 04 contribution.

Four comparison methods, applied by signal type.

- **Deterministic diff for retrieval.** For a new retriever (new embedding model, new chunking strategy, new reranker), compare the set and ordering of documents returned for the same query. Metrics: Jaccard overlap of top-k, rank correlation (Kendall's τ or NDCG against a gold relevance label if one exists). Deterministic because retrievers are expected to produce the same documents for the same query within a bundle version.
- **Semantic-similarity scoring for generation.** For a new prompt or new generator model, compute embedding cosine similarity between the control completion and the candidate completion. Flag pairs below a per-pilot threshold (typically 0.75-0.85 depending on task). Similarity is a cheap first-pass filter; a drop below threshold triggers judge scoring.
- **LLM-judge pairwise preference.** The online-eval judge from mod-107 Chapter 05 scores control-vs-candidate pairs blind to which arm is which. Aggregate preference is the headline shadow metric. The judge is the same judge that gates the canary; shadow is the rehearsal.
- **Human sampling on a stratified subset.** A per-pilot sampling plan (e.g. 50 invocations per intent class per week) is reviewed by SMEs. The sample is stratified by intent, by customer segment, and by confidence score — the tails matter.

A shadow analysis loop, pseudo-code:

```python
# shadow_analysis.py — runs nightly against the correlation-ID pair log
def analyse_shadow_window(start_ts, end_ts, control_bundle, candidate_bundle):
    pairs = pair_log.query(start_ts, end_ts, control_bundle, candidate_bundle)
    report = DiffReport(control_bundle, candidate_bundle)
    for pair in pairs:
        if pair.kind == "retrieval":
            report.retrieval.add(
                jaccard=jaccard(pair.control_docs, pair.candidate_docs),
                ndcg=ndcg_at_k(pair.candidate_docs, pair.gold_labels, k=5),
            )
        elif pair.kind == "generation":
            sim = cosine(embed(pair.control_out), embed(pair.candidate_out))
            report.generation.add(similarity=sim)
            if sim < THRESHOLDS.semantic_min:
                judge_verdict = online_judge.pairwise(
                    pair.input, pair.control_out, pair.candidate_out, blind=True)
                report.generation.add_judge(judge_verdict)
        report.cost.add(pair.candidate_cost_usd, pair.control_cost_usd)
        report.latency.add(pair.candidate_ms, pair.control_ms)
    report.human_sample = stratified_sample(pairs, plan=PILOT_SAMPLING_PLAN)
    return report
```

The shadow report lands in the per-pilot decision document ([mod-107 Chapter 06](../mod-107-eval-eng-on-customer-data/06-per-pilot-decision-document.md)); the CAB reads it before approving the canary start.

## A/B rollouts with statistical rigour

A/B on an LLM feature is a methodological commitment, not a flag setting. Five parameters must be pre-registered.

- **Hypothesis.** A specific, falsifiable, pre-registered claim: *"The new reranker increases resolved-ticket-rate by ≥ 1.5 percentage points with statistical power 0.8 at α = 0.05."* A claim without a minimum detectable effect (MDE) and a chosen power is not a hypothesis; it is a fishing expedition.
- **Randomisation unit.** *Per-user*, not per-request, for LLM features. A user who sees one bad answer behaves differently for the rest of the session — they re-ask, re-phrase, escalate, abandon. Per-request randomisation contaminates the control arm with the candidate's downstream user behaviour and voids the independence assumption of standard tests. <!-- needs-research: cite the Kohavi / Thomke / Xu experimentation literature on session-level contamination for the final reading list -->
- **Powered sample size.** For a two-proportion test comparing resolved-ticket-rate at baseline p₀ against p₁ = p₀ + MDE, the per-arm sample size is approximately `n ≈ ((z_{1−α/2} + z_{1−β})² · (p₀(1−p₀) + p₁(1−p₁))) / MDE²`. Inputs: baseline rate p₀, minimum detectable effect, power (1−β), significance level α. A pilot that cannot reach the required n within a reasonable window must widen the MDE, extend the window, or acknowledge the A/B is under-powered before reading the result.
- **Metric latency.** The metric that matters often arrives hours or days later: resolved-ticket-rate is known when the ticket closes; re-contact-rate requires a 7-day window; downstream workflow completion may take weeks. The A/B must be designed around the metric's natural latency; a 7-day window demands the experiment stays live at least that long after the last-enrolled user, which rules out short-cadence A/Bs on long-latency metrics.
- **Stopping rule.** Pre-registered: duration, sample size, and group-sequential boundaries if any. **No peeking.** Peeking at an unblocked metric inflates false-positive rate; the analyst who checks daily and stops when p < 0.05 appears is reading a nominal 5% test at an effective rate several times higher. Use a sequential test (Pocock, O'Brien-Fleming) or always-valid inference (Howard / Ramdas) if intermediate looks are required.

**Novelty effects and non-stationarity.** The first week of a new feature is not representative: users interact differently with something new, the SME programme is actively calibrating, and the data distribution the model sees is shifting. Plan for a burn-in period (often 1-2 weeks) during which metrics are observed but not read as part of the stopping rule. Likewise, the LLM feature itself is non-stationary — model providers roll silent updates (see [mod-108 Chapter 06](../mod-108-enterprise-security-and-compliance/06-ai-supply-chain-provenance.md) on provenance drift), the retrieval index grows, and the user base changes month to month. Longer A/Bs trade statistical power for distributional coverage.

## Blameless rollback when the canary burns the eval budget

Rollback is the designed path, not a failure. The team ran the right experiment and got a real result. Chapter 03's kill-switch is the mechanism; Chapter 04 is where the protocol is written.

**Trigger.** Any one gate from the six above crosses its threshold on the agreed window. Automated evaluation — not a human judgement call — flips the kill-switch. The audit log emits a `canary.rollback` event with the gate name, the measured value, the threshold, and the window.

**Pager routing.** Customer operations lead (Tier 1) is paged first; the vendor FDE (Tier 3) is paged for consult, not for operation. The escalation tiers are the ones Chapter 08 writes in detail; mod-110 instruments the pager. The customer-side admin does not have to approve the rollback — the kill-switch already fired — but is on the pager to run the comms, update the CAB, and annotate the incident timeline.

**Automated actions on rollback.**

- Kill-switch flips; candidate flag is set to `off` globally.
- An `IncidentCreated` event opens an incident in the customer's ticketing system (ServiceNow, Jira) linked to the CAB ticket that approved the canary.
- The mod-110 observability dashboard annotates the timeline: canary start, each expansion step, gate failure, rollback.
- The audit stream (customer SIEM) receives the `canary.rollback` event with full context for compliance.
- The per-pilot decision document ([mod-107 Chapter 06](../mod-107-eval-eng-on-customer-data/06-per-pilot-decision-document.md)) is updated with the rollback outcome.

**Post-mortem contents.** A blameless post-mortem lands within 72 hours containing: the change that was proposed, the gate that failed and its measured value, the counterfactual (what would have happened at full rollout had the gate been looser), what the gate taught about the change, and a recommendation (fix and retry, abandon, change design). The post-mortem is read by the vendor product team as mod-111 feedback and by the customer sponsor as an operational artefact.

**Re-try trigger.** The candidate bundle may be re-canaried only after (a) the specific failure mode is addressed in a new candidate bundle version (see Chapter 05 on bundle manifests), (b) the offline eval on the pilot gold set passes at a tighter threshold than before, and (c) the CAB re-approves. The canary is not re-tried with the same bundle on the hope that the first run was noise.

"Blameless" is not a feelings word. It is a design claim: the gate was the decision-maker, not the engineer. The engineer who proposed the change, the reviewer who approved it, and the SME who sampled the canary each did their job; the gate did its job; the system worked.

## Rollout automation

The primitives above are run by progressive-delivery tooling that supports custom analysis. Four reference tools worth knowing by name.

- **[Argo Rollouts](https://argoproj.github.io/argo-rollouts/).** Kubernetes-native progressive delivery; supports `AnalysisRun` resources that gate traffic promotion on a provider's result. Common for VPC-install customers who already run Argo CD.
- **[Flagger](https://fluxcd.io/flagger/).** Similar scope, part of the Flux ecosystem; supports metric-based canary promotion with Prometheus, Datadog, and custom webhook providers.
- **[LaunchDarkly release pipelines](https://docs.launchdarkly.com/home/releases).** Progressive-rollout automation tied to LaunchDarkly flags; integrates with release-monitoring via webhooks.
- **[Statsig Gradual Rollouts](https://docs.statsig.com/feature-flags/gradual-rollouts).** <!-- needs-research: confirm exact URL and current page structure on docs.statsig.com --> Statsig's flag-platform-native gradual rollout feature with integrated experiment analysis.

**The LLM gap.** None of these tools understand LLM-specific metrics out of the box. They understand Prometheus series, Datadog queries, generic HTTP request metrics. The FDE wires a *custom analysis provider* — a webhook or a Kubernetes-native metric exporter — that queries the offline eval harness ([mod-107 Chapter 04](../mod-107-eval-eng-on-customer-data/04-eval-harness-engineering.md)) and the online-eval judge (mod-107 Chapter 05) and returns a pass / fail per gate.

An Argo Rollouts `AnalysisTemplate` for the eval-gate, shape-only:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: llm-canary-eval-gate
spec:
  args:
    - name: candidate-bundle-version
    - name: pilot-id
  metrics:
    - name: offline-eval-pass-rate
      provider:
        web:
          url: https://eval-harness.internal/pilots/{{args.pilot-id}}/offline-eval
          jsonPath: "{$.pass_rate}"
      successCondition: result >= 0.85
      failureLimit: 0
    - name: online-judge-preference
      interval: 10m
      provider:
        web:
          url: https://eval-harness.internal/pilots/{{args.pilot-id}}/online-judge?bundle={{args.candidate-bundle-version}}
          jsonPath: "{$.pairwise_preference}"
      successCondition: result >= 0.48
      failureLimit: 2
    - name: p95-latency-ms
      provider:
        prometheus:
          address: https://prom.internal
          query: histogram_quantile(0.95, rate(llm_invocation_duration_ms_bucket{bundle="{{args.candidate-bundle-version}}"}[5m]))
      successCondition: result <= 1500
      failureLimit: 1
```

The analysis template is versioned in the same bundle manifest Chapter 05 defines; a change to the gate thresholds is a bundle change and runs through the same review.

## Running the three archetypes

**Archetype A — US healthcare payer.** Full canary + shadow available on live member-inquiry traffic inside the customer's VPC. LaunchDarkly release pipeline flipped by the customer's operations lead; monthly rollout cadence tied to the CAB calendar; the CAB ticket per canary expansion step names the gate thresholds and the current measured values. Shadow runs on 10-20% of traffic for every candidate that touches the generator or retriever. A/B on resolved-ticket-rate with a 7-day metric-latency window; per-user randomisation over the member population.

**Archetype B — US federal agency, air-gapped.** Live canary is often restricted: only internal agency traffic is allowed, no external frontier-customer exposure. Rollouts run as a staged ring deployment (dev → staging → sandbox → prod) with the same six gate criteria applied at each ring, read from the on-prem eval harness. Shadow is possible on *sanitised replay data* — stored invocations re-run against the candidate bundle in a sandbox ring. A/B is harder: small user population limits statistical power, and ATO change-control gates the cadence. The per-pilot gate thresholds are negotiated as part of the ATO body-of-evidence; a gate failure updates the POA&M item. <!-- needs-research: confirm whether current FedRAMP and OMB AI memos (M-24-10 and successors) have specific language on canary / staged-deployment expectations for GenAI systems -->

**Archetype C — EU retail bank.** Full canary + shadow on dedicated-SaaS tenancy; Statsig or LaunchDarkly tenant streaming into Microsoft Sentinel; CAB weekly with the current canary state read directly from the flag platform. [EU AI Act Article 72](https://artificialintelligenceact.eu/article/72/) post-market monitoring reads the online-eval dashboard and the canary logs as part of the deployer's ongoing obligations. <!-- needs-research: confirm the Article 72 post-market monitoring language and which obligations fall to the deployer vs provider for the specific use category this pilot sits in --> A/B on draft-reply acceptance rate and case-triage first-touch-resolution; per-user randomisation over the customer-support agent pool.

## Anti-patterns

**Canary without an eval gate.** Routing 1% of traffic to the candidate, watching HTTP 5xx and p95 latency, declaring victory. The LLM regressions do not appear in those signals. Chapter 01 named this; Chapter 04's six-gate table is the remedy.

**Shadow analysis never looked at.** Request-clone wired, correlation IDs flowing, no one reads the output. Shadow without a nightly diff report, a pre-registered threshold, and a named reviewer is log volume at a cost. The pseudo-code loop above is the minimum; it is run every night, not on request.

**A/B with per-request randomisation on an LLM feature.** A user randomised fresh on every request sees inconsistent behaviour, learns to re-phrase around the bad arm, and contaminates both arms. Per-user randomisation is the floor; per-session is sometimes defensible. Per-request is a statistical mistake.

**Rollback requires a human to approve.** If the gate burns, the kill-switch flips — automated, logged, paged. A human-in-the-loop rollback adds minutes to hours of user harm during which the gate was already telling the truth. The post-mortem is human; the rollback is not.

**Rollout automation that only looks at HTTP 5xx.** Argo Rollouts, Flagger, or LaunchDarkly release pipelines on their default metric set are necessary but not sufficient. The custom LLM analysis provider is the piece the FDE wires; without it, the tool is giving the team false confidence.

**Success criteria written after launch.** The CAB ticket named "upgrade generator model" with no gate thresholds; the gate is debated after the canary is at 10%; a disagreement about the threshold becomes a disagreement about the rollback. Thresholds are written before the canary starts and are in the audit log.

**Statistical power waved away.** A one-week A/B at 2% traffic on a long-latency metric with no sample-size calculation is not an A/B — it is a flag flip with a graph. The team either does the power calculation and runs the experiment long enough to read it, or they run a canary and label the output as a safety check, not a comparison.

**Novelty-effect windows used as results.** Reading the first-week A/B metric as the effect size and promoting to full rollout on that read. The first week is burn-in; the stable metric is weeks three onward (per-pilot, but never day-one).

## Summary

- Three rollout primitives — canary, shadow, A/B — are distinct in mechanics and in the question they answer; collapsing them into one flag flip is how silent LLM regressions ship.
- The canary gate is six-dimensional: offline eval pass-rate, online judge preference, p95 latency, error / refusal rate, cost-per-call, human review sampling. Each dimension has a pre-registered threshold and a window; failure of any one triggers automated rollback via the Chapter 03 kill-switch.
- Shadow analysis uses deterministic diff for retrieval, semantic-similarity + judge scoring for generation, and human sampling for the tails. A nightly diff report is the minimum artefact; shadow logs no one reads are a cost without a return.
- A/B rollouts require a pre-registered hypothesis with MDE, per-user randomisation, a powered sample size, metric-latency-aware window, a stopping rule, and a burn-in to control novelty effects. Per-request randomisation voids the test on LLM features.
- Blameless rollback is the designed path: the gate decides, the kill-switch fires, Tier 1 is paged, the post-mortem is read as mod-111 feedback. Re-try requires a new bundle version that addresses the specific failure mode and a fresh offline eval pass.
- Argo Rollouts, Flagger, LaunchDarkly release pipelines, and Statsig Gradual Rollouts are the automation layer; the FDE wires a custom LLM-eval analysis provider on top because none of these tools understand judge pass-rate or refusal-rate by default.
- Archetype A runs full live canary + shadow with LaunchDarkly and monthly CAB cadence; Archetype B substitutes a staged ring deployment and sanitised-replay shadow for the air-gapped constraint; Archetype C adds EU AI Act Article 72 post-market-monitoring integration on top of a full canary + shadow pipeline.
- Chapter 05 is next: the artefact bundle — prompts, retriever config, model pin, tool allow-list, judge, guardrails, gateway config — versioned as a single unit with explicit lineage, is what the canary promotes and the kill-switch rolls back to.
