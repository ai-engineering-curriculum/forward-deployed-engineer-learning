# Chapter 08 — Role Honesty in Customer-Facing Communications

## Why this chapter exists

Every chapter in this module has quietly assumed the FDE knows their role boundary. [Chapter 02](./02-engineer-executive-demos.md) assumed the exec-demo narrator is not the Field CTO. [Chapter 04](./04-weekly-pilot-review-cadence.md) assumed the weekly-review chair is not the Customer Success Manager. [Chapter 06](./06-scope-discipline-and-decision-log.md) assumed the person saying "no, later, after hand-off" is not the Professional Services consultant who would be happy to say yes and bill the hours. Those assumptions are reasonable in the abstract. Executive comms is where they are tested.

Executive comms is the single place in the pilot where the FDE's role boundary is under the most pressure. The VP asks for a commercial commitment in the demo; the customer sponsor asks for a roadmap commitment in the weekly review; the SME asks for a bespoke deliverable at the hand-off ceremony; the CIO asks for a strategic-roadmap conversation after a successful read-out; the CSM asks the FDE to downplay a yellow metric so the quarterly business review (QBR) looks cleaner. Every one of those asks is a request that the FDE briefly stop being an FDE. Every one of them is answered by a reflex this chapter writes.

[Mod-101 Chapter 04](../mod-101-fde-discipline-and-position/04-adjacent-customer-facing-roles.md) is the canonical reference for the roles themselves — what each adjacent role is measured on, what their primary artefact is, where the FDE touches them, and what sharp boundary this track enforces. This chapter does not re-author that depth. This chapter takes those boundaries and operationalises them for the executive-facing conversation.

## The adjacent-role recap

This table is a pointer back to [mod-101 Chapter 04](../mod-101-fde-discipline-and-position/04-adjacent-customer-facing-roles.md). The full writeup lives there. This is the handshake the FDE uses in an exec meeting when the role is in the room (or should be, and is not).

| Role | What their quarter is measured on | The FDE's handshake in exec comms |
|---|---|---|
| `account-executive` (AE) | Bookings, signed contracts, quota | "That's an AE call — let me bring them in before we commit on paper." |
| `solutions-engineer` (SE) / pre-sales | Technical win that unlocks the signature | Pre-sales conversation; the FDE defers any "would the product do X in a POC" question that is really a sales question |
| `customer-success-manager` (CSM) / `customer-success-engineer` (CSE) | Account health, renewal, expansion (NRR) | "Account health and the QBR narrative are the CSM's turf — let me loop them in." |
| `professional-services-consultant` (PS) | Hours delivered against SOW | "That's a Professional Services engagement — let me check whether the model fits what you're asking for." |
| `partner-engineer` | Partner-ready integration; partner enablement | "That's partner-engineering shaped — my customer is your team, not the SI." |
| `developer-advocate` (DevRel) | External developer ecosystem, public content | "I can write the internal artefact; DevRel owns the public version." |
| `technical-account-manager` (TAM) | Broad technical account relationship | "The platform-wide question is a TAM question; the pilot is mine." |
| `field-cto` / field engineering leader | VP+ strategic-technical narrative | "The strategic-architecture conversation sits with our Field CTO — let me set that up." |

Every row is a specific hand-off the FDE rehearses before the exec meeting starts. The handshake is a one-sentence reflex, not a paragraph.

## The six executive-comms scenarios where role honesty is tested

### 1. The VP asks for pricing or contract terms during the pilot review

The scenario fires in the pilot review, the exec demo, the end-of-pilot read-out, or the informal hallway conversation after the read-out. The customer VP or CFO has seen the demo land; they are modelling the Q2 budget; they want a number.

> "Great demo. If we wanted to roll this out to the other three business units in Q2, what would that run us?"

The right answer is not a number. The right answer is not "that depends." The right answer is not a range. The right answer routes the question — by name, in the room, in front of the sponsor — to the role that holds the authority.

> "That's my `account-executive`'s call, not mine — the FDE role does not carry a quota or price authority. Let me loop Sam in today and get you a scoped number before the end of the week. In the meantime, I can describe what the technical scope of a BU-2/3/4 rollout looks like — the integration surface, the new SSO tenants, the incremental gold-set work — and Sam can price that shape."

The two moves inside that script:

1. Decline the price question by name of role, not vaguely. "Let me get back to you" reads as uncertainty; "that's my AE Sam's call" reads as structure.
2. Offer the technical-scope framing *anyway*, because the AE needs it to quote. The FDE has made themselves useful to the AE without stepping on the AE's deliverable.

Follow-up email template (same day, to the AE with the sponsor copied):

> "Sam — the CFO raised rolling out to BU-2/3/4 in Q2 during today's pilot review. I did not quote a number. Rough technical-scope sketch attached (three new SSO tenants, two new data-source integrations, an estimated 60 incremental gold-set entries). Can you own the follow-up with Priya by Friday? I'll support the technical-scoping side when you're ready."

The failure mode is the number quoted off-the-cuff in the room. The AE finds out two days later from the customer's procurement team; the AE's prep is undermined; the quote is wrong, or it is right but inconsistent with the AE's negotiation posture (discount strategy, multi-year terms, concessions already offered elsewhere in the account). The FDE has now cost the account trust *inside the vendor*, which is harder to repair than the equivalent customer-side trust cost.

### 2. The customer sponsor asks when a product feature will ship

The scenario fires in the weekly review, usually when a sponsor has been patient about a known gap and is now trying to pin down a date.

> "The multi-tenant tenant-isolation feature — when will that be in general availability? Our BU-2 rollout depends on it."

The FDE does not set the roadmap. [Mod-111 Chapter 05](../mod-111-field-to-product-loop/05-roadmap-influence.md) wrote the influence channel the FDE uses to *lobby* for roadmap moves; it is explicit that commitment authority sits with the Product Manager. The FDE does not commit on behalf of the PM in a customer meeting, even when the FDE believes the feature is coming, even when the PM has verbally told the FDE it is coming.

> "That's a commitment I can't make in this room — roadmap dates belong to the PM for that surface. What I can do is write up this ask as a platform-request packet this week, get it into the PM's queue, and come back to you with their commitment at the next review. If the date matters to your Q2 planning, I will flag that explicitly in the packet so it is prioritised against the right window."

The rule is: lobby inside the vendor; report back the PM's answer to the customer; never speak for the PM. The sponsor who is given a shipping date the PM did not sign for is a sponsor who will eventually be told a different date, and the FDE will carry the cost of the retraction. See [mod-111 Chapter 04](../mod-111-field-to-product-loop/04-platform-request-packets.md) for the packet shape and [mod-111 Chapter 05](../mod-111-field-to-product-loop/05-roadmap-influence.md) for the influence protocol.

### 3. The SME asks the FDE to build a bespoke SOW-style deliverable

The scenario fires most often at the hand-off ceremony or at the moment the SME realises the FDE's calendar is about to disappear.

> "Could you also build us a bulk-export tool for the legacy ticket archive while you're here? We'd just need a one-time run to migrate 2017–2019. It'd probably take you a week."

This is a Professional Services ask. It is well-formed, it is paid-for-able, it is a genuine customer need — and it is not FDE-shaped. It will not generalise; it will not inform the product roadmap; it will consume two weeks of FDE time that the pilot does not have; and it establishes the pattern that bespoke-only one-offs are what the FDE does.

> "That's great Professional Services work — one-time, scoped, deliverable-shaped. Let me check with my account team whether a `professional-services-consultant` engagement fits what you're asking for. The FDE scope is the pilot workflow we signed; I don't want to crowd that out with a bespoke one-off we won't generalise. If PS isn't the right fit either, there's an option to have your `applied-ai-engineer` team pick it up after hand-off — I can help scope it either way."

The follow-up: loop in the AE and the PS practice lead; confirm the engagement model — is it a paid SOW? is it a design-partner concession? is it something the customer's own applied-AI-engineering team should own post-hand-off? Return to the SME with a named owner inside 48 hours.

The FDE declines the work *and* keeps the ask alive — which is what distinguishes role honesty from brush-off. An SME who sees their ask routed to a named owner feels heard; an SME who sees their ask routed to silence feels brushed off, and the next time will not mention the ask at all, which is worse.

See [Chapter 06](./06-scope-discipline-and-decision-log.md) for the full four-step scope check and [mod-101 Chapter 04](../mod-101-fde-discipline-and-position/04-adjacent-customer-facing-roles.md) Role 4 for why bespoke-only SOW work is not FDE work.

### 4. The customer's CIO asks for a strategic-roadmap conversation

The scenario fires after a successful exec demo or at the quarterly business review, often as a sign of engagement the FDE has worked hard to earn.

> "This has gone well. I'd like to talk through how AI fits into our three-year architecture roadmap. Can we book an hour?"

This is a Field CTO conversation. The FDE is engineer-level technical voice; the CIO is asking for the senior strategic-technical voice — the voice that speaks for the vendor's architectural direction, the voice whose statements will land on the customer's executive-architecture slide. Even an FDE who could handle the conversation on competence should not handle it on role.

> "That's a conversation I'd want our `field-cto` in — the FDE role is pilot-scoped and the three-year architecture arc is the Field CTO's territory. Let me set up the intro this week. I'll be in the room because the pilot's specifics are useful context, but I'll not be driving the strategic frame."

If the vendor does not have a Field CTO, the escalation is to the `ai-engineering-manager` or the VP of Field Engineering — but the escalation happens *by name* in the customer meeting, not as a vague "let me check."

Do not improvise senior-technical strategy at the FDE level. The exec in the room will take the FDE's statements as commitments the vendor has not actually made, and those statements will reappear in the customer's internal decks six months later as "your vendor told us X." The retraction, when it comes, lands harder than a cleanly-routed decline would have.

[Mod-101 Chapter 04](../mod-101-fde-discipline-and-position/04-adjacent-customer-facing-roles.md) Role 8 is explicit about this boundary.

### 5. The CSM asks the FDE to downplay a yellow metric in the quarterly review

The scenario fires in the pre-QBR prep meeting, usually the day before the actual QBR, when the CSM has scanned the deck and seen a row that will not land well.

> "Can we present policy-citation at 91% as 'approaching green'? The 93% threshold is a stretch and the QBR is tomorrow — the sponsor will react badly to a yellow row and we've got a renewal conversation after it."

No. The answer is no, and the FDE should expect that saying no here is the single highest-stakes role-honesty moment in the pilot.

> "I can't do that. The QBR table has to match the dashboard the sponsor sees, otherwise the next time they check the dashboard the whole QBR narrative falls over. If 91% is yellow against the 93% threshold we signed, that row is yellow. What I *can* do is add the trend arrow — we're up from 86% six weeks ago, which is the real story — and bring the SME to the QBR to speak to the gold-set additions driving the improvement. That's a stronger story than 'approaching green' because it's defensible."

[Mod-110](../mod-110-observability-and-oncall/) wrote the blameless-and-honest posture on operational signals; [Chapter 05](./05-customer-side-resistance.md) wrote this module's radical-transparency stance on customer-side difficult conversations. Both apply here. A QBR that under-reports is a QBR the sponsor cannot trust, and the sponsor who catches it once — and they do catch it, because the sponsor pulls the dashboard independently more often than the FDE expects — stops trusting every green row that follows.

The CSM is not asking for anything malicious; they are asking for a narrative adjustment they believe helps the account, which it would, briefly, at the cost of the account's credibility with the sponsor over the full engagement.

The FDE's counter-offer is a *better* narrative the CSM can use: trend arrow + SME quote + named driver. That is the script that holds the line without a fight — and it gives the CSM something to walk into the renewal conversation with, which is what the CSM actually needed. The structural pattern here is general: when an adjacent role asks the FDE to compromise the technical signal, the FDE's response is to give them a stronger non-compromising alternative. That is the FDE's actual value to the CSM's job, not the compromise they asked for.

### 6. The customer's sponsor asks the FDE to represent the FDE's team's headcount or utilisation

The scenario fires when the sponsor is starting to think about the next phase of the engagement and wants to budget for more FDE time.

> "How many FDEs does your team have? Could we get another one on this account next quarter? And is yours full-time on us, or are you split?"

Capacity and headcount are the `ai-engineering-manager`'s turf. The FDE does not quote their own team's headcount in a customer meeting, does not commit another FDE to the account, does not describe the vendor's field-engineering capacity model, and does not describe their own allocation split across accounts. Each of those is a vendor-internal number the customer does not need to see to run the pilot, and that the customer *will* reason from if it is given to them.

> "Capacity and staffing are my manager's call — let me set up a 1-on-1 with my `ai-engineering-manager`, Priya, so she can walk you through how we staff design-partner accounts and whether a second FDE fits the roadmap for this engagement. I don't carry that authority."

This mirrors [mod-111 Chapter 05](../mod-111-field-to-product-loop/05-roadmap-influence.md) — commitments the FDE does not hold are escalated, by name, to the person who does.

## The three sentence-patterns the FDE learns to say instinctively

Underneath all six scenarios are three sentence-patterns. The FDE drills these until they come out without thinking — because the moment the FDE has to construct the sentence from scratch, they are already a half-beat behind the room and the yes-in-the-room is halfway out of their mouth.

1. **"That's a `<role>` conversation — let me bring them in."** Used when someone with the authority is reachable and the FDE just needs to route. Variants: "That's an AE call," "That's a PM call," "That's a Field CTO conversation." The role is named explicitly in the same breath as the decline, because the customer needs to know *who* to expect contact from.
2. **"I can help you scope it, but the delivery model is `<professional-services | applied-ai-engineer | partner-engineer>`, not FDE."** Used when the work is real work the vendor (or the customer's own team post-hand-off) can do, just not under the FDE engagement. Variants: "...the delivery model is a paid SOW," "...the delivery model is your own team after hand-off." The FDE offers scope-help as a concession, which keeps the relationship warm.
3. **"I don't carry that measure / that quota / that authority — here's who does."** Used when the ask is commitment-shaped and the FDE has no mandate to commit. Variants: "I don't carry a quota," "I don't carry roadmap commitment authority," "I don't sign BAAs." The admission is specific; vagueness here reads as evasion.

Three sentences; six scenarios; indefinitely many variations. The customer learns within two or three meetings that certain asks route and certain asks land — and the routing is fast, named, and respected. By the end of the pilot the customer's own executive team has internalised which asks go to whom, which is itself a hand-off deliverable.

## Signals this is the moment to route

The FDE who has not yet developed the reflex benefits from a short mental checklist. When any of the following are true in a customer meeting, the ask is probably a route, not an answer:

- The ask would, if answered yes, commit the vendor to a *number* — price, headcount, date, SLA percentage.
- The ask would commit the vendor to a *contractual obligation* — a BAA, an attestation, an SLA, an indemnity.
- The ask would commit the vendor to a *strategic direction* — a roadmap claim, an architectural direction, a long-term stance.
- The ask would require the FDE to *re-frame a metric* that is already measured and dashboarded.
- The ask is addressed to the FDE in the room only because the role-correct person is absent.

If any of those triggers fire, the FDE pauses, routes by name, and keeps the thread alive. If none fire, the FDE is almost certainly in-scope and can answer.

## How to hand off without dropping the ball

Role honesty is not "throw it over the wall and walk away." A route that collapses on the customer's side is worse than no route at all; the FDE's reflex is to route by name, to confirm the hand-off in writing, and to stay on the thread until the named owner has picked it up.

- **Named introduction in the room.** "Priya will own this — I'll introduce you over email today." A nameless "someone from our team will follow up" is an orphan hand-off waiting to happen. The name is spoken in the meeting so the customer can write it down.
- **Written follow-up inside 24 hours.** Email the named owner, copy the customer, state the ask in one sentence, state the expected next step, state the expected timing. The customer sees the hand-off happen, not just hears it promised.
- **Owner confirms in-thread.** The FDE does not close the loop until the named owner has replied in the thread acknowledging the ask. If they have not acknowledged within 48 hours, the FDE nudges — because the customer is waiting, and the customer's patience is the FDE's credibility.
- **The FDE stays on the thread, not on the work.** Observer presence, not driving presence. The owner drives. If the owner needs technical context, the FDE supplies it; the FDE does not re-seize the work.

An orphaned hand-off — the customer told to expect a follow-up from someone who has no idea they own it — is worse than no hand-off. The customer loses trust in the FDE's routing; the next time, they will ask the FDE to just handle it, and the FDE will be back under scope pressure with a weakened argument for declining.

## The drift check — the FDE's quarterly self-audit

Role honesty erodes slowly. The weekly review the CSM cannot attend; the pricing question the AE was travelling for; the roadmap conversation the PM did not answer fast enough; the SME ask the FDE just quietly took on because declining would have been socially awkward. Each is small; cumulatively they are drift.

Once per quarter the FDE audits their own calendar and work-log. Categorise every hour against the eight adjacent roles from the table above plus "FDE-shaped work." The target is >85% FDE-shaped; the warning line is any single adjacent role exceeding ~10% of hours; the red line is any single adjacent role exceeding ~20%. The audit is honest when it is uncomfortable — the FDE who records every "quick favour" to the CSM is the FDE whose numbers are usable.

If the audit shows 15% in CSM work, the FDE raises it with their `ai-engineering-manager` and the account's CSM — the CSM is overloaded, or the FDE has drifted, or both, and the fix is a staffing conversation, not a resolution the FDE makes alone. If the audit shows 20% in PS work, the pilot's scope has slipped and [Chapter 06](./06-scope-discipline-and-decision-log.md)'s scope check has stopped firing; the fix is to re-anchor on the scope contract and route open bespoke-asks to PS before more accumulate. If the audit shows 15% in AE work, the AE is absent from the account and the FDE is proxying for commercial conversations — a conversation the FDE's manager needs to have with the AE's manager, not a conversation the FDE resolves in-account.

Either way, the recovery reference is [mod-101 Chapter 02](../mod-101-fde-discipline-and-position/02-the-four-stances.md) — the four stances the FDE resets to when drift has set in.

The audit is cheap. Thirty minutes of calendar review per quarter, plus a quick scan of the decision log and the open-asks tracker. The cost of not running it is a pilot in which the FDE is doing three adjacent roles badly and their own role poorly — the state in which the next pilot is already under-staffed and the FDE does not know it yet.

## Why role honesty strengthens customer trust

The intuition cuts the other way. The FDE who routes feels, in the moment, like they are refusing to help. The FDE who commits on the spot feels generous. This is a trap — and it is the single most consistent reason FDEs in their first pilot drift off-role.

The customer who sees the FDE route pricing to the AE learns that the FDE's technical statements are not coloured by a commercial incentive. The customer who sees the FDE decline to downplay a yellow metric learns that the FDE's green metrics are credible. The customer who sees the FDE refer the CIO-level architecture conversation to the Field CTO learns that the FDE is not improvising seniority. The customer who sees the FDE route a bespoke SOW ask to Professional Services learns that the FDE's scope-protection is not stinginess — it is discipline. Each visible routing is a *credibility deposit*. Over the course of a pilot those deposits compound, and by the renewal conversation the FDE's word on a technical claim carries more weight than most of the room.

The FDE who tries to be AE and CSM and PS and Field CTO in one body does not become more valuable; they become harder to trust, because the customer can no longer tell which hat is speaking. Was that technical architecture claim real, or was it a commercial soft-sell? Was that green metric honest, or was it a relationship-management gloss? Was that roadmap date a commitment, or wishful thinking? Role honesty is the structural cost of being the single most credible voice in the room on the technical pilot — and the structural reason customers ask for FDEs specifically on their next engagement.

**The renewal signal.** At renewal the sponsor's internal conversation almost always includes "whose word did we believe over the last year?" The FDE whose statements were cleanly bounded — this is a technical claim, that was a route to the AE, this metric is honest, that commitment is the PM's — sits at the top of that list. The FDE who improvised authority sits lower, sometimes much lower, even when their technical work was strong. Role honesty is a quiet variable in renewal maths; it is also the variable the FDE has the most direct control over.

## Preparing the meeting so routing is easy

Role honesty in the moment is easier when the prep has been done the day before. The FDE's pre-meeting checklist includes a role-honesty row that is just as important as the metrics row.

Before every exec-facing meeting the FDE answers four questions:

1. **Who holds authority for each likely ask?** The FDE lists the three or four most likely asks (pricing, roadmap, bespoke, strategic, metric, headcount, security) and names the role-correct owner for each.
2. **Is that owner reachable today?** If the AE is on PTO and a pricing ask is likely, the FDE either defers the question to the next meeting or pre-coordinates a backup — "if a pricing question comes up, I'll route to the AE's manager, who is Dan."
3. **What is the handshake sentence for each?** The FDE drafts the one-sentence route for each likely ask. In-the-room is not the moment to compose.
4. **What is the written follow-up template for each?** The FDE has a drafted email template per anticipated route, so the 24-hour follow-up actually happens.

Thirty minutes of prep. The ROI is the difference between a VP who hears a confident, named route and a VP who hears "let me check and get back to you" — the former banks credibility, the latter spends it.

## Running the three archetypes

**Archetype A — HIPAA payer.** The customer's VP of Compliance commonly asks for compliance-posture commitments the FDE cannot make live.

"Will your platform sign our Business Associate Agreement (BAA) with the data-processor addendum language from our template?" is a legal and contractual question routed to the vendor's legal + `ai-engineering-manager` chain. The FDE acknowledges the ask in the room, confirms the BAA-and-HIPAA-Security-Rule discussion happens with the compliance leads on both sides, and does not improvise an answer about de-identification obligations under 45 CFR §164, incident-notification timelines, or breach-definition edge cases.

The follow-up thread names the vendor's HIPAA compliance owner and the customer's privacy office contact, in writing, within 24 hours. The FDE's role in that thread is observer — the compliance leads drive. The FDE's role in the *next* pilot review is to report back on the compliance leads' progress, not to carry the compliance conversation inside the review. <!-- needs-research: confirm the specific 45 CFR §164 subparts that typically land in BAA addendum negotiations for AI-processing use cases -->

**Archetype B — FedRAMP agency.** The customer's Contracting Officer (CO) asks for schedule commitments mid-pilot.

"Can you accelerate the ATO-boundary rollout to BU-2 by Q3?" is routed to the `account-executive` and the `ai-engineering-manager`; schedule acceleration in a federal contract has contracting-vehicle, authorisation-boundary, and NIST SP 800-53 Rev 5 control-tailoring implications the FDE does not resolve in a review, and may require an SSP revision under FedRAMP continuous-monitoring guidance.

The FDE's script in the room: "Schedule commitments against the ATO boundary sit with our account team and our AO point of contact — let me loop Sam and Priya in today, and we'll come back to you at the next review with what the authorisation implications look like."

The FDE describes the *technical* shape of accelerating BU-2 — integration surface, data-source additions, new SSO tenants — because the AE will need it to scope the contracting-vehicle conversation. The FDE does not describe the schedule itself. <!-- needs-research: confirm whether a scope change like this typically requires an SSP revision under FedRAMP continuous-monitoring guidance or triggers a separate authorisation -->

**Archetype C — EU retail bank.** The bank's model-risk director asks the FDE for ongoing model-performance attestation beyond the pilot.

"Can you provide monthly attestation against our model-validation standard for the deployed system?" is routed to the vendor's trust and assurance function and the Field CTO. Ongoing attestation is an obligation with DORA ICT-third-party and EU AI Act post-market-monitoring implications the FDE does not sign for unilaterally; monthly attestation that references the bank's internal model-validation standard will become the governing document the bank's second-line risk team audits against, and misaligning it against the vendor's actual assurance function is a liability the FDE has no authority to accept.

The FDE confirms the ask in the room, names the vendor's trust-function owner, and sets up the three-way between the model-risk director, the vendor's trust function, and the bank's second-line risk team within the week. The FDE stays in the thread as technical observer, not signatory. <!-- needs-research: confirm which EU AI Act Article 72 reporting obligations fall on the deployer vs. the provider in a bank's typical deployment pattern -->

## In-the-room routing vs between-meeting routing

A note on cadence. Three of the six scenarios fire *in the room* with the customer's exec present: the VP pricing ask, the CIO strategic-roadmap ask, and the sponsor headcount ask. The other three fire in smaller settings — the SME in the weekly review, the sponsor on the roadmap question in a working session, the CSM in the pre-QBR prep.

The in-the-room asks are higher-stakes because the exec sees the routing happen; the routing *is* the credibility deposit. A hesitant or vague route in front of a VP undoes two months of pilot trust. These scripts are the ones the FDE rehearses out loud before the meeting, in the same way they would rehearse the demo. The between-meeting asks have a slower clock; the FDE can take a breath, draft the email, and route in writing — but the same three sentence-patterns apply.

The FDE who has not rehearsed the in-the-room script is the FDE who defaults to "let me get back to you on that" — which is better than yes, but worse than a named, routed, confident decline.

## Anti-patterns

- **The yes-in-the-room.** The VP asks a question outside the FDE's authority; the FDE answers rather than routes; the AE / PM / CSM / legal finds out from the customer two days later. The exec-comms-specific variant of the generic scope-yes failure — and the single most common role-honesty failure in a first pilot.
- **The shadow AE.** The FDE quotes a price, a discount, a contract-term, or a renewal number without the `account-executive` in the loop. Even directionally-correct quotes damage the AE's negotiation posture; wrong quotes damage the account.
- **The shadow CSM.** The FDE makes health-check promises ("don't worry, we'll make sure your renewal looks good"); the CSM later has to manage a conversation the FDE primed in a direction the CSM would not have chosen, or has to retract a commitment the FDE did not have authority to make.
- **The Field-CTO cosplay.** The FDE accepts the CIO's strategic-roadmap conversation; the FDE speaks to three-year architecture arcs the vendor has not formally committed to; the customer treats the statements as commitments; the Field CTO later walks them back and the trust line to both roles is damaged.
- **The silenced yellow.** The FDE lets the CSM-nudged "approaching green" phrasing survive into the QBR; a yellow row is presented as near-green; three weeks later the sponsor discovers the actual number and loses faith in every row of the QBR going forward.
- **The bespoke yes.** The SME's one-off deliverable is quietly taken on under FDE scope; two weeks of FDE time disappears; the pilot's actual deliverable slips; PS loses a billable engagement they should have had; the next bespoke ask arrives with the expectation that it will be said yes to.
- **The orphaned hand-off.** The FDE routes the customer to a named colleague without warning that colleague; the customer emails; the colleague has no context and takes four days to respond; the customer concludes the routing was a brush-off and asks the FDE directly next time.
- **The tolerated drift.** The quarterly self-audit shows 20% of hours in CSM-shaped work; the FDE notes it and does nothing; by the next quarter it is 30%; the FDE's own pilot deliverables are two weeks behind and there is no clean recovery path. [Mod-101 Chapter 02](../mod-101-fde-discipline-and-position/02-the-four-stances.md) is the reset.

## Summary

- Executive comms is where the FDE's role boundary is under the most pressure. The VP, the sponsor, the SME, the CIO, the CSM, and sometimes the sponsor again on headcount — each will ask the FDE to briefly become an adjacent role.
- The canonical role boundaries live in [mod-101 Chapter 04](../mod-101-fde-discipline-and-position/04-adjacent-customer-facing-roles.md). This chapter operationalises them for the exec-facing conversation only; the adjacent-role recap table points back to that chapter and is not a substitute for it.
- Six scenarios are rehearsed with explicit scripts: pricing asks (route to `account-executive`), roadmap asks (route via the [mod-111 Chapter 05](../mod-111-field-to-product-loop/05-roadmap-influence.md) influence channel), bespoke SOW asks (route to `professional-services-consultant`), strategic-roadmap asks (route to `field-cto`), downplay-the-yellow asks (decline and counter-offer), and headcount asks (route to `ai-engineering-manager`).
- Three sentence-patterns carry most of the load: "that's a `<role>` conversation — let me bring them in," "I can help you scope it, but the delivery model is `<role>`, not FDE," and "I don't carry that measure / that quota / that authority — here's who does."
- A five-trigger checklist — number commitment, contractual obligation, strategic-direction claim, metric re-framing, role-correct person absent — identifies in the moment when the FDE should route rather than answer.
- Hand-offs are named, written up within 24 hours, acknowledged by the owner in-thread; an orphaned hand-off is worse than no hand-off.
- A quarterly self-audit against the eight adjacent roles catches drift early; >10% of hours in any single adjacent role is a warning, >20% is a red line and triggers the [mod-101 Chapter 02](../mod-101-fde-discipline-and-position/02-the-four-stances.md) reset.
- Role honesty is a credibility deposit, not a limitation. The customer who sees the FDE route is a customer who trusts the FDE's technical statements more, not less — and the compounding of those deposits is what makes the FDE the highest-credibility technical voice in the room by renewal time.
- Eight anti-patterns name the common ways role honesty fails: yes-in-the-room, shadow AE, shadow CSM, Field-CTO cosplay, silenced yellow, bespoke yes, orphaned hand-off, tolerated drift.
- Chapter 08 closes the module. The FDE who has internalised chapters 02–07 and holds the boundary written here delivers a pilot whose technical excellence survives its executive-communication surface — and whose renewal is the AE's to close, not the FDE's to improvise.
