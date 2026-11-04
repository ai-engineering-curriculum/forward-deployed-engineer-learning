# Chapter 03 — The LLM Observability Platform at the Customer's Compliance Posture

## Why this chapter exists

Chapter 02 shipped OpenTelemetry traces, metrics, and logs into the customer's existing APM — Datadog, Splunk, Elastic, Grafana, Dynatrace, New Relic, Sentinel. That machinery answers "is the pilot alive and are tokens / cost / latency inside their envelopes?" What it does not answer is the LLM-specific question the SME and the model-risk reviewer open every morning: *"Show me the twenty worst traces from yesterday. Why did this completion disagree with the gold set? Which prompt-template version did it run on? What did the judge say? Can I annotate it as a bad answer for the next eval run?"*

That is the **LLM observability platform**'s job. It is a second-tier store, specialised for LLM traces: full prompts and completions, user-and-SME feedback annotations, judge-score rollups, prompt-template versioning as a first-class object, dataset / experiment management that ties into the eval harness ([mod-107 Chapter 03](../mod-107-eval-eng-on-customer-data/03-offline-eval-design.md)). Five serious options at publication: **Langfuse, LangSmith, Arize Phoenix, Weights & Biases Weave, Braintrust**. The right choice is a function of the customer's compliance posture, the deployment archetype, the vendor's existing tenancy, and — for air-gapped customers — whether the platform can run entirely inside the customer's enclave.

This chapter has four jobs. Name the five platforms and what they commit to. Specify the decision matrix for choosing between them per archetype. Walk through the self-hosted deployment pattern for the air-gapped and VPC-install archetypes. Define the daily-dashboard discipline that keeps the customer-side operator opening the platform every day (which is the whole point).

## What an "LLM observability platform" commits to

A working definition: a datastore + UI + API that treats one LLM invocation as a first-class object with these properties.

- **Trace shape.** Hierarchical spans — generation, retrieval, tool call, judge, rerun — carrying the full prompt and completion along with structured metadata.
- **Content capture.** Native support for storing full prompts, completions, retrieved documents, tool arguments / results.
- **User and SME feedback.** Thumbs-up / thumbs-down, free-text annotations, structured scores. Ideally feedback writes back to the trace it refers to.
- **Prompt-template versioning.** Templates versioned as code; traces carry the template version they ran on; the UI supports diff and promotion.
- **Dataset and experiment management.** Trace-to-dataset conversion; dataset / fixture storage; experiment runs that replay a dataset against a new prompt / model / retriever.
- **Judge integration.** LLM-as-judge rubrics, judge-call tracing, score rollups.
- **Export to the eval harness.** The regression-fixture back-feed from Chapter 06 of this module and the gold-set builder from [mod-107 Chapter 02](../mod-107-eval-eng-on-customer-data/02-customer-gold-set-bootstrap.md) both land here.

A generic APM (Datadog, Splunk, etc.) can do pieces of this through configuration, but doing the whole set well requires the specialised product.

## The five platforms

Covered with the properties the FDE will be asked about: hosting model, licence, major integrations, posture-relevant facts. Every statement here is marked for verification at publication — these products ship at a speed that outpaces any static chapter.

### Langfuse

- **Website.** [langfuse.com](https://langfuse.com/).
- **Licence.** Open source (MIT for the core, with an Enterprise Edition available under a commercial licence). <!-- needs-research: confirm current licence split between OSS / EE at publication time -->
- **Hosting models.** Fully self-hostable via Docker Compose or Helm chart; managed cloud offering (EU and US regions) available. <!-- needs-research: confirm Langfuse Cloud region availability and the specifics of EU-region data handling -->
- **OTel integration.** Accepts OTLP natively; can act as a backend for GenAI OTel traces.
- **Notable.** First-class prompt-template versioning as a managed object; dataset-and-experiment UI; feedback-score API; integrations with the major Python / TypeScript LLM clients and frameworks.
- **Compliance.** The managed cloud is SOC 2 reported; the self-hosted mode puts compliance posture entirely in the customer's hands (which is exactly the point for the HIPAA / FedRAMP / DORA archetypes). <!-- needs-research: confirm current Langfuse Cloud SOC 2 / HIPAA / BAA posture -->

### LangSmith

- **Website.** [smith.langchain.com](https://smith.langchain.com/).
- **Licence.** Proprietary; part of the LangChain commercial offering.
- **Hosting models.** Managed cloud; self-hosted enterprise deployment ("LangSmith Self-Hosted" via Docker / Kubernetes) available under an enterprise contract. <!-- needs-research: confirm current self-hosted deployment support and the regions / clouds covered -->
- **OTel integration.** OTLP support; also native LangChain-tracing SDK.
- **Notable.** Tight coupling to LangChain / LangGraph; evaluation + dataset management surface; prompt versioning via LangChain Hub integration.
- **Compliance.** Commercial offering publishes SOC 2 posture; HIPAA via enterprise agreement; EU hosting addressed through the enterprise plan. <!-- needs-research: confirm current LangSmith compliance posture (SOC 2, HIPAA BAA, EU residency, FedRAMP status) -->

### Arize Phoenix

- **Website.** [phoenix.arize.com](https://phoenix.arize.com/).
- **Licence.** Open source (Elastic License 2.0 for Phoenix OSS). <!-- needs-research: confirm current licence -->
- **Hosting models.** Fully self-hostable (Docker, Helm, pip-installable local mode); managed cloud via Arize AX platform for broader use. <!-- needs-research: confirm Phoenix managed-hosting situation at publication time -->
- **OTel integration.** First-class — Phoenix was designed around OpenTelemetry's trace model; `openinference` instrumentation libraries (an Arize-led, OpenTelemetry-compatible instrumentation set) export directly.
- **Notable.** Strong for retrieval analysis (embeddings drift, clustering, troubleshooting RAG specifically); Phoenix OSS is the lightest-weight option to spin up inside an air-gapped enclave.
- **Compliance.** Arize commercial product publishes SOC 2; HIPAA via Arize AX enterprise agreement; FedRAMP-relevant deployments typically done via the self-hosted OSS variant inside the agency's enclave. <!-- needs-research: confirm Arize AX FedRAMP status and Arize's HIPAA posture -->

### Weights & Biases Weave

- **Website.** [wandb.ai/site/weave](https://wandb.ai/site/weave).
- **Licence.** Proprietary (W&B commercial); the parent W&B platform has an OSS client SDK but the hosted trace store is commercial.
- **Hosting models.** Managed cloud; dedicated cloud (single-tenant managed); customer-managed (on-prem / self-hosted) available under the W&B Enterprise agreement. <!-- needs-research: confirm Weave on-prem deployment options and current supported clouds -->
- **OTel integration.** OTLP support; native W&B tracing SDK. <!-- needs-research: confirm current OTLP ingestion support in Weave -->
- **Notable.** Deep integration with the W&B experiment-tracking platform (datasets, models, runs) that ML / data-science teams already use; useful when the customer's own data-science team is a stakeholder.
- **Compliance.** W&B's dedicated cloud offers SOC 2 and HIPAA postures; FedRAMP posture via the customer-managed self-hosted deployment. <!-- needs-research: confirm W&B's current HIPAA BAA / FedRAMP / IL5 posture -->

### Braintrust

- **Website.** [braintrust.dev](https://www.braintrust.dev/).
- **Licence.** Proprietary SaaS; self-hosted deployment available to enterprise customers. <!-- needs-research: confirm Braintrust self-hosted enterprise deployment specifics at publication time -->
- **Hosting models.** Primarily SaaS; enterprise self-hosted available.
- **OTel integration.** OTLP support; proprietary SDK for richer features. <!-- needs-research: confirm OTLP support -->
- **Notable.** Strong evaluation framework; prompt-and-playground tooling; opinionated workflow for iterating prompts against datasets.
- **Compliance.** SOC 2 published on the trust centre; HIPAA / EU posture via enterprise agreement. <!-- needs-research: confirm Braintrust's current compliance posture -->

## The decision matrix

Choosing between the five is driven by seven axes. The FDE scores each option against each axis for the specific pilot.

| Axis | What to ask | Why it matters |
|---|---|---|
| **Self-host-ability** | Does it deploy inside the customer's enclave with no external egress? | Air-gapped customers and some VPC-install customers require this; it is the single most common gate. |
| **Customer-managed KMS / CMK** | Does the storage layer support customer-held keys? | HIPAA, FedRAMP, DORA, and most CISO shops write this into the data contract. |
| **Customer-managed IdP (SSO / SCIM)** | Does the platform federate with Okta / Entra ID / Ping / Keycloak? | The customer's IdP, with their MFA, is non-negotiable in enterprise ([mod-103 Chapter 03](../mod-103-enterprise-integration/)). |
| **Retention controls** | Can retention be set per-store to match the customer's regulatory floor? | HIPAA six years, DORA, EU AI Act Article 72, FedRAMP; one global retention setting is rarely enough. |
| **Prompt / dataset / experiment APIs** | Does the eval harness push and pull via a documented API? | Chapter 06's back-feed depends on this. |
| **OTLP support** | Does it accept OTLP-shaped GenAI spans natively? | Chapter 02 produced OTLP; a second non-OTLP SDK path is a tax. |
| **Fit with vendor's existing tenancy** | Does the vendor already run the platform for other pilots? | The second pilot on the same platform is far cheaper than the first; standardising across the fleet is `ai-platform-engineer`'s preference. |

An illustrative per-archetype scoring (shape; the actual table is re-run with current product data per pilot):

| Axis | Archetype A (healthcare, VPC install) | Archetype B (federal, air-gapped) | Archetype C (bank, dedicated-SaaS on Azure) |
|---|---|---|---|
| Self-host required | Yes | Yes (strict) | Yes |
| CMK on storage | Yes | Yes | Yes |
| Customer IdP | Okta | Agency IdP | Entra ID |
| Retention | 6 years (HIPAA) | NARA + agency | 7 years (DORA / financials) |
| Eval API | Yes | Yes | Yes |
| OTLP | Yes | Yes | Yes |
| Vendor existing tenancy | Langfuse OSS standard | Phoenix OSS standard | Langfuse OSS standard |
| Decision | **Langfuse self-hosted** | **Arize Phoenix self-hosted** | **Langfuse self-hosted (Azure)** |

The scores above are illustrative. The FDE re-runs the matrix each pilot because vendor postures change. Any score written at pilot-week-two should be re-verified at month six.

<!-- needs-research: all five platform entries require per-pilot verification against the vendor's current trust centre at the time of selection -->

## Deploying self-hosted Langfuse — the common case

The exercise walks through Langfuse specifically because it is the most common open-source choice and the pattern transfers to Phoenix and Weave.

### Baseline architecture

- **App tier.** Langfuse web + worker containers running in Kubernetes (Helm chart) or Docker Compose for small pilots.
- **Postgres.** Metadata, project / organisation / user records, structured scores. CMK-encrypted. HA via RDS Multi-AZ, Azure Flexible Server HA, or on-prem HA Postgres.
- **ClickHouse.** Traces, observations, scores at scale. CMK-encrypted. HA via managed service (ClickHouse Cloud is a SaaS, so for self-host the customer runs their own cluster) or self-managed.
- **Object storage.** S3 / Azure Blob / MinIO for large payloads and media. CMK-encrypted.
- **Redis / Valkey.** Queue / cache. <!-- needs-research: confirm Langfuse v3 architecture includes all these components as of publication -->

### Deployment under Archetype A — HIPAA VPC install

Helm chart (shape; the official chart is the source of truth):

```yaml
# values-archetype-a.yaml for Langfuse Helm chart
langfuse:
  web:
    replicaCount: 3
    ingress:
      enabled: true
      hosts:
        - host: langfuse.pilot.customerdomain.internal
      tls:
        - secretName: langfuse-tls
    oidc:
      enabled: true
      issuer: https://customer.okta.com
      clientIdSecretRef: {name: langfuse-oidc, key: clientId}
      clientSecretSecretRef: {name: langfuse-oidc, key: clientSecret}
    allowDefaultSignInProvider: false
  worker:
    replicaCount: 2

postgresql:
  deployed: false  # using customer-managed RDS

externalPostgresql:
  host: langfuse-db.cluster-<customer>.rds.amazonaws.com
  storageEncryption:
    enabled: true
    kmsKeyAlias: alias/customer-pilot-hipaa

clickhouse:
  deployed: false  # customer runs managed ClickHouse

externalClickhouse:
  host: clickhouse.pilot.customerdomain.internal
  kmsKeyAlias: alias/customer-pilot-hipaa
  retention:
    traces: 2190d  # 6 years
    scores: 2190d
    events: 2190d

s3:
  bucket: pilot-langfuse-blobs-{{tenant}}
  region: us-east-1
  serverSideEncryption:
    kmsMasterKeyId: alias/customer-pilot-hipaa

networkPolicy:
  enabled: true
  egressAllow:
    - from: langfuse-worker
      to: cidr(10.0.0.0/8)  # intra-VPC only
    - from: langfuse-web
      to: cidr(10.0.0.0/8)

serviceAccount:
  irsaArn: arn:aws:iam::<account>:role/langfuse-sa

telemetry:
  anonymousUsage: false  # disable outbound telemetry to Langfuse
  remoteConfig: false
```

The YAML above is illustrative — the official Helm chart's values schema is the ground truth. Four properties the FDE enforces regardless of how the YAML spells them:

1. **No outbound telemetry.** Many open-source products ship anonymous-usage pings by default; HIPAA / FedRAMP / DORA workloads turn them off.
2. **CMK on every store.** Postgres / ClickHouse / S3 all encrypt with the customer's KMS key alias; the key policy restricts use to the pilot's IAM role.
3. **Egress restricted.** NetworkPolicies and VPC security groups enforce intra-VPC egress only; no call home.
4. **SSO forced.** Default sign-in providers (email+password) are disabled; only the customer's OIDC / SAML IdP is accepted; SCIM provisions / de-provisions users.

### Deployment under Archetype B — air-gapped federal agency

Phoenix OSS is the typical pick (lighter weight, single-pod viable for a small pilot). The deployment pattern:

- Phoenix container image mirrored into the agency's internal registry (per the [mod-108 Chapter 06](../mod-108-enterprise-security-and-compliance/06-ai-supply-chain-provenance.md) supply-chain discipline — Sigstore-verified at mirror time).
- Phoenix running in the agency's on-prem Kubernetes with a Postgres backend inside the enclave.
- OIDC against the agency's IdP; no local account provisioning.
- OTel Collector from Chapter 02 exports to Phoenix's OTLP endpoint.
- Retention per the agency's records schedule; archive to on-prem object storage.
- No outbound network access at all; the agency's egress policy blocks it.

Shape-only Phoenix deployment, Docker Compose form for a small-pilot trial inside the enclave (the production deployment would be Kubernetes):

```yaml
version: "3.8"
services:
  phoenix:
    image: registry.enclave.agency/arize-phoenix:${PHOENIX_VERSION}
    environment:
      PHOENIX_SQL_DATABASE_URL: postgresql+psycopg://phoenix:${PG_PASSWORD}@phoenix-db:5432/phoenix
      PHOENIX_PORT: "6006"
      PHOENIX_GRPC_PORT: "4317"
      PHOENIX_ENABLE_AUTH: "true"
      PHOENIX_OIDC_ISSUER: https://idp.enclave.agency
      PHOENIX_OIDC_CLIENT_ID: ${OIDC_CLIENT_ID}
      PHOENIX_OIDC_CLIENT_SECRET_FILE: /run/secrets/oidc_client_secret
      PHOENIX_ENABLE_PROMETHEUS: "true"
    ports:
      - "6006:6006"
      - "4317:4317"
    secrets:
      - oidc_client_secret
    read_only: true
    tmpfs:
      - /tmp
    networks:
      - enclave-internal
  phoenix-db:
    image: registry.enclave.agency/postgres:16
    environment:
      POSTGRES_DB: phoenix
      POSTGRES_USER: phoenix
      POSTGRES_PASSWORD: ${PG_PASSWORD}
    volumes:
      - phoenix-pg:/var/lib/postgresql/data
    networks:
      - enclave-internal
volumes:
  phoenix-pg:
secrets:
  oidc_client_secret:
    file: ./secrets/oidc_client_secret
networks:
  enclave-internal:
    internal: true
```

<!-- needs-research: confirm Phoenix environment variable names and image path format in the current release; these have changed between versions -->

### Deployment under Archetype C — Azure dedicated-SaaS

Langfuse self-hosted in the bank's Azure tenancy under the EU Data Boundary. Key differences from Archetype A:

- Managed services: Azure Database for PostgreSQL Flexible Server; Azure Blob with customer-managed keys in Azure Key Vault.
- ClickHouse: an Azure-region-pinned cluster (self-managed or via a third-party provider operating in-region).
- Private endpoints only; no public IP on any component.
- Entra ID OIDC with conditional access requiring compliant devices and MFA.
- Sentinel receives a diagnostic-settings pipe for the Langfuse application logs (not the trace payload — that stays in the Langfuse ClickHouse store).
- Retention seven years per the bank's regulated-financials policy. <!-- needs-research: confirm DORA + bank-sector retention requirements for this specific jurisdiction -->

## When the managed cloud is the right answer

Self-hosted is correct for Archetypes A / B / C; it is not correct for every customer. The managed cloud of each platform is right when:

- The customer's data contract permits the vendor to be a sub-processor for prompts and completions, with BAA / DPA / SCCs in place.
- The customer has no CMK requirement, or the managed offering supports it.
- The customer's IdP federates with the managed offering.
- The retention windows available in the managed offering meet the customer's regulatory floor.
- The customer's residency requirement maps to a region the managed offering serves.

For a non-regulated SaaS multi-tenant Archetype (not one of our three), the managed cloud is often the fast path — shipping a Langfuse Cloud or LangSmith workspace in the first week accelerates time-to-first-trace by weeks.

## The daily-dashboard discipline

The LLM observability platform's point is *daily use by the customer-side operator*. If it is a tool the FDE opens and no one else does, the pilot's operability is weak. Four dashboards the operator opens daily, each with named owner and named action.

**Dashboard 1 — Worst traces of the last 24 hours.** Filtered to: judge-score below the gate threshold, finish-reason containing `safety` / `content_filter`, user feedback `thumbs_down`, cost-per-call above the per-call budget. Top 20 by severity. Owner: customer operations analyst. Action: triage — triage the trace, add an annotation (`real_regression` / `expected` / `judge_mistake`), and if `real_regression`, open a bug.

**Dashboard 2 — SLO burn rates.** Error-budget burn for each of the four SLO categories (quality / latency / cost / safety; see Chapter 04). Owner: customer operations lead. Action: when any budget crosses 50 %, open the error-budget-policy conversation; when any crosses 75 %, follow the slow-burn page policy from Chapter 05.

**Dashboard 3 — Rollout / experiment health.** Any active canary, shadow, or A/B arm from [mod-109 Chapter 04](../mod-109-deployment-rollout-change-management/04-canary-shadow-ab-rollouts.md); the six-dimensional canary gate current readings. Owner: customer release manager. Action: approve expansion, hold, or roll back per the canary gate policy.

**Dashboard 4 — Cost by tenant / model / workflow.** Rolling 24-hour and 30-day cost broken down by tenant, by model, by prompt-template version, by rollout arm. Owner: customer FinOps or operations lead. Action: investigate any tenant whose rolling-24h cost exceeds 2× its 30-day mean.

A dashboard the operator cannot read is a failure. The dashboard names, legends, and metric labels use the customer's taxonomy, not the vendor's internal naming. "Member Assist case triage" beats "pilot-agent.answer_member_inquiry" on the operator's screen.

## Prompt-template versioning as a first-class object

The LLM observability platform's prompt-template management is what closes the gap between "which prompt version did this answer come from?" and "what did we change from v3.2 to v3.3 and did it move the judge score?"

Three disciplines.

1. **Every production prompt is versioned in the platform.** The pilot's code loads prompts by name + version (never as string literals). A change to a prompt is a new version; the artefact bundle manifest ([mod-109 Chapter 05](../mod-109-deployment-rollout-change-management/05-per-tenant-versioning-and-replay.md)) pins to the specific version.
2. **Every trace carries `bundle.prompt_template.version`.** From Chapter 02's attribute set; this is how the platform ties traces to templates.
3. **Promotion is a reviewed workflow.** Moving a prompt from `staging` to `production` is a two-person review inside the platform's UI; the diff and the eval-run against the gold set are both part of the review. The audit trail lands in the customer's SIEM.

## Dataset and experiment management for the eval back-feed

Chapter 06 of this module writes the back-feed in detail. The platform-side mechanics here:

- A trace flagged `real_regression` by the SME is converted (one-click) to a **dataset entry** with the input, the actual output, the SME's correction (if any), and the reason classification.
- The dataset is referenced by the eval harness ([mod-107 Chapter 03](../mod-107-eval-eng-on-customer-data/03-offline-eval-design.md)) at every CI run.
- An **experiment** runs a candidate prompt or model against the dataset; the result lands in the platform with per-case pass / fail and aggregate pass-rate. The canary gate ([mod-109 Chapter 04](../mod-109-deployment-rollout-change-management/04-canary-shadow-ab-rollouts.md)) reads the experiment result.

Every platform in the five supports this flow; the API shapes differ. The FDE wires the back-feed as part of pilot setup, not as an afterthought.

## Running the three archetypes

**Archetype A — US healthcare payer.** Langfuse self-hosted in the pilot's EKS cluster, AWS KMS on Postgres / ClickHouse / S3, Okta OIDC + SCIM, six-year retention on traces and scores, OTel Collector from Chapter 02 exports via OTLP, admin access to full content gated by break-glass + four-eyes. Daily dashboards opened by the member-service operations lead and a rotating analyst.

**Archetype B — US federal agency.** Phoenix OSS self-hosted on the agency's enclave Kubernetes, no outbound egress, agency IdP OIDC, retention per NARA + agency records policy, OTel Collector exports via OTLP over the enclave network. Phoenix is sized minimally — the pilot's volume is low; the primary user is the analyst-training team. <!-- needs-research: confirm Phoenix sizing guidance for a 1-10 QPS pilot -->

**Archetype C — EU retail bank.** Langfuse self-hosted in the bank's Azure tenancy under the EU Data Boundary, Azure Key Vault CMK on every store, Entra ID OIDC with conditional access, seven-year retention, OTel Collector exports via OTLP. Sentinel receives application-log diagnostic stream; the Langfuse content view is for the bank's model-risk and support-operations teams.

## Anti-patterns

**Using the vendor's managed cloud for a regulated pilot without a cleared data contract.** The vendor's Langfuse Cloud / LangSmith / Weave managed instance sits outside the customer's trust boundary; raw prompts stream into it; the customer's DPO discovers it in month three. Chapter 03's discipline is: a managed offering is used only when its trust boundary is formally cleared.

**Self-hosted platform as a one-pod Docker Compose.** The platform went live on a single pod with no backup, no monitoring, no restart policy; three months in, the Postgres volume fills and the trace history is lost. The LLM observability platform is a production-grade service with its own SLO; it gets the same architectural treatment as the pilot itself.

**Two platforms in parallel because the pilot could not choose.** Langfuse *and* Phoenix running; SMEs annotate in one, release managers look in the other; the data drifts. Pick one per pilot; standardise across the vendor's fleet with `ai-platform-engineer`'s help.

**Vendor FDE as the only user.** The FDE opens the platform; the customer operator never does. The platform is the customer's primary daily surface; if the operator does not open it by month three, the training was wrong.

**Prompt templates still in the application source.** `prompt = "You are a helpful assistant..."` in the Python file; the platform's template registry is empty; every incident requires reading the specific commit the app was deployed from. Chapter 03 insists on externalised, versioned prompts from day one.

**Content access without break-glass.** The platform's admin role has unrestricted read of raw prompts and completions; FDE and SME accounts both have the role; the data contract's "access logged and justified" clause is not enforced. Chapter 03 requires break-glass + four-eyes for raw-content view; the platform's audit log streams to the customer's SIEM.

**Managed-cloud retention set to 30 days on a HIPAA pilot.** The platform defaults to 30-day retention; nobody changed it; the HIPAA six-year audit floor is silently violated. Retention is set per store, per pilot, in the Helm values or the managed-cloud project settings, and the setting is part of the pilot-setup checklist.

## Summary

- The LLM observability platform is the specialised second-tier store: full prompts / completions, SME feedback, prompt-template versioning, dataset / experiment management, judge integration. A generic APM cannot do this well.
- Five serious options at publication — [Langfuse](https://langfuse.com/), [LangSmith](https://smith.langchain.com/), [Arize Phoenix](https://phoenix.arize.com/), [W&B Weave](https://wandb.ai/site/weave), [Braintrust](https://www.braintrust.dev/). Each requires per-pilot verification against the current trust centre before selection.
- Decision axes: self-host-ability, customer KMS / CMK, customer IdP, retention controls, prompt / dataset / experiment APIs, OTLP support, fit with the vendor's existing tenancy. Air-gapped and HIPAA / FedRAMP / DORA pilots gate on self-host-ability.
- Self-hosted deployment pattern: app tier in Kubernetes, managed Postgres + ClickHouse + object storage with CMK, OIDC against the customer's IdP, outbound telemetry disabled, retention set per store to the regulatory floor, egress restricted to intra-VPC / intra-enclave.
- Daily-dashboard discipline: four dashboards (worst traces, SLO burn, rollout health, cost) opened by named customer-side operators with named actions. If the operator does not open them, the pilot's operability is weak.
- Prompt-template versioning as a first-class object + dataset / experiment management are the mechanisms Chapter 06's eval back-feed depends on.
- Archetype A: Langfuse self-hosted in EKS with KMS and Okta; Archetype B: Phoenix OSS on-prem in the agency's enclave; Archetype C: Langfuse self-hosted in Azure under the EU Data Boundary.
- Seven anti-patterns: managed cloud without cleared contract, one-pod self-host, two platforms in parallel, FDE-only user, prompts in source, content without break-glass, retention left at default.
- Chapter 04 is next: LLM-specific SLOs — quality / latency / cost / safety — tied to the customer's business KPI, with the error-budget policy that decides who gets paged when the budget burns.
