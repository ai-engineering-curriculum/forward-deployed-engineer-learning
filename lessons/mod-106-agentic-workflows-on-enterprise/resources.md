# Resources for mod-106-agentic-workflows-on-enterprise (Agentic Workflows on Enterprise Surfaces)

Primary-source references for the chapters and exercises. Vendor docs change often; verify the specific page is still current before citing to a customer.

## Pattern catalogue and agent design philosophy

- Anthropic, [*Building effective agents*](https://www.anthropic.com/engineering/building-effective-agents) — the pattern catalogue Chapter 02 uses (prompt chaining, routing, parallelisation, orchestrator-workers, evaluator-optimiser, autonomous agent) and the deterministic-code reflex.
- Anthropic, [*How we built our multi-agent research system*](https://www.anthropic.com/engineering/built-multi-agent-research-system) — a case study in orchestrator-workers at scale, with evaluation notes.
- OpenAI, [*A practical guide to building agents*](https://openai.github.io/openai-agents-python/) — the OpenAI Agents SDK's documentation includes design-level framing alongside the API.

## Framework and SDK documentation

- Anthropic, [Claude Agent SDK overview](https://docs.claude.com/en/api/agent-sdk/overview) and [Claude tool use overview](https://docs.claude.com/en/docs/agents-and-tools/tool-use/overview).
- OpenAI, [Agents SDK documentation](https://openai.github.io/openai-agents-python/) and [function calling guide](https://platform.openai.com/docs/guides/function-calling).
- LangChain, [LangGraph documentation](https://langchain-ai.github.io/langgraph/) — graph-based agent orchestration with checkpointing and interrupts.
- Microsoft, [Semantic Kernel documentation](https://learn.microsoft.com/en-us/semantic-kernel/) — .NET-first agent framework.
- Model Context Protocol, [specification and host/server docs](https://modelcontextprotocol.io/) — the protocol for exposing tools from an in-boundary server to an agent host.

## Prompt-engineering fundamentals for agents

- Anthropic, [prompt engineering overview](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview) — delimiter discipline, structured outputs, chain-of-thought.
- Anthropic, [Extended thinking guidance](https://docs.claude.com/en/docs/build-with-claude/extended-thinking) — using reasoning tokens in agent loops.
- OpenAI, [structured outputs guide](https://platform.openai.com/docs/guides/structured-outputs).

## Identity, permissions, and audit

- IETF, [RFC 8693 — OAuth 2.0 Token Exchange](https://datatracker.ietf.org/doc/html/rfc8693).
- IETF, [RFC 6749 — The OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/html/rfc6749) (incremental authorization pattern referenced in Chapter 05).
- Microsoft, [On-Behalf-Of flow (Microsoft identity platform)](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-on-behalf-of-flow).
- Okta, [set up token exchange](https://developer.okta.com/docs/guides/set-up-token-exchange/main/).
- Google Cloud, [Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation) and [Service account impersonation](https://cloud.google.com/iam/docs/service-account-impersonation).
- AWS, [STS AssumeRole](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html) and [IAM Roles for Service Accounts in EKS](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html).
- SPIFFE, [SPIFFE specification](https://spiffe.io/) — workload-identity primitives for on-prem / cross-cloud attestation.
- HashiCorp, [Vault documentation](https://developer.hashicorp.com/vault/docs) — secrets management and dynamic credentials.

## Threat modelling

- OWASP GenAI Security Project, [Agentic AI — Threats and Mitigations](https://genai.owasp.org/resource/agentic-ai-threats-and-mitigations/) — the threat taxonomy Chapter 06 uses.
- OWASP GenAI Security Project, [OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/) — underlying LLM risks that still apply to agent systems.
- OWASP, [Non-Human Identities Top 10](https://owasp.org/www-project-non-human-identities-top-10/) — NHI-specific risks.
- Greshake et al., [*Not what you've signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection*](https://arxiv.org/abs/2302.12173) — the canonical indirect prompt injection paper.
- Anthropic, [research on alignment faking](https://www.anthropic.com/research/alignment-faking) — representative of the frontier-agent safety literature `agentic-safety-engineer` (level 40) operates in.
- MITRE, [ATLAS — Adversarial Threat Landscape for AI Systems](https://atlas.mitre.org/) — attack-pattern knowledge base complementary to OWASP.

## Human-in-the-loop, oversight, and governance

- European Parliament and Council, [EU AI Act (Regulation (EU) 2024/1689)](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=OJ:L_202401689) — in particular Article 14 on human oversight for high-risk systems.
- Federal Reserve, [SR 11-7 — Guidance on Model Risk Management](https://www.federalreserve.gov/supervisionreg/srletters/sr1107.htm) — governs model-risk management in US banking; applies to agent-driven workflows in regulated financial contexts.
- NIST, [AI Risk Management Framework (AI RMF 1.0)](https://www.nist.gov/itl/ai-risk-management-framework) and the [GenAI Profile](https://airc.nist.gov/AI_RMF_Knowledge_Base/AI_RMF/5-sec-5).
- ISO/IEC, [ISO/IEC 42001 — AI Management System standard](https://www.iso.org/standard/81230.html).

## Agent evaluation

- Sierra AI, [τ-bench](https://github.com/sierra-research/tau-bench) — benchmark for user-facing agent tasks.
- Princeton + others, [SWE-bench](https://www.swebench.com/) — benchmark for coding agents on real GitHub issues.
- CMU + others, [WebArena](https://webarena.dev/) — benchmark for web-based agents.
- HuggingFace, [GAIA benchmark leaderboard](https://huggingface.co/spaces/gaia-benchmark/leaderboard).
- Anthropic, [agent-eval guidance in the Claude docs](https://docs.claude.com/en/docs/agents-and-tools) — provider-level framing on measuring agent trajectories.

## Observability and audit destinations

- Splunk, [documentation and detection content](https://docs.splunk.com/Documentation).
- Microsoft Sentinel, [documentation](https://learn.microsoft.com/en-us/azure/sentinel/).
- Google Chronicle (now Google Security Operations), [documentation](https://cloud.google.com/chronicle-security-operations/docs).
- Elastic SIEM, [documentation](https://www.elastic.co/guide/en/security/current/es-overview.html).
- Datadog, [LLM Observability and APM documentation](https://docs.datadoghq.com/).

## Peer-track references

- [`agentic-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/agentic-ai-engineer-learning) — the peer at level 30 for framework and pattern depth.
- [`agentic-safety-engineer-learning`](https://github.com/ai-engineering-curriculum/agentic-safety-engineer-learning) — the peer at level 40 for frontier red-team methodology.
- [`rag-engineer-learning`](https://github.com/ai-engineering-curriculum/rag-engineer-learning) — the peer at level 25 for retrieval depth (mod-105 Chapter 07).
- [`applied-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) — the peer at level 30 for applied-LLM craft (mod-104 Chapter 07).

<!-- needs-research: the agent-framework and standards space is moving quickly; re-check link freshness and status (e.g., ISO/IEC 42001 adoption trajectory, OWASP Agentic release numbering, framework version-specific URLs) at least quarterly -->
