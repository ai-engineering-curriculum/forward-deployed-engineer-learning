# mod-107-eval-eng-on-customer-data: Eval Engineering on Customer Data

> Scaffolded by `aicg org execute-plan`. Lecture chapters and exercise content are authored on subsequent autonomous cycles.

**Estimated effort:** 16 hours

## Learning objectives

- Bootstrap a customer-data gold set — SME-paired labelling workflow, stratified sampling across the customer's actual case distribution, double-labelling + inter-rater reliability, labelling-UI shape, versioned and auditable gold-set storage
- Design the offline eval — task-level correctness vs. subjective judge-as-LLM vs. pairwise comparison vs. factuality / groundedness / citation-accuracy — and the eval-vs-metric-vs-business-KPI mapping that lets the customer sign off on 'ready to go live'
- Ship the eval tooling stack — Promptfoo + RAGAS + DeepEval + Inspect (UK AISI) at the customer's compliance posture; wire into the pilot CI/CD so a PR cannot merge until the eval gate passes; author judge-as-LLM prompts with calibrated accept rates
- Ship online eval on production customer traffic — judge-as-LLM at sample rate, implicit user-feedback capture (did the user accept / edit / discard the suggestion), per-tenant quality dashboards, drift detection
- Run the pilot-to-production exit eval — pairwise human-vs-AI comparison on a holdout slice, blind SME scoring, business-KPI uplift, and the decision document the customer's governance board signs
- Keep customer data private — eval infrastructure deployed inside the customer VPC or a zero-retention sandbox, no customer data in external evaluation pipelines; coordinate with `ai-eval-engineer` (peer, level 30) on methodology depth and with `ai-evaluation-engineer` (peer, level 35) on release-assurance packaging

## Structure

- `01-…md` … `0N-…md`: lecture chapters.
- `exercises/`: per-exercise prompts.
- `labs/`: long-form hands-on labs.
- `quizzes/`: knowledge checks.
- `resources.md`: external references.
