# Chapter 04 — Shipping the Eval Tooling Stack and the CI Gate — Promptfoo, RAGAS, DeepEval, Inspect, and the PR-Merge Boundary

## Why this chapter exists

Chapter 03 defined the eval portfolio — which metrics live at task level, which are judge-as-LLM, which are grounded-ness checks, which are pairwise. That portfolio only pays off when it has merge-blocking authority. A metric dashboard that nobody can block a deploy on is observability, not evaluation. The behaviour of the system drifts to the loudest stakeholder; the eval loses contact with the customer's accept bar; the pilot ships regressions.

This chapter wires the portfolio to CI so a PR fails when the portfolio regresses. It picks between four production-grade eval harnesses — Promptfoo, RAGAS, DeepEval, and UK AISI Inspect — on compliance posture rather than on feature glossiness, then shows the YAML, Python, and workflow snippets that make the gate real. Judge-as-LLM depth (rubric design, calibration, kappa against SME labels) lives alongside the harness choice, because a harness with an uncalibrated judge is a random number generator wearing a tie.

Tool-selection depth is `ai-eval-engineer` (peer, level 30) craft and lands in Chapter 07 when the harness gets handed off. The FDE's job is to pick what fits the customer's compliance posture, ship the first working gate, and brief the peer when depth is required.

## The four tools and where each fits

Four harnesses cover most pilots. Each has a canonical shape, a license, and a distinctive strength. Adjacent options exist — **OpenAI Evals** (github openai/evals), **MLflow LLM Evaluate** ([mlflow.org LLM evaluation docs](https://mlflow.org/docs/latest/llms/llm-evaluate/index.html)), **LangSmith** ([docs.smith.langchain.com](https://docs.smith.langchain.com/)), **Braintrust** ([braintrust.dev](https://www.braintrust.dev/)), **Humanloop** — and legitimately fit specific pilots. Start the matrix with the four below.

| Tool | Primary strength | Shape | License / maintainer | Typical use in a pilot |
|---|---|---|---|---|
| **Promptfoo** ([promptfoo.dev](https://www.promptfoo.dev/), [github promptfoo/promptfoo](https://github.com/promptfoo/promptfoo)) | Prompt / model regression with provider-matrix tests and pairwise comparisons | YAML-first config, TypeScript core, `npx promptfoo eval` CLI | MIT, promptfoo | The CI gate's default entry point. Provider matrix + assertions + pairwise. |
| **RAGAS** ([docs.ragas.io](https://docs.ragas.io/), [github explodinggradients/ragas](https://github.com/explodinggradients/ragas)) | RAG-specific metrics (faithfulness, answer relevance, context precision / recall, noise sensitivity) | Python library, requires a judge-LLM | Apache-2.0, Exploding Gradients | The retrieval-grounded slice of the portfolio. Pointed at mod-105's retrieval output. |
| **DeepEval** ([deepeval.com](https://www.deepeval.com/), [github confident-ai/deepeval](https://github.com/confident-ai/deepeval)) | Pytest-style LLM assertions with built-in metrics (hallucination, bias, toxicity, summarisation, contextual-recall) plus G-Eval for custom rubrics | Python, pytest-native | Apache-2.0, Confident AI | Unit-test-flavoured gating of individual prompts and chains. |
| **UK AISI Inspect** ([inspect.aisi.org.uk](https://inspect.aisi.org.uk/), [github UKGovernmentBEIS/inspect_ai](https://github.com/UKGovernmentBEIS/inspect_ai)) | Opinionated solver / scorer separation, built-in sandboxing (Docker / K8s), long-horizon and agentic evals | Python, task + solver + scorer | MIT-ish, UK AI Security Institute <!-- needs-research: confirm the exact license and current version of inspect_ai at pilot time --> | Agentic / tool-using / long-horizon workflows where sandboxed execution matters. |

Rules of thumb for the matrix:

- **Promptfoo is the default CI entry point.** Its YAML shape clears review faster than a Python eval script, and its provider-matrix feature covers the "does this still work on the newer model" question the customer will ask every quarter.
- **RAGAS owns the retrieval slice.** When the pilot is a RAG workflow, RAGAS's faithfulness and context-precision metrics are the shortest path to the portfolio Chapter 03 defined.
- **DeepEval is the pytest shape.** When the customer's platform team runs on pytest and wants LLM assertions to look like test cases, DeepEval drops in with less friction than wrapping a different harness.
- **Inspect is the agentic / sandboxed shape.** When the pilot is tool-using and the eval needs a real sandbox (filesystem, network, process), Inspect's solver / scorer separation and built-in Docker / K8s sandbox earn their weight.

Mixing two or three is normal. Three is a ceiling — four separate harnesses in one CI pipeline is a maintenance burden that drives the gate towards abandonment.

## The customer-compliance posture question

Every eval run moves customer data through a judge-LLM. The judge is a model call; the model call lands somewhere; wherever it lands is inside or outside the customer's trust boundary. This is the Chapter 01 gate restated for the eval layer: **no customer data leaves to a SaaS eval vendor unless the customer has a signed zero-retention + no-training DPA**.

The concrete rules:

- **Harness binary / library runs on the customer's CI runner.** Promptfoo as a self-hosted GitHub Actions runner or GitLab self-hosted runner. RAGAS / DeepEval / Inspect as Python packages installed into the CI container. The container runs inside the customer's CI VPC, not on hosted SaaS CI with the customer's data in logs.
- **Judge-LLM endpoint is inside the VPC.**
  - Archetype A: AWS Bedrock with zero-retention default and a BAA covering the model.
  - Archetype B: self-hosted vLLM / SGLang on GovCloud, serving a FedRAMP-aligned model.
  - Archetype C: Azure OpenAI under the EU Data Boundary with a zero-retention SKU and the data-residency addendum signed.
- **Gold set lives in the customer's repo or in a customer-owned bucket.** The gold set often contains PII (mod-104 Chapter 04's handling applies). It does not leave the boundary to a vendor eval dashboard.
- **Results are stored in a customer-owned artifact store.** CI run outputs, judge rationales, pairwise traces — all under the customer's retention policy (mod-108 Chapter 05 handles audit retention).

Use the harness's hosted dashboard only if the customer has signed the DPA that permits it. Default to local / in-VPC storage of results and build whatever dashboard is needed on top of the customer's existing BI stack.

## Wiring the CI gate

The gate has four moving parts: the workflow file, the harness config, the gold set, and the branch-protection rule. Shape each one.

### Shadow mode first, required-check second

For the first calibration week, run the gate but let it pass. In GitHub Actions, set `continue-on-error: true` on the eval step and post the result as a PR comment. In Azure DevOps, set the pipeline stage to `condition: always()` and `continueOnError: true`. Read the results. Adjust thresholds (and the gold set — regressions in week one are as likely the gold set's fault as the system's). After calibration, flip to a required status check on branch protection. The branch-protection rule is the actual block mechanism; the workflow's success / failure is the input to it.

### GitHub Actions workflow sketch

```yaml
# .github/workflows/eval-gate.yml
name: eval-gate
on:
  pull_request:
    paths:
      - "src/**"
      - "prompts/**"
      - "evals/**"
jobs:
  eval:
    runs-on: [self-hosted, customer-ci-vpc]
    timeout-minutes: 15
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - name: Install harnesses
        run: |
          npm install -g promptfoo
          pip install -r evals/requirements.txt  # ragas, deepeval, inspect_ai
      - name: Promptfoo regression (PR sample)
        env:
          # Judge-LLM endpoint inside the VPC; credentials via OIDC to the customer's KMS
          BEDROCK_REGION: us-east-1
          PROMPTFOO_CACHE_ENABLED: "true"
        run: |
          npx promptfoo eval \
            -c evals/promptfooconfig.yaml \
            --filter-sample 50 \
            --output results/promptfoo.json \
            --no-progress-bar
      - name: DeepEval pytest
        run: pytest evals/test_eval.py -q --junit-xml=results/deepeval.xml
      - name: RAGAS retrieval slice
        run: python evals/ragas_run.py --out results/ragas.json
      - name: Threshold check
        run: python evals/check_thresholds.py results/
      - name: Post PR comment
        if: always()
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require("fs");
            const body = fs.readFileSync("results/summary.md", "utf8");
            await github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body,
            });
```

Branch-protection: in GitHub, require the `eval / eval-gate` check to pass before merge; in Azure DevOps, require the pipeline check in the branch policy.

### Promptfoo config

```yaml
# evals/promptfooconfig.yaml
description: Pilot regression gate
prompts:
  - file://prompts/assist.v3.txt
providers:
  - id: bedrock:anthropic.claude-3-5-sonnet-v2
    config:
      region: us-east-1
  - id: bedrock:anthropic.claude-3-5-haiku  # cheaper candidate for comparison
tests:
  - vars:
      case_id: GOLD-0001
      question: "What is the prior-authorization requirement for MRI of the lumbar spine?"
      reference: "Plan X requires PA; see policy PA-MRI-003."
    assert:
      - type: contains
        value: "PA-MRI-003"
      - type: llm-rubric
        provider: bedrock:anthropic.claude-3-5-sonnet-v2
        rubric: file://evals/judge/correctness_v3.yaml
        threshold: 0.75
      - type: latency
        threshold: 6000  # ms
  # ...more rows loaded via tests: file://evals/gold/pr-sample.jsonl
defaultTest:
  options:
    cache: true
```

### DeepEval pytest

```python
# evals/test_eval.py
import json
import pytest
from deepeval import assert_test
from deepeval.metrics import (
    HallucinationMetric,
    AnswerRelevancyMetric,
    ContextualRecallMetric,
)
from deepeval.test_case import LLMTestCase
from pilot.app import run_pipeline  # the system under test

with open("evals/gold/pr-sample.jsonl") as f:
    GOLD = [json.loads(line) for line in f]

@pytest.mark.parametrize("row", GOLD, ids=[r["case_id"] for r in GOLD])
def test_case(row):
    result = run_pipeline(row["question"])
    case = LLMTestCase(
        input=row["question"],
        actual_output=result.answer,
        expected_output=row["reference"],
        retrieval_context=result.retrieved_chunks,
    )
    assert_test(
        case,
        metrics=[
            HallucinationMetric(threshold=0.2),
            AnswerRelevancyMetric(threshold=0.7),
            ContextualRecallMetric(threshold=0.7),
        ],
    )
```

### RAGAS retrieval slice

```python
# evals/ragas_run.py
from datasets import Dataset
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision
import json, sys, argparse

parser = argparse.ArgumentParser()
parser.add_argument("--out", required=True)
args = parser.parse_args()

with open("evals/gold/retrieval-slice.jsonl") as f:
    rows = [json.loads(line) for line in f]

ds = Dataset.from_list([
    {
        "question": r["question"],
        "answer": r["answer"],
        "contexts": r["contexts"],
        "ground_truth": r["reference"],
    }
    for r in rows
])

result = evaluate(
    ds,
    metrics=[faithfulness, answer_relevancy, context_precision],
    # judge-LLM is configured via RAGAS's run config to the in-VPC Bedrock / Azure endpoint
)
with open(args.out, "w") as f:
    json.dump(result.to_pandas().to_dict(orient="records"), f)
```

### Inspect task scaffold

```python
# evals/inspect_tasks/pilot_eval.py
from inspect_ai import Task, task
from inspect_ai.dataset import json_dataset
from inspect_ai.solver import generate, use_tools
from inspect_ai.scorer import match, model_graded_fact
from pilot.tools import search_policy, lookup_member  # agent's tool catalogue

@task
def pilot_eval():
    return Task(
        dataset=json_dataset("evals/gold/agentic-slice.jsonl"),
        solver=[use_tools([search_policy, lookup_member]), generate()],
        scorer=[match(), model_graded_fact()],  # exact match + judge-LLM factuality
        sandbox="docker",
    )
```

Run with `inspect eval evals/inspect_tasks/pilot_eval.py --model bedrock/anthropic.claude-3-5-sonnet-v2`.

## Authoring judge-as-LLM prompts with calibrated accept rates

A judge-LLM is a scoring function, not a free-text ask. Six disciplines separate a usable judge from a random number generator.

1. **Rubric as scoring function.** Define the scale explicitly — 0 to 3 is a defensible shape (fail / partial / substantive / complete). Avoid 1 to 5 unless there is real signal in five levels; three-plus-fail is often as much as the judge can discriminate reliably.
2. **Chain of thought in the rubric, separated from the score field.** Ask for `<rationale>` and `score` as separate JSON fields. The rationale is auditable; the score is machine-readable. A judge that cannot explain itself cannot be reviewed when it fails.
3. **Few-shot at both ends and the hard middle.** Three anchors minimum: a gold positive, a gold negative, and a genuinely-hard middle case where SMEs themselves split. The middle anchor is what makes a judge discriminate.
4. **Position randomisation for pairwise.** Swap A / B on half the rows. Order bias is a published, measurable artifact of current judge models. Randomise to control it.
5. **Judge-model diversity.** Two judges from different families (for example, Claude Sonnet and GPT-4.1, both deployed in-boundary), aggregated by majority or mean. Flag disagreement rows for SME review. A single-family judge inherits that family's blind spots.
6. **Calibration against SMEs.** On the SME-labelled hold-out from Chapter 02, compute judge-vs-SME agreement. Cohen's kappa for binary thresholded scores; Krippendorff's alpha for ordinal. Pick the threshold that matches the customer's accept rate via ROC-style reasoning — the operating point where false-positive and false-negative rates fit the risk the governance board signed.

Example judge spec:

```yaml
judge:
  name: correctness_v3
  judge_model: azure-openai/gpt-4.1-2025-04-14
  rubric: |
    Score the ASSISTANT_OUTPUT against the REFERENCE_ANSWER on a 0-3 scale:
    3 = fully correct and complete
    2 = correct in substance but missing a minor element
    1 = partially correct with a material gap
    0 = incorrect or hallucinated
    Think step by step in <rationale>. Return a JSON object: {"rationale": "...", "score": 0|1|2|3}.
  few_shot:
    - input: "Does Plan X cover MRI of the lumbar spine without prior authorization?"
      reference: "No. Plan X requires PA; see policy PA-MRI-003."
      output: "Plan X requires prior authorization per PA-MRI-003."
      rationale: "Correct substance, cites the policy identifier, fully complete."
      score: 3
    - input: "Does Plan X cover MRI of the lumbar spine without prior authorization?"
      reference: "No. Plan X requires PA; see policy PA-MRI-003."
      output: "Yes, Plan X covers MRI without prior authorization."
      rationale: "Directly contradicts the reference; hallucinated coverage."
      score: 0
    - input: "Does Plan X cover MRI of the lumbar spine without prior authorization?"
      reference: "No. Plan X requires PA; see policy PA-MRI-003."
      output: "Prior authorization is typically required for advanced imaging."
      rationale: "Substance correct but does not identify Plan X or cite PA-MRI-003."
      score: 1
  output_schema:
    type: object
    properties:
      rationale: { type: string }
      score: { type: integer, enum: [0, 1, 2, 3] }
    required: [rationale, score]
  calibration:
    last_calibrated_at: 2026-10-14
    kappa_vs_sme: 0.78
    chosen_threshold: 2  # score >= 2 is "acceptable"
```

The `calibration` block is the artifact the customer's governance board reads. It dates the calibration, records the kappa, and names the threshold. When a judge-model upgrade happens, the calibration is re-run; the block gets a new date and a new kappa. A judge spec with a stale calibration date is a lint error.

## Cost, latency, and token budget for eval runs

Eval runs are not free. 500 gold rows × 3 metrics × 2 judges at 1k input tokens per judge call is 3 million tokens per run. On a Sonnet-class judge at current list price that is on the order of tens of dollars per PR; on a self-hosted judge the cost is runner time rather than token price but the arithmetic still constrains the dev loop. <!-- needs-research: confirm current Bedrock / Azure OpenAI token pricing at pilot time -->

Three disciplines:

- **Sample for PR runs; full portfolio on merge-to-main and nightly.** 50 rows on PR (stratified across the Chapter 02 customer-case distribution) catches regressions in the dev loop. The full 500 to 2000 row portfolio runs on merge to `main` and on a nightly cron. The nightly run is the authoritative regression signal the governance board reviews.
- **Cache aggressively.** Promptfoo's cache keys on prompt + provider + inputs; identical rows across PRs hit the cache. RAGAS and DeepEval support caching the judge-LLM calls — enable it. The embedding cache is separate; enable it too.
- **Concurrency to hold latency under 10 minutes per PR.** The dev loop fails if the gate takes 30 minutes. Parallelise rows (Promptfoo's `-j` / concurrency setting; DeepEval's parallel flag; RAGAS via dataset sharding). Watch the judge-LLM rate limit — Bedrock and Azure OpenAI both have account-level TPM limits that become the binding constraint before token cost does.

## Three-archetype application

**Archetype A — healthcare payer.** Promptfoo as the CI entry point for provider-matrix regressions, DeepEval as the pytest-style gate wired into the existing pytest suite. Both run on a GitHub Actions self-hosted runner inside the customer's CI VPC. Judge-LLM is Bedrock Claude Sonnet under the existing BAA with zero-retention on by default. RAGAS covers the retrieval-grounded subset — the pilot is a RAG workflow over policy documents, so faithfulness and context-precision are first-class. Inspect skipped for the pilot phase; the workflow is not tool-using yet.

**Archetype B — federal agency.** Promptfoo plus Inspect. Inspect carries the weight because the pilot includes sandboxed tool-use over FOUO documents and Inspect's Docker sandbox fits the FedRAMP posture more cleanly than ad-hoc subprocess calls. Judge-LLM is a self-hosted model (Llama-3.3-70B or similar) on vLLM inside GovCloud, behind the customer's existing FedRAMP boundary. RAGAS is a candidate for the retrieval slice; current FedRAMP status of the RAGAS dependency set should be verified against the customer's package-approval system <!-- needs-research: confirm RAGAS and transitive dependencies on the customer's FedRAMP-approved package list -->. CI is GitLab CE on a GovCloud self-hosted runner.

**Archetype C — EU retail bank.** Promptfoo plus RAGAS plus DeepEval. The pilot is a RAG workflow with multiple candidate prompts across Azure OpenAI model versions; the provider-matrix and pairwise features of Promptfoo earn their place. Judge-LLM is Azure OpenAI (GPT-4.1 class) under the EU Data Boundary addendum and the zero-retention / no-training SKU. CI is Azure DevOps Pipelines on a self-hosted agent inside the customer's Azure tenancy. The calibration-date lint rule (judge spec stale-after 90 days) is a DORA-aligned governance control the ICT register references.

## Anti-patterns

**Gold set in the repo but the judge-LLM is a vendor SaaS with retention on by default.** This is data exfiltration dressed up as evaluation. The customer's gold set is often the richest PII artifact in the pilot — it is labelled, representative, and curated. Shipping it to a hosted eval dashboard without a signed zero-retention DPA fails the Chapter 01 gate and is a stop-ship finding in a security review.

**Thresholds drifting upward quietly.** Nobody owns the calibration; the gate quietly starts passing anything. Lower thresholds are easier than fixing regressions; the gate degrades into decoration. Mitigation: calibration date and kappa in the judge spec, lint rule on staleness, quarterly re-calibration reviewed by the governance board.

**Pytest assertions pinned too tight.** DeepEval metrics on noisy judge scores at `threshold=0.95` are flaky. The gate fires on noise; the team learns to ignore it; a real regression slips through. Set thresholds from the calibration curve, not from the number that passed on the first run.

**Judge prompts never re-calibrated after a model-upgrade.** When Bedrock bumps the Claude Sonnet version, or Azure rotates to the next GPT-4.1 release, the judge-model under the rubric changes. Judge scores shift 5-10% on unchanged inputs. The portfolio looks like a regression when the eval itself moved. Mitigation: the calibration block in the judge spec. When the judge model changes, calibration re-runs before the gate re-enables.

**Running the full portfolio on every commit.** Token cost explodes. Dev-loop latency goes above 20 minutes. The team disables the gate to ship. The gate becomes decorative. Mitigation: PR-sample sizing (50 rows), merge-to-main and nightly full runs, aggressive caching.

## Summary

- The eval portfolio has merge-blocking authority or it is observability, not evaluation. The CI gate is the enforcement mechanism.
- Four harnesses cover most pilots: Promptfoo (YAML regression and provider matrix), RAGAS (RAG-specific metrics), DeepEval (pytest-style assertions), Inspect (solver / scorer separation with sandboxing). Pick one to three; four is a maintenance ceiling.
- The harness runs on the customer's self-hosted CI runner; the judge-LLM endpoint is inside the customer's trust boundary; the gold set and results stay under customer retention policy. No SaaS-bound evaluation without a signed zero-retention + no-training DPA.
- Shadow mode first (run the gate, do not block, post a comment), required-check second (branch-protection rule enforces the block). The branch-protection rule is the real mechanism, not the workflow file.
- Judge-as-LLM prompts have six disciplines: rubric as scoring function, CoT separated from the score field, few-shot at both ends plus the hard middle, position randomisation for pairwise, judge-model diversity, calibration against SMEs with kappa or alpha recorded and dated in the spec.
- PR runs sample 50 rows; merge-to-main and nightly run the full portfolio. Cache the judge-LLM calls. Parallelise under the TPM limit. Keep the PR gate under 10 minutes.
- Archetype A: Promptfoo + DeepEval + RAGAS on Bedrock Sonnet. Archetype B: Promptfoo + Inspect on a self-hosted Llama-class judge in GovCloud. Archetype C: Promptfoo + RAGAS + DeepEval on Azure OpenAI under EU Data Boundary.
- Cross-references: Chapter 03 defines the portfolio feeding this gate; Chapter 05 adds the online complement that watches production; Chapter 07 hands the harness to `ai-eval-engineer` for depth; mod-108 Chapter 05 owns the audit-trail retention for CI outputs; mod-109 treats the CI gate as one of the named rollout gates.
