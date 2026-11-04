# Chapter 03 — The Executive Brief and the Five-Minute Standup

## Why this chapter exists

[Chapter 02](./02-engineer-executive-demos.md) wrote the quarterly executive demo — the forty-five-minute event where the VP sees the pilot and makes a continuation judgment. The demo is not enough. Between demos, the same VP must hold a position on this pilot across a portfolio of twelve other initiatives, defend it in their own operating reviews, and brief their own leadership in corridor conversations the FDE will never attend. The VP's position on the pilot, between demos, is not formed by the pilot's technical reality — it is formed by the one-pager on top of the Monday reading pile and the five minutes of narrative at a standup two Thursdays from now.

This chapter writes two artefacts and one ritual. The **exec-brief one-pager** — the artefact the VP reads *before* the meeting rather than a ten-pager they will not. The **five-minute standup narrative** — the clock-enforced spoken version of the brief for the VP's weekly operating cadence. And the **delivery ritual** — when the brief lands in the VP's inbox, how the standup opens, how out-of-date claims get red-lined before the meeting begins. The exec brief is the artefact that keeps a pilot alive between the demos that renew it. [Chapter 04](./04-weekly-pilot-review-cadence.md) will write the deeper weekly pilot review for the sponsor-plus-SME audience; this chapter is the thinner, VP-facing instrument that sits above it.

Executive-narrative quality is an optimisation dimension in [Chapter 01](./01-consulting-craft-overlay.md)'s five-dimension model. The exec brief is where that optimisation compounds most: a well-maintained brief is read by the VP's chief of staff, quoted in the VP's own leadership updates, re-skimmed six months later at renewal, and lifted as a template by the next FDE in the portfolio. A badly-maintained brief — stale, uncited, nine bullets under the "asks" section — is a political liability the FDE did not price.

## Why a VP reads a one-pager and never the ten-pager

The VP is not under-informed; the VP is over-asked. A healthcare-payer COO is deciding a Medicare Advantage bid, a reorg of the member-services operation, a new contact-centre vendor, three IT investments, and a regulatory posture shift — simultaneously. The FDE's pilot is one of thirty initiatives on the VP's one-pager stack this quarter. The ten-pager is a request the VP will not fulfil; the one-pager is an attention-budget match.

The one-pager succeeds because it is *skim-first, deep-on-request*. The VP reads the one-pager in ninety seconds before the standup, forms a position, and surfaces specific questions that pull the longer deck forward in the conversation. The longer deck is **backup** — it exists, it is linked, it is ready — but the VP never reads it cover-to-cover. The ratio is right when the FDE's weekly time budget to maintain the brief is thirty minutes and to maintain the deck is zero minutes most weeks (the deck sits ready; updates are per-material-event, not per-cadence).

Three consequences of this attention-budget shape:

- **The one-pager carries the narrative; the deck carries the evidence.** If the one-pager cannot survive on its own, the brief has failed.
- **The one-pager has a shape the VP's eye learns.** Same four sections, same order, same table shape, every fortnight. Novelty in the shape costs attention that should go to the content.
- **The one-pager is honest about what it does not contain.** A brief that pretends to be complete is a brief the VP eventually distrusts; a brief that says "see linked deck §3 for the eval-methodology detail" is a brief the VP learns to use as an index.

## The four sections of the exec-brief one-pager

The one-pager is four sections and one table. In order:

### Section 1 — The problem, in the customer's language

One paragraph. Written by the FDE; approved by the sponsor at the pilot's kickoff and re-approved on each quarterly refresh. The problem is stated in the words the operating function uses internally — not the vendor's product language, not AI terminology, not the pilot's codename. If the payer's member-services team calls it "handle time on complex benefit questions," the brief calls it that. If the bank's customer operations team calls it "case triage for cross-border transfers," the brief calls it that.

The sponsor-approved paragraph is load-bearing: when the brief reaches an audience the sponsor did not expect (the VP forwards it; a board member asks for context), the first paragraph is what that audience uses to decide whether the pilot is addressing a real operational problem they recognise. A problem paragraph written in vendor language is a brief that evaporates at forwarding.

The paragraph does not change fortnight-over-fortnight unless the scope shifts. The problem is the stable anchor.

### Section 2 — The pilot shape

A short bullet list. Three sub-headings — **Scope**, **Success criteria**, **Non-goals** — each with two to four bullets. Lifted directly from the pilot's [mod-102](../mod-102-customer-discovery-and-pilot-scoping/) scoping artefacts; no re-writing.

```markdown
## Pilot shape

**Scope**
- Member-inquiry resolution for benefit-interpretation questions, routed to
  the AI-assisted drafting surface.
- 15 % of inbound volume by end of pilot; HIPAA Minimum Necessary scope on
  retrieved context; human-in-the-loop on every send.

**Success criteria (sponsor-signed at kickoff)**
- AHT reduction ≥ 25 % on in-scope tickets (baseline 11m42s).
- FCR ≥ 68 % on in-scope tickets (baseline 61 %).
- Policy-citation rate ≥ 95 % on sampled drafts.
- Member CSAT delta non-negative versus control queue.

**Non-goals**
- Autonomous send (human-in-the-loop remains for the pilot).
- Clinical-advice content (routed to clinical reviewer; out of pilot scope).
- Out-of-network benefit interpretation (routed to specialist queue).
```

The non-goals section is the brief's single most useful bullet list. The VP who reads "autonomous send is not in scope" at week six will not later ask "why are we not just auto-sending these" at week fourteen. The non-goals carry the pilot's political perimeter.

### Section 3 — Current state

RAG with rationale, mini-table of the KPI shape, timeline position. This is the section that *does* change fortnight-over-fortnight.

```markdown
## Current state

**Overall RAG: AMBER.**

- AMBER because FCR has held at 64.1 % against a 68 % target for three
  consecutive weeks; AHT and citation-rate are tracking to target.
- Timeline: week 9 of 16; go/no-go gate at week 12.
- Three risks on the open-risks register; one trending — see linked
  weekly-review minutes (2026-11-02).

**KPI snapshot (fortnight ending 2026-11-02):**

| Metric | Baseline | Current | Target | Confidence |
|---|---|---|---|---|
| AHT (in-scope) | 11m42s | 9m01s | ≤ 8m47s | n=1,284 tickets, 6-week window |
| FCR (in-scope) | 61 % | 64.1 % | ≥ 68 % | n=1,284 tickets, 6-week window |
| Policy-citation rate | n/a | 96.2 % | ≥ 95 % | n=200 sampled drafts, SME-reviewed |
| Member CSAT delta | 0 | +0.07 | ≥ 0 | n=412 survey responses (preliminary) |
```

The RAG has a rationale sentence. "AMBER because FCR has held at 64.1 % against a 68 % target for three consecutive weeks" is a rationale; "AMBER" alone is not. The VP forms a position on the *rationale*, not the colour. [Chapter 04](./04-weekly-pilot-review-cadence.md) will enforce the RAG-with-rationale rule across the weekly review; here it is the exec-brief default.

### Section 4 — What's needed

A short bullet list. **Never more than three asks.** Each ask is decision-shaped: the decision-maker named, the by-when named, the consequence of the delay named.

```markdown
## What's needed

1. **Decision — SME pod expansion.** Sponsor + VP to approve +1 SME reviewer
   seat to clear the citation-review backlog blocking FCR improvement. Needed
   by 2026-11-09 to recover before week-12 gate.
2. **Unblocker — data-platform access for case-triage sampling.** Member-services
   IT lead needs sponsor nudge on the DPIA-approved extract (ticket ID
   DPIA-2026-404, open since 2026-10-14). Needed by 2026-11-07.
3. **Exec ask — alignment on the Member Experience advisory board readout
   slot.** 10-minute briefing at the 2026-12-04 session; sponsor to confirm
   agenda inclusion by 2026-11-15.
```

Three asks is the ceiling, not the floor. A brief with zero asks in a given fortnight (nothing is blocked, nothing requires a decision) is honest and healthy; the FDE says so explicitly ("No asks this fortnight — the pilot is executing against the plan"). A brief with five asks is a brief that has either accumulated unresolved asks across cycles (a discipline failure — see [Chapter 06](./06-scope-discipline-and-decision-log.md)) or escalated non-asks into the brief to fill space (a credibility failure).

## The before/after KPI table — the artefact of record

Section 3's mini-table is a snapshot. The before/after KPI table is the full version — a one-table artefact linked from the brief, refreshed on the same fortnightly cadence, and lifted directly into the quarterly executive demo as its evidence substrate. Columns:

| Column | Content | Notes |
|---|---|---|
| Metric | The named KPI | One row per success-criteria KPI + one row per operating metric the sponsor tracks |
| Baseline | The pre-pilot measurement | Named with its measurement window and sample size |
| Current | The latest measurement | Timestamped; flagged "preliminary" if < 1 month data |
| Target | The sponsor-signed target | From [mod-102](../mod-102-customer-discovery-and-pilot-scoping/) |
| Delta | Current − Baseline | Signed; absolute and relative |
| Confidence | Source + sample size + known caveats | Dashboard ID or report link |

Every row cites its source. A row without a source is a row the VP cannot defend upward; a row without a sample size is a row the FDE cannot defend when the sponsor's analytics lead asks. The rule is uniform: cite source, cite sample size, flag preliminary.

### Archetype A — US healthcare payer

```markdown
# Before/after KPI table — Acme Health member-inquiry pilot
**Last updated:** 2026-11-02 by <FDE name>. Version 7.
**Data-handling subtitle:** Metrics computed on de-identified tickets per
HIPAA Safe Harbor (§164.514(b)(2)); member identifiers removed at the
analytics boundary; all rates aggregate (no per-member statistics).

| Metric | Baseline | Current | Target | Delta | Confidence |
|---|---|---|---|---|---|
| AHT (in-scope, seconds) | 702 (n=3,104, 2026-06 to 2026-08) | 541 (n=1,284, last 6w) | ≤ 527 | -161s (-22.9 %) | Contact-centre BI dashboard MX-114; SME-reviewed |
| FCR (in-scope, %) | 61.0 % (n=3,104) | 64.1 % (n=1,284) | ≥ 68.0 % | +3.1 pp | Contact-centre BI dashboard MX-114 |
| Policy-citation rate (%) | n/a (new metric) | 96.2 % (n=200 drafts) | ≥ 95.0 % | n/a | Weekly SME-sampled audit; gold set v3 |
| Member CSAT delta | 0 (control) | +0.07 (n=412, preliminary) | ≥ 0 | +0.07 | Post-resolution survey, 5-point scale; 3-week window |
| Cost-per-resolution ($) | 8.42 (FY2026 Q2 actual) | 6.91 (last 6w) | ≤ 6.50 | -$1.51 (-17.9 %) | Finance monthly actuals; allocated-cost model |
| Draft acceptance rate (%) | n/a (new metric) | 78.3 % (n=1,284) | ≥ 75.0 % | n/a | Agent-UI telemetry; operational metric |
```

The data-handling subtitle is non-negotiable for the HIPAA archetype. The rates are aggregate; no row carries a per-member statistic; the Safe Harbor citation is specific. <!-- needs-research: confirm HIPAA Safe Harbor at 45 CFR §164.514(b)(2) is the correct citation shape when a vendor-produced analytics dashboard references de-identification posture in an executive artefact -->

### Archetype B — US federal agency

```markdown
# Before/after KPI table — Policy-analyst drafting pilot
**Last updated:** 2026-11-02 by <FDE name>. Version 4.
**Authorisation boundary:** Pilot operates within [agency] GovCloud ATO
boundary; this artefact is read inside the enclave only; printed copies
destroyed per records-retention schedule after each session.

| Metric | Baseline | Current | Target | Delta | Confidence |
|---|---|---|---|---|---|
| Analyst drafting time (hours, rounded) | ~6 per memo (n≈40, FY26 Q2) | ~3 per memo (n≈25, last 6w) | ≤ 3 | ~-3h (~-50 %) | Analyst time-tracking; rounded to nearest 30m |
| Draft-to-review iteration count | ~3.5 rounds (n≈40) | ~2 rounds (n≈25) | ≤ 2 | ~-1.5 rounds | Review-platform telemetry; rounded |
| Citation-verification rate (%) | n/a (new metric) | 94 % (n≈60 drafts) | ≥ 95 % | n/a | Senior-analyst sampled audit; preliminary |
| Analyst-reported quality score (1–5) | 3.2 (FY26 Q2 survey) | 3.6 (last-month survey) | ≥ 4.0 | +0.4 | Monthly analyst survey; preliminary (n < 1 month at this cadence) |
```

Rounded metrics are policy — the agency's data-sensitivity posture forbids the per-ticket precision used in Archetype A. The artefact's read-in-enclave notation is operational. <!-- needs-research: confirm which FedRAMP Moderate / agency records-retention posture governs printed artefacts carrying operational statistics inside the enclave -->

### Archetype C — EU retail bank

```markdown
# Before/after KPI table — Customer-support reply + case triage pilot
**Last updated:** 2026-11-02 by <FDE name>. Version 9.
**EU AI Act Article 72 posture:** Post-market monitoring record maintained
by the bank (data controller); this artefact is a vendor-side input to that
record, not the record itself. **DORA ICT-third-party register entry:**
REG-2026-0117 (bank's third-party-risk catalogue).

| Metric | Baseline | Current | Target | Delta | Confidence |
|---|---|---|---|---|---|
| Case-triage accuracy (%) | 82.4 % (n=4,210, 2026-Q2) | 89.6 % (n=2,108, last 6w) | ≥ 90.0 % | +7.2 pp | Triage-QA dashboard CO-21; QA-team sampled |
| Average handle time (seconds) | 638 (n=4,210) | 491 (n=2,108) | ≤ 500 | -147s (-23.0 %) | Contact-centre BI dashboard CO-14 |
| Auto-resolve rate (tier-1 cases, %) | 11.0 % (n=4,210) | 18.4 % (n=2,108) | ≥ 20.0 % | +7.4 pp | Case-management system export; weekly |
| Policy-citation rate (%) | n/a (new metric) | 97.1 % (n=220 drafts) | ≥ 95.0 % | n/a | Model-risk-team sampled audit; v4 rubric |
| Customer Effort Score (1–7, lower=better) | 3.8 (2026-Q2) | 3.5 (last 6w, preliminary) | ≤ 3.6 | -0.3 | Post-resolution survey; n=612; preliminary |
| Model-risk approval status | n/a | Conditional (pending gold-set v5) | Full | Partial | Model-risk function 2026-10-28 minutes |
```

The model-risk-approval row is a non-numeric metric but belongs in the table because the bank's model-risk function tracks it alongside the numeric ones. <!-- needs-research: confirm EU AI Act Article 72 post-market monitoring record shape and the vendor's input-to-record-of-record distinction for a deployer operating under Article 26 / 72 -->

## The exec-brief template — lift-and-use

The full template, in a form the learner copies into their pilot's `exec-brief.md` and refreshes fortnightly:

```markdown
# Exec brief — [Pilot codename]

**Last updated:** 2026-11-02 by <FDE name>. **Version 7.**
**Sponsor:** <sponsor name and title>
**VP audience:** <VP name and title>
**Linked:** [pilot deck] · [KPI table of record] · [weekly-review minutes]
[**Data-handling subtitle, if applicable:** HIPAA de-identification posture / FedRAMP authorisation boundary / EU AI Act Article 72 reference]

## 1 — The problem, in [the customer function]'s language

[One paragraph, sponsor-approved at kickoff, re-approved on each quarterly
refresh. No vendor language, no AI terminology, no pilot codename. The
operational pain as the function states it internally.]

## 2 — The pilot shape

**Scope**
- [2–4 bullets, lifted from mod-102 scoping artefacts.]

**Success criteria (sponsor-signed at kickoff)**
- [2–4 bullets, each a named KPI with a numeric target.]

**Non-goals**
- [2–4 bullets, the pilot's explicit perimeter.]

## 3 — Current state

**Overall RAG: [GREEN / AMBER / RED].**

- [RAG colour with rationale — one sentence naming the metric or risk
  driving the colour.]
- Timeline: week N of M; next gate at week [G].
- [Open-risk register pointer; aged-risk count if any.]

**KPI snapshot (fortnight ending [date]):**

| Metric | Baseline | Current | Target | Confidence |
|---|---|---|---|---|
| [metric] | [baseline w/ n] | [current w/ n] | [target] | [source, sample size, preliminary flag] |
| ... | ... | ... | ... | ... |

[Full before/after KPI table of record linked above.]

## 4 — What's needed

1. **[Decision / Unblocker / Exec ask] — [one-line title].** [Decision-maker
   named; by-when named; consequence of the delay named.]
2. ...
3. ...

[Maximum three. If zero, write: "No asks this fortnight — the pilot is
executing against the plan."]

---

**Appendix: linked materials**
- [Weekly-review minutes, last 4]
- [Decision log]
- [Evaluation methodology (mod-107 reference)]
- [Security-review response library (mod-108 reference)]
```

The template is the artefact; the FDE lifts it, fills it in, refreshes it fortnightly. The template's section numbering (1, 2, 3, 4) is deliberate — the VP learns "section 4 is where the asks are" and skims to it under time pressure.

## The five-minute standup narrative

The VP's weekly operating cadence has the pilot on the agenda for five minutes. The clock is enforced; the VP has four other agendas; there is no slippage budget. The FDE's standup budget:

| Segment | Time | Content |
|---|---|---|
| Context | 30 s | One sentence: "This is week 9 of the 16-week member-inquiry pilot at the drafting-assistance surface." |
| Before-state recap | 60 s | Two sentences on the operational problem; one sentence citing the baseline metric. The same VP has heard this fourteen times; the recap is for the two other attendees who have not. |
| Current-state highlights | 90 s | Three bullets maximum. RAG colour + rationale; two numeric highlights from the KPI table; one risk-register item if trending. |
| What's next | 90 s | What the next fortnight will prove or disprove. The one metric that will move; the one decision that will land; the one risk that will clear or escalate. |
| Asks | 60 s | Up to three. Each ask stated as a decision-shaped sentence with named owner and by-when. |

Five minutes; five segments; content proportionate to attention. The FDE rehearses the standup once before delivery — not to memorise, but to confirm the clock. A standup that runs seven minutes is a standup that cost the VP time they will reclaim from the next fortnight's slot (the pilot drops to three minutes next time). A standup that runs three minutes is a standup where the FDE left asks unspoken; the VP's operating rhythm is "use the slot you have."

The narrative structure maps to the brief's four sections but inverts the order — current state leads, problem recap is brief, asks close. The VP who read the brief in the morning does not need the problem recap at length; the two other attendees who did not read the brief get thirty seconds of context in segment 1 and sixty in segment 2. The brief and the standup are twins, not duplicates.

### The failure-recovery clause

If the standup opens and the VP says "I didn't get to the brief" — the FDE adjusts the time allocation in the first ten seconds: context stretches to 60s, before-state recap stretches to 90s, current-state compresses to 60s, what's-next compresses to 60s, asks hold at 60s. The *shape* of the narrative is preserved; the *weights* shift. A rehearsed FDE can do this live; an unrehearsed FDE delivers the brief-having-been-read version to a VP who did not read the brief and loses the room.

## When to update the brief — the cadence

Four update triggers; one version-control discipline.

| Trigger | Window | Note |
|---|---|---|
| Standing fortnightly | Every two weeks, same business day | The default rhythm; brief refreshed and sent whether or not anything material has changed |
| On request | 24h SLA | Sponsor or VP chief of staff asks for the latest; the FDE refreshes and sends within one business day |
| Material change | Within 24h | New red risk; sponsor change; timeline slip > 1 week; scope change; incident with executive visibility |
| Pre-meeting | 48h before the standup | The fortnightly refresh is timed to this cadence so the standing brief *is* the pre-meeting brief |

Version control sits at the top of the file: last-updated date, author, version number. Version 7 → 8 is a visible increment the VP's chief of staff notices; a brief that has not incremented in four weeks is a brief the chief of staff raises with the FDE. The version number is a cheap signal; the FDE respects it.

A brief older than two weeks is stale by definition. A stale brief in the VP's hands is worse than no brief — it makes a decision on a metric that is no longer current. The FDE's calendar carries a standing fortnightly block for the refresh; the discipline is of the same kind as the weekly-review minutes SLA in [Chapter 04](./04-weekly-pilot-review-cadence.md).

## The pre-meeting delivery ritual

Three rituals, executed every standup cycle.

**T-minus 48h — brief in the inbox.** The refreshed brief lands in the VP's inbox (and the sponsor's, and the chief of staff's) 48 hours before the standup. The subject line is uniform: `Exec brief — [Pilot codename] — v[N] — [date]`. The body is three sentences: "Version [N] of the exec brief is attached and linked. Key change this cycle: [one sentence]. Happy to answer any questions in writing before the standup." The brief itself is the attachment; the body is the hook.

**T-minus 24h — standing offer to pre-answer.** The FDE's standing offer — "happy to answer any questions in writing before the standup" — is not polite filler. It is an explicit invitation for the VP's chief of staff to surface tough questions before the live slot. A chief of staff who sends "before Thursday, can you confirm the citation-rate denominator?" twenty-four hours ahead is a chief of staff saving the FDE from being ambushed on Thursday. The FDE answers in writing and updates the brief if the answer changes anything.

**T-minus zero — red-line stale claims.** The standup opens; the FDE's first move is to red-line (verbally, in the first thirty seconds of segment 1) any claim in the brief that has changed between T-minus-48h and T-minus-zero. "The brief says FCR is 64.1; an overnight recomputation has it at 64.3; this does not change the colour or the ask but I want you to have the current number." The red-line is a credibility deposit. The FDE who red-lines their own stale numbers is the FDE whose brief the VP trusts; the FDE who hopes the stale number goes unnoticed is the FDE whose brief the chief of staff starts verifying against the dashboard.

The ritual is three touchpoints across 48 hours. Total FDE time cost: under an hour. The return is attention compound: the VP enters the standup already having read the brief, the chief of staff enters already having surfaced their tough question, and the FDE enters having pre-cleared the stale-number risk.

## Running the three archetypes

**Archetype A — US healthcare payer.** The exec brief carries the HIPAA data-handling subtitle immediately under the version line; the KPI table's rates are aggregate; no row carries a per-member statistic. The sponsor-approved problem paragraph is reviewed by the payer's privacy office at each quarterly refresh (not at each fortnight — the paragraph is stable). Member CSAT metrics are flagged preliminary until four weeks of post-resolution survey data accumulate; CSAT deltas are reported as point estimates with the window named, not as single-fortnight spot values. The brief is emailed inside the payer's email boundary; forwarding outside the payer's organisation requires sponsor approval. <!-- needs-research: confirm whether HIPAA Business Associate Agreement standard templates constrain forwarding of de-identified aggregate metrics in a vendor-produced executive artefact -->

**Archetype B — US federal agency.** The exec brief is read in-enclave only; it is not emailed. The authorisation boundary is named in the subtitle; the data-sensitivity posture requires rounded metrics (nearest 30 minutes on time, nearest full percentage point on rates, nearest five on counts under 100). The problem paragraph uses the mission-owner's language, not the agency-wide language; the specific mission office — not the agency — is the audience. The KPI table omits any metric whose precision would exceed the aggregate-reporting posture of the mission office's existing dashboards. A brief printed for an in-enclave session is destroyed at session end per the records-retention schedule. Standing fortnightly cadence operates inside the agency's weekly program-review slot, not a vendor-side calendar event. <!-- needs-research: confirm whether current OMB AI memoranda govern the aggregate-reporting posture of vendor-produced executive artefacts read inside a FedRAMP-authorised enclave -->

**Archetype C — EU retail bank.** The brief carries two references in the subtitle: the EU AI Act Article 72 posture statement (the bank is the data controller and maintains the post-market monitoring record; the vendor's brief is an input, not the record) and the DORA ICT-third-party register entry ID. The model-risk function is a standing recipient of the brief alongside the VP — the brief is a dual-audience artefact and the FDE pre-briefs the model-risk lead on any amber-to-red RAG transition before the VP's standup. The KPI table includes the model-risk approval status row; EU customers reading the brief understand the row even though it is non-numeric. GDPR Article 5 minimisation constrains any customer-identifying example in the problem paragraph. <!-- needs-research: confirm EU AI Act Article 72 post-market monitoring reporting granularity for a Chapter III high-risk system deployed by a bank under the deployer obligations in Article 26 -->

## Anti-patterns

**The ten-pager.** The FDE produces a comprehensive fortnightly deck; the VP reads the title slide and the last slide; the middle is wasted. The one-pager is the artefact of record; the deck is backup. An FDE whose fortnightly cycle produces no one-pager has produced no artefact the VP will read.

**The vanity metric.** The KPI table carries "number of prompts refined this fortnight" or "number of SME training sessions held." These are activity metrics, not outcome metrics; they belong in the weekly-review minutes, not the VP's brief. The VP does not operate on activity; the VP operates on member-experience outcomes. A table row that cannot be traced to a sponsor-signed success criterion does not belong in the exec brief.

**The uncited number.** A row in the KPI table reads "64.1 %" with no source column, no sample size, no window. The sponsor's analytics lead asks "where did that come from?" and the FDE improvises. The rule is uniform: cite source, cite sample size, flag preliminary. A row without these three pieces is not ready for the brief.

**The stale brief.** The brief's "last updated" date is three weeks ago; the FDE skipped a fortnightly cycle; the VP is making a Thursday decision on a two-and-a-half-week-old number. Standing fortnightly is a floor, not an aspiration. If a cycle is genuinely blocked (the SME is on leave, the data refresh failed), the FDE still ships the brief on time with the section-3 note "no new data this cycle; next refresh expected 2026-11-16" — the brief ships; the versioning holds.

**The hero narrative.** Current-state section opens with "I shipped the citation-rate improvement this fortnight." The VP does not want the FDE's hero narrative; the VP wants the pilot's outcome. The brief is written in the pilot's voice, not the FDE's. Replace "I shipped" with "the citation-rate improvement shipped; measurement at 2026-11-02 shows…"

**The metric cherry-pick.** The current-state section reports the first fortnight's eye-catching number ("AHT dropped 30 % in week 2") as if it were a steady-state outcome. First-fortnight numbers are ramp-in artefacts; they do not survive to steady state. The brief reports *windowed* metrics (last 6 weeks, last 4 weeks) with the window named; it never reports a single-fortnight spot number as if it were the pilot's standing outcome. Preliminary metrics are flagged `(preliminary, n=…)` and never given decision weight.

**The "we are green" bluff.** The RAG is green; the rationale is "on track." No rationale is a bluff. Green with a rationale ("GREEN — all three KPIs tracking to target; three risks on the register, none trending") is credible; green-no-rationale is a brief the sponsor stops trusting. The rule is RAG-plus-rationale, not RAG alone.

**The brief with no asks (that should have asks).** The fortnight carries a known blocker — the DPIA ticket is stuck at the privacy office, the SME pod is under-resourced — and the asks section is silent. The FDE has traded the political cost of making an ask for the operational cost of the blocker. The brief exists *to surface the asks*; silence on asks when asks exist is the brief's single most expensive failure.

## Summary

- The VP reads the one-pager and never the ten-pager. The one-pager is the artefact of record; the deck is backup, linked, and exists for specific questions the VP pulls forward.
- Four sections: the sponsor-approved problem paragraph in the customer function's language; the pilot shape with scope, success criteria, and non-goals; current state with RAG-plus-rationale and the KPI snapshot; what's needed with up to three decision-shaped asks.
- The before/after KPI table of record is a separate artefact linked from the brief; every row cites source, sample size, and flags preliminary. The three archetypes constrain the table's posture: HIPAA aggregate rates, FedRAMP rounded metrics, EU AI Act Article 72 reference plus model-risk status.
- The five-minute standup narrative has a strict time budget: 30s context, 60s before-state recap, 90s current state, 90s what's next, 60s asks. The clock is enforced; the FDE rehearses once and holds the shape.
- Four update triggers: standing fortnightly, on request (24h SLA), material change (within 24h), pre-meeting (48h before). Version control at the top of the file; a stale brief (> 2 weeks) is worse than no brief.
- Pre-meeting ritual at T-minus-48h (brief in inbox), T-minus-24h (standing offer to pre-answer), T-minus-zero (red-line stale claims in the first 30 seconds). Total FDE time cost under an hour per cycle.
- Archetype A carries a HIPAA data-handling subtitle with aggregate-only rates; Archetype B reads in-enclave with rounded metrics and the authorisation boundary named; Archetype C carries the EU AI Act Article 72 posture plus the DORA ICT-third-party register entry and is co-addressed to the model-risk function.
- Eight anti-patterns: the ten-pager, the vanity metric, the uncited number, the stale brief, the hero narrative, the metric cherry-pick, the "we are green" bluff, the brief with no asks when asks exist.
- Next: [Chapter 04](./04-weekly-pilot-review-cadence.md) — the deeper weekly/fortnightly pilot review for the sponsor-plus-SME audience that sits under the exec brief.
