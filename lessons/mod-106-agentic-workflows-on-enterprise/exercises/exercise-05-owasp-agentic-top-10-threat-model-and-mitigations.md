# exercise-05: OWASP Agentic Top 10 — Threat Model and Mitigations

**Estimated effort:** 3 hours

## Objective

Produce a complete OWASP-Agentic-coverage matrix and a lightweight red-team findings log for **one** of the three archetypes. The matrix is the artefact the customer's AppSec reviewer will trace end-to-end before the pilot ships; the findings log is the result of the FDE-led adversarial pass that must happen before any write-capable tool is enabled.

This is where the decisions from exercises 01–04 (pattern, framework, HITL, permissions) converge into a defensible security posture.

## Prerequisites

- Read [Chapter 06](../06-owasp-agentic-top-10.md) end to end.
- Skim [OWASP's Agentic AI Threats and Mitigations](https://genai.owasp.org/resource/agentic-ai-threats-and-mitigations/) and the [OWASP LLM Top 10](https://genai.owasp.org/llm-top-10/).
- Have exercises 01–04's artefacts to hand — this exercise references them repeatedly.
- Optional: skim [Greshake et al., "Not what you've signed up for"](https://arxiv.org/abs/2302.12173) for a concrete feel of indirect prompt injection.
- Optional: skim [MITRE ATLAS](https://atlas.mitre.org/) for complementary attack-pattern language.

## Problem statement

Pick one archetype (the one you built the agent for in exercise-02 is a natural choice). Produce:

1. **The coverage matrix** — one row per OWASP Agentic threat, with columns: threat description, how this threat manifests in your archetype, severity for this workflow (low / medium / high), chosen mitigations (one or more from Chapter 06's catalogue), residual-risk statement (what the mitigation stack does not fully prevent), link to the artefact that implements the mitigation (exercise-02's code, exercise-03's HITL spec, exercise-04's permission map).
2. **The architecture diagram** — a rendered diagram (text, mermaid, or image) that shows the agent's trust boundaries: which content is trusted (system prompt, orchestrator state), which is untrusted (tool outputs, user input, retrieved content), where sanitisers sit, where sub-agents sit, where HITL fires, where the kill-switch is. Threat IDs from the matrix annotate the arrows and nodes they apply to.
3. **The red-team findings log** — the result of your own adversarial pass. For each of the six risk families in Chapter 06 (indirect prompt injection, tool misuse, memory poisoning, confused deputy, NHI weaknesses, HITL overwhelm), craft at least one attack attempt, run it against your exercise-02 agent (or simulate on paper if exercise-02 is not done), record the outcome, and note the mitigation that caught it (or the gap that needs closing).
4. **The go/no-go note** — one page. Would you recommend the pilot graduates to its first write-capable phase? If yes, under what conditions; if no, what specific evidence would be needed. This note is what you would hand the customer's risk committee.
5. **The peer-escalation brief** — if any finding or residual risk crosses the Chapter 07 threshold for `agentic-safety-engineer` escalation (frontier-safety-level red-team), draft the brief you would hand the peer.

## Requirements

- One archetype, selected and named.
- Coverage matrix with a row for each OWASP Agentic threat the chapter named (at least the six risk families; include additional OWASP-named threats where they apply — resource overload, cascading hallucinations, goal manipulation, repudiation, identity spoofing, multi-agent-system threats where relevant).
- Every mitigation in the matrix traces to a specific artefact: a line of code in exercise-02, a row in exercise-03's HITL table, a scope entry in exercise-04's permission map, or a stated design decision in this exercise.
- Residual-risk statements are honest — "this does not prevent X" is more valuable than "this fully mitigates Y" if the latter is overclaiming.
- At least six attack attempts in the red-team findings log, one per risk family, with concrete attacker-input examples where applicable.
- At least one finding should be a *success* for the attacker — pick a plausible gap and demonstrate it; this builds the FDE's threat-modelling muscle. Record the mitigation that would close it.
- The architecture diagram clearly shows the trust tiers and the kill-switch. A simple text diagram is acceptable; mermaid is preferred.
- The go/no-go note is one page and references at least three specific signals from the coverage matrix.
- Every unverifiable claim about the customer's actual threat environment is tagged `<!-- needs-research: ... -->`.

## Starter guidance

- **Threat-modelling is a verb.** Walk through the agent step by step; at each step ask "what is an attacker's move?" and "what catches it?" Do not treat the matrix as a checkbox exercise — the findings come from genuine adversarial thinking.
- **Start with indirect prompt injection.** It is the highest-weight threat for almost every FDE pilot. Pick a realistic injection vector for your archetype (a case comment, a document attachment, a webform submission) and work through what it takes to turn that content into unauthorised action.
- **The sub-agent discipline is the single strongest mitigation for injection.** If your exercise-02 agent reads untrusted content, is it in a capability-constrained sub-agent? If not, name the redesign.
- **Confused deputy is subtle.** The attacker's move looks legitimate at the user-identity layer but exploits the agent's elevated privileges at the service layer. Walk through: "could a tenant-A user's input, via the agent, cause an action that uses tenant-A-service-account privileges to read tenant-B data?"
- **HITL overwhelm is a UX attack.** The volume version is easier to mitigate (rate-limit the HITL queue); the content version is harder (a subtly misleading rationale that the approver confirms). Pick a representative example.
- **Red-team your own HITL UX.** If the confirm dialog's rationale is the primary artefact and the "what will actually execute" summary is secondary, an injected rationale can slip past the approver. Is that your design?
- **Memory poisoning requires persistent memory.** If your pilot's memory is session-only, the threat shrinks substantially. Name the memory tiers from exercise-04 and show which ones carry the risk.
- **NHI weaknesses tie back to exercise-04.** If your NHI inventory has static shared secrets, the finding is "static shared secret" and the mitigation is "replace with workload-identity attestation." Don't hand-wave.
- **Residual risk is honest, not scary.** Every mitigation has limits; name them. The customer's risk committee is making a probabilistic decision, not a guaranteed one.
- **Composition matters.** No single mitigation is sufficient. The chapter argues for seven layers for one action — walk the matrix and verify that each high-severity threat is caught by at least two layers.

## Acceptance criteria

- One archetype, selected and named, with artefacts that trace back to exercises 01–04.
- A complete coverage matrix for at least the six risk families from Chapter 06, plus any additional OWASP Agentic threats that apply.
- Every mitigation traces to a specific artefact.
- An architecture diagram showing trust boundaries, sub-agents, HITL points, sanitisers, and the kill-switch, with threat annotations.
- A red-team findings log with at least six attack attempts, at least one of which succeeds (gap identified + closure recommended).
- A one-page go/no-go note with specific signal references.
- A peer-escalation brief if any finding crosses the Chapter 07 threshold.
- The documents are reviewable by an AppSec reviewer in one sitting — threat names, mitigation names, and artefact references are all consistent with the module's vocabulary.
- Every unverifiable claim about the customer's environment is tagged `<!-- needs-research: ... -->`.

## Stretch goals

- Run the red-team pass against exercise-02's agent as code: craft attack inputs, run the agent, capture the audit log, analyse. Record the full session log for each attack.
- For the one successful attack from the red-team findings log, implement the closure in exercise-02 and demonstrate the attack failing.
- Produce a *regression suite* that replays the red-team attacks on each pilot change, so the same gap does not re-open. One test per attack, same shape as exercise-02's test suite.
- Write a *disclosure runbook* — what the FDE and the customer do if a real incident matching one of these threats is detected in production. Compose with mod-110's on-call shape.
- Compose with mod-108: produce the two-paragraph "security case" the pilot's design doc would hand to the customer's formal security review. One paragraph on the threat model and mitigation stack; one paragraph on the red-team evidence and residual risk.
- If your archetype landed on Pattern 6 or auto-run on medium-cost actions, draft the full `agentic-safety-engineer` peer-escalation brief (Chapter 07's hand-off brief shape) with the explicit ask — "we need adversarial assessment at scale before auto-run graduates above class X."
