# Chapter 06 — The Pilot-to-Production Exit Eval — Pairwise Human-vs-AI, Blind SME Scoring, and the Governance Decision Document

## Why this chapter exists

The exit eval is the evidence packet the customer's governance board uses to approve go-live. It is not a deliverable the FDE writes for themselves. It is the artifact a Chief Risk Officer, a Chief Medical Officer, a records officer, a FedRAMP-liaison, or a model-risk committee reads and signs.

The offline portfolio of Chapters 03 and 04 and the online eval of Chapter 05 are necessary but not sufficient. A board that is being asked to put a workflow into production demands three things a per-metric dashboard does not deliver on its own:

1. **Human preference evidence.** The AI arm and the human arm compared on the same work, under a pre-registered protocol the board can defend when audited.
2. **Business-KPI uplift.** A credible estimate of the pilot's effect on the business number the pilot was commissioned to move, with confidence intervals.
3. **Residual-risk acceptance.** A register of what still goes wrong, who signed up to accept it, and under what conditions the acceptance expires.

This chapter is the design of that evidence packet. The pairwise comparison is the preference instrument. Blind SME scoring is how the preference instrument stays honest. The governance decision document is how the preference instrument, the KPI uplift, and the residual risks become a signature.

The goal of the exit eval is not to produce a passing grade. It is to produce a *defensible* grade — one that survives the auditor's question, "how did you decide to go live?"

## The pairwise human-vs-AI protocol

The core instrument is a pairwise comparison. For each case in a holdout slice, the customer compares a human-authored output to the pilot's AI output on the rubric columns the customer's workflow cares about.

**The holdout slice.** Carve ~20% of the Chapter 02 gold set as an exit-eval holdout from day 0. The holdout is sealed. It is not used in prompt tuning, retrieval tuning, offline-portfolio calibration, or judge-as-LLM calibration. The sealing is a process control, not a technical one — the FDE and the customer's data owner co-sign that no engineer has seen the holdout queries. Chapter 02 is where this is set up; this chapter is where it is spent.

**The two arms.** For each holdout case, produce:

- **Arm A — Human.** A human-authored output by the SME doing the work today. Sampled from real workflow outputs where possible (de-identified), authored fresh in a sit-with session otherwise. The human arm is the status quo; it is not a straw man.
- **Arm B — AI.** The pilot's output on the same case, generated with the production prompt, production retrieval, production tools, and the HITL shape the pilot ships with (mod-106 Chapter 04). If the pilot is HITL, the "AI output" is the AI-drafted output *before* the human edits; the edit record is captured separately as an auxiliary signal.

**Randomise assignment.** Which arm renders as "Output 1" and which as "Output 2" is randomised per rater per case. Humans anchor on the first-shown option (position bias). Half of cases should show the AI first; half should show the human first. Record the randomisation seed; the auditor will ask.

**Blind raters.** Three SMEs score each pair. The raters are different from the authors. They are cleared for the data. They do not know which output was authored by which arm. Stripping is not optional — see the next section.

**The rubric.** Score on the columns the customer's workflow cares about. Standard starting columns, adjusted per archetype:

| Column | What it measures | Scale |
|---|---|---|
| Correctness | Factual accuracy of claims and citations | 1–5 |
| Completeness | Whether the output addresses the full case | 1–5 |
| Tone | Register appropriate to the recipient | 1–5 |
| Policy compliance | Alignment with customer policy / regulatory constraints | 1–5 |
| Actionability | Whether the recipient can act on the output without rework | 1–5 |

Add an overall preference question: *"If you had to send one of these two outputs, which would you send?"* with options A, B, or tie. The overall preference is the headline; the rubric columns are the diagnostic.

**Aggregation.** For a small pool (dozens of cases, three raters), report:

- **Win rate.** Fraction of cases where the AI arm was preferred.
- **Tie rate.** Fraction of cases scored as a tie.
- **Loss rate.** Fraction where the human arm was preferred.
- **Per-slice breakdown.** Intent × role × freshness, mirroring Chapter 03's slicing. The board cares which slices win and which lose; a strong average with a losing slice is a go-with-conditions.

For larger pools (hundreds of cases, many raters, multiple AI variants), move to a pairwise rating model. **Bradley-Terry** fits a strength parameter per arm from pairwise outcomes; **Elo** updates strengths online. Both are the framework behind community pairwise leaderboards; see [Chatbot Arena / LMArena](https://lmarena.ai/) for the prevailing implementation. For a single AI arm vs. a single human arm, Bradley-Terry collapses to a simple win-rate test; the framework earns its keep when the board wants to compare multiple pilot variants or multiple human cohorts.

A pairwise case + rater scoring schema in YAML:

```yaml
case_id: EX-0317
holdout_slice: "exit-eval-v1"
intent: policy_lookup
role: csr_tier_1
freshness_bucket: "30-180d"
input_context: <sanitised case text and retrieved evidence>

outputs:
  arm_A:
    author: human_sme
    author_id: "SME-11"
    text: <output>
  arm_B:
    author: pilot_ai
    pipeline_version: "pilot-v1.3.2"
    text: <output>

randomisation:
  rendered_first: "arm_B"
  seed: "rand-0317-a9f3"

rater_scores:
  - rater_id: "R-04"
    correctness: 4
    completeness: 5
    tone: 4
    policy_compliance: 5
    actionability: 4
    overall_preference: "arm_B"
    free_text: "A missed the Gold-plan exception; B surfaced §4.2 explicitly."
  - rater_id: "R-07"
    correctness: 4
    completeness: 4
    tone: 5
    policy_compliance: 4
    actionability: 5
    overall_preference: "tie"
    free_text: "Both acceptable; A warmer, B more precise."
  - rater_id: "R-12"
    correctness: 3
    completeness: 4
    tone: 3
    policy_compliance: 4
    actionability: 4
    overall_preference: "arm_B"
    free_text: "A buried the action item; B led with it."

aggregated:
  majority_preference: "arm_B"
  mean_correctness: 3.67
  mean_policy_compliance: 4.33
```

Numbers above are illustrative, not drawn from a real pilot.

## Blinding the SME scoring

Blinding is a process control, not a nicety. If the rater can tell which arm is AI, the preference data is not evidence; it is a preference about AI, not about the output.

**Strip author metadata.** Remove signatures, user handles, author fields, revision histories, system-generated boilerplate the AI arm tends to emit (e.g., "Based on the retrieved policy..."), timestamps embedded in drafts, and model-version strings. The stripping script is a reviewable artifact; the data owner signs off on it before the rating session.

**Strip formatting tells.** The AI arm often has a recognisable formatting fingerprint — bullet patterns, section headings, a "Summary:" lead-in. Normalise formatting across both arms before rendering. The normalisation is not editorial; the content is untouched, only the layout is unified.

**Strip timing hints.** If the human arm's output was written in a session the rater knows happened last Tuesday, the rater guesses. Render both arms stamped with the same opaque case ID and nothing else.

**Pre-register the rubric.** The rubric columns, the scale, the overall-preference question, the aggregation rule (majority vote, mean score, Bradley-Terry), the per-slice cuts — all pre-registered before any rater sees any output. Post-hoc rubric edits are p-hacking the exit eval. If a rubric column is clearly broken mid-session (raters all confused, scores pile at one value), stop the session and re-baseline; do not quietly rewrite.

**Inter-rater reliability.** ~20% of cases rated by two raters; compute agreement:

- **Krippendorff's alpha** for ordinal 1–5 scales. Alpha ≥ 0.667 is the common "acceptable" bar; ≥ 0.8 is "high." The thresholds are domain-heuristic, not laws of nature. <!-- needs-research: Krippendorff 2011 acceptable-alpha thresholds exact wording -->
- **Fleiss' kappa** for nominal rubric columns (e.g., a pass/fail compliance flag).
- **Cohen's kappa** when only two raters overlap.

Low agreement is itself a finding: the rubric is under-defined, or the raters are drawn from populations with different working definitions. Report the finding; do not average it away.

**Rater training.** A 30-minute calibration round before the real session. Three sample cases (not from the holdout) are rated by all raters; discrepancies are discussed; the rubric wording is clarified; then the real session begins. The calibration cases are archived as part of the eval record.

## Business-KPI uplift measurement

The pairwise instrument captures preference. The business-KPI instrument captures value. Both go in the decision document.

**Pre-register the KPI.** The KPI was named in mod-102 Chapter 03 — time saved, deflection rate, escalation rate, handle time, mis-routing rate, first-contact resolution, analyst draft-time. The pre-registered target value is stated *before* raters see outputs and before KPI data is pulled. The pre-registration lives in the eval protocol document alongside the rubric.

**Where the pilot ran in shadow or HITL mode**, instrument the pilot to emit per-task cycle time, override rate, and acceptance rate. Compare AI-assisted cohort to non-assisted cohort. The comparison is only credible if the two cohorts are comparable on the dimensions that drive the KPI (case complexity, user tenure, time of day). State the comparability assumptions.

**Design options.** Three families for the KPI uplift measurement, in decreasing order of inferential strength:

| Design | Strength | When applicable |
|---|---|---|
| Randomised A/B at task level | Highest. Causal estimate by construction. | When the workflow allows random assignment and the regulator does not object. Rare in regulated workflows. |
| Interrupted time-series (ITS) | Moderate. Controls for pre-trend. | Pre / post with a modelled trend line; useful when the pilot turns on for everyone at once. |
| Pre / post with control arm | Moderate. Controls for concurrent changes. | A nearby team, region, or product line serves as the non-assisted control. |

Randomised A/B at task level is the ideal and is often impossible in regulated workflows — a payer cannot randomly deny AI assistance to half of member calls, and a bank cannot randomly withhold the pilot from half of complaints. ITS and pre/post with controls are the common fallbacks.

**The statistical bar.** The board will want a 95% confidence interval on the uplift. State the CI and the practical-significance threshold agreed with the business owner in mod-102 Chapter 03. The two together give the four-cell outcome:

| | Practically significant | Not practically significant |
|---|---|---|
| Statistically significant | Pass | Go-with-conditions; the measured effect is real but small |
| Not statistically significant | Insufficient data; continue the pilot | No-go on the KPI dimension |

A statistically-significant but practically-trivial uplift is a go-with-conditions, not a hard pass. The board is not measuring p-values; they are measuring whether this pilot earned its budget.

**A worked snippet** — illustrative, not from a real pilot:

```yaml
kpi: "handle_time_seconds"
pre_registered_target_reduction: 45  # seconds per call
practical_significance_threshold: 20  # seconds per call
design: "pre_post_with_control_arm"
control_arm: "team-west"
treatment_arm: "team-east"
observation_window:
  pre:  "2026-09-01 .. 2026-09-28"
  post: "2026-10-01 .. 2026-10-28"
observed_reduction_seconds: 37
confidence_interval_95:
  lower: 22
  upper: 52
decision: "pass"
notes: "CI excludes 20; practical-significance threshold cleared."
```

## The governance decision document

The exit eval produces a one-pager (with appendices) the board signs. The one-pager is readable in five minutes by a Chief Risk Officer who did not attend the pilot. The appendices are where the harness output, the pairwise raw data, and the KPI methodology live.

**Required sections.**

1. **Scope.** Workflow, archetype context, phases evaluated (shadow / HITL draft / HITL action / selective auto-run). Named user population. In-scope regions, products, or case types; explicit out-of-scope.
2. **Evidence summary.** Latest offline portfolio metrics (Chapter 03 slice table), online rolling-window metrics (Chapter 05), pairwise outcome (win / tie / loss per slice), KPI uplift with CI. Each with a pointer to the appendix row.
3. **Residual-risk register.** Known failure modes — slice losses, drift observations from the rolling window, adversarial-test outcomes from mod-108 — each with a mitigation: HITL thresholds, kill-switch, rollback plan, incident-response runbook from mod-110. Risks without mitigations are either accepted or blocking; the register says which.
4. **Decision.** Go / go-with-conditions / no-go. Conditions, if any, are enumerated with owners and deadlines. "Go-with-conditions" is not "we'll figure it out" — each condition has a verification step and a date.
5. **Signatories.** The humans who sign. Varies by archetype; typical columns below.
6. **Review cadence.** Quarterly at minimum. Earlier triggers: drift threshold breach from Chapter 05, incident of named severity, major model-version change, major corpus / policy change, regulatory event. Each trigger names who re-opens the decision.

A YAML template of the one-pager:

```yaml
decision_document:
  id: "gov-exit-eval-pilot-v1"
  issued: "2026-11-04"
  scope:
    workflow: "member inquiry resolution — policy lookup + reply draft"
    archetype_context: "Archetype A — US healthcare payer"
    phase: "HITL action"
    population: "Tier-1 CSR, US region"
    out_of_scope: ["prior-auth decisions", "clinical advice"]
  evidence_summary:
    offline_portfolio:
      ndcg_at_10: 0.78
      hallucination_rate: 0.018
      slice_regressions: ["freshness:<30d supervisor queries"]
      appendix_ref: "A.1"
    online_rolling_window:
      window_days: 28
      sample_rate: 0.05
      judge_pass_rate: 0.91
      drift_flag: false
      appendix_ref: "A.2"
    pairwise:
      cases: 140
      ai_win_rate: 0.52
      tie_rate: 0.31
      human_win_rate: 0.17
      per_slice_losses: ["escalation_triage"]
      appendix_ref: "A.3"
    kpi_uplift:
      metric: "handle_time_seconds"
      observed_reduction: 37
      ci_95: [22, 52]
      practical_threshold: 20
      appendix_ref: "A.4"
  residual_risk_register:
    - risk: "Hallucinated policy citation on fresh-policy queries"
      observed_rate: 0.008
      mitigation: "HITL confirm required when retrieval confidence < 0.6"
      owner: "FDE + CSR supervisor"
    - risk: "Tone regression on escalation-triage slice"
      observed: "Pairwise loss rate 0.22 on this slice"
      mitigation: "Route escalation_triage intent to human queue in Phase 1"
      owner: "Clinical ops lead"
  decision:
    outcome: "go-with-conditions"
    conditions:
      - condition: "Route escalation_triage to human queue for first 60 days"
        verification: "0 auto-handled escalation_triage cases in audit log"
        deadline: "2027-01-03"
      - condition: "Quarterly re-eval of fresh-policy slice"
        verification: "Re-eval report filed by date"
        deadline: "2027-02-04"
  signatories:
    - role: "Business owner (VP Member Experience)"
      name: "<named>"
    - role: "Clinical Operations Lead"
      name: "<named>"
    - role: "Privacy Officer"
      name: "<named>"
    - role: "AppSec Lead"
      name: "<named>"
    - role: "Vendor FDE"
      name: "<named>"
    - role: "Vendor On-call Lead"
      name: "<named>"
  review_cadence:
    minimum: "quarterly"
    triggers:
      - "judge_pass_rate rolling 7d < 0.85"
      - "hallucination_rate rolling 7d > 0.03"
      - "sev-2+ incident"
      - "major model-version change"
      - "corpus migration or policy-doc overhaul"
```

Names, numbers, and thresholds above are illustrative.

## Governance frameworks to reference

The decision document is not freelance. The customer's governance team expects it to map to one or more recognised frameworks. The FDE's job is to anchor the document in the frameworks the customer has already adopted, not to invent a parallel framework.

- **NIST AI Risk Management Framework 1.0** and the **Generative AI Profile** (U.S. National Institute of Standards and Technology). The **Manage** function is the direct fit for the decision document — specifically, the actions around risk response, risk prioritisation, and ongoing monitoring. <!-- needs-research: NIST AI RMF 1.0 Manage function sub-category exact wording and NIST AI 600-1 GenAI Profile mapping -->
- **ISO/IEC 42001:2023** — AI management systems. The clauses on operational controls and the plan-do-check-act implementation cycle map to the review cadence. <!-- needs-research: ISO/IEC 42001 clause numbers for operational control and the PDCA cycle wording -->
- **EU AI Act (Regulation 2024/1689)** — in particular **Article 14** on human oversight and **Annex III** for high-risk-system classification. The pairwise evidence and the HITL design (mod-106 Chapter 04) are how the "effective oversight" criterion is met. <!-- needs-research: EU AI Act Article 14 exact clause text and Annex III scope bullets -->
- **SR 11-7** — Board of Governors of the Federal Reserve System, Guidance on Model Risk Management. Applies to Archetype C if the workflow is model-driven decisioning. The decision document is an input to the model-approval memo. <!-- needs-research: SR 11-7 exact wording on effective challenge and ongoing monitoring -->

Where the decision document references a specific clause, quote by framework name and number; add a `needs-research` marker if exact clause wording is required and has not been verified. The customer's legal team owns the final clause-to-clause mapping; the FDE drafts the first pass.

## Three-archetype application

The exit eval shape is constant; the signatories, the rubric emphasis, and the KPI framing change.

### Archetype A — Mid-size US healthcare payer (HIPAA + SOC 2)

**Board composition.** Clinical operations lead; privacy officer; VP of member experience; AppSec lead; vendor FDE and on-call lead. Records-retention officer attends for pilots that touch PHI retention policy.

**Rubric emphasis.** Policy compliance is the highest-weight column — payer workflows live or die on plan-document adherence. Member tone is a close second; the member is on the phone, and tone drives satisfaction and complaint rates. Correctness is table stakes.

**KPI framing.** Handle-time reduction (seconds per call) and escalation-rate drop (fraction of calls that go to supervisor). First-contact resolution as a secondary metric.

**Decision document hooks.** Ties to the HIPAA record-of-processing (the pilot's role in the PHI handling flow); the pilot's position inside the SOC 2 Type II scope; the member-communication policy owner's sign-off column.

### Archetype B — US federal agency (FedRAMP Moderate)

**Board composition.** Agency CIO; records officer; mission lead (the program the pilot serves); agency ISSO (Information System Security Officer); vendor FDE and on-call lead. The records officer's seat is non-negotiable; federal records retention is a compliance surface the FDE must respect.

**Rubric emphasis.** Factual accuracy and citation integrity are the highest-weight columns — a fabricated regulation citation in an agency analyst's draft is a records-integrity failure, not a style issue. Classification handling (does the output respect FOUO / CUI markings) is its own rubric column, scored pass/fail.

**KPI framing.** Analyst draft-time saved (minutes per memo) and zero fabricated regulation citations in the exit-eval sample. The second is a gate, not a continuous metric; one fabricated citation blocks go-live until the retrieval + grounding path is re-examined.

**Decision document hooks.** Ties to the agency's FedRAMP authorization package; the ATO (Authority to Operate) renewal cycle; the agency records officer's retention schedule for AI-assisted drafts.

### Archetype C — EU retail bank (GDPR + DORA + EU AI Act Article 14; SR 11-7 if decisioning)

**Board composition.** Chief Risk Officer; model-risk committee representative; operations lead for the pilot workflow; Data Protection Officer; vendor FDE and on-call lead. For model-driven decisioning workflows, the model-risk committee's sign-off is the critical gate.

**Rubric emphasis.** Regulatory tone (the reply is in the register the bank uses with retail customers) and customer-complaint routing correctness (the pilot routes to the right queue, including the ombudsman queue when applicable). Correctness and policy compliance co-equal below those.

**KPI framing.** Case-handling time and first-contact-resolution rate. For decisioning workflows, the KPI framing is more conservative: uplift measured against the human arm's defensible decisions, not against the raw time saved — the SR 11-7 framing is that the model's output is challenged, not just faster.

**Decision document hooks.** The document is an input to the SR 11-7 model-approval memo for decisioning workflows; the EU AI Act Article 14 human-oversight evidence is attached (the HITL design from mod-106 Chapter 04 plus the pairwise human-vs-AI evidence from this chapter); DORA operational-resilience requirements map to the review cadence and incident-trigger section.

## Anti-patterns

**Non-blind SME scoring.** The rater knows which output is from the AI and anchors on their prior. The pairwise numbers measure a prior, not a preference. Blinding is a process control; without it, the eval is decoration.

**AI arm with system-prompt hints the human arm did not get.** The AI's prompt includes a cheat sheet of SME anchors, a list of policy sections to consider, a formatting template drawn from the SME's house style. The human arm was not given the same brief. The comparison is unfair in the AI's favour; the eval is unfair in the human's favour the moment the pilot ships. Equalise the briefs.

**Rubric authored after raters see outputs.** The team runs the pairwise session, looks at where the AI did well, then writes the rubric to emphasise those columns. This is p-hacking the exit eval. Pre-register the rubric; freeze it before rating starts.

**KPI uplift measured on the training slice.** The pilot was tuned against the gold set the KPI uplift is now being reported on. Any uplift is partly memorisation. The exit-eval holdout exists specifically to prevent this; use it.

**Decision document with no signatory list.** "We agreed in principle during the steering committee" is not sign-off. A document the auditor subpoenas has names and dates. If the signatory column is empty, the pilot has not been approved; it has been tolerated.

**"We'll check back in a year."** No quarterly cadence; no automatic re-visit triggers tied to Chapter 05's drift metrics or mod-110's incident levels. Model behaviour drifts, corpora change, policies change; a decision that is not re-visited is a decision that will be wrong before it is reviewed.

## Where this chapter sits with the rest

- **Chapter 02** is where the holdout slice was carved. The exit eval is where that carve is spent. If the holdout has been seen by any tuning loop, the exit eval cannot run on it; a new holdout must be carved.
- **Chapter 03** is where the slice metrics were formalised. The exit-eval rubric reuses the slice cuts; the pairwise report mirrors the Chapter 03 slice table.
- **Chapter 05** is where the online baseline lives. The decision document locks the Chapter 05 thresholds in; drift-over-threshold is the re-eval trigger.
- **mod-108** is where the compliance / release-assurance packet is assembled. The decision document feeds the packet; the auditor reads the two together.
- **mod-109** is where the rollout gate opens. The decision document is the artifact that opens the gate; "go" turns Phase 2 on, "go-with-conditions" turns a scoped Phase 2 on.
- **mod-112** is where the executive debrief is delivered. The one-pager is the primary narrative artifact; the appendices are the backup.

## Summary

- The exit eval is the evidence packet the governance board signs. Human-preference evidence, business-KPI uplift, and residual-risk acceptance are its three pillars.
- The pairwise human-vs-AI protocol is the preference instrument. Holdout slice from Chapter 02; two arms per case; three blind raters; pre-registered rubric; aggregation via win/tie/loss and, at scale, Bradley-Terry or Elo.
- Blinding is a process control. Strip author metadata, formatting tells, and timing hints. Pre-register the rubric; measure inter-rater reliability (Krippendorff's alpha or Fleiss' kappa); low agreement is a finding, not noise.
- KPI uplift is pre-registered against the mod-102 Chapter 03 value. Randomised A/B at task level is the gold standard and is often impossible; ITS and pre/post-with-controls are the common fallbacks. Report CI and practical-significance threshold together.
- The governance decision document is a one-pager with appendices. Scope; evidence summary; residual-risk register; decision; signatories; review cadence. YAML template codifies the fields.
- Anchor the document in NIST AI RMF (Manage function), ISO/IEC 42001, EU AI Act Article 14, and SR 11-7 where applicable. The legal team owns the final clause-to-clause mapping; the FDE drafts the first pass.
- Three-archetype application: payer boards weight policy compliance and member tone; federal agency boards weight citation integrity and classification handling; bank boards weight regulatory tone, routing correctness, and SR 11-7 effective challenge for decisioning workflows.
- Six anti-patterns guard the eval's defensibility. The two that kill credibility fastest: non-blind scoring and post-hoc rubric edits.
- Chapter 07 is next: the eval as a product — how the gold set, the harness, the online eval, and the exit-eval protocol live together under version control and roll forward across pilot iterations.
