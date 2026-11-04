# Chapter 02 — The Four Stances of the Discipline

## Why this chapter exists

Chapter 01 named the four structural stances that survived every re-instantiation of the Forward Deployed Engineer role. This chapter unpacks each one. The point is not slogans — it is the specific behaviours each stance demands and, more importantly, the behaviours each stance *rules out*. If you cannot name what a stance costs you, you do not hold the stance.

Think of this chapter as a self-check: at the end of each section you should be able to say "here is what I would stop doing to live this stance, and here is what I would start doing."

## Stance 1 — Embedded delivery

**The stance.** You work close enough to the customer's real workflow that the first thing you do each day is see or hear it happen. You sit in their standup when they'll let you. You watch a user do the job. You are not downstream of a requirements document someone else wrote. You are the person writing the requirements document, inside the customer's building (physical or Slack), by observing.

**Why.** LLM features fail in the field for reasons that do not appear in a requirements document — the operator has a workflow habit nobody wrote down, the data the product assumes is clean is 40% junk, the "success" the product promises is the third-most-important thing to the customer that quarter. The only reliable way to catch these is proximity.

**What you do.**

- Shadow the user. In the first week of a pilot, you spend hours watching the person whose work you are trying to change do it today.
- Run discovery against the customer, not from a desk at HQ. The customer teaches you what the pilot should be. You bring the engineering vocabulary.
- Ship inside their environment. Your pilot code runs on their cloud or their VPC or their laptop under their SSO — not on your laptop with a tunnel.
- Share on-call with them. When the pilot pages at 3am, you are on the bridge next to their operator, not waiting for a ticket to be filed back to HQ.

**What you stop doing.**

- You stop writing detailed design docs before you have seen the workflow. (A one-page discovery brief after week one, then a scoped pilot plan — not a 40-page architecture document on day one.)
- You stop treating the Account Executive's "the customer said they want X" as the spec. You go verify directly.
- You stop building anywhere you cannot show the customer the working thing by Friday.

**How you know you are living it.** The customer can name you. The customer's engineer has opened your PR. The customer's security team recognises you in a review. You can draw their workflow from memory.

## Stance 2 — No-ego engineering craft

**The stance.** You ship whatever gets the pilot to work. Python glue. A SQL view. A 50-line retrieval layer. A webhook handler that will embarrass you. A config patch. A manual-for-now step. You do not defend a pet architecture, and you do not spend the pilot budget on a framework the customer will never maintain.

**Why.** The pilot is a learning instrument *and* a production system. If you optimise for engineering elegance, you miss the learning window. If you optimise for engineering shortcuts the wrong way, the pilot cannot go live. The craft is in picking the right level of rigour for each piece — and the honesty to admit when the thing you shipped is the simplest thing that could work, not a cathedral.

**What you do.**

- Prefer boring tech. The customer will inherit this. Python + a managed Postgres + a known LLM provider + a thin FastAPI is almost always right. Exotic frameworks are a tax on the hand-off.
- Match rigour to risk. A pilot workflow that an SME reviews before action can run on a prompt and a judge. A pilot action that posts to an SAP record under an authenticated service account needs a test suite, structured outputs, retries, and an audit log.
- Delete your own code. The pilot that does not go live is a successful piece of learning. Do not sunk-cost it.
- Admit what the shipped thing is not. If the retrieval layer is going to fall over at 10× scale, write that down for the hand-off packet and stop pretending.

**What you stop doing.**

- You stop benchmarking vector databases in month one of a pilot that is going to send ten queries a day for a month.
- You stop re-implementing an agent framework because you want to learn LangGraph on the customer's dollar.
- You stop defending prototype code that the customer's engineer will have to inherit. If they would not merge it in their own review process, do not ship it.

**How you know you are living it.** You can describe three things you deleted this month. You can show the customer's engineer exactly where the shortcuts are and why you took them. You have passed up at least one architecturally interesting detour because the pilot did not need it.

## Stance 3 — Design-partner-first mindset

**The stance.** The customer in front of you is a co-author of the product, not an endpoint for a shipped feature. You are there because this specific customer, this specific workflow, and this specific stack are going to teach the product something the product team cannot learn from inside the office. Your contract with them — written or implicit — is that you will give them attention, speed, and a working thing, in exchange for the right to feed what you learn back to product.

**Why.** Design partners are how the product team gets to see failure modes before the general market does. They are the customers who agree to a less polished product in exchange for influence on where it goes. If you treat a design partner like a standard customer, you waste the arrangement. If you treat a standard customer like a design partner, you overpromise and under-deliver.

**What you do.**

- Name the design-partner contract. In the kickoff, you and the customer both agree what the arrangement is: are they paying full price for a stable product, or paying reduced-or-zero for a less stable product with influence? If the answer is unclear, you have not had the conversation yet.
- Give the design partner attention. A design partner at a frontier lab or applied-AI company often gets the FDE's direct Slack, not a support queue.
- Give the design partner *speed*, not *bespokeness*. A design-partner engagement that produces a one-off feature only this customer can use is a failed engagement. A design-partner engagement that produces a feature every future customer will inherit is the goal.
- Negotiate scope ruthlessly. The design partner will ask for things that are not design-partner-shaped — a UI change for their tenant, a compliance report for their board. You say no to the ones that do not produce product value, and you say so out loud.

**What you stop doing.**

- You stop quietly building bespoke code paths for one tenant that the next customer would never use. If you have to, you flag it.
- You stop treating pushback from the customer as the final word. Design partners *should* push on the product. You bring it back to product, together with your recommendation.
- You stop pretending the design-partner contract is a normal customer contract. The expectations on both sides are different.

**How you know you are living it.** At the end of each pilot cycle you can name the three product changes that came out of this partner. You can also name the two feature asks you turned down, and the customer knows and respects why.

## Stance 4 — Field → product loop as a first-class deliverable

**The stance.** Every engagement produces artifacts that the product and platform teams back home can use. Pattern writeups. Pain-point taxonomies. Platform-feature requests with two customers' evidence attached. Failure-mode library entries. Not a single blob of tribal knowledge in your head — written, versioned, searchable. The loop is not something you do when a pilot winds down. It is a deliverable at every milestone.

**Why.** Without the loop, an FDE team is a set of disconnected consultants: each engagement helps one customer, nothing compounds, the product learns nothing. With the loop, each engagement makes the product measurably better for the next customer and the next FDE. The *compounding* is the business reason the role exists.

**What you do.**

- Capture field telemetry with intent. A failure is not just "this pilot broke" — it is a candidate regression fixture, a candidate eval case, a candidate product bug, a candidate platform-gap writeup. You triage every failure into one of those buckets.
- Package platform requests properly. "I want a feature" is not a packaged request. "I want a feature; here are two customers who need it; here is the proposed interface; here is the workaround I shipped in the interim; here is the risk if we do not do it" — that is a packaged request.
- Author design-partner artifacts on a product-team-friendly cadence. A quarterly pattern writeup, a per-pilot "what we learned" one-pager, a per-engagement failure-mode library entry. The product team knows when to expect them.
- Influence without owning. You do not own the roadmap. You show up prepared, bring data, bring the customer voice, propose, and accept the answer. If the product team says "not this quarter," you ship the workaround and come back next quarter.

**What you stop doing.**

- You stop keeping lessons in your head. If it is not in a writeup, it is not a deliverable.
- You stop lobbying for your pet feature. You propose with evidence; you accept the no.
- You stop treating the hand-off packet as the only artifact. The real deliverable is the *difference* this engagement made to the product.

**How you know you are living it.** At the end of a pilot the product team can point to three things on their roadmap that your engagement caused. Another FDE, picking up the next customer, can open your pattern writeup and avoid the mistakes you made. Your manager can show your engagement on a slide at the all-hands.

## Why these four, together

The four stances are mutually reinforcing, and each one is slightly uncomfortable to a different part of your engineering brain:

- **Embedded delivery** is uncomfortable if you learned software engineering as "ship against a spec." You will not have a spec. You will produce one.
- **No-ego engineering** is uncomfortable if your engineering identity is tied to architectural ambition. You will ship small things. You will delete them.
- **Design-partner-first** is uncomfortable if your engineering identity is tied to pleasing the customer in the room. You will say no, out loud, with reasons.
- **Field → product loop** is uncomfortable if your engineering identity is tied to the code that shipped. Much of your value will be written in prose, not Python.

You cannot hold any one of them without the other three — the embedded engineer with an ego defends a bespoke prototype; the no-ego engineer without a design-partner contract becomes a free consultant; the design-partner mindset without the feedback loop produces a lot of bespoke code and no product change. Together they are the discipline. The rest of this module — the ladder position, the role boundaries, the deliverables, the escalation logic — is the operational expression of these four stances.

## Summary

- **Embedded delivery**: close to the real workflow; ship inside the customer's environment; share their on-call.
- **No-ego engineering**: boring tech, right-size rigour, delete without pride, admit what the shipped thing is not.
- **Design-partner-first**: the customer is a co-author of the product; attention + speed, not bespokeness; say no to asks that do not produce product value.
- **Field → product loop**: writeups on a cadence, packaged platform requests, pattern libraries, influence without ownership.
- The four stances reinforce each other and each one is uncomfortable in a different place. The discipline is holding all four.
