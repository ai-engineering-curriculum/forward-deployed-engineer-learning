# Chapter 07 — Boundaries to the Security and Governance Peers

## Why this chapter exists

Mod-101 Chapters 03 and 06 fixed the general rule: the FDE at level 35 is a specialist IC among peers, and escalation to the peer who owns a specific craft is a discipline, not a failure. Mod-103 Chapter 09, mod-105 Chapter 07, mod-106 Chapter 07, and mod-107 Chapter 07 instantiated the rule for the infra, retrieval, agentic, and eval peer tracks. This chapter closes mod-108 by naming the three peers the security-and-compliance layer brings in.

- [`ai-infra-security`](https://github.com/ai-engineering-curriculum/ai-infra-security-learning) / [`security-learning`](https://github.com/ai-engineering-curriculum/security-learning) (peer, level 35, Infra family) — deep MLSec at platform scale.
- [`ai-risk-engineer`](https://github.com/ai-governance-curriculum/ai-risk-engineer-learning) (peer, level 25, Governance family) — risk-engineering for the AI estate.
- [`ai-evaluation-engineer`](https://github.com/ai-governance-curriculum/ai-evaluation-engineer-learning) (peer, level 35, Governance family) — release-assurance packaging, already named in mod-107 Chapter 07.

Three peers, three different bars. One lives next to the FDE in the applied-AI craft (`ai-infra-security` on platform-security depth). One lives next to the customer's risk officer running the AI risk-management program (`ai-risk-engineer`). One lives next to the customer's board or regulator (`ai-evaluation-engineer`). The FDE has to brief each one correctly, and must know which pager each one answers.

The symmetry with mod-107 Chapter 07 matters. That chapter named two peers because the eval layer splits methodology from assurance. This chapter names three because the security-and-compliance layer splits platform depth from risk engineering from release assurance. The split is load-bearing: `ai-infra-security` makes the platform defensible; `ai-risk-engineer` makes the risk register coherent; `ai-evaluation-engineer` makes the package legible to the regulator. Confusing them is how pilots ship with a strong per-pilot threat model and no platform-level enforcement, or with a clean risk register and no supporting engineering evidence, or with a signed packet that collapses on the regulator's first adversarial question.

## The `ai-infra-security` / `security-learning` boundary

### What this peer owns

Platform-level security of the AI estate, independent of where a given pilot runs. Mod-107 Chapter 07 named `ai-eval-engineer` as the eval-depth peer; `ai-infra-security` is the analogous depth peer for security:

- **MLSec at inference edge.** Model-extraction defence (query-rate limiting with similarity detection, watermarked outputs, challenge-response), guardrail-as-a-service architecture, prompt-injection-detection platform services shared across pilots.
- **Judge supply-chain hardening.** Verified build and provenance for judge-LLMs the eval stack uses, attack-resistant judge-prompt construction, protection against eval-set exfiltration.
- **MLSecOps / platform SBOM program.** Fleet-wide ML-BOM pipelines, Sigstore / SLSA policy enforcement in CI/CD at the platform level, continuous vulnerability scanning for weights across the organisation.
- **Platform-scale threat modelling.** STRIDE / PASTA / OCTAVE adapted for the AI stack; platform-scale mitigations landed as controls (WAF-for-LLM, gateway-side DLP service, central OIDC) rather than per-pilot code.
- **Platform-scale red-team program.** Scheduled adversarial exercises across the fleet; emergent-behavior discovery; frontier-model capability elicitation.
- **AI-specific incident response.** Runbooks for model-exfiltration incidents, data-poisoning incidents, large-scale injection campaigns, judge-compromise events. Ties into the organisation's CSIRT.
- **Security of the detection stack.** Making sure the guardrails, judge-LLMs, and audit pipeline themselves cannot be poisoned, bypassed, or spoofed at platform scale.

In short: *everything that makes the AI platform defensible as a platform, independent of which pilot is running on it*.

### What the FDE adds

- Per-pilot threat model instantiation (Chapter 04 matrix).
- Per-pilot DFD, sub-processor list, DPA / BAA architectural backing (Chapters 02, 05).
- Per-pilot ML-BOM, model / system / dataset cards, scan reports (Chapter 06).
- Lightweight adversarial pass before go-live.
- Customer-facing security-review shepherding.

### What the FDE does NOT own

- Platform-level WAF-for-LLM or guardrail-as-a-service architecture.
- Fleet-wide red-team programs with rotation.
- AI-specific CSIRT runbook authoring.
- Platform-level ML-BOM pipeline and SLSA policy design.
- Cross-pilot security-posture reporting to leadership.
- Model-extraction defence research.

### When this boundary fires

Four concrete cases.

**Case 1: a pilot needs a security control that no platform service provides.** The pilot needs model-extraction defence (adversary iteratively probing to reconstruct the model), and the vendor's platform has no inference-edge defence service. The FDE escalates — either the platform builds it, or the pilot documents a residual.

**Case 2: an adversarial finding during the pilot's red-team pass has fleet-wide implications.** The FDE's lightweight adversarial pass (Chapter 04) finds a prompt-injection pattern that bypasses the vendor's guardrail. One pilot's bug is every pilot's bug. Escalate for platform-level patching, not per-pilot workaround.

**Case 3: the customer requests an adversarial assessment beyond the standard threat model.** The customer's security office wants a frontier-grade red-team — adversarial multi-turn evaluation, emergent-behavior discovery, capability elicitation. That depth lives in the peer, not the FDE.

**Case 4: an AI-specific incident fires in production.** A suspected prompt-injection-enabled exfiltration, a poisoned-retrieval-corpus event, a compromised third-party weight scan after deploy. The FDE pages the peer; the peer runs the incident; the FDE provides the pilot's evidence.

### What the FDE hands to the peer

A specific brief, mirroring mod-107 Chapter 07's shape.

| Field | Example |
|---|---|
| Customer + workflow | US healthcare payer; member-inquiry resolution |
| Deployment + compliance posture | VPC install on AWS; HIPAA + SOC 2 Type II; BAA signed |
| Threat model matrix | v2.1 signed 2026-10-28; residual-risk entries |
| Lightweight adversarial pass results | Playbook + logs + findings |
| Sub-processors | Current list (Chapter 02); any new sub-processor in flight |
| Observed incident / finding | Attack narrative, trajectory, impacted asset |
| Change constraints | Customer AppSec requires review before new tools; customer prefers one architectural change per sprint |

Without this brief the peer spends a week asking for it back.

### What the FDE receives back

- A platform-level mitigation (if the finding is platform-shaped) or a per-pilot mitigation recommendation (if pilot-shaped).
- An updated threat-model matrix row with the mitigation and evidence.
- A statement of residual risk at the platform level.
- For incidents: an IR report and a remediation roadmap.
- Recommended detections to wire into the audit stream (mod-110).

### Who answers which pager

- Platform guardrail misses a prompt injection fleet-wide -> peer.
- Pilot-specific adversarial finding -> FDE triages; escalates to peer if the finding is not pilot-local.
- Customer's security team requests a frontier red-team -> peer.
- Suspected model-exfiltration incident -> peer leads; FDE is first responder with the pilot's context.
- SIEM detection rule authoring -> shared (peer owns the platform catalogue; FDE adds the pilot-specific signature).
- Threat-model matrix update after a tool change -> FDE handles.
- SLSA / Sigstore deploy-gate policy change -> peer; FDE consumes.

The FDE triages. Escalates with the DFD, the threat matrix, the audit sample, and the pilot's deployment context.

### What the FDE must know — the surface

- OWASP LLM Top 10 and Agentic Top 10 at working fluency.
- MITRE ATLAS tactics and the most-common techniques.
- STRIDE basics; when to compose it with OWASP.
- The vendor's platform-level security services — which guardrails, which DLP, which observability pipeline enforces what.
- SOC 2 Trust Services Criteria; NIST 800-53 Rev 5 control-family names; ISO/IEC 27001 Annex A.
- The audit shape mod-105 / mod-106 / mod-107 produce.

The infra-security peer owns platform depth. The FDE's job is to recognise which platform lever applies and brief it.

### When to try first vs when to escalate

- **Try first**: per-pilot threat-model matrix; lightweight red-team pass; DFD; sub-processor list; evidence packet assembly.
- **Escalate**: platform-level guardrail design; fleet-wide red-team program; AI-specific IR; model-extraction defence research; SLSA policy at the platform.

## The `ai-risk-engineer` boundary

### What this peer owns

Risk engineering for the AI estate at the customer's governance function (or at the vendor's own governance function when the vendor is also the deployer):

- **Risk taxonomy.** Cataloguing AI-specific risks with likelihood / impact / ownership; harmonising with the enterprise risk register and the enterprise risk committee's vocabulary.
- **Control mapping.** Translating risks to controls that map to NIST AI RMF (Govern / Map / Measure / Manage), ISO/IEC 42001, and sector regimes (SR 11-7, FDA GMLP, EU AI Act Annex IV, HITRUST AI).
- **Residual-risk statements.** After mitigations, what remains; who signs; at what cadence the residual is re-evaluated.
- **Ongoing risk monitoring and reporting.** Trend reports to the risk committee; triggers for re-classification; portfolio views across pilots.
- **Impact assessments.** Algorithmic impact assessments (Canada's AIA, agency-specific), DPIAs (Article 35), bias impact assessments (LL144, Colorado AI Act).
- **Control testing.** Running the controls against real traffic and real incidents to confirm they fire.

In short: *everything that turns per-pilot risks into a coherent portfolio posture, mapped to frameworks the customer's governance function speaks*.

The peer is at level 25 (Governance family) — closer in level to the customer's control-mapping analyst or the junior-to-mid risk engineer than to the FDE. The boundary is not "senior defers to junior" — the boundary is *craft specialisation*. The risk engineer brings a control vocabulary and a portfolio view the FDE does not maintain.

### What the FDE adds

- The per-pilot technical facts — threat matrix (Chapter 04), DPA levers (Chapter 05), provenance artefacts (Chapter 06), evidence packet (Chapter 02).
- The compliance routing (Chapter 03) — which regimes apply, which obligations route to which party.
- The per-pilot residual-risk entries sourced from the Chapter 04 matrix.

### What the FDE does NOT own

- Enterprise risk-register schema definition.
- Framework-level control mapping across the AI portfolio.
- Risk-committee reporting cadence or format.
- Cross-pilot aggregated risk views.
- The customer's own AI governance program structure.

### When this boundary fires

Three concrete cases.

**Case 1: the customer is building an AI governance program.** The customer's risk / privacy / compliance teams are standing up a new AI-use inventory, a new risk-register taxonomy, a new review board. The FDE's per-pilot artefacts are inputs to the program; the peer helps the customer compose the program. Common at organisations rolling out NIST AI RMF or ISO/IEC 42001 adoption.

**Case 2: the pilot's risk needs to roll up to an enterprise framework.** The customer's CRO wants every AI pilot's residual risks mapped to the existing enterprise risk categories (operational, compliance, model, strategic). The FDE produces the inputs; the peer does the mapping.

**Case 3: a regulated impact assessment is required.** DPIA under GDPR Article 35; algorithmic impact assessment under Canada's Directive on Automated Decision-Making; bias impact assessment under LL144 or Colorado AI Act. The FDE produces the technical slice; the peer runs the assessment.

### What the FDE hands to the peer

| Field | Example |
|---|---|
| Customer + workflow | US healthcare payer; member-inquiry resolution |
| Classification | Customer-side: non-rights-impacting under NIST AI RMF; customer's NAIC-aligned risk tier 2 |
| Threat matrix | v2.1 (Chapter 04) |
| Residual risks | 3 named residuals with mitigation evidence |
| DPA levers | Chapter 05 artefacts |
| Provenance artefacts | Chapter 06 bundle |
| Evaluation evidence | mod-107 per-pilot decision document |
| Target framework | NIST AI RMF 1.0; ISO/IEC 42001 if in scope |

### What the FDE receives back

- A risk-register entry (or set of entries) mapped to the customer's taxonomy.
- Control-mapping statements aligned to NIST AI RMF / ISO 42001 / sector regime.
- A written residual-risk statement signed by the customer's risk owner.
- Recommendations for ongoing risk monitoring — what to measure, at what cadence, with what trigger thresholds.

### Who answers which pager

- Customer risk committee asks "where do our AI pilots sit in our risk taxonomy?" -> peer.
- Customer DPO asks for a DPIA on the pilot -> peer runs; FDE supplies technical input.
- Customer CRO asks for a quarterly risk trend -> peer.
- Customer business owner asks "what's the current residual risk on this pilot?" -> FDE first responder; cites the risk-register entry authored by the peer.
- Pilot-specific threat changes -> FDE updates matrix; peer updates register if the residual changes.

### What the FDE must know — the surface

- NIST AI RMF 1.0 functions (Govern / Map / Measure / Manage).
- ISO/IEC 42001 top-level structure.
- The difference between threat (Chapter 04 OWASP matrix) and risk (likelihood × impact framing the peer uses).
- The standard impact-assessment flows (DPIA, AIA, bias impact assessment).
- Enough control vocabulary to recognise when a mitigation the FDE wrote maps to a named control family.

The risk-engineer owns the register. The FDE's job is to produce technical inputs the peer can lift.

### When to try first vs when to escalate

- **Try first**: per-pilot threat matrix; per-pilot residual-risk statement; per-pilot artefact packet.
- **Escalate**: enterprise risk-register taxonomy; cross-pilot risk aggregation; framework-level control mapping; formal impact-assessment authoring.

## The `ai-evaluation-engineer` boundary — a reminder

Mod-107 Chapter 07 named this peer in full. The security-and-compliance layer re-engages the peer for three specific cases:

1. **Release-assurance packet composition.** The per-pilot artefacts this module produces (Chapter 02 evidence packet, Chapter 03 compliance matrix, Chapter 04 threat matrix, Chapter 05 DPA artefacts, Chapter 06 cards and ML-BOM) are the inputs; the peer composes them into the regulator-grade packet.

2. **Regulator-facing documentation shape.** EU AI Act Annex IV technical documentation, SR 11-7 validation memo, FedRAMP ATO body-of-evidence annex, Colorado AI Act impact statement, NAIC AI Model Bulletin filing. The peer authors; the FDE supplies evidence.

3. **External-auditor interface.** Big-4 or specialist AI auditor engagement; response to audit findings; third-party assurance letters. The peer runs the interface; the FDE answers audit-trail-detail questions.

Pager rules unchanged from mod-107 Chapter 07: the FDE owns the per-pilot decision document and the operational evidence; the peer owns the regulator-grade artefacts.

## Composing the three boundaries

All three peers often in the same pilot phase:

**Case A — EU AI Act high-risk classification mid-pilot.** `ai-infra-security` tightens the guardrail stack for the new classification; `ai-risk-engineer` re-maps the pilot into the new risk-register tier with new monitoring triggers; `ai-evaluation-engineer` composes the Annex IV documentation. The FDE runs point: technical facts to the first, residuals to the second, evidence pack to the third.

**Case B — security incident during pilot pre-launch.** Suspected prompt-injection-enabled data access. `ai-infra-security` leads the IR; `ai-risk-engineer` updates the risk register with the new exposure; `ai-evaluation-engineer` updates the model card and the residual-risk statement in the release packet. The FDE is incident first responder, evidence producer, and briefer.

**Case C — new customer in a new regime.** First Colorado AI Act deployer in the vendor's portfolio. `ai-infra-security` reviews the deployment's attack surface; `ai-risk-engineer` builds the risk-register template for Colorado AI Act deployers; `ai-evaluation-engineer` builds the Colorado-filing template. The FDE pilots the shape; the templates become reusable artefacts.

Four peers in some cases (adding `agentic-safety-engineer` from mod-106 Chapter 07 when the pilot is an agent handling high-stakes actions). One integration design.

## The compliance privacy line — the hard rule

The eval-data privacy line from mod-107 Chapter 07 has a direct analogue here: compliance artefacts can themselves leak. The evidence packet, the threat matrix, the DPA drafts, the DPIA technical inputs — each contains information about the customer, the deployment, and sometimes about the workflow's weaknesses. The rule:

- **Treat the evidence packet as sensitive.** It is shared under NDA. The customer's authenticated channel (OneTrust, Whistic, SafeBase, customer portal) is the delivery vector. Email as a secondary channel; public shares never.
- **Threat matrices are especially sensitive.** They enumerate exploitable attack scenarios with current mitigation evidence. Share with named reviewers only. Store encrypted at rest.
- **DPA drafts are negotiation artefacts.** Pre-executed drafts are both legal and technical; both sides treat them as confidential until executed.
- **DPIAs and impact assessments are both compliance artefacts and customer business information.** Their distribution is the customer's choice.

Breaking this line is not just an operational mistake — it is itself a data-handling incident the customer will escalate.

## When to try first vs when to escalate — the honest table

| Situation | Try first | Escalate |
|---|---|---|
| Per-pilot threat matrix (OWASP + ATLAS) | FDE | — |
| Per-pilot DFD | FDE | — |
| Per-pilot DPA engineering attestations | FDE | — |
| Lightweight red-team pass | FDE | — |
| Platform guardrail design | — | Peer (infra-security) |
| Fleet-wide red-team program | — | Peer (infra-security) |
| AI-specific incident response | — | Peer (infra-security); FDE first responder |
| SLSA / Sigstore deploy-gate policy at platform level | — | Peer (infra-security) |
| Per-pilot residual-risk statement | FDE | — |
| Enterprise risk-register taxonomy | — | Peer (risk-engineer) |
| DPIA / algorithmic impact assessment | — | Peer (risk-engineer); FDE technical input |
| NIST AI RMF control mapping across portfolio | — | Peer (risk-engineer) |
| Per-pilot ML-BOM + cards | FDE | — |
| Regulator-facing release packet | — | Peer (evaluation-engineer) |
| External audit walk-through | — | Peer (evaluation-engineer) |
| EU AI Act Annex IV documentation | — | Peer (evaluation-engineer); FDE evidence |

Mod-101 Chapter 06's escalation rule applies throughout: try first on the customer-environment overlay; escalate on depth, scale, compliance, or novelty.

## Anti-patterns

1. **FDE as the sole security authority.** Platform-level findings quietly become pilot-level workarounds because the FDE did not escalate. Fleet-wide bugs ship repeatedly. Escalate on platform-shaped findings even when the pilot can paper over them.
2. **Threat matrix authored by AppSec alone.** AppSec reviews; the FDE authors. The FDE's architectural knowledge is the thing that makes the matrix accurate. AppSec-authored matrices often miss pilot-specific mitigations already in code.
3. **Risk register entries authored by the FDE.** The FDE's language is architectural; the risk register's language is probability × impact mapped to the enterprise risk vocabulary. Hand the facts to the peer.
4. **Regulator packet authored by the FDE.** Regulators rely on vocabulary the peer knows and the FDE does not. A plausible-looking packet by the FDE collapses on the auditor's first follow-up.
5. **No sign-off ritual on the Chapter 04 matrix.** A threat matrix without a dated, signed-by-reviewer field reads as a draft to the customer's AppSec. The signature is the deliverable.
6. **No peer cited in the customer's documentation.** When the customer asks "who at your organisation is on the hook for a platform-level security finding?" the FDE must name `ai-infra-security`, the pager, and the escalation SLA. Silence reads as the FDE owning everything.
7. **Incident response without the peer.** A production security event triggers the FDE to debug alone. Even if the fix is architecturally simple, the peer has the fleet-wide context (has this happened elsewhere? is a platform mitigation in flight?) and should be looped in on page one.
8. **Risk-register entry stale after a mitigation ships.** The FDE ships a Chapter 04 mitigation; the register still carries the old residual. Keep the peer in sync.

## Summary

- Three peer boundaries live in the security-and-compliance layer: `ai-infra-security` / `security-learning` (level 35, Infra family) for platform-security depth; `ai-risk-engineer` (level 25, Governance family) for risk-engineering portfolio craft; `ai-evaluation-engineer` (level 35, Governance family) for release-assurance packaging.
- The infra-security peer owns platform guardrails, fleet-wide red-team, AI-specific IR, SLSA policy, model-extraction defence. The FDE owns per-pilot threat matrix, DFD, DPA engineering backing, lightweight red-team, evidence packet.
- The risk-engineer peer owns enterprise risk-register taxonomy, control mapping to NIST AI RMF / ISO/IEC 42001, formal impact assessments, cross-pilot risk aggregation. The FDE owns the per-pilot technical facts and residual-risk statements.
- The evaluation-engineer peer owns regulator-grade packets, external-auditor interface, EU AI Act Annex IV / SR 11-7 / ATO documentation. The FDE owns the per-pilot decision document and the evidence packet.
- Four cases fire the infra-security boundary (platform gap, fleet-wide finding, frontier red-team ask, incident). Three cases fire the risk-engineer boundary (governance program stand-up, enterprise roll-up, formal impact assessment). Three cases fire the evaluation-engineer boundary (release packet, regulator documentation, external audit).
- The compliance privacy line is a hard rule: evidence packets, threat matrices, DPA drafts, and DPIAs are sensitive customer-shared artefacts; authenticated channels only.
- Pager rules partition cleanly among the three peers and the FDE.
- Mod-101 Chapter 06's escalation rule applies: try first on the customer-environment overlay; escalate on depth, scale, compliance, or novelty.

With this chapter the module closes. Mod-109 (deployment, rollout, change management) picks up next: how the signed threat matrix, the executed DPA, the published ML-BOM, and the evidence packet become gates in the rollout plan — and how feature flags, canary slices, and shadow-mode runs sequence the risk from pilot to production.
