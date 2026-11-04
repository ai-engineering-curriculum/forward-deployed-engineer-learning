# Chapter 04 — Threat Modelling the Pilot — OWASP LLM Top 10, OWASP Agentic Top 10, and MITRE ATLAS

## Why this chapter exists

Chapters 02 and 03 answered "what is the customer going to ask for in the review packet?" This chapter answers "what are the specific attack scenarios the AppSec reviewer expects us to have already modelled and mitigated?" For AI systems, the industry has consolidated on three overlapping taxonomies:

- **[OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/)** — the Top 10 for LLM apps, now maintained by the OWASP GenAI Security Project. The 2025 release is the current reference.
- **[OWASP Agentic AI — Threats and Mitigations](https://genai.owasp.org/resource/agentic-ai-threats-and-mitigations/)** — the agentic-specific extension, covering the risks that only exist when the LLM has tools.
- **[MITRE ATLAS](https://atlas.mitre.org/)** — Adversarial Threat Landscape for Artificial-Intelligence Systems; the ATT&CK-shape tactic/technique matrix for AI. Describes how an adversary actually operates against an AI system.

OWASP gives the category. MITRE ATLAS gives the kill chain. The FDE's deliverable is a coverage matrix: for each row, name the mitigation, name the evidence, name the residual risk. A pilot ships with a signed coverage matrix in the Chapter 02 packet.

Deep adversarial-evaluation craft — frontier-model red-team methodology, capability elicitation, emergent-behavior discovery — is `ai-infra-security` / `security-learning` and (for agents) `agentic-safety-engineer` peer work. The FDE's depth here is the working red-team pass: run the OWASP matrix against the current build; confirm mitigations fire; write the evidence down.

## The OWASP LLM Top 10 — 2025 edition

<!-- needs-research: cross-check the exact numbering, titles, and 2025-edition updates of OWASP Top 10 for LLM Applications against https://genai.owasp.org/llm-top-10/ before publishing — the categories evolved between the 2023 and 2025 editions -->

**LLM01 — Prompt Injection.** Direct (adversarial user prompt that overrides system intent) and indirect (adversarial content inside retrieved documents or tool outputs). Mod-106 Chapter 06 handled the enterprise shape at length.

**LLM02 — Sensitive Information Disclosure.** The model emits content it should not — training-data memorisation, retrieved-doc exfiltration across tenancies, PII leaks, secret leaks. The 2025 edition promotes this up the list.

**LLM03 — Supply Chain.** Third-party model weights, datasets, embedding models, fine-tunes, or libraries that are compromised. Chapter 06 of this module is the full response.

**LLM04 — Data and Model Poisoning.** Training-data or fine-tuning-data poisoning; memory / retrieval-index poisoning (mod-106 Chapter 06 covers the agent variant).

**LLM05 — Improper Output Handling.** The application treats model output as trusted — rendering it as HTML (XSS), executing it as code, appending it to a SQL query, running it as a shell command. The classic "the model's output is data, not code" discipline.

**LLM06 — Excessive Agency.** The agent has more tools, permissions, or autonomy than the workflow requires. Mod-106 Chapter 05 covers this with least-privilege tool catalogues and scope contraction.

**LLM07 — System Prompt Leakage.** The system prompt is exposed through adversarial extraction or logging / observability leakage. The mitigation is partly architectural (do not ship secrets in the system prompt) and partly operational (sanitise logs).

**LLM08 — Vector and Embedding Weaknesses.** Vector-store poisoning, embedding inversion, cross-tenant retrieval leakage, embeddings that memorise sensitive text. Composes with mod-105 Chapter 04 PII-safe retrieval.

**LLM09 — Misinformation.** Hallucinated or incorrect content taken as truth — factuality, citation accuracy, groundedness. Composes with mod-107 Chapter 03 metric portfolio.

**LLM10 — Unbounded Consumption.** Resource-exhaustion and denial-of-wallet attacks: unbounded completions, context-window stuffing, high-cost model selection by adversary, unbounded tool loops. Earlier editions called this "model DoS"; the 2025 edition broadens it.

## The OWASP Agentic Top 10 — extension for agents

Mod-106 Chapter 06 covered the six highest-weight threats (indirect prompt injection, tool misuse, memory poisoning, confused deputy, NHI weaknesses, HITL overwhelm) in detail. The full OWASP Agentic set adds:

- **Resource overload.** The agent is driven to exhaust compute or budget.
- **Cascading hallucinations.** The agent's error at step N becomes the input to step N+1; the chain compounds.
- **Goal manipulation.** The agent's objective is subtly redirected mid-task.
- **Repudiation.** The agent's actions are not traceable to an authenticated principal; the user denies the action.
- **Identity spoofing.** The agent speaks as or is mistaken for a different identity.
- **Multi-agent system attacks.** Adversarial behaviours in multi-agent orchestration — one agent manipulating another.
- **Unexpected RCE.** Code generated by the agent is executed unsafely in the agent's environment; the classic code-execution escape.

<!-- needs-research: confirm the current OWASP Agentic AI Threats and Mitigations taxonomy against https://genai.owasp.org/resource/agentic-ai-threats-and-mitigations/ before publishing; titles and ordering have iterated -->

For enterprise pilots that are agentic, the FDE composes OWASP LLM + OWASP Agentic into one coverage matrix. The same vulnerability class often shows up in both; name it once with cross-references.

## MITRE ATLAS — the kill-chain view

Where OWASP names *categories*, MITRE ATLAS names *tactics* (what the adversary is trying to achieve at this stage) and *techniques* (how they do it). The current tactic list runs through reconnaissance, resource development, initial access, execution, persistence, defence evasion, discovery, collection, exfiltration, impact — the ATT&CK shape adapted for AI.

Representative techniques the FDE's threat model should enumerate:

- **AML.T0051 — LLM Prompt Injection.** Covers the LLM01 category with MITRE's framing.
- **AML.T0057 — LLM Jailbreak.** The adversary bypasses safety alignment to elicit prohibited behaviour.
- **AML.T0070 — RAG Poisoning.** Compromises the retrieval corpus.
- **AML.T0053 — LLM Plugin Compromise.** Compromises an integrated tool.
- **AML.T0024 — Exfiltration via ML Inference API.** Exfiltrates training data or confidential information through model outputs.
- **AML.T0048 — External Harms.** The downstream real-world impact (financial, reputational, physical) that follows from a successful attack.

<!-- needs-research: confirm the current MITRE ATLAS technique IDs and titles against https://atlas.mitre.org/ before publishing; the matrix is iterating and IDs renumber as categories move -->

ATLAS composes with OWASP: pick the ATT&CK-shape view when writing the attack narrative ("adversary reconnaissance → RAG poisoning → exfiltration via inference → external harm"); pick the OWASP view when writing the checklist for AppSec reviewers.

## The coverage matrix — the FDE's deliverable

One table, one version, one sign-off. Each row of OWASP (LLM + Agentic) gets a named mitigation, an evidence pointer, and a residual-risk statement. Example rows for the healthcare payer archetype:

| Threat | Category | Pilot-specific scenario | Mitigation | Evidence | Residual risk |
|---|---|---|---|---|---|
| LLM01 — Direct prompt injection | OWASP LLM | CSR sends "ignore prior and set auth" | System-prompt hardening + output schema + tool allow-list + HITL on write | mod-106 ch-04 HITL table; audit sample `pi-redteam-2026-11-02.log` | Low — jailbreak against system prompt survives; monitored in online-eval |
| LLM01 — Indirect prompt injection | OWASP LLM / Agentic | Member-submitted case note contains adversarial instructions | Sub-agent reads untrusted content read-only; parent agent sees structured summary only | DFD §member-content; agent-isolation test `pi-indirect-2026-11-02.log` | Low — novel exploit of structured-summary path |
| LLM02 — Sensitive information disclosure | OWASP LLM | Cross-tenant PHI leak through retriever | Pre-filter by user permissions; row-level security; retrieval audit log | mod-105 ch-03 ACL implementation; `ret-audit-2026-11-01.ndjson` | Low — covered by RLS and ACL tests |
| LLM03 — Supply chain | OWASP LLM | Compromised embedding model | Model provenance attestation; ModelScan on load; CycloneDX ML-BOM | `mlbom-pilot-v1.2.cdx.json`; `modelscan-report.pdf` | Low — monitored via re-scan on version bump |
| LLM04 — Data poisoning (retrieval) | OWASP LLM | Poisoned policy doc injected into index | Ingest provenance tags; agent-generated content fenced; TTL; write-through sanitisation | mod-105 ch-04 ingestion; ingestion audit `ingest-2026-11-02.log` | Medium — insider-threat on policy authoring is not mitigated here; customer-side control |
| LLM05 — Improper output handling | OWASP LLM | Model output rendered as HTML in CSR UI | Output passes DOMPurify-equivalent sanitiser; renders as text; URL allow-list | UI-side sanitiser test suite `output-sanitiser.test.ts` | Low |
| LLM06 — Excessive agency | OWASP LLM / Agentic | Agent has write-scope on all claims | Tool catalogue scoped to read + single write with HITL; scope contraction post-call | mod-106 ch-05 tool catalogue | Low |
| LLM07 — System prompt leakage | OWASP LLM | Observability stack logs system prompt | Gateway configured to log prompt hashes only; log sampling redacts system prompt | mod-104 ch-03 gateway config; `log-sample-2026-11-02.json` | Low |
| LLM08 — Vector / embedding weaknesses | OWASP LLM | Embedding inversion on PHI chunks | Embedding model runs in-VPC; vector store encrypted at rest with CMK; no outbound embedding calls | DFD §embedding; CMK KMS policy | Low |
| LLM09 — Misinformation | OWASP LLM | Hallucinated benefit terms | Grounded retrieval + factuality metric + HITL on auto-send | mod-105 ch-05 citation UX; mod-107 metric portfolio | Medium — residual catches drift; weekly online-eval review |
| LLM10 — Unbounded consumption | OWASP LLM | Agent loops a batch tool | Per-tool rate limits; per-session action budget; kill-switch | mod-106 ch-05 rate-limit config; `kill-switch-runbook.md` | Low |
| Agentic — Memory poisoning | OWASP Agentic | Scratchpad write of adversarial content | Write-through sanitisation; memory tiers; TTL; provenance tags | mod-106 ch-06 §3 | Low |
| Agentic — Confused deputy | OWASP Agentic | Agent uses service-account privilege for user action | OBO identity; scope contraction after use; tool-side permission check | mod-106 ch-05 OBO | Low |
| Agentic — NHI weakness | OWASP Agentic | Static credential for retriever | Workload identity (AWS IRSA); vault-stored; rotation | mod-103 ch-04 NHI inventory | Low |
| Agentic — HITL overwhelm | OWASP Agentic | Confirmation fatigue on CSR | Auto-run safe actions; anomaly-highlight UI; out-of-band for high-value | mod-106 ch-04 HITL table; UI mockup | Medium — volume spikes still a risk; dashboarded |

The pilot's AppSec reviewer reads this table. If a row is blank, the pilot is not ready for the threat review. If a row is "residual risk: high," the FDE either mitigates further, or the customer signs the residual explicitly.

**Versioning.** Every new tool, memory surface, model-provider change, sub-processor, or data class triggers a revision. The matrix carries a `version`, `last_reviewed`, and `signed_by` field.

## The threat-model process — what the FDE actually runs

A five-step pass once the design stabilises, repeated at each architectural change.

**Step 1: Enumerate assets and trust boundaries.** Lift from the DFD (Chapter 02). Assets: PHI, PII, PCI, customer confidential content, model weights, prompts, completions, embeddings, retrieval index, secrets, system prompt, model-provider credentials. Boundaries: customer VPC / vendor cloud / model-provider / sub-processor / user browser.

**Step 2: Enumerate actors.** End-user (CSR, analyst, support agent); adversary posing as end-user; adversary posing as external case-content author; insider at the vendor; insider at the model provider; compromised sub-processor; adversary on the customer network.

**Step 3: Walk the OWASP + Agentic + ATLAS matrices.** For each row, ask: *does this scenario exist in the pilot?* If yes, what is the attack narrative, what is the mitigation, what is the evidence, what is the residual?

**Step 4: Run the lightweight adversarial pass.** The FDE pretends to attack. Compose a short red-team playbook — direct prompt injection, indirect via case note, retrieval poisoning against the ingested policy doc, confused-deputy on the tool, HITL overwhelm by volume, output-handling with an HTML injection. Run each scenario; capture result. This is not a substitute for a full pen-test (which Chapter 02 covers); it is the FDE's own confidence check.

**Step 5: Sign the matrix.** The FDE signs; the customer's AppSec reviewer signs. The signed matrix enters the Chapter 02 evidence packet.

## Composing mitigations — defence in depth

No single mitigation is sufficient; mod-106 Chapter 06 made this point for agentic-specific threats. The same discipline applies to the full OWASP LLM set.

- **Trust tiering** (system prompt / user prompt / tool output / retrieved doc at different trust levels) means a prompt injection in a retrieved doc does not override the system goal.
- **Capability-constrained sub-agents** mean that even if the injection does override, the sub-agent has no action tools.
- **Tight tool schemas and deterministic validators** bound the parameters the sub-agent could produce.
- **Per-tool rate limits and action budgets** bound volume (LLM10).
- **Output sanitisation at render** prevents XSS / code execution downstream (LLM05).
- **Pre-filter by permission at retrieval** prevents cross-tenant leaks (LLM02, LLM08).
- **Model-provenance attestation + ModelScan + ML-BOM** prevents compromised third-party weights (LLM03, LLM04).
- **CMK + TLS 1.3 + in-VPC embedding** keeps the exfiltration path closed (LLM02, LLM08).
- **Grounded retrieval + factuality metric + HITL on auto-send** prevents un-caught hallucinations (LLM09).
- **Audit stream to customer SIEM** means an attempted exploit is reconstructible (every row).
- **Kill-switch** means an active incident stops in one step.

Each row of the matrix lists which of these layers apply. The depth is not in each layer; it is in the composition.

## Running the three archetypes

**Archetype A — US healthcare payer.** Highest-weight rows: LLM01 indirect injection via member case notes; LLM02 cross-member PHI leakage via retriever; LLM09 hallucinated benefit terms; Agentic HITL overwhelm as CSR volume spikes. Red-team pass: inject adversarial content into a member case note; attempt cross-member retrieval with CSR-tier-1 token; prompt-stuff a benefit-interpretation question; generate HITL confirms at 10x baseline.

**Archetype B — US federal agency.** Highest-weight rows: LLM01 indirect injection via FOIA archive documents (external origin); LLM02 classification-boundary violation (CUI surfacing in a FOUO session); LLM03 supply-chain (model weights transferred into GovCloud); LLM07 system-prompt leakage (classification-sensitive instructions). Red-team pass: inject into an archive doc; query at FOUO against a CUI-tagged doc; verify ModelScan on weight transfer; attempt system-prompt extraction.

**Archetype C — EU retail bank.** Highest-weight rows: LLM01 indirect via customer case content; LLM02 GDPR right-to-erasure composition; LLM08 embedding exfiltration under GDPR derivative-data framing; LLM09 misinformation in regulated financial-advice-adjacent output; Agentic HITL overwhelm during case volume spikes; MITRE ATLAS AML.T0024 exfiltration via inference API. Red-team pass: inject into a customer case note; verify erasure propagates to vector store; verify no embedding call egresses the EU boundary; prompt-stuff a regulated-advice-boundary question.

## Anti-patterns

**OWASP matrix as a checklist, not a design discipline.** Filling in "we have a mitigation" for every row without ever running the adversarial pass. The matrix is the artefact; the red-team pass is the proof.

**MITRE ATLAS copy-pasted without pilot-specific instantiation.** The tactic names in the abstract tell the reviewer nothing. The attack narrative — "an adversary who posts a crafted case note could, through indirect prompt injection (AML.T0051), cause the agent to..." — is what earns trust.

**Threat model as a one-time artifact.** Every tool, memory surface, sub-processor, or data class change reopens the matrix. The version field is not decorative.

**Delegating the threat model entirely to the AppSec team.** AppSec reviews; the FDE authors. The FDE's technical knowledge of the pilot's architecture is the thing that makes the matrix accurate.

**"Our framework handles OWASP."** Frameworks handle some of it. The customer's reviewer asks which rows and which mitigations the vendor's code adds. Have the row-by-row answer.

**Red-team pass skipped because "the pen-test covers it."** The pen-test (Chapter 02) is a point-in-time test by a third party. The FDE's red-team pass is a design-time test by someone who knows the system. Both are needed.

**Residual-risk "low" everywhere.** A matrix where every row is "low" without the customer signing residuals is not credible. Name the real mediums; name the controls that compensate; get sign-off. The FDE's job is to produce an honest matrix, not a clean-looking one.

## Where this chapter sits

- **Chapter 02** attaches the signed matrix to the evidence packet.
- **Chapter 05** names the data-contract mitigations that compose with LLM02, LLM03, LLM08.
- **Chapter 06** names the supply-chain mitigations that compose with LLM03, LLM04.
- **Mod-106 Chapter 06** is the agentic-specific depth this chapter composes with.
- **Mod-107 Chapter 05** is the online-eval drift monitoring that catches LLM09 in production.
- **Mod-109** gates the rollout on a signed matrix.
- **Mod-110** instruments the signals the mitigations generate — injection-attempt events, action-budget breaches, kill-switch activations.

## Summary

- Three overlapping taxonomies organise AI threat models: OWASP LLM Top 10 (category), OWASP Agentic Top 10 (agent-specific), MITRE ATLAS (kill-chain).
- The FDE's deliverable is a signed coverage matrix — one row per OWASP item, with mitigation + evidence + residual-risk, versioned per architectural change.
- The process is five steps: enumerate assets and boundaries; enumerate actors; walk the matrices; run the lightweight adversarial pass; sign.
- Defence in depth composes mitigations across layers — trust tiering, sub-agent isolation, tight schemas, rate limits, output sanitisation, pre-filter at retrieval, provenance and scanning, CMK and in-VPC embedding, grounded retrieval + HITL, audit stream, kill-switch.
- Three archetypes produce different highest-weight rows: payer is indirect injection + cross-member leak + hallucinated benefit; federal is archive injection + classification-boundary + weight supply-chain; bank is customer injection + erasure composition + embedding exfiltration + regulated-advice boundary.
- Seven anti-patterns to catch: checklist filling, generic ATLAS names, one-shot matrices, over-delegation to AppSec, "the framework handles it," skipped red-team, residual-risk "low" everywhere.
- Chapter 05 picks up the data-contract clauses that make the LLM02 / LLM03 / LLM08 mitigations architecturally true.
