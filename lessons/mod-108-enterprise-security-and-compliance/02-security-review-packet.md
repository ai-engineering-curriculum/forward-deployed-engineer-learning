# Chapter 02 — Passing the Security Review — SOC 2, SIG, CAIQ, DFD, Sub-Processors, Pen-Test

## Why this chapter exists

Every regulated-enterprise customer runs a vendor security review before the pilot can go live. The review has a predictable shape: a questionnaire (SOC 2 mapped, SIG Lite / SIG Core, CAIQ, or a customer-house variant), a request for supporting artefacts (SOC 2 Type II report, pen-test summary, data-flow diagram, sub-processor list, security policies), and a cadence of follow-up questions until the customer's CISO office is satisfied. A customer's pipeline will not approve a purchase order until the review is signed off.

The craft of *answering* that review is tactical, repeatable, and overlay — it rests on the vendor's platform-level security posture (`ai-infra-security` owns the depth) and the customer's regulatory regime (Chapter 03 names the vertical specifics). This chapter sets the engineering-side discipline for producing the packet at the pace the pilot demands.

## The two reference questionnaires: SIG and CAIQ

Most customer questionnaires are variants of two:

- **SIG (Standardized Information Gathering) from Shared Assessments.** Comes in SIG Lite (~130 questions) and SIG Core (~900+ questions) plus custom SIG. Used widely in financial services, insurance, and healthcare. The Shared Assessments program publishes the question library annually. See [Shared Assessments](https://sharedassessments.org/).
<!-- needs-research: confirm current SIG Lite / SIG Core question counts and the latest annual release before publishing — the question library updates each year -->
- **CAIQ (Consensus Assessments Initiative Questionnaire) from the Cloud Security Alliance.** Latest release is CAIQ v4; maps 1:1 to the Cloud Controls Matrix (CCM). Used widely by cloud-first customers, especially in Europe. See [CSA CAIQ](https://cloudsecurityalliance.org/research/cloud-controls-matrix/).

Customer-house questionnaires almost always borrow from these two. A new one is rarely truly new; it is SIG or CAIQ with the house's phrasing plus AI-specific supplements. The vendor's response library is built against the two canonical shapes and re-used across customers.

**AI-specific questionnaire supplements.** As of 2026 the AI slice of vendor questionnaires has been consolidating. Recognise and respond to:

- **CSA AI Controls Matrix (AICM)** — the CSA's AI-extension of CCM, with AI-specific controls on model lifecycle, training data, prompt handling, output filtering. Customers increasingly ship an AICM-aligned supplement with their CAIQ.
<!-- needs-research: confirm CSA AI Controls Matrix latest version and any renaming (STAR for AI / AICM) before publishing — the AICM is new as of 2024-2025 and still evolving -->
- **NIST AI RMF–aligned questions.** Customers under NIST 800-53 Rev 5 (control SR, PM, SI families) or NIST AI RMF 1.0 often append 20-50 AI-focused questions.
- **Customer-house AI supplements.** Each enterprise is iterating its own — common themes are model-provider list, retention clauses, training-opt-out posture, prompt-injection mitigations.

Chapter 03 is where the vertical regimes (HIPAA, FedRAMP, EU AI Act, etc.) attach. This chapter deals with the general-security questionnaire discipline.

## The response-library discipline

The FDE does not answer a customer questionnaire from scratch. The vendor organisation maintains a response library — one canonical, evidence-backed answer per question, versioned, dated, and owned. The FDE's per-customer job is:

1. **Receive** the customer's questionnaire.
2. **Map** each question to the response-library entry (fuzzy-match by question text; the hard-mapped Shared Assessments and CSA IDs save weeks).
3. **Contextualise** the response for this customer — fill in the deployment specifics, the data-class specifics, the sub-processor specifics.
4. **Flag** gaps for the platform-security team (`ai-infra-security`) — any answer that is not in the library, or any answer the library gives at platform depth that the customer wants deployment-specific.
5. **Assemble** the supporting evidence: SOC 2 Type II report, pen-test summary, DFD, sub-processor list, policies.
6. **Submit** through the customer's channel (SecurityScorecard, HICX, OneTrust, Whistic, or email).
7. **Shepherd** the back-and-forth — the customer's analyst will ask follow-ups; a one-day turnaround is the norm for mid-market, two to three days for enterprise.

A response library entry carries, at minimum:

```yaml
# response-library/control-cc6.1-logical-access.yaml
id: cc6.1-logical-access
source_frameworks: [SOC 2 CC6.1, SIG I.4, CAIQ IAM-01, ISO 27001 A.9]
canonical_answer: |
  Access to production systems is governed by SSO (Okta) with hardware MFA for
  privileged roles. Access is granted via role-based access control (RBAC) with
  just-in-time elevation through a reviewed PR / change-ticket. Access is
  reviewed quarterly by control-owner teams.
evidence:
  - type: policy
    path: evidence/access-control-policy-v3.pdf
    last_reviewed: 2026-09-15
  - type: soc2_control
    report_section: CC6.1
    period_end: 2026-06-30
customer_context_slots:
  - name: deployment_target
    note: "Name the deployed environment (AWS region, GovCloud, Azure EU)."
  - name: customer_specific_roles
    note: "If the customer-dedicated deployment adds roles, name them."
owner: security-platform-team
last_updated: 2026-10-01
```

The FDE treats the library as production. A new question the library does not cover is a library gap — the FDE opens a ticket to `security-platform-team`, drafts a candidate answer for that pilot, and feeds the canonicalised answer back to the library on sign-off.

## The per-pilot evidence packet

The questionnaire answers are one half of the review. The other half is the artefact packet. Six artefacts land in almost every review.

### 1. SOC 2 Type II report

Owned by the vendor's security team; produced annually (or on a shorter cycle). The FDE's job is to know the report's current validity window and to share it under NDA. Customers running procurement in [OneTrust](https://www.onetrust.com/), [Whistic](https://www.whistic.com/), or [SecurityScorecard](https://securityscorecard.com/) may be able to access it directly from a vendor trust centre; others want the PDF emailed.

Red flags the FDE must know how to triage:

- **Expired report.** If the report's audit period ended more than a year ago, the customer will reject it. Escalate to the security team for the current cycle's status.
- **Qualified or adverse opinion.** Rare, but the FDE must know whether any exceptions were noted; the customer will ask.
- **Scope mismatch.** The report's scope must include the system the pilot runs on. If the pilot uses a new region, service, or sub-system the report does not cover, name the gap and commit to a scope-extension letter from the auditor.

### 2. Penetration-test report

Produced by a third party on at least an annual cadence. Customers rarely want the full report; they want the executive summary + remediation status. Share:

- The executive summary PDF under NDA.
- A remediation-status one-pager listing any high / critical finding + its remediation date (or compensating control).
- The 3PAO name and date range.

Never share the full technical body without customer-specific NDA and security-team approval; vulnerability detail is sensitive even post-remediation.

### 3. Data-flow diagram (DFD)

The DFD is the artefact the FDE authors most directly. It is the per-pilot picture of every trust-boundary crossing the pilot makes. A security reviewer's first question is almost always "walk me through the data flow." A packet without a DFD signals immaturity.

The DFD must show:

- **Source.** Where does data originate (customer application, customer data warehouse, user input)?
- **Trust boundaries.** Customer VPC, vendor cloud, model-provider API, sub-processor API, each labelled.
- **Data classes at each hop.** PHI, PII, PCI, confidential business data, non-sensitive — colour-coded or annotated.
- **Protection at each hop.** TLS cipher suite, KMS key, DLP step, field-level encryption, tokenisation.
- **Logging.** Which audit log captures this crossing (customer SIEM, vendor audit log, cloud CloudTrail / Cloud Audit Logs, model-provider log).
- **Retention.** How long each hop retains data in-flight and at-rest.

A minimal DFD, textually:

```
[ Customer app ] --(TLS 1.3, customer OIDC token)--> [ Vendor API gateway (VPC) ]
    |                                                     |
    | PII policy: no PHI in telemetry                     | audit: customer Splunk HEC
    v                                                     v
[ Vendor app (VPC) ] --(no public egress)--> [ Bedrock private endpoint (same-VPC) ]
    |                                             |
    | retention: 0 for prompt+completion          | zero-retention attested
    v                                             v
[ pgvector in customer VPC ]                 [ returns to vendor app, then customer app ]
```

Diagrams (drawio, Lucidchart, Excalidraw, PlantUML) are preferred over prose but the structure above is a working minimum when a visual tool is not available. Chapter 05 drills into the *contract* that each hop implements.

### 4. Sub-processor list

A list of every third party that processes customer data during the pilot. Columns the FDE must populate:

| Sub-processor | Purpose | Data classes | Region | Contract | Retention |
|---|---|---|---|---|---|
| AWS (Bedrock, S3, KMS) | Model inference, storage | PHI, PII | us-east-1, us-west-2 | AWS BAA executed 2026-04-11 | Zero-retention (model inputs); 7 years (S3 archive) |
| OpenAI (if used as embedding) | Embedding | Non-PHI chunks only | N/A | OpenAI Enterprise DPA executed 2026-05-02; zero-retention; no training | Zero |
| Datadog | Observability | Non-PHI telemetry | US1 | Datadog DPA; PII-stripped telemetry | 30 days |

For HIPAA customers, every sub-processor that touches PHI must have a BAA — that is the single most-asked question. For GDPR customers, the sub-processor list is required under Article 28 and is subject to notice-and-objection rules. For FedRAMP customers, every sub-processor must be FedRAMP-authorised at or above the customer's baseline.

The list changes. Any addition, removal, or re-scoping triggers a customer notification; most DPAs give the customer a 30-day objection window. Chapter 05 details the DPA clause that commits to this.

### 5. Information-security policies

A summary set of policies. Customers rarely read the full text; they want the inventory + the "last reviewed" dates. Standard inventory:

- Information security policy
- Access control policy
- Data classification and handling policy
- Incident response plan
- Business continuity / disaster recovery plan
- Change management policy
- Secure development lifecycle policy (with AI / ML addendum)
- Vendor risk management policy
- Vulnerability management / patching policy
- Cryptographic policy (with key management)
- Logging and monitoring policy
- AI use and model-risk policy (new; increasingly asked)

The FDE's job is to know which policies exist and to produce the inventory table. Writing policies is not the FDE's job; the vendor's platform-security / GRC team owns them.

### 6. Model card + system card + ML-BOM slice

The AI-specific artefacts the customer increasingly asks for during security review (not only governance review). Chapter 06 details composition. For packet assembly, include:

- The per-pilot model card (what model, what version, what fine-tuning if any, what inputs, what outputs, known limitations).
- The per-pilot system card (end-to-end system behaviour — not only the model).
- The ML-BOM (CycloneDX) for the deployed artefacts.
- ModelScan / picklescan report for any third-party weights consumed.
- SLSA / Sigstore attestation for the release.

Customer CISOs increasingly want these in the security packet even for non-regulated pilots, because the audit pattern is spreading.

## The response cadence: compressing time-to-sign-off

Time-to-sign-off is one of the module's six budget dimensions. Factors that compress it:

- **A current response library.** If 85% of the questions are canonical and the FDE only writes the deployment-specific 15%, the first-pass response takes 2-3 days instead of 2 weeks.
- **A pre-staged evidence folder.** The SOC 2 Type II, pen-test summary, policy inventory, standard DFD template, sub-processor list in current form, model card template, ML-BOM template — in one folder, dated.
- **Named reviewers on both sides.** One FDE drives the response; one customer analyst drives the review. Multi-party threads produce weeks of thrash.
- **A trust centre.** [SafeBase](https://safebase.io/), Whistic, or a self-hosted equivalent. Customers can pull the base artefacts without vendor involvement; the review narrows to the pilot-specific slice.
- **Up-front flagging of likely redlines.** If the FDE knows the DPA clause on sub-processor notice will come back, send the vendor's standard language with the first response rather than waiting for the customer to ask.

Factors that lengthen it:

- Stale SOC 2 or pen-test report.
- A sub-processor added since the questionnaire template was last written.
- An AI-specific supplement the response library has not yet covered.
- A fresh vulnerability disclosure (log4j shape) that forces a customer-specific response.
- A customer legal team that treats every security question as a contract negotiation.

The FDE's discipline is to keep the first four in order and to triage the fifth to the customer's security liaison plus the vendor's legal team.

## Running the three archetypes

**Archetype A — US healthcare payer.** Review shape: SIG Lite + customer-house HIPAA addendum (~30 extra questions). Evidence packet: SOC 2 Type II, pen-test summary, signed BAA, HIPAA-addendum DPA, DFD showing PHI stays in-VPC, sub-processor list with BAAs noted, access-control + incident-response policies, member-inquiry workflow model card, ML-BOM for the gateway release. Common follow-ups: BAA scope of sub-processors, breach-notification SLA (45 CFR 164.410 is 60 days from discovery), audit-log retention (6 years is typical for HIPAA covered entities to retain policies and procedures). Timing: aim for sign-off in 2–3 weeks.
<!-- needs-research: confirm HIPAA breach-notification SLA wording from 45 CFR 164.410 (60-day outer limit) and the 6-year retention requirement from 45 CFR 164.316 before publishing -->

**Archetype B — US federal agency.** Review shape: FedRAMP SSP-aligned control narrative + agency ATO-package addendum. Evidence packet: FedRAMP Moderate PATO / ATO letter, System Security Plan (SSP), Plan of Action and Milestones (POA&M), 3PAO Security Assessment Report (SAR), FIPS 140-2 / 140-3 validation certs for cryptographic modules, DFD showing GovCloud boundary, sub-processor list (all FedRAMP-authorised), ML-BOM with SLSA Level 2+ provenance, policy-analyst-workflow model card + system card, agency-specific AI use statement under OMB M-24-10 / M-25-21 guidance. Common follow-ups: FIPS validation on every cryptographic surface, SBOM / ML-BOM in the body of evidence, incident-response alignment with the agency's SOC. Timing: 2–4 months for new ATO; 2–4 weeks for a pilot inside an existing ATO boundary.
<!-- needs-research: confirm current OMB AI-use memo numbers (M-24-10 and successor memos) before publishing; the OMB memo series continues to iterate -->

**Archetype C — EU retail bank.** Review shape: CAIQ v4 + CSA AICM supplement + bank-house GDPR / DORA addendum. Evidence packet: SOC 2 Type II, ISO/IEC 27001 cert, pen-test summary, GDPR Article 28 DPA, DPIA under Article 35, sub-processor list with EU residency confirmed, DFD showing all data inside EU Data Boundary, CMK posture on Azure, Azure OpenAI zero-retention attestation, DORA ICT-third-party register entry draft, customer-support-workflow model card + EU AI Act Annex IV technical-documentation slice, ML-BOM. Common follow-ups: GDPR cross-border transfer analysis (Chapter 03), DORA exit-plan clause (bank must be able to migrate off the vendor), sub-processor notice period, right-to-audit clause. Timing: 4–8 weeks in regulated financials.
<!-- needs-research: confirm DORA (Regulation (EU) 2022/2554) ICT-third-party register and exit-plan requirements before publishing — DORA came into application 2025-01-17 -->

## Anti-patterns

**Answering from memory.** The FDE writes a questionnaire response from what they remember of the platform. Six months later, the platform has changed and the customer catches the drift in an audit. Answer from the response library, not from memory.

**One response library, no customer context slots.** Every customer gets the same boilerplate. Half the security reviewer's follow-ups are "but what about *our* deployment?" The library has slots for per-deployment detail; the FDE fills them.

**Sharing the full pen-test report.** The vulnerability detail is sensitive even after remediation. Share the executive summary and the remediation-status one-pager; the full body only under customer-specific NDA with security-team approval.

**Omitting a sub-processor because it is "indirect."** The customer's privacy office does not distinguish direct from indirect sub-processors. If data touches it, it is on the list.

**Using a public SaaS form for the questionnaire upload.** Some customers' forms are OneTrust public URLs; the response content is sensitive. The FDE confirms the destination is the customer's authenticated portal before uploading.

**DFD that shows the ideal, not the as-built.** A DFD that omits a diagnostic path a developer added last week will not survive the security review when the reviewer asks to see CloudTrail. The DFD matches the deployed system, not the aspirational one.

**Treating AI-specific questions as "not applicable."** Writing "N/A" to a question like "do you retain prompts for training?" because "we do not train on them" is a dodge. The right answer names the model provider, the provider's training-opt-out posture, and the contractual commitment (Chapter 05). "N/A" without that context will come back.

## Summary

- Security review is overlay craft: the vendor's platform-level posture is the base layer; the FDE assembles the per-pilot evidence on top.
- SIG (Lite / Core) and CAIQ (v4) are the two canonical questionnaires; CSA AICM is the emerging AI supplement. Customer-house questionnaires are variants of these.
- The vendor keeps a dated, versioned response library; the FDE's per-customer job is to map, contextualise, flag gaps, and shepherd the back-and-forth.
- The per-pilot evidence packet has six anchors: SOC 2 Type II, pen-test summary, DFD, sub-processor list, policies, and the AI-specific cards (model, system, ML-BOM).
- The DFD is the artefact the FDE authors most directly; it shows every trust-boundary crossing with data class, protection, log, and retention.
- Response-library discipline, pre-staged evidence, named reviewers, a trust centre, and up-front redline flagging compress time-to-sign-off.
- Each customer archetype produces a predictably different packet shape, timing, and set of follow-ups.
- Chapter 03 picks up vertical compliance — HIPAA, FedRAMP, PCI-DSS, EU AI Act, GDPR, CCPA, and state-level regimes — and names which obligations route where.
