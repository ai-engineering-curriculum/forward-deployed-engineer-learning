# Chapter 05 — Authoring Per-Customer Cost and Latency SLOs

## Why this chapter exists

SLOs are the numbers the pilot signs with the customer. They set the rollback trigger (mod-109), the pager rule (mod-110), the renewal conversation, and the decision in week 14 of "is this pilot ready to go live?" The in-house applied-LLM craft at level 30 sets SLOs on a product serving all users; the FDE at level 35 contracts SLOs with a *specific customer* grounded in *that customer's workflow and business KPI*. The overlay is the grounding.

This chapter writes the LLM-specific latency and cost metrics down with precise definitions, derives SLOs from a customer workflow, provides a section template, and closes the loop to the gateway (Chapter 03) and the observability stack (mod-110).

## The LLM-specific latency metrics

LLM inference is streaming by default, so standard web-request latency vocabulary splits in a specific way. Three metrics the FDE always carries:

### TTFT — time-to-first-token

Latency from the gateway receiving the request to the first token arriving back at the gateway (or, if the pilot is measuring at the user edge, from the user's browser dispatching the request to the first token rendered).

- **Why it matters:** TTFT is what the user perceives as "the model started." For streaming UX, a low TTFT makes the system feel responsive even when total completion takes seconds.
- **Where it is set:** the SME's visual-cue threshold, roughly the point past which the user wonders "is this working?" — commonly in the hundreds-of-milliseconds to low-single-digit-seconds range for non-reasoning tasks.
- **References:** [NVIDIA — Metrics for evaluating LLM inference performance](https://developer.nvidia.com/blog/metrics-for-evaluating-llm-inference-performance/) and the [vLLM documentation](https://docs.vllm.ai) use TTFT as a primary metric. The [OpenTelemetry GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) include a time-to-first-token attribute for span observability.

### TPOT — time-per-output-token

Average inter-token latency after the first token has arrived, measured across the completion.

- **Why it matters:** TPOT governs the *streaming feel*. A low, stable TPOT means the user sees text flowing smoothly; a variable or high TPOT means text stutters.
- **Where it is set:** by the SME's reading-speed and the UI's buffer behaviour. For most text tasks, a TPOT well below the human reading rate is sufficient; for display patterns that batch tokens into chunks, TPOT just needs to be below the chunk-emit rate.
- **Note:** some sources use ITL (inter-token latency) synonymously with TPOT; verify the convention your observability stack uses.

### E2E completion latency — p50 / p95 / p99

Total time from request to final token. Measured at the pilot's user-visible edge (not just at the gateway) so it captures the network path to the user.

- **Why percentiles matter:** the mean hides the tail. A p50 of 1.5 s with a p99 of 12 s is a very different experience from a p50 of 2.5 s with a p99 of 3.0 s. The user's bad day is a tail day.
- **Reference:** [Google SRE Book — Service Level Objectives](https://sre.google/sre-book/service-level-objectives/) is the canonical discipline on percentile-based SLOs. The FDE operates against the same patterns the SRE org does, scaled to the pilot.
- **Which percentile to contract:** p95 is the standard SRE handle for user-visible systems. p99 is appropriate when the pilot is a background flow where tail matters disproportionately (e.g. a batch queue where the straggler holds the whole batch). p50 is informative but rarely the contract number.

### Why all three are needed

A single "latency SLO" collapses three distinct user experiences:

- A pilot with excellent TTFT and bad TPOT feels responsive-then-stuttering.
- A pilot with bad TTFT and excellent TPOT feels frozen-then-fast.
- A pilot with excellent TTFT and TPOT but a 1% p99 catastrophic tail will still get complaints — the 1% is the SME's worst day.

Contract all three, measure all three.

## The cost metrics

Four costs the pilot tracks; each ties to a specific business decision.

### Cost per task

Dollars (or cents) per single completed pilot task. Includes input tokens, output tokens, cached prefix tokens (discounted per provider), gateway overhead (negligible per call but cumulative).

- **Business tie:** the per-task budget guardrail from the pilot-scope document (mod-102 Chapter 03's value-per-task check). Enforced at the gateway (Chapter 03) as a soft/hard cap.

### Cost per user session

Dollars per end-to-end user interaction with the pilot, which may include several model calls (draft, revise, critique, finalise).

- **Business tie:** if the pilot's UX is session-oriented, the user-value is per-session; the cost cap composes task costs within a session.

### Cost per tenant per day / per month

Aggregated cost across all users in a tenant over a time window.

- **Business tie:** the commercial agreement. The customer's bill (or the vendor's cost-to-serve if the pilot is at the vendor's expense) runs on these aggregates. Over-run triggers budget enforcement and account-team escalation.

### Cost per tenant per task-type

When the pilot has multiple task types (draft, classify, summarise), per-task-type cost reveals which flow is expensive and which is cheap. Feeds mod-109's rollout decisions (ship the cheap flow first, tune the expensive flow before expanding).

## Grounding SLOs in business-KPI value-per-task

This is the overlay. The in-house default "fast and cheap" is not a contract; it is a wish. The FDE's contract SLO is derived from three customer-side facts.

### Fact 1 — manual-alternative baseline

What does the SME do today, in the absence of the pilot? Measure or estimate:

- **Manual-alternative latency** — time from SME starting the task to completing it without the pilot.
- **Manual-alternative cost** — SME hourly cost × manual-alternative minutes.
- **Manual-alternative quality** — the quality bar the pilot is at minimum matching (mod-107 handles quality in depth; cross-reference here).

A manual-alternative latency of 30 minutes and a manual-alternative cost of $25 per task is a very different value framing than a 10-second alternative at $0.05.

### Fact 2 — SME workflow tolerance

What first-visible-text delay and total-response delay can the SME tolerate without workflow breakage?

- Interactive SME workflow (draft-and-refine): TTFT under a couple of seconds is typically required; total completion within the SME's attention span.
- Batch background workflow: TTFT may be minutes if the output lands in a queue the SME processes asynchronously; what matters is throughput and the SLA on the queue, not TTFT.

The SME is the right source for this answer. Mod-102's discovery interviews are where it was captured; Chapter 05 re-reads those notes.

### Fact 3 — business-KPI value-per-task

What does completing one task well deliver to the customer?

- Revenue uplift (sales reply drafted twice as fast → more replies per rep per week → more pipeline).
- Cost avoided (appeal drafted correctly first time → no re-work, no compliance miss).
- Hours returned to the SME (freed capacity for higher-value work).

A value-per-task of $500 supports a very different cost cap than a value-per-task of $5. Mod-102 Chapter 03's viability check derived the number; Chapter 05 uses it.

### The derivation

From the three facts:

```
value_per_task        = $V    (business-KPI uplift or cost avoided per task)
cost_cap              = c × V   (fraction c defended in the pilot-scope ADR; often 5-20%)
latency_cap_total     = SME tolerance for completion (interactive) or queue SLA (batch)
latency_cap_ttft      = SME visual-cue threshold for interactive; N/A for batch
quality_cap           = manual_alternative_quality × (1 + Δ)   (mod-107 writes Δ)
```

The derivation is defended in the ADR. Reviewers will push on `c` ("why 10% and not 5%?") and on the SME tolerance ("where did that number come from?"). Document the source.

## Writing the SLO section of the pilot-scope document

Mod-102 Chapter 04 fixed the pilot-scope template; the SLO section lives there. One block per metric; every block fills every field.

```
SLO section — <workflow name>

TTFT (interactive)
  Metric:         p95 TTFT for draft-generation requests
  Target:         < 1500 ms
  Rationale:      SME visual-cue threshold measured in discovery (~2s);
                  1500 ms leaves headroom for first-visible-text UX.
  Measurement:    OpenTelemetry span 'llm.request' with gen_ai.response.first_token_time;
                  measured at the gateway (not at the provider), so network to user
                  is observable but not inside the span.
  Error budget:   5% over rolling 30-day window.
  Owner (FDE):    <FDE name>
  Owner (peer):   N/A for hosted provider;
                  ai-infra-inference-engineer if self-hosted (mod-103 Ch 09).
  Rollback trig:  3 consecutive hours > 2× target OR 24 h at > 1.5× target.
  Review cadence: weekly in the pilot standup (mod-112).

TPOT (interactive)
  Metric:         p95 average inter-token latency
  Target:         < 30 ms/token
  Rationale:      Streaming UX feels smooth below the SME reading rate; 30 ms
                  guarantees under-chunk render cadence for the current UI.
  Measurement:    OpenTelemetry span 'llm.request' with gen_ai.usage metrics
                  and derived per-token timing.
  Error budget:   5% over 30-day window.
  Owner (FDE):    <FDE name>
  Rollback trig:  as above.
  Review cadence: weekly.

Completion latency (interactive)
  Metric:         p95 e2e completion latency at pilot UI edge
  Target:         < 6 s
  Rationale:      Manual-alternative latency is ~30 min. 6 s p95 is well under
                  SME workflow tolerance derived from discovery (~15 s).
  Measurement:    Browser Navigation Timing + API trace propagation.
  Error budget:   5% over 30-day window.
  Rollback trig:  as above.

Cost per task
  Metric:         Average cost per completed draft (input + output + cached).
  Target:         < $0.08
  Rationale:      value_per_task = $4 (SME time avoided × hourly cost); cap = 2%.
  Measurement:    Gateway audit log (Chapter 03); aggregation daily.
  Error budget:   10% monthly overrun before alert.
  Owner (FDE):    <FDE name>
  Rollback trig:  7-day moving average > 1.5× target.
  Review cadence: weekly.

Cost per tenant per day
  Metric:         Aggregated spend per tenant per calendar day.
  Target:         < $N per 100 SMEs in tenant
  Rationale:      Commercial-agreement envelope.
  Measurement:    Gateway budget ledger + audit log rollup.
  Error budget:   per the commercial agreement.
  Rollback trig:  hard-cap triggers per Chapter 03.

Quality (forward-reference mod-107)
  Metric:         <per-task quality metric from mod-107 gold-set>
  Target:         <TBD by mod-107's eval protocol>
  Rationale:      mod-107 owns the depth.
  Measurement:    offline + online eval, per mod-107.
  Review cadence: weekly.

Sign-off:
  - FDE:              <name>  Date: <YYYY-MM-DD>
  - Customer sponsor: <name>  Date: <YYYY-MM-DD>
  - Peer specialists (if applicable per mod-103 Ch 09): <name>  Date: <YYYY-MM-DD>
```

The numbers in the block above are illustrative. In a real pilot, every number is derived from the three customer-side facts and defended in the ADR; nothing is a default.

## How the SLO ties to the gateway and the observability stack

### Measurement location matters

TTFT measured at the SDK (inside the Python/TypeScript client) is different from TTFT measured at the gateway (Chapter 03) is different from TTFT measured at the user's browser. The three can diverge by meaningful margins.

- **At the SDK** — fastest to instrument but hides network to the gateway and from the gateway to the user.
- **At the gateway** — captures provider latency and gateway overhead; misses the pilot-to-gateway hop and gateway-to-user hop.
- **At the user edge** — captures everything the user experiences; requires browser or app instrumentation.

The FDE picks measurement locations per metric and documents them in the SLO section. Common pattern: gateway-side for TTFT and TPOT (the provider is the dominant cost; the gateway hop is sub-millisecond LAN); user-edge for total completion latency.

### OpenTelemetry GenAI semantic conventions

The [OpenTelemetry GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) define attribute names for LLM spans — operation name, model, usage tokens, response first-token time. The FDE's SLOs reference these attribute names so dashboards and alerts are portable across observability vendors. Mod-110 revisits this in depth.

The gateway (Chapter 03) emits spans using these conventions; the pilot code propagates trace context to the gateway; the observability stack (Datadog, Grafana, Honeycomb, customer-specific) stores and alerts on them.

### Error budgets

The Google SRE Book's [service level objectives](https://sre.google/sre-book/service-level-objectives/) and [embracing risk](https://sre.google/sre-book/embracing-risk/) chapters are the reference. A 5% error budget over a 30-day rolling window on a p95 latency SLO means: the pilot can be above target 5% of the time and still be in SLO. Exhausting the budget triggers process — pause risky changes, dig into what's driving the burn — not immediately a pager.

The FDE's overlay on error-budget management:

- The customer owns the consequence of budget exhaustion with the vendor, not the other way around. Discuss the customer's tolerance in the pilot scope.
- A fast-burn rule (e.g. 20% of budget consumed in an hour) triggers pager; a slow-burn rule (e.g. 100% consumed over a week) triggers process. Mod-110 writes the specific thresholds.

## Capacity planning under the SLO

The FDE does not own capacity planning at serving scale — [`ai-infra-inference-engineer`](https://github.com/ai-infra-curriculum/ai-infra-inference-engineer-learning) (peer, level 35, Infra family) does. For self-hosted serving, the SLO drives the capacity question, and the FDE briefs the peer with the shape from mod-103 Chapter 09.

Brief shape, SLO-focused:

| Field | Example |
|---|---|
| Model family and version | Llama 3.1 70B Instruct, bf16 |
| Expected QPS | 5 QPS p50, 15 QPS p95 at peak |
| Latency SLO at steady state | p95 TTFT < 1500 ms, p95 TPOT < 30 ms/token, p95 total < 6 s at 2048-token output |
| Input prompt size distribution | median 4k tokens, p95 12k tokens (cached prefix 3k) |
| Concurrency distribution | per-session ~1 concurrent; tenant concurrency variable |
| Burst behaviour | Monday morning spike expected; no specific headroom required |

The brief is iterative. The peer may come back with a realistic floor the FDE did not know (e.g. "p95 TTFT 1500 ms is achievable only on L40S or better; A100-40G sits at 2000 ms"). The SLO either (a) adjusts, with customer sign-off, or (b) drives the hardware choice.

For hosted-provider pilots, the capacity question is simpler but not absent: provisioned throughput on hyperscaler stacks (Bedrock PTC, Azure OpenAI PTUs) is a separate commercial envelope the FDE scopes against the pilot's projected load.

## Anti-patterns

**Committing an SLO without the peer specialist's sign-off.** The FDE commits p95 TTFT < 1 s on a 70B model; the inference engineer sees a realistic floor at 1.8 s; the customer is told two weeks later that the SLO has to move. The peer signs off before the SLO lands in the pilot scope.

**Measuring TTFT at the SDK rather than at the user's edge.** The SDK number looks good; the user's actual experience is worse. The SLO section always names the measurement location; the gap is what mod-110 helps you close.

**Conflating availability SLO with latency SLO.** "99.9% availability" and "p95 latency < 2 s" measure different things; one fast request every nine minutes satisfies availability-of-sort but not usefulness. Contract both when both matter.

**Promising p99 before the pilot has traffic.** Pilot traffic in week 2 is five hand-picked SME tests; a p99 claim on that sample is meaningless. Baseline p99 after mod-109's canary has real traffic (hundreds of requests); contract p99 after baseline.

**SLOs without rollback triggers.** An SLO that no one can act on is a vanity number. Every SLO block in the section above has a rollback-trigger line; without it, the SLO is not operational.

**Forgetting cost SLOs.** Latency SLOs are glamorous; cost SLOs are not. Cost SLOs are what make the pilot commercially viable; forget them and the commercial conversation in week 10 goes badly.

## Summary

- The FDE's SLO contract is per-customer, grounded in the customer's workflow and business-KPI value-per-task, not inherited from in-house defaults.
- Latency decomposes into TTFT, TPOT, and e2e completion; percentiles (p50 / p95 / p99) capture the distribution the user experiences, with p95 the standard contract handle.
- Cost decomposes into per-task, per-session, per-tenant-per-day, per-tenant-per-task-type; each ties to a specific business decision (per-call guardrail, commercial envelope, rollout prioritisation).
- The derivation flows from three facts: manual-alternative baseline, SME workflow tolerance, business-KPI value-per-task. Every number in the SLO section is defended in the pilot-scope ADR.
- Each SLO block names the metric, target, rationale, measurement location, error budget, owner, rollback trigger, review cadence.
- Measurement location matters — SDK vs. gateway vs. user edge diverge; the SLO section specifies which.
- The gateway (Chapter 03) emits OpenTelemetry GenAI-semconv spans; the observability stack (mod-110) stores them and alerts on them.
- Capacity planning for self-hosted serving is owned by `ai-infra-inference-engineer`; the FDE briefs the peer with the SLO-shaped table from mod-103 Chapter 09.
- Anti-patterns: committing SLOs without peer sign-off, SDK-only measurement, availability-vs-latency confusion, early p99 promises, SLOs without rollback triggers, forgetting cost SLOs.
- Chapter 06 is next: when the SLO gap is big enough to escalate to fine-tune or adapter workflows, and how to brief the peer who owns the training.
