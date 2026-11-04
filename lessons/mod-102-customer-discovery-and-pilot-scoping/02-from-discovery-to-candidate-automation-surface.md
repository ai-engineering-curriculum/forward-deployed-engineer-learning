# Chapter 02 — From Discovery to a Candidate Automation Surface

## Why this chapter exists

After Chapter 01's interviews, shadowing, and process-mining, you are in a specific state: you have a stack of notes, a workflow map, a handful of pain points and divergences, and a sponsor who wants "an AI pilot." You do not yet have a *candidate intervention* you could estimate, defend, or put through the four decision tests of Chapter 03.

This chapter teaches the bridge step. It turns raw discovery into one or two named automation candidates — each one with a hypothesis, an input surface, an output surface, an SME-in-the-loop checkpoint, and an explicit counter-candidate. The output of this chapter is the discovery brief, the first FDE deliverable from mod-101 Chapter 05.

The temptation at this step is to brainstorm ten candidates and present a menu. Resist it. A pilot has one primary bet and perhaps one contingency bet. Everything else belongs on the deferred list, with a named reason for why it is deferred.

## The opportunity-solution-tree

Teresa Torres's [opportunity-solution-tree](https://www.producttalk.org/opportunity-solution-tree/) is the simplest structure that supports this translation. The tree has four levels:

- **Outcome** (root) — the business outcome the sponsor is chasing. One metric, one target. *"Reduce prior-authorisation turnaround from 48h to 24h at p90."*
- **Opportunities** — the observed user needs, pain points, or workflow friction that, if addressed, would move the outcome. Each opportunity should be traceable back to a specific SME quote or shadow observation.
- **Solutions** — specific ways to address a given opportunity. A solution is more concrete than an opportunity (not "reduce drafting time" but "draft a first-pass response the SME edits").
- **Experiments** — the test that would de-risk a solution before full commitment (a prompt walkthrough on 20 real cases, a dry-run against the gold set, a two-day usability pilot).

A worked fragment for the prior-authorisation example from Chapter 01 (indented bullets, for text):

```
Outcome: Reduce prior-auth turnaround from 48h to 24h at p90
├── Opportunity: Drafting the response text consumes ~7 minutes per case
│   ├── Solution: LLM drafts first-pass response, SME edits
│   │   └── Experiment: Run draft on last week's 50 cases, measure edit distance + SME QA time
│   └── Solution: Template library with keyword triggers (deterministic)
│       └── Experiment: Build 10 templates covering top-30 patterns, measure coverage
├── Opportunity: SMEs waste time on tab-switching between EHR, formulary, member chart
│   └── Solution: Unified case view (no LLM)
│       └── Experiment: Mock-up + 30-min SME walkthrough
└── Opportunity: Edge cases (appeals, missing-info) interrupt the typical-case flow
    └── Solution: Classifier that routes edge cases to a specialist queue
        └── Experiment: Classify last-quarter cases, measure routing accuracy
```

Three things to notice about the tree:

1. **Every opportunity cites an observation from Chapter 01.** Opportunities invented in a vacuum (not observed) go in the brainstorming parking lot, not on the tree. The tree is the structure for what you *learned*, not for what you can imagine.
2. **Not every solution is an LLM solution.** The unified case view is plain engineering. The template library is deterministic. The LLM solution sits next to the deterministic one so the comparison is explicit.
3. **Experiments are small.** An experiment that takes a week to run is a pilot. An experiment on the tree is a 1-2 day validation.

The tree is not a plan. It is a map that makes the choices legible. The FDE walks the sponsor and the SME through the tree in a 30-minute meeting and together they pick the one or two leaves to carry forward. This conversation is the single highest-leverage conversation in the module.

## The candidate-intervention naming pattern

A solution leaf is not yet a candidate the FDE can put through Chapter 03's tests. It needs to be named with five specific fields. Use this template for every candidate that survives the sponsor conversation:

```
Candidate: <short name>
Hypothesis:  If <intervention>, then <observable change> within <timeframe>.
Input surface: <what signal the model/code sees>
Output surface: <what the model/code produces, and what happens to that output>
SME-in-the-loop checkpoint: <where a human reviews or approves>
Counter-candidate:
  Deterministic alternative: <non-LLM alternative or "none">
  Leave-alone alternative: <cost of not doing this, honestly estimated>
```

A worked example:

```
Candidate: LLM-drafted prior-auth response
Hypothesis: If the model drafts the first-pass response using the case detail +
            member benefits + the formulary rule, then median SME time-per-case
            drops from 7 minutes to <3 minutes within 4 weeks of pilot launch.
Input surface: Case detail (structured), member benefits (structured), formulary
               rule (text, retrieved), last 3 similar cases (text, retrieved).
Output surface: A structured draft letter with named fields (decision, rationale,
                formulary citation). Saved to the SME's drafts folder; not sent
                to the member until the SME clicks Send.
SME-in-the-loop checkpoint: SME reviews draft before it is sent. No auto-send
                            in pilot scope.
Counter-candidate:
  Deterministic alternative: Template library with 30 handwritten templates.
    Likely covers ~60% of cases (based on process-mining of last quarter).
    Cheaper to ship; cheaper to maintain; weaker on edge cases.
  Leave-alone alternative: SMEs continue drafting from scratch or from their
    own personal template files. Total cost: ~7 min * 800 cases/week per SME =
    ~93 SME-hours/week across the team (as observed in Chapter 01).
```

The discipline is in the fifth field. Every candidate must have its deterministic and leave-alone counter-candidates written down, because Chapter 03's deterministic-code test requires it. If you cannot write the deterministic alternative, you have not thought about the candidate hard enough to put it in the brief.

A good candidate statement is 10-20 lines. Two pages is a design doc, not a candidate statement.

## Triage into three buckets

Every leaf on the opportunity-solution-tree ends up in one of three buckets after the FDE's triage pass:

**Bucket A — AI intervention candidate.** Goes to Chapter 03's four tests. If it passes all four, it is a serious candidate for the pilot's primary or contingency bet. Expect 2-3 candidates in this bucket.

*Example:* The LLM-drafted response above. Carries a hypothesis, a counter-candidate, and plausible pilot-scale value.

**Bucket B — Deterministic-code intervention.** The solution would move the outcome, but an LLM is not the right tool. These stay in the discovery brief under a "deterministic improvements the customer's engineering team could own" section, or get flagged to the customer's own backlog. The FDE does not usually ship these inside the pilot — the pilot is for the AI bet — but naming them is part of honest discovery.

*Example:* The unified case view. Removes 2 minutes of tab-switching per case; needs no LLM; is a frontend + a few queries; belongs to the customer's internal team or a non-pilot sprint.

**Bucket C — Leave alone.** A real pain point but not a good pilot bet. The reasons tend to be one of: too far from the sponsor's KPI, data not available at pilot scope, SME would not adopt the change in pilot timeframe, or the failure-mode cost is intolerable. These are named explicitly in the brief's "where AI does not help" section (mod-101 Chapter 05) so the sponsor can see the FDE made the call deliberately.

*Example:* Auto-approval of the easy cases without SME review. The deterministic classifier could probably reach 95% precision; the 5% false-approval rate is intolerable in regulated clinical review; the SME culture is not ready for unsupervised auto-send. Leave alone in pilot; revisit after exit.

A candidate that ends up in Bucket C is not a failure of discovery. A discovery pass that puts *every* candidate in Bucket A is a failure of discovery — it means the FDE skipped the "where AI does not help" conversation. The sponsor will later discover the gap and lose trust.

## Writing the discovery brief

The discovery brief is the deliverable this chapter produces. Mod-101 Chapter 05 defines the shape; this chapter fills in what each section contains for an LLM pilot specifically.

**Workflow as-is.** A 1-2 page swim-lane or narrative description of the workflow from Chapter 01's shadowing. Who touches it, how long each step takes, what the operators actually do (not what the org chart says). Include the story-map backbone if you drew one. Attach the quantitative mining summary if you ran it.

**Pains and metrics.** 3-6 pain points, each quantified where possible. Cite source: "SME shadow, Mariana, 2026-11-10" or "event-log extract, 2026-10-30 to 2026-11-06." Unquantified pain points are allowed (not everything is measurable at discovery), but flag them as such so the pilot's success metrics (Chapter 04) are built on the quantified ones.

**Candidate interventions.** The 2-3 candidates from Bucket A, each with the five-field statement from the "naming pattern" section above. One of them is marked as the recommended primary; one is marked as the contingency; the third (if present) is marked as "backup, deferred unless primary falls out."

**Where AI does not help.** The Bucket C candidates. Short reason per candidate. This section is what prevents the sponsor from later asking "why didn't we try X?" — because you already explained why you didn't.

**Deterministic improvements the customer could own.** The Bucket B items. Short list, directed to the customer's engineering or ops team. This earns goodwill with the customer's engineers who will later maintain the pilot; it shows the FDE is not inflating scope to keep themselves employed.

**Open questions.** The things the discovery pass did not resolve. Each with an owner and a due date. "Can we get a 90-day event-log extract to sanctioned environment? — Owner: customer-side data engineer; Due: 2026-11-18." Open questions close via the sponsor; they do not sit open into pilot scope.

**Recommendation.** One of three options. *Proceed to pilot scope on Candidate A (primary) with Candidate B as contingency.* *Spend another week on discovery — specifically to resolve open questions 1 and 3.* *Decline the engagement because none of the candidates survive Chapter 03's tests.* The recommendation is a sentence, not a paragraph.

Length target: 2-4 pages (plus appendices). The brief is a decision document, not a research report. If you need more than 4 pages you are either copying your notes in or using the brief to think through the problem — do both before writing, not inside the brief.

## How many candidates is right

The answer is 2-3, almost never more.

The argument for more candidates ("optionality is good; let the sponsor pick from a wider set") breaks on three counts:

1. **Each candidate costs you a Chapter 03 pass.** Running four tests against a candidate rigorously is a half-day of work. Ten candidates = 20 half-days you don't have.
2. **Each candidate costs a sponsor conversation.** The sponsor is not going to compare ten candidates in their head. They will pick the one you lean into, the two with the best-named hypotheses, or the one closest to a political priority. The extra candidates waste the FDE's thinking time and dilute the sponsor's attention.
3. **The pilot can only run one or two.** You will not pilot five candidates in parallel; the SME time, the gold-set labelling, the integration surface, the eval infrastructure — none of it scales that way. A brief with five candidates is an FDE who has not yet made the harder call of which one to lean into.

The right pattern is: 2-3 candidates carried forward, with explicit *ranking* from the FDE. Not "here are three options, you pick." But "we recommend A as primary, B as contingency, and we are deferring everything else — here is why."

## The handoff into pilot scope

A signed discovery brief tees up the Chapter 04 pilot-scope document. The handoff is specific:

- The brief's recommended primary candidate becomes the pilot's problem statement (Chapter 04 section 1).
- The brief's workflow-as-is and story map inform the pilot's user stories (Chapter 04 section 2).
- The brief's "where AI does not help" directly feeds the pilot's non-goals (Chapter 04 section 3).
- The brief's pain quantification feeds the pilot's primary and secondary success metrics (Chapter 04 section 4).
- The brief's candidate statement feeds the pilot's ADRs on stack choices (Chapter 04 section 5).
- The brief's open questions, if any remain, become decisions the pilot scope must make explicitly.

If a section of Chapter 04's pilot-scope document has no source in the discovery brief, it means you are drafting scope on assumption rather than on discovery. Go back a step.

## Summary

- The bridge between discovery and pilot scope is a short, structured translation: opportunity-solution-tree → named candidates → triage into three buckets → discovery brief → handoff to pilot scope.
- Every opportunity on the tree cites an observation from discovery. Every solution is paired with its deterministic and leave-alone counter-candidates on paper.
- Each candidate intervention gets a 10-20 line statement with five fields: hypothesis, input surface, output surface, SME-in-the-loop checkpoint, counter-candidate.
- Triage every candidate into Bucket A (AI pilot), Bucket B (deterministic improvement for customer team), or Bucket C (leave alone, with named reason). A brief where every candidate is in Bucket A has skipped the "where AI does not help" conversation.
- The discovery brief is 2-4 pages and ends in a one-sentence recommendation: proceed, discover more, or decline.
- Carry 2-3 candidates forward, ranked. More is false optionality.
- The discovery brief directly seeds every section of the pilot-scope document in Chapter 04. If a scope section has no brief source, you are drafting on assumption.
