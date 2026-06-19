# ADR 0003 — Azure Sandpit Deployment Strategy

**Date:** 2025-07-13  
**Status:** Accepted  
**Deciders:** Platform team  
**Tags:** infrastructure, azure, cost, deployment

---

## Context

Project Atlas targets teams that need rapid, low-cost infrastructure for internal tooling and AI-native workloads. "Sandpit" environments are ephemeral or semi-persistent Azure deployments used for development, demos, and team-level experimentation, distinct from production infrastructure.

Challenges:
- Azure costs spiral quickly without deliberate guardrails
- Teams want self-service environments without infra team bottlenecks
- Sandpits must be reproducible, codified, and destroyable on demand
- Most Atlas contributors are on free/student Azure credits or small organisational budgets

---

## Decision

We adopt a **cost-aware, Terraform-managed sandpit pattern** on Azure with the following principles:

### 1. Everything-as-Code

All sandpit infrastructure is defined in Terraform under `infrastructure/azure/`. No click-ops. Sandpits are created and destroyed via CLI commands or GitHub Actions workflows.

```
infrastructure/
  azure/
    modules/
      sandpit/          # reusable sandpit module
        main.tf
        variables.tf
        outputs.tf
    environments/
      dev/              # developer sandpit
      staging/          # pre-prod sandpit
    main.tf
    variables.tf
```

### 2. Sandpit Resource Composition

A standard Atlas sandpit includes only what is needed for the MVP:

| Resource | SKU / Tier | Monthly est. cost |
|---|---|---|
| Azure Container Apps (API) | Consumption plan | ~£0–2 (pay-per-request) |
| Azure Container Apps (Workers) | Consumption plan | ~£0–1 |
| Azure Database for PostgreSQL Flexible Server | Burstable B1ms | ~£13 |
| Azure Cache for Redis | Basic C0 | ~£13 |
| Azure Container Registry | Basic | ~£4 |
| Azure Static Web Apps | Free tier | £0 |
| Azure Blob Storage | LRS, hot tier | ~£1 |
| **Total estimate** | | **~£31–35/month** |

Prod-like scale can be reached by upgrading individual resources independently. No re-architecture required.

### 3. Auto-shutdown and TTL Tags

All sandpit resources carry a `ttl` tag. A scheduled Azure Function (or GitHub Actions cron) checks for expired sandpits and runs `terraform destroy` against them.

```hcl
tags = {
  environment = "sandpit"
  owner       = var.owner_email
  ttl         = var.ttl_date        # e.g. "2025-08-01"
  project     = "atlas"
  cost-centre = var.cost_centre
}
```

### 4. Cost Budget Alerts

Every sandpit resource group has an Azure Cost Management budget alert at £50/month. At 80% spend, an email alert fires. At 100%, an Action Group triggers a Teams/Slack webhook.

### 5. Golden-Path Sandpit Command

Developers create a sandpit with a single command:

```bash
pnpm atlas sandpit:create --name my-feature --ttl 7d
```

This invokes a Terraform workspace, provisions resources, outputs connection strings, and registers the sandpit in the Atlas catalog automatically.

Destroy:
```bash
pnpm atlas sandpit:destroy --name my-feature
```

### 6. CI/CD Integration

PRs targeting `main` can opt-in to a sandpit deployment via label `deploy-sandpit`. The GitHub Actions workflow:

1. Provisions a Terraform workspace named after the PR (`pr-{number}`)
2. Deploys the current branch
3. Posts the sandpit URL as a PR comment
4. Destroys the sandpit when the PR is merged or closed

### 7. Secrets Management

Secrets are stored in Azure Key Vault, referenced by Container Apps via managed identity. No secrets in environment variable files or Terraform state.

---

## Consequences

### Positive

- Full cost visibility and predictability from day one
- Self-service sandpits reduce infra team dependency
- TTL tags prevent forgotten, billable resources
- Terraform workspaces allow per-PR environments with zero naming conflicts
- Pattern is portable: same modules work for staging and prod with different variable files

### Negative / trade-offs

- PostgreSQL Flexible Server has a ~5 minute cold start time when paused; not suitable for latency-sensitive demos
- Azure Container Apps has limited support for long-running background workers; may need ACA Jobs for batch tasks
- Terraform state must be stored remotely (Azure Blob backend); adds a bootstrap step for new contributors

### Neutral

- This pattern is Azure-specific; a GCP or AWS equivalent would require different resource mappings but the same principles apply
- Meilisearch is not included in the sandpit by default; Postgres FTS is used instead to keep costs low

---

## Alternatives considered

| Approach | Reason not chosen |
|---|---|
| Azure Kubernetes Service (AKS) | Overkill for MVP; minimum node pool cost exceeds sandpit budget |
| Pulumi | Team familiarity with Terraform; Pulumi adds language-specific complexity |
| Manual Azure Portal provisioning | Not reproducible, not destroyable, not auditable |
| App Service Plan (always-on) | Higher baseline cost than Container Apps consumption plan |
| Local Docker Compose only | Insufficient for realistic cloud testing; doesn't validate networking or managed identity |
