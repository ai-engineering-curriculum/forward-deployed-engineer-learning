# Chapter 01 — SME-Paired Customer Discovery

## Why this chapter exists

Mod-101 named the four stances of the Forward Deployed Engineer: embedded delivery, no-ego engineering, design-partner-first, and the field → product loop. This chapter operationalises the first two inside the first phase of an engagement — customer discovery. It teaches how an FDE sits with a subject-matter expert (SME) and, through interviewing, observation, and lightweight process mining, builds the ground truth that every later artifact depends on.

Discovery is not a prelude to engineering. Discovery *is* engineering. The output of this chapter is a requirements document written by the FDE by observing the work happen — not a requirements document written by a stakeholder and handed down. If you skip this chapter, every downstream deliverable (the pilot-scope document in Chapter 04, the design-partner commercial in Chapter 05, the governance coordination in Chapter 06) is built on a guess.

Three craft traditions converge on this chapter: Teresa Torres's continuous-discovery interviewing, Jeff Patton's user story mapping, and workflow process-mining from the business-process-management community. The FDE picks from all three — none of them was invented for AI pilots, and none is quite enough on its own.

## Discovery is a deliverable, not a prelude

The commonest early-career failure of an FDE is to start building before discovery finishes — because building feels like progress and discovery feels like talking. The pilot ships. The sponsor says "this does not solve the actual problem." The FDE discovers, in retrospect, that the SME has a workflow habit no one wrote down, that the data assumed clean is 40% junk, and that the KPI the sponsor cares about is the third-most-important metric to the operator.

The fix is to treat discovery itself as a dated, signed deliverable — the discovery brief described in mod-101 Chapter 05 Deliverable 1. The brief has a drop-dead date (end of week 1 or 2), a sponsor who signs it, and a recommendation that commits the engagement to the next step (scope a pilot / do another week of discovery / decline). Without a dated deliverable, discovery expands to fill the quarter. With one, it compresses to the shape the pilot needs.

The budget for discovery on a 12-week pilot is roughly 10-20% of total engagement time — enough to produce a signable brief, not enough to be mistaken for a research project. On a 6-week pilot it is a single week with the sponsor's full attention. On a 26-week regulated-vertical engagement it may be 3 weeks with multiple SME shadows.

## Teresa Torres — continuous-discovery interviewing

The most disciplined framework for ongoing discovery conversations with real users is Teresa Torres's [Continuous Discovery Habits](https://www.producttalk.org/continuous-discovery-habits/). Three ideas carry most of the weight for the FDE.

**Weekly cadence with your users.** The FDE commits to at least one user-touch per week for the whole pilot — not a one-time discovery phase followed by silence. In the first two weeks that touch is a 60-minute observation plus a 30-minute debrief. Later it drops to a 30-minute check-in. The cadence is what produces the "they changed their quarterly target in week 6" signal that would otherwise land as a post-mortem surprise.

**Specific-past-behaviour stance.** You do not ask "would you use a draft-reply feature?" You ask: "Tell me about the last time you wrote a response to a priority-1 claim. Walk me through what you did." Torres frames this as the most reliable way to reduce the gap between what users say and what users do. Hypothetical questions produce aspirational answers. Past-behaviour questions produce observable facts.

Concrete swaps:

| Hypothetical (do not ask) | Specific-past-behaviour (ask) |
|---|---|
| "Would it help if the system flagged urgent cases?" | "Tell me about the last three urgent cases you handled this week. Walk me through how you knew they were urgent." |
| "How often does X happen?" | "When was the last time X happened? And the time before that?" |
| "What information do you need to decide?" | "Pull up the last case you decided on. Show me every screen you opened." |

**The opportunity-solution-tree.** The output of weekly interviewing is not a list of feature requests. It is a tree with the desired business outcome at the root, discovered opportunities (user needs, pain points) in the middle, and solution candidates as leaves — with experiments branching under the leaves. See [Torres's opportunity-solution-tree primer](https://www.producttalk.org/opportunity-solution-tree/). Chapter 02 shows how the FDE uses the tree to decide which opportunities become candidate automation surfaces.

Interviewing is not survey research. A survey tells you what a hundred people claim to do. An interview with observation tells you what one person actually does. The FDE takes the second over the first every time, because the pilot will ship against *one* workflow in *one* customer's environment — and the long tail of real behaviour is where pilots fail.

## Jobs-to-be-done — the complementary lens

Continuous-discovery interviewing teaches *how* to interview. Jobs-to-be-done (JTBD) teaches *what to ask about*. Clayton Christensen's [HBR piece "Know Your Customers' 'Jobs to Be Done'"](https://hbr.org/2016/09/know-your-customers-jobs-to-be-done) frames every workflow as a user "hiring" a tool to do a job. Anthony Ulwick's [Outcome-Driven Innovation](https://strategyn.com/outcome-driven-innovation-process/) sharpens the framing into measurable desired outcomes.

For the FDE, the useful minimum is one or two JTBD-flavoured questions mixed into the specific-past-behaviour interview:

- *"When you did that step, what job were you hiring your current tool to do? What would success have looked like?"*
- *"What forces push you toward the current way of doing this? What pulls you toward a different way? What makes you anxious about switching? What habits keep you doing what you do?"*

The push/pull/anxiety/habit framing — popularised in [Intercom's free JTBD book](https://www.intercom.com/resources/books/intercom-jobs-to-be-done) — surfaces the real competition for your pilot. The pilot is not competing with a blank slate. It is competing with the existing spreadsheet, the existing escalation playbook, the existing email thread, and the SME's habit of distrusting automated decisions after a bad experience two years ago. If you cannot name the forces, you will not design around them.

JTBD is not a full methodology for the FDE. It is a lens for one or two questions per interview.

## Jeff Patton — user story mapping

An interview gives you fragments. A story map turns fragments into shape.

Jeff Patton's [*User Story Mapping*](https://www.oreilly.com/library/view/user-story-mapping/9781491904893/) (and the free summary at [jpattonassociates.com](https://jpattonassociates.com/the-new-backlog/)) describes a visual technique for laying a workflow out horizontally — left to right in the actual order the user performs it — and then stacking alternative paths, edge cases, and release slices vertically below each step. The horizontal row is the "narrative flow" or backbone.

For the prior-authorisation review workflow at a mid-sized health-insurance payer, a simplified story-map backbone might look like this (text form; in practice draw on sticky notes or a whiteboard):

```
Backbone (narrative flow):
  1. Intake   -> 2. Triage    -> 3. Clinical review   -> 4. Draft decision   -> 5. QA review   -> 6. Send decision
     |              |                 |                     |                       |                |
     ----  cases below each step: typical case / urgent case / appeal / missing-info case ----
```

Vertical slicing below each backbone step shows the long tail — the typical case, the urgent case, the appeal, the missing-information case, the edge case where the member's plan does not match the formulary. Each is a row of cards.

The map earns its keep in three ways for the FDE:

1. It makes the workflow visible to the sponsor and the SME in the same room. People argue about workflows when they are words; they align when they are cards on a wall.
2. It makes the *scope* decision visible. A pilot that touches steps 3-4 is a different pilot from one that touches steps 1-6. The non-goals section of your pilot scope (Chapter 04) comes directly from the parts of the map you deliberately leave out.
3. It makes the "which step do we automate?" conversation structured. Each backbone step is a candidate intervention point. Chapter 02 picks among them.

Patton's point that is easy to miss: *a story map is not a backlog*. A backlog is a flat list that loses the relationship between stories. The story map keeps the relationship. That matters for the FDE because the pilot's user stories (Chapter 04 section 2) are *slices* of the map, not a flat list of tickets.

## Workflow process-mining — two flavours

Interviews and story maps give you the shape the SME remembers. They do not give you the shape the data reflects. Process-mining — treating the event log of the system of record as the source of truth about what actually happened — closes that gap.

There are two flavours the FDE uses:

**Qualitative process mining** is shadowing. You sit next to the SME for a 90-minute block, you time-box every step, you capture every handoff, and you note where the SME waits, where the SME decides, where the SME types, and where the SME re-reads the same thing from two different systems. Capture into a simple table:

| # | Activity | Starts at | Ends at | Duration | Mode (wait/decide/type/switch) | Notes |
|---|---|---|---|---|---|---|
| 1 | Open inbound queue | 09:00 | 09:01 | 00:01 | type | queue is in internal portal; search broken for 2 weeks |
| 2 | Read case detail | 09:01 | 09:04 | 00:03 | decide | opens 3 tabs: EHR view, formulary PDF, member chart |
| 3 | Pull member benefits | 09:04 | 09:06 | 00:02 | switch | copies member ID manually, no deep link |
| 4 | Decide pre-auth criteria | 09:06 | 09:12 | 00:06 | decide | uses mental shortcut: "if drug X and plan Y, usually approve" |
| 5 | Draft response | 09:12 | 09:19 | 00:07 | type | pulls last week's similar case and edits |
| 6 | Click send | 09:19 | 09:20 | 00:01 | type | no second sign-off because SME is senior |

Six rows is enough to show the pattern. Shadow five cases and the picture sharpens: step 3 is a tab-swapping cost the SME has internalised as "just the job," step 5 is a drafting cost that is actually editing of a prior template, step 4 has a mental shortcut the SME cannot articulate until step 5 reveals it.

**Quantitative process mining** is extracting a longitudinal event log from the system of record and mining the structure. The mature practice is documented in the [IEEE Process Mining Manifesto](https://www.tf-pm.org/resources/manifesto) and in Wil van der Aalst's textbook [*Process Mining: Data Science in Action*](https://link.springer.com/book/10.1007/978-3-662-49851-4). The common commercial tool is [Celonis](https://www.celonis.com/process-mining/what-is-process-mining/); the open-source counterpart the FDE typically reaches for is [pm4py](https://pypi.org/project/pm4py/).

An event log is a tuple per event: *(case_id, timestamp, activity, resource, optional attributes)*. Given a few hundred cases, a mining pass yields the handoff matrix, the dominant process variants (the top N case paths by frequency), the median and p90 case duration, the activity-level wait times, and the rework rate (cases where the same activity repeats — a signal of a missing precondition upstream).

Two constraints the FDE always checks before starting a quantitative mining pass:

- **Access and consent.** The event log is customer data. The FDE pulls an extract under the DPA, keeps it in a sanctioned environment (customer VPC or a zero-retention sandbox), and does not copy identifiers out. If in doubt, defer to the data-access clauses of the design-partner agreement (Chapter 05).
- **Read-only.** The pilot's quantitative discovery does not modify the source system. The extract is a snapshot.

Quantitative mining is not a substitute for shadowing. It tells you what the system *recorded*; shadowing tells you what the SME *actually did*. A step the SME performs outside the system (a phone call, a Teams message, a Post-it note) is invisible to the event log. The two together are the triangulation.

## The observe-first-code-later discipline

Three anti-patterns kill discovery faster than anything else. Learn to recognise them in yourself.

**The "we already know the problem" meeting.** The AE, the product sponsor, and the customer VP have a shared story about what the pilot should be. They repeat it in the kickoff. If the FDE agrees in the kickoff and skips observation, the pilot gets built against the shared story — which is usually a plausible but incomplete model of the actual workflow. *The fix:* say in the kickoff "we will schedule two SME shadows and come back with the brief." Not as a request for permission; as a statement of practice.

**Demo-driven discovery.** Someone on the team has a prototype. The team walks the SME through the prototype. The SME nods politely. The team interprets the nodding as validation. In fact the SME has adapted their workflow description to the prototype on the spot. *The fix:* no prototypes shown to SMEs until the discovery brief is signed. If a prototype is unavoidable (a sales motion), show it with the explicit frame "we are not asking whether you like this; we are asking what you do today, and we will decide afterward whether this fits."

**AE-filtered spec.** The spec reaches the FDE as "the customer wants automated draft replies." The FDE builds against it. Six weeks in, the SME says "we do not actually need drafts; we need the system to tell us which cases to pick up first." Both can be true for different SMEs, or the AE's summary can be wrong, but the FDE treats AE-filtered input as a *starting hypothesis to test* in discovery, never as a spec to build to.

The practical shape of the first two weeks of a pilot for an FDE living this stance:

- *Week 0 end:* kickoff with sponsor + AE + customer SME lead. FDE commits to a dated discovery brief.
- *Week 1:* two 90-minute SME shadows, one with a senior SME, one with a mid-level. One 45-minute interview with the sponsor on business KPI. Capture tables, notes, candidate questions.
- *Week 2 early:* one data-access check — can the event log for the last 30 days be exported to a sanctioned environment? One quantitative mining pass on the extract. One re-interview with the SME to reconcile what the log shows against what the SME said.
- *Week 2 end:* draft discovery brief. Share with sponsor. Walk through in a 30-minute meeting. Revise. Sign.

Not every pilot can afford all of this. Compressing is fine. Skipping — building without shadowing — is where the discipline breaks.

## SME pairing as the unit of production

An FDE does not have "the customer" as a counterpart. An FDE has one or two named SMEs per workflow surface. Everything else — the sponsor, the exec, the AE, the CSM — is support staff relative to the SME pairing.

What a named SME pairing looks like:

- **Named.** The SME is a person with a name, a role, and a photograph in your notes. "Mariana, senior clinical reviewer, prior-auth team, 11 years at the payer, member of the committee that defined the current rubric." Not "a clinical reviewer."
- **Named stake.** You have asked directly what the SME gets out of the pilot working, and what they lose if it fails. "If this ships and lands, I spend less time on the easy cases and more time on appeals, which is the work I was hired for." Or: "If this ships and lands, my role shrinks, and I need the sponsor's commitment that the new scope is bigger than the old one." The second is a change-management signal (Chapter 06); the first is a champion signal.
- **Named time commitment.** The SME is contractually committed (via the design-partner agreement; see Chapter 05) for a specific weekly hour budget. 2-4 hours a week is typical; in a labelling-heavy pilot it reaches 8. Without the number in writing, the SME's time disappears as the pilot heats up.
- **Backup.** One named backup SME. The primary will take PTO, go on leave, or change roles mid-pilot. If you discover that mid-pilot, you lose a month.

A diagnostic: *at the end of week 2, can your primary SME name you, your role, and what you are building, without prompting?* If not, you are not embedded. Fix the shadowing cadence and the communication rhythm before you draft the pilot scope.

## What SMEs say vs what SMEs do

The single most useful observation the FDE will make across the first two weeks is where the SME's description of their work and their actual work diverge. The divergence is never disingenuous. The SME has internalised their habits to the point that they no longer see them. Discovering and documenting the divergence is the FDE's contribution.

A short anonymised example. A clinical reviewer says in interview: *"I start every case with the formulary guidance."* On shadow, you observe the reviewer open the inbound queue, glance at the drug name, and decide (correctly, for the obvious cases) within 20 seconds without opening the formulary at all; the formulary only comes out for the edge cases. The reviewer's self-description is not false — the formulary *is* the ground-truth source — but the pattern is "pattern-match first, formulary only for the hard cases." A pilot built against the self-description would optimise formulary lookup. A pilot built against the observation would optimise edge-case detection. These are different pilots.

Capture the divergences in the discovery brief, with the self-description, the observed behaviour, and your recommendation for which the pilot should be built against. The SME, reading the brief, will often confirm the observation and sometimes update their own understanding of their own work. That is one of the ways the FDE earns the engineering shadow role the discipline requires.

## Summary

- Discovery is a signed, dated deliverable (the discovery brief), not a prelude. Budget it; produce it; move on.
- Teresa Torres's continuous-discovery interviewing supplies the weekly cadence, the specific-past-behaviour stance, and the opportunity-solution-tree shape; survey research is not a substitute.
- Jobs-to-be-done supplies one or two interview questions that surface the real competition for your pilot (habits, prior tools, anxieties about switching).
- Jeff Patton's story mapping turns interview fragments into a workflow backbone the sponsor and SME can argue over in one room and the FDE can slice into scope and non-goals.
- Workflow process-mining comes in two flavours: qualitative shadowing with a time-boxed activity table, and quantitative event-log extraction + mining with tools like Celonis or pm4py. Do both; they triangulate.
- The observe-first-code-later discipline kills three anti-patterns: the "we already know the problem" kickoff, demo-driven discovery, and the AE-filtered spec.
- SMEs are named, with named stake, named time commitment, and named backup. If the SME cannot name you by week 2, you are not embedded.
- The most valuable artifact from discovery is often the documented divergence between what SMEs say and what SMEs do.
