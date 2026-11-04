# Chapter 06 — Scope Discipline and the Countersigned Decision Log

## Why this chapter exists

Scope discipline is the craft that separates a Forward Deployed AI Engineer from an implementation consultant. An implementation consultant bills for additional scope — new asks are revenue. An FDE does not. The FDE's success is measured along two axes: did the pilot land its mod-102 success criteria, and did the field patterns feed [mod-111](../mod-111-field-to-product-loop/)'s product-influence loop. Both axes are poisoned by scope creep. A pilot that said yes to six extra workflows produced neither a clean renewal nor a clean pattern writeup; the FDE is six weeks behind on both the exec brief refresh and the Chapter 03 packet, and the renewal conversation surprises everyone.

The role boundary is written in [mod-101 Chapter 04](../mod-101-fde-discipline-and-position/04-adjacent-customer-facing-roles.md). The FDE is not the customer's `applied-ai-engineer`; the FDE is not Professional Services; the FDE is not a staff-aug contractor. Customers do not read that boundary out of the page naturally — the sponsor is paying attention to their own problem, not to the vendor's role matrix. The FDE's job is to carry the boundary into every scope conversation and make the routing decision explicit, in writing, every time.

This chapter has five jobs. Name the **three responses** the FDE must become fluent in — no, later, and *your applied-AI-engineer owns this after hand-off* — with scripts. Specify the **four-step scope-check** that every new ask runs through. Write the **countersigned decision log** that keeps the scope conversation from decaying into verbal agreements no one remembers. Script the **two handshakes** — with the post-hand-off customer `applied-ai-engineer` and with vendor Professional Services. Trace a plausible **ten-asks-in-six-weeks** scenario through the log so the shape is concrete.

Scope integrity is an optimisation dimension in [Chapter 01](./01-consulting-craft-overlay.md)'s five-dimension model. Review-cadence discipline (Chapter 04) is the gate; scope integrity is what makes the gate meaningful. A weekly review that catches every new ask and never routes any of them is theatre. The decision log is the mechanism that converts the catch into a decision.

## The three fluent responses

An FDE under scope pressure has a reflex. The healthy reflex has three shapes, each with a script. The unhealthy reflex has one shape — *sure, we can add that* — and it is the single most common consulting-craft failure in the role.

### "No" — the ask conflicts with a pilot non-goal or a documented constraint

This is the hardest of the three. The sponsor is in the room. The ask is reasonable from their vantage. The FDE's job is to refuse it without making the sponsor feel refused. The script:

> "That's a good ask, and I want to make sure we handle it well. When we scoped the pilot in [date], we wrote a non-goal that specifically excluded [X] because [reason — regulatory boundary / authorisation scope / risk tier / out-of-domain]. If we take this on now we are changing the pilot's shape, which means re-running [the specific gate — the AI-RMF classification / the authorisation boundary / the model-risk sign-off]. I want to capture this in the decision log as out-of-scope with that rationale, and we can talk about whether a phase-two pilot would re-open the non-goal. Are you comfortable with that framing?"

Three features of the script. First, the FDE names the specific document — the mod-102 scoping artefact — rather than a vague "scope." Second, the FDE names the mechanism the ask would re-trigger — reclassification, re-authorisation, re-sign-off — so the no is grounded in a concrete cost, not a stylistic preference. Third, the FDE offers a path forward (phase-two consideration) without committing to it. The sponsor's assent closes the loop; the decision log row is written that afternoon.

### "Later" — the ask is in-scope eventually but not in this pilot phase

Easier than no, but with its own failure mode: *later* becomes *never* unless the log carries a review date. The script:

> "That's in-scope for where we want this to end up, but it's not in the current phase. The pilot's phase-one success criterion is [X]; phase two was shaped to carry [Y] and this ask fits there. I'm going to log it as deferred to phase two with a review date of [specific date — typically the phase-two kickoff]. If the ask becomes urgent before then, we can escalate it in the weekly review."

The FDE's commitment is: the log row exists; the review date is specific; the weekly review (Chapter 04) sweeps deferred items on the review date and either promotes them to in-scope, keeps them deferred with a new date, or closes them with rationale. A deferred row that has been deferred three times without a decision is a signal that the FDE should take back to their `ai-engineering-manager`.

### "The applied-AI-engineer in your org can own this after hand-off"

This is the response unique to the FDE role and the one that most protects both the pilot's shape and the customer's long-term independence. The ask in question is customer-specific — it will not generalise across pilots, it is not a product feature, it is the shape of this customer's workflow — and it belongs inside the customer's own engineering craft. The FDE's refusal is not a refusal of the work; it is a refusal to perform work that will produce an indispensable FDE instead of an independent customer team.

The script:

> "That's great work for your team's `applied-ai-engineer` to own after we hand off. It's specific to [customer workflow X] and it won't generalise to other pilots, which means it's not platform-team work and it's not something I should be building — if I build it, your team doesn't own it, and when I leave the knowledge leaves with me. What I can do in the remaining pilot time is make sure the runbook, the eval pipeline, and the gold set are ready so your team can take this on in the first month post-hand-off. Can we talk about which of your engineers will own it, and whether they're in the pair-programming cadence [Chapter 07] already?"

The response has three parts. First, the FDE names the receiver — *your team's `applied-ai-engineer`* — which is a direct cross-link to [mod-104](../mod-104-llm-app-in-customer-envs/) and to [mod-101 Chapter 04](../mod-101-fde-discipline-and-position/04-adjacent-customer-facing-roles.md) role boundaries. Second, the FDE justifies the route by the FDE's own exit — the indispensable-FDE anti-pattern from Chapter 01 is the explicit warning. Third, the FDE commits to the pre-conditions for a clean hand-off — the runbook, the eval pipeline, the gold set — which is the Chapter 07 hand-off material. The customer gets the work done; the FDE did not do it.

When the customer does not yet have an `applied-ai-engineer`, the FDE's language shifts to *the engineer you will hire or assign for this* — the FDE's refusal of the work is a forcing function for the customer to staff the role, which is a healthier outcome than the FDE silently doing the work the role was supposed to do.

### Handling sponsor pushback on a "no"

The sponsor's first response to a refusal is often a variant of *but you're already in there — can't you just do it?* The FDE's follow-up script, delivered calmly:

> "I could, and I understand why it looks that way. But if I do it, two things happen. First, your team doesn't learn the shape of it, which means a month after I hand off, the first time it needs a change, you'll be back to the vendor asking for help on something that should sit inside your own engineering craft. Second, I'll be behind on [the committed phase-one criterion — SC-04], which is what we agreed the pilot would be measured on. I'd rather hold the line on the committed scope, log this decision, and set up your engineer to own it after hand-off. Does that work?"

Two moves inside the follow-up. The FDE names the specific future-cost — *a month after I hand off* — so the sponsor can see the shape of the indispensable-FDE anti-pattern without the FDE having to name it jargon. And the FDE grounds the refusal in the sponsor's own committed criterion, which is the sponsor's own prior commitment, not the FDE's preference. Sponsors rarely override their own prior commitments; the follow-up almost always closes the loop.

If the sponsor still presses, the FDE's third move is to escalate to their own `ai-engineering-manager` rather than to concede in the room. *Let me talk to my manager about whether we can take this on and still land phase one on time; I'll come back to you by [date] with a decision.* The escalation is a legitimate use of the FDE's own reporting line; it also signals to the sponsor that the FDE is not the unilateral scope authority they might have assumed. See [Chapter 08](./08-role-honesty-in-customer-facing-comms.md) on the role-honesty posture under scope pressure.

## The four-step scope-check algorithm

Every new ask runs through four questions, in order. The algorithm is written down because the FDE under room pressure will skip a step; the written version makes the skip visible.

1. **Does the ask serve a documented success criterion from the mod-102 scoping artefact?** The criteria are listed by row; the FDE names the row the ask maps to. If no row matches, the ask is at best out-of-scope and at worst a non-goal violation. The FDE does not invent a new criterion in the room.
2. **Does the ask conflict with a documented non-goal?** Non-goals are the mod-102 artefact's negative space — the things the pilot explicitly agreed not to pursue, often because of a regulatory, authorisation, or risk-tier constraint. If the ask conflicts, the response is "no" per the script above.
3. **Does the FDE's remaining capacity and timeline carry the ask without displacing a committed criterion?** The weekly review's open-items register (Chapter 04) names what the FDE is already carrying. An ask that fits but cannot be delivered without displacement routes to "later" unless the sponsor is willing to trade a committed criterion out — which is itself a decision-log entry.
4. **Does the ask generalise to the product, or is it one-pilot bespoke?** This is the [mod-111](../mod-111-field-to-product-loop/) triage question, applied inside the scope conversation. Generalisable asks may be worth doing inside the pilot and feeding to the platform packet. Bespoke asks route to the customer's post-hand-off `applied-ai-engineer` — or, if the ask is SOW-shaped and the customer wants the vendor to perform it, to Professional Services.

The decision tree:

```text
new ask received
   │
   ├─ step 1: maps to a success-criterion row?
   │     ├─ no  → ask is out-of-scope (unless non-goal override; rare)
   │     └─ yes → continue
   │
   ├─ step 2: conflicts with a non-goal?
   │     ├─ yes → "no" (log row, countersign)
   │     └─ no  → continue
   │
   ├─ step 3: FDE capacity carries without displacement?
   │     ├─ no  → "later" (log row with review date)
   │     └─ yes → continue
   │
   └─ step 4: generalises to product, or one-pilot bespoke?
         ├─ generalises → in-scope; also feed mod-111 capture log
         ├─ bespoke, customer-specific → route to post-hand-off applied-ai-engineer
         └─ bespoke, SOW-shaped        → route to Professional Services (mod-101 Ch 04)
```

The FDE runs the algorithm in private — typically in the 24 hours between the weekly review where the ask surfaced and the following day's log update. Running it in the room is both rude (it looks like deliberation theatre) and brittle (the room is the worst place to catch a non-goal conflict). The scope-check is batch work, done weekly.

### The capture surface for inbound asks

Asks arrive through five channels. The FDE catches all five into a single inbound surface — typically a repo-tracked `asks.md` file, sometimes a labelled issue tracker queue — so that the weekly scope-check runs against one list, not five.

| Channel | Typical origin | Capture discipline |
|---|---|---|
| Weekly review minutes | sponsor, SME lead, ops director | Logged by the FDE in the minutes under *new asks since last week* |
| Direct Slack / Teams DM | any stakeholder | FDE replies *captured; will run through scope-check*; copies into `asks.md` |
| Email thread | sponsor, IT, privacy office | FDE replies with capture confirmation; copies into `asks.md` |
| Side conversation in a meeting room | SME, mid-level manager | FDE writes it down in-meeting; mentions it during close-out |
| Over-the-shoulder during pair work | customer engineer | FDE pauses the pair session for 30 seconds to capture |

An ask that arrived through any channel but did not land in `asks.md` is an ask the FDE will forget — and an uncaptured ask is the silent precursor to the yes-in-the-room failure. The capture discipline is five seconds per ask; the forgetting-cost is measured in weeks of scope drift. The FDE's `ai-engineering-manager` reviews `asks.md` monthly alongside the decision log — a healthy pilot shows every row in the decision log tracing back to an ask row in `asks.md`.

## The countersigned decision log

The decision log is a shared document — typically a repo-tracked markdown file, sometimes a shared spreadsheet, occasionally a dedicated page in the customer's Confluence — that records every scope decision over the pilot's life. It is the single most important artefact of the FDE's consulting-craft discipline *after* the pilot is over, because it is the artefact the renewal conversation references when someone asks "wait, why didn't we build X?"

### What the log records

One row per scope decision. Columns:

| Field | Shape | Notes |
|---|---|---|
| decision-id | `DEC-NNNN` | Monotone, zero-padded, never reused |
| date | `YYYY-MM-DD` | Date the decision was logged, not necessarily when the ask arrived |
| ask | one paragraph | In the requester's own language where possible, so the log is admissible |
| origin | role, not individual | *SME lead, operations director, model-risk reviewer* — durable across staff turnover |
| decision | enum | `in-scope` / `out-of-scope` / `deferred` / `routed-to-applied-ai-engineer` / `routed-to-professional-services` / `routed-to-platform-request` |
| rationale | one paragraph | Mapped to a specific success-criterion row or non-goal; cites the mod-102 artefact row ID |
| approvers | names and roles | FDE + customer sponsor + optionally vendor AE |
| review-date-if-deferred | `YYYY-MM-DD` or `—` | Only present on `deferred` rows; the Chapter 04 review sweeps these |
| countersignature | `acknowledged on YYYY-MM-DD by <role>` | The sponsor's assent, written as a text line or a digital signature |

The log lives in a location both sides can read and neither side can silently edit. A repo with branch protection is ideal; a shared drive with revision history is acceptable; a vendor-only document is not acceptable — if the customer cannot see the log, the log is not doing its job.

### The template

```markdown
# Pilot decision log — <customer> / <pilot name>

Scope decisions for the <customer> <pilot name> pilot, running
<start-date> through <end-date>, under the mod-102 scoping artefact
version <X>.

Every row is countersigned by the customer sponsor. Rows without a
countersignature are drafts, not decisions.

| ID | Date | Ask | Origin | Decision | Rationale | Approvers | Review | Countersignature |
|---|---|---|---|---|---|---|---|---|
| DEC-0001 | 2026-10-14 | Add Spanish-language support to the member inquiry assistant for the pilot cohort's bilingual members. | operations-director | deferred | Phase-one success criterion (SC-03) is scoped to English-language inquiries. Spanish-language support is in the phase-two shaping notes but requires an additional model-eval pass on Spanish gold-set entries (mod-107) and a privacy-office review for the additional language. | FDE (J. Reyes); sponsor (A. Patel, VP Member Experience); AE (M. Chen) | 2026-12-16 (phase-two kickoff) | acknowledged on 2026-10-18 by VP Member Experience |
| DEC-0002 | 2026-10-21 | Expand the pilot to include outbound call-centre voice transcription so that member call summaries are captured alongside chat. | sme-lead | out-of-scope | Pilot non-goal NG-02 from the mod-102 artefact explicitly excludes call-centre voice transcription for this pilot phase. Expansion would re-open the HIPAA data-handling review (mod-108) because voice recordings fall under a different retention policy than chat. | FDE; sponsor; privacy-office rep (R. Kumar) | — | acknowledged on 2026-10-24 by VP Member Experience |
| DEC-0003 | ... | ... | ... | ... | ... | ... | ... | ... |
```

The template is deliberately light. The log's job is not to be exhaustive — it is to be unambiguous. A reviewer reading row DEC-0002 in six months, with no memory of the conversation, can tell what the ask was, who asked, what was decided, why, who signed, and whether to revisit.

### Why the countersignature is load-bearing

An unsigned decision in a shared document is not a decision. It is a draft that both sides will later remember differently. The FDE's version of a scope conversation and the sponsor's version diverge across six weeks — not from bad faith, but because humans remember what they wanted to hear. The countersignature is the mechanism that collapses the ambiguity.

The parallel is [mod-108](../mod-108-enterprise-security-and-compliance/)'s audit-trail reasoning. A security event without a timestamped, authenticated record is a rumour; a security event with an audit-log entry is a fact. Scope decisions are the same. A decision acknowledged in writing on a specific date by a role with the authority to acknowledge it is a decision; anything less is a conversation that someone will later dispute.

The acknowledgement line can be as simple as a text row in the shared doc — *acknowledged on 2026-10-18 by VP Member Experience* — or as formal as a digital signature. What matters is that the sponsor (or a named delegate) wrote the acknowledgement themselves, within a short window of the decision (typically within one week), and that the acknowledgement is visible to anyone who reads the log. Rows without a countersignature are marked as drafts and reviewed again at the next weekly cadence until acknowledged.

### Who holds sponsor authority

The sponsor in the mod-102 scoping artefact is the person who holds budgetary and scope authority for the pilot. Typically a VP or a director-level equivalent. The FDE's decision log should name that role explicitly in its header — *this pilot's sponsor is VP Member Experience* — so that the approver column is unambiguous.

Sponsor-adjacent roles do not hold scope authority by default. The SME lead, the operations director, the privacy-office rep, the model-risk reviewer — each is a *stakeholder* whose input shapes the decision, but none is the decision-maker. A decision logged with "SME lead" in the approver column but no sponsor is a scope-governance failure (see the sponsor-approval-bypass anti-pattern).

Named delegation is the exception. The sponsor can delegate counter-signature authority to a specific person for a bounded window — "my deputy has sign-off for scope decisions through December while I'm on sabbatical." The delegation is recorded in the decision log's header as a dated note, with the sponsor's own countersignature on the delegation itself. Delegations that are not written down do not exist.

For federal and regulated-industry pilots, the authorising official (ATO holder, model-risk head, data-protection officer) is a *co-approver* rather than a delegate — some scope decisions require both the sponsor's countersignature *and* the authorising official's. The log's approver column names both. [Archetype B](#running-the-three-archetypes) below shows the shape.

The FDE's `ai-engineering-manager` reviews the log monthly — the counter-signature rate across decisions is one of the dimension-3 measurements from Chapter 01. A log with a 60 % counter-signature rate is signalling that the sponsor is not being held to the rhythm; the manager's job is to escalate.

### The manager-review ritual

The monthly manager review of the decision log is a 30-minute meeting between the FDE and their `ai-engineering-manager`. The agenda is fixed:

1. **Counter-signature rate.** The fraction of rows written since the last review that carry a countersignature. Target: ≥ 90 %. Below 75 % is a yellow flag; below 50 % is a red flag requiring escalation to the sponsor.
2. **Deferred-review aging.** Rows marked `deferred` whose review date has elapsed without a follow-up decision. The target is zero. Any aged row is picked up in the next weekly review with a forced decision: promote to in-scope, re-defer with a new date and rationale, or close with rationale.
3. **Routing distribution.** The count of rows in each `decision` enum value. A pilot with zero `out-of-scope` rows across two months is signalling that the FDE is saying yes too often; a pilot with every row `out-of-scope` is signalling that the scoping artefact was too tight or that the sponsor has disengaged.
4. **Named-receiver status for `routed-to-applied-ai-engineer` rows.** How many of the routed-to-applied-ai-engineer rows name a specific customer engineer. Target: 100 % by two weeks after routing. Rows without a named engineer are flagged for sponsor escalation.
5. **Override count.** Rows accepted under sponsor override (DEC-0008 in the sample above). More than one override per month is a signal; more than three across the pilot's life is a signal the scope artefact needs re-authoring.

The manager's output is one of three statements, in writing in the FDE's one-on-one notes: *log is healthy; carry on*; *log has a yellow flag — FDE to address by next review*; *log has a red flag — manager will co-present with FDE at the next sponsor review*. The co-present option is the escalation mechanism; the FDE is not alone in surfacing a decaying scope conversation with the sponsor.

## Pre-empting scope creep: the standing agenda item

Scope creep is pre-empted not by being clever in the moment but by running a cadence that catches asks weekly, batches them, and runs them through the scope-check offline. [Chapter 04](./04-weekly-pilot-review-cadence.md)'s weekly pilot review carries a standing agenda item — *new asks since last week* — that lists every ask captured in the FDE's inbound channels (Slack, meetings, email, back-channels) since the previous review. Each ask is read into the minutes with its origin role; the FDE announces the intended routing; the sponsor acknowledges or pushes back. The log rows are drafted that afternoon and countersigned by the following cadence.

The discipline that makes the standing item work is *never say yes in the room*. Every ask gets the same response: "I've captured that; I'll run it through the scope-check and we'll log a decision by [next review]." The response is identical for asks the FDE is almost certainly going to approve, for asks the FDE is almost certainly going to decline, and for asks that are genuinely borderline. Uniform treatment of every ask produces a log that is credible; preferential real-time agreement for the asks the FDE likes produces a log that reads as political.

The only exception to *never say yes in the room* is a safety or compliance ask that the FDE has standing authority to accept — for example, the SME requesting an additional guardrail because an edge case was identified. Even then the FDE's response is "yes, I'll implement that this week and log it as an in-scope expansion under safety criterion SC-07." The yes is logged, the rationale is explicit, the countersignature follows.

### The weekly scope-triage rhythm

The scope-check work fits into a 30-minute block the FDE schedules the day after the weekly review. The rhythm:

1. **Open `asks.md`.** Read every row captured since the last block. There are typically three to eight rows per week in an active pilot.
2. **Run each ask through the four steps.** The step 4 generalisability call is where the FDE pauses longest — that's the mod-111 triage question and has consequences beyond the immediate pilot.
3. **Draft a decision-log row for each.** Decision, rationale, approvers, review date if deferred. Mark the countersignature column as "pending as of [date]."
4. **Send the drafts to the sponsor.** One message, one asynchronous ack expected. The FDE's language is: *here are this week's scope decisions; please acknowledge each row by [end of week]; let me know if any need a conversation before you ack.*
5. **Follow up on prior-week drafts that are still unsigned.** Un-signed rows older than two weeks are flagged in red at the next weekly review.

The rhythm produces a steady throughput of countersigned decisions and a visible backlog of pending ones. The visible backlog is the manager-review ritual's input; the steady throughput is what keeps the pilot's scope from drifting.

## The handshake with the post-hand-off applied-AI-engineer

The customer's own `applied-ai-engineer` is the receiver for most bespoke customer-specific asks. The handshake begins before hand-off — ideally in the first weeks of the pair-programming cadence from [Chapter 07](./07-knowledge-transfer-to-customer-engineering.md) — so that by the time the scope ask surfaces, the customer engineer is already familiar with the pilot's internals. The FDE's language in front of the sponsor:

> "I'm not going to build [ask X] during the pilot phase because it's workflow-specific and your team will need to own it going forward. [Customer engineer name] is already pairing with me on the eval pipeline; I'll make sure the runbook covers the pattern needed for [ask X] and that the gold set has the shape they'll need. Realistically, that puts them in a position to deliver [ask X] in the first four to six weeks after hand-off."

The commitment is not the deliverable — it is the *enablement*. The FDE guarantees runbook coverage, eval pipeline parity, gold-set shape. The customer engineer is on the hook for the deliverable itself. If the customer does not yet have the engineer named, the decision-log row carries the dependency: *routed-to-applied-ai-engineer; customer to name engineer by [date]*.

The FDE's `ai-engineering-manager` reviews the pipeline of post-hand-off-routed asks in the manager's quarterly review with the customer sponsor. A pilot with twelve rows routed to applied-ai-engineer but no named engineer at the customer is a pilot at risk of post-hand-off decay — the manager's escalation to the sponsor is "we need a named receiver by [date], or we need to re-scope these decisions."

## The handshake with vendor Professional Services

Some bespoke asks are not for the customer's `applied-ai-engineer` — they are SOW-shaped, bespoke-deliverable work that the customer wants the vendor to perform, typically at additional cost, typically for a specific business outcome the customer will pay for. That work belongs to vendor Professional Services (PS), per [mod-101 Chapter 04](../mod-101-fde-discipline-and-position/04-adjacent-customer-facing-roles.md)'s role matrix. The FDE's job is to recognise the shape and route it.

The telltale signs of a PS-shaped ask:

- The ask would require a bespoke deliverable (a custom report, a one-off integration, a workflow-specific model fine-tune) that the customer would pay for as a project.
- The customer has budget allocated for implementation work beyond the pilot scope.
- The ask is time-bounded and has a clear acceptance criterion the customer can sign off on.
- The customer is willing to engage under a Statement of Work rather than under the pilot's operating model.

The script to the sponsor:

> "That's a well-defined deliverable and I think it's worth doing — but it's SOW-shaped, not pilot-shaped, and my role doesn't carry that kind of bespoke work. Let me introduce you to [AE name] and our Professional Services team; they'll scope the deliverable, give you a timeline and a budget, and we can coordinate so the PS delivery and the pilot's hand-off don't collide. In the meantime I'll log it as routed-to-professional-services in our decision log with a reference to the SOW ticket."

The FDE's AE is looped in because the AE owns the commercial conversation. The FDE's job is the routing, not the pricing. The decision-log row carries the SOW ticket ID (or a placeholder until PS creates one) so that the row is traceable from either side.

A PS route is not an escape hatch for scope creep — the PS team has their own backlog and will not pick up every routed ask. The FDE's judgement is still in play: route only asks that are genuinely SOW-shaped. Routing a bespoke-specific ask that belongs to the customer's `applied-ai-engineer` into PS is a failure mode; the PS team's time is more expensive than the FDE's and should be allocated to work that justifies it.

The three routing destinations — customer `applied-ai-engineer`, vendor Professional Services, vendor platform team — form a decision triangle. The FDE's judgement call on each bespoke ask places it at one vertex. The quick heuristic:

| Shape of the ask | Routes to |
|---|---|
| Customer-specific; will not generalise; belongs inside customer workflow knowledge | customer `applied-ai-engineer` after hand-off |
| Bespoke deliverable; customer will pay for it; SOW-shaped acceptance criterion | vendor Professional Services |
| Generalises across two or more pilots; shape fits the product surface | vendor platform team via mod-111 packet |

Mis-routing is expensive on both sides. Routing a customer-specific ask into PS consumes vendor-side budget and frustrates the PS team with work that should have stayed inside the customer. Routing a generalisable pattern to the customer's engineer buries the signal that could have shaped the product. The FDE's weekly scope-check is where the routing call is made deliberately.

## A ten-asks-in-six-weeks scenario

A plausible trace across a six-week window of a healthcare payer pilot (Archetype A). Rows abbreviated for readability; a real log carries full rationale and countersignatures.

| ID | Week | Ask (abbreviated) | Origin | Decision | Why |
|---|---|---|---|---|---|
| DEC-0001 | 1 | Add escalation-urgency tagging to the triage output | sme-lead | in-scope | Maps to SC-04 (triage quality); generalises; low effort |
| DEC-0002 | 1 | Spanish-language inquiry support | ops-director | deferred | SC-03 is English-only in phase one; phase-two shaping covers it |
| DEC-0003 | 2 | Expand to call-centre voice transcription | sme-lead | out-of-scope | Non-goal NG-02 excludes voice this phase |
| DEC-0004 | 2 | Custom weekly report format for the CEO's dashboard | sponsor | routed-to-professional-services | SOW-shaped bespoke deliverable; PS ticket PS-1142 |
| DEC-0005 | 3 | Add citation-section depth to policy answers | sme-lead | in-scope | Maps to SC-06 (citation quality); generalises — feed mod-111 feature request |
| DEC-0006 | 3 | Customer-specific rule for prior-auth edge case | clinical-reviewer | routed-to-applied-ai-engineer | Payer-specific business rule; customer engineer K. Liu to own post-hand-off |
| DEC-0007 | 4 | Sentiment analysis on member complaints | ops-director | deferred | Outside SC-01 through SC-07; phase-two candidate; review 2026-12-20 |
| DEC-0008 | 4 | Dashboard filter by member tenure | sponsor | in-scope (override) | Does not map directly to a criterion; sponsor cites executive-reporting need; FDE accepts under sponsor override, log row annotates the override |
| DEC-0009 | 5 | Weekly model retraining on customer data | sme-lead | out-of-scope | Non-goal NG-05 excludes continuous retraining this phase; would re-trigger model-risk sign-off |
| DEC-0010 | 6 | Privacy-friendly logging improvement surfaced by SOC 2 reviewer | it-security-lead | in-scope; also fed to mod-111 platform packet | Generalises across pilots; the FDE authors the Chapter 03 packet alongside the pilot-local fix |

Ten asks; two in-scope; one in-scope with a sponsor override; one routed to PS; one routed to the customer `applied-ai-engineer`; one out-of-scope hard non-goal; one out-of-scope compliance; two deferred with review dates; one dual-tracked as in-scope-and-platform-packet. Six weeks, roughly 90 minutes of weekly scope-check work (one weekly batch of ~15 min), ten countersigned rows. The pilot stayed on its phase-one shape; the field patterns fed [mod-111](../mod-111-field-to-product-loop/); the hand-off has a clear backlog of routed-to-applied-ai-engineer items.

Row DEC-0008 is the instructive one. The sponsor over-rode the FDE's scope-check — the ask did not map to a criterion but the sponsor accepted responsibility for the exception. The log row *names* the override explicitly; six months from now, if anyone asks "how did this slip into scope," the row's rationale carries the answer. Overrides are allowed; silent overrides are not.

Row DEC-0010 is the second instructive one: an ask that lands as in-scope for the pilot *and* as a candidate for the mod-111 platform-request packet. The FDE's rationale on the row names both tracks: the pilot-local fix, and the platform packet authored in parallel. The decision log and the mod-111 capture log cross-reference each other — the row's rationale carries the capture-log entry ID, and the capture-log entry carries the decision-log row ID. Cross-referencing is what keeps the two systems from diverging over the pilot's life.

### Reading the log at renewal

Six weeks before the pilot's end date, the FDE and the sponsor hold a renewal conversation. The decision log is the first artefact read. The renewal conversation runs along four axes:

1. **Scope-delta.** How much did the pilot's scope change between the mod-102 scoping artefact and the current state? A pilot with a scope-delta above 20 % (by criterion count) is signalling either that the scoping artefact was wrong or that discipline slipped. Either way, the renewal shape is affected.
2. **Deferred-row backlog.** What's still deferred, and what would the phase-two shape look like if the backlog converted to in-scope? The deferred list is the raw input to phase-two scoping.
3. **Routed-to-applied-ai-engineer backlog.** What work is the customer's engineer going to pick up at hand-off? This is the input to the Chapter 07 knowledge-transfer plan.
4. **Routed-to-PS backlog.** What work is the Professional Services team going to pick up? This is the input to the AE's renewal-commercial conversation.

A well-maintained log makes the renewal conversation a 60-minute meeting; a poorly maintained log makes it a weeks-long reconstruction exercise. The decision log's value compounds across the full life of the engagement — the discipline pays its largest dividend at exit.

## Running the three archetypes

**Archetype A — US healthcare payer.** The pilot's phase-one non-goals (NG-01 through NG-05) exclude call-centre voice transcription, continuous retraining, and expansion into clinical-decision support. A representative scope-expansion ask is the SME lead proposing member-call summarisation beyond the chat-based inquiry flow — summarising inbound member calls into the agent's context. The scope-check fails step 2: non-goal NG-02 explicitly excludes call-centre voice transcription for this phase, because voice data falls under HIPAA retention and consent regimes different from chat. The FDE's response is "no" with the script above; the log row cites NG-02 and references the mod-108 privacy-office note that established the non-goal. The sponsor countersigns within four days. A phase-two consideration is logged separately so the ask is not lost. <!-- needs-research: confirm whether call-recording retention under 45 CFR 164.530 matches the chat-retention floor used in the payer's pilot design -->

The payer's decision log carries a privacy-office co-approver column on any row whose decision touches PHI handling. The privacy-office rep is not the sponsor — the VP Member Experience is — but any HIPAA-touching decision requires both signatures. The log's header names this explicitly: *rows whose ask touches PHI handling require both VP Member Experience and Privacy Office countersignatures*. The co-approver pattern is a direct consequence of the mod-108 audit-trail reasoning.

**Archetype B — US federal agency.** The pilot is authorised for policy-analyst drafting under a specific FedRAMP Moderate ATO and a NIST AI RMF 1.0 classification for assistive content generation. A representative ask is the mission-owner asking to expand the assistant from drafting policy to providing decision support for policy adoption — effectively moving from an assistive-generation classification to a decision-recommendation classification under AI RMF. The scope-check fails step 2: the pilot's authorisation boundary does not cover decision-support use, and expanding would require an AI-RMF re-classification plus an ATO amendment. The FDE's response is "no" with explicit reference to the authorisation boundary; the log row names the ATO document and the AI-RMF category; the ISSO is a named approver on the row. The log is reviewed inside the enclave; the countersignature is the agency's mission-owner acknowledgement. <!-- needs-research: confirm which NIST AI RMF function and category covers decision-support use vs. assistive drafting in the current RMF guidance -->

The agency log lives entirely inside the enclave — the FDE cannot export rows to vendor-internal systems. The vendor's `ai-engineering-manager` reviews the log by accessing the enclave under the FDE's screen-share, not by reading an exported copy. The counter-signature discipline is identical; only the storage shape changes. Where the payer's log has two approver columns (sponsor + privacy office), the agency's has three (sponsor + ISSO + mission-owner) and some rows require all three.

**Archetype C — EU retail bank.** The pilot is scoped for case triage inside the operations team, under an EU AI Act limited-risk classification (internal-use decision-support for operations staff, not customer-facing). A representative ask is the Head of Customer Operations proposing to expand the assistant into a customer-facing chatbot handling direct member interactions. The scope-check fails step 2: the pilot's EU AI Act classification does not cover customer-facing use, and expansion would move the system into a higher-risk tier with additional transparency, post-market-monitoring, and human-oversight obligations under Articles 13, 14, and 72. The FDE's response is "no" with reference to the Act's risk-tier distinction; the log row cites the specific articles and the bank's model-risk function's sign-off from pilot scoping; the model-risk function head is a named approver and co-countersigner. <!-- needs-research: confirm the specific EU AI Act risk-tier distinction between internal operations-support and customer-facing deployment in the current Act text -->

The bank's decision log is cross-referenced into the DORA ICT-third-party register — scope decisions that affect the pilot's inclusion in the register (expansion, contraction, re-classification) carry a DORA-register-impact annotation. The DORA liaison at the bank is a notified party on any such row, though not an approver. The model-risk function head *is* an approver on rows that would change the pilot's model-risk classification. This is the same co-approver pattern as the payer's privacy office and the agency's ISSO — a regulated environment's authorising officials layered into the countersignature discipline.

## Anti-patterns

**Yes in the room.** The sponsor asks during the weekly review; the FDE says *sure*; two more asks follow in the same meeting; by Friday the FDE has committed to three pieces of work that no one has checked against the pilot's success criteria. The healthy response is uniform: *I've captured that; I'll run it through the scope-check and we'll have a logged decision by the next review.* Every ask, every time.

**The uncaptured decision.** A scope conversation happens on a one-off call; the FDE and the sponsor reach an understanding; no row is written. Six weeks later, both parties remember the conversation differently and neither has a record. The discipline is: if it was discussed, it was logged; if it was logged and not countersigned, it was a draft.

**The undated log.** The decision log exists but rows have no dates — just "October" or "last month." When someone needs to reconstruct the sequence of decisions six months later, the log is useless. Every row carries a date in `YYYY-MM-DD` form; every countersignature carries a date; review dates on deferred items are absolute.

**The bespoke rabbit-hole.** The FDE routes a customer-specific ask to themselves rather than to the customer's `applied-ai-engineer` or to Professional Services. The ask is interesting, the FDE is capable, the customer is grateful — and three months later the pilot is behind schedule, the customer has no knowledge of how the feature works, and the hand-off's runbook does not cover it. The FDE's craft is in refusing bespoke work; the ability is not the point.

**The back-channel ask.** A customer SME sends a one-liner in the FDE's Slack: *hey, can you just add [X] while you're in there?* The FDE's reflex is to say yes because it's a small ask from a stakeholder they're trying to build trust with. The healthy response is: *good ask — I'll pull it into the standing agenda for this week's review so the sponsor sees it and we log the decision.* Back-channel asks are the most common source of silent scope drift; the discipline is to surface them into the review rather than resolve them privately.

**The *just one more thing* pattern.** The sponsor consistently ends the weekly review with "oh, and one more thing" — an ask tacked on in the last two minutes when the FDE is distracted and about to leave the room. The asks accumulate; none land in the log. The counter is a standing agenda item *before* the close-out that explicitly solicits new asks, and a verbal checkpoint: *is there anything else you'd like me to run through the scope-check this week?* The ask is captured in the structured moment, not the last two minutes.

**The log with no signatures.** A log exists; rows are written; no sponsor has ever countersigned anything. The rows are drafts. The FDE's `ai-engineering-manager` reads the log in the monthly review and the first question is *why is the counter-signature column empty?* A log without countersignatures is a notebook, not a decision record.

**The sponsor-approval bypass.** The FDE logs a decision citing the SME lead as the approver because the sponsor was unavailable. The SME is not an approver of scope — the sponsor is. The log's approver column must name the sponsor or a sponsor-named delegate; routing approval around an unavailable sponsor is a scope-governance failure. If the sponsor is persistently unavailable, that is itself a signal the `ai-engineering-manager` should escalate. See [Chapter 05](./05-customer-side-resistance.md) for the sponsor-disengagement pattern.

## Summary

- Scope discipline is the defining craft that separates the FDE from an implementation consultant. The FDE's success is pilot outcome plus product influence ([mod-111](../mod-111-field-to-product-loop/)); both are poisoned by scope creep.
- Three fluent responses: *no* (non-goal or constraint conflict), *later* (in-scope eventually, deferred with review date), and *your team's `applied-ai-engineer` owns this after hand-off* (customer-specific work that belongs inside the customer's own craft).
- A fourth response exists for SOW-shaped bespoke deliverables: *routed to Professional Services*, with the AE looped in for the commercial conversation.
- The four-step scope-check runs every ask through: success-criterion match, non-goal conflict, FDE capacity, and generalisability. The algorithm runs in private batch work, not in the room.
- The decision log is a shared, version-controlled, countersigned record of every scope decision. Columns: ID, date, ask, origin (role not individual), decision, rationale, approvers, review date for deferred rows, countersignature.
- The countersignature is load-bearing — the parallel to [mod-108](../mod-108-enterprise-security-and-compliance/)'s audit trail. A decision without a dated acknowledgement by the sponsor (or named delegate) is a draft, not a decision.
- Pre-empt scope creep with a standing *new asks since last week* agenda item in the Chapter 04 weekly review, and the discipline of *never say yes in the room*.
- Archetype A's representative out-of-scope ask is call-centre voice transcription (non-goal NG-02; HIPAA retention regime). Archetype B's is expansion from drafting to decision support (AI-RMF reclassification; ATO boundary). Archetype C's is expansion to customer-facing chatbot (EU AI Act risk-tier change).
- Eight anti-patterns to catch early: yes-in-the-room, the uncaptured decision, the undated log, the bespoke rabbit-hole, the back-channel ask, the *just one more thing* pattern, the log with no signatures, the sponsor-approval bypass.
- [Chapter 07](./07-knowledge-transfer-to-customer-engineering.md) is next: the knowledge transfer to the customer's engineering team — the pair-programming, documentation, runbook, and on-call shadowing craft that makes the FDE's exit the measure of the engagement's success.
