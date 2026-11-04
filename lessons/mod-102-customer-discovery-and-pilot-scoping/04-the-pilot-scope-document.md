# Chapter 04 — Drafting the Pilot-Scope Document

## Why this chapter exists

Mod-101 Chapter 05 named the pilot-scope document as Deliverable 2 — the contract both sides point to when scope creep arrives, and the pre-commitment to a go/no-go decision at pilot exit. This chapter unpacks each section of that document for an LLM pilot, with concrete patterns for what good looks like, what the common failure modes are, and a small template fragment or two.

The pilot-scope document is what turns Chapter 02's signed discovery brief into an engineering contract. It is 4-8 pages. It is signed by the customer sponsor, by the customer-side operator-function head (in regulated verticals), and countersigned by the FDE's engineering manager. It is written before the serious code is written. Nothing in it is hypothetical: every section is a commitment.

The section order in this chapter matches the writing order. A pilot-scope document drafted in any other order tends to acquire mismatches — success metrics that have nothing to do with the user stories, pre-mortems that bear no relation to the success metrics, ADRs that ignore the problem statement. Write it top-to-bottom.

## 1. Problem statement

**What it is.** One or two paragraphs. The workflow, the hypothesis, the bet. No more.

**What good looks like.** A new engineer joining mid-pilot can read it and know what the pilot is. The customer sponsor can read it and recognise their own problem. A product leader three organisational levels away can read it and understand why their team is spending FDE time here.

**Common failure modes.**

- Burying the actual bet under vague ambition ("we are exploring the application of generative AI to…").
- Describing the technology rather than the problem ("we will use a retrieval-augmented pipeline with a frontier-lab model to…").
- Expanding into a mini-design-doc. If the problem statement is longer than two paragraphs, cut.

**Template fragment.**

```
Problem statement

The <customer business function> currently <workflow description from Chapter 01>.
This takes <observed time> and costs the team <observed quantified pain>.
We pilot an LLM intervention that <one-sentence hypothesis from Chapter 02
candidate statement> and bet that <specific success threshold>. Scope: the
<named workflow sub-step(s)> only; see non-goals.
```

## 2. User stories

**What it is.** 3-10 stories in Jeff Patton's shape — named user + named action + named success. Not JIRA tickets, not epics. Each story is one card on a story map. The whole set is the "release slice" you are piloting.

**What good looks like.** Each story has a user who is a real named role (not "the user"). Each story has a success condition that is observable (not "we want to improve things"). The set of stories can be read as a narrative — in the order they happen, with the before-and-after distinguishable.

A sample pattern:

```
US-01  As a senior clinical reviewer, when I open a priority-1 case,
       I see the LLM-drafted response pre-populated in the draft field so
       that I can edit and send without opening the response template.
       Success: median time-from-case-open to draft-ready drops from
       ~7min (baseline, Chapter 01 shadow) to <3min.

US-02  As a senior clinical reviewer, when the LLM draft is missing a
       required citation to the formulary rule, I see a UI warning before
       I click Send. Success: zero drafts sent without citation in
       sampled audit rows.
```

**Common failure modes.**

- Stories that are "the model will…" rather than "the user will…". The user is the subject; the model is a tool the user uses.
- Stories that duplicate the success-metrics section. User stories describe the interaction; success metrics describe the measurement.
- Stories that number into the teens. If you have 20 stories you are scoping a quarter of work, not a pilot.

Reference Patton's [*User Story Mapping*](https://www.oreilly.com/library/view/user-story-mapping/9781491904893/) and the [jpattonassociates.com](https://jpattonassociates.com/the-new-backlog/) summary for the narrative-flow discipline.

## 3. Non-goals

**What it is.** An explicit list of what the pilot does NOT do. The most under-used and highest-leverage section in the document.

**What good looks like.** The non-goals are specific, not general. They are the exact things the sponsor, an SME, or a well-meaning AE might reasonably ask for during the pilot — and they are called out as *deliberately excluded* in advance.

Patterns of non-goals the LLM-pilot FDE routinely writes:

- *This pilot does not take action on behalf of the user; every model output is reviewed by the SME before any downstream effect.* (Most LLM pilots need this.)
- *This pilot does not integrate with the back-office reconciliation step; the output of the pilot is written to <draft folder/queue>, and downstream processing is unchanged.*
- *This pilot does not expand to the <adjacent case type>; appeals, prior-denial reversals, and missing-information returns are out of scope.*
- *This pilot does not provide a customer-facing chatbot or any end-user-visible AI UI; model use is internal to the SME tool.*
- *This pilot does not re-train or fine-tune a model on customer data; the pilot uses off-the-shelf models with prompt + retrieval.*
- *This pilot does not replace the formulary system of record; the model consumes formulary data via a read-only query.*

**Common failure modes.**

- Vague non-goals ("we won't scope-creep"). Specific or it does not count.
- Non-goals that duplicate goals in negative form ("we will not fail to deliver the primary metric"). Non-goals are *about scope*, not about commitment.
- Missing the one everyone will ask for. If you are already dreading the request, write it into non-goals explicitly.

Every FDE pilot is scoped as much by its non-goals as by its goals. When scope creep arrives — and it will — the non-goals section is the artifact the FDE points to. A scope change is a countersigned amendment to this section, not an email thread.

## 4. Success metrics

**What it is.** The quantitative definition of "this pilot succeeded." Four tiers:

- **Primary** — ONE business-level metric linked directly to the customer's sponsor KPI. The number the pilot is bet on.
- **Secondary** — 2-3 operational metrics that explain how the primary moves. Time per case, queue depth, SME capacity utilisation, cases-per-SME-day.
- **Model-quality** — the metrics you will track in the eval suite. Reference mod-107 for the craft.
- **Guardrail** — what is not allowed to regress. Safety refusal rate, cost per task, user-reported complaint volume, latency p95.

All thresholds are written *before you have any data*. Reference Google SRE's chapter on [Service Level Objectives](https://sre.google/sre-book/service-level-objectives/) for the discipline of writing numeric thresholds in advance.

**What good looks like.** The primary metric links one-for-one to a line in the sponsor's quarterly dashboard. The secondary metrics are observable from Chapter 01's shadowing and process-mining baselines. The model-quality metrics map to specific judges or eval harnesses the FDE will stand up in the first two weeks of build. The guardrails are things you can measure and alert on in production (see mod-110).

A sample:

```
Success metrics

Primary (business KPI)
  P90 prior-auth turnaround time
    Baseline: 48 hours
    Target at pilot exit: < 24 hours
    Measurement: customer-side case-tracking system; calculated weekly

Secondary (operational)
  Median SME time per routine case:  7min  ->  < 3min
  Senior-SME capacity freed per week: 0h   ->  > 20h
  Routine-case queue depth p90:      ~120h ->  < 60h

Model-quality (eval suite)
  Draft acceptance rate (SME edits < 25% of chars):  > 70%
  Factuality judge (benefits determination):          > 4.5/5
  Citation accuracy (formulary rule present in draft): = 100%

Guardrails (do not regress)
  Safety-refusal rate (model declines to draft):     < 3%
  Cost per task (prompt + output + retrieval + eval): < $0.15
  User complaints per 1000 cases:                     <= baseline
  p95 end-to-end latency (open case to draft ready):  < 10s
```

**Common failure modes.**

- Primary metric that is not actually the sponsor's KPI. Ask the sponsor directly which line on their dashboard this pilot will move; use that language verbatim.
- No model-quality metrics. "Draft acceptance rate" is a model-quality metric; "the SME is happy" is not.
- Thresholds that are round numbers without rationale. 70% draft acceptance is a defensible floor if, for example, the deterministic-template baseline already reaches 55%; 70% means the LLM must beat the baseline meaningfully. Document the rationale in a footnote.
- Setting thresholds *after* you have early data. This is called target shopping and it destroys the integrity of the exit-to-production decision.

The model-quality metrics seed the eval protocol (mod-101 Chapter 05 Deliverable 3). The primary metric seeds the exit-to-production criteria (below). The guardrails seed the pre-mortem.

## 5. Decision records (ADRs) for stack choices

**What it is.** One Architecture Decision Record per non-trivial stack choice. Michael Nygard's canonical form ([Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions.html), templates at [adr.github.io](https://adr.github.io/), and Spotify's [*When should I write an ADR*](https://engineering.atspotify.com/2020/04/when-should-i-write-an-architecture-decision-record/) for the "when to bother" test). Each ADR has: context, decision, status, consequences — and a named alternatives-considered section.

**Which choices get an ADR.** As a rule, any choice that (a) is hard to reverse once code is written against it, (b) a future engineer would reasonably question, or (c) the customer's security or governance reviewer will ask about. In an LLM pilot these typically include:

- Model provider and model family (frontier lab vs hyperscaler-managed vs self-hosted).
- Retrieval engine (or the explicit decision to run with no retrieval).
- Orchestration framework (direct API with structured outputs vs Claude Agent SDK vs OpenAI Agents SDK vs LangGraph vs in-house).
- Hosting / deployment model (SaaS vs dedicated SaaS vs VPC-install vs on-prem).
- Prompt-authoring approach and version-control location.
- Eval stack choice.
- Observability / LLM-observability platform.
- Human-in-the-loop checkpoint design (where in the workflow the SME reviews).

**What good looks like.**

```
ADR-003 Model provider selection

Status: accepted (2026-11-05)

Context
  The pilot requires a frontier-capability model that can draft prior-auth
  response text at <3s p95 latency, with structured output support (JSON
  schema), and that the customer's security team will accept. The customer
  is on AWS, operates under HIPAA + BAA, and requires data-residency in
  us-east. See Chapter 03 viability check for cost arithmetic.

Decision
  Use Anthropic Claude Sonnet via AWS Bedrock in us-east-1. Enable BAA
  through AWS Bedrock HIPAA-eligible services. Cost at projected volume:
  $N/month (see cost model in ADR-003 Appendix A).

Alternatives considered
  - OpenAI direct API: good model quality, but direct DPA + data-residency
    discussion would lengthen security review; declined for pilot timeline.
  - Self-hosted open-weights model in customer VPC: escalates to
    ai-infra-inference-engineer; not pursued at pilot scope due to
    hardware acquisition timeline.
  - Google Vertex Gemini: viable; customer runs primarily on AWS and
    stakeholder preference is AWS; declined for integration simplicity.

Consequences
  Positive: BAA-covered, data-residency met, simple procurement under
    existing AWS agreement, cost projection fits pilot budget.
  Negative: model behaviour locked to one family for pilot; a provider
    outage impacts the pilot until we add a gateway (mod-104).
  Risk commentary (ai-risk-engineer): data-residency implications tracked
    in risk-register entry R-004; AWS Bedrock SOC 2 Type II report
    attached to the security-review packet.
```

**Common failure modes.**

- Missing the alternatives-considered section. An ADR without alternatives is a declaration, not a decision.
- Treating the ADR as a design doc. ADRs are short — half a page to a page. The deep design goes in a separate design doc if it goes anywhere.
- Updating the ADR silently when the decision changes. A changed decision is a new ADR that supersedes the prior one, not an edit. The audit trail matters for customer governance review.

## 6. Exit-to-production criteria

**What it is.** The threshold on each success metric that constitutes "pilot succeeded, promote to production." Written up front, before any data. Signed by both sides.

**What good looks like.** Exit criteria make the go/no-go decision mechanical. Both sides read the metric; both sides read the threshold; the decision follows.

Include three branches:

1. **Pilot succeeded — promote to production.** All primary and secondary thresholds hit. All model-quality thresholds hit. No guardrail violated.
2. **Pilot inconclusive — re-scope and extend.** Primary threshold not hit but trend favourable; one or two secondary thresholds missed by <X%. Specific re-scope proposal within Y weeks.
3. **Pilot failed — decline to productise.** Primary threshold missed by >X%; guardrail violated; or the sponsor's KPI has moved such that the pilot no longer serves it.

A sample:

```
Exit-to-production criteria

Promote to production if, on the final 2 weeks of the pilot:
  - P90 prior-auth turnaround < 24h sustained for 10 consecutive business days
  - Median SME time per routine case < 3min on the full gold set
  - Draft acceptance rate >= 70% on the final week
  - Zero guardrail violations in the final 2 weeks
  - Customer governance review signed off (see Chapter 06)

Re-scope and extend if:
  - P90 turnaround between 24h and 30h with 2-week improving trend AND
    draft acceptance rate >= 60% with improving trend
  - Approved scope-change amendment with new target date

Decline to productise if:
  - P90 turnaround > 30h at week 10
  - Any guardrail violated for 2+ consecutive weeks without remediation
  - Customer governance review surfaces unresolvable control gap
  - Sponsor KPI has been re-prioritised away from this metric
```

**Common failure modes.**

- Writing exit criteria that only define "succeeded." You need the "inconclusive" and "failed" branches to prevent "well, let's see" indefinite extension.
- Thresholds that are soft. "Materially improve" is not a threshold; "<24h" is.
- No timeline dimension. "<24h sometimes" is not an exit criterion; "<24h sustained for 10 consecutive business days at the end of the pilot" is.

The exit criteria will be referenced in the design-partner commercial document (Chapter 05) and signed by the customer sponsor. They are not negotiable later.

## 7. Pre-mortem

**What it is.** Gary Klein's [pre-mortem exercise](https://hbr.org/2007/09/performing-a-project-premortem): before any code ships, you imagine the pilot has failed and reconstruct why. Each failure gets an early-warning indicator, a mitigation, and an owner.

**Discipline: three entries.** A 15-item pre-mortem is a security review disguised as a pre-mortem. Pick the top three. If it turns out you missed one during the pilot, add it in the next cycle; do not try to enumerate every failure mode at scoping time.

**What good looks like.**

```
Pre-mortem — top 3 failure modes

PM-01  Factuality drift on benefits determination
  Scenario: The draft contains a wrong benefits determination; the SME,
    under time pressure, misses it on review; the member receives an
    incorrect denial; compliance incident.
  Early-warning indicator: factuality judge score drops below 4.2/5 on
    rolling weekly window.
  Mitigation: freeze model version; re-run eval; investigate drift;
    hold rollout to additional SMEs until judge recovers.
  Owner: FDE (lead) + customer-side medical director (approver).

PM-02  SME adoption craters after week 3
  Scenario: SMEs stop using the draft feature after week 3 because the
    review overhead exceeds the drafting saving for the cases they handle.
  Early-warning indicator: daily SME usage falls <40% of eligible cases
    in any 3-day window.
  Mitigation: shadow SMEs who have stopped using the feature; identify
    friction (common cases where review overhead exceeds saving); narrow
    scope to cases that benefit most.
  Owner: FDE + ai-enablement-lead (see Chapter 06).

PM-03  Audit-log retention insufficient for customer governance
  Scenario: Customer's SOC 2 period requires 7-year audit-log retention
    including model inputs/outputs; our observability stack retains 90 days.
  Early-warning indicator: this is a governance-review question, not an
    operational one — flagged during security-review packet review.
  Mitigation: engage ai-risk-engineer at week 2 to specify retention
    requirement in writing; design retention tier at week 3; validate with
    customer's compliance officer at week 4.
  Owner: ai-risk-engineer + FDE (implementation).
```

**Common failure modes.**

- Pre-mortem bloat. 15 items is a risk register, which is Chapter 06 / mod-108 territory — not a pre-mortem.
- Scenarios without named consequence. "The model doesn't work" is not a scenario; "benefits determination is wrong -> denial letter sent in error -> grievance filed" is.
- No owner. Every scenario has one named person on the hook for the mitigation.

The pre-mortem seeds the Chapter 06 coordinations with `ai-enablement-lead` and `ai-risk-engineer` — at least one entry per pre-mortem should trigger each peer.

## 8. Timeline with a re-decide checkpoint

**What it is.** Week-by-week schedule to pilot exit, with at least one explicit re-decide gate at roughly 50% of the planned duration where either side can call the pilot based on partial evidence.

**What good looks like.** Dates, not week-labels. Named checkpoints with named participants. The re-decide gate has its own partial exit-criteria — not the full exit criteria, but an interim set on which both sides can decide "keep going" vs "pause and re-scope."

A sample for a 12-week pilot:

```
Timeline (12-week pilot, kickoff 2026-11-10)

Week 1-2   Discovery (chapter 1 of this module)  -> discovery brief signed
Week 2-3   Pilot scope, eval protocol            -> scope signed (this doc)
Week 2-5   Integration design + early build
Week 3-6   Gold-set labelling with SMEs (mod-107)
Week 5-6   RE-DECIDE CHECKPOINT (2026-12-15)
             Interim exit criteria:
               - Draft acceptance rate on 100-case cold run: >= 50%
               - Primary KPI directional trend positive on pilot users
               - No unresolvable security-review blockers
             Participants: FDE, sponsor, customer medical director, AE
             Options: continue / re-scope / decline
Week 6-9   Rollout to broader SME cohort; eval-gated CI live
Week 9-11  Production monitoring + pre-exit eval on hold-out
Week 10    SECURITY & GOVERNANCE SIGN-OFF REVIEW
Week 11-12 Exit decision on full criteria -> promote / re-scope / decline
```

**Common failure modes.**

- No re-decide. "We'll see how it goes" is not a schedule. The re-decide gate is what prevents a 12-week pilot from silently becoming a 20-week pilot.
- Re-decide on the same criteria as final exit. Interim criteria are directionally looser; they are about "is the trend favourable" not "has it landed."
- Soft dates. "Late November" is not a date; "2026-11-25" is.

## 9. Who does what

**What it is.** A named list. Not roles. People.

```
Participants and responsibilities

Customer side
  Executive sponsor:    Chen Wei (VP, Clinical Operations)
  Business sponsor:     Priya Nair (Director, Prior-Auth)
  SME lead (primary):   Mariana Diaz (Senior Clinical Reviewer)
  SME backup:           David Okonkwo (Clinical Reviewer, 3-yr tenure)
  Data owner:           Fatima Al-Mansouri (Data Engineering Lead)
  Security reviewer:    Henri Laurent (Security Program Manager)
  Operator-lead:        Mariana Diaz (same)
  On-call during pilot: TBD by week 6 (named in deployment runbook)

FDE side
  FDE (primary):        <learner>
  FDE engineering mgr:  <learner>'s manager
  AE:                   <AE name>
  CSE (hand-off):       <CSE name>
  ai-enablement-lead:   <peer name>
  ai-risk-engineer:     <peer name>
```

**Common failure modes.**

- "The customer team." Not a name. Not a RACI.
- Role without name. "The operator lead" is a job description, not a signatory.
- Missing the peer roles (`ai-enablement-lead`, `ai-risk-engineer`). Chapter 06 requires they are named at scoping time, not when the pilot is in trouble.

## 10. How the document is used after signing

Once signed, the pilot-scope document becomes the artifact against which every scope-change proposal is measured.

A scope-change request has three possible outcomes:

1. **Fits an existing user story.** No amendment needed; the work proceeds inside the existing scope.
2. **Does not fit any user story but should.** Draft a written amendment: new user story, updated success metric if relevant, updated non-goal if the change narrows something else, updated timeline. Both sides sign. One line in the scope-change log.
3. **Out of scope.** The work is deferred to a post-pilot follow-on or to the customer's own engineering team. The ask is logged (so the customer sees the FDE heard it) and the pilot proceeds unchanged.

The scope-change log is a one-row-per-change table living with the pilot-scope document. It is countersigned at the re-decide gate and at pilot exit.

The purpose of this discipline is not bureaucracy. It is to prevent the Week-6 drift where an email thread quietly expands the pilot to cover the back-office reconciliation step, which was explicitly in non-goals, and the pilot now misses its exit criteria because half the engineering time went to work that was never scoped.

## Summary

- The pilot-scope document is 4-8 pages, signed by the customer sponsor and the customer-side operator-function head, countersigned by the FDE's engineering manager.
- Write it top-to-bottom: problem statement, user stories, non-goals, success metrics (primary + secondary + model-quality + guardrails), ADRs for stack choices, exit-to-production criteria (three branches), pre-mortem (top three), timeline (with re-decide gate), who does what (named people), scope-change discipline.
- Every section traces back to Chapter 01 discovery, Chapter 02 candidate statement, and Chapter 03 test results. If a section has no discovery source, you are writing on assumption.
- The exit-to-production criteria are written before any data exists. The pre-mortem is three items, not fifteen. ADRs always include alternatives considered.
- Once signed, this document is the artifact against which every scope-change proposal is measured. A scope change is a countersigned amendment, not an email thread.
