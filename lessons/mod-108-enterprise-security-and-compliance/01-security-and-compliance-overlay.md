# Chapter 01 — The Security and Compliance Overlay — Why Enterprise Security Review Is Not Generic Security

## Why this chapter exists

Mod-104 landed the LLM layer inside the customer's environment. Mod-105 landed grounding on the customer's data. Mod-106 landed the agentic layer with HITL, scoped tool-calls, and OWASP Agentic mitigations. Mod-107 landed the eval that proves the workflow is good enough to ship. Mod-108 picks up the step that decides whether the customer's security and compliance organisations will *let* it ship.

Security review is where the pilot collides with the customer's assurance posture again — now from the other side of the house. The eval layer answered "is this good enough?" The security-and-compliance layer answers "is this safe enough, provable enough, and contractually clean enough that our CISO, DPO, model-risk committee, and auditor will each put their name on it?" A pilot that cannot produce the evidence does not go live, no matter how good its metrics are.

The deep MLSec craft — model-extraction defence at inference edge, judge supply-chain hardening, eval-set exfiltration prevention, MLSecOps at platform scale — is owned by the peer at [`ai-infra-security`](https://github.com/ai-engineering-curriculum/ai-infra-security-learning) / [`security-learning`](https://github.com/ai-engineering-curriculum/security-learning) (peer, level 35, Infra family). The risk-engineering slice the customer's governance team may run in parallel — risk taxonomies, control mapping to NIST AI RMF and ISO/IEC 42001, residual-risk statements — is owned by [`ai-risk-engineer`](https://github.com/ai-governance-curriculum/ai-risk-engineer-learning) (peer, level 25, Governance family). The release-assurance packet that regulators actually read is owned by [`ai-evaluation-engineer`](https://github.com/ai-governance-curriculum/ai-evaluation-engineer-learning) (peer, level 35, Governance family), which mod-107 Chapter 07 already named. This module does not re-teach any of the three. It teaches the overlay the FDE adds on top of all three, at working depth. Chapter 07 writes the boundaries.

## The in-house crafts and what they cover

Three peers, three crafts, all deep. Name them before adding the overlay.

**`ai-infra-security` / `security-learning` (level 35, Infra family)** owns, at platform scale and across the organisation's full AI estate:

- **MLSec at inference edge.** Model-extraction defence (query-rate limiting with similarity detection, watermarked outputs, challenge-response), prompt-injection detection as a platform service, guardrail-as-a-service architecture.
- **Judge supply-chain hardening.** Verified build and provenance for judge-LLMs, judge-prompt integrity attestations, defence against eval-set exfiltration and judge-spoofing.
- **MLSecOps.** SBOM / ML-BOM pipelines across the fleet, Sigstore / SLSA policy enforcement in CI/CD, continuous vulnerability scanning for weights, red-team exercise scheduling, incident-response runbooks for AI-specific incidents.
- **Platform-scale threat modelling.** STRIDE / PASTA / OCTAVE adapted for the AI stack, with mitigations landed as platform controls rather than per-pilot code.
- **Security of the security tools.** Making sure the detection stack itself is not the attack surface — guardrail poisoning, judge collusion, telemetry spoofing.

**`ai-risk-engineer` (level 25, Governance family)** owns, inside the customer's (or vendor's) risk function:

- **Risk taxonomies.** Cataloguing AI-specific risks with likelihood, impact, and ownership. Harmonising with the enterprise risk register.
- **Control mapping.** Translating risks into controls that map to NIST AI RMF ([AI RMF 1.0](https://www.nist.gov/itl/ai-risk-management-framework)) Govern / Map / Measure / Manage, [ISO/IEC 42001](https://www.iso.org/standard/81230.html), and sector regimes (SR 11-7, FDA GMLP, EU AI Act Annex IV).
- **Residual-risk statements.** After mitigations, what remains; who signs that the residual is accepted.
- **Ongoing risk monitoring.** Drift of risk posture over time; review cadences; re-classification triggers.

**`ai-evaluation-engineer` (level 35, Governance family)** owns, as mod-107 Chapter 07 named in full:

- Regulator-facing release-assurance packets, model cards, risk registers, audit-trail format, cross-model evidence consistency.
- External-auditor engagement, formal human-oversight sign-off process, cross-pilot program-level risk roll-up.
- Central model-card and eval-evidence registry schema for the AI portfolio.

Three crafts. Three peers. All of them operate against their own product, their own platform, their own enterprise AI portfolio. The FDE at level 35 re-opens parts of each one *per customer*, because every customer's security review is the customer's own: their questionnaire, their vertical obligations, their data-contract shape, their supply-chain policy, their signatories. That is the overlay.

## The security and compliance overlay

The security and compliance overlay is the set of concerns the FDE adds on top of all three peer crafts. Each row is a dimension the in-house craft resolves once and the FDE re-opens per customer.

| Dimension | What the in-house craft resolves once | What the FDE re-opens per customer | Where this module handles it |
|---|---|---|---|
| Vendor questionnaire posture | One canonical SOC 2 / SIG / CAIQ response library | This customer's variant of the questionnaire — custom addenda, AI-specific supplements, legal clauses | Chapter 02 |
| Evidence-packet assembly | Internal control catalogue | Mapped against this customer's control framework (NIST 800-53, HITRUST CSF, ISO 27001:2022, customer-house) | Chapter 02 |
| Data-flow diagram | Platform-level, generic | Per-pilot DFD showing which fields cross which trust boundary, which keys protect them, which log receives the audit | Chapter 02 |
| Sub-processor disclosure | Standing sub-processor list | Current pilot's sub-processors (model provider, embedding API, retrieval vendor, judge-LLM endpoint, observability stack), disclosed per the customer's notice requirements | Chapter 02 |
| Pen-test report sharing | One annual third-party report | Shared under this customer's NDA, with scope-appropriate caveats, under this customer's evidence-handling rules | Chapter 02 |
| Vertical regulatory framing | Named at platform level | Which specific vertical regime this customer's workflow falls under — and which obligations route to the customer, which to the vendor, which to this role | Chapter 03 |
| BAA / DPA / addendum negotiation | Standard templates | Per-customer redlines; which clauses the engineering team has to be able to attest to | Chapter 03, Chapter 05 |
| Threat model construction | Platform-level STRIDE / PASTA / OCTAVE | Per-pilot OWASP LLM Top 10 + Agentic Top 10 + MITRE ATLAS matrix with named mitigations | Chapter 04 |
| Red-team cadence | Fleet-level schedule | Pre-go-live adversarial pass for this pilot's attack surface | Chapter 04 |
| Model-provider data contract | Vendor-level master agreement | Per-pilot overlay: zero-retention attestation for *this* deployment, residency for *this* region, training-opt-out for *this* account, retention for *this* audit log | Chapter 05 |
| BYOK / CMK posture | Platform-level KMS | Per-customer customer-managed keys — creation, rotation, revocation, break-glass, scope | Chapter 05 |
| DPA / BAA engineering attestations | Legal team | The engineering-grade attestations the DPA / BAA is going to ask for — "we do not log prompts," "we do not retain completions," "we do not train on inputs" — and the architecture that makes them true | Chapter 05 |
| Supply-chain provenance | SLSA / Sigstore in the platform pipeline | Per-pilot ML-BOM, SPDX AI, attestations on this release artifact, ModelScan / picklescan on every third-party model consumed | Chapter 06 |
| Model / system / dataset cards | Platform model-card registry | The deployed-pilot slice of each card — what *this* pilot's model, system, and data actually are in this customer's trust boundary | Chapter 06 |
| Peer boundaries | Named once at platform level | Named per pilot — which platform peer answers which pager when this pilot's security surface fires | Chapter 07 |

Fifteen rows of overlay. Each chapter in this module takes one or two. The pattern holds from mod-104 / 105 / 106 / 107: craft dimensions the peer specialists resolve once get re-opened by the FDE per customer, with gate dimensions enforced first.

## The organising budget: six dimensions

Mod-104 through mod-107 all organised decisions around a six-dimensional budget. Mod-108 keeps the shape and re-labels the axes for security and compliance.

1. **Evidence completeness** — can the pilot produce the artefacts the customer's security review will ask for, at the quality bar the customer accepts? Questionnaire responses, DFD, sub-processor list, pen-test summary, model card, SBOM / ML-BOM, residual-risk statement.
2. **Regulatory fit** — does the deployment satisfy the vertical regime the workflow sits under (HIPAA, FedRAMP, PCI, EU AI Act, GDPR, CCPA, LL144, Colorado AI Act, NYC bias audit) without creative interpretation?
3. **Data-contract cleanliness** — zero-retention commitments honoured end-to-end; residency enforced at the API path, not only in a policy memo; BYOK where the customer requires it; training-opt-out verifiable on the model-provider side; audit-log retention aligned with the customer's regime.
4. **Threat-model coverage** — OWASP LLM Top 10 + Agentic Top 10 + MITRE ATLAS rows each carry a named mitigation and an evidence artefact; the gaps are documented.
5. **Supply-chain provenance** — every weight, every image, every library is attested; the deploy pipeline produces SLSA-compliant provenance; third-party models are scanned before load.
6. **Time-to-sign-off** — wall-clock from "security-review packet submitted" to "signed approval to launch." Rolls up questionnaire response time, DFD review time, DPA redline cycles, AppSec threat-review time, and governance-board decision.

### Gate dimensions

**Regulatory fit, data-contract cleanliness, and threat-model coverage are gates.** A choice that fails any of them is off the table — supply-chain or completeness advantages do not recover it.

Regulatory fit is a gate because a workflow that falls outside the customer's compliance envelope cannot ship regardless of how clean the rest of the packet is. A HIPAA-covered workflow on a sub-processor without a BAA is not a packet problem — it is a shipping problem. Chapter 03 draws the boundaries.

Data-contract cleanliness is a gate because a zero-retention attestation that is contradicted by the actual gateway logs, a residency promise contradicted by a cross-region API call, or a training-opt-out contradicted by the model provider's default policy, all turn the project into a notification-and-remediation project the moment the customer's auditor asks the next question. Chapter 05 makes the contract architecturally true, not only contractually true.

Threat-model coverage is a gate because an unmitigated OWASP row in an enterprise pilot is a known vulnerability shipped on purpose. AppSec reviewers reject these on sight. Chapter 04 is where coverage gets explicit.

### Optimisation dimensions

**Evidence completeness, supply-chain provenance, and time-to-sign-off are optimisations.** Among gate-passing designs, the FDE picks the point that best fits the customer's maturity, the pilot's phase, and the regulator's expected depth.

Evidence completeness is the dimension the customer's CISO cares about most. Chapter 02 is where the questionnaire-response library and evidence packet are assembled.

Supply-chain provenance is the dimension that quietly sinks pilots — not at AppSec review, but at the auditor's follow-up six months later. Chapter 06 sets the discipline early enough that the auditor's question is a lookup, not a retrofit.

Time-to-sign-off is the dimension the FDE's own schedule cares about most. A four-month security review consumes the pilot's runway; Chapter 02 names the practices that compress it.

## The three deployment-model archetypes, re-cast for security review

Mod-104 Chapter 01 named three deployment-model archetypes (SaaS multi-tenant, dedicated SaaS, VPC install) and every subsequent module inherited them. For security review, the archetypes shape three physical questions: *where does customer data cross a trust boundary; which audit log receives each crossing; whose key protects each crossing*.

**SaaS multi-tenant.** Customer data crosses the vendor's trust boundary at ingest, at prompt submission, at completion return, at embedding call, at eval sample. The vendor's SOC 2 Type II, pen-test report, sub-processor list, and master DPA bear most of the evidence weight. The customer accepts the vendor's residency region, the vendor's key management, and the vendor's audit-log retention. Rare for regulated verticals except in early pilot phases; common for lower-sensitivity workflows.

*Who owns what:* vendor owns the compliance posture end-to-end. Customer accepts the vendor's controls and tests them with a questionnaire response + sub-processor review. The FDE owns the questionnaire response and the DFD for this tenancy.

**Dedicated SaaS.** Customer data crosses into a customer-dedicated namespace or cluster inside the vendor's cloud. Keys are customer-managed (BYOK / CMK) where the engine supports it. Audit logs stream to the customer's SIEM. Residency is pinned to a customer-chosen region. Sub-processor list is scoped to the sub-processors touching this dedicated deployment.

*Who owns what:* vendor owns the running stack; customer owns the keys, the audit stream, and sometimes customer-dedicated accounts for sub-processors (dedicated Bedrock / Azure OpenAI / Vertex AI keys). The FDE owns the per-deployment DPA addendum shape, the CMK integration, and the SIEM wiring.

**VPC install.** Customer data never leaves the customer's cloud. Inference runs on the customer's GPUs or through the customer's hyperscaler-native inference (Bedrock in the customer's account, Azure OpenAI under the customer's subscription, Vertex AI in the customer's project). Retrieval, eval, observability, audit all stay inside the trust boundary. The vendor ships code and configuration; the customer's platform team runs it. Compliance posture is largely the customer's own; the vendor's role is the engineering-grade attestations for the code, the images, and the model-provenance evidence.

*Who owns what:* customer owns the running system end-to-end. Vendor owns the application code, the ingestion pipeline, the prompts, the eval harness, and the artefacts that let the customer prove the software behaves as described. The FDE owns the data-flow diagram, the ML-BOM, the model-provenance attestations, and the engineering-grade clauses in the DPA.

**On-prem / air-gapped.** As with the LLM, retrieval, agentic, and eval layers, this archetype is rare but real. The vendor's artefacts (code, attestations, provenance) move through an approved transfer; model weights are scanned before load; everything runs behind the customer's firewall. The FDE's deliverables are the same as VPC install, with harder egress rules on every sub-processor call. Sub-processors that cannot be reached without egress are not viable in this archetype — which usually means the model is a local open-weights model (Chapter 05), the embedding model is local, and the eval judge is local.

The distinctive emphasis for security review: *every trust-boundary crossing has to have an evidence artefact, a key, and an audit line*. The DFD is the diagram that proves it. The DPA is the contract that commits to it. The ML-BOM / SBOM proves the software that enforces it was built the way the vendor claims. Chapter 02 sets the DFD discipline; Chapter 05 sets the contract; Chapter 06 sets the provenance.

## The three customer archetypes for the rest of the module

The same three archetypes used through mod-104 / 105 / 106 / 107, with the security-review surface filled in. Each sentence frames the archetype's security-review posture.

**Archetype A — Mid-size US healthcare payer.** Workflow: member-inquiry resolution. Compliance: HIPAA + SOC 2 Type II; state-level privacy (CCPA/CPRA if California members); NAIC Model Bulletin on AI use in insurance sits in scope. Deployment: AWS Bedrock + VPC install. Security review posture: SIG Lite + customer-house HIPAA addendum; BAA signed with vendor; sub-processor list disclosed per 45 CFR 164.308(b)(3); pen-test report under NDA; DFD shows PHI never leaves the customer's VPC; CMK on all storage; audit stream to customer Splunk with HIPAA-aligned retention; EU AI Act scope-check negative (US-only).

**Archetype B — US federal agency.** Workflow: policy-analyst drafting + evidence packaging. Compliance: FedRAMP Moderate with documented path to High; NIST 800-53 Rev 5 control baseline; NIST AI RMF; OMB M-24-10 / M-25-21 AI-use guidance; agency-specific ATO process. Deployment: GovCloud + self-hosted inference. Security review posture: SSP-aligned control statements; FIPS 140-2 / 140-3 validated cryptography end-to-end; no public-internet sub-processors; pen-test by a FedRAMP 3PAO; model weights scanned before transfer into GovCloud; ML-BOM in the ATO body of evidence; SLSA Level 2+ provenance on every release artifact.
<!-- needs-research: confirm the current FedRAMP ATO body-of-evidence outline and the latest OMB AI-use memo numbers before publishing; OMB memo numbering and FedRAMP structure have been iterating -->

**Archetype C — EU retail bank.** Workflow: customer-support draft reply + case triage. Compliance: GDPR (Articles 22, 32, 35 central), DORA, EU AI Act (Article 14 oversight; Article 72 post-market monitoring; high-risk classification depends on use), SR 11-7 model-risk-management discipline through the bank's model-risk function, Basel-style operational-risk framing. Deployment: Azure AI Foundry under EU Data Boundary. Security review posture: CAIQ v4 + bank-house GDPR addendum; sub-processor disclosure under GDPR Article 28; DPIA under Article 35 before go-live; CMK on all storage; Azure OpenAI zero-retention attestation; audit stream to Microsoft Sentinel with GDPR-aligned retention; EU AI Act Annex IV technical-documentation slice drafted alongside the model card.

Each chapter runs decisions through these three archetypes. The archetypes are shapes, not prescriptions.

## Where mod-108 sits with the rest of the track

The security and compliance overlay only makes sense in context.

- **Mod-101 (FDE discipline).** The escalation rule from Chapter 06 applies here especially — security review is where the FDE must know to hand off to platform peers rather than attempt cross-domain depth alone.
- **Mod-102 (customer discovery).** The pilot's data classes and the sponsor's regulatory reality were named at scoping. Mod-108 formalises them into a review-ready packet.
- **Mod-103 (enterprise integration).** The identity surface, the data surface, and the NHI inventory from mod-103 carry the audit-trail shape forward into Chapter 02's DFD.
- **Mod-104 (LLM applications).** The gateway is where the zero-retention and residency promises are architecturally enforced (Chapter 05).
- **Mod-105 (grounding).** The permission model, the PII vault, the retrieval audit trail are the evidence Chapter 02 attaches to the packet.
- **Mod-106 (agentic workflows).** The OWASP Agentic Top 10 coverage matrix built in mod-106 Chapter 06 is a direct input to Chapter 04; the audit trail is the audit the customer's SIEM will replay.
- **Mod-107 (eval).** The per-pilot decision document and the online-eval instrumentation are evidence artefacts for the governance-board sign-off step.
- **Mod-109 (deployment, rollout).** Security gates (threat-model sign-off, DPA executed, ML-BOM published) are the gates that block feature-flag flips.
- **Mod-110 (observability, on-call).** The audit stream specified in mod-108 is the stream mod-110 instruments and pages on.
- **Mod-111 (field → product loop).** Reusable questionnaire-response fragments, DFD templates, and threat-model slices flow back to the product team.
- **Mod-112 (consulting craft).** How the DPA redline conversation, the DPIA walkthrough, and the governance-board decision are narrated to the customer's executive sponsor.

## Anti-patterns

**The paper-trail pilot.** A SOC 2 Type II report and a ten-page master DPA are not an evidence packet. They are the *base layer* on top of which the per-pilot packet is built — DFD, sub-processor list, threat model, residual risk, model card, ML-BOM. A customer's CISO will read the base layer in five minutes and then ask for the per-pilot artefacts. Chapter 02 sets the per-pilot discipline.

**Zero-retention as a slide, not an architecture.** Writing "we honour zero-retention with the model provider" in the DPA is not the same as actually routing every prompt through a gateway configuration whose storage class is "none" and whose audit log records that it was. Chapter 05 fixes the architectural side of the sentence — so the DPA clause is auditable, not aspirational.

**Threat model as a one-shot artifact.** Running the OWASP matrix once at design-review and never touching it again. Every new tool, memory surface, sub-processor, or data class re-opens the model. Chapter 04 insists on versioning.

**Vertical regime guessed, not confirmed.** Assuming HIPAA does not apply because the data "isn't technically PHI" when the workflow routinely sees member-identifying context; assuming the EU AI Act does not apply because the model "isn't high-risk" without walking Annex III; assuming CCPA does not apply because the customer is a B2B. Chapter 03's discipline is to walk the regime and *document* why it does or does not apply. Guessing is not a defence.

**Sub-processor list as a trailing afterthought.** The customer's privacy office will not approve a BAA / DPA that is missing a sub-processor currently in the pipeline. "We'll add the embedding API vendor later" is how a two-week review turns into a six-week review. Chapter 02 keeps the list current.

**Supply-chain provenance left for later.** The ML-BOM is not a stretch goal; it is table stakes for every FedRAMP, DORA, and ISO/IEC 42001-aligned customer, and increasingly for commercial enterprises too. Chapter 06 lands it on week one of the pilot.

**The FDE as the sole risk owner.** This module repeatedly cites peers — `ai-infra-security` / `security-learning`, `ai-risk-engineer`, `ai-evaluation-engineer`. If the pilot's security posture reads as *entirely* authored by the FDE, the pilot has silently absorbed a risk-owner role it should not hold. Chapter 07 writes the boundaries and the pager protocol.

## Summary

- The security and compliance overlay is the set of customer-environment concerns the FDE adds on top of the MLSec / platform-security craft (owned by `ai-infra-security` / `security-learning`, peer, level 35), the risk-engineering craft (owned by `ai-risk-engineer`, peer, level 25, Governance family), and the release-assurance craft (owned by `ai-evaluation-engineer`, peer, level 35, Governance family). Mod-108 does not re-teach any of the three; it layers at working depth.
- Fifteen dimensions are re-opened per customer — from vendor questionnaire posture through data-flow diagram, sub-processor disclosure, threat-model construction, BYOK posture, DPA engineering attestations, supply-chain provenance, and peer-boundary mapping. Each chapter in this module takes one or two.
- The organising budget is six-dimensional: evidence completeness, regulatory fit, data-contract cleanliness, threat-model coverage, supply-chain provenance, time-to-sign-off. Regulatory fit, data-contract cleanliness, and threat-model coverage are gates; the others are optimisations.
- Three deployment archetypes (SaaS multi-tenant, dedicated SaaS, VPC install, with on-prem / air-gapped as the rare case) shape where each trust-boundary crossing happens, which audit log catches it, and which key protects it.
- Three customer archetypes — US healthcare payer (HIPAA + BAA + VPC on AWS), US federal agency (FedRAMP + GovCloud + self-hosted inference), EU retail bank (GDPR + DORA + EU AI Act + Azure EU Data Boundary) — thread through every chapter.
- Seven anti-patterns to catch early: paper-trail pilot, zero-retention as a slide, one-shot threat model, vertical regime guessed, sub-processor list as afterthought, supply-chain for later, FDE as sole risk owner.
- Chapter 02 is next: answering SOC 2 / SIG / CAIQ / vendor-security questionnaires; assembling the per-pilot evidence packet; data-flow diagram; sub-processor disclosure; pen-test report sharing.
