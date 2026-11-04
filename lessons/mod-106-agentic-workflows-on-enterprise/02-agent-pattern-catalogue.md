# Chapter 02 — The Agent Pattern Catalogue and the Deterministic-Code Reflex

## Why this chapter exists

Chapter 01 named pattern choice as the first row of the agentic overlay. This chapter runs that choice: walk the six patterns Anthropic's [*Building effective agents*](https://www.anthropic.com/engineering/building-effective-agents) post catalogues, score each against the customer's workflow, and apply the deterministic-code-beats-LLM reflex to every step that does not actually need a model. The output of the chapter is a pattern decision per workflow, defensible to the customer's business owner, their AppSec reviewer, and the FDE's own on-call future self.

The patterns described in this chapter are not the FDE's invention; they are a distillation of the agent craft owned by `agentic-ai-engineer` (peer, level 30). The depth of each pattern — including the exact framework-level graphs, interrupt semantics, and multi-agent orchestration — is the peer's craft. The FDE picks the pattern that fits the customer's workflow and cites the peer for depth.

## Start with deterministic code, not with agents

Before any pattern fires, ask the deterministic-code question. Anthropic's post is explicit: "When building applications with LLMs, we recommend finding the simplest solution possible, and only increasing complexity when needed." The reflex, in FDE form:

1. **If a step can be done with a SQL query, a regex, or an existing API call, it is not an LLM step.** It is a line of code. Each LLM hop is a cost, an audit event, a failure mode, and a latency tax the customer's platform team will carry.
2. **If a step is deterministic rule-following, it is not an LLM step.** If the customer's policy says "tier-1 CSRs cannot process refunds over $500," that is a validator in code, not a prompt instruction. The model cannot be trusted to enforce a hard rule reliably; the deterministic validator can.
3. **If a step is a structured transformation the model does reliably, it may be an LLM step with structured output, not an agent step.** Mod-104 Chapter 04 covered this: structured-output mode with a schema and a retry-on-invalid is a *workflow*, not an agent.
4. **If a step requires iterative tool use, state over multiple hops, or routing based on model-produced judgement, it is an agent step.** Only then does the pattern catalogue apply.

The anti-pattern is the opposite reflex: every workflow becomes "an agent" because the team found a framework they like. The cost shows up in tokens per completed task, in evaluation cost (every step is non-deterministic), in audit volume (every step emits a tool call to review), and in the customer's willingness to turn the pilot on.

Two rules of thumb for the deterministic reflex:

- **Push the deterministic parts to the edges of the agent.** Pre-process input with code; post-process output with code; wrap every tool with a code-level validator. The agent's only job is the parts a model actually does well.
- **Count LLM hops per task before and after a design pass.** If the number went up but the task-success rate did not move, the design regressed.

## The six patterns

Anthropic's catalogue distinguishes five *workflow* patterns (predefined control flow, model is a step) from one *agent* shape (dynamic control flow, model drives the loop). The FDE needs all six in their vocabulary.

### Pattern 1 — Prompt chaining

A workflow in which the output of one LLM call becomes the input of the next, with a deterministic step between them. Classic shape: *extract → validate → transform → generate*.

- **When this pattern wins.** A task that decomposes cleanly into fixed sub-tasks, where latency budget can afford the serial hops, and each sub-task is easier (and cheaper to evaluate) than the whole.
- **When it loses.** When the task's shape depends on the input and the chain would need to branch. Then it is routing (Pattern 2), not chaining.
- **Enterprise-overlay implication.** Lowest blast radius of any model-driven pattern — every step is bounded, every step can be gated, every step emits an audit record the customer can read. Default to prompt chaining when the workflow fits.

*Archetype A example.* Member-inquiry resolution, draft-only phase: (1) classify inquiry type with a structured-output call; (2) retrieve the right policy section (mod-105); (3) draft the reply conditional on inquiry type; (4) run a PII sanitiser and a tone validator in code; (5) surface to the CSR with citations. Four LLM calls, three deterministic steps, zero autonomy.

### Pattern 2 — Routing

A workflow in which the first model call classifies the input and routes it to one of several specialised downstream chains. Classic shape: *classify → dispatch → handle*.

- **When this pattern wins.** Tasks with distinct input kinds, where each kind benefits from a different downstream chain (different prompts, different tools, different models). The classifier runs cheap; the downstream handler does the heavy lifting.
- **When it loses.** When the classes blur (model confidence low on a majority of inputs), or when the "right" handler for a borderline input is "a bit of both." In those cases, orchestrator-workers (Pattern 4) reshapes the problem.
- **Enterprise-overlay implication.** Routes directly onto the customer's own intake taxonomy. If the customer's CSR tool already classifies tickets into 12 categories, the router often shadows the existing classifier for the first week — a soft integration path and a free eval signal.

*Archetype C example.* Customer-support draft reply + case triage: (1) classify incoming case into {straightforward-reply, complex-regulatory, complaints-ombudsman, cross-border}; (2) dispatch to the matching chain; (3) the complaints-ombudsman branch does not draft a reply at all — it routes to a human queue with an evidence pack.

### Pattern 3 — Parallelisation

A workflow in which the model is called multiple times in parallel on the same or related inputs and their outputs are aggregated. Two shapes: *sectioning* (split a task into independent sub-tasks) and *voting* (run the same task N times and aggregate).

- **When this pattern wins.** When the sub-tasks are independent and the wall-clock saving matters (sectioning), or when model variance is a known risk and an aggregate is more reliable than a single run (voting).
- **When it loses.** When the sub-tasks are not actually independent (hidden dependencies cause inconsistent outputs) or when the cost of N× calls outweighs the reliability gain. Voting against hallucination in low-stakes tasks is cost nonsense; voting against hallucination in high-stakes classification is often worth it.
- **Enterprise-overlay implication.** Parallelisation's audit trail is N× the single-call shape. Price it in. Also: if the customer's rate limits on the provider or on a downstream tool are tight, parallel fan-out saturates them faster.

*Archetype B example.* Policy-analyst evidence packaging: fan out retrieval across (policy-memo index, regulation index, FOIA-archive index), run a per-source summariser in parallel, aggregate into the final evidence pack. Each per-source summariser is bounded and fast; the aggregator is a cheap structured-output call.

### Pattern 4 — Orchestrator-workers

A workflow in which a central "orchestrator" LLM decomposes a task, delegates sub-tasks to worker LLM calls, and synthesises the result. The orchestrator plans; the workers execute.

- **When this pattern wins.** When the task's decomposition is not known in advance (so routing doesn't fit) and the sub-tasks are too varied for parallelisation. The orchestrator produces a plan; the workers execute the plan's steps; the orchestrator synthesises.
- **When it loses.** When the orchestrator over-decomposes a task that should be one call, or when the planning step itself is unreliable. The orchestrator is a meta-planner and its mistakes multiply across the workers.
- **Enterprise-overlay implication.** The orchestrator is a decision-maker — its outputs become actions. Chapter 05 insists this is where tool permissions concentrate. Chapter 04 insists this is where HITL checkpoints often live.

*Archetype A example, later phase.* Member-inquiry resolution with action: the orchestrator plans the workflow (retrieve policy → draft reply → file a prior-auth correction if the policy indicates mis-application), delegates each step to a worker, and surfaces the plan to the CSR *before* any action worker runs. The CSR confirms or edits the plan; only then do the action workers execute. HITL at the plan step, not at each tool call.

### Pattern 5 — Evaluator-optimiser

A workflow in which one LLM call generates a candidate output and a second LLM call evaluates it, feeding the evaluation back to the generator. The loop runs until the evaluator passes or a budget is hit.

- **When this pattern wins.** When a clear evaluation criterion exists (and can be expressed as a prompt), and iterative refinement reliably improves the output. Content generation with a style guide, code generation against a test suite, draft replies against a quality rubric.
- **When it loses.** When the evaluator is only slightly better than the generator (both make the same mistakes), when the evaluation is subjective (two humans disagree), or when iteration doesn't converge and the loop just spins.
- **Enterprise-overlay implication.** The loop budget is a cost knob and a kill-switch. The evaluator's output is itself a decision log — the audit trail should capture *why* the loop stopped (passed, budget exhausted, hard-cap hit).

*Archetype C example.* Draft reply quality: generator writes the reply; evaluator checks against {tone guide, regulatory-tone constraints, length limit, mandatory-disclaimer-included}; loop capped at three iterations. The evaluator's verdict is logged with the final output.

### Pattern 6 — The autonomous agent

The dynamic shape: the model decides at each step which tool to call, inspects the tool output, decides the next step. The loop exits when the model decides it is done or a budget trips. No predefined control flow.

- **When this pattern wins.** When the workflow is genuinely open-ended — the number of steps is not known in advance, the tool set is large, and the model's judgement about "what next" is the hard part. Code-editing assistants, research assistants over a corpus, deeply tool-driven investigation tasks.
- **When it loses.** When any of the previous five patterns would do, when the blast radius of a mis-fire is unacceptable, or when the customer's compliance regime cannot accept a decision-chain the model owns. Most pilot workflows do not need Pattern 6. The honest answer in week two is often: "This is really Pattern 1 or Pattern 4, with Pattern 6 behind a feature flag for the later phases."
- **Enterprise-overlay implication.** This is the highest-blast-radius pattern. Chapter 04's HITL discipline, Chapter 05's permission discipline, and Chapter 06's OWASP mitigations tighten per step. Autonomous agents also need an *action-limit budget* — a hard cap on tool calls, tokens, or wall time — because an unbounded loop in a customer environment is a stop-ship finding.

*Archetype A example, counter-example.* It is tempting to make the whole member-inquiry workflow an autonomous agent. Don't. The patterns above cover it at lower blast radius with better eval economics. Keep Pattern 6 in your pocket for the workflows that actually need it.

## The decision table

A five-column table runs the choice for one workflow.

| Dimension | Prompt chain | Routing | Parallelisation | Orchestrator-workers | Evaluator-optimiser | Autonomous agent |
|---|---|---|---|---|---|---|
| Blast radius | Lowest | Low | Low (per call) × N calls | Medium | Low–medium | Highest |
| Latency profile | Serial, predictable | 1 classifier + handler | Fastest wall-clock on independent sub-tasks | Serial + sub-tasks | Serial × loop budget | Unpredictable |
| Cost per task | Low | Low | N × base | Medium | Medium × iterations | Highest |
| Evaluation shape | Per-step | Per-branch | Per-worker + aggregate | Per-plan + per-worker | Per-iteration | Trajectory |
| Audit record density | Low | Low | Medium | Medium | Medium | High |
| Customer explainability | High | High | Medium | Medium | Medium | Lowest |
| HITL surface | Pre/post | Pre/post | Pre/post | Pre-plan, pre-action | Per iteration | Per step (if any) |

Rules of thumb for the table:

- **Walk left-to-right.** Start with the leftmost pattern that could plausibly fit; move right only when the workflow reveals it cannot.
- **Customer-explainability is a real axis.** The customer's SMEs have to trust the pilot's answers. A prompt chain is easier to narrate than an autonomous agent. In regulated industries (healthcare, finance, federal), explainability may be a gate, not an optimisation.
- **Combine patterns at different scopes.** Most real pilots are a prompt chain at the top level with an evaluator-optimiser inside one step. That is a legitimate hybrid, not a failure.

## When patterns compose

The six patterns are not mutually exclusive. The useful compositions:

**Chain-of-chains.** A top-level prompt chain where one step is itself a chain. Natural when a sub-task has its own extract-transform-generate shape (e.g., "generate the reply" expands into sub-steps for structure extraction, body drafting, citation injection).

**Router of chains.** A router that dispatches to pattern-specific chains. Each branch is a dedicated chain tuned to its class. Archetype C's draft-reply + triage lives here.

**Orchestrator-over-tools.** An orchestrator where the "workers" are not LLM calls but deterministic tools. The orchestrator plans; the plan executes against code. This is often the correct "agentic" shape for a pilot with read-plus-action scope — the model does the hard reasoning once (the plan), and the actions are deterministic given the plan.

**Evaluator inside a chain.** A single step in a chain uses evaluator-optimiser to refine its output. Common for high-stakes drafting (Archetype C's regulatory-tone compliance).

**Autonomous agent inside a HITL harness.** Pattern 6 with Chapter 04's checkpoints making the autonomy safe: the agent proposes a plan, the human confirms, the agent runs the plan, every tool call above an action-class threshold pauses for confirmation.

The composition rule: compose up from the smallest pattern that works. Each nesting step adds blast radius, cost, and eval complexity; name the reason it is needed.

## The deterministic-code reflex, re-run per step

Even inside an agent shape, revisit the deterministic reflex at every step:

- **Does this step need an LLM?** If the step is "load the user's current tier from the CRM," that is a tool call to the CRM API, not an LLM call to "ask the model what the user's tier is."
- **Does this step need an LLM to make a decision, or just to format a string?** String formatting with a template is deterministic and cheap. If the model is doing nothing but filling slots, the slots are code.
- **Does this step need an LLM, or does a prior step's structured output already carry the answer?** A frequent regression — the model generates a classification, then a later step re-classifies the same input because the prompt author forgot the earlier step already decided.
- **Could a retrieval call replace an LLM call?** The grounded answer in the knowledge base is often more reliable than a re-derivation inside the model.

The reflex is not "avoid LLMs"; it is "put the LLM where it earns its latency, cost, and audit tax."

## Applying the catalogue to the three archetypes

**Archetype A — member-inquiry resolution.**
- *Draft-only phase:* prompt chain with one retrieval step. Pattern 1.
- *Action-enabled phase:* orchestrator-workers with HITL at the plan step and deterministic action workers. Pattern 4 + HITL harness (Chapter 04). No Pattern 6.

**Archetype B — policy-analyst drafting.**
- *Base shape:* routing on question type → prompt chain per type, with parallelisation of retrieval across sources. Patterns 2 + 3 + 1.
- *Evidence-pack packaging:* evaluator-optimiser inside the packaging step with a cap of three iterations. Pattern 5.
- No Pattern 6 in the pilot — the workflow is explainable enough without it.

**Archetype C — support reply + triage.**
- *Base shape:* router with five downstream chains (straightforward-reply, complex-regulatory, complaints-ombudsman, cross-border, unknown). Pattern 2.
- *Draft-reply chain:* prompt chain with evaluator-optimiser on tone + regulatory compliance. Patterns 1 + 5.
- *Triage-update, later phase:* orchestrator-workers with HITL at the plan step. Pattern 4.

The three archetypes share no autonomous-agent step in their pilot phase. That is not a coincidence — it is the deterministic-code reflex working.

## Anti-patterns

**"We're building an agent."** As a pitch, fine. As a design, harmful. Name the pattern. If the pattern is Pattern 6, name why Patterns 1–5 do not fit.

**Picking the pattern from framework examples.** LangGraph, the Claude Agent SDK, and the OpenAI Agents SDK each ship with example code that lands in Pattern 6 because that is the most demo-friendly shape. The example is not the pattern for the customer's workflow.

**Skipping the latency compound.** Each hop adds latency. A five-hop chain against a model with p95 ≈ 1.8 s per call is a nine-second p95 workflow before any tool calls. Compare with the mod-104 Chapter 05 budget.

**Letting the evaluator always pass.** An evaluator-optimiser whose evaluator always says "good enough" has no evaluator. Name the specific failure modes the evaluator checks for; measure its disagreement rate against a human.

**Running the autonomous agent on day one.** Chapter 09 of mod-109 will cover rollout, but the preview: Pattern 6 is never the first phase. Shadow-mode a simpler pattern first; graduate to Pattern 6 only when the data justifies it and the HITL harness is proven.

## Summary

- The first move is always the deterministic-code reflex: push steps into code when they do not need a model. Each LLM hop is a cost, an audit event, a failure mode, and a latency tax.
- Six patterns from Anthropic's *Building effective agents* make the vocabulary: prompt chaining, routing, parallelisation, orchestrator-workers, evaluator-optimiser, autonomous agent.
- Walk the decision table left-to-right. Pick the simplest pattern that fits; only move right when the workflow reveals it must.
- Patterns compose. Most real pilots are a chain or router at the top with specialised sub-shapes inside.
- The three archetypes land on Patterns 1–5 in their pilot phase. Pattern 6 earns its place only when 1–5 demonstrably cannot carry the workflow.
- Chapter 03 picks up the next question: once the pattern is chosen, how is it built — Claude Agent SDK, OpenAI Agents SDK, LangGraph, or an in-house orchestrator?
