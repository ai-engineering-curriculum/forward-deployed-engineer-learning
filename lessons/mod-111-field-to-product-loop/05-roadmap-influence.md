# Chapter 05 — Influencing the Roadmap Without Owning It

## Why this chapter exists

Chapter 04's packets and feature requests land in writing. Writing alone is rarely enough to move a roadmap. The quarterly **product-review meeting** is the forum where a packet that has been sitting in `ai-platform-engineer`'s queue for six weeks gets prioritised, where a feature request's rank shifts because the FDE walks in with a one-page writeup and two customer quotes, where the PM's roadmap for next quarter gets a specific line item named by a specific FDE.

The FDE does not own the roadmap. The PM owns the roadmap; `ai-engineering-manager` owns the FDE team's voice at the meeting; the vendor's product leadership owns the cross-product prioritisation. The FDE's role in the meeting is **influence** — the specific skill of shifting a decision the FDE does not make, by the strength of the evidence they bring and the quality of their proposal.

This chapter has four jobs. Describe the **pre-work** the FDE does before the meeting — reading the PM's current priorities, curating the evidence, co-authoring with `ai-engineering-manager`. Specify the **meeting mechanics** — agenda, roles, time budget, what good looks like. Specify the **follow-through** — how a verbal commitment in the meeting becomes a written commitment, how a "no" is received, how a "yes, but later" is tracked. Catalogue the anti-patterns of a roadmap demand (as opposed to a roadmap proposal).

The FDE's roadmap influence rests on the credibility of the field pattern evidence. Credibility is accumulated across packets, writeups, and meeting behaviour; it is spent one packet at a time. A chapter on influence is also a chapter on credibility management.

## What the meeting is and is not

The product-review meeting is a **standing** 60-to-90-minute meeting, quarterly, attended by:

- The PM for the product surface the FDE is working on.
- `ai-engineering-manager` on behalf of the FDE team.
- The FDE presenting the quarter's patterns.
- The PM's peers on adjacent product surfaces (optional; often attending to spot cross-surface implications).
- `senior-agentic-ai-engineer` and `ai-platform-engineer` as named recipients of any platform-request packets being discussed.
- Vendor product leadership (optional; attends the quarterly where strategic prioritisation is being set).

The meeting **is**:

- A standing forum where field patterns reach product prioritisation in a bounded time.
- A review of the quarter's pain-point taxonomy (Chapter 03) plus any platform-request packets (Chapter 04) that need decision.
- A conversation the PM owns the agenda for; the FDE is a named speaker, not the host.

The meeting **is not**:

- An escalation forum. An escalation goes through `ai-engineering-manager` directly, not through the quarterly review. A packet that cannot wait for the quarterly review is an escalation.
- An invention meeting. The FDE does not surface new feature ideas in the meeting that have not been written up. The meeting discusses artefacts the attendees have already read.
- A one-way presentation. The PM may re-prioritise some items and defer others; the FDE adapts in real time or withdraws an item for revision.

## The pre-work

The single most important hour the FDE spends in a quarter is the hour of pre-work for the product-review meeting. A well-prepared FDE converts the hour into a meeting that moves the roadmap; an unprepared FDE spends the meeting establishing context the attendees could have had in advance.

### Pre-work step 1 — Read the PM's current priorities

Two business weeks before the meeting, the PM publishes (or confirms) the current quarter's roadmap and the candidate items for the next quarter. The FDE reads both. The question the FDE answers is: *"which column of the PM's existing frame do my patterns fit into, and which do not?"* A pattern that aligns with the PM's existing priority is easier to get prioritised; a pattern that lives in a frame the PM has not yet written is harder — and requires the FDE to help the PM see the frame.

Specific reads:

- The quarter's shipped and in-flight line items. These are the PM's baseline — the FDE's proposals are compared to them.
- The PM's **public tradeoffs**. Items the PM has deferred, items the PM has sized as "big," items the PM has flagged as cross-team. The FDE's proposals are read against the same lens.
- The PM's **stated strategic lens**. If the PM has written "this quarter we are prioritising reliability of the shipped surface over new capability," the FDE reads every pattern writeup against that lens and knows which ones will land.

### Pre-work step 2 — Curate the quarter's evidence

The quarter's capture log has 100+ entries. The quarter's pattern writeups are 3–4 pages each. The quarter's pain-point taxonomy is the rollup. The meeting is 60–90 minutes. The curation is the FDE's picking of what to bring.

The default curation: *three patterns, three recommendations.*

- One high-magnitude pattern that aligns with the PM's existing frame — a pattern that is likely to be accepted and sets the meeting's rhythm.
- One pattern that lives in a frame the PM has not yet written — the hardest sell, where the FDE's job is to help the PM see the frame. The packet or writeup carries the weight.
- One pattern that is cross-surface — touches more than the PM's area; the adjacent PM is in the room because of this one.

Beyond three, the meeting runs out of time. The FDE's instinct to bring six is the first trap; the discipline of picking three is the hard one.

### Pre-work step 3 — Co-author with `ai-engineering-manager`

A standing 30-minute prep meeting with `ai-engineering-manager` the week before. Two outputs:

- The three-pattern slate is confirmed. The manager has visibility into the FDE team's other patterns; the manager may suggest a different selection based on cross-pilot visibility the single FDE lacks.
- The manager and the FDE agree on their roles at the meeting. Sometimes the manager speaks; sometimes the manager stays quiet and lets the FDE carry the conversation; sometimes the manager opens with a two-minute framing. The agreement is explicit.

The manager's job in the meeting is *not* to carry the FDE. The manager's job is to confirm the FDE team's position, to speak to the FDE team's capacity constraints, and to close any discussion that crosses into FDE-team process (the FDE should not have to defend their team's headcount in the meeting; that is the manager's turf).

### Pre-work step 4 — Send the packet 48 hours ahead

Every attendee receives:

- The quarterly pain-point taxonomy (Chapter 03).
- The three pattern writeups, printed and annotated.
- Any platform-request packets being discussed (Chapter 04).
- A one-page meeting agenda with the three asks named explicitly.

Attendees are expected to have read before arriving. The FDE's meeting opens with "I assume you've read the materials — I'll spend five minutes on each pattern, two minutes on each ask." If the PM has not read, the FDE adapts (shorter asks, longer context) but notes the adaptation in the meeting's follow-up.

## The meeting mechanics

A 60-to-90-minute meeting runs well against a standing agenda.

### Standing agenda (90-minute version)

1. **Framing — 5 minutes.** `ai-engineering-manager` opens: "Here are the three patterns the FDE team is bringing this quarter; here is the ask on each; here is where we would appreciate PM discussion vs sign-off." The FDE is silent; the manager carries the framing.
2. **Pattern 1 — the aligned pattern — 15 minutes.** The FDE presents the pattern, the evidence, the recommendation. The PM responds; discussion; decision noted by `ai-engineering-manager`.
3. **Pattern 2 — the frame-shifting pattern — 25 minutes.** Longer because the frame has to be established. The FDE presents the field evidence in more depth; the two customer quotes are read verbatim; the recommendation is left open for discussion. The PM may propose a different framing; the FDE engages.
4. **Pattern 3 — the cross-surface pattern — 15 minutes.** Adjacent PMs weigh in. The decision may be "we need a joint review with X team"; the FDE accepts and moves on.
5. **Platform-request packet discussion — 15 minutes.** If any packets are blocked or disputed, the relevant level-40 peer (`senior-agentic-ai-engineer` or `ai-platform-engineer`) and the PM jointly discuss. The FDE answers questions.
6. **Open floor — 10 minutes.** The PM's items — things *the PM* wants the FDE to hear. Often forgotten; always valuable. The FDE listens and notes.
7. **Decisions and action items — 5 minutes.** `ai-engineering-manager` reads back the decisions; the PM confirms; the FDE commits to the follow-up timeline.

### The FDE's conversational discipline

The meeting's influence is produced by the FDE's conversational behaviour, not the slides.

- **Propose, do not demand.** The FDE's sentences end with a question or a proposal, not a conclusion. "We propose the canary-gate preset change in the November packet; what would make that an easier yes?" is a question the PM can engage. "We need the canary-gate preset change" is a demand the PM has to accept or reject.
- **Carry the customer voice, not your own frustration.** The FDE's frustration is real; the meeting is not the forum for it. The customer's SME lead's verbatim quote is the voice that carries weight; the FDE's sigh does not.
- **Quantify carefully.** When the PM asks "how big is this?" the FDE has the number and the source. "In three of the twelve pilots this quarter, costing roughly one engineer-day per pilot per month in maintenance overhead" is actionable; "it's hitting a lot of us" is not.
- **Accept 'no' once; defer the re-ask.** The PM says "not this quarter." The FDE's first move is to understand the reason — scope, timing, strategic lens — and acknowledge. The re-ask lives in the next quarter's meeting with new evidence, not in the same meeting with more pressure.
- **Help the PM look good to their boss.** The PM has their own stakeholders. The FDE who frames the proposal as "this would help you ship against the Q1 reliability theme you've committed to" is an FDE the PM calls proactively next time; the FDE who frames every proposal as "customers are complaining" is an FDE the PM prepares against.

### The 'yes, but later' outcome

Most PM responses to a field-driven proposal are not "yes" and not "no." They are "yes, but later" or "yes, if you can do X." The FDE's job in the meeting is to turn the ambiguous response into a specific written commitment.

- "Yes, but later" → "which quarter? what is the condition that unblocks it?" The specific condition goes into the meeting minutes; the FDE tracks it; the next quarter's pre-work checks the condition.
- "Yes, if you can do X" → "what specifically does X look like? by when?" X is often additional evidence (a third customer instance), a different framing (not platform, product), or a dependency elsewhere. The FDE commits to X or renegotiates X before the meeting ends.

A commitment that is not written in the minutes is a commitment that evaporates. `ai-engineering-manager` owns the minutes; the FDE reads them the same day; the PM confirms within 48 hours.

## Follow-through

The meeting produces three outputs the FDE must act on within two weeks.

### Output 1 — Minutes with named decisions

Published within 48 hours by `ai-engineering-manager`. The format mirrors the meeting agenda; each pattern and packet has a row with the decision, the owner, the next-action, and the review date. The FDE's job is to confirm the minutes are accurate within one business day of receipt.

### Output 2 — The commitment register

The FDE team maintains a per-quarter commitment register — a table of every verbal commitment the PM made in the quarterly review and the status of each. Typical rows:

| Commitment | Owner | Due | Status | Review-date |
|---|---|---|---|---|
| Canary-gate preset change prioritised for Q1 2027 | vendor PM + senior-agentic-ai-engineer | 2027-03-31 | Scheduled | Q1 product-review |
| Section-level citations feature in next release | vendor PM | 2027-02-14 | In design | Q1 product-review |
| Cross-surface policy-citation review with Adjacent PM | both PMs | 2027-01-15 | Scheduled | Standing |

The commitment register is a Chapter 06 knowledge-base artefact. The next quarter's pre-work pulls from it: any commitment past its review date without a status update is the first item on the next meeting's agenda.

### Output 3 — Field-side follow-up

The patterns discussed in the meeting become monthly writeups in the next quarter — the FDE confirms the writeup content or revises it in light of the discussion. The platform-request packets accepted in the meeting move to the "accepted, in design" state; the FDE joins the design reviews (Chapter 04's follow-through).

A pattern rejected in the meeting is not discarded — it is captured as `platform_gap_candidate` or re-triaged based on the PM's feedback. A pattern that the PM reframed as a bug ("this is a pilot-local workaround, not a platform ask") is honestly re-triaged; the capture log's `triage_confirmed` is updated with the meeting as the rationale.

## Credibility management

Credibility in this context is the PM's and the level-40 peer's shared assessment of the FDE's judgement — the degree to which, when the FDE says "this is a pattern," the recipient believes them without re-doing the analysis.

Credibility compounds in both directions. A well-shaped packet accepted on first review is a credibility deposit; a half-shaped packet that gets rewritten is a withdrawal. A meeting where the FDE's three patterns all land is a deposit; a meeting where two of three get pushed back for "insufficient evidence" is a withdrawal.

Four disciplines keep the balance positive:

1. **Bring the strongest three, not the full seven.** A quarterly review that lands all three proposals is worth more than one that lands five of seven. The three-pattern discipline from the pre-work is the credibility-protecting choice.
2. **Over-deliver on the follow-up.** A pattern accepted in the meeting as a platform-request packet should land as a packet within one week. A commitment to send a second-customer instance by Friday lands on Thursday. Late follow-up is a credibility withdrawal.
3. **Say 'I don't know' when you don't.** The PM asks a question the FDE does not have the number for. "I don't have that measurement yet; I'll come back with it by end of week" is a credibility deposit; a guess is a withdrawal.
4. **Withdraw proposals that have not held up.** A pattern the FDE brought last quarter that turned out to be a one-customer shape is an honest "we withdraw this request; the two-customer threshold did not materialise" at the next meeting. Withdrawal is a signal of calibration, not of weakness.

The FDE whose credibility is high enough that a single packet carries a quarter's worth of priority-shifting is the FDE whose roadmap influence is operational. The FDE whose credibility has been spent over a year of poorly-shaped packets will take two years to recover it.

## Running the three archetypes

**Archetype A — US healthcare payer.** Pre-work reads the PM's priorities through the lens of healthcare-vertical pilot concentration (several similar pilots are in flight; the PM is likely to prioritise platform work that helps all of them). HIPAA-touching patterns require the pattern writeups to carry de-identified evidence; the SME quotes in the meeting are read verbatim but the surrounding context is abstracted. The customer's communications office has not pre-approved the SME quote for the meeting itself, but has approved the courtesy level; the meeting attendees sign a standing NDA with the vendor's product function. Co-authoring with `ai-engineering-manager` focuses on cross-healthcare-pilot patterns.

**Archetype B — US federal agency.** The agency pilot's patterns are abstracted before entering the meeting at all; the FDE presents "a US federal pilot under FedRAMP Moderate" patterns. `senior-agentic-ai-engineer` may request detail that the FDE cannot share in the meeting room; the response is a follow-up in a cleared channel, named in the meeting minutes. Platform-request packets originating from the agency pilot that use the exception path (one-customer because the agency is the only one of its kind) require explicit `ai-engineering-manager` sign-off before the meeting; the manager's role in the meeting is to reaffirm the exception's validity.

**Archetype C — EU retail bank.** Pre-work reads the PM's priorities through the EU AI Act lens — a pattern that would improve the bank's Article 72 posture is more valuable to the PM than the same pattern without the Article 72 framing, because the PM's broader EU customer base cares. GDPR-constrained evidence (specific PII flows) is pseudonymised in the meeting materials; DORA-adjacent patterns (ICT-incident classification) require the FDE to anticipate questions on whether the pattern affects the bank's DORA reporting. Model-risk function sign-off on the pattern writeups is noted in the packet; the PM may want to invite the vendor's own compliance function to the meeting when a DORA-adjacent pattern is being discussed.

## Anti-patterns

**The demand monologue.** The FDE presents each pattern as "this is what we need; when will you build it?" The PM responds defensively; the meeting becomes adversarial; the FDE's credibility takes a hit regardless of the pattern's merit. Proposals, not demands. The conversational-discipline bullets are not optional.

**Pre-work skipped.** The FDE walks in without having read the PM's current priorities; proposes patterns that misalign with the PM's frame; wastes the first fifteen minutes establishing what the PM already knew. The hour of pre-work is the single highest-leverage hour of the quarter.

**Seven patterns, not three.** The FDE brings seven patterns because "they are all important." The meeting runs over; the last four get five minutes between them; none of them land. The curation is uncomfortable because every pattern feels important; the discipline is the compression into three.

**Customer-voice replaced by FDE frustration.** The FDE substitutes personal frustration for the customer quote. "I'm sick of writing this override every release" lands as the FDE's complaint; "the customer's operations lead said 'we wish the canary gate held longer so our eval would catch it first'" lands as field evidence. The customer voice is the one the PM responds to.

**'Yes, but later' left verbal.** The PM says "yes, but later" and the FDE nods. The minutes do not capture a specific condition or a specific date. The commitment evaporates. Every verbal commitment becomes a written commitment or the meeting is not yet over.

**Escalation through the review.** The FDE uses the meeting to escalate a packet that has been stuck in `senior-agentic-ai-engineer`'s queue for ten weeks. The level-40 peer is in the room; the escalation is public and awkward; the relationship takes months to recover. Escalations go through `ai-engineering-manager` privately, not through the quarterly review.

**Manager's turf invaded.** The FDE finds themselves defending the FDE team's capacity to take on a packet; the manager is silent; the PM reads the FDE as not confident. Capacity and headcount are the manager's turf; the FDE knows the boundary and hands off.

**Follow-up missed.** The meeting commits the FDE to send a third-customer instance by Friday. Friday comes; the FDE has not sent it. Monday the PM's assistant chases; the FDE is a day late. Credibility withdrawal; the next meeting carries the baggage.

**Credibility spent, not deposited.** The FDE brings three patterns, all one-customer, all poorly-evidenced. The meeting is awkward; the PM prioritises none. The next quarter's three-pattern slate carries the previous quarter's shadow. The credibility curve is managed; the FDE picks fights worth winning.

**Reframing missed.** The PM reframes a pattern as "this is a pilot-local workaround, not a platform ask." The FDE argues; the PM becomes entrenched. The honest move is: "let me re-triage and bring it back next quarter if the two-customer threshold is met by then." Reframing is a reclassification signal; the capture log's triage is updated.

## Summary

- The product-review meeting is a quarterly 60-to-90-minute standing forum where field patterns reach product prioritisation. The FDE influences; does not own; the PM owns the roadmap and `ai-engineering-manager` owns the FDE team's voice.
- Pre-work has four steps: read the PM's current priorities, curate the quarter's evidence down to three patterns, co-author with the manager, send the packet 48 hours ahead.
- The standing 90-minute agenda is: framing, three patterns (aligned, frame-shifting, cross-surface), packet discussion, open floor, decisions. The FDE's conversational discipline — propose not demand, customer voice not frustration, quantify, accept 'no' once — produces the influence.
- The 'yes, but later' outcome is the most common and needs the hardest follow-up: specific condition, specific date, in the minutes within 48 hours.
- Follow-through in three outputs: minutes with named decisions, a per-quarter commitment register, field-side updates to pattern writeups and platform-request packets.
- Credibility compounds in both directions. Four disciplines keep it positive: bring the strongest three, over-deliver on follow-up, say 'I don't know', withdraw proposals that have not held up.
- Archetype A uses de-identified evidence and cross-healthcare-pilot patterns; Archetype B abstracts the agency and routes detail to cleared channels; Archetype C leans on the EU AI Act Article 72 frame and anticipates model-risk / DORA questions.
- Ten anti-patterns: demand monologue, skipped pre-work, seven patterns, frustration-as-evidence, verbal commitments left verbal, escalation through the review, manager's turf invaded, missed follow-up, credibility spent not deposited, reframing missed.
- Chapter 06 is next: the FDE team's internal knowledge base — pattern catalogue, failure-mode library, deployment playbook, security-review-answer library — the compounding artefact that scales the FDE team.
