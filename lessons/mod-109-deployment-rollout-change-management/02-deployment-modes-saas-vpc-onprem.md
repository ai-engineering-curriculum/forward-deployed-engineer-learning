# Chapter 02 — Deployment Modes: SaaS, Dedicated SaaS, VPC Install, and On-Prem / Air-Gapped

## Why this chapter exists

Chapter 01 named the four deployment archetypes and claimed each one shapes four physical questions — who owns the pipeline, what update cadence is realistic, how the kill-switch reaches production, and which inference options are reachable without breaking the data contract from [mod-108 Chapter 05](../mod-108-enterprise-security-and-compliance/05-data-contract.md). This chapter puts the per-environment trade-offs on the table and specifies what the FDE physically ships for each mode. Deployment-mode fit is a *gate* in the six-dimensional budget; a workflow placed in the wrong mode cannot be patched later with flags or canaries. The output of this chapter is a decision memo plus a packaged artefact — the artefact is the Terraform module, the Helm chart, the SaaS API contract, or the air-gapped bundle, depending on which mode the memo resolves to.

The chapter also makes two coupling moves explicit. First, **model-choice and deployment-mode are one decision, not two** — Chapter 01 already placed this coupling in the overlay matrix; this chapter writes the actual per-mode inference options the FDE must understand. Second, **the gateway from [mod-104 Chapter 03](../mod-104-llm-app-in-customer-envs/03-model-gateway-in-customer-env.md) is the attachment point** that keeps the data contract true across all four modes — zero-retention routing, residency enforcement, BYOK, and training-opt-out all land in the gateway regardless of mode.

## The four axes of trade-off

Every section below returns to the same four axes.

1. **Latency.** Internet-bound (SaaS multi-tenant) vs same-region (dedicated SaaS) vs same-VPC / same-AZ (VPC install) vs on-prem local-serving (on-prem / air-gapped). Each step down the list shaves tens to hundreds of milliseconds.
2. **Model choice.** Hosted frontier APIs (Anthropic, OpenAI direct) vs hyperscaler-native under customer keys (Bedrock, Azure OpenAI, Vertex AI) vs self-hosted open weights on customer GPUs (vLLM, TGI, NIM, Triton, llama.cpp). Each mode narrows the set of models that can legally be reached.
3. **Update cadence.** Continuous / daily (SaaS) → weekly or biweekly (dedicated SaaS) → monthly or quarterly with CAB approval (VPC install) → ATO-gated or regulatory-change-control-gated, quarterly to annual (on-prem). Cadence drives the whole rollout machinery downstream.
4. **Support model.** Vendor-paged (SaaS) → vendor-paged with customer-side Tier 1 (dedicated SaaS) → split (customer-side Tier 1/2, vendor FDE Tier 3 — VPC install) → customer-paged with vendor consultation (on-prem). The pager protocol in [Chapter 08](./08-peer-boundaries-and-pager-protocol.md) writes the exact tiers.

Four modes, four axes. Twelve of the sixteen numbers in the resulting matrix are already determined by the mode choice; the FDE only parameterises the remaining four per customer.

## Mode 1 — SaaS multi-tenant

**Who runs it.** The vendor. The customer consumes an endpoint (`https://api.vendor.ai/v1/...`) over the public internet, authenticated via API key or SSO-federated OIDC. The runtime, the pipeline, the model-provider integration, and the observability stack all live in the vendor's cloud.

**Model choice.** Whatever the vendor's platform offers — typically hosted frontier APIs (Anthropic, OpenAI) via the vendor's own enterprise contracts, plus a hyperscaler-native fallback. The customer does not pick the model account; the vendor does.

**Latency envelope.** Internet-bound. Realistic p95 for a single LLM call is the model-provider's own p95 plus transit RTT — on the order of 200-600ms for short-context single-turn, higher with RAG or agentic loops. <!-- needs-research: cite current public p95 latency numbers for Anthropic / OpenAI production APIs — these drift and should not be fabricated -->

**Update cadence.** Continuous. The vendor ships multiple times a day. The customer is on the current version by default, with at most a brief announcement window on breaking changes. There is no customer-side CAB — the vendor's internal CAB is the only gate.

**Support model.** Vendor-paged. The vendor's SRE rotation owns every alert; the customer has a CSM but no runtime responsibility. SLA is the vendor's master SLA with no per-customer modifications.

**What the FDE physically ships.** An API endpoint plus a thin client SDK (Python / TypeScript / Java / Go), onboarding docs, SSO configuration for the customer's IdP (Okta / Entra ID / Ping), and a tenant-scoped kill-switch read-flip in the vendor's flag platform ([LaunchDarkly](https://launchdarkly.com/), [Statsig](https://statsig.com/), [Unleash](https://www.getunleash.io/), [OpenFeature](https://openfeature.dev/)). No infrastructure is shipped into the customer's account.

**When it is right.** Low-sensitivity workflows; non-regulated or lightly-regulated verticals; customers with no data-residency constraint and no CMK requirement; early-stage pilots where time-to-first-value is the constraint; customers whose procurement accepts standard DPAs without BAA-grade addenda.

**When it is wrong.** HIPAA-covered workflows without a vendor BAA for every sub-processor in the path (mod-108 Chapter 05); workflows under GDPR with EU-residency requirements that cross the vendor's regional boundary; workflows under FedRAMP Moderate or High that touch non-authorised infrastructure; any workflow the customer's CISO has committed in writing to run inside the customer's own trust boundary.

## Mode 2 — Dedicated SaaS

**Who runs it.** The vendor still runs it, but the artefact lands in a customer-dedicated namespace or account — separate Kubernetes namespace, separate database schema, separate KMS key, separate log stream, separate flag environment, separate retrieval indices. Multi-tenancy ends at the compute pool; isolation is tenant-scoped above it.

**Model choice.** The vendor's frontier-API contracts plus a hyperscaler-native fallback under *the customer's own cloud account*. The customer can mandate "all inference goes through our Azure OpenAI deployment in West Europe" and the vendor's gateway routes calls accordingly.

**Latency envelope.** Same region as the customer when the customer requests it. Typically 50-200ms better than Mode 1 for same-region traffic; comparable to Mode 1 for cross-region.

**Update cadence.** Negotiated. Weekly or biweekly with named freeze windows is typical; the customer's operations lead holds a veto on specific release windows. The vendor still runs the pipeline but publishes release notes to the customer before any production push.

**Support model.** Vendor-paged with a customer-side Tier 1 for triage. SLA is a per-pilot addendum — the master SLA with availability / latency / incident-response floors tightened for this customer's workload.

**What the FDE physically ships.** The same client SDK as Mode 1, plus a tenant provisioning record (dedicated namespace / account identifiers, dedicated KMS alias, dedicated flag environment ID, dedicated log destination), SSO configuration, per-tenant admin surface for flag-flip visibility, and a per-tenant SLA addendum. Still no infrastructure in the customer's own account, but the vendor's gateway is pinned to use the customer's model-provider account credentials when configured.

**When it is right.** Mid-sensitivity workflows where the customer wants isolation and audit-stream segregation but does not require their own runtime; regulated verticals satisfied by signed sub-processor DPAs / BAAs with BYOK; EU customers under the EU Data Boundary (e.g., [Microsoft EU Data Boundary](https://learn.microsoft.com/en-us/privacy/eudb/eu-data-boundary-learn)) where the vendor can prove in-region containment.

**When it is wrong.** Workflows under an ATO that explicitly requires the runtime inside the agency's boundary; HIPAA workflows where the customer's compliance office rejects any sub-processor they have not individually BAA'd; anywhere the CISO has written "our VPC" into the security standard. Not air-gap-compatible.

## Mode 3 — VPC install (Terraform / Helm shipped by the FDE)

**Who runs it.** The customer. Their CI/CD applies the Terraform module and Helm chart the FDE ships; the runtime executes in their VPC, their Kubernetes cluster, under their IAM, writing logs to their SIEM and KMS-encrypting state with their keys.

**Model choice.** Hyperscaler-native under the customer's own account — AWS Bedrock, Azure OpenAI Service, GCP Vertex AI — or self-hosted open weights on customer GPUs via vLLM / TGI / NIM / Triton. Hosted frontier APIs are reachable only if the customer's egress policy allows a signed path to them, which is uncommon in regulated verticals.

**Latency envelope.** Excellent. Same VPC and often same AZ as the customer's application tier; LLM-call latency is dominated by the model provider, not transit. Gateway-to-model round trip can be single-digit milliseconds when both are in the same region of the same hyperscaler.

**Update cadence.** Customer change-management-driven. Monthly or quarterly with CAB approval is typical in regulated verticals ([Chapter 03](./03-feature-flag-platform-and-cab.md) wires the CAB integration). Emergency patches route through the customer's expedited-change path.

**Support model.** Split — customer-side Tier 1 and Tier 2 on the pager; vendor FDE on Tier 3 under a named SLA ([Chapter 07](./07-handoff-ceremony-and-sla.md) writes the packet). The customer's SRE owns the runtime; the vendor owns the artefact.

**What the FDE physically ships.** A versioned Terraform module, a versioned Helm chart, a release-notes document, a runbook (mod-109 Chapter 06), a security-review packet (mod-108 Chapter 02), and an ML-BOM slice (mod-108 Chapter 06). All of it in a signed, retrievable artefact bundle (mod-109 Chapter 05).

### Terraform module structure

Flat, auditable, one responsibility per file. The customer's platform team must be able to read every resource it will create.

```
modules/pilot-vpc-install/
├── main.tf              # Composition — pulls in the other files; no resources of its own.
├── variables.tf         # Customer-parameterised inputs — see list below.
├── outputs.tf           # IAM role ARNs, service endpoints, KMS key ID, log-group ARN.
├── versions.tf          # Terraform + provider version pins (aws >= 5.x, kubernetes, helm, tls).
├── locals.tf            # Naming conventions, tags, derived identifiers.
├── iam.tf               # IRSA role for the pod, model-provider invocation role, log-writer role.
├── networking.tf        # Security groups, VPC endpoints for Bedrock / S3 / KMS / Secrets Manager.
├── kms.tf               # Customer-managed key references + grants; no key creation unless opted in.
├── kubernetes.tf        # Namespace, resource quotas, network policies, pull secrets.
├── bedrock.tf           # (AWS variant) Bedrock model-access attachments, guardrail association.
├── azure_openai.tf      # (Azure variant) Cognitive Services account reference, private endpoint.
└── README.md            # Version, supported upgrade paths, variable reference, example tfvars.
```

Variables the customer parameterises — every one of these is a value the customer's platform team must supply, and every one of them changes per customer:

```hcl
# variables.tf (excerpt)
variable "region"                   { type = string }     # e.g., "us-east-1" or "us-gov-west-1"
variable "vpc_id"                   { type = string }
variable "subnet_ids"               { type = list(string) }
variable "kms_key_arn"              { type = string }     # customer-managed key, pre-existing
variable "oidc_issuer_url"          { type = string }     # EKS/AKS/GKE OIDC issuer for IRSA/WIF
variable "model_provider_role_arn"  { type = string }     # cross-account role to Bedrock, if used
variable "cluster_name"             { type = string }
variable "namespace"                { type = string }
variable "sso_provider"             { type = string }     # "okta" | "entra" | "ping"
variable "log_destination"          { type = string }     # SIEM log-group / Event Hub / Pub/Sub topic
```

### Helm chart layout

```
charts/pilot-app/
├── Chart.yaml           # name, version (semver), appVersion, dependencies (gateway, retriever).
├── values.yaml          # Defaults; no secrets inline (see anti-patterns).
├── values.schema.json   # JSON Schema for values; makes misconfiguration fail at `helm lint`.
└── templates/
    ├── deployment.yaml         # Workload Deployment(s) — app, gateway, retriever sidecar.
    ├── service.yaml            # ClusterIP Services.
    ├── configmap.yaml          # Non-secret runtime config (prompts path, model routing).
    ├── secret.yaml             # External-Secrets Operator / Secrets Store CSI references only.
    ├── hpa.yaml                # HorizontalPodAutoscaler per workload.
    ├── networkpolicy.yaml      # Egress to KMS / Bedrock / SIEM only; deny by default.
    ├── serviceaccount.yaml     # IRSA / Workload Identity annotations.
    ├── poddisruptionbudget.yaml
    └── servicemonitor.yaml     # Prometheus Operator scrape config.
```

A representative `values.yaml` — short, no secrets, every pointer named:

```yaml
# values.yaml (excerpt — kept under 25 lines on purpose)
image:
  repository: registry.vendor.ai/pilot-app
  tag: "v2.3.1-stable"
  pullPolicy: IfNotPresent
replicaCount: 3
resources:
  requests: { cpu: "500m", memory: "1Gi" }
  limits:   { cpu: "2",    memory: "4Gi" }
gateway:
  provider: bedrock            # bedrock | azure_openai | vertex_ai | vllm
  region: us-east-1
  modelId: anthropic.claude-3-5-sonnet-20241022-v2:0
  retention: zero              # routed through gateway per mod-108 Chapter 05
auth:
  oidcIssuer: "https://oidc.eks.us-east-1.amazonaws.com/id/EXAMPLE"
  serviceAccount:
    create: true
    annotations:
      eks.amazonaws.com/role-arn: "arn:aws:iam::123456789012:role/pilot-app"
observability:
  logs:  { destination: "cwlog://pilot-app-prod" }
  traces: { endpoint: "otlp://tempo.observability:4317" }
```

### Release notes shape

Every version ships a release-notes document; the CAB reads it before approving.

```yaml
artefact: pilot-app
version: v2.3.1-stable
channel: stable                        # stable | beta | rc
released: 2026-11-04
supported_upgrade_paths:               # which prior versions can upgrade to this one
  - from: v2.2.x
  - from: v2.3.0
rollback_path: v2.3.0                  # last known-good; see Chapter 05 artefact bundle
migration_notes: |
  Adds required variable `log_destination`. Customers upgrading from v2.2.x must
  supply the SIEM log-group ARN before applying. No data-plane migration.
breaking_changes: []
eval_delta:                            # see mod-107 Chapter 05 and Chapter 04 canary gate
  judge_pass_rate: "+0.4pp vs v2.3.0 on pilot gold set"
  p95_latency: "unchanged"
  cost_per_call: "-6% (prompt tightening)"
security_notes:
  ml_bom_diff: evidence/mlbom-diff-v2.3.0-to-v2.3.1.json
  cve_summary: none
```

### Versioning scheme

Semver plus release channel. `v<MAJOR>.<MINOR>.<PATCH>-<channel>` where channel is one of `stable`, `beta`, `rc<N>`. Chart.yaml's `version` tracks the chart; `appVersion` tracks the image. The Terraform module and the Helm chart have independent semver cycles but are pinned to one another in the artefact bundle manifest ([Chapter 05](./05-artefact-bundle-and-replay.md)).

**When it is right.** HIPAA, SR 11-7, DORA, SOC 2-strict customers with their own platform team; any customer whose CISO requires the runtime in the customer's VPC; any workload where the eval gold set contains data the customer will not permit to leave the boundary; any workload where the latency budget cannot absorb internet transit.

**When it is wrong.** Customers without a platform team willing to adopt a Terraform module and a Helm chart — the FDE ends up operating what was meant to be a customer-owned install. Also wrong when the workload is early-stage and the customer has not yet confirmed vertical regulatory posture; Mode 2 is cheaper to walk back from.

## Mode 4 — On-prem / air-gapped

**Who runs it.** The customer, with no network path to the vendor at runtime. The artefact arrives through an approved transfer process — physical media, a cross-domain diode, or a tightly-scoped data-transfer workflow. The runtime, the model weights, the retrieval index, the evaluation, the judge, and the logs all live inside the air gap.

**Model choice.** Local open-weights only. Served by [vLLM](https://docs.vllm.ai/), [Hugging Face Text Generation Inference (TGI)](https://huggingface.co/docs/text-generation-inference/), [Nvidia NIM](https://docs.nvidia.com/nim/), [Nvidia Triton Inference Server](https://docs.nvidia.com/deeplearning/triton-inference-server/), or [llama.cpp](https://github.com/ggerganov/llama.cpp) for CPU-first / edge. Hosted frontier APIs are unreachable by definition.

**Latency envelope.** Excellent and bounded. Network transit is zero; latency is whatever the local serving stack and hardware deliver. A quantised 70B-class open-weights model on a current-generation GPU node is in the same latency range as a hosted frontier API for short-context work. <!-- needs-research: cite current public benchmarks for vLLM / TGI / NIM latency on common hardware (H100 / L40S / A100) with common open-weights models — numbers drift and should not be fabricated -->

**Update cadence.** ATO-gated or regulatory-change-control-gated. Quarterly is fast; annual is common. Emergency security patches route through a POA&M item with expedited change-control; functional updates wait for the next ATO refresh.

**Support model.** Customer-paged. The vendor FDE is reachable through a defined consultation channel (often an hours-of-service hotline plus a ticket queue) but never on a live pager. The runbook ([Chapter 06](./06-admin-dashboard-runbooks-sme-training.md)) is the primary operational artefact.

**What the FDE physically ships.** An air-gapped bundle — model weights (or NIM container), application images, Helm chart, Terraform (if any cloud-adjacent infrastructure), SBOM, ML-BOM slice from [mod-108 Chapter 06](../mod-108-enterprise-security-and-compliance/06-ai-supply-chain-provenance.md), hash manifest, Sigstore attestations, release notes, and the runbook. Everything is content-addressed; everything is verifiable without network access.

```yaml
# bundle-manifest.yaml (excerpt)
bundle: pilot-app-airgap
version: v2.3.1-stable
channel: stable
sha256: 3b1f...e2a9                              # of the tarball itself
artefacts:
  - name: app-image
    oci: pilot-app:v2.3.1-stable
    digest: sha256:9c4a...1b77
  - name: nim-container                          # or vllm-image, tgi-image, llamacpp-binary
    oci: nvcr.io/nim/meta/llama-3.1-70b:1.0.3
    digest: sha256:a21b...77cc
  - name: weights
    path: weights/llama-3.1-70b-instruct.safetensors
    digest: sha256:0f8d...c411
  - name: helm-chart
    path: charts/pilot-app-2.3.1.tgz
    digest: sha256:71e2...9a04
  - name: sbom
    path: evidence/sbom.spdx.json
  - name: ml_bom
    path: evidence/mlbom.json                    # mod-108 Chapter 06
attestations:
  - type: sigstore-cosign
    path: evidence/attestations/cosign-bundle.json
transfer:
  method: approved-cross-domain                  # per customer process
  verified_by: customer-security-team
  hash_check: required
```

Local judge options matter here. Online evaluation ([mod-107 Chapter 05](../mod-107-eval-eng-on-customer-data/05-online-evaluation.md)) still runs, but the judge-LLM is a local model — a smaller open-weights model served from the same stack, with its prompts and thresholds pinned in the bundle. No external judge is reachable.

No-egress discipline is enforced architecturally, not by policy memo — the `networkpolicy.yaml` from the Helm chart denies egress by default; the only approved endpoints are the local model-serving Service, the local retrieval index, the local SIEM, and the local KMS (often HSM-backed). The gateway from [mod-104 Chapter 03](../mod-104-llm-app-in-customer-envs/03-model-gateway-in-customer-env.md) is still the attachment point for the data contract — zero-retention, residency, training-opt-out — even though all three are trivially satisfied by the air gap.

**When it is right.** US federal classified or CUI environments with no cloud path; defence industrial base; some healthcare and financial deployments with hard-air-gap mandates; edge deployments where connectivity is genuinely absent.

**When it is wrong.** Anywhere the customer's operations team cannot staff a local on-call rotation. Anywhere the workload requires a frontier-class model that is not available as open weights. Anywhere the update cadence the business needs is faster than ATO change-control can approve.

## Customer-side inference options the FDE must understand

Each deployment mode narrows the usable set. This section is the reference — the Mode 1-4 sections above call out the cases.

- **[AWS Bedrock](https://docs.aws.amazon.com/bedrock/)** inside the customer's account. Model availability varies by region; the authoritative list is [Models supported by Region](https://docs.aws.amazon.com/bedrock/latest/userguide/models-regions.html). The FDE must never assume a given model ID is available in a given customer region — it is a specific lookup every time. <!-- needs-research: confirm model-availability-by-region for the specific models used by Archetype A (us-east-1 and us-west-2) at the time the pilot is scoped — this changes continuously -->
- **[Azure OpenAI Service](https://learn.microsoft.com/en-us/azure/ai-services/openai/)** under the customer's subscription. Includes [Azure Government](https://learn.microsoft.com/en-us/azure/azure-government/) for federal archetypes and runs under the EU Data Boundary when configured for EU regions. Model availability and quota are both per-region and per-subscription. <!-- needs-research: confirm current Azure OpenAI Service model and region matrix for Archetype C (EU Data Boundary) at scoping time -->
- **[GCP Vertex AI Generative AI](https://cloud.google.com/vertex-ai/generative-ai/docs/overview)** in the customer's project. Supports hosted Google models (Gemini family) and third-party models (Anthropic, Meta) through the Model Garden; residency and CMEK are per-region. <!-- needs-research: confirm current Vertex AI Model Garden availability for third-party models in EU and US-only regions -->
- **[Nvidia NIM](https://docs.nvidia.com/nim/)** for containerised inference of open-weights models on customer GPUs — Bedrock-alternative for the self-hosted path, usable inside a VPC install or air-gapped bundle.
- **[vLLM](https://docs.vllm.ai/)** for high-throughput self-hosted inference with PagedAttention and continuous batching; the most common choice for VPC-install and air-gapped deployments of open-weights models.
- **[Hugging Face Text Generation Inference (TGI)](https://huggingface.co/docs/text-generation-inference/)** as a vLLM alternative with strong HF ecosystem integration.
- **[llama.cpp](https://github.com/ggerganov/llama.cpp)** for CPU-first, edge, or air-gapped deployments where GPU is not available. Lower throughput; higher deployment flexibility.
- **[Nvidia Triton Inference Server](https://docs.nvidia.com/deeplearning/triton-inference-server/)** as the multi-model, multi-framework serving layer — commonly paired with NIM or wrapping vLLM / TGI backends for ensembles.

Mode 1 and 2 can reach frontier APIs and any hyperscaler option; Mode 3 can reach hyperscaler-native under customer keys and any self-hosted option; Mode 4 can reach self-hosted only.

## Running the three archetypes

**Archetype A — US mid-size healthcare payer (VPC install).** AWS Bedrock inside the customer's account, in `us-east-1` and `us-west-2` on EKS. The FDE ships the Terraform module above with `bedrock.tf` populated, the Helm chart with `gateway.provider: bedrock` in `values.yaml`, and release notes with monthly cadence. Model is whatever the member-inquiry workflow's eval gate from [mod-107](../mod-107-eval-eng-on-customer-data/) pinned; the Bedrock model ID is checked against the region-availability table at scoping time. BYOK uses customer-managed KMS keys referenced in `kms.tf`; the gateway's zero-retention attestation (mod-108 Chapter 05) is honoured by routing all calls through Bedrock with retention explicitly disabled. CAB meets monthly; Tier 1/2 is the customer's platform team, Tier 3 is the vendor FDE.

**Archetype B — US federal agency (on-prem / air-gapped).** GovCloud for cloud-adjacent components (SSO, SIEM integration if the agency allows it); self-hosted inference on the agency's GPUs via a vLLM or Nvidia NIM stack for the actual model calls. The air-gapped bundle ships through the agency's approved transfer process with the hash manifest above; the local judge is a smaller open-weights model served from the same stack. Update cadence is tied to the system's ATO baseline — quarterly if the ATO permits, with emergency patches tracked as POA&M items. Pager is agency-side; the FDE consults through a defined channel. Model-availability constraints from AWS Bedrock / Azure OpenAI do not apply — the only constraint is "which open-weights model fits the GPUs and passes the pilot eval."

**Archetype C — EU retail bank (dedicated SaaS).** [Azure AI Foundry](https://learn.microsoft.com/en-us/azure/ai-foundry/) tenancy under the customer's subscription in an EU region, pinned inside the EU Data Boundary with CMK on all storage, audit stream to Microsoft Sentinel. The vendor runs the pipeline; the artefact lands in a bank-dedicated namespace and the vendor's LaunchDarkly tenant streams flag-flip events into Sentinel. Weekly release cadence with named freeze windows; the CAB meets weekly and reviews the release notes. The eval gate includes an SR 11-7-equivalent model-risk re-test. Support is vendor-paged with customer-side Tier 1 for triage.

## Anti-patterns

- **SaaS-first by habit.** The pilot is scoped against Mode 1 because "it's the fastest" before the customer's residency and vertical regulatory posture are confirmed. The six-dimensional budget treats deployment-mode fit as a gate; the gate must be evaluated before the mode is picked, not after.
- **VPC install shipped as a bash script.** A README with twelve `kubectl apply` commands is not a shippable artefact. The customer's CAB cannot read it, the customer's platform team cannot version it, and the customer's auditor cannot diff it. Mode 3 means Terraform plus Helm, both semver-pinned, both in the artefact bundle.
- **Air-gapped cadence promised but not architected.** The pilot commits to monthly updates on an air-gapped system whose ATO change-control cycle is quarterly. The commitment collapses on the first release window. Mode 4 cadence is governed by the customer's regulatory framework, not the vendor's roadmap.
- **Model-choice decision separated from deployment-mode decision.** The team picks a frontier-API model in a Mode 3 install, discovers the customer's egress policy blocks the model provider, and rewrites the integration against Bedrock or Vertex AI two weeks before go-live. The two decisions are *one decision*; this chapter couples them in every section for a reason.
- **Support-model never written down.** "The customer owns Tier 1" is an agreement until the first production incident, at which point it is a dispute. The pager tiers belong in the handoff packet ([Chapter 07](./07-handoff-ceremony-and-sla.md)) and the pager protocol ([Chapter 08](./08-peer-boundaries-and-pager-protocol.md)) — in writing, with named contacts and response-time SLAs.
- **Helm values with secrets inline.** The chart's `values.yaml` contains a model-provider API key, a KMS key alias, a SIEM credential. The customer's platform team cannot adopt the chart without rewriting it; the vendor's git history leaks credentials on every tag push. Secrets reference external secret stores ([External Secrets Operator](https://external-secrets.io/), [Secrets Store CSI Driver](https://secrets-store-csi-driver.sigs.k8s.io/), [HashiCorp Vault](https://developer.hashicorp.com/vault), AWS Secrets Manager / Azure Key Vault / GCP Secret Manager) and are never committed to the chart.

## Summary

- Deployment-mode fit is a gate in the six-dimensional budget; this chapter names the four modes (SaaS multi-tenant, dedicated SaaS, VPC install, on-prem / air-gapped) and the four axes they trade against (latency, model choice, update cadence, support model).
- Model-choice and deployment-mode are one decision, not two. Mode 1/2 can reach frontier APIs and hyperscaler-native options; Mode 3 can reach hyperscaler-native under customer keys and self-hosted; Mode 4 can reach self-hosted open-weights only via vLLM / TGI / NIM / Triton / llama.cpp.
- The gateway from [mod-104 Chapter 03](../mod-104-llm-app-in-customer-envs/03-model-gateway-in-customer-env.md) is the common attachment point for the data contract from [mod-108 Chapter 05](../mod-108-enterprise-security-and-compliance/05-data-contract.md) across all four modes — zero-retention, residency, BYOK, training-opt-out.
- Mode 3 is the mode with the most packaging discipline: a flat Terraform module (`main.tf`, `variables.tf`, `outputs.tf`, `versions.tf`, `locals.tf`, `iam.tf`, `networking.tf`, `kms.tf`, `kubernetes.tf`, `bedrock.tf` or `azure_openai.tf`, `README.md`), a Helm chart with `Chart.yaml` / `values.yaml` / templated Deployments / Services / ConfigMaps / Secrets-via-reference / HPA / NetworkPolicy / ServiceAccount, release notes with supported upgrade paths and rollback path, and semver-plus-channel versioning (`v2.3.1-stable`, `v2.4.0-rc1`).
- Mode 4 ships an air-gapped bundle with a hash manifest, SBOM, ML-BOM slice, and Sigstore attestations; the local judge, local retrieval, and local serving stack all live inside the air gap; update cadence is ATO or regulatory-change-control-gated.
- Three archetypes thread the module: Archetype A is Mode 3 on AWS Bedrock (EKS in `us-east-1` + `us-west-2`), Archetype B is Mode 4 with vLLM or NIM on agency GPUs, Archetype C is Mode 2 on Azure AI Foundry under the EU Data Boundary.
- Six anti-patterns to catch early: SaaS-first by habit, VPC install as a bash script, air-gapped cadence promised but not architected, model-choice and deployment-mode treated as separate decisions, support-model not written down, Helm values with secrets inline.
- Chapter 03 picks up the feature-flag platform and CAB integration against these four modes; [Chapter 05](./05-artefact-bundle-and-replay.md) versions the full per-customer bundle (Terraform + Helm + prompts + retriever + model pin + tool allow-list + judge + guardrails + gateway config) as a single unit; [Chapter 07](./07-handoff-ceremony-and-sla.md) indexes all of it in the handoff packet.
