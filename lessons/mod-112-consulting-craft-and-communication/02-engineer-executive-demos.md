# Chapter 02 — Engineering Executive Demos — Narrative, Staging, Failure-Recovery

## Why this chapter exists

The executive demo is the single artefact that most compounds across the vendor's portfolio. A demo that lands produces a quote the AE re-uses for a year, a reference the CSM books for three subsequent prospects, a template the next FDE adapts, and a VP who protects the pilot's budget. A demo that misses — the FDE opened with a feature tour, a hallucination went unaddressed in front of the COO, the live model was slow and the FDE had not rehearsed — is political debt the pilot carries for months.

The executive demo is distinct from the pre-sales **solutions-engineer demo** that [mod-101 Chapter 04](../mod-101-fde-discipline-and-position/04-adjacent-customer-facing-roles.md) scoped to the SE role. The SE's demo is longer, capability-oriented, sandbox-controlled, engineered for *reach* across prospects the SE may not meet again. The FDE's executive demo is the opposite shape: six minutes of hard content on exactly one workflow the customer has committed to, running on the actual pilot environment with the customer's own consented data, for a VP the FDE will meet again next week and whose SME lead the FDE will meet this afternoon. The scrutiny is higher by an order of magnitude. And the **no-fake-demo rule** is absolute — the artefact produced in this room leaks into runbooks, case studies, and internal training, and a faked demo is a technical-debt bomb that detonates months later when a different audience asks "we saw this working in the April demo, why does it not work now?"

This chapter engineers that six minutes. Narrative, staging, failure-recovery — in that order.

## The three narrative hooks

### Hook 1 — Open from the customer's problem, not the vendor's capability

The first 45 seconds of the demo are spoken in the customer's vocabulary. Not the vendor's. Not "our agent framework can orchestrate" — the customer's **actual phrases**, lifted from discovery interviews ([mod-102](../mod-102-customer-discovery-and-pilot-scoping/)) and the SME lead's language.

```text
WRONG (vendor-opening):
"Today I'll show you our retrieval-augmented generation pipeline and
agentic workflow orchestration running on the Bedrock gateway..."

RIGHT (customer-problem-opening):
"When a member calls about a specialist referral, your service rep
has 90 seconds to find the answer in a 400-page benefits document
or the member hangs up. The team told us this is the single most
common after-call-work driver on the queue. Today I'll show you
what a rep sees when the pilot handles that call."
```

The vendor's capability is implicit in the fact that the workflow runs; it is never a sentence the VP has to parse. If the VP cannot narrate what the demo is about after the first 45 seconds in their own words, the opener was wrong.

Archetype-specific openers:

| Archetype | First-45-second opener |
|---|---|
| A — Healthcare payer | "A member calls about a specialist referral and the rep has 90 seconds in the benefits document before the member hangs up." |
| B — Federal agency | "A policy analyst drafts a response to a congressional inquiry and spends four hours pulling the governing OMB memos manually before writing the first paragraph." |
| C — EU retail bank | "A customer-support agent in Dublin handles a complaint about a disputed SEPA payment and has to read three legacy case notes in two languages before replying." |

Each opener is **a sentence the customer's own SME lead has said in a discovery interview.** The FDE has that sentence written down, dated, and attributed. The authenticity of the opener is the first credibility signal.

### Hook 2 — Before / after, with explicit metrics and an SME quote

The demo's structure is three acts: before-state, the shipped workflow, after-state delta. The before-state is **not** a slide of generic pain points. It is one metric the sponsor signed off on at scoping, plus one verbatim SME quote.

```text
BEFORE-STATE (slide or spoken):
  Average handle time on specialist-referral calls, Q3 2026:
    p50 = 4m 12s, p95 = 11m 40s (n = 1,247 calls)
  SME lead (Member Services Floor Lead), discovery interview 2026-08-14:
    "I tell new reps to just read the policy aloud to the member
     if it takes too long — we lose five minutes a call that way."

WORKFLOW (live):
  [6 minutes of the shipped pilot handling an actual de-identified case]

AFTER-STATE:
  Average handle time on specialist-referral calls, pilot cohort Oct 2026:
    p50 = 2m 48s, p95 = 7m 10s (n = 342 pilot calls)
  Policy-citation rate: 94% (eval set n = 180, gold-set reviewed by SME)
  Member CSAT delta: +0.3 (5-point scale, n = 118 post-call surveys)
```

Three disciplines govern the before/after:

1. **The before-state metric is one the sponsor committed to at scoping** — not one the FDE picked after the fact to make the delta look good. [Mod-102](../mod-102-customer-discovery-and-pilot-scoping/)'s success criteria are the source of truth.
2. **The after-state metric uses the same measurement protocol.** Same funnel definition, same window shape, same exclusion rules. If the measurement changed between before and after, the demo says so out loud. No silent methodology swaps.
3. **The sample size is stated.** "n = 342" is a sentence. A delta without an n is theatre.

### Hook 3 — The SME third-party credibility hook

The vendor's engineer saying "this works well" carries roughly zero weight with a VP who has been sold to for twenty years. The customer's own SME lead saying "I used to spend Fridays doing this; now I spend Fridays on the escalations that actually need me" carries weight the vendor cannot manufacture.

Rules for SME quotes in an exec demo:

- **Verbatim.** The FDE does not paraphrase. The quote is lifted word-for-word from a discovery interview or a weekly review and read aloud.
- **Attributed to a role, not an individual.** "The SME lead on the member-services floor" — not "Maria" — unless Maria has explicitly consented to being named *and* her director has signed off. Default to role-attribution.
- **Consent recorded.** Written consent from the SME, via the sponsor, that the quote may be used in internal executive audiences. If the demo is being recorded, the consent covers the recording. Re-use for a case study is a separate conversation — almost always with the customer's communications office. The FDE does not stretch an exec-demo consent into a case study without re-checking.
- **One quote per demo.** More than one and the quote stops being a hook and starts being a performance.

```text
PATTERN:
"The SME lead on the member-services floor — the person who has been
reviewing the pilot's responses against the benefits document for the
past six weeks — told us at last Thursday's review:

  'For the first time, I'm not the bottleneck on specialist-referral
   calls. The reps are landing the answer on their own and only
   escalating to me when it's an actual clinical judgement call.
   That's what I thought this was supposed to do.'

That is the single sentence that tells us this pilot is working on
the ground."
```

The quote is read; a one-beat pause follows; the FDE continues. The VP remembers the quote; they rarely remember the capability.

## Demo staging craft

### Environments — prod-like, not prod

The executive demo never runs on the customer's production system with real customer data. Three reasons: (a) a hallucination in front of a VP on real member data is a reportable incident for Archetype A, a CUI exposure for Archetype B, and a DORA-reportable event for Archetype C; (b) demo latency should not couple to production workload variability; (c) the audit trail of "the vendor's FDE ran a prompt against a real member record for a demo" is not an audit trail the customer wants.

The demo runs on a **prod-like environment** — same model, same gateway, same retrieval index, same tools — with **consented scenarios** from one of three sources:

- Synthetic cases the SME has reviewed and approved as realistic.
- De-identified real cases signed off by the privacy office (Archetype A) or the records-management office (Archetype B).
- Explicitly consented real cases (rare; usually staff volunteering their own cases).

Scenarios are **time-bounded** — the same 3–5 cases every time. The FDE does not improvise a new case in front of a VP. Improvisation is the highest-probability source of an unrecoverable failure.

### Fall-back paths — recorded only when the live model is genuinely flaky

Default is live. A recorded Loom is used **only** when the live model is genuinely flaky — a provider-side incident, an index rebuild, a tool outage. If a recording is shown, it is **clearly labelled as recorded** on-screen the moment it plays, and the FDE says "this is a recording from Tuesday because our gateway is currently in a planned failover" out loud. Never play a recorded reply as live.

### Latency rehearsal — three dry runs in the last 24 hours

The FDE has run the exact demo prompts three times in the 24 hours before the demo — not the staging environment last week; the exact environment, the exact prompts, three times. The FDE knows p50 and p95 per prompt, whether any prompt has exceeded 10 seconds, and which retrieval calls hit a cold vs warm index.

Latency rehearsal lets the FDE **narrate during the wait**. "The agent is fetching the benefits section — this typically takes three to four seconds" converts a latency gap into a credibility moment. Silence during a 7-second wait is a credibility gap.

### The stage-manager pattern — one narrates, one operates

Demo craft is a two-person job. The FDE drives the narrative. A second person — another FDE on the engagement or the `ai-enablement-lead` — operates the system. Handoff cues are scripted.

```text
SCRIPTED HANDOFF (healthcare payer archetype):

NARRATOR (FDE):  "Now the rep sees the member calling about the
                 specialist-referral question — let's trigger the
                 inbound-call summary."

OPERATOR:        [Clicks the trigger; the summary generates live]

NARRATOR (FDE):  "The summary pulls three pieces of context — the
                 member's plan, the referral history, and the
                 governing benefits section. That's showing now."

OPERATOR:        [Does not speak; stays on-screen for 2 seconds;
                 advances to the next scenario on NARRATOR's cue]

NARRATOR (FDE):  "Next: the SME-review case — where the pilot
                 escalates instead of resolving."
```

The operator never narrates. The narrator never operates. The handoff cue — "that's showing now," "let's move to," "next:" — is explicit and short. A single-person demo is a **hero demo**, a named [anti-pattern](#anti-patterns) below.

## Failure-recovery craft

Live demos fail. The question is not whether, but how the FDE responds.

### The name-it, reframe-it, continue pattern

Three beats. Name what happened in plain English. Reframe it as a known failure mode the pilot has instrumentation for. Continue the demo — or abort, per [the abort criteria](#when-to-abort-the-demo) below.

A worked example for the hardest case — a hallucination in front of the VP:

```text
NAME IT:     "That response cited a policy section that doesn't exist
              in the benefits document. That's a hallucination."

REFRAME IT:  "This is exactly the failure mode we built the citation-
              check eval for — one of twelve categories in our gold
              set. I can show you the eval dashboard that catches this
              pre-production, and the regression fixture we have for it."

CONTINUE:    "Let me run the next scenario — this one hits the SME
              escalation path, which is the control we rely on when
              citation-check flags a low-confidence response."
```

Shorter name-it / reframe-it for the other three common failures:

| Failure | Name-it | Reframe-it |
|---|---|---|
| Retrieval returns empty | "Retrieval just returned no passages for that query." | "That's a coverage gap — our retrieval eval tracks these at 3% of queries. The fallback is SME escalation rather than answering ungrounded; let me show that path." |
| Latency spike beyond rehearsed p95 | "This is taking longer than I rehearsed — ~12s where I usually see 4." | "[glances at Mod-110 dashboard] Vector-store latency is elevated. Our SLO is p95 under 8s end-to-end; we're above it. This would page on-call if it held for 15 minutes." (continue or abort per dashboard) |
| Tool-call throws an exception | "The policy-lookup tool just errored. Let me see the trace." | "Tool-call errors are handled by retry-then-escalate — the agent moved to 'escalate to SME with the raw query.' That's the designed behaviour. The SME-escalation ticket just appeared in the queue." |

The reframe leans on two artefacts: the regression-fixture library from [Mod-107](../mod-107-eval-eng-on-customer-data/) and the observability dashboards from [Mod-110](../mod-110-observability-and-oncall/). The FDE with both ready turns a failure into a credibility moment. "We have a regression fixture for this" is the single most credibility-earning sentence the FDE can say after a failure — it tells the VP this was foreseen, the pilot is engineered against it, the team is not improvising.

### When to abort the demo

Not every failure is a continue. Abort criteria:

- Two independent failures in the first 90 seconds. The environment is not healthy; continuing is a bigger risk than aborting.
- A failure that exposes non-consented customer data. Abort immediately; follow the customer's incident process.
- A failure the FDE cannot name. "I don't know why that happened" is a signal the FDE is improvising on top of an unknown fault. Abort; follow up in writing.
- A latency spike that puts the demo over the 6-minute budget and the VP has a hard stop.

The abort sentence:

```text
"I'm going to stop here — what we're seeing is outside the behaviour
I want to show you. I'll send a written follow-up with the root cause
and a re-run proposal by end of day tomorrow."
```

The follow-up email that saves the relationship — sent within 24 hours:

```text
Subject: Demo follow-up — <workflow>, <date>

Hi <VP>,

Thanks for making time this morning. I aborted at minute 3 because
the retrieval index was returning empty results — a coverage gap our
eval tracks at 3% of queries that the scenario happened to hit twice
in a row. That's a fault in my scenario selection, not in the pilot.

What happened:
- <one-paragraph root cause>
- <link to the eval dashboard entry>
- <link to the regression fixture being added>

Proposal:
- A 20-minute re-run on <date> with scenarios that have passed three
  dry runs in the 24 hours prior.
- The before/after one-pager I planned to close with — attached.
  Production AHT delta is holding at the figures shared at Thursday's
  weekly review.

<FDE>
```

Short; names the fault; proposes a re-run; attaches the KPI one-pager the demo would have closed on. The VP who receives this email has a higher opinion of the pilot's engineering discipline than one who sat through a smooth-but-faked demo.

## The no-fake-demo rule

Verbatim and uncompromising.

- **Never stage outputs.** The output the VP sees is the output the model produced for the prompt the operator sent. No pre-written strings dressed as model output. No "we edited the response for clarity" revisions on the fly.
- **Never play pre-recorded replies as live.** A recording is labelled as a recording the moment it is on screen; see [the fall-back paths section](#fall-back-paths--recorded-only-when-the-live-model-is-genuinely-flaky).
- **Never redact an obvious failure mode on the fly.** If the response hallucinates, the response is shown and named; the FDE does not scroll past it or pretend the next prompt is a retry of the same scenario when it is a different scenario.
- **Never fabricate a metric.** If the before/after number is not yet measured, the FDE says "we do not have the measurement yet — this is week 3, the baseline measurement period is 6 weeks."
- **Never substitute a different customer's success for this customer's.** A pattern from another pilot is cited only when the customer has been told, in advance, that another-pilot evidence is being shown.

The reason is not ethical theatre; it is operational. **The executive demo artefact leaks** — into the AE's pitch deck for subsequent prospects, the CSM's quarterly-business-review template, the vendor's case-study-in-draft that goes through legal six months later, the customer's own "AI pilots we are running" slide shown to the sponsor's boss, and the next FDE's onboarding packet for a similar pilot. A faked demo embeds a fault in all five. Six months later a prospect says "we saw a demo where the citation rate was 100%; our evaluation is seeing 87%," and the FDE team is explaining, publicly, why the April demo was not representative. The credibility cost outweighs the short-term win of a smooth demo by at least an order of magnitude.

The FDE who cannot produce a working live demo this week says so. They re-schedule. They do not fake.

## A concrete six-minute demo structure

Six minutes of hard content, plus 4–6 minutes of questions. The beats differ per archetype.

### Archetype A — US healthcare payer (member-inquiry)

| Beat | Time | Content |
|---|---|---|
| 1. Customer-problem opener | 0:00–0:45 | "Member calls about specialist referral; rep has 90 seconds in 400-page benefits doc." |
| 2. Before-state metric + SME quote | 0:45–1:30 | AHT p50/p95 from Q3; floor lead's "just read the policy aloud" quote. |
| 3. Live scenario 1 — resolved call | 1:30–3:15 | Member-inquiry handled end-to-end; citation shown; latency narrated. |
| 4. Live scenario 2 — escalation path | 3:15–4:30 | A case the pilot escalates to the SME (clinical judgement). The *escalation* is the point. |
| 5. After-state metric + SME quote | 4:30–5:30 | AHT delta; policy-citation rate; floor lead's "I'm not the bottleneck anymore" quote. |
| 6. One-sentence close + next ask | 5:30–6:00 | "Next week we extend to pharmacy-benefit questions; I'd like 10 minutes of your SME lead's time." |

Emphasis: the **escalation path** is the trust-earning beat. A VP trusts a pilot that knows when to escalate more than a pilot that answers everything.

### Archetype B — US federal agency (policy-analyst drafting)

| Beat | Time | Content |
|---|---|---|
| 1. Customer-problem opener | 0:00–0:45 | "Analyst spends four hours pulling governing OMB memos before writing a congressional-inquiry response." |
| 2. Before-state metric + analyst quote | 0:45–1:30 | Time-to-first-draft baseline; senior analyst's verbatim quote on manual retrieval load. |
| 3. Live scenario — drafting assistance | 1:30–3:45 | Pre-approved prompt; abstracted scenario (no live CUI); citation-grounded draft. |
| 4. Auditability view — provenance trail | 3:45–5:00 | Every citation traceable; prompt-and-response logged into the enclave's audit system per NIST 800-53 AU-family controls. <!-- needs-research: confirm specific NIST 800-53 Rev 5 AU control baselines apply to AI pilot prompt-response logging in FedRAMP Moderate enclaves --> |
| 5. After-state metric | 5:00–5:40 | Time-to-first-draft delta; analyst-review acceptance rate. |
| 6. Close + next ask | 5:40–6:00 | "Next milestone is a second mission-area; ATO artefact package attached." |

Emphasis: **auditability** is the trust-earning beat. The federal audience trusts a pilot whose every output is traceable.

### Archetype C — EU retail bank (customer-support reply + case triage)

| Beat | Time | Content |
|---|---|---|
| 1. Customer-problem opener | 0:00–0:45 | "Dublin agent handles a SEPA-dispute complaint; three legacy case notes in two languages before replying." |
| 2. Before-state metric + agent quote | 0:45–1:30 | Case-triage accuracy baseline; floor team-lead's quote on language-switching load. |
| 3. Live scenario — triage + draft reply | 1:30–3:30 | Pseudonymised real-shape case; triage category; grounded draft reply; GDPR data-handling note on screen. |
| 4. Model-risk + DORA view | 3:30–4:45 | Validation evidence linked (pointing at the model-risk function's sign-off); DORA ICT-third-party register entry visible; EU AI Act Article 72 post-market monitoring record shown. <!-- needs-research: confirm EU AI Act Article 72 post-market monitoring record shape for deployers in the bank's use category --> |
| 5. After-state metric | 4:45–5:30 | Case-triage accuracy delta; agent CSAT delta; policy-citation rate. |
| 6. Close + next ask | 5:30–6:00 | "Next quarter we extend to the Dutch team; model-risk review scheduled for 2026-12-05." |

Emphasis: **governance-evidence visibility** is the trust-earning beat. The model-risk function is in the room; the demo shows the posture on their terms.

## Role mapping at the demo

| Role | Demo responsibility |
|---|---|
| FDE | Drives the technical narrative; operates the system **only if** no stage manager is available (hero-demo risk — avoid). |
| Second FDE / `ai-enablement-lead` | Stage-manages: operates the system, advances scenarios on the narrator's cue, watches the latency dashboard. |
| AE | Opens the room if they are present; introduces the FDE; does not interject during the technical content. |
| `ai-engineering-manager` | Attends the first exec demo of a pilot by default; subsequent demos only if a political dynamic needs manager-level presence. |
| CSM | Observes; takes the follow-up actions for the customer sponsor's team. |
| Customer sponsor | Pre-briefed before the demo (see [the pre-brief section](#pre-brief-the-customer-sponsor)); introduces the FDE to the VP audience; does not narrate the technical content (that is the FDE's job). |
| Customer VP(s) | The audience. The entire demo is engineered for their 6-minute attention. |
| Customer SME lead | Attends if possible; speaks only if the VP asks a question specific to their role; their presence is itself a credibility hook. |

Role boundaries at the demo match [mod-101 Chapter 04's](../mod-101-fde-discipline-and-position/04-adjacent-customer-facing-roles.md) catalogue. The FDE never answers pricing or contract questions in the room — routes to the AE. The FDE never commits to a feature not already in [Chapter 06](./06-scope-discipline-and-decision-log.md)'s decision log.

### Pre-brief the customer sponsor

24 hours before the demo, a 15-minute conversation with the sponsor covers: the six-minute structure, the scenarios that will run (no political landmines), the before/after metrics (aligned with committed success criteria), the SME quote (consent confirmed), and the follow-up ask (no surprise in the room). The sponsor who is surprised by any element of the demo is a sponsor who has been failed by the FDE.

## Running the three archetypes

**Archetype A — HIPAA audience staging.** The de-identification protocol is pre-signed-off by the customer's privacy office, which attends the demo as a standing attendee. Scenarios use the same protocol every time — no new de-identification in the 48 hours before the demo. SME quotes are role-attributed by default; only names appear if explicitly consented. The demo's prompts log into the customer's HIPAA audit system; the FDE confirms this logs cleanly as part of the dry run. <!-- needs-research: confirm HIPAA Privacy Rule 45 CFR §164.514 Safe Harbor method covers the demo's de-identification protocol -->

**Archetype B — FedRAMP audience staging.** The demo is **abstracted** — no live CUI, no classified content, no live calls to the agency's actual data stores. Prompts are pre-approved at least 72 hours in advance by the agency's records-management or information-security office. The environment is the agency's own ATO-boundary pilot enclave, not a vendor sandbox. If a scenario would produce output that could be labelled CUI, the FDE uses a synthetic equivalent and names it as synthetic. Improvisation in a federal demo can produce an ATO-impacting incident. <!-- needs-research: confirm which agency office approvals are required for demo prompts in a FedRAMP Moderate enclave; varies by agency -->

**Archetype C — EU bank staging.** The model-risk function is a named attendee alongside the COO and the Head of Customer Operations. The governance-visibility beat ([structure above](#archetype-c--eu-retail-bank-customer-support-reply--case-triage)) is not optional. DORA ICT-third-party visibility means the vendor is listed in the bank's ICT-third-party register; the demo environment is the bank's own Azure AI Foundry EU-Data-Boundary tenant. <!-- needs-research: confirm Azure AI Foundry EU Data Boundary coverage for the specific services used in the archetype C pilot --> The EU AI Act Article 72 post-market-monitoring record is referenced; the FDE has walked through its shape with the model-risk function in advance.

## Anti-patterns

**The fake live demo.** The FDE pre-runs prompts and plays cached responses as live; hallucinations are edited out; a latency spike is pre-empted with a saved output. In the next demo the real behaviour diverges; the VP notices; the credibility loss outstrips any demo win the fake produced. The no-fake-demo rule exists for exactly this cost.

**The untested environment.** The FDE has not run the demo prompts in the last 24 hours. A dependency has silently changed — index rebuild, provider-side version bump, tool-schema migration. The first prompt errors in front of the VP. Three dry runs in the preceding 24 hours is not optional.

**The FDE as sole narrator.** The FDE narrates **and** operates. Attention splits; narration stutters during tool-call waits; screen-advance is mis-timed; handoff cues disappear. The two-person stage-manager pattern is baseline, not luxury.

**The overlong setup.** The FDE spends the first 90 seconds on architecture diagrams and gateway topology, reaching the customer's actual problem at minute 2:30. The VP's attention is gone by 1:30. The first customer problem must be spoken by second 45.

**The ignored failure.** A hallucination appears. The FDE scrolls past it, continues to the next scenario, hopes the VP did not notice. The VP noticed. The VP now distrusts every subsequent output. Name-it, reframe-it, continue is not optional.

**The metric-free demo.** No before-state number. No after-state number. No sample size. The VP cannot report anything concrete to their own boss. Three numbers in three sentences is the minimum: a before, an after, an n.

**The leaked customer prompt.** The FDE runs a prompt that includes real customer data — a member ID, a case number, an analyst's draft on an active matter — that was not consented for the demo audience. The compliance-office follow-up begins; the pilot pays for weeks. De-identification and consent are gates the FDE does not shortcut under time pressure.

**The hero demo.** One FDE. No stage manager. No backup. No second-screen latency dashboard. No pre-brief. The demo lands on one person's performance alone; when the inevitable live failure comes, there is no team to catch it. The hero-demo shape is a credibility risk the FDE is taking on behalf of the pilot without permission.

## Summary

- The executive demo is engineered for a 6-minute window of VP attention on the customer's actual pilot. It is **not** the pre-sales SE demo — short, narrow, no-fake, customer-data-grounded.
- Three narrative hooks carry the room: open from the customer's vocabulary in the first 45 seconds; show a before/after with explicit metrics and sample sizes; use one verbatim SME quote with recorded consent and role-attribution.
- The before-state metric is the sponsor's committed success criterion from [mod-102](../mod-102-customer-discovery-and-pilot-scoping/); the after-state uses the same measurement protocol; the n is always stated.
- Staging craft is: prod-like-not-prod environments, consented scenarios, time-bounded cases, three dry runs in the preceding 24 hours, and a two-person stage-manager pattern that scripts the handoff cues.
- Failure-recovery runs the name-it, reframe-it, continue pattern; the trust-earning sentence is "we have a regression fixture for this," powered by [Mod-107](../mod-107-eval-eng-on-customer-data/)'s eval library and [Mod-110](../mod-110-observability-and-oncall/)'s dashboards.
- Abort criteria are explicit: two failures in 90 seconds, non-consented-data exposure, an unnamable fault, or a hard-stop budget overrun. The abort sentence and the 24-hour follow-up email are prepared in advance.
- The no-fake-demo rule is uncompromising because demo artefacts leak into case studies, pitch decks, and runbooks; a faked demo is a technical-debt bomb that detonates months later with the wrong audience.
- Six-minute demo structures differ by archetype: healthcare payer emphasises the escalation path, federal agency emphasises auditability, EU bank emphasises governance-evidence visibility with the model-risk function in the room.
- Role boundaries at the demo match [mod-101 Chapter 04](../mod-101-fde-discipline-and-position/04-adjacent-customer-facing-roles.md) — FDE drives narrative; AE opens the room; CSM observes; the FDE does not answer pricing, contract, or scope questions in the room.
- Eight anti-patterns to catch: the fake live demo, the untested environment, the sole narrator, the overlong setup, the ignored failure, the metric-free demo, the leaked customer prompt, the hero demo.
- Chapter 03 is next: the executive brief and the five-minute standup — the written and spoken companion to the demo, run on a fortnightly cadence.
