# Chapter 03 — Vertical Compliance — HIPAA, FedRAMP, PCI, EU AI Act, GDPR, CCPA, and the State Patchwork

## Why this chapter exists

Chapter 02 handled the general security review every regulated customer runs — questionnaires, DFD, sub-processors, pen-test. This chapter handles the vertical slice: the specific statutes and regulatory regimes that apply because of what the workflow *does*, not only because the customer is an enterprise. A member-inquiry workflow for a US payer sits under HIPAA regardless of how clean the SOC 2 looks. A credit-scoring workflow for an EU bank sits under the EU AI Act regardless of the cloud provider. A screening workflow used for New York City employment decisions sits under LL144 regardless of the model's accuracy.

The FDE is not the customer's compliance officer. The FDE is the engineer whose implementation makes the customer's compliance officer's job possible. That means: knowing which regime applies, knowing which obligations route to the customer vs. the vendor vs. this role, naming the engineering-level commitments each regime demands, and producing the artefacts each regime inspects.

Deep interpretive depth on any single regime — statutory analysis, regulator-opinion history, litigation posture — is work for the customer's compliance counsel and the vendor's legal function. The FDE's depth here is working: enough to spot a scope mismatch, write the DPA clause, route the question, and not ship a workflow that violates the regime by construction.

## The routing rule

Every obligation routes to one of three parties. The FDE's first job in a compliance conversation is to name the routing explicitly, so no obligation is dropped.

- **Customer-owned.** The customer is the data controller / covered entity / regulated entity. Obligations tied to those statutory roles stay with them. Examples: notice to data subjects, minimum necessary determinations, lawful basis selection, data-subject-rights fulfilment at first instance, DPIA ownership, regulator notification for customer-side incidents.
- **Vendor-owned (platform-level).** The vendor organisation's security, legal, and GRC teams own obligations tied to the vendor's statutory role (processor / business associate / service-provider). Examples: SOC 2 Type II, pen-test on the platform, master DPA terms, breach-notification SLA to customer, FedRAMP PATO sponsorship, ISO/IEC 27001 certification, platform-level encryption, BAA template.
- **This-role-owned (per-pilot, engineering layer).** The FDE owns obligations whose satisfaction lives in *this deployment's* configuration or code. Examples: the DFD, the per-pilot DPIA technical input, the zero-retention gateway configuration (Chapter 05), the OWASP threat-model matrix (Chapter 04), the ML-BOM and model card (Chapter 06), the training-opt-out verification for this account.

A clean per-regime matrix, done once per pilot, prevents the pattern where the customer's DPO assumes the vendor owns a clause and the vendor's legal team assumes the customer owns it.

## US healthcare: HIPAA

**The core regime.** [HIPAA Privacy, Security, and Breach Notification Rules (45 CFR Parts 160, 162, 164)](https://www.hhs.gov/hipaa/for-professionals/index.html) apply when a vendor processes Protected Health Information (PHI) on behalf of a covered entity (health plan, healthcare provider, healthcare clearinghouse) or another business associate.

**Routing.**

- *Customer (covered entity):* Notice of Privacy Practices, minimum-necessary determinations, patient-rights handling, OCR notifications for customer-side incidents.
- *Vendor (business associate):* execute the BAA (45 CFR 164.504(e)); implement administrative, physical, and technical safeguards under the Security Rule (45 CFR 164.308 – 164.312); breach-notification SLA to the covered entity on becoming aware of a breach of unsecured PHI; retain policies and procedures for six years from the date of creation or the date when last in effect, whichever is later (45 CFR 164.316(b)(2)). Watch the HHS Security Rule NPRM (published 2025-01-06; comments closed 2025-03-07) for pending changes that would tighten MFA, encryption-at-rest, and testing cadence requirements.
- *This role (per-pilot):* configure zero-retention on the model provider for PHI in prompts; keep PHI inside the customer's VPC; audit-log every PHI-touching action to the customer's SIEM with 6-year retention aligned to the customer's HIPAA policy; implement access controls that compose with the covered entity's workforce access.

**Engineering-level commitments to write into the BAA / DPA.**

- "No PHI is used to train vendor's or third-party models."
- "PHI in prompts and completions is not retained by the model provider beyond the request lifecycle." Cite the model provider's zero-retention attestation.
- "Audit logs of all PHI access are streamed to the customer's SIEM within [N] minutes."
- "Sub-processors handling PHI have executed BAAs with the vendor; the current list is appended."
- "Breach notification follows the 60-day outer limit from discovery (45 CFR 164.410)."

**The Omnibus Rule and sub-processor BAAs.** Under the HITECH Omnibus Rule, business associates must have BAAs with their own sub-processors that touch PHI. The FDE's sub-processor list from Chapter 02 is read in BAA terms here — every row that touches PHI must have a signed BAA.

**Vertical-adjacent regimes.** The NAIC Model Bulletin on the Use of Artificial Intelligence Systems by Insurers (adopted by the NAIC in December 2023) applies to insurer customers once the relevant state DOI adopts it; approximately half the states plus DC have adopted as of late 2026, with California, Colorado, New York, and Texas running their own rules instead. The FDE's working posture: ask the customer which state DOIs apply; attach the customer's answer to the DPIA.

## US federal: FedRAMP + OMB AI-use guidance

**The core regime.** [FedRAMP](https://www.fedramp.gov/) is the federal civilian government's standardised authorisation program for cloud services. Baselines: Low, Moderate, High (plus Tailored). Each baseline is a subset of [NIST 800-53 Rev 5](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final) controls. DoD workloads sit under the DoD Cloud Computing SRG (Impact Levels 2, 4, 5, 6).

**Routing.**

- *Customer (federal agency):* Authority to Operate (ATO) under agency authorizing official; customer-controlled implementation of NIST 800-53 controls assigned to them; agency-specific AI governance.
- *Vendor (CSP):* FedRAMP PATO / Agency ATO sponsorship; control-implementation statements for vendor-controlled controls in the SSP; annual continuous monitoring (ConMon); POA&M maintenance; 3PAO-conducted annual assessment; FIPS 140-2 / 140-3 validated cryptography.
- *This role (per-pilot):* control-implementation details for the pilot's deployment that fit inside the vendor's PATO boundary (or establish a new boundary); DFD and inventory updates; ML-BOM with SLSA Level 2+ provenance; model-weight integrity attestation; incident-response alignment with the agency SOC.

**OMB AI-use guidance.** M-24-10 "Advancing Governance, Innovation, and Risk Management for Agency Use of Artificial Intelligence" (March 2024) was rescinded and replaced on 2025-04-03 by [M-25-21](https://www.whitehouse.gov/wp-content/uploads/2025/02/M-25-21-Accelerating-Federal-Use-of-AI-through-Innovation-Governance-and-Public-Trust.pdf) "Accelerating Federal Use of AI through Innovation, Governance, and Public Trust," with procurement companion [M-25-22](https://www.whitehouse.gov/wp-content/uploads/2025/04/M-25-22-Driving-Efficient-Acquisition-of-Artificial-Intelligence-in-Government.pdf). Together they require agency-level AI governance, use-case inventories, minimum risk-management practices for "high-impact" AI (the M-25-21 consolidation of the earlier rights-impacting and safety-impacting categories), and chief AI officer assignments. The vendor's pilot must enable the agency's CAIO to populate the inventory and attest to the minimum practices.

**Engineering-level commitments.**

- "All cryptography uses FIPS 140-2 or 140-3 validated modules; the certificate numbers are attached."
- "The pilot runs entirely within the customer's GovCloud boundary; no public-internet endpoints are called."
- "Every release artifact carries SLSA Level 2+ provenance; the attestation is signed by Sigstore."
- "Model weights were scanned with ModelScan / picklescan before transfer into GovCloud; results are attached."
- "The system is covered by the vendor's FedRAMP [Moderate|High] PATO / ATO at [agency]."

**High-impact AI (M-25-21).** M-25-21 collapsed the earlier M-24-10 "rights-impacting" and "safety-impacting" categories into a single "high-impact" category (output serves as a principal basis for decisions or actions that have a legal, material, binding, or significant effect on rights or safety). For high-impact uses the agency must apply minimum risk-management practices, including pre-deployment testing, an AI impact assessment maintained throughout deployment, ongoing monitoring, remedies or appeals for affected individuals, and public use-case inventory; non-compliant use must be paused or stopped. The FDE's job is to produce the engineering evidence those practices rely on — not to run the practices themselves (that is the agency's).

## Payments: PCI-DSS

**The core regime.** [PCI-DSS v4.0.1](https://www.pcisecuritystandards.org/) applies when the pilot processes, stores, or transmits cardholder data (CHD) or sensitive authentication data (SAD) on behalf of a merchant / acquirer / processor.

**The FDE's working rule: avoid scope.** The best PCI-DSS posture for a non-payments-native pilot is *not to be in scope*. A workflow that never sees a primary account number (PAN) does not touch the cardholder-data environment (CDE) and does not inherit the 300+ requirements. Techniques to stay out of scope:

- **Tokenisation at ingest.** The customer's payments system replaces PAN with a token *before* data flows into the pilot. The pilot sees tokens, not PANs.
- **Field-level redaction in the ingestion pipeline.** The customer's ETL strips PAN / CVV / full track data.
- **No CHD in prompts.** The prompt architecture (mod-104 Chapter 04) explicitly fences cardholder data out.

If the pilot *is* in scope (the workflow requires processing CHD), route the obligation clearly:

- *Customer:* merchant-level PCI-DSS compliance; CDE boundary definition; qualified security assessor (QSA) engagement; reports on compliance (RoC) / self-assessment questionnaires (SAQ).
- *Vendor:* AoC (Attestation of Compliance) as a service provider; PCI-DSS controls for the vendor's segment of the CDE; annual on-site assessment or SAQ D-SP.
- *This role:* encryption of PAN at rest (AES-256 or equivalent); masking for display (first-six / last-four); DLP in logs; no PAN written to the model provider under any circumstances.

**Engineering-level commitments.**

- "The vendor does not store PAN, CVV, or full track data; the pilot's ingestion pipeline redacts these at the customer's boundary."
- "If storage is required by the workflow, PAN is encrypted at rest using AES-256 with keys managed by the customer's KMS; key rotation is at [customer-set cadence]."
- "Logs are scrubbed; a sample is attached showing no PAN appears in audit output."

## EU: EU AI Act

**The core regime.** [Regulation (EU) 2024/1689 (EU AI Act)](https://eur-lex.europa.eu/eli/reg/2024/1689/oj) entered into force 2024-08-01 with phased application (Article 113). Article 5 prohibitions and general provisions apply from 2025-02-02; GPAI obligations from 2025-08-02; general application from 2026-08-02; obligations for Annex III high-risk systems have been deferred to 2027-12-02 under the Commission's Digital Omnibus package; Annex I product-safety high-risk systems (e.g., toys, lifts) apply from 2028-08-02. Dates continue to shift — verify Article 113 and the current Omnibus text before committing to a pilot timeline.

**Risk classification.**

- **Unacceptable risk (Article 5).** Prohibited practices: subliminal / manipulative / deceptive techniques that materially distort behaviour and cause harm; exploitation of vulnerabilities of age, disability, or social / economic situation; social scoring by public or private actors leading to unjustified detrimental treatment; predictive policing based solely on profiling an individual; untargeted scraping of facial images to build recognition databases; emotion inference in workplace or education contexts; biometric categorisation inferring sensitive attributes; real-time remote biometric identification in public spaces by law enforcement (narrow exceptions). A pilot that touches these does not ship.
- **High-risk (Article 6 and Annex III).** Systems used in listed domains — biometrics, critical infrastructure, education, employment, essential services (incl. credit scoring and insurance risk), law enforcement, migration, administration of justice, democratic processes — unless an Annex III exception applies. These attract the full Chapter III Section 2 obligations.
- **Limited risk (Article 50).** Transparency obligations: inform users they are interacting with an AI; mark AI-generated / manipulated content; emotion-recognition and biometric-categorisation disclosures.
- **Minimal risk.** No specific obligations.
- **GPAI model obligations (Articles 51–55).** Apply to general-purpose AI model providers (the model-provider sub-processor, typically — not the pilot vendor unless the vendor trains models). Under Article 51(2) a GPAI model is presumed to have "high-impact capabilities" (systemic risk) when the cumulative compute used for its training measured in floating point operations exceeds 10^25 — a rebuttable presumption the Commission can adjust by delegated act — and additional obligations (Article 55) then attach.

**Articles 9–15 for high-risk systems (the engineering-dense ones).**

- **Article 9 — Risk management system.** A documented, iterative risk-management process across the system's lifecycle. Routes to: *this role* for identification and mitigation of pilot-specific risks (threat-model from Chapter 04 is a direct input); *vendor platform* for the risk-management policy; *customer* for operating-environment risk.
- **Article 10 — Data and data governance.** Training, validation, and testing data sets subject to appropriate data-governance practices. Mostly routes to the GPAI model provider for pre-training data; routes to *this role* for pilot-specific fine-tuning / retrieval-augmentation data; routes to *customer* for the data they supply.
- **Article 11 — Technical documentation.** The Annex IV documentation package: system description, design choices, data, training / testing / validation procedures, risk-management system, lifecycle changes, monitoring. Owned by *vendor* composed with *this role* — mod-107 Chapter 06 and this module's Chapter 06 produce the components.
- **Article 12 — Record-keeping.** Automatic recording of events ("logs") over the lifetime of the system. Routes to *this role* for log capture and retention; *customer* for log review and retention policy.
- **Article 13 — Transparency and provision of information to deployers.** Clear instructions for use. Routes to *vendor + this role* for the user-facing documentation; *customer* for operator training.
- **Article 14 — Human oversight.** The system must be designed such that it can be effectively overseen by natural persons. Routes to *this role* via the HITL design from mod-106 Chapter 04; *customer* for operator training and oversight staffing.
- **Article 15 — Accuracy, robustness, and cybersecurity.** The system must achieve appropriate levels of accuracy, robustness, and cybersecurity throughout its lifecycle. Routes to *this role* via the threat-model, mitigations, and eval artefacts; composes with `ai-infra-security` for platform-level robustness.

**Article 72 — Post-market monitoring.** Providers of high-risk AI systems must establish and document a post-market monitoring system proportionate to the AI system's risks; data it collects must be analysed. Under Article 73 serious incidents must be reported to the market-surveillance authority of the Member State where the incident occurred: within 15 days of becoming aware by default; within 10 days where a death is causally linked or reasonably suspected; within 2 days for widespread infringement or serious and irreversible disruption of critical infrastructure. The FDE's job: produce the engineering evidence that feeds the post-market monitoring system (mod-110 observability composes directly). Regulator engagement is the customer's and the `ai-evaluation-engineer` peer's work.

## EU: GDPR

**The core regime.** [Regulation (EU) 2016/679 (GDPR)](https://eur-lex.europa.eu/eli/reg/2016/679/oj) applies to processing of personal data of EU data subjects.

**Article 22 — Automated individual decision-making, including profiling.** A data subject has the right not to be subject to a decision based solely on automated processing, including profiling, which produces legal or similarly significant effects. *Routing:* customer, as controller, determines whether the pilot's decisions are "solely automated" or human-in-the-loop decisions; the FDE's job is to make the HITL design (mod-106 Chapter 04) auditable enough to support the controller's determination.

**Article 32 — Security of processing.** Controller and processor implement appropriate technical and organisational measures to ensure a level of security appropriate to the risk. *Routing:* vendor as processor (platform-level), this role (per-pilot architecture), customer (data-at-controller).

**Article 35 — Data protection impact assessment (DPIA).** Where a type of processing — in particular using new technologies — is likely to result in a high risk to natural persons' rights and freedoms, the controller carries out a DPIA prior to processing. AI workflows almost always trip Article 35(3). *Routing:* customer owns the DPIA; vendor supplies technical inputs; this role writes the engineering slice (data flows, categories, retention, measures). The EDPB's DPIA guidance ([Article 29 WP, WP248 rev.01](https://ec.europa.eu/newsroom/article29/items/611236)) is the standard reference.

**Cross-border transfer analysis.** Any flow of personal data to a non-EEA country needs an Article 46 transfer mechanism (SCCs most commonly) plus a transfer impact assessment (TIA, per the EDPB's [Recommendations 01/2020 on measures that supplement transfer tools](https://www.edpb.europa.eu/system/files/2021-06/edpb_recommendations_202001vo.2.0_supplementarymeasurestransferstools_en.pdf), final version v2.0 adopted June 2021 following Schrems II). If the model provider or any sub-processor sits outside the EEA, the FDE's job is to map the flow for the customer's DPO.

**Engineering-level commitments.**

- "Personal data is processed only in [EU region]; the DFD confirms no outbound call."
- "Pseudonymisation is applied where feasible under Article 32(1)(a); the vault is in the customer's KMS."
- "A DPIA technical input is attached; it names the lawful basis, processing purposes, categories of data subjects, categories of personal data, recipients, retention, and technical measures."

## US state laws

**CCPA / CPRA (California).** Applies to businesses doing business in California meeting threshold criteria. Routing: customer is the business; vendor is a "service provider" or "contractor" under the CCPA service-provider contract requirements (Cal. Civ. Code § 1798.140; CCPA Regulations § 7051, amended effective 2026-01-01, enumerates the required contract terms). Service-provider contracts must restrict the vendor's use of personal information to the business purposes specified in the contract, prohibit selling or sharing, and pass the obligations through to subcontractors. The FDE's engineering commitment is the training-opt-out and no-secondary-use attestation (Chapter 05).

**NYC LL144 (Local Law 144 of 2021).** Applies to automated employment decision tools (AEDTs) used in NYC to screen candidates or employees. NYC DCWP issued the Final Rule on 2023-04-06 and began enforcement on 2023-07-05. Requires an annual independent bias audit, posting the audit summary, and candidate notice. Routing: customer (the employer) runs the audit and the notice; vendor supplies the AEDT's statistical outputs in the audit shape; this role implements the data capture (prediction, candidate demographic data where available) that the audit consumes. Skilled bias-audit methodology is `ai-risk-engineer` / `ai-evaluation-engineer` territory; the FDE produces the engineering evidence.

**Colorado AI Act (SB 24-205).** Applies to "developers" and "deployers" of "high-risk artificial intelligence systems" making "consequential decisions" (employment, education, financial services, healthcare, housing, insurance, legal services, government services) in Colorado. Originally slated to take effect 2026-02-01; the effective date was pushed to 2026-06-30 by SB25B-004 and then superseded by SB26-189 (May 2026), which replaced the framework with a new effective date of 2027-01-01. Obligations include risk-management programs, impact assessments, and consumer notice. Routing: customer (deployer) runs the deployer obligations; vendor (developer) runs developer obligations; this role produces the technical documentation and impact-assessment inputs.

**Other state regimes to recognise.** Illinois BIPA (biometrics; strict consent and private right of action); Washington My Health My Data Act (consumer health data broader than HIPAA); Texas Data Privacy and Security Act; Connecticut, Virginia, Utah, Oregon, and others — a growing patchwork. The FDE's working posture: ask the customer which state regimes apply; map each to the routing table; flag the gaps for the customer's counsel.

## The per-pilot vertical compliance matrix

The artefact the FDE produces once per pilot, read by the customer's privacy office, the customer's compliance officer, and the vendor's legal team:

| Regime | Applies? | Scope / why | Customer owns | Vendor owns | This role owns | Evidence |
|---|---|---|---|---|---|---|
| HIPAA | Y | Member PHI in prompts | NPP, min-necessary, OCR notifications | BAA, Security Rule safeguards, breach SLA | VPC install, zero-retention, PHI audit stream to Splunk, 6-year retention | BAA, DPA-HIPAA addendum, DFD, sub-proc list with BAAs |
| FedRAMP | N | Non-federal | — | — | — | — |
| EU AI Act | N | US-only workflow | — | — | — | — |
| GDPR | N | No EU data subjects | — | — | — | — |
| CCPA / CPRA | Y (if CA members) | California members | Business determination | Service-provider contract | No-secondary-use, training-opt-out | DPA § service-provider clause |
| NAIC AI Model Bulletin | Y | Insurer workflow | AI governance program | — | Model card, ML-BOM, risk assessment input | Model card, risk register |

That table, filled in differently for each archetype, is a one-page artefact the FDE attaches to the Chapter 02 evidence packet.

## Running the three archetypes

**Archetype A — US healthcare payer.** HIPAA applies throughout (BAA, Security Rule, breach SLA). CCPA/CPRA applies to California members. NAIC AI Model Bulletin applies if the state DOI has adopted it. No EU, FedRAMP, or PCI unless the workflow extends.

**Archetype B — US federal agency.** FedRAMP applies (boundary + SSP + continuous ATO). OMB AI-use guidance applies (use-case inventory, minimum practices if rights-impacting). HIPAA may apply if the agency's workflow involves PHI (e.g., VA, HHS). Privacy Act of 1974 applies if personal information about US citizens is maintained in a system of records.

**Archetype C — EU retail bank.** GDPR applies (Articles 22, 32, 35; TIAs on any non-EEA hop). EU AI Act applies (likely high-risk if the workflow touches creditworthiness assessment or credit scoring under Annex III point 5(b), which carves out AI used to detect financial fraud; otherwise Article 50 transparency). DORA applies (ICT-third-party register; exit plan; incident classification; penetration-testing on critical functions). SR 11-7 discipline applies through the bank's model-risk function. National-level supervisory regime applies (ECB SSM for large banks).

## Anti-patterns

**"We're not in scope because we told the customer so."** Scope determinations belong to the customer's compliance office, informed by the vendor's data-flow and the vendor's statutory role. The FDE drafts the technical facts; the customer concludes.

**One global DPA, one global BAA, one global FedRAMP ATO, no per-pilot overlay.** The master instruments are the base; each pilot still needs the per-deployment addendum or SSP delta. Chapter 05 details the DPA shape.

**Treating the EU AI Act as "a model provider problem."** Article 26 sets out deployer obligations for high-risk AI systems (use per the provider's instructions; competent human oversight; relevant input data; monitoring and incident reporting; automatic logs retained at least six months; worker notice where used in the workplace). Article 25 further warns that a deployer can be reclassified as a provider — and inherit the full provider obligations — by placing the system under its own trademark, substantially modifying a high-risk system, or changing the intended purpose of a non-high-risk system in a way that makes it high-risk. The customer is a deployer; the FDE's job is to produce artefacts the customer can use to meet deployer obligations. Pointing at Anthropic / OpenAI / Google does not discharge the customer.

**Walking into a PCI scope.** A pilot that could easily be out of scope ends up in scope because no one reviewed what lands in the prompt. Fence card data at ingest; keep the pilot's scope artefactually zero where possible.

**LL144 / Colorado / state laws dropped on the floor.** Treating state regimes as "the customer's problem" when the engineering evidence is the only way the customer can meet them. Produce the output records, the impact-assessment inputs, the demographic-outcome breakdowns.

**Zero-retention promise with a model provider that quietly retains for abuse monitoring.** OpenAI retains abuse-monitoring logs for up to 30 days by default and excludes customer content from those logs for eligible enterprise customers under its Zero Data Retention / Modified Abuse Monitoring programs. Anthropic's enterprise retention posture shifted in September 2026 to Enterprise Frontier Safeguards, which pair ZDR with misuse-monitoring data stored in cloud infrastructure the *customer* controls. The FDE must know each provider's current default *and* the specific addendum or program that modifies it for this account, and name both in the DPA — the terms change release-to-release.

## Summary

- Vertical compliance is routed in a three-column matrix: customer (statutory role), vendor (platform-level), this role (per-pilot engineering).
- HIPAA attaches when PHI is processed on behalf of a covered entity; BAA, Security Rule safeguards, 60-day breach-notification SLA, 6-year retention of policies and procedures.
- FedRAMP attaches to federal workloads; baselines map to NIST 800-53 Rev 5; FIPS-validated cryptography, SLSA-attested artefacts, and GovCloud boundary; OMB AI-use memos add minimum practices for rights-impacting / safety-impacting AI.
- PCI-DSS: the first move is to stay out of scope via tokenisation / redaction at ingest; in scope, encrypt PAN at rest, mask for display, never send to the model provider.
- EU AI Act: walk Article 5 (prohibited), Article 6 + Annex III (high-risk), Articles 9–15 (engineering-dense obligations), Article 50 (transparency for limited-risk), Articles 51–55 (GPAI), Article 72 (post-market monitoring).
- GDPR: Article 22 (automated decision-making — HITL design is the lever), Article 32 (security of processing), Article 35 (DPIA — customer owns; FDE supplies technical inputs), cross-border transfers under Article 46 with TIA.
- US state patchwork: CCPA/CPRA service-provider restrictions; NYC LL144 bias audit for employment AEDTs; Colorado AI Act consequential-decision obligations; BIPA, WA My Health My Data, state-by-state consumer privacy regimes.
- The deliverable is a one-page per-pilot vertical compliance matrix, read by customer compliance, vendor legal, and the vendor's engineering team.
- Chapter 04 is next: OWASP LLM Top 10, OWASP Agentic Top 10 recap, MITRE ATLAS tactics, and the mitigation inventory the AppSec reviewer expects before go-live.
