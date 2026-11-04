# exercise-02: Author a Design-Partner Pattern Writeup Ready for Product Consumption

**Estimated effort:** 3 hours

## Objective

Produce one monthly design-partner pattern writeup that a vendor PM could consume as-is — one to two pages, structured to the Chapter 03 template, drawing on at least two independent pilot instances, carrying at least two verbatim SME quotes, with a named recommendation and a clear customer sign-off path per the chosen archetype. The deliverable is the shape an FDE would publish at month's end in a real engagement.

The exercise tests four things at once: the compression discipline (pattern not incident, one page expanded to two), the customer-voice discipline (verbatim quotes with dated consent), the recommendation discipline (specific proposed change, not "something to think about"), and the sign-off discipline (archetype-correct approver map and timeline).

## Prerequisites

- Read [Chapter 03](../03-design-partner-artefacts.md) end-to-end; the twelve-section template is the exercise's scaffold.
- Chapter 02's capture-log schema is the raw material shape; exercise-01's output (or a reconstructed equivalent) is the input to this exercise.
- From mod-110 Chapter 06, understand the back-feed pipeline so the writeup's "related signals" section can honestly reference regression-fixture-generating signals.
- If you completed exercise-01, reuse your capture log as the evidence base. If not, author at least six capture entries (across two pilots) before starting this exercise.

## Problem statement

Pick a pattern the capture log (yours or the one supplied by the course) can support. The pattern must satisfy:

- **At least two independent pilot instances.** Different customer organisations, different industry vertical or stack or compliance posture. If the capture log does not already contain two instances, this exercise is premature; add the second-instance capture entry first.
- **At least one verbatim SME quote per instance.** Quotes must carry speaker role, date, and consent status. The writeup may not use any quote marked `consent_at_capture: pending`.
- **A reasonable recommendation.** The pattern must suggest at least one concrete change — a canary-gate preset, a feature flag, a judge-rubric dimension, a response-library entry, a dashboard panel, a deployment-playbook step. If the honest recommendation is "no change, we are documenting the pattern," the writeup names that explicitly.

Choose the **primary archetype** as the archetype of the first pilot instance; the second instance can be any archetype. The sign-off discipline applies per-pilot — Archetype A requires DPO review on HIPAA-touching artefacts, Archetype B abstracts the agency, Archetype C applies GDPR Article 32.

## Requirements

1. **Twelve-section monthly pattern writeup.** Produce `writeups/YYYY-MM-<slug>.md` populating all twelve sections of the Chapter 03 template: date, author, pilot(s), surface, triage class, consumer, pattern-in-one-paragraph, evidence (two instances), why-it-is-a-pattern, recommended direction, interim workaround, related signals, customer sensitivity + sign-off.

2. **Length between 600 and 1,400 words (body only).** The writeup is one page expanded to two, not three. Count your words; trim or expand as needed. A writeup under 600 words usually fails the evidence discipline; one over 1,400 usually fails the compression discipline.

3. **Evidence per instance.** Each instance paragraph (Section 2.A / 2.B) cites at least: the pilot date, one metric or dashboard reference, one verbatim SME quote with date, and one backing artefact link (post-mortem URL, ADR, capture-log entry ID). Quotes are *verbatim* — do not paraphrase. If you must redact for consent reasons, replace with "[consent pending; paraphrase on file]" and note it in the sign-off section.

4. **Pattern paragraph first.** Section 1 — "The pattern in one paragraph" — is a five-sentence maximum summary of the generalisation. A reader who stops after this paragraph should understand the pattern, its context, and roughly what you are going to recommend.

5. **Recommendation explicitly named.** Section 4 — "Recommended direction" — names a specific proposed change. The recommendation is not the full Chapter 04 packet; it is the writeup-level summary with a pointer to the packet if one exists. Even "no change needed" is a named recommendation with a one-sentence justification.

6. **Interim workaround named.** Section 5 — "What we are doing in the interim" — describes what each pilot is currently doing to carry the pattern. Mandatory: a writeup that proposes a platform change without naming the workaround looks like a demand (Chapter 01 anti-pattern).

7. **Customer sign-off per archetype.** Section 7 — "Customer sensitivity + sign-off" — produces, per pilot, the sensitivity level (low / medium / high), the named approver roles (SME lead / operations lead / communications office / legal / DPO), the requested sign-off date, and the redaction fallback if sign-off is declined. The approver map must be correct for each pilot's archetype.

8. **Two readers' passes documented.** Before finalising, have the writeup read by a peer FDE (or role-played persona) for the shape-of-the-pattern check, and by a non-FDE reviewer (a PM or a non-technical peer standing in for the PM) for the compression-and-recommendation check. Each reader leaves two or more comments in a `reviews/*.md` file. You address the comments and note the revisions.

9. **Depth-of-technical-detail set at "medium."** The writeup is the organisational narrative; the deep implementation detail belongs in the paired Chapter 04 packet (which this exercise does not require, but exercise-03 does). Explicitly note "Interface sketch and risk / scope in the paired platform-request packet (link)" or "Feature request below in a separate artefact" rather than inlining the deep detail.

10. **At least two `<!-- needs-research: ... -->` markers.** Primary sources you could not verify — a compliance clause citation, a specific metric from a pilot you did not run, a product feature claim — get the flag rather than a guess.

## Starter guidance

- **Open with the pattern, not the incident.** The template's Section 1 is the pattern-in-one-paragraph; if you find yourself writing "On November 4 2026…," rewrite. "On November 4 2026…" is the evidence section, not the opening.
- **The customer quote carries the writeup.** If you find yourself writing analysis in Section 2, go back to the capture log and pull a stronger quote. A quote that is three sentences long is almost always more persuasive than three sentences of your paraphrase.
- **Specifics over generalities.** "In three of the twelve pilots this quarter" beats "in several pilots." "Costing roughly one engineer-day per pilot per month" beats "at considerable cost." Chapter 03's writing-for-product discipline is quantify where possible.
- **Write the recommendation before the evidence.** Many FDEs write evidence first and discover the recommendation on the way; the resulting recommendation is often weaker than it could be. Write the recommendation first; make the evidence support it; revise the recommendation if the evidence does not actually carry it.
- **Sign-off is not optional.** The writeup is a draft until the approvers sign. Author the sign-off section as a block the approvers can read and tick; make it easy for them to say yes.
- **The honest "no change" recommendation is a valid one.** Some patterns exist because the trade-off at the current shape is the right one; the writeup's value is in documenting that the trade-off has been considered. If this is your recommendation, one sentence of justification is enough.

## Acceptance criteria

- One `writeups/YYYY-MM-<slug>.md` file populates all twelve Chapter 03 template sections, between 600 and 1,400 words of body text.
- Section 1 is five sentences maximum and summarises the pattern in a form a reader could stop after.
- Section 2 has at least two instance sub-sections, each citing pilot date, metric / dashboard, at least one verbatim SME quote with date, and one backing artefact link.
- Section 3 names the architectural shape shared across the two instances in a way a sceptical reviewer could challenge concretely.
- Section 4 names a specific recommendation (or "no change" with justification) and points to the paired packet or feature-request if applicable.
- Section 5 names the interim workaround in each pilot with the maintenance cost.
- Section 7 produces the archetype-correct sign-off path per pilot, with approver roles, requested sign-off date, and redaction fallback.
- A `reviews/*.md` file captures at least two peer-FDE comments and at least two non-FDE-reviewer comments; the writeup's revision log notes how each comment was addressed.
- At least two `<!-- needs-research: ... -->` markers are present and contextualised.
- The writeup's depth-of-technical-detail is explicitly "medium" with inline pointers to the deep-detail artefacts.

## Stretch goals

- **Companion one-page PM-facing summary.** From the two-page writeup, produce a one-page summary targeted at a PM who has 5 minutes before a meeting. The summary must preserve the pattern, the recommendation, and the customer voice while fitting a single page. Note what you dropped and why.
- **Marketing adaptation.** Produce a 300-word marketing-voice adaptation suitable for an engineering blog post, with pilot names redacted and the customer voice preserved. Note what the adaptation gained and lost versus the technical writeup.
- **Sign-off dry-run.** Role-play the sign-off conversation with each archetype's approvers (SME lead, operations lead, communications office, legal, DPO where applicable). For each approver, author two or three plausible objections and your response. Capture in `signoffs/*.md`.
- **Writeup self-review against the Chapter 03 anti-patterns.** Score your own writeup against the nine Chapter 03 anti-patterns. For any you fail, produce the one revision that would fix it.
- **Third-instance preview.** If a third pilot could plausibly add evidence, draft the Section 2.C paragraph speculatively (clearly marked as preview-not-evidence). Note what would have to be verified before the paragraph could be published.
