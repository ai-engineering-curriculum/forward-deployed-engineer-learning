# mod-106-agentic-workflows-on-enterprise: Agentic Workflows on Enterprise Surfaces

**Estimated effort:** 14 hours

## Learning objectives

- Choose the right agent shape for each enterprise workflow — Anthropic's 'Building effective agents' pattern catalogue (prompt chaining, routing, parallelisation, orchestrator-workers, evaluator-optimiser, autonomous agent) and the deterministic-code-beats-LLM reflex for the parts of the workflow that do not need an LLM
- Implement production agents with the Claude Agent SDK, the OpenAI Agents SDK, LangGraph, or an in-house orchestrator — tool-calling, structured outputs, state management, retries, parallel tool calls, sub-agents, handoffs
- Design the human-in-the-loop checkpoint model — when the agent confirms before action, when the agent asks for clarification, when the agent falls back to a human queue — grounded in the customer's risk tolerance per action class
- Map agent tools to the customer's permission model — least-privilege per-tool-call, per-session scoping, per-tenant tool allow-lists, and the audit trail the customer's security team will subpoena
- Harden against the OWASP Agentic Top 10 — indirect prompt injection via retrieved content or tool output, tool-abuse, memory poisoning, confused-deputy, non-human identity management, authorisation drift — with concrete mitigations (output sanitisation, tool sandboxing, action-plan review, action-limit budgets, kill-switch)
- Cite the boundary to `agentic-ai-engineer` (level 30) on deep agent-framework internals; cite the boundary to `agentic-safety-engineer` (level 40) on frontier-agent red-team methodology

## Chapters

- [01 — The Agentic Overlay — Why Enterprise Agents Are Not Generic Agents](01-agentic-overlay.md)
- [02 — The Agent Pattern Catalogue and the Deterministic-Code Reflex](02-agent-pattern-catalogue.md)
- [03 — Framework and Runtime Selection — Claude Agent SDK, OpenAI Agents SDK, LangGraph, or In-House](03-framework-and-runtime-selection.md)
- [04 — Human-in-the-Loop Checkpoints — Confirm, Clarify, Queue](04-human-in-the-loop-checkpoints.md)
- [05 — Tool Permissions, Scoping, and the Audit Trail](05-tool-permissions-and-audit.md)
- [06 — The OWASP Agentic Top 10 — Threat Model and Mitigations](06-owasp-agentic-top-10.md)
- [07 — Boundaries to the `agentic-ai-engineer` and `agentic-safety-engineer` Peers](07-boundaries-to-agentic-peers.md)

## Exercises

- [01 — Pick the right pattern from Anthropic's effective-agents catalogue](exercises/exercise-01-pick-the-right-pattern-from-anthropics-effective-agents-catalogue.md)
- [02 — Ship a production agent with Claude Agent SDK or OpenAI Agents SDK](exercises/exercise-02-ship-a-production-agent-with-claude-agent-sdk-or-openai-agents-sdk.md)
- [03 — Design the human-in-the-loop checkpoint model](exercises/exercise-03-design-the-human-in-the-loop-checkpoint-model.md)
- [04 — Map agent tools to the customer's permission model](exercises/exercise-04-map-agent-tools-to-the-customers-permission-model.md)
- [05 — OWASP Agentic Top 10 threat model and mitigations](exercises/exercise-05-owasp-agentic-top-10-threat-model-and-mitigations.md)

## Structure

- `01-…md` … `07-…md`: lecture chapters.
- `exercises/`: per-exercise prompts.
- `labs/`: long-form hands-on labs (scaffolded placeholder; labs land on later cycles).
- `quizzes/`: knowledge checks (scaffolded placeholder).
- `resources.md`: external references.

## How this module composes with the rest of the track

- Reads from mod-103 (identity + data surfaces), mod-104 (gateway + SLOs), mod-105 (grounding + citation UX).
- Writes to mod-107 (agent-trajectory gold set), mod-108 (threat-model artefacts for formal review), mod-109 (rollout phases), mod-110 (observability signals), mod-111 (reusable patterns), mod-112 (narration to the customer).
- Names the boundaries to two peer tracks: `agentic-ai-engineer` (level 30) and `agentic-safety-engineer` (level 40).
