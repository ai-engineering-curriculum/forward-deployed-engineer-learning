# Chapter 04 — Integrating with the Customer's Data Surface

## Why this chapter exists

The LLM pilot does nothing until it reads the customer's data. Chapter 02 authenticated the user; Chapter 03 authenticated the pilot's service to the customer's service. This chapter is about the shape of the data the pilot reads and writes — which protocols the customer already exposes, which schema registries the customer already runs, which lineage story the customer's data team is already defending, and how the LLM pilot plugs into all of that without introducing a parallel pipeline.

Six protocol families cover nearly every FDE-pilot data integration:

| Family | Shape | Typical latency to customer SoR |
|---|---|---|
| REST / OpenAPI 3.1 | Request/response, human-readable, broad tooling | 50 ms – 2 s per call |
| GraphQL | Request/response, one call aggregates many entities | 100 ms – 3 s per call |
| Webhooks | Push, customer calls pilot on event | ~event-time |
| Change Data Capture (CDC) | Stream, customer's DB mutations replicated to pilot | seconds – minutes |
| Event buses (Kafka / PubSub / EventHub / SNS+SQS) | Pub/sub, CloudEvents payloads common | seconds |
| Batch (SFTP / S3 / Snowflake external stages) | File drop or table stage, scheduled | hours – days |

The right protocol is not "the one you prefer." It is the one the customer already runs that fits the pilot's latency budget — the three-way LRC trade from Chapter 01.

This chapter covers each family, plus the two cross-cutting concerns the integration design must address: schema registry (how the shape is defined and versioned) and lineage (where the data came from and where it goes).

## The latency × freshness trade

Before the protocol choice, an LLM pilot has one framing decision: *how fresh must the data be at the moment the LLM reads it?*

- **Must be current to the second** (an ongoing case the SME is actively working) → synchronous REST / GraphQL from the customer's system of record.
- **Current within minutes is acceptable** (a backlog of cases triaged in bulk; a dashboard that updates every few minutes) → CDC or event-bus consumption with the pilot maintaining a mirror.
- **Current within a day is acceptable** (a weekly-reporting pilot; a retrospective analysis) → batch load.

Two anti-patterns kill pilots at this step. First: the FDE picks synchronous REST because it is familiar, then the customer's system-of-record rate limits the pilot to 10 QPS and the pilot cannot serve concurrent SMEs. Second: the FDE picks batch because the customer's data team already runs batch, then the SME finds that the LLM feature is drafting against yesterday's case state and ships a wrong response.

The right answer is the one the pilot scope's user stories require, not the one the current infra invites. If the scope needs real-time but the customer's SoR cannot serve the QPS, the integration design surfaces that mismatch before the pilot builds — and either (a) uses CDC + a local mirror to absorb the QPS, or (b) negotiates a higher rate-limit tier, or (c) narrows scope to batch.

## REST and OpenAPI 3.1

### Why OpenAPI 3.1

REST is a description of a style; OpenAPI is the schema language the customer's data API almost certainly publishes to. [OpenAPI 3.1.0](https://spec.openapis.org/oas/v3.1.0) is the current major version, aligned with JSON Schema 2020-12. The FDE pattern:

1. Get the customer's OpenAPI spec (or write one jointly if the customer's API has none — more common than one expects).
2. Code-generate a client with a well-maintained generator. [openapi-generator](https://openapi-generator.tech/) and [openapi-typescript](https://github.com/drwpow/openapi-typescript) are the standards; frameworks like FastAPI generate OpenAPI directly and consume it directly.
3. Validate every response at the pilot boundary against the spec. Defending against a silently-renamed field is non-negotiable.
4. Pin to a version; upgrade on a scheduled cadence, not when a build happens to pick up a new generator.

### Pagination and rate limiting

Two patterns to expect across the customer's REST surfaces:

- **Pagination**: cursor-based is strongly preferred (stable under concurrent writes); offset-based is still common but drifts. The pilot's client must honour `Link: <...>; rel="next"` or the API's own cursor field.
- **Rate limiting**: `X-RateLimit-Limit` / `X-RateLimit-Remaining` / `X-RateLimit-Reset` headers (or `Retry-After` on 429). The pilot's client has an exponential-backoff + jitter retry — the Google SRE chapter on [handling overload](https://sre.google/sre-book/handling-overload/) is the reference pattern.

### Error taxonomy

The pilot's client handles three error classes distinctly:

| Class | Example | Retry? |
|---|---|---|
| 4xx (except 429) | Bad request, auth failure, not-found | No. Fix the call. |
| 429 | Too many requests | Yes, after `Retry-After`. |
| 5xx | Server error, gateway timeout | Yes, with backoff + jitter. Budget a max retry count. |

A retry-everything-forever pattern against a 500 is how a pilot's retry storm takes down the customer's data API at 10 am.

## GraphQL

### When GraphQL is right

GraphQL ([spec](https://spec.graphql.org/October2021/)) shines when the pilot needs to aggregate across several entity types in one call and the customer already runs GraphQL. The LLM pilot context where it pays:

- Fetching the full case graph (case + member + plan + claims history) in one request instead of 4 REST roundtrips for each case.
- Projecting only the fields the prompt needs — reducing wire cost and the LLM's token-input cost.

### When to avoid

- The customer's GraphQL gateway is thin and backs REST under the hood — in which case the pilot is paying for GraphQL complexity without the aggregation benefit.
- The customer has no GraphQL surface — do not stand one up for a pilot; use the REST surface already there.

### Watch-outs

- **N+1 queries**: GraphQL makes it easy to request nested relations that resolve to N backend queries. Ask the customer's GraphQL team for query-cost limits; consume with batched / persisted queries where available.
- **Persisted queries**: [Apollo Server persisted queries](https://www.apollographql.com/docs/apollo-server/performance/apq/) (and the equivalent on other GraphQL servers) give the customer the ability to pre-approve a specific set of queries. In production pilots, the customer's platform team will ask for this.
- **Subscriptions vs. webhooks**: GraphQL subscriptions (over WebSocket or SSE) are rare in enterprise pilots; prefer webhooks or CDC for push semantics.

## Webhooks

### Shape

A webhook is: the customer's system calls an HTTPS URL the pilot exposes, with a JSON body describing an event. The pilot ACKs with 2xx. This is the push side of the integration and is how the pilot stays current without polling.

The three invariants the FDE engineers for:

1. **Signature verification.** The customer's webhook signs the request body with an HMAC-SHA-256 keyed by a shared secret (or an asymmetric signature). Reject unsigned, mis-signed, or stale (replayed) requests. The SaaS platforms (Chapter 05) each have their own signature convention — [Stripe webhooks](https://stripe.com/docs/webhooks/signatures) is a widely-copied reference pattern.
2. **Idempotency.** Networks re-deliver. Every webhook has an event ID; the pilot stores the ID in a short-TTL seen-set and no-ops on duplicate. Doing this at the application layer rather than relying on "we re-receive rarely" is the only correct answer.
3. **Fast ACK, slow process.** The handler validates signature, writes the event to an internal queue, ACKs 200. The actual LLM work runs out of band. A synchronous handler that calls the LLM before ACKing will time out under load and the customer will dead-letter the events.

### Delivery semantics

Nearly all enterprise webhook producers are *at-least-once*. Design accordingly. Exactly-once is a property of your handler (idempotent), not of the delivery.

## Change Data Capture

### Why CDC

CDC reads the customer's database write log (MySQL binlog, Postgres logical replication slots, SQL Server CDC, Oracle GoldenGate) and replays the mutations into a pilot-side mirror. The pilot reads the mirror; the customer's source database is spared the query load; the mirror lags seconds to tens of seconds.

The standard open implementation is [Debezium](https://debezium.io/documentation/reference/stable/) running as Kafka Connect connectors; managed services exist on every major cloud (AWS DMS, GCP Datastream, Azure Database Migration Service).

### When CDC fits

- The pilot needs near-real-time reads against the customer's SoR, but synchronous calls would overwhelm the SoR's QPS budget.
- The customer's data team is comfortable owning a CDC pipeline (or is already running one).
- The data volume is high enough that batch does not fit the latency budget.

### What the FDE decides

- **Scope of replication.** Which tables, which columns. Minimum-necessary is the rule; PHI / PII columns are included only if the pilot's purpose requires them (coordinate with ai-risk-engineer).
- **Transformation at the edge.** Where PII is replicated for operational reasons (keys, join columns), redact / tokenise at the CDC edge — not after it has landed in the pilot's mirror. mod-105 picks this up for retrieval.
- **Replication slot ownership.** A Postgres logical-replication slot holds WAL until read; a stuck pilot consumer can fill the source DB's disk. The customer's DBA team sets the backpressure / abandonment policy; the FDE agrees to it in writing.

### CDC trade-offs

| Property | Cost |
|---|---|
| Freshness | Seconds; occasionally tens of seconds under load |
| Source DB impact | Near zero at steady state; non-zero during snapshot / catchup |
| Operational surface | Non-trivial — Kafka Connect / managed CDC service to run |
| Schema change handling | Must be designed (schema evolution; see below) |

## Event buses and CloudEvents

### The pattern

The customer runs an event bus. The pilot either consumes events from a topic (case-created, case-updated) or produces events to a topic (draft-ready, review-completed). The four common buses:

- **Apache Kafka** / Confluent Cloud. The default enterprise bus. [Kafka docs](https://kafka.apache.org/documentation/) and [Confluent's documentation](https://docs.confluent.io/) are the references.
- **Google Cloud Pub/Sub**. [Pub/Sub overview](https://cloud.google.com/pubsub/docs/overview).
- **Azure Event Hubs**. [Event Hubs documentation](https://learn.microsoft.com/en-us/azure/event-hubs/).
- **AWS SNS + SQS** (SNS for fan-out, SQS for durable work queues). [SNS documentation](https://docs.aws.amazon.com/sns/) and [SQS documentation](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/).

### CloudEvents

The envelope format most enterprises converge on is [CloudEvents 1.0](https://github.com/cloudevents/spec), a CNCF specification for event metadata. A CloudEvents-shaped event:

```json
{
  "specversion": "1.0",
  "type": "com.customer.case.created",
  "source": "https://cases.customer.example",
  "id": "a1b2c3d4-...",
  "time": "2026-11-04T13:22:11Z",
  "datacontenttype": "application/json",
  "subject": "case/12345",
  "data": {
    "caseId": "12345",
    "memberId": "67890",
    "priority": "P1"
  }
}
```

The pilot's consumer code binds the payload to a schema (see below), validates, and processes. CloudEvents' bindings for Kafka, Pub/Sub, EventHub, and SNS+SQS are all in the spec.

### Consumer discipline

- **Consumer groups / subscriptions**: the pilot's consumer gets its own named group / subscription. Sharing a group with the customer's own services means events disappear to one or the other.
- **Offset / ack discipline**: ack after durable processing, not after receive. On Kafka: commit offset only after the work is persisted. On Pub/Sub: explicit ack after success. On SQS: delete-message after success, use long-poll + visibility timeout for backoff.
- **Dead-letter queues**: every pilot consumer has a DLQ. Messages that fail after N retries go there; the pilot has a human-facing surface to inspect the DLQ and a runbook for re-queueing (mod-110).

### Semantics

Event buses are at-least-once. Idempotency is the pilot's responsibility — same discipline as webhooks. Kafka's "exactly once" semantics are a specific property of transactional producers / Kafka Streams; they do not relieve the pilot's consumer from being idempotent.

## Batch — SFTP, S3, Snowflake external stages

### When batch fits

- Historical bulk training / fine-tune data: batch is natural; there is no realtime need.
- Overnight data refresh: a dataset drops at 2 am; the pilot picks it up at 2:30 am; the SME uses it the next morning.
- Low-value realtime but high-value historical: a weekly report on pilot throughput can be batch; individual case draft generation cannot.

### Shapes

- **SFTP**: file drop to an SFTP server, usually inside the customer's DMZ. The FDE's pilot fetches on a schedule (or is pushed to via the customer's own SFTP watcher). Chapter 06 covers SFTP in depth as a legacy surface; here, note it is still common in modern pilots.
- **S3 (or GCS / Azure Blob)**: a cloud bucket as drop zone. The pilot has read-only IAM access; the customer's pipeline writes; the pilot processes.
- **Snowflake external stages** ([Snowflake external stages documentation](https://docs.snowflake.com/en/user-guide/data-load-local-file-system-stage)): an S3 / GCS / Azure Blob prefix registered as a stage in Snowflake, loadable with COPY INTO.
- **Databricks volumes / autoloader** ([Databricks Auto Loader](https://docs.databricks.com/en/ingestion/auto-loader/index.html)) are the Databricks parallel.

### Format

- **Parquet** for anything table-shaped. Compressed, columnar, schema-embedded. The default for analytical data.
- **Avro** when the data is streamed and needs tight schema-evolution semantics (see Schema registry below).
- **JSONL** for unstructured or semi-structured records.
- **CSV** only when the customer's upstream pipeline writes CSV; prefer Parquet for any new drop.

## Schema registry

### Why one at all

The pilot that reads a field called `priority` today will misinterpret the event the week the upstream team renames it to `priorityLevel`. A schema registry prevents that:

- The event producer registers its schema in a central registry at publish time.
- The consumer reads the schema by ID and binds payloads safely.
- Breaking changes require a new version with explicit deprecation.

### The two common registries

- **[Confluent Schema Registry](https://docs.confluent.io/platform/current/schema-registry/)** — the default for Kafka-based stacks; supports Avro, JSON Schema, and Protobuf; compatibility modes (BACKWARD, FORWARD, FULL) enforce what kinds of change are allowed.
- **[Apicurio Registry](https://www.apicur.io/registry/)** — open-source alternative, Kafka-oriented, with similar compatibility semantics.

OpenAPI specs serve the same role for REST (they *are* the schema registry for REST surfaces). GraphQL has its own schema. The three together cover nearly every FDE-pilot data surface.

### Compatibility modes

- **BACKWARD** — new schema can read old data. Adding an optional field is fine.
- **FORWARD** — old schema can read new data. Removing a field is fine.
- **FULL** — both. Hardest to satisfy, cleanest to consume.

The pilot's producer and consumer should both sit at the compatibility mode the customer's data team already uses. If the customer has no preference, FULL is the safe default.

## Lineage

### What the customer wants to see

A lineage view answers: given this LLM output, what data produced it? Which source system, which pipeline, which schema version, which timestamp?

The two common open standards:

- **[OpenLineage](https://openlineage.io/docs/)** — emits lineage events during pipeline runs. Integrates with Airflow, Spark, dbt, Flink.
- **[Marquez](https://marquezproject.ai/)** — the metadata server that collects OpenLineage events.

Commercial data-catalogue platforms (Collibra, Alation, Atlan, DataHub) can consume OpenLineage.

### What the FDE records

For each downstream LLM output, the pilot records (in its own trace / observability layer — mod-110):

- Source system (e.g. `customer.cases.v3`).
- Query / event / batch job that produced the input (`case_123, event_id abc-...`).
- Schema version (`cases.v3.5`).
- Retrieval corpus version (from mod-105).
- Prompt version and model version.

That tuple is the "lineage receipt" for any one LLM output and is the artifact the customer's data governance team will ask for.

## Composition: which protocols the pilot typically uses together

A real integration design does not pick one protocol. A common shape for a payer case-triage pilot:

- **CDC** from the customer's case SoR → pilot-local Postgres mirror. Reads are fast and the SoR is not loaded.
- **Webhooks** from the case SoR → pilot handler. Push on new case so the pilot can enqueue work without polling.
- **Kafka topic** `pilot.drafts.v1` → pilot writes CloudEvents when a draft is ready; the customer's downstream pipeline consumes.
- **REST (OpenAPI 3.1)** against the customer's member / plan services → fetched per-request for the handful of attributes needed on the draft.
- **Batch Parquet drop** to S3 → eval dataset for mod-107 and weekly retrospective.

Five protocols, one pilot. Each has its own section in the integration design, its own authentication posture (Chapter 03), its own schema reference, its own lineage story.

## Schema evolution and version pinning

The pilot pins to specific schema versions and upgrades deliberately. The anti-pattern is "latest" everywhere — the first time a producer makes a breaking change, the pilot breaks silently in production.

The FDE's default:

- Pin consumer schemas to a specific version.
- Have a renovate-style schedule for schema upgrades, with the compatibility mode (above) as the gate.
- When the customer's producer changes schema, both sides coordinate: new schema version published; pilot adds consumer for new version; grace period; pilot removes consumer for old version.

In the integration design, write down which schema versions the pilot depends on for each upstream feed, and who the FDE will coordinate with on changes.

## Common failure modes

**Polling when webhooks or CDC exist.** The pilot polls the SoR every 30 seconds. The customer's data team sees the pilot's traffic in their rate-limit dashboard. The fix is almost always to receive events, not to ask every 30 seconds.

**Trusting webhook payloads without signature verification.** The pilot handles webhook events as received, with no HMAC validation. An attacker who finds the endpoint can inject synthetic events. Signature-verify in the handler; reject unsigned.

**Caching without an invalidation story.** The pilot caches member profile for 10 minutes to reduce calls to the customer's member API. A member updates their consent flag; the pilot's draft still reflects the old consent for up to 10 minutes. The fix: either lower the TTL to sub-second (nearly a no-cache), or subscribe to the member-updated webhook and invalidate on event.

**Bulk reads without row-level security awareness.** The pilot reads cases in bulk for an offline eval. The bulk surface may expose rows the per-user API would have filtered out. The pilot's bulk reader must apply the same ACL filter as retrieval (mod-105) — or pre-filter server-side before extraction.

**Forgetting about audit log exposure.** The pilot writes to the customer's case store; the customer's case-store audit log shows the pilot's writes. Who reads that audit log? Is the pilot expected to make its own audit log available? Chapter 03 covered the user-of-record question; the write-shape question is answered here: the pilot's own audit trail and the customer's data-system audit trail must agree on what each write actually did.

## Summary

- The customer's existing data surface decides the protocol; the pilot's latency budget decides which part of it to use. Freshness-to-second → synchronous REST/GraphQL; freshness-to-minutes → CDC / event bus; freshness-to-day → batch.
- REST surfaces follow OpenAPI 3.1 and JSON Schema 2020-12. Validate every response at the boundary; honour pagination cursors and rate-limit headers; retry only on 429 + 5xx with backoff + jitter.
- GraphQL is valuable when the pilot aggregates across many entities per call; it requires watching for N+1 and the customer's query-cost policy.
- Webhooks are push; three invariants: signature verify, idempotent on event ID, fast-ACK + out-of-band process.
- CDC via Debezium or a managed service gives near-real-time reads without loading the SoR; the FDE negotiates replication scope and backpressure in advance.
- Event buses (Kafka, Pub/Sub, Event Hubs, SNS+SQS) carry CloudEvents-formatted messages in most mature enterprises. Consumer discipline: own group, ack-after-process, DLQ + runbook.
- Batch (SFTP / S3 / Snowflake external stages / Databricks Auto Loader) is right for overnight-refresh or historical data; Parquet is the default format.
- Schema registry (Confluent / Apicurio for streams; OpenAPI for REST; GraphQL SDL for GraphQL) lets the pilot bind safely to upstream shape. Pin versions; agree on compatibility mode.
- Lineage (OpenLineage + Marquez or a commercial catalogue) is what the customer's data governance team asks for; the pilot records source system, query, schema version, retrieval corpus, prompt, and model version for each LLM output.
- Five common failure modes — polling when push is available, unverified webhook payloads, caching without invalidation, bulk reads without ACL filtering, missing audit-log parity. Each has a standards-grounded fix.
