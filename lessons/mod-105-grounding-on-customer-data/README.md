# mod-105-grounding-on-customer-data: Grounding on Customer Data — Retrieval, Access, and Provenance

> Scaffolded by `aicg org execute-plan`. Lecture chapters and exercise content are authored on subsequent autonomous cycles.

**Estimated effort:** 14 hours

## Learning objectives

- Select and wire a retrieval engine per customer deployment — Pinecone / Weaviate / Qdrant / Chroma / pgvector / Elasticsearch / Vespa / Azure AI Search / Vertex AI Search — against deployment-model, hybrid-search, filter-capability, cost, and compliance-posture scoring
- Propagate the customer's permission model into the retriever — document ACLs, row-level security, group / role / attribute filters — so the FDE's pilot cannot surface a document the current user is not authorised to read; design the invalidation path for permission changes
- Ship PII-safe retrieval — Presidio or equivalent PII detection at ingest + retrieval + output boundaries, token-level redaction, reversible pseudonymisation when the business flow requires it, and the audit trail that proves the PII contract
- Design the citation / evidence / freshness UX — inline citations, source-attribution on every answer, freshness indicators (as-of timestamps), evidence-missing refusal shape, confidence surfacing
- Bootstrap a customer-data gold set for retrieval eval — SME-paired labelling of 50-200 queries against the customer's actual corpus, pairwise retrieval precision / MRR / nDCG metrics, business-KPI linkage (did the AI answer eliminate a ticket / shorten a workflow)
- Cite the boundary to `rag-engineer` (level 25) on retrieval-engine depth — chunking strategies, embedding-model trade-offs, rerankers, multi-index fusion, retrieval-level evaluation methodology — linked out for depth

## Structure

- `01-…md` … `0N-…md`: lecture chapters.
- `exercises/`: per-exercise prompts.
- `labs/`: long-form hands-on labs.
- `quizzes/`: knowledge checks.
- `resources.md`: external references.
