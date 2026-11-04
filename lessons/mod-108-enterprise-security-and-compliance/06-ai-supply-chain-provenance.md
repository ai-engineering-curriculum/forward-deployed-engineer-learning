# Chapter 06 — Packaging AI Supply-Chain Provenance — Cards, ML-BOM, SLSA, ModelScan

## Why this chapter exists

Chapter 05 closed the data-contract loop — what the vendor promises about *customer* data. This chapter closes the other loop — what the vendor can prove about its *own* software and the third-party AI components it depends on. The question the customer's auditor asks at the six-month follow-up, not at the kickoff review, is: "which model, built by whom, trained on what, served through which pipeline, is running in production today, and how do you prove it hasn't been tampered with?"

Traditional software supply-chain security has coalesced around three artefacts: an SBOM (what's in the binary), a build attestation (how it was built, SLSA), and a signature (who published it, Sigstore). AI systems need the same three plus three more: a model card (what the model is and is not), a dataset card (what data trained it), and a model-scan report (is the weight file itself hostile).

The deep craft of supply-chain-security policy at platform scale — fleet-wide attestation enforcement, continuous re-scanning, the MLSecOps control stack — is `ai-infra-security` / `security-learning` peer territory. The FDE's job is to produce the *per-pilot* slice: the deployed model's card, the pilot release's ML-BOM and SBOM, the attestations on this build, and the scan reports for every third-party weight this pilot loads.

## The six artefacts

**1. Model card.** Describes the model. Introduced by [Mitchell et al., "Model Cards for Model Reporting" (FAT* 2019)](https://arxiv.org/abs/1810.03547); extended by Hugging Face's [model-card template](https://huggingface.co/docs/hub/model-cards), Google's [Model Card Toolkit](https://github.com/tensorflow/model-card-toolkit), and sector-specific adaptations (SR 11-7 validation memo, EU AI Act Annex IV technical documentation).

Minimum sections: model details (provider, version, training cutoff, context window, modalities); intended use and out-of-scope use; training data (summary, provenance, known biases); evaluation (benchmarks, customer-specific gold set — mod-107 Chapter 06); limitations; ethical considerations; license; contact.

**2. System card.** Describes the end-to-end system. Introduced by frontier labs (Anthropic's [Claude 3 model / system cards](https://www-cdn.anthropic.com/de8ba9b01c9ab7cbabf5c33b80b7bbc618857627/Model_Card_Claude_3.pdf), OpenAI's [GPT-4o System Card](https://openai.com/index/gpt-4o-system-card/)). The system card describes behaviour that emerges when the model is embedded in an application — the retrieval, the guardrails, the HITL, the tooling, the evals.

For the FDE's pilot, the system card covers: the production pipeline (gateway, retrieval, agent orchestrator, HITL, audit), the mitigations layered on top (Chapter 04 matrix), the evaluation (Chapter 02 evidence + mod-107 metrics), the residual risks, the known anti-patterns in the deployed form.

**3. Dataset card.** Describes the data. Introduced by [Gebru et al., "Datasheets for Datasets" (ACM 2021)](https://arxiv.org/abs/1803.09010); extended by Hugging Face's [dataset card template](https://huggingface.co/docs/hub/datasets-cards).

For FDE pilots, the dataset card most often covers the *customer's* retrieval corpus (mod-105) and the customer's gold set (mod-107): provenance of each document / row; collection method; known biases; retention; access.

**4. ML-BOM.** Machine-learning bill of materials. The authoritative format is [CycloneDX ML-BOM](https://cyclonedx.org/capabilities/mlbom/) (an extension of CycloneDX 1.5+). Complementary: [SPDX 3.0 AI Profile](https://spdx.dev/learn/areas-of-interest/ai/). The ML-BOM enumerates models, datasets, and AI-related dependencies in a machine-readable form that the customer's SBOM-ingest pipeline can consume.

A CycloneDX ML-BOM fragment (JSON, abbreviated):

```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.6",
  "serialNumber": "urn:uuid:3e671687-...",
  "version": 3,
  "metadata": {
    "timestamp": "2026-11-02T12:00:00Z",
    "component": {
      "type": "application",
      "name": "payer-support-pilot",
      "version": "0.7.1"
    }
  },
  "components": [
    {
      "type": "machine-learning-model",
      "bom-ref": "pkg:huggingface/anthropic/claude-sonnet-4-6",
      "name": "claude-sonnet-4-6",
      "version": "4.6",
      "supplier": {"name": "Anthropic"},
      "licenses": [{"license": {"id": "proprietary"}}],
      "externalReferences": [
        {"type": "documentation", "url": "https://www-cdn.anthropic.com/.../Model_Card_Claude_Sonnet_4_6.pdf"}
      ],
      "modelCard": {
        "modelParameters": {
          "task": "text-generation",
          "architectureFamily": "transformer"
        },
        "quantitativeAnalysis": {
          "performanceMetrics": [
            {"type": "accuracy", "value": "N/A — customer-specific gold set applies"}
          ]
        },
        "considerations": {
          "users": ["csr_tier_1", "csr_tier_2", "supervisor"],
          "useCases": ["member-inquiry resolution drafting"],
          "technicalLimitations": ["Known to hallucinate specific policy numbers absent retrieval"],
          "ethicalConsiderations": [{"name": "PHI exposure", "mitigationStrategy": "zero-retention + VPC"}]
        }
      }
    },
    {
      "type": "machine-learning-model",
      "bom-ref": "pkg:huggingface/BAAI/bge-m3",
      "name": "bge-m3",
      "version": "2024-09-01",
      "supplier": {"name": "BAAI"},
      "licenses": [{"license": {"id": "MIT"}}],
      "modelCard": {
        "modelParameters": {"task": "embedding"}
      }
    },
    {
      "type": "data",
      "bom-ref": "data:customer-pilot/policy-corpus-v4",
      "name": "policy-corpus",
      "version": "v4"
    }
  ],
  "dependencies": [
    {"ref": "pkg:huggingface/anthropic/claude-sonnet-4-6",
     "dependsOn": ["pkg:huggingface/BAAI/bge-m3", "data:customer-pilot/policy-corpus-v4"]}
  ]
}
```

<!-- needs-research: confirm current CycloneDX ML-BOM spec version (1.5 vs 1.6) and the machine-learning-model field names before publishing — the spec continues to evolve -->

**5. SLSA / Sigstore attestations.** Software supply-chain attestations for the pilot's deployment artefacts.

- [SLSA (Supply-chain Levels for Software Artifacts)](https://slsa.dev/) — an incremental framework (L1-L4) for build integrity. L2 is table stakes for enterprise; L3 adds hardened build; L4 requires hermetic reproducible builds.
- [Sigstore](https://www.sigstore.dev/) — keyless signing via OIDC identities; provides a signature on the artefact tied to the signer's identity.
- [in-toto](https://in-toto.io/) attestations are the common representation.

A SLSA Provenance v1 attestation payload (abbreviated):

```json
{
  "_type": "https://in-toto.io/Statement/v1",
  "subject": [
    {
      "name": "payer-support-pilot:0.7.1",
      "digest": {"sha256": "ab12..."}
    }
  ],
  "predicateType": "https://slsa.dev/provenance/v1",
  "predicate": {
    "buildDefinition": {
      "buildType": "https://slsa-framework.github.io/github-actions-buildtypes/workflow/v1",
      "externalParameters": {
        "workflow": {
          "ref": "refs/tags/v0.7.1",
          "repository": "https://github.com/vendor/payer-support-pilot",
          "path": ".github/workflows/release.yml"
        }
      }
    },
    "runDetails": {
      "builder": {"id": "https://github.com/actions/runner-images"},
      "metadata": {"invocationId": "...", "startedOn": "...", "finishedOn": "..."}
    }
  }
}
```

<!-- needs-research: confirm SLSA v1.0 provenance predicate structure and the current in-toto Statement/v1 against https://slsa.dev before publishing -->

**6. Model-scan reports.** Weight files and model repositories are themselves code; some formats (pickle in PyTorch, pre-2024 Hugging Face uploads) execute on load. Static scanning catches hostile weights.

- [ModelScan from Protect AI](https://github.com/protectai/modelscan) — scans serialised model files for deserialisation vulnerabilities.
- [picklescan](https://github.com/mmaitre314/picklescan) — scans pickle files for dangerous opcodes.
- [Hugging Face's built-in scanning](https://huggingface.co/docs/hub/security) — pickle scanning and ClamAV on HF-hosted files.
- Safetensors format — [`safetensors`](https://github.com/huggingface/safetensors) is a code-execution-free weight format and is the default-good choice for every weight the pilot loads. Prefer safetensors by construction.

A ModelScan CI gate looks like:

```yaml
# .github/workflows/model-scan.yml — illustrative
name: Model scan
on:
  pull_request:
    paths:
      - 'models/**'
      - 'requirements.txt'
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install modelscan picklescan
      - name: Pull referenced third-party weights
        run: ./scripts/pull-pinned-weights.sh
      - name: Scan with ModelScan
        run: modelscan -p ./weights -r reports/modelscan.json --fail-on severity:MEDIUM
      - name: Scan with picklescan
        run: picklescan -p ./weights --fail-on dangerous
      - name: Upload reports
        uses: actions/upload-artifact@v4
        with:
          name: scan-reports
          path: reports/
```

A scan report published on every release becomes an artefact in the Chapter 02 evidence packet.

## The provenance pipeline — assembled

The pilot's deploy pipeline produces all six artefacts as part of each release. In flow:

```
developer push
   |
   v
CI pipeline
   |-- unit + eval gates (mod-107)
   |-- security scans (SAST, dependency scan)
   |-- model scans (ModelScan, picklescan) on any third-party weights
   |-- build container image(s)
   |-- CycloneDX SBOM + ML-BOM generation
   |-- Sigstore keyless-sign the image(s) and the ML-BOM
   |-- SLSA provenance attestation (keyless Sigstore, GitHub OIDC identity)
   |-- push to registry
   |
   v
deploy gate (mod-109)
   |-- policy engine verifies Sigstore signature + SLSA Level
   |-- policy engine verifies ML-BOM presence + model-scan pass
   |-- model card + system card + dataset card published and dated
   |
   v
production
```

Policy enforcement at the deploy gate is where this pipeline earns its keep. [Sigstore Policy Controller](https://docs.sigstore.dev/policy-controller/overview) or Kyverno can be configured to reject any image missing a valid SLSA-level provenance attestation signed by the vendor's build identity.

## Cards at the pilot layer

**Model card.** The customer-deployed slice of the provider's model card, extended with pilot context:

```markdown
# Model card — payer-support-pilot, Claude Sonnet 4.6, deployment v0.7.1

## Model details
- Provider: Anthropic via AWS Bedrock (us-east-1)
- Base model card: https://docs.anthropic.com/.../Model_Card_Claude_Sonnet_4_6.pdf
- Access: Bedrock InvokeModel, zero-retention attested (see DPA §3.2)

## Intended use
- Draft replies to member inquiries (mod-102 workflow definition)
- CSR tier 1 / tier 2 / supervisor review surface

## Out-of-scope use
- Prior-auth determination (HITL approval required on every write)
- Clinical advice (prohibited by system-prompt + output-sanitiser)
- Billing adjustments outside documented tier-2 scope

## Training data
- Base: Anthropic training data per base model card
- Pilot-side fine-tune: none
- Retrieval corpus: customer policy corpus v4 (dataset card attached)

## Evaluation
- Customer gold set v1.3; 120 rows; portfolio from mod-107 Chapter 03
- Pass bar: correctness >= 0.85; faithfulness >= 0.90; refusal recall >= 0.90
- Online eval: sample-rate 5%, Krippendorff alpha 0.78 vs SMEs

## Limitations
- Hallucinates benefit numbers absent retrieval citations
- Refusal recall dips on compound queries (known; mitigated by HITL on write)

## Ethical considerations
- PHI never leaves customer VPC (DFD §2)
- Member demographic parity monitored (mod-107 online eval)

## License and contact
- Anthropic commercial terms; pilot engagement SOW 2026-07-01
- Contact: fde@vendor.com; customer: dpo@payer-a.com
```

**System card.** The end-to-end system; includes the surfaces not in the base model card.

```markdown
# System card — payer-support-pilot, v0.7.1

## System composition
- LLM: Claude Sonnet 4.6 via Bedrock
- Retrieval: pgvector (customer VPC) + BM25 fusion
- Guardrails: gateway-level DLP + output sanitiser
- Agent orchestrator: LangGraph; tool catalogue scoped to read + single write with HITL
- HITL: CSR UI with anomaly-highlight; supervisor approval on write amounts > $X
- Audit: Splunk HEC with mutual TLS; CMK-protected

## Threat model (Chapter 04)
- OWASP LLM Top 10 matrix: signed 2026-10-28 (attached)
- OWASP Agentic Top 10 matrix: signed 2026-10-28 (attached)
- MITRE ATLAS tactics: attack narratives for AML.T0051, T0070, T0024 attached

## Evaluations (mod-107)
- Offline portfolio v3
- Online eval sample
- Exit-eval decision document v1 signed 2026-10-30

## Human oversight (mod-106 Chapter 04, EU AI Act Article 14 posture)
- Tier 1: auto-send after HITL click on draft
- Tier 2: HITL with supervisor approval on write > $X
- Kill-switch: feature flag `payer-support-enabled`; disable in <60s

## Residual risks
- Volume spike -> HITL overwhelm (mitigation: anomaly highlight + rate-limit; monitored)
- Benefit-term drift (mitigation: weekly online-eval)
- Novel injection via future member channels (threat model revisit on channel addition)

## Known anti-patterns in deployed form
- (none identified at v0.7.1)
```

**Dataset card.** The policy corpus and the gold set:

```markdown
# Dataset card — policy-corpus v4

## Overview
- Provenance: customer's internal policy management system, exported 2026-10-01
- Size: 32,418 documents, 4.1 GB
- Licence: customer confidential; vendor-handled under DPA §2

## Collection
- Automated export via customer API; identity: service-account scoped to policy-read
- Transformation: PDF -> markdown via pypdf + customer-reviewed cleanup

## Intended use
- Retrieval corpus for payer-support pilot
- Not used for training or fine-tune

## Known biases / limitations
- State-specific policies: 42 states covered; 8 under-represented
- 2019-2020 policies partially redacted on export

## Access and retention
- pgvector in customer VPC; CMK-protected
- Retention: lifetime-of-pilot; right-to-erasure per-doc supported

## Maintenance
- Weekly refresh; deltas captured in ingestion audit log
```

Three cards. One ML-BOM. One SLSA attestation. One Sigstore signature. One ModelScan report (zero third-party weights outside the ML-BOM in this pilot). Attached to the release.

## Running the three archetypes

**Archetype A — US healthcare payer.** Six artefacts: Claude Sonnet model card with deployment slice; payer-support-pilot system card; policy-corpus + gold-set dataset cards; CycloneDX ML-BOM; Sigstore-signed container image + SLSA L2 attestation; ModelScan pass on bge-m3 embedding weights. All versioned; all in the Chapter 02 evidence packet.

**Archetype B — US federal agency.** The ML-BOM and SLSA posture are required in the ATO body of evidence. Open-weights models (self-hosted Llama 3 / Qwen 3 variants under agency approval) require ModelScan + picklescan + safetensors-only policy enforced at the deploy gate. Weight provenance must include the hash of the file transferred into GovCloud and the evidence of the transfer being authorised. Dataset card for the policy corpus must cover classification labels (FOUO, CUI).
<!-- needs-research: confirm whether specific agency ATO bodies of evidence formally require ML-BOM / SLSA; adoption is growing but still agency-dependent -->

**Archetype C — EU retail bank.** EU AI Act Article 11 + Annex IV technical documentation is composed from the model card + system card + dataset card + ML-BOM + evaluation evidence. SR 11-7 validation memo is built on the same artefacts. Sigstore attestations are required under the bank's own supply-chain policy as part of the DORA ICT-third-party register entry.
<!-- needs-research: confirm Annex IV technical-documentation section headings and whether any map directly to ML-BOM / SLSA before publishing -->

## Anti-patterns

**Model card as marketing material.** A model card that reads as feature list without naming out-of-scope use, limitations, and residual risk is a selling document, not a technical artefact. Regulators discount these on sight. Follow the Mitchell et al. structure.

**ML-BOM in a drawer.** Producing the ML-BOM on release but never surfacing it to the deploy gate — no enforcement, so no guarantee. The ML-BOM has to be *checked* by the deploy policy; otherwise it is ceremony.

**SLSA L1 and calling it L2.** The level is specific: L2 requires the build to produce signed provenance from a hosted builder. Claiming L2 without the signed provenance is a finding.

**Sigstore signature without a verifying policy.** Signing the image is only half. The Kubernetes / deploy-gate side must verify the signature matches the expected identity.

**Weight loaded without a scan.** An FDE pulls a Hugging Face model for a quick experiment, loads it, moves on. If that experiment ever becomes a production step, the unverified weight is a supply-chain surface. Rule: every weight the pilot loads, even in a notebook, has a ModelScan + picklescan + prefer-safetensors check.

**Dataset card omitted because "it's the customer's data."** The customer's data in the pilot is still processed by the vendor's pipeline; the card is what the customer's privacy office audits. Produce it.

**Provenance retro-fitted at audit.** A customer's auditor at month six asks for the SLSA attestation on v0.3.1; the pipeline did not produce one at v0.3.1. The gap is now a finding. Rule: provenance from v0.1.0.

## Boundaries revisited

- **`ai-infra-security` / `security-learning` (peer, level 35, Infra family)** owns platform-level provenance policy — the SLSA-level required across the fleet, the deploy-gate policy engine configuration, the continuous re-scanning infrastructure. The FDE's per-pilot artefacts are inputs; the platform peer sets the bar.
- **`ai-evaluation-engineer` (peer, level 35, Governance family)** owns the composition of these artefacts into regulator-facing release-assurance packets. The FDE produces the artefacts; the peer wraps them into the Annex IV / SR 11-7 / ATO-annex documents.
- **`ai-risk-engineer` (peer, level 25, Governance family)** owns the risk-register entries and the control-mapping to NIST AI RMF / ISO/IEC 42001. The FDE's model card + system card + residual-risk statement become inputs to the register.

Chapter 07 writes the pager protocol across all three.

## Summary

- The modern enterprise AI pilot ships six supply-chain artefacts: model card, system card, dataset card, ML-BOM, SLSA + Sigstore attestations, ModelScan / picklescan reports.
- CycloneDX ML-BOM is the authoritative machine-readable format; SPDX 3.0 AI Profile is the complement; the ML-BOM is composed in the CI pipeline and verified at the deploy gate.
- SLSA L2 is table stakes for enterprise; Sigstore keyless signing is the signature layer; in-toto attestations are the on-wire format.
- Third-party weights are scanned with ModelScan + picklescan; safetensors is the preferred format by construction.
- Cards follow the standard shape (Mitchell et al. for model cards; Gebru et al. for dataset cards; frontier labs' pattern for system cards) with pilot-specific extensions.
- The deploy gate verifies the chain — signature, SLSA level, ML-BOM present, scan pass — otherwise the chain is ceremony.
- Each archetype uses the same artefacts to satisfy different downstream obligations: HIPAA evidence, FedRAMP ATO body of evidence, EU AI Act Annex IV and DORA ICT-third-party register.
- Seven anti-patterns to catch: marketing-material cards; ML-BOM-in-drawer; mis-claimed SLSA level; unverified signatures; unscanned weights; omitted dataset card; retro-fitted provenance.
- The platform peers (`ai-infra-security`, `ai-evaluation-engineer`, `ai-risk-engineer`) consume these artefacts; Chapter 07 writes the boundaries.
