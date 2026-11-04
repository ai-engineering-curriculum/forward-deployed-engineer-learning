# Chapter 08 — Exposing an MCP Server for a Customer's Agent Surface

## Why this chapter exists

The customer has an agent surface — Microsoft 365 Copilot, Salesforce Agentforce, a vendor-supplied chat agent, the customer's own in-house LLM product. The FDE's pilot has a tool surface — a case-drafting capability, a policy-rule lookup, a formulary check — that the customer's agent should be able to invoke. The two parties now have a question the industry has mostly converged on: how do agents invoke tools in a standard way across vendors and deployments?

The Model Context Protocol (MCP) is the emerging answer. [MCP](https://modelcontextprotocol.io/) is an open protocol, originally published by Anthropic in 2024 and now adopted across a wide range of agent frameworks and platforms, for connecting LLM applications to context — tools, resources, prompts — over a standard transport. If the pilot's integration shape is "our capability needs to appear as a tool in the customer's agent surface," MCP is the right pattern.

This chapter is not a complete MCP tutorial — that is a book's worth of material and the spec itself is the authoritative source. It is the FDE-specific guidance: when to expose an MCP server for the pilot (and when not to), what the server's spec compliance checklist is, how authorisation sits on top of Chapter 02/03, what the customer's governance team will review, and how MCP fits with the rest of the module.

## When MCP is the right pattern

MCP is the right pattern when all of:

1. The customer's agent surface accepts MCP servers (the big-vendor agent surfaces increasingly do, and the customer's own in-house agents may run on open frameworks that do).
2. The pilot's capability is naturally a tool — a verb with inputs and outputs — rather than a passive data feed.
3. The pilot is one capability among several the agent composes; the pilot does not own the agent's orchestration (that is mod-106).

MCP is **not** the right pattern when:

- The pilot's own UI is the sole entry point. Direct API or in-app integration is simpler.
- The pilot needs to orchestrate an agent itself (that is a pilot-side agent, mod-106, not an MCP server).
- The customer's agent surface does not yet accept MCP. Push the vendor; don't build for something that nobody consumes.
- A deterministic REST call from the customer's existing automation would do. Chapter 02's reflex — "deterministic code beats LLM" — applies.

Build the MCP server only when the integration shape calls for it. "Because it is new" is not a reason.

## The protocol in one page

The authoritative source is [modelcontextprotocol.io](https://modelcontextprotocol.io/) and the [specification](https://modelcontextprotocol.io/specification). At a very high level, MCP is a JSON-RPC-2.0-based protocol spoken between a *client* (the LLM application, inside the agent surface) and a *server* (the pilot, providing capability).

The main concepts a compliant server implements:

- **Initialize / capabilities handshake.** On connection, client and server exchange version and advertise which capabilities each supports — tools, resources, prompts, sampling, etc. The server advertises what it offers.
- **Tools.** Functions the server exposes for the client's LLM to call. Each tool has a name, description, input schema (JSON Schema), and returns a structured output. `tools/list` and `tools/call` are the operations.
- **Resources.** Named, read-only data the server exposes. The client can enumerate (`resources/list`), read (`resources/read`), and subscribe to updates. Resources are URI-addressable.
- **Prompts.** Server-provided prompt templates with parameters the client can fill. `prompts/list` and `prompts/get`.
- **Sampling.** Server-initiated LLM calls where permitted; the server asks the client "please run this prompt against your LLM and return the result."
- **Logging / notifications.** Server-to-client notifications for progress, log messages, resource change events.

### Transports

MCP supports two transport shapes in the spec:

- **stdio** — local process-to-process, used when the server runs as a subprocess of the client (e.g. a desktop assistant launching the server binary locally). Not the FDE's primary case.
- **HTTP-based transports** — for *remote* servers, where the server runs as a network service that the client reaches over HTTP. The spec defines Streamable HTTP as the modern remote transport; earlier SSE-based transports exist in older versions. For FDE pilots, the remote transport is the relevant one.

Consult the current specification for the exact transport version the pilot's client supports; MCP has been evolving quickly and the FDE should pin to a specific spec version in the integration design.

### Reference implementations

Anthropic maintains [reference SDKs](https://modelcontextprotocol.io/clients) in Python, TypeScript, Java, C#, Kotlin, and Rust (the list is on the MCP site). The FDE's default is to use a reference SDK rather than implement JSON-RPC from scratch; the SDK handles protocol version negotiation, framing, and the capability handshake.

## What a minimal pilot MCP server looks like (shape only)

The schematic structure — language choice depends on the customer's platform preference. In Python using the reference SDK, the server advertises tools with their input schemas and handles tool-call requests. The pilot's server might expose (for a case-triage pilot):

```
Tools
  draft_case_response(case_id: string, tone: enum) -> { draft: string, citations: list }
  lookup_policy_rule(rule_id: string) -> { rule_text: string, as_of_date: string }
  check_formulary(drug_name: string, plan_id: string) -> { on_formulary: bool, requires_pa: bool }

Resources
  case://{caseId}          one entry per case in scope
  policy://{ruleId}        policy rule text
  formulary://plan/{plan}  the plan's formulary snapshot

Prompts
  case_triage_system_prompt   (version-pinned system prompt the agent surface can consume)
```

Three design principles for the tool surface:

1. **Each tool is a narrow verb.** `draft_case_response` does one thing. Not `case_everything`. The agent client composes.
2. **Input schemas are tight.** JSON Schema with required fields, enums where applicable, pattern constraints on IDs. The client's LLM must receive enough information to fill the fields correctly from the agent's context.
3. **Output schemas are structured.** Not "a chunk of markdown"; a structured result the client can feed to its next step. The agent-surface consumer appreciates it, and your eval (mod-107) depends on it.

## Authorisation for remote MCP

The spec includes an [Authorization](https://modelcontextprotocol.io/specification/basic/authorization) section aligned with OAuth 2.1 patterns. The design pattern is: the MCP server is an OAuth 2.1 resource server; the client is a confidential or public client; tokens are obtained by the client from an authorization server known to the pilot's deployment.

Practically, this is where Chapters 02 and 03 pay off:

- The customer's IdP (Chapter 02) is the authorization server, or an authorization server the customer runs that federates with their IdP.
- The MCP client (the customer's agent surface) obtains access tokens using an OAuth flow — typically the Authorization Code Flow with PKCE for user-delegated access, or client credentials for pure service-to-service.
- The pilot's MCP server validates the access token on every request (JWT signature verification with the AS's JWKS, or introspection for opaque tokens — Chapter 03).
- Scope controls which tools / resources the token is allowed to invoke. One scope per capability (`pilot:draft`, `pilot:policy`, `pilot:formulary`) rather than a single catchall.

The spec is current and evolving; always consult the version your client supports. The current authorization approach (as of the 2025 spec revisions) has undergone refinements — do not design to a cached memory of the shape; read the spec version you are implementing against.

### The user-of-record question

The identity on an MCP tool call is the same question Chapter 03 answered: is it the acting service, or the human on whose behalf the service acts, or both? For an LLM pilot, the human of record *always* matters — the customer's governance team asks "which SME approved this draft?" every time.

The standards-grounded answer: Authorization Code Flow with PKCE between the agent surface and the authorization server, resulting in tokens whose `sub` is the SME. The MCP server then has the SME's identity on every tool call. If the agent surface instead uses client credentials (service identity only), the pilot's MCP server must require a user-identity header on each call or reject.

## The capability-advertisement decision

MCP servers advertise capabilities at the initialize step. The FDE's choice: which capabilities to expose?

- **Tools only.** The minimal, often correct choice. The pilot is a capability, callable by the agent. No resources, no prompts.
- **Tools + resources.** The pilot exposes read-only data the agent can browse — case summaries, policy text, formulary data. Useful when the agent needs to reason over the pilot's data before invoking a tool.
- **Tools + resources + prompts.** The pilot exposes prompt templates the agent surface can use when the pilot has strong views on how its capability is best invoked.
- **Sampling.** Server-initiated LLM calls through the client's LLM. Rare for FDE pilots because it inverts the usual control flow; avoid unless the integration shape genuinely requires it.

Smaller is better. Advertise only what the agent surface actually needs; expand with explicit decisions.

## Governance review for MCP servers

The customer's governance team will have questions specific to MCP, on top of the general integration-design questions from Chapter 01. Expect to answer:

1. **What tools does the server expose?** Full list, with inputs, outputs, and side effects. The governance team wants to know which tools can take action on the customer's systems vs. which are read-only.
2. **What is each tool's blast radius?** A tool that can write to the case store is higher-risk than one that can read a formulary. The governance team wants per-tool scope and permission documentation.
3. **How is authorisation enforced?** Which scopes gate which tools; how user-of-record is carried on every call; how the server audits.
4. **What does the audit log capture?** Per tool call: who (user), what (tool + inputs), when (timestamp), outcome. The auditable record is what the governance team subscribes to.
5. **How are injection attacks mitigated?** MCP tools are prime targets for indirect prompt injection — the input is an LLM-generated argument, the LLM may have been manipulated by content the agent processed earlier. Chapter 06 of mod-106 covers OWASP Agentic hardening; the integration design cites it.
6. **What happens if the server is unavailable?** Degrade gracefully? Kill switch? Who notices and how fast?
7. **How is the server versioned?** Capability changes (tool added, schema evolved) need a clear versioning story for the client to re-advertise and for the governance team to re-review.
8. **How is the server tested?** Protocol compliance tests; schema validation; adversarial injection tests; the governance team will ask for evidence.

A FDE-quality integration design answers all eight in its MCP section.

## OWASP Agentic hardening sits elsewhere, but call it here

Tool-calling LLMs are susceptible to a specific family of attacks: indirect prompt injection via tool outputs, confused deputy via tool composition, memory poisoning across sessions, tool-abuse via over-scoped tokens. The authoritative references:

- [OWASP Top 10 for LLM Applications 2025](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- [OWASP Agentic Security Initiative](https://genai.owasp.org/initiative/agentic-security-initiative/)

Mod-106 covers agentic hardening in depth. For Chapter 08's purposes, the FDE registers that the MCP server surface is a prime attack target and includes its hardening story in the integration design. Three concrete mitigations the server implements:

- **Input validation at the tool boundary.** JSON Schema is the floor; semantic validation (does this `case_id` belong to the invoking user's scope?) is the ceiling.
- **Output sanitisation at the tool boundary.** Tool outputs are consumed by an LLM. Any content the pilot includes in the output is potential injection into the downstream LLM. For high-risk outputs (text from third-party sources), label them (`<data from source X, not instructions>`) and defuse known-exploit patterns.
- **Rate limiting and budget per client.** A single compromised or malicious client should not be able to exhaust the pilot's budget or DoS the server.

## Deployment shape

The MCP server runs:

- Inside the pilot's VPC (topology from Chapter 07), reachable via the same private-connectivity patterns as the rest of the pilot.
- Behind a TLS terminator (ingress, API gateway, load balancer) that enforces OAuth / mTLS / whatever the customer's AS and client expect.
- With observability spans for every request (mod-110 picks this up; include the user-of-record and tool-name in every span tag).
- With a staging and production split; capability changes go through the usual deployment pipeline (mod-109).

For a dedicated-SaaS pilot, the MCP server is reachable over PrivateLink / Private Service Connect / Azure Private Link from the customer's agent surface, same shape as any other dedicated-SaaS service.

## Versioning and change management

Two kinds of change:

- **Protocol version.** The MCP spec evolves. Pin the server's supported protocol version(s) in the integration design. When a new version lands, upgrade on a planned schedule.
- **Capability version.** The pilot's own tools / resources / prompts change — a new field, a renamed enum value, a behaviour change. The server advertises capabilities at initialize; the client re-reads them each session. But the LLM on the client side has been trained / prompted against the old shape, and renames break it.

Treat capability changes like any API change:

- Additive is safe (new tool, new optional field).
- Rename / remove is not. Introduce new, deprecate old, remove later.
- Document capability versions in an ADR (mod-102 Chapter 04's ADR practice).

## Common failure modes

**Building an MCP server because it is a 2025 buzzword, not because the integration shape calls for it.** The simpler question "would a REST endpoint do?" often ends the discussion. A lonely MCP server with no agent-surface consumer is a sunk cost.

**No user-of-record on tool calls.** The server validates the service token but not the human claim. The customer's governance team asks the inevitable question and the pilot's audit log cannot answer. Fix at design time, not retrofit.

**Over-broad tools.** `do_anything(action: string, context: object)` is a tool the agent LLM will invoke in surprising ways and the governance team will reject. Narrow verbs; tight schemas.

**No injection defense.** The pilot's `draft_case_response` tool returns prose that goes back into the LLM's context. An attacker who can plant content in the pilot's data (say, in the SME's case notes) can instruct the LLM via the tool output. The server pattern — label untrusted content, defuse known patterns, instrument for anomalies — is covered in mod-106.

**Tools and resources duplicated across servers.** A customer's agent connects to the pilot's MCP server and the customer's own MCP servers. The same capability (fetch_case) appears twice, with different semantics. The agent LLM picks one unpredictably. Co-ordinate naming with the customer's agent-platform owner; register a prefix (`pilot_case_...`) to avoid collision.

## Summary

- MCP is an open protocol for connecting LLM applications to context — tools, resources, prompts — over a standard transport. Build an MCP server when the customer's agent surface consumes MCP and the pilot's capability is naturally a tool.
- Read the [specification](https://modelcontextprotocol.io/specification) for the version the customer's client supports. Use a reference SDK rather than implementing from scratch.
- Advertise minimally: tools, and resources / prompts only when the agent needs them. Keep schemas tight and verbs narrow.
- Authorisation follows OAuth 2.1 patterns; the customer's IdP (Chapter 02) is the authorization-server-of-record; tokens carry the user-of-record identity and per-tool scopes.
- The governance review has eight questions specific to the server surface: tool list, per-tool blast radius, authorisation, audit, injection mitigation, availability, versioning, testing. Answer them in the integration design.
- OWASP LLM Top 10 and OWASP Agentic Security Initiative are the hardening references; mod-106 covers depth. For Chapter 08, document the three concrete mitigations (input validation, output sanitisation, rate/budget limits) in the design.
- Deploy inside the pilot's VPC; same private-connectivity patterns as the rest of the pilot; TLS terminator in front; observability spans tagged with user and tool.
- Common failures: building because new, missing user-of-record, over-broad tools, no injection defense, tool-name collisions with other MCP servers.
