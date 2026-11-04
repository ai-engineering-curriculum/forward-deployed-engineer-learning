# Chapter 06 — Blameless Post-Mortems and the Field-Feedback-into-Eval Back-Feed

## Why this chapter exists

The on-call answered the page, ran the playbook, rolled back the bundle, updated the status page, and closed the incident. The pilot is nominally healthy again. **The incident has not produced value yet.** The value arrives when:

- A **blameless post-mortem** lands within the agreed window (72 hours for SEV-1 / SEV-2 is the common target) with a defendable narrative of what happened, why it happened, and what will prevent recurrence.
- At least one **regression fixture** is permanently committed to the pilot's eval gate ([mod-107 Chapter 03](../mod-107-eval-eng-on-customer-data/03-offline-eval-design.md)) — if the same prompt / retrieval / model path arrives again on a future bundle, the gate fails *before* it reaches production.
- At least one **labelled gold-set entry** is added for every customer bug report that touched this incident, feeding the gold set from [mod-107 Chapter 02](../mod-107-eval-eng-on-customer-data/02-customer-gold-set-bootstrap.md) with the exact shape of the customer's problem.
- A **judge-rubric update** or a **playbook edit** or a **dashboard improvement** lands, so the next occurrence of the same class is detected earlier.

Every incident that does not produce at least the first two of those outcomes is an incident the team paid for and did not learn from. This chapter is the discipline that converts the paid-for cost into a compounding quality curve.

The chapter has three jobs. Define the blameless post-mortem for a shared customer-production system — who writes it, who reviews it, what template it follows, who signs it. Specify the regression-fixture back-feed — how a trace flagged `real_regression` in the LLM observability platform (Chapter 03) becomes a permanent entry in the eval gate. Specify the customer-bug-report pipe — how a ticket reaching the customer's operations analyst converts to a gold-set entry in the same system.

## What "blameless" is and what it isn't

Blameless post-mortems come from the [Google SRE Postmortem Culture chapter](https://sre.google/sre-book/postmortem-culture/) and the broader resilience-engineering literature (Dekker, Allspaw, Woods). The claim in one sentence: *the engineer who introduced the change, the reviewer who approved it, and the on-call who responded each acted in good faith with the information they had; the incident's cause is in the system, not the individual.*

Blameless is **not**:

- "Nobody is responsible." Actions have ownership; individuals own their actions; the post-mortem names the actions.
- "Everyone is a victim." Users suffered; customers paid; the harm is real. The emotional posture is seriousness, not evasion.
- "We can't say what went wrong." Specificity is non-negotiable. A post-mortem that reads "something in the pipeline failed" produces no learning.

Blameless **is**:

- A commitment that the output is *systemic improvement*, not personnel action. If an individual's judgement is genuinely the root cause, the fix is still in the system — a review step they missed, a signal they weren't given, a dashboard they couldn't see.
- A commitment that engineers can describe their mistakes in writing without consequence to their employment. The reciprocal is that engineers describe their mistakes fully.
- A commitment to specificity. Named change. Named action. Named timing. Named counterfactual.
- A commitment to generate at least one durable artefact (regression fixture, playbook edit, judge-rubric update, dashboard improvement) per incident — else the post-mortem is theatre.

## The post-mortem template

One template, one shape, versioned alongside the playbook library. The template lands as a Markdown or Confluence / Notion / Google Docs document in the customer's incident-management tool, linked from the closed incident, signed by the IC and the FDE.

```markdown
# Post-mortem: [incident title]

- **Incident ID:** INC-...
- **Date / time:** 2026-11-04T14:03Z – 2026-11-04T14:42Z (UTC)
- **Severity:** SEV-2
- **Customer impact:** Elevated response time and degraded answer quality on Member Assist affecting approximately 2,400 member inquiries over 35 minutes.
- **Detection:** Automated — fast-burn SLO alert on `pilot-member-assist.quality.online-judge-pass-rate` (Chapter 04).
- **IC:** Customer operations lead (<name>).
- **Scribe:** Customer operations analyst (<name>).
- **LLM SME:** Vendor FDE (<name>).

## Timeline

[Copied from the incident timeline, filtered to key events.]

## What happened

[Two to four paragraphs. Specific. Names the change — "bundle v4.2.0 promoted at 13:45Z changed the response-format prompt from a bullet-list directive to a free-form directive." Describes the user-visible shape — "the judge rubric penalises paragraph-form responses on inquiry types where SMEs rated bullet-list structure higher." Links to traces, dashboards, SLO burn panels.]

## Why it happened — the five whys, applied forward

A chain of causation from the user harm back to the systemic cause. Not a list of unrelated contributors; a chain.

1. The user saw slow / low-quality answers.
2. Because the online judge rated the new paragraph-form responses lower.
3. Because the prompt-template v4.2.0 rewrote the response-format directive in a way that changed the rated dimensions.
4. Because the offline eval set did not cover the bullet-list vs paragraph distinction as a scored dimension.
5. Because the judge rubric was authored before the SME team formalised their preference for bullet-list responses in high-volume intents.

## What was done well

[One or two paragraphs. Specific. The fast-burn alert fired within 18 minutes of the first judge-fail trace; the on-call acknowledged within one minute; the rollback via kill-switch executed in under three minutes. The communications cadence held on the public status page. Celebrating what worked is part of the design.]

## What was missed

[One or two paragraphs. Specific. The pre-promotion offline eval did not test against a bullet-list-vs-paragraph dimension; the canary expansion from 1 % to 10 % happened on an automation window that did not require a 24-hour hold at 1 % for a prompt-only change, which would have surfaced the issue during canary.]

## Durable artefacts produced (the back-feed)

Non-negotiable. Every post-mortem names at least two.

- **Regression fixture.** [Fixture name and link]. The specific prompt / retrieval context / expected-shape that would catch this regression at the eval gate. Links to the fixture's PR into the eval repo.
- **Gold-set entry.** [Entry name and link]. A labelled gold-set entry derived from one or more of the affected member inquiries, de-identified per the data contract, added to the pilot's gold set.
- **Judge-rubric update.** [Rubric change or no-change-needed, with reasoning.] If the judge rubric needs a new scored dimension, the rubric change PR is linked.
- **Playbook edit.** [Playbook or no-change-needed.] The prompt-regression playbook adds a specific check for "bullet-list vs paragraph in response-format" as a triage step.
- **Dashboard improvement.** [Dashboard or no-change-needed.] A new panel in the LLM observability platform's daily-dashboard that surfaces "response-format distribution by intent class" so the SME team spots shape drifts earlier.
- **Canary gate update.** [Gate or no-change-needed.] The gate from [mod-109 Chapter 04](../mod-109-deployment-rollout-change-management/04-canary-shadow-ab-rollouts.md) adds a check for prompt-only changes requiring a 24-hour hold at 1 %.

## Action items

A short list of named-owner, due-dated items. Each ties to one of the artefacts above.

| Item | Owner | Due | Status | Links |
|---|---|---|---|---|
| Add fixture `member-inquiry-bullet-vs-paragraph-regression` to eval gate | <FDE name> | 2026-11-08 | Open | PR #1234 |
| Add 20 de-identified gold-set entries from INC-42 affected inquiries | <customer analyst name> | 2026-11-11 | Open | Dataset #87 |
| Update judge rubric `judge-member-inquiry-v4` to score response-format fit | <FDE name> | 2026-11-15 | Open | PR #1240 |
| Update canary gate to require 24h hold at 1% for prompt-only changes | <customer release manager> | 2026-11-18 | Open | mod-109 Chapter 04 gate manifest |
| Add response-format-by-intent panel to daily dashboard | <customer operations analyst> | 2026-11-12 | Open | Langfuse dashboard #42 |

## Counterfactual

[One paragraph. Honestly: what if the canary had held at 1 % for 24 hours? What if the eval had tested bullet-vs-paragraph? Answer, in each case: the incident would not have reached 100 % traffic. This paragraph is the reader's primer on how much the back-feed items are worth.]

## Attachments

- Incident timeline (link)
- Chat log from `#inc-42-member-assist` (link)
- Judge-fail trace sample (link to Langfuse dataset)
- Previous bundle (v4.1.3) comparison traces (link)
- Status-page history (link)

## Sign-off

- IC: ___________________________________________ Date: _______
- LLM SME: ______________________________________ Date: _______
- Customer operations lead: ______________________ Date: _______
```

The template is one page expanded to three. The What-went-well, What-was-missed, and Counterfactual sections are the ones most frequently skipped and the ones most essential to a learning system.

## The review meeting

Every SEV-1 and SEV-2 post-mortem gets a live review. A 30-minute meeting within a week of the incident, attended by:

- The IC.
- The scribe.
- The LLM SME (vendor FDE).
- The release manager.
- The SME lead (so field feedback lands cleanly).
- The customer operations lead.
- A rotating peer reviewer from outside the pilot — another FDE, or `ai-platform-engineer`. This is the structural enforcement of blamelessness: an outsider's read catches the "too kind to ourselves" drift.

The meeting's output is sign-off on the action-items table. Open items roll into the next post-mortem's review if they slip; a slip is itself a learning signal.

## The regression-fixture back-feed — the mechanics

A trace from an incident becomes a fixture through a specific pipeline. The pipeline is the single most important plumbing the FDE builds in mod-110.

### Step 1 — SME annotation in the LLM observability platform

Inside Langfuse / Phoenix / Weave (Chapter 03), the SME opens the trace that caused the incident, reviews the inputs and outputs, and applies structured annotations:

- `review_verdict`: `real_regression` | `expected_behaviour` | `judge_mistake` | `data_issue`.
- `failure_mode`: one of the six from Chapter 05.
- `root_cause_classification`: short tag (`prompt_change`, `retrieval_stale`, `model_drift`, `cost_overshoot`, `guardrail_overblock`, etc.).
- `reproduction_notes`: free-text — what the SME did to reproduce or verify.
- `suggested_correction`: free-text or structured — what the ideal output would have been.

The annotations are structured fields on the trace record; they are stored in the platform's Postgres metadata store (encrypted, retention per the pilot's policy).

### Step 2 — Trace-to-fixture conversion

A tool (shipped with the pilot's eval harness) reads traces annotated `real_regression` and emits a fixture:

```python
# scripts/trace_to_fixture.py — part of the eval harness
def trace_to_fixture(trace, dataset_id):
    fixture = {
        "fixture_id": f"inc-{trace.incident_id}-{trace.trace_id[:8]}",
        "source": "incident_backfeed",
        "incident_ref": trace.incident_id,
        "pilot_id": trace.attributes["pilot.tenant_id"],
        "input": _redact_pii(trace.input),
        "retrieval_context_ids": trace.attributes["retrieval.document.ids"],
        "expected": trace.sme_annotations.suggested_correction,
        "scored_dimensions": _derive_dimensions(trace),
        "failure_mode": trace.sme_annotations.failure_mode,
        "created_at": trace.timestamp,
        "retention_class": "regression_permanent",
    }
    _attach_to_dataset(fixture, dataset_id)
    _open_pr(fixture)  # PR into the eval repo for FDE + customer review
    return fixture
```

The fixture PR is reviewed by the FDE and the customer SME lead. On merge, the eval harness runs the fixture against the current bundle to verify the fixture *actually catches* the regression (if the current bundle rolled back to v4.1.3, the fixture should pass; if it ran against v4.2.0, it should fail). If the current bundle passes, the fixture is marked `active`; if it fails, that's expected and the fixture is marked `active, failing-on-rollback-candidate` until the regression is fixed.

### Step 3 — Permanent in the eval gate

The fixture is in the pilot's offline eval set (per [mod-107 Chapter 03](../mod-107-eval-eng-on-customer-data/03-offline-eval-design.md)); every CI run and every bundle promotion runs it. The retention class `regression_permanent` means the fixture is never removed without a review (an explicit deprecation PR naming why).

The fixture's input / expected are PII-redacted per the data contract ([mod-108 Chapter 05](../mod-108-enterprise-security-and-compliance/05-data-contract.md)); the eval harness uses the redacted form. For an Archetype A pilot, PHI never enters the eval repo; the redaction happens inside the content-cleared destination (Langfuse self-hosted) and the fixture that leaves it carries placeholder tokens.

### Step 4 — Judge-rubric update if warranted

If the regression revealed a dimension the existing judge rubric didn't score (the bullet-list example above), the rubric gets a new dimension in a separate PR. The rubric version bumps; Chapter 02's `judge.rubric.version` attribute reflects the change; the eval harness re-runs historic comparisons on the new rubric to establish the expected-pass line.

## The customer-bug-report pipe

The post-mortem flow starts with an alert. A customer bug report starts with a human — a user who told the support line "the assistant said the wrong thing." The bug report is a lower-signal input than an alert but a higher-value learning if the system captures it.

The pipe:

### Step 1 — Capture at intake

The customer's support tool (Zendesk / ServiceNow / internal) has a classification for "AI assistant quality" or similar. Every ticket categorised this way flows into a pilot-owned bug intake queue, with the trace ID (if the user interaction captured one) attached.

### Step 2 — SME triage

A customer SME reviews the ticket within one business day. Three outcomes:

1. **Not a pilot issue.** Classified; closed; no further action.
2. **Pilot issue, isolated.** The ticket's specific case gets a gold-set entry with the user's inquiry, the model's answer, and the SME's view of the correct answer. The gold-set entry labels: `label=sme_disagreement`, `source=customer_bug_report`, `ticket_ref=<id>`.
3. **Pilot issue, pattern.** Multiple tickets of similar shape in a short window → open an incident (SEV-3 or SEV-2 depending on magnitude); run the post-mortem flow. The incident uses the normal back-feed.

### Step 3 — Weekly pattern review

A weekly meeting of the SME lead and the FDE reviews the week's intake: how many tickets, what classifications, which are being seen repeatedly. A ticket that surfaces three weeks in a row is a signal the eval gate missed something — it gets lifted into an incident retroactively.

### Step 4 — Gold-set hygiene

Every gold-set entry sourced from a bug report carries its ticket ref; the eval harness reports on "coverage of gold-set entries by source" — how many are ticket-sourced, how many are proactively bootstrapped, how many are regression-fixtures. A gold set that is 100 % proactively bootstrapped is a gold set that is not reflecting the customer's real distribution; a gold set that is 100 % ticket-sourced is a gold set driven by recent complaints only. A healthy balance typically lands 40 % proactive bootstrap, 30 % regression fixtures, 30 % bug-report-sourced. (The ratio is a signal, not a target; it drifts with the pilot's stage.)

## The organisational flywheel

The two pipes compose. A high-functioning pilot's monthly rhythm:

- **Every incident:** at least one regression fixture + at least one playbook / dashboard / rubric update.
- **Every week:** a bug-report intake pass; three to ten gold-set entries added.
- **Every month:** a quality trend review — did the SLOs trend favourably? how many incidents? what fixtures? what common root causes across the fleet (shared via `ai-platform-engineer`)?
- **Every quarter:** an SLO retrospective ([mod-104 Chapter 05](../mod-104-llm-app-in-customer-envs/05-per-customer-cost-latency-slos.md) referenced); a playbook retest (drills from Chapter 05); a dashboard review; a judge-rubric calibration.

The flywheel's measurement is **fixture coverage** — the fraction of past incidents that, if their exact prompt re-appeared, would be caught at the eval gate. A healthy pilot approaches 100 % after six months; a stalled pilot sees fixture coverage drift down (meaning new incident classes are appearing faster than fixtures are being written).

## Feeding `ai-platform-engineer` and mod-111

The pilot's fixtures and the pilot's bug-sourced gold-set entries are per-pilot artefacts. Many of them are specific to the customer (this customer's member-inquiry language, this customer's policy taxonomy). Some are *generic* — a prompt-injection shape, a provider-drift signal, a PII-leak near-miss. The generic ones are the input to mod-111 and to `ai-platform-engineer`'s cross-fleet patterns.

The FDE tags every fixture with a `generalisability` classification:

- `pilot_specific` — not transferable; this customer's language or data.
- `pattern_potentially_shared` — a shape that might show up in other pilots; worth reviewing.
- `platform_level` — a generic shape (prompt injection, specific guardrail bypass, specific provider behaviour) that should go into the vendor's shared eval library.

The `platform_level` entries are the pipe into mod-111's field-to-product loop; `ai-platform-engineer` reviews them weekly and lifts the valuable ones into the shared judge / eval / guardrail libraries.

## Running the three archetypes

**Archetype A — US healthcare payer.** Post-mortems in the customer's Confluence-based IR documentation, with the FDE drafting the LLM-SME narrative; sign-off by the customer's member-services operations lead. Regression fixtures land in a customer-owned Git repo in GitHub Enterprise under the customer's AWS CodeCommit; de-identification per HIPAA before the fixture leaves the Langfuse content-cleared environment. Bug-report pipe: Zendesk tickets classified `member_assist_quality` flow into a pilot queue; weekly review by the SME lead. Fixture-coverage reporting monthly. HIPAA breach-notification post-mortems have an additional privacy-office sign-off (per [Chapter 05](./05-oncall-rota-and-llm-triage.md) on HIPAA reporting).

**Archetype B — US federal agency.** Post-mortems in the agency's SharePoint / M365 GCC High environment; sign-off by the agency's system owner and the ISSO. Regression fixtures land in an on-prem Git (GitLab or Bitbucket Server) inside the enclave. Bug-report pipe: agency's internal ticketing tool; weekly SME review. FedRAMP continuous-monitoring reporting on any incident touching controlled information; POA&M item opened for any control breach. `ai-platform-engineer` back-feed is pilot_specific-only by default (no cross-pilot generalisation without ATO review).

**Archetype C — EU retail bank.** Post-mortems in the bank's Confluence / ServiceNow integration; sign-off by the bank's operations lead and the model-risk function (SR 11-7 equivalent). Regression fixtures land in a bank-managed GitLab inside Azure; de-identification per GDPR Article 32 + bank's data classification before fixture lands in the eval repo. Bug-report pipe: bank's customer-service ticketing; weekly SME review. DORA major-ICT-incident post-mortems feed the bank's DORA reporting to the ESAs. EU AI Act Article 72 post-market-monitoring record pulls from every month's fixture-coverage report. <!-- needs-research: confirm EU AI Act Article 72 post-market monitoring reporting shape and cadence for the specific deployer obligations in the bank's use category -->

## Anti-patterns

**Post-mortem as a slide deck.** A 15-minute presentation; nobody writes the three-page document; the action items are on the slide and nowhere else. Six weeks later the same failure recurs. Chapter 06's template is mandatory; the slide deck is optional.

**Action items without owners and due dates.** The template section is filled, but every row says "TBD" and "someone." The item ages into the backlog and dies. Every row has a name and a date; slip is reported at the next review.

**Regression fixture promised but not delivered.** The action item "add fixture for INC-42" closes because "the fixture is in the dataset"; the eval harness never runs it against bundle promotions; the guard is theatrical. Chapter 06's fixture pipeline requires the merged PR to be verifiable via the eval harness's next run; closure without run-verification is not closure.

**Bug reports treated as customer-service tickets only.** Zendesk tickets close with a reply to the user; the SME lead never sees the pattern; the gold set never grows with customer reality. Chapter 06's bug-report pipe is a weekly meeting with a named participant list.

**Pilot-specific fixture tagged platform-level.** The fixture leaks customer language or data into a shared library; a second customer discovers details about the first. The generalisability tag is a data-contract-grade decision; FDE and SME lead both sign.

**Blame disguised as process.** The post-mortem names the engineer who deployed the change ("Alice promoted v4.2.0 without sufficient eval coverage"); the systemic fix is a training requirement for Alice. That is blame wearing process clothes. The systemic fix is the eval coverage, the canary gate update, and the dashboard panel — not the training.

**No counterfactual.** The template's counterfactual section is skipped because "it feels speculative." The counterfactual is where the back-feed value is quantified; skipping it means the action items have no priority ranking.

**Fixture coverage never measured.** The team adds fixtures diligently but never measures what fraction of past incidents they catch. Six months in, a regression slips through the eval gate — the fixture for that class exists but was retired for being "flaky." Fixture-coverage reporting is a monthly artefact.

**Review meeting without a peer reviewer.** The post-mortem review is the IC and the FDE, both of whom were in the incident; the discussion drifts toward self-congratulation or self-criticism in roughly equal measure. The rotating peer reviewer from outside the pilot is the structural check against both.

## Summary

- A blameless post-mortem lands within 72 hours (SEV-1 / SEV-2) with a defined template: timeline, what-happened, why-it-happened (chain of causation), what-worked, what-was-missed, durable artefacts, action items, counterfactual, sign-off.
- "Blameless" is specific: systemic fix, not personnel action; engineers can describe their mistakes without consequence; mistakes are described fully.
- The regression-fixture back-feed pipeline is four steps: SME annotation inside the LLM observability platform; trace-to-fixture conversion with PII redaction; merge into the eval gate as `regression_permanent`; judge-rubric update if warranted.
- The customer-bug-report pipe is four steps: intake classification; SME triage within one business day (closed / gold-set entry / open an incident); weekly pattern review; gold-set hygiene with source-mix reporting.
- The organisational flywheel: every incident produces at least one fixture and one playbook / dashboard / rubric update; every week bug reports feed the gold set; every month quality trends are reviewed; every quarter SLOs, playbooks, dashboards, and judge rubrics are retested.
- Fixture coverage is the measurement — the fraction of past incidents whose exact shape would be caught at the current eval gate. Healthy pilots trend toward 100 %; stalled pilots see the fraction drift down.
- Generalisability tags route the fixtures: pilot_specific, pattern_potentially_shared, platform_level. Platform-level is the input into [mod-111](../mod-111-field-to-product-loop/) and `ai-platform-engineer`'s shared libraries.
- Archetype A integrates with HIPAA breach-notification post-mortems and a customer-owned fixture Git; Archetype B lands post-mortems in SharePoint GCC High and feeds the FedRAMP POA&M; Archetype C integrates with DORA major-ICT reporting and EU AI Act Article 72 post-market monitoring.
- Nine anti-patterns: post-mortem as slides, action items without owners, fixtures promised-not-delivered, bug reports as support tickets only, mis-tagged generalisability, blame-in-process-clothing, no counterfactual, unmeasured fixture coverage, no peer reviewer.
- This is the module's final chapter. The pilot is now instrumented (Chapter 02), observed on an LLM-specific platform (Chapter 03), running against defined SLOs (Chapter 04), paged correctly on a joint rota (Chapter 05), and generating permanent memory in its eval gate through every incident and every bug report (Chapter 06). What was once a one-shot deployment is now a learning system.
