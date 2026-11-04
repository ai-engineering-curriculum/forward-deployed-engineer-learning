# Chapter 01 — Origin and Modern Instantiations of the Forward Deployed Engineer

## Why this chapter exists

Before anything about pilots, evals, or integrations, you need to be able to say what Forward Deployed Engineering *is*, in your own words, and name the companies that hire for it. The role shows up under a dozen titles — *Forward Deployed Engineer*, *Forward Deployed Software Engineer*, *Forward Deployed AI Engineer*, *Field AI Engineer*, *Applied AI Engineer (customer-embedded)*, *Solutions Engineer (AI)*, *Deployment Engineer (AI)*, *Customer Engineering — AI*, *Partner Engineer — AI* — and the job description varies with each. This chapter traces the lineage so you can read a posting and tell whether it is the role this track teaches, a relative of it, or a different job that happens to use overlapping words.

## The Palantir origin

The role was named and institutionalised at Palantir Technologies in the mid-2000s. Palantir's product was a general-purpose data analysis platform; the customers were intelligence agencies, defence contractors, and later banks, hospitals, and manufacturers — all organisations with gnarly data, strict access rules, strong legacy systems, and no appetite for a product that showed up as a blank canvas.

Palantir's structural answer was to send a software engineer to live at the customer site. That engineer would:

- Learn the customer's actual workflow by watching it happen.
- Write code *inside the customer's environment* — against their data, behind their firewall, under their auth — to automate the part of the workflow that mattered most to them that week.
- Carry what they learned back into the product so the next customer would get a slightly more capable platform out of the box.

That person was called a **Forward Deployed Software Engineer** — "forward deployed" borrowing the military term for a unit stationed forward of its home base. Palantir has published multiple essays describing the practice; the canonical one is [*A day in the life of a Forward Deployed Software Engineer*](https://blog.palantir.com/a-day-in-the-life-of-a-forward-deployed-software-engineer-b2142f1e36c8) on the Palantir blog.

The pattern that survived from Palantir into modern instantiations is not the specific product, nor the specific industries. It is the structural shape:

1. **Embed with the customer.** Physical or remote, but close enough to see the work happen.
2. **Ship code inside the customer's environment.** Not a prototype on your laptop — a thing running in their stack.
3. **Feed what you learn back into the product.** The engineer is a two-way pipe between the customer and the product team.

Everything else in modern FDE job descriptions — the frontier-lab variant, the applied-AI product variant, the hyperscaler variant — is a refinement of those three structural moves.

## The frontier-lab instantiation

Starting around 2023, as frontier-lab APIs (OpenAI, Anthropic, later Google DeepMind, xAI, Cohere, Mistral) hit a capability threshold where enterprise customers would actually buy them, the labs had to answer the same question Palantir did two decades earlier: *how does a customer get this into production when neither side yet knows what "production" means for a model this capable*?

The answer was the same answer. Hire engineers whose job is to sit inside a design-partner customer, work out what the model can actually do on *that* customer's data, ship it into *that* customer's workflow, and push the hardest lessons back into the model, the API, and the product surface. [OpenAI's customer stories](https://openai.com/customer-stories) and [Anthropic's careers page](https://www.anthropic.com/careers) both document applied-AI / forward-deployed / solutions engineering teams of this shape. Scale AI has a public [Forward Deployed Engineering team](https://scale.com/careers); so does xAI; so, under different titles, do Cohere and Mistral.

What the frontier-lab FDE added to the Palantir template:

- **The product is a model, not a platform.** The levers available to the FDE are prompts, context, retrieval, tools, structured outputs, fine-tunes, and orchestration — not SQL transforms over the customer's warehouse.
- **Model behaviour is probabilistic.** You cannot test a Forward Deployed LLM feature with "does it return the correct row" — you have to build an eval. The role absorbs eval engineering as a core skill, not an optional one.
- **The feedback that goes back to the lab is different.** A Palantir FDE fed workflow patterns and UI needs back to the product team. A frontier-lab FDE feeds failure cases, dangerous-capability edges, cost blowouts, and "we needed a feature on the API that does not exist" requests back to the model-training, product, and platform teams at once.

## The applied-AI product-company instantiation

In parallel, a cohort of applied-AI product companies — companies whose entire product is an LLM-powered application for a specific vertical — built out teams with the same structural job. These companies include Sierra (customer support agents), Decagon (support automation), Hebbia (knowledge retrieval for finance), Writer (enterprise writing), Harvey (legal), Glean (enterprise search), Cresta (contact-center AI), Augment and Factory and Cursor/Anysphere (developer tools), Perplexity Enterprise, Replit, Vercel's AI team. They also appear inside vertical AI companies: Qventus and Abridge and PathAI in health, EvenUp and Casetext/Thomson Reuters CoCounsel in legal, Moveworks in IT, Chargeflow in payments.

The applied-AI product FDE's job is closer to Palantir's than to the frontier lab's. The product is a product (not a platform and not a raw model), but it needs customer-specific adaptation to win and keep an enterprise account. The FDE:

- Embeds with the design partners for new product lines.
- Ships integrations against the customer's identity provider, data surface, and SaaS stack.
- Runs the pilot through security review, compliance review, and go-live inside a shared on-call rotation with the customer's operators.
- Carries the pattern — what works, what breaks, what the product needs — back into the product and platform engineering teams.

What the applied-AI product FDE added to the template:

- **Compliance is the gating step.** SOC 2, HIPAA, FedRAMP, GDPR, EU AI Act — the pilot does not ship if the customer's security review does not clear. The FDE has to be fluent enough to carry this review, not just handwave it.
- **The vertical shape matters.** A healthcare FDE and a legal FDE are solving structurally different problems — different regulations, different data shapes, different stakeholder maps, different definitions of "good enough".
- **Shared on-call.** The pilot runs in the customer's production. When it breaks at 3am, the FDE is in the incident bridge.

## The hyperscaler and SaaS-platform instantiation

A third variant has grown inside the hyperscalers and enterprise SaaS platforms — AWS Generative AI Innovation Center, Google Cloud Vertex AI Customer Engineering, Microsoft Azure AI GBB, Databricks Field Engineering, Snowflake Field Engineering, Salesforce AgentForce engineering, ServiceNow Now Assist engineering, Workday AI, SAP Joule. These teams exist to help the platform's biggest customers put AI features into production on top of the platform. The structural job is the same; the stack constraint is narrower (you are shipping on your employer's platform, usually); the counterpart is often the customer's own platform team rather than a product team.

## What the four instantiations share

Across Palantir, frontier labs, applied-AI product companies, and hyperscaler/SaaS platforms, the role shares four structural stances. These are the subject of the next chapter. Preview:

1. **Embedded delivery stance.** The engineer works close to the customer's real workflow, not downstream of a requirements document.
2. **No-ego engineering craft.** The engineer ships whatever gets the pilot to work — Python glue, SQL hacks, config patches, a 50-line retrieval layer — and does not defend a pet architecture.
3. **Design-partner-first mindset.** The customer is a co-author of the product, not an endpoint for a shipped feature.
4. **Field → product loop as a first-class deliverable.** The FDE is explicitly measured on what they push back from the customer into the product, not only on what they ship at the customer.

## How to read a job posting

Given a job posting with "Forward Deployed Engineer" (or one of the equivalent titles) in the subject line, you can tell which instantiation it is by checking five signals:

| Signal | Palantir-style | Frontier-lab | Applied-AI product | Hyperscaler/SaaS |
|---|---|---|---|---|
| Product shape | Platform | Model/API | Vertical app | Cloud platform |
| Primary technical lever | Pipelines + UI on customer data | Prompts + eval + agents against API | App config + integrations + evals | Platform features + partner integrations |
| Customer-side peer | Analyst / ops lead | AI lead / CTO | Line-of-business sponsor | Customer platform team |
| Compliance surface | FedRAMP / IL5 / internal sec | Customer sec review + lab's responsibility policy | Vertical regs (HIPAA, FedRAMP, EU AI Act) | Customer sec review + platform SOC/ISO |
| What goes back to product | Workflow patterns, UI needs | Failure cases, capability edges, API gaps | Feature requests, pattern writeups | Platform feature requests, partner-integration gaps |

The track you are reading teaches to the applied-AI-product and frontier-lab variants primarily, with enough of the hyperscaler/SaaS variant to be dangerous. The Palantir variant is the ancestor and is treated as historical reference rather than target.

## A note on what the FDE is *not*

The role is not any of these, even though the boundary gets fuzzy:

- Not an Account Executive (AE) — the FDE does not carry a quota and does not close deals.
- Not a Solutions Engineer / pre-sales engineer in the narrow "build the demo that wins the deal" sense — the FDE ships *after* the deal is signed (or inside a design-partner contract that explicitly swaps revenue for product influence).
- Not a Customer Success Manager — the FDE's job is technical delivery, not account health metrics.
- Not a Professional Services Consultant — the FDE's work is billed-against-product-roadmap-influence, not billed-against-customer-hours.
- Not a Developer Advocate — the FDE serves one customer at depth, not an ecosystem at breadth.

Chapter 04 goes through each of these adjacent roles in detail and names the sharp boundaries this track enforces. For now, the point is that "customer-facing engineering" has at least a half-dozen distinct roles, and the FDE is one of them.

## Summary

- The Forward Deployed Engineer role was invented at Palantir to put software engineers inside customers' environments to ship against real workflows and feed learnings back to the product.
- The role has been re-instantiated by frontier labs (OpenAI, Anthropic, Scale AI, xAI, Cohere, Mistral), applied-AI product companies (Sierra, Decagon, Hebbia, Writer, Harvey, Glean, Cresta, and others), and hyperscaler / enterprise-SaaS customer-engineering orgs (AWS GenAI Innovation Center, Google Cloud, Azure AI GBB, Databricks Field, Snowflake Field, Salesforce AgentForce).
- The four instantiations share a structural shape: embedded delivery, no-ego engineering, design-partner-first mindset, field → product loop as a first-class deliverable. The next chapter unpacks each stance.
- You can read a posting and classify it against the four instantiations by checking product shape, technical lever, customer-side peer, compliance surface, and what goes back to the product.
- The FDE is distinct from AE, pre-sales SE, CSM, Professional Services, and Developer Advocate. Chapter 04 draws those boundaries.
