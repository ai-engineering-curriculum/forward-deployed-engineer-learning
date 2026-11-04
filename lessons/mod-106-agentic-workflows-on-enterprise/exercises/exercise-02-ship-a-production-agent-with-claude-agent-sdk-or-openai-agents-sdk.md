# exercise-02: Ship a Production-Shaped Agent with the Claude Agent SDK or the OpenAI Agents SDK

**Estimated effort:** 3–4 hours

## Objective

Build a runnable agent for **one** of the three archetypes, implementing the pattern selected in exercise-01. The agent must demonstrate the Chapter 03 mechanics — tool-calling, structured outputs, state management, retries, parallel tool calls, sub-agents or handoffs where the pattern calls for them — using either the [Claude Agent SDK](https://docs.claude.com/en/api/agent-sdk/overview) or the [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/). This is a *shape* exercise: the agent should look like a production agent even though the tools are stubs and the customer environment is simulated locally. LangGraph, an in-house orchestrator, or an MCP-backed shape are legitimate alternative runtimes if the archetype's ADR argued for them.

## Prerequisites

- Completed [exercise-01](exercise-01-pick-the-right-pattern-from-anthropics-effective-agents-catalogue.md) or an equivalent pattern decision for the archetype you pick.
- Read [Chapter 02](../02-agent-pattern-catalogue.md) and [Chapter 03](../03-framework-and-runtime-selection.md).
- Have an API key for the provider you will use (Anthropic, OpenAI, or an in-trust proxy). If you cannot run against a real provider, the exercise is still valuable if you mock the provider SDK's response types and run the loop against the mocks.
- Python 3.11+ or the language the SDK supports. The chapter's examples are in Python; Node.js/TypeScript equivalents exist for both SDKs.

## Problem statement

Pick one archetype:

- **A — healthcare payer:** member-inquiry resolution, draft phase. The agent accepts an inquiry, retrieves relevant policy sections (stub tool returning seeded JSON), drafts a reply, and runs a deterministic PII sanitiser and tone validator in code. Output: a structured reply object with citations.
- **B — federal agency:** policy-analyst drafting + evidence packaging. The agent accepts a policy question, retrieves from (stub) policy-memo, regulation, and FOIA-archive indexes in parallel, and packages a citation-rich evidence pack.
- **C — EU retail bank:** customer-support draft reply + case triage. The agent accepts a case, classifies it into one of {straightforward-reply, complex-regulatory, complaints-ombudsman, cross-border, unknown}, dispatches to the matching chain, drafts a reply or routes to a queue, and returns the structured output.

Build the agent against seeded stub tools that return realistic JSON shapes. The pattern you implement must be the one you chose in exercise-01 (or a justified near-alternative).

## Requirements

1. **Runnable code.** A single-command entry point that runs the agent over 3–5 seeded inputs and prints the structured output. Include a `README.md` for the project explaining how to run it.
2. **Framework.** Use the Claude Agent SDK, the OpenAI Agents SDK, LangGraph, or an in-house orchestrator. Name the choice in the project README with a one-paragraph rationale that references your archetype's ADR.
3. **Tool shapes.** Each tool is defined with a tight schema — typed parameters, length limits, enums where applicable. Tools are plain Python / TypeScript functions with deterministic seeded outputs; a real deployment would swap in the customer-system client.
4. **Structured outputs.** Every decision the model makes that downstream code consumes is a structured output with a schema. No free-text parsing in the orchestrator.
5. **State management.** At least one of: a scratchpad kept in orchestrator state, a checkpoint that could resume the workflow, or a session-scoped memory. Demonstrate it on at least one seeded input.
6. **Retries.** Implement at least one retry policy — model-side (429/5xx with exponential backoff and a hard timeout) or tool-side (idempotent retry with a correlation key). The retry must be observable in the logs.
7. **Parallel tool calls OR sub-agents/handoffs.** Depending on your pattern: Archetype B's base shape asks for parallel retrieval; Archetype C's routing pattern composes with per-branch sub-chains; Archetype A's later-phase plan-first shape can demonstrate sub-agent delegation. Pick the one your ADR asks for.
8. **Observability.** Every tool call is logged with event_id, timestamp, tool_name, tool_input (sanitised), tool_output_hash, step_index. The log is structured JSON lines, writable to stdout or a file. This is a dress rehearsal for the Chapter 05 audit record.
9. **Kill-switch.** A single flag (environment variable or config file) disables the agent — tool calls return a deterministic refusal, the model is not called, the log records the kill-switch state. Demonstrate it on at least one seeded input.
10. **Tests.** At least five tests: one happy-path per pattern branch, one retry demonstration, one kill-switch demonstration, one schema-violation recovery (the model returns invalid output, the orchestrator recovers), one tool-error demonstration.

## Starter guidance

- The point of the exercise is **shape**, not polish. A 300-line Python project can satisfy every requirement; a 3000-line project is almost certainly over-built.
- Seed everything. Deterministic seeded inputs plus deterministic seeded tool outputs means the test suite is a replay, not a flake-prone integration test.
- The tool schemas are the contract with the model. Keep them tight — enums, length limits, required fields — and the model's parameter-abuse surface collapses.
- The orchestrator's state is where the FDE's code lives. Keep it typed (Pydantic / Zod / dataclasses) and tested.
- `agent-sdk` and `openai-agents` each ship their own retry defaults; verify they do what your ADR expects before relying on them. If you roll your own retries, bound total elapsed time in addition to retry count.
- Structured-output recovery usually belongs inside the orchestrator loop, not inside the model prompt. The repair prompt includes the validation error; cap iterations at two or three; after that, escalate.
- The log format you produce here will be the input for exercise-04 (permission audit) — so pick field names you can live with.
- If you cannot run against a real provider, use the SDK's test / mock utilities where they exist and stub `httpx` where they do not. The agent's shape should not depend on the provider being live.

## Acceptance criteria

- The project runs with a single command and prints structured outputs for 3–5 seeded inputs.
- The chosen framework and pattern match (or justified-ly differ from) your exercise-01 ADR.
- Every tool has a tight schema; the project would fail to start if the schemas were malformed.
- Every decision the model makes that reaches downstream code is structured, not free text.
- At least one retry demonstration, one parallel-tool-call OR sub-agent demonstration, one kill-switch demonstration, and one schema-violation recovery are observable in the logs.
- The audit-log shape has every field from the Chapter 05 table that applies at this phase (identity / scope fields can be placeholder values marked `TODO(exercise-04)`).
- The test suite has five tests, all passing.
- The project README names the framework choice, the pattern, the seeded inputs, and the limitations ("tool outputs are seeded JSON; swap for real clients in deployment").
- No secrets in source. API keys read from environment variables, keys for the test suite stubbed.

## Stretch goals

- Add an evaluator-optimiser step inside one of the pattern branches (even if your primary pattern is a chain or router). Loop capped at three iterations; evaluator's verdict logged alongside the final output.
- Add a second provider (if you built on Claude Agent SDK, add an OpenAI fallback or vice versa). Route one specific step to the alternative provider; keep the rest on the primary. Document what changes.
- Expose the tool catalogue as a tiny [MCP](https://modelcontextprotocol.io/) server and run the agent against the server instead of against in-process tools. You now have Chapter 03's Shape 4.
- Add a trajectory evaluation harness that scores each run against a seeded expected trajectory. This is the seed of mod-107's eval work; the output is a rubric score per run.
- For Archetype A later-phase or Archetype C later-phase: add the plan-first shape with a human-approval mock (a stub function that approves or rejects based on seeded policy) and demonstrate the orchestrator-workers pattern end-to-end.
