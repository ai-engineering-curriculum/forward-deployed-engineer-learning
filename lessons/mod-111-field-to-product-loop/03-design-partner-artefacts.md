# Chapter 03 — Design-Partner Artefacts and the Cadence Product Leadership Can Count On

## Why this chapter exists

Chapter 02's capture log is where the FDE's raw material accumulates. It is not an artefact product leadership consumes. A Notion page with 142 signals, 38 of them triaged `platform_gap_candidate`, is a reading task the PM will not do; a monthly one-page pattern writeup, a quarterly 'what we learned' one-pager, and a per-outcome customer success story are the shapes product leadership *will* consume.

This chapter has four jobs. Specify the four canonical artefacts — the monthly **pattern writeup**, the quarterly **pain-point taxonomy**, the quarterly **'what we learned' one-pager**, and the per-outcome **customer success story** — with templates and depth-of-technical-detail guidance for each. Write the **customer sign-off path** that keeps artefacts from leaking. Write the **cadence** that product leadership can count on — including the mechanics of what happens when the FDE misses a window. Catalogue the anti-patterns that reduce artefact readiness (the gate dimension from Chapter 01).

Artefact readiness is a gate; triage accuracy is a gate; everything else in this chapter is optimisation.

## The four artefacts in one table

| Artefact | Cadence | Length | Consumer | Signals drawn from |
|---|---|---|---|---|
| Pattern writeup | Monthly | 1–2 pages | Vendor PM + vendor engineering | Capture log entries triaged this month |
| Pain-point taxonomy | Quarterly | 2–3 pages | Vendor PM + `ai-engineering-manager` + product leadership | Rolled-up capture log from the quarter |
| 'What we learned' one-pager | Quarterly | 1 page | Product leadership + sales + marketing | The quarter's successes and failures |
| Customer success story | Per sold outcome (irregular) | 1–3 pages | Marketing + sales + customer | Specific pilot's outcome against mod-102 success criteria |

Four artefacts; three consumers overlap; four different purposes. The FDE authors; the customer approves where sign-off is needed; the vendor publishes. Nothing in this chapter requires the FDE to become a writer — the templates carry most of the shape.

## Writing for product, not for engineering

Chapter 03's audience is not the audience chapters 02 through 10 of mod-104 through mod-110 were written for. Product leadership reads differently than engineering. Four shifts the FDE learns to make:

1. **Lead with the pattern, not the incident.** Engineering reads post-mortems; product reads patterns. A writeup that opens with "INC-42 on Nov 4 2026…" asks the reader to piece the generalisation together. A writeup that opens with "Prompt-only changes propagated through our standard canary gate cause SLO burn when the quality dimension is new to the judge rubric" surfaces the generalisation in the first sentence.
2. **Quantify where possible, abstract where not.** Product decisions on prioritisation live or die by magnitude. A writeup that says "several customers have hit this" is a writeup the PM cannot prioritise; a writeup that says "three of our twelve pilots in the last quarter saw this shape; each cost roughly one engineer-day of unplanned work" is one the PM can rank against other investments. Where specific numbers cannot be shared (Archetype A's HIPAA posture, Archetype B's enclave), *bucket* the magnitude rather than invent it (`<!-- needs-research: ...-->` as an escape hatch for unverifiable claims).
3. **Carry the customer voice.** The verbatim SME quote from the capture log is the writeup's strongest asset. A writeup with two customer quotes reads as field evidence; a writeup with none reads as the FDE's opinion. Chapter 02's capture discipline exists because Chapter 03's writeups need the quotes.
4. **Name the recommendation explicitly.** Product can act on *a specific proposed change*. An artefact that describes a problem in three paragraphs and ends with "we should think about this" does not produce an action. The template's recommendation section is non-negotiable; even a "no change needed, this is just a pattern we see" is a recommendation — of silence as the right action.

These shifts are the connective tissue between mod-110's engineering-side writing and mod-112's executive-side communication.

## The monthly pattern writeup

The FDE's workhorse artefact. Authored the second week of the month, about the previous month's confirmed-triage patterns. Length: one to two pages. The template:

```markdown
# Pattern writeup: [short pattern name]

**Date:** 2026-11-14
**Author:** <FDE name>
**Pilot(s):** acme-health, bank-c — Archetype A and C
**Surface:** canary-gate + online-judge
**Triage class:** platform gap
**Consumer:** vendor PM + senior-agentic-ai-engineer

## The pattern in one paragraph

Prompt-only changes propagate through the standard canary gate in a window that is
too short to detect SLO burn caused by a rated-dimension drift. In two pilots
we have observed the pattern: a prompt change that is behaviourally correct on
the offline eval drops the online-judge pass-rate within the first 25 minutes
of 10 % canary. The current gate's progression cadence does not surface this
before 100 % traffic.

## Evidence

**Pilot A (acme-health, 2026-11-04).** Bundle v4.2.0; online-judge pass-rate
dropped 0.84 → 0.71 in 25 minutes; rolled back via kill-switch after SEV-2
declared. See INC-42 post-mortem. Verbatim from the customer SME lead at the
2026-11-05 review: *"The judge caught it, but we wish the canary had held
longer so the eval would have caught it first."*

**Pilot C (bank-c, 2026-10-18).** Bundle v2.8.3; online-judge pass-rate dropped
0.81 → 0.74 in 40 minutes; mitigated by prompt revert without SEV declaration.
Verbatim from the bank's model-risk lead at the 2026-10-25 quarterly review:
*"If the gate had required a day at 1 %, our offline sweep would have had time
to surface it."*

## Why it is a pattern, not a bug

Both pilots have the same canary-gate shape (bundled from the shared feature-flag
platform's prompt-rollout preset). Both have a comparable online-judge
infrastructure. Both are missing the same rated dimension in the judge rubric
at the time of promotion. The fix at each pilot is independent (update the
judge rubric to score the missing dimension) but the *platform shape* — the
gate's default progression for prompt-only changes — is common.

## Recommended direction

A canary-gate preset for **prompt-only changes** that requires a 24-hour hold
at 1 % traffic before any expansion. The current preset collapses the hold to
the default (30 minutes) because the diff is bundle-metadata-only; the preset
should be sensitive to the *class* of change, not the diff size.

Interface sketch and risk/scope in the paired Chapter 04 platform-request
packet (link).

## What we are doing in the interim

Both pilots now apply a pilot-local override on the canary-gate preset to hold
at 1 % for 24 hours on any prompt-template change. The override is maintained
by the FDE at each pilot; cost is approximately 15 minutes per release.

## Related signals

- sig-2026-11-04-0012 (INC-42 acme-health)
- sig-2026-10-18-0003 (bank-c prompt-regression)
- sig-2026-09-22-0014 (acme-health earlier near-miss)

## Customer sensitivity + sign-off

- acme-health: sensitivity medium; names pilot ID. Sign-off requested from the
  customer SME lead by 2026-11-16; the pilot ID will be redacted to
  "Archetype-A pilot" if sign-off is declined.
- bank-c: sensitivity high; names pilot ID. Sign-off requested from the bank's
  communications office by 2026-11-16; the writeup will be redacted to
  "Archetype-C pilot" and the model-risk-lead quote will be replaced with a
  paraphrase if sign-off is declined.
```

Twelve sections. One page expanded to two. The FDE spends roughly ninety minutes authoring a well-shaped writeup from a fully-populated capture log; the discipline of Chapter 02's capture is what keeps that ninety minutes from stretching to a full day.

### The depth-of-technical-detail dial

Product leadership is not a monolith. The vendor's product engineering reads deeper than the vendor's product marketing; the PM sits between. The writeup's depth is one dial; the FDE learns to pick a setting.

- **Low** — "the shape and the recommendation, no implementation detail." Appropriate for marketing circulation; appropriate when the writeup is competing for the PM's attention among twelve others.
- **Medium** — "the shape, the evidence, the recommendation, one layer of implementation detail." The default for the monthly pattern writeup.
- **High** — "the shape, the evidence, the recommendation, full implementation detail, failure modes." Appropriate for direct consumption by `senior-agentic-ai-engineer` or `ai-platform-engineer`; usually written as the paired Chapter 04 platform-request packet rather than as the Chapter 03 writeup.

The template is medium by default. When the pattern crosses into a platform request, the writeup stays medium (it is the organisational narrative) and the Chapter 04 packet carries the high-detail interface sketch.

## The quarterly pain-point taxonomy

A rollup, not a report. Every quarter, the FDE (or the FDE team collectively) reads the quarter's capture log, groups the confirmed-triage entries into pain-point classes, and produces a two-to-three-page document that answers two questions for product leadership:

1. **Where is the pain concentrated this quarter?** (Which surfaces? Which triage classes? Which pilots?)
2. **How has the shape shifted from last quarter?** (Classes new this quarter; classes resolved; classes trending up.)

The structure:

```markdown
# Pain-point taxonomy: Q4 2026

**Period:** 2026-10-01 to 2026-12-31
**Pilots covered:** 12 active, 3 completed, 2 in handoff
**Author:** <FDE name> on behalf of the FDE team
**Consumer:** vendor PM, ai-engineering-manager, product leadership

## Top-line shape

- 142 capture entries triaged this quarter.
- Triage distribution: 61 % bug, 24 % feature, 15 % platform gap.
- Platform-gap distribution across level-40 surfaces: 45 % senior-agentic-ai-engineer,
  32 % ai-platform-engineer, 23 % ai-engineering-manager.
- Pilots with no confirmed platform gaps: 4 of 17. Pilots with ≥ 3 confirmed
  platform gaps: 2 of 17.

## Pain-point classes (ranked by signal count)

### 1 — Canary-gate cadence for prompt-only changes (8 signals, 5 pilots)

[Short writeup; link to the monthly pattern writeup from November.]

### 2 — Response-library gaps on AI-specific security-review questions (6 signals, 4 pilots)

[Short writeup; link.]

... [continues for N classes, typically 5–10]

## Classes new this quarter

- [list]

## Classes resolved this quarter

- [list — signals that were captured as platform gaps and are now shipped
  by the core team]

## Classes trending up

- [list — classes with increasing signal count quarter-over-quarter]

## What the FDE team is asking for

- From senior-agentic-ai-engineer: [top 3 platform requests ranked]
- From ai-platform-engineer: [top 3 platform requests ranked]
- From ai-engineering-manager: [top 3 organisational asks — headcount, process,
  tooling]

## Customer sensitivity + sign-off

[per-pilot sensitivity; which classes name specific customers vs are
generalised; who signs off on what by when]
```

The taxonomy is the artefact that gets read in the quarterly product-review meeting (Chapter 05). The FDE reads from it; the PM holds a printed copy; the `ai-engineering-manager` uses it to shape the next quarter's headcount and prioritisation conversation.

### The rankings decision

Ranking pain-point classes is a contested exercise. The honest ranking is *signal count × average time cost per signal*; a class that has eight entries at thirty minutes each ranks below a class with four entries at six hours each. The taxonomy ranks by composite, not by raw count, to surface the heavy-tailed costs the signal-count view hides. The ranking formula is named in the taxonomy's introduction so the PM can understand the ordering.

## The 'what we learned' one-pager

A different beast. The pain-point taxonomy is inside-facing; the 'what we learned' one-pager is published more broadly — the vendor's engineering all-hands, the sales enablement deck, sometimes the customer newsletter (anonymised). Length: one page. Tone: honest, specific, generalisable.

The structure:

```markdown
# What we learned this quarter — Q4 2026

**From the FDE team, on 2026-12-20.**

## Three things that worked

1. [One paragraph. Specific. Names the pilot's shape; links the FDE pattern writeup.
   Carries the metric where sharable.]
2. [Same shape.]
3. [Same shape.]

## Three things that did not

1. [One paragraph. Honest. Names the pilot's shape; names what we would do
   differently; carries the metric where sharable. Blameless in the mod-110
   Chapter 06 sense — the shape is in the system, not the engineer.]
2. [Same shape.]
3. [Same shape.]

## One thing the field is telling us about the product

[One paragraph. The one most actionable signal of the quarter. Names the
pattern; names the recommendation; links to the paired platform-request packet.]

## What's next

[Two to four bullets. The FDE team's focus for next quarter. Answers the
reader's implicit "what should I expect from the field in the next 90 days?"]
```

Seven paragraphs, one page. The one-pager is what gets quoted in the next all-hands; the quarterly pain-point taxonomy is what gets reviewed in the next product-review meeting.

## The customer success story

Marketing's requirement. Per sold outcome, not on a fixed cadence. The FDE authors the technical substrate; the vendor's product-marketing team adapts to the public-facing shape. The FDE's version is the version the *customer's* internal reviewers read; the public version is the version marketing publishes once the customer's legal and communications offices sign off.

### The FDE's version — the technical substrate

```markdown
# Customer success story (technical substrate): [pilot name]

**Customer:** <name> (Archetype A — US healthcare payer)
**Workflow:** member-inquiry resolution
**Pilot period:** 2026-03-01 to 2026-09-30 (go-live + 6 months steady state)
**Author:** <FDE name>
**Status:** Draft; awaiting customer sign-off

## Business context in two paragraphs

[Who the customer is. Why they wanted this. The mod-102 success criteria they
committed to at scoping.]

## The pilot in five sentences

[What was built. What customer system it sits inside. Data sources. Model
provider. Observability posture. One line on security-compliance.]

## The outcome against the success criteria

[Table of the mod-102 success criteria with the measured outcome at the 6-month
mark. Each row cites the dashboard / report / SLO measurement the number came
from. If a criterion was missed, the row says so and explains why.]

| Success criterion | Target | Measured | Source |
|---|---|---|---|
| ... | ... | ... | ... |

## The two or three decisions that mattered

[Specific. "We chose VPC install over SaaS because X." "We chose Langfuse
self-hosted because Y." "We held the canary at 1 % for 24 hours on
prompt-only changes after INC-42." Each decision links its ADR or its pattern
writeup.]

## What the SME team is saying at 6 months

[Two to three verbatim quotes from the customer's SME team, dated, with
speaker attribution. Each quote has a customer-approval line below it noting
the customer's signed consent to use it externally.]

## What did not work the first time

[Honest. One or two shapes the FDE and the customer had to re-approach. The
customer's reviewers appreciate the candour; the vendor's marketing team will
decide whether to keep this section in the public version.]

## What we would recommend to a similar customer

[Three to five bullets. The deployment choices, the pilot-scope decisions,
the handoff cadence the FDE would recommend to a next-similar-customer FDE.
This section is the article's reusability.]

## Customer sign-off

- [ ] Customer SME lead
- [ ] Customer operations lead
- [ ] Customer communications office
- [ ] Customer legal

Each sign-off is a named approver and a date.
```

The technical substrate is one to three pages. Marketing's adaptation is typically one page plus a hero image. The FDE's job is the substrate; the marketing team's job is the adaptation; the customer owns the final approval.

### The honeymoon-risk discipline

A customer success story published two weeks after go-live is almost always wrong — the honeymoon effect inflates adoption metrics; early SME praise exaggerates the finished shape; one bad quarterly review would overturn the narrative. The FDE insists on the six-month data cut: the metrics reported are the metrics at the six-month-post-go-live mark, and the review that produced them is the fourth or later quarterly review. A success story drafted against the first-month data is held until the fourth-month data confirms it.

The exception is a pilot that completes before the six-month mark (a short pilot with a clean "yes, we ship it to broader rollout" outcome at four months). In that case the story cites the *final* measurement, not an interim one.

## Customer sign-off — the single most important discipline

Every artefact that leaves the FDE's hands and reaches an audience the customer does not control requires the customer's sign-off. The sign-off path is the discipline that prevents the "customer artefact that leaks" anti-pattern from Chapter 01.

### The three signoff levels

- **Courtesy review.** The customer sees the artefact before the vendor publishes it; the customer's objections produce edits; there is no formal signature. Appropriate for the monthly pattern writeup when the pilot is named generically (the writeup's pilot_id is redacted to "Archetype A" or similar).
- **Customer approval.** The named customer approver signs the artefact before publication; the artefact names the pilot specifically. Appropriate for the monthly pattern writeup when the pilot is named; the quarterly pain-point taxonomy if it carries pilot-specific content; the 'what we learned' one-pager if a specific pilot is mentioned.
- **Customer legal + communications sign-off.** The customer's legal office and communications office both sign; the artefact is published externally. Required for the customer success story and any external 'what we learned' content; required for any writeup that will be visible to prospects, sales targets, or the public.

### Who the approvers are

The customer's named approvers are agreed at the pilot's kickoff (mod-102). A typical approver map:

- **Customer SME lead.** Reviews for factual accuracy of SME quotes and SME-team references; signs the courtesy and approval levels.
- **Customer operations lead.** Reviews for operational accuracy; signs the courtesy and approval levels; co-signs with the SME lead on the customer success story.
- **Customer communications office.** Reviews for the customer's brand voice and public-messaging alignment; required at the legal + communications level.
- **Customer legal.** Reviews for contractual and regulatory alignment (confidentiality clauses, data-residency representations, regulatory disclosure constraints); required at the legal + communications level.
- **Customer privacy / DPO office.** Reviews for data-handling claims; required at the legal + communications level for any artefact that names data flows; required at the approval level for HIPAA / GDPR-touching content.

### Sign-off timing

The FDE builds the sign-off window into the cadence. For a monthly pattern writeup intended for external publication:

- Authored on the second week of the month.
- Sent to the SME lead for approval sign-off on day 2 of that week.
- Approval closed by day 7.
- Published on day 10.

For the quarterly customer success story:

- Draft authored at month 6 of the pilot.
- Sent to the SME + operations leads for approval on day 5.
- Approvals closed by day 20.
- Sent to communications + legal on day 20.
- Approvals closed by day 45.
- Marketing adapts; published on day 60.

A sign-off window missed is not an excuse for shortcut. The alternative to "the customer did not approve in time" is "publish redacted" or "hold to the next cycle," not "publish anyway."

## Running the three archetypes

**Archetype A — US healthcare payer.** All external artefacts named with "Archetype A" by default; the customer-specific name used only when the customer's communications office signs off at the legal + communications level. HIPAA-touching content gets DPO sign-off as a required additional approver. The customer's privacy office reviews every sentence that describes data flow or data classes; a sentence using "PHI" gets reviewed specifically for scope accuracy. SME quotes require signed consent from the SME per the customer's media-release policy.

**Archetype B — US federal agency.** Default posture: no external artefacts. The pattern writeup is vendor-internal; the pain-point taxonomy is vendor-internal with the agency's content abstracted ("a US federal agency operating under FedRAMP Moderate"); the 'what we learned' one-pager omits the agency entirely by default. A customer success story exists only if the agency's public-affairs office authorises it, which is rare; the typical outcome is a vendor-internal "anonymous federal pilot case study" shared with the vendor's product team. <!-- needs-research: confirm which OMB / GSA / agency policies currently govern public disclosure of FedRAMP-authorised vendor engagements -->

**Archetype C — EU retail bank.** Dual-track artefacts: the vendor-internal pattern writeup carries full detail; the external version replaces specific DORA major-incident references with the generic "DORA-scope ICT incident" and removes the bank's name unless the bank's communications office signs the legal + communications-level approval. GDPR Article 32 scope constrains any sentence that names data-flow specifics. EU AI Act Article 72 post-market monitoring record is itself an internal artefact the FDE contributes to monthly — the bank is the data controller, the FDE's quarterly contribution lands in the bank's Article 72 file, not in the vendor's public artefact.

## Anti-patterns

**Pattern writeup with no recommendation.** The writeup describes the shape in two pages and ends with "something to think about." The PM reads it, nods, moves on. The template's recommendation section is mandatory; "no change needed, this is just a pattern we see" is a valid recommendation — vagueness is not.

**One-pager with eight things.** The 'what we learned' one-pager has twelve bullets across three sections and spans two pages. The one-page constraint is doing work — forcing the FDE to pick the three most important. Twelve bullets is the FDE avoiding the picking.

**Customer success story before the honeymoon ends.** Marketing wants the story in month two; the FDE agrees; the quoted metrics evaporate in month four. Six-month discipline is non-negotiable except for pilots that close before six months with a clean completion signal.

**Taxonomy without the formula.** The quarterly pain-point taxonomy ranks classes without stating the ranking formula. Readers disagree with the order; the disagreement devolves into method rather than content. The ranking formula is named in the introduction.

**Verbatim quote without consent.** The FDE quotes the SME verbatim; the SME was speaking informally; the customer's HR team flags the quote on review. The capture-log discipline includes consent-at-capture; the artefact's quote carries the dated consent trail.

**Marketing writes the technical substrate.** Marketing drafts the customer success story from the pilot's dashboard; the metrics are correct but the two-or-three-decisions-that-mattered section is missing or wrong. The FDE owns the technical substrate; marketing adapts.

**Sign-off treated as a formality.** The FDE sends the writeup to the customer with a request for same-day signoff; the customer reviewer, under pressure, signs without reading; publication proceeds; the customer's legal team objects a week later. The sign-off window has a floor (not less than 48 hours for courtesy review; not less than two weeks for legal + communications) and a reviewer-approved exception path.

**Writeup without customer voice.** The pattern writeup is entirely the FDE's analysis. The verbatim quotes from the capture log are omitted. The writeup reads as an FDE opinion rather than a field pattern. Two customer quotes, at minimum, carry the writeup's voice.

**Missed cadence met with a double-up next month.** The FDE misses November's writeup; attempts a combined November + December writeup in December; the combined artefact is twice the length, half the signal, and inferior to either one alone would have been. If a cadence slips, the FDE *ships the minimum* for the next window on time and notes the slip; the double-up anti-pattern is tempting and consistently wrong.

## Summary

- Four artefacts: monthly pattern writeup (1–2 pages, PM + engineering), quarterly pain-point taxonomy (2–3 pages, product leadership), quarterly 'what we learned' one-pager (1 page, broad), per-outcome customer success story (1–3 pages technical substrate + marketing adaptation).
- Writing for product differs from writing for engineering: lead with the pattern, quantify where possible, carry the customer voice, name the recommendation explicitly.
- The monthly pattern writeup has a twelve-section template; the FDE authors it in roughly ninety minutes given a fully-populated Chapter 02 capture log.
- The quarterly pain-point taxonomy ranks pain-point classes by composite (signal count × average time cost), names the formula in its introduction, and feeds the Chapter 05 product-review meeting.
- The 'what we learned' one-pager is honest and specific: three things that worked, three that did not, one thing the field is telling us, what's next.
- The customer success story has an FDE-authored technical substrate and a marketing-adapted public version; the six-month discipline prevents honeymoon-inflated metrics from carrying the narrative.
- Three sign-off levels: courtesy review, customer approval, legal + communications. The sign-off window has floors; shortcuts are not an option.
- Archetype A requires DPO review on HIPAA-touching artefacts; Archetype B defaults to no external artefacts with the agency abstracted; Archetype C applies GDPR Article 32, dual-tracks the artefact, and contributes to the bank's EU AI Act Article 72 record.
- Nine anti-patterns: writeup without recommendation, one-pager with eight things, early success story, taxonomy without formula, quote without consent, marketing writing the technical substrate, formality sign-off, writeup without customer voice, double-up after a miss.
- Chapter 04 is next: the heavyweight platform-request packet and the lightweight feature request — the shapes a Chapter 03 writeup converts into once the two-customer threshold is met.
