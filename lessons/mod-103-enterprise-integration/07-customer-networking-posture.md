# Chapter 07 — Customer-Side Networking Posture

## Why this chapter exists

Chapter 02 wired the identity edge. Chapter 03 wired the auth edge. Chapters 04–06 wired the data edges. None of those edges operates in free space: each is a packet on a wire, crossing a trust boundary that the customer's security team has defined, often years before the pilot started. This chapter is about those boundaries — PrivateLink, VPC peering, Site-to-Site VPN, egress proxying, split tunneling — and how the FDE chooses among them against the customer's security-review checklist.

Networking posture is the *first* thing the customer's security team reads in the integration design and the *last* thing most FDEs want to think about. Done late, it is the reason a pilot slips four weeks before go-live. Done early, it is a two-week set-up that then recedes into the background and does not show up as a blocker.

The three decisions this chapter covers:

1. Where does the pilot run — in the customer's cloud account / VPC, in a dedicated vendor-side cloud account, or on infrastructure the customer owns on-prem?
2. How does the pilot reach the customer's data surfaces — over a private backbone (PrivateLink / VPC peering / VPN / ExpressRoute / Direct Connect) or over the internet with TLS + egress controls?
3. How does the pilot reach the model provider — directly over the internet, through an in-VPC egress proxy, or through a dedicated endpoint the model provider publishes?

Each decision has a canonical set of patterns, each with known trade-offs.

## Where the pilot runs

Four common topologies. The customer's security posture usually dictates one.

### 1. Vendor-managed SaaS, multi-tenant

The pilot runs in the vendor's (FDE's company's) SaaS cloud account, shared across customers. The customer reaches the pilot over the internet (TLS) and the pilot reaches the customer's data over the internet (TLS), with API tokens and OAuth client credentials (Chapters 02 and 03) carrying the identity.

Fits: lower-regulated verticals, early-stage pilots, customers whose own security posture accepts third-party SaaS. Mod-104 Chapter calling the provider landscape defines this more precisely as the base "SaaS" deployment mode.

### 2. Vendor-managed dedicated SaaS

The pilot runs in a dedicated vendor cloud account per customer, isolated from other customers but still vendor-operated. Common for mid-regulated verticals (finserv, healthcare where PHI stays in a dedicated environment).

The posture between the customer and this dedicated environment is what Chapters below cover — PrivateLink from the customer's VPC into the dedicated account is the common pattern.

### 3. Customer-VPC install

The pilot runs inside the customer's cloud account (customer-owned VPC in AWS / GCP / Azure). The vendor (FDE's company) operates a deployment inside the customer's trust boundary. The pilot's data path is entirely intra-VPC — no public internet crossing except for the model provider call (and even that is often routed via PrivateLink or a VPC endpoint).

Fits: high-regulated verticals (HIPAA, FedRAMP-equivalent-posture, EU data-residency), customers whose policy is "data must not leave our trust boundary." The customer's platform team is deeply involved from day 1.

### 4. On-prem install

The pilot runs on infrastructure the customer owns on-premises. Rare for LLM pilots because of the hardware-serving dependency (`ai-infra-inference-engineer` — see Chapter 09), but still real for customers in regulated verticals that do not run meaningful cloud. If the pilot needs to host a model on-prem, defer to `ai-infra-inference-engineer`.

### Decision inputs

The FDE and customer sponsor (mod-102 Chapter 05 commercials) agree on topology based on:

- **Data residency.** Where is the pilot's input data allowed to reside? If "customer's tenancy only," topology 3 or 4.
- **Regulatory posture.** HIPAA / PCI-DSS / FedRAMP / SOC 2 — each raises the floor.
- **Customer's existing operational model.** If the customer runs other cloud workloads in their own AWS / GCP / Azure, topology 3 is natural. If not, topology 2 may be simpler to stand up.
- **Operational ownership during the pilot.** Topology 3 requires the customer's platform team to co-operate on cluster administration; topology 2 is operated end-to-end by the vendor. If the customer's platform team does not have bandwidth, topology 3 is slower to start.

## Private connectivity between the pilot and the customer

The four major patterns, one per hyperscaler-ish pattern:

### AWS PrivateLink / VPC Endpoint Services

[AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/privatelink/) gives a customer VPC a private DNS endpoint that reaches a service in another account (the vendor's or AWS's own) without traversing the public internet. The endpoint is realised as an elastic network interface in the customer's VPC. It is the default private-connect pattern on AWS and the one most customer security teams accept quickly.

Pattern for a dedicated-SaaS pilot: the vendor stands up a VPC endpoint service in their AWS account; the customer creates a VPC endpoint in their VPC pointing at it; the customer's traffic to the pilot stays on AWS's backbone. Supports cross-region and cross-account.

Operationally: the customer's platform team allow-lists the specific endpoint service name; DNS is handled via PrivateLink's automatic DNS records or the customer's private hosted zone.

### VPC Peering

[VPC peering (AWS)](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html) / [VPC Network Peering (GCP)](https://cloud.google.com/vpc/docs/vpc-peering) / [VNet peering (Azure)](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview) connects two VPCs/VNets directly, with route tables routing traffic between them. Non-transitive on AWS.

Suitable when both sides are in the same cloud and both are agreeable to the peering (less common at large customers; peering opens the whole CIDR to the peered side unless controls are added).

### Site-to-Site VPN and ExpressRoute / Direct Connect / Interconnect

[AWS Site-to-Site VPN](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html) / [GCP Cloud VPN](https://cloud.google.com/network-connectivity/docs/vpn) / [Azure VPN Gateway](https://learn.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpngateways) open an IPSec tunnel between a customer's on-prem network and their cloud VPC. [AWS Direct Connect](https://aws.amazon.com/directconnect/) / [Azure ExpressRoute](https://learn.microsoft.com/en-us/azure/expressroute/) / [GCP Cloud Interconnect](https://cloud.google.com/network-connectivity/docs/interconnect) provide a dedicated private circuit between on-prem and cloud (more expensive, more performant, more stable than VPN).

For an LLM pilot: this layer is most relevant when the customer's data surfaces are on-prem (mainframe, legacy, regulated-vertical data). The pilot runs in the customer's cloud VPC; the on-prem data flows through VPN/ExpressRoute into the VPC; the pilot reads from there.

### Cloud-provider native: PrivateLink equivalents for other hyperscalers

- GCP has [Private Service Connect](https://cloud.google.com/vpc/docs/private-service-connect) as the PrivateLink-equivalent.
- Azure has [Private Link](https://learn.microsoft.com/en-us/azure/private-link/private-link-overview).

Each is the default private-publish pattern on its respective cloud; all three accept similar security-review patterns.

### Multi-cloud

If the pilot is on AWS and the customer is on Azure, the private-backbone options narrow to VPN or a cloud-to-cloud private connectivity provider (Megaport, Equinix). For a pilot, this is often the trigger to deploy inside the customer's cloud rather than fighting cross-cloud networking.

## Egress patterns for model-provider calls

The pilot that calls OpenAI / Anthropic / Google Vertex / AWS Bedrock needs an egress path. The patterns:

### 1. Direct over public internet with TLS

Simple; works out of the box; acceptable for lower-regulated pilots. The pilot's egress DNS resolves to the provider's public endpoint; TLS 1.2+ is enforced; provider-specific auth is in headers.

Accepted by most pilots at topology 1 (vendor SaaS). Not accepted by many topology-3 pilots because the egress leaves the customer's VPC.

### 2. Egress proxy inside the customer VPC

An egress proxy (Squid, Envoy, cloud-native: AWS Network Firewall / GCP Cloud NGFW / Azure Firewall) sits in the customer's VPC. Pilot traffic to `api.anthropic.com` / `api.openai.com` routes through the proxy, which enforces:

- **Allow-list by destination DNS name.** Only a defined set of model-provider endpoints and dependencies (telemetry, auth).
- **TLS termination optional.** Some proxies terminate TLS to inspect headers / URLs; most pilots do not for privacy reasons.
- **Logging.** Every outbound call logged for the customer's SOC. This is a common SIEM-ingest requirement for regulated pilots.

The egress proxy is the single most useful pattern for topology-3 deployments. It lets the pilot reach the model provider while keeping the customer's security team in control of what leaves their VPC.

### 3. Provider-published private endpoints

Several model providers publish private endpoints:

- [AWS Bedrock VPC endpoints](https://docs.aws.amazon.com/bedrock/latest/userguide/usingVPC.html) — Bedrock callable over PrivateLink without internet traversal.
- [Azure OpenAI with Private Endpoints](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/use-your-data-securely#disable-public-network-access-1) — Azure OpenAI reachable through Azure Private Link.
- Google Vertex AI — [Private Service Connect for Vertex AI](https://cloud.google.com/vertex-ai/docs/general/vpc-sc) for accessing generative AI endpoints privately.
- [Anthropic Claude on AWS Bedrock](https://aws.amazon.com/bedrock/claude/) is reachable through Bedrock's VPC endpoint.

Picking a provider that publishes a VPC endpoint for the customer's cloud often removes the egress-proxy debate entirely. Mod-104 picks up the provider-selection story; Chapter 07 just observes that private endpoints exist on the three big clouds for the most-used providers.

### 4. Split tunneling

The pilot's VPC has a selective egress — most destinations go through a private path (VPN/PrivateLink), a narrow allow-list goes to the internet directly. This is the pattern when the customer's data path is private but the model-provider path is accepted as internet-direct.

Avoid split tunneling for the customer's data path itself; it is a common data-exfiltration misconfiguration and most customer security teams will reject it.

## DNS, certificates, naming

Three crosscutting concerns that break pilots in surprising ways.

### DNS

PrivateLink / Private Service Connect / Azure Private Link all introduce a private DNS namespace. The pilot's host must resolve `api-pilot.customer.example` to the private IP of the endpoint — not to a public IP. Patterns:

- The customer's DNS owner creates a private hosted zone (Route 53 Private Hosted Zone / Cloud DNS / Azure DNS).
- The pilot's VPC is associated with the private hosted zone.
- The pilot's host's resolver uses the VPC's DNS, which returns the private answer.

Pitfalls:

- A pilot that hard-codes an IP breaks when the endpoint ENI is recreated.
- A pilot that uses the public DNS unwittingly routes through the public internet.
- A pilot that uses its own application-layer DNS library (not the OS resolver) may bypass the VPC's resolver.

### Certificates

Private endpoints still need TLS, and the certificate's subject must match the private hostname. Two patterns:

- Public CA cert with a public DNS name that resolves privately. Clean; works with system trust stores; the customer's security team usually accepts.
- Private CA cert with a private DNS name. The pilot must trust the customer's internal CA. Common in large regulated enterprises; the pilot's trust store must be managed.

The anti-pattern is `InsecureSkipVerify=true` or TLS verification disabled. If you find it in the pilot's code, it is a security-review blocker.

### Naming

Pilot services get consistent naming inside the customer's namespace: `pilot-web.<customer-prefix>.<customer-domain>`. The customer's platform team's naming convention drives this; do not stand up a parallel name scheme.

## Firewall rules and allow-lists

The customer's platform team will ask for a precise list of:

- Destinations the pilot connects to (model provider, SaaS platform APIs, observability endpoints).
- Inbound sources that connect to the pilot (SMEs' browsers, customer's agent surface if MCP, webhook producers).

The FDE produces this list in the integration design, as a table with `CIDR / FQDN / Port / Protocol / Purpose`. Updates to it go through a change process; "we added one more endpoint" is a one-line amendment.

For egress, FQDN-based allow-lists (via proxy or DNS-aware firewall) are preferred over IP-based; model-provider IPs change. For inbound, the pilot's publicly-reachable surface (if any) is behind a WAF or cloud-provider ingress with per-IP rate limits and geo restrictions as appropriate.

## Observability and audit

The networking posture must produce an audit trail the customer can inspect:

- Flow logs on each VPC / VNet (VPC Flow Logs, Cloud Flow Logs, NSG Flow Logs) retained for the customer's compliance period.
- Egress proxy logs to the customer's SIEM.
- Load balancer / API gateway access logs on any publicly reachable surface.

These are the artifacts the customer will subpoena when a security question arises. Chapter 09 covers the handoff to the customer's data-engineering team if those logs feed a lake.

## The security-review checklist

Every regulated customer has a security-review checklist. The vendor-neutral version the FDE should be able to answer:

1. Where does the pilot run? (Topology.)
2. What data does the pilot process? (Scope — ties to mod-108's data-handling.)
3. How does the pilot authenticate users? (Chapter 02.)
4. How does the pilot authenticate services? (Chapter 03.)
5. Where does traffic from the customer to the pilot flow? (Private / public / proxied.)
6. Where does egress from the pilot flow? (Proxy / private endpoint / direct.)
7. What are the DNS and certificate postures?
8. How is secret material managed? (Chapter 03.)
9. What logs are produced, where do they live, who can read them?
10. What is the incident-response contact and the SLO for breach notification?

The integration design's networking section answers 5, 6, 7, 10 directly; the rest are referenced elsewhere. If the pilot cannot answer each, the security review will produce a finding — and findings before go-live move go-live.

## Getting this done in time

The single most useful FDE discipline on networking: *it is a week-2 decision, not a week-10 decision.*

- Week 1 of the pilot — kickoff, discovery (mod-102 Chapter 01).
- **Week 2** — initial integration design covers topology and private-connectivity pattern; put the request to the customer's platform team.
- **Week 3–4** — customer's platform team executes (PrivateLink endpoint service, VPC association, DNS record, firewall rule). This is multi-week; starting it at week 8 makes go-live slip.
- Week 4+ — pilot's own development can proceed in a staging environment while the production networking is being laid.

The pilot team does not need to wait on networking to start building; it does need to not block go-live on networking that was not started.

## Common failure modes

**"We'll figure out PrivateLink closer to go-live."** PrivateLink / Private Service Connect / Azure Private Link are multi-week to set up in a large customer because of change-management windows, DNS zone changes, firewall-rule approvals, and endpoint-service limits. Start it at week 2, not week 8.

**Egress is unbounded by default.** The pilot can reach anything on the internet because the VPC has a default route. The customer's security review flags it. The fix is an egress proxy + FQDN allow-list; the install is a week; retrofit cost is low; running without one is a sure finding.

**Private DNS forgotten.** The customer's VPC endpoint resolves privately — but the pilot's host is using a public resolver, so the pilot's call to `api-pilot.customer.example` resolves to the public IP. The private endpoint is bypassed; the customer's firewall alerts. Fix: enable the VPC's resolver, associate the private hosted zone, verify with `dig` from inside the pilot host.

**Model-provider certificate pinning.** The pilot's HTTP client pins the model provider's TLS certificate (or its CA) to a specific value from last year. The provider rotates. The pilot fails to connect. Do not pin provider certs at this layer; let the system trust store handle it.

**Firewall rules updated manually, not in version control.** Week 10 of the pilot, an engineer SSHes in and opens a port to debug. At week 12 the customer's security scan finds it. The change wasn't reviewed, it wasn't logged, nobody owns closing it. Firewall rules live in Terraform / CloudFormation / the customer's IaC; changes go through PRs.

## Summary

- Networking posture is the first thing the customer's security team reads and the single most common cause of late-pilot slip. It is a week-2 decision.
- Four pilot topologies: multi-tenant SaaS, dedicated SaaS, customer-VPC install, on-prem. The customer's regulatory posture and operational model usually pick one.
- Private connectivity between pilot and customer: PrivateLink (AWS), Private Service Connect (GCP), Private Link (Azure) are the preferred patterns. VPC peering and VPN are available where those are not.
- Mainframe / on-prem data flows reach the pilot through VPN or ExpressRoute / Direct Connect / Interconnect, typically via a staging VPC.
- The pilot's egress to the model provider goes through either an egress proxy with FQDN allow-list, a provider-published VPC endpoint (Bedrock, Azure OpenAI, Vertex AI PSC), or — in lower-regulated pilots — direct over TLS.
- DNS, certificates, and naming are the three cross-cutting concerns that silently break pilots. Private DNS zones; TLS-verified certs; customer-namespace naming. No `InsecureSkipVerify`.
- Firewall rules and allow-lists go into the integration design as a table; updates go through PRs, not through SSH.
- The security-review checklist is answered in the integration design. If you cannot answer each of the ten questions, the review will produce findings — and findings before go-live move go-live.
