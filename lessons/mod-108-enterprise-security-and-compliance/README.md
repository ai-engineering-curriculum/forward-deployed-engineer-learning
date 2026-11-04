# mod-108-enterprise-security-and-compliance: Enterprise Security, Privacy, and Compliance in Customer Environments

> Scaffolded by `aicg org execute-plan`. Lecture chapters and exercise content are authored on subsequent autonomous cycles.

**Estimated effort:** 16 hours

## Learning objectives

- Pass the customer's security review — SOC 2 Type II evidence collection, SIG / CAIQ / vendor-security questionnaire responses, pen-test report sharing, data-flow diagrams, sub-processor disclosures — at the pace customer onboarding demands
- Meet the vertical-specific regulatory ask — HIPAA + BAA for health, FedRAMP Moderate / High for federal, PCI-DSS for payments, EU AI Act Articles 9-15 + 72 for EU high-risk, GDPR Articles 22 + 32 + 35 for EU any, CCPA for California, state-level LL144 / Colorado AI Act / NYC bias audits when hiring / credit / housing — and know which obligations route to the customer vs. this role
- Threat-model the pilot — OWASP LLM Top 10 (prompt injection, insecure output handling, training-data poisoning, model DoS, supply chain, sensitive information disclosure, insecure plugin design, excessive agency, overreliance, model theft) + OWASP Agentic Top 10 + MITRE ATLAS tactics — and ship mitigations before go-live
- Negotiate the data contract — zero-retention with the model provider, data-residency constraints (EU-only, in-region, on-prem), customer-provided keys / BYOK, prompt + completion storage policy, training-data-opt-out, audit-log retention; author the Data Processing Addendum shape from the engineering side
- Package AI supply-chain provenance — model card + system card + dataset card slice for the customer deployment, CycloneDX ML-BOM + SPDX AI, Sigstore / SLSA attestations for the deployment artifacts, Hugging Face ModelScan / picklescan results for every third-party model consumed
- Cite the boundary to `ai-infra-security` / `security-learning` (peer, level 35) on deep MLSec at platform scale (model-extraction defence at inference edge, judge supply-chain, eval-set exfiltration prevention, MLSecOps) — working depth here, platform-scale depth linked out; cite the boundary to `ai-risk-engineer` (level 25, Governance family) on the risk-engineering slice the customer's governance team may run in parallel
- Cite the boundary to `ai-evaluation-engineer` (peer, level 35, Governance family) on regulator-facing release-assurance packets — this module prepares the engineering half; packet assembly is linked out

## Structure

- `01-…md` … `0N-…md`: lecture chapters.
- `exercises/`: per-exercise prompts.
- `labs/`: long-form hands-on labs.
- `quizzes/`: knowledge checks.
- `resources.md`: external references.
