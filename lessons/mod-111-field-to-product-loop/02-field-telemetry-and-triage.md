# Chapter 02 — Field Telemetry Capture and the Bug / Feature / Platform-Gap Triage

## Why this chapter exists

The FDE on a live pilot absorbs a continuous stream of signal: an incident fires on Tuesday, a customer SME says *"this would be so much easier if…"* on Wednesday, a security-review question arrives on Thursday that doesn't fit the response-library template, the retrieval dashboard shows a drift the FDE routes around with a prompt tweak on Friday, the kickoff for a new sub-team mentions a tool the pilot doesn't integrate with on Monday. Each of those is either a product-visible pattern now, a product-visible pattern once it recurs, or an organisational tax that nobody but the FDE sees.

Nothing in the pilot's production telemetry captures the second three of those. The pilot's SIEM has the incident; the pilot's dashboards have the drift. The *Wednesday wish*, the *Thursday mismatch*, the *Monday mention* live only in the FDE's head unless the FDE's process puts them somewhere.

This chapter has three jobs. Write the **capture discipline** that turns the FDE's continuous stream into a structured log the FDE's future self and the next FDE can read. Write the **triage model** — bug / feature / platform gap — with explicit criteria so the classification is not a feel. Write the **weekly triage ritual** that keeps the log alive and prevents the "silent escalation" anti-pattern from Chapter 01.

Capture completeness is an optimisation dimension (Chapter 01); triage accuracy is a gate.

## What the capture log is and is not

The FDE's capture log is a per-pilot structured note store. Each entry is a *signal* — one observable, named, dated — with enough structured fields that the FDE can later ask "how many of these have we seen?" without re-reading every note. Think Linear tickets, Notion pages, or a flat markdown repo; any of them works. The tool is less important than the shape.

The log **is**:

- A record of *every* event the FDE thought worth noting at the time, regardless of whether it later proves to be a pattern.
- A triage substrate — the entries are the units the weekly ritual classifies.
- A raw material feed for monthly pattern writeups (Chapter 03) and platform-request packets (Chapter 04).
- A searchable artefact — tagged, dated, sortable by customer / pilot / surface.

The log **is not**:

- A substitute for the pilot's SIEM, incident timeline, or post-mortem machinery. Those live in the customer's observability stack; the capture log *references* them rather than duplicating them.
- A production dashboard. Nobody is paging off the capture log; it is an authoring surface, not an operational one.
- A public document. Entries may reference PHI / PII context in ways that would never survive the data contract from [mod-108 Chapter 05](../mod-108-enterprise-security-and-compliance/05-data-contract.md). The log lives in the vendor's internal store, with access scoped to the FDE team; any artefact leaving the log is de-identified per Chapter 03.

## The capture-entry schema

One schema. Every signal lands in this shape, whether it took thirty seconds to capture (a slack message pointed at a wish) or thirty minutes (the writeup after an incident).

```yaml
# signals/2026-11-04-member-assist-bulletlist-regression.md
signal_id: sig-2026-11-04-0012
pilot_id: pilot-acme-member-assist
customer_id: acme-health
date_observed: 2026-11-04
captured_by: <FDE name>
surface:                       # which part of the stack the signal touches
  - prompt-template
  - online-judge
  - canary-gate
source:                        # where the signal came from
  - incident                   # INC-42 in the customer's IR tool
source_refs:
  - "customer-ir-tool://INC-42"
  - "langfuse://trace/4a7f..."
observation: |                 # one or two paragraphs; plain language
  Bundle v4.2.0 promoted at 13:45Z changed the response-format directive
  from a bullet-list to free-form. Online judge pass-rate dropped from
  0.84 to 0.71 within 25 minutes; SME team's post-incident annotation
  is that bullet-list structure is the dimension the judge rubric was
  previously silent on.
sme_quotes:                    # verbatim, with speaker + date
  - speaker: "Customer SME lead"
    date: 2026-11-04
    text: "We've been telling you bullet-list is the format we want."
triage_initial: bug            # see below
triage_confirmed: null         # filled at weekly triage ritual
generalisability: unknown      # pilot_specific | pattern_potentially_shared | platform_level
related_signals:               # back-pointers to other entries
  - sig-2026-10-28-0007
customer_sensitivity: high     # low | normal | high — used by Chapter 03
links:
  - artefact: post-mortem      # becomes INC-42's post-mortem
    url: "customer-confluence://pm/inc-42"
  - artefact: eval-fixture
    url: "eval-repo://fixtures/inc-42-bullet-vs-paragraph"
```

Nine fields plus observation and SME quotes. The schema is intentionally shallow — the FDE should be able to author an entry in under five minutes on a mid-week afternoon. A schema that takes an hour to fill is a schema that doesn't get filled.

### Field-by-field discipline

- **signal_id** is monotonic by date. The FDE's shell command for "new signal" generates one in the right shape; nobody hand-counts.
- **surface** is a vocabulary from the module's own structure — prompt-template, retrieval, embedding, agent-tool, judge, guardrail, canary-gate, feature-flag, telemetry-schema, cost-budget, deployment-mode, security-posture, data-contract, identity-sso, identity-scim, admin-dashboard, sme-training, pager-rotation, status-page, post-mortem-template. The list grows; the authority is the FDE team's shared vocabulary file in the Chapter 06 knowledge-base.
- **source** is the modality — incident, bug-report, sme-wish, security-review-q, sales-call, kickoff-conversation, failed-sow-item, weekly-review, adr, pm-ask, fde-peer-signal.
- **observation** carries the *facts* only. The entry's analysis lands in the triage field; keeping them separate prevents re-reading from re-interpreting.
- **sme_quotes** carries the customer's voice verbatim. Customer voice is the single most persuasive element in a Chapter 03 pattern writeup; it dies if it is paraphrased. The FDE captures as early as possible, with consent, and records speaker + date.
- **generalisability** inherits Chapter 06's taxonomy from mod-110. The initial value is often `unknown`; the weekly ritual refines it.
- **customer_sensitivity** governs the de-identification cost downstream. `high` means the artefact from this signal cannot leave the pilot without the customer's communications office; `low` means the signal is generic enough that the external artefact can be authored without customer sign-off (though the customer still gets a courtesy review).

### Where the log lives — three archetypes

**Archetype A — US healthcare payer.** The vendor's internal Notion (or Confluence) with `pilot-acme-member-assist` restricted to the FDE team. No PHI in the log body; references point to the pilot's HIPAA-content-cleared stores (Langfuse self-hosted, customer's Zendesk, customer's Confluence IR). SME quotes recorded with the SME's knowledge and consent; the customer SME lead has read access to the pilot's section of the log by standing agreement.

**Archetype B — US federal agency.** Dual-log. The pilot-specific details live inside the agency's enclave in the agency's SharePoint / GCC High; the vendor-internal log carries *only* generalisable abstracts with no agency-specific content. Pattern recognition across pilots runs on the vendor-internal abstracts; the enclave log is the source of truth. The FDE routinely writes both.

**Archetype C — EU retail bank.** Vendor's EU-region Confluence with `pilot-bank-support-assist` restricted to the FDE team. GDPR Article 32 scope applied — data minimisation in the log body, pseudonymised identifiers for any ticket-sourced observation, bank's data-classification applied to each entry at the row level. SME quotes recorded with signed consent from the SME per the bank's internal communications policy. Monthly entries contribute to the EU AI Act Article 72 post-market monitoring record.

## The triage model

Three buckets. Clear criteria. The weekly ritual applies them.

### Bug — ship a fix inside the pilot

A signal classifies as *bug* when **all** of the following hold:

1. The pilot is not meeting the committed behaviour (an SLO, an SLA, a success criterion from [mod-102](../mod-102-customer-discovery-and-pilot-scoping/), a customer expectation set during scoping).
2. The fix fits within the pilot's configuration — prompt-template, retrieval filter, judge rubric, guardrail threshold, dashboard panel, feature-flag setting, playbook edit, canary-gate tweak.
3. The fix is deliverable inside the FDE's current sprint without a change to the shared platform's interfaces.
4. If the same shape has been seen at another pilot, it has been seen *only* in a way that would be fixed at *that* pilot in the same way — i.e. no cross-pilot platform change is required even in principle.

Default disposition: FDE implements, PR into the pilot's config repo, eval-gate verifies, canary promotes, post-mortem back-feed if the bug came from an incident. The product team hears about the bug only through the monthly pattern writeup (Chapter 03) if the pattern is worth generalising.

### Feature — surface to product for the next release

A signal classifies as *feature* when **all** of the following hold:

1. The pilot is working as designed — no SLO burn, no success-criterion miss. The customer *could* be better served by a capability that doesn't exist.
2. The capability fits the shape of the vendor's existing product — a PM can prioritise it, engineering can build it, the capability makes sense as a release-note bullet.
3. The capability would benefit this customer and would plausibly benefit others. (The two-customer rule does not formally apply here — the PM decides prioritisation across customers — but a feature request with zero cross-customer value gets deprioritised anyway.)
4. The delivery cadence fits the vendor's release train: weeks to a quarter, not multiple quarters.

Default disposition: FDE packages a feature request (lightweight form, Chapter 04) and submits through the vendor's product-feedback channel. The feature request contains the evidence the PM needs to prioritise; the FDE shows up at the quarterly product-review meeting (Chapter 05) with the prioritisation case if the feature matters.

### Platform gap — surface to platform / senior / manager at level 40

A signal classifies as *platform gap* when **all** of the following hold:

1. The pilot is working *around* something the platform does not do. The workaround costs the FDE time, risk, or both.
2. The workaround would be redundant at another pilot that shares the shape — i.e. the gap is architectural, not incidental.
3. A fix at the platform level fits the shape of the vendor's shared infra (`ai-platform-engineer`), shared agent SDK / judge / guardrail library (`senior-agentic-ai-engineer`), or the FDE team's shared surface (`ai-engineering-manager`).
4. The **two-customer rule** is met: the same shape has been observed at at least two independent pilots — different customer, different industry, different stack — *or* the signal is recorded as `platform_gap_candidate` pending the second observation.

Default disposition: FDE packages a platform-request packet (heavyweight form, Chapter 04) with the two instances of evidence, the proposed interface sketch, the risk / scope size, and the alternative workaround the FDE is shipping in the interim. Routes to the level-40 peer who owns the surface.

### The classification is often two of three at once

A single signal can legitimately be two buckets at once.

- A bundle regression is a *bug* the FDE fixes in the pilot. It may also surface a *feature* request — "the canary gate should require a 24-hour hold at 1 % for prompt-only changes." It may further surface a *platform gap* if the vendor's feature-flag platform doesn't support that hold shape. One signal; three triage outputs; three sets of action items. Chapter 02's capture schema accommodates all three.

The weekly ritual is where the splits happen.

## The weekly triage ritual

A 45-minute meeting. Standing agenda. Standing participants.

- **Participants.** The FDE on the pilot. The customer SME lead (optional; invited quarterly or on demand). A rotating peer FDE from a different pilot (structural blamelessness check; mirrors Chapter 06 of mod-110's post-mortem peer reviewer). The pilot's `ai-engineering-manager` once a month as an observer.
- **Cadence.** Weekly, 45 minutes, same slot.
- **Input.** Every capture entry since the last ritual. Order by date; group by surface.
- **Output.** `triage_confirmed` populated on every entry. New action items opened on each one. Related-signal back-pointers updated.

### Standing agenda

1. **New signals review (20 min).** Walk every entry captured since last week. For each: read the observation aloud (not the paraphrase — the facts as captured); the initial triage proposal from the author; the peer-FDE's read. Confirm or re-triage. If the triage changes from the initial, record why (a one-line `triage_rationale` field, appended).
2. **Re-triage queue (10 min).** Entries previously triaged `bug` that have recurred; entries previously triaged `pattern_potentially_shared` that have now been seen at a second pilot (check the peer FDE's log); entries previously parked as `platform_gap_candidate` with the two-customer threshold now met.
3. **Action-item dispatch (10 min).** For every entry with a confirmed triage: who is doing what by when? Bugs go to the FDE's sprint; features go to the Chapter 04 feature-request queue; platform gaps go to the Chapter 04 packet queue or stay parked at `platform_gap_candidate`.
4. **Cadence check (5 min).** Monthly pattern-writeup is drafted the second ritual of the month; the quarterly one-pager is drafted in the third month's second ritual. Confirm who is drafting what.

### Why the peer FDE is standing

The peer FDE from a different pilot does three things the pilot FDE cannot do unaided. First — the peer catches the shape at the second pilot that promotes a `platform_gap_candidate` to a two-customer packet. Second — the peer catches the "we always do it this way" assumption that reads as a feature request but is actually a bug because *this customer* was promised something different. Third — the peer carries the FDE team's shared vocabulary; `surface` values that the pilot FDE is inventing get normalised to the team's list by the peer who's used that list before.

The pattern mirrors mod-110 Chapter 06's rotating peer reviewer; the structural logic is identical.

## Triage worked examples

Three signals, three triage walkthroughs. Each shows the capture, the initial triage, the ritual's re-triage, and the downstream disposition.

### Example 1 — the Wednesday wish

Capture: *"Customer SME lead said in Wednesday's stand-up: 'I wish the assistant could tell me which policy section it pulled the answer from, not just the title.' — SME is talking about citation specificity."*

Initial triage: `feature`. The pilot is meeting its quality SLO; the customer would be better served by a capability the pilot doesn't have.

Weekly ritual: peer FDE confirms `feature`. The peer notes the second pilot (EU bank) saw a similar ask from the model-risk reviewer two months ago, logged as `sig-2026-09-12-0003`. The ritual revises the generalisability from `pilot_specific` to `pattern_potentially_shared`.

Disposition: Chapter 04 feature request. Evidence: SME quote (this pilot) + model-risk quote (EU bank pilot). Delivered through the vendor's product-feedback channel; the FDE on the member-assist pilot adds the request to the quarterly product-review agenda (Chapter 05).

### Example 2 — the Thursday security-review mismatch

Capture: *"Customer CISO asked: 'What's your approach to detecting prompt injection via indirect retrieval content?' Our response library has an entry for prompt injection but it's agent-tool-call focused. I answered freehand; the answer was accepted but took me 90 minutes to write."*

Initial triage: `bug` — the response library is incomplete for this pilot.

Weekly ritual: peer FDE flags that *they also wrote a freehand answer* to the same question last quarter at their pilot; their answer was 45 minutes to write because they adapted the CISO's framing. Two instances; the response-library gap is pattern-wide.

Re-triage: `bug` (fix the response library at this pilot) + `platform gap` (the vendor's shared response-library — Chapter 06's security-review-answer library — needs a canonical entry for indirect-retrieval prompt injection).

Disposition: FDE at this pilot adds a pilot-specific entry; a Chapter 04 platform-request packet is drafted (lightweight — no interface sketch needed, just a content ask) to the FDE team's knowledge-base owner (`ai-enablement-lead` could also take this, see Chapter 06). The packet names both pilot instances as evidence.

### Example 3 — the Monday retrieval drift

Capture: *"During retrieval dashboard review, noticed the pilot's reranker is dropping 30 % of the retriever's top-10 for the 'benefit-coverage' intent class. I added an intent-specific rerank weight override. Took 90 minutes. Noted the SDK doesn't expose per-intent reranker config natively; I had to monkey-patch the config loader."*

Initial triage: `bug` (the pilot's reranker was miscalibrated for this intent class — fix shipped) + `platform_gap_candidate` (the shared retrieval SDK should expose per-intent reranker config).

Weekly ritual: peer FDE confirms the bug fix. Peer has not seen the same SDK limitation at their pilot; the `platform_gap_candidate` stays parked. The signal is tagged with `related_signals: []` for now; the weekly ritual revisits in four weeks or when a second pilot sees it.

Disposition: bug fix in flight at this pilot. No Chapter 04 packet yet; the parked candidate will convert to a packet when the two-customer threshold is met, or will be closed as `pilot_specific` if no second pilot sees the shape within a quarter.

## The source-modality discipline

Six modalities produce most capture entries. Each has a discipline that keeps the capture quality high.

- **Incidents.** The post-mortem (mod-110 Chapter 06) is the source; the capture entry *references* it, does not duplicate. Capture happens the same day the post-mortem is signed; the `source_refs` includes the post-mortem URL and the key trace IDs.
- **Customer bug reports.** The weekly bug-report review (mod-110 Chapter 06) is the source; one capture entry per distinct pattern, not per ticket. Multiple tickets of the same shape roll into one signal with a `ticket_refs` list.
- **SME wishes.** Standups, hallway conversations, async notes. Capture happens *the same day*; the SME quote is verbatim; the FDE confirms by paraphrasing back to the SME before the entry is saved ("you said X — am I reading you right?"). The SME's willingness to be quoted is noted in the entry's `customer_sensitivity`.
- **Security-review questions.** Mod-108's response-library discipline is the source. The capture entry fires every time the FDE writes a non-boilerplate answer — i.e. an answer that is not a direct read of the response-library's templated response. The capture entry includes the question verbatim and the time the answer took.
- **Pilot ADRs.** The ADR template from [mod-104 Chapter 02](../mod-104-llm-app-in-customer-envs/02-provider-stack-and-gateway.md) is the source. Every ADR the FDE authors in the pilot is also a capture entry, because the ADR records a *choice* that other FDEs may face. The capture entry's `generalisability` is often `pattern_potentially_shared` because ADR shapes repeat across pilots.
- **Weekly-review frustrations.** Not every frustration from the weekly review surfaces as a bug or feature; some are purely organisational (the FDE's week ran hot because the customer rescheduled three meetings). Capture those too, in a dedicated `workload-signal` modality — they feed the FDE team's utilisation picture `ai-engineering-manager` reviews quarterly.

Six modalities; one schema; one weekly ritual.

## Running the three archetypes

**Archetype A — US healthcare payer.** Capture log in vendor-internal Notion; SME quotes recorded with standing consent from the customer SME lead; HIPAA-content never in the observation body (references to the pilot's Langfuse trace IDs only). Weekly ritual Thursdays 15:00 ET; the customer SME lead attends monthly. Peer FDE is the FDE from a second US healthcare pilot (cross-pilot vocabulary alignment is particularly valuable because HIPAA posture repeats).

**Archetype B — US federal agency.** Dual-log; the enclave log is the source of truth for pilot specifics, the vendor-internal log carries abstracts only. Weekly ritual held inside the enclave for the full detail; the vendor-internal ritual uses abstracts from the enclave ritual's minutes. Peer FDE is cleared for the appropriate controlled-unclassified level; if no peer FDE is cleared, the pilot FDE self-triages with the ISSO as a standing reviewer for the enclave ritual. `ai-engineering-manager` observer attendance requires enclave access. Platform-request packets originating from the agency pilot are abstracted before leaving the enclave — the specific agency workflow is replaced with a generic "policy-analyst drafting" shape before the Chapter 04 packet lands with `senior-agentic-ai-engineer`.

**Archetype C — EU retail bank.** Capture log in vendor's EU-region Confluence with GDPR Article 32 scope applied — data minimisation in observation bodies, pseudonymised identifiers for ticket-sourced signals. Weekly ritual Tuesdays 14:00 CET; the bank's SME lead attends monthly; the bank's model-risk lead attends quarterly to validate `platform_gap` triages against the bank's model-governance expectations. DORA major-ICT-incident-sourced signals get a `dora_major_incident_ref` field; EU AI Act Article 72 post-market monitoring record pulls the month's confirmed-triage summary.

## Anti-patterns

**Capture-as-slack-message.** The FDE notes the Wednesday wish in a #pilot-channel slack message and never promotes it to a capture entry. Six weeks later, in the Chapter 03 writeup push, the FDE tries to reconstruct the quote from scratch. Reconstruction loses the customer voice. The entry is captured the same day; slack is the inbox, not the store.

**Observation-as-analysis.** The FDE writes "the customer is complaining because the prompt is bad and we need to fix it" in the observation field. The analysis is in the field reserved for facts; the triage field is silent. The weekly ritual has nothing to work with. The observation field is facts only — what was said, what was observed, what the trace shows. The analysis is in triage, generalisability, and the weekly-ritual's rationale.

**Triage-by-feel.** The FDE classifies a signal `platform gap` because it *feels* like the shape of a platform gap. The weekly ritual demands the two-customer rule be named; the signal is downgraded to `platform_gap_candidate`. The discipline is there to prevent the one-customer platform ask anti-pattern from Chapter 01.

**Perpetual parking.** A `platform_gap_candidate` signal sits for six months without re-triage. Either it should have converted or it should have been closed as `pilot_specific`. The quarterly knowledge-base review (Chapter 06) sweeps every open candidate; parking beyond two quarters requires an explicit justification.

**Customer voice lost.** The SME quote is paraphrased because the FDE didn't catch it at the time. The downstream pattern writeup reads like the FDE's opinion rather than the customer's. The customer voice is the single most persuasive element of the Chapter 03 artefacts; capturing it verbatim is non-negotiable.

**Log without a peer.** The FDE triages solo; every entry classifies as the FDE saw it; the peer FDE's catch-rate is zero. Six months in, the FDE's biases are the log's biases. The weekly peer FDE is structural; a ritual without one is not a ritual.

**Capture without downstream pointer.** The FDE captures diligently but never promotes signals to Chapter 03 writeups or Chapter 04 packets. The log becomes a graveyard. The weekly ritual's cadence-check item exists for this reason; the knowledge-base review in Chapter 06 catches it at a quarter.

**Signals in the customer's tool.** The FDE writes the capture log entries inside the customer's Confluence or Jira. The customer-sensitive content stays bounded, but the FDE team cannot search across pilots. The capture log is a vendor-internal artefact that *references* customer-side tools; dual-write when needed (Archetype B), never write in the customer's tool alone.

## Summary

- The FDE's capture log is a per-pilot structured note store — vendor-internal, schema'd, searchable — that records every field signal the FDE thought worth noting. Capture completeness is an optimisation dimension.
- The capture schema has nine required fields plus observation and SME quotes. Observation is facts only; analysis lives in triage, generalisability, and the weekly-ritual rationale.
- Three triage buckets: bug (ship a fix inside the pilot), feature (surface to product for the next release), platform gap (surface to level-40 peer after the two-customer rule is met). A single signal can be two of three at once; the schema accommodates all three.
- Weekly triage ritual: 45 minutes, standing agenda (new signals / re-triage queue / action-item dispatch / cadence check), with a rotating peer FDE as the structural blamelessness check.
- Six source modalities: incidents, customer bug reports, SME wishes, security-review questions, pilot ADRs, weekly-review frustrations — each with a capture discipline.
- Three worked examples: Wednesday wish → feature (two pilots → `pattern_potentially_shared`); Thursday security-review mismatch → bug + platform gap (two pilots → packet); Monday retrieval drift → bug + parked candidate (one pilot → `platform_gap_candidate`).
- Archetype A: HIPAA scope on observation body, standing SME consent, cross-pilot HIPAA peer FDE. Archetype B: dual-log with enclave source of truth, cleared peer FDE or ISSO reviewer, abstracts for the vendor-internal packet. Archetype C: GDPR Article 32 scope on log, pseudonymised ticket IDs, DORA / EU AI Act Article 72 integrations.
- Eight anti-patterns: capture-as-slack-message, observation-as-analysis, triage-by-feel, perpetual parking, customer voice lost, log without a peer, capture without downstream pointer, signals in the customer's tool.
- Chapter 03 is next: the design-partner artefacts — pattern writeups, pain-point taxonomies, 'what we learned' one-pagers, customer success stories — that convert the capture log into the shape product leadership consumes.
