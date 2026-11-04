# Chapter 02 — Selecting a Provider Stack per Customer Deployment

## Why this chapter exists

The provider-stack decision is the first LLM-layer choice an FDE makes on a pilot. It constrains everything downstream — the gateway (Chapter 03), the prompt-caching strategy (Chapter 04), the SLO numbers the pilot can realistically contract (Chapter 05), the fine-tune escalation path (Chapter 06). It also happens to be the decision customer security teams scrutinise hardest in their review (mod-108).

Three categories of provider cover the realistic design space: **frontier labs** (hosted APIs from the labs that train frontier models), **hyperscaler managed stacks** (AWS Bedrock, Azure AI Foundry, Google Vertex AI), and **self-hosted inference** (vLLM, TGI, SGLang running inside the customer's trust boundary). Each sits differently against the five-dimensional budget from Chapter 01. This chapter walks the categories, writes the five decision dimensions down with their definitions, offers a scoring template, and runs three customer archetypes through it.

## The three provider categories

### Frontier labs

The hosted APIs from the labs that train the current frontier models. The ones an FDE commonly encounters:

- **OpenAI** — [platform.openai.com/docs](https://platform.openai.com/docs). GPT-4o, GPT-4.1, o-series reasoning models. First-party structured outputs, prompt caching, streaming. Enterprise plans publish SOC 2 Type II and offer Business Associate Agreements where applicable.
- **Anthropic** — [docs.anthropic.com](https://docs.anthropic.com). Claude Sonnet / Opus / Haiku. First-party prompt caching, tool use for structured return, extended thinking for the reasoning line.
- **Google** — [ai.google.dev](https://ai.google.dev) for the Gemini API; also available through Vertex AI in the hyperscaler stack (see below). Gemini 1.5 / 2.x, long context, controlled generation.
- **xAI** — [docs.x.ai](https://docs.x.ai). Grok models via REST API.
- **Cohere** — [docs.cohere.com](https://docs.cohere.com). Command / Embed / Rerank. Enterprise emphasis, on-cloud and on-prem options.
- **Mistral** — [docs.mistral.ai](https://docs.mistral.ai). Mistral / Mixtral / Codestral. Hosted API and open-weights variants.

Common traits:

- **Fastest access to the current frontier.** New models land here first; hyperscaler listings lag by weeks to months.
- **Thinnest enterprise surface by default.** Zero-retention, data-residency, and BAAs exist, but they require specific plan tiers — the sales motion is real. Do not assume the default API plan satisfies the customer's data-use expectations.
- **Fewest knobs for in-region processing.** US-primary infrastructure; EU / regional offerings exist on specific tiers for specific providers; the shape varies per provider and changes. <!-- needs-research: confirm current EU residency tiers per provider at the time of use -->

When frontier-lab direct is the right pick: pilot has no residency constraint beyond what the lab's enterprise tier offers; the customer's AppSec allowlist includes the specific lab; the pilot wants the newest model on day of release.

### Hyperscaler managed stacks

The three hyperscalers run model-serving services that bundle multiple providers' models behind the hyperscaler's enterprise surface:

- **AWS Bedrock** — [docs.aws.amazon.com/bedrock](https://docs.aws.amazon.com/bedrock/). Anthropic (Claude), Meta (Llama), Mistral, Cohere, AI21, Amazon's own Titan / Nova. Regional availability varies by model; not every model is in every region.
- **Google Vertex AI** — [cloud.google.com/vertex-ai](https://cloud.google.com/vertex-ai/docs). Google's own Gemini plus Model Garden offerings (Anthropic, Llama, Mistral via partner integrations). Regional residency controls through Vertex AI's region configuration and VPC Service Controls.
- **Azure AI Foundry** (formerly Azure AI Studio; subsumes Azure OpenAI Service) — [learn.microsoft.com/en-us/azure/ai-services](https://learn.microsoft.com/en-us/azure/ai-services/). Azure OpenAI gives OpenAI models inside Azure's enterprise boundary; Foundry adds Meta, Mistral, and other model families.

Common traits:

- **The enterprise contract is already in place.** The customer already has an AWS / GCP / Azure contract, a BAA (if applicable), SOC 2 Type II, a data-processing agreement, a sub-processor list, a network path. The LLM traffic rides those contracts — no new vendor onboarding.
- **Regional and data-residency controls are first-class.** Pick a region, the data lives there; wire PrivateLink / Private Service Connect / Private Link Service and the data does not transit the public internet.
- **Model availability lags.** A frontier model released by its lab on Monday may not appear in a specific hyperscaler region for weeks or months, and may lag further behind in older regions.
- **The surface is the hyperscaler's, not the lab's.** Request shapes are Bedrock-style or Foundry-style or Vertex-style, not identical to the lab's native API. Gateways (Chapter 03) hide this, but it is real complexity.

When the hyperscaler route is the right pick: customer is already deeply on that cloud, data-residency is a hard requirement, security review is faster through an existing contracted vendor than a new one.

### Self-hosted inference

Open-weights (or customer-licensed) models running inside the customer's trust boundary on compute the customer or the vendor provides:

- **vLLM** — [docs.vllm.ai](https://docs.vllm.ai). High-throughput inference, PagedAttention, OpenAI-compatible server.
- **Hugging Face TGI** — [huggingface.co/docs/text-generation-inference](https://huggingface.co/docs/text-generation-inference). Production-grade serving, broad model support.
- **SGLang** — [github.com/sgl-project/sglang](https://github.com/sgl-project/sglang). Structured-generation-first runtime.

These are *serving engines*; the FDE hands model-server standup to [`ai-infra-inference-engineer`](https://github.com/ai-infra-curriculum/ai-infra-inference-engineer-learning) (peer, level 35, Infra family — the boundary is written in mod-103 Chapter 09).

Common traits:

- **Data never leaves the customer trust boundary.** The strongest residency / sovereignty posture available.
- **Open weights, with licensing** — Llama community license, Mistral licensing, DeepSeek, Qwen. Each has terms; mod-108 covers licensing. A fine-tuned derivative inherits those terms.
- **Hardware cost shifts to the customer.** H100 / A100 / L40S / Inferentia / TPU — someone is paying for the GPUs and operating them.
- **Model freedom constrained to open weights** — the current frontier closed models (GPT-5, Claude Opus, Gemini Ultra class) are not available to run here.
- **Serving depth is specialist work.** Batching, quantisation, KV-cache tuning, autoscaling — these are not FDE jobs. See mod-103 Chapter 09.

When self-hosted is the right pick: hard data-residency posture (data must not leave the customer's VPC, period); regulated verticals where provider egress is a blocker; cost at scale where self-hosted economics beat API billing; cases where the customer owns a fine-tuned adapter on open weights (Chapter 06).

## The five decision dimensions

Chapter 01 named the five-dimensional budget. This chapter uses those five as the scoring axes.

### Data residency and sovereignty

The question: *where does the prompt, the completion, and the logs physically process and rest?*

Specific sub-questions the FDE answers per candidate stack:

- In which region(s) does inference run?
- Does the prompt transit the public internet, or is it inside a private network path (PrivateLink, Private Service Connect, VPC endpoints)?
- Does the provider retain prompts for any purpose (abuse monitoring, training, debugging)? For how long? Where?
- Does the provider log prompts? Where? Who has access?
- What happens on an incident — is the customer's data in the provider's incident-response scope?

Relevant external anchors:

- **GDPR** — [General Data Protection Regulation](https://gdpr-info.eu/) and specifically Chapter V on cross-border transfers. Schrems II restricts US transfer paths; standard contractual clauses and transfer impact assessments apply.
- **EU Data Boundary** — Microsoft's [EU Data Boundary](https://learn.microsoft.com/en-us/privacy/eudb/eu-data-boundary-learn) commitments for Azure.
- **AWS data residency** — [AWS and data residency](https://aws.amazon.com/compliance/data-residency/).
- **Google Cloud data residency** — [Data residency, operational transparency, and privacy](https://cloud.google.com/security/compliance/data-residency).
- **Sector-specific** — HIPAA for health, FedRAMP for US federal, DORA for EU financial services. Mod-108 covers each in depth.

This dimension is a **gate**. A provider that cannot satisfy the customer's residency and sovereignty posture is not a candidate at any score.

### Compliance

The question: *does this stack sit inside the customer's regulatory regime?*

Specific sub-questions:

- SOC 2 Type II — current report, in-scope controls, clean opinion.
- HIPAA — BAA available and signed at the plan tier the pilot uses.
- FedRAMP — Moderate or High authorization on the hyperscaler region (see the [FedRAMP Marketplace](https://marketplace.fedramp.gov/)); specific services inside that boundary.
- PCI DSS — attestation of compliance for payment-card workloads.
- ISO/IEC 27001, 27701 — information security and privacy information management.
- EU AI Act — high-risk system obligations (Articles 9-15 technical requirements, Article 72 post-market monitoring). Takes effect in staged dates; mod-108 covers.
- State / local — NYC LL 144 bias audits, Colorado AI Act, CCPA, Illinois BIPA.

This dimension is also a **gate**. "Not yet certified" is an acceptable answer only if the customer accepts it in writing.

### Latency

The question: *can this stack meet the SLOs the pilot is contracting?*

Specific sub-questions:

- TTFT target (Chapter 05) vs. this provider's observed TTFT at the model tier the pilot uses.
- Regional availability — if the pilot's workload is in us-east-1 and the model is only in us-west-2, add 60-80ms round trip.
- Cold-start behaviour for less-used models / regions.
- Steady-state throughput headroom at the pilot's projected load.
- For self-hosted: cold-start on scale-from-zero, batching latency penalty, queue depth at peak.

This dimension is an **optimisation**, not a gate — unless the SLO is tight enough that only one category can meet it (rare at pilot stage, common post-pilot).

Concrete numbers are omitted here because they move month to month. Benchmark anchors the FDE reads before committing:

- **Artificial Analysis** — [artificialanalysis.ai](https://artificialanalysis.ai/) — ongoing, provider-reported latency and quality.
- **Each provider's own status / latency dashboards** — [OpenAI status](https://status.openai.com), [Anthropic status](https://status.anthropic.com), hyperscaler status pages.
- **Internal baselining** — the only authoritative latency number is the one measured on the customer's egress path to the specific region at the pilot's prompt sizes.

### Cost

The question: *does per-task cost fit inside the value-per-task the pilot promised?*

Specific sub-questions:

- Per-token cost for the model tier the pilot uses, at the typical prompt + completion sizes. Published pricing pages:
  - [OpenAI pricing](https://openai.com/api/pricing/)
  - [Anthropic pricing](https://www.anthropic.com/pricing#api)
  - [AWS Bedrock pricing](https://aws.amazon.com/bedrock/pricing/)
  - [Google Vertex AI pricing](https://cloud.google.com/vertex-ai/pricing)
  - [Azure AI Foundry / Azure OpenAI pricing](https://azure.microsoft.com/en-us/pricing/details/cognitive-services/)
- Prompt-caching discount shape — provider-specific (Chapter 04).
- Commitment-based pricing — provisioned throughput on hyperscalers, enterprise contracts on frontier labs.
- Self-hosted total cost — GPU hours, GPU utilization, idle-time cost on autoscaling, operator time.

This dimension is an **optimisation**. Named in shape here because specific numbers change; the FDE always reads the current pricing page before writing numbers into the pilot scope.

### Model-choice freedom

The question: *can the pilot route sub-tasks to the model each one needs, or is it locked to one family?*

Routing patterns the overlay cares about:

- Draft generation on a mid-tier model; critique / judge on a higher tier.
- Fast classification on a small model; complex reasoning on a reasoning model.
- Vision on a vision-capable model; text on text-only.
- Embedding on a specialised embedding model (Cohere, OpenAI, Vertex, local).

Scoring:

- Frontier labs give the fullest model shelf within their own family.
- Hyperscaler managed stacks give a mixed shelf (Bedrock: Anthropic + Meta + Mistral + Cohere + Amazon; Foundry: OpenAI + Meta + Mistral; Vertex: Gemini + Model Garden) — fewer than any single lab's full offering but more cross-provider choice than any one lab.
- Self-hosted limits to open weights the serving engine supports.

The gateway (Chapter 03) is what *operationalises* model-choice freedom in code — one SDK surface routing to many providers. Without a gateway, model-choice freedom is a theoretical property.

## A scoring approach

Score each candidate stack on each dimension 1–5. The FDE weights the dimensions by customer context — residency and compliance weight higher in regulated verticals, latency weights higher for real-time user-facing workflows, cost weights higher for high-volume background workflows.

Starter template:

```
Customer: <name>  |  Pilot: <workflow>  |  Date: 2026-11-04

Dimension             Weight   Frontier Labs   Hyperscaler    Self-hosted
                               (Lab X)         (AWS Bedrock)  (vLLM + Llama)
---------------------------------------------------------------------------
Data residency        W_R      s_FR * W_R      s_HS * W_R     s_SH * W_R
Compliance            W_C      s_FC * W_C      s_HC * W_C     s_SC * W_C
Latency               W_L      s_FL * W_L      s_HL * W_L     s_SL * W_L
Cost                  W_$      s_F$ * W_$      s_H$ * W_$     s_S$ * W_$
Model-choice freedom  W_M      s_FM * W_M      s_HM * W_M     s_SM * W_M
---------------------------------------------------------------------------
                                 TOTAL           TOTAL          TOTAL

Gate check:
  - Any s_R = 0 → category disqualified on residency
  - Any s_C = 0 → category disqualified on compliance
```

Rules:

- **Gate before score.** Any category that fails residency or compliance drops out. No summing against a disqualified category.
- **Rationale per cell.** Every cell has a one-line rationale citing a primary source (the provider's compliance page, the region table, the pricing page). The matrix is a security-review artifact; a reviewer will ask about any cell.
- **Weights are defended in the ADR.** Why residency weight = 3 and cost weight = 1 for this customer? Point to the data-contract conversation and the business-KPI value-per-task.
- **Date the matrix.** Pricing and model availability change. A matrix from six months ago is a snapshot, not a current recommendation.

## Three customer archetypes and their likely stack

Three sketches to make the scoring concrete. These are shapes, not prescriptions — a specific customer's pilot can land anywhere.

### Archetype A — Mid-size US healthcare payer

Constraints:

- HIPAA + state-level privacy. BAA required at every hop.
- SOC 2 Type II onboarding checklist.
- Cloud-of-record: AWS, three years of operational maturity.
- Latency tolerance: a few seconds p95 on member-inquiry summarization.

Likely stack: **AWS Bedrock** with Anthropic Claude (BAA covered by AWS's BAA, data-residency inside the customer's AWS region, enterprise contract already signed). Gateway (Chapter 03) is LiteLLM self-hosted on EKS in the customer's VPC. Direct frontier-lab access is possible if the lab's HIPAA tier is in scope, but the AWS path is the shorter review.

Rationale rows: residency = inside AWS region (5); compliance = BAA under AWS (5); latency = acceptable within AWS region (4); cost = standard Bedrock rates (3); model-choice = Anthropic + Mistral + Meta on Bedrock is adequate (4).

### Archetype B — US federal agency

Constraints:

- FedRAMP Moderate baseline with a path to High.
- Data classified at For Official Use Only.
- Must deploy inside their GovCloud enclave.
- Latency tolerance: several seconds on policy-analyst drafting.

Likely stack: **Self-hosted** open-weights model (Llama 3 class) on GovCloud GPU instances via vLLM, with model-serving owned by `ai-infra-inference-engineer`. Alternative: Bedrock in [AWS GovCloud](https://aws.amazon.com/govcloud-us/) <!-- needs-research: confirm Bedrock model availability in AWS GovCloud at pilot time --> — if the required model is available there and FedRAMP-High-authorized. Frontier-lab direct is off the table because of the FedRAMP boundary.

Rationale rows: residency = inside GovCloud (5); compliance = FedRAMP-authorized serving (5 or disqualify depending on status); latency = GovCloud regional latency plus self-hosted overhead (3); cost = GPU time plus operations (2); model-choice = limited to open weights the serving engine supports (2).

### Archetype C — EU-based retail bank

Constraints:

- GDPR + DORA (Digital Operational Resilience Act for EU financial services; see [DORA](https://www.digital-operational-resilience-act.com/) / [EIOPA](https://www.eiopa.europa.eu/)).
- Data must remain inside the EU.
- Cloud-of-record: Azure, with EU Data Boundary commitments in place.
- Latency tolerance: a few seconds p95 on customer-support draft replies.
- Must support function calling for CRM workflow handoff.

Likely stack: **Azure AI Foundry / Azure OpenAI** in an EU region (Sweden Central / West Europe / North Europe), with the EU Data Boundary active. Gateway inside the customer's Azure Virtual Network, egress to the model service via a Private Endpoint. Frontier-lab direct is harder to justify under DORA's third-party ICT risk regime unless the lab holds equivalent assurances.

Rationale rows: residency = EU region under EU Data Boundary (5); compliance = SOC 2 + ISO 27001 + customer's own DORA control mapping (4 — customer still writes the ICT concentration-risk register); latency = EU-region native (4); cost = Azure contract pricing (3); model-choice = OpenAI + Meta + Mistral in Foundry (4).

Note the shape pattern: the scoring table changes across archetypes mainly because the weights change, not because the raw provider facts do. Residency weight is maximal for Archetype B, high for Archetypes A and C, and would be moderate for a US SaaS with no vertical regulation.

## The provider decision as an ADR

Mod-102 Chapter 04 fixed the pilot-scope ADR shape. The provider-stack ADR follows the same template, scoped to this decision:

```
ADR-NN: Provider stack for <customer> pilot <workflow>

Status: proposed | accepted | superseded (date, by ADR-MM)

Context
  - Customer: <name, vertical>
  - Workflow: <one-line description, from pilot-scope doc>
  - Regulatory regime: HIPAA / FedRAMP / GDPR+DORA / none
  - Cloud of record: AWS / GCP / Azure / multi
  - Latency SLO draft (Chapter 05): p95 TTFT < X, p95 completion < Y
  - Cost cap draft: < $Z per task
  - Model-choice needs: single family / multi-provider / reasoning tier

Decision
  <Category + specific provider(s)>
  Example: AWS Bedrock as primary, with Anthropic Claude Sonnet as the default model
  and Claude Haiku as the fast-tier for classification sub-tasks.

Alternatives considered
  - <Frontier labs direct> — rejected because <reason>
  - <Other hyperscaler> — rejected because <reason>
  - <Self-hosted> — deferred; see ADR-MM for the self-hosted path if pilot graduates.

Consequences
  - Positive: <residency satisfied, BAA path clear, existing contract>
  - Negative: <latest model will lag the lab's release by weeks>
  - Risks: <model availability in required region; mitigation: dual-region failover>

Gate checks
  - Residency: pass (in customer AWS region)
  - Compliance: pass (BAA under AWS, SOC 2 Type II)

Sign-off
  - FDE: <name>
  - Customer AppSec: <name>
  - Customer Legal (if required by regime): <name>
  - Date: <YYYY-MM-DD>
```

The ADR is a page. It is read by the customer's security team at onboarding, the account team at renewal, and the next FDE at hand-off. Keep it short enough to read in three minutes.

## Per-category eyebrow-raises

Specific facts to check before committing to each category, from the field:

- **Bedrock** — model availability varies by region; verify the target model is in the pilot's target region before scope-lock. Bedrock Guardrails are configurable but opinionated; test with real pilot content before relying on them. Cross-region inference (Bedrock Cross-Region Inference) has implications for data residency — read the regional routing policy.
- **Vertex AI** — some third-party model access is through Model Garden with specific regional restrictions. VPC Service Controls are powerful and configured by the customer's platform team, not the pilot; coordinate early.
- **Azure OpenAI / Foundry** — content filter is on by default and may flag otherwise-expected enterprise content (legal, medical, policy); the filter policy can be adjusted but requires explicit configuration. Capacity (PTU — provisioned throughput units) is a separate contract from pay-as-you-go and is relevant at scale.
- **Frontier labs (any)** — zero-retention and residency tiers exist but require enterprise plans; do not assume defaults.
- **Self-hosted** — serving operations is a specialist job for `ai-infra-inference-engineer`. The FDE briefs, does not operate. See mod-103 Chapter 09 for the brief shape.

## Anti-patterns

**Picking by the most recent benchmark headline.** The frontier model with the headline MMLU score may not be in the customer's region, approved by the customer's AppSec, or appropriate for the pilot's structured-output shape. Benchmarks inform; they do not decide.

**Ignoring residency until the security review.** Chapter 01 named this. It is worth repeating: residency is a gate, scored in week 2, not surfaced in week 10.

**Assuming the pilot locks the stack forever.** The gateway (Chapter 03) is the abstraction that keeps the provider-stack decision a reversible choice. Writing the pilot to call `anthropic.messages.create(...)` directly locks in; writing it to call the gateway's OpenAI-compatible endpoint keeps the option open.

**Not writing the ADR.** Decisions that live only in a slide deck are decisions that get re-litigated in week 10 by someone who was not in the room in week 2. The ADR is the artifact that survives.

## Summary

- Three provider categories span the design space: frontier labs direct, hyperscaler managed stacks, self-hosted inference.
- Frontier labs ship the newest models fastest; hyperscalers carry the customer's existing enterprise contract; self-hosted keeps data inside the trust boundary at the cost of operational depth.
- Five decision dimensions score each candidate: data residency (gate), compliance (gate), latency, cost, model-choice freedom.
- Residency and compliance are gate dimensions — fail either and the category drops out of the scoring, regardless of other strengths.
- The scoring matrix lives in the pilot's documentation, dated, with a one-line rationale per cell citing a primary source; weights are defended in the ADR.
- Three customer archetypes point to typical landing spots: AWS Bedrock for a US healthcare payer; self-hosted (or GovCloud managed) for a US federal agency; Azure Foundry for an EU financial-services customer under DORA.
- The provider decision is captured as a one-page ADR signed by the FDE, the customer's AppSec, and the customer's legal where required.
- Chapter 03 is next: the model gateway that makes the provider decision reversible and operationalises per-tenant budget, failover, caching, PII guards, and audit logs on top of it.
