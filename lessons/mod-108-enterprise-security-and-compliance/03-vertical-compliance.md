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
- *Vendor (business associate):* execute the BAA (45 CFR 164.504(e)); implement administrative, physical, and technical safeguards under the Security Rule (45 CFR 164.308 – 164.312); breach-notification SLA to the covered entity on becoming aware of a breach of unsecured PHI; retain policies and procedures for a minimum of six years from creation or last-effective-date, whichever is later (45 CFR 164.316(b)(2)).
<!-- needs-research: confirm wording of 45 CFR 164.316 retention requirements and the current HIPAA NPRM status on security-rule updates (2024-2026 NPRM activity) before publishing -->
- *This role (per-pilot):* configure zero-retention on the model provider for PHI in prompts; keep PHI inside the customer's VPC; audit-log every PHI-touching action to the customer's SIEM with 6-year retention aligned to the customer's HIPAA policy; implement access controls that compose with the covered entity's workforce access.

**Engineering-level commitments to write into the BAA / DPA.**

- "No PHI is used to train vendor's or third-party models."
- "PHI in prompts and completions is not retained by the model provider beyond the request lifecycle." Cite the model provider's zero-retention attestation.
- "Audit logs of all PHI access are streamed to the customer's SIEM within [N] minutes."
- "Sub-processors handling PHI have executed BAAs with the vendor; the current list is appended."
- "Breach notification follows the 60-day outer limit from discovery (45 CFR 164.410)."

**The Omnibus Rule and sub-processor BAAs.** Under the HITECH Omnibus Rule, business associates must have BAAs with their own sub-processors that touch PHI. The FDE's sub-processor list from Chapter 02 is read in BAA terms here — every row that touches PHI must have a signed BAA.

**Vertical-adjacent regimes.** NAIC Model Bulletin on the Use of Artificial Intelligence by Insurers applies to insurer customers; state DOI adoptions differ (several state DOIs have adopted or are adopting it). The FDE's working posture: ask the customer which state DOIs apply; attach the customer's answer to the DPIA.
<!-- needs-research: confirm NAIC Model Bulletin on AI Systems by Insurers title and adoption-by-state status before publishing (Model Bulletin #4 and successor activity in 2024-2026) -->

## US federal: FedRAMP + OMB AI-use guidance

**The core regime.** [FedRAMP](https://www.fedramp.gov/) is the federal civilian government's standardised authorisation program for cloud services. Baselines: Low, Moderate, High (plus Tailored). Each baseline is a subset of [NIST 800-53 Rev 5](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final) controls. DoD workloads sit under the DoD Cloud Computing SRG (Impact Levels 2, 4, 5, 6).

**Routing.**

- *Customer (federal agency):* Authority to Operate (ATO) under agency authorizing official; customer-controlled implementation of NIST 800-53 controls assigned to them; agency-specific AI governance.
- *Vendor (CSP):* FedRAMP PATO / Agency ATO sponsorship; control-implementation statements for vendor-controlled controls in the SSP; annual continuous monitoring (ConMon); POA&M maintenance; 3PAO-conducted annual assessment; FIPS 140-2 / 140-3 validated cryptography.
- *This role (per-pilot):* control-implementation details for the pilot's deployment that fit inside the vendor's PATO boundary (or establish a new boundary); DFD and inventory updates; ML-BOM with SLSA Level 2+ provenance; model-weight integrity attestation; incident-response alignment with the agency SOC.

**OMB AI-use guidance.** Executive and OMB memos ([M-24-10](https://www.whitehouse.gov/wp-content/uploads/2024/03/M-24-10-Advancing-Governance-Innovation-and-Risk-Management-for-Agency-Use-of-Artificial-Intelligence.pdf) "Advancing Governance, Innovation, and Risk Management for Agency Use of Artificial Intelligence" and successor memos) require agency-level AI governance, use-case inventories, minimum risk-management practices for rights-impacting and safety-impacting AI, and chief AI officer assignments. The vendor's pilot must enable the agency's CAIO to populate the inventory and attest to the minimum practices.
<!-- needs-research: confirm current OMB AI-use memo numbering (M-24-10 and successors through 2025-2026) before publishing; the series continues to evolve -->

**Engineering-level commitments.**

- "All cryptography uses FIPS 140-2 or 140-3 validated modules; the certificate numbers are attached."
- "The pilot runs entirely within the customer's GovCloud boundary; no public-internet endpoints are called."
- "Every release artifact carries SLSA Level 2+ provenance; the attestation is signed by Sigstore."
- "Model weights were scanned with ModelScan / picklescan before transfer into GovCloud; results are attached."
- "The system is covered by the vendor's FedRAMP [Moderate|High] PATO / ATO at [agency]."

**Rights-impacting and safety-impacting AI (M-24-10 §5).** If the workflow is rights-impacting or safety-impacting, the agency must apply minimum practices: AI impact assessment, real-world performance testing, ongoing monitoring, consultation with affected communities, and public use-case inventory. The FDE's job is to produce the engineering evidence those practices rely on — not to run the practices themselves (that is the agency's).
<!-- needs-research: confirm M-24-10 §5 minimum-practices exact list and the companion data-governance requirements; verify M-25-21 (if still extant) and any successor numbering -->

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

**The core regime.** [Regulation (EU) 2024/1689 (EU AI Act)](https://eur-lex.europa.eu/eli/reg/2024/1689/oj) entered into force 2024-08-01 with phased application. Prohibitions apply from 2025-02-02; GPAI obligations from 2025-08-02; high-risk obligations from 2026-08-02; limited extensions for Annex I high-risk systems to 2027-08-02.
<!-- needs-research: confirm each application date from the official EUR-Lex text before publishing — dates have been repeated incorrectly in secondary sources -->

**Risk classification.**

- **Unacceptable risk (Article 5).** Prohibited practices: social scoring by public authorities, exploitation of vulnerabilities, biometric categorisation for sensitive attributes in most contexts, real-time remote biometric identification in public spaces by law enforcement (narrow exceptions), workplace / education emotion inference, predictive policing based solely on profiling, untargeted scraping of facial images. A pilot that touches these does not ship.
<!-- needs-research: confirm each Article 5 prohibition's exact wording before publishing; Article 5 is the most-litigated article pre-application -->
- **High-risk (Article 6 and Annex III).** Systems used in listed domains — biometrics, critical infrastructure, education, employment, essential services (incl. credit scoring and insurance risk), law enforcement, migration, administration of justice, democratic processes — unless an Annex III exception applies. These attract the full Chapter III Section 2 obligations.
- **Limited risk (Article 50).** Transparency obligations: inform users they are interacting with an AI; mark AI-generated / manipulated content; emotion-recognition and biometric-categorisation disclosures.
- **Minimal risk.** No specific obligations.
- **GPAI model obligations (Articles 51–55).** Apply to general-purpose AI model providers (the model-provider sub-processor, typically — not the pilot vendor unless the vendor trains models). Systemic-risk thresholds (Article 51(2)) and additional obligations (Article 55) attach to models exceeding ~10^25 FLOPs training compute.
<!-- needs-research: confirm Article 51(2) systemic-risk threshold wording and the latest Commission delegated acts adjusting it before publishing -->

**Articles 9–15 for high-risk systems (the engineering-dense ones).**

- **Article 9 — Risk management system.** A documented, iterative risk-management process across the system's lifecycle. Routes to: *this role* for identification and mitigation of pilot-specific risks (threat-model from Chapter 04 is a direct input); *vendor platform* for the risk-management policy; *customer* for operating-environment risk.
- **Article 10 — Data and data governance.** Training, validation, and testing data sets subject to appropriate data-governance practices. Mostly routes to the GPAI model provider for pre-training data; routes to *this role* for pilot-specific fine-tuning / retrieval-augmentation data; routes to *customer* for the data they supply.
- **Article 11 — Technical documentation.** The Annex IV documentation package: system description, design choices, data, training / testing / validation procedures, risk-management system, lifecycle changes, monitoring. Owned by *vendor* composed with *this role* — mod-107 Chapter 06 and this module's Chapter 06 produce the components.
- **Article 12 — Record-keeping.** Automatic recording of events ("logs") over the lifetime of the system. Routes to *this role* for log capture and retention; *customer* for log review and retention policy.
- **Article 13 — Transparency and provision of information to deployers.** Clear instructions for use. Routes to *vendor + this role* for the user-facing documentation; *customer* for operator training.
- **Article 14 — Human oversight.** The system must be designed such that it can be effectively overseen by natural persons. Routes to *this role* via the HITL design from mod-106 Chapter 04; *customer* for operator training and oversight staffing.
- **Article 15 — Accuracy, robustness, and cybersecurity.** The system must achieve appropriate levels of accuracy, robustness, and cybersecurity throughout its lifecycle. Routes to *this role* via the threat-model, mitigations, and eval artefacts; composes with `ai-infra-security` for platform-level robustness.

**Article 72 — Post-market monitoring.** Providers of high-risk AI systems must establish and document a post-market monitoring system proportionate to the AI system's risks; data it collects must be analysed, and serious incidents must be reported to the market-surveillance authority (Article 73) within specific timelines. The FDE's job: produce the engineering evidence that feeds the post-market monitoring system (mod-110 observability composes directly). Regulator engagement is the customer's and the `ai-evaluation-engineer` peer's work.
<!-- needs-research: confirm Article 72 post-market monitoring requirements and Article 73 serious-incident notification timelines (15-day / 10-day / immediate per category) before publishing -->

## EU: GDPR

**The core regime.** [Regulation (EU) 2016/679 (GDPR)](https://eur-lex.europa.eu/eli/reg/2016/679/oj) applies to processing of personal data of EU data subjects.

**Article 22 — Automated individual decision-making, including profiling.** A data subject has the right not to be subject to a decision based solely on automated processing, including profiling, which produces legal or similarly significant effects. *Routing:* customer, as controller, determines whether the pilot's decisions are "solely automated" or human-in-the-loop decisions; the FDE's job is to make the HITL design (mod-106 Chapter 04) auditable enough to support the controller's determination.

**Article 32 — Security of processing.** Controller and processor implement appropriate technical and organisational measures to ensure a level of security appropriate to the risk. *Routing:* vendor as processor (platform-level), this role (per-pilot architecture), customer (data-at-controller).

**Article 35 — Data protection impact assessment (DPIA).** Where a type of processing — in particular using new technologies — is likely to result in a high risk to natural persons' rights and freedoms, the controller carries out a DPIA prior to processing. AI workflows almost always trip Article 35(3). *Routing:* customer owns the DPIA; vendor supplies technical inputs; this role writes the engineering slice (data flows, categories, retention, measures). The EDPB's DPIA guidance ([Article 29 WP, WP248 rev.01](https://ec.europa.eu/newsroom/article29/items/611236)) is the standard reference.

**Cross-border transfer analysis.** Any flow of personal data to a non-EEA country needs an Article 46 transfer mechanism (SCCs most commonly) plus a transfer impact assessment (TIA, per the EDPB's Schrems II recommendations). If the model provider or any sub-processor sits outside the EEA, the FDE's job is to map the flow for the customer's DPO.
<!-- needs-research: confirm EDPB Recommendations 01/2020 on measures that supplement transfer tools latest version before publishing -->

**Engineering-level commitments.**

- "Personal data is processed only in [EU region]; the DFD confirms no outbound call."
- "Pseudonymisation is applied where feasible under Article 32(1)(a); the vault is in the customer's KMS."
- "A DPIA technical input is attached; it names the lawful basis, processing purposes, categories of data subjects, categories of personal data, recipients, retention, and technical measures."

## US state laws

**CCPA / CPRA (California).** Applies to businesses doing business in California meeting threshold criteria. Routing: customer is the business; vendor is a "service provider" or "contractor" under the CCPA service-provider contract requirements (Cal. Civ. Code § 1798.140(ag); regulations by CPPA). Service-provider contracts must restrict the vendor's use of personal information to the business purpose. The FDE's engineering commitment is the training-opt-out and no-secondary-use attestation (Chapter 05).
<!-- needs-research: confirm CPRA regulations (CPPA final text, effective dates) and service-provider / contractor contract content requirements before publishing -->

**NYC LL144 (Local Law 144 of 2021).** Applies to automated employment decision tools (AEDTs) used in NYC to screen candidates or employees. Requires an annual independent bias audit, posting the audit summary, and candidate notice. Routing: customer (the employer) runs the audit and the notice; vendor supplies the AEDT's statistical outputs in the audit shape; this role implements the data capture (prediction, candidate demographic data where available) that the audit consumes. Skilled bias-audit methodology is `ai-risk-engineer` / `ai-evaluation-engineer` territory; the FDE produces the engineering evidence.
<!-- needs-research: confirm LL144 Final Rule from NYC DCWP, adoption date 2023-04-06 and enforcement from 2023-07-05 before publishing -->

**Colorado AI Act (SB 24-205).** Applies to "developers" and "deployers" of "high-risk artificial intelligence systems" making "consequential decisions" (employment, education, financial services, healthcare, housing, insurance, legal services, government services) in Colorado. Effective date 2026-02-01 (as amended). Obligations include risk management programs, impact assessments, consumer notice. Routing: customer (deployer) runs the deployer obligations; vendor (developer) runs developer obligations; this role produces the technical documentation and impact-assessment inputs.
<!-- needs-research: confirm Colorado AI Act (CRS 6-1-1701 to 6-1-1707) current effective date and definitions of developer/deployer before publishing — the effective date was adjusted during the 2024-2025 legislative cycles -->

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

**Archetype C — EU retail bank.** GDPR applies (Articles 22, 32, 35; TIAs on any non-EEA hop). EU AI Act applies (likely high-risk if the workflow touches creditworthiness assessment under Annex III §5(b); otherwise Article 50 transparency). DORA applies (ICT-third-party register; exit plan; incident classification; penetration-testing on critical functions). SR 11-7 discipline applies through the bank's model-risk function. National-level supervisory regime applies (ECB SSM for large banks).
<!-- needs-research: confirm Annex III §5(b) "creditworthiness" wording and the scope of carve-outs (e.g., fraud detection) before publishing -->

## Anti-patterns

**"We're not in scope because we told the customer so."** Scope determinations belong to the customer's compliance office, informed by the vendor's data-flow and the vendor's statutory role. The FDE drafts the technical facts; the customer concludes.

**One global DPA, one global BAA, one global FedRAMP ATO, no per-pilot overlay.** The master instruments are the base; each pilot still needs the per-deployment addendum or SSP delta. Chapter 05 details the DPA shape.

**Treating the EU AI Act as "a model provider problem."** Article 25 is explicit: deployers of high-risk AI systems have obligations distinct from providers'. The customer is a deployer; the FDE's job is to produce artefacts the customer can use to meet deployer obligations (Article 26). Pointing at Anthropic / OpenAI / Google does not discharge the customer.
<!-- needs-research: confirm Article 25 (chain of providers, deployers becoming providers) and Article 26 (deployer obligations) wording before publishing -->

**Walking into a PCI scope.** A pilot that could easily be out of scope ends up in scope because no one reviewed what lands in the prompt. Fence card data at ingest; keep the pilot's scope artefactually zero where possible.

**LL144 / Colorado / state laws dropped on the floor.** Treating state regimes as "the customer's problem" when the engineering evidence is the only way the customer can meet them. Produce the output records, the impact-assessment inputs, the demographic-outcome breakdowns.

**Zero-retention promise with a model provider that quietly retains for abuse monitoring.** OpenAI and Anthropic each have abuse-monitoring retention defaults (30 days typically) that zero-retention enterprise agreements waive by addendum. The FDE must know the model provider's default *and* the addendum that modifies it, and name both in the DPA.
<!-- needs-research: confirm current OpenAI and Anthropic zero-retention / abuse-monitoring waiver terms and retention windows before publishing — these change with the enterprise agreement template -->

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
