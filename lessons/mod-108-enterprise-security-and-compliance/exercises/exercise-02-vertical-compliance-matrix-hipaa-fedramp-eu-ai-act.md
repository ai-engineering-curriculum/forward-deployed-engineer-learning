# exercise-02: Build the Per-Pilot Vertical Compliance Matrix (HIPAA / FedRAMP / EU AI Act and the State Patchwork)

**Estimated effort:** 3 hours

## Objective

For each of the three customer archetypes from Chapter 01, author the one-page vertical compliance matrix that routes every statutory obligation to customer / vendor / this role, with evidence pointers. The matrix is the artefact the customer's privacy office, the customer's compliance officer, and the vendor's legal team each read before a DPA / BAA / ATO-annex is signed. Confusing routing is the single most common reason the pilot's legal cycle lengthens from two weeks to three months.

You will produce three matrices — one per archetype — plus a short narrative identifying which obligations surprised you during the exercise. The matrices will be referenced by exercise-04 (data contract) and exercise-05 (supply-chain provenance).

## Prerequisites

- Read [Chapter 03](../03-vertical-compliance.md) in full. Many obligations are named only once; re-reading as you populate the matrix is expected.
- Skim the primary statutes you have not previously worked with — the gov-published text of HIPAA Security Rule (45 CFR 164.308–164.316), OMB M-24-10, the EU AI Act (Articles 5, 6, 9–15, 25, 26, 50, 72; Annex III), GDPR Articles 22, 32, 35. Not the full text — the headings and the engineering-salient sub-sections.
- From mod-107 Chapter 06, your archetype's per-pilot decision document shape (where the vertical compliance matrix attaches).

## Problem statement

For each archetype:

- **Archetype A — US healthcare payer.** HIPAA covered entity (member PHI in prompts); CCPA/CPRA if California members; NAIC AI Model Bulletin if adopted by the member's state DOI; SOC 2 Type II as commercial baseline. No federal, no PCI, no EU.
- **Archetype B — US federal agency.** FedRAMP Moderate baseline with documented path to High; NIST 800-53 Rev 5; NIST AI RMF 1.0; OMB M-24-10 (and successor memos); agency-specific ATO process; HIPAA may apply if health-related agency; Privacy Act of 1974 if PII in a system of records.
- **Archetype C — EU retail bank.** GDPR (Articles 22, 32, 35); DORA (ICT third-party register, exit plan, incident reporting); EU AI Act (Article 5 negative; Annex III §5(b) creditworthiness analysis; Articles 9–15 if high-risk; Article 14 oversight; Article 72 post-market monitoring); SR 11-7 equivalent discipline through model-risk function.

Build the three matrices as if you were the FDE handing them to each customer's privacy office at the pilot's first compliance review meeting.

## Requirements

For each archetype, produce one matrix row per regime that applies to the pilot. Columns:

1. **Regime.** Statute / regulation / framework name.
2. **Applies?** Yes / No / Partial, with a one-line "why." For No, explain the scoping logic that excluded it (do not simply omit it — the scoping decision is itself a compliance artefact).
3. **Scope touched.** Which data classes, user categories, or workflow steps trip the regime.
4. **Customer-owned obligations.** Statutory-role obligations that stay with the customer. Cite the specific section (e.g., "45 CFR 164.502 — permitted uses and disclosures").
5. **Vendor-owned obligations (platform level).** Platform-wide clauses.
6. **This-role-owned (per-pilot engineering).** The specific per-deployment engineering commitments.
7. **Evidence pointers.** Which artefacts (SOC 2 section, BAA, DPA clause, SSP control narrative, Annex IV documentation slice, DPIA, bias audit) carry each obligation's proof.
8. **Open gaps.** Any obligation you cannot confidently route today (customer's own posture unknown; vendor's platform evidence not yet produced; `<!-- needs-research: ... -->`).

Specific requirements:

- **Archetype A** must include at least: HIPAA Security Rule; HIPAA Breach Notification Rule; HITECH Omnibus Rule (sub-processor BAAs); SOC 2 Type II; CCPA/CPRA (service-provider status); NAIC AI Model Bulletin status (with a note on state-DOI adoption variability).
- **Archetype B** must include at least: FedRAMP Moderate baseline; NIST 800-53 Rev 5 (SR, PM, SI families named); NIST AI RMF 1.0; OMB M-24-10 (or successor) minimum practices for rights-impacting / safety-impacting AI; agency ATO process named; FIPS 140-2/140-3 cryptographic module requirement; Privacy Act of 1974 if PII in a system of records.
- **Archetype C** must include at least: GDPR Articles 22, 32, 35; Article 28 processor terms; Article 46 cross-border transfer mechanism analysis; EU AI Act Articles 5 (negative scoping), 6 (classification), 9–15 (if high-risk), 25, 26, 50, 72; DORA ICT third-party register entry; DORA exit-plan clause; SR 11-7-equivalent model-risk framing; ISO/IEC 27001 and ISO/IEC 42001 if applicable.

For each archetype also produce:

- **A scoping narrative.** 2–4 paragraphs walking the reader through why each in-scope regime applies and why each out-of-scope regime does not. The narrative is the auditor's walk-through, not a checklist.
- **A routing-anomalies list.** 2–5 items where the routing surprised you — obligations that looked like vendor-owned but are actually customer-owned (or vice versa), or obligations that split across all three columns. Short explanations.

## Starter guidance

- Walk Chapter 03's regimes in order. For each archetype, re-verify in-scope / out-of-scope against the archetype's workflow definition from mod-102 and mod-104 Chapter 01.
- The routing rule is: statutory-role obligations stay with the party who holds that role. The FDE never owns a statutory role — the vendor and the customer do. Per-pilot engineering commitments are what routes to this role.
- For every citation, prefer the primary source (eur-lex for EU, govinfo / CFR for US federal, HHS / OCR for HIPAA, CSRC for NIST, FedRAMP.gov for FedRAMP). Secondary summaries drift.
- For state regimes (LL144, Colorado AI Act, BIPA, state consumer privacy regimes), ask "does the archetype's workflow produce a consequential decision in the state's jurisdiction?" If yes, map it; if no, scope out explicitly.
- Do not invent BAA or DPA language — this exercise produces the routing matrix, not contract text. The DPA shape exercise is exercise-04.
- The EU AI Act application-date timeline is iterating. Mark any date-specific claim `<!-- needs-research: ... -->` and cite your source.
- For FedRAMP, do not invent an agency's ATO timeline; use "varies by agency" where specifics would be fabricated.

## Acceptance criteria

- Three vertical compliance matrices (one per archetype), each covering at least the regimes named in the Requirements section.
- Each row has all eight columns filled; "Open gaps" column is honest (empty is suspicious; most pilots have open gaps on a first-pass matrix).
- Three scoping narratives (one per archetype), each 2–4 paragraphs, each walking in-scope and out-of-scope regimes.
- Three routing-anomalies lists (one per archetype), each 2–5 items.
- Every citation points to a primary source, with a URL.
- Every unverifiable claim is marked `<!-- needs-research: ... -->` with enough context that a reviewer can resolve.
- At least one row per archetype demonstrates split ownership across customer / vendor / this role — the exercise is wrong if no row has split ownership.
- No invented quote from a statute. If you quote, the quote is the statute's wording or you do not quote.

## Stretch goals

- **Cross-archetype crosswalk.** A single table listing each regime down the left and the three archetypes across the top, with "applies / not applies / depends" cells. Useful when a customer in a fourth sector appears.
- **Regime trigger table.** For each regime, name the event that would newly bring it into scope (e.g., "pilot extends to California members → CCPA applies"). Captures the decision thresholds explicitly.
- **DPIA technical-input draft.** For Archetype C, draft the technical slice of the GDPR Article 35 DPIA — categories of data subjects, categories of personal data, recipients, retention periods, technical measures — one page.
- **Impact-assessment draft.** For Archetype B, draft the AI impact assessment slice required under OMB M-24-10 §5 (if rights-impacting or safety-impacting).
- **State-DOI adoption tracker.** For Archetype A, produce a one-page tracker of NAIC AI Model Bulletin adoption status across the states the customer operates in. Use `<!-- needs-research: ... -->` where the tracker is incomplete.
- **Change-of-posture memo.** A one-pager for each archetype answering: "if the customer expands the workflow from draft-assist to auto-send, which regimes change posture?"
