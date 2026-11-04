# Chapter 04 — Adjacent Customer-Facing Roles and Sharp Boundaries

## Why this chapter exists

"Customer-facing engineering" is not one role. At a mature AI company it is six or seven distinct roles, each with its own quota or OKR structure, its own tenure model, and its own definition of a good month. The Forward Deployed Engineer sits in the middle of that cluster and shares boundaries with every one of them. If you cannot tell your work apart from the work of the Solutions Engineer who hands you a pilot, or the Customer Success Manager who inherits it, you will end up doing all of their jobs badly and your own not at all.

This chapter goes role-by-role. For each adjacent role you get the same four cells:

1. **Primary goal** — what their incentive structure actually rewards.
2. **Primary artifact** — what lands on a slide at the quarterly review.
3. **Where you touch them** — the specific handoff(s) the FDE has with that role.
4. **The sharp boundary this track enforces** — the thing you do not do, even when asked.

The adjacent roles vary from company to company in title and scope. The *functions* are stable. Match the function to the role at your company; do not take the titles too literally.

## Role 1 — Account Executive (AE) / Sales

**Primary goal.** Close the deal. Hit quota. Drive bookings and pipeline.

**Primary artifact.** Signed contracts.

**Where you touch them.** The AE brings in the design-partner opportunity and is involved in scoping commercials. During a pilot, the AE typically stays in the account relationship — especially for the "is this going to renew / expand" question — but is not in the technical delivery.

**Sharp boundary.** You **do not carry a quota**. You **do not negotiate price**. You do not represent the company's commercial position in a customer meeting unless the AE has briefed you to and is in the room. When a customer asks "what would you charge to do X," the answer is "I'll bring in my AE — that's their call, not mine." If you are routinely acting as a backup AE, you have drifted off-role.

## Role 2 — Solutions Engineer (SE) / Pre-Sales Engineer

**Primary goal.** Win the deal technically. Prove to the prospect that the product can solve their problem. This is primarily *pre-sales*.

**Primary artifact.** Technical win in a POC, demo, or RFP response that unlocks a signed contract.

**Where you touch them.** The SE runs the pre-sales POC. In many companies the handoff from SE to FDE happens at contract signature — the SE's POC becomes the FDE's pilot scope. In some companies the SE and FDE are the same person (small teams) or two phases of the same role (named "Solutions Engineering, pre-sales and post-sales").

**Sharp boundary.** You ship **after** the contract is signed (or inside a design-partner contract that explicitly swaps revenue for product influence). The SE is often rewarded for a working demo on a short timeline that will not hold up in production. You are rewarded for a thing that lives in the customer's environment, through security review, on their SSO, under on-call. **Do not inherit the SE's prototype wholesale.** Inspect it; keep what passes your bar; rewrite the rest. "Keep the SE's code" is a classic failure mode that leads to pilots that cannot go live.

## Role 3 — Customer Success Manager (CSM) / Customer Success Engineer (CSE)

**Primary goal.** Account health, renewal, expansion. "The customer is getting value and will still be a customer next quarter."

**Primary artifact.** Renewal, expansion revenue, health metrics (NRR, usage, satisfaction).

**Where you touch them.** The CSM/CSE holds the longer-term account relationship. The FDE delivers the pilot and hands it off to the CSM/CSE once it is in a steady state. On enterprise accounts a CSM will often sit in a weekly standing with the customer alongside the FDE during the pilot — not as a technical contributor, as a relationship manager.

**Sharp boundary.** You **do not own account health metrics**. You do not try to upsell during a pilot. You do not quiet a legitimate technical concern because it would be awkward with the CSM. When the pilot has an honest failure, you report it honestly in the delivery channel and let the CSM manage the customer-relationship side. If a customer feature ask comes up that is not pilot-shaped, you route it to the CSM and the AE, not to your own code.

Note on **Customer Success *Engineer* (CSE)** vs. CSM: at some companies the CSE is a technical escalation role for post-pilot customers — more like a reactive support engineer with ownership of a subset of large customers. Where a CSE exists, the FDE hand-off goes "FDE → CSE after pilot" and the CSE operates on a steady-state system you built. Your job is to make that handoff work, which means documentation, runbooks, and a working admin dashboard. See mod-109.

## Role 4 — Professional Services Consultant (PS)

**Primary goal.** Deliver billable hours against a Statement of Work (SOW). PS is a revenue line, not a product-influence line.

**Primary artifact.** Hours delivered against SOW; often a deliverable the customer accepts on a defined cadence.

**Where you touch them.** At larger companies, Professional Services and Forward Deployed Engineering are separate orgs. PS takes on paid engagements that do not qualify for FDE (too bespoke, too short-cycle, too low on product-influence value). On smaller AI companies the two blur — but the structural difference remains.

**Sharp boundary.** You are not paid by the hour. Your incentive is *product influence and pilot success*, not maximising billable time on an account. **You are not a PS consultant.** You can decline work that a PS consultant would accept — "this is a bespoke deliverable that will not generalise, we are not doing it on FDE time." You can bring in PS for work that the customer wants and is willing to pay for, that is not design-partner-shaped. The CURRICULUM.md for this track explicitly scopes out "full-time professional services delivery" — the FDE is design-partner engineering, not billable consultancy. Re-read that line whenever you are tempted to say yes to a bespoke customer-specific deliverable that will never ship into the product.

## Role 5 — Partner Engineer / Technology Partner Engineering

**Primary goal.** Enable *another company's engineers* (a reseller, integrator, ISV, hyperscaler marketplace partner) to build against the product.

**Primary artifact.** Partner-ready integration, co-engineered reference architecture, partner enablement materials.

**Where you touch them.** Partner Engineering works with *partners*, not end customers. An FDE engagement that passes through a partner (e.g., a system integrator who is delivering the pilot) has the Partner Engineer on the partner-relationship side and the FDE on the pilot-delivery side.

**Sharp boundary.** Your customer is the *end customer* — the enterprise whose workflow is being automated. The Partner Engineer's customer is a *partner firm*. You do not accidentally take on partner-enablement work (training a system integrator's engineering team, writing a reference integration a dozen SIs will use) as FDE work — package that up and route it to Partner Engineering.

## Role 6 — Developer Advocate (DevRel) / Developer Relations Engineer

**Primary goal.** Serve the external developer ecosystem at breadth — documentation quality, example quality, community engagement, external credibility, inbound developer interest.

**Primary artifact.** Public writeups, example repositories, conference talks, documentation improvements, open-source engagement.

**Where you touch them.** The pattern writeups the FDE produces in the field → product loop (mod-111) often inform DevRel content — but the FDE writes for product and platform engineering audiences, not public developer audiences, and the DevRel engineer converts internal artifacts into public ones.

**Sharp boundary.** **You serve one customer at depth, not an ecosystem at breadth.** You do not write public blog posts off your pilot without product approval; you do not become the public face of the product at conferences unless that is explicitly part of your role (it is not, by default, for an FDE); you do not attempt to serve the general developer community. When something you learn in the field *should* be public, you write the internal artifact and hand it to DevRel to adapt. Public credibility is DevRel's deliverable, not yours.

## Role 7 — Technical Account Manager (TAM)

**Primary goal.** Long-term technical account relationship for named enterprise accounts, bridging support, platform engineering, and the customer's technical leadership. A mix of CSE and SE for post-sales.

**Primary artifact.** Technical health of named accounts; customer-side escalation pathway.

**Where you touch them.** TAMs exist most clearly at hyperscalers and enterprise SaaS platforms. An FDE engaging with a hyperscaler customer may be operating alongside a TAM from the hyperscaler's side.

**Sharp boundary.** The TAM owns the broad technical account relationship. The FDE owns a specific pilot inside that account. **You defer broad-account technical questions to the TAM** ("this customer is unhappy with our overall platform's latency SLO" is a TAM question, not an FDE question).

## Role 8 — Field CTO / Field Engineering Leader

**Primary goal.** Senior-level technical narrative with strategic accounts; executive-to-executive technical trust.

**Primary artifact.** Executive briefings, architecture reviews at the VP+ level, strategic account steering.

**Where you touch them.** For very large design-partner engagements, a Field CTO (or equivalent) is the senior technical voice to the customer's CTO / CIO. The FDE is the engineer-level voice.

**Sharp boundary.** You are not the senior technical narrative. You are the engineer who ships. When a customer VP asks for a strategic-roadmap conversation, you escalate — to the Field CTO if your org has one, to your engineering manager otherwise, to the product team if the question is roadmap-specific. See mod-112 for how to handle these exec conversations when you *are* the senior voice in the room.

## The composite map

Visualise the engagement lifecycle with the roles overlaid:

```
Deal cycle                | Pilot cycle              | Steady state
--------------------------|--------------------------|--------------------------
AE drives deal            | AE stays on commercial   | AE on renewal / expansion
SE runs POC / demo        | (SE sometimes hands off  | --
                          |  to FDE at signature)    |
--                        | FDE owns delivery        | FDE hands off to CSE / CSM
--                        | CSM on relationship      | CSM owns health + renewal
--                        | Field CTO on exec arc    | Field CTO on strategic
                          |  (large accounts)        |  arc
Partner Eng on partner    | Partner Eng on partner   | Partner Eng on partner
PS on bespoke SOW         | PS on bespoke SOW        | PS on bespoke SOW
                          |  (routed from FDE)       |
DevRel on ecosystem       | DevRel on ecosystem      | DevRel on ecosystem
TAM on broad-account      | TAM on broad-account     | TAM on broad-account
 technical                |  technical               |  technical
```

Three things fall out of this picture:

- **Only the FDE is primarily measured by what runs in the customer's production after the deal is signed.** Every other role on the diagram has a different primary measure. That is why the role is distinct, and why it is tempting to drift into one of the other roles' measures.
- **Each adjacent role is a potential point of overload.** When the AE needs help on an RFP, the SE is on PTO, the CSM is in a renewal push, Professional Services is understaffed — the FDE will be asked to cover. The discipline is to cover briefly where it helps the account, and to say no out loud when covering has become routine.
- **Handoffs are explicit.** The FDE → CSE/CSM hand-off is a documented event, not a drift. See mod-109 (deployment and change management) and mod-112 (hand-off craft).

## The sharp boundaries this track enforces

Pulling the "sharp boundary" cells from each role into one list:

- **Not AE.** No quota, no price negotiation, no acting as commercial lead.
- **Not pre-sales SE.** No pre-sales demos as the primary deliverable; no inheriting prototype code wholesale.
- **Not CSM.** No account-health ownership; no silencing legitimate technical concerns for relationship reasons.
- **Not CSE (long-term).** You deliver the pilot and hand off; you are not the permanent reactive-support engineer for the account.
- **Not PS consultant.** You do not take on bespoke-only SOW work. Route it to PS.
- **Not Partner Engineer.** Your customer is the end customer, not a partner firm.
- **Not DevRel.** You serve one customer at depth; you do not write public content off pilots without product approval.
- **Not TAM.** You do not own the broad technical account relationship; you own a pilot inside it.
- **Not Field CTO.** You do not carry the senior strategic-technical narrative; you escalate.

These boundaries are not about hoarding — they are about staying in the role the business hired you to do. Every time you reach across one of these boundaries, you are not doing your own job; and someone else's organisation is being deprived of its practice.

## Summary

- "Customer-facing engineering" is a cluster of 6–8 distinct roles, each with its own incentive structure and primary artifact.
- The Forward Deployed Engineer is distinct from Account Executive, Solutions Engineer, Customer Success Manager, Customer Success Engineer, Professional Services Consultant, Partner Engineer, Developer Advocate, Technical Account Manager, and Field CTO.
- The FDE is the only role whose primary measure is *what runs in the customer's production after the deal is signed, and what it teaches the product*.
- Every adjacent role has a specific handoff with the FDE; every adjacent role is a specific point of overload; every adjacent role has a specific sharp boundary this track enforces.
- When you find yourself routinely acting as one of the adjacent roles, you have drifted off-role. Reset to the four stances in Chapter 02.
