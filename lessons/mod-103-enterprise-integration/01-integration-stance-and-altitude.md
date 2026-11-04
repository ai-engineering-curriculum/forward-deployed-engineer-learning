# Chapter 01 — The Integration Stance and the FDE Altitude

## Why this chapter exists

Mod-101 Chapter 05 named the *integration design* as Deliverable 4 — the artifact the customer's security team, data team, and identity team countersign before any production workload lights up. Mod-102 Chapter 04 fixed the pilot scope and the stack ADRs that framed the choice. This module is where those artifacts collide with the reality of the customer's actual stack: an identity provider that the customer has owned for a decade, a service mesh someone on the platform team finished last year, a legacy mainframe batch feed the controller runs reports on every morning, and a security review checklist that will refuse to approve a pilot that does not honour each of them.

Integration engineering at the FDE altitude is not the same craft as the application-layer integration work covered in `applied-ai-engineer` (level 30) or `llm-application-developer` (level 25). Those tracks assume the enterprise surface is a given — a REST endpoint with a stable OpenAPI spec, a webhook registered in a dashboard, an OAuth app installed with a click. The FDE operates a level up: the surface is *negotiated* with the customer, the auth story is *designed* against the customer's identity-and-access platform, and the networking is *shaped* against the customer's security posture. The interface the applied-AI engineer eventually calls is the interface the FDE first made exist.

This chapter frames the stance the next eight chapters build on: what the integration design covers, how the FDE holds altitude between the L30 craft and the L35 peer specialists, and the single organising budget — latency × reliability × cost — that every integration decision in this module trades against.

## What "integration" covers for an LLM pilot

Over the course of a pilot the FDE touches seven surfaces of the customer's stack. Each chapter in this module takes one.

| Surface | Chapter | What the FDE owns |
|---|---|---|
| Identity provider | 02 | SAML 2.0 / OIDC 1.0 with PKCE / SCIM 2.0 against the big-4 IdPs and the long-tail |
| Service-to-service auth | 03 | OAuth 2.0 client-credentials, mTLS, RFC 8693 token exchange, STS / Workload Identity |
| Data surface | 04 | REST (OpenAPI 3.1), GraphQL, webhooks, CDC, event buses (CloudEvents), batch (SFTP / S3 / Snowflake stages) |
| SaaS platforms | 05 | Salesforce, ServiceNow, SAP, Workday, Snowflake, Databricks, Google Workspace, Microsoft 365, HubSpot, Zendesk |
| Legacy surface | 06 | SOAP, SFTP, mainframe gateways, screen-scrape, batch |
| Customer networking posture | 07 | PrivateLink / VPC endpoints, VPC peering, Site-to-Site VPN, egress proxying, split tunneling |
| Model Context Protocol server | 08 | MCP spec-compliant server to a customer agent surface when the integration shape calls for it |

A ninth surface — the boundary to `ai-infra-inference-engineer` (model-serving inside the customer VPC) and `ai-data-engineer` (customer-side ETL / ingestion pipelines) — is covered in Chapter 09. These are not FDE-owned; they are deferral contracts the FDE invokes when the integration shape requires them.

Nothing in the list is novel standards. Every protocol in the table is well-specified, battle-tested, and documented by its publisher of record. The FDE's craft is not inventing protocols — it is knowing which protocol the customer's platform team expects, which authentication posture their security team will approve, and how the five or six protocols in a single pilot compose into one coherent architecture that the customer can audit end-to-end.

## The integration design deliverable

Mod-101 Chapter 05 fixes the deliverable shape; this chapter sharpens the content for an integration specifically. An integration design is a 6–12 page document with the following sections. Every section is countersigned by a named person on the customer side.

```
Integration design — <workflow name>

1. Data-flow diagram (one page)
   - Every system that reads or writes production data for the pilot
   - Every network boundary crossed
   - The direction of every arrow, labelled with the protocol and the auth posture
   - The identity propagation path (user -> service -> downstream call)

2. Identity and access
   - End-user SSO story: SAML / OIDC, IdP, attribute/claim set, group mapping
   - Service-to-service story: OAuth client-credentials / mTLS / token exchange
   - User / group provisioning: SCIM or custom
   - Permission model propagation (see mod-105 for retrieval-side ACL)

3. Data surface contracts
   - For each surface: protocol, schema (OpenAPI 3.1 / Avro / Proto / JSON Schema),
     schema-registry location, rate limits, error taxonomy, latency expectation
   - Lineage: upstream source, pipeline that produces the shape we read, downstream consumer

4. Networking posture
   - PrivateLink / VPC endpoint / VPN / public-with-egress-proxy
   - Named CIDR ranges, DNS strategy, certificate strategy
   - Egress-proxy ACL if the pilot calls out to a model provider

5. Secrets, keys, and credentials
   - Where each credential lives (customer vault vs. FDE-managed)
   - Rotation story (who owns, what cadence, what alerting)
   - Break-glass access: who can read what, under which audit log

6. Observability and audit
   - What events are logged, where, for how long, under whose retention policy
   - Where the customer's security team subscribes to the audit log
   - How a customer-side incident responder would reconstruct a single request

7. Legacy / long-tail surfaces (if any)
   - The trade-off calculus for each: latency vs. cost vs. reliability vs. LLM budget

8. Trade-off decisions (short ADRs)
   - For each non-trivial choice: context, decision, alternatives considered, consequences

9. Open items / deferred
   - What is explicitly out of scope for the pilot
   - What is punted to ai-infra-inference-engineer, ai-data-engineer, or the customer's platform team
```

This is a long deliverable. Writing it is not optional. In regulated verticals the signed integration design is a prerequisite for the security review packet (mod-108); in SaaS-install deployments it is the prerequisite the customer's platform team will cite when they refuse to open firewall rules without one.

## The FDE altitude: between L30 craft and L35 peers

The applied-AI-engineer at level 30 integrates against the interface the FDE hands them. The FDE at level 35 holds the altitude where the interface is still being decided. Two concrete asymmetries:

**Protocol choice is still live.** For an L30 engineer, "we use GraphQL" is a given — the schema is in the repo, the client library is generated, the query pattern is the surrounding codebase. For an FDE at week 3 of a pilot, "we use GraphQL vs REST vs CDC against the customer's system of record" is still a choice, driven by the customer's rate-limit posture (Chapter 04), the SME workflow's latency budget (Chapter 06), and what the customer's platform team will let the FDE authenticate against (Chapter 03). The L30 engineer's craft starts after this choice lands.

**Identity propagation is still being drawn.** For an L30 engineer, `request.user` is populated by the framework the surrounding product already runs. For an FDE, the end-user identity has to pass from the customer's IdP (Chapter 02), through the pilot's service-layer auth (Chapter 03), through any orchestration layer (mod-106), and down to the retrieval layer (mod-105), with a signed-trail every hop. Three hops, four systems, three different token shapes along the way. The FDE draws this path; the L30 engineer later consumes `user.permissions`.

Against the peer L35 specialists the altitude works in the other direction. The FDE does *not* own model-serving internals (`ai-infra-inference-engineer`), platform-scale ML security (`ai-infra-security`), or release-assurance packet composition (`ai-evaluation-engineer`). The FDE invokes them when the integration shape requires them — see Chapter 09 for the explicit boundaries to `ai-infra-inference-engineer` and `ai-data-engineer`.

In short: the FDE is the generalist who assembles the eight surfaces in the table above into one coherent pilot architecture, escalating to peer specialists for the parts that are deeper than generalist altitude. Mod-101 Chapter 06's generalist-vs-escalation rule is the one to re-read if the boundary feels unclear.

## The organising budget: latency × reliability × cost against the LLM feature budget

Every integration decision in the next eight chapters trades against one three-dimensional budget.

**Latency** is the SLO the pilot's LLM feature carries, minus the model-call latency, minus the orchestration overhead. If the pilot promises a 10-second p95 draft-ready latency (mod-102 Chapter 04 sample) and the model call budgets 3 seconds, then the entire integration layer — identity validation, data fetch, retrieval, write-back — has to fit inside 7 seconds p95.

**Reliability** is the error budget the pilot carries. If the pilot promises 99.0% availability on draft generation and the model provider's SLA covers 99.5%, then the integration layer must sustain less than 0.5% error-contribution across its surfaces. Google SRE's chapter on [error budgets](https://sre.google/sre-book/embracing-risk/) is the discipline here; it is used across this module without re-derivation.

**Cost** is the per-task budget from the pilot scope (mod-102 Chapter 04 guardrail "Cost per task < $0.15"). Every integration surface has a per-call cost — token cost for retrieval-style calls, infra cost for data-pipeline queries, licensing cost for SaaS-platform API consumption units, egress cost for cross-cloud traffic. These add up.

The trade-offs appear concretely in the chapters:

- **Chapter 02:** The decision whether to roll SCIM 2.0 user provisioning vs. just trust IdP claims at each request has a latency cost (claim parsing per request is small; SCIM sync is a one-time setup cost) and a reliability cost (SCIM decouples the auth path from the provisioning path, so an IdP outage affects auth but not already-provisioned users).
- **Chapter 04:** The decision between pulling customer data synchronously (REST, GraphQL) vs. pre-staging it (CDC into a mirror, batch load into a vector store) is the most common integration trade-off in LLM pilots. Synchronous keeps freshness; pre-staging keeps latency.
- **Chapter 06:** A legacy mainframe batch feed with a 4-hour cadence can be adequate for a weekly-reporting LLM feature and completely wrong for a real-time case-triage feature. Chapter 06 writes the trade-off down explicitly.
- **Chapter 07:** PrivateLink / VPC-endpoint termination has a measurable one-time setup cost and a measurable per-GB egress cost vs. public-internet-with-TLS; the FDE chooses based on the customer's security posture, not on "which is cheaper in isolation."

Whenever a chapter presents a decision, it will sketch the row in this three-dimensional budget it trades against. The number will not always be exact; the shape of the trade-off will always be stated.

## Four discipline anti-patterns

The commonest failure modes for an FDE at the integration altitude:

**Skipping the identity-propagation diagram.** The FDE builds the integration layer against a single test user, with a hard-coded service account. At pilot exit the customer's security reviewer asks: *when the SME submits a case to the LLM feature, which identity do you log, which identity do you authorise against the data source, and which identity appears on the downstream audit log?* The three may not match. Chapter 02 Section 4 fixes the pattern: draw the identity path on day 1 and keep it current.

**Choosing the fashionable protocol rather than the customer's.** GraphQL is a lovely query language. If the customer's system of record exposes only SOAP and the platform team refuses to stand up a GraphQL gateway for a pilot, the right choice is SOAP, however unfashionable. Chapter 04 and Chapter 06 are explicit: the customer's existing surface is the interface; the FDE adapts, it does not insist on rewriting the customer's stack to fit the pilot.

**Treating networking as an afterthought.** PrivateLink is not the last-week-of-pilot checklist item. If the customer runs HIPAA / FedRAMP-equivalent / regulated-vertical posture, the data path from the FDE's service to the customer's data source *must* be inside the customer's trust boundary. Chapter 07 shows this is a week-2 decision, not a week-10 decision — because the setup cost (DNS, routing tables, certificate issuance, security-review sign-off) is multi-week.

**Building MCP first because it is new.** MCP is the right tool when the customer's agent surface (Copilot, a vendor platform agent, a custom tool palette the customer's internal LLM product exposes) needs to expose the FDE's pilot surface as a tool. It is not the right tool when a direct API call would do. Chapter 08 names the decision test explicitly.

Each of these anti-patterns compounds. All four together are the shape of the pilot that reaches week 10 and is blocked from production by a security-review surprise.

## How this module sits with the rest of the track

The chapters that follow pair in both directions with the rest of the track:

- **Mod-104 (LLM application in customer environments)** consumes the model-provider auth choices (Chapter 03) and the gateway egress posture (Chapter 07) made in this module.
- **Mod-105 (grounding on customer data)** consumes the data-surface integration decisions (Chapter 04) and the SCIM / IdP group-mapping (Chapter 02) to propagate ACLs into retrieval.
- **Mod-106 (agentic workflows on enterprise surfaces)** consumes the tool-auth model from Chapter 03 and the MCP server pattern from Chapter 08.
- **Mod-107 (eval engineering on customer data)** depends on the audit-log shape defined in the integration design.
- **Mod-108 (enterprise security and compliance)** consumes the signed integration design as the first page of the security-review packet.
- **Mod-109 (deployment, rollout, change management)** consumes the networking posture (Chapter 07) when it designs the canary and kill-switch topology.
- **Mod-110 (observability and on-call)** consumes the identity-propagation path (Chapter 02 §4) to make trace spans correctly attribute user-of-record.

If a chapter in the rest of the track refers back to "the integration design," it means the artifact this module builds.

## Summary

- The FDE at level 35 negotiates the enterprise interface the applied-AI engineer later consumes: protocol choice is still live, identity propagation is still being drawn.
- Integration engineering for an LLM pilot covers seven surfaces: identity provider, service-to-service auth, data surface, SaaS platforms, legacy systems, networking, and — when the shape calls for it — an MCP server. One chapter per surface.
- The deliverable is a signed 6–12 page integration design with nine sections — from data-flow diagram through open items. In regulated verticals it is the prerequisite for the security-review packet.
- Every integration decision trades against a three-dimensional budget: latency, reliability, cost — each measured against the LLM feature's end-to-end SLO from the pilot-scope document.
- Four anti-patterns to catch early: skipping the identity-propagation diagram, choosing the fashionable protocol over the customer's, treating networking as last-week work, building an MCP server because it is new rather than because the shape requires it.
- Deeper peer specialists own model-serving (`ai-infra-inference-engineer`), customer-side ETL (`ai-data-engineer`), and platform-scale ML security (`ai-infra-security`). The FDE's craft is to compose the surfaces; Chapter 09 draws the two primary deferral contracts.
