# Chapter 07 — Boundary to the `rag-engineer` Peer

## Why this chapter exists

Mod-101 Chapters 03 and 06 fixed the general rule: the FDE at level 35 is a specialist IC among peers, and escalation to the peer who owns a specific craft is a discipline, not a failure. Mod-103 Chapter 09 instantiated that rule for `ai-infra-inference-engineer` and `ai-data-engineer`. This chapter does the same for the peer most relevant to grounding on customer data: [`rag-engineer`](https://github.com/ai-engineering-curriculum/rag-engineer-learning) (peer, level 25, AI Engineering family).

The retrieval craft — chunking, embedding, rerankers, multi-index fusion, retrieval-level evaluation methodology — is the RAG engineer's full-time depth. The FDE does the customer-environment overlay (Chapters 02–06) and knows enough about the craft to brief the peer correctly; the peer owns the tuning that moves nDCG and MRR against the gold set the FDE assembled.

## What the peer owns

`rag-engineer` (level 25) owns:

- **Chunking strategy.** Fixed-size, semantic / recursive splitting, parent-document pattern, structural / HTML-aware splitting, table-aware chunking, markdown-aware chunking. Overlap, header propagation, chunk-id design.
- **Embedding model selection and tuning.** Which embedding model for which corpus and query distribution — OpenAI `text-embedding-3-large`, Cohere `embed-v3`, Voyage AI, Nomic, bge-m3, e5-mistral, domain-specific embedders. Dimensionality / quantisation trade-offs (binary, int8 embeddings; Matryoshka representation learning).
- **Query transformation.** HyDE, query decomposition, step-back prompting, multi-query retrieval.
- **Hybrid retrieval tuning.** Weighting between dense and sparse, [reciprocal rank fusion](https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf) parameters, learned sparse models (SPLADE, ELSER).
- **Reranking.** Cross-encoder rerankers (`bge-reranker`, Cohere Rerank, Jina), LLM-as-judge rerank, listwise rerank; cost / latency / quality trade.
- **Multi-index fusion.** Multiple indexes serving the same query (e.g., policy index + case-notes index + external-knowledge index), result fusion strategies.
- **Retrieval-eval methodology.** Beyond the customer gold set: embedding-benchmark design on the customer corpus, ablation studies, A/B testing of retrieval configurations, [BEIR](https://github.com/beir-cellar/beir) / [MTEB](https://github.com/embeddings-benchmark/mteb) interpretation on the customer domain.

In short: *everything that makes retrieval return the right chunks with the right order at the right latency, independent of where the retrieval runs*.

## What the FDE adds

The FDE owns the *deployment-aware* parts of retrieval:

- Which retrieval engine runs inside the customer's trust boundary (Chapter 02).
- How the customer's permission model composes into the retriever's filter (Chapter 03).
- What the PII contract looks like end-to-end (Chapter 04).
- What the citation / freshness / refusal UX commits to (Chapter 05).
- How the customer gold set is assembled and labelled, and how it links to business KPIs (Chapter 06).
- How retrieval composes with the gateway (mod-104 Chapter 03), the prompt architecture (mod-104 Chapter 04), and the end-to-end latency SLO (mod-104 Chapter 05).
- How the pilot's runbook (mod-110) handles retrieval-side incidents.

The FDE does **not** own:

- Chunking strategy tuning.
- Embedding model selection.
- Reranker choice and tuning.
- Query transformation strategies.
- Embedding-benchmark design.
- Index parameter tuning (HNSW `M` / `efConstruction`, IVF lists, quantisation).

## When this boundary fires

Three cases.

**Case 1: initial retrieval stack standup for a non-trivial corpus.** The pilot's corpus is more than a few hundred documents; the query distribution is varied; the customer's workflow has three or more intents (Chapter 06). The RAG engineer scopes chunking, embedding, reranker, and base retrieval configuration; the FDE scopes engine selection, permission and PII wiring, and UX.

**Case 2: retrieval quality has plateaued below the gold-set target.** The pilot is at nDCG@10 = 0.62 and the business value model needs 0.78. The FDE handed the gold set and the hard-case rationales; the RAG engineer runs ablations on chunking, embedding, reranker, query rewriting.

**Case 3: the customer asks for a retrieval capability that needs depth.** Multi-index fusion across structured + unstructured; agent-grade retrieval (with iterative query refinement — composed with mod-106); domain-specific embedders (clinical, legal, financial). The FDE identifies the ask; the peer executes.

## What the FDE hands to the peer

A specific brief, mirroring mod-103 Chapter 09's shape.

| Field | Example |
|---|---|
| Customer + workflow | US healthcare payer; CSR assistant for policy + member lookup |
| Deployment model | VPC install; AWS region us-east-1 |
| Engine selected | Elastic Cloud (hosted) with ELSER + dense; see ADR-05 |
| Corpus description | ~500k docs, mean 2.5 KB, 20% HTML, 30% PDF (some scanned), 50% markdown. Growth ~10k docs/week. |
| Query distribution | See gold set G-01 (120 queries); intents and their distribution |
| Permission model | RBAC (3 roles) + DAC on 18% of docs; see Chapter 03 brief |
| PII contract | See PII-01; names/emails/phones pseudonymised reversibly; MRNs hashed; CCs scrubbed+alert |
| Freshness SLO | p95 source→index lag ≤ 5 min on policy repo; ≤ 24 h on case notes |
| Retrieval latency SLO | p95 ≤ 300 ms for top-k=20 incl. rerank (slice of 3 s end-to-end) |
| Current metrics | precision@5 = 0.71, MRR = 0.64, nDCG@10 = 0.69, hallucination rate 4% |
| Target metrics | precision@5 ≥ 0.85, MRR ≥ 0.80, nDCG@10 ≥ 0.80, hallucination rate ≤ 1% |
| Budget constraints | Embedding cost ≤ $X/month; rerank cost ≤ $Y/1000 queries |
| Change constraints | Chunking changes require re-index; customer prefers at most one re-index in pilot |
| Gold set | G-01-v1.2 at <path>; held-out split 20% |

With that brief, the peer can scope. Without it, the peer asks the same questions back and three days are lost.

## What the FDE receives back

- A retrieval configuration (chunking + embedder + reranker + hybrid weights + index params) with measured gold-set metrics and ablation notes.
- A re-index runbook when chunking / embedding changes.
- Latency and cost measurements under the target configuration.
- An opinion on which further retrieval moves are worth another cycle vs. which have plateaued.
- Documentation of the retrieval stack that the FDE can embed in the pilot's documentation and the customer's handoff.

## Who answers which pager

Mirroring mod-103 Chapter 09:

- Retrieval latency regression → first page the peer (unless the pilot deployed a new chunking / embedder / reranker config, which the FDE deployed). Observability dashboards show which.
- Retrieval returns obviously wrong top-k on a known gold-set query → page the peer for a retrieval-side regression.
- Retrieval returns a doc the user isn't authorised to see → page the FDE; it's a permission-propagation issue, not a retrieval-engine issue.
- Retrieval surfaces PII that should have been redacted → page the FDE; it's a PII-pipeline issue.
- Model says "no evidence" when evidence is clearly in the corpus → both — low retrieval recall (peer) *or* bad prompt / rerank threshold (FDE).

The FDE triages first; escalates to the peer with the gold-set query that reproduces the failure.

## What the FDE must know — the surface

The FDE is not expected to own the depth of the peer's craft, but is expected to speak the vocabulary correctly. The minimum surface:

**Chunking.**
- Fixed-size chunking (e.g., 512 tokens) is the baseline; semantic / recursive chunking usually improves results but costs more to pre-process.
- Overlap (~10–20%) helps claims that span chunk boundaries.
- Parent-document pattern: retrieve small chunks, return larger parents for context.
- Markdown / HTML / table structure matters — generic splitters shred tables and code blocks.
- Reference: [Lewis et al., "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks"](https://arxiv.org/abs/2005.11401) for the baseline framing.

**Embedding.**
- Instruction-tuned embedders (Cohere `embed-v3`, bge) are often stronger on domain queries than generic.
- Dimensionality matters for cost at scale; [Matryoshka representation learning](https://arxiv.org/abs/2205.13147) gives a dimensionality knob without re-embedding.
- Re-embedding on model change rewrites the index. Plan it; budget for it.
- Reference: [MTEB leaderboard](https://huggingface.co/spaces/mteb/leaderboard) — regularly updated embedder rankings by task.

**Hybrid retrieval.**
- Dense captures semantic similarity; sparse (BM25, SPLADE, ELSER) captures exact and lexical matches.
- RRF is the default fusion; weighted linear fusion is another.
- Hybrid consistently beats dense-only on long-tail queries, proper nouns, acronyms. Reference: see the extensive literature around [TREC Deep Learning tracks](https://trec-deep-learning.github.io/).

**Reranking.**
- Cross-encoders are more accurate and slower than bi-encoder retrieval; place after the initial top-k.
- Cohere Rerank, bge-reranker, Jina Rerank are common managed and self-hosted options.
- LLM-as-judge rerank is expensive and sometimes catches cases cross-encoders miss.
- A good reranker on a modest top-k often matters more than a better embedder.
- Reference: see BEIR's analysis of rerank gains across tasks.

**Multi-index fusion.**
- Multiple indexes (per source, per intent, per language) serving one query, with per-index top-k and fusion at the end.
- Enables mixing structured + unstructured retrieval, or hot + cold indexes with different freshness.
- Fusion strategies: RRF (parameter-free), weighted linear, learned.

**Retrieval evaluation (beyond the gold set).**
- Ablation studies: turn off rerank, swap embedder, change chunking; measure the delta.
- Slice analysis: nDCG by intent, by role, by source — overall numbers hide slice regressions (mod-107).
- Online evaluation: click-through, dwell time, rewrite rate, escalation rate.

The RAG engineer owns all of this depth. The FDE's job is to recognise which lever applies, brief it, and read the result.

## Anti-patterns

**The FDE tunes chunking themselves because "it's just a parameter."** The first ablation reveals how much hidden complexity there is — different chunkers for different doc types, overlap policy, header propagation, parent-document patterns. Hand it to the peer; stay on the deployment-overlay craft.

**The FDE commits a retrieval target without peer sign-off.** The customer asks for nDCG@10 ≥ 0.85 on 10k queries/day at $0.10 per 1000 queries. The peer is the one who has to meet that. If the peer sees that 0.80 is the realistic ceiling on this corpus at that cost, the peer's input must change the pilot scope — not the peer's reality.

**No gold set handed to the peer, only vibes.** Mod-107's gold set (bootstrapped in Chapter 06) is the input to the peer's tuning cycle. Without it the peer tunes against MTEB and the pilot's measured value does not move.

**Treating "retrieval" as one monolithic switch.** Chunking, embedder, hybrid weights, reranker, query rewriting, multi-index fusion — each is a separate lever with its own cost / quality curve. The peer runs them as separate experiments; the FDE reads each.

**Late escalation.** The pilot runs for 10 weeks on default LangChain chunking + OpenAI embedder + dense-only retrieval; metrics plateau at 0.55 nDCG; the customer is losing patience. The peer gets called in week 11 and has two weeks. Default to peer-in-week-2 for any corpus > a few thousand docs.

## When to try first vs. when to escalate

Mod-101 Chapter 06's rule applies: default to "you do it" unless depth, scale, compliance, or novelty triggers fire. For retrieval:

- **Try first** when the corpus is small (hundreds of docs), the workflow has one intent, and defaults will plausibly work.
- **Escalate** when the corpus is > a few thousand docs, when multiple intents compete for top-k, when the customer gold set shows plateau below target, or when the workflow needs hybrid / rerank / multi-index.
- **Try first** when wiring the retrieval engine to the customer's permission model (Chapter 03) — that is FDE craft.
- **Escalate** when tuning the engine's chunking / embedding / rerank to lift gold-set metrics — that is peer craft.

The honest week-2 answer is often: *bring the peer in now; the gold set will be ready by week 4; the tuning cycle starts then.* Planned co-ownership from the start.

## Where this boundary composes with the others

Three cases where the retrieval peer intersects with the peers from mod-103 Chapter 09:

- **Case A: retrieval corpus with managed ingest pipeline.** `ai-data-engineer` owns the ingest pipeline; `rag-engineer` owns the chunking / embedding that the pipeline runs; the FDE owns what the retriever and the gateway see.
- **Case B: self-hosted embedding model.** `ai-infra-inference-engineer` owns the embedding-model serving (e.g., bge-m3 on vLLM); `rag-engineer` owns the model choice; the FDE owns the deployment and the trust-boundary posture.
- **Case C: retrieval behind an MCP server (mod-103 Chapter 08).** The FDE exposes retrieval as MCP tools; `rag-engineer` tunes the underlying retrieval; `ai-infra-inference-engineer` serves any in-boundary models the retrieval uses.

Three to four people; one integration design.

## Summary

- The retrieval craft — chunking, embedding, rerankers, query transformation, multi-index fusion, retrieval-eval methodology — is `rag-engineer` (peer, level 25) depth. The FDE adds the customer-environment overlay from Chapters 02–06 and briefs the peer.
- The FDE hands the peer a specific brief: engine, corpus, query distribution, permission + PII contracts, freshness and latency SLOs, current and target metrics, budget, and the gold set with hold-out split.
- Three cases fire this boundary: initial stack standup, plateau below target, and depth-demanding capabilities.
- Pager rules partition cleanly: engine / chunking / embedding / rerank issues → peer; permission / PII / UX / deployment issues → FDE.
- The FDE speaks the retrieval vocabulary (chunking patterns, embedder choice, hybrid fusion, rerank, multi-index) at surface depth and reads peer outputs intelligently.
- Mod-101 Chapter 06's escalation rule applies: default-try-first on small corpora and permission wiring; escalate on scale and on retrieval-quality tuning.
- With this chapter the module closes. The next module (mod-106) builds on permissioned, PII-safe, cited, measured retrieval to design the agentic workflows that call it.
