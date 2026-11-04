# Chapter 02 — Bootstrapping the Customer Gold Set — Stratified Sampling, SME Pairing, and Inter-Rater Reliability

## Why this chapter exists

The gold set is the artifact the customer's governance board sees. It is the pilot's measurement substrate. Every claim the FDE makes about quality — "retrieval nDCG is 0.78," "refusal recall is 0.94," "the pairwise human-preference rate is 62%" — resolves back to the labels in this one file. Without the gold set there is no signal. Without signal there is no drift detection, no regression gate in CI, no exit eval the governance board can sign. The pilot's measurement story is only as reliable as the gold set underneath it.

Mod-105 Chapter 06 covered the *retrieval-side* gold set — queries plus graded source-of-truth chunks, used to score precision@k, MRR, and nDCG against the customer's corpus. That chapter is the floor of this one. This chapter extends to the **task-level** gold set: the whole pilot output, not just the retrieved chunks. The row now carries the expected final answer, the expected refusal flag, the pairwise anchor if one exists, and the business-KPI linkage that lets a score roll up to a value number. The retrieval row is a strict subset of the task row.

The FDE owns the bootstrap of both. The retrieval-side shape and the SME-pairing mechanics from mod-105 Ch 06 carry over unchanged; do not restate them. This chapter adds the parts a task-level gold set demands that a retrieval gold set does not: stratified sampling against the real case distribution, double-labelling with inter-rater reliability, a labelling UI that survives an audit, and signed versioned storage in the customer's object store.

### Boundary with the eval-engineer peers

Two peer roles sit next to the FDE on this work. The methodology depth — judge-as-LLM calibration, pairwise human-preference design, slice-aware aggregation — belongs to `ai-eval-engineer` (peer, level 30). The release-assurance packaging — the formal evidence packet the regulator reads, the attestation chain, the model-release board process — belongs to `ai-evaluation-engineer` (peer, level 35). The FDE's job here is the *bootstrap*: get 50–200 task-level rows into a shape both peers can consume, labelled by the customer's own SMEs, stored in the customer's own boundary. Hand the metric-design and release-packaging questions back to the peers. The reverse handoff is just as important — a peer who designs a brilliant judge-LLM prompt against a gold set that was never signed by the SMEs who labelled it has no audit trail to show the governance board.

## What a task-level gold-set row looks like

Minimum shape per row. The retrieval fields from mod-105 Ch 06 remain; the task-level additions are flagged below.

```yaml
case_id: C-0117
query_id: Q-0042               # links to the mod-105 Ch 06 retrieval row
input:
  text: "Member on Gold plan, outpatient MRI next Tuesday, pre-auth status?"
  channel: csr_desktop
user_envelope:
  role: csr_tier_1
  groups: ["g:us-support"]
  clearance: internal
  tenant: payer-east
reference_output: |              # task-level addition
  Pre-auth required for outpatient MRI on Gold plans (policy §4.1). Submit
  form PA-212 at least 5 business days before service. Gold plan carries
  the §4.2 orthopaedic exception — if referring diagnosis is M54.5 or
  related, pre-auth is waived.
expected_refusal: false
sme_rationale: |                 # task-level addition
  §4.1 names MRI pre-auth; §4.2 names the Gold-plan exception. CSR misses
  the exception ~1 in 6 times without prompting.
pairwise_anchors:                # optional; task-level addition
  anchor_a: "Pre-auth required; submit PA-212."           # minimal correct
  anchor_b: "Pre-auth required; submit PA-212; note §4.2 exception."
  preferred: anchor_b
provenance:
  support_ticket_id: TT-55123
  workflow_mining_session: WM-07
  sme_interview: SMI-02
business_kpi_linkage:
  workflow_step: "PA guidance step 3 of 6"
  value_if_correct: "saves CSR 45s lookup + prevents 1 escalation in 6"
  cost_if_wrong: "escalation to supervisor; adds ~5 min to call"
labeler_id: "SME-02"
labeled_at: "2026-11-04T14:22:00Z"
row_schema: v3
```

Notes on the additions:

- **`reference_output`** is the SME's answer, not the model's. Written in the SME's own voice during the labelling session. This is the task-level truth the correctness metric (Ch 03) scores against.
- **`expected_refusal`** is already in mod-105 Ch 06; it carries forward because refusal is the task-level failure mode AppSec cares about.
- **`sme_rationale`** names *why* this is the right answer. Auditable; also the raw material for the judge-as-LLM anchor prompts in Ch 04.
- **`pairwise_anchors`** is optional. When the pilot's output quality is graded, a pair of worked anchors — one minimally acceptable, one preferred — lets pairwise eval (Ch 03) run without re-asking the SME for every new model variant.
- **`business_kpi_linkage`** keeps the row tied to the KPI the pilot committed to (mod-102 Ch 03). Without it the eval report cannot translate score deltas into value deltas.
- **`labeler_id` + `labeled_at`** are mandatory for the audit log. The SME who labelled the row is as much part of the provenance as the row itself.
- **`row_schema: v3`** lets the storage layer migrate rows forward safely; the gold set lives for years and the schema will drift.

The row is JSON/YAML in the sanctioned sandbox. Treat the file as a product artifact: reviewed, versioned, signed.

## Stratified sampling across the customer's case distribution

Naïve random sampling under-covers the rare-but-costly slices. A gold set of 100 queries drawn uniformly from last quarter's ticket log will have ~3 escalation-triage queries in Archetype A, because escalation-triage is 3% of volume. Three rows cannot measure a slice. Three rows cannot detect a regression. The pilot's exit eval will land on "we don't know" for the slice whose mistakes cost the most.

Stratified sampling is the fix: partition the population into strata (intent × role × doc-freshness × tenant × channel × refusal), then sample from each stratum. Two allocation rules:

- **Proportional allocation.** Sample each stratum in proportion to its population share. If escalation_triage is 3% of volume, sample 3% of rows from it. Simplest; preserves the volume mix; still under-covers rare-but-costly slices.
- **Neyman allocation.** Sample proportional to stratum size × within-stratum variability of the measured outcome. For a fixed total budget it minimises the overall variance of the mean. In practice it means: sample more heavily from strata where the pilot's error rate (or value variance) is higher. Escalation_triage gets *over*-sampled, not because it is frequent, but because mistakes there are expensive and the model's quality there is noisier.

Cochran, *Sampling Techniques* (3rd ed., Wiley) is the conventional reference for both rules.

### Worked example — Archetype A

Last quarter's volume mix across 18,400 CSR cases:

| Intent            | Share | Error-cost-weighted priority |
|-------------------|-------|------------------------------|
| policy_lookup     | 70%   | Low  (single-source lookup)  |
| member_history    | 20%   | Medium (PII + staleness)     |
| plan_comparison   | 7%    | Medium (multi-doc synthesis) |
| escalation_triage | 3%    | High (supervisor callback)   |

Proportional allocation for a 120-row gold set: 84 / 24 / 8 / 4. Neyman allocation, after the SME pins "escalation mistakes cost ~15 min + a supervisor callout, versus ~45s for a policy_lookup mistake": 60 / 30 / 15 / 15. The Neyman cut gives escalation_triage five times the cell count and makes a per-slice metric actually move.

### Stratification matrix

Publish the matrix alongside the gold set. The cells carry counts, not just labels. Gaps are visible; the next labelling session targets them. Minimum cell count rule-of-thumb: **10 rows per cell**, or an explicit written note explaining why fewer is acceptable (e.g., "FOUO × escalation × stale-doc has three cases in the quarter total; six rows is the realistic ceiling").

| Intent             | Role          | Doc freshness | Tenant      | Channel    | Refusal expected | Cell count |
|--------------------|---------------|---------------|-------------|------------|------------------|-----------:|
| policy_lookup      | csr_tier_1    | fresh (<30d)  | payer-east  | desktop    | no               | 12 |
| policy_lookup      | csr_tier_1    | stable (>1y)  | payer-east  | desktop    | no               | 18 |
| policy_lookup      | csr_supervisor| stable        | payer-west  | desktop    | no               | 10 |
| member_history     | csr_tier_1    | fresh         | payer-east  | desktop    | no               | 12 |
| plan_comparison    | csr_tier_1    | fresh         | payer-east  | desktop    | no               | 10 |
| escalation_triage  | csr_supervisor| fresh         | payer-east  | desktop    | no               | 10 |
| any                | csr_tier_1    | n/a           | payer-east  | desktop    | yes              | 12 |
| any                | csr_tier_1    | n/a           | payer-west  | chat       | yes              | 10 |

Cells below 10 require a written justification in the gold-set manifest. Cells above 20 for a single intent are a smell — the gold set is over-indexed on an easy slice.

## The SME-paired labelling workflow

Mod-105 Ch 06 covered the mechanics. The core carries over: FDE paired with the actual *doer* (CSR, analyst, branch support), not the executive sponsor; 90-minute sessions, no more, because labelling fatigue is real; two to four sessions to reach 50–100 rows. The additions for task-level labelling:

**Before the session.** Candidates come from three sources (identical to mod-105 Ch 06): workflow mining, historical tickets, SME-proposed hard cases. For the task-level set, also carve out a **pairwise-anchor slice** — 20–30% of candidates where the SME will be asked for two anchor outputs (minimally acceptable, preferred) rather than one reference output. These seed Ch 03's pairwise metric.

**During the session.** Per row: SME reads the input; SME states the expected output in their own words; FDE transcribes verbatim and reads back; SME confirms or edits; SME names rationale; FDE records provenance link. Pace: 15–25 rows in the first session, 25–40 after the SME finds the rhythm. Pairwise-anchor rows are slower (two outputs plus a preference) — budget 10–15 per session for those.

**After the session.** Reconciliation. FDE reviews transcripts for consistency, clarifies ambiguous rows with the SME asynchronously, and pushes the labelled batch to the double-labelling queue.

The anti-pattern — AI-pre-labelled rows the SME confirms — is treated in the "Anti-patterns" section below. It matters enough to call twice.

### Constructing pairwise anchors

Pairwise anchors are the raw material for Ch 03's pairwise-preference metric: given a model output and a baseline, which does the SME prefer. The gold-set row carries two anchor outputs so the metric does not have to re-interview the SME every time the model changes.

The construction heuristic:

- **Anchor A — minimally acceptable.** The shortest output that would not be a correctness failure. Stripped of nice-to-haves. For Archetype A's MRI pre-auth question: "Pre-auth required; submit form PA-212." Correct, incomplete, not harmful.
- **Anchor B — preferred.** The output the SME would write for a colleague. Includes the material details a doer actually uses. For the same question: Anchor A plus the §4.2 exception note and the five-business-day window.
- **Preference label.** The SME names the preferred anchor and a one-line reason. Pairwise-eval judge prompts cite the reason as an anchor in Ch 04.

Anchors are not two attempts at the right answer; they are two points on the quality gradient. If the SME writes two outputs that are both "fully correct and complete," there is no gradient to measure and the row is not carrying pairwise signal. Push the SME to make Anchor A genuinely minimal.

## Double-labelling and inter-rater reliability

A gold set labelled by one SME has no signal about label quality. The second SME exists to answer "would a different qualified doer label this the same way?" If the answer is "often no," the gold set is not measuring the task; it is measuring one person's opinion.

Pick an **anchor slice** of ~20% of the gold set. Both SMEs label it blind — SME B must not see SME A's labels or rationales. Compute agreement across the slice.

### Which metric for which label

| Label shape                              | Metric                   | Convention / reference |
|------------------------------------------|--------------------------|------------------------|
| Binary (e.g., `expected_refusal`)        | Cohen's kappa            | Cohen (1960)           |
| Nominal with 3+ raters                   | Fleiss' kappa            | Fleiss (1971)          |
| Ordinal, graded, or with missing data    | Krippendorff's alpha     | Krippendorff (2004)    |
| Free-text (e.g., `reference_output`)     | Human-reviewed semantic  | See note below         |

Free-text reference outputs do not have a clean numeric agreement score. The practical move is to run **semantic agreement** — a third SME reads both outputs and marks "same intent / same facts / resolvable differences / incompatible." Record the fraction in each bucket; treat >15% incompatible as a signal to re-run the labelling session.

Landis & Koch (1977) proposed interpretive bands for kappa and alpha:

| Value range | Band          |
|-------------|---------------|
| < 0.0       | Poor (worse than chance) |
| 0.0 – 0.20  | Slight        |
| 0.21 – 0.40 | Fair          |
| 0.41 – 0.60 | Moderate      |
| 0.61 – 0.80 | Substantial   |
| 0.81 – 1.00 | Almost perfect / strong |

The bands are conventional, not canonical. Krippendorff recommends α ≥ 0.80 for conclusions and α ≥ 0.667 as a cautious floor; many labelling projects accept α ≥ 0.7 as a working threshold. Pick the target with the SME pool and the governance board; write it into the gold-set manifest.

### Computing the scores

```python
from sklearn.metrics import cohen_kappa_score

# Binary refusal flag, two SMEs over the anchor slice
sme_a_refusal = [0, 0, 1, 0, 1, 1, 0, 1, 0, 0]
sme_b_refusal = [0, 1, 1, 0, 1, 1, 0, 1, 0, 0]
kappa = cohen_kappa_score(sme_a_refusal, sme_b_refusal)
# kappa ≈ 0.78  (substantial)

import krippendorff

# Graded relevance 0–3 (ordinal), two raters, with missing values as None
ratings = [
    [3, 2, 1, 0,    3, 2, None, 1],   # SME A
    [3, 2, 2, 0,    3, 1, 1,    1],   # SME B
]
alpha = krippendorff.alpha(
    reliability_data=ratings,
    level_of_measurement="ordinal",
)
# alpha ≈ 0.82  (strong)
```

Both packages are permissive-licensed and VPC-installable. The `statsmodels` library offers Fleiss' kappa via `statsmodels.stats.inter_rater.fleiss_kappa` for the 3+ raters case.

Report the per-label-type score in the gold-set manifest. Low agreement on a specific label type (e.g., `expected_refusal`) is a real finding, not a failure to be hidden — it tells the pilot team that the refusal policy itself is ambiguous and needs tightening before the eval goes live.

### What to do when IRR comes in low

The default move on α = 0.52 is not "label harder." It is to diagnose. Three common causes:

| Cause                                         | Symptom                                                                                | Fix                                                                           |
|-----------------------------------------------|----------------------------------------------------------------------------------------|-------------------------------------------------------------------------------|
| Ambiguous guideline                           | Disagreement concentrated on one label type (e.g., refusal)                            | Re-write the labelling guideline with worked examples; re-label the anchor slice |
| Different SME seniority or tenure             | Disagreement scales with one rater being a tier-1 and the other a supervisor           | Decide whose judgement the pilot optimises for; label with that role          |
| Genuinely ambiguous rows                      | Disagreement spread evenly; both SMEs can defend their labels                          | Flag the rows with `uncertain: true` (mod-105 Ch 06); exclude from the strict-correctness metric; keep for the hedging-behaviour metric in Ch 03 |

Do not throw out the low-agreement rows silently. The rows are evidence — either about the guideline, about the SME pool, or about the task's inherent difficulty. Each path feeds back into the gold-set manifest's change log.

## Labelling-UI shape

A labelling UI that cannot support double-blind labelling, free-text rationale, provenance link-back, and a diff-friendly export is not an audit-grade tool. A Google Sheet does not meet the bar. Two realistic choices and one commercial alternative worth naming:

| Tool          | Maintainer     | Strengths                                    | Trade-offs                                     |
|---------------|----------------|----------------------------------------------|-----------------------------------------------|
| Argilla       | Hugging Face   | Strong text-classification + feedback flows, Hugging Face Hub integration, Python-native | Newer than Label Studio; smaller plugin surface |
| Label Studio  | HumanSignal    | Broad modality support (text, image, audio, time-series), mature templating | Config files can be heavy for text-only projects |
| Prodigy       | Explosion      | Streaming active-learning workflow, tight spaCy integration | Commercial licence; not typically VPC-first out of the box |

Checklist the UI must satisfy, or the gold set is not audit-ready:

- Single-rater mode and **double-blind mode** — rater B cannot see rater A's labels, rationales, or even the fact that the row has already been labelled.
- Free-text **rationale capture** per row, searchable and exportable.
- **Provenance link-back** — each row carries its source ticket ID / mining session / interview ID without the labeller having to retype it.
- **Diff-friendly export** — JSONL or Parquet, not XLSX. The gold set has to survive git diffs for the versioning story to work.
- **VPC-deployable** — the UI runs inside the customer's network. Argilla and Label Studio both self-host; neither requires outbound calls.
- **SSO integration** — SAML or OIDC against the customer's IdP. No separate password store.
- **Immutable audit log** — who labelled, when, what they saw, what they wrote. Not optional for regulated customers.

A Google Sheet fails the double-blind, provenance, and audit-log tests. Per-row cell-level history is not an audit log; it is a convenience feature. Disqualify it early; do not relitigate in week six.

## Versioned and auditable storage

Treat the gold set the way the customer's core engineering treats release artifacts. The specifics:

- **SemVer the gold set.** `v1.0.0` for the first signed release, `v1.1.0` when rows are added, `v2.0.0` when the row schema changes incompatibly. The eval report cites the exact version.
- **Row-schema versioning.** The `row_schema: v3` field on every row lets consumers migrate forward safely. Ch 04's harness pins a minimum row-schema.
- **Immutable object storage** in the customer's environment: AWS S3 with **Object Lock in Compliance mode**, Azure **Immutable Blob** with time-based retention, GCS **Bucket Lock**. Compliance mode is the stronger of the AWS modes — not even the root account can shorten the retention window.
- **Hash manifest.** SHA-256 over the full JSONL / Parquet artifact. The manifest is small, text, and signed.
- **Signature by the labellers.** Each release signed by the SMEs who labelled, using gpg or the cloud KMS key the customer already runs. "SME-02 signed v1.1.0" is the audit trail.
- **Records-of-processing register.** GDPR Article 30 requires the controller to maintain a register of processing activities. For Archetype C, the gold-set's provenance IDs (source ticket, workflow mining session) feed this register. Mod-108 owns the full records-of-processing discipline.
- **Hold-out slice sealed.** The 20% hold-out carved here is the exit-eval slice (Ch 06). It must be marked in the manifest, its hashes recorded separately, and its rows excluded from the training / tuning artifacts the pilot team sees.

A short manifest YAML:

```yaml
gold_set:
  name: payer-east-csr-task-goldset
  version: v1.1.0
  row_schema_min: v3
  rows_total: 122
  rows_training: 97
  rows_holdout_sealed: 25
  stratification_matrix: strata-v1.1.0.yaml
  irr:
    cohen_kappa_refusal: 0.81
    krippendorff_alpha_grade: 0.76
    anchor_slice_frac: 0.21
  provenance_registry: rop-register-2026Q4.csv
  artifact:
    path: s3://payer-east-pilot-artifacts/goldset/v1.1.0/goldset.parquet
    sha256: 9f2c1e8a3b47d9f1e0b2c5a6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6
    object_lock_mode: COMPLIANCE
    retention_until: "2033-11-04"
  signatures:
    - signer: "SME-02"
      kms_key_arn: "arn:aws:kms:us-east-1:111122223333:key/<redacted>"
      signed_at: "2026-11-04T18:02:00Z"
    - signer: "SME-07"
      kms_key_arn: "arn:aws:kms:us-east-1:111122223333:key/<redacted>"
      signed_at: "2026-11-04T18:11:00Z"
  prior_version: v1.0.0
  change_log: |
    v1.1.0 adds 22 rows covering escalation_triage × fresh-doc per Neyman
    re-allocation; refusal cell grew from 8 to 12; hold-out re-sealed.
```

The manifest is small, text, and cheap to re-generate. The point is not the YAML; the point is that an auditor can walk from "the pilot reports 0.78 nDCG" back to "the 122 rows of v1.1.0, signed by SME-02 and SME-07, with Cohen's kappa 0.81 on refusal and alpha 0.76 on graded relevance, stored in Compliance-mode Object Lock until 2033."

## Scheduling the labelling arc end-to-end

The bootstrap is not one meeting. It is a two-to-three-week arc with specific deliverables per week. For a 120-row Archetype A target:

| Week | FDE activity                                                     | SME activity                                     | Deliverable                                         |
|------|------------------------------------------------------------------|--------------------------------------------------|-----------------------------------------------------|
| 1    | Build candidate pool from workflow-mining + ticket sample; draft stratification matrix with proportional and Neyman allocations; agree target cell counts with the pilot sponsor | Review draft stratification matrix; suggest hard cases | Candidate pool (200+ rows); stratification matrix v0.1 |
| 1–2  | Session 1 with SME A (90 min, 20 rows); Session 2 with SME A (90 min, 30 rows) | Label with FDE in-session; narrate rationale aloud | ~50 single-labelled rows; row_schema confirmed |
| 2    | Session 3 with SME B on the anchor slice (90 min, ~25 rows blind) | Label the anchor slice blind to SME A's labels   | ~25 double-labelled rows; raw IRR computable        |
| 2    | Compute Cohen's kappa + Krippendorff's alpha; diagnose any low agreement | Review low-agreement rows with FDE (async)      | IRR report v0.1; guideline edits if needed          |
| 2–3  | Session 4 with SME A + SME B on hard / pairwise-anchor cases (90 min) | Co-label pairwise anchors                      | ~15 pairwise-anchor rows; ~70 single-labelled rows total |
| 3    | Reconciliation; seal 20% hold-out; compute sha256; sign the artifact; wire manifest into the records-of-processing register | SMEs co-sign the release | Signed `v1.0.0` artifact + manifest in customer object store |

Two sources of slippage show up repeatedly. The first is SME availability — the actual doer is also doing the day job, and a 90-minute block is not free. Book the full arc at project kickoff, not week two. The second is guideline drift — the labelling guideline is often half-written when Session 1 starts, gets tightened between Session 1 and Session 2, and then Session 1's rows no longer match. Budget a re-labelling pass on Session 1's rows if the guideline moves materially.

## Three-archetype application

**Archetype A — US healthcare payer.** Stratify intent × CSR tier × doc freshness × refusal. Target ~120 labelled rows across two tier-1 CSR SMEs plus one supervisor; 15% of rows carry pairwise anchors. IRR target: Krippendorff α ≥ 0.7 on the double-labelled 20% anchor slice; Cohen's kappa ≥ 0.75 on the binary refusal flag. Refusal coverage ≥ 10% of the set. Storage: S3 Object Lock in Compliance mode, SSE-KMS with the pilot's own key; signatures via AWS KMS. Review cadence quarterly.

**Archetype B — US federal agency.** Stratify by classification (FOUO / CUI) × intent × document freshness. Target ~80 rows; cleared-personnel SME pool is small, so sessions compress (fewer SMEs, more sessions each). Refusal coverage higher — ~20% of the set — because out-of-scope queries are common and the cost of answering out-of-scope is a records-officer finding. Storage: GovCloud S3 Object Lock, FIPS-140 endpoints, KMS-backed signatures. The records-of-processing analogue is the FedRAMP system security plan artifact; the gold set's provenance maps into the pilot's SSP control family CA-7.

**Archetype C — EU retail bank.** Stratify by branch region × product × regulatory relevance × refusal. Target ~100 rows; one branch SME plus one compliance SME per session. Compliance SME also acts as second rater on the regulatory-relevance anchor slice. Storage: Azure Immutable Blob with time-based retention, in the EU Data Boundary. EU AI Act Article 14 traceability feeds in: the gold set is one of the "technical documentation" artifacts the deployer maintains, and its provenance register ties the pilot's outputs back to the SME who labelled the equivalent case. Mod-108 picks up the Article 14 obligations in detail.

## Anti-patterns

**AI-pre-labelled and SME-confirmed.** The LLM proposes an answer; the SME ticks "looks fine." The SME anchors on the LLM's answer; the labels regress toward whatever the pilot already returns. The gold set becomes an echo of the model, not a measurement of the task. Label from scratch, with the SME speaking first.

**Single-rater everywhere.** One SME labels the whole set; no double-labelling; no IRR. The gold set has no quality estimate of its own. The first time a governance-board member asks "how do you know two qualified reviewers would agree on these labels?" there is no answer. Always carve out an anchor slice.

**"One Google Sheet."** No double-blind mode, no provenance link-back, no audit log, no versioning worth the name. The sheet disqualifies the gold set from the exit-eval evidence packet. The fix is cheap (Argilla or Label Studio, self-hosted); the delay once the governance board flags it is not.

**Labelling in the vendor's SaaS when customer data cannot leave the boundary.** The UI lives at `labelling.vendor.com`; the SME pastes customer PII into it. A data-exfiltration path dressed up as productivity. Mod-108 will formalise this; here the rule is simple: the labelling UI runs where the data can run, not somewhere convenient.

**Ignoring refusal cases.** The gold set covers happy-path queries. The pilot's dangerous failure mode — a confident grounded-looking answer on a query the corpus does not cover — stays invisible until production. Refusal coverage ≥ 10% (higher for Archetype B) is not a nicety; it is the measurement of the pilot's safety floor.

**Freezing the gold set and never refreshing.** The corpus evolves; workflows change; new products appear. A gold set last touched eight months ago is measuring a pilot that no longer exists. Quarterly review with the SMEs; report churn (fraction of rows retired or edited); refresh the stratification matrix; re-seal the hold-out.

## Summary

- The task-level gold set extends mod-105 Ch 06's retrieval row with `reference_output`, `expected_refusal`, optional pairwise anchors, business-KPI linkage, and labeller provenance. The retrieval row is a subset.
- Stratify across the customer's real case distribution. Proportional allocation preserves the volume mix; Neyman allocation over-samples rare-but-costly slices. Publish the stratification matrix with cell counts; apply the 10-per-cell rule-of-thumb or justify.
- SME-paired labelling with the actual doer, 90-minute sessions, 15–40 rows per session. Mod-105 Ch 06's mechanics carry over; this chapter adds pairwise-anchor rows.
- Double-label a ~20% anchor slice blind. Compute Cohen's kappa (binary), Fleiss' kappa (3+ raters nominal), Krippendorff's alpha (ordinal / graded / missing). Target α ≥ 0.7 as a working floor; write the target into the manifest.
- The labelling UI must support double-blind mode, free-text rationale, provenance link-back, diff-friendly export, VPC deployment, SSO, and an immutable audit log. Argilla and Label Studio clear the bar; a Google Sheet does not.
- Store the gold set as a signed, SemVer-ed, hash-manifested artifact in the customer's object store under immutable retention (S3 Object Lock Compliance, Azure Immutable Blob, GCS Bucket Lock). Seal the 20% hold-out separately.
- The FDE owns the bootstrap; `ai-eval-engineer` owns metric-design depth; `ai-evaluation-engineer` owns release-assurance packaging. The gold set is the shared artifact across all three.
- Ch 03 picks up how these labels drive the metric portfolio (correctness, judge-anchor, pairwise, groundedness). Ch 04 picks up Promptfoo / RAGAS / DeepEval / Inspect integration. Ch 06 picks up the hold-out-driven exit eval the governance board signs. Mod-108 picks up records-of-processing and the audit artifacts the regulator reads.
