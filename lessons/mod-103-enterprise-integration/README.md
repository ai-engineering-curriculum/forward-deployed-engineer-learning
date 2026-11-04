# mod-103-enterprise-integration: Enterprise Integration Engineering

> Scaffolded by `aicg org execute-plan`. Lecture chapters and exercise content are authored on subsequent autonomous cycles.

**Estimated effort:** 18 hours

## Learning objectives

- Wire the pilot into the customer's identity provider — SAML 2.0, OpenID Connect 1.0 (with PKCE where applicable), SCIM 2.0 user / group sync — against the big-4 IdPs (Okta, Entra / Azure AD, Ping, Google Workspace) and the long-tail (OneLogin, Auth0, JumpCloud, ADFS)
- Author service-to-service auth — OAuth 2.0 client-credentials, mTLS, token-exchange (RFC 8693), short-lived service tokens (STS / Workload Identity / Pod Identity) — against the customer's service mesh or API gateway
- Integrate with the customer's data surface — REST (OpenAPI 3.1), GraphQL, webhooks, CDC, event buses (CloudEvents on Kafka / PubSub / EventHub / SNS/SQS), and batch (SFTP / S3 / Snowflake external stages) — each with the appropriate schema registry and lineage story
- Integrate with the customer's SaaS platforms — Salesforce (REST / SOAP / Bulk / AgentForce actions), ServiceNow (Table API / IntegrationHub / Now Assist), SAP (OData / Joule), Workday (RaaS / Integration Cloud), Snowflake / Databricks / Google Workspace / Microsoft 365 / HubSpot / Zendesk — understanding each platform's permission model, rate limits, and audit-log shape
- Integrate with the customer's legacy surface — SOAP, SFTP, batch jobs, mainframe gateways, screen-scraping — with explicit cost / reliability / latency trade-offs documented against the LLM-feature latency budget
- Design the customer-side networking posture — PrivateLink / VPC endpoints, VPC peering, Site-to-Site VPN, egress proxying, split tunneling — against the customer's security review checklist
- Expose a Model Context Protocol (MCP) server to a customer agent surface when the integration shape calls for it — spec-compliant server implementation, auth, resource listing, tool capability advertisement, and the customer's governance review checklist for MCP
- Cite the boundary to `ai-infra-inference-engineer` (peer, level 35) on model-server deployment inside the customer VPC; cite the boundary to `ai-data-engineer` (peer) on customer-side ETL / ingestion pipelines

## Structure

- `01-…md` … `0N-…md`: lecture chapters.
- `exercises/`: per-exercise prompts.
- `labs/`: long-form hands-on labs.
- `quizzes/`: knowledge checks.
- `resources.md`: external references.
