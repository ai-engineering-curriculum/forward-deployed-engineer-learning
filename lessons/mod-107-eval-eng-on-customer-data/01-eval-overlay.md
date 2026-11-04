# Chapter 01 — The Eval Overlay — Why Enterprise Evals Are Not Generic Evals

## Why this chapter exists

Mod-104 landed the LLM layer of a pilot inside the customer's environment. Mod-105 landed grounding on the customer's data. Mod-106 landed action — tools, HITL, permissioning, OWASP Agentic mitigations. Mod-107 picks up the next question, and it is the one the customer's governance board actually cares about: *is any of this good enough to go live, and good enough to prove it to the board that has to sign?*

Eval is where the pilot collides with the customer's assurance posture. Up to mod-106 the FDE has been building. From mod-107 onward the FDE is building *and arguing a case* — to the SME who has to approve the outputs, to the risk team that has to accept the model, to the regulator who may be waiting behind both of them. A pilot that cannot carry its own evidence does not ship.

The eval craft itself — the metric portfolios, the judge-LLM calibration, the statistical rigor of pass bars, the harness internals — is a specialist depth owned by [`ai-eval-engineer`](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning) (peer, level 30, AI Engineering family). The release-assurance packaging — model cards, risk registers, audit-trail format, regulator-facing evidence packets — is owned by [`ai-evaluation-engineer`](https://github.com/ai-engineering-curriculum/ai-evaluation-engineer-learning) (peer, level 35, AI Governance family). This module does not re-teach either; it layers the customer-environment overlay on top of both. Chapter 07 writes both boundaries explicitly.

## The in-house eval craft and what it covers

Two peers, two craft areas, both deep. Name them before adding the overlay.

**`ai-eval-engineer` (level 30, AI Engineering family)** owns, inside their own product or on a generic workflow:

- **Metric portfolios.** Task-level correctness, pairwise preference, factuality and groundedness, citation accuracy, format and schema compliance, safety-category metrics. Composition rules: one portfolio per workflow shape, not per product.
- **Judge-as-LLM calibration.** Writing judge prompts, measuring judge-vs-human agreement (Cohen's / Fleiss' kappa), detecting judge drift across model versions, A/B-ing judge-model choice, bias audits (position bias, verbosity bias, self-preference).
- **Statistical rigor.** Sample size for a pass bar, confidence intervals on a metric, bootstrap on small gold sets, power analysis before claiming a regression, significance of a delta between two prompt revisions.
- **Agent-trajectory eval.** Task success rate, trajectory grading, tool-call-correctness, cost-per-completed-task, intervention rate — the metric shapes that only make sense once the system has verbs.
- **Benchmark construction.** Gold-set design, stratified sampling, double-labelling, inter-rater reliability, versioning, benchmark leakage detection.
- **Harness internals.** Promptfoo, RAGAS, DeepEval, OpenAI Evals, UK AISI Inspect — their test-case schemas, their assertion grammars, their parallelism and caching behaviour.

**`ai-evaluation-engineer` (level 35, AI Governance family)** owns, inside their own product or at a vendor's AI office:

- **Release-assurance packets.** The bundle a release goes through before shipping — risk classification, mitigation inventory, residual-risk statement, sign-offs, dated evidence.
- **Model cards.** [Model Cards for Model Reporting](https://arxiv.org/abs/1810.03547) as the common shape; NIST AI RMF–aligned sections; sector-specific extensions (SR 11-7 for banking, FDA SaMD for medical devices where applicable).
- **Risk registers.** Named risks, likelihood, impact, mitigations, owners, review cadence, link into the enterprise risk register.
- **Regulator-facing evidence.** The form in which eval results, HITL logs, drift logs, incident timelines get handed to an auditor — chain of custody, immutability, sign-off trail.
- **Audit-trail format.** What fields every eval record carries, retention period, access control, exportability for subpoena.
- **Governance-board ritual.** The meeting structure, the artefacts, the vocabulary a board actually accepts as "go live" evidence.

Both areas are craft. Both peers resolve those craft dimensions *once* inside their own practice. The FDE at level 35 re-opens every one of them per customer, because every customer scores them against their own regime, their own board, their own data boundary. That is the overlay.

## The eval overlay

The eval overlay is the set of concerns the FDE adds on top of both peers' crafts. Each row is a dimension the in-house craft resolves once and the FDE re-opens per customer.

| Dimension | What the in-house craft resolves once | What the FDE re-opens per customer | Where this module handles it |
|---|---|---|---|
| Gold-set provenance | Internal labelers or vendor crowd; one shared gold set per workflow | Customer SMEs, customer-case stratification, double-labelling under the customer's NDA, versioned under the customer's retention policy | Chapter 02 |
| Judge-prompt localisation | One judge prompt per metric, English, vendor-house style | Rewritten against the customer's tone, terminology, jurisdictional idioms, and acceptable-refusal shape; re-calibrated per deployment | Chapter 03, Chapter 04 |
| Metric portfolio composition | Standard portfolio per workflow shape | Composed against the *customer's* failure taxonomy — a payer cares about benefit-terminology accuracy, a bank cares about mis-selling risk | Chapter 03 |
| Pass-bar co-sign | Engineering head sets the bar | Customer's workflow owner co-signs the bar with the FDE; risk team signs residual-risk; both are dated artefacts | Chapter 03, Chapter 06 |
| Business-KPI mapping | Metrics stand alone | Every metric wired to one or more business KPIs the pilot promised (mod-102 Chapter 03) — AHT, deflection, error rate, cycle time | Chapter 03 |
| Residency of eval data | Vendor's cloud, vendor's region | Customer's VPC, customer's region, customer's retention; judge-LLM calls routed through the customer's gateway (mod-104 Chapter 03) | Chapter 04 |
| Auditable sign-off | Version-tag on the model | Signed, dated artefact per release, exportable to the customer's GRC tool, retained under the customer's regime | Chapter 04, Chapter 06 |
| Per-tenant drift | Global drift dashboard | Drift measured and alerted *per customer* because each customer's distribution moves independently; thresholds co-owned with the customer | Chapter 05 |
| Online-eval sample posture | Random sample of production traffic | Sample rate, consent posture, PII handling, cross-border routing all negotiated per customer; some customers forbid vendor-side sampling entirely | Chapter 05 |
| Rubric and rater pool | One rubric, internal rater pool | Rubric written with the customer's SMEs; rater pool is the customer's SMEs; inter-rater reliability measured on *them* | Chapter 02, Chapter 06 |
| Exit-eval rite | Ship when metrics clear bar | Pilot-to-production exit eval — pairwise human-vs-AI on a holdout slice, SME scoring, business-KPI uplift, governance-board decision document | Chapter 06 |
| Release-assurance packet shape | Vendor's template | Customer's template — model card in the customer's format, risk register in the customer's GRC, evidence packet in the auditor's expected schema | Chapter 04, Chapter 06, Chapter 07 |

Twelve rows of overlay — matching the shape of mod-104, mod-105, and mod-106. The pattern holds: craft dimensions the peer specialists resolve once get re-opened by the FDE per customer, with gate dimensions enforced first.

## The organising budget: six dimensions

Mod-104 through mod-106 all organised decisions around a six-dimensional budget. Mod-107 keeps the shape and re-labels the axes for eval.

1. **Fidelity** — how well does the eval reflect the actual workflow? A high-fidelity eval uses customer-shape data, customer SMEs, customer failure modes. A low-fidelity eval uses a generic benchmark the model has probably seen in pretraining.
2. **Business-KPI coverage** — how much of the pilot's promised business impact (mod-102 Chapter 03) does the eval actually measure? An eval that moves a technical metric but leaves AHT, deflection, or error rate unmeasured is not evidence a board accepts.
3. **Privacy posture** — can the eval run end-to-end without customer data leaving the customer's trust boundary? Judge-LLM calls, label storage, dashboarding, drift logs each have a residency answer.
4. **Cost per eval run** — token cost of judge-LLM calls, SME-hours cost of rater passes, infrastructure cost of the harness, re-run cost per PR gate. Eval is cheap only until it is gated to CI.
5. **Time-to-sign-off** — wall-clock from "prompt revision proposed" to "customer governance board signs off." This includes the SME review loop, the risk-team review loop, and the packet-assembly step.
6. **Drift resilience** — does the eval catch a quality regression after it ships? An offline-only eval goes stale the day the customer's own traffic shifts.

### Gate dimensions

**Fidelity, privacy posture, and time-to-sign-off are gates.** A choice that fails any of them is off the table — cost or drift advantages do not recover it.

Fidelity is a gate because a low-fidelity eval does not earn board trust no matter how clean its numbers are. Chapter 02 (gold set) and Chapter 03 (metric portfolio + judge prompts) are the chapters where fidelity is bought.

Privacy posture is a gate because the moment customer data leaves the customer's trust boundary — through a judge-LLM call to a public endpoint, through a hosted eval dashboard, through a drift-log upload — the pilot becomes a data-processing-agreement issue, not an eval issue. Chapter 04 and Chapter 05 handle this.

Time-to-sign-off is a gate because an eval cycle that takes six weeks blocks every rollout gate in mod-109. A customer whose release rituals require a four-week SME review and a two-week risk-team review has already spent the project's weekly-release budget. Chapter 06 names the shape of a sign-off ritual that completes inside the cadence the pilot promised.

### Optimisation dimensions

**Business-KPI coverage, cost per eval run, and drift resilience are optimisations.** Among gate-passing designs, the FDE picks the point that best fits the customer's workflow and the business the pilot is accountable to.

Business-KPI coverage is the dimension the sponsor cares about most. Chapter 03 wires it into the portfolio.

Cost per eval run is the dimension the FDE's own budget cares about most. Judge-LLM calls on every PR for a 5,000-item gold set at a frontier model gets expensive quickly; Chapter 04 shows how to tier the harness so the CI gate uses a tight slice and nightly / weekly runs expand coverage.

Drift resilience is the dimension the customer's on-call team cares about most. Chapter 05 is where the online-eval harness and the drift signals get specified.

## The three deployment-model archetypes, re-cast for eval

Mod-104 Chapter 01 named three deployment-model archetypes and every subsequent module inherited them. For eval the archetypes shape three physical questions: where does the *gold set* live, where does the *judge-LLM call* run, where does the *online-eval sample* end up.

**SaaS multi-tenant.** The gold set lives in the vendor's SaaS, logically partitioned per tenant. The judge-LLM runs through the vendor's eval harness against the vendor's chosen model endpoint. The online-eval sample flows into the vendor's telemetry pipeline and is surfaced to the customer through a per-tenant dashboard. Residency is the vendor's region policy; some customers reject this entire archetype for eval even when they accept it for inference.

*Who owns what:* vendor owns the eval harness, the judge-LLM budget, the dashboard. Customer owns the SME time to label the gold set and co-sign the pass bar.

**Dedicated SaaS.** Gold set lives in a customer-dedicated namespace in the vendor's cloud. The judge-LLM runs through a customer-dedicated gateway with audit streamed to the customer's SIEM. The online-eval sample stays in the customer-dedicated bucket. This is the most common landing spot for a mid-maturity regulated customer: they accept vendor operations but require per-tenant isolation at the storage and audit layers.

*Who owns what:* vendor owns the running harness and the dashboard template; customer-dedicated storage, audit stream, and sometimes customer-dedicated judge-LLM accounts.

**VPC install.** Gold set lives in the customer's own data store — Snowflake schema, S3 bucket, Postgres table — under the customer's retention and ACLs. The judge-LLM runs through the inference gateway already deployed inside the customer's VPC (mod-104 Chapter 03), with no outbound call to a public model endpoint. The online-eval sample never leaves the customer boundary; dashboards are rendered from the customer's observability stack (mod-110). This archetype is the only one that composes cleanly with FedRAMP High and with EU Data Boundary-strict deployments.

*Who owns what:* customer owns the running harness at the infra layer. Vendor owns the eval-shaped code — the metric definitions, the judge prompts, the harness configuration, the upgrade cadence. On-call is shared.

**On-prem / air-gapped.** As with LLM serving and retrieval, rare but real. The entire eval stack — harness, judge-LLM, dashboards, drift logs — runs inside the customer's trust boundary with no network egress. Open-weights judge models (Chapter 03) are the usual landing spot for the judge-LLM; the harness runs on the customer's Kubernetes. Online eval runs against the customer's own traffic without any sample leaving the boundary.

The distinctive emphasis for eval: *the eval artefacts are themselves regulated data*. Gold-set entries contain customer cases; judge-LLM outputs are opinions about customer data; drift logs trace customer traffic. Every deployment archetype has to answer "where does each eval artefact physically rest, who can read it, and how long does it live?" Chapter 04 makes this precise.

## Three customer archetypes for the rest of the module

The same three archetypes used in mod-104, mod-105, and mod-106, with the eval surface filled in. Each sentence frames the archetype's eval posture.

**Archetype A — Mid-size US healthcare payer.** Workflow: member-inquiry resolution (CSR tier 1 / tier 2 / supervisor). Compliance: HIPAA + SOC 2 Type II. Deployment: AWS Bedrock + VPC install. Eval posture: gold set inside the customer's AWS account under HIPAA controls, judge-LLM runs through the Bedrock endpoint the gateway already brokers, SMEs are tier-2 CSRs and clinical ops reviewers, and the exit eval requires a HITL-approver sign-off on a holdout slice before auto-send is enabled for any benefit category.

**Archetype B — US federal agency.** Workflow: policy-analyst drafting + evidence packaging. Compliance: FedRAMP Moderate with a documented path to High. Deployment: GovCloud + self-hosted inference. Eval posture: gold set inside GovCloud with FIPS-validated storage, judge-LLM is an open-weights model on the customer's GPUs (no public model endpoint), SMEs are senior analysts and FOIA-cleared reviewers, and the release-assurance packet is sized to drop into the agency's existing ATO evidence bundle — NIST AI RMF cross-references, SSP-aligned control statements, dated sign-offs from the AO.

**Archetype C — EU retail bank.** Workflow: customer-support draft reply + case triage. Compliance: GDPR + DORA; EU AI Act Article 14 (human oversight) applies if the workflow is classified high-risk; SR 11-7 model-risk-management discipline applies through the bank's model-risk function. Deployment: Azure AI Foundry under EU Data Boundary. Eval posture: gold set in an EU region with GDPR right-to-erasure wired through, judge-LLM calls stay inside the EU Data Boundary, pairwise human-vs-AI eval is the shape the model-risk committee expects, and the release-assurance packet carries both an EU AI Act Article 14 oversight statement and an SR 11-7–style model-risk packet so the second-line function signs off before the first-line goes live.

Each chapter runs decisions through these three archetypes. The archetypes are shapes, not prescriptions.

## Where mod-107 sits with the rest of the track

The eval overlay only makes sense in context.

- **Mod-102 (customer discovery and pilot scoping)** — the business KPIs the pilot promised (Chapter 03) are the targets every metric in mod-107's portfolio has to roll up to.
- **Mod-103 (enterprise integration)** — the customer's data surfaces are where gold-set cases come from; the identity surface is where SME labelling-UI access is gated.
- **Mod-104 (LLM applications)** — the gateway (Chapter 03) is the choke point for judge-LLM calls; the per-tenant prompt architecture (Chapter 04) is what gets evaluated.
- **Mod-105 (grounding)** — the retrieval gold set bootstrapped in mod-105 Chapter 06 is the seed for mod-107 Chapter 02's full gold set; retrieval metrics (recall, nDCG, citation accuracy) compose into the portfolio in Chapter 03.
- **Mod-106 (agentic workflows)** — agent-trajectory eval, intervention rate, tool-call correctness, and cost-per-completed-task are agent-specific metric shapes mod-107 adds to the portfolio.
- **Mod-108 (enterprise security and compliance)** — the formal review of the eval harness choice (Chapter 04), the judge-LLM data-flow, and the release-assurance packet lives here in draft and gets formalised in mod-108.
- **Mod-109 (deployment, rollout)** — the eval gate is the thing that blocks a feature-flag flip; the exit eval (Chapter 06) is the thing that decides whether the pilot becomes production.
- **Mod-110 (observability, on-call)** — online-eval signals (Chapter 05) are the production signals that fire pages; drift alerts flow through the customer's on-call.
- **Mod-111 (field → product loop)** — reusable gold-set shapes, judge prompts, pass-bar templates flow back to the product team.
- **Mod-112 (consulting craft)** — how the exit-eval result is narrated to the customer's executive sponsor; how a quality regression is communicated.

## Anti-patterns

**Generic benchmark theatre.** Reporting MMLU, HELM, or any public leaderboard score to the customer's governance board. The score is uninformative — the model has seen those items, the distribution has nothing to do with the customer's cases, and the board will (correctly) ask what any of it says about *their* workflow. Chapter 02 is explicit: the only eval that counts is the one bootstrapped on the customer's own case distribution with the customer's own SMEs. Benchmarks are useful for comparative model selection *before* the pilot gold set exists; they are never evidence of fitness to deploy.

**A dashboard with no sign-off ritual.** Standing up a quality dashboard, wiring it into Grafana or the vendor's console, and calling that "eval." A dashboard shows numbers; it does not produce decisions. Without a dated, signed, co-authored pass-bar document — this metric, this threshold, this sample size, these signatures — a quality regression has no owner and no action. Chapter 06 makes the sign-off ritual the central artefact; the dashboard is a support instrument.

**Offline-only eval and a surprise at week 8.** Building a strong offline eval, passing the pass bar, shipping to 5% of real traffic, and then discovering at week 8 that production traffic looks nothing like the gold set — because the gold set was drawn from the pilot phase and the customer's real cases include a long tail nobody sampled. Chapter 05 is the discipline that prevents this: online eval, implicit feedback capture, per-tenant drift monitoring, and a pre-agreed response protocol when drift crosses threshold.

**Judge-LLM that ships customer data to the vendor you are evaluating.** Routing judge-LLM calls to a public endpoint of the same vendor (or a competing vendor) whose model is being evaluated. Two problems at once: a data-residency violation that will fail security review, and a self-preference bias (and in cross-vendor cases a competitive-intelligence exposure) that invalidates the eval. Chapter 04 is explicit: the judge-LLM runs through the same gateway as the inference call, under the same data-processing agreement, and in the same trust boundary. For FedRAMP High, EU Data Boundary strict, and most on-prem deployments, that means an open-weights judge on the customer's own compute.

## Summary

- The eval overlay is the set of customer-environment concerns the FDE adds on top of the eval craft owned by `ai-eval-engineer` (peer, level 30, AI Engineering family) and the release-assurance craft owned by `ai-evaluation-engineer` (peer, level 35, AI Governance family). Mod-107 does not re-teach either; it layers.
- Twelve dimensions are re-opened per customer — from gold-set provenance through judge-prompt localisation to release-assurance packet shape. Each chapter in this module takes one or two.
- The organising budget is six-dimensional: fidelity, business-KPI coverage, privacy posture, cost per eval run, time-to-sign-off, drift resilience. Fidelity, privacy posture, and time-to-sign-off are gates; business-KPI coverage, cost, and drift resilience are optimisations.
- Three deployment archetypes (SaaS multi-tenant, dedicated SaaS, VPC install, with on-prem / air-gapped as the rare case) shape where the gold set, the judge-LLM call, and the online-eval sample physically live.
- Three customer archetypes — US healthcare payer (HIPAA + HITL approver on exit), US federal agency (FedRAMP evidence packet), EU retail bank (EU AI Act Article 14 + SR 11-7 model-risk packet) — thread through every chapter.
- Four anti-patterns to catch early: generic benchmark theatre, a dashboard with no sign-off ritual, offline-only eval and a surprise at week 8, and a judge-LLM that ships customer data to the vendor being evaluated.
- Chapter 02 is next: bootstrapping the customer gold set — SME pairing, stratified sampling across the customer's actual case distribution, double-labelling with inter-rater reliability, labelling-UI shape, and versioned auditable storage.
