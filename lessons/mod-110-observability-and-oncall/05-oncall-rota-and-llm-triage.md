# Chapter 05 — The Shared On-Call Rota and LLM-Specific Failure-Mode Triage

## Why this chapter exists

Chapter 04 wrote the SLOs and their burn-rate alerts. Those alerts have to route *somewhere*, and the somewhere is a rotation of named humans who answer the page, open the right dashboard, run the right playbook, and — critically — know when to pull in the customer-side operator or the vendor FDE. The rotation is **joint**: the customer's operations team and the vendor's on-call share the pager, not in the sense that both get every page, but in the sense that for every alert there is a defined Tier 1, Tier 2, and Tier 3, with the customer owning Tier 1 by handoff and the FDE owning Tier 3 under a named SLA ([mod-109 Chapter 06](../mod-109-deployment-rollout-change-management/06-customer-side-handoff-and-sme-training.md)).

LLM pilots introduce failure modes a generic SRE playbook does not cover. The gateway reports healthy 2xx, the quality SLO burns. The retrieval index returned documents, but all of them are stale. A new prompt template is live; refusals spiked 10×. The upstream provider rolled a silent update; p99 latency doubled. A judge model drifted; the online judge pass-rate ticks down three points without a bundle change. The cost dashboard shows a tenant has burned the month's envelope by day eight. Each is a distinct incident shape with a distinct first move; the generic "something is broken" playbook does not help.

This chapter has four jobs. Design the joint rotation (tiers, handoff, overlap window, on-call tool integration). Write the SEV classification that routes LLM-specific failure modes. Catalogue the six canonical LLM failure modes with their playbooks. Specify the incident-command and status-page communication protocol for a shared customer-production system.

## The joint rotation

Three tiers, split across two organisations.

**Tier 1 — Customer operations analyst (business hours) + customer on-call (24×7).** Owns the first response to every alert the pilot generates. Reads the dashboards from Chapter 03; runs the playbook linked from the alert; acknowledges within the SLA's acknowledgement window; escalates if the playbook does not resolve within the time box.

**Tier 2 — Customer-side SRE / platform engineer.** Owns the second response when Tier 1 exhausts the playbook or when the issue is infrastructural (collector down, LLM-obs platform unreachable, gateway down, model-provider outage of the customer's cloud account). Can modify runtime configuration; owns the kill-switch flip authority ([mod-109 Chapter 03](../mod-109-deployment-rollout-change-management/03-feature-flag-strategy.md)).

**Tier 3 — Vendor FDE.** Owns the third response — consulting on prompt / bundle / retrieval / judge issues, pattern recognition across other pilots, escalation to the vendor's product team and to `ai-platform-engineer` when a vendor-side change is required. Business-hours response during the pilot's post-handoff steady state; emergency response for SEV-1 any time.

The tier model inverts what new FDEs expect. The customer's team is Tier 1 because they are the pilot's owner once handed off; the FDE is Tier 3 because the vendor's role is specialist consultation, not primary operations. This is the discipline from [mod-109 Chapter 06](../mod-109-deployment-rollout-change-management/06-customer-side-handoff-and-sme-training.md); Chapter 05 makes it concrete.

### Handoff window

Pilots transition from "FDE as Tier 1" during active pilot to "FDE as Tier 3" at handoff. The transition is a sliding window, typically eight to twelve weeks.

| Week | Tier 1 | Tier 2 | Tier 3 |
|---|---|---|---|
| 0 (pilot start) | Vendor FDE | Vendor FDE | Vendor product |
| 4 (SME training) | Vendor FDE + customer operations analyst (shadow) | Vendor FDE | Vendor product |
| 8 (soft handoff) | Customer operations analyst + vendor FDE (co-owns) | Customer SRE + vendor FDE | Vendor product |
| 12 (hard handoff) | Customer operations analyst | Customer SRE | Vendor FDE |
| 16+ (steady state) | Customer operations analyst | Customer SRE | Vendor FDE |

The hard-handoff is a ceremony, not a date — the customer's operations lead signs when Tier 1 is theirs. After the ceremony, a page on the pilot goes to the customer first and reaches the FDE only through escalation.

### The on-call tool

The rotation lives in the customer's on-call tool, not the vendor's. Common tools: [PagerDuty](https://www.pagerduty.com/), [Opsgenie](https://www.atlassian.com/software/opsgenie), [ServiceNow Major Incident Management](https://www.servicenow.com/products/it-operations-management/major-incident-management.html), [FireHydrant](https://firehydrant.com/), [incident.io](https://incident.io/), [Rootly](https://rootly.com/), [xMatters](https://www.xmatters.com/). The FDE integrates into the customer's tool as a service in the customer's PagerDuty / Opsgenie / etc, not as an external escalation contact. Three reasons:

1. The audit trail stays in the customer's tool, which the customer's compliance function already reviews.
2. The status-page integration and the post-mortem tool chain (Chapter 06) chain off the on-call tool; the FDE's own tool would create a parallel world.
3. The customer's SEV classification is the taxonomy the pilot inherits; shoehorning the vendor's SEV language is the handoff anti-pattern.

### Pager routing from alerts

Each alert from Chapter 04 emits with routing metadata:

```yaml
# Alert manifest for the fast-burn SLO alert on online-judge-pass-rate
alert:
  id: slo.online-judge-pass-rate.fast-burn
  slo_id: pilot-member-assist.quality.online-judge-pass-rate
  severity_default: SEV-2
  tier_1: customer-operations-oncall
  tier_2: customer-sre-oncall
  tier_3: vendor-fde
  escalation:
    - after: 15m without ack → tier_2
    - after: 30m without ack → tier_3
    - after: 1h unresolved → incident commander
  runbook: ../runbooks/quality-drop.md
  dashboard: https://langfuse.pilot.internal/dashboards/judge-pass-7d
  silence_window_during_canary: true
```

The YAML lives in the artefact bundle manifest. Every alert in production has a runbook link, a dashboard link, and an explicit escalation path. The runbook is tested in the drill (below) before the alert goes live.

## SEV classification for LLM pilots

Every customer has a SEV taxonomy; the FDE inherits it. The taxonomy varies, but the shape is consistent: SEV-1 is an outage with major customer impact; SEV-5 is cosmetic. The LLM-specific mapping is the FDE's contribution to the customer's existing taxonomy.

A representative mapping (adjusted to the customer's taxonomy):

| SEV | Definition | Example LLM incident |
|---|---|---|
| **SEV-1** | Pilot wholly down OR producing demonstrably harmful output in production | Gateway down for all tenants; or a prompt regression produces PII leaks; or the judge pass-rate crashed to < 20 %. Response: page immediate; war room; executive notification. |
| **SEV-2** | Major degradation; SLO burning fast; user-visible | Quality SLO burning fast (fast-burn alert); upstream provider outage; refusal spike > 3× control. Response: page; dedicated incident channel; hourly comms. |
| **SEV-3** | Minor degradation; SLO burning slow; limited user impact | Slow-burn alert on acceptance rate; cost SLO over 1.5×; retrieval latency slow for one tenant. Response: ticket; batch response; daily comms. |
| **SEV-4** | Near-miss or telemetry anomaly; no user impact | Collector lag spike; one dashboard renders blank; judge drift signal without quality move. Response: ticket; next-business-day. |

Two LLM-specific SEV rules.

**Rule 1 — PII / PHI leak is always SEV-1.** A trace in a non-content-cleared destination contains raw PHI; a completion embedded a user's SSN that should have been masked. The data contract was breached. SEV-1 regardless of volume.

**Rule 2 — Cost blowout crossing the monthly envelope is SEV-2.** A tenant blew their daily cap and the rate-limit kicked in — SEV-3 (planned response). The pilot blew the monthly envelope at mid-month — SEV-2, because the sponsor's budget commitment is at stake.

## The six canonical LLM failure modes

Every LLM incident resolves to one of six shapes. Each has a dedicated runbook. The runbooks live in the customer's runbook tool (Confluence, Notion, the customer's IR tool's built-in runbooks); the FDE ships a template per shape and the customer customises for their context.

### Failure mode 1 — Prompt regression

**Shape.** A new bundle promoted; the online judge pass-rate drops; the acceptance rate drops; refusals may spike. A regression is a change-driven failure — the cause is almost always a recent promotion.

**Triage.**

1. Open the bundle-version dashboard; identify the active bundle and the previous bundle.
2. Open the SLO burn panel from Chapter 04; identify which SLOs are burning (quality metrics alone vs quality + refusal + cost).
3. Open the LLM observability platform's trace view, filtered to the active bundle, judge-score below threshold, ordered by time.
4. Sample 20 traces; identify the failure pattern (format regression? hallucinated reference? refusal when asked? tone change?).
5. Compare against traces from the previous bundle on matching inputs.
6. If the pattern is clear and the previous bundle looked good: **roll back** via the kill-switch ([mod-109 Chapter 03](../mod-109-deployment-rollout-change-management/03-feature-flag-strategy.md)); page the release manager; schedule a post-mortem.
7. If the pattern is unclear: escalate to Tier 2 / Tier 3 for a joint review; the on-call does not roll back unilaterally when the signal is noisy.

**Common root causes.** Prompt-template change that didn't pass the full eval set; an unversioned prompt edit that reached production; a model pin change; a tool-allow-list narrowing that broke an intent class.

### Failure mode 2 — Provider outage

**Shape.** Model provider's API returns 5xx or elevated latency; error rate spikes; TTFT / p95 breach. Entire tenants affected if the pilot only uses one provider; some tenants affected if the pilot uses provider-fallback.

**Triage.**

1. Open the provider's status page ([anthropic.com/status](https://status.anthropic.com), [status.openai.com](https://status.openai.com), [health.aws.amazon.com](https://health.aws.amazon.com), [status.azure.com](https://status.azure.com), [status.cloud.google.com](https://status.cloud.google.com)).
2. Confirm the error pattern matches — is it 5xx, is it rate-limit (429), is it timeouts?
3. If the gateway supports provider-fallback, confirm fallback engaged (via Chapter 02 traces: `gen_ai.system` showing the fallback provider).
4. If no fallback: the gateway's queue depth is increasing; the gateway's rate-limit response is sending callers graceful 503s.
5. Communicate: customer status page update ("We are observing elevated latency on the Member Assist service due to an upstream provider issue."), SEV-2 if widespread.
6. Monitor; stand down when provider's status returns to normal.
7. Post-mortem: was the fallback configured? did it engage? did the SLO absorb correctly? did the customer feel the breach?

**Common root causes.** Provider-side regional degradation; regional capacity exhaustion; rate limit hit after a traffic spike; API breaking change (rare — providers are careful about this; still happens on experimental endpoints).

### Failure mode 3 — Retrieval drift

**Shape.** Quality SLO drifting down over days / weeks without a bundle change. User-and-SME feedback mentions "the assistant is referring to old information." Judge scores drift; retrieval p95 may or may not move.

**Triage.**

1. Open the retrieval dashboard; check index freshness ([mod-105 Chapter 03](../mod-105-grounding-on-customer-data/) territory).
2. Check the indexing pipeline's state: last successful run, backlog, errors.
3. Sample 10 recent judge-fail traces; look at the retrieved documents' timestamps.
4. If the index is stale: identify the staleness cause (crawler failure, data-source permission change, embedding-model unavailable).
5. If the index is fresh but retrieval quality has dropped: investigate reranker health, embedding-model drift (per [mod-108 Chapter 06](../mod-108-enterprise-security-and-compliance/06-ai-supply-chain-provenance.md) — did the embedding model get silently updated?).
6. Short-term: update the index or roll back the embedding-model change.
7. Medium-term: investigate monitoring to catch staleness earlier.

**Common root causes.** Indexing pipeline failed silently; data source permission revoked; embedding model silently updated; reranker configuration change; document corpus changed shape (new document class the retriever hasn't seen).

### Failure mode 4 — Hallucination spike

**Shape.** Judge scores holding; SME feedback flagging specific traces as "model made up this fact." The signal is in SME annotations and user complaints, not always in automated metrics.

**Triage.**

1. Open the LLM observability platform's SME-annotation view; filter to the past week's `hallucination` annotations.
2. Look for patterns: a specific intent class, a specific document type, a specific user population.
3. Sample 10 hallucination traces; identify the common shape (fabricated citation? wrong numeric? confabulated process?).
4. Hypothesise cause: retrieval returning wrong docs (which looks like a hallucination at generation); prompt not firmly requiring citation; model-change silently loosened grounding; a new intent class not covered by the gold set.
5. Short-term: tighten the prompt (if the pilot's release cadence permits), restrict the tool-call surface, add a guardrail check for cited-sources.
6. Medium-term: a new judge rubric specifically for hallucination; back-feed the hallucination traces into the gold set (Chapter 06); run them through the next eval.

**Common root causes.** Retrieval mis-selection (model wasn't given the right context); prompt design (no "cite your source" discipline); model drift; genuine out-of-distribution question.

### Failure mode 5 — Cost blowout

**Shape.** Cost SLO burning fast. One tenant's daily cost 5× the baseline; or every tenant's cost up 30 % overnight.

**Triage.**

1. Open the cost dashboard; identify the dimension — which tenant, which model, which prompt-template, which rollout arm.
2. If one tenant: open their trace view; look at per-invocation token counts; a single-tenant spike is usually a user running a script or a prompt-injection trying to extract system behaviour.
3. If every tenant, per-model: a provider rate change (check `pilot.cost.rate_card.version`); or a model pin change that moved to a more expensive model.
4. If every tenant, per-prompt-template: a template change increased output length (common when a template adds "explain step-by-step").
5. If no clear dimension: look at prompt-cache-hit-rate; a drop in cache hits (prompt restructuring, cache TTL change) is a common silent cost multiplier.
6. Short-term: rate-limit the offending tenant at the gateway, roll back the template / model pin change, re-tune cache-friendly prompt structure.
7. Medium-term: cost-per-task SLO at tighter threshold going forward.

**Common root causes.** Prompt-template change that increased output length; model-pin change to a more expensive model; rate-card update; prompt-cache miss rate increase; a user or external caller pathologically expensive.

### Failure mode 6 — Guardrail false-positive storm

**Shape.** Guardrail trip-rate SLO burning high; SMEs complaining that answers they expect are blocked; user escalations referencing "the assistant won't help with X."

**Triage.**

1. Open the guardrail dashboard; identify which guardrail rule / category is the top source of trips.
2. Sample 10 recent trips; SME review each — are these actually blocks or false positives?
3. If false positives dominate: identify the cause — a new guardrail rule deployed; the underlying classifier updated (per supply-chain provenance); the user population or intent distribution shifted.
4. Short-term: narrow the guardrail rule or reduce its sensitivity; SME-led review of each false-positive class.
5. Medium-term: a guardrail-specific judge rubric; back-feed the false-positive cases into the eval gate so next guardrail change tests against them.

**Common root causes.** New guardrail rule shipped without enough eval coverage; classifier model drift; a legitimate user intent that pattern-matches the rule's language.

### Playbook structure

Each playbook (one per failure mode) is a short Markdown document with:

- **Shape.** One-paragraph description of the symptom.
- **Dashboards.** Direct links to Langfuse / Datadog / Splunk / etc. panels that visualise the shape.
- **Triage steps.** Ten or fewer, numbered.
- **Escalation triggers.** Conditions that force a tier escalation.
- **Rollback mechanism.** Named flag, named command, named authority.
- **Known false positives.** Scenarios that look like this failure mode but aren't.
- **Post-mortem template section.** The lines the post-mortem must fill in.

Playbooks are tested quarterly in a drill (below). An untested playbook is not a playbook.

## Incident command on a shared system

When a SEV-1 or SEV-2 is declared, the pilot runs a formal incident process. Roles:

- **Incident Commander (IC).** Customer-side. Owns the overall response, decides on actions, calls the all-clear. The customer's standard IC on duty; not the FDE.
- **Scribe.** Customer-side. Documents the timeline in the incident-management tool — every action, every observation, every decision — with timestamps.
- **Communications Lead.** Customer-side. Owns the status-page update cadence, internal stakeholder comms, executive updates.
- **LLM Subject-Matter Expert (SME).** Vendor FDE or customer SME. Interprets model behaviour, prompt / retrieval / judge diagnostics, recommends technical mitigations.
- **Operations.** Customer SRE / platform engineer. Runs runbook steps, flips flags, scales infrastructure.

The IC is always customer-side; the FDE is always the LLM-SME role. A pilot where the FDE is the IC is a pilot that has not handed off.

### Joint incident channel

A dedicated channel per incident — `#inc-<id>-<pilot>` — in the customer's chat tool (Slack, Teams, Mattermost). The vendor FDE joins as a guest user per the customer's access policy; the customer's incident-management tool mirrors every chat message into the incident timeline. The channel closes automatically when the incident is resolved; the chat log attaches to the post-mortem.

### The incident timeline

Every action lands in the timeline. The incident-management tool structures it; the scribe fills it. A minimal timeline entry:

```
2026-11-04T14:03Z | alerted | slo.online-judge-pass-rate.fast-burn fired on pilot-member-assist, SEV-2
2026-11-04T14:04Z | ack    | tier_1_oncall (jsmith@customer) ack within 1m SLA
2026-11-04T14:05Z | triage | opened runbook prompt-regression.md; last bundle promotion at 13:45Z (v4.2.0)
2026-11-04T14:12Z | finding| sampled 20 judge-fail traces; 18 show format regression (bullet-list → paragraph) matching v4.2.0 change
2026-11-04T14:14Z | decide | rollback to v4.1.3 via killswitch
2026-11-04T14:15Z | action | killswitch flipped by jsmith@customer; audit logged
2026-11-04T14:20Z | observe| judge-pass rate recovering; current 5m window at 0.79 (target 0.80)
2026-11-04T14:35Z | observe| judge-pass rate stable at 0.83 (baseline)
2026-11-04T14:40Z | comms  | status page updated: resolved
2026-11-04T14:42Z | close  | incident resolved; post-mortem scheduled for 2026-11-06
```

Chapter 06 picks up the post-mortem.

### Status-page communication

A status page is public-facing for Archetype A (healthcare, member-facing) or Archetype C (bank, customer-facing); internal-only for Archetype B (federal, agency-internal).

The cadence:

- **Investigate.** First update at acknowledgement, naming the component, the user-visible symptom, and the impact. "We are investigating slower responses on Member Assist. Members may see delays answering common questions."
- **Ongoing.** Updates every 30 minutes during an unresolved SEV-2; every 15 minutes during an unresolved SEV-1.
- **Mitigated.** When the immediate harm is stopped (e.g. rollback complete), update: "We have deployed a fix for the slower responses; we are monitoring for recovery."
- **Resolved.** When the SLO burn has stopped and metrics have returned to baseline, update: "Member Assist is operating normally. A detailed post-mortem will be shared within 5 business days."

The communications lead writes the updates; the FDE reviews technical accuracy; the customer's PR / regulatory team reviews for external language. Three lines the FDE writes once per pilot and the comms lead reuses:

- The investigate line template (per failure mode).
- The mitigated line template (per failure mode).
- The resolved line template (per failure mode).

### DORA incident reporting for Archetype C

A major ICT-related incident under DORA has specific reporting obligations. The bank's major-incident process already captures this; the FDE's contribution is making sure the pilot's incident classification maps to the bank's DORA taxonomy. The timing — initial, intermediate, final — and the content fields the bank submits to the ESAs are the bank's responsibility; the FDE provides the technical narrative. <!-- needs-research: confirm current DORA RTS on major ICT-related incident classification thresholds, reporting windows (initial / intermediate / final), and the ESA ingestion mechanism -->

### FedRAMP incident reporting for Archetype B

FedRAMP continuous monitoring and the agency's ATO change control require specific incident reporting for events that touch controlled information. The agency's own IR process captures this; the FDE provides the technical narrative; the POA&M item (per [mod-108 Chapter 03](../mod-108-enterprise-security-and-compliance/03-vertical-compliance.md)) opens as a corrective action if a control was breached. <!-- needs-research: confirm current FedRAMP incident reporting obligations and timelines -->

### HIPAA breach-notification for Archetype A

The HIPAA Breach Notification Rule has specific timing (60 days to notify affected individuals in most cases; shorter in some states; HHS and sometimes media notification). The customer's privacy office owns this; the FDE provides the technical narrative on whether the breach was acquisition, access, use, or disclosure of PHI. The single most important job: catch any incident where PHI left its compartment and classify it correctly so the privacy office's timer starts at the right moment. <!-- needs-research: confirm current HIPAA Breach Notification Rule timelines (45 CFR 164.404-408); state addenda vary -->

## Drill cadence

An on-call rota that has never answered a page is a rota that will answer the first page badly. The FDE schedules drills quarterly, co-run with the customer's operations lead.

**Drill 1 — Prompt regression.** Deploy a known-bad prompt to a staging tenant; the alerting fires; the on-call runs the playbook end-to-end to rollback. Measure: time-to-ack, time-to-rollback, playbook completeness.

**Drill 2 — Provider outage.** Simulate upstream outage (gateway configured to return 503s from the configured provider); verify fallback or graceful degradation; verify status-page flow.

**Drill 3 — Cost blowout.** Fire a synthetic tenant producing 10× expected volume; verify the cost SLO alerts fire; verify rate-limit kicks in.

**Drill 4 — Hallucination escalation.** Insert a known-fabricated trace into the LLM observability platform via a test harness; verify the SME-feedback path opens a ticket; verify back-feed to the gold set.

Each drill produces a report: what worked, what the runbook missed, what the dashboard needed. Playbook edits land in the manifest.

## Running the three archetypes

**Archetype A — US healthcare payer.** Rotation in the customer's PagerDuty; FDE on Tier 3 under the handoff SLA; public Statuspage.io for member communication; HIPAA breach-notification path integrated with the customer's privacy office; drills quarterly.

**Archetype B — US federal agency.** Rotation in the agency's IR tool (potentially ServiceNow Major Incident Management or an agency-specific system); FDE on Tier 3 with consultation-only authority; no public status page; FedRAMP continuous-monitoring reporting integrated with the agency's ISSO; drills aligned with the agency's annual tabletop exercise.

**Archetype C — EU retail bank.** Rotation in the bank's IR tool (ServiceNow or PagerDuty); FDE on Tier 3 under the DORA-aligned SLA; bank-internal status page; DORA major-ICT-incident reporting integrated with the bank's incident-response function; drills quarterly aligned with the bank's operational-resilience programme.

## Anti-patterns

**Vendor FDE as Tier 1 indefinitely.** No ceremony marked the handoff; every page still reaches the FDE first. The pilot never ended its ramp. Chapter 05 requires a hard-handoff ceremony signed by the operations lead.

**Alerts without runbooks.** The page arrives with a metric name, no context, no linked playbook. The on-call searches Confluence. The time-to-ack becomes time-to-google. Every alert in production has a runbook link and a dashboard link; both are tested in a drill.

**One all-purpose "LLM incident" playbook.** The playbook is generic: "open the dashboard; investigate; fix." The six LLM failure modes have six different first moves; conflating them loses 30+ minutes to triage every incident.

**Vendor's SEV language inside the customer's tool.** The alert fires with "SEV-A" (vendor's internal taxonomy); the customer's tool expects "SEV-1". Severity routing breaks. Chapter 05 inherits the customer's SEV taxonomy and translates at the alert boundary.

**Status page updated only at resolution.** Members feel the outage; the status page is silent; the support line lights up. The communications cadence is investigate → ongoing every 30 min → mitigated → resolved, from acknowledgement, not from fix.

**Scribe-less incidents.** The IC drives the response; nobody writes the timeline; the post-mortem is a reconstruction. The scribe role is non-optional; without a scribe, the post-mortem's regression fixture (Chapter 06) can't be sourced.

**Vendor FDE as Incident Commander.** The FDE knows the model; the customer knows their business. The IC is the customer's role; the FDE is the SME. Reversing this is the single most common handoff anti-pattern after Tier 1 inversion.

**Drill-free rotation.** The rotation exists on paper; the on-call has never answered a synthetic page. The first real incident is the drill. Chapter 05 requires quarterly drills with named outcomes.

**No DORA / HIPAA / FedRAMP integration.** The incident resolves; two weeks later the compliance office asks "why did I only hear about this through channels?" because the regulatory timer required reporting in 72 hours. The FDE's job: make sure every incident's classification touches the customer's compliance intake on the same day it's declared, not on the day of the post-mortem.

## Summary

- The on-call rota is joint: Tier 1 customer operations analyst (24×7), Tier 2 customer SRE, Tier 3 vendor FDE. The handoff from "FDE as Tier 1" to "FDE as Tier 3" slides over 8–12 weeks and ends in a signed ceremony.
- The rotation lives in the *customer's* on-call tool with the customer's SEV taxonomy. The FDE's team shows up as an escalation service, not a parallel universe.
- Six canonical LLM failure modes with distinct playbooks: prompt regression, provider outage, retrieval drift, hallucination spike, cost blowout, guardrail false-positive storm. Each has dashboards, triage steps, escalation triggers, rollback mechanism, and known false positives.
- Every alert in production carries a runbook link, a dashboard link, and explicit tier routing. Alerts without runbooks are dropped from production.
- Incident command on a shared system: customer-side IC + scribe + comms lead + operations; vendor FDE as LLM SME. The incident channel is in the customer's tool with the FDE as a guest.
- Status-page cadence: investigate → ongoing every 30 min (SEV-2) or 15 min (SEV-1) → mitigated → resolved. Templates per failure mode written once by the FDE, reused by the comms lead.
- Regulatory reporting integration: DORA major-ICT-incident for Archetype C, FedRAMP continuous-monitoring for Archetype B, HIPAA breach-notification for Archetype A. The FDE's job is accurate classification; the customer's compliance function runs the clock.
- Drills quarterly: prompt-regression, provider-outage, cost-blowout, hallucination-escalation. Each produces a report; playbook edits land in the manifest.
- Nine anti-patterns: FDE-as-Tier-1 forever, alerts without runbooks, one-size playbook, vendor SEV language, late status-page updates, scribe-less incidents, FDE-as-IC, drill-free rotation, missed regulatory reporting.
- Chapter 06 is next: blameless post-mortems and the regression-fixture / gold-set back-feed that turns every incident into a permanent memory inside the eval gate.
