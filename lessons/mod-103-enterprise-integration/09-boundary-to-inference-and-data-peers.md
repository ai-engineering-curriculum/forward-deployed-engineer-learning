# Chapter 09 — Boundaries to the `ai-infra-inference-engineer` and `ai-data-engineer` Peers

## Why this chapter exists

Mod-101 Chapters 03 and 06 fixed the general rule: the FDE at level 35 is a specialist IC among peers, not a replacement for them, and escalation is a discipline, not a failure mode. This chapter instantiates that rule for the two peer specialists whose work is most entangled with integration engineering: `ai-infra-inference-engineer` (peer, level 35, Infra family) and `ai-data-engineer` (peer, ML Engineering family).

These two boundaries show up in almost every integration design. The first arises every time the pilot's model-serving posture is "inside the customer VPC" rather than "hosted SaaS." The second arises every time the pilot consumes a customer data feed that requires a real pipeline rather than a one-off read. The FDE's craft is to build the integration layer that *sits between* the pilot's LLM feature and these two peer specialisms — and to know exactly what to hand to each of them, when.

## The `ai-infra-inference-engineer` boundary

### What the peer owns

[`ai-infra-inference-engineer`](https://github.com/ai-infra-curriculum/ai-infra-inference-engineer-learning) (peer, level 35, Infra family) owns inference serving depth: vLLM / TGI / SGLang internals, batching, quantisation, KV-cache management, autoscaling, model-server capacity planning, hardware selection (H100 / A100 / L4 / AWS Inferentia / TPU), and the production operation of the model server itself.

In short: *everything that makes the model respond to a prompt, quickly, at scale, inside the customer's trust boundary*.

### What the FDE adds

The FDE owns the *integration* of the model-serving layer into the pilot:

- Which model gets called from which pilot code path (mod-104 handles the routing).
- Which identity the pilot's call carries (Chapter 03).
- Which network path the call takes (Chapter 07).
- Which observability spans the call appears in (mod-110).
- Which budget the call consumes (mod-104's cost / latency SLOs).

The FDE does **not** own:

- The inference server's batching strategy.
- The hardware the server runs on.
- The quantisation choice.
- The KV-cache policy.
- The autoscaling setup for the model server.

### When this boundary fires

In three common cases.

**Case 1: customer policy requires model serving inside their VPC.** The customer is HIPAA, FedRAMP-equivalent, or has data-residency constraints that forbid sending prompts to a hosted model provider. The pilot deploys an open-weights model (Llama 3, DeepSeek, Mistral, or a customer-licensed model) inside the customer's VPC. Someone has to stand up the model server. That someone is `ai-infra-inference-engineer`, not the FDE.

**Case 2: the pilot's throughput needs exceed a hosted provider's rate limit.** Rare at pilot scale; real at post-pilot scale. The decision to move from hosted to self-hosted is often made in mod-109's rollout planning. The FDE escalates the decision; the peer owns the execution.

**Case 3: the customer wants a specific model that only runs inside their trust boundary.** A fine-tuned model from `training-pipeline-engineer`, a customer-provided model, a specialised vertical model. Same story: the FDE integrates against whatever model server the peer stands up.

### What the FDE hands to the peer

A specific brief, not "please run a model." The brief includes:

| Field | Example |
|---|---|
| Model family and version | Llama 3.1 70B Instruct, bf16 |
| Expected QPS, p50 / p95 | 5 QPS p50, 15 QPS p95 |
| Latency SLO | p95 time-to-first-token < 1.5 s, p95 total < 3.0 s at 2048-token output |
| Request / response shape | OpenAI-compatible chat completions API, structured outputs on |
| Auth posture | OAuth 2.0 client credentials from pilot; token-exchange for user-of-record |
| Network posture | Reachable only from pilot's subnet in VPC |
| Observability | OpenTelemetry traces emitted to pilot's collector; metrics to pilot's Prometheus |
| Change window | Pilot deploys Mon–Thu, no deployments Fri |
| Rollback | Pilot needs server-side model-version pinning + fast rollback to previous version |

With that brief, the peer can scope and execute the model server. Without it, the peer asks the same questions back; three days are lost.

### What the FDE receives back

A model server deployed inside the VPC, reachable via a defined endpoint, meeting the requested SLOs. Plus:

- Runbook for the model server (owned by the peer; referenced in the pilot's runbook).
- On-call rotation: typically the customer's platform team on L1, the peer's team on L2 for deep model-server issues, the FDE on L2 for pilot-side regression.
- Observability dashboards the FDE can embed in the pilot's dashboards.

### Who answers which pager

Mod-110 covers shared on-call in depth. The quick rule for the FDE/inference-engineer boundary:

- Latency regression on model calls → first page the peer (unless the pilot deployed a new prompt). Observability dashboards tell you which.
- Model returns an obviously wrong draft → page the FDE; it is nearly always a prompt / RAG / eval issue (mods 104–107).
- Model server down → page the peer; the FDE triages pilot's own availability.
- Customer VPC networking blip → page the customer's platform team + the FDE.

### Anti-patterns

**The FDE tries to run vLLM themselves because "it's just docker-run."** The first production incident in the model-server layer reveals how much hidden complexity there is. Hand it to the peer; stay on integration.

**The FDE agrees to an aggressive SLO without the peer's sign-off.** The peer is the one who has to meet it. If the FDE commits to p95 < 2.0 s on a 70B model at 10 QPS and the peer sees a 3.5 s realistic floor, the peer's input must change the pilot scope — not the peer's reality.

**No observability handshake.** The peer's dashboards are great for model-server health; the FDE's are great for pilot health. Without a shared span-tagging convention, no one can tell "the pilot is slow" from "the model server is slow" in a 2 a.m. incident. Mod-110 fixes this; Chapter 09 names the handshake at integration design time.

## The `ai-data-engineer` boundary

### What the peer owns

[`ai-data-engineer`](https://github.com/ml-engineering-curriculum/ai-data-engineer-learning) (peer, ML Engineering family) owns customer-side ETL and ingestion depth: Airflow / Dagster / Prefect pipelines, dbt models, Spark / Databricks transforms, lakehouse / warehouse table design, data-quality monitoring, schema evolution, bulk reprocessing, and backfill operations.

In short: *everything that moves and reshapes the customer's data at pipeline scale*.

### What the FDE adds

The FDE owns the *integration* of the data pipeline into the pilot:

- Which upstream feeds the pilot consumes (Chapter 04).
- The shape of the contract between the pipeline's output and the pilot's consumer (schema registry; Chapter 04).
- The freshness SLO the pilot asks the pipeline to meet.
- The identity under which the pipeline writes to pilot-visible storage.
- The lineage story the customer's data governance team reads.

The FDE does **not** own:

- Pipeline orchestration internals.
- Spark transform tuning.
- dbt model structure.
- Data-quality rules at pipeline scale.
- Bulk reprocessing / backfill policy.

### When this boundary fires

**Case 1: CDC pipeline standup.** Chapter 04 introduced CDC as a pattern. Standing up Debezium / managed CDC into a pilot-visible store is a pipeline job. The peer owns pipeline ops; the FDE owns what the pilot reads.

**Case 2: the gold-set extraction.** Mod-107's gold-set construction requires pulling a representative dataset out of the customer's SoR, scrubbing PII, landing it in a sanctioned sandbox, and refreshing periodically. The peer owns the extract / transform / land loop; the FDE owns what the eval consumes.

**Case 3: retrieval-corpus ingestion.** Mod-105's grounding story depends on the customer's docs / knowledge base being chunked, embedded, and indexed. For any non-trivial corpus the ingestion is a pipeline, and the peer should own it. The FDE owns the retrieval API contract.

**Case 4: write-back to the customer's data lake.** The pilot produces drafts, decisions, audit records. These flow into the customer's lake for downstream reporting and retraining. The peer owns the sink pipeline; the FDE owns the producing contract.

### What the FDE hands to the peer

Again, a specific brief:

| Field | Example |
|---|---|
| Source | Customer's Postgres `cases_v3` DB, specific schemas/tables |
| Sink | Pilot-visible S3 bucket in customer VPC; Parquet; partitioned by date |
| Schema | Avro schema `cases.v3.5` registered in customer's Confluent registry |
| Transformations | PII tokenisation on `member_name`, `member_dob`; drop `ssn` entirely |
| Freshness SLO | Rows reach sink within 5 minutes p95 of source write |
| Volume | ~50k rows/day typical; spike to 200k/day at month-end |
| Backfill | Initial 30-day backfill, then steady state |
| Lineage | OpenLineage events to the customer's Marquez instance |
| Data quality | Row-count drift alert; schema-violation alert |
| Access | Pilot's role has read-only to the sink bucket |

That brief lets the peer scope the pipeline; "we need some case data" does not.

### What the FDE receives back

- A pipeline the peer operates with their own observability and runbook.
- A documented sink surface (schema, partition layout, SLA).
- Lineage events flowing into the customer's catalogue.
- An on-call rotation for the pipeline (peer's team).
- A change-management process for schema evolution.

### Who answers which pager

- Pipeline lag beyond SLO → page the peer.
- Pipeline produces a row with an invalid shape → page the peer; the FDE triages the pilot's handling of the bad row.
- Pilot can't read the sink (permission error, missing partition) → page the FDE first; could be the pilot's access or the pipeline's output.
- Schema change breaks the pilot → both — the peer's pipeline is upstream of the break; the FDE's consumer didn't absorb the change gracefully.

### Anti-patterns

**The FDE builds a one-off ETL script "because the pilot is small."** It grows. It starts failing silently. No one has observability on it. The data governance review asks about lineage and the FDE has no answer. Hand pipeline work to the peer the moment it is more than a single SQL query.

**The FDE agrees to a freshness SLO without peer sign-off.** Same shape as the inference-engineer anti-pattern. "5-minute lag p95" is a peer's decision to accept, not the FDE's to commit.

**No contract between pipeline and consumer.** The pipeline's output shape is in the pipeline code. The pilot's consumer binds against it by inspection. When the shape changes, the pilot breaks. The contract lives in the schema registry (Chapter 04), it is versioned, and both sides read from it.

## Where the two boundaries intersect

Three cases in which both peers are involved:

**Case A: fine-tuned model on customer data.**
- `ai-data-engineer` owns pulling the training data out of the SoR into a sanctioned environment.
- `training-pipeline-engineer` (another peer — see mod-101 Chapter 03) owns the training pipeline itself.
- `ai-infra-inference-engineer` owns serving the resulting model.
- The FDE owns the pilot's consumption of the served model.

Four people; one integration design that makes it one project.

**Case B: retrieval corpus with re-embedding on content change.**
- `ai-data-engineer` owns detection of upstream doc changes and the triggering of re-embedding jobs.
- The pilot's retrieval layer (mod-105) handles the retrieve-side.
- `ai-infra-inference-engineer` is relevant only if the embedding model itself runs inside the customer VPC.

**Case C: output drafts feeding downstream reporting and future training.**
- The FDE's pilot produces drafts that land in a sink (via `ai-data-engineer`'s pipeline).
- Those drafts feed into future training iterations — a deliberate [human-feedback-driven improvement loop](https://arxiv.org/abs/2203.02155), which the field→product loop (mod-111) picks up.

## Co-escalation sequencing

When both peers are involved, the FDE's job includes sequencing. A real case-triage pilot order:

1. **Week 2**: FDE and data engineer agree on CDC pipeline brief; peer starts the pipeline build.
2. **Week 2**: FDE and inference engineer agree on model-server brief (if in-VPC model); peer stands up the server in staging.
3. **Week 3–4**: pipeline available in staging; pilot's consumer binds against it.
4. **Week 4–5**: model server available in staging; pilot's routing points at it.
5. **Week 5**: integrated staging end-to-end; mod-107 evals run.
6. **Week 6**: production cutover; both peers on standby for cutover day.

Starting either in week 8 is a schedule slip. Starting the integration-design work in week 2 is table stakes.

## The FDE's shared on-call posture

Mod-110 covers shared on-call in depth. The quick summary for this chapter: in a pilot with model-server and data-pipeline peers involved, the FDE is the *integrator* on call, not the specialist on call for either specialism. The pager rules in the sections above are the reference; adjust to the customer's incident framework in the pilot's runbook.

## When to escalate vs. when to try first

Mod-101 Chapter 06 fixed the test: default to "you do it" unless one of the four triggers fires (depth, scale, compliance, novelty). For integration engineering:

- **Try first** when you are configuring a CDC pipeline the customer's data team runs (no new pipeline). The peer may not be needed.
- **Escalate** when the pilot needs a pipeline that doesn't exist yet. Standing up Debezium in a customer VPC is a peer's job.
- **Try first** when you are integrating against a hosted model-provider API.
- **Escalate** when the pilot needs model serving inside the customer VPC.
- **Try first** when you are writing OpenAPI clients and schema validation for a REST surface.
- **Escalate** when the customer asks for a data-quality framework across many tables with monitoring.

The honest answer at week 2 of a pilot is: often you do not know yet which trigger is going to fire. Write down "this capability is TBD; coordinate with ai-data-engineer in week 4" in the integration design; do not leave it unlabelled.

## Common failure modes

**Treating peers as blockers rather than co-owners.** The FDE who reaches out in week 10 saying "I need a model server by Friday" has already failed the discipline. Peers are planned-in from week 2.

**Writing an integration design without peer co-authorship.** The peer reads the design at review time; the peer says "I can't make that SLO with the hardware we have." The design goes back three weeks. Fix: peer reviews the design draft before the sponsor signs it.

**Forgetting the data-pipeline change-management process.** The pipeline's schema evolves; the pilot's consumer breaks silently. Both the peer and the FDE agreed a schema; neither documented the coordination process for changes. The schema-registry compatibility mode (Chapter 04) is the place to pin this.

**No shared observability.** FDE and peer each log in their own tool; nothing correlates at request level. An incident in week 8 takes four hours to triage because nobody can tie a slow pilot request to the model-server span. Fix: OpenTelemetry trace propagation end-to-end, with a shared service convention (the [OpenTelemetry GenAI semconv](https://opentelemetry.io/docs/specs/semconv/gen-ai/) is one of the references).

## Summary

- Two peer boundaries show up in almost every integration design: `ai-infra-inference-engineer` for model-serving inside the customer VPC, and `ai-data-engineer` for customer-side ETL pipelines.
- Each peer owns a deep specialism. The FDE owns the integration layer — how the pilot's code consumes what the peer builds — and the coordinates: brief, SLO, auth, observability, runbook, change management.
- The FDE hands each peer a specific, actionable brief at week 2 — not "we need a model server" or "we need some case data," but a full table of SLOs, schemas, sinks, rotations.
- The FDE receives back a deployed capability, a runbook, an on-call rotation, and observability dashboards that compose with the pilot's own.
- The pager rules partition cleanly: model-server issues to the inference engineer, pipeline-lag issues to the data engineer, prompt/eval/model-output issues to the FDE.
- Sequencing matters: both peers start in week 2, not week 8. The anti-pattern is late escalation; the discipline is planned co-ownership.
- Common failures — late escalation, no peer co-authorship on the design, unmanaged schema evolution, siloed observability — are all preventable with discipline the FDE sets at integration-design time.
- Mod-101 Chapter 06's generalist-vs-escalation rule is the one to re-read when a boundary feels unclear.
