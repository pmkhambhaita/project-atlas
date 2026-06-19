# Atlas Docs Skill

**Trigger:** `/atlas-docs`  
**Scope:** Use when writing, reviewing, or generating documentation in this project

---

## Documentation model

All docs in Atlas are structured, versioned artifacts. They are not a wiki. They are not free-form notes. They are operational knowledge that must remain accurate, searchable, and linked to catalog entities.

---

## Required document types per app

Every active app must have all of the following documents:

| Doc type | Purpose |
|---|---|
| `overview.md` | What the app is, who it's for, and its lifecycle status |
| `setup.md` | How to run it locally from zero |
| `deploy.md` | How to deploy it to each environment |
| `rollback.md` | How to revert a bad deployment |
| `troubleshooting.md` | Known issues, error codes, and resolutions |
| `cost.md` | Cost profile, scale-to-zero behaviour, shared infra |
| `ownership.md` | Owner, support contact, escalation path |

---

## Document frontmatter

Every document must have frontmatter:

```yaml
---
title: Setup Guide
entityType: app
entityId: my-app-slug
docType: setup
status: draft | review | published
aiEdit: true           # set to false to block AI edits
owner: team-slug
lastReviewed: 2026-06-19
---
```

---

## Writing style rules

1. **Imperative voice** — "Run `pnpm install`", not "You should run `pnpm install`"
2. **Steps are numbered** — all sequential instructions use numbered lists
3. **Commands are in code blocks** — never inline prose
4. **Gotchas get their own callout** — use `> **Note:**` or `> **Warning:**`
5. **No assumed knowledge** — link to prerequisites rather than assuming them
6. **One doc, one purpose** — don't merge setup and deploy into one file
7. **Keep troubleshooting current** — every resolved thread should result in a troubleshooting entry

---

## AI patch proposal rules

- AI proposes patches as unified diffs against specific sections
- Patches must carry: source thread ID, confidence score, agent ID, timestamp
- Patches must not modify sections with `aiEdit: false` in frontmatter
- Patches in `pending` status are not visible to end users
- Patches require human approval before publication
- Approved patches trigger re-indexing

---

## Troubleshooting entry format

```markdown
### [Error or symptom]

**Symptoms:** What the user sees
**Cause:** Why it happens
**Resolution:**
1. Step one
2. Step two

**Tags:** `vpn`, `windows`, `networking`  
**Source:** Thread #123 (2026-06-19)
```

---

## When to use

- Writing or reviewing any doc in `docs/` or app-linked documents
- Generating a doc skeleton for a new app
- Reviewing an AI patch proposal for correctness
- Writing troubleshooting entries from resolved threads
