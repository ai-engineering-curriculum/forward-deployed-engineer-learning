# Chapter 07 — Boundaries to the `agentic-ai-engineer` and `agentic-safety-engineer` Peers

## Why this chapter exists

Mod-101 Chapters 03 and 06 fixed the general rule: the FDE at level 35 is a specialist IC among peers, and escalation to the peer who owns a specific craft is a discipline, not a failure. Mod-103 Chapter 09 instantiated that rule for `ai-infra-inference-engineer` and `ai-data-engineer`. Mod-105 Chapter 07 instantiated it for `rag-engineer`. This chapter closes mod-106 by naming the two peers the agentic layer brings in: [`agentic-ai-engineer`](https://github.com/ai-engineering-curriculum/agentic-ai-engineer-learning) (peer, level 30) on deep framework internals, and [`agentic-safety-engineer`](https://github.com/ai-engineering-curriculum/agentic-safety-engineer-learning) (peer, level 40) on frontier-agent red-team methodology.

Two peers, two different bars.

## The `agentic-ai-engineer` boundary

### What this peer owns

The agent craft at depth, independent of where the agent runs:

- **Pattern-catalogue depth.** The six patterns in Chapter 02 at the theoretical level — when does orchestrator-workers beat autonomous? What is the reasoning-vs-tool-use trade at scale? When does a multi-agent debate beat a single-agent chain?
- **Framework internals.** The actual state-graph semantics of LangGraph; the interrupt mechanics of the Claude Agent SDK; the handoff semantics of the OpenAI Agents SDK; the sub-agent lifecycle in each. Evaluating a framework upgrade against a specific deployment.
- **Tool-calling mechanics at depth.** Parallel tool-call scheduling, retry policy design across heterogeneous tools, structured-output recovery strategies, streaming tool use.
- **Agent-level evaluation.** Trajectory evaluation methodology, cost-per-completed-task modelling, agent-level A/B testing, intervention-rate analysis, model-vs-model comparisons on agent benchmarks ([τ-bench](https://github.com/sierra-research/tau-bench), [SWE-bench](https://www.swebench.com/), [WebArena](https://webarena.dev/), [GAIA](https://huggingface.co/spaces/gaia-benchmark/leaderboard)).
- **Multi-agent orchestration patterns.** Supervisor-worker, debate, planner-executor, blackboard, and when each helps vs. hurts.
- **Long-horizon state design.** Memory architectures, checkpoint stores, resumption semantics, context-window management across hours and days.
- **Reasoning models in agent loops.** When to use a reasoning model (Claude Opus/Sonnet with extended thinking, OpenAI o-series, Gemini Flash Thinking) as the orchestrator; how to budget reasoning tokens.
- **Agent benchmarking against generic workflows.** The craft's own benchmarks on open-source datasets.

In short: *everything that makes an agent complete its task reliably, independent of where the agent runs*.

### What the FDE adds

The FDE owns the *deployment-aware* parts:

- Pattern choice grounded in the customer's workflow, risk tolerance, and explainability needs (Chapter 02).
- Framework choice grounded in the customer's language stack, security review, and deployment model (Chapter 03).
- Human-in-the-loop design grounded in per-action-class risk, co-signed by the business owner (Chapter 04).
- Tool-permission design grounded in the customer's IdP and audit requirements (Chapter 05).
- Standard-threat-model mitigation grounded in the OWASP Agentic Top 10 (Chapter 06).
- Composition with retrieval, gateway, SLOs, observability, rollout.

The FDE does **not** own:

- Multi-agent coordination research.
- Framework internals tuning.
- Agent-level evaluation methodology (uses it; the peer designs it).
- Trajectory-evaluation harness design.
- Model-selection depth for agent workloads.
- New-pattern discovery.

### When this boundary fires

Three cases.

**Case 1: initial agent architecture for a non-trivial workflow.** Pattern 4 (orchestrator-workers) or Pattern 6 (autonomous) is on the table, the tool catalogue is > 5 tools, the workflow has multi-step state, and the pilot timeline is > 6 weeks. Default: bring the peer in during week 2 of the pilot, in parallel with the HITL and permission design.

**Case 2: pattern plateau.** The pilot ships at Pattern 1 or 2 and metrics plateau; the business KPI needs the extra lift from Pattern 4 or 5. The FDE hands the gold set, the current trace samples, and the business goal; the peer scopes the redesign.

**Case 3: framework-level change.** A LangGraph major-version upgrade, a Claude Agent SDK migration from v1 to v2, a transition from in-house orchestrator to a framework. These are framework-depth decisions.

### What the FDE hands to the peer

A specific brief, mirroring mod-105 Chapter 07's shape.

| Field | Example |
|---|---|
| Customer + workflow | Mid-size US healthcare payer; CSR member-inquiry resolution + prior-auth correction (later phase) |
| Deployment model | VPC install; AWS region us-east-1; Bedrock inference |
| Current runtime | Claude Agent SDK, Python 3.12 |
| Pattern selected | Prompt chain (draft phase); planned orchestrator-workers (action phase); see ADR-06 |
| Tool catalogue | 7 tools; see catalogue v1.3; 3 read, 3 write (HITL per call), 1 queue |
| Permission model | OBO via Okta + downscoped tokens; see Chapter 05 brief |
| HITL table | See HITL v1.2; business owner signed off on draft phase |
| Threat model | OWASP Agentic coverage matrix v1.1; red-team findings log |
| Audit contract | Splunk-destined; schema v1.0 |
| Latency SLO | p95 workflow end-to-end ≤ 8 s (draft); ≤ 12 s (action phase including HITL round-trip) |
| Cost budget | ≤ $0.18 per resolved inquiry (tokens + tool calls); human-queue cost excluded |
| Current metrics | task-success 71%, intervention rate 24%, mean tool calls per task 3.8 |
| Target metrics | task-success ≥ 85%, intervention rate ≤ 15%, mean tool calls ≤ 3 |
| Change constraints | AppSec review required before adding tools or sub-agents; customer prefers one pattern migration per quarter |
| Gold set | Agent trajectory gold set G-06-v1.1; 80 labelled trajectories; held-out split 20% |

With that brief the peer can scope. Without it, the peer spends a week asking for it back.

### What the FDE receives back

- An agent architecture (pattern graph + tool catalogue + state shape + interrupt points) with measured gold-set metrics and ablation notes.
- A framework configuration (SDK version, state-store choice, checkpoint policy) with upgrade and rollback runbook.
- Latency and cost measurements under the target configuration.
- An opinion on which further pattern moves are worth another cycle vs. which have plateaued.
- Agent trajectory samples the peer wants the FDE to watch in production.

### Who answers which pager

Mirroring mod-103 Chapter 09 and mod-105 Chapter 07:

- Agent trajectory regression after a pattern or framework change → first page the peer (the FDE's deploys show which).
- Task-success drop on a known gold-set trajectory → page the peer for a pattern-side regression.
- Tool call fails because the IdP returned 401 → page the FDE; it's a permission / NHI issue.
- HITL confirmation rate spike → page the FDE; it's a UX / rollout issue.
- Prompt injection attempt detected → page the FDE (standard threat mitigation); escalate to the safety peer if the attack class is novel.
- Auto-run action taken against a user who then disputes it → page the FDE (the audit and HITL design) first; include the peer if the pattern itself is implicated.

The FDE triages; escalates to the peer with the trajectory that reproduces the failure.

### What the FDE must know — the surface

The FDE is not expected to own the depth of the peer's craft, but is expected to speak the vocabulary. The minimum surface:

**Pattern catalogue.** The six patterns from Chapter 02 with their trade-offs in blast radius, cost, and explainability. Where each has composed well historically.

**Framework shape.** For the chosen framework: its graph model, its interrupt and resume model, its state-store, its retry defaults, its parallel-tool-call shape, its sub-agent / handoff mechanics. The FDE can read the trace; the peer tunes it.

**Agent evaluation.** The standard agent-eval metrics — task success rate, cost per completed task, mean tool calls per task, trajectory match on labelled trajectories, intervention rate. The FDE builds the customer gold set (which trajectories matter) and reads peer-produced eval runs.

**Reasoning vs. tool use.** When the model should reason before calling a tool vs. call-and-see. When extended-thinking / reasoning-model use is worth the token cost.

**Multi-agent patterns.** Supervisor-worker, debate, planner-executor — when each is the right composition. Anti-patterns like "three agents arguing in a loop with no exit condition."

**Long-horizon state.** How session memory, scratchpad, checkpoint store, and vector-store memory compose; when each is the right layer.

The agentic AI engineer owns all of this at depth. The FDE's job is to recognise which lever applies and brief it.

### When to try first vs. when to escalate

Mod-101 Chapter 06's rule applies: default to "you do it" unless depth, scale, compliance, or novelty triggers fire. For agents:

- **Try first** when the pattern is 1 (chain) or 2 (routing) with a small tool catalogue and the customer's workflow is well-shaped.
- **Escalate** when the pattern is 4 (orchestrator-workers) or 6 (autonomous), when the tool catalogue is > 5 tools, when the workflow is multi-step with state, or when business metrics require framework-depth tuning.
- **Try first** on the customer-environment overlay — pattern choice, HITL, permissions, audit, standard threat model.
- **Escalate** on framework-level decisions — graph design, interrupt semantics, state-store choice, agent-eval methodology.

Planned co-ownership from week 2 for non-trivial agents.

## The `agentic-safety-engineer` boundary

### What this peer owns

Frontier-agent safety and red-team methodology, at a level the FDE's standard-threat-model work does not reach:

- **Adversarial evaluation at scale.** Multi-turn attacks; evaluation across thousands of trajectories; discovery of emergent attack classes.
- **Capability elicitation under duress.** Testing what the agent *could* do if adversarially prompted, not just what it does on canonical inputs.
- **Alignment-faking and reward-hacking detection.** Does the agent behave differently when it suspects it is being evaluated? Reference: [Anthropic's alignment-faking research](https://www.anthropic.com/research/alignment-faking).
- **Emergent-behavior discovery.** Behaviours that do not appear in single-turn or short-horizon evaluation but emerge over long sessions or across agent-agent interactions.
- **Red-team harness design.** The test suites, the attacker models, the measurement criteria.
- **Sandbox isolation design for high-risk agents.** When the agent's blast radius must be bounded by infrastructure, not just by policy.
- **Agent governance patterns.** Policies that mix formal verification, runtime checks, and oversight boards.

In short: *everything that assesses whether an agent will remain safe under pressure it was not nominally designed for*.

### What the FDE adds — and what the FDE does not

The FDE does standard threat-model coverage (Chapter 06). The FDE runs a lightweight red-team cycle before go-live and documents it. The FDE builds the audit trail that lets an incident be reconstructed.

The FDE does **not**:

- Design novel attack classes or run large-scale adversarial campaigns.
- Evaluate the model's underlying alignment behaviour.
- Decide whether an agent should be deployed on a given high-stakes workflow — that is the customer's risk committee's decision, informed by the peer when the pilot escalates.
- Publish red-team findings externally.

### When this boundary fires

Fewer cases, but each is high-weight.

**Case 1: high-stakes action scope.** The pilot's workflow has actions with financial, safety-of-life, or legal consequences at significant scale. Standard mitigations may not suffice; the customer's risk committee asks for an adversarial assessment. The FDE briefs the peer.

**Case 2: autonomous agent in production.** Pattern 6 with meaningful action scope, outside shadow mode. Even with standard mitigations in place, the customer may require an adversarial assessment before the auto-run threshold moves above trivial action classes.

**Case 3: novel attack class discovered in the wild.** An injection pattern the FDE's standard mitigations did not anticipate. The FDE escalates to understand the pattern's generality; the peer scopes the mitigation redesign.

**Case 4: multi-agent system with cross-agent action.** When agents are acting on agents, the attack surface compounds; the peer's methodology is needed to characterise it.

**Case 5: regulatory ask.** Some regimes are starting to require frontier-model safety evidence for high-risk AI systems. The customer's compliance lead escalates the question; the peer provides evidence.

### What the FDE hands to the peer

A different brief shape:

| Field | Example |
|---|---|
| Customer + workflow | US retail bank; customer-support reply + case triage |
| Deployment context | Azure OpenAI in EU region; agent runtime is OpenAI Agents SDK |
| Workflow blast radius | Triage updates affect SLAs; ombudsman routing has 72-hour regulatory clock |
| Pattern in production | Router + chain (draft phase); planned orchestrator-workers (triage phase) |
| Current mitigation stack | OWASP Agentic coverage matrix v1.1; detailed in security brief |
| Trigger for escalation | Pre-go-live for triage-update auto-run at low-cost class |
| Attack classes of concern | Customer-submitted content → injection; HITL overwhelm at volume; confused deputy on triage updates |
| Prior red-team work | Lightweight internal pass completed; findings log attached |
| Customer's risk appetite | Per-action-class table from HITL design; business-owner-signed |
| Timeline / stakeholders | Risk committee review date; names |

### What the FDE receives back

- An adversarial assessment report: classes attempted, successes, mitigation recommendations, residual-risk statement.
- A go / no-go opinion against the proposed auto-run boundary.
- Red-team harness artefacts the FDE can re-run as a regression suite.
- A statement for the customer's risk committee the FDE can forward.

### Who answers which pager

- A detected prompt-injection attempt with a mitigated outcome → FDE handles; log and move on.
- A detected attack class the standard mitigations do not cover → escalate to the peer within hours.
- A trajectory the FDE suspects of reward-hacking or alignment-faking → escalate; the peer has the methodology.
- A multi-agent incident with unclear actor attribution → escalate.
- A regulator asks a safety-methodology question → escalate.

### What the FDE must know — the surface

- The OWASP Agentic Top 10 (Chapter 06) at working fluency.
- The high-level shape of adversarial evaluation: multi-turn attacks, sandbox evasion attempts, emergent-behaviour discovery.
- Named terms: alignment faking, reward hacking, specification gaming, Goodhart effects in agent eval.
- The regulatory shape where it exists: [EU AI Act's requirements for high-risk systems](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=OJ:L_202401689); [NIST AI RMF](https://www.nist.gov/itl/ai-risk-management-framework) and its [GenAI Profile](https://airc.nist.gov/AI_RMF_Knowledge_Base/AI_RMF/5-sec-5); emerging [ISO/IEC 42001](https://www.iso.org/standard/81230.html) certifications.

The agentic safety engineer owns the depth. The FDE's job is to recognise the escalation trigger.

## Composing the two boundaries

Both peers can be involved in the same pilot phase, and often are:

- **Case A.** The pilot moves from Pattern 2 to Pattern 4 and introduces auto-run on a low-cost action. `agentic-ai-engineer` designs the orchestrator-workers graph and eval harness; `agentic-safety-engineer` assesses the auto-run threshold adversarially.
- **Case B.** A framework upgrade (e.g., LangGraph v0.x → v1.x) changes interrupt semantics. `agentic-ai-engineer` runs the migration and eval; `agentic-safety-engineer` assesses whether the new semantics change the HITL adversarial surface.
- **Case C.** A novel injection class is discovered. `agentic-safety-engineer` characterises the class; `agentic-ai-engineer` prototypes the framework-level mitigation; FDE lands it in the pilot.

Three peers in some cases (adding `rag-engineer` from mod-105 when the attack vector is retrieved content). One integration design.

## Anti-patterns

**The FDE tries to run the red-team themselves.** Lightweight is fine. Full adversarial assessment requires the peer's methodology and the time budget to run it properly.

**The FDE lets the agentic-AI-engineer peer redesign the HITL or permission model.** Those are deployment-specific; the peer can advise, but the design is signed by the customer's business owner through the FDE. Keep ownership clear.

**Late escalation on the safety side.** The customer's risk committee asks three weeks before go-live for a red-team assessment; the peer needs six weeks. The FDE should have engaged the peer at the pilot's mid-point, not at its endpoint.

**Treating "agent eval" as the peer's and never inspecting it.** The FDE is accountable for the pilot's measured outcomes; the peer's eval is read, questioned, and tied back to the customer's business KPI. Not inspected eval is unused eval.

**Not citing the boundaries in the customer's documentation.** The handoff and escalation story is part of the pilot's narrative. When the customer asks "who is on the hook if something breaks," the FDE names the peers and the pager protocol.

## When to try first vs. when to escalate — the honest table

| Situation | Try first | Escalate |
|---|---|---|
| Pattern 1 (chain) with 2–3 tools | FDE | — |
| Pattern 2 (routing) with 3–5 tools | FDE | — |
| Pattern 3 (parallelisation) | FDE | — |
| Pattern 4 (orchestrator-workers), short tool catalogue | FDE | Peer (agentic-AI) for design review |
| Pattern 5 (evaluator-optimiser) | FDE | — |
| Pattern 4 or 6 with > 5 tools | — | Peer (agentic-AI) from week 2 |
| Autonomous agent (Pattern 6) on action tools | — | Peer (agentic-AI) + Peer (safety) |
| Framework-level upgrade | — | Peer (agentic-AI) |
| Novel injection class in production | — | Peer (safety) |
| Pre-go-live risk-committee review | — | Peer (safety) |
| OWASP Agentic standard coverage | FDE | — |
| Multi-agent production system | — | Both peers |
| Agent-level evaluation methodology | — | Peer (agentic-AI) |

## Where this chapter closes the module

With Chapters 02–06 building the agentic overlay and this chapter writing the peer boundaries, the module closes. The next module, mod-107, picks up the evaluation discipline: the gold-set work from mod-105 Chapter 06 extended to agent trajectories, task success, cost-per-completed-task, intervention rate, with the harness that keeps the pilot measurable as patterns and tools change.

## Summary

- Two peer boundaries live in the agentic layer: `agentic-ai-engineer` (level 30) for framework and pattern depth, `agentic-safety-engineer` (level 40) for frontier red-team methodology.
- The agentic AI engineer owns pattern-catalogue depth, framework internals, agent evaluation methodology, multi-agent coordination, long-horizon state. The FDE owns pattern choice, framework selection, HITL design, permissions, audit, standard threat model.
- The agentic safety engineer owns adversarial evaluation at scale, capability elicitation, alignment-faking detection, red-team harness design. The FDE owns the OWASP Agentic standard coverage and lightweight red-team passes; escalates on high-stakes auto-run, novel attack classes, multi-agent systems, and regulator asks.
- The FDE hands each peer a specific brief and a gold-set/trajectory set; the peer returns an architecture / assessment the FDE lands in the pilot.
- Three cases fire the agentic-AI boundary (initial architecture, plateau, framework change); five fire the safety boundary (high-stakes action scope, autonomous in production, novel attack, multi-agent, regulatory).
- Pager rules partition cleanly between the two peers and the FDE.
- Mod-101 Chapter 06's escalation rule applies throughout: try first on the customer-environment overlay; escalate on depth.
- The module closes here; mod-107 picks up the evaluation discipline for agent trajectories on customer data.
