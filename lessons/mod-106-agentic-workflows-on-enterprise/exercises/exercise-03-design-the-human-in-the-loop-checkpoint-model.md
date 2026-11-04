# exercise-03: Design the Human-in-the-Loop Checkpoint Model

**Estimated effort:** 2 hours

## Objective

Produce a signed checkpoint table and HITL UX spec for each of the three archetypes. The output is the design artefact an FDE would bring to the customer's business owner, risk team, and (where applicable) legal/compliance review in pilot week 3–4 — the artefact that gates the transition from draft phase to action-enabled phase.

The exercise exists because HITL design is where the deterministic-code reflex (Chapter 02), the permission model (Chapter 05), and the OWASP mitigations (Chapter 06) converge into one customer-signed document.

## Prerequisites

- Read [Chapter 04](../04-human-in-the-loop-checkpoints.md) end to end.
- Have exercise-01's pattern decisions to hand — the HITL surface differs by pattern.
- Have the module's Chapter 01 three archetypes to hand (healthcare payer, federal agency, retail bank).
- Optional: skim [EU AI Act Article 14 (human oversight)](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=OJ:L_202401689) and [SR 11-7](https://www.federalreserve.gov/supervisionreg/srletters/sr1107.htm) to see where the regulatory shape bites for Archetypes B and C.

## Problem statement

For each archetype, produce:

1. **The complete tool catalogue** for the action-enabled phase (draft phase will typically have no write tools). The catalogue should include retrieval tools (compose with mod-105 Chapter 03), draft/write tools, and queue/hand-off tools. 6–10 tools per archetype is the realistic shape; fewer is also fine if justified.
2. **The checkpoint table** — one row per tool. Columns: action class (read / draft / write / queue), reversibility (fully / partially / irreversible), time-to-notice (immediate / within hour / within day / longer), business cost (low / medium / high), chosen checkpoint (auto-run / confirm per call / confirm per plan / clarify / queue), approver (CSR / supervisor / analyst / team lead / etc.), two-person required (yes / no), rationale (one line).
3. **A UX spec** for the confirm step. What does the UI show the approver? The proposed action and parameters; the inputs that drove the decision; the model's structured rationale; the identity under which the action will execute; the reversibility class; anomaly flags. Specify each as a bullet list; a wireframe sketch is a stretch goal.
4. **A UX spec** for the clarify step. What does a good clarify question look like for this archetype? Include one fully-worked example question with options enumerated and consequence stated.
5. **A UX spec** for the queue hand-off. What context goes into the queue entry? Who picks it up and under what SLA? What happens when they close it (capture resolution into eval signal)?
6. **The business-owner sign-off block** at the bottom of each archetype's section — unfilled is fine for a learning exercise, but the shape should make clear the document is designed to be countersigned.

## Requirements

Three completed designs. Each design:

- Covers both a draft phase and an action-enabled phase where the archetype's description supports it. Name the phase transitions.
- Identifies at least one tool that is **not** a simple confirm-per-call — something that requires confirm-per-plan, two-person approval, or an out-of-band channel. Explain why.
- Identifies at least one action class that remains in auto-run with justification (the reversibility and time-to-notice argue for it). Confirm-everything is wrong (Chapter 04 anti-patterns); be opinionated.
- Specifies the HITL instrumentation signals (confirm rate, edit rate, clarify rate, queue rate) the pilot will measure and the dashboard they appear on (compose with mod-110 where applicable).
- References the regulatory posture that the HITL design is designed to satisfy: Archetype A on HIPAA + SR 11-7-adjacent model-governance posture if applicable; Archetype B on NIST AI RMF + agency-specific oversight requirements; Archetype C on EU AI Act Article 14 + DORA + GDPR.
- Flags every unverifiable claim about the customer's actual risk tolerance or workflow with `<!-- needs-research: ... -->`. A real pilot would resolve these through customer discovery (mod-102); for this exercise, flagging is acceptable.

## Starter guidance

- **Design the table from the tool side, not the model side.** Each tool is a row; the checkpoint is a property of the tool, not a property of the model's confidence. The model's confidence can shift a borderline row from auto-run to confirm, but it does not change the baseline.
- **Reversibility is a product of the tool AND the customer's rollback capability.** "The agent sends an email" is irreversible once the mail server accepts it. "The agent drafts an email into the user's drafts folder" is fully reversible. The *same semantic action* can land in either column depending on how the tool is wired.
- **Business cost is customer-defined, not FDE-defined.** For Archetype A, a mis-filed prior-auth correction is high-cost because it affects claims. For Archetype C, a mis-routed case is medium-cost because it shifts the SLA clock and may miss a regulatory window.
- **The two-person rule earns its weight on legal-exposure actions.** Policy exceptions, bulk refunds, customer-identity updates, classification-boundary crossings — each is a candidate. Do not sprinkle the two-person rule everywhere; name the rows that deserve it.
- **Clarify is for genuine ambiguity, not for design laziness.** If your design puts clarify on a step that could have been resolved by better classification, say so — and name what you would redesign to remove the clarify.
- **Queue is a workflow.** The queue entry's shape and SLA are part of the design. "Dumps into a backlog" is not a design.
- **Compose with your exercise-01 pattern.** The pattern-canonical HITL surfaces from Chapter 04 — HITL on the plan for Pattern 4, HITL on the final iteration for Pattern 5, HITL per step for Pattern 6 — should match your pattern decision.
- **Rollout preview.** Each archetype's design should name the shadow → HITL-first → selective auto-run path. Which action classes graduate first? Which never graduate?

## Acceptance criteria

- Three archetype designs, each containing: a tool catalogue, a checkpoint table, confirm UX spec, clarify UX spec, queue UX spec, a sign-off block, and a signals-to-measure list.
- At least one tool per archetype uses confirm-per-plan OR two-person OR out-of-band channel.
- At least one tool per archetype is auto-run with explicit reversibility justification.
- The checkpoint table's "approver" column maps to a real role in the archetype (CSR, supervisor, analyst, case worker, team lead). "User" or "operator" is not acceptable — be specific.
- Each design references the regulatory posture the HITL stance is defending.
- Each design cites exercise-01's pattern decision and composes its HITL surface with the pattern's shape.
- The documents are legible to a business owner — pattern names are glossed, acronyms expanded on first use.
- Every unverifiable workflow claim is tagged `<!-- needs-research: ... -->`.
- The signals-to-measure lists are testable in exercise-02's agent — i.e., a confirm rate per tool is a thing the orchestrator can emit from the audit log.

## Stretch goals

- Produce a *rollout appendix* for one archetype: the shadow phase's success criteria, the HITL-first phase's graduation criteria for each tool, the specific override rate that would unblock auto-run per tool.
- Produce a *confirmation-fatigue mitigation* note for one archetype: how many confirms per approver per hour is sustainable? What UX moves reduce the fatigue without reducing safety?
- Design an *anomaly-flagging* heuristic for the confirm UX: what differences between this proposal and the user's historical approvals should raise a visual flag?
- For Archetype C, write the paragraph the pilot's design doc would include to support the EU AI Act Article 14 compliance case. The paragraph should name specific oversight mechanisms the HITL design provides.
- Wire the HITL design into exercise-02's agent — the confirm/clarify/queue paths are mocked functions the orchestrator calls, with seeded policies that approve / reject / queue based on the proposal shape. Run the agent against seeded inputs and observe the HITL events in the audit log.
