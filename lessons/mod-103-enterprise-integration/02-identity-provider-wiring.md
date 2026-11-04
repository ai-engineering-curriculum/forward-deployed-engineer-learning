# Chapter 02 — Wiring the Pilot into the Customer's Identity Provider

## Why this chapter exists

Every end-user in the pilot is already a user in the customer's identity provider. The pilot does not stand up its own login screen, its own password reset, its own group model. It wires into whatever the customer has — in most enterprises that is one of four platforms (Okta, Microsoft Entra ID, Ping Identity, Google Workspace); in a non-trivial minority it is one of the long-tail (OneLogin, Auth0, JumpCloud, ADFS). Wiring correctly means three protocol-level jobs: Web-SSO against the IdP so a human can sign in, token exchange so the signed-in identity propagates to downstream service calls, and user / group provisioning so the pilot's permission model stays in sync with the customer's HRIS-driven group changes.

This chapter writes the three protocols the FDE encounters on every pilot — SAML 2.0, OpenID Connect 1.0 with PKCE where applicable, and SCIM 2.0 — against the vendor-specific quirks of the big-4 IdPs and the long-tail. The downstream service-call auth sits in Chapter 03. The ACL propagation into retrieval sits in mod-105. This chapter is concerned with the identity edge.

## The three jobs of identity wiring

The three jobs of the identity edge map cleanly to three standards:

- **Human sign-in** → either SAML 2.0 Web Browser SSO Profile or OIDC 1.0 Authorization Code Flow (with PKCE for public clients). The output is a signed assertion or ID token that identifies the user and their attributes (email, groups, custom claims).
- **Downstream propagation** → OAuth 2.0 (RFC 6749) access tokens issued by the IdP's authorization server or a dedicated STS, with claims about the user carried forward. The protocol here is not SAML; SAML tokens are heavy and SOAP-oriented. OIDC/OAuth is the lingua franca of service-to-service propagation (see Chapter 03).
- **Lifecycle (join/change/leave)** → SCIM 2.0 (RFCs [7643](https://datatracker.ietf.org/doc/html/rfc7643) and [7644](https://datatracker.ietf.org/doc/html/rfc7644)). The IdP pushes user and group changes to the pilot in real-time; the pilot does not reach back to query "does this user still exist?" every request.

The FDE will encounter pilots that do only two of the three (SAML + SCIM, or OIDC alone with per-request lookups), but the three-job framing is the one to design against.

## SAML 2.0 Web Browser SSO Profile

### The protocol in one page

The authoritative source is the OASIS SAML 2.0 specification set, especially the [Core](https://docs.oasis-open.org/security/saml/v2.0/saml-core-2.0-os.pdf), [Bindings](https://docs.oasis-open.org/security/saml/v2.0/saml-bindings-2.0-os.pdf), and [Profiles](https://docs.oasis-open.org/security/saml/v2.0/saml-profiles-2.0-os.pdf) documents. For a readable short reference, OASIS publishes the [SAML Technical Overview](https://wiki.oasis-open.org/security/Saml2TechOverview). The pilot almost always uses the SP-initiated Web Browser SSO Profile with HTTP-POST binding:

```
User -> SP (pilot web UI): request protected resource
SP   -> User browser: 302 redirect with SAMLRequest to IdP SSO endpoint
User browser -> IdP: SAMLRequest (POST or Redirect binding)
IdP  -> User browser: HTML form with signed SAMLResponse, auto-POSTs to SP ACS URL
User browser -> SP (ACS URL): SAMLResponse
SP   -> User browser: set session cookie; redirect to resource
```

Three artifacts matter at the FDE altitude:

- **SP metadata**: the XML document describing the pilot's SP side — entity ID, ACS (Assertion Consumer Service) URL, signing/encryption certificates, NameID format, required attributes. The customer's IdP admin imports this to create the IdP-side application.
- **IdP metadata**: the XML document describing the IdP side — IdP entity ID, SSO endpoint, signing certificate. The pilot imports this to validate inbound assertions.
- **The assertion**: the actual signed XML document in the SAMLResponse that carries the user's identity, authentication method, attribute statements, and conditions (NotBefore / NotOnOrAfter, audience restriction).

### What a real SAML assertion looks like (schematic)

```xml
<saml:Assertion ... IssueInstant="2026-11-04T13:22:11Z">
  <saml:Issuer>https://customer-okta.okta.com/exk...</saml:Issuer>
  <ds:Signature>...</ds:Signature>
  <saml:Subject>
    <saml:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress">
      mariana.diaz@customer.example
    </saml:NameID>
    <saml:SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
      <saml:SubjectConfirmationData NotOnOrAfter="2026-11-04T13:27:11Z"
                                     Recipient="https://pilot.customer.example/saml/acs"
                                     InResponseTo="_abc123..."/>
    </saml:SubjectConfirmation>
  </saml:Subject>
  <saml:Conditions NotBefore="2026-11-04T13:21:41Z" NotOnOrAfter="2026-11-04T13:27:11Z">
    <saml:AudienceRestriction>
      <saml:Audience>https://pilot.customer.example</saml:Audience>
    </saml:AudienceRestriction>
  </saml:Conditions>
  <saml:AuthnStatement AuthnInstant="2026-11-04T13:22:10Z">
    <saml:AuthnContext>
      <saml:AuthnContextClassRef>
        urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport
      </saml:AuthnContextClassRef>
    </saml:AuthnContext>
  </saml:AuthnStatement>
  <saml:AttributeStatement>
    <saml:Attribute Name="email"><saml:AttributeValue>...</saml:AttributeValue></saml:Attribute>
    <saml:Attribute Name="groups">
      <saml:AttributeValue>pilot-reviewers</saml:AttributeValue>
      <saml:AttributeValue>senior-clinical-reviewers</saml:AttributeValue>
    </saml:Attribute>
  </saml:AttributeStatement>
</saml:Assertion>
```

The FDE's validation checklist against an inbound assertion (the [NIST SP 800-63C federation guidance](https://pages.nist.gov/800-63-4/sp800-63c.html) is the right reference for the *why*):

1. XML signature verifies against the IdP's known signing certificate (pinned via metadata, not fetched at request time).
2. `Issuer` matches the configured IdP entity ID.
3. `Audience` matches the SP's entity ID.
4. `Recipient` matches the ACS URL on which the assertion was received.
5. `NotBefore` ≤ now ≤ `NotOnOrAfter` with a small clock skew tolerance (≤ 3 minutes in NIST guidance).
6. `InResponseTo` matches the AuthnRequest ID the SP stored when it initiated, and that ID has not been replayed in the SP's used-request cache.
7. The assertion has not been replayed (store `ID` in a short-TTL cache).
8. `SubjectConfirmation` method is `bearer` (and the SP is on HTTPS; the bearer pattern depends on TLS for transport protection).

Failing any of these is a hard reject. A SAML stack that silently accepts an unsigned assertion is the Golden SAML attack class; see the [CISA alert on SolarWinds / Golden SAML](https://www.cisa.gov/news-events/cybersecurity-advisories/aa21-008a) for the reference incident.

### Library choices

The FDE rarely implements the XML-signature verification from scratch. Use a well-known library:

- Python: [`python3-saml`](https://github.com/SAML-Toolkits/python3-saml) (OneLogin toolkit, OASIS-compliant).
- Node: [`@node-saml/node-saml`](https://github.com/node-saml/node-saml), or framework wrappers like [`passport-saml`](https://github.com/node-saml/passport-saml).
- Java: [Spring Security SAML](https://docs.spring.io/spring-security/reference/servlet/saml2/index.html).
- Go: [`crewjam/saml`](https://github.com/crewjam/saml).

Regardless of library, the FDE keeps the four pieces that are environment-specific out of code: SP entity ID, ACS URL, IdP metadata (URL or inline XML), and the SP signing certificate + private key. These live in the secret store (Chapter 03's secrets discussion) and are rotated against the customer's certificate-rotation policy.

## OpenID Connect 1.0

### Why OIDC when SAML exists

SAML is heavy (XML, XML-DSig, often SOAP-flavoured), browser-only in practice, and painful to use for service-to-service token propagation. OpenID Connect is a thin identity layer on OAuth 2.0 that gives:

- An ID token (JWT) that is parseable with a standard JWT library.
- Discovery via a well-known endpoint (`/.well-known/openid-configuration`).
- JWKS (JSON Web Key Set) for key rotation at the IdP without client reconfiguration.
- The same tokens flow from the browser login all the way through service-to-service calls (Chapter 03 picks this up).

The authoritative source is the [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html) specification; the [Discovery 1.0](https://openid.net/specs/openid-connect-discovery-1_0.html) and [Dynamic Client Registration](https://openid.net/specs/openid-connect-registration-1_0.html) specs are the two most-referenced satellites. The RFC that underpins OIDC is [OAuth 2.0 (RFC 6749)](https://datatracker.ietf.org/doc/html/rfc6749); the token structure is [JWT (RFC 7519)](https://datatracker.ietf.org/doc/html/rfc7519).

### Authorization Code Flow, with PKCE for public clients

For a server-side pilot web UI (confidential client), the Authorization Code Flow with a client secret is the standard. For any public client — a SPA, a mobile app, a desktop tool — PKCE ([RFC 7636](https://datatracker.ietf.org/doc/html/rfc7636)) is mandatory. The [OAuth 2.0 Security Best Current Practice](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics) recommends PKCE for *all* clients (including confidential ones) at this point — the FDE should treat PKCE as default.

```
Client -> Authorization endpoint:
  GET /authorize?response_type=code
                 &client_id=pilot-web
                 &redirect_uri=https://pilot.customer.example/oidc/callback
                 &scope=openid+profile+email+groups
                 &state=<csrf-token>
                 &nonce=<replay-token>
                 &code_challenge=<sha256(code_verifier) base64url>
                 &code_challenge_method=S256

User authenticates; IdP redirects back:
  GET https://pilot.customer.example/oidc/callback?code=<code>&state=<csrf-token>

Client -> Token endpoint:
  POST /token
    grant_type=authorization_code
    code=<code>
    redirect_uri=https://pilot.customer.example/oidc/callback
    client_id=pilot-web
    client_secret=<secret>           # for confidential client only
    code_verifier=<verifier>         # proves this is the same client that started the flow

Token endpoint returns:
  {
    "access_token": "<opaque or JWT>",
    "id_token":     "<JWT>",
    "refresh_token":"<opaque>",
    "token_type":   "Bearer",
    "expires_in":   3600
  }
```

### What a real ID token looks like (JWT payload)

```json
{
  "iss":   "https://customer-okta.okta.com",
  "sub":   "00u1a2b3c4d5",
  "aud":   "pilot-web",
  "exp":   1764857600,
  "iat":   1764854000,
  "auth_time": 1764853999,
  "nonce": "<as-sent-in-authorize>",
  "amr":   ["pwd", "mfa"],
  "email": "mariana.diaz@customer.example",
  "email_verified": true,
  "name":  "Mariana Diaz",
  "groups": ["pilot-reviewers", "senior-clinical-reviewers"]
}
```

The ID token validation checklist (from Core §3.1.3.7):

1. Signature verifies against the IdP's JWKS (keys fetched from the discovery document's `jwks_uri` and cached with a TTL).
2. `iss` matches the expected issuer.
3. `aud` contains the client's `client_id`.
4. `exp` ≥ now with a small skew allowance.
5. `iat` present and reasonable.
6. `nonce` matches the one sent in the `/authorize` request.
7. `azp` (if present) is the client's `client_id`.
8. `acr` and `amr` claims satisfy any step-up policy (e.g. MFA required for the pilot).

The access token is used for API calls (Chapter 03); the ID token is for the user-identity hand-off to the pilot's session store. Do **not** use the ID token as an API bearer token; the two serve different purposes.

### OAuth 2.1 and where the IETF is going

OIDC still points at OAuth 2.0 (RFC 6749). The [OAuth 2.1 draft](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1) consolidates the OAuth 2.0 Security BCP: PKCE for all clients, no implicit flow, no password grant, redirect URIs must be exact string matches. The FDE should design to the OAuth 2.1 shape today — all four of those are already best practice.

## SCIM 2.0 for user and group provisioning

### Why SCIM

SAML / OIDC tell you *who* is signing in *right now*. SCIM tells you *who exists* and *what groups they belong to* — independent of a sign-in event. Without SCIM, the pilot has to call back to the IdP on every request to resolve group membership, or trust the group claim in the SAML assertion (which is fine until the SME's group changes mid-session and the pilot's cached state lags).

The authoritative sources are [RFC 7643 — SCIM Core Schema](https://datatracker.ietf.org/doc/html/rfc7643) and [RFC 7644 — SCIM Protocol](https://datatracker.ietf.org/doc/html/rfc7644).

### The SCIM API the pilot exposes

The pilot stands up a `/scim/v2` endpoint, authenticated with a bearer token the customer's IdP will send on every request. The IdP's SCIM client calls:

```
POST /scim/v2/Users
PUT  /scim/v2/Users/{id}
PATCH /scim/v2/Users/{id}
DELETE /scim/v2/Users/{id}
GET  /scim/v2/Users?filter=userName eq "mariana.diaz@customer.example"

POST /scim/v2/Groups
PATCH /scim/v2/Groups/{id}  (add/remove members)
DELETE /scim/v2/Groups/{id}
```

The payload shape is defined in RFC 7643. A `User` looks like:

```json
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
  "userName": "mariana.diaz@customer.example",
  "name": { "givenName": "Mariana", "familyName": "Diaz" },
  "emails": [{ "value": "mariana.diaz@customer.example", "primary": true }],
  "active": true,
  "groups": [{ "value": "<group-id>", "display": "pilot-reviewers" }]
}
```

`active=false` is the deprovisioning signal — the pilot MUST treat a PATCH to `active=false` as "revoke all sessions, strip permissions" within whatever the customer's offboarding SLA is (24 hours is a common HR-ish requirement; for terminated-for-cause the customer will want minutes).

### What the FDE commonly gets wrong at SCIM

- **Not supporting PATCH.** Okta, Entra ID, and OneLogin all prefer PATCH (RFC 7644 §3.5.2) over PUT for incremental group changes. A pilot that supports only PUT forces the IdP to send a full user document for a single membership change.
- **Not honouring `externalId`.** The IdP's internal user key may not match the pilot's; the IdP sends its own key as `externalId`. Index on both.
- **No ETag on `Meta.version`.** Concurrent IdP operations can race; ETag (RFC 7644 §3.14) is the standard way to make SCIM updates compare-and-swap.
- **Silent 200 on errors.** If the pilot cannot persist a user, return a SCIM error structure (RFC 7644 §3.12). An IdP that sees 200 will consider the user provisioned; a later de-provisioning will miss.

## The big-4 IdPs and their vendor-specific quirks

Protocol compliance does not eliminate vendor-specific handling. The pilot's integration with each big-4 IdP has a narrow set of consistent quirks the FDE should know.

### Okta

Okta publishes definitive documentation at [help.okta.com](https://help.okta.com/) and the [Okta developer portal](https://developer.okta.com/). The three documents the FDE opens first:

- [Build an SP-initiated SAML SSO flow](https://developer.okta.com/docs/guides/build-sso-integration/saml2/main/) — SAML app configuration end-to-end.
- [Create an OpenID Connect app](https://developer.okta.com/docs/guides/sign-into-web-app-redirect/-/main/) — OIDC flows with code + PKCE.
- [SCIM protocol and compatibility](https://developer.okta.com/docs/concepts/scim/) — Okta's SCIM client expectations.

Concrete quirks:

- Okta's SAML group attribute is customisable — the IdP admin picks the attribute name (`groups` is conventional but not forced). Agree on the name with the customer's IdP admin in advance; the SP cannot infer.
- Okta's access tokens from the Authorization Server can be JWT or opaque depending on resource server configuration. Default to JWT for inspectability.
- Okta's SCIM client issues provisioning on assignment to the app, not on user creation globally. Users not assigned to the pilot app do not appear.

### Microsoft Entra ID (Azure AD)

The authoritative source is [Microsoft Entra documentation](https://learn.microsoft.com/en-us/entra/identity/). Entra supports SAML 2.0, OpenID Connect, and SCIM 2.0 for enterprise apps. Specifically the FDE opens:

- [Single sign-on SAML protocol](https://learn.microsoft.com/en-us/entra/identity-platform/single-sign-on-saml-protocol).
- [Microsoft identity platform and OpenID Connect protocol](https://learn.microsoft.com/en-us/entra/identity-platform/v2-protocols-oidc).
- [Tutorial: Develop a SCIM endpoint for user provisioning to apps from Microsoft Entra ID](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/use-scim-to-provision-users-and-groups).

Concrete quirks:

- Entra's OIDC multi-tenant apps require explicit `tenantid` handling — `iss` is `https://login.microsoftonline.com/<tenantid>/v2.0`. For a single-tenant pilot, this is one tenant; for an ISV-style pilot touching more than one Entra tenant, it is per-tenant.
- Entra's SAML assertion emits groups as object IDs (GUIDs), not names, unless the app manifest opts in to names. Decide which the pilot consumes and configure the manifest accordingly.
- Entra's SCIM client uses PATCH heavily and expects RFC 7644-compliant PATCH responses.
- Entra supports [Conditional Access](https://learn.microsoft.com/en-us/entra/identity/conditional-access/overview) policies that may require device compliance / MFA before token issuance; the pilot cannot bypass these, and should not attempt to.

### Ping Identity

The Ping product family includes PingFederate (federation server), PingOne (cloud), PingDirectory, and PingAccess. SAML and OIDC are first-class; SCIM 2.0 is supported with PingFederate's provisioning server or PingOne's directory connector. The authoritative docs sit at [docs.pingidentity.com](https://docs.pingidentity.com/).

Concrete quirks:

- PingFederate on-prem is common at large enterprises; metadata exchange is a file exchange rather than URL-based in many deployments.
- Attribute contracts are explicit — the IdP admin must declare the claim names the SP will receive, and the SP must match the contract verbatim.

### Google Workspace

Google Workspace supports SAML for SSO ([Set up your own custom SAML application](https://support.google.com/a/answer/6087519)) and OIDC primarily through Google Identity Platform. SCIM is supported for user provisioning.

Concrete quirks:

- Google's SAML group attribute is not shipped by default — the IdP admin configures custom attribute mapping from directory group membership.
- OIDC with Google uses Google Identity Platform's endpoints (`https://accounts.google.com/.well-known/openid-configuration`); for Workspace-scoped sign-in, use the `hd` (hosted-domain) parameter to constrain.

## The long-tail IdPs

The long-tail still shows up in enterprise pilots. One-page notes:

- **OneLogin.** Full SAML / OIDC / SCIM support, close to the OASIS and RFC specs. The [OneLogin developer docs](https://developers.onelogin.com/) are the reference.
- **Auth0 (now Okta Customer Identity Cloud).** Auth0 is both an IdP and a federation hub. The [Auth0 docs](https://auth0.com/docs) are comprehensive; rules and Actions complicate token issuance in ways the pilot cannot see from outside — coordinate with the customer's Auth0 tenant admin for custom claim mapping.
- **JumpCloud.** Directory-as-a-Service; full SAML / OIDC / SCIM support. The [JumpCloud docs](https://jumpcloud.com/support) are the reference.
- **AD FS (Active Directory Federation Services).** On-prem Microsoft, still common at large regulated enterprises. SAML 2.0 and WS-Federation are first-class; OIDC support arrived with Windows Server 2016 / AD FS 4.0 and is complete in current versions. SCIM is not native — provisioning happens via other channels. Microsoft's [AD FS documentation](https://learn.microsoft.com/en-us/windows-server/identity/active-directory-federation-services) is the reference. Pilots that rely on AD FS often live behind a VPN or ExpressRoute (Chapter 07).

Each long-tail IdP has a quirk or two the vendor documents; always read the vendor docs before writing code against the admin UI.

## The identity-propagation diagram

The single most valuable artifact from this chapter is the identity-propagation diagram — one page that traces a single user action from the browser to the deepest downstream call.

```
Browser (SME: Mariana)
   |  (1) SAML or OIDC authz code flow w/ PKCE
   v
Pilot web UI  (customer VPC)
   |  (2) ID token -> session cookie; store user identity in session
   |      Access token stored for downstream API calls
   v
Pilot backend  (customer VPC)
   |  (3) On-request: validate access token (JWKS cache)
   |      Derive user identity for logs, metrics, traces
   |  (4) For downstream call -> OAuth 2.0 token exchange (Chapter 03) or
   |      service token w/ user-on-behalf-of claim
   v
Customer data surface (Chapter 04/05)  or  model gateway (mod-104)
   |
   v
Audit log  (observable in both the pilot and the customer's SIEM)
```

The diagram is the artifact Chapter 09's mod-110 trace-span attribution depends on. It is also the artifact the customer's security reviewer asks for in the security packet (mod-108).

Three things the diagram must make explicit:

1. Whose identity appears on every outbound call (user-identity propagated vs. service-identity-with-user-claim vs. pure service identity for jobs).
2. Which trust boundaries the identity crosses (VPC → customer SaaS, VPC → model provider, VPC → data surface).
3. What the audit log of record is at each hop, and who owns it.

If the diagram shows pure service identity on the downstream call (no user propagation), the pilot cannot satisfy a "who approved this LLM output?" audit question. The fix is OAuth 2.0 token exchange with an `act` claim identifying the pilot and a `sub` claim identifying the human — covered in Chapter 03.

## SCIM-integrated group model vs. claim-only

At pilot scale (a handful of SMEs, one group) the claim-in-token model is enough: the SAML assertion or ID token carries the `groups` attribute, the pilot reads it, authorisation flows from there. SCIM is overkill.

At a dozen SMEs or more, with group membership changing week-to-week, the claim-in-token model develops a latency problem: the SME's group changes in Workday at 10 am, the HR system propagates to Okta at 10:05, the SME's existing session sees the old `groups` claim until next sign-in. Group-based authorisation is now stale by an unbounded amount.

The SCIM-integrated model keeps group membership in the pilot's own store, updated in near-real-time by the IdP's SCIM push. The authorisation decision is against the pilot's own group store; the token is only consulted for the identity proper (`sub`, `email`).

The pilot that needs SCIM has at least one of:

- >10 SMEs with routine group changes, OR
- a compliance requirement that user offboarding complete within N hours of HR termination, OR
- a permission model that uses attributes beyond group membership (manager, department, cost centre).

If none of these applies, claim-in-token is the right choice; Chapter 09's mod-110 observability chapter will still want the diagram.

## Two failure modes the FDE should expect

**Mis-synced clock.** Both SAML and OIDC rely on signed tokens with time-bounded validity. If the pilot's host and the IdP's host are more than 3 minutes apart (NIST 800-63C guidance), tokens will reject intermittently — and the error surface will look like "it worked on Monday, not on Tuesday." The fix is always NTP-synced hosts (the customer's VPC should have one; if not, use the cloud provider's default time service). It is not an application-layer fix.

**Session fixation after group change.** A session cookie stays valid for 8 hours; an SME is promoted at hour 2; the new `groups` do not appear until re-login. The right behaviour: the pilot re-fetches group membership from the SCIM store (if present) or forces a token refresh before any step-up authorisation. The wrong behaviour: trust the session's cached group list for all 8 hours. The pattern to read is [OAuth 2.0 Token Introspection (RFC 7662)](https://datatracker.ietf.org/doc/html/rfc7662) for near-real-time token validity checks when the stakes warrant it.

## Summary

- Identity wiring has three jobs: human sign-in (SAML or OIDC + PKCE), downstream propagation (OAuth 2.0 tokens, carried forward — Chapter 03), lifecycle (SCIM 2.0).
- SAML 2.0 is still the default for enterprise Web-SSO against SP applications. Follow NIST SP 800-63C's validation checklist; the Golden SAML attack class is what you avoid.
- OIDC 1.0 is the preferred shape when the pilot can choose, because the same token flows all the way through service-to-service propagation. Use Authorization Code Flow with PKCE, regardless of client type.
- SCIM 2.0 keeps the pilot's user / group store in sync with the IdP in near-real-time. The pilot needs it once group changes or offboarding timelines make claim-in-token stale.
- The big-4 IdPs (Okta, Entra, Ping, Google Workspace) all speak the three standards; each has a small set of vendor-specific quirks — group attribute shape, metadata exchange style, PATCH expectations. Read the vendor doc before coding.
- The long-tail (OneLogin, Auth0, JumpCloud, ADFS) still appears. ADFS specifically tends to require network-boundary considerations (Chapter 07).
- The single artifact from this chapter is the identity-propagation diagram — one page that traces a user action from browser to deepest downstream call, used in Chapter 03, mod-105, mod-108, and mod-110.
- Two recurring failure modes: clock skew (NTP) and session fixation across group change (re-fetch, don't trust the cache).
