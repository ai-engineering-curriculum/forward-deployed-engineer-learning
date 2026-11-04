# AI Engineering · Forward Deployed AI Engineer — Learning Repository

<!-- aicg:site-banner -->
> 🎓 Part of the free, open-source **AI Career Curriculum** ecosystem — [Infrastructure](https://github.com/ai-infra-curriculum) · [ML Engineering](https://github.com/ml-engineering-curriculum) · [AI Engineering](https://github.com/ai-engineering-curriculum) · [Governance](https://github.com/ai-governance-curriculum) · [Startup](https://github.com/ai-startup-curriculum) · [Workforce](https://github.com/ai-workforce-curriculum).
<!-- /aicg:site-banner -->

<!-- aicg:sponsor -->
> 💜 **[Sponsor this curriculum](https://github.com/sponsors/ai-engineering-curriculum)** — sponsorships keep the whole open-source AI Career Curriculum free and moving.
<!-- /aicg:sponsor -->

**Role level:** 35 (hands-on specialist IC — AI Engineering family)

Embed with customers to take AI systems from demo to production inside their environment, data, and workflows. Discovery → integration into messy enterprise stacks → evals on customer data → deployment → observability and shared on-call → feeding field lessons back to product. The role Palantir popularised, and that OpenAI, Anthropic, Scale AI, Sierra, Decagon, Hebbia, Writer, Harvey, Glean, Cresta, Cursor/Anysphere, Replit, Vercel, Snowflake, Databricks, and the hyperscaler customer-engineering orgs now hire for under the same or closely-equivalent titles.

> **Status**: planned. This bootstrap packet lands the requirements catalog and the module / project plan authored from [`.aicg/curriculum-plan.json`](.aicg/curriculum-plan.json). Lessons and projects will be drafted by subsequent autonomous content cycles. Live posting evidence was deferred (WebSearch / WebFetch was not granted in the bootstrap session) — see [`JOB_REQUIREMENTS.md`](JOB_REQUIREMENTS.md) `Status`.

## What you get

Planned commitment: **170 hours across 12 modules** + **108 hours across 4 projects** = **~278 hours**.

- **Module 101** — Forward Deployed Engineer discipline and position on the ladder. Palantir-origin + frontier-lab + applied-AI-product-company instantiations; scope against applied-ai-engineer / ai-eval-engineer / ai-infra-inference-engineer / ai-evaluation-engineer / senior-agentic-ai-engineer / ai-engineering-manager; relationship to adjacent customer-facing roles.
- **Module 102** — customer discovery and pilot scoping. SME-paired JTBD interviewing, workflow process-mining, pilot-scope document with success criteria and exit-to-production gates.
- **Module 103** — enterprise integration engineering. SSO (SAML/OIDC/SCIM), service auth, data integration (REST/GraphQL/webhooks/CDC/event buses), SaaS platforms (Salesforce/ServiceNow/SAP/Workday/Snowflake/Databricks), legacy surfaces, networking (PrivateLink/VPC/VPN), MCP.
- **Module 104** — LLM application engineering in customer environments. Provider stack selection, model gateways (LiteLLM/Portkey/OpenRouter), per-tenant model routing, cost/latency SLOs, when-to-fine-tune decision.
- **Module 105** — grounding on customer data. Retrieval engine selection, ACL propagation into the retriever, PII-safe retrieval, citation/freshness/evidence UX, customer-data gold-set bootstrap.
- **Module 106** — agentic workflows on enterprise surfaces. Anthropic's effective-agents pattern catalogue, Claude Agent SDK / OpenAI Agents SDK / LangGraph, human-in-the-loop checkpoints, tool-permission mapping, OWASP Agentic Top 10 hardening.
- **Module 107** — eval engineering on customer data. SME-paired gold sets, business-KPI-linked evals, Promptfoo + RAGAS + DeepEval + Inspect, eval-gated CI, online eval with judge-as-LLM, pilot-to-production exit eval.
- **Module 108** — enterprise security, privacy, compliance. SOC 2 Type II, HIPAA/BAA, FedRAMP Moderate/High, GDPR, OWASP LLM Top 10 + Agentic Top 10, MITRE ATLAS, NIST AI RMF, ISO 42001, zero-retention/BYOK/data-residency, AI supply-chain provenance (ML-BOM/SPDX AI/Sigstore/SLSA/ModelScan).
- **Module 109** — deployment, rollout, change management. VPC install, feature flags (LaunchDarkly/Statsig/Unleash), canary/shadow/kill-switch, per-tenant versioning with replay, SME training + admin dashboards.
- **Module 110** — observability and on-call for shared customer systems. OpenTelemetry + GenAI semantic conventions, LLM-observability dashboards at customer compliance posture, LLM-specific SLOs, joint incident command + blameless post-mortems with permanent regression-fixture back-feed.
- **Module 111** — field→product feedback loop. Design-partner artifacts, pattern writeups, platform-request packaging, roadmap-influence practice, FDE-team knowledge-base authorship.
- **Module 112** — consulting craft. Exec demos, exec briefs, pilot narration, resistance handling, scope discipline, knowledge transfer to customer engineering.
- **Four projects** — end-to-end pilot for one customer workflow; enterprise integration + compliance kit for one target vertical; eval + observability bundle on customer-shape data; field→product playbook & executive debrief packet.

## Level ladder — where this track fits

- **level 10** — [`ai-infra-junior-engineer-learning`](https://github.com/ai-infra-curriculum/ai-infra-junior-engineer-learning) (engineering-craft prerequisites)
- **level 20** — [`ml-engineer-learning`](https://github.com/ml-engineering-curriculum/ml-engineer-learning) (classical ML fundamentals)
- **level 25** — [`llm-application-developer-learning`](https://github.com/ai-engineering-curriculum/llm-application-developer-learning) (LLM app fundamentals), [`rag-engineer-learning`](https://github.com/ml-engineering-curriculum/rag-engineer-learning) (retrieval depth), [`ai-risk-engineer-learning`](https://github.com/ai-governance-curriculum/ai-risk-engineer-learning) (cross-family peer-down)
- **level 30** — [`applied-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) (direct predecessor — in-house applied-AI feature engineering), [`agentic-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/agentic-ai-engineer-learning) (agent-framework depth), [`ai-eval-engineer-learning`](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning) (evaluation methodology)
- **level 35** — **`forward-deployed-engineer-learning`** (this track — customer-embedded delivery) · peers: [`ai-infra-inference-engineer-learning`](https://github.com/ai-infra-curriculum/ai-infra-inference-engineer-learning), [`ai-infra-security-learning`](https://github.com/ai-infra-curriculum/ai-infra-security-learning) / [`security-learning`](https://github.com/ai-infra-curriculum/security-learning), [`ai-evaluation-engineer-learning`](https://github.com/ai-governance-curriculum/ai-evaluation-engineer-learning), [`training-pipeline-engineer-learning`](https://github.com/ml-engineering-curriculum/training-pipeline-engineer-learning), [`voice-agent-engineer-learning`](https://github.com/ai-engineering-curriculum/voice-agent-engineer-learning), [`ai-enablement-lead-learning`](https://github.com/ai-workforce-curriculum/ai-enablement-lead-learning)
- **level 40** — [`senior-agentic-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/senior-agentic-ai-engineer-learning) (build leadership), [`ai-engineering-manager-learning`](https://github.com/ai-engineering-curriculum/ai-engineering-manager-learning) (engineering management), [`agentic-safety-engineer-learning`](https://github.com/ai-governance-curriculum/agentic-safety-engineer-learning) (frontier-agent safety)
- **level 48** — [`agentic-systems-architect-learning`](https://github.com/ai-engineering-curriculum/agentic-systems-architect-learning) (multi-agent architecture), [`principal-ml-engineer-learning`](https://github.com/ml-engineering-curriculum/principal-ml-engineer-learning) (principal IC)
- **level 70** — [`chief-ai-officer-learning`](https://github.com/ai-governance-curriculum/chief-ai-officer-learning) (executive)

See [`CURRICULUM.md`](CURRICULUM.md) for the ownership rule this track applies and [`PREREQUISITES.md`](PREREQUISITES.md) for the assumed lower-level curriculum.

## Layout

```
forward-deployed-engineer-learning/
├── lessons/mod-XXX-*/        modules with lectures, exercises, labs, quizzes
├── projects/project-XXX-*/   multi-module capstones
├── CURRICULUM.md             role-level coverage map
├── PREREQUISITES.md          assumed entry skills
├── VERSIONS.md               release history
├── JOB_REQUIREMENTS.md       requirements catalog with cited authoritative references
├── .aicg/job-requirements.json    machine-readable requirements & references
├── .aicg/curriculum-plan.json     machine-readable module / exercise / project plan
└── README.md                 this file
```

## Paired Solutions Repo

[`forward-deployed-engineer-solutions`](https://github.com/ai-engineering-curriculum/forward-deployed-engineer-solutions) carries the reference implementations.

---

<!-- aicg:maintained-by -->
Maintained by [VeriSwarm.ai](https://veriswarm.ai)
