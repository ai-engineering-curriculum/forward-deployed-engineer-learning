# exercise-01: Pick the Right Pattern from Anthropic's Effective-Agents Catalogue

**Estimated effort:** 2 hours

## Objective

For each of the three customer archetypes carried through the module (US healthcare payer, US federal agency, EU retail bank), produce a pattern-selection decision and a one-page agent-architecture ADR. The artefact is the kind of document an FDE would hand to the customer's business owner, their AppSec reviewer, and the pilot's engineering lead in week 2–3.

You will choose from Anthropic's catalogue (prompt chaining, routing, parallelisation, orchestrator-workers, evaluator-optimiser, autonomous agent) and apply the deterministic-code-beats-LLM reflex to every workflow step. For pilots that compose patterns, name the composition.

## Prerequisites

- Read [Chapter 01](../01-agentic-overlay.md) and [Chapter 02](../02-agent-pattern-catalogue.md).
- Skim Anthropic's [*Building effective agents*](https://www.anthropic.com/engineering/building-effective-agents) post — the pattern catalogue is the source material.
- Have the ADR template from mod-104 Chapter 02 (or mod-105 Chapter 02) to hand; this exercise's ADR mirrors its shape.
- Optional: mod-102's pilot-scope ADR to see how customer context flows into decision docs.

## Problem statement

For each archetype:

**Archetype A — Mid-size US healthcare payer.** Pilot workflow: member-inquiry resolution. In the draft phase the agent retrieves policies and plan docs (mod-105) and drafts a reply for the CSR. In the action phase it can, in addition, file a prior-authorisation correction ticket and post an internal case note. Risk tolerance: medium — CSR supervisor is the HITL approver for writes.

**Archetype B — US federal agency.** Pilot workflow: policy-analyst drafting + evidence packaging. The analyst asks a question; the agent retrieves memos, regulations, and FOIA-archive items and packages a citation-rich evidence pack into the analyst's draft workspace. Risk tolerance: write-to-analyst-draft-only in the pilot phase; no writes outside the analyst's own workspace.

**Archetype C — EU retail bank.** Pilot workflow: customer-support draft reply + case triage. In the draft phase the agent drafts a reply and flags cases for regulatory or complaints-ombudsman routing. In the action phase it can update case triage status and assign to a specialist team. Risk tolerance: medium — case worker is the HITL approver, team lead for high-cost routings.

For each archetype, produce the pattern-selection decision and the ADR as if you were the FDE in pilot week 3.

## Requirements

For each archetype:

1. Step through the workflow and list every step it needs to perform. For each step, apply the deterministic-code-beats-LLM reflex — mark the step as code, structured-output LLM call, or agent step. Justify the mark in one line.
2. For the agent-step portion, pick one of the six patterns from Chapter 02 (or a named composition, e.g., "router of chains" or "orchestrator-over-tools"). State the pattern name verbatim from the catalogue.
3. Populate the Chapter 02 decision table (blast radius, latency profile, cost per task, evaluation shape, audit record density, customer explainability, HITL surface) for the chosen pattern against this workflow. Scores can be ordinal (low/medium/high) with a one-line rationale.
4. For any step that falls outside the chosen top-level pattern, name the sub-pattern used — e.g., an evaluator-optimiser inside one step of a top-level chain.
5. Write a one-page ADR per archetype with context, decision, alternatives considered, consequences, and a named pattern exit criterion (when would you revisit and move to a different pattern).
6. In an appendix per archetype, list any claim about the customer's workflow, risk tolerance, or latency/cost budget you could not verify from the module's archetype description — flag each with `<!-- needs-research: ... -->` so a reviewer can backfill.

## Starter guidance

- Walk the decision table **left to right.** Start at prompt chain; only move right when a specific property of the workflow tells you the leftward pattern cannot carry it. Name the property in the ADR.
- Separate **phases** of the pilot (draft / action-enabled) as distinct decisions. The same workflow often starts at Pattern 1 and graduates to Pattern 4 in a later phase; both decisions deserve space in the ADR.
- The customer's **explainability requirement** is often a gate dimension in healthcare, federal, and banking contexts — that is why the archetypes do not land on Pattern 6 in their pilot phase. If your design does land on Pattern 6, you need an explicit defence.
- The **deterministic-code reflex applies even inside the agent**. A step that looks like "call the model to classify" may really be "call an existing classifier" or "look it up in a lookup table." Say which.
- **Latency compounds** — a five-hop chain with p95 ≈ 1.8 s per model call is a nine-second p95 workflow before any tool calls. Compare your design against mod-104 Chapter 05's budget if you have one for the workflow.
- For composition, name the composition explicitly in the pattern slot: e.g., "Top-level router (Pattern 2) with per-branch prompt chains (Pattern 1); draft-reply chain contains an evaluator-optimiser step (Pattern 5) on tone and regulatory-tone compliance."
- Be honest about where you would normally bring in the `agentic-ai-engineer` peer (Chapter 07). If the pilot's pattern is Pattern 4 with > 5 tools, flag the week-2 peer brief in the ADR's Consequences section.

## Acceptance criteria

- Three workflow step-lists, one per archetype, with each step marked code / structured-output LLM / agent and justified.
- Three pattern-selection decisions, one per archetype, naming the pattern (or composition) verbatim from Chapter 02.
- Three populated decision tables with rationale cells.
- Three one-page ADRs with context, decision, alternatives, consequences, and a pattern-exit criterion.
- At least one archetype shows a *composed* pattern (not a single top-level pattern) — real pilots compose.
- At least one archetype has a draft-phase decision and a later-phase decision that differ.
- Zero archetypes land on Pattern 6 (autonomous agent) in the pilot phase unless the ADR carries an explicit justification and a named peer co-signature path.
- Every unverifiable workflow claim is tagged `<!-- needs-research: ... -->` with enough detail that a reviewer can resolve it.
- The ADRs are legible to a business owner without a CS background — pattern names explained in one-sentence glosses the first time they appear.

## Stretch goals

- For one archetype, write a short sketch of what the Pattern 6 (autonomous agent) version would look like and why you would *not* ship it in the pilot phase. The sketch should name two specific blast-radius or explainability properties that disqualify Pattern 6 for this workflow.
- For each archetype, name one **adjacent workflow** the customer would likely want next, and predict whether the pattern selection changes.
- Produce a "how we would evolve the pattern" roadmap: shadow → HITL-first → selective auto-run per Chapter 02's rollout note, with the pattern transitions each phase requires.
- Draft the brief you would hand to the `agentic-ai-engineer` peer if pattern depth is on the near horizon (Chapter 07's hand-off brief shape).
