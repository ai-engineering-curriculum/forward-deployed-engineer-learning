# exercise-01: Field Telemetry Capture with Bug / Feature / Platform-Gap Triage

**Estimated effort:** 2.5 hours

## Objective

Set up a working capture log for one pilot archetype, populate it with ten realistic signals drawn from a simulated pilot week, and run one weekly triage ritual on them with a peer-FDE partner — producing a confirmed triage for every signal and the first-pass artefact pipeline (which signals become platform-request packets, which become feature requests, which get fixed in the pilot, which stay parked as `platform_gap_candidate`).

The point is not to produce a beautiful knowledge base. The point is to make the capture discipline and the triage model so concrete that in a real pilot the FDE spends five minutes per capture and forty-five minutes per weekly ritual rather than vaguely "keeping notes."

## Prerequisites

- Read [Chapter 02](../02-field-telemetry-and-triage.md) carefully; skim [Chapter 01](../01-field-to-product-overlay.md) for the overlay vocabulary.
- From mod-110 Chapter 06, the regression-fixture back-feed pipeline and the generalisability taxonomy (`pilot_specific`, `pattern_potentially_shared`, `platform_level`); your capture-log entries inherit this.
- From mod-108 Chapter 05, the data-contract vocabulary — this determines what content can legitimately appear in the `observation` field.
- Choose a partner for the weekly ritual — another learner, a team peer, or (if solo) a role-played "peer FDE" persona you commit to writing their comments in a distinct voice.

## Problem statement

Pick **one** of the three archetypes from Chapter 01:

- **Archetype A — US healthcare payer.** Member-inquiry resolution; HIPAA + SOC 2 Type II; AWS Bedrock + VPC install.
- **Archetype B — US federal agency.** Policy-analyst drafting; FedRAMP Moderate; GovCloud + self-hosted inference.
- **Archetype C — EU retail bank.** Customer-support reply + case triage; GDPR + DORA + EU AI Act; Azure AI Foundry under EU Data Boundary.

Imagine you are the FDE on this pilot in month six of a twelve-month engagement. You will simulate a week of field signals, author ten capture-log entries using the Chapter 02 schema, then run a weekly triage ritual with a peer FDE (or role-played persona) and finish with a prioritised action list.

## Requirements

1. **Capture-log repo or folder.** Create a dedicated location (a Git repo, a Notion page, or a flat markdown folder) called `fde-capture-log/<your-pilot-id>/`. All ten signals live here. The location is vendor-internal by assumption; the archetype's sign-off discipline governs any downstream artefact, not the log itself.

2. **Ten signals, authored against the Chapter 02 schema.** Each signal is a single markdown file named `signals/YYYY-MM-DD-<slug>.md` with all required schema fields (`signal_id`, `pilot_id`, `customer_id`, `date_observed`, `captured_by`, `surface`, `source`, `source_refs`, `observation`, `sme_quotes`, `triage_initial`, `triage_confirmed`, `generalisability`, `related_signals`, `customer_sensitivity`, `links`). The ten signals must distribute across the six source modalities from Chapter 02:
   - At least two from **incidents** (e.g. an SLO-burn alert that fired, a kill-switch invocation).
   - At least two from **customer bug reports** (tickets from the customer's support tool, classified AI-quality).
   - At least two from **SME wishes** (verbatim quotes from the customer's SME lead or reviewer).
   - At least one from a **security-review question** (a non-boilerplate answer the FDE had to author).
   - At least one from a **pilot ADR** (a decision the FDE wrote this week).
   - At least one from a **weekly-review frustration** (an organisational / process signal, not a technical one).
   The signals should feel like they come from the same pilot — a coherent setting, consistent surfaces, cross-referencing where plausible.

3. **Observation discipline.** Every `observation` field contains **facts only**. Analysis, hypothesis, and opinion live in the triage fields or in the ritual output; a reviewer should be able to read the observation and form their own triage read. Any observation that fails this test gets a `<!-- facts-only-check: ... -->` note identifying which sentence crosses into analysis.

4. **SME quotes verbatim.** At least five of the ten signals carry at least one SME quote with a speaker role (not a named individual — role, since this is a learning exercise), a date, and a `consent_at_capture: confirmed | implicit | pending` field. The consent field reflects whether the archetype's SME-media-release discipline from Chapter 03 would permit the quote to appear in a downstream artefact.

5. **Initial triage per signal.** Author `triage_initial` on every signal using the three Chapter 02 buckets (`bug`, `feature`, `platform_gap_candidate`, or multi-value when applicable). Document your reasoning in a one-sentence `triage_rationale` field alongside the initial triage.

6. **Weekly triage ritual.** Run one ritual against the ten signals with your partner (or role-played peer persona). Capture the ritual output as a single `rituals/YYYY-MM-DD-weekly-triage.md` document following the Chapter 02 standing agenda (new-signals review / re-triage queue / action-item dispatch / cadence check). Each signal has a `triage_confirmed` value and, if different from `triage_initial`, a one-line `triage_rationale` that names why the ritual changed the call. At least one signal must be re-triaged during the ritual to prove the peer-FDE check is doing work (or an honest justification if genuinely none should be).

7. **Action-item dispatch.** From the ritual's output, produce an `actions/YYYY-Q<quarter>-dispatch.md` table. Each row: signal_id, confirmed triage, action owner, due date, destination (pilot repo PR / feature-request queue / platform-request-packet draft / parked). At least one action must route to each of: pilot PR, feature-request queue, platform-request-packet draft (draft, not submitted — this exercise does not require the Chapter 04 packet itself).

8. **Two-customer check.** For every signal triaged as a platform-gap (confirmed or candidate), document whether the two-customer rule from Chapter 02 is satisfied, is pending (candidate), or is being invoked under an exception path from Chapter 04. Each decision has a one-sentence justification.

9. **Archetype discipline.** At least five signals demonstrate archetype-specific capture decisions — HIPAA scoping on observations for Archetype A, dual-logging abstract for Archetype B, GDPR Article 32 scope for Archetype C — explicitly noted in the signal's `observation` or `customer_sensitivity` field.

## Starter guidance

- **Make the signals plausible, not dramatic.** The strongest capture logs are mundane. A Wednesday SME wish about citation depth, a Friday prompt-only canary that blew the online judge metric by three points, a Thursday security-review question about retrieval ACLs — these are the shapes a real pilot generates. Resist the urge to turn every signal into a SEV-1.
- **Vary the triage distribution.** A log that is 100 % `bug` is unrealistic; so is a log that is 50 % `platform_gap`. The healthy distribution per Chapter 03 is roughly 60 % bug, 25 % feature, 15 % platform gap (at any stage of a pilot); a one-week slice can deviate.
- **Let the peer FDE catch something real.** The partner's job in the ritual is not to agree with you. If every signal confirms as-authored, your initial triage was suspicious; if nothing confirms, your capture was suspicious. One or two re-triages per ten signals is a healthy ritual.
- **Use `<!-- needs-research: ... -->` for anything you cannot verify from a primary source** (e.g. HIPAA records retention, DORA major-ICT-incident criteria, OMB M-24-10 scope). Flag honestly rather than invent.
- **Don't shortcut the SME quotes.** The one-line quotes are the hardest field to author well, because they need to sound like a specific real person speaking. If every quote reads the same, re-read the capture entries and vary the SME's voice by their role (the SME lead is confident; the model-risk reviewer is cautious; the operations analyst is pragmatic).
- **For Archetype B**, if you cannot plausibly simulate controlled-information content, abstract to "a US federal pilot under FedRAMP Moderate with a policy-analyst workflow" and note the abstraction in each signal's `observation`. This is the real-pilot discipline.

## Acceptance criteria

- Ten signals in the capture-log location, each authored against the Chapter 02 schema with all required fields populated.
- The six source-modality distribution is satisfied.
- Every `observation` is facts-only; any that crosses into analysis is explicitly annotated with a `facts-only-check` note.
- At least five signals carry at least one verbatim SME quote with speaker role, date, and consent status.
- Every signal has `triage_initial` with a one-sentence rationale.
- One `rituals/*.md` document captures the weekly triage ritual output, with every signal receiving a `triage_confirmed` value and at least one re-triage.
- One `actions/*.md` dispatch table routes every signal to a destination — at least one to each of pilot PR, feature-request queue, platform-request-packet draft.
- Every platform-gap (confirmed or candidate) has a documented two-customer-rule status.
- At least five signals show archetype-specific capture decisions explicitly.
- At least three `<!-- needs-research: ... -->` markers appear across the ten signals, each with enough context that a reviewer could backfill.

## Stretch goals

- **Second-archetype crosswalk.** Pick three of your ten signals and rewrite them as if they had occurred in a different archetype (A→C, B→A, C→B). What changes in the `observation` field? What changes in `customer_sensitivity`? What changes in the triage? Produce a one-paragraph reflection on which fields are archetype-invariant and which are not.
- **Vocabulary contribution.** Review your ten signals' `surface` values. Propose a canonical vocabulary (five to ten terms) for the module's `surfaces` field, naming for each term the signals in your log that would map to it. This simulates the quarterly knowledge-base-review vocabulary-alignment step from Chapter 06.
- **Triage decision tree.** Author a one-page decision tree the weekly ritual would walk through to classify a signal. Verify your tree correctly classifies all ten of your signals when applied blind by your partner (they walk the tree without having pre-read your triage).
- **Capture-time measurement.** Time yourself on each of the ten signals. Report the median and p95 time-to-author per signal. If any signal takes more than fifteen minutes, identify which schema field drove the time and propose a template improvement.
- **Pre-ritual peer read.** Have your partner read the ten signals *before* the ritual and author their own `triage_initial` without seeing yours. Diff the two; discuss the divergences in the ritual; capture the diff and the resolution. This mirrors the structural blamelessness discipline from mod-110 Chapter 06.
