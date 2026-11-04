# mod-110-observability-and-oncall: Observability and On-Call for Shared Customer Systems

> Scaffolded by `aicg org execute-plan`. Lecture chapters and exercise content are authored on subsequent autonomous cycles.

**Estimated effort:** 12 hours

## Learning objectives

- Instrument the pilot with OpenTelemetry — traces, metrics, logs — using the OpenTelemetry GenAI semantic conventions so customer-side SIEM / observability platforms (Datadog, Splunk, Elastic, Grafana, Dynatrace, New Relic) consume them natively
- Wire the LLM-observability platform — Langfuse / LangSmith / Arize Phoenix / Weave / Braintrust — at the customer's compliance posture (self-hosted when the customer requires); dashboards the customer-side operator opens daily
- Author LLM-specific SLOs — quality (judge score, user-feedback acceptance), latency (TTFT, TPOT, p50 / p95 / p99), cost (per task, per tenant, per day), safety (refusal rate, guardrail trip rate); tie each SLO to the customer's business KPI
- Ship the on-call rota for a shared customer-production system — joint incident command with the customer's operator, triage for LLM-specific failure modes (prompt regression, provider outage, retrieval drift, hallucination spike, cost blowout, guardrail false-positive storm), SEV classification, status-page communication, blameless post-mortems with permanent regression-fixture back-feed into the eval gate
- Design the field-feedback-into-eval loop — every incident becomes at least one permanent regression fixture; every customer bug report becomes at least one labelled gold-set entry

## Structure

- `01-…md` … `0N-…md`: lecture chapters.
- `exercises/`: per-exercise prompts.
- `labs/`: long-form hands-on labs.
- `quizzes/`: knowledge checks.
- `resources.md`: external references.
