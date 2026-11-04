# Chapter 03 — Position on the Level Ladder

## Why this chapter exists

The Forward Deployed AI Engineer sits at **level 35** — a hands-on specialist IC in the AI Engineering family. The level number by itself tells you nothing. What matters is who is next to you on the ladder, what they own, what you own, and the sharp lines where one role's deliverable becomes another role's.

This chapter walks through the role boundary against every neighbour the FDE actually touches. The pattern is always the same:

- **What that role owns** (their primary deliverable, in one sentence).
- **What the FDE adds on top** (the delta — the thing this role does that the neighbour does not).
- **Where the deferral contract is** (the thing the FDE escalates to that neighbour, by name).

Keep this chapter handy. Every other module in the track references the ladder; every project in the track has at least one "defer to peer" moment.

## The ladder in one view

```
Level 70   Executive              chief-ai-officer
Level 50   Program                senior-ai-governance-architect
Level 48   Principal IC           principal-ml-engineer · agentic-systems-architect
Level 40   Build leadership /     senior-agentic-ai-engineer · ai-engineering-manager
           Management             agentic-safety-engineer (Governance)
Level 35   Specialist IC         ┌── forward-deployed-engineer  (YOU)
           (peer band)           ├── ai-infra-inference-engineer (Infra)
                                 ├── ai-infra-security / security (Infra)
                                 ├── ai-evaluation-engineer (Governance)
                                 ├── training-pipeline-engineer (ML)
                                 ├── voice-agent-engineer (AI Eng)
                                 └── ai-enablement-lead (Workforce)
Level 30   Craft IC               applied-ai-engineer · agentic-ai-engineer · ai-eval-engineer
Level 25   Foundational IC        llm-application-developer · rag-engineer · ai-risk-engineer
Level 20   ML fundamentals        ml-engineer
Level 10   Engineering basics     ai-infra-junior-engineer
```

Three things to notice in the picture:

1. The FDE is **in the peer band**, not above the craft IC band. You should expect to ship code alongside level-30 engineers — you are a specialist, not a tech lead over them.
2. The peer band is **cross-family**. Your level-35 peers include an infra engineer, a security engineer, a governance engineer, a training-pipeline engineer. That is the FDE's working surface.
3. The deferral contracts go **both down and sideways** more than they go up. Most of your "this is not my job" lines point at a craft IC (level 30) or a peer specialist (level 35), not up at the manager (level 40).

## Boundary 1 — vs. `applied-ai-engineer` (level 30, direct predecessor)

**What they own.** In-house applied-AI feature engineering end-to-end: production prompting with caching / streaming / versioning, feature-side evaluation loops, guardrails on the surface, provider gateway routing, per-feature cost / latency SLOs, feature flags + canary, observability + feedback + incident response. All of it *inside their own company*.

**What the FDE adds on top.** The customer-environment overlay. The `applied-ai-engineer` ships a feature that serves their employer's product. The FDE ships that same shape of feature *inside the customer's SSO, VPC, data surface, SaaS stack, security posture, on-call rotation, change-management process, and regulatory boundary*. Every piece of applied-AI craft the level-30 engineer has gets a "…and now do it inside a customer environment you do not control" overlay.

**Deferral contract.** For questions of in-house applied-AI craft depth — "what is the right prompt caching strategy?", "how do I design per-feature cost SLOs?", "what is the right structured-output schema pattern?" — the FDE reads the `applied-ai-engineer-learning` curriculum, uses the pattern, and does **not** try to invent that craft here. This track references into `applied-ai-engineer-learning` and adds the overlay. See the track's [`PREREQUISITES.md`](../../PREREQUISITES.md) — practical fluency in the applied-AI engineer craft is an assumed entry skill.

## Boundary 2 — vs. `agentic-ai-engineer` (level 30)

**What they own.** Deep agent-framework internals — LangGraph state machines, OpenAI Agents SDK internals, Claude Agent SDK, Model Context Protocol transport, tool-calling patterns, agent memory shapes, planning patterns, agent debugging.

**What the FDE adds on top.** Running those agents against the customer's permission model, tool surface, audit log, and governance. The FDE decides which agent shape fits the customer's workflow (from Anthropic's [Building effective agents](https://www.anthropic.com/research/building-effective-agents) pattern catalogue), binds the agent to the customer's identity + authorisation + audit, and hardens it against the OWASP Agentic Top 10 — but the FDE does **not** re-derive agent-framework theory.

**Deferral contract.** For questions of agent-framework depth — "what is the right memory pattern for a multi-turn negotiation agent?", "how does LangGraph handle resumable state?", "what is the MCP transport layer doing?" — the FDE reads `agentic-ai-engineer-learning` and the primary framework docs. For questions of frontier-agent red-team methodology, the escalation goes to `agentic-safety-engineer` (level 40, Governance). See mod-106 in this track for how those boundaries show up in practice.

## Boundary 3 — vs. `ai-eval-engineer` (level 30)

**What they own.** Evaluation methodology at the application layer — traces, judges, RAG eval, eval-gated CI/CD, online eval, cost/latency/quality trade-offs. They own *how to build good evals*.

**What the FDE adds on top.** Running that methodology *on customer data*, with SME-paired gold sets, inside the customer's compliance posture (so eval infrastructure sits in VPC or a zero-retention sandbox), wired to the customer's business KPI (so the eval conclusion translates to a decision the customer's governance board can sign). The FDE does not invent the methodology; the FDE applies it under customer constraints.

**Deferral contract.** For methodology questions — "what sampling strategy gives me a defensible gold set?", "how do I design an LLM-as-judge that does not pay attention to style?" — the FDE reads `ai-eval-engineer-learning` and brings in the peer. For release-assurance methodology (how to compose an audit trail the regulator reads, how to run the formal gate), the escalation goes sideways to `ai-evaluation-engineer` (peer, level 35, Governance). See mod-107.

## Boundary 4 — vs. `rag-engineer` (level 25) and `llm-application-developer` (level 25)

**What they own.** Retrieval-engine depth (chunking strategies, embedding-model trade-offs, rerankers, multi-index fusion, retrieval-level evaluation) and LLM application fundamentals (SDKs, structured outputs, streaming, tool-calling basics).

**What the FDE adds on top.** Choosing a retrieval engine that fits the customer's compliance posture and deployment model; propagating the customer's permission model into the retriever (document ACLs, row-level security, group / role / attribute filters); shipping PII-safe retrieval end-to-end; designing the citation / evidence / freshness UX the customer's SME will trust.

**Deferral contract.** For retrieval-engine depth, the FDE reads `rag-engineer-learning`. For LLM-application fundamentals, `llm-application-developer-learning`. The FDE consumes those crafts and adds the customer-overlay. See mod-105.

## Boundary 5 — vs. `ai-infra-inference-engineer` (peer, level 35)

**What they own.** Inference serving depth — vLLM / TGI / SGLang internals, batching, quantisation, KV-cache, autoscaling, fleet-level SLOs, GPU operations.

**What the FDE adds on top.** When a customer insists the model runs inside their VPC (or air-gapped on-prem), the FDE is the person who gets a serving stack *up*. Not who tunes it, not who scales it, not who runs it in production at fleet scale — who gets it *up* with an appropriate configuration for the pilot's traffic shape.

**Deferral contract.** For a vLLM tuning issue, batching strategy, quantisation choice, KV-cache sizing, or an autoscaling decision at scale, the FDE pulls in `ai-infra-inference-engineer`. The FDE's job is to characterise the customer's constraint (data residency, latency budget, model-mix needs, cost ceiling) and bring that to the inference engineer for the design. The FDE does not try to out-tune vLLM against a specialist. See mod-104 and mod-109.

## Boundary 6 — vs. `ai-evaluation-engineer` (peer, level 35, Governance family)

**What they own.** Release-assurance methodology — audit trails, regulator-facing packet composition, formal release gates, defensible evidence, independent governance.

**What the FDE adds on top.** The FDE produces the inputs: eval results, failure-mode inventories, deployment runbooks, incident logs, model/system/dataset cards. The `ai-evaluation-engineer` composes those inputs into the packet a regulator or governance board signs.

**Deferral contract.** For the formal release packet, for the methodology-of-methodology question ("is the eval I ran good enough to carry a FedRAMP High authorisation?"), for independent sign-off, the FDE pulls in `ai-evaluation-engineer`. The FDE does **not** sign the release packet themselves, especially in regulated verticals — the independence is part of the point. See mod-107 and mod-108.

## Boundary 7 — vs. `ai-infra-security` / `security` (peer, level 35, Infra family)

**What they own.** Deep ML/AI security at platform scale — model-weight protection, supply-chain attestation, red-team at platform level, security architecture across tenants.

**What the FDE adds on top.** Shipping a specific pilot through a specific customer's security review (SOC 2 Type II evidence collection, SIG / CAIQ responses, pen-test report sharing, data-flow diagrams, sub-processor disclosures), in a specific vertical's regulatory frame. The FDE threat-models the pilot against OWASP LLM Top 10 + Agentic Top 10 + MITRE ATLAS and ships mitigations.

**Deferral contract.** For platform-scale security questions (how to design a multi-tenant key rotation system, how to red-team a frontier model), the FDE escalates to `ai-infra-security` / `security`. The FDE's remit is pilot-shaped, not platform-shaped. See mod-108.

## Boundary 8 — vs. `voice-agent-engineer` (peer, level 35, AI Engineering)

**What they own.** Voice channel specifics — real-time ASR/TTS, latency budgeting for speech, barge-in, endpointing, voice-channel compliance (telephony regulations).

**What the FDE adds on top.** If a design-partner engagement targets a voice channel, the FDE picks up the engagement end-to-end but defers voice-channel specifics to the peer. The FDE still owns discovery, scoping, integration, deployment, security review.

**Deferral contract.** For voice-channel engineering decisions, the FDE pulls in `voice-agent-engineer` on the engagement.

## Boundary 9 — vs. `ai-enablement-lead` (peer, level 35, Workforce family)

**What they own.** Customer-side change management at organisational scale — adoption measurement, training curricula for operator populations, operating-model redesign, change-communications programs.

**What the FDE adds on top.** The FDE ships the technical hand-off packet, SME training materials for the pilot, admin dashboards, and playbooks for common failure modes — i.e., the enablement work *for the pilot user population*. The organisation-wide rollout — "we are going to change how 2,000 contact-center agents work" — is the `ai-enablement-lead`'s job.

**Deferral contract.** When a pilot succeeds and the customer wants to scale it across an organisation, the FDE hands off to `ai-enablement-lead` (either on your team, or on the customer's side). See mod-109 and mod-112.

## Boundary 10 — vs. `senior-agentic-ai-engineer` and `ai-engineering-manager` (level 40)

**What they own.** Build leadership and shared-infra authorship (`senior-agentic-ai-engineer`); engineering management scope — hiring, performance, calibration, planning, program health (`ai-engineering-manager`).

**What the FDE adds on top.** Nothing — this is the upward boundary. The FDE is a specialist IC. The FDE does not hire, does not calibrate, does not plan the roadmap. The FDE *influences* the roadmap by feeding evidence back through the field → product loop.

**Deferral contract.** For "we need a platform capability to unblock three pilots" conversations, the FDE packages the request and brings it to `senior-agentic-ai-engineer` or the engineering manager. For "we need to hire a second FDE on this account," the engineering manager runs the process. See mod-111 and mod-112.

## Boundary 11 — vs. `agentic-systems-architect` (level 48)

**What they own.** Multi-agent architecture — large-scale agent design, agent-protocol standardisation, cross-agent coordination patterns.

**What the FDE adds on top.** Nothing — this is also upward. If a customer engagement surfaces a genuine multi-agent redesign (not "we need two prompts that call each other"), the FDE escalates. See mod-106.

**Deferral contract.** Multi-agent redesigns go to `agentic-systems-architect`. The FDE's agent work tops out at the pattern catalogue in Anthropic's [Building effective agents](https://www.anthropic.com/research/building-effective-agents), productionised at pilot scale.

## The deferral contract pattern

Across all eleven boundaries you can see the same shape:

- **The FDE is a generalist across the stack**, from identity integration to eval design to agent hardening to observability.
- **The FDE's depth is in combining those surfaces inside a customer environment**, not in the deepest point of any one surface.
- **Every surface has a named peer who goes deeper.** When you hit a question the pilot genuinely needs the depth for, you pull the peer in by name — not an anonymous "subject matter expert."
- **The escalation does not transfer the engagement.** You stay the engagement owner. The peer joins; they do not take over.

Chapter 06 of this module turns this pattern into a decision rule: *when do I keep going as the generalist, and when do I escalate?* This chapter is the map; chapter 06 is the compass.

## Summary

- The Forward Deployed AI Engineer is level 35, hands-on specialist IC, AI Engineering family. Peers at level 35 are cross-family: inference, security, governance eval, training pipeline, voice, enablement.
- The FDE extends the level-30 craft of `applied-ai-engineer`, `agentic-ai-engineer`, `ai-eval-engineer`, `rag-engineer`, `llm-application-developer` with a *customer-environment overlay*.
- The FDE defers sideways on inference depth (`ai-infra-inference-engineer`), platform-scale security (`ai-infra-security`), release-assurance methodology (`ai-evaluation-engineer`), voice-channel specifics (`voice-agent-engineer`), and organisational change management (`ai-enablement-lead`).
- The FDE defers upward to `senior-agentic-ai-engineer` and `ai-engineering-manager` (level 40) for shared-infra and management, and to `agentic-systems-architect` (level 48) for multi-agent architecture.
- Every deferral names a specific role. The FDE is a generalist in breadth, a specialist in *customer-embedded delivery*, and is deliberately shallower than each peer at that peer's depth.
