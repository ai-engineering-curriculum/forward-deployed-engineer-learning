# Chapter 05 — Tool Permissions, Scoping, and the Audit Trail

## Why this chapter exists

Chapter 04's HITL table names the *human approver* per action. This chapter names the *identity under which the action executes*, the *scope that identity carries*, and the *audit record* that proves what happened. Together they are the mechanical layer under HITL.

The question the customer's security team will ask on the pilot's first security review: "When the agent files a ticket under user Alice's session, under which identity does the ticket-system API call execute, which permissions were in scope, and what is the audit record I can hand to legal if she disputes the action?" If the FDE's answer is "the agent uses a service account," the pilot has already failed the review. This chapter teaches the shape of a defensible answer.

Depth on identity-and-access-management is a specialist peer craft — [`cloud-security-engineer`](https://github.com/ai-engineering-curriculum) in some orgs, or the customer's own IAM team. The FDE's job is to design the agent-to-customer-system boundary correctly and brief the peer for depth. This chapter composes with mod-103 Chapter 02 (identity surface) and mod-108 (enterprise security formal review).

## Least privilege, per tool call

The default is wrong: a single service account that owns every tool the agent might call. The right default: the tool executes with **the thinnest identity that can perform this specific call for this specific user**. "Thin" is three dimensions:

1. **Subject.** Who is the call made for? The end user (via on-behalf-of / token-exchange), a specific service account, a tenant-scoped application identity.
2. **Scope.** Which permissions are active for this call? The user's full scope, a reduced scope ("read:cases only"), or a specific delegation.
3. **Window.** How long is this identity valid? A per-session token, a per-call token, a short-lived credential.

Three patterns realise least privilege:

### Pattern A — On-behalf-of (OBO) token exchange

The agent inherits the end user's identity by exchanging the user's token for a downstream-system token that identifies the user. The downstream system sees the user, applies the user's permissions, logs under the user's identity.

- **Standards.** [OAuth 2.0 Token Exchange (RFC 8693)](https://datatracker.ietf.org/doc/html/rfc8693); [OpenID Connect token binding / audience constraints](https://openid.net/specs/openid-connect-core-1_0.html).
- **Microsoft Entra (Azure AD)** exposes the OBO flow natively; see [Microsoft identity platform and OAuth 2.0 On-Behalf-Of flow](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-on-behalf-of-flow).
- **Okta** supports token exchange via [Okta's documentation on OAuth 2.0 Token Exchange](https://developer.okta.com/docs/guides/set-up-token-exchange/main/).
- **Google Cloud** implements a similar pattern via [Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation) for machine-to-machine; end-user OBO goes through Google Identity's ID-token exchange.

**When this pattern wins.** The agent acts on behalf of an identified end user whose permissions should bound the action. Every CSR-assistant, analyst-assistant, case-worker-assistant pilot lands here.

**Enterprise-overlay implication.** This is almost always the right default when there is an end user to inherit from. Audit records naturally carry the user's identity. Permission changes to the user propagate immediately — no agent-side cache to invalidate.

### Pattern B — Service account with permission downscoping

The agent calls under a service account (because there is no end user — a batch job, a scheduled summarisation — or because the downstream system does not support OBO), but the service account's permissions are *downscoped per call* to the minimum the current task needs.

Mechanics:

- Single long-lived service identity; permissions requested per call via scoped tokens, [AWS STS `AssumeRole`](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html) with a narrow policy, GCP [service account impersonation](https://cloud.google.com/iam/docs/service-account-impersonation) with scoped privileges, or Azure [workload identity](https://learn.microsoft.com/en-us/azure/aks/workload-identity-overview) with per-request scoping.
- The policy attached to the per-call token references only the specific resources the current step needs (a specific S3 prefix, a specific case ID, a specific project).

**When this pattern wins.** Batch / scheduled workflows; ingest pipelines; workflows the customer explicitly runs under a named agent identity.

**Enterprise-overlay implication.** The audit record identifies the service account, not the user. The runtime must emit a *correlation record* that maps the service-account call back to the requesting user (where there is one). Without that, the audit trail is unusable.

### Pattern C — Per-tenant application identity

The agent runs across many tenants; each tenant has its own application identity registered in their IdP / cloud account. Tool calls for tenant T use tenant T's application identity.

- **SaaS multi-tenant.** The vendor's agent runs with per-tenant OAuth grants; each tenant's grant gives access only to that tenant's resources.
- **VPC install.** The agent runs *inside* each tenant's environment; the identity is scoped by the deployment itself.

**When this pattern wins.** The agent itself is a tenant-aware service; the downstream systems do not know about end users directly but do know about tenants.

**Enterprise-overlay implication.** Combines with Patterns A/B at the tool-call layer — the per-tenant application identity is the *agent's* identity; the per-call identity for the specific tool call is still OBO or downscoped service account.

### The decision tree

- Is there an end user whose permissions should bound the action? → Pattern A (OBO).
- Is this a batch / scheduled / service-initiated action? → Pattern B (downscoped service account).
- Is the agent itself multi-tenant and the downstream systems tenant-aware? → Pattern C composed with A or B.

Patterns are not mutually exclusive; most real pilots compose two or three.

## Per-session scoping

Within a single user session, not all tool calls need the same scope. A customer-service session that starts in *draft reply* mode and later escalates to *file a correction* mode should not carry write-scope tokens the entire session. The scope expansion happens *at the moment the agent proposes the escalating action*, after HITL confirmation (Chapter 04), and the scoped token is minted for that specific call.

Three disciplines:

1. **Session start with read-only scope.** The agent's initial token grants only what retrieval needs.
2. **Scope expansion is a step, not a prerequisite.** The orchestrator requests the expanded scope at the moment of action, capturing the user's consent (HITL confirm) as the authorisation signal.
3. **Scope contraction is automatic.** The expanded scope's lifetime is bounded to the action it was granted for.

This maps directly onto [OAuth 2.0 incremental authorization](https://datatracker.ietf.org/doc/html/rfc6749) patterns and modern IdP step-up authentication flows. It matters because *any token the agent holds is a token an injected prompt could try to abuse* (Chapter 06). Minimising the window of high-privilege tokens minimises the blast radius.

## Per-tenant tool allow-lists

Not every tenant wants every tool. Tenant A's contract allows the agent to file prior-auth corrections; Tenant B's does not; Tenant C's allows only in shadow mode. Per-tenant tool allow-lists enforce this.

**Where the allow-list lives.** A tenant configuration store, read at session start, referenced by the orchestrator's tool-dispatch layer. The list is *enforced* by the orchestrator — not by the model. The model should not have a tool in its catalogue that the current tenant has not allowed.

**Where the allow-list is set.** The contract. The customer's contracted scope — what the pilot is authorised to do for this tenant — is the source of truth for the allow-list. It is the FDE's responsibility to keep the allow-list and the contracted scope aligned.

**Why not let the model see all tools and refuse the forbidden ones.** Three reasons:
- Prompt injection could override the refusal.
- The model's refusal is non-deterministic.
- Showing tools in the catalogue the user is not authorised to call leaks capability intelligence.

**Implementation.** The orchestrator's tool catalogue is parameterised by tenant. At session start, the orchestrator loads `tools_for_tenant(tenant_id)` and the model sees only that subset. Any later scope expansion (Section above) also checks the allow-list.

## The audit trail

Every action the agent takes must leave a record the customer's security team can subpoena. "Record" is not "logged somewhere" — it is a specific shape with specific guarantees.

### What a tool-call audit record contains

Minimum fields per tool call:

| Field | Example |
|---|---|
| `event_id` | UUID, unique per call |
| `timestamp` | ISO 8601 with timezone, monotonic from server clock |
| `tenant_id` | Which customer tenant |
| `session_id` | The user's session |
| `user_id` | The identified end user (or `null` with reason) |
| `agent_version` | The pilot's deployed agent version |
| `pattern` | Which Chapter 02 pattern was active |
| `step_index` | Which step in the chain / plan |
| `tool_name` | e.g., `file_prior_auth_correction` |
| `tool_input` | The serialised parameters (with Chapter 06 sanitisation) |
| `tool_output_hash` | Hash of the tool's output (full output stored separately if volume allows) |
| `identity_used` | The identity the tool call executed under (OBO, service account, per-tenant app) |
| `scope_used` | The specific permissions/scopes active for this call |
| `hitl_checkpoint` | `auto_run`, `confirmed`, `clarified`, `queued` |
| `hitl_approver` | The user who confirmed (when applicable) |
| `hitl_timestamp` | When the confirmation happened |
| `rationale_ref` | Pointer to the model's rationale for this call (stored separately) |
| `inputs_ref` | Pointer to the retrieval evidence / prior turns that drove this call |
| `result_status` | success / error / skipped |
| `error_detail` | Structured error if applicable |

The record is *write-once*. The pilot's architecture should prevent later edits to the audit stream.

### Where the audit trail lives

- **SaaS multi-tenant.** Vendor-run audit pipeline with per-tenant retention; records exportable to the customer's SIEM on demand or on a schedule.
- **Dedicated SaaS.** Audit stream can be routed directly to the customer's SIEM in near-real-time.
- **VPC install.** Audit stream lives entirely in the customer's environment, ingested into their Splunk / Datadog / Elastic / Chronicle / Sentinel deployment.

Common destinations: [Splunk Enterprise](https://www.splunk.com/), [Datadog](https://www.datadoghq.com/), [Elastic SIEM](https://www.elastic.co/security/siem), [Microsoft Sentinel](https://azure.microsoft.com/en-us/products/microsoft-sentinel), [Google Chronicle](https://cloud.google.com/chronicle-security-operations). Each has its own ingest shape; the FDE pins the format early so the customer's SOC can build detections.

### What the audit trail must *not* contain

- Raw PII that should have been redacted before logging (mod-105 Chapter 04 governs; the gateway from mod-104 Chapter 03 is the enforcement layer).
- Full tool-call outputs that include customer PII unless the retention policy permits it; prefer hashes with separate (shorter-retention) storage of the full payload under a stricter access policy.
- Model system prompts in plaintext if they contain vendor proprietary content; hash or reference by version.

### The Chapter 04 integration

Every HITL event (confirm / clarify / queue) is an audit record too, with its own fields (who approved, how long they took, whether they edited the proposal). The audit trail is the end-to-end story: *user asks → agent plans → human confirms → tool runs → result recorded*.

## Non-human identity (NHI) hygiene

The agent's identities — service accounts, machine credentials, API keys to downstream systems — are *non-human identities*. The customer's security team now treats NHI as a first-class identity class. The FDE's obligations:

- **Storage.** In the customer's secrets vault — HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager — never in config files, environment variables managed outside the vault, or source-control.
- **Rotation.** On a cadence the customer sets. The pilot's architecture must support rotation without downtime: either dynamic secret issuance from the vault, or a rolling deployment that can pick up new credentials.
- **Attestation.** The identity should be tied to a workload-identity proof (not a shared static secret). GCP Workload Identity Federation, AWS IAM Roles for Service Accounts (IRSA) in EKS, Azure Managed Identity, SPIFFE/SPIRE in self-hosted environments.
- **Scope.** NHI identities get least-privilege scope exactly as user identities do. The service account that posts a Jira comment does not need read-write access to every project.
- **Inventory.** The customer's security team maintains an inventory of NHIs. The pilot adds to it; the inventory record names the identity, its purpose, its scope, its rotation cadence, its owner.

See [OWASP Non-Human Identities Top 10](https://owasp.org/www-project-non-human-identities-top-10/) for the taxonomy of NHI-specific risks.

## Authorisation drift

Permissions change. Users change roles, documents get reclassified, tools get retired, tenants renegotiate scope. *Authorisation drift* is the gap between the permissions the pilot was reviewed against and the permissions it currently has. Three disciplines prevent drift:

**Drift detection.** A scheduled job that reconciles the pilot's current effective permissions (service account scopes, per-tenant allow-lists, NHI inventory) against a signed manifest of what the pilot *should* have. Diffs page the FDE.

**Permission invalidation.** When a user changes role or leaves, their active sessions should invalidate. When a tool is removed from a tenant's allow-list, in-flight actions using that tool should fail-closed. The pilot's architecture should not cache permission decisions beyond their natural lifetime.

**Periodic access review.** On a cadence (monthly or quarterly), the customer's security team reviews the NHI inventory, the per-tenant allow-lists, and the pilot's effective permissions. The FDE participates.

## Running the three archetypes

**Archetype A — healthcare payer.** Primary identity pattern: OBO for CSR actions (file prior-auth correction, post case note) via Azure Entra or Okta (whichever the customer uses) through the ticketing and claims APIs. Scope: read + draft at session start; write scope expanded on HITL confirm. NHI: a single service account for retrieval into the policy index with read-only access, per-tenant separation. Audit: records streamed to the customer's Splunk, HIPAA-compliant retention (6 years, per customer's retention policy; may be longer per state law).

**Archetype B — federal agency.** Primary identity pattern: OBO for analyst actions (read documents at the analyst's clearance level) through the agency's federated IdP. Scope: read-only in pilot; later-phase write-to-analyst-draft only. NHI: minimal — one service account for retrieval over the policy corpus, scoped per index. Audit: records streamed to the agency's audit system; records must satisfy NARA retention schedules. <!-- needs-research: confirm NARA requirements for AI-assisted analyst drafts at agency-specific disposition schedules -->

**Archetype C — EU retail bank.** Primary identity pattern: OBO for case-worker actions (update triage status, route to ombudsman queue) via the bank's SSO. Scope: read + draft at session start; write scope expanded on HITL confirm. NHI: per-tenant application identity for the agent service itself; per-call service accounts for retrieval. Audit: records to the bank's Sentinel deployment; GDPR retention bounded by the bank's documented retention schedule; right-to-erasure design covers memory and audit records distinctly (audit records are typically retained under legal-basis-of-processing even when memory is erased).

## Anti-patterns

**One service account, all permissions.** The classic failure: the agent runs as `agent-svc` with read-write on everything the pilot might ever call. The security review stops here. Design OBO and downscoping from day one.

**Tool catalogue not parameterised by tenant.** The agent "has" the tool in its prompt; the orchestrator relies on refusal. Prompt injection breaks that. Parameterise the catalogue.

**Audit log that reads "agent executed tool X".** With no user, no tenant, no scope, no HITL evidence. The security team cannot use this. Build the full record from day one; it is harder to retrofit than to write.

**Secrets in environment variables managed outside the vault.** The default of many Docker / Kubernetes examples. Pull from the vault at runtime; rotate via the vault's lifecycle.

**No drift reconciliation.** The pilot's effective permissions drift over quarters and nobody notices until the quarterly review surfaces it. Automate the reconciliation; page on diffs.

**Confusing "audit log" with "application log".** The application log is the FDE's debugging aid. The audit log is a legal artifact. Separate the pipelines; separate the retentions; separate the access policies.

## Where this chapter sits with the rest

- **Chapter 04** named *who* approves; this chapter names *what identity* executes and *what* the audit record carries.
- **Chapter 06** names the threat-model mitigations that compose with the identity story — prompt injection against scope-expansion requests, tool-abuse under a leaked NHI, confused-deputy attacks.
- **mod-103 Chapter 02** named the customer's identity surface; this chapter is where that surface meets action.
- **mod-108** formalises the security-review package built on this chapter's artifacts.
- **mod-110** instruments the audit trail — detection rules, access anomaly alerts, out-of-policy tool-call alarms.

## Summary

- Least-privilege-per-tool-call is three dimensions: subject (who), scope (which permissions), window (how long). OBO (Pattern A), downscoped service account (Pattern B), per-tenant application identity (Pattern C) compose to realise it.
- Session scope starts narrow and expands only on HITL confirmation; expanded scope has a bounded lifetime.
- Per-tenant tool allow-lists are enforced by the orchestrator, not by the model. The catalogue the model sees is pre-filtered.
- The audit trail has a specific shape: per-tool-call records with identity, scope, HITL evidence, rationale references. Write-once, streamed to the customer's SIEM.
- NHI hygiene — vault-stored, rotation-capable, workload-identity-attested, scoped, inventoried — is a first-class security discipline the pilot inherits.
- Authorisation drift is prevented by drift detection, permission invalidation, and periodic access review.
- Three archetypes land on composed OBO + downscoped NHI + tenant-scoped audit, with per-regime retention rules.
- Chapter 06 is next: the OWASP Agentic Top 10 threat model and concrete mitigations layered on top of this chapter's identity and audit story.
