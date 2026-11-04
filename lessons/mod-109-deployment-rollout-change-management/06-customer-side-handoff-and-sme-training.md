# Chapter 06 — Customer-Side Handoff: Admin Dashboards, SME Training, Playbooks, Escalation

## Why this chapter sits here

Mod-107 answered "is this good enough?" Mod-108 answered "is this safe enough that the CISO / DPO / model-risk committee / auditor will sign?" The earlier chapters of mod-109 shipped the rollout infrastructure — deployment-mode selection (Chapter 02), the per-tenant flag taxonomy with CAB-tied audit (Chapter 03), the LLM-specific canary and shadow design (Chapter 04), the versioned artefact bundle with replay (Chapter 05). The signed-off pilot is now in production. The FDE's remaining job is to make the system run after the FDE goes home.

The customer-side operator — an operations lead inside the customer organisation, often an SME manager rather than an engineer — must be able to drive it. That means a dashboard they can read without help, a training programme that gets their cohort to competency, playbooks for the failure modes the system actually has, and an escalation ladder that routes a page to the right tier instead of back to the FDE.

This chapter owns the *engineering interfaces* the operator uses. It names the boundary with [`ai-enablement-lead`](https://github.com/ai-governance-curriculum/ai-enablement-lead-learning) (peer, level 35, Governance / Enablement family) — the peer who owns the *organisational* change-management slice: programme design, benefit-realisation reporting against the mod-102 commitments, exec-sponsor cadence, train-the-trainer, resistance management. The admin dashboard, the playbook library, the replay guide — FDE. The programme itself — peer. Chapter 08 writes the full peer boundary; this chapter writes the handoff artefacts.

<!-- needs-research: confirm the current repo URL for ai-enablement-lead-learning in the ai-governance-curriculum org before publishing -->

## The admin dashboard

The admin dashboard is the single surface the customer's operations lead reads daily. It is not a Grafana link, not an opaque vendor console — it is an SME-readable surface that lives in the customer's environment (or at minimum under customer SSO) and names only the things an operator can act on.

```
┌──────────────────────── Member-Inquiry Admin ────────────────────────┐
│ [ACTIVE INCIDENT: None]                        Bundle: v2.1.0-stable │
├──────────────────────────────────────────────────────────────────────┤
│  TODAY        Volume: 4,217    SMEs active: 23    Workflows: 1       │
│  ─────                                                               │
│  Judge pass-rate ────── 91.3%  (gate: ≥85%)   [trend sparkline]      │
│  Refusal-rate ───────── 2.1%   (gate: ≤5%)                           │
│  p95 latency ────────── 1.8s   (SLO: ≤3.0s)                          │
│  Cost / resolved call  $0.14   (budget: ≤$0.25)                      │
├──────────────────────────────────────────────────────────────────────┤
│  KILL-SWITCHES (per flag)                                            │
│  [killswitch.member-inquiry.generator]  OFF (last flip: 2026-10-14)  │
│  [killswitch.member-inquiry.retriever]  OFF                          │
│  [killswitch.member-inquiry.tools]      OFF                          │
├──────────────────────────────────────────────────────────────────────┤
│  RECENT FLAG FLIPS                                                   │
│  2026-11-01  rollout.member-inquiry.reranker-v3  on->  jane.doe      │
│              reason: canary passed gate CAB-2026-10-0319             │
│  2026-10-14  killswitch.retriever                off-> ops-lead      │
│              reason: rerank regression resolved, re-enabling         │
├──────────────────────────────────────────────────────────────────────┤
│  REPLAY  [search: tenant / workflow / user / date / invocation-id ]  │
│          [button: Replay against current bundle]                     │
└──────────────────────────────────────────────────────────────────────┘
```

Required components, each sourced from a named upstream:

- **Active-incident banner.** Set by the on-call rotation; cleared by post-incident review. Default "None" is itself informative.
- **Bundle pin.** The current artefact-bundle version from Chapter 05 — single string, resolvable to prompt + retriever + model + tool allow-list + judge + guardrails + gateway config.
- **Today's volume panel.** Invocations today, by workflow and tenant; SMEs active today; workflows live. One number per line.
- **Eval-metrics panel.** Judge pass-rate, refusal-rate, p95 latency, cost-per-resolved-call. Each with its gate value, a current value, and a 24-hour sparkline. These numbers come from the mod-107 Chapter 05 online-eval surfaces; the dashboard does not recompute them.
- **Kill-switch controls per flag.** One row per kill-switch flag from Chapter 03, with current state and last-flip timestamp. A flip from this surface writes to the same CAB-tied audit log; the dashboard is not a bypass.
- **Flag-flip feed.** Reverse chronological. Each entry: flag name, old-state -> new-state, actor, reason, linked CAB ticket. Sourced from the flag platform's audit stream (Chapter 03).
- **Replay panel.** Search by tenant / workflow / user / date / invocation-id; "Replay against current bundle" button invokes the Chapter 05 replay API. Results render trajectory + diff.

The dashboard is SME-readable, not raw Grafana. Raw Grafana still exists for the vendor FDE and the customer platform team; mod-110 wires it. The admin surface is the operator's surface and it names only operator-actionable signals.

## SME training materials

Training is a layered curriculum, not a single session. Four layers, each with a named deliverable and a target duration. The FDE authors the engineering content; the [`ai-enablement-lead`](https://github.com/ai-governance-curriculum/ai-enablement-lead-learning) peer owns the programme — cohort schedule, train-the-trainer, refresh cadence, certification if the customer requires one.

**Layer 1 — Orientation (30-45 min).** Deliverables: slide deck + recorded walkthrough + one-page capability sheet. Target audience: every SME who will ever see the system. Learning objectives:

- What the system does, in the words of the mod-102 workflow definition.
- What the system does *not* do — the three or four cases the pilot deliberately excluded.
- Three failure modes to watch for, each tied to a visible signal in the SME UI.
- How to flag a bad answer — the override surface from mod-106 Chapter 05.

**Layer 2 — Workflow-specific (90-120 min hands-on).** Deliverables: facilitated session + exercise set + answer key. Target audience: SMEs who will handle cases daily. Learning objectives:

- Use the SME-facing UI end-to-end on a realistic case.
- Triage a case: accept, edit, override, escalate.
- Use the HITL override surface from mod-106 Chapter 05; see where the override lands in the audit log.
- Recognise the pilot's known edge cases and route them to the manual queue.

**Layer 3 — Operator (half-day).** Deliverables: sandbox tenant + hands-on lab + operator handbook. Target audience: the customer operations lead and their backup. Learning objectives:

- Read every panel on the admin dashboard — bundle, volume, eval metrics, kill-switches, flag-flip feed, replay panel.
- Flip a kill-switch in the sandbox tenant; see the entry land in the flag-flip feed.
- Trigger a replay against the current bundle and against a prior bundle; interpret the trajectory diff.
- Interpret the flag-flip feed well enough to narrate to the CAB what has changed this week.
- File a Tier 2 and Tier 3 escalation; see what information each tier needs to receive.

**Layer 4 — Governance (2-hour working session with customer compliance).** Deliverables: governance one-pager + audit-response template. Target audience: customer compliance, internal audit, model-risk, DPO. Learning objectives:

- How CAB approval flows from a proposed flag flip to a logged change.
- How to respond to an internal audit question with the replay panel and the flag-flip feed as primary evidence.
- How to pull replay evidence for a regulator question — SOC 2 CC6 logical-access review, HIPAA 164.312(b) audit controls, DORA ICT audit rights, EU AI Act Article 72 post-market monitoring. <!-- needs-research: confirm the SOC 2 Trust Services Criteria control CC6 reference remains current under the 2022 revision -->
- How the kill-switch audit log from Chapter 03 satisfies each of those audit controls.

The FDE *supports* this curriculum — authors the deck for Layer 1, writes the exercises for Layer 2, builds the sandbox for Layer 3, co-drafts the governance one-pager for Layer 4. The [`ai-enablement-lead`](https://github.com/ai-governance-curriculum/ai-enablement-lead-learning) peer *leads* the programme — delivery cadence, cohort composition, train-the-trainer, certification, refreshers after every bundle version bump.

## Playbooks for common failure modes

Playbooks live in the customer's runbook repository alongside the Chapter 07 handoff packet. Each has a one-sentence trigger and a 3-5 step response. All of them are written for the Tier 1 operator to execute without paging the FDE first; escalation is a step inside the playbook, not the first step.

**Judge pass-rate drop below gate.** Trigger: judge pass-rate panel red for 15 minutes.

1. Inspect the flag-flip feed for the last 24 hours — any rollout flag flipped on?
2. Check the canary status (Chapter 04) — is the canary arm driving the drop?
3. Pull a replay sample of the last hour's failing cases; look for a common pattern.
4. Flip the relevant `killswitch.*` to roll back to the last-known-good bundle.
5. Page vendor FDE (Tier 3) with the sample, the flag-flip feed, and the bundle versions.

**Model-provider outage.** Trigger: elevated error-rate from AWS Bedrock / OpenAI / Azure OpenAI / Google Vertex. <!-- needs-research: confirm the current list of model-provider status-page URLs each archetype relies on; vendors move these pages -->

1. Confirm against the provider's status page; log the incident reference.
2. If a secondary provider is configured (Chapter 02 dual-provider posture), fail over via the gateway flag.
3. Notify SMEs via the pre-arranged channel — case queue may back up.
4. Consider `killswitch.generator` to pause new drafts; keep the review-only mode live so SMEs can still close cases manually.
5. Document the outage window for the next CAB; attach to the monthly SLA report.

**Retrieval index stale.** Trigger: elevated refusal-rate, reduced grounding citations, or a direct customer-data-engineering notice.

1. Check the indexer pipeline's last-success timestamp (mod-105 Chapter 06 indexer health).
2. Trigger a re-index; watch the pipeline to completion.
3. If re-index fails, degrade to the last-known-good index snapshot via the retriever flag.
4. Page the customer's data-engineering team (Tier 2) with the pipeline failure log.

**Tool allow-list drift.** Trigger: a new MCP tool appears in the bundle that was not previously approved.

1. Freeze promotion — flip the relevant `rollout.*` flag to off.
2. Review the tool's declared scope against the mod-106 Chapter 04 allow-list spec.
3. Run the OWASP Agentic Top 10 check from mod-106 Chapter 06; attach results.
4. Reject or approve via a CAB ticket; do not back-door the approval through a flag flip.

**PII leak suspected.** Trigger: the PII detector fires, or an SME files a report that an output contained PII that should have been redacted.

1. Flip `killswitch.*` on the affected prompt immediately — do not wait for triage.
2. Preserve the invocation log and the replay inputs; do not delete.
3. Initiate the mod-108 Chapter 05 data-contract incident playbook.
4. Notify the customer's DPO per the DPA's breach-notification clause; the clock has started.
5. Open a Tier 3 page to the vendor FDE and the vendor security peer (mod-108 Chapter 07 `ai-infra-security`).

**Cost-per-call anomaly.** Trigger: anomaly detection on the cost panel — cost-per-resolved-call doubles from a 7-day baseline.

1. Identify the source — new prompt (longer context?), retrieval expansion (more chunks?), judge sampling rate (increased?), model upgrade (more expensive tokens?).
2. If source is a flag flip, flip it back; if source is organic (longer cases), re-cost the pilot against the mod-102 envelope.
3. If re-cost breaks the committed envelope, trigger a Chapter 07 handoff-SLA amendment.

**Customer data classification change.** Trigger: customer notice — a dataset the retriever indexes has been reclassified (e.g. internal -> restricted).

1. Pause the affected workflow via `killswitch.retriever`.
2. Review the retrieval topology against the new classification — does the current KMS / ACL posture still satisfy the new tier?
3. Re-run the mod-108 Chapter 02 DFD walk-through; record deltas.
4. Resume only after the data-contract (mod-108 Chapter 05) has been re-attested.

**Audit request from the customer's regulator.** Trigger: the customer receives a regulator letter requesting evidence on a named interaction, date range, or user.

1. Open a replay session; search by the fields named in the letter.
2. Pull invocations; render trajectory + bundle lineage per invocation.
3. Prepare an evidence packet matching the mod-107 Chapter 07 shape — decision document, replay transcripts, bundle manifest, flag-flip feed slice for the period, kill-switch audit excerpt.
4. Route the packet through the `ai-evaluation-engineer` peer (mod-108 Chapter 07) for regulator-grade packaging.

## Escalation paths

Three tiers. Each tier owns a scope, a response-time target, and a pager mechanism. The pager mechanism is customer-specific; the tiering is universal.

**Tier 1 — Customer operations lead.** Owns: admin dashboard, kill-switch flips, replay sessions, Tier 1 playbook execution, SME communication. First responder for every alert that fires from the admin dashboard or from an SME-filed override. Target response: 15 minutes business-hours, 1 hour off-hours.

**Tier 2 — Customer platform team.** Owns: the customer-side runtime — Kubernetes, cloud, IAM, networking, storage, the indexer pipeline, the customer's SIEM. Called by Tier 1 when the problem is infra-shaped (pipeline failure, cluster health, IAM drift, KMS key issue). Target response: 30 minutes business-hours, 1 hour off-hours.

**Tier 3 — Vendor FDE and vendor platform.** Owns: LLM-specific issues (model regression, guardrail miss, bundle defect, cross-tenant issue), and anything that implicates the vendor's platform services. Called when the issue is not Tier 1 playbook-covered and not Tier 2 infra. Target response: 1 hour business-hours, 4 hours off-hours; cases that implicate the vendor's security peer follow the mod-108 Chapter 07 IR route.

The pager mechanism varies by archetype:

- **Archetype A (US healthcare payer):** PagerDuty rotation inside the customer's existing incident-management tenancy; the vendor FDE is an external contact in the Tier 3 escalation policy.
- **Archetype B (US federal agency):** the agency's on-prem incident system (frequently ServiceNow ITSM with an air-gap-safe pager bridge); vendor FDE reached by secure email + a pre-arranged phone bridge, since the vendor's PagerDuty cannot reach the agency intranet.
- **Archetype C (EU retail bank):** Opsgenie or ServiceNow (depending on the bank's standard) with a Microsoft Teams channel bridge; vendor FDE in the Tier 3 group.

Escalation paths are tested, not assumed. The Chapter 07 handoff ceremony includes a Tier 2 and a Tier 3 drill — an intentional page against a sandbox incident to confirm the pager wiring works. Playbooks that assume the FDE is on-call default silently to the FDE.

## Coordination with `ai-enablement-lead`

The boundary with the enablement lead is sharp. Chapter 08 writes it in full; this chapter writes the handoff artefacts that cross it.

**FDE hands to the enablement lead:**

- Pilot workflow definition from mod-102 Chapter 05 (the scoping memo).
- SME UI reference — screens, states, override surface from mod-106 Chapter 05.
- Admin dashboard reference — layout, panels, operator actions.
- Playbook library — the eight playbooks above plus any pilot-specific additions.
- Eval dashboard pointer — mod-107 Chapter 05 online-eval URL, with read access for the enablement lead's cohort.
- Replay user guide — the Chapter 05 replay panel reference.

**Enablement lead hands back:**

- Change-plan stakeholder map — who needs to know what, when.
- SME communication cadence — weekly digest, monthly all-hands slot, pre-release notes.
- Benefit-realisation KPI set — tied to the mod-102 commitments; the metrics the exec sponsor will be shown at QBR.
- Adoption dashboard — not the admin dashboard; this one measures usage, SME-completion rate, override rate over time.
- Exec-sponsor update rhythm — monthly steering with a defined agenda shape.
- Training cohort schedule — who takes Layer 1 when, who is certified at Layer 3, when refreshers fire.

Running the SME programme beyond the initial four-layer curriculum is enablement work, not FDE work. Benefit-realisation reporting to the steering committee is enablement work, not FDE work. If the FDE is still running the SME cohort six months post-launch, the handoff did not happen. Chapter 08 names the pager rules that enforce the split.

## Running the three archetypes

**Archetype A — US healthcare payer.** The operations lead sits inside the payer's operations function. The LaunchDarkly tenant provided by the vendor exposes kill-switch read-flip controls directly in the admin dashboard via the LaunchDarkly SDK; the operations lead can flip without leaving the surface. The pager is PagerDuty. Training Layer 4 (governance) focuses on HIPAA PII-handling obligations, SOC 2 Type II evidence rhythm, and 45 CFR 164.312(b) audit-control satisfaction via the kill-switch log. <!-- needs-research: confirm the current mapping of kill-switch audit evidence to 45 CFR 164.312(b) and SOC 2 CC6 requirements -->

**Archetype B — US federal agency.** The operations lead sits inside the policy office, usually a GS-14 with an SME background rather than an engineering background. The agency's SIEM is the audit sink; the flag-flip feed and the kill-switch log stream there. The deployment is air-gapped, so Tier 3 escalation to the vendor FDE does not run through the agency intranet — playbook steps that call vendor services run through the pre-arranged secure channel. Training Layer 4 focuses on OMB M-24-10 (and successor memo) human-oversight obligations, NIST AI RMF Measure function evidence, and ATO POA&M rhythm. <!-- needs-research: confirm current OMB AI memo numbering (M-24-10 and successor memos) -->

**Archetype C — EU retail bank.** The operations lead sits inside first-line defence (the business line's operations function), with second-line defence (bank risk) consuming the adoption and residual-risk reports via the enablement lead. Microsoft Sentinel is the audit sink. DORA ICT third-party coordination fires every bundle version — the vendor is a critical ICT third-party under Article 28 and the bank's register expects the bundle manifest. Training Layer 4 focuses on EU AI Act Article 14 human-oversight obligations, GDPR Article 22 automated-decision interpretation, and SR 11-7-equivalent validation evidence that the bank's internal model-risk function requires. <!-- needs-research: confirm the current status of EU AI Act Article 14 implementation guidance for deployer obligations -->

## Anti-patterns

**The admin dashboard is a Grafana link no SME can read.** Operations leads bounce back to the FDE for every alert because the surface they were given is a 40-panel SRE dashboard. The admin surface must be *their* surface — four panels, operator-actionable, in their vocabulary.

**Playbooks that assume the FDE is on-call.** Every playbook ends with "page the FDE." The FDE is never off the rotation and the pilot never actually handed off. Tier 1 playbooks must be executable by the operations lead end-to-end; escalation is a named step inside the playbook, not the first and only step.

**Escalation paths tested only in post-mortem.** The pager wiring was never drilled. The first real page discovers that the Tier 3 contact is an ex-employee's email address. Chapter 07's handoff ceremony includes a Tier 2 and a Tier 3 sandbox drill for exactly this reason.

**SME training delivered once and never refreshed.** Layer 1 orientation is recorded for new joiners, Layer 2 is run on the launch cohort, Layer 3 is a brown-bag. Six months later there is no refresher, the bundle has versioned four times, and the override surface has changed. The enablement lead owns the refresh cadence; the FDE updates materials with every bundle version bump that touches SME-visible surface.

**FDE running the enablement programme because the customer hasn't appointed an operator.** A soft-handoff into an empty seat. The FDE ends up running Layer 2 sessions and the monthly exec update. The right escalation is upward — the vendor's and customer's sponsors need to resolve the vacancy before the pilot exits mod-109. The enablement lead is the correct peer to raise this.

**Playbook steps that require elevated vendor credentials.** A playbook step reads "SSH to the inference node and restart vLLM." The operations lead does not and should not hold that credential. Either the step belongs in a Tier 3 playbook the vendor FDE executes, or the operation needs an operator-facing control surface — a button on the admin dashboard, a flag flip, a scoped SRE action. Credentials follow role; playbooks respect the boundary.

## Summary

- The customer-side handoff is the engineering deliverable that lets the system run after the FDE goes home: admin dashboard, SME training, playbook library, escalation tiers. The [`ai-enablement-lead`](https://github.com/ai-governance-curriculum/ai-enablement-lead-learning) peer owns the organisational slice around it.
- The admin dashboard is SME-readable — bundle pin, today's volume, eval-metrics panel, kill-switches, flag-flip feed, replay panel, incident banner — not a raw Grafana link. Every panel sources from an upstream named in a prior chapter.
- SME training is a four-layer curriculum: orientation (30-45 min), workflow-specific (90-120 min hands-on), operator (half-day sandbox), governance (2-hour compliance working session). The FDE authors the engineering content; the enablement lead runs the programme.
- Eight playbooks cover the failure modes the system actually has — judge pass-rate drop, model-provider outage, retrieval index stale, tool allow-list drift, PII leak, cost anomaly, data-classification change, regulator audit request. Each is executable by the Tier 1 operator; escalation is a step inside.
- Escalation runs three tiers: operations lead (Tier 1, 15-minute business-hours), customer platform team (Tier 2, 30-minute), vendor FDE and platform (Tier 3, 1-hour). Pager mechanism varies by archetype — PagerDuty, agency ITSM, Opsgenie / ServiceNow + Teams.
- The FDE hands the enablement lead the engineering artefacts (workflow definition, UI references, dashboard, playbooks, eval and replay pointers); the enablement lead hands back the programme artefacts (stakeholder map, cadence, KPI set, adoption dashboard, exec rhythm, cohort schedule).
- Six anti-patterns to catch early: unreadable dashboard, FDE-assumed-on-call playbooks, untested escalation, one-off training, enablement-by-FDE, credential-exceeding playbook steps.
- Chapter 07 picks up next: the handoff packet shape — architecture, runbook, SLA, cost model, eval dashboard, security-evidence index — versioned alongside the bundle, and the handoff ceremony that includes the escalation drills named here.
