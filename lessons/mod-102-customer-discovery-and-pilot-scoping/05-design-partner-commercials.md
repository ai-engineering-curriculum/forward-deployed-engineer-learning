# Chapter 05 — Design-Partner Commercials: Shaping the Deal with Product, Sales, and Legal

## Why this chapter exists

The pilot-scope document (Chapter 04) is the *engineering* contract between the FDE and the customer-side sponsor. It is not the *commercial* contract. The commercial contract — the Order Form, the SoW, the DPA, the design-partner side letter — is written by Sales and Legal, signed by the customer's procurement and legal counterparts, and carries the obligations both sides are committed to in a form their lawyers can enforce.

The FDE does not own that contract. The FDE *does* own the clarity of the engineering obligations inside it. A commercial contract written without engineering input routinely over-commits on SLAs, under-commits on customer-side obligations, omits the language that makes a design-partner engagement structurally different from a standard customer engagement, and leaves the FDE to discover the mismatches in Week 6 when there is no more leverage to fix them.

This chapter teaches the engineering substance the FDE pushes into the commercial conversation: what the customer is on the hook for, what the FDE is on the hook for, the explicit non-asks, and how to work with Product, Sales, and Legal without stepping on their craft. At the end, the FDE authors a one-page commercials kick-off brief that captures the commercial shape in engineering terms.

## Why the FDE has to be in the commercial conversation

Three things go wrong when the FDE is not in the room:

1. **The customer-side obligations list is thin.** Sales tends to under-ask so the deal closes. The FDE knows which obligations will determine whether the pilot succeeds (SME hours per week, named data access, named approvers). Without the FDE, these land as "best efforts" clauses that evaporate when the customer's priorities shift.
2. **The FDE-side obligations are over-committed.** Technical SLAs the FDE cannot hit ("99.9% availability during pilot" for a pilot that has one FDE and no on-call rotation). Hallucination-rate SLAs ("model will be accurate 99.5% of the time") that no FDE should ever commit to.
3. **The design-partner structure is not preserved.** The side-letter language that trades reduced-or-zero revenue for product-shape influence gets lost in a standard MSA. The customer ends up expecting a stable product; the FDE ends up shipping bespoke code; both sides are disappointed by Week 10.

The FDE is not the signer. The FDE is the engineering reviewer of what is about to be signed, and the author of the engineering-substance sections the commercial counterparts translate into contract language. The role is similar to how a security engineer reviews a contract's security exhibits — not drafting legalese, but refusing to let the legal document silently commit the engineering team to something infeasible.

## What the customer is on the hook for

The FDE pushes this list into writing — in the SoW, in an exhibit, or in a design-partner side letter. "Best efforts" is not a commitment; a number is.

**SME time.** Named SMEs, named weekly hour commitment, named backup. Not "the customer will make SMEs available." *"Mariana Diaz, Senior Clinical Reviewer, 4 hours/week; David Okonkwo, Clinical Reviewer, 2 hours/week as backup. Available means: 48h response on labelling asks; 24h response on blocking questions; attend weekly review for the duration of the pilot."* The word "available" needs its own definition because "available" means different things to different corporate cultures.

**Data access.** Named systems, named protocol, named timeline, named signer. *"Customer will provide read-only API access to the case-tracking system (REST, OAuth2 client-credentials), the benefits database (read-only Snowflake share), and the formulary store (REST). DPA signed and returned by 2026-11-20 by Henri Laurent (Security Program Manager). Access provisioned by 2026-11-25. Sanctioned environment: dedicated AWS account in us-east-1 under customer's AWS Organizations with BAA."* Without the names and the dates, data access slips by weeks and the pilot's build phase slips with it.

**Pilot evaluation obligations.** The customer commits to labelling capacity. *"SMEs will label 150 gold-set cases by 2026-11-30 using the labelling tool and protocol provided by the FDE (details in eval protocol, mod-107 Chapter 05 Deliverable 3). Sponsor will sign the exit decision based on the agreed exit criteria in the pilot-scope document."* Labelling is the single most-forgotten customer obligation; it is also the one the FDE cannot work around.

**Governance review commitments.** Named reviewers, named dates. *"Security review kickoff by 2026-11-15 (Henri Laurent, customer; FDE + ai-risk-engineer). Compliance / privacy review sign-off by 2026-12-15. Customer's AI governance committee review in week 10."* The customer's own governance team is often the pilot's longest critical path; pin their timeline early.

**Change-management obligations.** Named customer-side operator-lead who runs the hand-off with the FDE. Communications to affected users (SME team announcement, broader org communication at pilot exit). Training-session attendance by named SMEs. This section is co-authored with `ai-enablement-lead` (see Chapter 06).

**Point of escalation.** A named executive sponsor who can unblock when the pilot is stuck on internal access. *"Chen Wei (VP, Clinical Operations) is escalation point for blocked access, delayed reviews, or unresolved internal decisions. Escalation path: FDE -> customer sponsor -> Chen Wei within 48 business hours."* Without a named exec sponsor, the pilot has no lever when a cross-team dependency slips.

## What the FDE is on the hook for

Visible, dated, and specific.

**Dated deliverables.** *"Discovery brief by 2026-11-24. Pilot scope and eval protocol by 2026-12-05. Integration design v1 by 2026-12-19. Deployment runbook v1 by 2027-01-30, finalised by go-live."* Dates, not weeks. The mod-101 Chapter 05 deliverable sequence maps to a commercial calendar one-for-one.

**Build-out cadence.** *"Weekly demo (Thursdays, 30min, customer sponsor + SME lead + FDE). Bi-weekly re-decide check-in (every other Thursday, 60min, extended attendance). Monthly steering committee (exec sponsor + sponsor + FDE engineering manager + AE)."* The cadence makes drift visible before it becomes costly.

**Security-review packet responsiveness.** *"FDE will return SIG/CAIQ responses within 5 business days of receipt. Pen-test report shared under customer NDA at review kickoff. Sub-processor disclosures updated within 10 business days of material change."* Define the response times; do not leave them implicit.

**Hand-off artifacts.** The mod-101 Chapter 05 Deliverable 6 list promised explicitly. *"At pilot exit, FDE delivers: admin dashboards, SME training materials, playbooks for the six most common failure modes, deployment runbook (current), stack documentation, open-items list, contact graph."*

**On-call during pilot.** This is the clause most often mis-shaped. The FDE is not customer support; the FDE is a shared on-call engineer during the pilot. *"During pilot run-phase (weeks 6-12), FDE participates in the pilot incident rotation during business hours (customer time zone), 8am-6pm, with 1-hour acknowledgement SLA and 4-hour engagement SLA for SEV-2+ incidents on the piloted features. Not customer support. Not 24/7. Not scope for non-pilot features."* The explicit non-commitment is what prevents a Week-6 "but you're our AI engineer, can you also look at our totally unrelated ML pipeline" ask.

**Field → product deliverable commitments.** This is the design-partner-specific obligation. *"The engagement produces product learnings fed back to the FDE's product and platform teams. Customer may be named as a design partner in marketing (subject to separate marketing approval), or not — defaults to not. Customer receives a field-to-product readout at pilot exit summarising what their engagement taught the product."* The language preserves the design-partner shape of the deal.

## The explicit non-asks

The hardest section to get into a design-partner contract, and the most important. Each non-ask names a specific thing the customer, on the strength of the relationship, might reasonably ask for during the pilot — and that the FDE commits in advance is *not* in scope.

The canonical non-asks for an FDE design-partner engagement:

**The FDE is not staff augmentation.** *"The FDE is embedded for the pilot's engineering deliverables. The FDE does not take on additional non-pilot engineering work from the customer. If the customer needs non-pilot engineering support, the customer engages their own engineering team or a separate professional-services engagement (not an FDE engagement)."* This is the single most-attempted boundary violation in the FDE discipline. Put it in writing.

**The pilot is not a product commitment.** *"Pilot deliverables are design-partner artifacts. Successful pilot promotion to a general product feature is subject to the FDE's product team's roadmap decision and is not committed by this agreement. Customer continues to have access to the pilot output under the current-state terms independent of product roadmap decisions."* The customer sometimes assumes a successful pilot auto-becomes a committed GA product feature. It does not. State this explicitly.

**Scope changes require written amendment.** *"Changes to the user stories, non-goals, success metrics, or exit criteria of the pilot-scope document require a written amendment countersigned by the customer sponsor and the FDE engineering manager. Scope-change requests via email, Slack, or verbal are logged but not in effect until amended."*

**No bespoke tenant-only code paths without a written exception.** *"The FDE will not ship bespoke code paths serving only this customer's tenant without a written exception approved by the FDE's engineering manager. The design-partner arrangement trades the customer's early engagement for product-shape outputs that benefit all future customers; bespoke-only code violates the arrangement."* This is the line that preserves the design-partner shape of the deal against the gravitational pull toward "just add a flag for us."

**No SLAs on model behaviour beyond the eval protocol thresholds.** *"The FDE commits to the eval-gate thresholds in the pilot-scope document's success-metrics section. The FDE does not commit to a specific hallucination rate, factuality rate, or other model-behaviour metric beyond those thresholds."* The FDE never commits a hallucination rate. The FDE commits an eval gate. The distinction is fundamental.

**IP and data handling carve-out for field-to-product learnings.** *"Customer-specific data remains the customer's. Model prompts, retrieval-pipeline code, and infrastructure code written by the FDE remain the FDE's company's. Learnings from the engagement (patterns, failure modes, platform-gap writeups) may be used by the FDE's company to improve its product and platform for all customers, subject to Customer's confidentiality protections on specific customer data."* The FDE does not draft this; Legal does. The FDE flags it is required so it does not get dropped.

## Working with Product, Sales, and Legal

Three short sections — the FDE's interface with each.

### Product

The design-partner engagement exists because the product team has a specific bet it wants to validate. Before the FDE agrees to the engagement, the FDE asks the product counterpart directly: *"What is the specific product bet this engagement will validate or falsify?"* If the answer is vague — "we want to learn about enterprise AI deployment" — the engagement is not a design-partner engagement; it is a general customer engagement wearing the design-partner label.

The FDE is not the product manager. The FDE's job in the commercial conversation is to confirm the engagement's product thesis is nameable and to make sure the pilot-scope document's user stories are shaped so that success or failure falsifies something the product team cares about. Reference mod-111 for the full field-to-product loop; this chapter is the commercial handshake that opens it.

### Sales

The AE carries the commercial narrative to the customer. The FDE carries the engineering obligations inside that narrative. The division of labour is clean:

- *AE owns:* overall relationship, deal shape, pricing (or free-partner terms), order form draft, procurement process navigation, follow-on sales conversation.
- *FDE owns:* engineering-substance sections of the SoW or side letter, customer-obligations list, FDE-obligations list, non-asks, engineering review of any technical SLA the AE proposes.
- *Shared:* kickoff meetings, sponsor updates, re-decide gate meetings, exec sponsor escalations.

The anti-pattern to avoid: the FDE sits silent during the AE's commercial pitch because the FDE does not want to "get in the way of the deal." The AE writes a technical SLA the FDE cannot deliver. The deal signs. The pilot fails on the SLA. The FDE's silence was the proximate cause. Never sit silent through an AE-written technical SLA you cannot deliver — flag it, propose the achievable alternative, and let the AE negotiate from the FDE's version.

### Legal

Legal drafts the DPA, data residency, zero-retention, BYOK, audit-log retention, IP, confidentiality carve-outs, and sub-processor disclosures. The FDE does not draft these. The FDE reads them and flags engineering/legal mismatches.

Common mismatches to look for:

- The DPA commits to a data-retention window the FDE's observability stack cannot honour.
- The audit-log retention commits to a period the FDE's logging infrastructure does not support (see pre-mortem example in Chapter 04).
- The zero-retention clause commits to not retaining prompts; the FDE's eval infrastructure requires retaining prompts for the eval gold set. Resolve the mismatch (retain in a customer-controlled environment; pseudonymise; etc.).
- The IP clause claims customer ownership over "all outputs," ambiguously including model weights or the FDE's codebase.

The FDE is not legal counsel. The FDE's reading is: "does this contract commit the engineering team to something the engineering team cannot or will not do?" If yes, flag to Legal and to the FDE's engineering manager, and propose the engineering-feasible alternative in writing. Legal translates back into contract language. Reference [WorldCC (World Commerce & Contracting)](https://www.worldcc.com/) for background on contracting craft — the FDE does not need to be a contracts expert, but knowing the vocabulary helps.

## The kick-off brief

The one-page artifact the FDE produces that captures the commercial shape in engineering terms. It is not the contract. It is the summary Sales and Legal can read without re-asking the FDE every two days.

Structure:

```
Engagement kick-off brief

Engagement summary
  One paragraph: who, what, by when.

Customer obligations
  - SME time, with hours/week and names
  - Data access, with systems and protocols and dates
  - Pilot evaluation, with labelling capacity and sign-off commitment
  - Governance review, with named reviewers and dates
  - Change-management, with named operator-lead and training
  - Exec sponsor as escalation point

FDE obligations
  - Dated deliverables (discovery brief, pilot scope, integration design,
    deployment runbook, hand-off packet)
  - Build cadence (weekly demo, bi-weekly re-decide, monthly steering)
  - Security-review packet responsiveness
  - On-call shape during pilot (business hours, SLAs, explicit
    non-commitment to non-pilot features)
  - Field-to-product readout at exit

Non-asks (explicit, each on its own line)
  - FDE is not staff augmentation
  - Pilot is not a product commitment
  - Scope changes require written amendment
  - No bespoke tenant-only code paths without exception
  - No SLAs on model behaviour beyond eval-gate thresholds
  - IP / data handling per DPA; learnings for product improvement

Peer coordinations (see Chapter 06)
  - ai-enablement-lead: customer-side change management
  - ai-risk-engineer: governance pre-work / risk register

Named points of contact
  Customer: sponsor, SME lead, operator lead, exec sponsor, security
    reviewer, data owner
  FDE:      FDE, FDE engineering manager, AE, CSE, peer coordinations

Scope-change log (template, one row)
  Date | Request | Decision (amend / defer / decline) | Signed by
```

The brief is one page. If it exceeds one page you are duplicating the pilot-scope document. The two documents point at each other: the kick-off brief is the summary Sales and Legal use; the pilot-scope document is the engineering contract the FDE and the sponsor use. Both attach to the commercial agreement.

## Diagnostics for a healthy commercial

Before signature:

- The SME time commitment is in hours per week with names, not "best efforts."
- The non-asks are itemised, not left implicit.
- The exit-to-production criteria from the pilot-scope document appear in the SoW (or its exhibits) in substantially the same form.
- The customer-obligations list has a named signer per obligation.
- The on-call commitment is scoped to pilot features and business hours (unless the customer is paying for 24/7, which is unusual for a design partner).
- Both sides have read the non-asks and still signed.
- Legal has flagged nothing the FDE cannot deliver.
- The product team has named the specific bet the engagement is designed to validate or falsify.

If any of these is "no" at signature time, the pilot starts with a commercial shape that will fight the engineering plan. The FDE's job is to slow the signing conversation until they are all "yes." Sales will push back on this. The FDE's answer is specific: "I cannot deliver the pilot without X; here is the shape of X we need; let's take 48 hours and get it right." A pilot that signs with the wrong commercial shape is a pilot that fails on the commercial shape, not on the engineering.

## Summary

- The FDE does not own the commercial contract; the FDE owns the engineering substance inside it. Sales owns the deal; Legal owns the contract language; the FDE owns the obligations list, the non-asks, and the review of technical SLAs.
- The customer's obligations are specific: SME time with hours/week and names, data access with systems/protocols/dates, pilot-evaluation labelling capacity, named governance reviewers, named change-management lead, named exec sponsor as escalation.
- The FDE's obligations are dated and specific: the mod-101 Chapter 05 deliverable sequence, the build cadence, security-review responsiveness, hand-off artifacts, scoped on-call, and the field-to-product readout.
- The non-asks preserve the design-partner shape of the deal: no staff augmentation, no product commitment, no email-based scope changes, no bespoke tenant-only code paths without exception, no model-behaviour SLAs beyond eval-gate thresholds.
- Never sit silent through an AE-written technical SLA you cannot deliver.
- Legal translates the FDE's engineering substance into contract language; the FDE reads back for mismatches and proposes feasible alternatives.
- The kick-off brief is a one-pager Sales and Legal can use without re-asking the FDE. It summarises the pilot-scope document's commercial shape for a non-engineering audience.
- A pilot that signs with the wrong commercial shape fails on the commercial shape. Slow the signing until the diagnostics are all green.
