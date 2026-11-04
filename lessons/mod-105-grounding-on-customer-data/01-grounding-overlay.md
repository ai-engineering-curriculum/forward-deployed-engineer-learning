# Chapter 01 — The Grounding Overlay — Why Grounding Is Harder in a Customer Environment

## Why this chapter exists

Mod-104 landed the LLM layer of a pilot inside the customer's environment — the provider stack, the gateway, the prompt architecture, the per-customer SLOs. Mod-105 picks up the next layer: the pilot's answers have to be *grounded in the customer's own data*, not in the model's pretraining. Retrieval-augmented generation (RAG) is the mechanism; the customer's documents, records, tickets, policies, chats, and knowledge-base articles are the corpus; the FDE is the integrator who makes retrieval work under the customer's constraints.

The retrieval mechanics themselves — chunking, embedding, rerankers, multi-index fusion, retrieval-eval methodology — are a deep craft owned by [`rag-engineer`](https://github.com/ai-engineering-curriculum/rag-engineer-learning) (peer, level 25). This module does not re-teach that craft; Chapter 07 writes the boundary. What this module teaches is the set of *customer-environment* concerns the FDE has to layer on top of a working retrieval stack: engine selection against the customer's deployment posture, permission propagation so the retriever cannot leak, PII handling that survives security review, a citation / freshness UX the customer's SMEs trust, and a gold set grounded in the customer's workflow — not a generic benchmark.

## The in-house craft and what it covers

As with mod-104, name the ground floor before adding the overlay. The RAG engineer at level 25 owns, inside their own product or on a generic corpus:

- **Chunking** — fixed-size vs. semantic chunking, overlap policies, chunk headers, parent-document patterns.
- **Embedding** — model selection (OpenAI, Cohere, Voyage, Nomic, open weights), dimension trade-offs, re-embedding costs.
- **Indexing** — vector index parameters (HNSW `M` / `efConstruction`, IVF lists), hybrid-index construction.
- **Retrieval** — top-k, hybrid dense + sparse (BM25 + vector), [reciprocal rank fusion](https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf), query rewriting.
- **Reranking** — cross-encoders (Cohere Rerank, bge-reranker), LLM-as-judge rerank, cost / latency trade.
- **Retrieval evaluation** — recall@k, nDCG@k, MRR on standard benchmarks ([BEIR](https://github.com/beir-cellar/beir), [MTEB](https://github.com/embeddings-benchmark/mteb)).

Those six areas are *craft*. The RAG engineer builds them against generic or in-house corpora: the shape of the data is theirs to choose, the permission model is "all internal users see everything indexed," and the PII posture is the vendor's standard.

The FDE at level 35 operates one level up. The same retrieval pipeline has to work inside a customer's trust boundary, on *that* customer's documents, under *that* customer's permission model, with *that* customer's PII rules, judged by *that* customer's SMEs. Each of those re-opens a dimension the in-house craft treats as fixed.

## The grounding overlay

The grounding overlay is the set of concerns the FDE adds on top of retrieval craft. Each row is a dimension the in-house craft treats as given and the FDE re-opens per deployment.

| Dimension | What the in-house craft assumes | What the FDE re-opens | Where this module handles it |
|---|---|---|---|
| Engine deployment model | Pick one hosted or one self-hosted; stay | Match to customer's trust boundary — hosted SaaS, VPC-install, or inside hyperscaler's search service | Chapter 02 |
| Compliance posture of the engine | Vendor's own | Customer's regime (HIPAA, FedRAMP, GDPR, DORA) | Chapter 02, mod-108 for depth |
| Permission model | "All indexed users may read all indexed docs" | Document ACLs, row-level security, groups / roles / attributes, department-of-record | Chapter 03 |
| Permission invalidation | Not modelled | Users join, leave, change role; docs get reclassified; permissions must propagate to the index | Chapter 03 |
| PII at ingest | Scrub if present | Named PII contract with the customer — what gets detected, what gets redacted, what gets pseudonymised reversibly, who holds the vault key | Chapter 04 |
| PII at retrieval | Not usually distinguished | Re-check on retrieval against the current user's clearance; sanitise returned chunks | Chapter 04 |
| PII at output | Guardrail in the gateway | Final sanitiser with audit trail of what was redacted and why | Chapter 04, mod-104 Chapter 03 for gateway |
| Citation UX | Nice-to-have | Table stakes — source on every answer, freshness timestamp, evidence-missing refusal shape | Chapter 05 |
| Freshness contract | Not fixed | Customer SMEs need "as-of" times; stale answers erode trust faster than wrong ones | Chapter 05 |
| Evaluation corpus | Generic benchmarks (BEIR, MTEB) | Customer's actual queries, labelled by customer SMEs, tied to customer KPIs | Chapter 06 |
| Business-KPI linkage | Retrieval metrics only | Did the retrieved answer eliminate a ticket, shorten a workflow, change a decision? | Chapter 06 |
| Index ownership | Vendor's SRE team | Shared — customer's platform team may run the index; the FDE briefs them | Chapter 02, Chapter 07 |

Twelve rows of overlay — matching mod-104's shape. The pattern holds: craft dimensions the peer specialist resolves once get re-opened by the FDE per customer, with gate dimensions enforced first.

## The three deployment-model archetypes, re-cast for grounding

Mod-104 Chapter 01 named three deployment-model archetypes for the LLM layer: SaaS multi-tenant, dedicated SaaS, VPC install. The retrieval layer inherits them but tilts the decision table.

**SaaS multi-tenant.** Retrieval runs in the vendor's SaaS. Partitioning by tenant ID is at the index-level (namespace / tenant filter). Hosted vector databases (Pinecone, Weaviate Cloud, Qdrant Cloud) land here easily; the FDE still owns per-tenant metadata filtering (Chapter 03). Compliance surface is the vendor's; data-residency is the vendor's region policy.

*Who owns what:* vendor owns the index, the embedding jobs, the ingestion pipelines. Customer owns the source data and its feed into the ingestion.

**Dedicated SaaS.** Retrieval still runs in the vendor's cloud but in a customer-dedicated namespace or cluster. Isolation is stronger; some hosted engines (Pinecone's pod-based, Weaviate's dedicated cluster, Qdrant Cloud's dedicated cluster) support this. Residency posture is still vendor-chosen; compliance surface is the vendor's with customer-specific controls.

*Who owns what:* vendor owns the running index; customer-dedicated keys and audit stream.

**VPC install.** Retrieval runs inside the customer's own cloud. Self-hosted open-source engines (Weaviate / Qdrant / Chroma on Kubernetes, pgvector on the customer's Postgres, Vespa on the customer's cluster) or the hyperscaler's own search service (Azure AI Search, Vertex AI Search, AWS OpenSearch with vector support) are the usual landing spots. The customer's platform team often ends up operating the engine; the FDE briefs them with the same discipline mod-103 Chapter 09 names for inference and data engineering.

*Who owns what:* customer owns the running system at the infra layer. Vendor owns the application code that calls the retriever and the ingestion pipeline that feeds it. Shared on-call.

**On-prem / air-gapped.** As with LLM serving, rare but real. Only self-hosted engines apply. Embedding models also have to run in-trust — open-weights embedding models (Nomic Embed, bge-m3, e5-mistral) on the customer's GPUs.

Different from the LLM layer: for retrieval the *customer's data* is the hottest asset in the system. Every archetype has to answer "where does the corpus physically rest, who can read the index, and who can read the embeddings?" The embedding vectors are derived from the source documents and can leak information about them; some regulators treat them as derivative personal data.

## The organising budget: six dimensions

Mod-104's five-dimensional budget stretches by one for retrieval. The shape:

1. **Data residency / sovereignty** — where do the source docs, the chunks, the embeddings, the query logs physically process and rest?
2. **Compliance** — does this engine / deployment satisfy the customer's regulatory regime (SOC 2 Type II, HIPAA, FedRAMP, GDPR, DORA)?
3. **Permission fidelity** — can the engine express the customer's actual permission model with enough fidelity that no unauthorised document can surface?
4. **Latency** — can retrieval meet the pilot's end-to-end latency SLO (mod-104 Chapter 05) after every hop in the retrieval path?
5. **Cost** — does per-query and per-document-stored cost fit inside the value-per-task the pilot promised (mod-102 Chapter 03)?
6. **Hybrid / filter / freshness capability** — does the engine natively support the retrieval shape the pilot needs (hybrid dense+sparse, structured filters, incremental updates)?

Rules of thumb that carry from mod-104:

- **Residency, compliance, and permission fidelity are gate dimensions.** A choice that fails any of them is off the table — cost or latency advantages do not recover it.
- **Latency, cost, and capability are optimisation dimensions.** Among gate-passing candidates, the FDE picks the point that best fits the pilot's workflow and business KPI.

Permission fidelity is a new gate specifically introduced here because retrieval is the first place where an engine's shape *determines whether a leak is possible*. An engine that cannot express the customer's groups / roles / attribute policy cannot be fixed by application code alone without introducing a different leak vector (post-filtering, which exposes IDs and counts). Chapter 03 makes this precise.

## Three customer archetypes for the rest of the module

As with mod-104, three sketches keep the discussion concrete. The archetypes are the same as mod-104's (so the pilots compose across the two modules) with the grounding surface filled in.

**Archetype A — Mid-size US healthcare payer.**
Pilot workflow: member-inquiry summarisation. Corpus: member policies, plan documents, prior authorisations, call-centre notes. Permission shape: role-based (CSR tier 1 vs. tier 2 vs. supervisor), with document ACLs on member-sensitive items. Compliance: HIPAA + SOC 2 Type II. Deployment: AWS Bedrock LLM + a VPC-install or hyperscaler-native retrieval engine.

**Archetype B — US federal agency.**
Pilot workflow: policy-analyst drafting. Corpus: internal policy memos, regulations, FOIA-released docs. Permission shape: FOUO classification, cleared-personnel attributes. Compliance: FedRAMP Moderate baseline. Deployment: GovCloud with self-hosted inference and self-hosted retrieval; embeddings on customer GPUs.

**Archetype C — EU retail bank.**
Pilot workflow: customer-support draft replies. Corpus: product T&Cs, internal procedures, prior case notes, regulatory updates. Permission shape: branch / region / team filters, with GDPR right-to-be-forgotten obligations. Compliance: GDPR + DORA. Deployment: Azure AI Foundry + Azure AI Search in an EU region under the EU Data Boundary.

Each chapter in this module runs decisions through these three archetypes to show how the scoring moves. The archetypes are shapes, not prescriptions.

## Where mod-105 sits with mod-104 and the rest of the track

The grounding overlay only makes sense in context.

- **Mod-103 (enterprise integration)** — the data surfaces (Chapter 04) are what the retrieval ingestion pipeline reads. The MCP server (Chapter 08) is sometimes the surface through which grounded retrieval is exposed. `ai-data-engineer` boundary (Chapter 09) is the peer who owns the ingestion pipeline for a non-trivial corpus.
- **Mod-104 (LLM applications in customer environments)** — the gateway (Chapter 03) is where retrieved chunks arrive on their way to the model. The per-tenant prompt architecture (Chapter 04) composes with per-tenant retriever scoping. The per-customer latency SLO (Chapter 05) includes the retrieval hop.
- **Mod-106 (agentic workflows)** — many agent tool calls are retrieval. OWASP LLM Top 10 risks like Vector-and-Embedding weaknesses and Excessive Agency compose with this module's permission-propagation story.
- **Mod-107 (eval engineering on customer data)** — the retrieval gold set this module bootstraps (Chapter 06) is the input to the eval harness mod-107 builds. Mod-107 owns the eval *harness*; mod-105 owns the *retrieval metric definitions and the labels*.
- **Mod-108 (enterprise security and compliance)** — the formal review of the retrieval engine choice (Chapter 02) and of the PII contract (Chapter 04).
- **Mod-109 (deployment, rollout)** — rollouts of chunking / embedding changes need re-indexing plans and backfill policies that compose with the feature-flag tool.
- **Mod-110 (observability, on-call)** — retrieval latency, recall-at-k, permission-denied counts are the signals that fire in production.
- **Mod-111 (field → product loop)** — reusable retrieval patterns, chunking heuristics, and engine-choice rationales flow back to the product team.
- **Mod-112 (consulting craft)** — how the citation / evidence UX is narrated to the customer's VP of Operations; how a retrieval regression is communicated.

## Four anti-patterns to catch early

**Treating retrieval as the LLM team's problem.** The RAG engineer at level 25 owns the retrieval-stack internals, but the customer-environment overlay (permissions, PII, citation UX, gold set) is squarely the FDE's. Punting on these in week 2 means the pilot discovers them in week 10 — typically in the customer's security review, where "we haven't modelled your permission system yet" is a stop-ship finding.

**Picking the vector engine on the team's favourite stack.** The team's favourite may be a hosted SaaS the customer's residency policy forbids, or an open-source engine the customer's platform team refuses to run. Chapter 02 scores against the customer's budget, not the team's comfort.

**Indexing the corpus without a permission model.** The classic failure: the FDE ingests the whole SharePoint site / Confluence instance / Jira project into the index "to see what retrieval looks like," then discovers that the index now holds documents half the users are not authorised to read, with no cheap way to filter after the fact. Chapter 03's first rule: never index a document whose permission model is not represented alongside it.

**Treating retrieval evaluation as a per-corpus science project.** Generic MTEB / BEIR numbers do not tell the customer anything about *their* workflow. A retrieval system scoring 0.65 nDCG on MTEB-FiQA may still miss the three types of queries the customer's SMEs care about. Chapter 06's gold set is the discipline that keeps eval anchored to the customer's workflow — and ties it back to a business KPI (mod-102 Chapter 03).

## Summary

- The grounding overlay is the set of customer-environment concerns the FDE adds on top of the retrieval craft owned by `rag-engineer` (level 25). Mod-105 does not re-teach that craft; it layers.
- Twelve dimensions are re-opened per customer — from engine deployment model through PII contract to evaluation corpus. Each chapter in this module takes one or two.
- Three deployment-model archetypes (SaaS multi-tenant, dedicated SaaS, VPC install) carry over from mod-104, with retrieval's distinctive emphasis: the customer's data is the hottest asset, and the embeddings are derived personal data in some regimes.
- The organising budget is six-dimensional: residency, compliance, permission fidelity, latency, cost, hybrid/filter/freshness capability. Residency, compliance, and permission fidelity are gates; latency, cost, capability are optimisations.
- Three customer archetypes — US healthcare payer, US federal agency, EU retail bank — thread through every chapter to keep decisions concrete.
- Four anti-patterns to catch early: treating retrieval as "someone else's problem," picking engines on team preference, indexing without a permission model, and evaluating against generic benchmarks.
- Chapter 02 is next: scoring the nine candidate retrieval engines against this six-dimensional budget for a specific customer.
