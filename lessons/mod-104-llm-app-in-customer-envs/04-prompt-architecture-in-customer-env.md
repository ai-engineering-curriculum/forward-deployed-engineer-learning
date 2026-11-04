# Chapter 04 — Production Prompt Architecture at the Customer Level

## Why this chapter exists

Prompt design, structured outputs, streaming, prompt caching, and A/B rollout are craft owned by [`applied-ai-engineer`](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) at level 30. This chapter does not re-teach that craft. It writes the overlay the FDE adds when those prompts have to run inside N different customer environments, with per-tenant overrides, with cache strategies that compose with the gateway from Chapter 03, with a review workflow that answers to the customer's change-management process as well as the vendor's.

The core question this chapter answers: *how does the prompt become a per-deployment, per-tenant artifact with a signed lineage, without losing the properties that make the in-house craft work (cacheable prefixes, validated outputs, streaming UX)?*

## Structured outputs under the overlay

### The in-house craft

The applied-AI engineer picks a schema (JSON Schema / Pydantic / Zod), passes it to a provider that supports structured output, validates on return, retries on invalid. Three primary idioms:

- **JSON Schema** — [json-schema.org](https://json-schema.org). The lingua franca. OpenAI's [Structured Outputs](https://platform.openai.com/docs/guides/structured-outputs) consumes JSON Schema directly. Google Gemini's [controlled generation](https://ai.google.dev/gemini-api/docs/structured-output) accepts a subset. Anthropic's tool-use shape ([tool use](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use)) takes `input_schema` as JSON Schema; for structured return, declare a single tool whose schema *is* the output shape and prompt the model to always call it.
- **Pydantic** — [docs.pydantic.dev](https://docs.pydantic.dev). Python-first, compiles to JSON Schema (`model_json_schema()`). Straight path to provider-native structured outputs.
- **Zod** — [zod.dev](https://zod.dev). TypeScript-first, with JSON Schema bridges ([zod-to-json-schema](https://github.com/StefanTerdell/zod-to-json-schema) and Vercel AI SDK integrations).

### The overlay

The customer-environment twist: **the schema is a contract with the customer's downstream consumer.** Changes to the schema are customer-facing changes with their own review and rollout — they are not "just a prompt update."

Concretely:

- The pilot's output schema is referenced by at least one downstream consumer inside the customer environment: a workflow engine (ServiceNow / Salesforce / Jira), a data warehouse load job, a UI the SME uses, a monitoring dashboard. Each is a binder.
- Adding an optional field is low-risk (consumers that don't know the field ignore it). Removing a field, renaming a field, tightening a type, or changing an enum value are *breaking*.
- Breaking changes cross the integration-design boundary (mod-103 Chapter 04's schema-registry contracts). They go through the customer's change-management process (mod-109), not just the vendor's prompt-review process.

The schema lives in a registry with versions. `cases.v3.2` is a specific locked shape. Prompt v5 may emit `cases.v3.2`; prompt v6 may emit `cases.v4.0` — the pilot's rollout wires the two versions together with a feature flag.

### Provider structured-output constraints

Each provider has specific constraints the FDE must know before committing a schema:

- **OpenAI** — Structured Outputs supports JSON Schema with a defined subset (no `patternProperties`, limited support for `oneOf`, properties required by default). The [guide](https://platform.openai.com/docs/guides/structured-outputs) lists the supported subset at the time of reading.
- **Anthropic** — No "structured outputs" API endpoint; use tool-use with a single tool whose `input_schema` is the output shape. The model's reliability on this pattern is high but not provider-enforced.
- **Google Gemini** — Controlled generation supports `response_schema` with a defined JSON Schema subset; `enum` and `nullable` are first-class.
- **Bedrock** — passthrough to the underlying provider's shape (Anthropic via Messages API; Meta / Mistral vary).

The FDE writes against the intersection when the pilot may route across providers — otherwise against the specific provider's shape with the gateway alias pinned.

## Streaming under the overlay

### The in-house craft

Stream the completion token-by-token. Render progressively. Parse structured output incrementally if the UI needs partial state. Primary references:

- OpenAI streaming: [streaming responses](https://platform.openai.com/docs/api-reference/responses-streaming).
- Anthropic streaming: [Messages streaming](https://docs.anthropic.com/en/docs/build-with-claude/streaming).
- Vercel AI SDK: [AI SDK — streaming](https://sdk.vercel.ai/docs) for client-side patterns.

### The overlay

Three things the customer-environment overlay adds:

1. **Streaming through the gateway.** The gateway (Chapter 03) must proxy the stream without buffering. LiteLLM proxy supports streaming end-to-end; verify this on deployment with a smoke test, because some reverse-proxy layers (nginx without the right config, enterprise WAFs) buffer SSE responses by default.
2. **Streaming through the egress proxy.** The customer's egress proxy (mod-103 Chapter 07) may also buffer. Both the customer's AppSec and the FDE run a joint test — send a known-slow prompt (deliberately long context, slow model) and verify the UI sees the first token quickly.
3. **Streaming with structured outputs.** Partial JSON is not valid JSON. The pilot either (a) renders the raw text stream and switches to structured parsing on close, (b) uses a streaming JSON parser tolerant of trailing partial state (e.g. `partial-json` libraries), or (c) streams at the schema-field granularity if the provider supports it. Choose explicitly; do not leave undefined behaviour.

## Prompt caching under the overlay

### Why provider-native prompt caching matters

Every provider that supports it gives a meaningful cost and latency reduction when the *static prefix* of the prompt is reused across requests:

- **Anthropic prompt caching** — [prompt caching docs](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching). Mark cacheable blocks with `cache_control`. Significant cost reduction on cached prefix tokens.
- **OpenAI prompt caching** — [prompt caching guide](https://platform.openai.com/docs/guides/prompt-caching). Automatic on supported models when prompts share a long static prefix.
- **AWS Bedrock prompt caching** — [Bedrock prompt caching](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-caching.html) for supported models. Mark cache checkpoints in the request.
- **Google Vertex AI** — context caching available on supported Gemini models; see [context caching](https://cloud.google.com/vertex-ai/generative-ai/docs/context-cache/context-cache-overview).

Each provider's primary docs are the source of truth for cacheable-token minimums, TTL defaults, and pricing. Numbers change; the FDE reads the current docs when designing the prompt.

### The overlay: factoring the prompt for cacheability with per-tenant overrides

The naive customer-environment pattern — one prompt per tenant, forked — destroys cacheability. Each tenant's prompt has a different prefix; the provider's cache does not share across prefixes.

The right pattern factors the prompt into three sections, ordered carefully:

```
┌────────────────────────────────────────────────────────┐
│  (A) Static global prefix                              │
│      - Persona / role                                  │
│      - Task description                                │
│      - Output schema definition                        │
│      - Global few-shot examples                        │
│      - Static tool definitions                         │
│      *** cacheable boundary (cache_control / checkpoint)***
├────────────────────────────────────────────────────────┤
│  (B) Tenant-config section (semi-stable)               │
│      - Tenant-specific terminology dictionary          │
│      - Tenant-specific tone / style preferences        │
│      - Tenant-specific output-format preferences       │
│      - Tenant-specific small examples                  │
│      *** optional second cacheable boundary ***
├────────────────────────────────────────────────────────┤
│  (C) Per-request content (not cacheable)               │
│      - The actual input to process                     │
│      - Retrieval-grounded chunks (mod-105)             │
│      - Session history if any                          │
└────────────────────────────────────────────────────────┘
```

Properties:

- Section (A) is identical across tenants; it caches once at the provider and every tenant's requests benefit.
- Section (B) is per-tenant but static for that tenant; if the provider supports two cache tiers (Anthropic supports multiple `cache_control` blocks), the per-tenant prefix caches at tenant scope.
- Section (C) is per-request and never caches.

The anti-pattern — putting tenant-specific text anywhere inside section (A), such as "You are an assistant for ACME Corp." at the top — moves the whole prefix out of the shared cache. Every tenant then has their own cache entry, and the cross-tenant benefit vanishes.

Schematic Python (shape only; the provider SDK call is schematic):

```python
def build_prompt(tenant_id: str, user_input: str, retrieved: list[str]) -> dict:
    # (A) Static global prefix — shared across tenants, cached by provider.
    static_prefix = load_prompt_template(name="draft-case-summary", version="v5")

    # (B) Tenant-config — same across requests for this tenant.
    tenant_cfg = load_tenant_config(tenant_id)
    tenant_block = render_tenant_block(tenant_cfg)

    # (C) Per-request content.
    request_block = render_request(user_input, retrieved)

    # Provider-specific cache_control shape (Anthropic example)
    return {
        "system": [
            {"type": "text", "text": static_prefix, "cache_control": {"type": "ephemeral"}},
            {"type": "text", "text": tenant_block, "cache_control": {"type": "ephemeral"}},
        ],
        "messages": [{"role": "user", "content": request_block}],
    }
```

For other providers, the cache-marking idiom differs; the three-section factoring is the same.

### Verifying the cache is actually warm

Every provider that supports prompt caching surfaces a response metric telling you how many cached tokens were used — Anthropic returns `cache_read_input_tokens`; OpenAI's response usage includes `cached_tokens`; Bedrock returns cache metrics in the response. The gateway's audit log (Chapter 03) captures these fields; a dashboard tracks cache-hit rate per tenant per model.

Target cache hit rates depend on workload. For a steady background task over a single tenant, cache hit rate on prefix tokens should be high after warm-up. A sudden drop indicates either a prompt-version change, a tenant-config change, or a provider-side cache eviction — all three worth paging on.

## Prompt versioning and review workflow

### The prompt as a versioned artifact

A prompt is a named, versioned, hashed, authored, reviewed artifact. Each prompt in the registry carries:

```
Prompt record

  id:            draft-case-summary
  version:       v5
  hash:          sha256:3f9c...
  author:        <email>
  status:        proposed | approved | deployed | deprecated
  cacheable_prefix_hash: sha256:aa21...
  schema_ref:    cases.v3.2
  reviewed_by:   [<reviewer email>, ...]
  review_log:    [<link to review record>, ...]
  eval_results:  [<link to mod-107 eval runs>, ...]
  deployed:      [{env: staging, tenants: [*], at: <ts>}, ...]
  created_at:    <ts>
  deprecated_at: null
```

Where it lives: a prompt registry — can be a simple Git repository of YAML / Markdown files under version control, or a managed tool (see below). Either way:

- The prompt content is version-controlled, every change is a PR, every review is a review comment.
- The pilot code loads prompts by `(id, version, tenant_id)` — never by file path.
- The current-deployed version is chosen by a feature flag (mod-109), not by code.

### The review workflow

Steps:

1. **Propose** — author opens a change (PR or tool-equivalent). Diff against the previous version is auto-generated.
2. **Eval on a sample set** — the proposed prompt runs against a small sample set (gold-set-adjacent per mod-107). The runner captures outputs, structured-output validity, latency, token usage, and (if available) judge-as-LLM scores.
3. **Side-by-side review** — the reviewer sees the diff and the sample outputs side by side with the previous version.
4. **Approve / changes-requested / reject** — standard code-review semantics.
5. **Rollout** — the approved prompt is tagged "deployed" and bound to a feature flag that controls rollout (mod-109). Options: switch all tenants, canary to a subset, A/B against the previous version.
6. **Monitor** — post-rollout, the gateway's audit log and the LLM-observability stack (mod-110) watch for cache-hit regression, latency drift, quality drift.
7. **Deprecate** — the old version is kept in the registry (immutable) but marked deprecated once rollout completes.

### Tooling

Several tools occupy this space, each with different emphasis. The FDE picks per customer based on AppSec, operational familiarity, and eval-tool continuity (mod-107):

- **Promptfoo** — [promptfoo.dev](https://www.promptfoo.dev). Open-source eval + prompt-management CLI, YAML-defined test cases, side-by-side comparisons. Integrates with CI.
- **Langfuse** — [langfuse.com](https://langfuse.com). Open-source and SaaS; prompt management, tracing, eval. Self-hostable.
- **LangSmith** — [docs.smith.langchain.com](https://docs.smith.langchain.com). SaaS (and dedicated instances); strong tracing, prompt playground, eval. LangChain ecosystem integration.
- **PromptLayer** — [promptlayer.com](https://promptlayer.com). SaaS prompt-management and observability.

Choice pattern:

- **Promptfoo + Git** when the customer prefers no additional SaaS and the FDE team is comfortable operating lightly.
- **Langfuse self-hosted** when the customer wants a proper UI inside their VPC and no vendor onboarding.
- **LangSmith / Langfuse SaaS / PromptLayer** when the DPA already covers the vendor and the UX is worth the dependency.

Avoid picking a winner at module level; the right tool is customer-shape dependent.

## A/B prompt shape

A/B here means: route a slice of traffic to prompt `v5` and another slice to `v6`, measure, decide. Three levers:

- **Per-tenant** — tenant A is on v5, tenant B is on v6. Simplest. Appropriate for first rollout to a new tenant.
- **Per-user inside a tenant** — the tenant's users are split into cohorts. Appropriate when the tenant is large and the pilot wants statistical significance within one customer.
- **Per-request** — random assignment. Appropriate for canary / shadow where the request itself is the unit of randomisation.

Mod-109 owns the feature-flag tool choice (LaunchDarkly / Statsig / Unleash / OpenFeature). Chapter 04's concern is that the prompt system can be *called* by an identifier picked by the feature-flag layer, and that the audit log records which version actually answered.

## Per-tenant prompt overrides

The pattern:

- One default prompt per task (sections A + stub-B + C factoring).
- A tenant-config registry keyed by `tenant_id`, containing the data that renders section B (terminology dictionary, tone directives, output-format preferences, small tenant-specific examples).
- At request time: `render(prompt, tenant_cfg(tenant_id), user_input)`.

This composes with prompt caching (A is shared, B is per-tenant-cached, C is fresh) and with versioning (prompt versions are task-level; tenant-configs are tenant-level; they evolve independently).

The alternative — fork the whole prompt per tenant — is tempting in week 2 ("I just need to add this one tenant-specific instruction"). It compounds. By week 10 the registry has ten prompts that are 90% identical, each requiring a separate review for every change. The shared-with-config pattern scales; the forked pattern does not.

### When a forked prompt is justified

A tenant whose workflow is genuinely different from the baseline — a specialised vertical, a materially different output schema, a legally mandated phrasing requirement — is a candidate for a fork. Document the fork as a design decision:

- **Rationale** — why a config patch cannot express the difference.
- **Divergence scope** — what is different, what is shared.
- **Maintenance owner** — who keeps the fork in sync as the base evolves.
- **Sunset condition** — what would let us un-fork later.

Without the document, the fork is drift.

## Prompt security concerns under the overlay

Depth lives in mod-106 (agentic hardening) and mod-108 (security review). The overlay point here:

- **Per-tenant override edits create a tenant-level injection surface.** A tenant admin can edit their terminology dictionary; if the edit flows directly into the prompt without review, a crafted entry can alter system behaviour. Tenant-config edits route through the same review workflow as prompts, scaled to the risk.
- **Retrieved content (mod-105) is an injection surface.** The gateway's PII guard (Chapter 03) does not catch prompt injection in retrieved chunks; that is mod-106's domain. The FDE pairs with the customer on retrieval-source policy before enabling retrieval against user-generated content.
- **Audit the prompt, not just the response.** The audit log (Chapter 03) records `prompt_version`; the registry records the prompt content at that version. The two together reconstruct exactly what was sent. Never log the fully-rendered prompt (section C contains PII); the version hash is the reconstruction key.

## Anti-patterns

**Forking per-tenant prompts without a registry.** Covered above. The next FDE inherits ten lookalike prompts and no diff story.

**Shipping schema changes as "just a prompt update."** The schema binds the pilot to downstream consumers. Schema changes get their own ADR and rollout (mod-109).

**Caching across tenants.** Covered in Chapter 03 for the gateway's cache; same principle applies to provider-native caching if a shared prefix contains tenant-specific text. The three-section factoring prevents this at the source.

**No eval gate on the review workflow.** A prompt change that looks good on three hand-picked examples regresses on the SME's actual edge cases. The review workflow runs the eval sample set before approval; mod-107 writes the evals.

**Trusting the registry UI as the source of truth.** If prompts live in a SaaS prompt-management tool and the pilot loads them at runtime over the network, the tool becomes a new availability dependency. For self-hosted pilots, the registry contents should be mirrored to the pilot's own store (filesystem or database) so a network blip to the SaaS tool does not take the pilot down.

## Summary

- The customer-environment overlay on prompt architecture: schema is a customer-facing contract, prompts carry per-tenant overrides, the review workflow composes with the customer's change-management, cache strategy composes with the gateway.
- Structured outputs — provider-specific shapes (OpenAI structured outputs, Anthropic tool-use, Gemini controlled generation, Bedrock passthrough); schemas are versioned artifacts that bind downstream consumers.
- Streaming — verify end-to-end through the gateway and the egress proxy; decide the structured-output streaming strategy explicitly.
- Prompt caching — factor the prompt into three sections (static global, tenant-config, per-request) so the cacheable boundary is preserved; tenant-specific text must not appear in the static prefix.
- Prompt versioning — a prompt is a registered artifact with id, version, hash, author, reviewer, eval results, deployment history; the pilot loads by `(id, version, tenant_id)`.
- Review workflow — propose, eval on sample set, side-by-side review, approve, roll out via feature flag (mod-109), monitor, deprecate.
- A/B — per-tenant / per-user / per-request; feature-flag tool choice belongs to mod-109; the prompt system exposes the identifier.
- Per-tenant overrides — one default prompt plus a tenant-config registry patches section B. Forked prompts are justified only with a sunset document.
- Anti-patterns: unregistered forks, schema-change-as-prompt-update, cross-tenant cache, review without eval gate, SaaS-registry as runtime dependency.
- Chapter 05 is next: how the latency and cost measured through the gateway translate into customer-contracted SLOs.
