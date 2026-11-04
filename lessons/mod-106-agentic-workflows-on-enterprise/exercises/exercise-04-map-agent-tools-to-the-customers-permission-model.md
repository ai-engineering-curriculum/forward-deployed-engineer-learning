# exercise-04: Map Agent Tools to the Customer's Permission Model

**Estimated effort:** 3 hours

## Objective

Produce a complete per-tool identity, scope, and audit-record specification for each of the three archetypes — the document the customer's AppSec reviewer will read in week 4–5 of the pilot and the schema the pilot's runtime will emit to the customer's SIEM. The output is the formal expression of Chapter 05's least-privilege-per-tool-call discipline.

## Prerequisites

- Read [Chapter 05](../05-tool-permissions-and-audit.md) end to end.
- Have exercise-03's checkpoint tables to hand — the "approver" column there feeds the "identity" column here.
- Skim the identity references in `resources.md`: [RFC 8693 (Token Exchange)](https://datatracker.ietf.org/doc/html/rfc8693), [Microsoft Entra OBO](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-on-behalf-of-flow), and whichever cloud-native workload-identity story (AWS IRSA, GCP WIF, Azure Managed Identity) is relevant to your archetype.
- Optional: [OWASP Non-Human Identities Top 10](https://owasp.org/www-project-non-human-identities-top-10/) to inform the NHI inventory.

## Problem statement

For each archetype, produce:

1. **The permission map** — one row per tool from exercise-03's catalogue. Columns: subject (end user via OBO / service account / per-tenant application identity), scope requested (specific OAuth scopes / IAM permissions), scope lifetime (session / per-call / short-lived), identity attestation pattern (OBO / downscoped STS / Managed Identity / etc.), NHI (if applicable: name, vault path, rotation cadence).
2. **The scope-expansion flow** — when does the session's scope expand beyond read-only? Diagram or describe: trigger (a HITL confirm on an escalating action), how the orchestrator requests the expanded scope, how long the expanded scope lives, how it is revoked. Reference the specific IdP flow your archetype uses.
3. **The per-tenant tool allow-list** — for each archetype, pretend there are two tenants (T1 and T2) with slightly different contracted scope. T1 has the full catalogue; T2 is missing one write tool. Show how the allow-list is represented in configuration and enforced in the orchestrator.
4. **The audit-record schema** — a complete JSON Schema or equivalent, with every field from Chapter 05's table that applies, plus any archetype-specific fields (e.g., `classification_tier` for Archetype B, `case_region` for Archetype C). Include at least three example records: a successful tool call, a HITL-confirmed tool call, and a tool call that queued.
5. **The NHI inventory** — the service accounts and workload identities the pilot introduces. For each: purpose, owner, scope, rotation cadence, vault location, workload-identity-attestation mechanism.
6. **The drift-detection sketch** — a short description of how the pilot would reconcile effective permissions against the signed manifest. What checks run on what cadence? What triggers a page?

## Requirements

Three completed permission specifications. Each specification:

- Has one row per tool covering subject, scope, lifetime, attestation pattern, NHI.
- Shows OBO (Pattern A from Chapter 05) as the default for any tool that acts on behalf of an identified end user. If a tool cannot use OBO, say why and name the downscoped-service-account (Pattern B) alternative.
- Has scope-expansion triggered by HITL confirmation from exercise-03, not by model confidence.
- Represents the per-tenant allow-list in a reviewable configuration format (YAML, JSON, TOML — pick one). The configuration must be the source of truth; the orchestrator should fail-closed if the tenant is unknown or the tool is not in the allow-list.
- Has an audit-record schema with every field a customer's SOC would need to reconstruct an incident: event_id, timestamp, tenant_id, session_id, user_id, agent_version, pattern, step_index, tool_name, tool_input (with sanitisation where required), tool_output_hash, identity_used, scope_used, hitl_checkpoint, hitl_approver, hitl_timestamp, rationale_ref, inputs_ref, result_status, error_detail.
- References the specific SIEM destination (Splunk / Sentinel / Chronicle / Elastic / Datadog) for the archetype, with a one-line note on the ingest format.
- Specifies retention per the archetype's regime: HIPAA retention for A (6 years baseline; may be longer under state law — flag with `<!-- needs-research: ... -->` where unclear); NARA disposition-schedule-specific for B; GDPR + DORA + record-of-processing retention for C.
- Names the NHIs introduced and their attestation; no static shared secrets allowed in the design.
- Describes drift detection with a cadence that fits the archetype's change velocity.

## Starter guidance

- **OBO is the default when there is an end user.** If your design has a service account doing the work for a named user, you have the wrong pattern. OBO through the customer's IdP is the correct shape; the audit record will show the user by identity, not just by session.
- **The expanded scope's lifetime is the design knob.** Chapter 05 argues for bounding it to the specific action it was granted for. Document the specific lifetime per expanded-scope tool (e.g., `write:case:triage` valid for 30 seconds, single-use, bound to a specific case ID).
- **The allow-list belongs in the orchestrator, not in the prompt.** The model should not see tools that the current tenant has not authorised. Enforcement is deterministic, not prompt-driven.
- **The audit record is a legal artefact.** The customer's security team should be able to reconstruct a three-step agent trajectory from the records alone. If they cannot, the records are missing fields. Walk through an incident mentally — "the agent filed a prior-auth correction that was later disputed" — and verify every field the security team would ask for is present.
- **Hashes vs. full payloads.** Full tool outputs can contain PII the retention policy forbids in the audit stream. Hash the outputs; store the full payloads in a shorter-retention store with tighter access.
- **The NHI inventory should be small.** A pilot with 20 NHIs has a scope problem. Aim for 2–5; justify each.
- **Workload-identity attestation is the shape, not static keys.** Static API keys are the OWASP NHI anti-pattern. GCP Workload Identity Federation, AWS IRSA, Azure Managed Identity, SPIFFE/SPIRE — pick the one that fits the archetype's deployment model.
- **Right-to-erasure composes with retention.** For Archetype C, the design must show how agent memory supports per-record deletion while the audit records are retained under legal-basis-of-processing. These are two different retention policies, not one.

## Acceptance criteria

- Three complete permission maps, one per archetype, covering every tool from exercise-03.
- OBO is the subject pattern for every tool that acts on behalf of an identified user. Deviations are explicitly justified.
- A scope-expansion flow per archetype with HITL as the trigger and a bounded lifetime.
- A per-tenant allow-list in configuration format, enforced by the orchestrator, with a fail-closed semantics note.
- A complete audit-record schema per archetype with three example records.
- A named SIEM destination per archetype with a retention statement tied to the archetype's regime.
- An NHI inventory per archetype with 2–5 entries and workload-identity attestation (not static keys).
- A drift-detection sketch per archetype.
- Every unverifiable regulatory or customer-specific claim tagged `<!-- needs-research: ... -->`.
- The documents are reviewable by a security engineer in one sitting — concise, schema-shaped, with no hand-waving about "the IdP will handle it."

## Stretch goals

- Wire the audit-record schema into exercise-02's agent — replace the placeholder `TODO(exercise-04)` identity/scope fields with real values drawn from a mocked IdP (a stub that returns deterministic claims). The agent's log should match the archetype's schema exactly.
- Produce a *detection-rule sketch* for the SIEM: name three specific alerts a SOC would build on top of the audit stream (e.g., "same session executes scope-expansion more than N times in 10 minutes," "tool call succeeds without a prior HITL event on a tool that requires HITL," "NHI used outside its declared workflow").
- Design the *access-review artefact* the customer's security team would produce monthly from the pilot's audit stream — the list of effective permissions, the NHI usage counts, the drift from the manifest.
- For Archetype B, produce the *classification-tier enforcement* design: how does the agent ensure that a tool-call never returns content above the current session's clearance level? This composes with mod-105 Chapter 03's permission propagation.
- For Archetype C, produce the *right-to-erasure runbook*: a GDPR erasure request arrives; walk through what happens to agent memory, audit records, and any derivative content. Name the retention policy that keeps the audit records under legal-basis-of-processing.
