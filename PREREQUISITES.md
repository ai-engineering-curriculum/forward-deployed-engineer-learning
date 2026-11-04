# Prerequisites — Forward Deployed AI Engineer track

**Role level:** 35 (hands-on specialist IC, AI Engineering family)
**Track:** `forward-deployed-engineer-learning`

This track is a hands-on customer-embedded delivery specialty. It assumes the learner arrives with applied-AI engineering craft already fluent, evaluation-methodology fundamentals in place, and strong engineering-generalist muscle. Fundamentals are **not** re-taught here — the modules step directly onto customer discovery, enterprise integration, compliance review, and the field→product loop.

## Assumed lower-level curriculum

Complete or be practically fluent in the equivalent of these lower-level tracks before starting:

- **[`ai-infra-junior-engineer-learning`](https://github.com/ai-infra-curriculum/ai-infra-junior-engineer-learning) — level 10** — engineering-craft prerequisites (Linux, Git, Python packaging, HTTP APIs, Docker, unit testing, logging, CI basics). If you cannot ship a small Python service and instrument it, start here.
- **[`ml-engineer-learning`](https://github.com/ml-engineering-curriculum/ml-engineer-learning) — level 20** — classical ML with scikit-learn, deep learning with PyTorch and Hugging Face Transformers, packaging with FastAPI + Docker, experiment tracking with MLflow / W&B. If you cannot fine-tune a classifier and defend its eval, start here.
- **[`llm-application-developer-learning`](https://github.com/ai-engineering-curriculum/llm-application-developer-learning) — level 25** — LLM-application fundamentals: prompts, structured outputs, streaming, tool-calling basics, provider SDKs (OpenAI / Anthropic / Google), the OpenAI and Anthropic cookbooks worked through end-to-end.
- **[`rag-engineer-learning`](https://github.com/ml-engineering-curriculum/rag-engineer-learning) — level 25** — retrieval-engine depth: chunking, embeddings, hybrid search, rerankers, retrieval-level evaluation. This track consumes the retriever and layers customer-auth / PII / access scoping on top.
- **[`applied-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) — level 30** — in-house applied-AI feature engineering end-to-end: production prompting with caching / streaming / versioning, feature-side evaluation loops, guardrails on the surface, provider gateway routing, per-feature cost / latency SLOs, feature flags + canary, observability + feedback + incident response. This track **extends** that craft into customer-embedded delivery.

Practical familiarity (not full completion) with the following peer tracks is also useful — the FDE collaborates with their owners:

- **[`agentic-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/agentic-ai-engineer-learning) — level 30** — agent-framework depth (LangGraph, OpenAI Agents SDK, Claude Agent SDK, MCP).
- **[`ai-eval-engineer-learning`](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning) — level 30** — evaluation methodology at the application layer.
- **[`ai-risk-engineer-learning`](https://github.com/ai-governance-curriculum/ai-risk-engineer-learning) — level 25** — risk-engineering craft (harm models, red-team, adversarial-ML, fairness, privacy, guardrails).

## Assumed working knowledge

Even without holding the equivalent role formally, you should be practically fluent in:

- **Python at production depth** — packaging (pyproject.toml, uv / poetry, virtualenvs), async/await, FastAPI, pytest + hypothesis, typed code (mypy / pyright), CI (GitHub Actions). Comfortable debugging a stack trace across libraries you have not seen before.
- **TypeScript at functional depth** — enough to read and modify a Next.js or Express front-of-house for a pilot, call LLM SDKs, and ship a small customer-facing admin page. Full-depth FE frameworks (React / Vue hooks, SSR, state) are not required but useful.
- **SQL fluency + data wrangling** — joins, window functions, CTEs, explain plans, performance intuition; pandas / polars for Python-side data handling; a working feel for columnar formats (Parquet / Arrow) and warehouse semantics (Snowflake / BigQuery / Databricks).
- **LLM provider SDKs** — comfortable calling OpenAI / Anthropic / Google / Cohere / self-hosted (vLLM / TGI / SGLang) from Python and TypeScript; comfortable reading trace output from Langfuse / LangSmith / Arize Phoenix / Weave / Braintrust at the applied level.
- **HTTP + API design** — REST / JSON APIs, OpenAPI 3.1 shape, pagination, idempotency, retries, timeouts, circuit breakers; GraphQL basics; webhook design.
- **Authentication and authorisation** — HTTPS / TLS, bearer tokens, OAuth 2.0 (authorisation-code with PKCE, client-credentials, device-code flows), JWT; basic RBAC/ABAC intuition.
- **Statistics & probability** — reading a confusion matrix, calibration curves, ROC/PR, bootstrap confidence intervals, sample-size intuition, inter-rater reliability. Deep methodology-of-methodology is linked out to `model-evaluation-engineer` and `ai-eval-engineer` peers.
- **Linux / Docker / Kubernetes basics** — Docker build + run, Compose, Helm chart readability, kubectl basics; enough to deploy inside a customer cluster when their SRE lets you.
- **Terraform basics** — reading and modifying modules, provider configuration, state hygiene; enough to ship a VPC-install kit.
- **Git + GitHub workflow at depth** — PRs, reviews, branch protection, CI configuration; comfortable shepherding a feature through someone else's review process.
- **Security literacy** — OWASP Top 10 for web, HTTPS / TLS fundamentals, secrets handling, SSRF / IDOR / CSRF at awareness level; enough to not drop a hole in the customer's attack surface.
- **Reading the primary literature + practitioner essays** — comfortable opening Chip Huyen's AI Engineering, Eugene Yan's patterns writeups, Hamel Husain's evals essays, Anthropic's 'Building effective agents', and the OpenAI / Anthropic cookbooks, and translating the method into working code within the same day.

## Not assumed, taught here

You do **not** need prior experience with:

- The Forward Deployed Engineer role itself — mod-101 level-sets the role and its positioning against adjacent customer-facing roles.
- Customer discovery discipline — mod-102 covers Teresa Torres continuous-discovery, Jeff Patton story mapping, and the pilot-scope document shape end-to-end.
- Enterprise identity + integration — mod-103 covers SAML / OIDC / SCIM against the big-4 IdPs, service auth with OAuth2 / mTLS / token exchange, OpenAPI / GraphQL / webhooks / CDC / event buses / CloudEvents, SaaS-platform connectors (Salesforce / ServiceNow / SAP / Workday / Snowflake / Databricks), legacy surfaces (SOAP / SFTP / mainframe), customer-side networking (PrivateLink / VPC / VPN), and MCP server implementation.
- Model-gateway stack — mod-104 covers LiteLLM / Portkey / OpenRouter deployment inside a customer environment.
- Hyperscaler managed-AI platforms — mod-104 covers AWS Bedrock, Google Vertex AI, Azure AI Foundry, Snowflake Cortex, Databricks Mosaic at the integration level.
- ACL-propagated retrieval — mod-105 covers row-level security + role-based filtering + PII-safe retrieval end-to-end.
- Enterprise-agent hardening — mod-106 covers OWASP Agentic Top 10 + MITRE ATLAS mapping, human-in-the-loop checkpoint model, tool-permission mapping.
- SOC 2 / HIPAA / FedRAMP / EU AI Act review process from the engineering side — mod-108 covers security-review packet authoring end-to-end.
- Feature-flag strategy for LLM features — mod-109 covers per-tenant / per-prompt / per-model rollout with LaunchDarkly / Statsig / Unleash.
- OpenTelemetry GenAI semantic conventions and LLM-observability platform integration — mod-110 covers this at the customer compliance posture.
- Field→product feedback loop discipline — mod-111 covers design-partner artifact production and platform-request packaging.
- Consulting-craft — mod-112 covers demo engineering, exec briefs, pilot narration, resistance handling, and knowledge transfer.

## Recommended reading before starting

- [Palantir — A day in the life of a Forward Deployed Software Engineer](https://blog.palantir.com/a-day-in-the-life-of-a-forward-deployed-software-engineer-b2142f1e36c8) — the role's origin story. Read the full tagged collection.
- [Chip Huyen — AI Engineering (O'Reilly 2024)](https://www.oreilly.com/library/view/ai-engineering/9781098166298/) — front-to-back once.
- [Anthropic — Building effective agents](https://www.anthropic.com/research/building-effective-agents) — the pattern catalogue.
- [Eugene Yan — Patterns for Building LLM-based Systems & Products](https://eugeneyan.com/writing/llm-patterns/).
- [Hamel Husain — Your AI Product Needs Evals](https://hamel.dev/blog/posts/evals/).
- [OWASP Top 10 for LLM Applications (2025)](https://owasp.org/www-project-top-10-for-large-language-model-applications/) + [OWASP Agentic Security Initiative Top 10](https://genai.owasp.org/initiative/agentic-security-initiative/).
- [NIST AI Risk Management Framework 1.0](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.100-1.pdf) — front-to-back once.
- [Model Context Protocol specification](https://modelcontextprotocol.io/) — read the spec and one reference server.
- [Geoffrey Moore — Crossing the Chasm (3rd ed.)](https://www.harpercollins.com/products/crossing-the-chasm-3rd-edition-geoffrey-a-moore) + [Teresa Torres — Continuous Discovery Habits](https://www.producttalk.org/continuous-discovery-habits/).
- A recent public AI system card or deployment case study — read it as an FDE would, tracing each claim to an evidence artifact and asking 'what would I ship to the customer here'.

## Not required, useful to have

- **Prior work with an eval-tooling platform** (Promptfoo, RAGAS, DeepEval, Inspect, Phoenix, LangSmith, Weave, Braintrust) — the peer `ai-eval-engineer-learning` (level 30) owns this stack; a working feel is useful before mod-107.
- **Prior work with an LLM-observability platform** (Langfuse, LangSmith, Phoenix, Weave, Braintrust, Datadog LLM Observability) — useful before mod-110.
- **Prior work with a feature-flag platform** (LaunchDarkly, Statsig, Unleash, Flagsmith) — useful before mod-109.
- **Prior work inside a hyperscaler managed-AI platform** (Bedrock / Vertex / Foundry) — useful before mod-104.
- **Prior customer-facing experience** in any shape (solutions engineering, implementation consulting, developer advocacy, technical product management, customer success engineering) — the consulting-craft modules (mod-102 + mod-111 + mod-112) land easier if you have shipped a customer meeting before.
- **Compliance familiarity** (SOC 2 Type II onboarding, HIPAA / BAA negotiations, FedRAMP sponsor, EU AI Act / GDPR DPO contact) — useful before mod-108.

## Not in scope for this track (linked out)

- **In-house applied-AI feature engineering craft** (prompts, structured outputs, caching, streaming, cost/latency SLOs at the in-house level) — owned by [`applied-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) (level 30, AI Engineering family). This track adds the customer-environment overlay, not the full craft.
- **Deep agent-framework internals** (LangGraph state machines, OpenAI Agents SDK internals, Claude Agent SDK internals, MCP transport) — owned by [`agentic-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/agentic-ai-engineer-learning) (level 30, AI Engineering family).
- **Application-layer evaluation methodology depth** (traces, judges, RAG eval, eval-gated CI/CD, online eval, cost/latency/quality) — owned by [`ai-eval-engineer-learning`](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning) (level 30, AI Engineering family).
- **Retrieval-engine depth** (chunking strategies, embedding-model trade-offs, rerankers, multi-index fusion, retrieval-level evaluation) — owned by [`rag-engineer-learning`](https://github.com/ml-engineering-curriculum/rag-engineer-learning) (level 25, ML Engineering family).
- **Fine-tuning / RLHF / RLAIF depth** — owned by [`fine-tuning-engineer-learning`](https://github.com/ml-engineering-curriculum/fine-tuning-engineer-learning) (level 30, ML Engineering family) and [`training-pipeline-engineer-learning`](https://github.com/ml-engineering-curriculum/training-pipeline-engineer-learning) (level 35, ML Engineering family).
- **Inference serving depth** (vLLM / TGI / SGLang internals, batching, quantisation, KV-cache, autoscaling) — owned by [`ai-infra-inference-engineer-learning`](https://github.com/ai-infra-curriculum/ai-infra-inference-engineer-learning) (level 35, Infra family).
- **Deep ML/AI security at platform scale** — owned by [`ai-infra-security-learning`](https://github.com/ai-infra-curriculum/ai-infra-security-learning) / [`security-learning`](https://github.com/ai-infra-curriculum/security-learning) (level 35, Infra family).
- **Release-assurance / audit-trail / regulator-facing methodology depth** — owned by [`ai-evaluation-engineer-learning`](https://github.com/ai-governance-curriculum/ai-evaluation-engineer-learning) (level 35, Governance family).
- **Risk-engineering craft** (harm models, red-team, adversarial-ML, fairness, privacy, guardrail engineering, risk register) — owned by [`ai-risk-engineer-learning`](https://github.com/ai-governance-curriculum/ai-risk-engineer-learning) (level 25, Governance family).
- **Frontier-agent red-team methodology and dangerous-capability evaluation depth** — owned by [`agentic-safety-engineer-learning`](https://github.com/ai-governance-curriculum/agentic-safety-engineer-learning) (level 40, Governance family).
- **Build leadership, shared-infra authorship, engineering management scope** — owned by [`senior-agentic-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/senior-agentic-ai-engineer-learning) (level 40) and [`ai-engineering-manager-learning`](https://github.com/ai-engineering-curriculum/ai-engineering-manager-learning) (level 40).
- **Multi-agent architecture** — owned by [`agentic-systems-architect-learning`](https://github.com/ai-engineering-curriculum/agentic-systems-architect-learning) (level 48).
- **Legal opinion** — out of scope; customer contract review informs requirements, but the FDE does not deliver legal opinion. Customer legal counsel remains the authority on their side.
- **Deal-level sales** — out of scope; the FDE is adjacent to AE / SE / pre-sales motion but does not own quota.
- **Full-time professional services delivery** — out of scope; the FDE is design-partner engineering, not billable consultancy.
- **Customer-side organisational change management at scale** — coordinated with the customer's own operators and with `ai-enablement-lead-learning` (peer, level 35) internally.
