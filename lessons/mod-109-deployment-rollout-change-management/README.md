# mod-109-deployment-rollout-change-management: Deployment, Rollout, and Change Management with the Customer

> Scaffolded by `aicg org execute-plan`. Lecture chapters and exercise content are authored on subsequent autonomous cycles.

**Estimated effort:** 14 hours

## Learning objectives

- Deploy inside the customer's environment — SaaS, dedicated SaaS, VPC install (Terraform / Helm shipped by the FDE), on-prem for air-gapped customers — with explicit per-environment trade-offs on latency / model choice / update cadence / support model
- Ship a feature-flag strategy with LaunchDarkly / Statsig / Unleash — per-tenant, per-user, per-prompt, per-model rollout; shadow deployments; kill-switch controls; audit-log of every flag flip
- Run canary + shadow + A/B rollouts on LLM features — canary on 1% of traffic with explicit success criteria, shadow comparison of the new prompt / model / retriever against the current in production, blameless rollback if the canary burns the eval budget
- Version per-tenant artifacts — prompts, retrievers, models, tool allow-lists, judges — with explicit lineage, the ability to replay any past customer interaction against any past artifact bundle, and a diff review workflow
- Hand off to the customer-side operator — admin dashboards, SME training materials, playbooks for common failure modes, escalation paths; coordinate with `ai-enablement-lead` (peer, level 35) on the organisational change-management slice
- Author the pilot-to-production hand-off packet — architecture diagram, runbook, SLA shape, cost model, eval dashboard, security-review evidence, licence / commercial contract pointer

## Structure

- `01-…md` … `0N-…md`: lecture chapters.
- `exercises/`: per-exercise prompts.
- `labs/`: long-form hands-on labs.
- `quizzes/`: knowledge checks.
- `resources.md`: external references.
