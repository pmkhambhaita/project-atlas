# SKILLS.md — Project Atlas

> Curated skills, MCPs, and tooling for everyone building on Project Atlas. Install what fits your role.

---

## Quick-install reference

```bash
npx -y skills add <org>/<repo> --skill <skill-name> --agent claude-code
```

---

## Core skills (all contributors)

### Ponytail — anti-overengineering review

```bash
npx -y skills add DietrichGebert/ponytail --skill ponytail-review --agent claude-code
```

Systematic PR review that hunts complexity over correctness. Flags reinvented stdlib, unneeded deps, speculative abstractions, dead flexibility. One line per finding: location, what to cut, replacement. Run before every PR.

Triggers: `/ponytail` `/ponytail-review` `/ponytail-help`

---

### Writing plans

```bash
npx -y skills add anthropics/claude-code --skill writing-plans --agent claude-code
```

Creates actionable scoped plans before implementation. Use before any feature, agent, or integration work.

---

### Executing plans

```bash
npx -y skills add anthropics/claude-code --skill executing-plans --agent claude-code
```

Tracks plan items and keeps execution aligned to original scope. Pair with writing-plans.

---

### Systematic debugging

```bash
npx -y skills add anthropics/claude-code --skill systematic-debugging --agent claude-code
```

4-phase framework: Reproduce > Isolate > Identify > Fix. Prevents shotgun debugging. Essential for agent pipeline and async worker issues.

---

### Test-driven development

```bash
npx -y skills add anthropics/claude-code --skill test-driven-development --agent claude-code
```

Enforces Red-Green-Refactor. Mandatory for all domain logic, agent contracts, and API handlers.

---

### Requesting code review

```bash
npx -y skills add anthropics/claude-code --skill requesting-code-review --agent claude-code
```

Generates structured review requests with full context so reviewers don't have to ask follow-up questions.

---

### Receiving code review

```bash
npx -y skills add anthropics/claude-code --skill receiving-code-review --agent claude-code
```

Structured process for ingesting feedback, planning changes, and tracking resolution.

---

### Brainstorming

```bash
npx -y skills add anthropics/claude-code --skill brainstorming --agent claude-code
```

Forces exploration of alternatives before committing to an approach. Use before designing any new agent, flow, or tech choice.

---

## Frontend / UI skills

For `apps/web` contributors.

### Frontend Design (Anthropic official)

```bash
# https://github.com/anthropics/claude-code/tree/main/plugins/frontend-design
# Place skill.md in .claude/skills/designer/skill.md
# Trigger: /frontend-design
```

Production-grade frontend design. Breaks AI slop aesthetic by enforcing real design thinking: visual hierarchy, interaction clarity, all component states (hover/focus/loading/error/empty), accessibility-first implementation. Atlas portal UI must feel like a professional tool, not a generated demo.

Covers: design tokens, spacing systems, component specs, WCAG 2.2 AA, responsive layout, dark mode.

---

### UI/UX Design Expert

```bash
# https://mcpmarket.com/tools/skills/ui-ux-design-expert-4
# Trigger: /ui-ux
```

Senior-level UI/UX consultant. Covers design systems, user research workflows, WCAG compliance, design tokens, component libraries, cross-platform consistency. Use when planning new screens or auditing existing UI.

---

### UI Design Principles

```bash
# https://mcpmarket.com/tools/skills/ui-design-principles
```

Research-driven design. Context before design. Ensures all component states are documented, embeds accessibility into the design process, produces implementation-ready specs. Use when starting a new component or screen from scratch.

---

### UI Designer (shadcn-native)

```bash
# https://mcpmarket.com/tools/skills/ui-designer-5
```

Bridges feature planning and implementation using shadcn/ui. Enforces component reuse and project-wide consistency. Atlas uses shadcn/ui as its base component library.

---

## Architecture and backend skills

For `apps/api`, `apps/worker`, `packages/domain`, `packages/agents`.

### Architecture

```bash
npx -y skills add anthropics/claude-code --skill architecture --agent claude-code
```

Forces trade-off analysis, dependency mapping, and ADR documentation. Use for any structural decision.

---

### API Design Principles

```
Trigger: /api-design-principles
```

Enforces REST conventions: resource naming, versioning, error shapes, pagination, OpenAPI hygiene. Use when adding or modifying routes in `apps/api`.

---

### Security Auditor

```
Trigger: /security-auditor
```

Systematic review for injection, auth bypass, insecure defaults, exposed secrets, IDOR, missing rate limits. Mandatory on any PR touching auth, permissions, or agent-to-git workflows.

---

### Lint and Validate

```
Trigger: /lint-and-validate
```

Runs linting, type checking, schema validation before commit. Prevents broken types reaching main. Run before every commit on API or domain code.

---

## MCP server integrations

### GitHub MCP (essential)

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "<your-pat>" }
    }
  }
}
```

Repo management, issues, PRs, code search, Actions status. Essential — mirrors how Atlas agents interact with GitHub.

---

### PostgreSQL MCP

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": { "POSTGRES_CONNECTION_STRING": "postgresql://localhost/atlas_dev" }
    }
  }
}
```

Natural language SQL, schema inspection, migration assistance. Use when developing catalog, docs, or thread data models.

---

### Filesystem MCP

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/project-atlas"]
    }
  }
}
```

Advanced file operations for bulk scaffolding and multi-file refactors.

---

### Linear MCP

```bash
npx -y @linear/mcp-server
```

Create issues, update statuses, link PRs to issues, manage cycles. Atlas dev is tracked in Linear.

---

### Tavily MCP

```json
{
  "mcpServers": {
    "tavily": {
      "command": "npx",
      "args": ["-y", "@tavily/mcp"],
      "env": { "TAVILY_API_KEY": "<key>" }
    }
  }
}
```

AI web search for current library docs, error messages, CVEs without leaving the editor.

---

## Skill bundles by role

| Role | Skills | MCPs |
|---|---|---|
| Full-stack | ponytail, writing-plans, executing-plans, tdd, frontend-design, api-design-principles | GitHub, PostgreSQL, Filesystem |
| Frontend / UI | frontend-design, ui-ux-expert, ui-principles, ui-designer, ponytail | GitHub, Filesystem |
| Backend / agents | ponytail, tdd, systematic-debugging, architecture, security-auditor, api-design | GitHub, PostgreSQL |
| Docs / knowledge | doc-coauthoring, brainstorming, writing-plans | GitHub, Filesystem |
| Platform / DevOps | architecture, security-auditor, lint-validate, ponytail | GitHub, PostgreSQL, Linear |

---

## Project conventions

1. `/ponytail-review` before every PR. No exceptions.
2. `/writing-plans` before touching agent code.
3. `/security-auditor` on anything touching auth, permissions, or GitHub integration.
4. `/frontend-design` for every new portal screen.
5. `/tdd` for all domain and agent logic.
6. `/lint-and-validate` before every commit.

---

## Custom skills folder

Project-specific skills live in `skills/` at the repo root:

```
skills/
  atlas-catalog.md    # Catalog domain model conventions
  atlas-agents.md     # Agent development patterns
  atlas-docs.md       # Doc style and patch proposal rules
  atlas-api.md        # API conventions and error shapes
  atlas-ui.md         # UI component and design system rules
```
