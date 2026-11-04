# Chapter 01 — The Deployment and Rollout Overlay — Why FDE Deployment Is Not Generic Deployment

## Why this chapter exists

Mod-107 answered "is this good enough?" and mod-108 answered "is this safe enough, provable enough, contractually clean enough that the customer's CISO, DPO, model-risk committee, and auditor will sign?" Mod-109 picks up the step that decides whether a signed-off pilot actually *reaches* production — through a change-advisory board, a feature-flag platform, a canary, an SME programme, and a signed handoff — and keeps reaching production on every subsequent release.

This module is deliberately an *overlay* on three in-house crafts, not a reimplementation of them. The vendor's release-engineering / platform-engineering team already runs CI/CD, SLSA provenance, cluster operations, and cost dashboards at platform scale. The [`ai-enablement-lead`](https://github.com/ai-governance-curriculum/ai-enablement-lead-learning) (peer, level 35, Governance / Enablement family) already runs the organisational change-management programme inside the customer — exec-sponsor comms, SME programme design, adoption measurement, benefit realisation. The customer's own release-engineering / SRE / change-management functions already run everything that lives in the customer's cloud or data centre. The FDE does not re-teach any of the three. The FDE layers per-customer rollout overlay on top of all three, at working depth.

<!-- needs-research: confirm the current repo URL for ai-enablement-lead-learning in the ai-governance-curriculum org before publishing -->

## The in-house crafts and what they cover

Three peers, three crafts, all deep. Name them before adding the overlay.

**Vendor release-engineering / platform-engineering (in-house, Infra family)** owns, across every pilot the vendor runs:

- **CI/CD at platform scale.** Multi-stage pipelines; SBOM / ML-BOM generation integrated per [mod-108 Chapter 06](../mod-108-enterprise-security-and-compliance/06-ai-supply-chain-provenance.md); Sigstore / SLSA attestations on every release artifact; artefact registry with retention policies.
- **Cluster operations.** Kubernetes platforms (EKS, GKE, AKS), service mesh, autoscaling, cost allocation, multi-tenant isolation.
- **Platform observability.** Centralised metrics, logs, traces, alerting at a fleet level.
- **Platform feature-flag account.** The vendor adopts LaunchDarkly / Statsig / Unleash / OpenFeature once at the organisation level; individual pilots inherit the SDK, the SSO integration, and the audit stream.
- **Platform cost dashboards.** Model-call cost, egress, inference GPU-hour, retrieval compute — rolled up across the fleet.

**`ai-enablement-lead` (peer, level 35, Governance / Enablement family)** owns, inside the customer organisation:

- **Change-management programme design.** Stakeholder maps, impact assessments, resistance management, exec-sponsor cadence.
- **SME programme operations.** Curriculum design, cohort scheduling, train-the-trainer, certification, ongoing refreshers.
- **Adoption measurement and benefit realisation.** KPIs the business committed to in mod-102 (customer discovery) are now tracked by this role.
- **Executive communications.** Monthly steering updates, quarterly business reviews, year-end adoption write-ups.
- **Organisational readiness.** Who is going to run this when the FDE goes home? When is the first customer-side admin trained? Who is the fall-back?

**The customer's own release-engineering / SRE / change-management function** owns, inside the customer's cloud or data centre:

- **The runtime.** Kubernetes clusters, VMs, inference GPUs, storage, network, IAM.
- **CI/CD into the customer environment.** Terraform / Helm applied from a customer-trusted pipeline.
- **The change-advisory board (CAB).** Risk-tiered approvals for production changes; outage windows; freeze periods.
- **The pager.** Customer-side SRE rotation that fires when something in production misbehaves.
- **Steady-state operations.** Patching, scaling, incident response, cost management, after the vendor FDE moves on.

Three crafts. Three peers. All of them operate against their own product, their own platform, their own enterprise. The FDE at level 35 re-opens parts of each one *per customer*, because every customer's rollout is the customer's own: their CAB, their SIEM, their change window, their SME cohort, their signatories. That is the overlay.

## The deployment and rollout overlay

The overlay is the set of concerns the FDE adds on top of all three in-house crafts. Each row is a dimension the craft resolves once and the FDE re-opens per customer.

| Dimension | What the in-house craft resolves once | What the FDE re-opens per customer | Where this module handles it |
|---|---|---|---|
| Release pipeline design | Vendor platform CI/CD | This customer's Terraform / Helm artefact — regions, VPCs, KMS keys, IAM roles, OIDC issuer | Chapter 02 |
| Deployment mode | Platform supports SaaS, dedicated SaaS, VPC install, on-prem bundle | Which mode this customer's regulatory / residency / support posture demands; the trade-offs on latency, model choice, update cadence | Chapter 02 |
| Model choice coupling | Platform offers hosted frontier APIs, hyperscaler-native, self-hosted | Which models are reachable *from this deployment mode* without breaking the data contract from [mod-108 Chapter 05](../mod-108-enterprise-security-and-compliance/05-data-contract.md) | Chapter 02 |
| Feature-flag platform | Vendor's LaunchDarkly / Statsig / Unleash / OpenFeature account | Per-tenant flag taxonomy; kill-switch wiring; audit-log integration with customer SIEM; naming convention that keeps rollout / killswitch / experiment flags from tangling | Chapter 03 |
| Canary strategy | Vendor runs a canary programme internally | LLM-specific canary over the customer's own traffic — eval pass-rate, judge agreement, cost-per-call, p95 latency — gated against mod-107 pilot gold set | Chapter 04 |
| Shadow deployments | Platform supports request-cloning in the gateway from [mod-104 Chapter 03](../mod-104-llm-app-in-customer-envs/03-model-gateway-in-customer-env.md) | What is shadowed per pilot — candidate prompt, model upgrade, new reranker — and how outputs are compared | Chapter 04 |
| A/B experiment design | Platform has experimentation tooling | Per-pilot hypothesis, randomisation unit, powered sample size, metric-latency handling for LLM outcomes | Chapter 04 |
| Per-tenant artefact versioning | Platform artefact registry | Per-customer bundle (prompt + retriever + model + tool allow-list + judge + guardrails + gateway config) with explicit lineage | Chapter 05 |
| Replay harness | Platform logs invocations | Per-customer replay API — bit-for-bit re-run of any past interaction against any past bundle — with retention tuned to the customer's regime | Chapter 05 |
| Diff-review workflow | Platform PR tooling | What the review surface shows for an LLM bundle: natural-language diff + eval-regression diff + replay-sample diff; who the reviewers are for this customer | Chapter 05 |
| Change-advisory board integration | Vendor internal CAB | This customer's CAB — risk tier, approval cadence, freeze windows; how a flag flip is tied to an approved ticket | Chapter 03, Chapter 06 |
| Runbook | Vendor platform runbook | This pilot's playbook library — model-provider outage, retrieval index stale, judge pass-rate drop, PII leak suspected, cost-per-call anomaly, audit request | Chapter 06 |
| Admin dashboard | Platform observability dashboards | Customer-facing admin surface — SME-readable metrics, kill-switch controls per flag, flag-flip feed, replay panel, incident banner | Chapter 06 |
| SME training | Vendor product documentation | Layered curriculum for this customer's SMEs — orientation, workflow-specific, operator, governance — co-authored with `ai-enablement-lead` | Chapter 06 |
| SLA shape | Vendor master SLA | Per-pilot SLA — availability target, latency SLO, incident-response SLA, incident-communication SLA, mapped to this customer's regulatory floor | Chapter 07 |
| Cost model | Platform cost dashboard | Per-pilot envelope — per-call cost by model / retriever / judge; monthly projection with assumptions; alerting thresholds | Chapter 07 |
| Handoff packet | Vendor product collateral | Per-pilot pilot-to-production packet — architecture + runbook + SLA + cost model + eval dashboard + security-evidence index + contract pointer + feedback-loop pointer | Chapter 07 |
| Peer boundaries | Platform RACI | Per-pilot mapping to `ai-enablement-lead`, customer-side SRE, vendor-side release engineering; pager protocol | Chapter 08 |

Sixteen rows of overlay. Each chapter in this module takes one or two. The pattern holds from [mod-104 through mod-108](../): craft dimensions the peer specialists resolve once get re-opened by the FDE per customer, with gate dimensions enforced first.

## The organising budget: six dimensions

Mod-104 through mod-108 organised decisions around a six-dimensional budget. Mod-109 keeps the shape and re-labels the axes for deployment and rollout.

1. **Rollout safety.** Probability that a new prompt, model, retriever, tool allow-list, or judge reaches production without regressing a customer-visible outcome. Measured by canary-gate pass rate, rollback count per quarter, and eval-budget burn from the new release. A high rollout-safety score means candidate bundles almost always pass the canary; a low score means the team is firefighting rollbacks instead of shipping.
2. **Change-management legibility.** The customer's CAB, CISO office, model-risk function, and operations team can each read the release note and understand what changed, why, who approved it, and how to roll it back. Measured by CAB-approval first-pass rate, average release-note review time, and how often the customer's audit team can answer a question without paging the FDE.
3. **Replay fidelity.** Any past customer interaction can be re-run against any past artefact bundle, bit-for-bit, for audit, incident response, or regression debugging. Measured by replay-coverage of production invocations, replay-store retention vs the customer's regulatory requirement, and determinism gap for sampled model calls.
4. **Customer-side operability.** The customer-side admin can run the system without the vendor on the pager, through documented dashboards, runbooks, and training; failure modes have playbooks; escalation paths are named and tested. Measured by the "vendor-page rate" post-handoff and by SME time-to-competency.
5. **Deployment-mode fit.** The deployment mode satisfies the customer's data-residency, regulatory, update-cadence, and support-model constraints without creative interpretation. A HIPAA-covered workflow routed through a sub-processor without a BAA is not a fit; an air-gapped customer running a hosted frontier API is not a fit.
6. **Time-to-production.** Wall-clock from "pilot eval gate passed" (end of mod-107) to "all traffic on the production configuration with customer-side operations in steady-state." Rolls up security-review time (mod-108), CAB-approval time, canary expansion time, SME-training time, and signoff-ceremony time.

### Gate dimensions

**Deployment-mode fit, change-management legibility, and replay fidelity are gates.** A choice that fails any of them is off the table — rollout-safety or time-to-production advantages do not recover it.

Deployment-mode fit is a gate because a pilot that lands on the wrong mode cannot be patched later with flags or canaries. If the customer's regulation requires inference inside their VPC and the pilot is routing prompts through a SaaS endpoint, the problem is the architecture, not the release cadence. Chapter 02 draws the lines.

Change-management legibility is a gate because the customer's CAB *will not* approve a release it cannot read. A release note that says "upgraded retriever" without naming the version, the approval ticket, the rollback path, and the expected eval delta is a release that stalls at the CAB door — regardless of how safe the underlying change is. Chapter 03 (flag audit log) and Chapter 07 (release-note shape) make legibility architectural.

Replay fidelity is a gate because the customer's auditor, their incident-response team, and their regulator each at some point ask "walk me through why this specific interaction produced this specific answer." A system that cannot bit-for-bit replay a past interaction against its own bundle at the time cannot answer the question. Chapter 05 is where replay becomes a capability, not a promise.

### Optimisation dimensions

**Rollout safety, customer-side operability, and time-to-production are optimisations.** Among gate-passing designs, the FDE picks the point that best fits this customer's maturity, phase, and risk appetite.

Rollout safety is the dimension the vendor's product team cares about most — a team firefighting weekly rollbacks has no room for the next release. Chapter 04 is where safety is engineered via canary gates, shadow comparison, and blameless rollback.

Customer-side operability is the dimension the customer's operations lead cares about most — a dashboard the SME cannot read means the FDE is on the pager forever. Chapter 06 sets the discipline.

Time-to-production is the dimension the FDE's and sponsor's schedules care about most. Compressing it is a question of parallelising security review (mod-108) and CAB approval (this module) with canary expansion, not a question of cutting corners on either gate. Chapter 07 names the ceremony that locks the date.

## The four deployment archetypes, re-cast for rollout

Mod-104 Chapter 01 named three deployment-model archetypes and mod-108 Chapter 01 added the air-gapped case as a rare fourth. For rollout, the four archetypes shape four physical questions: *who owns the pipeline, what update cadence is realistic, how does the kill-switch reach production, how does a canary get wired, how does a replay fire*.

**SaaS multi-tenant.** The vendor owns the pipeline. Updates roll continuously — daily or weekly is normal. The kill-switch is a flag in the vendor's LaunchDarkly / Statsig / Unleash account, reachable from the vendor's control plane. Canary expansion runs over pooled multi-tenant traffic; the vendor's internal eval gate is the arbiter. Replay runs against the vendor's invocation log. Customer involvement in rollout decisions is low — mostly release notes and an opt-out mechanism. Latency envelope is internet-bound; model choice is whatever the vendor offers; support is vendor-paged.

**Dedicated SaaS.** The vendor still owns the pipeline, but the artefact lands in a customer-dedicated namespace. Update cadence is negotiated — weekly or biweekly is common, with named freeze windows. The kill-switch is per-tenant scoped; the customer's operations lead has read-access and sometimes flip-authority. Canaries run inside the customer-dedicated namespace; the eval gate must include customer-specific pass criteria. Replay runs against the dedicated namespace's invocation log, stored under customer-scoped retention. Latency is better (same region, same account); model choice includes hyperscaler-native under the customer's keys; support is vendor-paged with customer-side Tier 1.

**VPC install (Terraform / Helm shipped by the FDE).** The customer owns the pipeline — their CI/CD applies the Terraform / Helm charts the FDE ships. Update cadence is customer change-management-driven; monthly or quarterly with CAB approval is typical for regulated verticals. The kill-switch is a flag in the vendor's SDK consumed by the customer's runtime, or in the customer's own flag platform if they have one — either way, the customer's operations lead flips it. Canaries run in the customer's cluster against the customer's traffic; the eval gate runs in the customer's environment. Replay runs against the invocation log stored in the customer's cloud under customer-scoped retention. Latency envelope is excellent (same VPC, often same AZ); model choice is hyperscaler-native under customer keys; support is split — customer-side Tier 1/2, vendor FDE Tier 3.

**On-prem / air-gapped.** The customer owns the pipeline and the entire runtime. Update cadence is ATO-gated or regulatory-change-control-gated — quarterly is fast; annual is common. The kill-switch must be flippable without any dependency outside the air-gap — local flag state, local admin console, local audit log. Canaries run on a staged ring of the customer's own GPUs; no live traffic from external customers is involved. Replay runs entirely on-prem against a local invocation log. Latency envelope is excellent but constrained by local serving stack (vLLM, Hugging Face TGI, Nvidia NIM, llama.cpp); model choice is local open-weights only; support is customer-paged with vendor consultation.

Each archetype has distinct trade-offs on latency / model choice / update cadence / support model. Chapter 02 drills in.

## The three customer archetypes for the rest of the module

The same three archetypes used through mod-104 / 105 / 106 / 107 / 108, with the rollout posture filled in.

**Archetype A — US mid-size healthcare payer.** Workflow: member-inquiry resolution. Compliance: HIPAA + SOC 2 Type II. Deployment: AWS Bedrock + VPC install using Terraform + Helm shipped by the FDE, in us-east-1 and us-west-2 on EKS. Rollout posture: customer platform team runs the Helm chart; monthly release cadence with named CAB approval; LaunchDarkly tenant provided by the vendor with kill-switch read-flip authority delegated to the customer's operations lead; canary at 1% of live member-inquiry traffic with a customer-specific eval gate (judge pass-rate, resolved-ticket-rate, p95 latency, cost-per-call); replay retained 6 years per HIPAA records policy; handoff packet 25–50 pages. First go-live in month 4; steady-state by month 6. <!-- needs-research: confirm HIPAA records retention for medical records / audit logs — 45 CFR 164.316(b)(2) specifies 6 years, but the applicability to specific log classes varies -->

**Archetype B — US federal agency.** Workflow: policy-analyst drafting + evidence packaging. Compliance: FedRAMP Moderate (path to High), NIST 800-53 Rev 5, NIST AI RMF 1.0, OMB M-24-10 (and successor memos). Deployment: GovCloud + self-hosted inference on the agency's GPUs via a vLLM or Nvidia NIM stack; air-gapped update cadence governed by ATO change-control — quarterly nominal, with emergency patches via POA&M item. Rollout posture: agency platform team runs everything; flag platform is the agency's own if one exists, otherwise a minimal file-based flag store in-cluster; canary is offline-eval + staged ring deployment (no live frontier-customer traffic); replay retention per NARA records-retention plus the agency's own policy; handoff packet is the longest (ATO body-of-evidence appendix). First go-live is slower — month 6 to month 12 depending on ATO baseline. <!-- needs-research: confirm NARA records-retention general schedules relevant to AI system logs; current OMB AI memo numbering (M-24-10 and successors) -->

**Archetype C — EU retail bank.** Workflow: customer-support draft reply + case triage. Compliance: GDPR (Articles 22, 32, 35), DORA, EU AI Act (Article 14 oversight; Article 72 post-market monitoring; Annex III applicability depends on use). Deployment: Azure AI Foundry under the EU Data Boundary; dedicated SaaS tenancy with CMK on all storage; audit stream to Microsoft Sentinel. Rollout posture: bank runs CAB weekly; the vendor's LaunchDarkly tenant streams flag-flip events into Sentinel; canary at 1% of live case-triage traffic with an eval gate that includes an SR 11-7-equivalent model-risk re-test; replay retention per DORA + GDPR + bank-house policy (seven years typical for regulated financials); handoff packet tightly scoped to DORA Article 28 and EU AI Act Annex IV obligations. First go-live month 4; steady-state by month 7.

Each chapter runs decisions through these three archetypes. The archetypes are shapes, not prescriptions.

## Where mod-109 sits with the rest of the track

The deployment and rollout overlay only makes sense in context.

- **[Mod-101](../mod-101-fde-discipline-and-position/) (FDE discipline).** The escalation rule from Chapter 06 — know when to hand off to a peer — fires repeatedly in this module; Chapter 08 writes the specific pager protocol.
- **[Mod-102](../mod-102-customer-discovery-and-pilot-scoping/) (customer discovery).** The benefit-realisation KPIs the sponsor committed to at scoping are the KPIs the `ai-enablement-lead` tracks in Chapter 06.
- **[Mod-103](../mod-103-enterprise-integration/) (enterprise integration).** The identity surface, the data surface, and the NHI inventory are referenced in the admin-dashboard contents (Chapter 06) and the handoff-packet architecture diagram (Chapter 07).
- **[Mod-104](../mod-104-llm-app-in-customer-envs/) (LLM applications).** The gateway is where feature flags attach (Chapter 03) and where the shadow request-clone is wired (Chapter 04); the ADR template from Chapter 02 is the template the deployment-mode decision memo in exercise-01 uses; the cost/latency SLOs from Chapter 05 are the SLOs the canary gate (Chapter 04) and the handoff SLA (Chapter 07) commit to.
- **[Mod-105](../mod-105-grounding-on-customer-data/) (grounding).** The retriever version pinned in the artefact bundle (Chapter 05) is a retriever first defined in mod-105; the permission model and PII-vault surfaces show up on the admin dashboard.
- **[Mod-106](../mod-106-agentic-workflows-on-enterprise/) (agentic workflows).** The tool allow-list pinned in the artefact bundle (Chapter 05) is the allow-list first defined in mod-106; tool-allow-list drift is one of the playbooks (Chapter 06).
- **[Mod-107](../mod-107-eval-eng-on-customer-data/) (eval).** The pilot gold set (mod-107 Chapter 02), the online-eval judges (Chapter 05), and the per-pilot decision document (Chapter 06) are direct inputs to the canary gate (Chapter 04) and the handoff eval dashboard (Chapter 07); the exit-eval snapshot ends up in the handoff packet.
- **[Mod-108](../mod-108-enterprise-security-and-compliance/) (security and compliance).** The DPA (Chapter 05), the DFD (Chapter 02), the sub-processor list (Chapter 02), the threat-model matrix (Chapter 04), and the ML-BOM (Chapter 06) are the evidence the handoff packet (Chapter 07) indexes; the kill-switch audit log (Chapter 03) is a SOC 2 CC6 / HIPAA audit-control artefact.
- **Mod-110 (observability, on-call).** The admin dashboard specified here (Chapter 06), the escalation paths, and the pager protocol (Chapter 08) are the ones mod-110 instruments and pages against. The handoff SLA (Chapter 07) is the SLA mod-110 operates to.
- **Mod-111 (field-to-product loop).** The handoff packet's feedback-loop pointer (Chapter 07) is the pointer mod-111 reads from; reusable flag-registry templates, bundle manifests, and runbook fragments flow back to the product team.
- **Mod-112 (consulting craft and communication).** How the CAB-approval conversation, the handoff ceremony, and the quarterly business review are narrated to the customer's executive sponsor.

## Anti-patterns

**Shipping an LLM pilot like a static web service.** Rolling forward with blue-green + health-check + HTTP 5xx-based rollback and no eval gate. An LLM pilot regresses in ways that do not surface as 5xx — a drop in judge pass-rate, a spike in refusal-rate, a shift in output distribution — all of which look healthy to a generic release pipeline. Chapter 04 wires the LLM-specific canary gate; a generic release pipeline alone is a silent-regression generator.

**Feature flags as rollout, flags as kill-switch, flags as A/B all tangled together.** Treating one flag type as another — a rollout flag left live past rollout becomes a de-facto kill-switch; an experiment flag reused for rollout gets flipped by an experimentation stop-rule at an inconvenient moment. Chapter 03 names the taxonomy and the naming convention that keeps the three use-cases separate.

**Canary without an eval gate.** Routing 1% of traffic to the new arm, watching HTTP 5xx and latency, declaring victory. The LLM-specific regressions do not surface in those signals. Chapter 04 insists the canary gate reads the online eval, the judge agreement, the cost-per-call, and a human review sample — before expansion is allowed.

**Versioning the code, not the artefact bundle.** The Git commit for the vendor's service is pinned; the prompt is in a database; the retriever index is in a Pinecone collection whose version is implicit; the judge is another endpoint. Six months later an auditor asks "which prompt and which retriever answered this question?" and the audit trail is a reconstruction project. Chapter 05 is where the bundle — prompts + retriever config + model pin + tool allow-list + judge + guardrails + gateway config — is versioned as a single unit with explicit lineage.

**The customer-side operator as an afterthought.** The admin dashboard is a Grafana link no SME can read; the "playbook" is a Confluence page no one tested; the SME training is one brown-bag the week before launch. Chapter 06 sets the discipline; the admin dashboard is a *customer-side* surface, not a *vendor-side* surface made available.

**A handoff packet made of slides.** The handoff ceremony runs on a deck; six months later the deck is in someone's Google Drive and the actual runbook has never been touched. Chapter 07 versions the packet alongside the artefact bundle in the same manifest repo — a reader can `git diff v1.3..v1.4` on the packet.

**The FDE as the sole on-call after handoff.** The FDE's name is in the PagerDuty escalation policy for every alert; the customer's operations lead is a courtesy recipient. The pilot never actually hands off — it just adds another customer to the FDE's pager. Chapter 06 and Chapter 08 write the escalation tiers; mod-110 will enforce them from the observability side.

## Summary

- Mod-109 is overlay craft: the vendor release-engineering team, [`ai-enablement-lead`](https://github.com/ai-governance-curriculum/ai-enablement-lead-learning) (peer, level 35), and the customer's own release-engineering / SRE / change-management functions each own the base layer; the FDE assembles the per-pilot rollout overlay on top. <!-- needs-research: confirm ai-enablement-lead-learning repo URL before publishing -->
- Sixteen dimensions are re-opened per customer — from release pipeline through deployment mode, flag platform, canary strategy, replay harness, runbook, admin dashboard, SLA shape, cost model, and handoff packet. Each chapter in this module takes one or two.
- The organising budget is six-dimensional: rollout safety, change-management legibility, replay fidelity, customer-side operability, deployment-mode fit, time-to-production. Deployment-mode fit, change-management legibility, and replay fidelity are gates; the others are optimisations.
- Four deployment archetypes (SaaS multi-tenant, dedicated SaaS, VPC install, on-prem / air-gapped) shape who owns the pipeline, what cadence is realistic, how the kill-switch reaches production, how a canary is wired, and how a replay fires.
- Three customer archetypes — US healthcare payer (VPC install on AWS Bedrock + Terraform/Helm), US federal agency (GovCloud + self-hosted inference + air-gapped update cadence), EU retail bank (Azure AI Foundry EU Data Boundary + dedicated SaaS + Microsoft Sentinel audit stream) — thread through every chapter.
- Seven anti-patterns to catch early: static-web-service rollout, tangled flag types, canary without eval gate, code-not-bundle versioning, operator-as-afterthought, slides-as-handoff, FDE-as-sole-on-call.
- Chapter 02 is next: four deployment modes with per-environment trade-offs on latency, model choice, update cadence, and support model, plus the Terraform + Helm packaging discipline that makes a VPC install shippable.
