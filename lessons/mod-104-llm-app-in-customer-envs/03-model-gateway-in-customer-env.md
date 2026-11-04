# Chapter 03 — Deploying a Model Gateway Inside the Customer Environment

## Why this chapter exists

Chapter 02 fixed the provider stack. This chapter fixes the code path between the pilot and that stack: a model gateway that sits inside the customer environment, exposes one SDK surface to the pilot code, and routes, budgets, caches, redacts, and audits every call underneath.

Without a gateway, the FDE is left with a direct provider-SDK coupling in every pilot code path, no per-tenant budget enforcement, no provider failover, no centralised audit log, and a per-request PII exposure the customer's AppSec team will flag the moment it reaches review (mod-108). The gateway is the overlay's single most leveraged piece of infrastructure — it is the abstraction that keeps the provider-stack decision from leaking across the codebase.

The in-house applied-LLM craft already knows that gateways exist; `applied-ai-engineer` (level 30) teaches the unified-SDK pattern. The customer-environment overlay here is: the gateway lives *inside the customer's trust boundary*, authenticates on the customer's identity path (mod-103 Chapter 03), egresses on the customer's network path (mod-103 Chapter 07), writes audit logs into the customer's SIEM (mod-110), and enforces budgets per *tenant* — a concept the in-house product may not even have.

## What a model gateway provides

Six concerns, each one of which the pilot needs at some point in a real engagement:

1. **Unified SDK surface.** Pilot code makes one call shape ("OpenAI-compatible chat completions" is the de facto standard) regardless of the underlying provider. Switching from Anthropic direct to Bedrock-Anthropic to self-hosted Llama is a gateway-config change, not a pilot-code rewrite.
2. **Per-tenant budget enforcement.** Soft caps (warn in the response) and hard caps (fail-closed) per tenant, per day, per month. Budget state persists; it does not live in RAM.
3. **Provider failover.** Primary provider down → secondary. Rate-limit storm on primary → shed to secondary. Health signal propagates to the pilot so it can downgrade UX gracefully.
4. **Request / response caching.** Exact-match caching for identical prompts. Optional semantic caching for near-duplicates. Native prompt-caching headers forwarded to Anthropic / Bedrock / OpenAI (Chapter 04 covers the native path).
5. **PII guards.** Pre-send redaction stage (catch obvious patterns before they leave the customer VPC). Post-receive leak-detection stage (catch obvious patterns in completions before they reach the user). Trip logging for later review.
6. **Audit logs.** Per-request structured record: user identity, tenant, provider, model, prompt version, cost, latency, cache-hit flag, guard-trip flag. Lands in the customer's SIEM.

Each concern has both an in-house analogue and a customer-overlay twist. The customer-overlay twist is what this chapter writes.

## The three gateway options

Three families of tools cover the realistic design space. The comparison below is deliberately balanced — each has a place.

### LiteLLM

[docs.litellm.ai](https://docs.litellm.ai). Open-source (Apache 2.0), Python-first, broadest provider coverage. Has two deployment modes:

- **SDK mode** — `import litellm; litellm.completion(...)` from the pilot code. One Python process, no separate gateway service.
- **Proxy mode** — a standalone HTTP service that other services call via OpenAI-compatible endpoints. This is the mode an FDE deploys inside a customer VPC.

Strengths: proxy supports per-tenant budget via its `virtual_keys` + budget tables, Prometheus metrics, OpenTelemetry integration, a management UI, and a long list of providers. Config is a YAML file.

Trade-offs: the proxy's feature surface is wide and still-maturing in places; cutting-edge features change release-to-release. Operate against pinned versions.

When to pick: VPC-install pilots where the customer's AppSec allowlist accepts open-source Python services with an active upstream; most common FDE default.

### Portkey

[portkey.ai](https://portkey.ai) / [docs.portkey.ai](https://docs.portkey.ai). Hybrid SaaS + self-hosted, strong observability UX, commercial plans with enterprise features.

Strengths: polished dashboard, audit log and guardrail features as first-class concerns, self-hosting path for enterprise. Feature parity with LiteLLM on the gateway basics, with a different UX.

Trade-offs: commercial — the customer takes on a vendor relationship for the gateway itself. For some customers that is simpler than operating LiteLLM; for others it is a new sub-processor to add to the DPA.

When to pick: customer wants a polished gateway UX with enterprise support; AppSec is comfortable with an additional vendor; the vendor-managed shape clears the customer's bar.

### OpenRouter

[openrouter.ai](https://openrouter.ai) / [openrouter.ai/docs](https://openrouter.ai/docs). Hosted multi-provider router — one API key, routes to many providers with a unified surface.

Strengths: easiest way to try many providers; useful for the FDE's own prototyping before the pilot lands.

Trade-offs: hosted-only in its default shape; data transits OpenRouter's service. For most customer-environment deployments this disqualifies OpenRouter as the primary gateway — the whole point of the overlay is that the pilot runs inside the customer's trust boundary. OpenRouter solves provider access; it does not solve customer-side deployment.

When to pick: pre-pilot experimentation, or pilots where the customer explicitly accepts OpenRouter's data path in the DPA.

### Other tools worth naming

- **Vendor-native gateways** — the hyperscalers' own routing layers (AWS Bedrock itself is effectively a multi-model gateway for its providers; similarly Vertex AI). These are not full gateways in the LiteLLM sense (no per-tenant budget across providers, no cross-provider failover), but they cover a slice.
- **In-house thin gateway** — a 300-line Python / Go service the FDE writes against three provider SDKs. Appropriate only when the customer's AppSec rejects all off-the-shelf options AND the pilot is small enough to maintain in-house. See the build-vs-buy section.

## Build vs. buy — the explicit customer-side decision

The decision is customer-specific and must be documented. The trade-off:

| Criterion | Buy (adopt OSS) | Buy (commercial) | Build (thin in-house) |
|---|---|---|---|
| Time to first working gateway | Hours | Hours | Days–weeks |
| Customer AppSec review scope | OSS review + operational review | Vendor onboarding (DPA, sub-processor, SIG) | Pilot code review only |
| Audit-log integration | OSS plugin or write-through | Vendor integration (Splunk HEC, Datadog) | Pilot writes to customer SIEM directly |
| PII guard integration | OSS library (Presidio) or vendor feature | Vendor feature | Pilot writes to a library |
| Cross-provider failover | Config | Config | Pilot code |
| Per-tenant budget ledger | OSS feature + customer-DB persistence | Vendor-managed | Pilot writes to customer DB |
| Operational familiarity | FDE team runs it | Vendor runs it (or customer, in self-host) | FDE team runs it, lightly |
| Upgrade burden | Follow upstream releases | Vendor-managed | Rebuild on each feature need |
| Reversibility | High (change config) | Medium (vendor lock) | High (code) |

Decision defaults:

- **Default to buy-OSS** (LiteLLM proxy or equivalent) when the customer's AppSec allowlist accepts active-maintained OSS. This is the most common landing point.
- **Buy-commercial** (Portkey or equivalent) when the customer prefers a vendor-managed story or needs features only the commercial tier ships.
- **Build-thin** only when the customer rejects both AND the gateway surface the pilot needs is small enough (say, one provider, one tenant, no failover). As the pilot grows, build-thin tends to acquire every feature OSS already has — at which point the reason for building has evaporated.

Write the decision as an ADR. Reviewers (customer AppSec, the next FDE, mod-108's security packet) read this choice and the reasoning; "we used LiteLLM because it's open source" is not a reasoning.

The in-house applied-AI engineer makes a related decision inside the vendor product — same options, different weights. The FDE's overlay is: the customer's AppSec and DPA constraints dominate; the decision is per-deployment, not per-product.

## Deploying LiteLLM inside a customer VPC — reference shape

A concrete reference deployment for a VPC-install pilot. Adapt to the specific customer; the shape is the point.

### Topology

```
Customer VPC
   ┌─────────────────────────────────────────────────────────────────┐
   │                                                                 │
   │  Pilot subnet (private)                                         │
   │  ┌───────────────┐                                              │
   │  │ pilot-app     │ --auth (OIDC bearer, mod-103 Ch 03)----┐     │
   │  └───────────────┘                                        │     │
   │                                                           v     │
   │  Gateway subnet (private)                   ┌─────────────────┐ │
   │  ┌───────────────┐                          │ gateway (LiteLLM│ │
   │  │ gateway       │<-- OTel traces ----------│   proxy)        │ │
   │  │  (LiteLLM)    │-- budget/audit log ----->│                 │ │
   │  └───────┬───────┘                          └────────┬────────┘ │
   │          │                                           │          │
   │          │ egress via customer's egress proxy        │          │
   │          │ (mod-103 Ch 07)                           │          │
   │          v                                           │          │
   │  ┌───────────────┐                                   │          │
   │  │ egress-proxy  │                                   │          │
   │  └───────┬───────┘   ┌───────────────┐   ┌───────────┴────────┐ │
   │          │           │ Postgres /    │   │ customer SIEM feed │ │
   │          │           │ DynamoDB      │<--│ (audit log push)   │ │
   │          │           │ (budget + log │   └────────────────────┘ │
   │          │           │   ledger)     │                          │
   │          │           └───────────────┘                          │
   │          v                                                      │
   │  ───────────────────── VPC endpoint / PrivateLink ───────────── │
   │                                 │                               │
   └─────────────────────────────────┼───────────────────────────────┘
                                     v
                             Provider API endpoint
                             (Bedrock / OpenAI /
                              Anthropic / etc.)
```

Key properties:

- The gateway runs in its own subnet, reachable only from pilot subnets. No public IP.
- Auth from pilot → gateway is the pilot's existing OIDC bearer path from mod-103 Chapter 03. The gateway validates the bearer, extracts tenant ID from a claim, and proceeds.
- Egress to the provider API goes through the customer's egress proxy (mod-103 Chapter 07). For hyperscaler providers inside the same cloud, egress may be via PrivateLink / Private Service Connect / Private Endpoint — zero public internet.
- Budget ledger and audit log live in the customer's database (Postgres is the common default; customer SIEM may add duplicate storage).
- Observability flows via OpenTelemetry to the customer's collector; traces land in the customer's observability stack (mod-110).

### Configuration

LiteLLM proxy config is YAML. The FDE pins a specific version and writes a config with named model groups, provider routing, budget rules, and guard integrations. The exact config shape is documented in the [LiteLLM proxy docs](https://docs.litellm.ai/docs/proxy/configs); the FDE reads that doc at the time of deployment rather than templating from memory, because the schema evolves.

What the config captures in words:

- **Model aliases** — `customer-fast`, `customer-default`, `customer-reasoning` — mapping pilot-visible names to provider-specific model IDs. The pilot code never names a provider directly.
- **Routing policy** — primary and fallback per alias. Fallback triggers on timeout, 5xx, or rate-limit.
- **Virtual keys** — one per tenant (or one per environment × tenant for staging/prod separation). Each virtual key carries its budget cap, allowed models, and metadata (tenant_id).
- **Guards / hooks** — pre-call and post-call hooks for PII redaction and leak detection.
- **Caching** — cache store (Redis in-VPC is the common default), TTL, cache-key policy.

### Secrets

The provider credentials and the LiteLLM admin key live in the customer's vault — HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager, depending on the customer's cloud-of-record. Rotation follows the customer's rotation policy (common: 90-day rotation; the FDE honours the customer's cadence). Never commit secrets to the gateway image or config file.

## Per-tenant budget enforcement

Tenant identification runs on one of three mechanisms:

- **JWT claim** — the OIDC bearer the pilot sends carries a `tenant_id` claim (set at mod-103 Chapter 03). The gateway parses the claim.
- **Virtual key** — each tenant gets a distinct LiteLLM virtual key; the key implicitly identifies the tenant.
- **Request metadata** — a header (`X-Tenant-ID`) set by the pilot. Simplest, but requires trust that the pilot sets it correctly.

For a multi-tenant SaaS deployment pattern, JWT-claim or virtual-key are the right mechanisms; a header-only mechanism makes tenant spoofing trivial.

Budget semantics:

- **Soft cap** — warn in the response (metadata field) and emit a metric. Pilot UX can show a budget-warning banner. Not fail-closed.
- **Hard cap** — reject the request with a 429 or 402. Pilot UX degrades: switch to cached-only responses, degrade to a cheaper model, or surface a cooldown.
- **Rate-limit vs. budget-limit** — rate-limit is requests-per-second (protection against runaway code); budget-limit is dollars-per-day (protection against cost blowout). Both exist; they are different metrics.
- **Overage behaviour** — fail-closed is the default for untrusted spend; degrade-gracefully is the right choice for user-visible workflows where blocking the user is worse than overrunning. Pick per workflow, document in the pilot-scope.

The budget ledger is a durable table — not a Redis counter alone, because a cache flush must not grant free budget. Redis can carry the hot counter with a write-through to Postgres (or equivalent) every N seconds.

## Failover and caching

### Failover

Each model alias declares a primary and one or more fallbacks. Fallback triggers:

- Timeout at the request level (gateway-configured; typically a few seconds beyond TTFT target).
- 5xx from the provider.
- Rate-limit (429) from the provider.
- Health signal — if N consecutive failures over window W, open the circuit for cooldown C.

Important discipline: **the pilot code sees the failover**. The gateway returns a response header (`X-Model-Used`, `X-Provider-Used`) so the pilot's observability captures which model actually answered. A silent failover hides regressions.

### Caching

Three layers, each with different scope:

- **Exact-match cache** — SHA-256 of the full prompt. Hit → return cached completion. Appropriate for deterministic workloads (temperature=0, no time-sensitive context). TTL conservative (minutes to hours).
- **Semantic cache** — embedding of the prompt compared against stored embeddings; threshold hit → return cached completion. Appropriate only when the task's semantics tolerate close-but-not-exact matches. Dangerous for tasks where small wording differences matter (contracts, medical).
- **Provider-native prompt caching** — Anthropic / OpenAI / Bedrock all expose prompt-caching headers that let the *provider* cache the static prefix of the prompt on their side. This is additive to the gateway's cache, not a replacement. Chapter 04 covers the native path in depth.

Cache keys *must* include tenant ID. A cross-tenant cache hit is a data-leak incident.

## PII guards

The gateway is the chokepoint where every outbound prompt passes. Two stages:

### Pre-send redaction

The outbound prompt is scanned for the customer's defined PII patterns before it leaves the VPC. On a match:

- **Redact** — replace with a token, forward the redacted prompt to the provider. Appropriate for most cases.
- **Reject** — fail-closed with a specific error the pilot can show the user. Appropriate when the prompt contains a pattern that must not appear in the task at all.
- **Alert-only** — log the trip but forward anyway. Appropriate during policy tuning, never in steady state.

Reference tooling: [Microsoft Presidio](https://microsoft.github.io/presidio/) is a common OSS PII-detection framework. The customer may have their own DLP solution (Symantec, Microsoft Purview, custom) — the gateway should integrate with the customer's DLP rather than ship a parallel policy.

### Post-receive leak detection

The completion is scanned before being returned to the pilot. On a match, either redact or reject. Leak detection is necessary because:

- The model may have repeated back PII from the input that redaction missed (edge-case failure of pre-send).
- The model may have hallucinated a pattern that looks like PII (number sequences that match credit-card formats; names that match a known individual).

### Logging discipline

Audit-log entries must not contain the raw PII that was redacted. Log the hash, the pattern that matched, and the position; never the raw match. Mod-108 revisits this in the compliance context.

## Audit log shape

Every gateway call produces an audit record. Fields:

| Field | Source |
|---|---|
| `request_id` | Gateway-generated UUID |
| `trace_id` | Propagated from pilot's OpenTelemetry trace (mod-110) |
| `timestamp` | UTC ISO-8601 |
| `tenant_id` | From JWT claim / virtual key |
| `user_id` | From JWT `sub` claim (mod-103 Chapter 02) |
| `user_email_hash` | Hashed email if the audit log must support "who" queries without raw email |
| `provider` | e.g. `aws-bedrock`, `openai`, `anthropic`, `self-hosted-vllm` |
| `model` | Provider-specific model ID actually used (after failover) |
| `model_alias` | Gateway alias the pilot called (e.g. `customer-default`) |
| `prompt_version` | Hash of the prompt template version (Chapter 04) |
| `input_tokens` | Provider response or tokeniser count |
| `output_tokens` | Provider response or tokeniser count |
| `cached_input_tokens` | Provider-native cache-hit input tokens (Chapter 04) if available |
| `cost_usd` | Computed from provider pricing × tokens |
| `latency_ttft_ms` | Gateway-measured TTFT (Chapter 05) |
| `latency_total_ms` | Gateway-measured completion latency |
| `cache_hit` | `exact` / `semantic` / `provider` / `none` |
| `failover_used` | boolean |
| `pii_pre_trip` | boolean + pattern name (not raw match) |
| `pii_post_trip` | boolean + pattern name (not raw match) |
| `status` | `success` / `rate_limited` / `budget_exceeded` / `guard_rejected` / `error` |
| `error_code` | if `status != success` |

The audit log lands in two places:

1. **The gateway's own operational store** (Postgres + S3 archive, or equivalent). Used for immediate operational debugging.
2. **The customer's SIEM** (Splunk, Sentinel, Chronicle, Datadog, Elastic). Pushed via the customer's preferred mechanism — Splunk HEC, Kafka topic, S3 bucket for log ingestion. The customer's AppSec team subscribes to this; mod-110 covers the full observability story.

The audit log is a mod-108 security-review artifact. A clean schema here saves days of back-and-forth with the customer's AppSec team later.

## Anti-patterns

**No gateway "until we need one."** Needing a gateway appears suddenly — first production incident, first per-tenant budget ask, first PII near-miss, first security review question. Each one adds weeks if the gateway is retrofitted. Ship the gateway in week 2 even if only one of the six concerns is active.

**Rolling a thin gateway because "LiteLLM is overkill."** LiteLLM proxy may feel heavy at week 2, but by week 10 the thin gateway has accumulated per-tenant budget tables, a cache layer, a guard pipeline, and a Prometheus exporter — and it is less tested than LiteLLM upstream. Default to adopting.

**Per-tenant isolation via a shared Python dict.** The budget-enforcement code with tenant counters in `MEMORY[tenant_id]` will leak across tenants on process restart, lose state on Redis flush, and race under concurrent writes. Use a persisted ledger with proper transactional semantics.

**Skipping the audit log until security review asks.** The audit log must capture week-1 traffic for it to help answer "when did this start?" questions in week 8. Retrofitting is possible but partial.

**Caching across tenants.** Cache key omits tenant_id → a tenant hits another tenant's cached completion → data-leak incident. Cache keys must always compose tenant_id with the prompt hash.

**Trusting an unsigned `X-Tenant-ID` header.** The pilot code sets it, the gateway trusts it — but any client that can reach the gateway can forge it. Prefer JWT-claim or virtual-key for tenant identification.

## Summary

- A model gateway is the overlay's most leveraged piece of infrastructure: it provides a unified SDK surface, per-tenant budget enforcement, provider failover, caching, PII guards, and a per-request audit log.
- Three families cover the design space — LiteLLM (OSS, FDE default), Portkey (commercial, polished UX), OpenRouter (hosted router, usually unsuitable for in-VPC pilots) — with a thin in-house build as a fallback.
- The build-vs-buy decision is customer-specific; document it as an ADR with the customer-side criteria (AppSec allowlist, DPA scope, operational familiarity) explicit.
- A reference VPC deployment: gateway in a private subnet, auth from pilot via OIDC bearer (mod-103 Ch 03), egress via customer's egress proxy or PrivateLink (mod-103 Ch 07), budget ledger and audit log in the customer's database, OpenTelemetry out to the customer's SIEM.
- Per-tenant budget runs on JWT claims or virtual keys, with soft/hard caps, a durable ledger, and per-workflow overage behaviour.
- Caching composes three layers: exact-match (gateway), semantic (gateway, optional), provider-native (Chapter 04). Cache keys always include tenant ID.
- PII guards are pre-send redaction + post-receive leak detection, integrated with the customer's DLP rather than ship parallel policy. Audit logs never contain raw matched PII.
- The audit log is a mod-108 artifact — ship a clean schema week 2, pushed to the customer's SIEM in their preferred format.
- Anti-patterns: deferring the gateway, rolling a thin build without reason, in-RAM tenant state, header-only tenant trust, cross-tenant cache keys.
- Chapter 04 is next: the prompt architecture that runs behind the gateway — structured outputs, streaming, prompt caching, versioning, A/B, per-tenant overrides.
