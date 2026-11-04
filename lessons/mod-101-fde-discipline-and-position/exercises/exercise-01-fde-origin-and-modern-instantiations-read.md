# Exercise 01 — FDE Origin and Modern Instantiations: A Structured Read

**Estimated effort:** 1 hour
**Output:** one page of annotated notes + a classification table

## Objective

Internalise the Forward Deployed Engineer role's lineage by reading primary sources from the role's three major instantiations and classifying them against the framework in [Chapter 01](../01-origin-and-modern-instantiations.md). By the end, you should be able to pick up any FDE-shaped job posting and tell the reader which variant it is from, which product company's lineage it inherits, and what the probable day-to-day looks like.

## Prerequisites

- You have read Chapters 01 and 02 of this module.
- You have `PREREQUISITES.md`-level fluency in LLM-application vocabulary.
- You can reach the open web (all sources below are public).

## Steps

1. **Read the Palantir origin source.** Start with [*A day in the life of a Forward Deployed Software Engineer*](https://blog.palantir.com/a-day-in-the-life-of-a-forward-deployed-software-engineer-b2142f1e36c8) and browse the [Forward Deployed Engineering tag on the Palantir blog](https://blog.palantir.com/tagged/forward-deployed-engineer). Take notes against three questions:
   - What is the customer actually doing when the FDE is on-site?
   - What does the FDE physically ship?
   - What goes back to the Palantir product team?
2. **Read one frontier-lab source.** Pick *one* of: [OpenAI customer stories](https://openai.com/customer-stories), [Anthropic careers — Applied AI / Solutions Engineering](https://www.anthropic.com/careers), or the Scale AI careers page. Find the specific team/role page (not a corporate overview). Take notes against the same three questions.
3. **Read one applied-AI product-company source.** Pick *one* of: a Sierra, Decagon, Hebbia, Writer, Harvey, Glean, or Cresta engineering-role page or engineering blog. Take notes against the same three questions.
4. **Build the classification table.** Fill a 4-column table (Palantir / Frontier lab / Applied-AI product / Hyperscaler-SaaS) with five rows: product shape, primary technical lever, customer-side peer, compliance surface, what goes back to product. Use the table in Chapter 01 as the template; refine it from your reading.
5. **Pick a wild posting.** Find one live job posting *not* from the sources above (search LinkedIn / the company's careers site / a job aggregator). Paste the posting's title and company into your notes, then classify it against your table. Which instantiation does it match? What signals told you?
6. **Write the comparison paragraph.** In 150–250 words, answer: "Across these three sources (plus the wild posting), what changes and what stays the same?" Point to specific phrases in the sources, not generalities.

## Starter guidance

- Treat marketing language (customer stories) with mild skepticism — they will over-sell influence on the product and under-mention the long nights of integration work. Look for the implied pains ("we worked with the customer's security team for X weeks") rather than the celebratory outcomes.
- Expect each posting's requirements list to be aspirational. Infer the day-to-day from the *responsibilities* list and the *team fit* language, which are usually more honest.
- Keep your classification table short. If you find yourself writing paragraph-length cells, you have not synthesised yet.

## Acceptance criteria

- Notes against the three questions for each of the three sources are in your file, with at least one direct quotation or specific example per source.
- The classification table is populated for all four columns × five rows, with cells you could defend to another engineer.
- The wild posting is classified with the signal that drove the classification.
- The comparison paragraph identifies at least one *difference* across the sources and at least one *invariant* across all of them.

## Stretch goals

- Add a fifth column to the table for the vertical-AI variant (one of: Qventus, Abridge, PathAI, Casetext/Thomson Reuters CoCounsel, Moveworks, EvenUp). Compare against the first four columns.
- Identify one phrase in one posting that looks FDE-shaped but, on closer reading, is a different role in disguise (AE in technical clothing, pre-sales SE with no post-sales scope, CSM with no engineering authority). Name the giveaway.
