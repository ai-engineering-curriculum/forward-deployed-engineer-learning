# Chapter 07 — Boundaries to the `ai-eval-engineer` and `ai-evaluation-engineer` Peers — and the Eval-Data Privacy Line

## Why this chapter exists

Mod-101 Chapters 03 and 06 fixed the general rule: the FDE at level 35 is a specialist IC among peers, and escalation to the peer who owns a specific craft is a discipline, not a failure. Mod-103 Chapter 09, mod-105 Chapter 07, and mod-106 Chapter 07 instantiated that rule for the infra, retrieval, and agentic peer tracks. This chapter closes mod-107 by naming the two peers the eval layer brings in: [`ai-eval-engineer`](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning) (peer, level 30, AI Engineering family) on eval methodology depth, and [`ai-evaluation-engineer`](https://github.com/ai-governance-curriculum/ai-evaluation-engineer-learning) (peer, level 35, AI Governance family) on release-assurance packaging.

Two peers, two different bars. One lives next to the FDE in the applied-AI craft; the other lives next to the customer's risk officer in the governance craft. The FDE has to brief both correctly, and must know which pager each one answers.

The symmetry with the prior boundary chapters matters. Mod-105 Chapter 07 named one peer (`rag-engineer`) because retrieval is one craft. Mod-106 Chapter 07 named two (`agentic-ai-engineer` and `agentic-safety-engineer`) because the agentic layer splits capability from safety. Mod-107 names two because the eval layer splits methodology from assurance. The split is load-bearing: the eval engineer makes the measurement defensible; the evaluation engineer makes the measurement legible to a regulator. Confusing the two is how pilots ship with a strong internal eval and no external sign-off, or with a signed packet that collapses on the first adversarial question.

## The `ai-eval-engineer` boundary

### What this peer owns

Eval methodology at depth, independent of where the pilot runs:

- **Metric-design methodology.** When a new metric is needed — a workflow the existing portfolio does not cover, or a failure mode slipping through all four metric families from Chapter 03. The peer derives the metric, validates it against SME labels, and documents its variance.
- **Judge-model calibration research.** Judge-selection methodology, prompt-structure design for judges, bias characterisation (verbosity, position, self-preference), judge-vs-SME agreement studies, calibration curves.
- **Statistical rigor.** Confidence intervals, significance testing, multiple-comparison correction (Bonferroni, Benjamini-Hochberg), power analysis for sample-size decisions, variance decomposition.
- **Harness internals.** Framework contributions, novel scorer design, parallelism and caching in eval runs, reproducibility guarantees under non-deterministic judge models.
- **Benchmark construction.** Bespoke benchmarks for in-house applied-AI products — the data curation, the distribution design, the leak controls.
- **Agent-trajectory evaluation methodology.** Trajectory matching, task-success rate, intervention-rate, cost-per-completed-task. Composes with mod-106 Chapter 07's `agentic-ai-engineer` boundary when the pilot has an agent.
- **Eval of fine-tuned and domain-adapted models.** Pre- vs post-fine-tune evaluation, test-set contamination checking, scaling-law-informed eval sizing.

In short: *everything that makes an eval reliable, statistically defensible, and methodologically sound, independent of where the pilot runs*.

### What the FDE adds

- Customer-workflow framing: *which* metric portfolio fits *this* customer's workflow and KPI (Chapter 03).
- Gold-set bootstrap: stratified sampling, SME-paired labelling, IRR on the customer's data (Chapter 02).
- Portfolio selection against the archetype's compliance posture and budget.
- CI wiring (Chapter 04) and online-eval integration (Chapter 05).
- Exit-eval ritual with the customer's governance board (Chapter 06).

### What the FDE does NOT own

- New metric families (e.g., a novel reasoning-chain eval metric).
- New judge-architecture (e.g., specialised rubric-reasoning judge).
- Scaling-law-informed eval sizing.
- Cross-model benchmark design for a broad model portfolio.
- Publication-grade methodology.

### When this boundary fires

Three concrete cases.

**Case 1: a portfolio is not catching a known failure mode.** Repeated incidents in production that the offline portfolio did not flag. The FDE hands the case plus the portfolio to the peer, who designs a new metric or redesigns the judge rubric. Without that depth the pilot papers over the hole with a manual check.

**Case 2: the judge-LLM is uncalibrated and threshold tuning is not producing reliable accept rates.** Judge-vs-SME kappa stuck below 0.5 across multiple prompt revisions. The FDE escalates for calibration redesign — judge-model swap, rubric restructure, few-shot overhaul.

**Case 3: the pilot wants a fine-tune.** Before/after eval at statistical bar. The peer designs sample size (power analysis), test-set contamination controls, and the pre-registered test. Without that discipline the fine-tune's measured lift is not defensible.

### What the FDE hands to the peer

A specific brief, mirroring mod-106 Chapter 07's shape.

| Field | Example |
|---|---|
| Customer + workflow | US healthcare payer; member-inquiry resolution + drafted reply |
| Current portfolio | correctness-judge (v3), faithfulness (RAGAS), refusal recall, pairwise win rate |
| Gold-set shape | v1.3, 120 rows, stratified intent x CSR tier x freshness; 20% hold-out sealed |
| IRR on labels | Krippendorff alpha = 0.78 on 20% double-labelled slice |
| Judge-LLM posture | Bedrock Claude Sonnet 4.6 in-VPC; zero-retention; last calibrated 2026-10-14; kappa vs SME = 0.72 |
| Metrics + thresholds | correctness >= 0.85; faithfulness >= 0.90; refusal recall >= 0.90 |
| Known gaps | member_history intent slice dips 10 pts vs overall; 3 production incidents in 6 weeks not caught offline |
| Change constraints | AppSec review required before adding tools; customer prefers one portfolio migration per quarter |

Without this brief the peer spends a week asking for it back.

### What the FDE receives back

- A redesigned metric or judge prompt with measured gold-set agreement.
- An ablation on the hold-out showing the change catches the failure mode.
- A sample-size + significance plan if a fine-tune or major prompt change is scoped.
- An opinion on whether the current gold-set size is sufficient for the metric's variance.
- Recommended regression tests to lock into CI.
- A written statement of residual methodological risk — what the new metric does *not* measure and what the FDE should watch for in online eval.

### Who answers which pager

- Judge-vs-SME agreement regression after a judge-model upgrade -> peer.
- Portfolio metric drops after a prompt change -> FDE triages; escalates to peer if the metric itself is the suspect.
- Statistical-significance question from the customer's business owner -> peer.
- Customer asks "is 80 rows enough?" -> peer.
- CI gate fails on a real regression -> FDE handles.
- Judge-LLM endpoint returns 500 -> FDE; it's a plumbing issue.

The FDE triages. Escalates with the trajectory, the metric, and the gold-set rows that reproduce the finding.

### What the FDE must know — the surface

- The four metric families from Chapter 03 at working fluency.
- The common judge biases (verbosity, position, self-preference) and the mitigations (random swap, few-shot anchoring, CoT rubric, judge diversity).
- Cohen's kappa / Krippendorff's alpha / Fleiss' kappa — when each applies.
- Power analysis basics — larger effect sizes need fewer samples; the customer always asks this.
- PSI / KL / KS — the drift detectors from Chapter 05.

The eval engineer owns the depth. The FDE's job is to recognise which lever applies and brief it.

### When to try first vs when to escalate

- **Try first**: wiring a known metric into Promptfoo / RAGAS / DeepEval / Inspect; standard judge-prompt authoring; CI gating; online-eval instrumentation.
- **Escalate**: new metric design; uncalibrated judge; fine-tune evaluation; benchmark construction; statistical-significance publication.

## The `ai-evaluation-engineer` boundary

### What this peer owns

Release-assurance packaging at a level the FDE's per-pilot evidence does not reach:

- **Regulator-facing release-assurance packets.** Model cards, risk registers, audit-trail format, cross-model evidence consistency.
- **Model-card composition** per NIST AI RMF, ISO/IEC 42001, and sector-specific regimes (SR 11-7 for banking, EU AI Act Annex IV for high-risk).
- **Risk-register construction.** Taxonomy of risks, mitigation attestations, residual-risk statements.
- **Audit-trail compliance.** Evidence preservation, chain-of-custody for eval runs, signed sign-off chains, retention policy alignment.
- **Formal human-oversight sign-off process.** The protocol by which human oversight is evidenced (EU AI Act Article 14), not just the per-pilot decision.
- **External-auditor engagement.** Big-4 or specialist AI auditor liaison, response to audit findings, third-party assurance letters.
- **Cross-pilot program-level risk posture.** Portfolio view across many pilots, aggregated residual risk, enterprise AI risk committee reporting.
- **Central model-card and eval-evidence registry schema.** The source-of-truth schema for the whole AI portfolio, not one pilot.

In short: *everything that turns per-pilot eval evidence into a regulator-grade, enterprise-level assurance posture*.

### What the FDE adds

- The **per-pilot decision document** (Chapter 06) — the one-pager with scope, evidence summary, residual-risk register, decision, signatories, review cadence.
- The archetype-specific signatory map (clinical ops + privacy officer; agency CIO + records officer; CRO + model-risk committee).
- The compliance-posture choice embedded in the deployment (VPC install, EU Data Boundary, GovCloud) — already decided in mod-104 / mod-105 / mod-106.
- The operational evidence (online-eval history, incident log, drift alerts).

### What the FDE does NOT own

- Regulator engagement (EU AI Office, state regulators, FINRA, OCC).
- External audit firm liaison.
- Cross-pilot program-level risk roll-up.
- The central model-card registry's schema definition.

### When this boundary fires

Four concrete cases.

**Case 1: high-risk regime under EU AI Act Article 6 / Annex III.** The workflow is classified high-risk; a full model card plus human-oversight evidence packet is mandatory. The FDE escalates once the classification is confirmed.
<!-- needs-research: exact quote wording from EU AI Act Article 6 and Annex III -->

**Case 2: SR 11-7 model-risk management packet for Archetype C (bank).** The workflow qualifies as a "model" under the guidance; the Chief Risk Officer requires an SR 11-7-aligned validation.
<!-- needs-research: exact quote wording from SR 11-7 three-pillar text -->

**Case 3: FedRAMP authorization package for Archetype B (federal agency).** The eval evidence is one component of the ATO package; the peer owns the formal composition.
<!-- needs-research: exact body-of-evidence wording from FedRAMP ATO guidance -->

**Case 4: external audit firm review scheduled.** A Big-4 or specialist auditor wants to verify the eval evidence; the peer owns the interface and the walk-through.

### What the FDE hands to the peer

| Field | Example |
|---|---|
| Customer + workflow + classification | EU retail bank; customer-support reply + case triage; Annex III high-risk; SR 11-7 model |
| Deployment + compliance posture | Azure AI Foundry EU Data Boundary; GDPR + DORA; EU AI Act Article 14 oversight |
| Per-pilot decision document | v2.1 signed by CRO + model-risk committee; dated 2026-10-28 |
| Residual-risk register | 4 items; HITL on all triage updates; kill-switch on refusal-rate drift |
| Operational evidence pack | Online-eval 90-day rolling; drift incidents log; 2 production incidents root-caused |
| Prior red-team / adversarial test | mod-106 OWASP Agentic coverage matrix v1.1 |
| Target assurance artefact | SR 11-7 validation memo; EU AI Act Annex IV documentation; ATO annex if applicable |
| Timeline / stakeholders | Model-risk committee review 2026-12-15; external auditor walk-through 2027-01 |

### What the FDE receives back

- A regulator-grade model card and risk register derived from the per-pilot artefacts.
- A chain-of-custody attestation for the eval evidence.
- A statement the customer's board or auditor can forward.
- Recommendations on evidence-preservation improvements for the next pilot phase.

### Who answers which pager

- Regulator asks for model-card update -> peer.
- External auditor finds an evidence gap in the eval packet -> peer.
- Customer's internal AI risk committee raises a cross-pilot concern -> peer.
- Customer's business owner needs the per-pilot decision document updated -> FDE.
- Drift incident requires a regulator-notification decision (EU AI Act Article 72) -> peer (escalated from FDE's initial triage).
<!-- needs-research: exact notification-threshold wording from EU AI Act Article 72 -->

### What the FDE must know — the surface

- NIST AI RMF (AI RMF 1.0) functions (Govern, Map, Measure, Manage) at working fluency.
- ISO/IEC 42001 top-level structure.
- EU AI Act Article 14 (human oversight), Article 72 (post-market monitoring), Annex III (high-risk classification), Annex IV (technical documentation).
- SR 11-7 three-pillar structure (model development, implementation, validation; ongoing monitoring; governance).
- FedRAMP boundary and body-of-evidence structure.
- Model Cards framework (Mitchell et al., FAT* 2019) as the industry vocabulary anchor.

The evaluation engineer owns the regulatory depth. The FDE's job is to produce evidence the peer can lift into a packet.

### When to try first vs when to escalate

- **Try first**: per-pilot decision document, per-pilot risk register, per-pilot audit-trail wiring.
- **Escalate**: regulator-facing model card; cross-pilot program-level risk roll-up; external audit; formal assurance letters.

## Composing the two boundaries

Both peers often in the same pilot phase:

- **Case A.** The pilot moves to production under EU AI Act Article 14. `ai-eval-engineer` designs the online-eval statistical bar for the human-oversight evidence; `ai-evaluation-engineer` packages the evidence into the Article 14 oversight artefact.
- **Case B.** A fine-tune is proposed mid-pilot. `ai-eval-engineer` designs the before-after test with power analysis; `ai-evaluation-engineer` updates the model card and the risk register to reflect the fine-tune.
- **Case C.** An external auditor flags a judge-calibration gap. `ai-eval-engineer` redesigns the judge plus calibration; `ai-evaluation-engineer` re-authors the assurance letter after the gap is closed.

Three peers in some cases (adding the `agentic-ai-engineer` from mod-106 when the pilot is an agent). One integration design.

## The eval-data privacy line

Hard rule, not preference. This is the dedicated section that the module's "keep customer data private" learning objective lands in.

- **Eval infrastructure runs inside the customer's trust boundary.** Two shapes:
  1. **VPC install** — the CI runner, the judge-LLM endpoint, the gold-set storage, and the online-eval pipeline all run inside the customer's VPC or tenant. Preferred for regulated workloads.
  2. **Zero-retention + no-training sandbox** — a vendor-managed enclave with a signed DPA that (a) prohibits retention of inputs beyond the evaluation call, (b) prohibits use for training, (c) commits to an agreed residency region, (d) is auditable. Acceptable for non-regulated data classes and for pilots where VPC install is not feasible.

- **Specific vendor postures to anchor on:**
  - **AWS Bedrock** — zero-retention by default; customer-managed KMS; residency per region.
  - **Azure OpenAI** — zero-retention under data-residency SKUs; EU Data Boundary addendum for Archetype C; customer-managed keys.
  - **Google Vertex AI** — CMEK plus Vertex AI ML Privacy commitment.
  - **Self-hosted open-weights** (Llama, Qwen, DeepSeek via vLLM or SGLang) — zero exfiltration by construction; the choice for Archetype B.

- **No customer data in an external SaaS eval pipeline unless the DPA covers it.** Common patterns that need the DPA check: Promptfoo Cloud, Humanloop, LangSmith hosted, Braintrust hosted — all can be valid *if* the data classification and DPA allow. The default assumption is NO.

- **Judge-LLM endpoint is the quietest exfiltration path.** A VPC pilot with an out-of-VPC judge-LLM is a data-exfiltration path dressed up as productivity. The judge call sits inside the same trust boundary as the production call.

- **Audit requirement.** Every eval run logs:
  - Model-ID that produced the output.
  - Judge-model plus judge-prompt-hash that scored it.
  - Authenticated identity of the operator who triggered the run.
  - Gold-set version used.
  - Storage location of the result.
  - Hash of the inputs (not the inputs themselves if PII policy forbids it).

  SIEM-destined; same discipline as production traffic (see mod-110).

- **Three-archetype application:**

  | Archetype | Eval infrastructure | Judge-LLM | Audit destination |
  |---|---|---|---|
  | A (US payer) | AWS VPC | Bedrock Claude Sonnet, zero-retention, in-VPC | Customer Splunk; HIPAA-aligned |
  | B (federal agency) | GovCloud | Self-hosted open-weights; no vendor SaaS | Agency Elastic SIEM |
  | C (EU retail bank) | Azure under EU Data Boundary | Azure OpenAI; EU residency; zero-retention; CMK | Microsoft Sentinel |

The eval-data line is enforced in the deployment architecture, not in a policy memo. If the architecture lets a judge call leave the trust boundary, the policy memo cannot save it.

## When to try first vs when to escalate — the honest table

| Situation | Try first | Escalate |
|---|---|---|
| Standard offline portfolio on a labelled gold set | FDE | — |
| Judge-LLM calibration against SME labels | FDE | Peer (eval-engineer) if kappa < 0.5 |
| Novel metric needed for a new failure mode | — | Peer (eval-engineer) |
| Fine-tune before/after evaluation | — | Peer (eval-engineer) |
| Per-pilot decision document | FDE | — |
| Regulator-facing model card | — | Peer (evaluation-engineer) |
| SR 11-7 validation memo | — | Peer (evaluation-engineer) |
| External audit walk-through | — | Peer (evaluation-engineer) |
| Cross-pilot program-level risk roll-up | — | Peer (evaluation-engineer) |
| Online-eval instrumentation | FDE | — |
| CI gate + judge-prompt authoring | FDE | — |
| EU AI Act Article 14 oversight artefact | — | Both peers |

Mod-101 Chapter 06's escalation rule applies throughout: try first on the customer-environment overlay; escalate on depth, scale, compliance, or novelty.

## Anti-patterns

1. **Judge-LLM running in the vendor's SaaS while the pilot runs in the customer's VPC.** An exfiltration path masquerading as productivity. The judge sees every production input. Pull the judge inside the boundary before the first eval run.
2. **FDE drafts the regulator-facing model card unaided.** The regulator catches the gaps the peer would have closed. Hand the per-pilot decision document to the peer; let the peer produce the regulator-facing artefact.
3. **`ai-eval-engineer` pulled in for CI plumbing.** Misuse of depth. The FDE owns the pipes. The peer is for metric design, judge calibration, statistical rigor.
4. **Release-assurance packet assembled post-hoc, week before launch.** Not enough runway for the peer to do a real review. Engage the peer at the pilot's mid-point, not at its endpoint.
5. **No signed sign-off from the business owner even though the decision document shipped.** The sign-off is the deliverable, not the document. An unsigned decision document is a draft.
6. **Not citing the boundaries in the customer's documentation.** When the customer asks "who is on the hook," the FDE must name the peers and the pager protocol. Silence here reads as the FDE owning everything, which the FDE does not.

## Summary

- Two peer boundaries live in the eval layer: `ai-eval-engineer` (level 30, AI Engineering family) for methodology depth, `ai-evaluation-engineer` (level 35, AI Governance family) for release-assurance packaging.
- The eval engineer owns metric design, judge calibration, statistical rigor, harness internals, benchmark construction, agent-trajectory eval methodology, and fine-tune evaluation. The FDE owns portfolio selection, gold-set bootstrap, CI wiring, online-eval integration, and the exit-eval ritual.
- The evaluation engineer owns regulator-facing model cards, risk registers, audit-trail compliance, formal human-oversight sign-off, external-auditor engagement, cross-pilot risk roll-up, and the central registry schema. The FDE owns the per-pilot decision document, the signatory map, the deployed compliance posture, and the operational evidence pack.
- Three cases fire the eval-engineer boundary (portfolio miss, judge uncalibrated, fine-tune). Four cases fire the evaluation-engineer boundary (EU AI Act high-risk, SR 11-7, FedRAMP, external audit).
- The eval-data privacy line is a hard rule: eval infrastructure runs inside the customer's trust boundary (VPC install or zero-retention sandbox under DPA); the judge-LLM is the quietest exfiltration path and must sit inside the same boundary as production; audit logs go to the customer's SIEM.
- Pager rules partition cleanly between the two peers and the FDE.
- Mod-101 Chapter 06's escalation rule applies: try first on the customer-environment overlay; escalate on depth, compliance, or novelty.

With this chapter the module closes. The immediate next module, mod-108 (enterprise security and compliance), formalises the release-assurance packet the `ai-evaluation-engineer` peer produces, and composes it with the control frameworks the customer's security organisation signs against.
