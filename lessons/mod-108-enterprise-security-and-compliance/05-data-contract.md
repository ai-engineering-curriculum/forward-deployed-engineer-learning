# Chapter 05 — Negotiating the Data Contract — Zero-Retention, Residency, BYOK, Training-Opt-Out, and the DPA Shape

## Why this chapter exists

Chapter 02 produced the DFD and the sub-processor list. Chapter 03 named which statutes apply. Chapter 04 named which threats need mitigating. Those three combine into one artefact the customer's privacy office, compliance office, and legal team all read: the **data contract** — the Data Processing Addendum (DPA), its vertical overlays (BAA for HIPAA, EU Standard Contractual Clauses for GDPR cross-border, service-provider addendum for CCPA), and the engineering attestations that prove the vendor can deliver on the clauses.

A DPA is a legal document, but every clause in it is an engineering commitment. "No PHI is retained by the model provider" is a contract term and an architectural fact. "Customer-managed keys protect data at rest" is a legal promise and a KMS configuration. "Training-opt-out is in effect for this account" is a sentence in the DPA and an account-level checkbox in the model provider's console. The FDE's job is to make sure every clause the lawyers negotiate corresponds to a configuration, a code path, or a log the engineering team can point to.

This chapter walks the six engineering levers in a modern enterprise AI DPA — retention, residency, keys, prompt / completion storage, training opt-out, audit retention — and the architecture that makes each one true, not aspirational.

## The six levers of the engineering-side DPA

Every AI-vendor DPA in 2026 has to answer six questions in a way the customer's privacy office can test:

1. **Retention of customer data by the model provider.** Does the model provider retain prompts or completions beyond the request lifecycle? If yes, for how long, for what purpose (abuse monitoring, safety research), with what access controls?
2. **Data residency.** Where does the processing physically happen? Where is the data at rest? Which sub-processors can see the data outside that region?
3. **Encryption keys.** Who holds the keys at rest, in transit, in use (where supported)? Is the customer's KMS integrated (BYOK / CMK), and what happens if the customer rotates or revokes the key?
4. **Prompt and completion storage inside the vendor's own stack.** The vendor's own logs, the vendor's eval harness, the vendor's observability pipeline — how long does each retain the prompt or completion, and what is redacted before it stores?
5. **Training opt-out.** Is customer data used — ever — to train or fine-tune any model, by the vendor or by the model provider? What is the attestation shape, and what verifies it?
6. **Audit-log retention.** How long are audit logs retained, who can read them, under what retention and legal-hold rules, and how are they produced to an auditor or a subpoena?

Each lever has a contract clause and an architecture that implements it. The failure mode is the clause without the architecture.

## Lever 1 — Zero-retention with the model provider

**The commitment.** Prompts and completions are processed by the model provider's inference endpoint and not persisted beyond the request lifecycle.

**The defaults you must know.**

- **AWS Bedrock.** Does not store prompts or completions by default; does not use customer data to train or improve the service; FedRAMP and HIPAA eligibility per region. Reference: [AWS Bedrock Data Protection](https://docs.aws.amazon.com/bedrock/latest/userguide/data-protection.html).
- **Azure OpenAI.** Prompts and completions are stored by Microsoft for up to 30 days by default for abuse monitoring; customers can apply for the "Limited Access Review" / "Modified Abuse Monitoring" exception to disable abuse-monitoring storage. Reference: [Azure OpenAI Data, Privacy, and Security](https://learn.microsoft.com/en-us/legal/cognitive-services/openai/data-privacy).
- **Google Vertex AI.** Does not use prompts or completions to train foundation models by default; prompts and completions are logged in the customer's Google Cloud project per customer's logging settings. Reference: [Vertex AI generative AI data governance](https://cloud.google.com/vertex-ai/generative-ai/docs/data-governance).
- **Anthropic (direct API).** Enterprise plans support zero-retention with abuse-monitoring waiver under the Zero Data Retention Addendum. Reference: [Anthropic privacy and security](https://www.anthropic.com/legal/privacy).
- **OpenAI (direct API).** Enterprise / Team plans do not use customer data for training by default; abuse-monitoring retention (typically 30 days) is waivable under enterprise agreement. Reference: [OpenAI Enterprise Privacy](https://openai.com/enterprise-privacy/).
<!-- needs-research: verify current abuse-monitoring retention windows and ZDR terms with each provider's current enterprise agreement text before publishing — these change with the current enterprise agreement template -->

**The architectural lever.** The vendor's gateway (mod-104 Chapter 03) is the single enforcement point.

```python
# gateway/policy/retention.py — illustrative
class RetentionPolicy:
    def inspect_route(self, request: InferenceRequest) -> None:
        provider = request.provider  # 'bedrock' | 'azure_openai' | 'vertex' | 'anthropic_api' | 'openai_api'
        tenant = request.tenant_id

        # Zero-retention attestation table — committed in the DPA
        if not self.policy[tenant].zero_retention_provider_allowlist[provider]:
            raise PolicyViolation(
                f"tenant {tenant} DPA does not permit {provider}; no ZDR attestation on file"
            )

        # Verify the request path goes through the ZDR-attested endpoint
        if not request.endpoint.startswith(self.policy[tenant].attested_endpoint[provider]):
            raise PolicyViolation(
                "request endpoint does not match ZDR-attested endpoint for tenant"
            )
```

The clause the DPA writes:

> *"Model Provider does not retain Customer Data beyond the lifecycle of the inference request. Where Model Provider's default retention includes abuse monitoring (e.g., Azure OpenAI's 30-day default), Vendor shall ensure the applicable waiver / Modified Abuse Monitoring / Zero Data Retention Addendum is in effect for Customer's account and shall provide a current attestation on request."*

The evidence the engineering team produces:

- The current ZDR / abuse-monitoring-waiver attestation PDF from the provider for this account.
- The gateway policy that routes only to ZDR-attested endpoints.
- A log sample showing the gateway's retention-policy enforcement.

## Lever 2 — Data residency

**The commitment.** Processing and storage of customer data occurs only in a defined geography.

**The commercially common residency shapes.**

- **US (default for Archetype A).** All processing in US regions; sub-processors confirmed US.
- **EU Data Boundary (Archetype C).** Azure's EU Data Boundary commitment; AWS European Sovereign Cloud; Google Cloud EU region plus Assured Workloads for EU. Processing, storage, and operational access all inside the EU.
- **In-country (sovereign).** Germany, France, Switzerland, UK, UAE, KSA, Singapore — country-boundary residency; requires in-country regions and in-country support operations.
- **GovCloud (Archetype B).** AWS GovCloud (US) / Azure Government / Google Government Cloud. FedRAMP High-eligible regions with US-citizen personnel operational access.
- **On-prem / air-gapped.** No cross-boundary flow at all.

**The architectural lever.** Three places residency must be enforced simultaneously:

- **Inference call routing.** Gateway configuration pins the model provider's endpoint to the residency-correct region.
- **Storage.** Vector store, document store, audit log, eval gold set — all configured to the residency-correct region, with region-lock enforced in IaC.
- **Sub-processor flows.** Every sub-processor the DFD names must have a residency-correct endpoint (an embedding API's US endpoint will silently break EU residency).

Terraform fragment illustrating region-lock discipline:

```hcl
# infra/eu-pilot/main.tf — illustrative
locals {
  pilot_region = "eu-west-1"
  allowed_regions = ["eu-west-1", "eu-central-1"]
}

resource "aws_s3_bucket" "pilot_data" {
  bucket = "customer-pilot-eu"
  # Fails the plan if a US region is chosen
  lifecycle {
    precondition {
      condition     = contains(local.allowed_regions, local.pilot_region)
      error_message = "EU residency violation: ${local.pilot_region} not in allow-list"
    }
  }
}

resource "aws_bedrock_guardrail" "pilot_guardrail" {
  provider = aws.eu_west_1
  # ...
}
```

Clause:

> *"All Customer Data shall be processed and stored only in [Region Set]. Vendor shall maintain architectural controls (region-locked resources, region-pinned API endpoints) preventing processing outside [Region Set], and shall provide evidence of these controls on request."*

Evidence:

- Region-lock IaC with the precondition that fails in CI if a resource is placed outside the allow-list.
- Sub-processor list columns confirming each sub-processor's residency.
- Gateway logs showing endpoint pinning.

## Lever 3 — BYOK / CMK and key management

**The commitment.** Customer-managed keys protect customer data at rest; the vendor cannot read customer data without the customer's cooperation.

**The patterns.**

- **BYOK (Bring Your Own Key).** The customer generates or supplies the key material; the key lives in the vendor's KMS or the cloud provider's KMS; the customer can rotate.
- **CMK (Customer-Managed Keys).** The customer's own KMS owns the key; the vendor's compute accesses it via a cross-account grant or key-use permission. Revocation is immediate.
- **HYOK (Hold Your Own Key) / confidential computing.** The key never leaves the customer's HSM / trusted execution environment; decryption happens inside confidential computing hardware (Intel SGX, AMD SEV, AWS Nitro Enclaves). Rare but required for the most sensitive deployments.
<!-- needs-research: confirm the current commercial availability of confidential-computing options across Bedrock, Azure OpenAI, and Vertex AI (Nitro Enclaves, Azure Confidential Computing, Google Confidential VMs) before publishing — these services are expanding -->

**The architectural lever.** Every data-at-rest surface the DFD names has to carry a key policy:

| Surface | Encryption | Key owner | Rotation |
|---|---|---|---|
| S3 / Blob / GCS bucket | AES-256 SSE | Customer KMS | 90-day |
| Postgres / vector DB | TDE | Customer KMS | 180-day |
| Audit log (Splunk / Sentinel / Elastic) | At-rest | Customer SIEM key | SIEM-aligned |
| Eval gold set | AES-256 | Customer KMS | 90-day |
| Model cache (if any) | AES-256 | Customer KMS | 90-day |
| Secrets (API keys, tokens) | Vault / KMS wrap | Vault (customer-side) | 30-day |

Clause:

> *"Customer Data shall be encrypted at rest using Customer-Managed Keys ('CMKs') stored in Customer's AWS KMS / Azure Key Vault / Google Cloud KMS. Vendor shall not retain unencrypted copies of Customer Data outside the authorised processing path. Customer may rotate or revoke CMKs; Vendor shall support zero-downtime rotation and shall, on revocation, cease processing within [N] hours and provide evidence of cessation."*

Evidence:

- KMS key policy showing vendor's role as consumer, customer as owner.
- Rotation runbook, tested on dates, archived.
- Revocation runbook, tested on dates, archived.
- CloudTrail / Azure Monitor / Cloud Audit Logs entries showing key-use attribution.

## Lever 4 — Prompt and completion storage inside the vendor's own stack

**The commitment.** The vendor's own logs, eval harness, and observability pipeline do not retain prompts or completions beyond a named, short window, and what they do retain is redacted of sensitive data classes.

**The failure mode.** The DPA says zero-retention with the model provider, but the vendor's own observability stack captures the full prompt + completion in a Datadog log, in a LangSmith trace, in a Humanloop record, in a Promptfoo log. The customer's auditor asks "where else does this live?" and the FDE discovers three stores nobody wrote down.

**The architectural lever.** Gateway + observability configuration:

- **Prompt / completion redaction at logging boundary.** Full prompts and completions never land in general observability. Instead the gateway writes a hash and a token-count, or a structured envelope (user id, tenant, model, token-count, latency, status). mod-110 covers the stream shape.
- **Opt-in detailed-trace capture.** For debugging, a time-boxed, per-tenant opt-in captures full prompt + completion for N hours; the opt-in is audited.
- **Eval-harness storage separation.** Eval traces that contain production prompts live under the pilot's trust boundary (mod-107 Chapter 04 — eval infra inside the customer's trust boundary).
- **No hosted third-party LLM observability by default.** LangSmith hosted, Humanloop hosted, Braintrust hosted, Weights & Biases hosted — any of them is a customer-data egress path unless the customer-specific DPA covers it. Default: self-host or VPC-install.

Clause:

> *"Vendor shall not retain Customer Data in its own observability, logging, or evaluation systems beyond [Window] except where expressly opted-in by Customer for debugging purposes. Opt-in retention is time-boxed, logged, and auto-expires. Where retained, Customer Data is minimised to the fields required for the stated purpose."*

Evidence:

- Observability configuration showing hash-only payloads.
- Opt-in trace-capture runbook with audit samples.
- Eval-harness deployment inside the customer's trust boundary (DFD confirmation).

## Lever 5 — Training opt-out

**The commitment.** Customer data is not used — ever — to train or improve any model operated by the vendor or by the model provider.

**The defaults you must know, again.** Most enterprise agreements already set training opt-out as the default, but the FDE must confirm the attestation on file for *this* customer's account, not rely on the general policy page:

- Anthropic enterprise: no training on API inputs or outputs.
- OpenAI enterprise / Team / API: no training on customer data; the consumer product's default differs.
- Azure OpenAI: no training on customer data.
- AWS Bedrock: no training on customer data.
- Google Vertex AI: no training of foundation models on customer data; customer-tuned models are the customer's.
<!-- needs-research: verify current training-opt-out language for each provider's current enterprise agreement before publishing; the specific wording changes with each contract revision -->

**The architectural lever.** This is almost entirely contractual, with one architectural sub-lever: for providers that log for abuse monitoring (Azure OpenAI's 30-day default), the abuse-monitoring waiver must be in effect — otherwise the logs are a retention surface even without training.

Clause:

> *"Model Provider and Vendor shall not use Customer Data (including prompts, completions, embeddings, feedback signals, or other operational data) to train, fine-tune, improve, or otherwise develop any model, service, or feature. This prohibition covers direct training, reinforcement learning from human feedback, and any form of training-data derivation. Vendor shall provide, on request, Model Provider attestation confirming this posture for Customer's account."*

Evidence:

- Model-provider attestation PDF per account.
- The subprocessor list with a "training-opt-out" column populated.

## Lever 6 — Audit-log retention

**The commitment.** Audit logs are retained for a defined period, protected from tampering, accessible for audit and legal-hold, and produced to the customer on request.

**The regime-driven windows to know.**

- **HIPAA (Archetype A).** Policies and procedures retained 6 years from creation or last effective date (45 CFR 164.316(b)(2)). Audit-log retention is set by the covered entity's policy; 6 years is a common HIPAA default; audit-trail of PHI access is Security Rule-required.
- **FedRAMP (Archetype B).** NIST 800-53 AU-11 record retention — often 3 months in log store, long-term archive to agency SIEM. Varies by agency.
<!-- needs-research: confirm NIST 800-53 Rev 5 AU-11 default values and typical FedRAMP implementation before publishing -->
- **GDPR / DORA (Archetype C).** No specific retention floor; "necessary for stated purpose" governs. DORA ICT-related incident records are retained per the regulated entity's incident-management policy; DORA has specific major-incident retention for supervisory reporting.
<!-- needs-research: confirm DORA incident-record retention under RTS for incident reporting before publishing -->

**The architectural lever.** Audit stream to the customer's SIEM, retention at the SIEM, with a tamper-evident posture at the customer boundary.

Audit fields (consistent with mod-105, mod-106, mod-107):

```ndjson
{"ts":"2026-11-02T09:14:32Z","tenant":"payer-a","user":"csr_12","request_id":"req_...","model":"claude-sonnet-4-6","token_in":512,"token_out":128,"prompt_hash":"sha256-...","completion_hash":"sha256-...","tool_calls":[{"tool":"policy_lookup","args_hash":"sha256-...","result_hash":"sha256-..."}],"retrieval":{"index":"policy-v4","top_k":8,"doc_ids":["policy-2201","policy-2207"]},"hitl":{"approver":"sup_03","decision":"approved","latency_ms":12400},"outcome":"success"}
```

Field choices worth naming:

- **Hashes, not raw content.** The audit records prove the action without storing the content; the content lives behind an access-controlled lookup if retention policy allows.
- **Full identity chain.** Authenticated user + authenticated workload + request id. Repudiation is prevented only if the identity is strong (mod-106 Chapter 05 agentic-NHI discipline).
- **Retrieval doc ids.** The retriever's provenance is auditable; a cross-tenant leak is detectable.
- **Tool arguments and results as hashes.** A confused-deputy incident is reconstructible.
- **HITL decision + approver.** The human-oversight record demanded by EU AI Act Article 14 lives here.

Clause:

> *"Vendor shall emit audit events to Customer's SIEM in the schema agreed in Exhibit [X]. Events shall include [fields]. Retention shall be [period], not less than [regime floor]. Access to raw events shall be restricted to Customer; Vendor shall have access only to the subset needed for incident response, and such access shall itself be audited."*

Evidence:

- Audit-stream sample.
- SIEM retention configuration.
- Tamper-evident transport evidence (HEC with mutual TLS, Sentinel with CMK on log analytics workspace, etc.).

## The DPA shape, assembled

Pulling the six levers into a DPA skeleton the FDE can walk a customer's legal team through. Headings roughly match what commercial DPAs carry in 2026:

1. **Definitions.** "Customer Data," "Model Provider," "Zero Data Retention," "Customer-Managed Key," "Sub-Processor," "Pilot."
2. **Scope and roles.** Processor / sub-processor relationships; GDPR Article 28 content; HIPAA 164.504(e) content; CCPA service-provider content; EU AI Act deployer / provider roles (Chapter 03's routing).
3. **Retention.** Lever 1 clause + Lever 4 clause.
4. **Residency.** Lever 2 clause + sub-processor region table.
5. **Encryption and keys.** Lever 3 clause + per-surface key table.
6. **Training opt-out.** Lever 5 clause + per-provider attestation list.
7. **Audit logging and retention.** Lever 6 clause + schema reference.
8. **Sub-processors.** The list (Chapter 02), notice period, objection mechanism, BAA / SCC coverage.
9. **Breach notification.** SLA (HIPAA 60-day outer limit; GDPR 72-hour to supervisory authority; sector-specific); content requirements; escalation.
10. **Audit rights.** The customer's right to audit the vendor; SOC 2 / ISO cert in lieu; pen-test-report sharing.
11. **Return and deletion.** On termination, the vendor returns or deletes customer data; attestation of deletion.
12. **Cross-border transfer.** GDPR SCCs module 2/3; UK IDTA; EU-US DPF where applicable; transfer-impact-assessment (TIA) obligation.
13. **Confidential computing / BYOK / special arrangements.** Optional annexes for the deployments that require them.
14. **Change management.** How changes to this DPA are proposed, reviewed, signed.
<!-- needs-research: cross-check the GDPR Article 28(3) mandatory DPA contents, the HIPAA 164.504(e) BAA requirements, and the latest EU-US Data Privacy Framework status before publishing -->

The FDE does not write the DPA alone. The FDE walks the vendor's legal team and the customer's legal team through each clause's engineering backing; the legal teams draft the contract language.

## Running the three archetypes

**Archetype A — US healthcare payer.** DPA + HIPAA BAA addendum. Levers: Lever 1 Bedrock ZDR attestation; Lever 2 US regions; Lever 3 Customer-managed KMS on S3 and pgvector; Lever 4 Datadog redacted, no hosted LangSmith; Lever 5 Bedrock + Anthropic training-opt-out attestations; Lever 6 Splunk HEC, 6-year retention aligned with HIPAA policy.

**Archetype B — US federal agency.** DPA + FedRAMP Interconnection Security Agreement (ISA) addendum. Levers: Lever 1 provider-side zero-retention (open-weights self-hosted in GovCloud); Lever 2 GovCloud-only; Lever 3 FIPS-validated CMK; Lever 4 opt-in debug capture logged in agency SIEM; Lever 5 no external model-provider so no training-opt-out needed but attested in writing; Lever 6 agency SIEM retention per agency policy.

**Archetype C — EU retail bank.** DPA + Article 28 processor terms + SCCs if any non-EEA flow + DORA ICT-third-party contract terms. Levers: Lever 1 Azure OpenAI with Modified Abuse Monitoring exception in effect; Lever 2 EU Data Boundary end-to-end; Lever 3 Azure Key Vault with HSM-backed CMK; Lever 4 Sentinel redacted, no hosted eval; Lever 5 Azure OpenAI training-opt-out attestation + OpenAI model attestation; Lever 6 Sentinel in EU region with GDPR-aligned retention and DORA incident-records retention.

## Anti-patterns

**Zero-retention clause, non-zero-retention default.** The DPA says ZDR; the account has Azure OpenAI's default 30-day abuse-monitoring storage active because the Modified Abuse Monitoring exception was never applied for. First commitment, no verification. Fix: confirm the attestation before signing the clause.

**Residency as a region label, not a flow.** The infra is in `eu-west-1`, but the embedding API call egresses to a US endpoint because the embedding vendor's default was never changed. Fix: walk the DFD; verify every sub-processor's endpoint.

**CMK on S3, not on vector DB.** CMK is set on the obvious surface but missed on the vector DB or the audit log. Fix: table of all data-at-rest surfaces with CMK status (Lever 3 table above).

**Observability as a silent retention surface.** Zero-retention with the model provider but Datadog captures the full prompt and completion. Fix: Lever 4 redaction at logging boundary.

**Training-opt-out as a slide, not an attestation.** The vendor says "we don't train on customer data" without a document from the model provider confirming it for this account. Fix: attach the attestation; refresh annually.

**Audit-log retention aligned to the vendor's default, not the regime.** A 90-day SIEM default with a 6-year HIPAA requirement is a compliance gap discovered at audit. Fix: Lever 6 retention configured to the regime floor at pilot start.

**Non-zero-downtime key rotation.** The customer rotates the CMK; the pilot fails for 30 minutes because the vendor's code did not gracefully re-wrap. Fix: tested rotation runbook, Lever 3 evidence.

**Sub-processor change without notice.** Adding a new embedding provider mid-pilot without customer notice. Fix: notice workflow driven by the sub-processor-list diff.

## Summary

- The DPA (plus BAA / SCCs / service-provider-addendum / DORA-ICT addenda as needed) is the contract; every clause is also an engineering commitment.
- Six engineering levers: zero-retention with model provider; data residency; BYOK / CMK; prompt / completion storage in the vendor's stack; training opt-out; audit-log retention.
- Each lever has a commitment, a default-you-must-know for the common model providers, an architectural lever, a clause, and evidence.
- The gateway (mod-104 Chapter 03) is the enforcement point for Lever 1 and Lever 4; IaC is the enforcement point for Lever 2 and Lever 3; account-level attestations enforce Lever 5; SIEM configuration enforces Lever 6.
- The assembled DPA has a predictable skeleton — 14 sections — that the FDE walks with legal, naming the engineering backing per clause.
- Three archetypes produce three DPA shapes: HIPAA BAA with ZDR Bedrock and Splunk HEC; FedRAMP ISA with GovCloud and FIPS CMK and agency SIEM; GDPR + DORA + EU AI Act with EU Data Boundary, Azure CMK, Sentinel.
- Eight anti-patterns to catch: ZDR clause without attestation; residency as label; partial CMK; observability retention; training-opt-out as slide; wrong retention window; non-zero-downtime rotation; silent sub-processor change.
- Chapter 06 picks up the last engineering commitment the modern enterprise DPA increasingly asks for: the AI supply-chain provenance — ML-BOM, SLSA, Sigstore, and ModelScan.
