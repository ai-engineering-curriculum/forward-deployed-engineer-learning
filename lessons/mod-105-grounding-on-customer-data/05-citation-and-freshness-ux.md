# Chapter 05 — Citations, Freshness, and the Evidence-Missing Refusal

## Why this chapter exists

The customer's SMEs — the CSR, the policy analyst, the branch banker, the compliance reviewer — do not accept an LLM answer on faith. The first question a competent user asks about a grounded answer is some variant of *"where did you get that?"* The second is *"when was that last updated?"* The third, for high-stakes workflows, is *"what else did you consider?"*

Three answers to those three questions are the UX contract: **citation** (which source document backs this answer), **freshness** (how recent is it), and an **evidence-missing refusal** shape (the pilot must say "I don't have grounds to answer that" without pretending). A fourth element — **confidence surfacing** — is more subtle and, applied wrong, actively hurts. This chapter walks each.

The UX also carries the permission (Chapter 03) and PII (Chapter 04) decisions through: a citation that silently points to a document the user cannot open, or that reveals a classification label the user is not cleared to see, is itself a leak. Design is the integrating surface for the three preceding chapters.

## The citation contract

A citation is a verifiable pointer. The pilot's output links a specific assertion in the answer to the specific source span (document, chunk, page, paragraph) that supports it. The user can open the source and confirm.

### Shape

Three pieces of metadata per citation:

- **Source handle** — a URL or deep link the user can open in the system of record (SharePoint page, Confluence page, Jira ticket, Salesforce record, warehouse row, Drive doc). Not a pilot-internal `doc_id` — a *human-navigable* identifier.
- **Span anchor** — chunk id, page number, paragraph, section heading. Enough that opening the source lands near the cited span. [Text fragment URLs](https://wicg.github.io/scroll-to-text-fragment/) (`#:~:text=...`) work for web-rendered sources; page numbers work for PDFs; section anchors for wiki.
- **As-of timestamp** — when the source was last modified (or last re-ingested if the system of record is append-only).

Example rendering (one of many — the shape matters, the pixels do not):

> Pre-authorisation for outpatient MRI typically requires clinical documentation attached to the PA request before the imaging centre schedules. [[Policy OP-MRI-2024, §4.1 · updated 2026-09-12]](https://sharepoint.customer.example/policies/op-mri)

### Per-assertion vs. per-answer

An answer typically asserts several things. The question is whether each assertion gets its own citation or the whole answer gets one footer.

Rule of thumb from the field:

- **Per-assertion** for answers that mix multiple sources (summarisation of three policy docs, triage of five tickets). Each asserted fact links to its own evidence. The LLM's structured-output shape (mod-104 Chapter 04) emits `{claims: [{text: ..., citations: [...]}, ...]}`.
- **Per-answer** for answers that draw from one document end-to-end (direct quote, policy lookup). One footer is honest.

The anti-pattern to avoid: **one catch-all citation at the end that lists every retrieved chunk**. The user cannot tell which chunk supports which assertion; the citation is theatre, not evidence.

### Reconstruction, not re-embedding

The citation is reconstructed from retrieval, not asked of the model. The model's job is to ground on the retrieved chunks and emit structured output marking which chunk supports which claim; the citation metadata (URL, as-of, span) is added by the pilot code from the retrieval payload. The model does not generate URLs.

Why: models hallucinate URLs. The citation is a factual pointer; it has to be exact. Have the retrieval layer return source URL + as-of per chunk, and the pilot's rendering layer build the citation from the chunk the model cited.

### The classified / sensitive case

The citation surface is itself a leak vector. If a user queries "what's the policy on X" and the pilot cites a document with classification `Restricted` or `FOUO`, the mere existence of the citation (and sometimes its title) leaks that such a document exists even to a user cleared only to see `Internal`.

Rules:

- **Filter sources before the model sees them** (Chapter 03). If the user cannot read the doc, it does not enter retrieval. Not surfaced; not cited.
- **Treat the citation title and classification as sanitised metadata.** Show source handles and span anchors only for docs the user may open. If the chunk made it through retrieval, the user is cleared to see the cite.
- **Watch the `allow_users` DAC case.** A document granted individually to the user must not reveal its title to the user's peers in a shared view. Per-user citation; no leak across users.

## The freshness contract

Freshness is a trust multiplier for grounded answers and a trust-destroyer when it is silent.

### Shape

Two things surface per answer:

1. **Per-citation as-of.** The last-modified date of the cited source. Shown inline with the citation.
2. **Answer-level freshness summary.** The oldest `as_of` across citations, or a bucketed indicator — "all sources ≤ 7 days old" / "some sources older than 30 days." Shows at a glance how confident the SME can be that nothing has shifted since.

Example:

> ...clinical documentation attached to the PA request. [[Policy OP-MRI-2024 · updated 2026-09-12]](...)
>
> *Sources consulted: 3; oldest updated 2026-09-12 (54 days ago).*

### SLOs around freshness

Two different clocks:

- **Source → index lag** (Chapter 03's invalidation SLO). When the source changes, how long until the index reflects it? Example target: *p95 ≤ 5 minutes for CDC sources, ≤ 24 hours for scheduled-crawl sources.*
- **Source last-modified → user's answer.** How old is the actual source content when the pilot cites it? Example surface: *flag answers whose newest source is older than 90 days.*

The first SLO is a pilot-side engineering target. The second is a corpus-maintenance signal the customer's content owners act on.

### Staleness actions

When the pilot detects stale evidence, three shapes of action:

- **Surface, don't suppress.** The default. Show the freshness indicator; let the SME decide.
- **Flag for the SME.** When the cited sources exceed a threshold (e.g., "all cited sources older than 6 months"), prepend a "sources may be out of date" note.
- **Refuse.** For high-stakes workflows (regulatory lookup, clinical decision support) with hard-coded freshness requirements, refuse and point to an alternative (call a human; consult the source directly). The refusal shape, not a stale answer.

The decision is per workflow. A support reply with policy from a 3-year-old doc is fine if the policy hasn't changed; a clinical dosing lookup from a 3-year-old monograph is not.

## The evidence-missing refusal

Grounded retrieval must refuse when it cannot ground. The failure mode to prevent is the LLM filling in from training data and emitting a plausible-sounding answer with no supporting evidence from the customer's corpus.

### When to refuse

Four signals, any one of which triggers refusal:

1. **No chunks retrieved.** The retriever returned zero results that passed the permission filter at the pilot's score threshold.
2. **Low top-1 score.** The highest-ranked chunk's score is below a threshold tuned on the gold set (Chapter 06).
3. **Low rerank score.** The reranker's score for every candidate is below threshold.
4. **The model explicitly says it cannot answer from the chunks** (structured output has a `grounded: false` field or similar).

### The refusal shape

The refusal is not "I'm sorry, I can't help with that" (the model's generic refusal — unhelpful, and often the trigger for the user to ask again until the model's filter misfires). The refusal is:

> I don't have grounds in your knowledge base to answer this. The sources closest to your query are [link A, updated 2024-03-11] and [link B, updated 2023-11-02] — both older than 180 days and not clearly on topic. Consider: (a) checking those sources directly, (b) escalating to [team / role], (c) rephrasing the question to be more specific.

Three properties:

- **Transparent.** Names the near-miss sources (they passed the permission filter, so they are safe to name).
- **Actionable.** Tells the user what to do next, specific to the pilot's workflow.
- **Honest about gaps.** The pilot's corpus is finite; refusal is a correct answer when the corpus doesn't cover.

### The "never make it up" prompt scaffold

The prompt (mod-104 Chapter 04) has to be shaped to refuse. A production-grade system-prompt fragment:

```
You answer only using the CONTEXT chunks provided. If the CONTEXT does not
clearly support an answer, respond with grounded=false and a one-paragraph
explanation naming which part of the question the CONTEXT does not cover.

You do not use knowledge from your training. If a fact is not in CONTEXT,
you do not state it. You may say "I don't know" or "the sources don't cover
this."

Every factual claim in your response must include a citation handle from
CONTEXT. Format: <claim text>[cite:<chunk_id>].
```

The scaffold composes with structured output. The pilot's output schema includes `grounded: boolean`, `grounded_confidence: float`, `claims: [{text, chunk_ids}]`, `no_grounds_explanation: string`. The UI renders based on `grounded`.

Mod-107 (eval engineering) is where the pilot measures *how often* the model refuses correctly vs. hallucinates. The gold set (Chapter 06) includes refusal cases.

## Confidence surfacing — handle with care

The impulse: show a confidence score next to every answer. The problem: confidence from an LLM is an unreliable signal in a well-known way — [Lin et al. (2022), "Teaching Models to Express Their Uncertainty in Words"](https://arxiv.org/abs/2205.14334) and subsequent work show that both logit-based and verbal confidence are miscalibrated. Models are often confidently wrong on what they don't know.

What to surface instead:

- **Retrieval signal.** Top-k scores, rerank scores — numbers tied to how well the corpus matched. Reliable in the "how well did we find evidence" sense; silent on whether the model used the evidence correctly.
- **Grounding verdict.** Structured `grounded: true | false` from the model, with the explanation. Not a probability; a verdict, which the UI can trust-but-verify.
- **Answer diversity signal.** If the pilot's architecture supports self-consistency ([Wang et al., "Self-Consistency Improves Chain of Thought Reasoning in Language Models"](https://arxiv.org/abs/2203.11171)), low diversity across samples correlates with answer stability. Still not a probability; a hedge.

What not to surface:

- **A raw model-emitted probability.** Miscalibrated; the UI effect is to sell false confidence.
- **A bar / meter / "high confidence."** Users read these as calibrated; they aren't.

The honest surface is often a hedge phrase in the answer itself ("Based on three sources, the typical PA requirement is..." vs. "One source from 2023 mentioned..."). Mod-112 (consulting craft) has more on narrating uncertainty.

## The UX layer and the three preceding chapters

The UX is where Chapter 02's engine, Chapter 03's permission, and Chapter 04's PII contract become visible to the user. Three integration points.

**Engine capabilities shape the UX.** Hybrid-retrieval engines (dense + sparse) support better long-tail queries and typographical variance; the UX should encourage natural-language queries rather than keyword-stuffing. Engines with first-class rerank produce tighter top-k; the UX can show fewer, higher-quality citations.

**Permission shapes the citation surface.** The citation can only point to what the user may see. Design the "no evidence" refusal to name sources the user could consult, and ensure cites never reveal document existence to unauthorised users.

**PII shapes the rendered chunks.** When a chunk is quoted in the answer, the pseudonymised tokens must either rehydrate (if the user is authorised) or render as pseudonyms. The UI is the last line; the gateway output boundary (Chapter 04) is upstream of it.

## Accessibility and jurisdiction

Two concerns the FDE will meet in some verticals.

**Accessibility.** [WCAG 2.2](https://www.w3.org/TR/WCAG22/) applies to customer-facing UX in many jurisdictions. Citation surfaces that rely on colour or hover reveals fail WCAG. Keep citations as text links, keep freshness indicators text-first.

**AI Act transparency obligations.** The [EU AI Act](https://eur-lex.europa.eu/eli/reg/2024/1689/oj) applies to many customer-facing AI features; Article 50 covers transparency obligations ("users must know they are interacting with an AI system"). For Archetype C (EU bank), the UX includes a clear "AI-generated draft" indicator and a disclosure that aligns with the customer's legal review. Mod-108 covers the formal compliance mapping.

## Three archetypes and their UX

### Archetype A — US healthcare payer (CSR assistant)

Users: CSRs tier 1 / tier 2 / supervisor. Workflow: pull up relevant policy + member history during a call.

UX: per-assertion citations into the policy repository and the member system. Freshness indicator prominent — stale policy citations drive escalations. Refusal shape: name the policy section the pilot could not find and the supervisor the CSR can consult. PII pseudonyms render to real names *only* for the member on the current call (rehydrate on the authorised case ID); generic policy quotes have no member PII to rehydrate.

### Archetype B — US federal agency (policy analyst drafting)

Users: cleared policy analysts. Workflow: draft a policy memo grounded in regulations and prior memos.

UX: per-citation to the regulation / memo plus classification label (U / FOUO / CUI). Freshness indicator critical — stale regulations are a correctness failure. Refusal shape: list the near-miss memos, decline to speculate. Sources are classified-level-aware — the draft and the citations are marked at the maximum classification of the cited sources.

### Archetype C — EU retail bank (support reply)

Users: branch support staff. Workflow: draft a reply to a customer email.

UX: per-assertion cite to product T&C / procedure. Freshness indicator — financial product T&Cs change with regulatory updates; stale citations = regulatory risk. Refusal shape: name the near-miss procedures and the escalation path. AI-generated draft indicator per EU AI Act Article 50. PII pseudonyms rehydrate only when the branch staff is on the authorised customer case.

## Anti-patterns

**Citing the chunk ID, not the source URL.** The user cannot open a chunk ID. Give them the system-of-record URL.

**One catch-all citation at the end.** Theatre, not evidence. Per-assertion when the answer mixes sources.

**Hiding freshness.** The pilot cites happily; the SME discovers in week 8 that the policy they relied on was from 2019. Freshness is non-negotiable.

**Suppressing the refusal.** The pilot tries to "always answer" because refusal is uncomfortable. The result is hallucinated grounded answers, which are worse than refusal. The refusal shape is a feature, not a last resort.

**Showing a confidence meter with no calibration.** Users read it as truth. If you cannot calibrate (and the model generally cannot), do not render a bar.

**Citing a document the user cannot open.** Permission check at retrieval (Chapter 03) is upstream of the citation. If a cite link 404s because the user is not authorised, the permission model and the citation model are not in sync.

**Showing pseudonyms the user is authorised to see raw.** Rehydration at the output boundary (Chapter 04) — not asked of the model, performed by the pilot post-generation for authorised calls.

## Summary

- The UX contract has four parts: citation (source + span + as-of), freshness (per-cite and answer-level), evidence-missing refusal (transparent, actionable, honest), and confidence (use retrieval and grounded-verdict signals; avoid raw model probability bars).
- Citations are reconstructed from retrieval, not emitted by the model. Models hallucinate URLs; the pilot's code builds the cite from the chunk payload.
- Per-assertion citations when the answer mixes sources; per-answer when it draws from one. Catch-all end-of-answer cite is theatre.
- Freshness has two SLOs: source→index lag (engineering target) and source-age-when-cited (content-maintenance signal). Stale-answer actions: surface, flag, or refuse by workflow.
- Evidence-missing refusal is table stakes. Four trigger signals; transparent refusal shape that names near-miss sources and tells the user what to do.
- Confidence: show retrieval scores and a grounded-verdict; avoid raw model-emitted probabilities and unsourced confidence meters.
- The UX layer is the integrating surface for Chapters 02–04: engine capabilities shape it, permission shapes the citation surface, PII shapes the rendered spans.
- The three archetypes land differently on freshness and classification handling but share the four-part contract.
- Chapter 06 is next: a retrieval eval loop needs a gold set, and the gold set has to come from the customer's SMEs — not from a benchmark.
