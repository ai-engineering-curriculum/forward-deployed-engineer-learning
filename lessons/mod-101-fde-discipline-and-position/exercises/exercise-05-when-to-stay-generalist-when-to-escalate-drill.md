# Exercise 05 — Generalist-vs-Escalate Decision Drill

**Estimated effort:** 2 hours
**Output:** decision log for 10 scenarios + a one-page escalation template + a short written reflection

## Objective

Operationalise Chapter 06's four escalation triggers. Work through ten pilot-shaped decisions, declare for each whether you own it, pull in a peer, or hand it off entirely, and justify each call against the triggers. By the end you should be able to make these calls in real time without a decision tree open in front of you, and to produce a clean escalation packet when you do escalate.

## Prerequisites

- You have read Chapter 06 and ideally re-skimmed Chapter 03's deferral contracts.
- You remember the four triggers: (1) depth needed > FDE depth and the pilot needs the depth, (2) decision crosses a formal boundary you cannot own, (3) work is pattern-shaped not pilot-shaped, (4) someone else is a faster, better fit.

## Steps

1. **Work through the 10 scenarios below.** For each, in your decision log (a simple table or markdown list), record:
   - **Call:** `own` / `escalate (peer)` / `hand off (not-FDE)`.
   - **Trigger(s):** none / 1 / 2 / 3 / 4, or a combination.
   - **Named peer or team** if escalating.
   - **One-sentence rationale** citing a specific constraint from the scenario.
   - **What you would ship if the answer is `own`,** or **what you would package if the answer is `escalate`.**

   **S1.** The customer's identity provider is Okta. You need SAML SSO + SCIM provisioning for ~200 users. Integration is a pattern you have used twice before.

   **S2.** The customer wants self-hosted Llama 3.1 70B inference inside their VPC on 2×A100-80GB. Required SLO: 400 ms p95, 100 RPS burst, 20 RPS sustained. No existing team at your company has shipped this exact config.

   **S3.** The customer's SME lead wants to double-label a 200-example gold set but is unsure how to measure inter-rater reliability for a nuanced legal-clause classification. You have shipped simpler gold sets but not with this label density.

   **S4.** During the pilot you discover that two unrelated customers (yours and a peer FDE's) both need a per-tenant audit-log export in a specific CSV schema. Neither pilot can go live without it. The product team has no plans for this feature.

   **S5.** The pilot is in a HIPAA-covered environment. The customer's governance board requires a formal release packet with a specific evidence structure before go-live.

   **S6.** The customer's sponsor asks you, at a weekly review, whether you could extend the pilot contract for another quarter and quotes a dollar figure they would be comfortable with.

   **S7.** Your pilot's prompt caching hit rate has dropped from 70% to 20% over the last week after a prompt-version update. The customer is seeing cost increase 3×.

   **S8.** The customer's workflow has turned out to need three cooperating agents with shared memory, a planner, and explicit message-passing — not two prompts that call each other. The design-partner spec allowed for "an agentic system" without specifying.

   **S9.** The customer's security reviewer asks for your model-weight protection posture at platform scale (how you prevent extraction attacks, how you rotate keys across tenants). You have the per-pilot answer; you do not have the platform answer.

   **S10.** A systems integrator the customer uses wants to build on top of the pilot — they are asking you for an API spec and some enablement time.

2. **Draft the escalation template.** For your three most important `escalate (peer)` cases above, draft a *single* reusable one-page escalation template — the pattern from Chapter 06: specific question, context packet, named ask. Produce it so that you could paste it into Slack / email and change only the pilot-specific fields.
3. **Write the reflection.** In 200–300 words, answer: *Which trigger do I personally over-fire, and which do I personally under-fire?* Point to one or two of your ten scenarios where you second-guessed yourself. If you cannot name a personal bias yet, describe the bias you expect to develop based on your current engineering identity (per Chapter 02's "uncomfortable where" framing).

## Starter guidance

- Resist the urge to escalate every scenario. The default is *you do it*. Half or more of the ten scenarios should land as `own`. If all ten are escalations, you are not holding the generalist stance.
- Watch for scenarios that look like one trigger but are actually another. S2 reads like depth (trigger 1) — it is. S4 reads like depth — it is actually pattern-shaped (trigger 3). S6 reads like a routine ask — it is a boundary (trigger 2). Being wrong here is fine for the exercise; being wrong in a real engagement is the drift this module is trying to prevent.
- The escalation template is a tool you will reuse in Projects 101–104 and in mod-109 and mod-111. Write it as if your future self will paste it a dozen times.

## Acceptance criteria

- The decision log covers all ten scenarios with all five cells filled in.
- Of the ten, each of `own`, `escalate (peer)`, and `hand off (not-FDE)` appears at least once (the scenarios are designed to require all three).
- Every `escalate (peer)` names a specific peer by role (from Chapter 03) or a specific team.
- The escalation template is a single page, reusable, with named slots for the pilot-specific content.
- The reflection names a personal bias and points to a specific scenario where it showed up.

## Stretch goals

- Rework one of your `escalate (peer)` scenarios as if it happened on day 5 of the pilot rather than day 50. Does the trigger still fire? Does the right peer change? Does the escalation packet change?
- Compare your decision log with a classmate's. Where you disagree, write a short arbitration note deciding which of you was closer to the Chapter 06 rule and why.
- Pre-write the 24-hour follow-up you would send to a peer after an `escalate (peer)` has resolved — what you decided together, what you shipped, what the open items are. This is the raw material for a mod-111 pattern writeup.
