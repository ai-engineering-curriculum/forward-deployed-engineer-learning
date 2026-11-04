# Chapter 04 — The Weekly / Fortnightly Pilot Review with the Customer

## Why this chapter exists

The weekly pilot review is the single cadence whose absence kills a pilot slowly. The exec demo ([Chapter 02](./02-engineer-executive-demos.md)) lands a moment; the executive brief ([Chapter 03](./03-exec-brief-and-five-minute-standup.md)) carries the narrative between moments; the weekly review is the rhythm that makes both of those possible. The sponsor who does not have a standing weekly conversation with the FDE is the sponsor who loses faith slowly and silently — and the FDE finds out at the renewal conversation, which is five months too late.

The cadence is load-bearing for four distinct reasons, and the FDE who treats the review as a status report is operationalising only one of the four.

1. **Rhythm.** The sponsor's calendar is full; the pilot competes with every other initiative in the sponsor's quarter for cognitive space. A standing weekly keeps the pilot on the sponsor's docket by default. The absence of a standing slot is not neutrality — it is the slow decay of the pilot's priority.
2. **Trust accrual.** Trust is not produced by a single great meeting; it is produced by twelve ordinary meetings in a row, run on time, with minutes that land in four hours and decisions that get executed. The weekly review is where trust compounds — or where it leaks.
3. **Early-signal surfacing.** The SME lead who is quietly losing confidence, the operations lead whose team is routing around the pilot, the IT liaison who has just read an uncomfortable incident report — these signals surface in a review attended by all four customer-side stakeholders and nowhere else. A weekly cadence catches a problem at week two; a monthly cadence catches it at week six, after the problem has become political.
4. **Scope-creep catchment.** The bespoke feature ask, the "while you're in there" request, the adjacent-department stakeholder who shows up and wants coverage — all of them surface in the weekly. [Chapter 06](./06-scope-discipline-and-decision-log.md) is the response; the weekly is the detection mechanism. A pilot without a weekly review has no systematic scope-detection; scope creeps in through the side door of ad-hoc Slack threads.

This chapter is one of the two gate dimensions named in [Chapter 01](./01-consulting-craft-overlay.md). A pilot whose weekly reviews have slipped — whose status is always green with no rationale, whose minutes publish erratically, whose risk register has not been updated in a month — has failed consulting craft regardless of how beautiful the technical delivery is. The gate is non-negotiable.

## Who is in the room and what they are measuring

The weekly review is not a vendor-plus-sponsor meeting. It is a cross-functional meeting with five to seven customer-side seats, and each seat has a different measurement lens. The FDE who runs the review for the sponsor alone is running a one-dimensional review.

| Seat | Role | What they measure |
|---|---|---|
| FDE (vendor) | Narrative + technical | Pilot delivery against success criteria; the week's decisions |
| Customer sponsor | Outcome | Business outcome against the sponsorship thesis — "did the pilot move the KPI I committed to?" |
| Customer SME lead | Quality | Does the output hold up under subject-matter scrutiny? Is the gold set growing? |
| Customer operations lead | Workflow impact | Is the pilot changing the team's workflow in ways the team can absorb? Where is friction? |
| Customer IT liaison | Systems posture | Integrations, security posture, operational health of the deployed surface |
| Vendor `account-executive` (sometimes) | Commercial signals | Renewal signals, expansion signals, pricing conversation readiness |
| Vendor `customer-success-manager` (often) | Relationship | Health of the relationship, adoption curve, executive alignment |

The FDE is the only vendor-side seat whose role is **technical narrative** — the translation of the week's engineering into the sponsor's outcome frame. The `customer-success-manager` is adjacent but distinct — they own the relationship across the vendor's full product surface; the FDE owns this pilot's delivery. The role handshake with the `customer-success-manager` is written in [mod-101 Chapter 04](../mod-101-fde-discipline-and-position/04-adjacent-customer-facing-roles.md); the short version is: the CSM speaks for the account; the FDE speaks for the pilot; when the CSM is in the room, the FDE runs the agenda but the CSM owns any conversation that crosses into other vendor products or the account's commercial posture.

A review where only the FDE and the sponsor attend has failed to convene the review. Two consecutive weeks of a missing SME lead is a signal — the FDE raises it 1-on-1 with the sponsor, not as a public callout.

## The standing agenda — 45 to 60 minutes

The review runs 45 to 60 minutes against a standing agenda with per-section time budgets. A 45-minute review is the steady-state shape; a 60-minute review is the active-delivery shape where decisions are landing. Beyond 60 minutes, the review is doing something the review is not supposed to do.

| # | Section | Budget | Owner |
|---|---|---|---|
| 1 | Status + RAG | 5 min | FDE |
| 2 | Metric updates against success criteria | 10 min | FDE + SME lead |
| 3 | Decisions required this week | 10 min | FDE (presents); sponsor (decides) |
| 4 | Open risks and mitigations | 10 min | FDE + risk-owner |
| 5 | Customer open items | 10 min | Customer sponsor or SME lead |
| 6 | Action items + owners | 5 min | FDE (reads back); all (confirm) |

Three mechanics make this agenda work.

- **Each section has a named owner.** The FDE does not present everything — the SME lead owns the quality narrative in section 2, the sponsor or SME lead owns the customer-open-items column in section 5. The review is a conversation, not a broadcast.
- **The time budgets are enforced.** The FDE wears the role of facilitator; if section 2 is running long, section 3 is not sacrificed — the FDE either extends the meeting by five minutes with explicit permission or carries an item into next week.
- **Nothing lives outside the agenda.** New items surface in section 5 (customer open items) or go into the risk register. "Just one more thing" at the end of a 60-minute meeting is the mechanism by which a 60-minute meeting becomes a 90-minute meeting.

### Section 2 — the metric spine

Section 2 is the spine of the review. The success criteria are the criteria the sponsor signed off on at scoping — [mod-102](../mod-102-customer-discovery-and-pilot-scoping/) wrote that dialogue. Every week, section 2 updates each criterion with: the current measurement, the trend against last week, the trend against the baseline, and the trajectory against the pilot's end-date target.

The FDE's metric spine is not three pretty charts; it is a short table the SME lead can read in forty seconds.

| Criterion | Baseline | Last week | This week | Target | Trajectory |
|---|---|---|---|---|---|
| Average handle time (minutes) | 11.4 | 9.8 | 9.3 | 8.5 | On track |
| First-contact resolution (%) | 62 | 68 | 69 | 72 | On track |
| Policy-citation rate (%) | n/a | 94 | 96 | ≥ 95 | Met |
| Member CSAT (1–5) | 4.1 | 4.2 | 4.1 | ≥ 4.2 | At risk |

If the metric spine is not on the slide, the review has drifted into vendor theatre.

## Status rubric — RAG with mandatory rationale

Status is green, yellow, or red. A RAG without a one-sentence rationale is theatre — a colour the sponsor cannot interrogate.

| Status | Definition | Rationale sentence must name |
|---|---|---|
| Green | On track; no blocker; all success criteria trending toward target | The trend — "AHT trending down 0.5 min/wk, FCR trending up, no open blockers" |
| Yellow | On track with a named risk requiring attention | The risk — "SME-lead review bandwidth at 60% of plan this week; may slip gold-set expansion by one week" |
| Red | Slipping against ≥ 1 success criterion; requires a decision this week | The criterion slipping and the decision asked for — "CSAT at 4.1 vs 4.2 target; recommend a two-week extension of the human-in-the-loop checkpoint on benefit-interpretation replies; decision requested from sponsor" |

Three disciplines around the rubric.

- **Yellow is not red-in-waiting.** Yellow is a legitimate state with a named risk and a mitigation. An FDE who is afraid to go yellow because "the VP might panic" has a sponsor-relationship problem that no colour will fix.
- **Red asks for a decision.** A red that does not ask for a specific decision is a red that cannot be resolved. Every red rolls into section 3.
- **The rationale sentence fits in one line of the minutes.** If the rationale is a paragraph, the colour is wrong — the FDE is either pre-explaining a red as a yellow, or is adding colour the status does not need.

A RAG that stays green for twelve consecutive weeks on a delivery pilot is itself a signal — either the pilot is remarkable, or the status rubric is broken. The FDE reviews the colour against the metrics with the SME lead monthly; a persistently-green pilot with a persistently-flat CSAT is not green.

## The decisions-required discipline

Section 3 is where the review earns its keep. A review that produces no decisions in four consecutive weeks has drifted into a status broadcast; the sponsor is not being asked to decide anything, which means either the pilot is on autopilot (rare) or the FDE is deciding things unilaterally (common).

Every decision item has five fields:

```
Decision: <one-sentence statement of what needs to be decided>
Options: <2–3 concrete options, each with a trade-off named>
FDE recommendation: <which option, with one sentence of rationale>
Decision owner: <named customer-side person — usually the sponsor or SME lead>
By-when: <specific date>
```

Example:

```
Decision: Should the pilot extend human-in-the-loop review to benefit-interpretation replies for two more weeks?
Options:
  (a) Extend HIL to 2026-11-22; CSAT recovery observed before full automation.
  (b) Lift HIL now; accept CSAT risk; monitor daily for two weeks.
  (c) Lift HIL only for FAQ-class replies; keep HIL on benefit-interpretation indefinitely.
FDE recommendation: (a) — the CSAT delta is the sponsor's success criterion and the two-week extension costs roughly 8 SME-reviewer-hours.
Decision owner: VP Member Experience
By-when: End of this review
```

Two disciplines govern this section.

- **Never more than three decisions per review.** More than three is either a batch-up from weeks of accumulated hesitation (the FDE should have surfaced earlier) or an attempt to push through decisions under time pressure. Three is the hard limit; the discipline mirrors [mod-111 Chapter 05](../mod-111-field-to-product-loop/05-roadmap-influence.md)'s three-pattern discipline.
- **The decision-owner is a named customer-side person, not "the team."** "The team will decide" is how a decision never happens. The sponsor decides, or the SME lead decides, or the IT liaison decides — and the name goes in the minutes.

Decisions that cannot be made in the review become their own agenda item next week with the condition that unblocks them explicitly named — "sponsor to consult with VP Compliance by Thursday; decision in next week's review." A deferred decision without a condition is a deferred decision that will defer again.

## The open-risks register

The risks register is a living artefact, not a slide. Each risk has six columns:

| Risk | Probability | Impact | Mitigation | Owner | Next-review-date |
|---|---|---|---|---|---|
| SME lead on parental leave weeks 8–12; gold-set expansion at risk | High | Medium | Deputy SME `sme-deputy@customer` shadows weeks 6–7; gold-set entries pre-staged | SME lead | 2026-11-14 |
| Azure AI Foundry capacity reservation not confirmed for Q1 2027 | Medium | High | IT liaison escalated to vendor account team; backup region identified | IT liaison | 2026-11-14 |
| Benefit-interpretation CSAT below target two weeks running | Medium | High | Extended HIL under review (section 3) | FDE | 2026-11-07 |
| Member-portal single-sign-on cert rotation window conflicts with pilot cutover | Low | Medium | IT scheduling coordination in flight | IT liaison | 2026-11-21 |

Four disciplines keep the register honest.

- **Probability and impact are not vibes.** The FDE defines "high / medium / low" explicitly in the risk register's README — e.g., high probability = > 50% likelihood in the next four weeks; high impact = would slip a success criterion by ≥ 2 weeks. The SME lead and sponsor can calibrate against the definition.
- **Risks age out.** A risk that has sat on the register for four weeks without discussion is either closed (the condition has passed; close it with a one-sentence rationale) or escalated (the mitigation has not worked; the risk is now a red-status decision). A stale risk register is a risk register that has stopped being used.
- **Every risk has a named owner.** Owner is a person, not "both teams." If both sides own something, the FDE picks the primary and the other becomes a contributor; a two-owner risk is a zero-owner risk.
- **The register is shared, not vendor-only.** The customer SME lead and IT liaison can add rows between reviews; the FDE is the editor, not the gatekeeper.

## The customer-open-items column

Section 5 is the column most often skipped — because the FDE is running the agenda and section 5 feels like free time at the end of the meeting. The discipline is to *protect* it.

The customer-open-items column is what the customer wants on the agenda — the IT liaison's question about the gateway's token-budget behaviour, the operations lead's observation about a workflow friction, the SME's concern about a new category of SME dissatisfaction. If the FDE never surfaces customer open items, the review is a vendor broadcast, not a review.

Two tactics keep section 5 alive.

- **The FDE opens section 5 by naming it.** "This is the customer open-items block. What does the team want on the agenda today that I haven't surfaced?" The naming is the invitation. Silence in section 5 for three consecutive weeks is itself a signal — either the pilot is remarkably smooth, or the customer is holding items for a different channel (which is worse).
- **The FDE sometimes pre-stages section 5.** If the IT liaison mentioned a concern in a 1-on-1 earlier in the week, the FDE asks in advance: "I'd like to give that time in Thursday's review — would you be comfortable raising it there?" Pre-staging builds the customer's muscle to use the column.

## Fortnightly vs weekly — when to pick which

The default is weekly during active delivery, fortnightly in steady-state operation or when the customer's calendar cannot support weekly. The FDE's choice between the two is a conversation with the sponsor, not a unilateral decision.

| Pilot phase | Default cadence | Rationale |
|---|---|---|
| First 30 days of delivery | Weekly, 60 min | Decisions density highest; trust still accruing; scope-creep risk highest |
| Mid-delivery (30–90 days) | Weekly, 45–60 min | Metric spine stabilising; risk register active |
| Steady-state operation | Fortnightly, 60 min | Delivery complete; monitoring and refinement phase |
| Pre-renewal (last 30 days) | Weekly, 45 min | Renewal conversation forming; executive narrative sharpening |
| Hand-off period (last 6–8 weeks) | Weekly, 45 min + hand-off pair sessions ([Chapter 07](./07-knowledge-transfer-to-customer-engineering.md)) | Knowledge transfer density high |

A hybrid pattern often works for mature pilots: a weekly 30-minute **ops sync** (status + metrics + action items only) combined with a fortnightly 60-minute **full review** (adding decisions, risks, customer open items). The hybrid preserves rhythm while respecting the customer's calendar. The hybrid is a sponsor conversation, not an FDE unilateral move — the sponsor's time budget is the constraint.

Switching from weekly to fortnightly mid-pilot without an explicit sponsor conversation is a form of sponsor neglect; the FDE who quietly drops a week to "give the team space" is the FDE who discovers at week six that the sponsor believed the pilot had wrapped up.

## Minutes — publish within 4 business hours

Minutes publish within 4 business hours of the review ending. The sponsor confirms by end-of-day. The minutes live in a per-pilot repo or shared doc that the FDE owns and the customer has read access to.

The 4-hour SLA is not arbitrary — it is the window in which the attendees still remember the conversation and can correct the record. Minutes published 24 hours later are minutes the attendees skim without reading; minutes published 48 hours later are minutes that are not confirmed. A minutes SLA that regularly slips to next-day is a review discipline that has collapsed.

### Minutes template

```markdown
# Weekly Review — <customer-short> × <vendor-pilot-name> — <YYYY-MM-DD>

**Attendees**
- <FDE name> (vendor, FDE)
- <Sponsor name> (customer, sponsor)
- <SME-lead name> (customer, SME lead)
- <Ops-lead name> (customer, operations)
- <IT-liaison name> (customer, IT)
- <CSM name> (vendor, customer-success-manager) [optional]

**Status**
- RAG: Yellow
- Rationale: SME-lead review bandwidth at 60% of plan this week; may slip gold-set
  expansion by one week.

**Metric updates (full table in metrics/<YYYY-MM-DD>.md)**
- AHT 9.3 min (target 8.5, trajectory on track)
- FCR 69% (target 72, trajectory on track)
- Policy-citation 96% (target ≥95, target met)
- Member CSAT 4.1 (target ≥4.2, at risk — see decision 1)

**Decisions made this review**
1. Extend HIL on benefit-interpretation replies to 2026-11-22.
   - Owner: VP Member Experience
   - Decided: in-review
2. SME-deputy shadow starts 2026-11-10 for weeks 8–12 coverage.
   - Owner: SME lead
   - Decided: in-review

**Decisions deferred**
- None this week.

**Risks updated (full register in risks/)**
- Added: member-portal SSO cert rotation window conflict (low / medium).
- Closed: Azure capacity reservation confirmed by vendor account team.

**Customer open items raised this week**
- Operations lead asked about weekend queue-depth dashboarding
  (FDE to spec by 2026-11-10).

**Action items**
- [FDE] Spec weekend queue-depth dashboard; by 2026-11-10.
- [SME lead] Confirm SME-deputy's access to gold-set repo; by 2026-11-07.
- [IT liaison] Schedule SSO cert rotation coordination call; by 2026-11-14.

**Next review**
- 2026-11-14, 09:30–10:30 local
- Standing agenda; no additions pre-staged.

**Confirmation**
- Minutes published: 2026-11-07 14:02 local
- Sponsor confirmation: <pending>
```

A minutes template that fits on a screen is a template that gets used. A minutes template that spills across three pages is a template that becomes aspirational.

## Per-pilot review repo structure

The weekly review's artefacts live in a per-pilot repo (git) or a shared doc tree (SharePoint / Google Drive / Confluence, depending on the customer's posture). The structure is consistent across pilots so the FDE's cognitive load does not compound.

```
pilot-<customer-short>-<vendor-pilot-name>/
├── README.md                       # pilot one-pager, success criteria, cadence
├── agenda/
│   └── YYYY-MM-DD.md               # weekly agenda (pre-staged)
├── minutes/
│   └── YYYY-MM-DD.md               # weekly minutes (published within 4 hours)
├── metrics/
│   └── YYYY-MM-DD.md               # weekly metric snapshot
├── risks/
│   └── register.md                 # the living risk register
├── decisions/
│   └── YYYY-MM-DD-<decision-slug>.md   # one file per decision, countersigned
└── exec-brief/                     # Chapter 03 artefacts
```

The `decisions/` directory is the countersigned decision log — [Chapter 06](./06-scope-discipline-and-decision-log.md) writes its full discipline. The `risks/register.md` is the live risk table. The `agenda/` and `minutes/` directories are symmetrical — every meeting has both.

Access rights: the customer has read access to everything in the repo; the FDE has write access; the sponsor has write access to `decisions/` for countersignature. A pilot whose repo is vendor-read-only is a pilot that has not yet decided who the artefacts belong to — the answer is "jointly."

## Escalation triggers

Escalation is the transition from "the FDE and the sponsor are managing this" to "we need someone above the sponsor involved." The trigger is not a single event; it is a combination of risk severity and mitigation-exhaustion.

| Trigger | Escalation target | Example |
|---|---|---|
| Red status two consecutive reviews with no clear path back to green | Customer sponsor's sponsor (VP or C-level above sponsor) | CSAT slipping with HIL extension not sufficient |
| Risk crosses threshold where the mitigation requires resource the sponsor cannot authorise | Customer sponsor's sponsor | SME-lead bandwidth requires org-level reallocation |
| Vendor-side delivery slip that will affect the sponsor's quarterly commitment | Vendor `ai-engineering-manager` | FDE is one engineer short for the next sprint |
| Security or compliance incident surfacing in the pilot | Both sides' security leadership, per [mod-108](../mod-108-enterprise-security-and-compliance/) | Suspected PII in a model trace |
| Role-boundary pressure — the customer is asking the FDE to do account-executive or professional-services work | Vendor `ai-engineering-manager` + `account-executive` | Customer requests FDE "for one more quarter" in a role the FDE is not |

The single most important discipline around escalation: **escalations never happen as a surprise in the weekly review.** The FDE socialises the escalation 1-on-1 with the sponsor *before* the review in which it becomes public. The sponsor has the right to know in advance when a risk is about to be escalated to their boss; a sponsor who hears "I need your VP involved" for the first time in a room with their VP present is a sponsor who will not trust the FDE again.

The 1-on-1 pre-socialisation also gives the sponsor the option to escalate themselves — often the right move, because the escalation lands better from the sponsor than from the vendor.

## Running the three archetypes

**Archetype A — US mid-size healthcare payer.** The weekly review is attended by the customer's privacy office as a standing seat — not optional. The privacy office reviews the minutes before publication for inadvertently-captured PHI; a mis-published minutes entry that names a member or a specific benefit-interpretation case is a HIPAA event. The minutes template's "metric updates" section carries aggregate numbers only; member-level evidence lives in a separately-controlled gold-set repo, not in the pilot minutes. The standing attendee list adds one row — "Customer Privacy Officer (customer, privacy)" — and the 4-hour SLA is a 4-hour-plus-privacy-review SLA; the FDE drafts in 2 hours, the privacy office reviews in 2 hours, publication is at 4 hours. <!-- needs-research: confirm whether HIPAA privacy-rule 45 CFR §164.502 minimum-necessary provisions constrain the member-identifiable detail that can appear in meeting minutes shared across vendor and customer -->

**Archetype B — US federal agency.** The FedRAMP agency review is inside the enclave. Minutes are controlled — they do not leave the authorised environment; they are not emailed. Attendees sign a standing privileged-information agreement. The per-pilot repo lives on the agency's internal collaboration platform under the pilot's ATO boundary; the FDE does not have personal-device access. The standing agenda shape is identical, but the vehicle for publication is different — the minutes are posted to the agency-internal repo and a courtesy "minutes published" one-liner (no content) is sent to the vendor's cleared channel. Escalations to vendor `ai-engineering-manager` go through a cleared-channel process; routine operational conversations stay inside. <!-- needs-research: confirm which OMB memos govern federal-agency meeting-minutes handling for AI pilots under FedRAMP Moderate and whether specific M-24-10 provisions touch customer-facing review artefacts -->

**Archetype C — EU retail bank.** The weekly review is co-attended by the model-risk function — a representative from the CRO's organisation sits in and treats the open-risks register as a feeder to the bank's own DORA ICT-third-party risk log. Risks that affect the bank's EU AI Act Article 72 post-market monitoring posture are tagged in the register and copied to the bank's model-risk tracking system within 24 hours. The customer open-items column (section 5) frequently carries model-risk-function items — questions about model drift reporting, questions about the bank's auditor's upcoming review. The FDE's minutes are additionally reviewed for anything that would need to appear in the bank's DORA incident-register; a risk that escalates to "major ICT-related incident" under DORA enters a separate reporting process that the FDE triggers via the IT liaison. <!-- needs-research: confirm DORA Article 17 major-ICT-incident classification thresholds and the specific reporting SLAs that would apply to a pilot-surfaced incident -->

## Anti-patterns

**No agenda.** The weekly runs without a published agenda; the FDE "walks the team through where we are"; the meeting rambles; no decisions are tracked. A review without a standing agenda is a review that cannot build rhythm. The agenda is published 24 hours in advance even when it mirrors last week's — the publication itself is the discipline.

**The vendor monologue.** The FDE talks for 40 of the 45 minutes; the customer stakeholders nod; section 5 is silent. The review is a vendor broadcast. The symptom is that the customer open-items column is empty three weeks running; the fix is to engineer section 5 by pre-staging items with customer stakeholders 1-on-1 earlier in the week.

**The RAG without rationale.** Status is green. Why? "Because things are going well." A RAG without a one-sentence rationale naming the trend, risk, or slipping criterion is theatre. The sponsor cannot interrogate it; the FDE cannot defend it; the minutes cannot preserve the state.

**The stale risk register.** Risks older than four weeks that have not been discussed are not risks — they are furniture. The symptom is a risk register with 14 rows and no `next-review-date` field. The fix is the aging discipline: at every review, every risk older than 4 weeks is either explicitly closed, explicitly escalated, or has a new mitigation and a fresh review-date.

**Five decisions brought at once.** The FDE arrives at the review with five decisions; three get rushed; two get deferred; the sponsor leaves uncertain which ones were actually decided. The three-decision limit is uncomfortable because every decision feels urgent; the discipline is that if a fourth decision is urgent, a decision from last week should already have been made.

**The missing minutes.** Minutes are "coming later today" and arrive Friday afternoon, 48 hours after the Thursday review. The attendees have forgotten the specifics; the sponsor does not confirm because the content is no longer fresh; the next review's agenda has no anchor. A review whose minutes are more than one business day late is a review that did not produce an artefact.

**The sponsor-absent review.** The sponsor misses three consecutive reviews; the FDE runs them with the SME lead and operations lead; the FDE does not raise the sponsor absence 1-on-1. At week six the sponsor is out of context on the pilot; a red status surprises them. Three weeks of sponsor no-show is a yellow risk on the register and a 1-on-1 conversation, not a thing to wait out.

**The two-hour review.** The 45-minute review stretches to 60; the 60-minute stretches to 90; the 90-minute becomes a standing 2-hour slot. Meeting budget bloat is a signal that the agenda has stopped being enforced. The fix is to cut back to 45 minutes for two consecutive reviews, move items to 1-on-1s or decision docs, and rebuild the discipline.

## Summary

- The weekly / fortnightly pilot review is one of the two gate dimensions of consulting craft. A pilot whose review cadence has slipped has failed the gate regardless of technical excellence.
- The cadence is load-bearing for four reasons: rhythm (keeps the pilot on the sponsor's docket), trust accrual (compounds across ordinary meetings), early-signal surfacing (catches problems at week two not week six), scope-creep catchment (detection mechanism for [Chapter 06](./06-scope-discipline-and-decision-log.md)).
- Seven seats, each with a measurement lens: FDE (narrative + technical), sponsor (outcome), SME lead (quality), operations lead (workflow), IT liaison (systems), `account-executive` (commercial, sometimes), `customer-success-manager` (relationship, often). Role handshake with the `customer-success-manager` is in [mod-101 Chapter 04](../mod-101-fde-discipline-and-position/04-adjacent-customer-facing-roles.md).
- Standing 45–60 min agenda: status + RAG (5), metric spine (10), decisions (10), risks (10), customer open items (10), action items (5). Time budgets are enforced; nothing lives outside the agenda.
- Status rubric: green / yellow / red with a mandatory one-sentence rationale naming the trend, risk, or slipping criterion. A RAG without rationale is theatre.
- Decisions discipline: five fields per decision (statement, options, FDE recommendation, decision-owner, by-when); never more than three decisions per review; decision-owner is a named person.
- Risks register: six columns (risk, probability, impact, mitigation, owner, next-review-date); probability/impact defined explicitly; risks age out at four weeks; the register is shared with the customer.
- Customer open items (section 5) is protected explicitly; three weeks of silence is itself a signal.
- Fortnightly vs weekly: weekly during active delivery, fortnightly in steady state; hybrid (30-min weekly ops sync + 60-min fortnightly full review) is a sponsor conversation, not a unilateral switch.
- Minutes publish within 4 business hours; sponsor confirms by EOD; per-pilot repo has `agenda/`, `minutes/`, `metrics/`, `risks/`, `decisions/`, `exec-brief/`.
- Escalations go through `ai-engineering-manager` (vendor side) or the sponsor's sponsor (customer side), and are socialised 1-on-1 before they appear in the weekly — surprise escalations in the review destroy sponsor trust.
- Archetype A adds a privacy-office standing seat and a 4-hour-plus-privacy-review minutes SLA; Archetype B runs inside the enclave with controlled minutes and a cleared-channel escalation path; Archetype C co-attends with the model-risk function and feeds the register to the bank's DORA ICT-third-party risk log.
- Eight anti-patterns: no agenda, vendor monologue, RAG without rationale, stale risk register, five decisions at once, missing minutes, sponsor-absent review untreated, two-hour review bloat.
- [Chapter 05](./05-customer-side-resistance.md) is next: customer-side resistance — the SME lead, the operations department, the IT review; the radical-transparency-plus-pragmatic-respect stance; the escalation pathway.
