# Chapter 04 — Packaging Platform Requests for the Core Team

## Why this chapter exists

Chapter 02's triage and Chapter 03's pattern writeup prove the pattern *exists*. A core-team recipient — `senior-agentic-ai-engineer`, `ai-platform-engineer`, or `ai-engineering-manager` at level 40 — reads the writeup, nods, and asks the hard question: *"what specifically do you want us to build, what is the cost to us, and what are you doing in the meantime?"*

A **platform-request packet** is the answer to those three questions in writing. The lightweight **feature request** is the PM-facing cousin — same discipline, less scope. Both exist because an unanswered platform-team question is a stalled request; a request with the answers in the packet is a request the core team can decide on without a round-trip.

This chapter has four jobs. Specify the **heavyweight platform-request packet** with all seven sections and the depth-of-detail each one carries. Specify the **lightweight feature request** for signals that are product-level rather than platform-level. Specify the **submission mechanics** — where packets go, how they are triaged, how rejection and acceptance look. Specify the **two-customer rule** operationally, including how a one-customer packet ever gets through.

Platform-request credibility is an optimisation dimension from Chapter 01. Every well-shaped packet deposits credibility; every poorly-shaped packet withdraws it. By mid-year the FDE's packet-acceptance rate is a visible measurement.

## The heavyweight platform-request packet

Seven sections. One packet. Length: typically four to six pages, longer when the interface sketch is substantial. Template:

```markdown
# Platform-request packet: [short name]

**Date:** 2026-11-18
**Author:** <FDE name>
**Routed to:** senior-agentic-ai-engineer (<name>)
**CC:** ai-engineering-manager (<name>), ai-platform-engineer (<name>)
**Surface:** canary-gate / feature-flag platform
**Triage class:** platform gap
**Two-customer evidence:** satisfied — acme-health (Archetype A), bank-c (Archetype C)
**Linked pattern writeup:** writeups/2026-11-canary-prompt-only-hold.md
**Status:** Draft; requested first-review by 2026-11-25

## Section 1 — The ask in one sentence

The vendor's canary-gate preset for prompt-only bundle changes should require a
24-hour hold at 1 % traffic before any expansion, rather than the current
default 30-minute hold at 1 %.

## Section 2 — The evidence

Minimum two customer instances with distinct customer / industry / stack.
Each instance carries: the incident or pattern reference, the pilot date,
one dashboard or trace link, one SME quote with a date.

### Instance A — acme-health (Archetype A, 2026-11-04)

[Two paragraphs. The incident shape; the metric that burned; the SME quote
with date; the link to INC-42 post-mortem.]

### Instance B — bank-c (Archetype C, 2026-10-18)

[Two paragraphs. Same shape; different workflow; different cost model; shows
the pattern is not acme-health-specific.]

### Signal across the fleet

Across the twelve pilots the FDE team is running in Q4 2026, four have the
same canary-gate preset and could plausibly hit the same shape. Two have hit
it; two have not hit it yet but are at risk. Two others use a non-standard
canary cadence (manual, or the half-hour → six-hour → day pattern) and are
not at risk.

## Section 3 — The proposed interface sketch

What the change looks like at the platform's interface level. Not a full
design — a sketch the core team can read in five minutes and either agree
with or push back on.

```yaml
# Current canary-gate preset (simplified)
canary_gate_preset:
  prompt_only_bundle_change:
    - stage: 1%
      hold: 30m
      checks: [offline_eval_passed, sli_burn_below_threshold]
    - stage: 10%
      hold: 30m
      checks: [online_judge_pass_rate_above_threshold, latency_p99_below_sla]
    - stage: 50%
      hold: 2h
    - stage: 100%
      hold: null

# Proposed preset (change in bold)
canary_gate_preset:
  prompt_only_bundle_change:
    - stage: 1%
      hold: 24h  # ← change
      checks:
        - offline_eval_passed
        - sli_burn_below_threshold
        - online_judge_pass_rate_above_threshold   # ← new check at 1%
    - stage: 10%
      ...
```

The interface sketch names the specific field, the specific value, and the
specific additional check. If the sketch contains an API change, the sketch
includes the before / after of the API shape; if the sketch contains a
dashboard change, the sketch includes a before / after of the dashboard
panel.

## Section 4 — Risk and scope

Honest. The FDE's best estimate of what the core team is signing up for.

- **Scope size.** Small. The preset is a config file in the canary-gate
  service; the change is a value flip plus a new check reference. Estimated
  effort: 2–3 engineer-days.
- **Compatibility risk.** Medium. Pilots currently using the prompt-only
  preset will see longer promotion times; pilots that need the shorter
  cadence (e.g. a bug-fix prompt revert) will need an override preset. The
  packet proposes an override preset `prompt_only_urgent` that retains the
  current cadence and requires explicit opt-in per release.
- **Operational risk.** Low. The longer hold cannot cause an incident; at
  worst it extends time-to-100 %. The override preset covers the "I need to
  roll faster" case.
- **Ripple to adjacent surfaces.** The feature-flag platform's release-note
  schema (mod-109 Chapter 03) carries the canary-gate preset's name; the
  override preset introduces a new name that marketing release notes will
  need to mention.

## Section 5 — The alternative workaround currently in flight

Mandatory. The core team wants to know what the FDE is already doing — both
to understand the cost the FDE is carrying and to gauge the urgency.

Both pilots currently apply a pilot-local override that sets `hold: 24h` at
the 1 % stage. The override is maintained in each pilot's Terraform state;
cost is approximately 15 minutes per release per pilot. The override breaks
on upstream canary-gate preset upgrades (the override's schema version is
pinned to the current version and must be migrated when the upstream preset
changes).

At current Q4 2026 release cadence, the maintenance cost across the four
at-risk pilots is approximately one engineer-day per month across the FDE
team.

## Section 6 — Alternatives considered

Shows the FDE has thought about the design space, not just the one option.
Three alternatives minimum.

### Alternative 1 — Keep the override per-pilot

Cost: continuing one engineer-day / month, plus the schema-pin fragility
noted above. Benefit: no platform-team cost. The packet's proposal is
preferable because the platform-team cost is a fraction of the ongoing
pilot-level cost at four pilots and rising.

### Alternative 2 — A canary-gate flag for the hold duration, defaulting to 30m

Each pilot sets the hold duration explicitly. Preserves current default;
requires per-pilot configuration. Rejected because the default is the
dangerous value; new FDEs onboarding a pilot are likely to miss the flag.

### Alternative 3 — Infer prompt-only from diff size

The gate inspects the bundle diff and auto-promotes to the long hold when
the diff is prompt-only. Rejected because diff inspection is ambiguous
(what counts as prompt-only when the prompt-template is composed from
multiple source files?); the preset-name convention from mod-109 is the
cleaner signal.

## Section 7 — The ask in full

A specific, actionable request the recipient can act on in ≤ 30 minutes of
their own reading.

- Agree / revise / reject the interface sketch (Section 3).
- If agree, name the engineer who owns the change and the target release.
- If revise, name the dimensions the FDE should explore further.
- If reject, name the rejection reason and the alternative the core team
  recommends.

**Response requested by:** 2026-11-25
**FDE availability for a 30-min review call:** 2026-11-19 10:00-12:00 UTC,
  2026-11-20 14:00-17:00 UTC, 2026-11-21 10:00-16:00 UTC.
```

Seven sections; roughly five pages; authored over two to four hours when the capture log and the pattern writeup are both ready. The packet is where credibility compounds — a well-shaped packet makes the recipient's decision fast; a decision-fast recipient becomes a reader of the FDE's next packet.

### The core-team reader's time budget

The level-40 recipient is time-poor. The packet is read in two passes:

- **First pass** — Section 1 (the ask), Section 2.A and 2.B (the two instances), Section 7 (the ask in full). Roughly ten minutes. The reader decides whether to read on.
- **Second pass** — Sections 3, 4, 5, 6. Roughly twenty to thirty minutes. The reader decides the response.

The packet is designed for that two-pass read. Section 1 is a single sentence; Section 2 is the evidence in a shape the reader can read without clicking out; Section 7 has an unambiguous verb (agree / revise / reject) and a date. If Section 1 alone cannot stand as a summary, the packet is not finished.

## The lightweight feature request

Features are smaller than platform gaps. A feature request goes to the vendor PM, not to the level-40 peer. Length: typically one page. Template:

```markdown
# Feature request: [short name]

**Date:** 2026-11-10
**Author:** <FDE name>
**Routed to:** vendor PM (<name>) for the [surface] product area
**Pilot(s):** acme-health (potentially broader)
**Linked capture entry:** sig-2026-11-03-0008
**Status:** Draft; requested first-review by 2026-11-17

## The ask in one sentence

The assistant should cite the specific policy section it pulled from, not
just the document title, when presenting a grounded answer.

## Who is asking

The customer SME lead at acme-health in the 2026-11-03 stand-up; the
model-risk reviewer at bank-c in the 2026-09-12 quarterly review. Two
pilots, two different customer roles, same ask shape.

## Why it matters

Section-level citations shorten the SME's verification step. The SME lead
estimated the verification step at 15 seconds per inquiry when the answer
cites a section, versus 60 seconds when the answer cites only the document.
Over 2,400 member inquiries per day, the saved time is roughly 30 minutes
of SME capacity.

## What the outcome looks like

A citation format the SME can scan quickly; stable across the vendor's
grounded-answer product; configurable at the pilot level so the citation
depth (document / section / passage) can be tuned to the customer's needs.

## Alternatives the FDE has considered

- Pilot-local prompt-template change asking the model to produce section
  references in the citation. Works in 70 % of cases; fails when the
  retriever's chunking does not produce clean section boundaries.
- Pilot-local post-processing that looks up the section from the chunk's
  metadata. Works, but requires per-pilot engineering and introduces
  citation-generation latency.

A product-level change would avoid the pilot-local work and would land
across every grounded-answer customer.

## Scope and priority as the FDE sees it

Small. The grounded-answer stack already carries section metadata in the
retriever's response; the citation generator needs to lift it into the
rendered citation. Estimated effort: one to two engineer-weeks including
UX review.

The FDE would rank this as a medium-priority feature — not blocking any
pilot, but present in roughly a third of pilots' SME-wish logs.
```

Six sections; one page; authored in forty-five minutes. The feature request is where most of the capture log's `feature`-triaged entries land; the heavyweight packet is reserved for platform-level requests where the interface shape itself is the proposal.

## The two-customer rule, operationally

Chapter 01 and Chapter 02 named the rule. Chapter 04 operationalises it.

### What counts as a second customer

Not every "second pilot" is a second customer for this rule. The second instance must satisfy:

- **Distinct customer organisation.** Not a second workflow at the same customer.
- **Distinct industry vertical, or distinct stack, or distinct compliance posture.** Three customers in the same vertical with the same stack and the same compliance posture are a *concentration*, not a shape-confirmation. The second-instance test wants independent hits.
- **The same underlying shape, not just the same symptom.** Two pilots that saw "online judge pass-rate dropped" for unrelated reasons (one a prompt change, one a provider drift) are not the same shape. The pattern writeup's "why it is a pattern, not a bug" section is the test — if the writeup cannot articulate the shared architectural shape, the two instances do not confirm the rule.

### The exception path — the single-customer packet

A packet with one customer instance *can* land, but the burden is higher. The packet carries a Section 2.X titled *"Why one instance warrants consideration"*. The valid exceptions:

- **Pre-emptive risk.** The FDE can show that the shape *will* hit a specific additional pilot within the quarter based on that pilot's configuration. Example: a one-customer platform request to fix a canary-gate flaw that a scheduled pilot migration in three weeks is known to trigger.
- **Catastrophic magnitude.** The one-customer instance produced a severe outcome — a SEV-1 with a long resolution time, a compliance breach, a customer-facing message with regulatory attention. The packet names the magnitude and references the post-mortem.
- **Platform-team-visible without the FDE.** The core team has independently noticed a shape through their own instrumentation; the FDE's packet is converging with their view. In that case the FDE's "one customer" is paired with the core team's existing evidence; the packet acknowledges this.

A packet routed under the exception path is reviewed by `ai-engineering-manager` before it reaches the level-40 peer. The manager's review is the gate that keeps the exception path from being overused.

### The parked-candidate workflow

A one-customer signal triaged `platform_gap_candidate` sits in a parking queue. The weekly triage ritual (Chapter 02) sweeps the queue for second-instance matches. If no second instance arrives within two quarters, the signal is closed as `pilot_specific` with the FDE's rationale noted; if a second instance arrives, the signal is promoted and the packet is authored. The quarterly knowledge-base review (Chapter 06) is the backstop.

## Submission mechanics

Where packets go. How they move. How they are closed.

### The submission channel

Every level-40 peer owns an intake channel for platform requests. Typical shapes:

- A dedicated Linear / Jira project (`PLATFORM-REQ` or similar) with a packet template.
- A dedicated Slack channel with a bot that captures packets into the project.
- A dedicated mailing list for teams that prefer email routing.

The FDE learns the recipient's preferred channel at the first packet and sticks to it. Packet metadata lives in the project / issue; the packet itself lives in a durable doc store (the FDE's repo; vendor Confluence) linked from the issue.

### Review cadence

Platform-request packets:

- **First-pass acknowledgement** within one week of submission. The recipient responds "received, assigned to <engineer>, response expected by <date>" or "received, needs reshape — <reasons>" or "received, we will respond in the quarterly review."
- **Decision** within four weeks for straightforward packets; within a quarter for packets that require design discussion.
- **Status update** at least monthly if the packet is accepted but not yet shipped.

Feature requests:

- **First-pass triage** within two weeks, into the PM's backlog with a priority label.
- **Decision** at the next quarterly product-review meeting (Chapter 05).

The FDE's part of the cadence is: do not re-ping before the window elapses; do not escalate before the manager reviews.

### Rejection and the rejection-reasons library

A rejected packet is a data point, not a failure. The core team's rejection carries one of a small number of reasons, each with an implication for the FDE's future packets.

- **"Not platform-level — this is product-level."** The packet should be a feature request to the PM, not a platform request to `senior-agentic-ai-engineer`. Re-route.
- **"Out of scope for this surface — talk to <other level-40>."** Re-route to the correct peer.
- **"We agree, but not in this quarter — parked."** The packet is accepted in principle; the FDE maintains the workaround; the quarterly review names the parked packet.
- **"We disagree with the direction."** The core team has a different design in mind. The FDE engages, revises, or withdraws.
- **"Insufficient evidence."** The two-customer rule is not satisfied, or the exception path's burden is not met. The FDE parks and resubmits when evidence arrives.
- **"Wrong shape — the sketch doesn't fit our architecture."** The core team sees a constraint the FDE did not. The packet's recommendation is revised with input from the recipient.

The FDE team maintains a rejection-reasons library in Chapter 06's knowledge base; a new FDE reads it before authoring their first packet.

### Acceptance and follow-through

An accepted packet is not done. The FDE's job continues until the platform change ships and the FDE's pilot-local workaround is retired. The follow-through:

- **Design review.** The FDE attends the core team's design review for the change, as a consulting participant. The interface sketch is almost always revised; the FDE's input ensures the revised shape still satisfies the field.
- **Release notification.** When the platform change ships, the FDE confirms the pilot's workaround can be retired. The retirement is itself a signal — captured in the capture log with `source: platform-request-landed`, generalisability `platform_level`, and a link to the original packet.
- **Retrospective.** If the packet took more than a quarter from submission to ship, the FDE and the recipient hold a 30-minute retrospective: what slowed it? what could speed the next one? The answer feeds the Chapter 06 knowledge-base.

## Routing to the right level-40 peer

Three level-40 peers. The FDE learns which is the right recipient per surface. The default routing:

| Surface | Primary | Secondary (CC) |
|---|---|---|
| Canary-gate, feature-flag, versioning | ai-platform-engineer | senior-agentic-ai-engineer |
| Telemetry schema, cost rollups, retention | ai-platform-engineer | — |
| Agent SDK, tool-call shape | senior-agentic-ai-engineer | — |
| Judge library, guardrail library | senior-agentic-ai-engineer | ai-platform-engineer |
| Response-library (security review), trust-centre | ai-engineering-manager | ai-enablement-lead |
| FDE team process, onboarding, knowledge base | ai-engineering-manager | — |

Edge cases are routed to both and let the recipients decide between them. Cross-routing is cheap when the packet is well-shaped; cross-routing a poor packet is expensive — it dilutes credibility across two peers.

## Running the three archetypes

**Archetype A — US healthcare payer.** Packets name the pilot under customer-approved sensitivity (Chapter 03 sign-off). HIPAA-touching evidence in Section 2 is de-identified before the packet leaves the vendor-internal store. Interface sketches that touch audit-logging surfaces route with the vendor's Privacy / Security Office on CC; the interface sketch names the retention floor explicitly (six years for HIPAA-touching stores). <!-- needs-research: confirm HIPAA records retention; the 6-year Security Rule documentation requirement under 45 CFR 164.316(b)(2) is reviewed per log class -->

**Archetype B — US federal agency.** Packets originating from the agency pilot abstract the agency's workflow before leaving the enclave; Section 2 carries "US federal pilot under FedRAMP Moderate, policy-analyst workflow" rather than naming the agency. The ISSO reviews any Section 3 interface sketch that touches controlled-information handling. Packets that could not satisfy the two-customer rule because the agency pilot is the only one of its kind in the FDE team's current fleet use the pre-emptive-risk exception path with explicit `ai-engineering-manager` sign-off.

**Archetype C — EU retail bank.** Packets name the pilot under bank-approved sensitivity. Section 2 evidence that touches customer PII is pseudonymised per GDPR Article 32. Interface sketches that touch audit-logging or model-governance surfaces route with the bank's model-risk function on CC (the model-risk function may be a review stakeholder on the core team's design review). DORA-adjacent packets (anything that would affect ICT-incident classification or reporting) require the bank's DORA liaison to be a named CC. <!-- needs-research: confirm current DORA RTS on classification thresholds for major ICT-related incidents -->

## Anti-patterns

**Packet without a workaround.** Section 5 says "we haven't figured out a workaround yet." The recipient reads it and concludes the FDE has not thought enough about the problem to deserve platform attention. The workaround is always named — even if it is "we are telling the customer the behaviour is a known limitation," that is a workaround.

**Two-customer rule faked.** The FDE lists two instances but both are from the same customer under two product SKUs. The recipient catches it on review; the packet's credibility takes a hit. The two-customer rule's definition is in writing; the FDE reads it before claiming it.

**Alternatives section skipped.** The FDE proposes one direction and says "this is the right call." The recipient asks "what else did you consider?" and the FDE has no answer. Section 6 with three alternatives is mandatory.

**Interface sketch too deep.** The FDE writes the design, not the sketch. The recipient reads twenty pages and cannot find the ask. The sketch is five minutes to read; the design is the core team's job.

**Interface sketch too shallow.** The FDE writes "we need better canary-gate behaviour." The recipient asks "specifically what?" and the FDE has not thought it through. The sketch is five minutes to read because the FDE did the thinking; a sketch that is five minutes to read because it is empty is worse than useless.

**Re-ping before the window elapses.** The FDE submits on day one; pings on day three; the recipient classifies the FDE as high-maintenance. The review cadence has windows; the FDE respects them.

**Routing to all three level-40 peers by default.** The FDE sends the packet to all three "to be safe." Each recipient assumes another owns it; the packet languishes. The default routing table picks a primary; CCs are deliberate.

**Over-claiming cost.** The Section 2 "signal across the fleet" paragraph inflates the pilot-count affected. The recipient's own instrumentation does not corroborate. The FDE's credibility takes a hit. The fleet signal is a careful read of the capture logs, not an aspirational count.

**Rejection as personal.** The FDE receives a "we disagree with the direction" and reads it as the core team being unresponsive. The rejection-reasons library (Chapter 06) exists to normalise the shape; rejection-as-data is the discipline.

## Summary

- Two packet shapes: the heavyweight **platform-request packet** (seven sections, 4–6 pages, level-40 recipient) and the lightweight **feature request** (one page, PM recipient). Both answer the three core-team questions: what do you want, what is the cost, what is your workaround.
- Seven sections of the platform-request packet: the ask in one sentence; two-customer evidence; proposed interface sketch; risk and scope; alternative workaround currently in flight; alternatives considered; the ask in full.
- The two-customer rule has an operational test — distinct customer, vertical / stack / compliance posture, and the same underlying shape. The exception path (pre-emptive risk, catastrophic magnitude, platform-team-visible without the FDE) is reviewed by `ai-engineering-manager` before submission.
- Submission mechanics: dedicated channel per recipient; first-pass acknowledgement within a week; decision within four weeks (typical) or a quarter (complex); monthly status while parked.
- Rejection carries one of six named reasons, each with an implication. The rejection-reasons library lives in Chapter 06's knowledge base.
- Follow-through on accepted packets: design-review attendance, release notification, pilot-local workaround retirement, and a retrospective if the packet took more than a quarter.
- Routing table maps surfaces to primary / CC level-40 peers: `ai-platform-engineer` for infra / telemetry / cost; `senior-agentic-ai-engineer` for agent SDK / judge / guardrail; `ai-engineering-manager` for FDE-team-process / response-library.
- Archetype A requires HIPAA de-identification in Section 2; Archetype B abstracts the agency and uses the exception path when the agency is the only pilot of its kind; Archetype C pseudonymises per GDPR and CCs model-risk / DORA liaisons.
- Nine anti-patterns: packet without a workaround, two-customer rule faked, alternatives skipped, interface sketch too deep, interface sketch too shallow, re-ping too early, routing to all three, over-claimed cost, rejection as personal.
- Chapter 05 is next: the product-review meeting — how the FDE shows up prepared, with data and customer voice, proposing rather than demanding.
