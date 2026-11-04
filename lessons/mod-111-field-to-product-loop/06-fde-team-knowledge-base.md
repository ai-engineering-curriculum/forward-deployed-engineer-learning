# Chapter 06 — The FDE Team Knowledge Base — The Compounding Artefact That Scales the Team

## Why this chapter exists

Chapters 02 through 05 operated per pilot — one FDE, one capture log, one quarter's writeups, one meeting. The knowledge base operates *across* pilots. It is the artefact that answers the organisational question *"how does the fourth FDE onboarded to a similar pilot shape find the pattern without re-asking the three who came before?"*

Without the knowledge base, each FDE rediscovers the same shapes. The retrieval-permission anti-patterns from mod-105 get relearned at every retail pilot; the FedRAMP-specific response-library adaptations get re-authored at every federal pilot; the Langfuse self-hosted-in-customer-cloud playbook gets rewritten at every healthcare pilot. The waste is invisible to any individual FDE and enormous across the team.

With the knowledge base — and with the review discipline that keeps it from becoming an unread graveyard — the second pilot's FDE opens the pattern catalogue, finds "healthcare payer, Langfuse self-hosted, HIPAA six-year retention" with a complete deployment checklist and a Terraform snippet that has been field-tested at two prior pilots, and ships the pilot faster than the first FDE managed. That is the compounding this chapter is responsible for producing.

This chapter has four jobs. Specify the **four knowledge-base slices** — pattern catalogue, failure-mode library, deployment playbook, security-review-answer library. Specify the **entry schema** and the ownership model. Specify the **review cadence** that keeps entries fresh. Catalogue the anti-patterns of a knowledge base that grows but is not read.

Knowledge-base compounding is an optimisation dimension from Chapter 01.

## What the knowledge base is and is not

The knowledge base is a **vendor-internal, FDE-team-owned, versioned, searchable** store of structured entries that describes recurring patterns, failure modes, deployment procedures, and security-review answers. Typical hosts: a dedicated repo in the vendor's GitHub organisation (`fde-knowledge-base`), a dedicated Confluence space, or a wiki with structured templates. The tool matters less than the discipline.

The knowledge base **is**:

- The onboarding surface for a new FDE. "Read these fifteen entries in this order" is the first-week assignment.
- The reference surface for a mid-career FDE starting a new pilot. "Before you write the ADR, read the five catalogue entries that touch your archetype."
- The feed-forward surface for the FDE team's shared vocabulary. The `surface` field in Chapter 02's capture schema derives from the knowledge base's registered vocabulary.
- An input to Chapter 03's design-partner artefacts — a pattern that has three catalogue entries' worth of history is a pattern ripe for a quarterly 'what we learned' one-pager.

The knowledge base **is not**:

- A customer-facing artefact. Entries may name specific pilots as evidence; those entries never leave the vendor's internal store without going through Chapter 03's sign-off discipline.
- A substitute for the vendor's product documentation. Product docs describe the platform's current behaviour; the knowledge base describes the field's accumulated experience *with* that platform.
- The capture log. The capture log is a stream (per-pilot, dated, heterogeneous); the knowledge base is a curated store (cross-pilot, versioned, structured).

## The four slices

Four knowledge-base slices, each with a distinct audience, template, and ownership.

### Slice 1 — The pattern catalogue

What it is: the FDE team's library of recurring patterns — architectural shapes, decision patterns, trade-off writeups — that repeat across pilots regardless of customer.

Entries in the pattern catalogue are *writeups*, not procedures. A pattern catalogue entry answers *"what is the pattern, when does it apply, what are the trade-offs, what have we learned?"* Examples:

- "VPC-install Langfuse for HIPAA customers" — when to pick VPC-install over SaaS; the compliance decisions that force the pick; the archetypal trade-offs.
- "Judge-rubric calibration against SME preferences" — how the rubric is iterated when SME-feedback diverges from the rubric's score.
- "Prompt-only canary with 24-hour hold" — when the hold is worth the cost; the signals that justify the pattern.

Pattern-catalogue entries typically originate from a Chapter 03 pattern writeup. Once a pattern has appeared in at least two writeups across two different FDEs' pilots, it is promoted to the catalogue with a consolidating entry. The Chapter 03 writeups remain in their per-quarter folders; the catalogue entry is the canonical reference.

### Slice 2 — The failure-mode library

What it is: a dictionary of the failure modes an FDE might encounter in production. Each entry is *symptom → diagnosis → response → root-cause check*.

The failure-mode library is the direct generalisation of mod-110 Chapter 05's six canonical failure modes (prompt regression, provider outage, retrieval drift, hallucination spike, cost blowout, guardrail false-positive storm). Each generalised entry carries:

- The symptom shape — what a dashboard, an alert, or a user report looks like when this failure mode is present.
- The diagnosis checklist — three to seven questions the FDE walks through to confirm the failure mode.
- The response sequence — the playbook steps; links to the specific per-pilot playbooks that instantiate the pattern.
- The root-cause check — once mitigated, how the FDE confirms the root cause before closing the incident.
- Known variations — how the same failure mode expresses differently at different archetypes.

The library's audience is the on-call FDE who has paged in at 3am and needs to triage a shape they have not seen before. The entries are terse by design — a library entry is three pages maximum, structured as a decision tree, scannable in under five minutes. The FDE reading at 3am wants a decision path, not a research paper.

### Slice 3 — The deployment playbook

What it is: the step-by-step procedures for deploying a pilot — per archetype, per deployment mode (SaaS / VPC / on-prem / air-gapped), per integration surface (SSO / SCIM / NHI / observability).

The playbook is the most procedural slice. A deployment-playbook entry answers *"we are about to deploy a $archetype pilot in $deployment-mode; what are the steps, in what order, with what checkpoints?"* Each entry carries:

- **Preconditions.** The customer-side artefacts that must exist before the step starts (signed BAA, agreed data contract, provisioned KMS keys, confirmed VPC peering, etc.).
- **Steps.** Ordered, named, each with a concrete action and a verification check.
- **Checkpoints.** Places where the FDE pauses and confirms with the customer side.
- **Rollback.** If a step fails, how to back out.
- **Known gotchas.** Specific failures seen in prior pilots at the same step; what caused them.

The playbook is the surface the new-FDE onboarding leans on heaviest. A new FDE walking a HIPAA-archetype VPC-install Langfuse deployment reads the entry's checklist and executes; the third FDE to execute the same playbook produces the same pilot shape as the first FDE did, without the variance of individual discretion.

### Slice 4 — The security-review-answer library

What it is: a response library for the AI-specific security questions that customer CISOs ask. The library extends the SIG / CAIQ / SOC 2 response library from [mod-108 Chapter 02](../mod-108-enterprise-security-and-compliance/02-security-review-packet.md) with the questions those frameworks do not cover natively — prompt-injection posture, training-data provenance, model-provenance, judge-evaluation posture, retrieval-ACL guarantees, guardrail bypass risk.

Each library entry carries:

- The question in a canonical shape (the CISO's phrasing varies; the canonical shape is what the library indexes on).
- The vendor's platform-level answer. Links to the authoritative product doc / policy / SOC 2 report.
- The per-archetype customisation guidance. "For HIPAA pilots, additionally cite 45 CFR 164.312(a)(2)(iv) and the specific BAA clause." "For FedRAMP Moderate pilots, additionally cite AC-4 and the agency-specific ATO boundary." "For GDPR / DORA pilots, additionally cite Article 32 and the specific transfer-mechanism."
- The known variations in the customer's phrasing. "Some customers ask about 'prompt-injection via tool-call results' rather than 'indirect prompt injection'; these are the same concern."
- The evidence artefact names. "Attach the ModelScan report, the ML-BOM, the model-provider's trust centre page."

The library is the artefact that most directly reduces the FDE's time-to-answer on a per-pilot security-review cycle. An FDE with a mature library answers a typical SIG Lite in a fifth of the time an FDE without one spends.

## The entry schema

One schema across all four slices. Each slice uses the common fields and adds slice-specific fields.

```yaml
# fde-knowledge-base/catalogue/vpc-install-langfuse-for-hipaa.md
entry_id: pat-vpc-langfuse-hipaa-001
slice: pattern-catalogue        # pattern-catalogue | failure-mode-library | deployment-playbook | security-review-answer-library
title: "VPC-install Langfuse for HIPAA customers"
summary: |                      # one-paragraph summary surfaced in search
  Customer-side VPC-installed Langfuse with CMK on Postgres and S3, SSO via the
  customer's IdP, six-year retention on all traces and completions for HIPAA
  Security Rule documentation. When to pick this over SaaS Langfuse; what the
  trade-offs are; what a second-pilot FDE should expect.

owners:                         # named FDEs; at least one is accountable
  accountable: <fde name>       # the single person who approves changes
  contributors:                 # additional FDEs who have shipped a pilot using this pattern
    - <fde name>
    - <fde name>

status: active                  # draft | active | deprecated
version: 2.3.0                  # semver on content; major = shape change, minor = new archetype, patch = correction
last_reviewed: 2026-10-15
next_review: 2027-01-15         # 90 days after last_reviewed

surfaces:                       # from Chapter 02's shared vocabulary
  - telemetry-platform
  - deployment-mode
  - compliance-posture

archetypes:                     # which of A / B / C this entry covers
  - archetype_a_us_healthcare_payer

evidence:                       # pilot references that back this entry
  - pilot_id: pilot-acme-health
    role: initial_shape
    date: 2026-03
  - pilot_id: pilot-beta-health
    role: second_instance
    date: 2026-07

related_entries:
  - pat-cmk-postgres-aws-002
  - sec-prompt-retention-hipaa-005

body: |
  [The entry's full content — the pattern writeup, the failure-mode decision
  tree, the deployment checklist, or the security-review answer. Format
  conventions vary per slice; the entry's slice determines the shape.]
```

Nine common fields; the body varies per slice. The schema is intentionally shallow — a new entry should be authorable in under two hours (pattern-catalogue entry) or under one hour (library entry). A schema that takes a day to fill is a schema that doesn't get filled.

### Versioning discipline

Entries are versioned semver-style. Three change classes:

- **Major (X.0.0).** The entry's shape has changed. The failure-mode decision tree has a new branch; the pattern's trade-off has flipped; the playbook now assumes a different precondition. A major version bump requires the accountable owner's review plus one peer FDE sign-off.
- **Minor (x.Y.0).** A new archetype has been added, or a new customer-specific variation. The entry's existing content still holds; new content is appended. Minor bumps require the accountable owner's review.
- **Patch (x.y.Z).** A correction — a URL fixed, a code snippet updated, a specific number revised. Patch bumps are self-service by any FDE; the accountable owner is CC'd.

The version is visible in search results; FDEs picking up an entry can see at a glance whether it is mature (3.4.2) or fresh (0.2.0).

### Ownership model

Every entry has one accountable owner. The accountable owner is responsible for:

- Keeping `last_reviewed` within 90 days of today.
- Confirming the entry's content still reflects the field.
- Dispatching corrections to contributors when the entry drifts.
- Deprecating the entry when its pattern no longer applies.

Accountability rotates with pilot assignments. A first FDE authors the pattern-catalogue entry when they ship the first pilot of that shape; accountability transfers to a second FDE when the first FDE moves to a different pilot family. The transfer is explicit — a PR into the entry naming the new accountable owner, signed by the outgoing owner and `ai-engineering-manager`.

An entry without an accountable owner is deprecated within 30 days. The deprecation PR either reassigns or archives; archives are preserved in the repo history but removed from active search results.

## The review cadence

A quarterly knowledge-base review, run by the FDE team collectively, is the discipline that keeps the knowledge base from becoming an unread graveyard.

### The quarterly review meeting

A 90-minute meeting at the start of each quarter, attended by:

- Every FDE on the team.
- `ai-engineering-manager`.
- `ai-enablement-lead` (peer-level, same family — some knowledge-base entries converge with customer-education content the lead owns).

The agenda:

1. **Staleness sweep — 20 minutes.** Entries whose `last_reviewed` is more than 90 days ago are listed. Accountable owners read each one aloud; status is confirmed, re-reviewed, or deprecated. Entries without owners are reassigned or archived.
2. **New-entry review — 30 minutes.** Entries created the previous quarter are reviewed for shape, naming, and placement. The FDE team catches the entries that would duplicate existing content, that should be merged with a related entry, or that should be split into two.
3. **Deprecation sweep — 15 minutes.** Entries flagged `deprecated` since the last review are confirmed for archive; related entries are updated to remove dead links.
4. **New-FDE onboarding read-list refresh — 15 minutes.** The onboarding read-list (15–20 entries, versioned) is revised based on the quarter's new content. A new FDE joining the team next quarter reads the revised list.
5. **Vocabulary alignment — 10 minutes.** The `surfaces` vocabulary from Chapter 02 is reviewed; new terms are added or consolidated; the shared vocabulary file is updated.

### The staleness index

The FDE team tracks a staleness index — the fraction of active entries whose `last_reviewed` is within 90 days. A healthy index stays above 90 %. An index that drifts below 80 % is a signal the team is not reviewing; `ai-engineering-manager` raises it at the FDE team's next planning meeting.

### The hit-rate measurement

A different signal: the fraction of new-FDE onboarding questions that are answerable from the knowledge base. Measured by shadowing a new FDE for a week — every time the FDE asks a question, record whether the answer was in the knowledge base. A healthy hit-rate is above 70 %; a hit-rate below 50 % is a signal the knowledge base is structurally thin, not just stale.

## Boundaries with peer roles

Two peer roles overlap with the knowledge base in productive ways.

### `ai-enablement-lead` — customer-education content

`ai-enablement-lead` (peer, level 35, FDE family) owns the public-facing patterns — customer training materials, SME certification curriculum, publicly-referenceable deployment patterns. A pattern-catalogue entry that has matured enough — reviewed, versioned, applied at three or more pilots — is a candidate for the `ai-enablement-lead` to adapt into a customer-education artefact.

The handoff is explicit. The FDE team's pattern-catalogue entry is the internal truth; `ai-enablement-lead` reads it and produces the public version, with per-pilot specifics redacted and the customer-facing framing added. The FDE team does not maintain the public version; `ai-enablement-lead` does. The two artefacts are linked back and forth in the schema (`related_entries`).

### `ai-platform-engineer` — shared platform documentation

`ai-platform-engineer` owns the vendor's cross-fleet GenAI platform documentation. The deployment-playbook slice overlaps with their documentation; a playbook entry that assumes a particular platform feature is a de-facto contract between the FDE team and the platform team.

The handoff is: the FDE team maintains the playbook entries that assume the platform's behaviour; the platform team notifies the FDE team when the behaviour changes. A platform change that invalidates a playbook entry becomes an open task for the accountable owner to revise the entry; `ai-platform-engineer` writes a release note referencing the playbook entry that needs update.

## The commitment register

A fifth slice — not quite a knowledge-base entry type, but lives in the same repo.

The commitment register from Chapter 05 — the table of every PM verbal commitment from quarterly product-review meetings — is a knowledge-base artefact. Each row is reviewed at the quarterly knowledge-base review as a standing check:

- Are commitments ageing? A commitment that is six months old without progress is a signal to revisit at the next product-review meeting.
- Are commitments landing? A commitment that has landed is converted into a pattern-catalogue entry ("the canary-gate preset for prompt-only changes landed in Q1 2027 — here is the catalogue entry describing the new behaviour and the FDE-side implications").
- Are commitments being withdrawn? A commitment the PM has walked back is captured as a learning — the FDE team's next similar ask knows the frame the PM did not buy.

The commitment register lives in the knowledge-base repo; its review is folded into the quarterly knowledge-base review.

## Running the three archetypes

**Archetype A — US healthcare payer.** The pattern catalogue carries archetype-A-specific entries for VPC-install Langfuse, Datadog OTel Collector for HIPAA pilots, BAA clauses the FDE commonly negotiates, Zendesk-to-gold-set pipes. The failure-mode library carries archetype-A-specific variations — the HIPAA breach-notification pathway is a named branch in the "cost blowout" and "retrieval-drift" entries. The deployment playbook has a "HIPAA VPC install" entry with eighteen numbered steps; the security-review-answer library has thirty-plus entries covering the HIPAA-specific questions CISO offices typically ask.

**Archetype B — US federal agency.** The knowledge base has a cleared-and-uncleared split. Pattern-catalogue entries covering FedRAMP-generic patterns live in the standard repo; entries that require controlled-information context live in a cleared repo accessible only to FDEs with the appropriate clearance. The deployment playbook has a "FedRAMP Moderate air-gapped" entry; some steps reference cleared supplements. Security-review answers cover NIST 800-53 Rev 5 controls, NIST AI RMF 1.0 mapping, and OMB M-24-10 compliance posture <!-- needs-research: confirm which OMB AI memos are currently in force; M-24-10 is the one most commonly cited but successor memos may apply -->.

**Archetype C — EU retail bank.** The pattern catalogue carries archetype-C-specific entries for Azure AI Foundry under EU Data Boundary, Langfuse self-hosted in Azure with Entra ID SSO, DORA major-ICT-incident integration patterns, EU AI Act Article 72 post-market monitoring artefacts. The failure-mode library has archetype-C-specific variations covering DORA incident-classification branches; the security-review-answer library has thirty-plus entries covering GDPR / DORA / EU AI Act questions.

## Anti-patterns

**Knowledge base without ownership.** Every entry is "owned by the FDE team." No one is accountable; nothing is reviewed. Six months in, half the entries are stale; the hit-rate collapses. Every entry has one named accountable owner; orphaned entries are deprecated within 30 days.

**Write-once entries.** The entry is authored at the first pilot that shipped the pattern and never revisited. The second pilot's FDE finds an entry that assumes a platform behaviour the platform no longer has. The quarterly staleness sweep is the counter; `last_reviewed` within 90 days is the target.

**Catalogue entry promoted too early.** A Chapter 03 pattern writeup is promoted to a catalogue entry after one pilot. The entry's generalisations do not hold; the second pilot's FDE finds the entry misleading. The two-writeups-across-two-FDEs threshold is the promotion test.

**Playbook with no gotchas.** The deployment playbook's steps are clean but the "known gotchas" section is empty. The second FDE executing the playbook hits the exact gotcha the first FDE hit silently. Known gotchas are not optional; a playbook entry without them is marked incomplete until the author adds at least one.

**Failure-mode library written as research.** The failure-mode entries are five-page essays. The on-call FDE at 3am does not read them; they call another FDE. The library's failure is in the entry's length, not its content; three pages maximum, decision-tree structured, scannable in five minutes.

**Security-review library with marketing voice.** The security-review-answer library's entries are written in the vendor's marketing voice ("our industry-leading security posture..."). A CISO reads the answer and perceives evasion. The library's voice is specific, technical, evidence-backed; the marketing adaptation (if any) is a separate artefact.

**Onboarding read-list not maintained.** The onboarding read-list has not been refreshed in two years; new FDEs read entries from a prior platform version. The quarterly review's refresh step is explicit; the read-list is versioned too.

**Knowledge base inside the customer's tool.** The FDE's team wiki is inside the customer's Confluence (because "we're already there"). Cross-pilot search does not work; entries leak customer-specific detail to the FDE's next customer. The knowledge base is vendor-internal, named vendor repo / vendor Confluence space / vendor wiki; customer tools are referenced but not inhabited.

**Vocabulary drift.** Each FDE invents new `surface` values in their capture logs; the knowledge base's vocabulary file ages out; search across pilots breaks. The vocabulary-alignment step of the quarterly review is the backstop; a value that appears in three pilots' capture logs without appearing in the shared vocabulary file is a signal to normalise.

**Commitment register ignored.** The PM made ten verbal commitments in last quarter's product-review meeting. The register records them; no one reviews. Six months later, three have landed, four are forgotten, and the FDE team cannot tell which. The register's quarterly review is standing; forgotten commitments are either followed up or formally closed.

## Summary

- The FDE knowledge base is the vendor-internal, FDE-team-owned, versioned, searchable store that answers "how does the fourth FDE onboarded to a similar pilot find the pattern without re-asking the three who came before?"
- Four slices: pattern catalogue (writeups of recurring architectural / decision shapes), failure-mode library (symptom → diagnosis → response → root-cause check, scannable in five minutes), deployment playbook (ordered procedures with preconditions, checkpoints, rollback, known gotchas), security-review-answer library (AI-specific extensions to SIG / CAIQ / SOC 2 response libraries).
- One schema across all slices: nine common fields plus slice-specific body. Semver versioning: major for shape changes, minor for new archetype, patch for corrections.
- Every entry has one accountable owner. Accountability rotates with pilot assignments. Entries without owners are deprecated within 30 days.
- Quarterly knowledge-base review: staleness sweep, new-entry review, deprecation sweep, onboarding read-list refresh, vocabulary alignment. 90 minutes, standing attendance.
- Staleness index: fraction of active entries reviewed within 90 days (target > 90 %). Hit-rate: fraction of new-FDE questions answerable from the knowledge base (target > 70 %).
- Boundaries: `ai-enablement-lead` adapts mature pattern-catalogue entries into customer-education content; `ai-platform-engineer` maintains the platform docs the deployment playbook assumes.
- The commitment register from Chapter 05 is a fifth artefact in the knowledge-base repo, reviewed at the quarterly review.
- Archetype A has HIPAA-specific entries and playbooks; Archetype B has a cleared / uncleared split for FedRAMP and controlled-information content; Archetype C has GDPR / DORA / EU AI Act-specific entries.
- Ten anti-patterns: no ownership, write-once entries, premature promotion, playbook without gotchas, failure-mode library as research, security-review library with marketing voice, stale onboarding read-list, knowledge base in the customer's tool, vocabulary drift, ignored commitment register.
- This is the module's final chapter. The pilot now has a capture log (Chapter 02), four design-partner artefact shapes on cadence (Chapter 03), two packet forms for platform influence (Chapter 04), a quarterly product-review discipline (Chapter 05), and a compounding knowledge base (Chapter 06) that scales the FDE team. The FDE is the vendor product team's field eye; the FDE team is the organisational leverage the vendor's roadmap depends on.
