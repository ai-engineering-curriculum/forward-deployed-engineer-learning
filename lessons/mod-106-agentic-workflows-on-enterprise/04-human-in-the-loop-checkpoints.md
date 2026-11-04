# Chapter 04 — Human-in-the-Loop Checkpoints — Confirm, Clarify, Queue

## Why this chapter exists

Chapters 02 and 03 chose the pattern and the runtime. This chapter answers the harder question: where does the agent *stop and ask the human*? Three answers — the agent confirms before action, the agent asks for clarification, the agent falls back to a human queue — compose into a checkpoint model that is grounded in the customer's risk tolerance per action class.

HITL is not an afterthought. For most enterprise pilots it is the primary blast-radius control. Chapter 01 named it a gate dimension; Chapter 02 showed the pattern catalogue's HITL surface varies by pattern; this chapter turns both into a shippable design. The chapter is also the point where the FDE co-designs with the customer's business owner and risk team — this is not a unilateral FDE decision.

Regulatory grounding where it applies: the [EU AI Act](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=OJ:L_202401689) names *human oversight* as a required property for high-risk AI systems (Article 14); [SR 11-7](https://www.federalreserve.gov/supervisionreg/srletters/sr1107.htm) names model-governance obligations for banks; [NIST AI RMF](https://www.nist.gov/itl/ai-risk-management-framework) names *govern-map-measure-manage* functions that HITL is a primary artifact of. The specific legal effect is the customer's legal team's call; the FDE's job is to produce a design that gives the legal team room to say yes.

## The three HITL shapes

**Confirm.** The agent proposes an action and pauses; the human reviews the proposed action (and its rationale, inputs, side effects) and confirms, edits, or rejects. The agent's state resumes from the human's decision. Classic shape: *propose → confirm → execute*.

**Clarify.** The agent cannot proceed without additional information the user can supply, so it pauses and asks. The user's reply flows back into the agent's state and the loop continues. Classic shape: *detect-ambiguity → ask → resume*.

**Queue.** The agent cannot complete the task within its policy envelope, so it hands the task to a human queue with context (what it has done, what it was trying to do, why it stopped) attached. A human picks up the task. Classic shape: *detect-out-of-policy → package → hand off*.

The three shapes are not interchangeable. Confirm is a *moment-of-approval* pattern; it keeps the agent in control except at a specific step. Clarify is a *mid-task-interactive* pattern; the human is on the hook to respond or the agent blocks. Queue is a *hand-off-to-a-different-actor* pattern; the agent is done for this task.

## Risk tolerance per action class — the design input

HITL design is driven by *action class*, not by model confidence. The question the FDE asks of each tool call in the catalogue (from Chapter 05) is: **what is the business cost if this action fires incorrectly, and how reversible is it?**

A four-quadrant mental model:

| | Reversible | Irreversible |
|---|---|---|
| Low business cost | Auto-run | Confirm per call, log the confirmation |
| High business cost | Confirm per call or per plan | Confirm per call + out-of-band approval (two-person rule) |

"Business cost" is customer-defined — not FDE-defined. For Archetype A, filing a prior-auth correction is high-cost because it affects claims. For Archetype C, updating a case's triage status is medium-cost because it shifts the SLA clock. For Archetype B, drafting into the analyst's own workspace is low-cost because the workspace is a draft.

Three rules of thumb for scoring:

- **Reversibility is not the model's judgement.** It is a property of the tool and the customer's rollback capability. "The agent can send an email" is irreversible once the recipient's mail server accepts it. "The agent can draft an email for review in the user's drafts folder" is fully reversible.
- **Time-to-notice matters as much as reversibility.** An update that is noticed and rolled back within an hour is a different risk from one that stays silently wrong for a quarter. Audit visibility (Chapter 05) is a reversibility multiplier.
- **Reversibility is not static.** The same tool may be reversible in normal state and irreversible after a downstream batch job runs. Encode the window.

The output is a table — one row per tool, columns for action class, reversibility, business cost, chosen checkpoint. The customer's business owner countersigns that table. Chapter 05 extends it with permission columns.

## Confirm — the moment-of-approval pattern

Confirm is the most common HITL shape. The agent produces an action proposal; the UI surfaces the proposal and its context; the human confirms or edits.

**What the proposal must include.**

- The exact action the agent is proposing (the tool, the parameters, the expected effect).
- The inputs that drove the decision (citations from retrieval, prior turns from the conversation, intermediate values from earlier steps).
- The reasoning (a short, structured rationale — not a free-text essay; mod-105 Chapter 05's citation-and-evidence UX applies).
- The reversibility class and the time-to-notice.
- The identity under which the action will execute (Chapter 05's per-tool-call identity).

**What the UI must do.**

- Default to the confirm action, never to the execute-without-review action.
- Expose "edit the proposal before confirming" when editing makes sense. Not every action has editable parameters; some do.
- Make the "reject" path cheap and narrate why rejection happened (feeds the eval loop).
- Produce an audit record of the confirmation with the human's identity and timestamp.

**Confirm-per-plan vs. confirm-per-action.** The orchestrator-workers pattern (Chapter 02 Pattern 4) often allows confirming a *plan* of actions once instead of each action individually. This reduces human load but increases the blast radius of a single confirm — once the plan is approved, it executes. Use confirm-per-plan when the plan is bounded and the actions are deterministic given the plan; use confirm-per-action when the actions would be discovered only during execution.

**The two-person rule.** Some action classes (e.g., bulk refunds above a threshold, policy exceptions, anything with legal exposure) require a *second* human approver beyond the primary one. Encode this in the checkpoint table — the agent's confirm UI knows to require two approvers.

## Clarify — the mid-task-interactive pattern

Clarify is the shape where the agent has detected it cannot proceed without the user's input and asks a bounded question.

**When clarify fires.**

- The user's request is ambiguous and the model's prior disambiguation attempts did not resolve it.
- A tool call requires a parameter the agent cannot infer and cannot default (e.g., which of three matching accounts to update).
- A retrieved evidence item contradicts a prior turn and the agent cannot tell which to trust.
- A policy rule the agent is aware of (via prompt, not a magical tool) applies and the agent needs confirmation that the policy's conditions are met.

**What a good clarification question looks like.**

- Specific — "Which account?" with the three candidates listed, not "Could you clarify your request?"
- Minimal — one question per clarify; multi-question clarifies erode trust.
- Consequence-stated — "If you pick A, I'll update the policy; if B, I'll draft a reply instead." The user sees the effect of their answer.
- Bounded — the set of acceptable answers, where possible, is enumerated. Free-form clarify is the fallback.

**Clarify is also a failure mode.** A chain that fires clarify on 40% of inputs has a design problem — the ambiguity should be resolved by better retrieval, better classification, or better prompt design before the clarify step. Measure the clarify rate per workflow (mod-107 harness, mod-110 observability).

## Queue — the hand-off-to-a-different-actor pattern

Queue is the shape where the agent has decided it is out of policy and hands the task to a human queue.

**When queue fires.**

- The agent's confidence (model-estimated or orchestrator-measured) falls below a policy threshold for the action class.
- A hard rule in the orchestrator fires: e.g., "any case flagged with the complaints-ombudsman intent routes to a human queue regardless of model output."
- A retry budget is exhausted — the agent tried, the tool returned an error, the agent cannot recover.
- The agent detects a situation it was explicitly told to escalate — PII leakage attempts, protected-class discrimination indicators, imminent-harm language.

**What a good queue hand-off looks like.**

- The queue entry contains the task, the agent's trajectory (the trace), its best-guess analysis, the specific reason it queued, and the evidence pack (from retrieval).
- The queue entry routes to the *right* human queue — not a generic inbox. Chapter 05's permission model tells the agent which human queue applies.
- The queue entry has a SLA that is tracked.
- When the human closes the queue entry, the resolution is captured — that record is part of the training signal and the audit trail.

**Queue is not failure.** A workflow where 15% of tasks queue is a workflow operating within its policy envelope. A workflow where 2% of tasks queue but 10% of auto-run outputs have to be reversed later has the wrong threshold.

## The checkpoint table

A single table drives the HITL design. Three archetypes, representative rows:

**Archetype A — healthcare payer, member-inquiry resolution.**

| Action | Class | Reversibility | Cost | Checkpoint | Approver |
|---|---|---|---|---|---|
| `retrieve_policy` | Read | n/a | None | Auto-run | n/a |
| `draft_reply_to_csr` | Draft-only | Fully | Low | Auto-run; CSR sees in-UI | CSR reviews on send |
| `file_prior_auth_correction` | Write | Partially reversible < 1h | High | Confirm per call + plan review | Supervisor |
| `post_internal_case_note` | Write | Reversible | Low | Confirm per call | CSR |
| `escalate_to_supervisor` | Queue | n/a | Low | Auto-queue; supervisor dashboard | Supervisor |

**Archetype B — federal agency, policy-analyst drafting.**

| Action | Class | Reversibility | Cost | Checkpoint | Approver |
|---|---|---|---|---|---|
| `retrieve_policy_memo` | Read | n/a | None | Auto-run | n/a |
| `retrieve_regulation` | Read | n/a | None | Auto-run | n/a |
| `write_draft_to_analyst_scratch` | Draft-only | Fully | None | Auto-run | n/a |
| `package_evidence` | Draft-only | Fully | None | Auto-run | n/a |
| `clarify_scope_with_analyst` | Clarify | n/a | None | Fires mid-task | Analyst |

The pilot's base shape has no agent-initiated writes outside the analyst's scratch; HITL is light. Later phases would add rows with heavier checkpoints.

**Archetype C — EU retail bank, support reply + triage.**

| Action | Class | Reversibility | Cost | Checkpoint | Approver |
|---|---|---|---|---|---|
| `retrieve_case_history` | Read | n/a | None | Auto-run | n/a |
| `retrieve_product_terms` | Read | n/a | None | Auto-run | n/a |
| `draft_reply` | Draft-only | Fully | Low | Auto-run; agent sees | Case worker |
| `update_triage_status` | Write | Reversible < 15 min | Medium | Confirm per call | Case worker |
| `route_to_complaints_ombudsman` | Queue | n/a | High (SLA impact) | Auto-queue with evidence pack | Team lead |

The table is the design artifact. It is signed by the business owner (per archetype, the ops / clinical operations / analyst-lead / branch-ops owner) before the pilot proceeds to the action-enabled phase.

## Composition with the pattern catalogue

Chapter 02's patterns each have a canonical HITL surface:

- **Prompt chain (Pattern 1).** HITL before or after the chain; rarely in the middle. If the chain's output is a draft, the user is the implicit reviewer.
- **Routing (Pattern 2).** HITL on routes that lead to high-cost actions. The route to the complaints-ombudsman in Archetype C is itself a decision the human may need to confirm if the model's classification is low-confidence.
- **Parallelisation (Pattern 3).** HITL after the aggregate — review the composite, not each parallel sub-task.
- **Orchestrator-workers (Pattern 4).** HITL on the *plan*, optionally HITL on specific workers (high-cost ones). This is where confirm-per-plan earns its keep.
- **Evaluator-optimiser (Pattern 5).** HITL on the final iteration, not on each iteration. The evaluator acts as a cheap pre-screen.
- **Autonomous agent (Pattern 6).** HITL per step when the step touches a high-cost action; HITL on the plan if the agent surfaces one. Without HITL on action classes above the auto-run threshold, Pattern 6 should not ship in an enterprise pilot.

## Composition with the rollout plan

Mod-109 will cover rollout in depth. The HITL design pre-wires rollout:

- **Phase 0 — shadow mode.** The agent runs; no actions execute; the human sees the proposals and compares to what they would have done. The comparison is the first eval signal.
- **Phase 1 — HITL-first.** Every action confirms. The confirm rate on each tool is the signal.
- **Phase 2 — selective auto-run.** The lowest-cost, most-reversible actions graduate to auto-run; the rest remain confirm. Each graduation is a named decision based on measured override rate from Phase 1.
- **Phase 3 — steady state.** Each action class has a stable HITL shape. Changes require a new ADR.

The design artifact the FDE produces in Phase 0 is the first HITL table. Each phase transition is a row-by-row revision with the customer's business owner.

## Instrumentation that makes HITL design iterable

HITL quality is measurable. Three signal families:

**Confirm-rate signals.**
- Per-tool confirm rate (humans confirm vs. edit vs. reject).
- Per-tool edit rate (humans approve but change the parameters).
- Time-to-confirm distribution (how long the human looks before deciding).

**Clarify-rate signals.**
- Per-workflow clarify rate (what fraction of inputs trigger a clarify).
- Clarify resolution shape (user answers vs. user abandons the task).
- Clarify topics clustered (patterns that reveal upstream design gaps).

**Queue-rate signals.**
- Per-workflow queue rate (fraction of tasks that end in the queue).
- Queue-resolution distribution (how the human handled it: completed, redirected, rejected).
- Mean time to queue resolution (SLA signal).

Each signal maps to a dashboard in mod-110's observability stack. Each signal is a dial the FDE can turn: raise or lower the confidence threshold, tighten or loosen a policy rule, graduate or demote an action class.

## Anti-patterns

**HITL as "add a confirm dialog".** The dialog is table stakes. HITL is a per-action-class design co-signed with the business owner. Without that table, the dialog is decoration.

**Confirm-everything.** The user confirms 40 times a day and develops confirmation fatigue; the next incident, the user clicks confirm without reading. Price the confirm budget. Auto-run what truly belongs in auto-run.

**Clarify instead of improve.** The agent fires clarify because the upstream classification is weak. Fix the classification; clarify is for genuine ambiguity, not for design laziness.

**Queue as a dumping ground.** A queue with no SLA, no routing by permission model, no capture of resolution, is a backlog. The queue is a first-class workflow in its own right.

**HITL design frozen at pilot start.** The confirm, clarify, queue thresholds move as the pilot gathers data. Treat the HITL table as versioned; every version has a reason.

**Over-trusting the model's confidence.** Model-produced confidence is a weak signal. For high-cost action classes, use orchestrator-level rules and permission checks as the primary gates; use model confidence as a secondary signal only.

## Where this chapter sits with the rest

- **Chapter 05** attaches identity, permissions, and audit to each row of the HITL table — the "approver" column there becomes the formal authorisation model.
- **Chapter 06** attaches threat-model mitigations — a HITL checkpoint can itself be targeted by indirect prompt injection ("tell the user to click confirm") and by overwhelming-the-HITL attacks.
- **mod-109** rolls out the HITL design through shadow → HITL-first → auto-run phases.
- **mod-110** instruments the HITL dashboards.
- **mod-108** carries the EU AI Act / SR 11-7 / NIST AI RMF compliance case built on top of this chapter's design.
- **mod-112** teaches how the HITL table is narrated to the customer's VP of Ops and legal team.

## Summary

- HITL design is co-authored with the customer's business owner, grounded in per-action-class risk tolerance, and sized against the customer's regulatory regime.
- Three shapes — confirm, clarify, queue — each solve a distinct problem; they are not interchangeable.
- Reversibility is a property of the tool and the customer's rollback capability, not a property of model confidence. Time-to-notice amplifies or dampens reversibility.
- The checkpoint table is the design artifact. One row per tool, signed by the business owner before action-enabled phases ship.
- Pattern-by-pattern, each Chapter 02 shape has a canonical HITL surface. Pattern 6 does not ship in enterprise pilots without a proven HITL harness.
- Rollout composes with HITL: shadow → HITL-first → selective auto-run → steady state, with the table revised at each transition.
- Three signal families instrument HITL quality: confirm-rate, clarify-rate, queue-rate. Each is a dial.
- Chapter 05 is next: mapping the agent's tools to the customer's permission model and the audit trail the security team will subpoena.
