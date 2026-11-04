# Exercise 04 — Design-Partner Engagement Deliverables Walkthrough

**Estimated effort:** 2 hours
**Output:** seven sketched deliverable outlines against a chosen scenario + a dependency diagram

## Objective

Walk a complete design-partner engagement from discovery to field→product review by sketching each of the seven deliverables from Chapter 05 against a realistic (chosen or simulated) customer scenario. The point is not to *produce* polished artifacts — you will do that in Projects 101–104. The point is to understand what each deliverable's *skeleton* looks like, and how each one feeds the next.

## Prerequisites

- You have read Chapter 05 and skimmed [`CURRICULUM.md`](../../../CURRICULUM.md) so you know roughly what each downstream module covers.
- Optional, but useful: skim the [Teresa Torres Continuous Discovery](https://www.producttalk.org/continuous-discovery-habits/) overview and [Michael Nygard's ADR writeup](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions.html) so you have shape vocabulary before you sketch.

## Steps

1. **Pick a customer scenario.** Either use a real one you are allowed to discuss, or pick a realistic simulated one. Suggested archetypes:
   - A health-insurance customer wants to pilot an LLM-assisted prior-authorisation drafter inside their claims-ops workflow (HIPAA + BAA, VPC install on their Azure).
   - A mid-market law firm wants to pilot a contract-clause extraction + redlining agent on Microsoft 365 + iManage (SOC 2 Type II, EU-tenant residency).
   - A field-service company wants to pilot an LLM-assisted dispatch summariser that reads ServiceNow and writes to a technician mobile app (SOC 2, on AWS, SSO via Okta).
   - A regional bank wants to pilot a knowledge-retrieval assistant for branch operations (GLBA, SOC 2, on-prem initially, cloud-ready).

   Write two paragraphs setting the scene: customer, workflow, sponsor, main constraint. This is your working example throughout the exercise.
2. **Sketch each of the seven deliverables.** For each of discovery brief, pilot scope, eval protocol, integration design, deployment runbook, hand-off packet, field→product review, write a one-page skeleton — headings + bullet stubs + 1–2 sentences of filler where it clarifies — against your scenario. Not full deliverables; enough that a peer could see the shape and tell you what is missing.
3. **Call out your open questions.** For each sketch, list 2–4 things you would need to go find out before you could produce the real version. These are the real engagement's first-week questions.
4. **Draw the dependency diagram.** A simple Mermaid or boxes-and-arrows diagram showing the seven deliverables with arrows for each dependency (e.g., "pilot scope → eval protocol" because the success metrics in the scope drive the eval design). Annotate each arrow with the *data* that flows, not the dependency itself.
5. **Name your weak deliverable.** Of the seven sketches, which one did you find hardest or weakest? In 100–150 words, say why, and what you would do in the first week of an engagement to shore up your weak spot.

## Starter guidance

- Keep each sketch to a page. If you need more, you are pre-writing the real deliverable; stop and move to the next.
- When you do not know something, say so — write `<!-- open: who signs this? -->` or `<!-- open: what SLA does the customer need? -->`. Marking unknowns is the deliverable; filling them in is the engagement.
- The point of the dependency diagram is to make it visually obvious why you cannot skip the discovery brief and jump to integration design (or eval protocol, or deployment runbook). You will produce it; the diagram should prove you understand why the sequence matters.

## Acceptance criteria

- The customer scenario is set in two paragraphs with clear sponsor, workflow, and constraint.
- All seven deliverable skeletons exist, each with the correct headings for its type (per Chapter 05) and with at least one bullet of concrete content per heading.
- Each sketch has 2–4 open questions called out.
- The dependency diagram shows all seven deliverables, with arrows annotated by the data flowing between them.
- The weak-deliverable reflection names one deliverable, explains why, and names a concrete first-week response.

## Stretch goals

- Produce the sketches against *two* of the four archetypes above, and compare — same workflow category, different regulation / integration posture. Where does the shape of the seven deliverables change, and where does it stay the same?
- For your chosen scenario, call out one deliverable where you would *proactively* pull in a peer specialist (per Chapter 06). Name the peer and the specific question.
- Co-locate with a classmate and swap scenarios. Review their seven sketches as if you were taking over their engagement in week 4. What would you ask them for first?
