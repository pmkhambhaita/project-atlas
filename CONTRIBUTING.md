# Contributing to Project Atlas

Thank you for contributing. Atlas is a FOSS project built to high engineering standards. This guide explains how to contribute effectively.

---

## Before you start

1. Read `CLAUDE.md` — it explains the project, stack, and conventions
2. Read `AGENTS.md` — mandatory if you're touching agent code
3. Install skills from `SKILLS.md` for your role
4. Make sure you have an issue or Linear ticket for your work

---

## Development setup

```bash
# Prerequisites: Node.js 20+, pnpm 9+, Docker (for Postgres)

git clone https://github.com/pmkhambhaita/project-atlas
cd project-atlas
pnpm install
cp .env.example .env.local
# Fill in .env.local values
pnpm db:migrate
pnpm dev
```

---

## Workflow

### 1. Branch

Create a branch from `main`:

```bash
git checkout -b feat/my-feature
# or
git checkout -b fix/my-fix
# or
git checkout -b docs/update-readme
```

Branch naming:
- `feat/<description>` — new feature
- `fix/<description>` — bug fix
- `docs/<description>` — documentation only
- `chore/<description>` — tooling, deps, config
- `refactor/<description>` — no behaviour change
- `agent/<description>` — agent-raised branch (do not use manually)

### 2. Write code

- Follow conventions in `CLAUDE.md`
- Run `/writing-plans` before starting complex features
- Write tests alongside code — see testing conventions in `CLAUDE.md`

### 3. Before committing

```bash
pnpm lint
pnpm typecheck
pnpm test
```

Run `/ponytail-review` in Claude Code on any changed files.

### 4. Commit

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add thread resolve endpoint
fix: patch proposal status not updating on approval
docs: add troubleshooting section to Bruno setup guide
chore: bump drizzle-orm to 0.31.0
test: add unit tests for insight classifier
```

### 5. Open a PR

- Use the PR template
- Reference the issue or Linear ticket
- Run `/requesting-code-review` to generate your review request description
- CI must be green before requesting review

### 6. Code review

- All PRs require at least 1 approval from a maintainer
- Agent-raised PRs require 1 human approval minimum
- Docs-only PRs can be self-merged by maintainers after CI passes
- Address all review comments before merging
- Use `/receiving-code-review` to structure your response

### 7. Merge

- Squash merge only (keeps history clean)
- Delete the branch after merge

---

## Agent code

Agent code in `packages/agents/` has stricter rules:

- Run `/atlas-agents` skill before writing any agent code
- Run `/architecture` before designing a new agent
- Every agent change requires human review, regardless of risk tier
- All agent changes must include tests for output contracts
- Prompts go in `packages/agents/prompts/` — never inline

---

## Docs contributions

- Run `/atlas-docs` skill before writing documentation
- Follow the frontmatter format defined in the docs skill
- Troubleshooting entries should follow the standard format
- Docs PRs are welcome without a linked issue if they fix clear inaccuracies

---

## UI contributions

- Run `/atlas-ui` skill before building new screens
- Run `/frontend-design` for any new page or major component
- Every component must handle all 4 states: default, loading, error, empty
- WCAG 2.2 AA is required
- Use shadcn/ui components first

---

## Reporting issues

- Use GitHub Issues for bugs and feature requests
- Search existing issues before opening a new one
- Include steps to reproduce for bugs
- Include context about your environment

---

## Questions

Open a thread in the relevant context on the platform itself (once it's running), or open a GitHub Discussion.

---

## Code of conduct

Be direct, be specific, be kind. Don't waste people's time with vague feedback or meandering PRs. This is a craft project — hold the code to a high standard and do the same for yourself.
