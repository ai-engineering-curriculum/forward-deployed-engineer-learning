# Chapter 07 — Knowledge Transfer — Becoming Unnecessary at Hand-Off

## Why this chapter exists

[Chapter 06](./06-scope-discipline-and-decision-log.md) wrote the discipline that protects the pilot's shape during delivery — scope integrity and the countersigned decision log. This chapter writes the discipline that *ends* the engagement on the right terms: the FDE's exit.

The FDE is design-partner engineering, not permanent embedded staff. [Mod-101 Chapter 04](../mod-101-fde-discipline-and-position/04-adjacent-customer-facing-roles.md) is explicit — the FDE is not a Professional Services contractor against the vendor's FDE budget; the FDE is not a Customer Success Manager; the FDE is not a Sales Engineer. The FDE enters at pilot scoping, ships the design-partner pilot, and exits when the customer's own engineering team can operate the system. If the FDE does not exit, every subsequent pilot is understaffed, the vendor's field-to-product loop ([mod-111](../mod-111-field-to-product-loop/)) stalls, and the pilot that was supposed to be a reference becomes a billable dependency.

The hand-off is the FDE's exit. The receiver is the customer's own engineering team — typically a customer-side `applied-ai-engineer`, backed by a `platform-engineer` for the deployment infrastructure, and (where the customer has one) an `ai-platform-engineer` for LLM infrastructure. The customer SME lead owns ongoing gold-set curation; the customer sponsor owns the outcome metrics. The FDE's measurable success at hand-off is **the customer engineering team shipping the next prompt change, running the next eval, handling the next on-call page, and walking the next deployment — without the FDE in the room**.

This chapter writes the four hand-off streams, the readiness gate, the last-mile exit ramp, the documentation craft that survives the FDE's departure, the adjacent-role handshake map on the customer side, and the hand-off failure modes that pull the FDE back into permanent embed.

Hand-off readiness is a **gate** dimension from [Chapter 01](./01-consulting-craft-overlay.md).

## The four hand-off streams

Hand-off is not a single meeting at the end of the pilot. It is four parallel streams running through the last 6–8 weeks of delivery, each with its own cadence, its own deliverable, and its own success criterion.

| Stream | Cadence | Deliverable | Success criterion |
|---|---|---|---|
| Pair-programming | Weekly, 6–8 weeks | Commits owned by customer engineer | Customer engineer drives 90 %+ of commits by week 6 |
| Documentation | Running, ships with pilot | Eight runbook / SOP artefacts | Reviewed by a customer engineer who did NOT pair with the FDE |
| Deployment walkthroughs | 3 cycles minimum | Co-walk → observe → notified-only | Customer engineer owns three consecutive deploys unaided |
| On-call shadowing | 2 rotations swapped, then transfer | Customer engineer primary on pager | One rotation each direction + primary transfer |

Each stream has its own subsection below. All four must be running by roughly 8 weeks before the planned exit; any stream that starts later than 2 weeks before exit has failed.

### Stream 1 — Pair-programming cadence

Weekly pairing on real work for the last 6–8 weeks of the pilot, driven by the customer's engineer. The FDE navigates — reads error traces aloud, suggests the next cursor, surfaces the hidden assumption — but the customer engineer owns the keyboard, owns the commit, owns the PR description.

The ramp is explicit. Agree it in writing with the customer engineer and the sponsor before week 1:

| Week | FDE drives | Customer engineer drives | Example work |
|---|---|---|---|
| 1 | 80 % | 20 % | FDE walks the prompt-change process end-to-end; customer engineer pairs |
| 2 | 60 % | 40 % | Customer engineer adds a gold-set entry; FDE navigates the eval re-run |
| 3 | 40 % | 60 % | Customer engineer owns a canary deploy; FDE navigates rollback drill |
| 4 | 30 % | 70 % | Customer engineer handles a retrieval-drift triage; FDE observes |
| 5 | 20 % | 80 % | Customer engineer tunes an alert threshold; FDE reviews the PR |
| 6 | 10 % | 90 % | Customer engineer ships a prompt revision; FDE is a reviewer |
| 7 | 10 % | 90 % | Customer engineer handles an on-call page; FDE is notified-only |
| 8 | 0 % | 100 % | FDE is in the retro, not the work |

Pair-programming is the stream that most reliably exposes tacit knowledge. Every "oh, I always do X before Y" that leaks out of the FDE's mouth during pairing is a documentation gap — capture it immediately. If the FDE finishes a pair session without having said at least three sentences the customer engineer wrote down, the session was not deep enough.

Two operational notes on pairing. First, **the customer engineer owns the PR description, not the FDE**. A PR description the FDE wrote is a signal the customer engineer did not internalise the change; even if the FDE's version would read better, resist. Second, **the pair sessions are recorded with the customer engineer's consent** — not for rewatching, but so the customer engineer can reference the exact moment the FDE said "the retrieval cache invalidates when the embedding model version changes, so after a model-version bump we always re-index before smoke-testing." That one sentence becomes a runbook note; without the recording it becomes a question the customer engineer has to re-ask three months later.

### Stream 2 — Documentation craft

Documentation ships *with* the pilot code, versioned in the same repo, reviewed in the same PRs. Eight artefacts are non-negotiable:

1. **Architecture overview.** One diagram, one page of prose, explicit "why" on every non-obvious choice. Reads like an ADR; links to the full ADR history.
2. **Deployment runbook.** Preconditions, ordered steps, verification checks, rollback procedure, known gotchas. Modelled on the deployment-playbook slice from [mod-111 Chapter 06](../mod-111-field-to-product-loop/06-fde-team-knowledge-base.md) but *specific to this pilot*, not the vendor's generic shape. Cross-link [mod-109](../mod-109-deployment-rollout-change-management/).
3. **Eval-pipeline runbook.** How to add a gold-set entry, how to re-run evals against a prompt change, how to interpret the judge-rubric output, how the regression suite is wired into CI. Cross-link [mod-107](../mod-107-eval-eng-on-customer-data/).
4. **On-call runbook.** The six canonical failure modes from [mod-110 Chapter 05](../mod-110-observability-and-oncall/05-oncall-rota-and-llm-triage.md) instantiated for this pilot: what the dashboard looks like under each, what to try first, when to page the LLM-provider's support channel, when to roll back.
5. **Prompt-change process.** Branch naming, PR template, required eval runs, required canary posture, required sign-off. The customer engineer will exercise this at least once before the FDE exits (readiness-gate item below).
6. **Gold-set-management SOP.** How gold-set entries are proposed (customer SME), reviewed (customer engineer + SME lead), labelled with provenance, exported with provenance preserved. The SOP names the SME lead as the ongoing curator.
7. **Security-review-answer library (localised).** The vendor's SIG / CAIQ / SOC 2 response library ([mod-108 Chapter 02](../mod-108-enterprise-security-and-compliance/02-security-review-packet.md)) narrowed and annotated for *this* pilot — the specific BAA clauses, the specific control mappings, the specific evidence artefacts this customer's IT team holds. The customer's IT reviewer reads this; the FDE's version is archived.
8. **Decision log.** The countersigned register from [Chapter 06](./06-scope-discipline-and-decision-log.md). Up-to-date through the most recent decision; countersignature current.

Each artefact has a characteristic length and audience, and a characteristic review cadence:

| Artefact | Target length | Primary audience | Review cadence post-exit |
|---|---|---|---|
| Architecture overview | 1 page + 1 diagram | New engineer, auditor | Every 90 days |
| Deployment runbook | 3–5 pages | `platform-engineer` on duty | After every deploy |
| Eval-pipeline runbook | 2–4 pages | `applied-ai-engineer` | Monthly |
| On-call runbook | 5–8 pages (decision-tree) | On-call engineer at 3am | After every page |
| Prompt-change process | 1–2 pages | `applied-ai-engineer` | Every 90 days |
| Gold-set-management SOP | 2–3 pages | SME lead + applied-AI-engineer | Monthly |
| Security-review-answer library | 10–30 entries | IT reviewer, CISO office | Every 180 days |
| Decision log | Grows with pilot | Sponsor, model-risk function | After every decision |

The lengths are targets, not ceilings — but an artefact that balloons past 2× its target is an artefact that is not being read. The on-call runbook is the one most at risk of bloat; it is also the one whose structure (decision-tree, not prose) most matters, because the person reading it is under time pressure. Mirror the shape of the failure-mode library in [mod-111 Chapter 06](../mod-111-field-to-product-loop/06-fde-team-knowledge-base.md) — symptom → diagnosis → response → root-cause check, scannable in five minutes.

Documentation craft principles the FDE writes to:

- **Write for the reader who was not in the room.** The person reading the architecture overview six months after the FDE exits has never heard the hallway conversation that shaped the choice. Name the conversation in the doc.
- **Name the "why" of every non-obvious choice.** ADR-style — context, decision, consequences. The ADR community conventions (Michael Nygard's original shape) are the baseline. <!-- needs-research: confirm the current canonical ADR template reference; Nygard's 2011 post and the adr.github.io templates are the two commonly cited sources -->
- **Keep examples copy-pasteable.** Literal filenames, literal commands, literal environment variables. A runbook that reads `set $PROMPT_VERSION to the new version` is a runbook that fails at 3am; a runbook that reads `export PROMPT_VERSION=2026-11-04-canary-02` is a runbook that works.
- **Version with the pilot code.** Docs in a Notion page the FDE wrote are docs that disappear at hand-off. Docs in `/docs` in the same repo as the pilot code are docs the customer engineer merges with their code changes.
- **Link every runbook to the regression fixture or eval it protects.** The on-call runbook's "hallucination spike" branch links to the gold-set entry that catches it; the deployment runbook's rollback step links to the smoke-test script that verifies the rollback landed.

A concrete shape for the eight artefacts, living in the pilot repo:

```
pilot-repo/
  docs/
    01-architecture-overview.md          # one diagram, one page of prose
    02-deployment-runbook.md             # preconditions, steps, rollback, gotchas
    03-eval-pipeline-runbook.md          # gold-set add, re-run, judge interpretation
    04-oncall-runbook.md                 # six failure modes, this pilot's specifics
    05-prompt-change-process.md          # branch naming, PR template, canary
    06-gold-set-management-sop.md        # proposal, review, provenance labels
    07-security-review-answers.md        # localised SIG / CAIQ / SOC 2 responses
    08-decision-log.md                   # countersigned; links to each ADR
    adr/
      0001-why-vpc-install-langfuse.md
      0002-why-prompt-canary-24h-hold.md
      ...
```

The eight files are first-class repo citizens. They are reviewed in PRs, versioned with the code, and linked from the root `README.md`. A docs change that is not merged is a docs change that does not exist.

### Stream 3 — Deployment runbook walkthroughs

Three deploys minimum, with shifting role:

1. **Co-walk.** FDE and customer engineer run the deploy together, step-by-step, against the runbook. Any step the runbook under-specifies is fixed in the same session.
2. **Observe.** Customer engineer runs the deploy; FDE is present but silent unless called. The FDE writes down every moment they *wanted* to speak but did not — those are the moments that become runbook additions.
3. **Notified-only.** Customer engineer runs the deploy. FDE is notified of start and finish; FDE is not present in Slack during the deploy. Any issues surface through the customer engineer's chosen channels.

Three cycles is the minimum; some pilots require more (a complex canary posture may warrant two notified-only cycles before readiness). The gate-item below reads "three consecutive unaided deploys"; walkthroughs below three do not count.

The deploy cycles must include at least one **real rollback**. A rollback drill against a known-good deploy is insufficient — the customer engineer has to execute rollback under the ambiguity of "something went wrong, I think it was the prompt change, I need to confirm before I roll back." The FDE stages the ambiguity deliberately — either by timing a canary against a mild known regression or by pairing the rollback drill with a randomly-injected guardrail false-positive. The customer engineer who has only rolled back clean deploys has not actually learned the shape.

Cross-link [mod-109](../mod-109-deployment-rollout-change-management/) — the feature-flag discipline, the canary-hold windows, the rollback drill. The customer engineer should have run a rollback drill at least once before taking primary.

A concrete rubric for the fresh-reader check, which the hand-off readiness gate requires. The reviewer — a customer engineer who did NOT pair with the FDE — walks through four scenarios cold, with only the eight artefacts and the pilot repo in front of them:

1. "The 3am page says retrieval-drift. Walk me through the first five minutes of your triage."
2. "You need to ship a prompt revision because the SME flagged a regression yesterday. Walk me through the end-to-end flow from branch to production."
3. "A CISO reviewer sends you a SIG asking about prompt-injection posture. Where do you find the answer?"
4. "You need to roll back the most recent deploy because the canary metrics went red. Walk me through the rollback."

A reviewer who cannot produce a plausible walk-through for any one of these with only the docs is a reviewer who has found a documentation gap. The gap is closed before the readiness gate signs.

### Stream 4 — On-call shadowing

On-call shadowing runs both directions, in sequence:

1. **Customer engineer shadows FDE — one rotation.** The customer engineer is on every page with the FDE. Reads every alert, hears every triage, watches every mitigation. Owns no decision yet.
2. **FDE shadows customer engineer — one rotation.** The customer engineer is primary; the FDE is backup. Every page is the customer engineer's; the FDE answers questions but does not take the keyboard unless the customer engineer explicitly asks.
3. **Customer engineer primary — hand-off.** The pager routes to the customer engineer's rotation. The FDE is out of the loop except for the agreed post-hand-off contact protocol (see last-mile section below).

Two rotations is the minimum, one each direction. A single rotation in either direction is insufficient — the customer engineer sees too few failure shapes to generalise. Cross-link [mod-110 Chapter 05](../mod-110-observability-and-oncall/05-oncall-rota-and-llm-triage.md) for the rota shape and the LLM-specific triage patterns.

A rotation that produces no pages is a rotation that teaches nothing. If the customer-shadow-FDE rotation ends with zero pages, the FDE runs a **synthetic triage** before the roles flip — a tabletop exercise against two of the six canonical failure modes, with the FDE narrating what would be on the dashboard and the customer engineer deciding the response. Tabletops are not a substitute for a real page, but they fill the gap when the pilot is quiet. Document which failure modes were tabletopped in the on-call runbook so the next quiet rotation covers different ones.

## The hand-off readiness gate

The hand-off readiness gate is a checklist. The customer sponsor and the FDE's `ai-engineering-manager` co-sign it before the FDE exits. Neither party's signature alone is sufficient — the FDE cannot exit on their own authority, and the customer sponsor cannot exit the FDE on their own authority. Both signatures. Dated. Countersigned.

The checklist:

- [ ] Runbooks complete and reviewed by the customer engineer (all eight artefacts from Stream 2).
- [ ] Two production incidents handled by the customer engineer with the FDE in notified-only posture.
- [ ] Eval pipeline running un-touched by the FDE for 30 consecutive days.
- [ ] Decision log up to date; countersigned through the most recent entry.
- [ ] Security-review-answer library exported and reviewed with the customer's IT team.
- [ ] On-call rota confirmed; pager routes to the customer engineer as primary; FDE is removed from the rota.
- [ ] Gold set exported with provenance labels; ownership transferred to the customer SME lead in writing.
- [ ] Prompt-change process exercised by the customer engineer at least once end-to-end (draft → eval → canary → promote).
- [ ] Dashboards owned by the customer engineer; no FDE-only credentials, no FDE-only SSO claims, no FDE-only cloud accounts.
- [ ] Monitoring alerts reviewed and tuned by the customer engineer (at least one threshold adjustment; demonstrates understanding).
- [ ] Documentation reviewed by a customer engineer who was NOT the FDE's pair partner — the "fresh-reader" check, which catches the shared-vocabulary blind spots the pair partner has absorbed.
- [ ] Post-hand-off contact protocol agreed in writing — what channel, what SLA, what counts as in-scope question.
- [ ] Customer sponsor confirms the outcome metrics are being reported through a customer-owned channel (not an FDE-owned dashboard).
- [ ] `ai-engineering-manager` confirms the FDE has written their mod-111 Chapter 02 capture log entries and promoted any new pattern-catalogue entries.

Fourteen items. The gate is pass / not-pass; partial credit is not a thing. An item that the customer sponsor and `ai-engineering-manager` disagree on triggers the sponsor-to-manager escalation, not an FDE unilateral call.

The gate is scheduled from day 1 of delivery — a calendar entry 2 weeks before the planned exit date, attended by the customer sponsor, the customer engineer taking primary, `ai-engineering-manager`, and the FDE. Scheduling the gate at the end of the pilot is scheduling it too late; the items that are not yet passing need the 2-week runway to close. A gate meeting that discovers five open items on the day is a gate meeting that was planned wrong.

The gate's output is a signed document living in the pilot repo at `docs/handoff-readiness-gate.md`, with the fourteen items each annotated with the evidence that passed them (links to PRs, incident tickets, dashboard screenshots, decision-log entries). The signatures are version-controlled — the sponsor and `ai-engineering-manager` sign by merging a PR that adds their names, dates, and "I attest the above items are passing" line. A verbal "yes we're good" is not a signature.

A failing gate is a legitimate outcome, not a failure of the FDE. If three items are not passing at the scheduled gate meeting, the honest response is to name them, agree remediation, and reschedule the gate 1–3 weeks out. The failure mode is not "the gate failed"; the failure mode is "the gate signed anyway because the FDE's next pilot is scheduled." `ai-engineering-manager` protects the gate's integrity by owning the "we're not ready yet" conversation with the customer sponsor rather than offloading it to the FDE.

## The last-mile pattern

After the gate signs, the FDE stays on reduced engagement for 4–6 weeks. The shape:

- **Cadence.** One day per week, or equivalent as an office-hours window. Not full-time, not zero. The reduction is visible to the customer.
- **Posture.** Answering questions only. No commits authored by the FDE. No PRs opened by the FDE. The FDE reviews, advises, occasionally pair-debugs — but the artefact is owned by the customer engineer.
- **Duration.** 4–6 weeks. A duration longer than 6 weeks is a signal the hand-off was not actually ready; the gate missed something. Escalate to `ai-engineering-manager` and the customer sponsor rather than silently extending.
- **Exit.** At the end of the last-mile period, the FDE sends a final note confirming the engagement is closed, naming the post-hand-off contact protocol, and copying `ai-engineering-manager` and the customer sponsor. The FDE's Slack / email access to the customer's internal channels is removed on a known date.

The last-mile pattern is the design-partner exit ramp. It is **not** a billable Professional Services engagement. Any ask during the last-mile that is SOW-shaped — "could you build us X", "could you spend a week on Y", "could you integrate with Z" — routes to Professional Services per [mod-101 Chapter 04](../mod-101-fde-discipline-and-position/04-adjacent-customer-facing-roles.md). The FDE's answer is "that's a PS conversation; let me loop in the account team."

The last-mile's office-hours shape is specific. A standing one-hour slot per week on a predictable day (Tuesdays 2–3pm is common), publicly visible on the customer engineer's calendar. Questions queue in a shared document between sessions — not Slack DMs, not emergency pings. If the customer engineer has accumulated fewer than two questions by the Tuesday morning, the FDE and the customer engineer agree to skip the session; a session with no agenda is a session that produces theatre, not learning. Three consecutive skipped sessions is a signal the hand-off actually landed; the last-mile ends early rather than extending.

## Naming the receiver before the pilot ships

The hand-off receiver is named at pilot scoping, not at pilot end. [Mod-102](../mod-102-customer-discovery-and-pilot-scoping/) treats "who inherits this" as a scoping artefact — if the customer cannot name an `applied-ai-engineer` (or hire for one) at scoping time, the pilot shape is wrong. A pilot whose receiver is "TBD" is a pilot whose hand-off is "TBD"; by delivery week 6 the FDE discovers the receiver is a contractor on a 30-day engagement and the hand-off is structurally impossible.

The receiver's calendar presence is also a scoping artefact. The customer engineer taking primary needs roughly 30 % of their time available in the final 6–8 weeks for pair-programming, documentation review, deployment walkthroughs, and shadowing. A customer engineer whose calendar is full is a customer engineer who cannot receive the hand-off; this gets negotiated with the sponsor at scoping and reconfirmed at the start of the hand-off window. If the customer engineer's availability has slipped below 20 % by week 1 of the hand-off window, the FDE escalates to the sponsor rather than silently absorbing the gap.

## The post-hand-off contact protocol

The last readiness-gate item — "post-hand-off contact protocol agreed in writing" — deserves its own treatment because it is the most commonly fumbled. The protocol answers four questions explicitly:

1. **What channel.** Not the FDE's personal Slack DM. Not the FDE's personal email. A named, shared channel — typically a Slack channel in the vendor's workspace that the customer engineer has guest access to, or a shared support alias. The channel is visible to `ai-engineering-manager` and (where relevant) the account team. A DM is an invisible dependency; a shared channel is a tracked one.
2. **What SLA.** Design-partner questions do not carry a production-support SLA. Typical shape: best-effort response within 2 business days, with a clear "this is not a support contract" header on the channel. Anything with a tighter SLA need is a PS conversation or a paid support tier.
3. **What counts as in-scope question.** Design intent, "why did we choose X", "where is Y documented", "who on our team would know about Z" — in-scope. New feature build, infrastructure change, incident response on the customer's production — out of scope; routes to PS or to the vendor's standard support channel.
4. **When the protocol expires.** Typically 90 days after exit. After 90 days the channel moves to read-only and the contact path becomes the vendor's standard support channel. An FDE still answering questions at month 6 is an FDE whose hand-off did not actually land; `ai-engineering-manager` reviews the protocol's usage at 60 and 90 days.

The protocol is stored in the pilot repo at `docs/post-handoff-contact.md` with the sponsor's and `ai-engineering-manager`'s signatures. The customer engineer, the sponsor, and the vendor's account team all have a link.

## Adjacent-role handshake map (customer-side)

The FDE's exit is to a named customer-side person for each responsibility. The handshake map is explicit before hand-off, signed by the sponsor, and included in the documentation:

| Responsibility | Customer-side owner | Receives from FDE |
|---|---|---|
| Prompt changes, eval runs, model work | `applied-ai-engineer` | Pair-programming history, prompt-change process, eval-pipeline runbook |
| Deployment infra, CI/CD, secrets rotation | `platform-engineer` | Deployment runbook, rollback drill, secrets-management SOP |
| LLM infra (gateway, provider contracts, usage budgets) | `ai-platform-engineer` (if the customer has one) | Gateway configuration, provider-contract terms, budget-alert thresholds |
| Ongoing gold-set curation, SME labelling | Customer SME lead | Gold-set-management SOP, provenance labels, review cadence |
| Outcome metrics, renewal decision, pilot narrative upward | Customer sponsor | Executive brief template, before/after KPI shape, decision log |
| Pager primary, incident response | `applied-ai-engineer` (typically) with `platform-engineer` backup | On-call runbook, rota, triage playbook |

If the customer does not have an `ai-platform-engineer`, the LLM-infra responsibilities fold into either `applied-ai-engineer` (the typical case at mid-size customers) or `platform-engineer`. The FDE names the fallback explicitly in the handshake map — "Customer X has no `ai-platform-engineer`; LLM-gateway configuration transfers to the `applied-ai-engineer` role, with `platform-engineer` backup on infrastructure questions."

A second kind of fallback: the customer has the roles on paper but the people have not yet been hired. The FDE's hand-off cannot wait for a hire. In that case the fallback is to the nearest in-seat engineer plus a written commitment from the sponsor that the role will be filled within a named window (typically 90 days). The documentation and runbooks are still written to the target role shape — not down-scoped to the in-seat engineer's current skill set — so the eventual hire inherits the pilot as intended. The FDE flags the hire dependency explicitly in the readiness-gate document; `ai-engineering-manager` tracks it through the post-exit period.

## Measuring hand-off health during the window

Four quantitative signals track hand-off health during the 8-week window. The FDE reports them to the sponsor and `ai-engineering-manager` weekly:

| Signal | Target by week 4 | Target by week 8 | What it means |
|---|---|---|---|
| Customer-engineer commit share (last 2 weeks) | ≥ 50 % | ≥ 90 % | Pair-programming ramp is tracking |
| Customer-engineer PR-author share (last 2 weeks) | ≥ 40 % | ≥ 90 % | Not just commits — the customer engineer is thinking the change through |
| Runbook coverage of recent incidents (last 30 days) | ≥ 70 % | ≥ 95 % | Every real incident in the window either had a runbook branch or added one |
| FDE-originated Slack messages in pilot channel (per day) | ≤ 10 | ≤ 2 | The FDE's presence is receding on a measurable curve |

A signal that is off target by more than 20 % at the mid-point (week 4) is a signal the hand-off is not tracking; the FDE raises it at the next weekly review per [Chapter 04](./04-weekly-pilot-review-cadence.md), not the week of exit. A sponsor who hears "hand-off is behind" with 2 weeks to go has fewer options than a sponsor who hears it with 4 weeks to go.

## Week-by-week hand-off plan

A concrete eight-week plan the FDE can lift and adapt:

| Week | Pair-programming | Documentation | Deployment | On-call |
|---|---|---|---|---|
| 1 | FDE drives 80 / customer 20 | Architecture overview drafted; ADR history reviewed | — | Customer engineer begins shadowing FDE's rotation |
| 2 | FDE drives 60 / customer 40 | Deployment runbook drafted; eval-pipeline runbook drafted | Co-walk deploy #1 | Customer engineer continues shadowing |
| 3 | FDE drives 40 / customer 60 | On-call runbook drafted; prompt-change process drafted | — | Customer engineer completes shadow rotation |
| 4 | FDE drives 30 / customer 70 | Gold-set SOP drafted; security-review library localised | Observe deploy #2 (customer engineer drives) | FDE shadows customer engineer's first rotation |
| 5 | FDE drives 20 / customer 80 | Fresh-reader review by non-pair customer engineer | — | FDE shadow rotation continues |
| 6 | FDE drives 10 / customer 90 | Documentation gaps from fresh-reader review fixed | Notified-only deploy #3 | FDE shadow rotation completes |
| 7 | FDE drives 10 / customer 90 | Decision log countersigned through latest; dashboards transferred | Notified-only deploy #4 | Customer engineer takes primary; FDE is backup only |
| 8 | FDE in retro only | Gate checklist assembled; co-signature scheduled | — | Customer engineer primary; FDE off the rota |

Weeks 9–14 are the last-mile: one day per week, questions-only, no commits by FDE.

## The tacit-knowledge problem

The hardest part of hand-off is not the artefacts; it is the knowledge the FDE has not noticed they have. The FDE knows that the provider's rate limits spike at the top of every hour because another large customer runs a batch job then; the FDE knows that the customer's SIEM mutes certain alert categories between 10pm and 6am; the FDE knows that the SME lead responds to a Slack question faster than an email. None of this is in the runbook. All of it is load-bearing in the first on-call rotation the customer engineer owns.

Three mechanisms surface tacit knowledge before exit. First, the **"what did I do today"** journal — the FDE spends 10 minutes at the end of each pairing week writing down the three non-obvious things they did that the customer engineer did not see. Second, the **ask-me-anything** session — one 90-minute block in week 7, run by the customer engineer, with a written list of "what still feels fuzzy." Third, the **fresh-reader walkthroughs** from the gate rubric above — a second customer engineer forcing the FDE to articulate what the first customer engineer has already absorbed implicitly.

A hand-off that discovers tacit knowledge only at the first post-exit incident is a hand-off that left too much in the FDE's head. The three mechanisms move the discovery earlier, into the window where the FDE can still answer.

## Running the three archetypes

**Archetype A — US mid-size healthcare payer.** On-call must integrate with the payer's Security Operations Centre rotation; a HIPAA-regulated incident (any potential breach) routes through the SOC's established escalation pathway, not the vendor's. Shadowing therefore happens on the customer's SIEM (typically Splunk or Microsoft Sentinel in payer environments) rather than on the vendor's observability stack. Runbooks are reviewed with the privacy office before sign-off — the breach-notification branch in the on-call runbook needs the privacy officer's countersignature, not just the engineer's. The security-review-answer library localised to this pilot cites the specific 45 CFR §164 subsections the BAA references; the gold-set export preserves PHI-redaction provenance labels; the last-mile contact protocol routes through the payer's existing vendor-support channel, not a new one. The documentation review step adds a HIPAA-training confirmation item — every customer engineer reviewing the documentation has completed the payer's annual HIPAA training in the current calendar year, which the privacy office tracks. <!-- needs-research: confirm whether a payer SOC's incident classification taxonomy is governed by a specific HIPAA Security Rule provision or by the payer's own policy — the practical observation is that payers set this internally but may reference §164.308(a)(6) administrative safeguards -->

**Archetype B — US federal agency.** The hand-off receiver is agency-internal engineers who hold the appropriate clearance and operate inside the enclave. The FDE's last-mile must occur inside the same enclave; office-hours from outside the boundary is not an option for controlled-information questions. Documentation stays inside the authorisation boundary — the pilot's runbooks, decision log, and localised security-review library are stored in the agency's cleared document system, not in the vendor's repo. The FDE's handshake map names specifically-cleared agency engineers for each responsibility; an uncleared engineer cannot inherit a controlled-information runbook. The readiness gate adds an ATO-boundary item: the Authorising Official or their designee confirms the system's operation is understood by the inheriting team under the existing ATO. The post-hand-off contact protocol is especially constrained — a Slack channel outside the enclave is not usable for any question that touches controlled information; the protocol typically routes through a government-furnished ticketing system or a vendor-cleared support engineer rather than the exit-ing FDE directly. <!-- needs-research: confirm whether ATO-inheritance at hand-off requires a formal re-authorisation action under NIST 800-37 Rev 2 or whether the system-steward change is handled under continuous-monitoring provisions; practice varies by agency -->

**Archetype C — EU retail bank.** The hand-off receiver is the bank's own ML-ops team. The readiness gate maps onto DORA's contractual exit provisions for ICT third-party service providers — the bank must be able to operate the pilot without the vendor's engineering, and the exit must be documentable to the bank's model-risk function and (on request) to the competent authority. The FDE's hand-off checklist items for "dashboards owned by customer engineer" and "no FDE-only credentials" are not just operational — they are DORA exit-plan evidence. The decision log's countersignature by the model-risk function head is an exit artefact. The bank's ML-ops team typically operates under a formal model-risk policy that imposes additional sign-offs on any model change post-hand-off; the prompt-change process documentation references this policy explicitly, naming the specific internal committee (often a Model Risk Committee or equivalent) that approves changes above a defined risk threshold. <!-- needs-research: confirm which DORA article governs ICT third-party exit requirements for Register-of-Information-tracked providers; Article 28 on general principles for ICT third-party risk and the EBA / ESA RTS on exit plans are the commonly cited references -->

## Anti-patterns

**The indispensable FDE.** The pilot is successful; the customer requests the FDE "for one more quarter"; the FDE agrees without routing through `ai-engineering-manager`; the FDE is now effectively a Professional Services contractor against the vendor's FDE budget. The hand-off never happens. The counter: the readiness gate is scheduled from day 1 of delivery, not negotiated at the end; the last-mile duration is agreed up-front; SOW-shaped asks route to PS.

**The undocumented prompt.** The pilot's production prompts live in a Notion page the FDE wrote. There is no version history, no PR trail, no rollback path. At hand-off, the customer engineer can see *what* the prompt is but not *why* it is that way. The counter: prompts are versioned in the pilot repo from day 1; prompt changes are PRs; the Notion page is at most an index.

**The no-pair period.** Pairing starts in the last two weeks of the pilot. The customer engineer has not touched the code; the hand-off is nominal. The counter: pair-programming starts 6–8 weeks before exit, not 2; the ramp is agreed in writing; week-1 pairing is on calendar before week 1.

**The "one more sprint" extension.** At the hand-off date, the customer sponsor asks for "one more sprint" to finish a feature. The FDE agrees. The sprint extends. A second sprint is requested. The engagement drifts toward permanent embed. The counter: scope-check per [Chapter 06](./06-scope-discipline-and-decision-log.md); the one-more-sprint pattern is a PS conversation, not an FDE extension; `ai-engineering-manager` is in the conversation before any commitment.

**The runbook nobody reviewed.** The FDE writes the eight documentation artefacts in the final two weeks. The customer engineer skims them, signs the gate, and discovers at the first 3am page that the on-call runbook assumes knowledge the customer engineer does not have. The counter: the fresh-reader review by a customer engineer who did NOT pair with the FDE is a gate item, not a courtesy; gaps surfaced in that review are fixed before sign-off.

**The on-call-not-shadowed.** The FDE hands off the pager with a runbook but no shadow rotations. The customer engineer takes their first page alone, with no muscle memory for the triage patterns. The counter: two rotations shadowed in sequence (customer-shadows-FDE, then FDE-shadows-customer) before the customer engineer takes primary; a single rotation is insufficient.

**The gold-set that stayed in the FDE's laptop.** The gold set accumulated through the pilot lives in a spreadsheet on the FDE's machine, synced nowhere. At hand-off, the FDE exports it to CSV, hands it over, and the provenance labels are lost in translation. The counter: gold-set storage is customer-owned from day 1, not exported at hand-off; provenance labels are a schema field, not a comment column; the gold-set-management SOP names the SME lead as the ongoing owner.

**The hand-off with no readiness gate.** The FDE and the customer engineer agree verbally that things are ready; the FDE exits; three weeks later a critical issue surfaces the customer engineer is not equipped to handle. The sponsor and `ai-engineering-manager` were never asked to co-sign. The counter: the readiness gate is the only mechanism by which the FDE exits; verbal agreement is not an exit. Both signatures. Dated. Countersigned.

**The FDE-authored PR trail.** The FDE wrote 60 % of the commits in the last 2 weeks because "it was faster." The git blame reads like a single-author project; the customer engineer has no fingerprint on the pilot's recent history. At the first incident, the customer engineer has to read code they did not write, while under pressure. The counter: the customer-engineer commit-share signal from the measurement table above; falling behind target is a sponsor-visible event, not an FDE optimisation.

**The Slack-as-runbook anti-pattern.** The runbook in `docs/` is thin; the real triage knowledge lives in the pilot's Slack channel scrollback. The customer engineer "has access to the channel" and is told to search when stuck. Six months in, Slack's retention policy has rolled over the channel and the knowledge is gone. The counter: every Slack triage that produced a learning is distilled into the on-call runbook within the same week; the Slack channel is a conversation, not a documentation surface.

**The hand-off that ignores the customer's change-management process.** The FDE's runbook assumes the customer engineer can deploy to production at will. The customer's actual change-management process requires a 48-hour advance ticket, a change-advisory-board approval, and a named rollback window. The runbook is unusable as written. The counter: the deployment runbook is co-authored with the customer's `platform-engineer`, incorporates the customer's actual change-management constraints, and is exercised against those constraints during the three walkthrough cycles.

## What the FDE does after exit

The FDE's calendar after hand-off is instructive — it is the behavioural evidence that the exit landed. Three signals the FDE and `ai-engineering-manager` watch:

- **Capture-log entries flip from this-pilot to patterns.** The FDE's contributions to the mod-111 capture log in the two weeks after exit should be pattern-oriented ("here is what this archetype's gold-set-curation handshake typically requires") rather than operational ("here is today's gold-set change"). Operational entries at month 2 post-exit are a signal the FDE did not actually detach.
- **The next pilot is scheduled.** An FDE whose next pilot is "TBD, waiting for X customer's questions to slow down" is an FDE whose exit did not complete. `ai-engineering-manager` schedules the next pilot's scoping conversation to start within 2 weeks of the exit date.
- **The pattern-catalogue promotion happens.** If the pilot produced a pattern that has now appeared across two FDEs' pilots (per [mod-111 Chapter 06](../mod-111-field-to-product-loop/06-fde-team-knowledge-base.md)), the catalogue promotion PR is the FDE's final artefact for the pilot. The pilot exits are complete when the knowledge base entry is merged.

A successful hand-off produces three outputs: a customer engineering team that can operate the pilot without the FDE, a vendor-side knowledge-base entry that compounds for the next pilot, and an FDE calendar that is available for new work. All three must land.

## Summary

- The FDE's goal at hand-off is to become unnecessary. The FDE is design-partner engineering, not permanent embedded staff; the receiver is the customer's own `applied-ai-engineer`, `platform-engineer`, and (where present) `ai-platform-engineer`, with the SME lead on gold-set curation and the sponsor on outcome metrics.
- Four hand-off streams run in parallel through the last 6–8 weeks: pair-programming on a weekly ramp from 80/20 to 10/90 (FDE/customer), documentation craft that ships eight artefacts with the pilot code, deployment runbook walkthroughs in three cycles (co-walk → observe → notified-only), and on-call shadowing in both directions before the customer engineer takes primary.
- Documentation ships with the pilot code, not in a Notion page the FDE wrote. Eight artefacts: architecture overview, deployment runbook, eval-pipeline runbook, on-call runbook, prompt-change process, gold-set SOP, localised security-review-answer library, decision log. ADR-style "why" on every non-obvious choice; copy-pasteable examples; every runbook links to the regression fixture or eval it protects.
- The hand-off readiness gate is a 14-item checklist co-signed by the customer sponsor and the FDE's `ai-engineering-manager`. Both signatures required; partial credit is not a thing. The fresh-reader check by a non-pair-partner customer engineer catches the shared-vocabulary blind spots.
- The last-mile pattern is a 4–6 week reduced engagement (one day per week, questions-only, no FDE commits) that is **not** a billable Professional Services engagement. SOW-shaped asks route to PS per [mod-101 Chapter 04](../mod-101-fde-discipline-and-position/04-adjacent-customer-facing-roles.md).
- The adjacent-role handshake map names a specific customer-side owner for each responsibility; where the customer lacks an `ai-platform-engineer`, the fallback owner is named explicitly.
- Archetype A integrates on-call with the payer's SOC rotation and routes breach-notification through the privacy office; Archetype B happens inside the agency's enclave and includes ATO-inheritance; Archetype C maps the readiness gate onto DORA ICT third-party exit requirements with model-risk function countersignature.
- Eight anti-patterns: the indispensable FDE, the undocumented prompt, the no-pair period, the "one more sprint" extension, the runbook nobody reviewed, the on-call-not-shadowed, the gold-set that stayed in the FDE's laptop, the hand-off with no readiness gate.
- Measurement is weekly, not end-of-window: customer-engineer commit share, PR-author share, runbook-coverage of recent incidents, and FDE-originated Slack messages in the pilot channel. Signals off target by more than 20 % at the mid-point surface at the sponsor review, not at the exit meeting.
- The post-hand-off contact protocol names the channel (shared, not DM), the SLA (best-effort, 2 business days, not a support contract), the in-scope question shape (design intent, not new build), and the expiry (typically 90 days after exit).
- The hand-off is complete when three outputs land: the customer engineering team operates the pilot without the FDE, the vendor-side knowledge-base entry is merged, and the FDE's next pilot is scheduled.
- Chapter 08 is next: role honesty in customer-facing communications — the pressured boundaries operationalised for the executive meetings where the customer's VP asks the FDE about pricing, roadmap, or renewal terms.
