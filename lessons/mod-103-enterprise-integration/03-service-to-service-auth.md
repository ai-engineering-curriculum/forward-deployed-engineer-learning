# Chapter 03 — Service-to-Service Auth Against the Customer's Service Mesh

## Why this chapter exists

Chapter 02 left the end-user authenticated in the pilot's browser session. This chapter picks up the identity on the next hop and traces it through every service-to-service call the pilot makes — from the pilot backend to the customer's data surface (Chapter 04), to the SaaS platform APIs (Chapter 05), to the model gateway (mod-104). Along the way the identity may change shape: a user identity for calls that act on behalf of the signed-in user, a service identity for calls that act for the pilot itself, or a combined identity ("the pilot, acting on behalf of Mariana") for calls that need both.

The customer's platform team has already made four choices before the pilot arrived: whether they run a service mesh (Istio, Linkerd, Consul, AWS App Mesh), whether that mesh enforces mTLS by default, whether they run an API gateway at the trust boundary (Kong, Apigee, AWS API Gateway, Azure APIM), and which identity broker issues service tokens (the IdP from Chapter 02, a dedicated STS, or a cloud-provider workload identity). The FDE does not change those choices. The FDE picks the service-to-service auth pattern that fits what the customer already runs.

This chapter writes the four patterns the FDE picks from — OAuth 2.0 client credentials, mTLS, token exchange (RFC 8693), and short-lived service tokens (STS / Workload Identity / Pod Identity) — against the two most common customer postures (service mesh with mTLS by default; API gateway at the trust boundary).

## Four patterns, when each fits

| Pattern | When it fits | Trade-off notes |
|---|---|---|
| OAuth 2.0 client credentials (RFC 6749 §4.4) | The pilot calls a REST / GraphQL API owned by the customer's platform team, no user identity required on the call | Simplest to set up; token lifetime vs. refresh cost; no user-of-record on audit log unless combined with token exchange |
| mTLS | The pilot runs inside the customer's service mesh (Istio / Linkerd / Consul / App Mesh) with mesh-enforced mTLS | Identity is the SPIFFE ID of the pod; no app-layer code needed; mesh config lives with the customer's platform team |
| Token exchange (RFC 8693) | The pilot acts on behalf of a user, downstream API wants the user-of-record claim, but the pilot's own identity must be auditable | Gives you the `act` + `sub` structure for impersonation; requires an STS that implements RFC 8693 |
| Short-lived service tokens from STS / Workload Identity | The pilot calls a cloud-provider service (S3, BigQuery, Snowflake, Azure Storage); no long-lived credentials; identity derived from the pod's workload identity | Zero long-lived secret; identity rotation is the cloud provider's problem; requires the pilot to run in a cluster the provider trusts |

In most pilots the FDE picks more than one. A real integration design might have mTLS between pilot services inside the customer's mesh, OAuth client credentials from the pilot to the customer's data API, and workload identity from the pilot to the model gateway on the hyperscaler.

## OAuth 2.0 client credentials

### The protocol

Defined in [RFC 6749 §4.4](https://datatracker.ietf.org/doc/html/rfc6749#section-4.4). The pilot (as a confidential client) authenticates to a token endpoint and receives an access token tied to its own identity — not any user's.

```
POST /oauth2/token HTTP/1.1
Host: auth.customer.example
Content-Type: application/x-www-form-urlencoded
Authorization: Basic <base64(client_id:client_secret)>

grant_type=client_credentials&scope=cases:read+cases:write
```

Response:

```json
{
  "access_token": "<JWT or opaque>",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "cases:read cases:write"
}
```

The access token is attached to downstream API calls as `Authorization: Bearer <token>`. If it is a JWT, the resource server validates signature + claims (same checklist as Chapter 02); if opaque, the resource server calls [token introspection (RFC 7662)](https://datatracker.ietf.org/doc/html/rfc7662) or relies on the issuer's validation endpoint.

### Client authentication methods

The pilot authenticates to the token endpoint with one of:

- **`client_secret_basic`** — HTTP Basic auth with `client_id:client_secret`. Simplest, acceptable over TLS, but the secret is a long-lived shared secret.
- **`client_secret_post`** — same, but secret in the body. Avoid; `client_secret_basic` is preferred by the spec.
- **`private_key_jwt`** — the pilot signs a JWT with a private key it alone holds (a `client_assertion`), and the authorization server validates with the pilot's registered public key. Defined in [RFC 7523](https://datatracker.ietf.org/doc/html/rfc7523). No long-lived shared secret; rotation is key rotation, which is cleaner.
- **`tls_client_auth` / `self_signed_tls_client_auth`** — [RFC 8705](https://datatracker.ietf.org/doc/html/rfc8705) mTLS client authentication. The pilot's TLS client certificate is the credential. Fits naturally if the pilot's service mesh already issues certs.

The FDE's default for new pilots is `private_key_jwt` where the AS supports it; for mesh-resident pilots, `tls_client_auth`. Long-lived `client_secret` is acceptable only when the customer's AS does not support anything else.

### Scope design

Client credentials tokens carry `scope`. The FDE's design discipline:

- One scope per logical capability (`cases:read`, `cases:write`, `members:read` — not `pilot:everything`).
- The pilot's client registration requests the union of scopes it may ever need.
- Each downstream call asks for a token with the minimum scope needed, if the AS supports per-request scope narrowing.

Least-privilege at the scope level is the baseline customer security teams expect. The audit-log question the customer will ask is: "for this token, which scopes did it carry when it was issued, and which scopes were used in the last 24 hours?" — design for that question.

### What is wrong with client credentials for user-facing calls

Client credentials carries no user identity. If the pilot is answering a case for Mariana and writes back to the customer's case store, the write appears from "pilot service account" in the audit log. The customer's case-store audit review will not be able to attribute that write to Mariana without a side-channel (the pilot's own log). That is sometimes acceptable; where it is not, use token exchange (below).

## mTLS inside the customer's service mesh

### The protocol

Mutual TLS — both sides present and validate certificates — is the default identity mechanism in modern service meshes. The authoritative reference for the cross-mesh identity model is [SPIFFE (Secure Production Identity Framework For Everyone)](https://spiffe.io/) and its implementation, [SPIRE](https://spiffe.io/docs/latest/spire-about/). A SPIFFE ID looks like:

```
spiffe://customer-mesh.example/ns/pilot/sa/pilot-worker
```

Services get SPIFFE IDs via their service mesh's SPIFFE-aware identity issuer:

- **Istio** — uses SPIFFE internally; see [Istio security — identity](https://istio.io/latest/docs/concepts/security/#istio-identity).
- **Linkerd** — uses mTLS with cert-manager-style identity issuance; see [Linkerd — automatic mTLS](https://linkerd.io/2/features/automatic-mtls/).
- **HashiCorp Consul** — Connect uses SPIFFE; see [Consul service mesh — mTLS](https://developer.hashicorp.com/consul/docs/connect).
- **AWS App Mesh / Google Anthos / Azure Service Mesh** — hyperscaler-managed mesh variants, all mTLS-enabled.

### What this means for the pilot

If the pilot's services are mesh-resident, the mesh does mTLS for them. The pilot's HTTP code can be identity-free at the application layer; identity is the SPIFFE ID of the sidecar. Authorisation is enforced by the mesh's authorisation policy (Istio's `AuthorizationPolicy`, Linkerd's `Server` + `ServerAuthorization`, Consul's intentions). The FDE writes those policies with the customer's mesh admin.

The practical shape:

- Enrol the pilot's services into the mesh (customer platform team's job; FDE coordinates).
- Request a SPIFFE ID / Kubernetes ServiceAccount binding for each pilot service (one per workload kind — e.g. `pilot-web`, `pilot-worker`, `pilot-retriever`).
- Write mesh authorisation policy: only `pilot-web` can call the customer's case API; only `pilot-worker` can write to the customer's event bus; and so on.
- Confirm the mesh's CA pins and rotation schedule — pilot deployment manifests pin the mesh's trust domain, not individual certs.

### Trade-offs

- **Pro:** no application-layer secret. Identity rotates at the mesh's cadence (often hourly). The customer's mesh admin already owns the policy surface.
- **Con:** the pilot inherits the mesh's blast radius. If the mesh CA has a bad day, every service is non-operational. Also, mesh-enforced mTLS only extends to services inside the mesh; calls that leave the mesh (to a SaaS platform, to a model provider) need their own posture (OAuth client credentials or service tokens).

### mTLS outside a mesh

Customers without a mesh may still want mTLS at a specific boundary — a legacy middleware interface, a regulated-vertical data broker. The pattern is the same (both sides present certs) but the pilot owns its client cert lifecycle. Key points:

- Certificates must be short-lived (24h – 7 days typical); long-lived client certs are an operational liability.
- Rotation runs through [ACME (RFC 8555)](https://datatracker.ietf.org/doc/html/rfc8555) against an internal CA, or an issuance workflow via the customer's PKI team.
- The pilot never prompts for a cert at startup; it loads from a mounted secret store (SPIRE, HashiCorp Vault, cloud-provider KMS-backed secret store).

## Token exchange — RFC 8693

### The problem it solves

The pilot is authenticated to its token endpoint as itself. Mariana is signed in to the pilot through SAML/OIDC (Chapter 02). The pilot needs to call the customer's case API as "the pilot, acting on behalf of Mariana." The customer's case API audit log should show *both*: the pilot (so the customer knows which system wrote) and Mariana (so the customer knows which human approved).

Pure client credentials loses the user identity. Pure user-token forwarding (sending Mariana's bearer token downstream) makes the downstream API trust the pilot more than it should and loses the pilot's identity.

Token exchange — [RFC 8693](https://datatracker.ietf.org/doc/html/rfc8693) — gives the pattern. The pilot presents both tokens to an STS (its own credential, and the user's ID token or access token) and receives a new access token whose `sub` is the user and whose `act` claim is the pilot.

### The flow

```
POST /token HTTP/1.1
Host: auth.customer.example
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:token-exchange
subject_token=<the-user's-access-or-id-token>
subject_token_type=urn:ietf:params:oauth:token-type:access_token
actor_token=<the-pilot's-own-access-token>
actor_token_type=urn:ietf:params:oauth:token-type:access_token
requested_token_type=urn:ietf:params:oauth:token-type:access_token
audience=https://cases.customer.example
scope=cases:write
```

Response (JWT payload of the issued token):

```json
{
  "iss": "https://auth.customer.example",
  "sub": "mariana.diaz@customer.example",
  "aud": "https://cases.customer.example",
  "exp": 1764857600,
  "scope": "cases:write",
  "act": {
    "sub": "pilot-worker@customer.example"
  }
}
```

The downstream API validates the token, reads `sub` for the human of record, reads `act.sub` for the calling service. The audit log carries both.

### Why token exchange matters for LLM pilots specifically

LLM pilots are a textbook case for the dual-identity audit question. The regulated-vertical customer will ask: "which human approved this action, and which system issued it?" Token exchange is the standards answer. Without it, the customer's security reviewer may require you to implement a side-channel audit trail — twice the work and a bespoke integration they have to inspect.

Not every customer's STS supports RFC 8693. The ones that do: Keycloak, Microsoft Entra ID (with on-behalf-of flow, which is RFC 8693-aligned), Auth0, Okta (via the Advanced Server Access or Workflows feature set), PingFederate. When the customer's STS does not, the practical fallback is: pilot calls with its own client-credentials token, and the pilot's request body carries a signed `user-context` JWT the downstream API validates separately. That is a workaround, not a standard, and the FDE should flag it in the integration design.

### Entra's "on-behalf-of" flow

Microsoft Entra's [On-Behalf-Of flow](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-on-behalf-of-flow) predates RFC 8693 and uses the same `grant_type` URN. If the customer runs Entra, this is the pattern their platform team will expect.

## Short-lived service tokens — STS, Workload Identity, Pod Identity

### The pattern

A shared secret is an operational liability. The modern pattern is: the pilot's workload has an *identity* in the cloud provider or cluster, and it exchanges that identity for a short-lived credential when it needs to call a provider API. No long-lived secret touches the pilot.

The names vary by provider:

- **AWS STS + IAM Roles for Service Accounts (IRSA)** or **Pod Identity** — [EKS IAM roles for service accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) and [EKS Pod Identity](https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html). The pod's Kubernetes ServiceAccount maps to an IAM role; the AWS SDK retrieves short-lived credentials via `AssumeRoleWithWebIdentity`.
- **Google Cloud Workload Identity** — [Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation). The GKE service account maps to a Google service account; the SDK exchanges the OIDC token for a short-lived access token.
- **Azure Workload Identity** — [Azure Workload Identity for AKS](https://learn.microsoft.com/en-us/azure/aks/workload-identity-overview). Kubernetes ServiceAccount maps to an Entra application via OIDC federation.

Each provider's doc is the authoritative source; all three implement the same conceptual pattern (OIDC-federated identity exchanged for cloud-provider credentials).

### For cross-cloud workloads

SPIFFE / SPIRE is the pattern for a pilot that runs in one cluster and calls another cluster's services, including across cloud boundaries. SPIRE Server issues JWT-SVIDs that downstream services validate against SPIRE's known trust domain. See [SPIRE architecture](https://spiffe.io/docs/latest/spire-about/) for the detail.

### Trade-offs

- **Pro:** no long-lived secret. Rotation is the cloud provider's job (typically hourly). The audit log on the provider side is strong.
- **Con:** identity is scoped to one cluster (or one trust domain). Federating identity across clusters is non-trivial; the FDE should not design a federated identity flow as part of a pilot unless the customer's platform team is already operating one.

## Token lifetime and refresh

The FDE's defaults:

- Access tokens: **1 hour** maximum. Short enough that compromise is time-boxed.
- Refresh tokens (user flows): **1 day to 90 days** depending on the compliance posture. For user-bound refresh tokens, rotate on use ([refresh-token rotation](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics#name-recommendations-2)).
- Service-to-service tokens: **5 minutes to 1 hour**; the pilot's client re-requests on expiry. Workload-identity flows typically do this automatically in the SDK.
- mTLS certs: **24 hours to 7 days**; the mesh rotates.

A pilot that holds an hour-long bearer token and does not invalidate it on user logout has a logout bypass window of up to an hour. For most LLM pilots that is tolerable; for a HIPAA workflow handling PHI, the customer's security reviewer may require an out-of-band revocation path (introspection on every call, or a short-lived token re-acquired per call).

## Choosing between patterns: a decision test

Ask three questions:

1. **Does this call require the user-of-record on the audit log?** If yes → token exchange or on-behalf-of. If no → client credentials, mTLS, or workload identity.
2. **Is the call entirely inside the customer's mesh?** If yes → mTLS carries identity; no app-layer work. If no → app-layer auth (OAuth token in the Authorization header, or signed workload-identity credentials).
3. **Is the called service a cloud-provider native service (S3, BigQuery, Snowflake)?** If yes → workload identity is the right pattern; do not create a long-lived access key. If no → OAuth.

A real pilot composes these. For example:

```
Mariana clicks Submit
  pilot-web -> pilot-worker                   # mesh mTLS
  pilot-worker -> customer case API           # OAuth 2.0 token exchange (user + act)
  pilot-worker -> model gateway (mod-104)     # OAuth 2.0 client credentials
  pilot-worker -> S3 (audit-log archive)      # IRSA workload identity
```

Four patterns, four trust boundaries, one integration design.

## Secrets management

Whichever pattern applies, the pilot ends up holding *some* credential material — a private key for `private_key_jwt`, a client certificate, an IRSA role binding. The FDE's secret-store defaults:

- **Inside the customer's cluster**: use the customer's existing secret store. HashiCorp Vault is common; [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/) + External Secrets Operator for EKS; [GCP Secret Manager](https://cloud.google.com/secret-manager/docs); [Azure Key Vault](https://learn.microsoft.com/en-us/azure/key-vault/general/). Do not introduce a new secret store for the pilot alone.
- **Rotation**: the pilot's config reloads secrets on a short cadence (seconds to minutes); pilot services do not restart on secret change.
- **Access**: the pilot's secret-read permission is scoped to the specific secrets it needs; the customer's security reviewer will ask for the IAM policy that permits those reads. Produce the policy as a side-artifact to the integration design.
- **Break-glass**: there is exactly one documented path by which a human can read a pilot secret (for debugging), and every use of that path is logged.

## Common failure modes

**Hard-coded client secret in a container image.** Still a weekly finding in enterprise code scans. The fix is any of the patterns above; the easiest is switching to `private_key_jwt` or workload identity. Do not "fix" it by moving the secret to an environment variable loaded from a config map.

**Missing token revocation.** A compromised bearer token is valid until it expires. For low-stakes pilots the short token lifetime is adequate; for HIPAA / FedRAMP-shape pilots the customer will require an explicit revocation path. [OAuth 2.0 Token Revocation (RFC 7009)](https://datatracker.ietf.org/doc/html/rfc7009) plus introspection on every downstream call is the standards answer.

**Implicit trust of untrusted TLS CA.** The pilot's HTTP client trusts the system CA bundle by default. If the customer runs an internal CA for mTLS, the pilot must pin the internal CA — not add it to the system trust store globally. Pinning scopes trust to the mesh's own domain.

**Audit log that records service identity only.** The downstream audit log reads "pilot-worker@customer.example" for every call. The regulated-vertical review asks "which human approved?" The answer is a side-table in the pilot's own log — which the auditor has to correlate by request-ID across two systems. Token exchange solves this; design for it from the start.

**Treating the client credential as an eternally-rotating item.** The pilot is on-call during the pilot period, including when the customer's auth team rotates certificates at 3am. The pilot needs automatic reload (cert-manager + file watcher; SDK auto-refresh). Manually-reloaded credentials are an outage waiting to be scheduled.

## Summary

- Four service-to-service auth patterns: OAuth 2.0 client credentials, mTLS inside the mesh, RFC 8693 token exchange for on-behalf-of, and short-lived service tokens from STS / Workload Identity / Pod Identity.
- A real pilot composes several of these — mesh mTLS inside, OAuth-with-token-exchange crossing into customer APIs, workload-identity for cloud-provider services.
- Prefer `private_key_jwt` or mTLS client authentication to shared `client_secret`; prefer workload identity to long-lived access keys. Both are standards-grounded and both lower the operational secret-management burden.
- Token exchange (RFC 8693, or Entra's on-behalf-of flow) is the only standards-compliant way to carry both the acting service's identity and the human-of-record identity through the call. LLM pilots in regulated verticals almost always need it.
- The customer's platform team already made four choices: service mesh? mTLS default? API gateway at the boundary? which STS? The FDE picks auth patterns that fit those choices, not patterns that force them to change.
- Secrets live in the customer's existing secret store (Vault, Secrets Manager, Key Vault), scoped by IAM, rotated automatically, read via SDK with reload-on-change.
- Three recurring failures: hard-coded secret, missing revocation path, service-identity-only audit log. Each has a standards-grounded fix; use it.
