# Chapter 03 — Framework and Runtime Selection — Claude Agent SDK, OpenAI Agents SDK, LangGraph, or In-House

## Why this chapter exists

Chapter 02 picked the pattern. This chapter picks the *runtime* — the SDK, framework, or in-house orchestrator that turns the pattern into shipping code. The FDE's choice has to survive the customer's language stack, their security review (code audit, SBOM, CVE posture), their observability requirements, and the next engineer who inherits the pilot.

Framework depth is `agentic-ai-engineer` (peer, level 30) craft: tool-calling internals, state-graph semantics, interrupt and resume mechanics, sub-agent spawning, multi-agent coordination. The FDE's job is to pick the framework that fits the customer-environment overlay from Chapter 01 and brief the peer when depth is needed. This chapter teaches the surface the FDE needs to make that choice defensibly.

## The four shapes

Four runtime shapes cover most pilots. Each has a canonical implementation, a common language, and a distinctive strength and weakness. Fourth-party frameworks (AutoGen, CrewAI, LlamaIndex Agents, Semantic Kernel, PydanticAI, and others) exist and are legitimate; they map onto the four shapes below for purposes of selection.

### Shape 1 — Vendor agent SDK (Claude Agent SDK, OpenAI Agents SDK)

Managed agent runtimes from the model vendor. [Anthropic's Claude Agent SDK](https://docs.claude.com/en/api/agent-sdk/overview) and [OpenAI's Agents SDK](https://openai.github.io/openai-agents-python/) each package tool-calling, structured outputs, sub-agents, handoffs, and streaming into one library tuned for their own models.

- **Strengths.** Lowest integration friction with the vendor's model and features (prompt caching, structured output mode, provider-native tools). First-class support for parallel tool calls and sub-agent/handoff patterns. The vendor patches the SDK when model behaviour changes.
- **Weaknesses.** Model lock-in is real — routing some steps to a competitor's model is harder, sometimes requiring a second runtime. The SDK's opinions (state model, tool spec format) are taken.
- **Security-review surface.** Vendor-maintained library; SBOM check is the vendor's package and transitive dependencies. The audit-log shape is the vendor's; mapping to the customer's SIEM is Chapter 05.
- **When it wins.** The pilot is committed to one provider family, the workflow pattern fits the SDK's opinions, and shipping speed beats framework flexibility.
- **When it loses.** Multi-provider routing is in scope, or the customer has a ban on the vendor's SDK (uncommon but not unheard of in federal and some banking settings).

### Shape 2 — Framework-level graph (LangGraph, LlamaIndex Workflows, Semantic Kernel, PydanticAI)

Vendor-agnostic frameworks that model the agent as a graph (nodes, edges, state). [LangGraph](https://langchain-ai.github.io/langgraph/) is the most common in enterprise contexts; others fill similar niches.

- **Strengths.** Explicit state machine means the pattern from Chapter 02 maps cleanly onto code. Checkpointing and resume are first-class, which composes with HITL (Chapter 04). Multi-provider routing is usually straightforward.
- **Weaknesses.** Framework surface is large and changes often — upgrade cadence matters for the pilot's maintenance posture. Some frameworks pull a transitive dependency set the customer's SBOM policy dislikes.
- **Security-review surface.** Third-party library with substantial transitive dependencies; SBOM and CVE scan are heavier than the vendor SDK. License (MIT / Apache-2.0 for most) tends to clear review; names to check with the customer's legal team early.
- **When it wins.** The pattern is Pattern 4 (orchestrator-workers) or Pattern 5 (evaluator-optimiser) with clear state transitions and HITL interrupts, or the pilot needs multi-provider routing.
- **When it loses.** The pilot is simple enough that the framework's weight is unjustified.

### Shape 3 — In-house orchestrator

A thin orchestration layer written in the customer's language, calling the model APIs directly. For a prompt chain or a router with a small tool catalogue, often a few hundred lines of code.

- **Strengths.** No framework dependency to review. Full control over state shape, retries, audit record format, SIEM integration. The code is directly reviewable by the customer's engineering team.
- **Weaknesses.** Every feature (parallel tool calls, sub-agent spawning, interrupt and resume) has to be built. Easy to under-invest in error handling, retries, and observability. Reinventing LangGraph badly is the characteristic failure mode.
- **Security-review surface.** The FDE's own code; SBOM footprint is minimal. Reviewable in a single sitting if kept small.
- **When it wins.** The pattern is Pattern 1 (prompt chain) or Pattern 2 (routing) with a short tool list, the customer's platform team is skilled and wants to own the running code, or framework dependency is a stop-ship.
- **When it loses.** The pattern needs real state management, multi-step interrupt/resume, or multi-provider routing. Then the orchestrator becomes a framework-in-disguise.

### Shape 4 — MCP-backed tool server with a host runtime

The agent runtime is one of the above shapes; the tool catalogue is exposed as an [MCP](https://modelcontextprotocol.io/) server. The customer's existing agent host (Claude Desktop, an IDE integration, a chat surface) or the pilot's host calls the server's tools over the protocol.

- **Strengths.** The tool catalogue is reusable across hosts, which composes with mod-103 Chapter 08's MCP story and with the customer's long-term integration posture. The server is auditable, versioned, and can be hosted inside the customer's trust boundary.
- **Weaknesses.** Adds a protocol boundary and the latency of that boundary. Security review covers *two* surfaces — the host and the server.
- **Security-review surface.** The MCP server is a service in the customer's environment; its authentication, its authorisation of tool calls, and its audit stream are first-class security artifacts (Chapter 05).
- **When it wins.** The pilot's tools will be reused by other agent hosts in the customer's environment, or the customer wants a protocol-stable tool catalogue the vendor cannot change unilaterally.
- **When it loses.** The pilot is a single-host, single-workflow deployment with no reuse horizon.

## The selection matrix

Score the candidate runtimes against the customer-environment overlay. Column weights are set per customer; the shape:

| Dimension | Vendor agent SDK | Framework graph | In-house orchestrator | MCP-backed |
|---|---|---|---|---|
| Multi-provider freedom | Low | High | High | Depends on host |
| SBOM / CVE footprint | Low–medium | Medium–high | Minimal | Server + host surfaces |
| State-graph / interrupt first-class | Varies | Yes | Build it | Depends on host |
| HITL / checkpoint fit | Varies | Strong | Build it | Depends on host |
| Multi-host reuse of tools | None | Within framework | None | First-class |
| Observability hooks | Vendor-shaped | Framework-shaped | FDE-shaped | Both surfaces |
| Customer audit-log clarity | Vendor-shaped | Framework-shaped | FDE-shaped (strongest control) | Server-shaped |
| Review-time cost | Lowest | Highest (framework surface) | Low (code only) | Medium × 2 surfaces |
| Upgrade cadence risk | Vendor's | Framework's | FDE's | Protocol + host |

Rules of thumb:

- **Review-time cost is a real axis.** A framework the AppSec reviewer will spend three weeks auditing is a three-week schedule slip. Vendor SDKs and small in-house orchestrators clear review fastest.
- **Multi-provider freedom is pattern-dependent.** If Chapter 02 picked a prompt chain where every step uses the same model, provider freedom does not matter.
- **Observability shape has to match the customer's SIEM.** mod-110 will cover the depth; the chapter's rule: whichever shape you pick, the audit record format is a non-negotiable artifact of the pilot (Chapter 05).

## Tool-calling, structured outputs, state — the mechanics the FDE must understand

Whichever framework wins, the FDE has to be able to read and audit the runtime loop. Six mechanics are the minimum vocabulary.

**Tool-calling.** The model emits a structured tool-call intent; the runtime validates the arguments against the tool's JSON Schema; the runtime calls the tool; the tool's output is appended to the conversation and the model is re-invoked. See the provider docs for the exact shapes: [Anthropic tool use](https://docs.claude.com/en/docs/agents-and-tools/tool-use/overview), [OpenAI function calling](https://platform.openai.com/docs/guides/function-calling). Mechanics the FDE must know:

- JSON Schema is the contract. Keep schemas tight — fewer optional fields is fewer failure modes.
- The tool's return must be bounded. A tool that can return an unbounded blob poisons the context window.
- Tools are the agent's attack surface. Chapter 05 and Chapter 06 go deep.

**Structured outputs.** When a step must return JSON that matches a schema, use the provider's structured-output mode rather than prompting the model to "return JSON." Mod-104 Chapter 04 covered the provider-side shape. Agent-specific implication: every decision the model makes that will be consumed by downstream code should be a structured output, not free text parsed in the orchestrator.

**State management.** Three state surfaces live in the agent:

1. **Conversation state** — the message list the model sees. Bounded by context window; tuned by selective inclusion.
2. **Scratchpad state** — intermediate values the orchestrator passes between steps. This is where the FDE's code lives; keep it typed.
3. **Persistent state** — durable memory, checkpoints, resumption tokens. First-class in LangGraph, built in in-house orchestrators, handled by the vendor SDK's session abstraction. Chapter 06 covers the retention-policy and GDPR implications.

**Retries.** Three sources of transient failure:

- **Model API** (429, 5xx, timeout) — exponential backoff with jitter, cap on total elapsed time. The vendor SDKs do this; frameworks usually do; in-house orchestrators must.
- **Tool call** (downstream API error) — tool-specific, often bounded retry. Some tools are *not* idempotent and must not be retried without a correlation key (Chapter 05).
- **Validation** (structured output failed the schema) — retry with a repair prompt that includes the validation error. Cap iterations to two or three; after that, escalate.

**Parallel tool calls.** Modern models can emit multiple tool-call intents in one turn. The runtime should dispatch them in parallel and splice the results back in. The FDE must ensure that parallel tool calls into the customer's systems don't violate rate limits or ordering constraints. Default to parallel; serialise when the downstream API requires ordering.

**Sub-agents and handoffs.** A sub-agent is a scoped model call with its own system prompt, tool catalogue, and conversation state. A handoff is a transfer from one agent to another, usually with state carried over. These are the mechanics behind the orchestrator-workers pattern (Chapter 02). Enterprise implication: every sub-agent is a separate audit scope; every handoff is a logged transition. Keep the catalogue of sub-agents small and named.

## The customer-environment overlay, by shape

Each row of Chapter 01's twelve-row overlay lands differently on different runtimes. Highlights:

- **Framework choice × Customer's language stack.** The customer's primary language is not always the pilot's language. A Python framework (LangGraph, OpenAI Agents SDK) in a .NET shop can be a stop-ship for the platform team. The vendor agent SDKs typically ship Python and TypeScript; LangGraph is Python-first; Semantic Kernel is .NET-first. If the customer's language stack is a hard constraint, filter the candidate set first, then score.
- **Framework choice × VPC install.** Any framework that needs outbound calls to a vendor-hosted LangSmith / tracing endpoint needs a decision: either the trace stream must stay in-region and in-trust (so disable the hosted tracing and point at an in-VPC collector), or the feature must be off.
- **Framework choice × Customer's compliance regime.** FedRAMP-bound deployments need every dependency to pass review. Verify that the specific version pins used in the pilot are in the customer's approved package set — or that the customer's process to add them is initiated early.
- **Framework choice × Right-to-erasure (GDPR).** Persistent agent memory (scratchpads, long-term memory, checkpoint stores) lives under the customer's data-retention policy. Design the agent's memory layer so individual records can be deleted on request. Chapter 06 revisits this under *memory poisoning* — the mitigation shapes compose with the retention shape.

## Running the three archetypes

**Archetype A — healthcare payer.** Primary runtime: Claude Agent SDK on AWS Bedrock (Anthropic Claude available on Bedrock via marketplace), with the retrieval tool backed by the Chapter 03 MCP wrapper from mod-103. Reasoning: (1) single provider in-pilot, (2) Bedrock BAA posture clears HIPAA review, (3) simple pattern (Pattern 1 → Pattern 4 in later phase) does not need framework-level graph. Alternative: LangGraph if the customer's platform team prefers Python and multi-provider routing is on the near horizon.

**Archetype B — federal agency.** Primary runtime: in-house orchestrator in the customer's chosen language (often Python or Go), calling a self-hosted model via vLLM (mod-104 Chapter 03). Reasoning: (1) FedRAMP posture favours reviewable in-house code over large framework surface, (2) the pattern (routing + parallelisation + chain) is tractable in a few hundred lines, (3) the customer's platform team owns the running system. Alternative: a minimal LangGraph deployment if the customer's platform team has already approved it from an earlier project. <!-- needs-research: current FedRAMP moderate / high status of common agent frameworks — this moves quickly and should be checked against the customer's package-approval system -->

**Archetype C — EU retail bank.** Primary runtime: OpenAI Agents SDK inside Azure AI Foundry's Azure OpenAI deployment, in EU region under the EU Data Boundary. Reasoning: (1) Azure AI Foundry is the customer's cloud of record for AI workloads and Azure OpenAI is in-boundary, (2) router + chain + evaluator-optimiser fit the SDK's opinions. Alternative: Semantic Kernel if the customer's platform team is .NET-heavy. <!-- needs-research: confirm current status of OpenAI Agents SDK support on Azure OpenAI deployments vs. direct OpenAI API -->

Each pick comes with an ADR (Chapter 03's exercise produces the shape) that the AppSec reviewer and the customer platform team can read in one sitting.

## Anti-patterns

**Picking the framework from Twitter.** The agent-framework space moves fast and the loudest options are not always the ones the customer's platform team will support for 18 months. Pick on the selection matrix, not on the week's trend.

**Rolling your own because "LangGraph is too complex."** The complexity is real; the alternative is often reinventing a chunk of it badly. If Pattern 4 or Pattern 5 is the shape, a framework graph pays for itself. Rolling your own is the right call for Patterns 1–2 with small tool lists; above that, reconsider.

**Taking on both a vendor SDK and a framework.** Some pilots end up running Claude Agent SDK *and* LangGraph *and* the vendor's own wrappers. The result is three state shapes and three retry policies to reconcile. Pick one top-level runtime; sub-agents may use a different shape only when the gain is specific and named.

**Shipping without an audit-log contract.** The framework's default logging is not the customer's audit-log contract. Chapter 05 insists on a named audit shape; the framework's logs are an input, not the output.

## Summary

- Four runtime shapes cover most pilots: vendor agent SDK, framework-level graph, in-house orchestrator, MCP-backed tool server with a host runtime.
- Each shape has distinctive strengths across multi-provider freedom, review-time cost, state-graph fit, HITL support, multi-host reuse, and observability.
- The customer's language stack, deployment model, and compliance regime are often filter dimensions — set them first, then score the remaining candidates.
- The six mechanics the FDE must speak fluently: tool-calling, structured outputs, state (three surfaces), retries (three sources), parallel tool calls, sub-agents and handoffs. Depth is `agentic-ai-engineer` (peer) craft; the FDE's surface is enough to brief.
- Three archetype picks: Claude Agent SDK on Bedrock for A; in-house orchestrator for B; OpenAI Agents SDK on Azure OpenAI for C. Each with a named alternative and documented reasoning.
- Chapter 04 is next: the human-in-the-loop checkpoint model, which is where the pattern-plus-runtime choice meets the customer's risk tolerance.
