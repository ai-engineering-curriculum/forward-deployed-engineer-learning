# Chapter 06 — Generalist Stance and When to Escalate

## Why this chapter exists

A Forward Deployed Engineer is a technical generalist across a wide stack — identity integration, data integration, provider selection, prompt engineering, retrieval, agents, evals, guardrails, observability, feature flags, security review, exec communication. The job description reads like five specialist jobs stacked on one person.

The honest version is this: **the FDE is a generalist by default and a peer specialist's client by exception.** The art is deciding when to stay the generalist and keep going, and when to stop and bring a peer in. Decide badly in one direction and you ship a garbage subsystem the customer will inherit. Decide badly in the other direction and you are rate-limited to how fast the peer network can move, and the pilot slips.

Chapter 03 named the peers. This chapter gives you the decision rule.

## The default — you do it

For most of what a pilot asks for, the FDE owns the work end-to-end. The default is not "escalate when in doubt." The default is **do it yourself, within a known pattern, right-sized to the pilot's risk, and move on.**

The default applies to all of these, and you should not pull a peer in unless something specific triggers:

- Shipping a FastAPI service with structured outputs, streaming, retries, logging.
- Wiring SAML or OIDC against Okta or Entra from a pattern you have used before.
- Deploying LiteLLM or Portkey inside a customer VPC with provider failover and cost caps.
- Building a retrieval layer with pgvector or a managed vector DB, with ACL filtering at query time.
- Designing an offline eval with a judge-as-LLM, running it on a bootstrapped gold set.
- Deciding the agent pattern from Anthropic's [Building effective agents](https://www.anthropic.com/research/building-effective-agents) catalogue and shipping it with Claude Agent SDK or OpenAI Agents SDK or LangGraph.
- Instrumenting the pilot with OpenTelemetry using the [GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) and wiring Langfuse or LangSmith.
- Running canary with LaunchDarkly or Statsig, with per-tenant flags and a kill-switch.

If you are hitting one of these in a pattern you have used before and the shape fits the customer's constraint, you *are* the person who ships it. Move.

## The escalation decision rule

Escalate when **any** of the following four conditions is true. Not when you are uncomfortable — when one of these specific triggers fires.

### Trigger 1 — Depth required exceeds your depth, and the pilot *needs* the depth

The question is not "could a specialist do this better than me?" — the answer is always yes, by definition. The question is whether the pilot's success *depends* on specialist-level depth. Ask:

- Is the thing you are about to decide reversible cheaply later? (If yes, decide now, revisit if it bites.)
- Is the blast radius of getting it wrong contained inside the pilot? (If yes, decide now.)
- Would a specialist disagree on *configuration tuning* or on *architecture*? (Tuning: decide now. Architecture: escalate.)

**Examples that pass the trigger.**

- The customer insists on self-hosted inference inside their VPC, with per-request latency budget below 400ms p95 and a specific model-mix. Tuning vLLM to that SLO is `ai-infra-inference-engineer`'s craft; getting it wrong means the pilot cannot meet SLO at go-live and either slips or ships broken. **Escalate.** (See mod-104 and mod-109.)
- The pilot sits in a FedRAMP High environment and the customer's governance board needs a release packet in a specific regulator-facing shape. Composing that packet defensibly is `ai-evaluation-engineer`'s craft. **Escalate.** (See mod-108.)
- The design partner's workflow turns out to require three cooperating agents with planning, message-passing, and shared memory — not two prompts that call each other. Multi-agent architecture is `agentic-systems-architect`'s domain. **Escalate.** (See mod-106.)
- The pilot wants an eval protocol that defends against judge-as-LLM drift and style-prior bias, and the methodology is novel. The *methodology-of-methodology* question is `ai-eval-engineer`'s craft. **Escalate for the methodology; keep ownership of the application.** (See mod-107.)

**Examples that do not pass the trigger.**

- You need to choose between pgvector and Pinecone for a 50k-document retrieval layer. That is a decision in `rag-engineer`'s craft but inside your scope — use the scoring framework, decide, document the ADR, move.
- You need to tune a prompt. That is applied-AI craft from `applied-ai-engineer`. Yours.
- You need to decide the chunking strategy. Yours (with a link to `rag-engineer-learning` if you want to double-check your pattern).

The trigger is not "there is a peer whose curriculum covers this." The trigger is "the pilot needs depth I do not have, on a decision whose cost of being wrong is not contained inside the pilot."

### Trigger 2 — The decision crosses a formal boundary you cannot own

Some things the FDE cannot sign, by design. The FDE is not the independent party. The FDE is not the accountable executive. The FDE is not the regulator-facing packet composer.

- **Formal release sign-off in a regulated vertical.** The FDE produces inputs; `ai-evaluation-engineer` (peer, level 35, Governance) or the customer's own governance board signs. Escalate when the pilot is approaching the go-live gate in a regulated environment.
- **Broad-account commercial or strategic commitments.** Those are AE / Field CTO / engineering manager, not FDE. Escalate every time a customer asks for a commitment that sits outside the signed pilot scope.
- **Legal opinion.** Scope boundaries, data contracts, DPA shape — your customer's legal counsel is authoritative on their side; yours is authoritative on ours. The FDE drafts requirements; legal signs. Escalate every time.
- **Independent red-team or dangerous-capability eval.** `agentic-safety-engineer` (level 40, Governance). The FDE cannot credibly red-team their own agent. Escalate when the pilot's risk surface requires it.

**Rule of thumb.** If what you are about to sign could be contested later and you would not be the right party to defend it, do not sign. Pull in the party who would.

### Trigger 3 — The work is pattern-shaped, not pilot-shaped

Some work that lands on the FDE's desk is not pilot-shaped. It is platform-shaped. It wants to be done once, correctly, and used by every future customer. If you solve it inside the pilot, you have built a bespoke solution that will not generalise and will have to be re-solved later.

- A Partner Engineer problem (enable a system integrator to build against the product). Route to Partner Engineering.
- A platform capability a half-dozen customers would benefit from (a new retrieval primitive, a new audit-log pattern). Package it as a platform-feature request to `senior-agentic-ai-engineer` or the engineering manager; ship a scoped workaround inside the pilot in the interim. See mod-111.
- A reusable integration kit for a whole vertical (the Project 102 shape in this track — HIPAA + BAA + de-id, FedRAMP Moderate, EU AI Act + GDPR + DORA, SOC 2 Type II + CCPA). That is a *project*, not a pilot deliverable; it should be scoped and resourced accordingly.
- A pattern the whole FDE team needs in their knowledge base. Write the pattern writeup; feed it in.

**Rule of thumb.** If you can see two customers who would want the same thing, you are probably building platform. Say so, propose it as platform, ship the pilot workaround.

### Trigger 4 — Your time is the bottleneck for someone else's deliverable

Sometimes the right move is not "do it yourself" or "escalate to a peer." It is "hand the work to whoever is actually best placed to do it right now." Specifically:

- When the customer's engineering team can do the integration work faster because they know their own stack and you are learning it — hand it to them, review it, ship the eval and guardrails you own.
- When the customer's SMEs can bootstrap the gold set faster than you can design one from scratch — hand it to them with labelling instructions and a review loop.
- When `applied-ai-engineer` or `agentic-ai-engineer` on your team has spare cycles and can land a specific slice (a prompt-caching optimisation, a tool-calling pattern) — route it internally.

**Rule of thumb.** The FDE's unique contribution is *the engagement, end-to-end, in the customer's environment*. Any work inside the engagement where someone else is a faster, better fit — move it to them. This is the "generalist" discipline working *outwards*, not just when you are stuck.

## The escalation protocol

When you escalate, do it well. A good escalation is three things:

1. **Specific question.** Not "can you help with the inference serving?" — "we need to serve Llama 3.1 70B in the customer's VPC, 400ms p95, 100 RPS burst, 20 RPS sustained, NVIDIA A100 80GB × 2, and we want to know (a) whether vLLM is the right choice, (b) what batching config to use, (c) whether we can hit the SLO on this hardware."
2. **Context packet.** Pilot scope one-liner, the customer's constraint shape, the decision you would make if the peer did not have time to join, and your current best guess.
3. **Named ask.** Review-only, join-the-engagement, or own-this-slice. Each has a different cost; name the one you need.

The peer then has an input they can act on in under an hour. They decide whether to engage at the depth you asked for. You get an answer that unblocks the pilot. The peer is not resentful because you did not dump an open-ended problem.

The anti-pattern: escalating by Slack-DM with "when you have a minute can we talk." That is an unbounded ask on the peer's time and it will sit. The specific-question + context-packet form moves.

## What escalation does *not* mean

Three things it is tempting to do that are not "escalation" — they are different failure modes:

- **Giving up.** Handing the problem off and walking away. If you own the engagement, you own the escalation too. The peer joins you; they do not take the engagement from you.
- **Blame-shifting.** "That's an inference-engineering problem, not mine" when the pilot's inference is on fire. If the pilot is on fire, you are on the bridge. You bring in the specialist, you do not duck.
- **Inviting a tourist.** Pulling a peer in for a problem the pilot does not actually need their depth on, because it feels safer. That is tax on the peer and on the team. The four triggers are specific for a reason; if none of them fires, keep going.

## A worked example: three decisions on one pilot

A healthcare pilot: a document-retrieval + summarisation feature for case workers, inside a HIPAA-covered environment, VPC install, on Azure.

**Decision 1 — Which vector DB?** Candidates: pgvector on the customer's existing Postgres, or Azure AI Search, or a BYOL deploy of Qdrant inside VPC. Trigger 1 does not fire (reversible, blast radius is inside the pilot, tuning not architecture). Trigger 2 does not fire (not a signing boundary). Trigger 3 does not fire (not pattern-shaped). Trigger 4 does not fire (nobody else is a better fit). **Decision: FDE scores the candidates against the pilot's compliance, cost, and dev-velocity constraints, picks one, writes an ADR, moves.**

**Decision 2 — The eval protocol for groundedness with PHI.** This is novel methodology — the standard RAGAS metrics do not cover the customer's exact groundedness requirement, and PHI constraints prevent sharing data with the standard judge pipelines. Trigger 1 fires (methodology depth, pilot needs the depth, blast radius extends to the governance board sign-off). **Decision: escalate to `ai-eval-engineer` for methodology review; keep ownership of implementation; also pre-wire `ai-evaluation-engineer` on the governance packet shape (Trigger 2 upcoming at go-live).**

**Decision 3 — Serving an open-weights model inside the customer's VPC at the required SLO.** Candidates: Azure AI Foundry managed vs. BYOL vLLM on the customer's AKS. Trigger 1 fires (vLLM tuning / batching / quantisation depth; pilot's latency SLO depends on getting this right; the choice is architecture, not tuning). **Decision: escalate to `ai-infra-inference-engineer` for the serving decision and the tuning; FDE owns the integration against the gateway and the per-tenant routing; FDE still owns the engagement.**

Three decisions, three different outcomes. One pilot. The FDE is still the engagement owner in all three cases.

## Summary

- The default is *you do it* — most of what a pilot asks for is within the FDE's generalist range and shipped end-to-end by the FDE.
- Escalate when one of four triggers fires:
  - **Depth** — the pilot needs specialist depth and the cost of being wrong exceeds the pilot.
  - **Formal boundary** — the decision crosses a signing authority you do not hold (regulated release sign-off, legal opinion, independent red-team, broad-account commitment).
  - **Pattern-shaped work** — the problem wants to be solved once for many customers (platform), not bespoke for this one.
  - **Better fit elsewhere** — someone on either side of the engagement (customer's team, your peer specialist) is a faster, better fit.
- When you escalate, use the protocol — specific question, context packet, named ask. The peer engages; they do not take the engagement.
- The FDE stays the engagement owner across every escalation. Giving up is not escalation.
- Over an engagement you will escalate several times. The *shape* of your escalations is the signature of good judgment under the generalist stance.
