# Chapter 01 — The Consulting-Craft Overlay — Why Executive Communication Is Load-Bearing

## Why this chapter exists

[Mod-111](../mod-111-field-to-product-loop/) closed the vendor-facing loop: the FDE is the product team's field eye; patterns from pilots become platform-request packets and feature requests; the FDE team's knowledge base compounds. Mod-112 turns back toward the customer. None of mod-111 produces anything if the pilot does not *survive* — and pilots survive by the discipline of the customer-facing conversation.

Consulting craft is not a soft skill. The executive demo that does not land, the exec brief that loses the VP's attention, the weekly review that drifts into theatre, the resistance the FDE does not know how to handle, the scope ask the FDE says yes to in the room, the hand-off that produces an indispensable FDE instead of an unnecessary one — every one of these is a mechanism by which an otherwise excellent technical pilot fails to produce its intended outcome. This module writes the craft for each one.

The FDE sits at the intersection of engineering and consulting. [Mod-101](../mod-101-fde-discipline-and-position/) named that position and the four stances. [Mod-102](../mod-102-customer-discovery-and-pilot-scoping/) wrote the discovery and scoping dialogue. Mod-104 through mod-110 delivered the technical content against which the customer judges the pilot. Mod-111 fed the loop back to product. Mod-112 is the layer of customer-facing conversation on top of everything above — the layer that determines whether the technical delivery converts to a renewed and expanded engagement, and ultimately to a case study and a referenceable customer.

## The one thing that is different from mod-111

Mod-111 operated inside the vendor. Mod-112 operates inside the customer's political economy. The audiences change: a VP who wants three bullets instead of a design doc; an SME who reads every claim with the eye of someone whose craft is being automated; a department director who has to protect their headcount; an IT reviewer who has lost sleep over a previous vendor's security failure. The artefacts change: a one-pager not a packet; a before/after KPI table not a pattern writeup; a decision log countersigned by the customer, not a triage ritual with a peer FDE. The failure modes change: not "the platform team rejected the packet" but "the customer sponsor lost patience, and the pilot is not renewed."

The FDE's technical competence has not changed between mod-111 and mod-112. What changes is the audience, the artefact shapes, and the stakes of getting the conversation wrong.

## The five dimensions of consulting craft

Mod-111 used five dimensions scoped to the field-to-product loop. Mod-112 shifts the shape to the customer-facing dimension.

1. **Executive-narrative quality.** The exec demo opens with the customer's problem, not the vendor's capability. The before/after structure is anchored on metrics the customer signed off on. The third-party credibility hook — the customer's own SME saying "this is how I used to spend my Fridays" — is in the room. The no-fake-demo rule is absolute. Measured by: VP engagement at the demo (did they ask follow-up questions, did they commit a next step), the demo artefact's reusability for subsequent executive audiences, and the absence of corrections required after the fact.
2. **Review-cadence discipline.** The weekly or fortnightly pilot review runs on a standing agenda. Status carries a rationale. Decisions are named, owned, and dated. Risks age. Customer open items are an agenda column, not a courtesy. Minutes publish within 4 business hours. Measured by: on-time minutes rate, decision-cycle time (ask-to-decision), and the fraction of status rows whose rationale a reviewer can verify from the metrics.
3. **Scope integrity.** New asks are evaluated against the pilot's success criteria and non-goals (from [mod-102](../mod-102-customer-discovery-and-pilot-scoping/)) before entering scope. "No", "later", and "the applied-AI-engineer in your org can own this after hand-off" are fluent responses. The decision log is countersigned. Measured by: scope-delta against original contract, counter-signature rate on decisions, and the fraction of asks correctly routed to the adjacent-role owner on first pass.
4. **Resistance handling.** The SME who prefers the legacy workflow, the department that reads the pilot as a threat, the IT review that feels adversarial — each has a named stance: radical transparency on failures + pragmatic respect for the resisting party's craft. Measured by: the pre-meeting 1-on-1 cadence with the SME lead, the escalation rate (healthy: 0–2 escalations per pilot; unhealthy: the FDE resolves everything unilaterally and surprises the sponsor at hand-off), and the fraction of customer-side stakeholders who sign the hand-off readiness gate.
5. **Hand-off readiness.** Pair-programming cadence through the last 6–8 weeks of the pilot. Documentation craft that the next engineer can act on. Runbook walkthroughs — co-walk, observe, notified-only. On-call shadowing both directions. A readiness gate the customer sponsor and the FDE's `ai-engineering-manager` co-sign. Measured by: the hand-off readiness checklist pass-rate, the frequency of post-hand-off FDE pings (healthy: trending to zero at week 8 post-hand-off; unhealthy: the FDE is still ticketed daily three months on), and the customer engineering team's confidence self-assessment.

### Gate dimensions

**Review-cadence discipline and hand-off readiness are gates.** A pilot whose weekly reviews have slipped, whose status is theatre, or whose hand-off is not readiness-gated has failed consulting craft regardless of how beautiful the demos were.

Review-cadence discipline is a gate because the standing weekly is the only durable mechanism for the customer sponsor to catch problems before they become political. An FDE who lets the weekly slip is an FDE discovering, four months later, that the sponsor has been quietly losing faith. [Chapter 04](./04-weekly-pilot-review-cadence.md) writes the agenda and the minutes SLA.

Hand-off readiness is a gate because the FDE's exit is the measure of the engagement's success. A pilot that produces excellent short-term outcomes but leaves a customer dependent on a specific FDE's tacit knowledge has not graduated to a sustainable deployment; the vendor must either bill the FDE's time permanently (which violates the FDE role per [mod-101 Chapter 04](../mod-101-fde-discipline-and-position/04-adjacent-customer-facing-roles.md)) or watch the pilot degrade when the FDE moves on. [Chapter 07](./07-knowledge-transfer-to-customer-engineering.md) writes the four streams and the readiness gate.

### Optimisation dimensions

**Executive-narrative quality, scope integrity, and resistance handling are optimisations.** Among gate-passing pilots, the FDE invests in whichever dimension most strengthens the specific pilot's political economy this week.

Executive-narrative quality is the dimension that most compounds across the vendor's portfolio. An exec demo that lands produces a reference, a quote the sales team uses, and a template subsequent FDEs adapt. [Chapter 02](./02-engineer-executive-demos.md) writes the narrative shape, the staging craft, and the failure-recovery playbook.

Scope integrity is the dimension that most protects the pilot's technical health. Every bespoke yes is a week the FDE does not spend on the two-customer platform request from mod-111 Chapter 04. [Chapter 06](./06-scope-discipline-and-decision-log.md) writes the four-step scope-check and the countersigned decision log.

Resistance handling is the dimension that most protects the hand-off. An SME who was never brought along will not champion the pilot inside the customer after the FDE leaves; a department that read the pilot as a threat will quietly drop the workflow back to the legacy process. [Chapter 05](./05-customer-side-resistance.md) writes the three resistance archetypes and the radical-transparency-plus-pragmatic-respect stance.

## The cadence model

The module runs on seven cadences the FDE's calendar must reflect.

| Cadence | Frequency | Principal audience | Chapter |
|---|---|---|---|
| Executive demo | Per executive audience (quarterly to monthly) | Customer VP + sponsor + optionally customer board | 02 |
| Executive brief refresh | Fortnightly (plus on-demand) | Customer VP + sponsor | 03 |
| Five-minute standup | Weekly inside the exec brief recipient's rhythm | Sponsor + VP of the operating function | 03 |
| Weekly / fortnightly pilot review | Weekly during delivery; fortnightly in steady state | Sponsor, SME lead, operations lead, IT liaison | 04 |
| Scope-decision triage | Weekly (inside the pilot review) | Sponsor + FDE | 06 |
| Resistance pre-meeting 1-on-1 | As-needed; standing with the SME lead | Specific resisting stakeholder | 05 |
| Hand-off pair + shadow | Weekly during the last 6–8 weeks | Customer `applied-ai-engineer` / platform team | 07 |

Seven cadences across six chapters. The FDE whose calendar has fewer than four of these is operating in ad-hoc mode; the FDE whose calendar has all seven is operating with consulting-craft discipline.

## Three customer archetypes for the module

Mod-104 onward used three archetypes; mod-112 fills in the executive-communication posture for each.

**Archetype A — US mid-size healthcare payer.** Member-inquiry resolution; HIPAA + SOC 2 Type II; AWS Bedrock + VPC install. Consulting-craft posture: the VP audience is the Chief Operating Officer and the Head of Member Experience; the executive brief carries a HIPAA data-handling subtitle; before/after KPIs are average handle time (AHT), first-contact resolution (FCR), policy-citation rate, and member CSAT; the weekly review includes the customer's privacy office as a standing attendee; the hand-off receiver is the payer's applied-AI-engineering team within the digital organisation. Resistance is often from the clinical reviewer who wants a human in the loop on any member communication that touches benefit interpretation — a legitimate quality concern that becomes part of the gold set. <!-- needs-research: confirm HIPAA Security Rule 45 CFR §164 subsections are the correct citation for de-identification and administrative safeguards when citing in the exec brief -->

**Archetype B — US federal agency.** Policy-analyst drafting; FedRAMP Moderate (path to High) + NIST 800-53 Rev 5 + NIST AI RMF 1.0 + OMB M-24-10 (and successor memos); GovCloud + self-hosted inference. Consulting-craft posture: the VP audience is the Chief Information Officer, the Chief Data Officer, and the mission-owner Assistant Secretary; executive demos are abstracted — no classified or CUI content in the demo, pre-approved prompts, no live calls to the agency's actual data; the executive brief is read inside the enclave and is not emailed; the weekly review is inside the ATO boundary; the hand-off receiver is the agency's internal engineering team under their own authorisation. Resistance is often from the mission-owner concerned about auditability of AI-generated policy drafts — a legitimate concern that drives the decision log's rigor. <!-- needs-research: confirm which OMB memos are currently in force for federal AI use and the specific M-24-10 provisions that touch executive reporting on AI pilots -->

**Archetype C — EU retail bank.** Customer-support reply + case triage; GDPR + DORA + EU AI Act posture; Azure AI Foundry under EU Data Boundary. Consulting-craft posture: the VP audience is the Chief Operating Officer, the Head of Customer Operations, and the model-risk function head (typically reporting to the CRO); the executive brief references the pilot's EU AI Act Article 72 post-market monitoring record and its DORA ICT-third-party register entry; the weekly review is co-attended by the model-risk function; the hand-off receiver is the bank's ML-ops team under DORA ICT-third-party exit requirements; before/after KPIs include customer-satisfaction (NPS or CES depending on the bank's measurement regime), case-triage accuracy, and policy-citation rate. Resistance is often from the model-risk function demanding evidence that the pilot satisfies the bank's existing model-validation standard. <!-- needs-research: confirm EU AI Act Article 72 post-market monitoring reporting shape and cadence for the specific deployer obligations in the bank's use category -->

Every chapter in this module runs its decisions through these three archetypes.

## Where mod-112 sits with the rest of the track

- **[Mod-101](../mod-101-fde-discipline-and-position/)** — the four stances, the position on the ladder, and the sharp role boundaries. Mod-112 operationalises the role boundaries specifically in executive-facing conversations (Chapter 08).
- **[Mod-102](../mod-102-customer-discovery-and-pilot-scoping/)** — the success criteria the sponsor committed to at scoping. Mod-112's exec brief, weekly review, and decision log are all anchored on those criteria.
- **[Mod-103](../mod-103-enterprise-integration/)** — the integration decisions that sit under the pilot's technical narrative; the exec demo's "before" state often references pain in these integrations.
- **[Mod-104](../mod-104-llm-app-in-customer-envs/)** — the gateway architecture; the exec demo frequently stages a request through the actual gateway.
- **[Mod-105](../mod-105-grounding-on-customer-data/)** — the retrieval behaviour and the citation story; the policy-citation metric in the KPI table is a mod-105 artefact.
- **[Mod-106](../mod-106-agentic-workflows-on-enterprise/)** — the tool-call surface and the human-in-the-loop checkpoints; the weekly review's open-risks register often carries agentic-workflow concerns.
- **[Mod-107](../mod-107-eval-eng-on-customer-data/)** — the gold set the SME has contributed to; the SME quote in the exec demo often names a gold-set entry the SME authored.
- **[Mod-108](../mod-108-enterprise-security-and-compliance/)** — the security-review-answer library; mod-112 Chapter 05's IT-resistance response leans heavily on mod-108's artefacts.
- **[Mod-109](../mod-109-deployment-rollout-change-management/)** — the feature-flag and canary posture; the hand-off's deployment-runbook walkthroughs are mod-109 territory.
- **[Mod-110](../mod-110-observability-and-oncall/)** — the dashboards shown in the exec demo and the on-call rota that the customer engineer shadows at hand-off.
- **[Mod-111](../mod-111-field-to-product-loop/)** — the field-to-product loop that mod-112's healthy pilots feed. A pilot that does not land its executive narrative does not survive to generate mod-111 patterns.

## Preview of chapters 02–08

- **[Chapter 02](./02-engineer-executive-demos.md) — Engineering Executive Demos.** The narrative shape, the before/after structure, the SME third-party credibility hook, staging and failure-recovery craft, the no-fake-demo rule.
- **[Chapter 03](./03-exec-brief-and-five-minute-standup.md) — The Executive Brief and the Five-Minute Standup.** The VP one-pager, the before/after KPI table, the five-minute standup narrative, the delivery ritual.
- **[Chapter 04](./04-weekly-pilot-review-cadence.md) — The Weekly / Fortnightly Pilot Review.** The standing agenda, RAG-with-rationale, decisions-required, open risks, minutes SLA.
- **[Chapter 05](./05-customer-side-resistance.md) — Customer-Side Resistance.** The SME, the department, the IT review; radical transparency + pragmatic respect; the escalation pathway.
- **[Chapter 06](./06-scope-discipline-and-decision-log.md) — Scope Discipline and the Decision Log.** Say no, say later, say the applied-AI-engineer owns this after hand-off; the countersigned decision log.
- **[Chapter 07](./07-knowledge-transfer-to-customer-engineering.md) — Knowledge Transfer.** Pair-programming, documentation craft, runbook walkthroughs, on-call shadowing; becoming unnecessary at hand-off.
- **[Chapter 08](./08-role-honesty-in-customer-facing-comms.md) — Role Honesty in Customer-Facing Communications.** The pressured boundaries — not AE, not SE, not CSM, not PS — operationalised for executive-facing comms.

## Anti-patterns

**The silent pilot.** The FDE delivers every technical milestone on time; the customer VP cannot name a single success metric from the pilot three months in. The absence of executive narrative is a political risk the FDE did not price; the renewal conversation surprises the FDE. [Chapter 02](./02-engineer-executive-demos.md) and [Chapter 03](./03-exec-brief-and-five-minute-standup.md) are the counter.

**The theatre review.** The weekly review runs; the RAG is always green; the sponsor nods; no decision ever lands; four months in a critical risk surfaces and the sponsor asks "why didn't I hear about this earlier?" The review existed; the discipline did not. [Chapter 04](./04-weekly-pilot-review-cadence.md) is the counter.

**The silent resistance.** The SME lead has not sent anyone to the weekly review for three weeks; the FDE lets it go; the SME's director drafts an email to the sponsor explaining the pilot is "not useful"; the sponsor is blindsided. The FDE's job was to notice the silence and surface it to the sponsor long before the email. [Chapter 05](./05-customer-side-resistance.md) is the counter.

**The yes in the room.** The customer sponsor asks for a bespoke feature during the weekly review; the FDE says "sure, we can add that"; the pilot's timeline slips by two weeks; two other asks follow. The yes-in-the-room reflex is the FDE's single most common scope failure. [Chapter 06](./06-scope-discipline-and-decision-log.md) is the counter.

**The indispensable FDE.** The pilot is successful; the customer requests the FDE "for one more quarter"; the FDE agrees; the FDE is now effectively a Professional Services contractor against the vendor's FDE budget; mod-111 patterns are not getting written; the next pilot is understaffed. [Chapter 07](./07-knowledge-transfer-to-customer-engineering.md) is the counter.

**The role-drifted exec meeting.** The VP asks the FDE about pricing; the FDE answers; the AE finds out through the customer two days later; the account team's internal process is undermined. [Chapter 08](./08-role-honesty-in-customer-facing-comms.md) is the counter.

## Summary

- Mod-112 is the customer-facing layer on top of the technical delivery (mod-104 through mod-110) and the vendor-facing field-to-product loop (mod-111). The pilot's technical excellence does not survive a broken customer conversation.
- Five dimensions govern the module: executive-narrative quality, review-cadence discipline, scope integrity, resistance handling, hand-off readiness. Review-cadence discipline and hand-off readiness are gates; the other three are optimisations.
- Seven cadences the FDE's calendar must reflect: executive demo, exec brief refresh, five-minute standup, weekly / fortnightly pilot review, scope-decision triage, resistance 1-on-1, hand-off pair + shadow.
- Three archetypes thread through every chapter — the HIPAA payer with privacy-office attendance and member-experience VPs, the FedRAMP agency with abstracted demos and in-enclave reviews, the EU retail bank with model-risk co-attendance and DORA ICT-third-party exit at hand-off.
- Six core chapters follow, one per learning objective, plus Chapter 08's role-honesty overlay for executive-facing conversations.
- Six anti-patterns to catch early: the silent pilot, the theatre review, the silent resistance, the yes-in-the-room, the indispensable FDE, the role-drifted exec meeting.
- Chapter 02 is next: engineering the executive demo — narrative, staging, and failure-recovery, grounded in the no-fake-demo rule.
