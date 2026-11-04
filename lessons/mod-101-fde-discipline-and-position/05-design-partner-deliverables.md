# Chapter 05 — Design-Partner Engagement Deliverables

## Why this chapter exists

The Forward Deployed Engineer is on the hook for a defined set of artifacts at defined points in an engagement. If you can recite the sequence from memory and know what good looks like for each one, you can run a design-partner engagement without a manager telling you what to do next. If you cannot, you will drift — the pilot will produce code but not a hand-off, or evals but not a scope, or a demo but no runbook.

This chapter lays out the seven deliverables the FDE owns across a design-partner engagement:

1. **Discovery brief** — what the customer actually does and where AI could change it.
2. **Pilot scope document** — the contract for the pilot.
3. **Eval protocol** — how you will decide the pilot worked.
4. **Integration design** — how the pilot fits into the customer's stack.
5. **Deployment runbook** — how the pilot ships and how it rolls back.
6. **Hand-off packet** — what the customer (and your CSE/CSM) inherit.
7. **Field → product review** — what the engagement taught the product team.

For each deliverable you get: when it lands, who signs it, what is in it, what "done" looks like, and which downstream module teaches the craft in depth. This chapter is the inventory; the rest of the track is the craft.

## Timeline at a glance

```
Week 0       Pre-pilot — commercials + design-partner contract (AE + Legal + FDE)
Week 1–2     Discovery brief
Week 2–3     Pilot scope document + Eval protocol
Week 2–6     Integration design (parallel to build-out)
Week 4–10    Deployment runbook (iterative, finalised pre go-live)
Week 10–12   Hand-off packet
Week 12+     Field → product review (recurring, not one-shot)
```

The weeks are indicative — a 12-week pilot is typical, but engagements range from 6 weeks (lightweight pilot on a known workflow) to 26 weeks (regulated vertical with FedRAMP / HIPAA review). The *sequence* is stable.

## Deliverable 1 — Discovery brief

**When.** End of week 1 or 2.

**Who signs it.** You and the customer-side sponsor (a product owner, operations lead, or line-of-business VP, depending on the shape of the workflow). Your AE and CSM are copied.

**Why it exists.** To stop the engagement before you spend a quarter's worth of engineering time solving the wrong problem. The discovery brief forces you to state in writing what the customer actually does today, and where AI could change it. If you cannot write it, you have not learned enough; the fix is more discovery, not more code.

**What is in it.**

- **Workflow as-is.** The current process in sentences or a swim-lane diagram. Who touches it. How long each step takes. What the operators actually do (not what the org chart says they do).
- **Pains and metrics.** What goes wrong today. Quantified where possible — "triage queue sits at 48 hours average, SLA is 24, we miss 30% of the week."
- **Candidate AI interventions.** Two or three — not ten. Each one named clearly, with the hypothesis ("if the model drafts the response, SMEs review and send in 3× less time").
- **Where AI does NOT help.** The parts you have deliberately ruled out. Usually because deterministic code is cheaper; because the failure mode is intolerable at any reasonable error rate; because the data is not there.
- **Open questions.** The things you need to answer before you can scope the pilot. Named, with owners and dates.
- **Recommendation.** Not a conclusion — a recommended *next step*: proceed to pilot scope on one of the candidate interventions, or spend another week on discovery, or decline.

**What "done" looks like.** Both you and the customer sponsor can read it without further explanation and act on the recommendation. It does not need to be long — 2–4 pages — but it must be concrete.

**Where the craft lives in this track.** [mod-102 — Customer Discovery, Pilot Scoping, and Success Criteria](../mod-102-customer-discovery-and-pilot-scoping/) covers continuous-discovery interviewing (Teresa Torres), story mapping (Jeff Patton), workflow process-mining, and the observe-first-code-later discipline.

## Deliverable 2 — Pilot scope document

**When.** End of week 2 or 3.

**Who signs it.** You, the customer-side sponsor, and (for regulated verticals) the customer-side head of the operator function. Your engineering manager and the customer AE countersign if the arrangement is commercial.

**Why it exists.** To give both sides a contract they can point to when scope creep arrives, and to pre-commit to a go/no-go decision at pilot exit.

**What is in it.**

- **Problem statement.** One or two paragraphs — the workflow, the hypothesis, the bet.
- **User stories.** Named user, named action, named success. 3–10 stories, no more.
- **Non-goals.** Explicit. "This pilot does *not* extend to the back-office reconciliation step." Every FDE pilot is scoped by its non-goals as much as by its goals.
- **Success metrics.** Primary (one business-level metric). Secondary (2–3 operational metrics). Guardrail (what is not allowed to regress — safety, latency, cost per task, user-reported complaints).
- **Exit-to-production criteria.** The threshold on each success metric that constitutes "pilot succeeded, promote to production." Written before you have any data.
- **Decision record for stack choices.** Model provider, retrieval engine, framework, hosting. Each choice has a one-paragraph rationale and an alternative considered. Cite Michael Nygard's [Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions.html) for the ADR shape.
- **Pre-mortem.** List the three most likely ways this pilot fails. For each, what you will watch for and what the mitigation is.
- **Timeline.** Week-by-week to go-live. Include a "re-decide" checkpoint at roughly 50% of the pilot.
- **Who does what.** Named engineers on both sides. Named SMEs. Named approvers.

**What "done" looks like.** The customer's sponsor and operator-head can sign it. You can tell a new engineer joining mid-pilot what the pilot is and is not in under five minutes with this document in front of you.

**Where the craft lives in this track.** mod-102 covers pilot-scope document authoring and the exit-to-production criteria pattern.

## Deliverable 3 — Eval protocol

**When.** End of week 3 or 4, before any significant feature engineering.

**Who signs it.** You, the customer-side SME lead (the person whose team will label), and your peer `ai-eval-engineer` if the methodology is non-trivial.

**Why it exists.** Because LLM features cannot be tested with "does it return the correct row." Without a written eval protocol, you will ship something, show a demo, and never know whether it is actually working. Without a *written* protocol, the customer will later dispute whether it worked at all. This document defines the shared ground truth for "did the pilot succeed."

**What is in it.**

- **Gold-set shape.** How many examples, stratified how, labelled by whom, with what instructions. Inter-rater reliability target.
- **Offline eval design.** Task correctness (how measured — exact match, judge-as-LLM, pairwise comparison?). Groundedness and citation accuracy if retrieval is involved. Safety / refusal rate. Who runs this and on what cadence.
- **Online eval design.** Judge-as-LLM at sample rate on production traffic. Implicit user-feedback capture. Drift detection. Per-tenant quality dashboard.
- **Eval-vs-metric-vs-KPI mapping.** How each eval metric maps to one of the pilot success metrics, and how those map to a customer business KPI. If the chain breaks anywhere, the eval is not useful.
- **Compliance posture.** Where the eval infrastructure runs (inside VPC, zero-retention sandbox, or sanitised extract). Who has access to labelled data. How long it is kept.
- **Pass/fail gates.** Which thresholds must be hit to merge a PR (eval-gated CI). Which thresholds must be hit to go live (exit criteria). Which thresholds trigger a rollback in production.

**What "done" looks like.** Your `ai-eval-engineer` peer reviews it and does not flag a methodology issue. The customer's SME lead has read it and has committed labelling capacity. You can run the offline eval on a trivial baseline (e.g., a frozen prompt) and get numbers that make sense.

**Where the craft lives in this track.** [mod-107 — Eval Engineering on Customer Data](../mod-107-eval-eng-on-customer-data/) covers the full craft — SME-paired gold sets, offline eval suite (Promptfoo + RAGAS + DeepEval + Inspect), online eval, exit eval.

## Deliverable 4 — Integration design

**When.** Through weeks 2–6, iteratively — the first version before you write non-trivial integration code, the final version before go-live.

**Who signs it.** You, the customer-side architect or lead engineer on the affected systems, and the customer-side security reviewer.

**Why it exists.** Because the pilot does not run in a vacuum — it integrates with the customer's identity provider (SAML/OIDC/SCIM), their data surface (REST/GraphQL/webhooks/CDC), their SaaS stack (Salesforce, ServiceNow, SAP, Workday, Snowflake, Databricks, Google Workspace, Microsoft 365), possibly a legacy system (SOAP, SFTP, mainframe gateway), and their networking posture (PrivateLink, VPC peering, VPN, egress proxying). If any of those integrations is wrong on the design, the pilot ships late or does not ship at all.

**What is in it.**

- **System context diagram.** The pilot in the middle; every system it reaches, every system that reaches it, with protocols labelled.
- **Identity & authentication.** Which IdP (Okta, Entra, Ping, Google Workspace). SSO protocol (SAML 2.0 or OIDC 1.0). SCIM provisioning if applicable. Service-to-service auth shape (OAuth 2.0 client-credentials, mTLS, token exchange per [RFC 8693](https://datatracker.ietf.org/doc/html/rfc8693)). Session and token lifetimes.
- **Data surfaces.** For each surface the pilot reads from or writes to: protocol (REST / GraphQL / CDC / event bus), rate limits, auth, data shape, and whether it is read-only or read-write.
- **Networking.** VPC placement. Private endpoints. Egress controls. Any firewall or proxy ask from the customer's security team.
- **MCP surfaces** (if the integration shape calls for a Model Context Protocol server) — tool surface, auth model, audit-log shape.
- **Failure handling.** Each integration: what happens on timeout, on error, on auth failure, on rate-limit, on data-shape change. Retries and idempotency keys where applicable.
- **Open questions / escalations.** Where you need the peer `ai-infra-inference-engineer` (if a model lives inside the customer VPC), the peer `ai-data-engineer` on the customer side (if customer-side ETL is in scope), or the customer's security reviewer.

**What "done" looks like.** The customer's architect agrees. The customer's security reviewer can start the formal review with this document as the input. The integration code can be built against it without ambiguity.

**Where the craft lives in this track.** [mod-103 — Enterprise Integration Engineering](../mod-103-enterprise-integration/) covers SSO, service auth, SaaS platform integration, legacy surfaces, networking posture, and MCP server implementation. mod-104 and mod-105 cover the LLM and grounding slices that layer on top.

## Deliverable 5 — Deployment runbook

**When.** First draft in week 4–6; finalised before go-live. Updated every time the deployment process changes.

**Who signs it.** You, the customer-side operator lead (the person whose team will run this), and the customer-side on-call lead.

**Why it exists.** Because on the day you flip the feature flag to ship the pilot to real users, you do not want to be inventing a rollout plan. And on the day something goes wrong in production, you do not want to be inventing a rollback plan.

**What is in it.**

- **Deployment model.** SaaS, dedicated SaaS, VPC install, or on-prem — with the environment-specific trade-offs the customer accepted.
- **Rollout plan.** Feature-flag strategy — which flags, which tenants/users/prompts/models are gated, what the per-tenant default is, who can flip them. Canary (5% → 25% → 100% with hold times). Shadow mode if used. Kill-switch and how to flip it in under a minute.
- **Pre-flight checklist.** Everything that must be green on go-live day — eval gates, security sign-off, on-call rotation staffed, status page ready, customer comms sent.
- **Rollback plan.** Specific steps to disable the feature, revert to prior behaviour, and preserve the state and logs for post-mortem. Tested at least once in a lower environment.
- **Monitoring and alerts.** Which dashboards. Which alerts page whom. What "normal" looks like for the first 24 hours after launch.
- **Per-tenant versioning.** How prompts, retrievers, models, tool allow-lists, and judges are versioned per tenant; how to replay a decision.
- **Known issues at ship.** Things you deliberately deferred. The list the customer and your CSE/CSM both have.

**What "done" looks like.** A rollback drill has been run successfully in a lower environment. The customer's operator lead can describe the rollout and rollback process without reading the doc. Alerts page the right people.

**Where the craft lives in this track.** [mod-109 — Deployment, Rollout, and Change Management with the Customer](../mod-109-deployment-rollout-change-management/) covers VPC install, feature-flag strategy (LaunchDarkly, Statsig, Unleash), canary / shadow / kill-switch, and per-tenant versioning. [mod-110 — Observability and On-Call for Shared Customer Systems](../mod-110-observability-and-oncall/) covers the dashboards and alerts.

## Deliverable 6 — Hand-off packet

**When.** End of the pilot, as the engagement transitions to steady state.

**Who signs it.** You, the customer-side engineer(s) who are taking operational ownership, and your CSE/CSM who will now hold the longer-term relationship.

**Why it exists.** The pilot stays running after you leave. If the hand-off is not real — if the customer's engineers cannot operate the pilot, cannot debug it, cannot extend it — then either the pilot fails over the next quarter, or you remain attached to it informally and your team cannot staff the next engagement. Either outcome is a failure for the business.

**What is in it.**

- **Admin dashboards.** The customer's operators can see per-tenant quality, cost, latency, usage, and incidents without your help.
- **SME training materials.** The people who label, review, or approve agent actions have a current, concrete training packet. Not a slide deck from four months ago — a current one.
- **Playbooks for common failure modes.** Prompt regression, provider outage, retrieval drift, hallucination spike, cost blowout, guardrail false-positive storm — for each one, "symptoms, diagnosis, mitigation, follow-up."
- **Runbook for the deployment.** The deliverable from (5) above, kept current.
- **Documentation of the stack.** Where the code lives, how CI/CD works, how to run it locally, how to run the eval, how to deploy, how to roll back. Written by the FDE, reviewed by the customer's engineers who now own it.
- **Open items.** What is not done. What is on the FDE team's product roadmap. What the customer will not get from this product without doing it themselves. The honesty of this list is the single biggest predictor of whether the hand-off holds.
- **Contact graph.** For the next 90 days: who the customer contacts for what, with response-time expectations.

**What "done" looks like.** The customer's engineering team opens a PR against your pilot repo (or its hand-off equivalent) and lands it without your help. The customer's operators handle a routine failure mode without your help. You can go on vacation.

**Where the craft lives in this track.** mod-109 covers the admin dashboards and SME training. [mod-112 — Consulting Craft](../mod-112-consulting-craft-and-communication/) covers the knowledge-transfer cadence and the "the FDE becomes unnecessary at hand-off" discipline.

## Deliverable 7 — Field → product review

**When.** Recurring — a short pass at every pilot milestone, a longer synthesis at pilot end. Not a one-shot.

**Who signs it.** You and your engineering manager, with product leadership as audience.

**Why it exists.** This is the deliverable that justifies the role existing. Without it, the FDE team is a set of consultants. With it, each engagement compounds into product improvement. See Chapter 02, Stance 4.

**What is in it.**

- **Pattern writeups.** Specific patterns that worked (or did not) on this engagement, named, with enough context that another FDE could apply or avoid them. One pattern per writeup.
- **Pain-point taxonomy.** Grouped failures or friction points from the engagement, categorised (product bug, feature gap, platform gap, process gap), prioritised.
- **Platform-feature requests.** Each one properly packaged — problem, evidence from this customer (and ideally one other), proposed interface sketch, risk / scope size, workaround shipped in the interim.
- **"What we learned" one-pager.** For the product team. Dense, honest, actionable.
- **Customer success story at product-actionable depth** (if the engagement warrants). For an all-hands or external case study, co-authored with marketing — but the engineering truth, not sales polish.
- **Regression fixtures and gold-set additions** fed back into the shared eval infrastructure.

**What "done" looks like.** Product and platform teams can point to three things on their roadmap that this engagement caused. Another FDE preparing for a similar engagement can read your writeups and avoid the mistakes you made.

**Where the craft lives in this track.** [mod-111 — Field → Product Feedback Loop and Internal Advocacy](../mod-111-field-to-product-loop/) covers the full craft — artifact authoring cadence, platform-request packaging, roadmap influence, FDE-team knowledge-base authorship.

## How the seven deliverables relate

Two useful ways to see the structure:

**As a pipeline.** Discovery brief → pilot scope → eval protocol → integration design → deployment runbook → hand-off packet → field→product review. Each deliverable's output is an input to the next. Skipping one leaves a hole the later deliverable cannot close.

**As two orthogonal axes.** Deliverables 1–5 are forward-looking (you are planning or building something). Deliverable 6 is a hand-off (you are transferring ownership). Deliverable 7 is backward-looking and outward-facing (you are feeding what you learned back to the product). The ratio of your time across the three categories is a diagnostic. If you are spending 95% on 1–5, your pilot is likely too ambitious or your field→product loop is broken. If you are spending 50% on 7, you have stopped shipping. The healthy ratio varies with pilot maturity, but it is never 100% in any category.

## Summary

- The FDE is on the hook for seven deliverables across a design-partner engagement: discovery brief, pilot scope, eval protocol, integration design, deployment runbook, hand-off packet, field → product review.
- Each deliverable has a defined time, signer set, content, and "done" definition.
- The sequence is a pipeline — each deliverable's output feeds the next. Skipping one leaves a hole.
- Six of the seven are taught in depth in later modules (102, 107, 103, 109, 110, 111, 112). This chapter is the index.
- The seventh deliverable — field → product review — is the one that makes the FDE role compound. Treat it as a deliverable, not a chore.
