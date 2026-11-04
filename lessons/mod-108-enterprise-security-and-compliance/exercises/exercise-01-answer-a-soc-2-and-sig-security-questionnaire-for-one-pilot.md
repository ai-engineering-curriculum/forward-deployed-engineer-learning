# exercise-01: Answer a SOC 2 / SIG Security Questionnaire and Assemble the Per-Pilot Evidence Packet

**Estimated effort:** 3 hours

## Objective

Produce the per-pilot security-review packet for one of the three customer archetypes from Chapter 01: questionnaire response (SOC 2-mapped + SIG Lite + customer-house AI supplement), data-flow diagram, sub-processor list, pen-test report cover sheet, policy inventory, and the AI-specific cards referenced at a high level (model card + ML-BOM listed, full composition in exercise-05). The output is the artefact bundle an FDE would hand to a customer CISO office in week 2–3 of the pilot.

You will play the FDE answering the questionnaire for the chosen archetype. The deliverable is the shape the customer reads; the vendor's platform-level base artefacts (SOC 2 Type II report, master DPA) are assumed to exist and are referenced, not re-authored.

## Prerequisites

- Read [Chapter 01](../01-security-and-compliance-overlay.md) and [Chapter 02](../02-security-review-packet.md).
- Skim one SIG Lite sample and one CAIQ v4 sample (publicly available at [Shared Assessments](https://sharedassessments.org/) and [CSA CAIQ](https://cloudsecurityalliance.org/research/cloud-controls-matrix/) respectively).
- From mod-103, your archetype's identity and data integration posture; from mod-104, the gateway configuration; from mod-105, the retrieval topology; from mod-106, the agent orchestration and audit shape.
- Have the ADR template from mod-104 Chapter 02 to hand; this exercise's response memo mirrors its clarity.

## Problem statement

Pick one archetype from Chapter 01 and build the packet for it:

- **Archetype A — US healthcare payer.** SIG Lite + customer-house HIPAA supplement (~30 questions). HIPAA + SOC 2 Type II regime. BAA executed in template. AWS Bedrock + VPC install. Member-inquiry resolution workflow.
- **Archetype B — US federal agency.** SIG Lite variant adapted for agency use + agency-house AI supplement + FedRAMP-aligned narrative. FedRAMP Moderate baseline. GovCloud + self-hosted inference. Policy-analyst drafting workflow.
- **Archetype C — EU retail bank.** CAIQ v4 + CSA AICM supplement + bank-house GDPR / DORA addendum (~40 extra questions). GDPR + DORA + EU AI Act posture. Azure AI Foundry under EU Data Boundary. Customer-support reply + case triage workflow.

Produce the packet as if you were the FDE on the pilot in week 2.

## Requirements

1. **Questionnaire response memo.** For the chosen archetype:
   - Answer at least 25 SIG / CAIQ questions covering the following control families: access control, cryptography, logging and monitoring, incident response, change management, data classification, AI-specific (training opt-out, prompt storage, sub-processor disclosure, model-provenance). Cite the originating control ID (SIG section, CAIQ ID, SOC 2 Trust Services Criteria, NIST 800-53 Rev 5 control code) for each answer.
   - At least 5 answers must fill customer-context slots (e.g., name the deployed AWS region, the specific KMS key ARNs, the Datadog retention window), not just boilerplate.
   - For every answer, name the evidence artefact that backs it (SOC 2 report section, policy document, audit sample, DFD reference). Where the artefact does not yet exist in your mental model, flag `<!-- needs-research: ... -->` so a reviewer can backfill.

2. **Data-flow diagram.** Author a per-pilot DFD covering:
   - Every source of customer data (customer app, data warehouse, user input).
   - Every trust boundary (customer VPC, vendor cloud, model-provider API, sub-processor API).
   - Data classes at each hop (PHI / PII / PCI / confidential / non-sensitive) colour-coded or annotated.
   - Protection at each hop (TLS version, KMS key ARN, DLP step).
   - Audit log destination for each crossing.
   - Retention for each hop (in-flight and at-rest).
   - Minimum: a textual-form DFD with every element above. Preferred: a drawio / PlantUML / Mermaid visual export in addition.

3. **Sub-processor list.** A complete per-pilot sub-processor table with columns: name, purpose, data classes touched, region, contractual coverage (BAA / SCCs / DPA), retention, link to the sub-processor's own trust centre. Include at least 6 sub-processors (model provider, embedding provider if separate, retrieval engine, observability stack, audit stream, identity provider).

4. **Policy inventory table.** The inventory format from Chapter 02 (12 named policies); mark each policy as "exists" / "needs authoring" / "customer-owned" per your chosen archetype's posture.

5. **Pen-test report cover sheet.** A one-page cover memo ready to share with the customer that references (not includes) the full pen-test report: 3PAO name, report date, audit period, scope statement, summary of findings, remediation-status one-pager, NDA clause the customer must sign to receive the full body.

6. **Packet index.** A single `packet-index.md` listing every artefact, its current version, its owner, its date, its status.

7. **Response timeline.** A one-paragraph narrative of how you would sequence the submission — which artefact lands first, which blocks on customer legal, which follows a redline cycle — with a target time-to-sign-off.

## Starter guidance

- Use the response-library discipline from Chapter 02. Your answers should feel like library entries contextualised for the archetype, not freehand prose.
- Any answer you cannot verify from a primary source (vendor policy, standards document, model-provider trust page) is marked `<!-- needs-research: ... -->`. Guessing is worse than flagging.
- For the DFD, start textually; if you have time, produce a visual. A visual without the textual content behind it will miss edges a reviewer will ask about.
- For the sub-processor list, every HIPAA pilot row that touches PHI must have a BAA noted; every GDPR row must have an EU residency or a transfer mechanism; every FedRAMP row must have the sub-processor's FedRAMP authorisation.
- Do not invent a specific pen-test 3PAO or specific CVE findings. Describe the shape of the cover sheet with placeholder values.
- The policy inventory is a *table*, not twelve policy documents. The exercise deliverable is the inventory, not the policies themselves.
- Keep the AI-specific cards (model card, ML-BOM, ModelScan report) as named entries in the packet index — the full composition is exercise-05.
- Treat `<!-- needs-research: ... -->` liberally rather than guess. The exercise's realism is improved by honest gaps.

## Acceptance criteria

- One completed questionnaire response memo (minimum 25 answers, minimum 5 with filled customer-context slots; all with citations and evidence pointers).
- One per-pilot DFD (textual minimum; visual bonus) showing every trust-boundary crossing with data class, protection, log, retention.
- One sub-processor list with at least 6 entries, all seven columns populated, with contractual coverage correct for the chosen archetype.
- One policy inventory table (12 named policies, each marked).
- One pen-test report cover sheet (one page).
- One `packet-index.md` listing every artefact with version / owner / date / status.
- One response-timeline paragraph with a target time-to-sign-off.
- At least 5 `<!-- needs-research: ... -->` markers, distributed across artefacts, each with enough context that a reviewer could resolve.
- The archetype-choice is explicit at the top of the submission; the packet matches that archetype's compliance posture throughout (HIPAA / FedRAMP / GDPR as applicable).

## Stretch goals

- **Trust-centre mockup.** Author the one-screen summary a customer would see on a trust-centre page: certifications, current reports, policies, sub-processor list — the shape SafeBase / Whistic / a self-hosted equivalent would publish. 
- **Legal-redline anticipation.** Predict three clauses from a standard DPA where the customer's legal team is likely to redline (sub-processor notice period, breach SLA, audit rights) and pre-draft the engineering-backed counter-language.
- **Response-library seed.** Extract 10 of your questionnaire answers into a `response-library/` folder in the format shown in Chapter 02, so a future FDE can reuse them.
- **Second-archetype crosswalk.** For the chosen archetype's packet, identify the five artefacts that would be *reusable* for the other two archetypes and the five that would need per-archetype re-authoring. One paragraph.
- **AICM-aware supplement.** Pre-answer 10 questions from the CSA AI Controls Matrix (AICM) even if the customer's questionnaire does not explicitly reference it, and attach them as an optional annex.
