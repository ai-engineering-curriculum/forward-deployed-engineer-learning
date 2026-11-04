# mod-104-llm-app-in-customer-envs: LLM Application Engineering in Customer Environments

> Scaffolded by `aicg org execute-plan`. Lecture chapters and exercise content are authored on subsequent autonomous cycles.

**Estimated effort:** 14 hours

## Learning objectives

- Select a provider stack per customer deployment — frontier labs (OpenAI, Anthropic, Google, xAI, Cohere, Mistral) vs. hyperscaler managed (AWS Bedrock, Google Vertex AI, Azure AI Foundry) vs. self-hosted (vLLM, TGI, SGLang) — scoring against the customer's data-residency, compliance, latency, cost, and model-choice-freedom constraints
- Deploy a model gateway (LiteLLM / Portkey / OpenRouter) inside the customer environment — unified SDK surface, per-tenant budget enforcement, failover, caching, PII guards, audit logs — with the explicit customer-side build-vs-buy decision documented
- Build production-shape prompt architecture at the customer level — structured outputs (JSON Schema / Pydantic / Zod-shaped), streaming, prompt caching (Anthropic / Bedrock / OpenAI prompt caching), prompt versioning + review workflow, A/B shape, per-tenant prompt overrides
- Author per-customer-contracted cost / latency SLOs — TTFT (time-to-first-token), TPOT (time-per-output-token), p50 / p95 / p99 completion latency, token cost per task, grounded in the customer's business-KPI value-per-task
- Decide when to escalate to fine-tune / adapter workflows — the when-to-fine-tune decision gate, lightweight LoRA / QLoRA shape, data-prep responsibilities split between FDE and `fine-tuning-engineer` (peer, level 30) / `training-pipeline-engineer` (peer, level 35)
- Cite the boundary to `applied-ai-engineer` (level 30) on in-house feature engineering craft — this module adds the customer-environment overlay, not the full craft

## Structure

- `01-…md` … `0N-…md`: lecture chapters.
- `exercises/`: per-exercise prompts.
- `labs/`: long-form hands-on labs.
- `quizzes/`: knowledge checks.
- `resources.md`: external references.
