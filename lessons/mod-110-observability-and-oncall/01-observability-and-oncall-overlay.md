# Chapter 01 — The Observability and On-Call Overlay — Why an LLM Pilot Is Not a Generic Service to Watch

## Why this chapter exists

[Mod-109](../mod-109-deployment-rollout-change-management/) landed the pilot into production and handed a runbook, a feature-flag platform, a canary gate, and an admin dashboard to the customer-side operator. Mod-110 picks up the step that keeps the pilot *alive and legible* once traffic is flowing: the instrumentation that produces signal, the observability platform that renders it, the SLOs that decide "is this good enough to still be running?", the on-call rota that answers the pager, and the loop that turns every incident into a permanent regression fixture.

The module is deliberately an *overlay* on three in-house crafts, not a reimplementation of them. The vendor's platform-engineering / SRE team already runs centralised metrics / logs / traces, PagerDuty rotations, status pages, and post-mortem review boards at fleet scale. The customer's own SRE / platform-observability function already runs their SIEM, their APM (Datadog, Splunk, Elastic, Grafana Cloud, Dynatrace, New Relic), their incident-management tool (PagerDuty, Opsgenie, ServiceNow IR, FireHydrant, incident.io), and their on-call discipline inside their cloud. A peer role — [`ai-platform-engineer`](https://github.com/ai-engineering-curriculum/ai-platform-engineer-learning) at level 35, Infra / Platform family — runs the vendor's cross-fleet GenAI reliability programme: shared telemetry schema, shared judge services, shared cost rollups. <!-- needs-research: confirm the exact repo URL and family positioning for ai-platform-engineer-learning in the ai-engineering-curriculum org before publishing -->

Three crafts. Three peers. All of them operate against their own product, their own platform, their own enterprise. The FDE at level 35 re-opens parts of each one *per customer*, because every customer's operations posture is the customer's own: their SIEM, their APM contract, their pager rotation, their status-page tool, their regulatory retention floor, their language for "SEV-1". That is the overlay.

## The in-house crafts and what they cover

Name the three before adding the overlay.

**Vendor platform-engineering / SRE (in-house, Infra family)** owns, across every pilot the vendor runs:

- **Centralised telemetry at platform scale.** An OpenTelemetry Collector fleet, a managed backend (Datadog / Honeycomb / Grafana Cloud / an internal ClickHouse lake), fleet-wide alerting, runbook template catalogues.
- **The vendor's own on-call.** A PagerDuty / Opsgenie / incident.io rotation covering the vendor's control plane, the vendor's hosted endpoints, and the shared LLM-observability platform if the vendor hosts it.
- **Platform SLOs.** Availability and latency SLOs for the vendor's own API surface; error budgets governed by the vendor's reliability council.
- **Post-mortem machinery.** A blameless-post-mortem template, a review board, a corrective-action tracker (Jira / Linear), a quarterly reliability-trend report.

**`ai-platform-engineer` (peer, level 35, Infra / Platform family)** owns, specifically for the vendor's GenAI products:

- **Shared GenAI telemetry schema.** The common attribute set (model id, prompt-template id, judge id, cost fields, token counts) rolled up across every pilot so the vendor can reason about model-provider drift, cost per model family, and global failure correlations.
- **Shared judge services.** A central judge-as-LLM service ([mod-107 Chapter 05](../mod-107-eval-eng-on-customer-data/05-online-eval.md)) available to every pilot, instrumented to the shared schema.
- **Shared cost rollups.** Per-pilot, per-tenant, per-model cost dashboards sliced by the shared schema's labels.
- **Guardrail-health signals.** Platform-wide guardrail trip rates, false-positive estimation, and health dashboards.

**The customer's own SRE / observability / IR function** owns, inside the customer's cloud or data centre:

- **The SIEM / APM.** Datadog, Splunk, Elastic, Grafana Cloud, Dynatrace, New Relic, Microsoft Sentinel — whichever the customer has standardised on. The customer's security team reads the SIEM; the customer's SRE reads the APM; the customer's DPO reads both.
- **The pager.** A customer-side SRE rotation already paged for the pilot's host application; the LLM feature adds alerts to that rotation.
- **Incident response.** The customer's IR tool (PagerDuty / Opsgenie / ServiceNow IR / FireHydrant / incident.io / xMatters) and its SEV classification taxonomy, status-page tool (Statuspage, Statuspal, Instatus, a bespoke one), and the customer's own post-mortem review.
- **Retention and audit.** Log / trace / metric retention tuned to the customer's regime — HIPAA six-year audit trail for PHI-touching systems, DORA ICT-incident records, FedRAMP continuous monitoring, EU AI Act Article 72 post-market monitoring.

Three crafts, each one deep and complete inside its own scope. The FDE does not re-teach any of them. The FDE adds the per-customer overlay on top.

## The observability-and-on-call overlay

The overlay is the set of concerns the FDE adds on top of all three in-house crafts. Each row is a dimension the craft resolves once at platform / customer scale and the FDE re-opens per customer. Pattern holds from mod-104 onward.

| Dimension | What the in-house craft resolves once | What the FDE re-opens per customer | Where this module handles it |
|---|---|---|---|
| Telemetry schema | `ai-platform-engineer` ships a shared GenAI attribute set | Which attributes this customer's SIEM needs re-named, dropped, or supplemented to match their house taxonomy | Chapter 02 |
| Telemetry transport | Vendor OTel Collector fleet | Which collector topology lands inside the customer's VPC; where it exports; which attributes are stripped before export for data-contract reasons | Chapter 02 |
| Semantic conventions | OTel GenAI semconv at platform level | Which attribute versions this customer's APM platform currently parses natively (Datadog LLM Observability, Dynatrace, New Relic, Grafana) and which it needs manual mapping for | Chapter 02 |
| LLM observability platform | Vendor's Langfuse / LangSmith / Arize Phoenix / Weave / Braintrust tenant | Which platform satisfies this customer's residency + self-hosted + CMK + SSO posture; whether the vendor-hosted tenant is usable or a customer-hosted instance is required | Chapter 03 |
| Trace / prompt retention | Vendor retention policy | This customer's regulatory floor (HIPAA six years, DORA, FedRAMP, EU AI Act Article 72) mapped to retention settings for every store | Chapter 02, Chapter 03 |
| SLO targets | Vendor platform SLOs on vendor endpoints | Per-pilot quality / latency / cost / safety SLOs tied to the customer's business KPI from [mod-102](../mod-102-customer-discovery-and-pilot-scoping/) | Chapter 04 |
| Error budget policy | Vendor reliability council | This pilot's error-budget policy — who gets paged when it burns 50 %, 75 %, 100 %; who signs the "stop releases" letter | Chapter 04 |
| Alert routing | Vendor PagerDuty escalation | This customer's on-call tool, SEV taxonomy, overlap window with the FDE; who is Tier 1 / 2 / 3 | Chapter 05 |
| Incident command | Vendor IR process | Joint-command protocol between the customer's IR commander and the vendor FDE — Scribe, IC, Comms, SMEs | Chapter 05 |
| Status-page posture | Vendor status page | Whether the customer's own status page updates for the pilot; who writes the "impact" line; cadence of updates | Chapter 05 |
| LLM failure-mode triage | Vendor playbook catalogue | Per-pilot playbooks for prompt regression, provider outage, retrieval drift, hallucination spike, cost blowout, guardrail false-positive storm | Chapter 05 |
| Post-mortem review | Vendor review board | Joint review with the customer's operator; action-item ownership between the two orgs | Chapter 06 |
| Field-to-eval feedback loop | Vendor product-feedback triage | The pipe that converts every incident into a permanent regression fixture ([mod-107 Chapter 03](../mod-107-eval-eng-on-customer-data/03-offline-eval-design.md)) and every customer bug report into a labelled gold-set entry ([mod-107 Chapter 02](../mod-107-eval-eng-on-customer-data/02-customer-gold-set-bootstrap.md)) | Chapter 06 |
| Dashboards | Vendor platform dashboards | Which dashboards the customer-side operator opens *daily* — intent is different from the vendor's fleet view | Chapter 03, Chapter 05 |

Fourteen rows of overlay. Each chapter in this module takes two or three.

## The organising budget: six dimensions

Mod-104 through mod-109 used a six-dimensional budget. Mod-110 keeps the shape and re-labels the axes for observability and on-call.

1. **Signal fidelity.** Every production invocation emits traces, metrics, and logs with the attributes needed to answer "which prompt-template, which model, which retriever, which judge, which guardrail produced this output at what cost and what latency on whose behalf?" Measured by trace-attribute completeness vs the OTel GenAI semantic-convention target, sampling coverage, and the time-to-reconstruct a specific invocation.
2. **Operator legibility.** The customer-side operations lead, the CISO office, the DPO, and the model-risk reviewer can each open the dashboard and read the state of the pilot *without the FDE*. Measured by SME time-to-insight on a cold open, the FDE's "dashboard escalation rate" (how often the customer asks the FDE to interpret a graph), and the alignment of dashboard metric names with the customer's house taxonomy.
3. **Retention sufficiency.** Every store — traces, prompts / completions, judge calls, cost events, incident records — retains data long enough to satisfy the customer's strictest regulatory floor and no longer. HIPAA six years, DORA ICT register, FedRAMP continuous monitoring, EU AI Act Article 72 post-market monitoring, GDPR storage-limitation each push the number in a different direction.
4. **SLO truthfulness.** The SLOs committed in the handoff SLA ([mod-109 Chapter 06](../mod-109-deployment-rollout-change-management/06-customer-side-handoff-and-sme-training.md)) are measured by the same instrumentation that pages the on-call. "The SLA says 99 % availability but we measure availability from the vendor side only" is a failure of truthfulness. Measured by SLO-to-instrumentation gap and by the number of SLOs whose definition pointer resolves to a working query.
5. **Pager appropriateness.** When the pager fires, it fires because a user-visible problem is real; when the pager is quiet, the pilot is actually healthy. False-positive pages exhaust the on-call; missed pages shred customer trust. Measured by actionable-page rate (pages where an operator took action), the time-to-ack on real pages, and the ratio of alerts to incidents.
6. **Feedback-loop closure.** Every incident produces at least one permanent regression fixture that would have caught it; every customer bug report produces at least one labelled gold-set entry. Measured by fixture-to-incident ratio, the number of production incidents reproduced inside the eval gate, and the backlog of unprocessed bug reports.

### Gate dimensions

**Signal fidelity, retention sufficiency, and SLO truthfulness are gates.** A choice that fails any of them is off the table — pager-appropriateness or feedback-loop improvements do not recover it.

Signal fidelity is a gate because an on-call who cannot answer "which prompt-template produced this output?" cannot debug. An LLM incident that bottoms out in "we have the HTTP log but not the prompt" is an incident the team cannot close. Chapter 02 is where every span, every metric, and every log line gets the attribute set that makes the invocation reconstructible.

Retention sufficiency is a gate because an auditor, a regulator, or a plaintiff will one day ask "walk me through the model's behaviour on this specific case from fourteen months ago," and a system whose prompt log rolled off at ninety days cannot answer. The retention decision is architectural — adding six years of prompt storage after the fact is not a retrofit, it is a rebuild. Chapter 02 and Chapter 03 set the floors.

SLO truthfulness is a gate because an untrue SLO is a contract trap. Promising 99.5 % availability without a measurement that reflects the customer's experience of availability — not the vendor's internal API success rate — is the fastest way to a mid-pilot escalation. Chapter 04 insists every SLO points to a query that reads the production telemetry the user's path depends on.

### Optimisation dimensions

**Operator legibility, pager appropriateness, and feedback-loop closure are optimisations.** Among gate-passing designs, the FDE picks the point that best fits this customer's maturity, phase, and risk appetite.

Operator legibility is the dimension the customer's operator cares about most. A handoff-day dashboard full of vendor-language names and vendor-internal IDs is a dashboard the operator will stop opening in week three. Chapter 03 writes the discipline; Chapter 05's playbooks insist on operator-side language.

Pager appropriateness is the dimension the on-call rotation cares about most. The on-call who gets paged three times a night for one real issue develops the "glance and dismiss" habit that kills the next real page. Chapter 04 wires SLO-driven alerting; Chapter 05 writes the SEV classification that filters noise from signal.

Feedback-loop closure is the dimension the vendor product team cares about most, through mod-111. An incident that teaches the team nothing is a tax; an incident that lands a permanent regression fixture is a payment on the quality-over-time curve. Chapter 06 is where the pipe is built.

## Three customer archetypes for the module

The same three archetypes used through mod-104 / 105 / 106 / 107 / 108 / 109, with the observability-and-on-call posture filled in.

**Archetype A — US mid-size healthcare payer.** Workflow: member-inquiry resolution. Compliance: HIPAA + SOC 2 Type II. Deployment: AWS Bedrock + VPC install using Terraform + Helm shipped by the FDE, in us-east-1 and us-west-2 on EKS. Observability posture: customer-side Datadog for APM and logs, with the OpenTelemetry Collector deployed as a DaemonSet inside the pilot's EKS cluster, exporting to the customer's Datadog account under the customer's intake API key. LLM observability platform: Langfuse self-hosted in the customer's VPC (CMK on Postgres and S3, SSO via the customer's Okta tenant, six-year retention on all traces and completions for HIPAA records policy). <!-- needs-research: confirm HIPAA records retention for medical records / audit logs — 45 CFR 164.316(b)(2) specifies 6 years for Security Rule documentation; applicability to specific log classes is reviewed per-pilot --> On-call: customer Tier 1 operations analyst (business hours), customer Tier 2 SRE (24×7 via PagerDuty), vendor FDE on Tier 3 under the handoff SLA. SEV taxonomy: the customer's internal 1–4 scale (SEV-1 = patient-facing outage). Status page: customer's existing Statuspage tenant adds a "AI Member Assist" component. Monthly SLO review with the customer's operations lead.

**Archetype B — US federal agency.** Workflow: policy-analyst drafting + evidence packaging. Compliance: FedRAMP Moderate (path to High), NIST 800-53 Rev 5, NIST AI RMF 1.0, OMB M-24-10 (and successor memos). Deployment: GovCloud + self-hosted inference on the agency's GPUs via a vLLM or Nvidia NIM stack; air-gapped update cadence governed by ATO change-control. Observability posture: agency's Splunk Enterprise instance running in GovCloud receives logs via the OTel Collector; traces and metrics go to a local Grafana / Mimir / Tempo stack inside the agency's enclave (no SaaS egress). LLM observability platform: self-hosted Arize Phoenix inside the enclave, with OIDC via the agency's IdP. <!-- needs-research: confirm Arize Phoenix on-prem deployment support and FedRAMP applicability of the hosted variant --> On-call: agency-side 24×7 operations watch (SOC-adjacent) on Tier 1 and Tier 2; vendor FDE on Tier 3 with consultation-only authority (the agency's SOC retains all decision-rights). SEV taxonomy: NIST 800-61 Rev 2 incident categories adapted to AI (Chapter 05 writes the mapping). <!-- needs-research: confirm NIST SP 800-61 is at Rev 2 and whether Rev 3 is current at time of publication --> No public status page; internal communications only. Quarterly SLO review aligned with continuous-monitoring cadence.

**Archetype C — EU retail bank.** Workflow: customer-support draft reply + case triage. Compliance: GDPR (Articles 22, 32, 35), DORA, EU AI Act (Article 14 oversight; Article 72 post-market monitoring; Annex III applicability depends on use). Deployment: Azure AI Foundry under the EU Data Boundary; dedicated SaaS tenancy with CMK on all storage; audit stream to Microsoft Sentinel. Observability posture: Azure Monitor + Application Insights + Sentinel natively consume the OTel OTLP exporter; the Sentinel connector reads the GenAI semconv attributes directly. LLM observability platform: Langfuse self-hosted in the bank's Azure tenancy under their EU Data Boundary policy, with CMK (Azure Key Vault), Entra ID SSO, seven-year retention for regulated financials. <!-- needs-research: confirm Langfuse support for EU-only deployment with CMK on Azure; verify DORA ICT-incident retention requirement is seven years for the bank's jurisdiction --> On-call: bank's Tier 1 operations centre (24×7), Tier 2 bank-side SRE, Tier 3 vendor FDE. SEV taxonomy: the bank's major-incident process aligned with DORA major-ICT-related-incident classification. <!-- needs-research: confirm current DORA RTS on classification of major ICT-related incidents thresholds --> Status page: bank-internal only. Weekly SLO review with the bank's operations and model-risk leads; EU AI Act Article 72 post-market-monitoring record updated monthly.

Each chapter runs decisions through these three archetypes.

## Where mod-110 sits with the rest of the track

The observability-and-on-call overlay only makes sense in context.

- **[Mod-101](../mod-101-fde-discipline-and-position/)** — the escalation discipline is the discipline that writes the pager protocol here.
- **[Mod-102](../mod-102-customer-discovery-and-pilot-scoping/)** — the business KPIs the sponsor committed to at scoping are the KPIs the SLOs in Chapter 04 attach to.
- **[Mod-103](../mod-103-enterprise-integration/)** — the identity surface is how SSO reaches the LLM-observability platform; the NHI inventory is what a credential-leak playbook touches.
- **[Mod-104](../mod-104-llm-app-in-customer-envs/)** — the gateway is where OTel instrumentation attaches; the per-customer cost / latency SLOs from [Chapter 05](../mod-104-llm-app-in-customer-envs/05-per-customer-cost-latency-slos.md) feed the SLO targets in Chapter 04 of this module; the ADR template from Chapter 02 is the template the observability-platform choice uses.
- **[Mod-105](../mod-105-grounding-on-customer-data/)** — retriever drift and index staleness are failure modes Chapter 05 pages against; the retrieval traces are where Chapter 02's attribute set captures `k`, index name, rerank score, document IDs.
- **[Mod-106](../mod-106-agentic-workflows-on-enterprise/)** — tool-call spans and human-in-the-loop checkpoint spans ([mod-106 Chapter 04](../mod-106-agentic-workflows-on-enterprise/04-human-in-the-loop-checkpoints.md)) are first-class spans in Chapter 02's schema.
- **[Mod-107](../mod-107-eval-eng-on-customer-data/)** — the pilot gold set, the online-eval judges, and the per-pilot decision document are the inputs to the SLO quality dimension in Chapter 04; the regression-fixture back-feed from Chapter 06 of this module lands in [mod-107 Chapter 03](../mod-107-eval-eng-on-customer-data/03-offline-eval-design.md).
- **[Mod-108](../mod-108-enterprise-security-and-compliance/)** — the data contract from [Chapter 05](../mod-108-enterprise-security-and-compliance/05-data-contract.md) names the PII / PHI handling rules that Chapter 02 of this module enforces through collector-level redaction; the audit stream is the Chapter 02 log destination.
- **[Mod-109](../mod-109-deployment-rollout-change-management/)** — the handoff SLA ([mod-109 Chapter 06](../mod-109-deployment-rollout-change-management/06-customer-side-handoff-and-sme-training.md)) is the SLA the SLOs in Chapter 04 operate against; the admin dashboard from mod-109 Chapter 06 is where the daily operator view lives; the kill-switch audit log from [mod-109 Chapter 03](../mod-109-deployment-rollout-change-management/03-feature-flag-strategy.md) is a span attribute in Chapter 02's schema; the canary gate from [mod-109 Chapter 04](../mod-109-deployment-rollout-change-management/04-canary-shadow-ab-rollouts.md) reads the same metrics Chapter 04 of this module defines.
- **Mod-111 (field-to-product loop)** — the regression-fixture pipe from Chapter 06 of this module is the input into mod-111; reusable playbook fragments, alert templates, and judge improvements flow back to the product team.
- **Mod-112 (consulting craft and communication)** — how the post-mortem is narrated to the executive sponsor, how the quarterly SLO review is prepared.

## Anti-patterns

**Watching an LLM like a web service.** Dashboards show HTTP 2xx / 4xx / 5xx, p95 request latency, and container CPU. None of these signals catch a prompt regression, a retrieval drift, a hallucination spike, or a cost blowout. The customer's generic APM default dashboards are necessary but far from sufficient. Chapter 02 and Chapter 03 write the LLM-specific surface.

**Vendor-side logs the customer cannot see.** The invocation log lives in the vendor's cloud; the customer's SIEM does not receive it. When an incident fires, the customer's SOC cannot answer their own CISO — they have to open a ticket with the vendor. The data-contract from [mod-108 Chapter 05](../mod-108-enterprise-security-and-compliance/05-data-contract.md) typically requires customer-visible logs; Chapter 02 wires the export.

**Raw prompts and completions in the APM.** PII, PHI, PCI, confidential data pumped into Datadog logs with default retention and default access controls. One field-level mistake breaches the data contract. Chapter 02 writes the collector-level redaction pipeline that precedes any customer-visible destination.

**SLOs copied from the vendor's master SLA.** 99.5 % availability promised with no definition of "available," no measurement query, no error-budget policy, no page on burn. The SLO exists only on a slide. Chapter 04 insists every SLO ties to an instrumented query, a numeric target, an error-budget policy, and an alerting rule.

**The pager points only at the FDE.** The customer's operations lead is a courtesy recipient; every page escalates immediately to the FDE. The pilot never hands off — it just adds the FDE to the customer's rotation forever. Chapter 05 writes the escalation tiers that force customer-side Tier 1 / 2 ownership.

**Post-mortems without eval back-feed.** The team debriefs, writes a doc, assigns action items to "add a dashboard," closes the ticket. Six months later the same failure recurs. Chapter 06 insists every post-mortem produces at least one regression fixture and at least one judge-rubric update — the eval gate is the memory, not the doc.

**Status-page updates after the incident is over.** The customer's end-users feel the outage in real time; the status page updates when the all-clear goes out. The customer's support team fields calls they cannot answer. Chapter 05 writes the update cadence and the template that forces updates at acknowledgement, every thirty minutes during, and at resolution.

**Alerts with no runbook link.** The page fires at 3 am. The on-call opens the alert, sees "LLM judge pass-rate dropped below 0.75," and has no idea what to do. Every alert in Chapter 05 ships with a runbook link that is tested in the drill ([Chapter 05's drill cadence](./05-oncall-rota-and-llm-triage.md)) before the alert goes live.

## Summary

- Mod-110 is overlay craft: the vendor's platform SRE, [`ai-platform-engineer`](https://github.com/ai-engineering-curriculum/ai-platform-engineer-learning) (peer, level 35), and the customer's own SRE / observability / IR functions each own the base layer; the FDE assembles the per-pilot observability-and-on-call overlay on top. <!-- needs-research: confirm ai-platform-engineer-learning repo URL before publishing -->
- Fourteen dimensions are re-opened per customer — from telemetry schema through LLM-observability platform, retention, SLOs, error budget, alert routing, incident command, failure-mode triage, post-mortem review, and eval back-feed. Each chapter in this module takes two or three.
- The organising budget is six-dimensional: signal fidelity, operator legibility, retention sufficiency, SLO truthfulness, pager appropriateness, feedback-loop closure. Signal fidelity, retention sufficiency, and SLO truthfulness are gates; the rest are optimisations.
- Three customer archetypes — US healthcare payer (Datadog + Langfuse self-hosted, HIPAA six-year retention), US federal agency (Splunk + self-hosted Phoenix in GovCloud, air-gapped), EU retail bank (Sentinel + Langfuse in Azure under EU Data Boundary, DORA / EU AI Act Article 72) — thread through every chapter.
- Eight anti-patterns to catch early: web-service watching, vendor-only logs, raw prompts in APM, slide-ware SLOs, FDE-only pager, post-mortems without back-feed, late status-page updates, alerts without runbooks.
- Chapter 02 is next: OpenTelemetry instrumentation with GenAI semantic conventions — the attribute set, the collector topology, the redaction pipeline, and the export fan-out to the customer's SIEM / APM.
