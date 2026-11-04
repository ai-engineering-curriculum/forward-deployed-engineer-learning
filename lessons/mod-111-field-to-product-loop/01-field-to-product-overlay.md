# Chapter 01 — The Field-to-Product Loop — Why the FDE Is the Product Team's Field Eye

## Why this chapter exists

[Mod-110](../mod-110-observability-and-oncall/) closed the operational loop: every production incident becomes at least one permanent regression fixture; every customer bug report becomes at least one labelled gold-set entry. That pipe keeps the *pilot* healthy. Mod-111 opens a different pipe — the one that keeps the *vendor's product* honest about what customers actually hit in the field — and the FDE's accumulated knowledge useful to the next FDE on the next pilot.

Three pipes live in this module, each with a different audience and a different cadence:

1. **Field telemetry triage** — every signal the FDE sees (an incident, a bug report, a pilot's design choice, a security-review answer, a customer frustration) is classified: *bug* the FDE fixes inside the pilot, *feature* the FDE surfaces to product for the next release, or *platform gap* the FDE surfaces to `ai-platform-engineer` / `senior-agentic-ai-engineer` / `ai-engineering-manager` at level 40.
2. **Design-partner artefacts** — the pattern writeups, pain-point taxonomies, 'what we learned' one-pagers, and customer success stories that the vendor's product, marketing, and engineering leadership consume on a cadence they can count on.
3. **Internal knowledge base** — the pattern catalogue, failure-mode library, deployment playbook, and security-review-answer library that scales the FDE team: the fourth FDE onboarded should be productive at month three, not month twelve.

This chapter frames the three pipes together so the rest of the module can specialise. Chapter 02 is field telemetry + triage. Chapter 03 is design-partner artefacts and cadence. Chapter 04 is the platform-request packet for the core team. Chapter 05 is the product-review meeting and roadmap influence. Chapter 06 is the knowledge-base slice the FDE team owns.

## What has *not* changed, and the one thing that has

The FDE's job through mod-104 — mod-110 has been to make one pilot work for one customer. Mod-111 is the chapter in which the *second customer* starts to matter. A finding from the healthcare payer in September only becomes a finding when it shows up again at the retail bank in November — two instances is the threshold that turns a per-pilot workaround into a platform-request packet. The in-house crafts the FDE has been overlaying since mod-104 are themselves relying on the FDE for exactly this signal: cross-customer pattern detection the vendor product manager cannot do from a roadmap view.

### The peers the FDE is now routing to

- **`ai-engineering-manager` (level 40, FDE family).** The FDE's manager. Owns the FDE team's hiring, headcount, utilisation, and — crucially for this module — the FDE team's voice in the vendor's roadmap. The roadmap influence channel goes *through* `ai-engineering-manager`; the FDE does not walk into the product-review meeting unaccompanied. [Mod-101 Chapter 02](../mod-101-fde-discipline-and-position/02-deferral-contract.md) named this boundary; mod-111 Chapter 05 operationalises it.
- **`senior-agentic-ai-engineer` (level 40, AI Engineering family).** The senior IC on the vendor's core agentic / LLM-platform team. Owns the shared agent SDK, the shared judge library, the shared guardrail library, the shared tool-call surface. Platform-request packets about agent patterns, tool-call shape, or judge/guardrail primitives route here. The handoff is: *FDE brings the field evidence; `senior-agentic-ai-engineer` owns the design and the implementation.*
- **`ai-platform-engineer` (peer, level 35, Infra / Platform family).** Already in play from [mod-110](../mod-110-observability-and-oncall/) — owns cross-fleet telemetry, judge, cost rollups. Platform requests about shared infra (telemetry schema, cost budget primitives, retention policy) route here as well. <!-- needs-research: confirm the exact repo URL and family positioning for ai-platform-engineer-learning in the ai-engineering-curriculum org before publishing -->
- **`ai-enablement-lead` (peer, level 35, FDE family).** Owns customer-education content — the training materials, the SME certification curriculum, the public-facing patterns. Field patterns that teach *the customer* (not the product team) route here. The FDE writes once; `ai-enablement-lead` adapts to the public-facing shape.
- **Vendor product manager (PM) for the pilot's surface area.** The owner of the roadmap the FDE is trying to influence. Reads the pattern writeups. Attends the quarterly product review where the FDE presents evidence. Decides, with their manager and the vendor's product leadership, which platform requests become next-release features.
- **Vendor product marketing / customer marketing.** The consumers of the 'customer success story' artefact. Read with a different lens than product engineering — looking for proof points, quotable language, metrics that can appear in a public case study. The FDE writes the technical version; marketing adapts.

Three in-house functions. Six peers. The FDE is at the intersection; nothing in this module lets the FDE skip the intersection.

## The organising budget: five dimensions

Mod-104 onward used a six-dimensional budget. Mod-111 shifts the shape — the dimensions below are the ones a field-to-product loop is measured against.

1. **Signal completeness.** Every pilot event that *could* become a product-visible pattern is captured somewhere a future FDE or PM can retrieve. Measured by field-telemetry-capture rate (incidents, bug reports, pilot ADRs, security-review Qs with a non-boilerplate answer, customer-frustration notes from weekly reviews) vs the module's capture discipline; the rate of findings that surface late ("we *had* that signal six months ago, we just didn't write it down").
2. **Triage accuracy.** Every captured signal is correctly classified as bug / feature / platform gap. Measured by reclassification rate (how often the triage is corrected on second look), by the median time from capture to triage, and by the fraction of platform-request packets rejected on first review because the triage was wrong.
3. **Artefact readiness.** On the cadence that product leadership expects (monthly pattern writeups, quarterly 'what we learned' one-pagers, per-sold-customer success story), the artefact is ready, factual, and signed off by the customer where sign-off is needed. Measured by on-time delivery against the cadence, by the number of artefacts returned for factual correction, and by the fraction of artefacts the customer blocks over confidentiality.
4. **Platform-request credibility.** A platform-request packet that lands on `senior-agentic-ai-engineer`'s or `ai-platform-engineer`'s desk has the evidence, the interface sketch, the risk/scope, and the interim workaround in a form the recipient can act on. Measured by acceptance rate on first review, by the number of platform requests the core team subsequently *ships*, and by the fraction of platform requests where the FDE's proposed interface sketch survives into the final implementation.
5. **Knowledge-base compounding.** The next FDE onboarded to a similar pilot shape finds the pattern in the knowledge base, reuses the deployment playbook, cribs from the security-review-answer library, and asks fewer questions. Measured by knowledge-base hit rate during onboarding (how often a new FDE finds the answer without asking a human), by new-FDE time-to-productive, and by the staleness index of the knowledge-base itself (last-edited-date distribution across entries).

### Gate dimensions

**Triage accuracy and artefact readiness are gates.** A pipe that routes bugs to the product team as features, or misses the product-review window because the one-pager isn't ready, has not passed the gate — knowledge-base cleanliness does not recover it.

Triage accuracy is a gate because a mis-routed signal is worse than a missed signal. The product team that receives three "platform gaps" that are actually one-pilot prompt bugs learns to ignore the FDE's inbox; the next real platform gap waits behind that reputation damage. Chapter 02 writes the triage discipline — including the "two-customer rule" for promoting a bug to a platform gap.

Artefact readiness is a gate because the PM's planning cycle is not flexible. Missing the quarterly product-review window because the one-pager wasn't ready means the finding ships *six months later*; in a market this fast, six months is the gap between "we were ahead" and "we were behind." Chapter 03 writes the cadence, the template, and the customer sign-off path that keeps artefacts on time.

### Optimisation dimensions

**Signal completeness, platform-request credibility, and knowledge-base compounding are optimisations.** Among gate-passing designs, the FDE invests in the ones that most improve this customer's pilot and this FDE team's leverage.

Signal completeness is the dimension that most protects *future* findings. An FDE who captures every pilot ADR, every frustration from the weekly review, every security-review answer where boilerplate wasn't enough is an FDE who, six months in, has the raw material to produce a pattern writeup from. Chapter 02 writes the capture discipline.

Platform-request credibility is the dimension the core team cares about most. The FDE who lands a well-formed platform-request packet twice develops credibility; the FDE who lands four unresolved "asks" becomes background noise. Chapter 04 writes the packet shape.

Knowledge-base compounding is the dimension the FDE team's manager cares about most. A knowledge base that grows but no one reads is a cost; a knowledge base that new FDEs open first is the organisational leverage the FDE team is supposed to produce. Chapter 06 writes the shape, the ownership, and the review cadence.

## The triage model in miniature

Chapter 02 goes deep; this section fixes vocabulary.

Every field signal lands in one of three triage buckets:

- **Bug (ship a fix inside the pilot).** Something is wrong in *this* pilot that the FDE can fix inside the pilot's configuration — a prompt change, a retrieval filter tweak, a judge-rubric calibration, a guardrail threshold adjustment, a dashboard clarification. Blast radius is this pilot. Delivery is this sprint. The product team hears about it as a monthly pattern-writeup if the fix is interesting; otherwise it is logged and moves on.
- **Feature (surface to product for the next release).** The pilot is working as designed but the customer would be better served by a product capability that does not exist. Fits the existing platform's shape; a product manager can prioritise it into the next release; the vendor's engineers can implement it without rearchitecting the platform. Blast radius is every customer. Delivery is the next product-release train. The FDE packages a feature request (Chapter 04's lightweight form) and shows up in the product-review meeting (Chapter 05) with the evidence.
- **Platform gap (surface to platform-engineering / senior-agentic-ai-engineer / ai-engineering-manager at level 40).** The pilot is working *around* something the platform does not do. The workaround is costing the FDE time, risk, or both. A fix at the platform level would help this pilot and every other pilot. Fits the shape of the vendor's shared infra, shared agent SDK, shared judge library, shared guardrail library, shared tool-call surface. Blast radius is every pilot. Delivery is a quarter or more. The FDE packages a platform-request packet (Chapter 04's heavyweight form) and routes it to the level-40 peer who owns the surface.

Three buckets. The classification is per-signal; a single incident can produce one bug fix, one feature request, and one platform-request packet if the shape warrants it.

### The two-customer rule

A single-pilot workaround is not a platform gap. The two-customer rule says a bug is not promoted to a platform-request packet until the same shape is observed in at least two independent pilots — different customer, different industry, different stack. Chapter 04 formalises this; Chapter 02's triage default is *bug* unless the FDE can name a second customer whose pilot would benefit.

The rule exists because a platform-request packet consumes core-team cycles, and a platform change costs every pilot the core team then supports. Two customer instances is the threshold where the platform cost pays itself back; one is not. The FDE who proposes a platform change on the strength of one pilot's experience is the FDE whose next three packets are ignored.

## The cadence model

The three pipes have different cadences, and the FDE's week should reflect them.

| Pipe | Cadence | Principal consumer | Chapter |
|---|---|---|---|
| Field telemetry capture | Continuous (as events happen) | Future FDE on this pilot + Chapter 03's writeups | 02 |
| Triage (bug / feature / platform gap) | Weekly | FDE + pilot SME lead + customer operations lead | 02 |
| Pattern writeup | Monthly | Vendor PM + vendor engineering | 03 |
| 'What we learned' one-pager | Quarterly | Vendor product leadership + sales + marketing | 03 |
| Customer success story | Per sold outcome (irregular) | Vendor marketing + sales + customer | 03 |
| Platform-request packet | As the two-customer threshold is crossed | `senior-agentic-ai-engineer` / `ai-platform-engineer` / `ai-engineering-manager` | 04 |
| Feature request | Continuous; batched into quarterly review | Vendor PM | 04 |
| Product-review meeting | Quarterly, standing | Vendor PM + `ai-engineering-manager` + FDE | 05 |
| Knowledge-base entry | On every pattern worth generalising | FDE team + onboarding FDEs | 06 |
| Knowledge-base review | Quarterly (per entry) | FDE team lead + contributing FDEs | 06 |

Nine cadences across five chapters. The FDE whose calendar has none of them is the FDE operating as a solo consultant; the FDE whose calendar has all nine is the FDE whose work is compounding.

## Three customer archetypes for the module

Same three archetypes used from mod-104 onward, with the field-to-product posture filled in.

**Archetype A — US mid-size healthcare payer.** Member-inquiry resolution; HIPAA + SOC 2 Type II; AWS Bedrock + VPC install. Field-to-product posture: customer sign-off on every artefact that leaves the pilot — a monthly pattern writeup goes through the customer's communications office; a customer success story through legal and marketing before it is public. Pattern writeups de-identified per the data contract from [mod-108 Chapter 05](../mod-108-enterprise-security-and-compliance/05-data-contract.md). HIPAA breach-adjacent findings route through the customer's privacy office before any external artefact is produced. The FDE drafts; the customer approves; the vendor publishes.

**Archetype B — US federal agency.** Policy-analyst drafting; FedRAMP Moderate (path to High) + NIST 800-53 Rev 5 + NIST AI RMF 1.0 + OMB M-24-10 (and successor memos); GovCloud + self-hosted inference. Field-to-product posture: no external artefacts without the agency's public-affairs office approval, which is rarely granted. Pattern writeups and platform requests circulate inside the vendor only; the "customer" in the customer success story is pseudonymised to "a US federal agency operating under FedRAMP Moderate." Findings that touch controlled information cannot leave the enclave; the FDE routes them to `ai-platform-engineer` through a controlled channel (vendor-internal only, no cross-fleet pattern dashboard that mentions the agency). <!-- needs-research: confirm NIST SP 800-61 is at Rev 2 and whether Rev 3 is current at time of publication; also confirm which OMB memos are currently in force for federal AI use -->

**Archetype C — EU retail bank.** Customer-support reply + case triage; GDPR + DORA + EU AI Act posture; Azure AI Foundry under EU Data Boundary. Field-to-product posture: GDPR Article 32 + bank data-classification applied to every artefact that leaves the pilot; DORA ICT-incident-related findings route through the bank's major-incident process before external communication; EU AI Act Article 72 post-market monitoring record is itself a field artefact the FDE contributes to monthly. The "customer success story" has a bank-approved anonymised version; the detailed technical version remains internal. <!-- needs-research: confirm EU AI Act Article 72 post-market monitoring reporting shape and cadence for the specific deployer obligations in the bank's use category -->

Each chapter runs decisions through these three archetypes.

## Where mod-111 sits with the rest of the track

The field-to-product loop only makes sense when the pilot is deep enough to produce patterns worth generalising.

- **[Mod-101](../mod-101-fde-discipline-and-position/)** — the deferral contract names which level-40 peer owns which surface; mod-111 routes the platform-request packets through that contract.
- **[Mod-102](../mod-102-customer-discovery-and-pilot-scoping/)** — the success criteria the sponsor committed to at scoping are the metrics the customer-success story in Chapter 03 attaches to.
- **[Mod-103](../mod-103-enterprise-integration/)** — integration patterns (SSO, SCIM, NHI inventory, egress) are prime platform-request candidates: the FDE discovers at one pilot, the two-customer rule triggers at the next, the packet lands with `senior-agentic-ai-engineer` or `ai-platform-engineer`.
- **[Mod-104](../mod-104-llm-app-in-customer-envs/)** — the per-customer ADRs from Chapter 02 are the raw material for monthly pattern writeups in this module; the gateway's shape gets refined as platform requests.
- **[Mod-105](../mod-105-grounding-on-customer-data/)** — retriever permission patterns and ingest-pipeline gaps feed Chapter 04's platform-request packets.
- **[Mod-106](../mod-106-agentic-workflows-on-enterprise/)** — tool-call patterns, agent-SDK gaps, human-in-the-loop checkpoint primitives are canonical `senior-agentic-ai-engineer` platform-request territory.
- **[Mod-107](../mod-107-eval-eng-on-customer-data/)** — judge-library gaps, gold-set-management primitives, offline-eval-harness features route to `ai-platform-engineer` or `senior-agentic-ai-engineer` depending on surface.
- **[Mod-108](../mod-108-enterprise-security-and-compliance/)** — every security-review answer the FDE writes is a potential entry in the Chapter 06 security-review-answer library; recurring customer asks become platform-request packets for policy inventory, trust-centre tooling, or audit-stream primitives.
- **[Mod-109](../mod-109-deployment-rollout-change-management/)** — feature-flag, canary, versioning, and handoff primitives all surface platform requests when the FDE finds themselves re-building the same scaffolding at three pilots.
- **[Mod-110](../mod-110-observability-and-oncall/)** — the regression-fixture back-feed from [Chapter 06](../mod-110-observability-and-oncall/06-blameless-postmortems-and-eval-backfeed.md) is the input to mod-111's field-telemetry capture; the generalisability tag (`pilot_specific` / `pattern_potentially_shared` / `platform_level`) is the first-pass triage Chapter 02 refines.
- **Mod-112 (consulting craft and communication)** — how the quarterly product-review meeting is actually run as an executive-level conversation; how the pattern writeup is narrated to leadership; how the FDE handles a "no" on a platform-request packet.

## Anti-patterns

**The silent escalation.** The FDE sees the same pattern at the third customer, feels the frustration mount, and vents in private — to the manager, to a peer, in a slack DM. The platform team never hears the pattern; the fourth pilot hits the same gap. Chapter 04's packet is the counter: a well-formed request lands in writing, with evidence, with the interim workaround visible so the core team knows what the FDE is already carrying.

**The one-customer platform ask.** The FDE submits a platform-request packet after one customer complained once. The core team rejects; the FDE's reputation takes a hit; the next *real* platform gap waits behind the recovery. Chapter 02's two-customer rule is the counter.

**The gold-plated monthly writeup.** The FDE spends three days authoring a beautifully-formatted twelve-page pattern writeup; the PM skims the first paragraph; the pattern lands with the same impact a one-pager would have had. Chapter 03's template is one page expanded to two — not twelve.

**The success story that was never a success.** Marketing wants a customer success story; the FDE writes one; the metrics it leans on are cherry-picked from the first fortnight before the honeymoon effect wore off. The customer sees it, is unhappy, pulls the quote. Chapter 03 insists customer success stories cite metrics the customer has signed off on *after* a full quarterly review cycle.

**The knowledge base no one opens.** The FDE team's wiki has 400 entries, no tagging, no review cadence, no ownership; new FDEs ask humans because searching the wiki is slower than asking. Chapter 06 writes ownership, review, and the "would a new FDE find this?" test.

**The platform request without the workaround.** The FDE packages a platform-request packet that says "we need this." The core team asks "what are you doing *in the meantime*?" The FDE has no answer; the request is downgraded. Chapter 04 insists every packet names the workaround currently in flight.

**The roadmap demand.** The FDE walks into the product-review meeting with a list of asks and a tone that reads as demand. The PM responds defensively; the relationship becomes transactional; the FDE's roadmap influence shrinks. Chapter 05 insists the FDE *proposes*, does not demand — bringing data and customer voice, letting the PM own the decision.

**The field-to-product monologue.** The FDE writes writeups, packages requests, attends the review, then is surprised that the roadmap rarely reflects them. The FDE never asked the PM what *they* are optimising for; the FDE's proposals land in the wrong column of the PM's prioritisation. Chapter 05 writes the pre-work the FDE does *before* the meeting — reading the PM's current priorities — so the proposals route into the frame the PM already has.

**The customer artefact that leaks.** A pattern writeup names the customer, cites specific numbers from the pilot, and is shared in a public-facing vendor forum. The customer's legal team calls. The vendor pulls the artefact; the relationship takes months to recover. Chapter 03's sign-off path exists to prevent this; every archetype has a specific route.

## Summary

- Mod-111 opens three pipes: field telemetry triage (bug / feature / platform gap), design-partner artefacts (pattern writeups, pain-point taxonomies, one-pagers, customer success stories), and the internal knowledge base that compounds the FDE team's leverage.
- The FDE is the vendor product team's field eye. The two-customer rule promotes a workaround to a platform-request packet. The triage default is *bug* until a second pilot appears.
- Five dimensions govern the module: signal completeness, triage accuracy, artefact readiness, platform-request credibility, knowledge-base compounding. Triage accuracy and artefact readiness are gates; the rest are optimisations.
- Four level-40 peers appear: `ai-engineering-manager` (roadmap framing, Chapter 05), `senior-agentic-ai-engineer` (shared-infra platform requests, Chapter 04), `ai-platform-engineer` (peer-level, cross-fleet patterns), and `ai-enablement-lead` (peer-level, customer-education content).
- Cadences: continuous capture, weekly triage, monthly pattern writeup, quarterly one-pager + product-review, per-outcome customer success story, as-needed platform-request packet, quarterly knowledge-base review.
- Three archetypes thread through every chapter — the HIPAA payer with customer-legal sign-off on every external artefact, the FedRAMP agency with no external artefacts by default, the EU bank with GDPR + DORA + EU AI Act Article 72 constraints on every pipe.
- Nine anti-patterns to catch early: silent escalation, one-customer platform ask, gold-plated writeup, premature success story, unread knowledge base, workaround-less platform request, roadmap demand, field-to-product monologue, leaking customer artefact.
- Chapter 02 is next: field telemetry capture + the bug / feature / platform-gap triage model in full.
