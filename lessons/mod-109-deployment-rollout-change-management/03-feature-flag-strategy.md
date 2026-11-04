# Chapter 03 — Feature-Flag Strategy for LLM Features — LaunchDarkly, Statsig, Unleash, OpenFeature

## Why this chapter exists

Chapter 02 landed the artefact — Terraform, Helm, the gateway, the retriever index, the judge endpoints — in the customer's environment, under the deployment mode that fits their regulatory posture. Nothing rolls out safely from there without a feature-flag surface that per-tenant, per-user, per-prompt, per-model routing can hang off. Flags are how a pilot expands from one SME to five to a department to a line of business; how a bad release gets cut in sixty seconds; how a shadow run compares a candidate model to the control without exposing it to end-users. Without flags the only rollback is `helm rollback`, and a Helm rollback is a 15-minute CAB event, not a kill-switch.

The organising concern of this chapter is the **tangle problem**: one flag reused across rollout, kill-switch, and experiment semantics. A rollout flag left live past rollout becomes a de-facto kill-switch; an experiment flag reused for rollout gets flipped by an experimentation stop-rule at an inconvenient moment; a kill-switch that doubles as an A/B assignment gate cuts the wrong cohort during an incident. The vendor platform team ([mod-109 Chapter 01](./01-deployment-and-rollout-overlay.md)) has already picked a flag platform at the organisation level; the FDE's per-customer job is to pour the taxonomy, the naming, the audit wiring, and the decommissioning cadence on top so that a flag flip means one unambiguous thing.

## The platforms

The FDE adapts to the vendor's incumbent platform rather than imposing one. Four commercial / spec surfaces and one open-source alternative are worth naming:

- **LaunchDarkly** ([docs.launchdarkly.com](https://docs.launchdarkly.com/)) — commercial, dominant in US enterprises; strong per-tenant / per-user targeting, enterprise SSO, data export to Splunk / Datadog / S3, relay-proxy for air-gap-adjacent topologies.
- **Statsig** ([docs.statsig.com](https://docs.statsig.com/)) — commercial, strong experimentation stack (sequential testing, CUPED, holdouts); the flag surface is a byproduct of the experiment engine.
- **Unleash** ([docs.getunleash.io](https://docs.getunleash.io/)) — open-source, Apache 2.0, self-hostable inside a customer VPC; an Enterprise tier exists for SSO / audit / compliance features.
- **OpenFeature** ([openfeature.dev](https://openfeature.dev/)) — CNCF-incubating vendor-neutral spec with SDKs that wrap LaunchDarkly, Statsig, Unleash, GrowthBook, Flagd, and others via providers. Code against the spec; swap providers when a customer insists on their own platform.
- **GrowthBook** ([docs.growthbook.io](https://docs.growthbook.io/)) — open-source, MIT-licensed; popular with data teams; SQL-backed experiment analysis; self-hostable.

The platform choice is often forced by the deployment mode from Chapter 02. For SaaS multi-tenant and dedicated-SaaS pilots the vendor's hosted LaunchDarkly or Statsig tenant is the path of least resistance. For VPC-install customers with strict data-residency clauses, a hosted SaaS flag platform means flag state and targeting rules transit a third party — self-hosted Unleash or GrowthBook inside the customer VPC often beats hosted on residency grounds. For air-gapped customers no SaaS flag platform is reachable at all; see the Archetype B discussion below.

OpenFeature is the hedge. Writing application code against `OpenFeatureAPI.getClient().getBooleanValue("killswitch.member-inquiry.v2.1", false, ctx)` means a customer that later insists on their own flag platform is a provider-swap, not a refactor.

<!-- needs-research: confirm OpenFeature's CNCF maturity status at time of publishing — was in incubation as of mid-2024, may have progressed -->

## Flag scopes an LLM pilot needs

A generic web-service flag platform targets by user ID and request header. An LLM pilot needs richer scopes because the artefact under rollout has more surfaces than a web handler. Seven scopes recur:

- **Per-tenant (customer-level).** The pilot's entire v2.1 bundle is on for `customer-a` and off for `customer-b`. Example: `rollout.member-inquiry.bundle-v2.1` targeted on `context.tenant_id == "customer-a"`.
- **Per-user (an SME inside the customer).** v2.1 is on for the five SMEs in the pilot cohort; everyone else sees v2.0. Example: `rollout.member-inquiry.bundle-v2.1` targeted on `context.user_email ∈ pilot-cohort-list`.
- **Per-prompt.** A named prompt bundle (the system prompt, the few-shot block, the output schema) is swapped. Example: `prompt.member-inquiry.answer-v3` returns `"candidate"` vs `"control"` and the gateway from [mod-104 Chapter 03](../mod-104-llm-app-in-customer-envs/03-model-gateway-in-customer-env.md) chooses which prompt bundle to assemble.
- **Per-model.** Model provider or model version. Example: `model.member-inquiry.answerer` returns `"bedrock:anthropic.claude-3-5-sonnet-20241022"` or `"bedrock:anthropic.claude-3-7-sonnet-20250219"`. Covers both provider swaps (OpenAI → Anthropic) and version pins (3.5 → 3.7).
- **Per-retriever.** Which retriever / reranker is live, pinning the index version from [mod-105](../mod-105-grounding-on-customer-data/). Example: `retriever.member-inquiry.index` returns `"pgvector-v12"` or `"pgvector-v13-with-new-reranker"`.
- **Per-tool allow-list.** Which tools the agent from [mod-106 Chapter 03/04](../mod-106-agentic-workflows-on-enterprise/) can call. Example: `tools.member-inquiry.agent-allowlist` returns `["lookup_member", "fetch_claim"]` or the expanded `["lookup_member", "fetch_claim", "submit_appeal"]` for a controlled rollout of a write-capable tool.
- **Per-judge.** Which online-eval judge from [mod-107 Chapter 05](../mod-107-eval-eng-on-customer-data/) is live. Example: `judge.member-inquiry.online` returns `"judge-v4-with-citation-check"` so a candidate judge can be swapped in without a code release.

A flag platform that cannot target on structured context — `tenant_id`, `user_email`, `workflow`, `environment`, `request_id` — forces the gateway to do the targeting in application code, which defeats the audit trail. Both LaunchDarkly and Statsig support arbitrary context attributes; Unleash's strategy constraints handle the same shape.

## Three use-cases, one platform — the tangle problem

Flags serve three distinct purposes and one platform hosts all three. Separating them by **prefix**, **lifetime**, and **approval surface** is the discipline that keeps them from tangling.

| Class | Prefix | Lifetime | Approval surface | Flip authority | Example |
|---|---|---|---|---|---|
| Rollout | `rollout.*` | Temporary; decommissioned after full rollout (target date set at creation) | CAB ticket per expansion step | FDE or customer ops lead | `rollout.member-inquiry.bundle-v2.1` |
| Kill-switch | `killswitch.*` | Permanent; off by default, flipped only during incidents | Incident ticket; retrospective CAB entry | Customer ops lead (fast path) | `killswitch.member-inquiry.v2.1` |
| Experiment | `experiment.*` | Temporary; bounded by experiment duration | Experiment-plan approval from `ai-enablement-lead` | Statsig / experiment-platform stop-rule | `experiment.member-inquiry.prompt-v3-vs-v2` |

Three classes, three lifetimes, three flip paths. A flag that is all three gets flipped by a stop-rule intended for the experiment semantics at the moment the operator needed the kill-switch semantics and the result is that the rollout cohort gets reshuffled during an incident. Chapter 04 depends on this separation: canary and shadow consume `rollout.*`; A/B consumes `experiment.*`; the incident playbook from Chapter 06 consumes `killswitch.*`.

The naming convention — `<class>.<workflow>.<slice>` — makes the class legible to every reviewer. A reviewer scanning a CAB ticket that flips `rollout.member-inquiry.bundle-v2.1` to `on` for tenant `customer-a` knows, without reading further, that this is a rollout step and should be reversed by setting it back to `off`, not by flipping some other flag.

## Shadow deployments via flag

Shadow deployment is the pattern where a flag routes a *copy* of production traffic through a candidate prompt / model / retriever / tool allow-list and only *shadows* its output — the candidate's response never reaches the end-user; it is logged for comparison against the control. The pattern lets the FDE run a candidate bundle against real customer traffic with zero customer-visible risk.

The plumbing lives in the gateway from [mod-104 Chapter 03](../mod-104-llm-app-in-customer-envs/03-model-gateway-in-customer-env.md):

1. Gateway receives request from the customer app.
2. Gateway evaluates `rollout.member-inquiry.shadow-v2.1` with request context.
3. If the flag returns `on` for this request:
   - The gateway clones the request (deep copy of prompt inputs, retrieval context, tool-call grants).
   - It runs the control arm (v2.0) and the shadow arm (v2.1) concurrently, each with its own timeout and cost budget.
   - It tags both invocations with the same `correlation_id` and distinct `arm` labels (`"control"` / `"shadow"`).
   - It returns the control arm's response to the user.
   - It persists both arms to the invocation log for off-line comparison.
4. If the flag returns `off`, only the control arm runs.

A minimal shadow flag:

```yaml
# unleash-style flag definition
key: rollout.member-inquiry.shadow-v2.1
class: rollout
description: |
  Shadow v2.1 bundle against control v2.0; v2.1 output is NOT returned to the user,
  only logged for Chapter 04 shadow analysis.
default_value: off
variants:
  - name: on
    weight: 0
  - name: off
    weight: 100
strategies:
  - type: gradualRolloutUserId
    parameters:
      percentage: 5
      stickiness: userId
    constraints:
      - contextName: tenant_id
        operator: IN
        values: ["customer-a"]
      - contextName: workflow
        operator: IN
        values: ["member-inquiry"]
owner: fde-pilot-customer-a@vendor.example
created: 2026-10-14
target_decommission: 2026-12-14
```

The gateway is responsible for cost accounting on the shadow arm — shadowing doubles model-call spend over the shadow percentage of traffic. Chapter 07 maps the cost model; Chapter 04 is where shadow analysis itself happens.

## Kill-switch design

A kill-switch flag has three non-negotiable properties.

**Reachable when the control plane is unhealthy.** Do not route the kill-switch through the thing you are trying to kill. If the LaunchDarkly SDK fetches rules from a relay proxy that runs inside the same cluster as the degraded workload, a cluster outage disables both the workload and the kill-switch. Mitigations: the flag platform must have an independent fetch path (LaunchDarkly Relay Proxy with cached rules, Unleash's bootstrap-from-file mode) *and* a sane default that returns "off" (feature disabled) when the platform is unreachable. The SDK's `defaultValue` argument is load-bearing — if the SDK falls back to `defaultValue` on timeout, that default must be the safe state (feature off), not the current state (feature on).

**Auditable.** Every flip is logged with actor, prior value, new value, reason, approval ticket, timestamp, and scope. The audit record is exported to the customer's SIEM, not just retained in the vendor's flag-platform console. The `## Audit log of every flip` section below specifies the field set.

**Scope-aware.** A kill-switch for customer A does not kill customer B unless intentional. The flip's targeting rules must be explicit; a kill-switch toggled "globally" because the console defaulted to "all environments" is the archetypal post-incident retro finding.

A concrete LaunchDarkly-shape kill-switch flag:

```json
{
  "key": "killswitch.member-inquiry.v2.1",
  "class": "killswitch",
  "description": "Disable member-inquiry workflow for tenant customer-a; bundle v2.1 or any later.",
  "variations": [
    { "value": "on",  "name": "Feature enabled (normal)" },
    { "value": "off", "name": "Feature disabled (kill-switch engaged)" }
  ],
  "defaults": {
    "onVariation": "on",
    "offVariation": "off"
  },
  "fallthrough": { "variation": "on" },
  "offVariation": "off",
  "clientSideAvailability": { "usingEnvironmentId": false, "usingMobileKey": false },
  "targets": [],
  "rules": [
    {
      "id": "customer-a-prod",
      "clauses": [
        { "attribute": "tenant_id",  "op": "in", "values": ["customer-a"] },
        { "attribute": "environment","op": "in", "values": ["prod"] }
      ],
      "variation": "on"
    }
  ],
  "sdk_defaults": {
    "default_on_unreachable": "off",
    "cache_ttl_seconds": 30
  }
}
```

Three features of this flag matter: (1) the SDK default on platform-unreachable is `off` — fail safe, not fail current; (2) the targeting rule names the scope explicitly, so flipping the rule to `off` kills the workflow only for `customer-a` in `prod`; (3) the flag's `class` is `killswitch`, so a reviewer cannot mistake it for an experiment. Unleash flags of the same shape use `constraints` on strategies and a boolean `enabled` field; the logic is identical.

## Audit log of every flip

A flag flip is a configuration change to a production system. The customer's CISO, auditor, and model-risk function treat it the same as a code deploy for audit purposes. Multiple regimes require the record:

- **SOC 2 CC6 (logical access) and CC8 (change management).** Every production configuration change has an audited record identifying the change, the approver, and the time.
- **HIPAA audit controls.** [45 CFR 164.312(b)](https://www.hhs.gov/hipaa/for-professionals/security/laws-regulations/index.html) requires mechanisms that record and examine activity in information systems containing or using ePHI.
- **SR 11-7 (US banking model-risk governance).** Model changes require documentation of the change and a governance approval chain.
- **DORA Article 28 (ICT third-party risk).** The financial entity retains audit rights over ICT third parties; configuration changes affecting the service are in-scope.
- **EU AI Act Article 72 (post-market monitoring).** Providers of high-risk AI systems collect, document, and analyse data about the system's operation.

<!-- needs-research: confirm SR 11-7 wording on "model change" vs "model-risk management" scope — SR 11-7 is the Federal Reserve's 2011-04-04 supervisory letter; verify current version and OCC 2011-12 companion -->

The minimum field set, as JSON:

```json
{
  "flag_key": "killswitch.member-inquiry.v2.1",
  "prior_value": "off",
  "new_value": "on",
  "actor": "jane.doe@customer.example",
  "actor_role": "operations-lead",
  "reason": "PII leak suspected in retriever reranker; rolling back to v2.0",
  "approval_ticket": "CAB-2026-11-0472",
  "timestamp": "2026-11-04T14:03:12Z",
  "scope": {"tenant_id": "customer-a", "environment": "prod"}
}
```

The record ships to the customer SIEM through the flag platform's native data-export surface (LaunchDarkly Data Export to Kinesis / Segment / webhook; Statsig audit-log export; Unleash event-log webhook). The SIEM pairs it with the CAB ticket (`approval_ticket`) and the incident-management record if one exists. [Mod-108 Chapter 02](../mod-108-enterprise-security-and-compliance/02-security-review-packet.md) specifies the SIEM integration shape; the admin dashboard from Chapter 06 of this module renders the same feed for SME-readable visibility.

A flip that is not attached to an approval ticket is an incident finding. The flag platform's workflow or an external approval gate (ServiceNow CAB workflow, Jira change ticket) enforces this at flip time — not retroactively.

## Flag decommissioning

Flags have a half-life problem. A `rollout.*` flag created for v2.1's expansion, never cleaned up after v2.1 reached 100%, is three risks at once:

- **Technical debt.** The gateway carries targeting code for a flag whose only purpose has been served. Twelve months of accumulation makes the gateway's flag evaluator a graveyard.
- **Security surface.** A dormant flag can be flipped by anyone with flag-platform access; the targeting rules may no longer reflect any live scope, so a flip produces unpredictable behaviour.
- **Audit noise.** Auditors reviewing the flag register ask about every live flag. A register bloated with dead rollout flags is a review-time multiplier.

Three disciplines prevent the half-life accumulation:

- **Target decommission date at creation.** Every `rollout.*` and `experiment.*` flag carries a `target_decommission` field (90 days for a rollout flag; experiment-duration + 14 days for an experiment flag). `killswitch.*` flags are exempt — they are permanent by design.
- **Quarterly review.** The FDE (or the customer-side admin after handoff — Chapter 06) reviews the flag register each quarter: every flag past its target date is removed or has its target extended with an explicit reason.
- **CI check.** The flag register is a file (or an API query) in the pilot's Git repository. A CI job fails the build when any flag is past its target decommission date without an extension note. This closes the loop; a flag cannot quietly outlive its purpose.

A rollout flag's exit sequence: (1) flag at 100% for two weeks with no regressions; (2) code path assumes the new behaviour and removes the control branch; (3) flag is deleted from the platform; (4) register entry moves to `history/` with retention per the customer's regime.

## Running the three archetypes

**Archetype A — US healthcare payer (HIPAA + SOC 2 Type II, AWS Bedrock + VPC install).** Flag platform: vendor-provided LaunchDarkly tenant; the customer's operations lead is granted read + flip authority on `killswitch.*` flags only, via a scoped API key with LaunchDarkly's role-based access. Audit stream: LaunchDarkly Data Export → Amazon Kinesis → the customer's Splunk HEC endpoint via the customer's own data pipeline (no cross-tenant SIEM sharing). Flag scopes in use: per-tenant (customer-a everywhere), per-user (five-SME pilot cohort by email), per-prompt (member-inquiry v2.1), per-model (Claude 3.5 → 3.7 upgrade), per-retriever (pgvector v12 → v13 reranker), per-judge (judge-v4 with citation-check). Kill-switch design: SDK `defaultValue = off` on unreachable; LaunchDarkly Relay Proxy inside the customer VPC with a 30-second cache TTL. Decommissioning: quarterly review co-run with the customer's quarterly CAB window.

**Archetype B — US federal agency (FedRAMP Moderate, GovCloud + self-hosted inference, air-gapped update cadence).** The SaaS LaunchDarkly endpoint is unreachable from the air-gap. Two options: self-hosted Unleash inside the GovCloud environment (if an authorised flag platform exists at FedRAMP Moderate), or a minimal file-based flag store (YAML in a customer-signed Git repository, mounted into the gateway via ConfigMap). Audit stream: flag flips go to the agency's SOC via syslog-forwarded events; the retention matches the agency's records-management schedule. Flag scopes: per-tenant is moot (the agency is the tenant); per-user, per-prompt, per-model, per-retriever, per-tool-allow-list, per-judge all apply. Kill-switch design: local flag state only — no external fetch path to fail on. Decommissioning: cadence is ATO-change-control-driven (quarterly nominal); every flag flip triggers a POA&M entry if it modifies a documented control surface. <!-- needs-research: which feature-flag platforms hold a FedRAMP Moderate or FedRAMP High ATO as of 2026 — LaunchDarkly has published FedRAMP work but the authorisation state should be verified against the FedRAMP Marketplace -->

**Archetype C — EU retail bank (GDPR + DORA + EU AI Act, Azure AI Foundry + dedicated SaaS).** Flag platform: vendor-managed LaunchDarkly or Statsig tenant under the EU Data Boundary — the vendor confirms with the platform that EU-resident storage is in place; CMK on the audit-stream export. Audit stream: flag-flip events to Microsoft Sentinel via the bank's Azure Monitor diagnostic setting; retention seven years per bank-house policy and DORA audit obligations. Flag scopes: per-tenant, per-user (bank-side SME SSO identity), per-prompt (customer-support draft reply), per-model (Azure OpenAI GPT-4.1 under the bank's Azure subscription), per-retriever, per-tool-allow-list, per-judge. Kill-switch design: the bank's operations lead has flip authority on `killswitch.*`; CAB retrospective entry is required within 24 hours; SDK fail-safe default is `off`. Decommissioning: monthly review (DORA's audit cadence is tighter than quarterly); CI check lives in the bank-house DevOps platform.

Each archetype inherits the same flag taxonomy and audit shape; the platform choice, the audit destination, the flip-authority delegation, and the review cadence vary with the regime.

## Anti-patterns

**Flags without decommissioning.** A `rollout.*` flag from v1.3 is still live in v2.4; the gateway's targeting code is thirty forks of dead branches. The register has no target decommission dates and no CI gate. The fix is the three disciplines above: target date at creation, quarterly review, CI check.

**A kill-switch that routes through the control plane.** The kill-switch SDK fetches rules from a relay proxy in the same cluster as the workload; a cluster outage disables both. The fix is an independent fetch path (platform SDK's SaaS endpoint from the operator's laptop, or a bootstrap-from-file mode) *and* a fail-safe SDK default of `off`.

**Rollout and experiment on the same flag.** `member-inquiry.v2.1` is both the rollout flag and the A/B assignment flag; an experiment stop-rule flips it off mid-rollout and the five-SME cohort lose access during their Tuesday workflow. The fix is the three-prefix taxonomy — one semantic per flag, one approval surface per flag.

**Flag-flip without approval ticket.** The ops lead flips a `killswitch.*` from the LaunchDarkly console with no reason, no ticket, no SIEM record attached to a change. The customer's auditor asks three weeks later and nothing joins up. The fix is enforcing the `approval_ticket` field at flip time — the flag-platform workflow, a webhook validator, or an external approval gate (ServiceNow / Jira integration) rejects the flip without it.

**Flag audit log only in vendor logs, not in customer SIEM.** The flip is recorded in LaunchDarkly's own audit log, which the customer can read via API but does not stream into their SIEM. SOC 2 CC6 reviews land on the customer's SIEM as the source of truth; a vendor-only audit trail fails the review. The fix is the data-export pipe to Splunk / Sentinel / the customer's SIEM of choice, as shown in [mod-108 Chapter 02](../mod-108-enterprise-security-and-compliance/02-security-review-packet.md).

**Tenant scope implicit, not enforced.** The flag's targeting rule reads "customer-a only" but the SDK is invoked without `tenant_id` in the context, so every evaluation hits the fallthrough rule; a flip meant for `customer-a` quietly affects everyone. The fix is to make `tenant_id` a required context attribute at the gateway level — the SDK call fails loudly if it is missing.

## Summary

- Feature flags are the surface that makes LLM rollout safe at per-tenant, per-user, per-prompt, per-model, per-retriever, per-tool-allow-list, and per-judge resolution; without them the only rollback is a Helm rollback and every incident is a CAB event.
- The vendor's platform team picks the flag platform once (LaunchDarkly, Statsig, Unleash, GrowthBook, OpenFeature-wrapped); the FDE adapts per customer. Deployment mode (Chapter 02) constrains the choice — hosted SaaS for multi-tenant and dedicated-SaaS pilots; self-hosted Unleash or GrowthBook for VPC installs; file-based flag state for air-gapped environments.
- The tangle problem is the chapter's organising concern: a `rollout.*` flag reused as `killswitch.*` or `experiment.*` gets flipped by the wrong stop-rule at the wrong moment. The three-prefix naming convention and three approval surfaces keep the classes separate.
- Shadow deployments are a `rollout.*` flag pattern: the gateway clones the request, runs control and candidate concurrently, logs both under a shared correlation ID, and returns only the control to the user. Chapter 04 covers the analysis layer.
- Kill-switches have three required properties: reachable when the control plane is unhealthy (fail-safe SDK default `off`); auditable (every flip logged with actor, reason, ticket, scope); scope-aware (targeting rules explicit, no implicit globals).
- Every flip ships to the customer SIEM with the ten-field audit record, feeding SOC 2 CC6/CC8, HIPAA 45 CFR 164.312(b), SR 11-7, DORA Article 28, and EU AI Act Article 72 obligations.
- Decommissioning is a discipline, not a chore: target date at creation, quarterly review, CI check — otherwise the flag register becomes technical debt, a security surface, and audit noise.
- Chapter 04 picks up canary, shadow analysis, and A/B — the consumers of the `rollout.*` and `experiment.*` flag classes defined here. Chapter 05 pins the bundle manifest that flags select over. Chapter 06 renders the flag-flip feed on the admin dashboard. Chapter 08 writes the CAB boundary that approves flag flips.
