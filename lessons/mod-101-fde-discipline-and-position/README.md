# mod-101 — The Forward Deployed Engineer Discipline and Position on the Ladder

**Track:** Forward Deployed AI Engineer (level 35, AI Engineering family)
**Estimated effort:** 10 hours

Before any pilot, any integration, any eval — you need to know *what role you are in*, where it came from, who it works alongside, and what it is deliberately not. This module answers those questions. Everything downstream (customer discovery, enterprise integration, compliance review, observability, field → product loop) assumes you can name the role and its deferral contracts cold.

## Learning objectives

- Characterise the Forward Deployed Engineer role as it evolved from the Palantir origin through the modern frontier-lab and applied-AI product-company instantiations.
- Position the role on the level ladder — what the peer specialists own, what the FDE adds, where each deferral contract runs.
- Diagram the relationship to adjacent customer-facing roles (AE, SE, CSM, CSE, PS, Partner Engineer, DevRel, TAM) and keep the sharp boundaries this track enforces.
- Name the seven deliverables the FDE is on the hook for across a design-partner engagement and how they feed each other.
- Decide when to stay a generalist and when to escalate to a peer specialist — against a reproducible rule, not a feeling.

## Chapters

1. [Origin and modern instantiations](01-origin-and-modern-instantiations.md) — Palantir, frontier labs (OpenAI / Anthropic / Scale AI / xAI / Cohere / Mistral), applied-AI product companies (Sierra, Decagon, Hebbia, Writer, Harvey, Glean, Cresta, and more), and hyperscaler / SaaS customer-engineering orgs. Four invariants across all instantiations.
2. [The four stances of the discipline](02-the-four-stances.md) — embedded delivery, no-ego engineering craft, design-partner-first mindset, and the field → product loop as a first-class deliverable. What each stance demands and what it rules out.
3. [Position on the level ladder](03-position-on-the-ladder.md) — eleven boundaries against `applied-ai-engineer` (L30), `agentic-ai-engineer` (L30), `ai-eval-engineer` (L30), `rag-engineer` / `llm-application-developer` (L25), `ai-infra-inference-engineer` (L35), `ai-evaluation-engineer` (L35), `ai-infra-security` / `security` (L35), `voice-agent-engineer` (L35), `ai-enablement-lead` (L35), `senior-agentic-ai-engineer` / `ai-engineering-manager` (L40), and `agentic-systems-architect` (L48).
4. [Adjacent customer-facing roles and sharp boundaries](04-adjacent-customer-facing-roles.md) — FDE vs. Account Executive, Solutions Engineer / pre-sales, Customer Success Manager, Customer Success Engineer, Professional Services, Partner Engineer, Developer Advocate, Technical Account Manager, Field CTO.
5. [Design-partner engagement deliverables](05-design-partner-deliverables.md) — the seven deliverables (discovery brief, pilot scope, eval protocol, integration design, deployment runbook, hand-off packet, field → product review) with timings, signers, and "done" definitions.
6. [Generalist stance and when to escalate](06-generalist-vs-escalation.md) — the default is "you do it"; four triggers that fire an escalation; the escalation protocol.

## Exercises

Each exercise turns one chapter into a usable artifact you keep. Together they become the foundation for Projects 101–104.

- [Exercise 01 — FDE origin and modern instantiations: a structured read](exercises/exercise-01-fde-origin-and-modern-instantiations-read.md) *(≈1 h)*
- [Exercise 02 — Role scope and deferral-contract diagram](exercises/exercise-02-role-scope-and-deferral-contract-diagram.md) *(≈2 h)*
- [Exercise 03 — FDE vs. adjacent customer-facing roles](exercises/exercise-03-fde-vs-adjacent-customer-facing-roles.md) *(≈1 h)*
- [Exercise 04 — Design-partner engagement deliverables walkthrough](exercises/exercise-04-design-partner-engagement-deliverables-walkthrough.md) *(≈2 h)*
- [Exercise 05 — Generalist-vs-escalate decision drill](exercises/exercise-05-when-to-stay-generalist-when-to-escalate-drill.md) *(≈2 h)*

## Resources

Primary sources for every claim in the chapters live in [`resources.md`](resources.md). If a reference becomes unreachable, go to the canonical publisher before any third-party mirror.

## Structure

```
mod-101-fde-discipline-and-position/
├── README.md                     this file
├── 01-origin-and-modern-instantiations.md
├── 02-the-four-stances.md
├── 03-position-on-the-ladder.md
├── 04-adjacent-customer-facing-roles.md
├── 05-design-partner-deliverables.md
├── 06-generalist-vs-escalation.md
├── exercises/                    one prompt per exercise
├── labs/                         (scaffold — longer hands-on labs land in later cycles)
├── quizzes/                      (scaffold — knowledge checks land in later cycles)
└── resources.md                  curated external references
```

## Where you go next

After this module you have the vocabulary and the boundary map. The rest of the track puts each of the seven deliverables, each of the eleven peer boundaries, and each of the four stances through hands-on practice:

- **mod-102** — discovery and pilot scoping. Turns Chapter 05's first two deliverables into craft.
- **mod-103** — enterprise integration engineering. Where Chapter 05's integration design gets real.
- **mod-104 / 105 / 106** — LLM application, grounding, and agentic workflow craft, each with the customer-environment overlay from Chapter 03.
- **mod-107** — eval engineering on customer data. Chapter 05's eval protocol in depth.
- **mod-108** — enterprise security and compliance. The regulated-vertical gate.
- **mod-109 / 110** — deployment, observability, and on-call. Chapter 05's deployment runbook in depth.
- **mod-111** — field → product loop. Chapter 02 Stance 4 and Chapter 05 Deliverable 7 in depth.
- **mod-112** — consulting craft. The Stance 3 and Deliverable 6 crafts in depth.

The role and the discipline do not change through the track. The practice gets deeper.
