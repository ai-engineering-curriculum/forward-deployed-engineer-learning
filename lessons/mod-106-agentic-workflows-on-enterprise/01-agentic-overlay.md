# Chapter 01 — The Agentic Overlay — Why Enterprise Agents Are Not Generic Agents

## Why this chapter exists

Mod-104 landed the LLM layer of a pilot inside the customer's environment. Mod-105 landed grounding on the customer's data. Mod-106 picks up at the next layer: workflows where the model does not just answer, it *acts* — reads the customer's ticketing system, writes to their CRM, routes a request to a human, posts a comment on a case, calls another service, maybe calls itself again. That is the agentic layer.

The agent craft itself — the pattern catalogue, the framework internals, the mechanics of tool-calling and handoffs — is a specialist depth owned by [`agentic-ai-engineer`](https://github.com/ai-engineering-curriculum/agentic-ai-engineer-learning) (peer, level 30). The frontier-model red-team methodology — adversarial multi-turn evaluation, emergent-behavior discovery, capability elicitation under duress — is owned by [`agentic-safety-engineer`](https://github.com/ai-engineering-curriculum/agentic-safety-engineer-learning) (peer, level 40). This module does not re-teach either. It teaches the *customer-environment overlay* on top: pattern selection grounded in the customer's workflow, human-in-the-loop design grounded in the customer's risk tolerance, tool permissioning grounded in the customer's IdP, and OWASP Agentic Top 10 mitigation grounded in the customer's threat model. Chapter 07 writes both boundaries explicitly.

## The in-house craft and what it covers

As with mod-104 and mod-105, name the ground floor before adding the overlay. The agentic AI engineer at level 30 owns, inside their own product or on a generic workflow:

- **Pattern catalogue.** When a workflow wants prompt chaining vs. routing vs. parallelisation vs. orchestrator-workers vs. evaluator-optimiser vs. an autonomous agent, grounded in [Anthropic's *Building effective agents*](https://www.anthropic.com/engineering/building-effective-agents) framing.
- **Framework depth.** The [Claude Agent SDK](https://docs.claude.com/en/api/agent-sdk/overview), the [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/), [LangGraph](https://langchain-ai.github.io/langgraph/), an in-house orchestrator — tool-calling loops, structured outputs, state graphs, interrupts, checkpointing, parallel tool calls, sub-agents, handoffs.
- **Tool design.** JSON-Schema tool specs, idempotency, cost of a call, error taxonomy, retry and backoff.
- **Agent-level evaluation.** Task success rate, trajectory evaluation, cost per completed task, average tool calls per task, intervention rate.
- **Multi-agent orchestration.** Supervisor / worker, debate, planner / executor, when more agents help and when they hurt.
- **Long-horizon state.** Memory, scratchpads, interrupt-and-resume, sub-agent spawning.

Those six areas are craft. The agentic AI engineer builds them against generic workflows: the tools are chosen by the team, the permission model is "application-level roles," the compliance posture is the vendor's own, and the risk model is set by the vendor's product org.

The FDE at level 35 operates one level up. The same pattern catalogue, the same framework, the same tool-calling mechanics have to run inside a customer's trust boundary, acting against *that* customer's systems, with *that* customer's permission model, under *that* customer's risk tolerance, reviewed by *that* customer's AppSec team. Each of those re-opens a dimension the in-house craft treats as fixed.

## The agentic overlay

The agentic overlay is the set of concerns the FDE adds on top of agent craft. Each row is a dimension the in-house craft resolves once and the FDE re-opens per deployment.

| Dimension | What the in-house craft assumes | What the FDE re-opens | Where this module handles it |
|---|---|---|---|
| Pattern choice | Pick the pattern that fits the task | Pick the pattern that fits the task *and* survives the customer's risk tolerance, their change-management cadence, and their on-call appetite | Chapter 02 |
| Deterministic-code reflex | "Use the model when the LLM clearly helps" | Same, applied harder — every LLM hop is an audit, a cost line, and a failure mode the customer's platform team will have to answer for | Chapter 02 |
| Framework choice | Team's favourite | Must survive the customer's language stack, deployment model, and security review (code audit, SBOM, CVE posture) | Chapter 03 |
| Tool catalogue | Vendor's internal APIs | Customer's systems — Salesforce, ServiceNow, Jira, Snowflake, M365, SAP, custom in-house APIs — through the customer's IdP | Chapter 03, Chapter 05, composes with mod-103 Chapter 03 (SaaS surfaces) and Chapter 08 (MCP) |
| Tool granularity | One tool per capability | Smallest tool that still composes — larger tools expand the blast radius the AppSec reviewer has to accept | Chapter 05 |
| Human-in-the-loop model | "Confirm destructive actions" | Per-action-class risk model, co-authored with the customer's business owner and risk team | Chapter 04 |
| Permission model | Application-level | Customer's IdP groups, roles, attributes propagated per tool call; token exchange, OBO, downscoped tokens; non-human identity (NHI) hygiene | Chapter 05 |
| Audit trail | Product telemetry | SIEM-grade — every tool call, every prompt, every model output that drove an action, captured in a form the customer's security team can subpoena | Chapter 05, composes with mod-110 |
| Prompt-injection posture | Known risk, mitigated at ingest | Threat-modelled — customer's data, customer's tool outputs, customer's cross-tenant exposure all feed into the agent's context | Chapter 06 |
| Tool-abuse posture | Not usually modelled | Explicit — action-limit budgets, kill-switch, per-tool rate limits, out-of-band approval for high-risk action classes | Chapter 06 |
| Non-human identity (NHI) hygiene | Vendor's secrets vault | Customer's secrets store, customer's rotation cadence, customer's attestation posture | Chapter 05, Chapter 06 |
| Memory and long-horizon state | Chosen by the team | Chosen under the customer's data-retention policy, right-to-erasure rules (GDPR), and record-of-processing obligations | Chapter 03, Chapter 06, composes with mod-105 Chapter 04 (PII) |

Twelve rows of overlay — matching mod-104 and mod-105. The pattern holds: craft dimensions the peer specialist resolves once get re-opened by the FDE per customer, with gate dimensions enforced first.

## The three deployment-model archetypes, re-cast for agents

Mod-104 Chapter 01 named three deployment-model archetypes and mod-105 Chapter 01 inherited them. Mod-106 inherits them again, with the agentic layer's distinctive emphasis.

**SaaS multi-tenant.** The agent runs in the vendor's SaaS. Tool calls into customer systems happen through per-tenant OAuth connections, per-tenant secrets, per-tenant tool allow-lists. Memory (if used) is per-tenant. Audit log is per-tenant with vendor-side retention.

*Who owns what:* vendor owns the agent runtime, the tool-call plumbing, the audit pipeline. Customer owns the systems-of-record the agent reaches and the OAuth grants that let it in.

**Dedicated SaaS.** As SaaS multi-tenant, but the agent's compute and memory run in a customer-dedicated namespace / cluster / project. Audit log can stream directly to the customer's SIEM rather than through the vendor's. Some customers require this archetype before they will agree to any tool-call action beyond read-only.

*Who owns what:* vendor owns the running agent; customer-dedicated keys, audit stream, and sometimes customer-dedicated provider accounts.

**VPC install.** The agent runs inside the customer's own cloud. Tool calls are often intra-VPC (lower blast radius for exfiltration-via-tool). The customer's platform team usually operates the agent at the infra layer. Secrets live in the customer's vault. The agent framework must survive the customer's language stack and SBOM scan.

*Who owns what:* customer owns the running system at the infra layer. Vendor owns the agent-shaped code — the prompts, the tool wrappers, the pattern graph — and the upgrade cadence. Observability flows into the customer's SIEM. On-call is shared.

**On-prem / air-gapped.** As with LLM serving, rare but real. Open-weights models (Chapter 02, mod-104 Chapter 02) plus a self-hosted agent runtime (Chapter 03) plus tool calls that never leave the customer's trust boundary. Memory, if used, often lives in the customer's own database.

The agentic layer sharpens a key difference from the LLM and retrieval layers: *the customer's systems of action are the hottest surface in the system*. An agent that can read a document is a retrieval risk; an agent that can file a ticket, send an email, update a record, or trigger a downstream workflow is an *action* risk. Every deployment archetype has to answer "which actions can this agent take, under whose identity, on whose authority, with what reversibility?" Chapter 04 and Chapter 05 make that precise.

## The organising budget: six dimensions

Mod-104 ran against five dimensions, mod-105 added one for retrieval. Mod-106 keeps the shape but tilts the labels.

1. **Residency / sovereignty** — where does the agent's context, its memory, its audit trail physically rest? Where do tool-call payloads traverse?
2. **Compliance** — does this agent shape (framework, tools, memory, HITL model) satisfy the customer's regulatory regime, including sector-specific rules like SR 11-7 for model risk in banking, EU AI Act Article 14 (human oversight) for high-risk systems, and HIPAA for healthcare agent-mediated workflows?
3. **Blast radius** — the maximum business impact a mis-firing agent can cause before a human can stop it. This is agent-specific and the single hardest dimension to reason about correctly.
4. **Latency** — can the agent complete its workflow inside the end-to-end latency budget the pilot promised (mod-104 Chapter 05)? Multi-step agents compound latency across every tool call.
5. **Cost** — does per-workflow cost (tokens + tool-call fees + human-queue cost when the agent escalates) fit inside the value-per-task promise (mod-102 Chapter 03)?
6. **Agent-choice freedom** — does this choice let the FDE route sub-tasks to the most-fit sub-pattern or model, or is it locked to one agent shape?

Rules of thumb that carry from mod-104 and mod-105:

- **Residency, compliance, and blast radius are gate dimensions.** A choice that fails any of them is off the table — cost or latency advantages do not recover it. Blast-radius-as-a-gate is new in this module: an agent that could legally and in-residency delete every row in the customer's primary CRM is not a cost-vs-latency trade; it is a stop-ship.
- **Latency, cost, and agent-choice freedom are optimisation dimensions.** Among gate-passing candidates, the FDE picks the point that best fits the workflow and the business KPI.

Blast radius deserves a sentence of foreshadowing because the next six chapters each sharpen it: Chapter 02 bounds it by pattern choice (prompt chaining has lower blast radius than autonomous agent); Chapter 04 bounds it by HITL checkpoints; Chapter 05 bounds it by tool-permission and action-limit budget; Chapter 06 bounds it by the OWASP Agentic Top 10 mitigation stack.

## Three customer archetypes for the rest of the module

As with mod-104 and mod-105, three sketches keep the discussion concrete. The archetypes are the same shape so pilots compose across the three modules, with the agentic surface filled in.

**Archetype A — Mid-size US healthcare payer.**
Workflow candidate: member-inquiry resolution. The CSR describes the inquiry; the agent retrieves policy + plan docs (mod-105), drafts a response, and — in a later pilot phase — can file a prior-authorisation correction ticket and post an internal note to the case. Permission shape: role-based (CSR tier 1 / tier 2 / supervisor), with DAC on member-sensitive items. Compliance: HIPAA + SOC 2 Type II; records-of-processing. Deployment: AWS Bedrock + VPC install. Action risk: medium — some actions (filing a prior-auth correction) have downstream claims impact.

**Archetype B — US federal agency.**
Workflow candidate: policy-analyst drafting + evidence packaging. The analyst asks a question; the agent retrieves memos / regulations / FOIA archives (mod-105), drafts, and packages a citation-rich evidence pack. Actions are read-plus-write-to-the-analyst's-draft-only — no actions outside the analyst's own workspace in the pilot phase. Permission shape: FOUO / CUI classification, cleared-personnel attributes. Compliance: FedRAMP Moderate baseline, path to High. Deployment: GovCloud + self-hosted agent runtime + self-hosted inference. Action risk: low in pilot — read-plus-draft; escalates in a later phase if the agent begins filing with internal systems.

**Archetype C — EU retail bank.**
Workflow candidate: customer-support draft reply + case triage. The agent drafts a reply (mod-105), flags cases that need regulatory or complaints-ombudsman routing, and can — in a later pilot phase — update the case record's triage status and assign to a specialist team. Permission shape: per-branch / per-region / per-team DAC. Compliance: GDPR + DORA; EU AI Act Article 14 if the workflow is classified high-risk. Deployment: Azure AI Foundry under EU Data Boundary. Action risk: medium — triage updates affect customer SLAs; routing errors have regulatory-timeline consequences.

Each chapter runs decisions through these three archetypes. The archetypes are shapes, not prescriptions.

## Where mod-106 sits with the rest of the track

The agentic overlay only makes sense in context.

- **Mod-103 (enterprise integration)** — the customer-systems surfaces (Chapter 03, Chapter 04) are what the agent's tools call. The MCP server (Chapter 08) is sometimes the exposure path for the agent's tool catalogue. The identity surface (Chapter 02) is the substrate for Chapter 05's tool-permission story.
- **Mod-104 (LLM applications)** — the gateway (Chapter 03) is the choke point every agent step traverses. The per-tenant prompt architecture (Chapter 04) composes with per-tenant tool-allow-lists. The per-customer latency SLO (Chapter 05) now has to cover multi-hop agent workflows.
- **Mod-105 (grounding)** — retrieval is the most common tool in the agent's catalogue. Permission propagation (Chapter 03) and PII hygiene (Chapter 04) from mod-105 compose with this module's permission and injection story.
- **Mod-107 (eval engineering)** — agent-level evaluation (task success, trajectory, cost-per-completed-task, intervention rate) is bootstrapped here and harnessed in mod-107.
- **Mod-108 (enterprise security and compliance)** — the formal review of the agent's tool catalogue, HITL model, and OWASP mitigation stack lives here in draft and gets formalised in mod-108.
- **Mod-109 (deployment, rollout)** — rollout of agent changes needs shadow-mode first, then HITL-first, then measured auto-run (Chapter 04 foreshadows this).
- **Mod-110 (observability, on-call)** — intervention rate, action-limit-breach rate, kill-switch activations, tool-error rate are the signals that fire in production.
- **Mod-111 (field → product loop)** — reusable patterns, tool wrappers, HITL templates flow back to the product team.
- **Mod-112 (consulting craft)** — how the agent's actions are narrated to the customer's executive sponsor; how an agent incident is communicated.

## Four anti-patterns to catch early

**Reaching for an autonomous agent when prompt chaining would do.** Pattern 6 in Anthropic's catalogue (the autonomous agent) is the highest-blast-radius shape and the hardest to evaluate. For most pilot workflows a prompt chain, a router, or an orchestrator-workers pattern is both cheaper and safer. Chapter 02 makes this the module's first discipline: deterministic code first, simple patterns second, autonomy last.

**Mapping tools to "whatever the model can call" rather than to the customer's permission model.** The customer's security team will look at the agent's tool list and ask: "Under which identity does each of these execute? What is the blast radius if we revoke tomorrow? Where is the audit record?" Chapter 05 answers that upfront; retrofitting it is costly.

**Treating human-in-the-loop as a fallback for low-confidence cases only.** Low-confidence confirms are table stakes, but Chapter 04 teaches that HITL is really a *per-action-class* design: some action classes require confirmation regardless of model confidence, because the business cost of an action is asymmetric with its reversibility.

**Running the agent against the customer's production data without threat-modelling the tool outputs.** The classic failure: an agent reads customer tickets or emails, treats their contents as trusted context, and follows injected instructions from an external user through a tool-returned document. Chapter 06 names this, and it is the OWASP Agentic Top 10's first finding.

## Summary

- The agentic overlay is the set of customer-environment concerns the FDE adds on top of the agent craft owned by `agentic-ai-engineer` (peer, level 30) and the red-team methodology owned by `agentic-safety-engineer` (peer, level 40). Mod-106 does not re-teach either; it layers.
- Twelve dimensions are re-opened per customer — from pattern choice through tool granularity to NHI hygiene. Each chapter in this module takes one or two.
- Three deployment archetypes (SaaS multi-tenant, dedicated SaaS, VPC install) carry over, with the agentic layer's distinctive emphasis: the customer's systems of action are the hottest surface.
- The organising budget is six-dimensional: residency, compliance, blast radius, latency, cost, agent-choice freedom. Residency, compliance, and blast radius are gates; latency, cost, and agent-choice freedom are optimisations.
- Three customer archetypes — US healthcare payer, US federal agency, EU retail bank — thread through every chapter.
- Four anti-patterns to catch early: reaching for autonomy by default, mapping tools to model capability rather than permission model, treating HITL as a confidence fallback only, and skipping tool-output threat-modelling.
- Chapter 02 is next: choosing the right agent shape from Anthropic's pattern catalogue, with the deterministic-code-beats-LLM reflex applied hard.
