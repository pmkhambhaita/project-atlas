# project-atlas

> AI-native internal developer platform — catalog, docs, contextual chat, and governed AI agents. FOSS, TypeScript monorepo.

[![CI](https://github.com/pmkhambhaita/project-atlas/actions/workflows/ci.yml/badge.svg)](https://github.com/pmkhambhaita/project-atlas/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-blue)](https://www.typescriptlang.org/)

---

## What is Atlas?

Atlas is an open-source internal developer platform that combines:

- **Service catalog** — register and discover apps, APIs, libraries, environments, and teams (Backstage-inspired, without the XML)
- **Living documentation** — Markdown docs attached to catalog items, with version history and inline editing
- **Contextual AI chat** — ask questions grounded in catalog metadata and doc content; sources cited per response
- **AI patch proposals** — AI summarises chat threads into Markdown diffs; humans review and approve; approved patches land as GitHub PRs
- **Governed agents** — tiered autonomy (Low / Medium / High risk) with full audit trail and provenance on every AI action
- **Azure sandpit patterns** — cost-aware, Terraform-managed ephemeral environments at ~£35/month

---

## Core loop

```
Register app → Attach docs → Ask question → AI proposes patch → Human approves → PR merged
```

---

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 14, React, shadcn/ui, Tailwind CSS |
| API | Fastify, TypeScript, Zod |
| Auth | Better Auth (GitHub OAuth, OIDC-compatible) |
| Database | PostgreSQL 16 + pgvector (Drizzle ORM) |
| Queue | BullMQ + Redis |
| AI | OpenAI / Anthropic (abstracted via LLMClient) |
| Infrastructure | Azure Container Apps, Terraform |
| CI | GitHub Actions (lint, typecheck, test, build) |
| Package manager | pnpm workspaces |

---

## Repository structure

```
project-atlas/
├── apps/
│   ├── web/          # Next.js frontend
│   └── api/          # Fastify API server
├── packages/
│   ├── agents/       # AI agent implementations
│   ├── db/           # Drizzle schema and migrations
│   ├── queue/        # BullMQ job definitions
│   └── shared/       # Shared types and utilities
├── infrastructure/
│   └── azure/        # Terraform modules for Azure sandpit
├── docs/
│   ├── adr/          # Architecture Decision Records
│   ├── architecture/ # C4 diagrams (Mermaid)
│   └── prd/          # Product Requirements Documents
├── .github/
│   ├── workflows/    # CI pipeline
│   └── PULL_REQUEST_TEMPLATE.md
├── AGENTS.md         # AI agent context for Claude Code / Codex
├── CLAUDE.md         # Platform context for Claude
├── CONTRIBUTING.md   # Contribution guide
└── SKILLS.md         # Skill map for contributors
```

---

## Agent roster

| Agent | Role | Risk tier |
|---|---|---|
| `ContextAgent` | Retrieves semantic context chunks for a query | Low |
| `ConversationDistillerAgent` | Summarises chat threads into patch proposals | Low |
| `DocPatchAgent` | Drafts Markdown diffs against existing doc pages | Medium |
| `IndexingAgent` | Keeps pgvector index in sync with doc changes | Low |
| `GovernanceAgent` | Classifies patch risk and routes for approval | Medium |
| `PRAgent` | Opens GitHub PRs for approved patches | High |

All agents communicate via BullMQ job queue with `traceId` propagation. Every patch carries provenance metadata (source thread, message IDs, agent version, reviewer).

---

## Getting started

### Prerequisites

- Node.js 20+
- pnpm 9+
- Docker (for local PostgreSQL + Redis)
- GitHub OAuth app credentials

### Local development

```bash
# Clone
git clone https://github.com/pmkhambhaita/project-atlas.git
cd project-atlas

# Install dependencies
pnpm install

# Start local services (Postgres + Redis)
docker compose up -d

# Copy and fill environment variables
cp .env.example .env

# Run database migrations
pnpm db:migrate

# Start all apps in dev mode
pnpm dev
```

The web UI will be available at `http://localhost:3000` and the API at `http://localhost:3001`.

### Azure sandpit deployment

```bash
# Create a named sandpit with a 7-day TTL
pnpm atlas sandpit:create --name my-feature --ttl 7d

# Destroy it when done
pnpm atlas sandpit:destroy --name my-feature
```

See [ADR 0003](docs/adr/0003-azure-sandpit-strategy.md) for the full sandpit architecture and cost breakdown (~£35/month).

---

## Documentation

| Document | Description |
|---|---|
| [MVP PRD](docs/prd/mvp.md) | Product requirements, user stories, and phased delivery plan |
| [ADR 0001 — Tech Stack](docs/adr/0001-tech-stack.md) | Why we chose this stack |
| [ADR 0002 — AI Agent Architecture](docs/adr/0002-ai-agent-architecture.md) | Agent roster, risk tiers, provenance model |
| [ADR 0003 — Azure Sandpit](docs/adr/0003-azure-sandpit-strategy.md) | Cost-aware sandpit deployment pattern |
| [C4 Architecture](docs/architecture/c4-context.md) | Context, container, component diagrams + sequence flows |
| [AGENTS.md](AGENTS.md) | AI agent context file for Claude Code / Codex |
| [CONTRIBUTING.md](CONTRIBUTING.md) | How to contribute |

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Key points:

- All PRs require passing CI (lint, typecheck, test, build)
- Use the PR template and skills checklist
- AI-generated patches must include provenance metadata
- High-risk agent actions require maintainer approval

---

## License

[MIT](LICENSE) — free to use, modify, and distribute.
