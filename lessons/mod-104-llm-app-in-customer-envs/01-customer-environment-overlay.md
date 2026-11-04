# Chapter 01 — The Customer-Environment Overlay — What This Module Adds Over the In-House Craft

## Why this chapter exists

Mod-103 landed the customer's identity, data, SaaS, legacy, and network surfaces inside a signed integration design. Mod-104 picks up at the next layer: the LLM feature itself. But almost everything about an LLM feature — which provider it calls, where the prompts live, which tenants share which cache, which SLO the pager fires on — looks different when the feature ships inside a customer's environment rather than inside the vendor's own product.

That difference is the *customer-environment overlay*. The in-house applied-LLM craft — authored in [`applied-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) at level 30 — is the *core craft*: prompt design, structured outputs, streaming, caching mechanics, cost/latency SLO authoring. This module does not re-teach that craft. It layers the per-customer, per-deployment, per-tenant concerns on top. Chapter 07 writes the boundary to that peer track explicitly; the next six chapters each add one slice of the overlay.

## The in-house craft and what it covers

Before the overlay can be understood, name the ground floor it sits on. The in-house applied-AI engineer at level 30 owns, inside their own product:

- **Prompt design** — system prompts, few-shot examples, structured-output schemas, chain-of-thought where it earns its tokens.
- **Structured outputs** — JSON Schema, Pydantic / Zod shapes, provider-native structured output, retry-on-invalid patterns.
- **Streaming** — SSE / provider streaming clients, incremental UI rendering, stream-parsing of partial structured outputs.
- **Caching mechanics** — exact-match caching, semantic caching, provider-native prompt caching, TTL design.
- **Cost / latency SLOs** — the quality / cost / latency triangle for the vendor's single-tenant product.
- **Model selection and versioning** — picking GPT-4o vs. Claude vs. Gemini for a specific feature, pinning versions, rolling model upgrades.

Those six areas are craft. The applied-AI engineer builds them against one deployment target: *the vendor's own production*, with one provider account, one secrets store, one observability stack, one prompt-review process, one compliance regime.

The FDE at level 35 operates one level up. The feature may exist in the vendor's product; it may not. Either way the FDE is landing it inside a *customer's* environment, and that environment has its own set of assumptions the in-house craft cannot carry on its back.

## The customer-environment overlay

The overlay is the set of concerns the FDE adds on top of the core craft. Each row below is a dimension the in-house craft treats as fixed and the FDE re-opens per deployment.

| Dimension | What the in-house craft assumes | What the FDE re-opens | Where this module handles it |
|---|---|---|---|
| Data residency / sovereignty | One region, product-chosen | Customer dictates — in-region processing, data-never-leaves-VPC, cross-border constraints | Chapter 02 |
| Compliance regime | Vendor's own (SOC 2 / ISO 27001 / etc.) | Vertical-specific (HIPAA, FedRAMP, PCI DSS, EU AI Act, DORA) | Chapter 02, mod-108 for depth |
| Provider allowlist | Vendor picks providers | Customer's AppSec maintains an allowlist that may exclude the vendor's preferred provider | Chapter 02 |
| Per-tenant isolation | Not needed — one tenant | Hard requirement — prompts, caches, budgets, audit logs partition by tenant | Chapter 03, Chapter 04 |
| Per-tenant budget enforcement | Not needed | Required — soft caps, hard caps, overage behaviour | Chapter 03 |
| Networking posture | Public internet from product → provider | PrivateLink, VPC endpoints, egress proxies, split-horizon DNS | mod-103 Chapter 07, revisited Chapter 03 |
| Observability destination | Vendor's own (Datadog / Honeycomb / internal) | Customer's SIEM — Splunk, Datadog, Grafana, whatever the customer runs | mod-110 for depth, Chapter 03 names the audit-log shape |
| Prompt-review workflow | Vendor's internal process | Composed with the customer's change-management process; per-tenant overrides live in a registry | Chapter 04 |
| Deployment model | SaaS multi-tenant | SaaS multi-tenant, dedicated SaaS, VPC install, or on-prem | This chapter + mod-109 for rollout |
| SLO contract | Vendor's product SLA | Per-customer contract grounded in their workflow and business KPIs | Chapter 05 |
| Fine-tune decision | Platform-level call | Per-customer call with customer data, customer licensing, customer-paid training | Chapter 06 |

Twelve rows of overlay. Each chapter in this module takes one or two. None of these rows is exotic; what is distinctive is that *every row is re-opened per customer*, which the in-house craft never has to do.

## The three deployment-model archetypes

The deployment model is the first-order sort. Three archetypes cover most FDE pilots; a fourth (on-prem / air-gapped) shows up rarely but is worth naming.

**SaaS multi-tenant.** The pilot runs in the vendor's SaaS, shared infrastructure, segmented by tenant ID at the application layer. The vendor owns the provider accounts, the secrets, the observability, the on-call. The customer consumes via the vendor's API or UI. The overlay here is lightest — mostly per-tenant budget, per-tenant prompt override, per-tenant audit log.

*Who owns what:* vendor owns everything except what the customer consumes. Customer owns their own account, their own data input, their own downstream integrations.

**Dedicated SaaS.** The pilot runs in the vendor's cloud but in a customer-dedicated namespace / project / cluster. The vendor still runs it; the customer gets a stronger isolation story and sometimes a dedicated provider account. Latency SLOs can be tighter because no noisy-neighbour contention.

*Who owns what:* vendor owns the running system; secrets sit in a customer-dedicated vault the vendor operates. Observability still flows to vendor-owned tooling, with a customer-visible dashboard. Customer owns input data and consumption patterns.

**VPC install.** The pilot runs inside the customer's own cloud account — the customer's VPC, the customer's Kubernetes cluster, the customer's observability stack. The vendor ships the artifact (Helm chart, Terraform module, container images) and supports it; the customer runs it. Provider accounts may be the customer's own (their AWS account calls Bedrock; their GCP project calls Vertex AI). Secrets live in the customer's vault.

*Who owns what:* customer owns the running system at the infra layer. Vendor owns the application code and the upgrade path. Observability flows to the customer's SIEM. On-call is shared.

**On-prem / air-gapped.** Rare, but real in federal, defence, and some healthcare settings. The customer runs everything behind a firewall with no provider egress. Self-hosted open-weights models are the only option (Chapter 02). The vendor's visibility into the running system is limited to what the customer chooses to export.

*Who owns what:* customer owns essentially everything. Vendor provides artifacts and documentation.

Each archetype pushes a different subset of the overlay rows harder. A VPC install demands the full networking-posture discussion from mod-103 Chapter 07 and the gateway-in-VPC pattern from Chapter 03. A SaaS multi-tenant pilot leans on per-tenant budget and per-tenant override. An on-prem pilot pulls Chapter 06's self-hosted decision forward.

## The organising budget: five dimensions

Mod-103 Chapter 01 introduced a three-dimensional budget (latency × reliability × cost) for integration decisions. This module operates against a five-dimensional budget, extended for the LLM layer:

1. **Data residency / sovereignty** — can the customer's data legally and contractually reach this provider in this region?
2. **Compliance** — does this provider / deployment satisfy the customer's regulatory regime?
3. **Latency** — can this choice meet the customer's latency SLO after every hop in the integration path?
4. **Cost** — does the per-task cost fit inside the value-per-task the pilot promised (mod-102 Chapter 03)?
5. **Model-choice freedom** — does this choice let the FDE route to the best model for each sub-task, or is it locked to one model family?

Every chapter in this module names the row(s) in this budget its decisions trade against. The number is rarely exact; the shape is always explicit.

Two shorthand rules:

- **Residency and compliance are gate dimensions.** If the choice fails either, it is off the table — no amount of latency or cost advantage recovers a choice that violates data-residency or compliance posture.
- **Latency, cost, and model-choice freedom are optimisation dimensions.** Among the choices that pass the gates, the FDE picks the point that best fits the pilot's workflow and business KPI.

## Where mod-104 sits with mod-103 and the rest of the track

The overlay only makes sense in context. The surrounding modules compose with it as follows:

- **Mod-103 (enterprise integration)** — the identity path, the network boundary, the data surface, the MCP server. Mod-104 assumes these exist and plugs into them. The gateway in Chapter 03 authenticates via the OAuth posture from mod-103 Chapter 03 and routes egress via the networking posture from mod-103 Chapter 07.
- **Mod-105 (grounding on customer data)** — retrieval-engine selection and ACL propagation. The prompts in Chapter 04 consume what mod-105 retrieves; the per-tenant overrides compose with per-tenant retriever scoping.
- **Mod-106 (agentic workflows on enterprise surfaces)** — agent shape, tool-use, OWASP Agentic hardening. The gateway in Chapter 03 is the egress point for agent tool calls; the prompt-caching strategy in Chapter 04 applies to agent system prompts.
- **Mod-107 (eval engineering on customer data)** — the eval gate on the prompt-review workflow (Chapter 04); the slice analysis that feeds the fine-tune decision (Chapter 06); the quality SLO cross-referenced from Chapter 05.
- **Mod-108 (enterprise security and compliance)** — the formal review of the provider stack choice (Chapter 02), the gateway audit-log shape (Chapter 03), the data-retention story in the fine-tune workflow (Chapter 06).
- **Mod-109 (deployment, rollout, change management)** — the feature-flag tool that ships prompt-version rollouts (Chapter 04), the canary / kill-switch topology that wraps model changes (Chapter 06).
- **Mod-110 (observability and on-call)** — the GenAI semantic conventions that instrument every SLO from Chapter 05, the shared on-call rotation with the customer and the peer specialists.
- **Mod-111 (field → product loop)** — the pattern writeups that come out of this module's decisions when they generalise across customers.
- **Mod-112 (consulting craft and communication)** — how the SLO contract (Chapter 05) and the fine-tune escalation (Chapter 06) get narrated to a customer VP.

## Four anti-patterns to catch early

**Treating the in-house defaults as portable.** The applied-AI engineer's prompt library, structured-output schema, caching configuration, and cost SLO numbers are tuned against the vendor's product. Carrying them unchanged into a customer deployment assumes that customer's data, workflow, and tenant isolation match the vendor's — which they do not. Chapter 04's per-tenant override pattern and Chapter 05's workflow-derived SLOs are the discipline.

**Skipping the data-residency conversation until security review.** By the time the customer's security reviewer asks "where does the prompt go, which region processes it, where are the logs stored?" (mod-108), it is week 8. If the FDE picked a provider in week 2 without checking the customer's residency posture, the pilot has either (a) re-scoped the stack under deadline pressure or (b) silently violated the customer's policy. Chapter 02 names this as a gate dimension for a reason.

**Picking a provider on model-quality benchmark without the customer-side constraints.** The frontier model with the highest MMLU is often not the model the customer's AppSec will approve to call from their VPC. Chapter 02 scores the full five dimensions; model quality is one of them, and gates come first.

**Treating the gateway as optional "until we need it."** A pilot that goes live without a gateway (Chapter 03) has: no per-tenant budget enforcement, no provider failover, no centralized audit log, no PII guard, and a direct provider-SDK coupling in every pilot code path. Every one of those becomes a required feature at some point in a serious customer engagement; Chapter 03 argues for landing them on day one.

## Summary

- The customer-environment overlay is the set of concerns the FDE adds on top of the in-house applied-LLM craft owned by `applied-ai-engineer` (level 30). Mod-104 does not re-teach the core craft; it layers.
- Twelve dimensions are re-opened per customer — from data residency through SLO contract to fine-tune decision. Each chapter in this module takes one or two.
- Three deployment-model archetypes — SaaS multi-tenant, dedicated SaaS, VPC install — plus a rare on-prem case. Each pushes a different subset of the overlay harder.
- The organising budget for the module is five-dimensional: data residency, compliance, latency, cost, model-choice freedom. Residency and compliance are gate dimensions; the other three are optimisation dimensions.
- Mod-104 composes with the rest of the track — mod-103's integration surfaces feed it, mod-105–107 build on its prompt and SLO decisions, mod-108–110 operate the result in production.
- Four anti-patterns to catch early: treating in-house defaults as portable, deferring residency, benchmark-first provider picks, and skipping the gateway "until later."
- Chapter 02 is next: how to score the three provider categories against the five-dimensional budget for a specific customer.
