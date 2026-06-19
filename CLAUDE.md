# CLAUDE.md — Project Atlas

> This file is read automatically by Claude Code when you open this repository. It gives Claude context about the project, conventions, and rules so you get accurate, on-brand output from the first prompt.

---

## What this project is

**Project Atlas** is an AI-native internal developer portal. It combines a software catalog, a structured knowledge base, contextual team discussions, and a governed AI agent system into one product.

The core loop is: developer asks a question in context → team answers → AI extracts the answer → AI proposes a docs update → human reviews and approves → knowledge becomes canonical.

Key properties:
- FOSS-first, MIT licensed
- TypeScript monorepo (Turborepo + pnpm)
- React + Next.js frontend (`apps/web`)
- Fastify API (`apps/api`)
- Async agent workers (`apps/worker`)
- PostgreSQL as the canonical data store
- AI agents are event-driven, narrow in scope, and always produce reviewable proposals in MVP
- No agent writes canonical docs directly — all changes go through `patch_proposals`

---

## Monorepo structure

```
project-atlas/
  apps/
    web/          # Next.js portal UI
    api/          # Fastify REST API
    worker/       # AI agent job runners
    ingest/       # Repo and doc ingestion
  packages/
    domain/       # Shared entities, types, DB schemas
    agents/       # Agent contracts, prompts, runners
    search/       # Search indexing and query logic
    policy/       # Risk tier and merge policy engine
    ui/           # Shared React component library (shadcn/ui base)
    integrations/ # Git provider, identity, notifications
  skills/         # Custom Claude Code skills for this project
  docs/           # Product docs, ADRs, architecture diagrams
    adr/          # Architecture decision records
    product/      # PRD, user stories, specs
    architecture/ # C4 diagrams and system docs
```

---

## Tech stack

| Layer | Choice | Notes |
|---|---|---|
| Frontend | Next.js + React | App router, TypeScript |
| UI components | shadcn/ui + Tailwind | Use `/ui-designer` skill |
| API | Fastify | Typed routes, Zod validation |
| ORM | Drizzle ORM | Type-safe, close to SQL |
| Database | PostgreSQL | Single source of truth |
| Search | Postgres FTS → Meilisearch | FTS first, migrate when needed |
| Auth | Better Auth / Auth.js | OIDC-compatible for Entra |
| Job queue | BullMQ (Redis-backed) | For agent workers |
| Monorepo | Turborepo + pnpm | Fast builds, smart caching |
| Testing | Vitest + Playwright | Unit + E2E |
| Linting | ESLint + Prettier + Biome | Enforced in CI |
| CI | GitHub Actions | See `.github/workflows/` |

---

## Core domain entities

These are the canonical objects in the system. Always use these names consistently:

- **App** — a deployable software unit
- **Pattern** — an approved build/deploy recipe
- **Environment** — a deployment target (sandpit, test, prod)
- **Repository** — a source control binding
- **Team** — an owning group
- **Document** — a knowledge artifact
- **DocSection** — an addressable heading within a document
- **Thread** — a contextual Q&A discussion tied to an entity
- **Message** — a post within a thread
- **Insight** — AI-extracted reusable knowledge from a resolved thread
- **PatchProposal** — an AI-drafted document change awaiting approval
- **Approval** — a review decision on a patch proposal
- **AgentRun** — an immutable audit record of an agent execution

---

## Agent rules (non-negotiable)

1. Agents never write to the `documents` table directly.
2. All agent doc changes create a `PatchProposal` with status `pending`.
3. Every `AgentRun` is logged and immutable.
4. Agents respect `ai-edit: false` frontmatter on any document section.
5. Agent prompts live in `packages/agents/prompts/` — never hardcoded inline.
6. Model names are never hardcoded — use `packages/agents/model-config.ts`.
7. See `AGENTS.md` for the full agent roster and risk tier definitions.

---

## Code conventions

### TypeScript
- Strict mode always on
- No `any` types without explicit justification in a comment
- Use Zod for all runtime validation (API inputs, agent outputs, env vars)
- Use `type` imports: `import type { Foo } from './foo'`
- Prefer `const` functions over `function` declarations in modules

### File naming
- kebab-case for all files and folders
- `.ts` for pure logic, `.tsx` for React components
- Co-locate tests: `foo.ts` → `foo.test.ts`
- Co-locate types when small: `foo.types.ts`

### API routes (Fastify)
- Every route has a Zod schema for params, query, body, and response
- Every route returns a typed response object
- Errors use the shared error shape from `packages/domain/errors.ts`
- No business logic in route handlers — delegate to service functions

### Database (Drizzle)
- Schema defined in `packages/domain/schema/`
- Migrations in `packages/domain/migrations/`
- Never write raw SQL strings in application code — use Drizzle query builder
- All timestamps are UTC ISO 8601 strings stored as `timestamptz`

### React / Next.js
- Use shadcn/ui components as the base — don't reinvent
- Every new UI component needs: default state, loading, error, empty
- WCAG 2.2 AA is the minimum accessibility standard
- Use `useQuery` / `useMutation` patterns (TanStack Query)
- No `useEffect` for data fetching — always use TanStack Query or server components

---

## Testing conventions

- Unit tests in Vitest: all domain logic, all agent output parsing, all utility functions
- Integration tests: API route handlers with real DB (test container)
- E2E tests in Playwright: critical user journeys only
- Test files co-located with source: `src/foo.ts` → `src/foo.test.ts`
- Run `pnpm test` before every PR

---

## Git conventions

### Branch naming
```
feat/<short-description>
fix/<short-description>
docs/<short-description>
chore/<short-description>
refactor/<short-description>
agent/<short-description>   # for agent-raised branches
```

### Commit messages (Conventional Commits)
```
feat: add patch proposal approval endpoint
fix: thread resolve event not triggering distiller
docs: update AGENTS.md with indexer agent
chore: bump drizzle-orm to 0.31.0
refactor: extract insight classifier into separate module
test: add unit tests for patch proposal service
```

### PR rules
- Every PR must reference an issue or Linear ticket
- Run `/ponytail-review` before opening
- CI must pass before merge
- Agent-raised PRs require at least 1 human approval
- Docs changes don't require code review if CI passes and content is from an approved agent run

---

## Skills to use

See `SKILLS.md` for the full list. Quick reference for the most important:

| Situation | Skill |
|---|---|
| Before any PR | `/ponytail-review` |
| Starting a feature | `/writing-plans` |
| Debugging an issue | `/systematic-debugging` |
| New UI screen | `/frontend-design` then `/ui-designer` |
| New API route | `/api-design-principles` |
| New agent | `/architecture` then `/writing-plans` then `/tdd` |
| Security-sensitive change | `/security-auditor` |

---

## What Claude should NOT do in this repo

- Do not use `any` types
- Do not write raw SQL strings
- Do not hardcode model names or API keys
- Do not write to `documents` table directly from agent code
- Do not create new UI components without checking shadcn/ui first
- Do not skip tests for domain or agent logic
- Do not merge agent changes without human approval, even in post-MVP phases
- Do not create files outside the monorepo structure above without discussing first
- Do not use `useEffect` for data fetching
- Do not skip error and loading states in UI components

---

## Useful commands

```bash
# Install all deps
pnpm install

# Run all apps in dev mode
pnpm dev

# Run tests
pnpm test

# Lint and type-check
pnpm lint
pnpm typecheck

# Build all
pnpm build

# DB migration
pnpm db:migrate
pnpm db:studio
```

---

## Getting started

1. Clone the repo
2. Copy `.env.example` to `.env.local` and fill in values
3. Run `pnpm install`
4. Run `pnpm db:migrate`
5. Run `pnpm dev`
6. Install skills from `SKILLS.md` for your role
7. Read `AGENTS.md` before touching anything in `packages/agents/`

---

## Links

- `AGENTS.md` — agent roster, contracts, governance
- `SKILLS.md` — Claude Code skills and MCPs
- `CONTRIBUTING.md` — contribution workflow
- `docs/product/PRD.md` — product requirements
- `docs/architecture/` — C4 diagrams and system docs
- `docs/adr/` — architecture decision records
