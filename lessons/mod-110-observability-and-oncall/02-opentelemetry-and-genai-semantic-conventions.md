# Chapter 02 — OpenTelemetry Instrumentation with GenAI Semantic Conventions

## Why this chapter exists

Chapter 01 named signal fidelity as a gate dimension — if the on-call cannot answer "which prompt-template, which model, which retriever, which judge produced this output?" the pilot is undebuggable. This chapter is where fidelity gets engineered. The answer is **OpenTelemetry** as the wire protocol and **the OpenTelemetry GenAI semantic conventions** as the attribute vocabulary. Picking those two decisions correctly means the customer's existing SIEM / APM — Datadog, Splunk, Elastic, Grafana, Dynatrace, New Relic, Microsoft Sentinel — consumes the pilot's telemetry natively, with no bespoke parser, no vendor-proprietary SDK locked to one backend, and no second schema to learn when the next pilot lands on a different APM.

The chapter has three jobs. First, name what OpenTelemetry is and what the GenAI semantic conventions commit to, so the FDE can speak the standard with the customer's observability team. Second, specify the collector topology — where the OpenTelemetry Collector runs, who owns its config, where it exports — for each deployment archetype from [mod-109 Chapter 02](../mod-109-deployment-rollout-change-management/02-deployment-modes-saas-vpc-onprem.md). Third, write the redaction pipeline that keeps raw prompts and completions from leaking into a destination that has not been cleared for them under the data contract from [mod-108 Chapter 05](../mod-108-enterprise-security-and-compliance/05-data-contract.md).

## OpenTelemetry at ten thousand feet

**OpenTelemetry (OTel)** is a CNCF project that standardises three things: a **data model** for traces / metrics / logs, an **SDK** that produces them in that model, and a **wire protocol (OTLP)** that ships them to a backend. It is the industry convergence layer. Every major APM — Datadog, Splunk Observability Cloud, Elastic APM, Grafana (Tempo / Mimir / Loki), Dynatrace, New Relic, Honeycomb, Azure Monitor, AWS CloudWatch Application Signals, Google Cloud Operations — ingests OTLP, either natively or through a translator. <!-- needs-research: confirm current OTLP ingestion status for each named APM at time of publication — the matrix evolves -->

The three signals, in LLM-pilot terms:

- **Traces.** A tree of spans representing one logical request through the pilot. An LLM invocation produces (at minimum) a root span for the user-facing operation (e.g. `agent.answer_member_inquiry`), a child span for each retrieval call, a child span for each model call, a child span for each tool call, a child span for each guardrail check, a child span for each judge call. Spans carry attributes (structured key-value fields) and events (timestamped structured messages).
- **Metrics.** Pre-aggregated numeric measurements over time. For LLM pilots: token usage counters (per model, per prompt-template, per tenant), latency histograms (TTFT, TPOT, end-to-end), cost counters, refusal-rate counters, guardrail-trip counters, judge-pass gauges.
- **Logs.** Text or structured records carrying detail that does not fit as trace attributes — long completions, retrieval payloads, stack traces, audit events. OTel logs are correlated to traces via `trace_id` / `span_id` so a log line resolves to its originating invocation.

The **OpenTelemetry Collector** is a separate binary — not part of the SDK — that receives OTLP from the SDK, transforms it (filter, batch, sample, redact, enrich), and exports it to one or more backends. The Collector is the architectural centre of this chapter: the SDK's job is to emit the standard; the Collector's job is to reshape and route.

### Why OTel specifically

The alternative is a vendor-proprietary SDK (the Datadog agent library, the New Relic APM agent, the Dynatrace OneAgent). Three reasons the FDE chooses OTel:

1. **Backend portability.** The pilot may land on Datadog in Archetype A, Splunk in Archetype B, Sentinel in Archetype C — and often more than one inside the same customer. OTel instruments once; routing moves in the Collector config, not in the application code.
2. **Shared vocabulary with the customer.** The customer's SRE / observability team already instruments their own services with OTel in most new build-outs. An LLM pilot that speaks OTel inherits the customer's existing dashboards, conventions, and runbook template. A proprietary SDK makes the pilot an island.
3. **Standards traction specifically for GenAI.** The [OpenTelemetry Semantic Conventions for Generative AI](https://opentelemetry.io/docs/specs/semconv/gen-ai/) are the only cross-vendor attempt at a stable attribute vocabulary for LLM operations; they are the input every APM vendor is now mapping to in their LLM Observability features. <!-- needs-research: the GenAI semconv status is "Experimental" at publication time; confirm current status (stable / experimental / release-candidate) before shipping to a customer -->

### What the GenAI semantic conventions commit to

The semconv specifies the **attribute names, value types, and span names** for GenAI operations so a span emitted by an OTel SDK from a Python app talking to Bedrock looks the same, at the attribute level, as a span emitted from a TypeScript app talking to Anthropic directly.

The attribute namespace is `gen_ai.*`. The commonly instrumented attributes for a model-call span:

- `gen_ai.system` — the provider identifier (e.g. `anthropic`, `openai`, `aws.bedrock`, `azure.ai.openai`, `vertex_ai`). <!-- needs-research: confirm current enumerated values; this list was in flux at publication -->
- `gen_ai.request.model` — the model identifier requested by the caller (e.g. `claude-opus-4-8`, `gpt-4o`).
- `gen_ai.response.model` — the model identifier the provider reports served the request. May differ from `request.model` when the provider aliases or routes.
- `gen_ai.operation.name` — the operation performed (`chat`, `text_completion`, `embeddings`, `generate_content`, `tool_call`). <!-- needs-research: confirm enumerated values; the operation vocabulary changed between semconv revisions -->
- `gen_ai.request.temperature`, `gen_ai.request.top_p`, `gen_ai.request.top_k`, `gen_ai.request.max_tokens` — standard sampling parameters.
- `gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens` — token counts as returned by the provider. (Earlier semconv revisions used `prompt_tokens` / `completion_tokens`; the stable names are the `usage.*` form.) <!-- needs-research: confirm whether the deprecation of prompt_tokens / completion_tokens is complete at publication time -->
- `gen_ai.response.finish_reasons` — array of per-choice finish reasons.
- `gen_ai.conversation.id` — a stable identifier for a multi-turn session, when the application tracks one.
- Span name convention: `{operation.name} {model}` (e.g. `chat claude-opus-4-8`) for provider-level spans.

For retrieval / tool-call spans, the semconv adds companion attributes — `gen_ai.tool.name`, `gen_ai.tool.call.id`, and a growing set for retrieval ([mod-105 Chapter 03](../mod-105-grounding-on-customer-data/) territory, worth cross-verifying before shipping). <!-- needs-research: the retrieval / RAG attribute namespace under gen_ai.* is actively being specced; confirm current attributes before authoring the schema -->

For *content* capture — the actual prompt and completion strings — the semconv provides opt-in event-based mechanisms (`gen_ai.user.message`, `gen_ai.assistant.message`, `gen_ai.choice`) rather than string attributes, because:

- Content is often PII / PHI / confidential and must be routed and retained differently from metadata.
- Content is typically large and bloats trace stores if emitted indiscriminately.
- Backends that render chat transcripts need structured event shape, not a flat attribute.

The redaction pipeline later in this chapter is where "to capture or not to capture" becomes "which destination gets which fields." <!-- needs-research: confirm current event-vs-attribute convention for message content in the GenAI semconv at publication time -->

## The attribute set the FDE actually instruments

Standards-compliance is a floor, not a ceiling. For a customer pilot, the attribute set extends the semconv with fields the customer's operator needs to triage, the FDE needs to debug, and the compliance team needs to audit.

The per-invocation attribute set, grouped:

**Identity and tenancy.**
- `gen_ai.conversation.id`, `gen_ai.session.id` (per semconv when applicable).
- `pilot.tenant_id` — the customer-scoped tenant identifier from [mod-103 Chapter 03](../mod-103-enterprise-integration/) (name may vary).
- `pilot.user.id` — pseudonymised user identifier (a hash, never the raw email or employee ID).
- `pilot.user.role` — the role classification the customer's RBAC exposes (e.g. `analyst`, `supervisor`).

**Artefact-bundle lineage** (from [mod-109 Chapter 05](../mod-109-deployment-rollout-change-management/05-per-tenant-versioning-and-replay.md)).
- `bundle.version` — the versioned artefact-bundle identifier the invocation ran on.
- `bundle.prompt_template.id`, `bundle.prompt_template.version` — the specific prompt-template.
- `bundle.retriever.id`, `bundle.retriever.index.version` — retrieval configuration.
- `bundle.model.pin` — the resolved model identifier at the time of invocation.
- `bundle.tool_allowlist.hash` — hash of the tool-allow-list applied.
- `bundle.judge.id`, `bundle.judge.version` — judge configuration.
- `bundle.guardrails.hash` — guardrail-config hash.
- `bundle.gateway.config.version` — gateway configuration version.

**Operation specifics** (from GenAI semconv).
- `gen_ai.system`, `gen_ai.request.model`, `gen_ai.response.model`, `gen_ai.operation.name`.
- `gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens`.
- `gen_ai.response.finish_reasons`.
- `gen_ai.request.temperature`, `gen_ai.request.top_p`, `gen_ai.request.top_k`, `gen_ai.request.max_tokens`.

**Cost.**
- `pilot.cost.input_tokens.usd`, `pilot.cost.output_tokens.usd`, `pilot.cost.total.usd` — cost rollup per invocation at the rate card known at the time.
- `pilot.cost.rate_card.version` — the specific per-model price table used.

**Latency.**
- `pilot.ttft.ms` — time-to-first-token (for streaming responses).
- `pilot.tpot.ms` — time-per-output-token mean for the response.
- End-to-end latency is captured naturally by the span duration.

**Retrieval** (per [mod-105](../mod-105-grounding-on-customer-data/) spans).
- `retrieval.query.hash` — hash of the query string (not the raw string unless the destination is content-cleared).
- `retrieval.k_requested`, `retrieval.k_returned`.
- `retrieval.rerank.score_mean`, `retrieval.rerank.score_min`, `retrieval.rerank.score_max`.
- `retrieval.document.ids` — opaque document identifiers (array of hashes / URIs; never raw content).
- `retrieval.index.name`, `retrieval.index.version`, `retrieval.embedding.model`.

**Agent / tool-call** (per [mod-106](../mod-106-agentic-workflows-on-enterprise/) spans).
- `gen_ai.tool.name`, `gen_ai.tool.call.id` (semconv).
- `tool.authorized_by.hitl_checkpoint_id` — if the call passed through a human-in-the-loop checkpoint ([mod-106 Chapter 04](../mod-106-agentic-workflows-on-enterprise/04-human-in-the-loop-checkpoints.md)).
- `tool.allowlist.decision` — `allowed` / `denied`.
- `tool.target.scope` — the resource scope touched (e.g. `salesforce.account.read`).

**Guardrails.**
- `guardrail.id`, `guardrail.version`, `guardrail.decision` — `allow` / `block` / `rewrite`.
- `guardrail.reason` — the guardrail rule or category that fired (never the raw user input).

**Online eval / judges** (per [mod-107 Chapter 05](../mod-107-eval-eng-on-customer-data/05-online-eval.md)).
- `judge.id`, `judge.version`, `judge.score` — judge configuration and score.
- `judge.rubric.id`, `judge.rubric.version` — the rubric evaluated.

**Rollout context** (per [mod-109 Chapter 04](../mod-109-deployment-rollout-change-management/04-canary-shadow-ab-rollouts.md)).
- `rollout.arm` — `control` / `canary` / `shadow` / `experiment:A` / `experiment:B`.
- `rollout.flag.id`, `rollout.flag.variation`.
- `rollout.canary.step` — `1%`, `5%`, `25%`, `100%` (when applicable).

**Incident / kill-switch state** (per [mod-109 Chapter 03](../mod-109-deployment-rollout-change-management/03-feature-flag-strategy.md)).
- `killswitch.active` — boolean; always emitted for defence-in-depth so an incident analyst can filter.

That set is the baseline. For each pilot, additional fields land for workflow-specific context (e.g. an Archetype A member-inquiry span adds `pilot.intent.classification` so dashboards can slice by intent class). The additions are versioned in the same manifest Chapter 05 of mod-109 names.

## A concrete instrumentation example

A Python agent orchestrating one model call plus one retrieval call, instrumented against the schema above. Shape-only; the imports and client wiring would come from the vendor's SDK.

```python
from opentelemetry import trace, metrics
from opentelemetry.trace import SpanKind, Status, StatusCode

tracer = trace.get_tracer("pilot.agent")
meter = metrics.get_meter("pilot.agent")

# Metrics set up once at import time
token_in_counter = meter.create_counter(
    "gen_ai.client.token.usage",
    unit="{token}",
    description="Input / output tokens consumed per invocation",
)
cost_counter = meter.create_counter(
    "pilot.cost.usd",
    unit="USD",
    description="Cost per invocation at the active rate card",
)
latency_histogram = meter.create_histogram(
    "gen_ai.client.operation.duration",
    unit="ms",
    description="End-to-end duration per LLM invocation",
)

def answer_member_inquiry(req, bundle, user_ctx):
    with tracer.start_as_current_span(
        "agent.answer_member_inquiry",
        kind=SpanKind.INTERNAL,
        attributes={
            "pilot.tenant_id": user_ctx.tenant_id,
            "pilot.user.id": user_ctx.user_id_hash,
            "pilot.user.role": user_ctx.role,
            "bundle.version": bundle.version,
            "bundle.prompt_template.id": bundle.prompt_template.id,
            "bundle.prompt_template.version": bundle.prompt_template.version,
            "bundle.model.pin": bundle.model.pin,
            "bundle.retriever.id": bundle.retriever.id,
            "bundle.judge.id": bundle.judge.id,
            "bundle.guardrails.hash": bundle.guardrails.hash,
            "rollout.arm": bundle.rollout_arm,
            "killswitch.active": bundle.killswitch_active,
        },
    ) as root:
        try:
            docs = _retrieve(req.query, bundle, user_ctx)
            response = _chat(req, docs, bundle, user_ctx)
            judge_score = _judge(req, response, docs, bundle)
            root.set_attribute("judge.score", judge_score)
            return response
        except ModelProviderError as e:
            root.set_status(Status(StatusCode.ERROR, "provider_error"))
            root.record_exception(e)
            raise

def _chat(req, docs, bundle, user_ctx):
    with tracer.start_as_current_span(
        f"chat {bundle.model.pin}",
        kind=SpanKind.CLIENT,
        attributes={
            "gen_ai.system": bundle.model.provider,
            "gen_ai.request.model": bundle.model.requested,
            "gen_ai.operation.name": "chat",
            "gen_ai.request.temperature": req.temperature,
            "gen_ai.request.max_tokens": req.max_tokens,
            "pilot.tenant_id": user_ctx.tenant_id,
            "bundle.version": bundle.version,
        },
    ) as span:
        start = _now_ms()
        resp = model_client.invoke(req, docs, bundle)
        span.set_attributes({
            "gen_ai.response.model": resp.served_model,
            "gen_ai.usage.input_tokens": resp.usage.input_tokens,
            "gen_ai.usage.output_tokens": resp.usage.output_tokens,
            "gen_ai.response.finish_reasons": resp.finish_reasons,
            "pilot.ttft.ms": resp.ttft_ms,
            "pilot.tpot.ms": resp.tpot_ms,
            "pilot.cost.input_tokens.usd": resp.cost.in_usd,
            "pilot.cost.output_tokens.usd": resp.cost.out_usd,
            "pilot.cost.total.usd": resp.cost.total_usd,
            "pilot.cost.rate_card.version": resp.cost.rate_card_version,
        })
        token_in_counter.add(resp.usage.input_tokens,
                             {"gen_ai.request.model": bundle.model.requested,
                              "token.type": "input",
                              "pilot.tenant_id": user_ctx.tenant_id})
        token_in_counter.add(resp.usage.output_tokens,
                             {"gen_ai.request.model": bundle.model.requested,
                              "token.type": "output",
                              "pilot.tenant_id": user_ctx.tenant_id})
        cost_counter.add(resp.cost.total_usd,
                         {"gen_ai.request.model": bundle.model.requested,
                          "pilot.tenant_id": user_ctx.tenant_id})
        latency_histogram.record(_now_ms() - start,
                                 {"gen_ai.request.model": bundle.model.requested,
                                  "pilot.tenant_id": user_ctx.tenant_id,
                                  "rollout.arm": bundle.rollout_arm})
        return resp
```

Three things to notice. First, every span and every metric point carries `pilot.tenant_id` so the customer's view is filterable by tenant without needing a separate pipeline. Second, cost and token counters are *metrics*, not derived at query time from traces — metric storage is orders of magnitude cheaper for aggregation queries that the customer will run constantly. Third, the raw prompt and completion strings are *not* in this code path — content capture happens separately, through the OTel event mechanism, routed to a separate destination.

## Semantic-convention adoption in the major APMs

The attribute set only buys native consumption if the customer's APM parses it. Current state as of publication:

- **[Datadog LLM Observability](https://docs.datadoghq.com/llm_observability/).** Ingests OTLP; maps `gen_ai.*` attributes to Datadog's LLM Observability feature; renders a "trace view" with model / tokens / cost / latency per span. <!-- needs-research: confirm the specific semconv revision Datadog LLM Observability maps against at publication time, and the current product name -->
- **[Dynatrace](https://www.dynatrace.com/platform/artificial-intelligence/).** AI Observability product reads OpenTelemetry GenAI spans. <!-- needs-research: confirm product name and current semconv-revision support -->
- **[New Relic AI Monitoring](https://newrelic.com/platform/ai-monitoring).** Maps GenAI semconv; dashboards render tokens / cost / model usage. <!-- needs-research: confirm current product capabilities -->
- **[Splunk Observability Cloud](https://www.splunk.com/en_us/products/observability.html).** Ingests OTLP; GenAI-specific views via APM or via log-based analytics on OTLP-shaped log records. <!-- needs-research: confirm Splunk's current GenAI-specific feature naming -->
- **[Elastic Observability](https://www.elastic.co/observability).** OTLP-native; integrates with Elastic's Kibana dashboards. <!-- needs-research: confirm Elastic's GenAI-specific analytic surface -->
- **[Grafana Cloud](https://grafana.com/products/cloud/).** Tempo (traces), Mimir (metrics), Loki (logs) ingest OTLP; Grafana's LLM-specific dashboard templates exist in the community. <!-- needs-research: confirm whether Grafana Labs ships an official LLM Observability package at publication time -->
- **[Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/) + [Azure Monitor / Application Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/).** Azure Monitor consumes OTLP; Application Insights renders GenAI spans; Sentinel reads the correlated log destination. <!-- needs-research: confirm Azure Monitor's current GenAI semconv coverage -->
- **[AWS CloudWatch Application Signals](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Application-Signals.html).** OTLP-native; LLM-specific dashboards via managed Grafana for Bedrock workloads. <!-- needs-research: confirm Application Signals' current GenAI attribute coverage -->
- **[Google Cloud Operations Suite](https://cloud.google.com/products/operations).** OTLP-native; Cloud Trace / Cloud Logging / Cloud Monitoring consume `gen_ai.*`. <!-- needs-research: confirm Google Cloud's current LLM-specific analytic surface -->
- **[Honeycomb](https://www.honeycomb.io/).** OTLP-native; general-purpose high-cardinality trace analysis renders GenAI spans without a dedicated product layer.

The pattern: every serious APM now ingests OTLP natively, and most ship a GenAI-specific analytic layer on top. The FDE's job is to verify *which revision of the semconv* the customer's APM currently parses — attribute renames between semconv revisions (e.g. `prompt_tokens` → `usage.input_tokens`) are a common source of "the Datadog LLM Observability dashboard is empty" tickets.

## Collector topology per deployment archetype

The SDK is the producer; the Collector is the shaper and the router. Where the Collector runs, who owns its config, and where it exports are per-archetype decisions.

### Archetype A — VPC install (healthcare payer)

Collector runs as a Kubernetes DaemonSet inside the pilot's EKS cluster in the customer's VPC. The pilot application exports OTLP over gRPC to the node-local Collector; the Collector exports:

- **Traces and metrics** to Datadog via the Datadog exporter, over a customer-held intake API key. The customer's SRE team already runs Datadog; adding an OTel pipeline is drop-in.
- **Logs** to the customer's Datadog logs intake with a parallel export to the customer's S3 bucket (for the HIPAA six-year retention floor; Datadog's default retention does not reach six years).
- **A copy of the trace stream** to the self-hosted Langfuse (see [Chapter 03](./03-llm-observability-platform-at-customer-posture.md)) over OTLP.

Config owned by the customer's platform team; schema evolution co-reviewed by the FDE. The Collector binary is pinned to a specific release per [mod-109 Chapter 06](../mod-109-deployment-rollout-change-management/06-customer-side-handoff-and-sme-training.md) ML-BOM.

### Archetype B — Air-gapped federal agency

Collector runs inside the agency's GovCloud enclave on dedicated nodes with no external egress. The agency's existing OTel Collector fleet (if present) is the primary; the pilot may add a tenant-scoped pipeline. Exports:

- **Logs** to the agency's Splunk Enterprise instance via the Splunk HTTP Event Collector exporter.
- **Traces** to a local Grafana Tempo instance inside the enclave.
- **Metrics** to a local Prometheus / Mimir stack inside the enclave.
- **A copy of the trace stream** to the self-hosted Arize Phoenix instance inside the enclave.

No SaaS exporters are enabled. The Collector config is reviewed against the SSP; any change runs through the agency's change-control.

### Archetype C — Dedicated SaaS on Azure (EU retail bank)

Collector runs as a sidecar in the vendor's dedicated-SaaS namespace inside the bank's Azure tenancy under the EU Data Boundary. Exports:

- **Traces, metrics, logs** to Azure Monitor + Application Insights via the Azure Monitor OTLP exporter.
- **Audit-grade log stream** to Microsoft Sentinel via the Sentinel diagnostic-settings pipe with structured retention per DORA / GDPR.
- **A copy of the trace stream** to the self-hosted Langfuse in the bank's Azure tenancy.

The vendor owns the Collector config; the bank's security team reviews before any change and the config lives in the artefact bundle manifest.

A reference Collector pipeline (YAML shape) for Archetype A:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  memory_limiter:
    check_interval: 1s
    limit_mib: 512
  batch:
    send_batch_size: 10000
    timeout: 5s
  attributes/scrub_pii:
    actions:
      - key: gen_ai.prompt
        action: delete
      - key: gen_ai.completion
        action: delete
  transform/redact_message_events:
    log_statements:
      - context: span
        statements:
          - 'delete_key(attributes, "gen_ai.user.message") where attributes["pilot.destination"] != "content_cleared"'
  filter/drop_healthchecks:
    traces:
      span:
        - 'attributes["http.route"] == "/healthz"'
  resourcedetection:
    detectors: [env, system, eks]

exporters:
  datadog:
    api:
      site: datadoghq.com
      key: ${DATADOG_API_KEY}
  otlphttp/langfuse:
    endpoint: https://langfuse.pilot.internal/api/public/otel/v1/traces
    headers:
      Authorization: ${LANGFUSE_AUTH}
  awss3/archive:
    s3uploader:
      region: us-east-1
      s3_bucket: pilot-telemetry-archive-{{tenant}}
      s3_prefix: traces
      s3_partition: minute
      compression: gzip

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, resourcedetection, attributes/scrub_pii, transform/redact_message_events, filter/drop_healthchecks, batch]
      exporters: [datadog, otlphttp/langfuse, awss3/archive]
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, resourcedetection, batch]
      exporters: [datadog]
    logs:
      receivers: [otlp]
      processors: [memory_limiter, attributes/scrub_pii, batch]
      exporters: [datadog, awss3/archive]
```

Three processors earn the architecture:

- `attributes/scrub_pii` deletes raw-content attributes before any customer-visible destination sees them. Defence in depth; the SDK should not emit them at all, but the Collector is the last line.
- `transform/redact_message_events` deletes GenAI message events except for a specifically-tagged `content_cleared` destination (the self-hosted LLM observability platform, cleared under the data contract for prompt / completion retention).
- `filter/drop_healthchecks` drops the Kubernetes health-check spans that otherwise pollute 60-80 % of the trace volume.

<!-- needs-research: confirm current OTel Collector processor names and OTTL (OpenTelemetry Transformation Language) syntax at publication time; both have evolved rapidly -->

## The redaction pipeline

Raw prompts and completions are the single largest data-contract risk in an LLM pilot. The data contract from [mod-108 Chapter 05](../mod-108-enterprise-security-and-compliance/05-data-contract.md) specifies what can land where; this chapter's redaction pipeline enforces it.

Four redaction layers, each applied at the earliest possible point.

1. **SDK-level opt-in for content capture.** The OTel instrumentation emits `gen_ai.user.message` / `gen_ai.assistant.message` events only when the pilot's config toggles `capture_content: true`. In Archetype A (PHI-bearing), the default is `false` for every destination except the self-hosted LLM observability platform, which is enabled per a specific feature flag (per [mod-109 Chapter 03](../mod-109-deployment-rollout-change-management/03-feature-flag-strategy.md)).
2. **Collector-level filtering.** As in the pipeline above, the `transform/redact_message_events` processor removes content fields before export to any destination that is not content-cleared. Each exporter is tagged; the processor routes against the tag.
3. **Destination-level detection and masking.** The content-cleared destination (Langfuse / Phoenix / Weave) runs its own PII / PHI detector (Presidio-based or equivalent) on ingest and masks named-entity spans in the stored content. <!-- needs-research: confirm Langfuse / Phoenix / Weave PII-masking capabilities at publication time; several of them integrate Microsoft Presidio but coverage differs -->
4. **Access-control-level separation.** The content-cleared destination's admin group is a *customer-held* IdP group (per [mod-103 Chapter 03](../mod-103-enterprise-integration/)). The vendor FDE does not have default access to raw prompts — the customer's operations lead or DPO grants read access under break-glass.

A table showing which destination gets which fields for Archetype A:

| Field class | Datadog APM | Datadog Logs | Datadog LLM Obs | S3 Archive | Langfuse (self-hosted) |
|---|---|---|---|---|---|
| `gen_ai.*` metadata (model, tokens, cost) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Bundle lineage (`bundle.*`) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Judge scores | ✅ | ✅ | ✅ | ✅ | ✅ |
| Retrieval metadata (doc IDs, scores) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Tool-call metadata | ✅ | ✅ | ✅ | ✅ | ✅ |
| Raw prompt (user message) | ❌ | ❌ | ❌ | ❌ (encrypted blob with BAA-scoped key) | ✅ (masked where PHI detected) |
| Raw completion (assistant message) | ❌ | ❌ | ❌ | ❌ (encrypted blob with BAA-scoped key) | ✅ (masked where PHI detected) |
| Retrieval document content | ❌ | ❌ | ❌ | ❌ (encrypted blob) | ✅ (masked) |
| User identifier (pseudonym) | ✅ | ✅ | ✅ | ✅ | ✅ |
| User identifier (raw email / MRN) | ❌ | ❌ | ❌ | ❌ | ❌ |

Archetype B's table differs on destinations (Splunk instead of Datadog; local S3-equivalent inside GovCloud); Archetype C's differs on destinations (Sentinel + Azure Blob) but the row-by-row shape is the same.

## Sampling strategy for LLM telemetry

Traces at 100 % cost real money. Metrics are already pre-aggregated; logs are already discrete. Traces are where sampling is a decision.

Three rules for LLM pilots.

1. **Head sampling for routine success is fine; tail sampling for interesting cases is mandatory.** A pilot at 10 % head sample throws away 90 % of the traces but keeps enough for most dashboards. Tail sampling — decide after the trace completes whether to keep it — catches:
   - Any invocation with `gen_ai.response.finish_reasons` containing `safety` or `content_filter`.
   - Any invocation where `judge.score` is below the pilot's rollout gate threshold.
   - Any invocation where the span duration exceeds p99 latency.
   - Any invocation where `rollout.arm` ≠ `control` (canary and shadow traces are always kept).
   - Any invocation where `pilot.cost.total.usd` exceeds a per-invocation threshold.
   - Any invocation with an error span or a guardrail block.
2. **Never sample incidents.** When an incident fires, raise the sample rate to 100 % for the pilot for the duration of the incident. The Collector config supports dynamic sample-rate reload; wire it to the incident-management tool.
3. **Keep every canary / shadow trace.** The sample budget does not apply to rollout-arm non-`control`. [Mod-109 Chapter 04](../mod-109-deployment-rollout-change-management/04-canary-shadow-ab-rollouts.md) relies on 100 % coverage of the canary arm for the gate analysis.

The OpenTelemetry Collector's `tailsamplingprocessor` implements the policy. <!-- needs-research: confirm `tailsamplingprocessor` name and current capabilities; cardinality and memory costs matter for high-volume pilots -->

## Content capture under the data contract

A separate note because it will be asked about on every pilot.

**Content capture is always a flag.** The default for a production pilot is content capture *enabled only for the content-cleared destination*, where "content-cleared" means:

- The destination is self-hosted (or dedicated-SaaS cleared under BAA / DPA).
- The destination's storage is CMK-encrypted with the customer's key.
- The destination's retention is set to the customer's regulatory floor.
- Access to the content view is governed by the customer's IdP and is subject to break-glass / four-eyes rules for any FDE access.
- A PII / PHI detector runs at ingest and masks detected entities.

For a staging or evaluation workflow, content capture may be enabled for the eval harness's gold-set builder (per [mod-107 Chapter 02](../mod-107-eval-eng-on-customer-data/02-customer-gold-set-bootstrap.md)) under the same four conditions scoped to the eval environment.

Any content capture that violates one of these conditions is a data-contract breach even if it is "temporary" or "just for this one debug session." Chapter 06 of this module writes the post-mortem convention that catches the near-miss.

## Running the three archetypes

**Archetype A — US healthcare payer.** OTel SDK in the Python pilot code; Collector as a DaemonSet in EKS; Datadog + Langfuse self-hosted + S3 archive destinations; raw prompts captured only in Langfuse, masked at ingest; six-year retention on the archive; metrics include per-tenant token / cost / latency counters. Daily dashboard the operations analyst opens: Datadog LLM Observability + Langfuse traces for triage.

**Archetype B — US federal agency.** OTel SDK in the Python agent code; Collector on dedicated enclave nodes; no SaaS exporters; Splunk + Tempo + Mimir + Phoenix destinations; content capture disabled by default, enabled for specific labelled traces only under a logged justification flag; retention per NARA + agency records policy; metrics include per-analyst and per-policy-category counters. Daily dashboard: Splunk + Phoenix. <!-- needs-research: confirm NARA records-retention general schedules relevant to AI system logs -->

**Archetype C — EU retail bank.** OTel SDK; Collector as a sidecar in the dedicated-SaaS namespace; Azure Monitor + Sentinel + Langfuse self-hosted destinations; EU Data Boundary enforced at every exporter; CMK via Azure Key Vault on every store; seven-year retention on the Langfuse archive. Daily dashboard: Azure Monitor + Langfuse; Sentinel feeds the SOC separately. EU AI Act Article 72 post-market-monitoring record pulls from Langfuse monthly.

## Anti-patterns

**Instrumenting the application but not the collector.** The SDK emits perfect `gen_ai.*` attributes; the Collector ships them raw to Datadog with no redaction, no sampling, no routing. The pipeline is one big "dump everything everywhere." Chapter 02's pipeline is an engineered pipe, not a passthrough.

**Prompt and completion strings as span attributes.** `gen_ai.prompt` as a 20KB string attribute inflates the trace store, slows every query, and routes raw content through destinations not cleared for it. The semconv uses *events* for a reason; the FDE honours the choice.

**Per-pilot bespoke attribute namespace.** The pilot ships `mycustomer_ai.model_name` and `mycustomer_ai.prompt_id` instead of `gen_ai.*` and `bundle.*`. The customer's APM renders nothing; the FDE writes a bespoke parser for every destination. Chapter 02's attribute set is the schema the pilot commits to.

**Sampling 100 % on launch, 1 % after the trace volume bill arrives.** The team instruments, launches, and then quietly turns the sample rate down when Datadog ingest costs spike. The lowered rate drops the long-tail traces the incident analysis needed. The sampling strategy is designed before launch, not retrofitted after the invoice.

**Content capture into the customer's APM "temporarily."** A debug session enables `capture_content: true` with destination `datadog`; the flag is left on; six weeks later the SIEM catches a PHI field in a Datadog log. The data-contract near-miss is a reportable incident even if it did not reach a human reader. Chapter 02 requires the content-capture flag to carry a TTL and a justification string.

**Collector run by the vendor inside the customer's VPC.** The vendor owns the Collector binary, pinned to a version the customer cannot see; a Collector CVE announcement goes unaddressed. In VPC installs, the customer owns the Collector lifecycle; the FDE ships the chart and the config.

**No per-tenant labels on metrics.** The token counter lacks `pilot.tenant_id`; the cost graph cannot slice by tenant. The customer's finance team asks "how much did Member Assist cost us last month, specifically?" and the answer is a trace-side join that costs more than the LLM did. Chapter 02's metrics carry the tenant label from day one.

## Summary

- OpenTelemetry is the wire protocol and the SDK; the OpenTelemetry Collector is the reshaper and the router; the GenAI semantic conventions are the attribute vocabulary. Picking these three lets the pilot speak the standard every serious APM already parses.
- The attribute set extends the semconv with artefact-bundle lineage, cost, latency (TTFT, TPOT), retrieval metadata, tool-call metadata, guardrail decisions, judge scores, and rollout-arm context — so one trace is enough to reconstruct the invocation end-to-end.
- Collector topology differs per archetype: VPC-install Collector as DaemonSet exporting to Datadog + self-hosted Langfuse + S3 archive; air-gapped Collector with no SaaS exporter, local Grafana / Splunk / Phoenix destinations; dedicated-SaaS Collector as sidecar in Azure under the EU Data Boundary, exporting to Azure Monitor + Sentinel + self-hosted Langfuse.
- The redaction pipeline is four-layered: SDK-level content-capture opt-in, Collector-level filter, destination-level PII / PHI detector, access-control separation. Raw prompts and completions are routed only to a content-cleared destination under customer-held keys and customer-held IdP.
- Sampling strategy: head-sample routine success; tail-sample all interesting cases (judge-fail, finish-reason safety, p99 latency, canary / shadow arm, cost spike); 100 % during incidents; 100 % on canary and shadow arms always.
- Archetype A (Datadog + self-hosted Langfuse + HIPAA six-year archive), Archetype B (Splunk + enclave Phoenix + agency records retention), Archetype C (Azure Monitor + Sentinel + Langfuse in Azure under EU Data Boundary) each illustrate the pipeline.
- Seven anti-patterns to catch early: uninstrumented Collector, prompts-as-attributes, bespoke namespace, retrofitted sampling, "temporary" content capture, vendor-owned Collector in customer VPC, unlabeled metrics.
- Chapter 03 is next: the LLM observability platform choice — Langfuse, LangSmith, Arize Phoenix, Weave, Braintrust — and how to deploy it at the customer's compliance posture.
