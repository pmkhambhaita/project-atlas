# Atlas Agents Skill

**Trigger:** `/atlas-agents`  
**Scope:** Use when working in `packages/agents/` or designing new agent workflows

---

## What this skill does

This skill gives Claude deep context on the Project Atlas agent system so it produces agent code, prompts, and contracts that are compliant with the governance model from the first prompt.

---

## Agent development rules

1. Every agent has a single responsibility. If you need two things done, create two agents.
2. Agents communicate via the event queue. They never call each other directly.
3. Every agent run must create an `AgentRun` record before doing any work.
4. All doc changes go to `patch_proposals`, never to `documents` directly.
5. Prompts live in `packages/agents/prompts/<agent-name>.md`. Never inline them.
6. Model names come from `packages/agents/model-config.ts`. Never hardcode.
7. Every agent output type must have a corresponding Zod schema in `packages/agents/contracts/`.
8. Confidence scores are required on all classification and extraction outputs (0.0–1.0).
9. Agents must handle errors by logging to `AgentRun.error` and halting cleanly.
10. Agents respect `ai-edit: false` in document frontmatter.

---

## Agent file structure

```
packages/agents/
  contracts/          # Zod schemas for agent inputs/outputs
  prompts/            # Prompt templates (Markdown)
  src/
    distiller/        # Conversation Distiller Agent
    doc-patcher/      # Doc Patch Agent
    indexer/          # Indexing Agent
  model-config.ts     # Model routing config
  registry.ts         # Event-to-agent bindings
  index.ts
```

---

## Event names (use exactly)

```
insight.extraction.requested
insight.candidate.created
doc.patch.requested
doc.patch.proposed
doc.published
proposal.approved
proposal.rejected
index.requested
```

---

## Confidence thresholds

| Score | Action |
|---|---|
| >= 0.85 | Auto-emit for review |
| 0.60–0.84 | Emit with `needs-human-review` flag |
| < 0.60 | Notify thread owner only, no proposal |

---

## When to use

- Implementing a new agent in `packages/agents/src/`
- Reviewing existing agent code
- Designing event flows between agents
- Writing agent prompt templates
- Adding agent output Zod schemas
