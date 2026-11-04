# Chapter 05 — Integrating with the Customer's SaaS Platforms

## Why this chapter exists

Chapter 04 generalised across data-protocol families. Chapter 05 is the vendor-specific chapter — the one that names the customer's actual SaaS platforms, the APIs they expose, the permission models they enforce, the rate limits they meter, and the audit log shape they produce. Nearly every FDE pilot touches at least one of these platforms as a source-of-record, a system-of-action, or both.

The scope: Salesforce, ServiceNow, SAP, Workday, Snowflake, Databricks, Google Workspace, Microsoft 365, HubSpot, Zendesk. The FDE does not need to be an expert in any of them — the applied-AI-engineer at level 30 or the customer's own admin is the depth. The FDE needs to be fluent enough to recognise the integration pattern each platform expects, the pitfalls specific to each, and the permission / rate / audit shape the integration design must cite.

For every platform below, three things matter: the **permission model** (what the pilot's service identity can do), the **rate limits** (how fast it can do it), and the **audit log shape** (what the customer's governance team sees after the fact). These are the three questions the customer's security reviewer will ask about each integration; the integration design answers them before the review arrives.

## Salesforce

Salesforce is the CRM backbone for a large share of enterprise pilots. The platform exposes four API surfaces, each with a different fit:

- **[REST API](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/)** — the modern default. CRUD on standard and custom objects, SOQL queries, Composite API for batched calls.
- **[SOAP API](https://developer.salesforce.com/docs/atlas.en-us.api.meta/api/)** — legacy but still widely used; required for some enterprise integrations.
- **[Bulk API 2.0](https://developer.salesforce.com/docs/atlas.en-us.api_asynch.meta/api_asynch/)** — asynchronous, CSV / JSON jobs for high-volume reads and writes; the right surface for pilot bootstrapping and eval-set extraction.
- **[Streaming API](https://developer.salesforce.com/docs/atlas.en-us.api_streaming.meta/api_streaming/) / Change Data Capture / Platform Events** — push, for near-real-time event propagation.

### Permission model

Salesforce's permission model stacks:

- **Profile + Permission Sets**: what the service user can do at object and field level.
- **Sharing rules + role hierarchy**: which records the user sees. Row-level is not just about CRUD permission; the sharing model determines record visibility.
- **Field-level security**: individual fields can be hidden from the service user even when the object is permitted.

A pilot's Salesforce service user is typically an Integration User, with a custom profile that grants the minimum object + field access the pilot needs. Coordinate with the customer's Salesforce admin; do not request System Administrator.

Salesforce also offers [Connected Apps](https://help.salesforce.com/s/articleView?id=sf.connected_app_overview.htm) with OAuth 2.0; the pilot registers as a Connected App and obtains access tokens via one of the OAuth flows (JWT Bearer flow is common for service-to-service with no user interaction).

### Rate limits

Salesforce meters at both the API-call level (daily / 24-hour rolling API request limit per org edition) and the per-user / concurrent-request level. The FDE finds the specific limits in [Salesforce API request limits and allocations](https://developer.salesforce.com/docs/atlas.en-us.salesforce_app_limits_cheatsheet.meta/salesforce_app_limits_cheatsheet/salesforce_app_limits_platform_apicalls.htm); the exact numbers depend on edition and licensed add-ons. Typical pilot-level considerations:

- Bulk API 2.0 is charged against the daily Bulk limit, not the general API limit.
- Composite API batches multiple REST calls into one request against the general limit.
- Streaming / CDC / Platform Events have their own event-bus-shaped limits; bursts matter.

### Agentforce actions

[Salesforce Agentforce](https://www.salesforce.com/agentforce/) is the Salesforce-native agent platform. FDE-relevant touchpoints: an Agentforce agent can invoke custom Apex / Flow / external services as actions; the pilot may need to publish an Agentforce Action that calls the pilot's own service. Treat it as a tool-registration surface — covered conceptually in Chapter 08 (MCP) and mod-106 (agentic workflows).

### Audit log

[Salesforce event monitoring](https://developer.salesforce.com/docs/atlas.en-us.object_reference.meta/object_reference/sforce_api_objects_eventlogfile.htm) exposes event log files (ApiEvent, LoginEvent, LightningUriEvent, etc.) for ingestion into a SIEM. The pilot's calls show up in `ApiEvent`; the customer's governance team may require those logs in near-real-time via [Event Streaming](https://developer.salesforce.com/docs/atlas.en-us.platform_events.meta/platform_events/).

## ServiceNow

ServiceNow is the ITSM / workflow-automation backbone for many enterprises. The FDE touches it to read incidents / requests / knowledge articles and to write back decisions or drafts.

- **[Table API](https://developer.servicenow.com/dev.do#!/reference/api/latest/rest/c_TableAPI)** — REST surface for every table in the instance; simple CRUD.
- **[IntegrationHub](https://developer.servicenow.com/dev.do#!/learn/learning-plans/yokohama/new_to_servicenow/app_store_learnv2_devprogram_yokohama_integrationhub_overview)** — the integration platform; outbound calls to external systems and inbound flows.
- **Now Assist** — ServiceNow's generative-AI product family, installed at the platform level.

### Permission model

ServiceNow permissions are role-based: application access, table ACLs, field ACLs, record-level scripted ACLs. The pilot's integration user has a role (or small set of roles) that confers the minimum needed access. The customer's ServiceNow admin grants roles; the FDE's integration design lists the roles it requires.

### Rate limits

ServiceNow implements [inbound request rate limits](https://docs.servicenow.com/csh?topicname=c_InboundRESTAPIRateLimiting.html) at the instance level, with per-user rules. Pilots that hit the rate limits receive HTTP 429 and must back off. For bulk operations the Table API's `sysparm_limit` + pagination is the right pattern; for very large reads, the platform's import set or CSV export tooling is preferable to spinning Table API in a loop.

### Audit log

ServiceNow's [system log tables (syslog, syslog_transaction)](https://docs.servicenow.com/csh?topicname=c_SystemLogs) and the Transaction Log capture every API transaction with the user, timestamp, object, and action. The customer's governance team typically subscribes to these via an [ingestion integration into SIEM](https://docs.servicenow.com/csh?topicname=c_SNFLogExport).

## SAP

SAP is the ERP backbone for many Fortune-500 pilots and brings a different API shape. The FDE typically touches one of two surfaces:

- **[OData](https://help.sap.com/docs/SAP_NETWEAVER_740/ddb54ce62d7a4c7fb4bdc8a8abd9b8ad/ac32e3f3bbe64aba84815ca5fad50ec8.html)** — the standard query protocol SAP exposes for most business objects; versions 2 and 4 are both in production.
- **[SAP RFC / BAPI](https://help.sap.com/docs/SAP_NETWEAVER_750/48c4300fca5d1014b565e6ffd99e94a3/48c42aa7d6ea4bfd923d26c89c70b39f.html)** — legacy remote-function-call interface still used for transactional calls.

[SAP BTP (Business Technology Platform)](https://www.sap.com/products/technology-platform.html) is the modern layer where SAP recommends new integrations are built; it exposes OData APIs for SAP products and hosts integration flows via SAP Integration Suite.

**Joule** is SAP's generative-AI assistant. For integration-engineering purposes it is an agent surface sitting on top of the SAP stack; the FDE may integrate the pilot as a surface Joule can call (again, Chapter 08's MCP discussion generalises).

### Permission model

SAP uses authorisations (via roles) with granular object-and-field ACLs; the role assignment model is more manual than modern SaaS. The customer's SAP Basis admin controls role grants.

### Rate limits

SAP's rate limiting is instance- and configuration-dependent. The pilot's integration design negotiates expected QPS with the customer's SAP admin; there is no single published limit.

### Audit log

SAP provides the [Security Audit Log](https://help.sap.com/docs/ABAP_PLATFORM_NEW/280f016edb8049e998237fcbd80558e7/4d41d5ed624e04f6e10000000a42189c.html) with records of each transaction and authorisation check. Export to the customer's SIEM is a standard integration.

## Workday

Workday is the HCM / HR backbone for many enterprises. The FDE touches it to read worker records, organisational hierarchies, and similar HRIS-driven data (which propagates to IdP via SCIM, Chapter 02).

- **[Workday Web Services / SOAP APIs](https://community.workday.com/sites/default/files/file-hosting/productionapi/index.html)** — the primary programmatic surface, SOAP-based.
- **Workday REST APIs** — a modern layer, published per module; smaller surface than SOAP, growing.
- **[Workday RaaS (Reports-as-a-Service)](https://doc.workday.com/admin-guide/en-us/reporting-and-analytics/reports/reports-as-a-service/qxw1466464062175.html)** — a custom report published as a URL for programmatic consumption.
- **Workday Integration Cloud** — the integration-platform-as-a-service layer; EIBs (Enterprise Interface Builder) for scheduled data exchange.

### Permission model

Workday's security is organised around Security Groups and Business Process definitions. The pilot's integration system user sits in an Integration System Security Group with specific domain-security-policy permissions. Workday's model is powerful and intricate; coordinate with the customer's Workday admin team.

### Rate limits

Workday publishes limits per tenant in its documentation; most real-world pilots are more constrained by the SOAP transaction model than by raw QPS.

### Audit log

Workday captures audit history per object; integrations have per-run logs; the customer's HRIS governance team is used to this surface.

## Snowflake and Databricks

### Snowflake

Snowflake is the warehouse / data-cloud backbone for many pilots' analytical reads and eval-set extraction.

- **[Snowflake SQL API](https://docs.snowflake.com/en/developer-guide/sql-api/intro)** — REST surface for SQL execution.
- **[Snowflake connectors](https://docs.snowflake.com/en/user-guide/snowflake-connectors)** for Python / JDBC / ODBC.
- **[Cortex](https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions)** — Snowflake's in-platform LLM surface; the FDE may read from Snowflake and push to an external LLM, or may use Cortex when customer policy requires data never leaves Snowflake.

Permission model: [role-based access control](https://docs.snowflake.com/en/user-guide/security-access-control-overview). Row- and column-level policies via [row access policies](https://docs.snowflake.com/en/user-guide/security-row-intro) and [masking policies](https://docs.snowflake.com/en/user-guide/security-column-ddm-intro).

Rate limits are governed by the warehouse's compute size and queue; the pilot's reads consume credits against the warehouse the integration is assigned to. The FDE and customer agree on a pilot-dedicated warehouse to keep cost allocation clean.

Audit: [Account Usage schema](https://docs.snowflake.com/en/sql-reference/account-usage) exposes query history, login history, access history for SIEM-style ingestion.

### Databricks

Databricks is the lakehouse / data-+-ML backbone for many pilots.

- **[REST API](https://docs.databricks.com/api/)** for cluster / job / data operations.
- **[Databricks SQL](https://docs.databricks.com/aws/en/sql/)** for warehouse-style query.
- **Unity Catalog** is the governance layer — grants, row filters, column masks across workspaces.
- **[Databricks AI/BI Genie / Mosaic](https://www.databricks.com/)** are the in-platform AI surfaces.

Permission model is Unity Catalog grants + legacy workspace permissions; for pilots sitting on top of Unity Catalog, the integration user sits in a Unity Catalog group with specific grants.

Audit: [system tables and audit logs](https://docs.databricks.com/aws/en/admin/system-tables/audit-logs) are published centrally and consumable in Databricks SQL or exportable to SIEM.

## Google Workspace and Microsoft 365

Pilots that touch productivity surfaces — mail, drive, calendar, documents, Teams — integrate against these two platforms. Each publishes a sprawling API surface; the FDE opens three primary references per platform:

### Google Workspace

- **[Google Workspace APIs](https://developers.google.com/workspace)** — Gmail, Drive, Calendar, Admin SDK, Docs, Sheets, Slides.
- **[OAuth 2.0 for Google APIs](https://developers.google.com/identity/protocols/oauth2)** — auth is OAuth 2.0 with scoped per-API scopes.
- **Domain-Wide Delegation** — a service account impersonates a user in the customer domain; the customer's Workspace admin authorises the SA's scopes.

Permission model: per-user OAuth scopes; Admin SDK operations require a Workspace admin's delegation.

Rate limits: per-API quotas, documented per surface, with per-user and per-project limits.

Audit: [Admin audit log](https://support.google.com/a/answer/4579579) with per-API event records.

### Microsoft 365

- **[Microsoft Graph](https://learn.microsoft.com/en-us/graph/overview)** — one API surface for Outlook, Teams, SharePoint, OneDrive, Entra ID, Intune, and more.
- **[Graph permissions model](https://learn.microsoft.com/en-us/graph/permissions-reference)** — delegated (acting as a user) vs. application (acting as the service itself).
- **[Microsoft 365 Copilot](https://learn.microsoft.com/en-us/copilot/)** — Microsoft's in-platform AI surface; the FDE's pilot may surface actions to Copilot via connectors or Graph connectors.

Rate limits: per-service [Graph throttling](https://learn.microsoft.com/en-us/graph/throttling) with 429 responses and `Retry-After` headers; some APIs charge against a per-mailbox quota.

Audit: [Microsoft Purview audit](https://learn.microsoft.com/en-us/purview/audit-solutions-overview) captures Graph and tenant activity.

## HubSpot and Zendesk

### HubSpot

HubSpot is the CRM / marketing stack for many mid-market pilots.

- **[HubSpot API](https://developers.hubspot.com/docs/api/overview)** — REST, OAuth 2.0 or private app tokens.
- **Webhooks** for event-driven push.
- **[HubSpot workflows](https://developers.hubspot.com/docs/reference/api/automation/automation-workflows)** for in-platform automation that can call external services.

Permission model: scopes on the OAuth app / private app; per-user permissions in HubSpot.

Rate limits: per-app and per-portal; 429 responses with `X-HubSpot-RateLimit-*` headers.

Audit: audit log in the HubSpot portal; API event logs.

### Zendesk

Zendesk is the support / helpdesk backbone for many pilots focused on ticket triage, drafting, or knowledge-base augmentation.

- **[Zendesk REST API](https://developer.zendesk.com/api-reference/)** — Tickets, Users, Organizations, Help Center.
- **[Zendesk Webhooks](https://developer.zendesk.com/api-reference/webhooks/webhooks-api/webhooks/)** for event push.
- **[Zendesk Marketplace apps](https://developer.zendesk.com/documentation/apps/)** for in-platform surfacing of pilot features.

Permission model: per-user + agent-role; API tokens or OAuth 2.0.

Rate limits: [per-endpoint and per-account](https://developer.zendesk.com/api-reference/introduction/rate-limits/).

Audit: [audit log](https://support.zendesk.com/hc/en-us/articles/4408824293786-Viewing-the-audit-log-for-changes) retains admin and agent activity.

## Cross-cutting: permission, rate, audit

The three questions the FDE answers for every platform the pilot touches:

1. **Which specific scopes / roles / permissions does the integration require, and who on the customer side grants them?** The integration design cites the role or permission set, not "Admin."
2. **What rate limit applies to the pilot's calls, and what is the pilot's expected QPS under load?** The integration design contains a math check: expected QPS × time-horizon × error-margin fits within the allotted limit.
3. **Which audit log does the pilot's activity appear in, and who on the customer side consumes it?** The integration design names the audit surface and the governance-review subscriber.

A pilot that cannot answer all three for a given platform is not ready for the security review.

## Rate-limit arithmetic

A concrete sketch. Suppose the pilot:

- Serves 20 SMEs.
- Each SME processes an average of 30 cases per hour at peak.
- Each case requires 4 API calls to the customer's Salesforce org (fetch case, fetch member, fetch plan, update case).

Peak QPS to Salesforce ≈ (20 × 30 × 4) / 3600 ≈ 0.67 QPS. That is well inside any Salesforce edition's limits.

Now suppose the pilot batches an overnight eval-set extraction of 100,000 cases:

- Bulk API 2.0 handles that in a single job; the extraction runs on Salesforce's side against the Bulk limit, not the general API limit.
- The FDE schedules the job off-peak (Salesforce admin's preference) and the Bulk API returns a downloadable result file.

The rate-limit arithmetic fits on one page of the integration design, and the FDE updates it when the user base or case volume changes materially.

## Audit-log parity

The recurring enterprise question: "the pilot made 1,247 writes to Salesforce yesterday. For each one, which SME was it on behalf of?" If Chapter 03's token exchange is in place the answer is on the Salesforce `ApiEvent` record (via `act.sub` or the pilot's custom header carrying user context). If not, the pilot's own audit log must correlate by request-ID.

Decide at integration-design time which side of the correlation lives where, and whether the customer's governance team has access to it. A pilot that can only produce the correlation by having an engineer SSH into a logging host is a pilot whose governance review will insist on improvement before production.

## Common failure modes

**Requesting System Administrator because scoping is hard.** Resist. The customer's security reviewer will reject it, and even if they approve, the pilot's blast radius is now the whole platform. Scope to minimum roles; spend the 2–3 extra hours it takes.

**Hitting the rate limit in production because the limit was not budgeted at design time.** If the pilot's expected load has not been math-checked against the limit, you have an outage waiting for a volume uptick.

**Missing the audit log's export path.** The pilot's governance review asks for a SIEM-exportable audit log. The pilot's Salesforce calls show in `ApiEvent` but the customer's Salesforce admin never enabled Event Monitoring, which is a paid add-on. That is a six-week procurement, not a two-week task. Surface the requirement in the integration design, early.

**Treating Agentforce / Joule / Copilot / Cortex / Genie as the only option because the customer's platform vendor is pushing them.** Each has a place; each is also the platform vendor's upsell of an LLM capability the pilot may already be doing well with an external model + platform-API call. The decision is in mod-104's provider-selection work. Chapter 08 is where an MCP server fits when the agent surface lives inside the platform.

## Summary

- Ten SaaS platforms cover nearly every FDE-pilot integration: Salesforce, ServiceNow, SAP, Workday, Snowflake, Databricks, Google Workspace, Microsoft 365, HubSpot, Zendesk.
- For each, the integration design answers three questions: which permissions / scopes / roles does the pilot need (coordinate with the customer's platform admin); what rate limit applies and does the pilot's math fit it; which audit log records the pilot's activity and who on the customer side consumes it.
- Each platform exposes multiple API surfaces — pick the one that fits the use (REST for realtime, Bulk for extract, Streaming / CDC for push, SOAP where it is the only option).
- Agentforce / Joule / Copilot / Cortex / Genie are each platform-vendor LLM offerings. Treat them as options alongside external model providers; the decision is mod-104's. Where the customer wants the pilot surfaced inside the agent platform, Chapter 08 and mod-106 cover the shape.
- The common failure modes — over-broad permissions, un-budgeted rate limits, missing audit-log export, and vendor-LLM dependence by default — are all preventable in the integration design.
