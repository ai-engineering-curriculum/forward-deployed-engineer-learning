# Chapter 03 — Where AI Wins vs Where AI Breaks

## Why this chapter exists

Chapter 02 produced 2-3 named candidate interventions. This chapter kills the ones that should not reach the pilot-scope document. The point is to be ruthless *before* scope is signed, not to rescue a doomed pilot in the post-mortem.

Four tests. Each one kills a specific failure mode. The sanity check kills ill-posed pilots. The deterministic test kills over-engineered pilots. The viability check kills money-losing pilots. The safety check kills reputation-losing pilots. A candidate that passes all four is a candidate that can be defended to the sponsor, to the FDE's manager, and to the customer's governance team.

The tests are not a scorecard. They are four independent gates. Failing one does not mean "subtract points." It means either "kill the candidate" or "reshape it until it passes." The FDE writes one short paragraph per test per candidate in the discovery brief, showing how the candidate passed — with numbers and named assumptions. If the paragraph cannot be written honestly, the candidate is not ready.

## Test 1 — Can a human do this in under a minute?

**The question.** Given the same inputs the model will see, can a reasonably trained human do the task well in roughly a minute?

**Why it works.** The test is a sanity check on whether the problem is well-posed at the inputs you have. If a trained human, given the same inputs, cannot do the task in a minute, one of three things is true:

- The task needs information that is not in the inputs. The model will not have it either. The problem is upstream — missing data, missing context, missing precondition.
- The task is ambiguous. The human spends ten minutes because they are hedging; the model will hedge too, and its hedges will be harder to audit.
- The task is actually three tasks. The human needs ten minutes because they switch between them. Decompose into sub-tasks each under a minute, and re-apply the test.

If a trained human easily can do the task in under a minute — glancing at the inputs and giving a defensible answer — the task is likely LLM-tractable. The pattern-match-and-emit shape is exactly what an LLM is good at.

**How to apply.** Run the test with a specific SME, on specific real inputs (anonymised if needed), in a real five-minute block. Not a thought experiment. Hand the SME the same payload the model will see, ask "what do you do?", and time it. If the SME needs to open another system, that system's content needs to be in the model's inputs — add it to the input surface and re-run the test. If the SME needs to make a call, no model will match them.

**Common mis-runs.**

- *The 20-year SME can do it in a minute but has to read six systems to do so.* The six systems are not in the model's inputs. Either enlarge the input surface to include them (and re-check the cost test) or acknowledge that the model will not match this SME.
- *The SME cannot do it in a minute but could in ten.* Different outcome. Decompose into sub-tasks of under a minute each. Some sub-tasks pass; some do not. Build the candidate around the ones that pass.
- *The SME can do it in a minute but you did not ask what "well" means.* "Well" is "within SME quality bar." If the SME would reject their own one-minute answer on review, the task is not one-minute-tractable at the required quality.
- *Running the test on the hardest cases only.* An SME who solves pathological cases in 10 minutes does not disprove the test — the pilot is scoped against typical cases. Stratify.

**Worked example.** *Draft a prior-auth response for a routine case.* An SME with five years of experience, given the structured case detail + the member's benefits + the two-sentence formulary rule, reads them in 20 seconds and types a 3-sentence response in 40 seconds. Total: 60 seconds. The task passes the sanity check. *Decide whether an appeal has merit.* The same SME reads the chart notes, pulls two prior denial letters, cross-checks against a 40-page clinical guideline, and takes 15 minutes. Fails the sanity check at the whole-task level. Decomposes: "classify the appeal type" is under a minute; "compose the response text" is under a minute given the type; "find relevant precedents" is itself a retrieval problem. The decomposition yields LLM-tractable sub-tasks; the end-to-end task does not.

## Test 2 — Does deterministic code beat the LLM?

**The question.** For this task, is there a deterministic solution (rules, regex, SQL join, lookup table, classifier against a closed vocabulary) that would be more accurate, cheaper, and faster to maintain than an LLM?

**Why it works.** LLMs are expensive per unit of work, probabilistic in output, and harder to audit than code. For any task where the signal is structured and the output space is closed, deterministic code wins on cost (zero per-call inference cost), reliability (deterministic outputs), and auditability (readable logic). Anthropic's [*Building effective agents*](https://www.anthropic.com/research/building-effective-agents) opens with exactly this reflex: start from the simplest pattern that works, and only add LLM capability when the simpler pattern does not.

The FDE's job here is to argue the deterministic side hard before accepting the LLM side. Writing the deterministic alternative in Chapter 02's candidate statement makes this test concrete: if the deterministic alternative is "a regex plus a lookup table," the LLM is probably over-engineering. If the deterministic alternative is "a 200-line rules engine with three nested branches per rule," the deterministic side is approaching the same complexity as the LLM and the LLM may be the cheaper long-term maintenance story.

**How to apply.** For each candidate, write out the deterministic alternative in enough detail to estimate its implementation and its coverage. Two numbers matter:

- *Coverage:* what percentage of real cases would the deterministic alternative handle correctly?
- *Maintenance:* how often would the deterministic logic need to change, and who maintains it?

If coverage is above, say, 80% and maintenance is bounded (a monthly rule review), lean deterministic. If coverage is below 60% or maintenance is open-ended (the rules proliferate as edge cases appear), lean LLM. The in-between is where you consider a hybrid: deterministic filter upstream, LLM on the residue.

**Common mis-runs.**

- *"The spec is messy, so we need an LLM."* Sometimes the spec is just not written. Write the spec first; see if the result is a classifier or a decision table rather than a model.
- *"It's AI, so we need an LLM."* Internal pressure to "ship AI" drives teams to LLM candidates that are actually classification problems with a closed vocabulary. Decline, honestly.
- *Pretending deterministic coverage is lower than it is.* A hastily-estimated "the rules would only cover 40%" is often an unexamined guess. Sample 30 real cases; apply the rule set; count. The real coverage often surprises both directions.
- *Pretending deterministic maintenance is cheaper than it is.* A rule engine is cheap until the rules multiply. Weigh maintenance realistically.

**Worked example.** *Route an incoming support ticket into one of 7 standing queues.* The output space is closed (7 classes). The historical ticket data is labelled. The deterministic alternative is a trained classifier with per-class thresholds — a 2-day build in scikit-learn, 95%+ accuracy on common classes, maintenance by retraining quarterly on fresh labels. LLM alternative would also work, at roughly 100× the per-ticket inference cost and with the same maintenance burden. Deterministic wins; the candidate goes to Bucket B of Chapter 02. *Draft a plain-language explanation of a benefits denial.* Open text generation against a varying input corpus. Deterministic alternative would be a template library — likely 60% coverage of simple cases, worse on the long tail where empathy and nuance matter. LLM wins; the candidate stays in Bucket A.

## Test 3 — Does cost-per-task beat value-per-task?

**The question.** At the model and provider you plan to use, what does one unit of work cost in dollars? What is the value per unit of work from the customer's perspective? Multiply by expected volume; does the pilot clear a sensible margin?

**Why it works.** A pilot at five cents per task that prevents a $200 claim denial is a slam dunk. A pilot at two dollars per task that saves two minutes of an SME paid fifty dollars an hour is losing money (roughly $2 cost vs ~$1.67 of saved SME time). The viability check surfaces these arithmetic mismatches before scope is signed, not six months into production when the finance team notices.

**How to apply.** Build a simple table with four columns: cost components, estimated cost per task, value components, estimated value per task. Multiply by volume to project monthly spend and monthly value.

Cost components the FDE must include:

- Prompt tokens × model input price (from the provider's public pricing page).
- Output tokens × model output price.
- Retrieval calls (per-query cost of the vector DB or search engine; often small but not zero).
- Tool calls (if agentic).
- Any extraction or classification sub-calls.
- Observability, logging, eval judge calls (often an order of magnitude smaller but not zero at scale).

Value components the FDE must include:

- Time saved per task × loaded SME hourly cost (loaded = salary + benefits + overhead; typically 1.3-1.5× base).
- Any revenue recovered per unit (if the pilot reduces a leak).
- Any cost avoided per prevented incident (chargebacks, reworks, SLA penalties).

Then apply the two corrections that most FDEs miss:

- **Review overhead.** If the SME spends 4 minutes QA'ing a draft, that is not "the SME saves the whole 7 minutes of drafting." The saving is drafting time minus review time. If review time approaches drafting time, the candidate is marginal — or reshape it (Test 1's decomposition).
- **Bottleneck pricing.** When the SME is the bottleneck on throughput (the business can't hire fast enough; the queue backs up and causes SLA misses), the real value per saved minute is not the SME's hourly rate but the throughput unlocked. In a prior-auth payer example, reducing time-to-decision from 48h to 24h may unlock a per-contract revenue or avoid a per-case regulatory penalty that dwarfs the SME hourly cost. Name which lens applies before you multiply.

**Common mis-runs.**

- *Forgetting retrieval and tool costs.* They are small per-call but add up.
- *Using list price of SME time when the constraint is throughput.* Under-counts value.
- *Using "hours saved" from a self-reported SME estimate rather than from Chapter 01's observed time.* Overstates value — SMEs usually over-estimate their own time per task.
- *Ignoring the eval-time cost during pilot.* The judge-as-LLM evaluator runs on a sample of production traffic; it is a line item, and in some pilots it is the biggest line item.
- *Projecting to future model prices.* Price with today's prices from the provider's public page. Prices do drop; do not bet the pilot's viability on that drop.

**Worked example.** Candidate: LLM-drafted prior-auth response. Current provider list prices (look them up at pilot kickoff; do not rely on figures that may be out of date in this chapter).

| Component | Cost per task |
|---|---|
| Prompt tokens (case detail, benefits, formulary rule, 3 retrieved similar cases) | ~4k tokens in |
| Output tokens (3-sentence draft) | ~300 tokens out |
| Retrieval (3 vector-DB queries) | 3 × unit price |
| Judge eval on 10% sample | 0.1 × model-call equivalent |
| **Estimated total** | *fill in with current provider pricing* |

Value side. Observed Chapter 01 metric: 7 min drafting. Loaded SME cost: estimate with the customer's HR. Review overhead: 2 min per case (from shadow). Net saved SME time: 5 min per case.

Volume: 800 cases per week per SME × 10 SMEs = 8,000 cases/week.

Margin check: *task_cost × 8,000 vs. (5 min × loaded_rate) × 8,000 per week*. If task cost is $0.05, weekly cost is $400. If loaded hourly rate is $75/hr, weekly value is $50,000. The margin passes dramatically. If task cost is $2 and the loaded rate is $40, weekly cost is $16,000, weekly value is ~$13,333 — negative margin. Same candidate; different arithmetic; different verdict.

## Test 4 — Is the worst failure mode tolerable at pilot error rate?

**The question.** What is the worst thing this intervention could do on its worst day? What error rate can you realistically achieve at pilot scope? Does the worst-day failure at that error rate remain tolerable to the customer, their users, and their regulator?

**Why it works.** A 5% error rate on a draft-reply-for-SME-to-review feature is fine — the SME catches the 5%. A 5% error rate on an auto-send-to-customer feature is a crisis — 5 in every 100 customers get the wrong information. The safety check sets an upper bound on how much autonomy the pilot gives the model, and that bound is where the SME-in-the-loop checkpoint from Chapter 02 gets its design.

**How to apply.** For each candidate, write the worst concrete failure. Not "bad output" — a specific concrete event. For the LLM-drafted prior-auth response: *"the draft contains a wrong benefits determination and the SME, under time pressure, misses it on review; the member receives an incorrect denial; the member or their provider files a grievance; the payer's compliance team opens an investigation."* For an auto-reply customer support bot: *"the bot tells a customer the wrong refund policy; the customer writes a public review; the support team spends a day in cleanup."*

Then two numbers:

- *Expected error rate at pilot scope.* Not aspirational; achievable with the gold set size, prompt effort, and model you plan to use. Base on comparable cases from the FDE team's prior engagements, or on a small 50-case dry run.
- *Cost per worst-case event.* Regulatory fine, reputational damage, customer-support cost, legal exposure. If the worst case is a regulatory incident at $10,000+, that must be in the number.

Error-rate × volume × cost-per-event = tolerability floor. If that number exceeds what the customer can accept, the candidate must be reshaped (add human review, shrink autonomy, narrow scope) or killed.

The reshape path is important. A candidate that fails the safety test at auto-action autonomy often passes at human-in-the-loop autonomy. The FDE's job is to find the autonomy level that passes — not to force the highest-autonomy design.

**Common mis-runs.**

- *Mixing "the worst case is intolerable" with "the worst case is unlikely."* An unlikely but intolerable event is still intolerable at scale. A 0.5% probability × 10,000 cases per week = 50 intolerable events per week.
- *Defining the worst case as "the model is wrong" without naming the downstream consequence.* The downstream is what the customer cares about.
- *Failing to consider the compounding failure.* An SME who reviews a draft in 30 seconds because the team is at 95% draft acceptance rate is not meaningfully reviewing. The model's effective autonomy is higher than the design says.
- *Pretending the worst case does not apply because "the SME will catch it."* Will they? At what review time? With what training? Instrument review quality in the eval protocol (mod-107) — do not assume it.

Reference the [OWASP Top 10 for LLM Applications (2025)](https://owasp.org/www-project-top-10-for-large-language-model-applications/) and the [OWASP Agentic Security Initiative](https://genai.owasp.org/initiative/agentic-security-initiative/) for the vocabulary of concrete LLM failure modes (prompt injection, excessive agency, data leakage, model denial of service, insecure output handling, and the agentic variants). Not every failure mode applies to every candidate, but running the candidate against the list surfaces the ones you have not yet thought about.

The failure modes named in this test also become the pre-mortem entries in Chapter 04's pilot-scope document.

**Worked example.** *Candidate: LLM-drafted prior-auth response, SME-reviewed.* Worst case: SME misses a wrong benefits determination; member is denied in error; compliance incident. Expected error rate at pilot: assume 15% of drafts contain at least one factual error; SME catches most; residual unflagged error rate aim ≤ 1%. Volume: 8,000 cases/week. Cost per worst event: $10,000-$50,000 regulatory/legal exposure. 1% × 8,000 × $10,000 = $800,000/week. Intolerable. *Reshape:* SME training to focus review on the benefits determination field specifically; UI highlights the model-generated fields; eval protocol measures SME-caught vs SME-missed errors weekly; auto-block if residual rate exceeds 0.1%. Re-check: with the reshape, residual target 0.1%, weekly expected loss drops to ~$80,000/week at the high cost assumption — still high; narrow pilot volume to a single product line for the first phase; re-check at $8,000/week expected worst-case exposure; acceptable to the customer if the pilot delivers the time-saving value. The candidate survives, but only at a reshaped autonomy and scope.

*Candidate: auto-send customer-support reply, unassisted, on password resets.* Worst case: customer receives wrong instructions, locks themselves out, escalates. Expected error rate at pilot: 5% (narrow domain, but LLMs still invent). Volume: 10,000/week. Cost per worst event: low — 15 minutes of support handling. 5% × 10,000 × $10 = $5,000/week. On the arithmetic, tolerable. On brand risk and customer-trust risk, still worth a human-in-the-loop check in pilot; graduate to auto-send after the pilot demonstrates the error pattern.

## Running the four tests as a sequence

The tests are independent but sequencing them in order speeds the triage:

1. **Sanity check first.** If a trained human cannot do the task in a minute on the inputs the model will have, the candidate is ill-posed. Kill it before spending time on cost arithmetic.
2. **Deterministic check second.** If deterministic code beats the LLM, the candidate goes to Bucket B (Chapter 02). Do not spend the pilot budget on an over-engineered solution.
3. **Viability check third.** If the cost-per-task is close to or above the value-per-task, the candidate is marginal. Reshape or kill.
4. **Safety check fourth.** If the worst-day failure at the achievable error rate is intolerable, reshape the autonomy level (human-in-the-loop, narrower scope, higher gating) or kill.

A candidate must pass all four. A candidate that passes three and marginally fails the fourth is still killed-or-reshaped — not pushed through on the hope that the fourth will improve. The fourth is the one most likely to be true on launch day and least likely to be fixed by iteration.

## What passing looks like in the brief

For each candidate the FDE carries forward to the pilot-scope document, write a short paragraph per test in the discovery brief. Four paragraphs of 3-5 sentences each.

An example paragraph for Test 1 on the prior-auth drafting candidate:

> *Sanity check — PASS.* Ran a shadow with Mariana (senior clinical reviewer) on five representative cases from last week. Given the structured case detail, the member's benefits payload, and the two-sentence formulary rule, Mariana read-and-drafted the response in a median of 55 seconds. The three edge cases (an appeal, a prior-denial reversal, a missing-info return) each took 4-7 minutes; those are out of scope for the pilot per the non-goals section of the brief.

The paragraph names what was done, what was observed, and what was concluded, with the numbers and the limits. Four paragraphs of this shape per candidate are what the sponsor and the FDE's manager need to see.

## Summary

- Four decision tests gate candidate interventions before pilot-scope is drafted: the human-in-under-a-minute sanity check, the deterministic-code-beats-the-LLM test, the cost-per-task vs value-per-task viability check, and the failure-mode-is-tolerable-at-pilot-error-rate safety check.
- Each test kills a specific failure mode: ill-posed pilots, over-engineered pilots, money-losing pilots, reputation-losing pilots.
- The tests are independent gates, not a scorecard. Failing one means reshape-or-kill, not "subtract points."
- Apply them in order; the sequencing speeds triage.
- Document each passing candidate with four short paragraphs in the discovery brief — named numbers, named sources, named limits.
- The safety test's named worst-case failures become the Chapter 04 pre-mortem entries. The viability test's cost arithmetic becomes the Chapter 04 ADR on provider choice. The deterministic test's argument becomes the Chapter 04 ADR on orchestration framework.
