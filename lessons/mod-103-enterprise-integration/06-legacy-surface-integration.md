# Chapter 06 — Integrating with the Customer's Legacy Surface

## Why this chapter exists

The customer's modernised APIs — the REST surface, the event bus, the SaaS platform integrations from Chapters 04 and 05 — are often the *facade*. Behind that facade sits SOAP, SFTP, CICS screens, nightly batch jobs, MQ queues the customer's integrator stood up in 2004. Nothing about an LLM pilot magically brings those surfaces forward. If the data the pilot needs is in the mainframe, the pilot talks to the mainframe. The FDE's craft here is twofold: knowing the protocol families well enough to engineer against them, and quantifying the latency / reliability / cost cost of each one against the LLM feature's budget so the pilot-scope document can either absorb the cost or scope it out.

The four protocol families this chapter covers:

- SOAP / WS-* (the XML-document-exchange family)
- SFTP + batch files (the file-drop pattern, still used by SaaS and legacy alike)
- Mainframe gateways (CICS / IMS over MQ, Connect:Direct, 3270 screen access)
- Screen-scraping / UI automation (last-resort patterns)

Each gets a section with the protocol shape, the realistic latency / reliability / cost profile, and when it is the right choice for an LLM pilot.

## SOAP and WS-*

### The protocol

SOAP is still in production at scale at large enterprises — in Salesforce (as an alternative surface), in Workday (as the primary surface for most modules), in insurance claims systems, in many government integrations. The authoritative references:

- [SOAP 1.2 Part 1: Messaging Framework](https://www.w3.org/TR/soap12-part1/)
- [WSDL 1.1](https://www.w3.org/TR/wsdl) and [WSDL 2.0](https://www.w3.org/TR/wsdl20/)
- [WS-Security](https://docs.oasis-open.org/wss-m/wss/v1.1.1/os/wss-SOAPMessageSecurity-v1.1.1-os.html) for message-level security
- [WS-Trust](https://docs.oasis-open.org/ws-sx/ws-trust/v1.4/ws-trust.html) for security-token exchange (the WS-* ancestor of RFC 8693)

A SOAP exchange looks like (schematic):

```xml
POST /services/CaseService HTTP/1.1
Host: legacy.customer.example
Content-Type: application/soap+xml;charset=UTF-8
SOAPAction: "http://customer.example/CaseService/GetCase"

<soapenv:Envelope xmlns:soapenv="http://www.w3.org/2003/05/soap-envelope"
                  xmlns:case="http://customer.example/case">
  <soapenv:Header>
    <wsse:Security xmlns:wsse="http://docs.oasis-open.org/wss/...">
      <wsse:UsernameToken>
        <wsse:Username>pilot</wsse:Username>
        <wsse:Password Type="...PasswordDigest">...</wsse:Password>
      </wsse:UsernameToken>
    </wsse:Security>
  </soapenv:Header>
  <soapenv:Body>
    <case:GetCase>
      <case:caseId>12345</case:caseId>
    </case:GetCase>
  </soapenv:Body>
</soapenv:Envelope>
```

### What the FDE does

- Get the WSDL. A WSDL is both the schema (XSD-based) and the endpoint descriptor. Treat it like an OpenAPI 3.1 spec from Chapter 04: generate a client, bind to the schema, validate responses at the boundary.
- Pick the client library the ecosystem has standardised on. Python: [zeep](https://docs.python-zeep.org/) is the modern choice. Java: JAX-WS; most enterprises are standardised on the JDK's built-in stack or Apache CXF. .NET: WCF or the modern [CoreWCF](https://github.com/CoreWCF/CoreWCF) for .NET 6+. Node: [strong-soap](https://github.com/loopbackio/strong-soap) or [easy-soap-request](https://github.com/CaleBower/easy-soap-request).
- Fix WS-Security early. SOAP authentication is often UsernameToken (over TLS) or WS-Security with X.509 certificate-based signing. Agree with the customer's SOAP admin which one.
- Handle the error shape. SOAP faults have a `faultcode` / `faultstring` structure (SOAP 1.2 uses `Code` / `Reason`). Translate them to the pilot's internal error taxonomy.
- Pay the latency cost. SOAP is XML; the parse + serialize cost is non-trivial. For a pilot that is budget-constrained on latency, SOAP typically adds 50-200ms per call over JSON on the same transport.

### LRC profile

| Property | Reality |
|---|---|
| Latency | Modestly higher than REST/JSON (XML parse + schema binding) |
| Reliability | Same as the underlying HTTP transport; SOAP itself is stateless |
| Cost | Engineering-hour cost higher than REST — generator + schema binding + error translation |

### When to pick SOAP

The customer only exposes SOAP. That is nearly always the reason. Do not stand up a REST adapter in front of someone else's SOAP for a pilot; use SOAP directly and defer the adapter decision to the customer's platform team.

### WS-Trust and the long tail

Older WS-* integrations (identity federation via WS-Federation, token exchange via WS-Trust) still exist in regulated verticals — insurance, defence, large financial. The FDE recognises the pattern (XML tokens exchanged for other XML tokens) and treats it as the SOAP-era equivalent of the OIDC/OAuth story from Chapters 02/03. Where the customer's identity stack is fully WS-* (ADFS with WS-Federation), see Chapter 02's ADFS discussion and coordinate on whether the pilot bridges to a modern IdP for its own sign-in.

## SFTP and batch file exchange

### Where SFTP still lives

SFTP is older than REST, more durable than any API fashion. It is the default for:

- Healthcare EDI (X12 837 claims, 834 enrolment, 835 remittance) and HL7 v2 batch between payers / providers.
- Financial services settlement / reconciliation feeds.
- B2B supplier / vendor exchanges — EDI 850 purchase order, EDI 810 invoice.
- Any "data drop" between systems that do not share a cloud.

The LLM pilot touches SFTP whenever the customer's upstream feed is a nightly file or whenever the pilot must hand data to a downstream system that only accepts SFTP (back-office reconciliation, compliance filing).

### Protocol baseline

[SFTP](https://datatracker.ietf.org/doc/html/draft-ietf-secsh-filexfer-02) runs over SSH (RFC 4251 family). Authenticate with SSH keys or (where legacy demands) username/password. Keep the key in the customer's secret store (Chapter 03).

### The file-exchange patterns

Two shapes:

- **Push**: the customer's system drops files into an SFTP directory on a server the pilot owns or has read access to. The pilot's watcher picks up new files and processes.
- **Pull**: the pilot's job polls an SFTP directory owned by the customer's upstream system on a schedule.

Watchers are preferable to pollers where supported; polling 30 seconds adds latency, 5 minutes is the common compromise.

### Format considerations

- **CSV** is the lowest-common-denominator; expect it. Parse defensively (quoted fields, escaped delimiters, encoding); use a robust library (Python: `csv` module with `dict=True`; Java: `OpenCSV`; nothing custom).
- **XML / X12 / HL7 v2** — healthcare and B2B feeds. Pick a well-known parser (Python: `pyX12`; Java: `Mirth`'s HL7 parsing).
- **Fixed-width** — legacy mainframe exports. Typically accompanied by a copybook that describes the fields. The FDE parses by position.
- **Encrypted archives (PGP / GPG)** — healthcare and financial feeds often ship encrypted on top of SFTP. The pilot decrypts with a key the customer rotates; key management is Chapter 03's secret store.

### LRC profile

| Property | Reality |
|---|---|
| Latency | Minutes to hours — the schedule is the latency. Cannot be used for per-request LLM feature paths. |
| Reliability | High — SFTP is simple and durable; the mode of failure is the file being late or corrupt, not the protocol. |
| Cost | Low engineering cost per feed; non-trivial ops cost if there are many feeds. |

### When to pick SFTP

The data arrives on a schedule the LLM pilot can tolerate (daily, hourly), and the customer's upstream does not expose anything faster. Also: the LLM pilot writes a periodic output (a weekly summary, a compliance report) that must land on an SFTP endpoint the customer's downstream system consumes.

### Hidden requirements

- **File naming conventions.** The customer's downstream parser matches filenames by pattern. Changing a file name is a silent breakage.
- **Manifest / control files.** Many legacy feeds include a sidecar `.ok` or `.done` file after the data file is fully written. The pilot should not process the data file until the control file arrives.
- **Retention.** Files sit on the SFTP server for a defined period; deleting too early breaks re-delivery; keeping too long breaks compliance. Agree on the policy.

## Mainframe gateways

### Where mainframes are

Mainframes still back the core-of-core for most banks, large insurers, airlines, and major government agencies. The surfaces the FDE sees:

- **IBM MQ** ([IBM MQ docs](https://www.ibm.com/docs/en/ibm-mq)) — enterprise messaging; durable, transactional; the typical bridge between mainframe applications and distributed systems.
- **CICS / IMS Transaction Gateway** — invoking a mainframe transaction from a distributed caller. IBM's [CICS Transaction Server](https://www.ibm.com/products/cics-transaction-server) and [IMS Enterprise Suite](https://www.ibm.com/products/ims/enterprise-suite) are the two major transaction monitors; gateways like [IBM MQ](https://www.ibm.com/docs/en/ibm-mq) + the [CICS Transaction Gateway](https://www.ibm.com/products/cics-transaction-gateway) bridge from TCP/HTTP into them.
- **z/OS Connect** ([IBM z/OS Connect](https://www.ibm.com/products/zos-connect)) — modern REST facade over mainframe transactions; the preferred path when the customer has stood it up.
- **Connect:Direct / NDM** ([IBM Connect:Direct](https://www.ibm.com/products/connect-direct)) — guaranteed-delivery file transfer, still the standard for high-value bulk between large institutions.
- **3270 screen scraping** — covered below.

### What the FDE does

If the customer already runs z/OS Connect, use it; it is a REST surface and Chapter 04 applies.

If the customer runs MQ as the integration point: the pilot subscribes to inbound queues for events and writes to outbound queues for requests. Transactional semantics apply (persistent messages, queue depth, poison-message handling). Libraries: IBM MQ client libraries exist for every major language; the client is officially supported.

If the customer runs CICS Transaction Gateway: the pilot sends a COMMAREA (fixed structure) via the gateway and receives a response COMMAREA. The copybook drives the structure.

If the customer runs Connect:Direct for bulk: the pilot is on the receiving / sending node of a NDM session; set up by the customer's mainframe-interop team.

### LRC profile

| Property | Reality |
|---|---|
| Latency | Depends entirely on the transaction. CICS transactions: 50 ms – several seconds. MQ round-trip: 10s of ms. NDM batch: hours. |
| Reliability | Very high — mainframes are the single most reliable surface in the stack. |
| Cost | Significant engineering-hour cost on first setup; often requires a mainframe-interop specialist on the customer side; cost per transaction (MIPS) can be material. |

### When to pick a mainframe path

The data is on the mainframe and there is no faster surface. Modernisation projects to pull data out of mainframes are long and expensive; the LLM pilot does not try to drive mainframe modernisation.

### Specific risks

- **MIPS cost.** Some customers charge each mainframe transaction against a MIPS budget. A pilot that triggers mainframe transactions for every LLM feature invocation can become material to cost. Get the per-transaction cost from the customer's mainframe-cost owner and do the math.
- **Change windows.** Mainframe deployments happen in change windows (often one per month). The pilot's work against mainframe interfaces cannot assume mid-month change availability; co-ordinate on windows early.
- **EBCDIC encoding.** Mainframe strings are EBCDIC; the pilot's code assumes UTF-8. The bridge layer translates; the pilot's client library should hide this but sometimes does not for edge data (special characters in names / addresses).

## Screen-scraping and UI automation

### When it is unavoidable

The pilot needs data from a system that has no API at all — only a human-facing UI (3270 terminal emulator, legacy web UI, thick Windows client). The data owner cannot or will not expose an API in the pilot's timeline. Robotic process automation (RPA) and 3270 screen-scraping are the fallback.

Three tool families the FDE recognises (the customer's platform team typically has a preference, driven by prior adoption):

- **3270 screen-scrape.** Programmatic terminal clients (IBM Host On-Demand, open-source `tn3270` libraries) read fields from named screen positions.
- **Web UI automation.** [Playwright](https://playwright.dev/) is the open-source default; Selenium exists for older compatibility; commercial options (UiPath, Automation Anywhere, Blue Prism) wrap these in an RPA platform.
- **Thick-client automation.** [WinAppDriver](https://github.com/Microsoft/WinAppDriver) or commercial RPA for Windows desktop applications.

### Why it is a bad habit

- **Brittle.** The UI changes; the scraper breaks. The UI does not come with a stable contract.
- **No audit trail from the data owner.** The audit log on the source system shows a human user, not the pilot — because the pilot is impersonating a human user. That is often a compliance problem.
- **Rate-limited by the UI.** A 3270 session supports one transaction at a time; a browser automation is limited by the browser.
- **Credential-handling risk.** The pilot has to supply human-user credentials to the UI, which is a credential-sharing pattern that most security teams rightly refuse.

### When it is the right choice anyway

- The pilot is a 6-week proof that ends with an API ask. Screen-scraping is the fastest path to prove value; the pilot ends by the customer's platform team scheduling an API for the next quarter.
- The data owner is a vendor system the customer cannot change (a third-party legacy clinical tool, a vertical-specific legacy COBOL app). Screen-scrape is the only path and the customer accepts the trade-offs.

### LRC profile

| Property | Reality |
|---|---|
| Latency | Seconds per transaction, often humanlike (UI elements take time to render). |
| Reliability | Low — expect weekly breakage during the pilot; rising operational cost. |
| Cost | Material — engineering hours on initial build, ongoing maintenance, and sometimes RPA platform licensing. |

### If you do it

- Pin the browser or terminal version aggressively; upgrades cause failures.
- Capture screenshots on every exception for forensics.
- Treat it as a stopgap in the integration design; name the API ask as the exit plan; put a quarterly review on it.

## Documenting the trade-off against the LLM-feature latency budget

The pilot-scope document (mod-102 Chapter 04) committed to an end-to-end latency budget — say, 10 seconds p95 from case-open to draft-ready, with a 3-second model budget. Any legacy surface the pilot calls consumes part of the remaining 7 seconds.

A concrete table from a real-shape integration design:

```
Latency budget — draft-ready path (10s p95 total)
  Model call                 3.0 s
  Retrieval                  1.0 s  (mod-105)
  Member profile REST        0.3 s  (Chapter 04 — REST)
  Formulary lookup           1.5 s  (Chapter 06 — SOAP legacy)
  Policy rule engine         0.5 s  (Chapter 06 — MQ round-trip)
  Compose + trace overhead   0.2 s
  ----
  Subtotal                   6.5 s
  Headroom                   3.5 s
```

That is a latency budget that fits. Change the formulary surface to a CICS transaction at p95 = 3 s and the budget blows out — the integration design should either (a) cache the formulary rule set and serve from an in-VPC mirror, or (b) narrow scope to cases the pilot can handle without a per-request formulary call, or (c) renegotiate the latency SLO with the sponsor.

The point is: the trade-off is explicit. The integration design does not say "SOAP is slow"; it says "the formulary SOAP call is 1.5 s p95 against our 7 s integration-layer budget."

## Reliability budget

An LLM pilot committed to 99.0% draft-generation availability has an error budget of ~1% of calls. If three legacy surfaces each contribute 0.3% error rate, the integration layer has already exhausted the budget before the model is called. Chapter 01's reliability budget concept matters most here:

- For each legacy surface, estimate (or measure) its contribution to error rate.
- Decide whether the pilot degrades gracefully when the surface is unavailable — serving a reduced-quality draft, queueing the case for later, or failing the feature cleanly.
- Write the degradation behaviour into the integration design; it will be referenced in the runbook (mod-110).

## Cost budget

The per-task cost guardrail from the pilot scope (say, < $0.15 / task) absorbs all surfaces' per-call costs. Legacy surfaces are often cheap (SFTP, SOAP) but can be material (mainframe MIPS). The FDE sums up:

- Model cost (token in + out).
- Retrieval cost (vector DB queries, index storage).
- Per-call cost of each integration surface used in the task.

A pilot that lights up 5 mainframe transactions per case and the MIPS cost is $0.08 / transaction has just budgeted $0.40 / task — well over the guardrail. Catch that at integration-design time; it is nearly impossible to fix after the pilot ships.

## Common failure modes

**Treating "we can screen-scrape" as a strategy, not a stopgap.** Screen-scraping is expensive to maintain; it degrades with every UI change; it does not pass a serious security review. If the integration design does not name the API-ask exit criteria, the pilot ships with a growing operational burden and no path to reduce it.

**Not budgeting the MIPS cost.** The pilot works in staging. Production MIPS cost is 10x the pilot's expected rate. The cost guardrail is blown within a month. Ask for the MIPS cost at design time.

**Missing the file-delivery handshake.** The pilot processes a file that was still being written. The process ingests an incomplete file and produces incorrect outputs. The fix is a control-file / manifest handshake — standard practice since the 1970s, often forgotten in a modern-first pilot design.

**Pretending the legacy surface does not exist.** The team commits to a real-time LLM feature with a 3-second latency budget. The data is on the mainframe behind a batch feed. The pilot is fundamentally miss-scoped. Catch this in discovery (mod-102 Chapter 03); do not catch it in integration design.

## Summary

- Legacy surfaces are still the system of record for the data most enterprise LLM pilots want. The FDE does not change the customer's legacy; the FDE engineers against it.
- Four families: SOAP / WS-* (XML message exchange with WSDL, WS-Security); SFTP + batch files (scheduled, high-reliability, slow); mainframe gateways (MQ, CICS, IMS, z/OS Connect, Connect:Direct); screen-scraping / UI automation (last resort, stopgap-only).
- Each family has a predictable latency / reliability / cost profile. The integration design writes those numbers down against the pilot's end-to-end budget from the scope document.
- SOAP is live at many Fortune 500s (Workday, insurance claims, government); use it when it is the customer's actual surface. SFTP is standard for healthcare EDI, financial settlement, and B2B exchange.
- Mainframe interop usually goes through MQ, CICS/IMS gateways, z/OS Connect (where available), or Connect:Direct. Budget MIPS; co-ordinate on change windows; handle EBCDIC in the bridge.
- Screen-scraping is a stopgap for data owners with no API. Name it a stopgap in the integration design, with an API-ask exit plan.
- The explicit trade-off table — model + retrieval + each integration surface against the SLO — is the artifact the integration design has to produce. A budget that does not fit is a scope decision for the sponsor, not an engineering problem to code around.
