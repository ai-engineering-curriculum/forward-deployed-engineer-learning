# Exercise 02 — Role Scope and Deferral Contract Diagram

**Estimated effort:** 2 hours
**Output:** one diagram + a deferral-contract table + a short written defense

## Objective

Make your position on the level ladder (Chapter 03) concrete. Draw where the Forward Deployed AI Engineer stands in relation to the peer specialists and the lower-level craft ICs, and name every deferral contract that comes with the position. By the end you should be able to walk a hiring manager through your diagram from memory and defend each line.

## Prerequisites

- You have read Chapters 02 and 03 of this module.
- You have skimmed [`CURRICULUM.md`](../../../CURRICULUM.md) in the track root, specifically the "Ownership rule" section.
- You are comfortable drawing in whatever tool you prefer (Excalidraw, draw.io, Miro, hand-drawn, tldraw, Mermaid — the tool does not matter, the output does).

## Steps

1. **Sketch the ladder.** Put the Forward Deployed AI Engineer in the centre at level 35. Put the level-30 craft ICs below (`applied-ai-engineer`, `agentic-ai-engineer`, `ai-eval-engineer`, with `rag-engineer` and `llm-application-developer` at level 25). Put the level-35 peer specialists around (`ai-infra-inference-engineer`, `ai-infra-security` / `security`, `ai-evaluation-engineer`, `training-pipeline-engineer`, `voice-agent-engineer`, `ai-enablement-lead`). Put the level-40 roles above (`senior-agentic-ai-engineer`, `ai-engineering-manager`, `agentic-safety-engineer`) and `agentic-systems-architect` (level 48) further above.
2. **Shade the FDE's territory.** Shade the area of the diagram that is the FDE's primary ownership. Do not shade peers; do not shade level-40 and above.
3. **Draw the deferral arrows.** For each peer and each craft IC, draw an arrow from the FDE to that role labeled with the *kind* of question that triggers the deferral. Examples: *"inference tuning in customer VPC → ai-infra-inference-engineer"*; *"methodology-of-methodology → ai-eval-engineer"*; *"release-assurance packet → ai-evaluation-engineer"*; *"multi-agent redesign → agentic-systems-architect"*.
4. **Build the deferral-contract table.** In a table, for each of the eleven boundaries from Chapter 03, write three cells: (a) *what the peer owns*, (b) *what the FDE adds on top*, (c) *the specific escalation trigger you would use to pull them in*. Do not copy the chapter text verbatim; paraphrase to make it your own.
5. **Stress-test two upward handoffs.** Pick *two* of the level-40+ handoffs (`senior-agentic-ai-engineer`, `ai-engineering-manager`, `agentic-systems-architect`, `agentic-safety-engineer`). For each, write a short paragraph answering: *When exactly would I escalate, and what would I package for them?* Reference Chapter 06.
6. **Write the defense.** In 200–300 words, defend your diagram against the question: "Why isn't the FDE just a senior version of the applied-ai-engineer at level 35?" Your answer must name at least two of the four FDE stances from Chapter 02 and at least two concrete deferrals from Chapter 03.

## Starter guidance

- The point is clarity, not visual polish. If a reviewer can read your diagram in 90 seconds and recite the deferral contracts from memory, you are done.
- Resist the urge to draw the FDE as "above" the craft ICs. Level 35 is *specialist-IC peer* to level 30, not managerial supervisor. The visual language should say "deferral contract" and "cross-family collaboration," not "org-chart authority."
- You will revisit this diagram at the end of the track (after Project 104). Save it in a durable location.

## Acceptance criteria

- The diagram shows all eleven boundaries called out in Chapter 03, each with a labelled arrow.
- The deferral-contract table has eleven rows, three cells each, cells you could defend to the named peer.
- The two stress-tested upward handoffs each have a concrete escalation scenario and a concrete package.
- The written defense names at least two of the four FDE stances and at least two concrete deferrals.
- The diagram does not place the FDE above its level-30 craft ICs — the relationship is deferral-contract, not authority.

## Stretch goals

- Add a second diagram showing the *engagement flow* — which peers typically get pulled in at which deliverable (mapping Chapter 05 to Chapter 03). For example, `ai-evaluation-engineer` shows up at the eval protocol and the deployment runbook; `ai-infra-inference-engineer` shows up at the integration design for VPC-install pilots.
- Find one place in the current track scaffold (`CURRICULUM.md`, any mod README) where the deferral contract is stated less precisely than your table — propose a sharper phrasing as a one-line PR-comment suggestion (do not actually open a PR; this is practice for mod-111).
