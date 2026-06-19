# AGENTS.md — Project Atlas

> This file defines the AI agent system for Project Atlas. All agents operate under a **human-in-the-loop** governance model in the MVP. Agents may draft, propose, classify, and summarise. Humans approve canonical changes.

---

## Agent operating principles

1. **Draft, don't publish** — agents create proposals, not facts.
2. **Narrow scope** — each agent has one responsibility.
3. **Provenance always** — every output must carry source, confidence, and timestamp.
4. **Reversible** — all agent actions can be rolled back.
5. **Audit-logged** — every run is recorded in `agent_runs`.
6. **Risk-tiered** — autonomy level is determined by risk class, not convenience.
7. **Fail safe** — on error or low confidence, agents notify and halt rather than guess.

---

## MVP agent roster

### 1. Conversation Distiller Agent

**ID:** `agent.distiller`  
**Trigger:** Thread marked as resolved  
**Input:** Thread messages, thread metadata, linked entity context (app/pattern/doc)  
**Output:** Resolution summary, insight classification, confidence score, suggested tags  
**Autonomy:** Draft only — no write access to docs  
**Risk class:** Low

**Responsibilities:**
- Summarise resolved threads into a short human-readable resolution.
- Classify the insight type: `setup-fix`, `troubleshooting`, `prerequisite`, `faq`, `security-note`, `pattern-improvement`.
- Score confidence based on signal clarity (e.g. single clear answer = high, conflicting replies = low).
- Suggest target document and section for a patch proposal.
- Emit an `InsightCandidate` event if confidence >= threshold.

**Confidence thresholds:**
- >= 0.85 → auto-emit patch proposal for review
- 0.60–0.84 → emit with `needs-human-review` flag
- < 0.60 → notify thread owner only, no proposal

---

### 2. Doc Patch Agent

**ID:** `agent.doc-patcher`  
**Trigger:** `InsightCandidate` event from Distiller, or manual trigger by maintainer  
**Input:** Insight, target document, existing section content, style rules  
**Output:** `PatchProposal` with before/after diff, rationale, destination section, source refs  
**Autonomy:** Draft only — patch enters approval queue, never auto-publishes  
**Risk class:** Low–Medium

**Responsibilities:**
- Locate the correct section in the target document.
- Generate a minimal, accurate patch (prefer additive changes over rewrites).
- Attach provenance: source thread ID(s), source messages, agent confidence, generated timestamp.
- Follow the project documentation style guide (`docs/style-guide.md`).
- Flag if the target section is marked `no-ai-edit` in frontmatter.

**Hard constraints:**
- Must never modify sections with frontmatter flag `ai-edit: false`.
- Must never publish directly — all output goes to `patch_proposals` table with status `pending`.
- Must include full diff in proposal.

---

### 3. Indexing Agent

**ID:** `agent.indexer`  
**Trigger:** Document published, app created/updated, thread resolved, patch approved  
**Input:** Entity type + ID  
**Output:** Updated search index entries  
**Autonomy:** Fully automated — no approval required  
**Risk class:** Low

**Responsibilities:**
- Re-index affected entities in the search layer after any canonical change.
- Extract and update keyword + embedding vectors if semantic search is enabled.
- Invalidate stale cache entries.

---

## Post-MVP agents (planned)

| Agent | ID | Description | Risk class |
|---|---|---|---|
| Repo Scanner | `agent.repo-scanner` | Scans connected repos for issues, drift, missing docs, outdated deps | Medium |
| Fix Agent | `agent.fixer` | Opens PRs for low-risk findings (lint, dep bumps, small docs fixes) | Medium |
| Review Agent | `agent.reviewer` | Reviews PRs raised by humans or Fix Agent, posts structured comments | Medium |
| Merge Agent | `agent.merger` | Merges PRs that pass all policy checks and risk tier rules | High |
| Governance Agent | `agent.governance` | Checks every active app has owner, docs, runbook, and deployment metadata | Low |
| Repo Ingest Agent | `agent.ingestor` | Reads new repos and drafts architecture summaries and doc skeletons | Low |

---

## Risk tiers

| Tier | Description | Merge autonomy |
|---|---|---|
| Low | Docs, typos, links, lint, markdown | Auto-merge after CI pass |
| Medium | Dep bumps, refactors, test additions, non-critical bug fixes | Requires 1 human approval |
| High | Auth, infra, networking, secrets, billing, prod deployment logic | Requires 2 human approvals + security review |

---

## Agent communication model

Agents communicate via an **event queue** (job queue / pub-sub). They do not call each other directly.

```
Thread resolved
  → [Queue] insight.extraction.requested
    → Distiller Agent runs
      → [Queue] insight.candidate.created
        → Doc Patch Agent runs
          → PatchProposal created in DB
            → Notification sent to maintainer
              → Human approves/rejects
                → [Queue] doc.published / proposal.rejected
                  → Indexing Agent runs
```

---

## Agent output contracts

### InsightCandidate
```typescript
type InsightCandidate = {
  threadId: string;
  type: 'setup-fix' | 'troubleshooting' | 'prerequisite' | 'faq' | 'security-note' | 'pattern-improvement';
  summary: string;
  tags: string[];
  confidence: number; // 0.0 - 1.0
  suggestedDocId?: string;
  suggestedSection?: string;
  sourceMessageIds: string[];
  generatedAt: string; // ISO 8601
};
```

### PatchProposal
```typescript
type PatchProposal = {
  id: string;
  insightId: string;
  targetDocId: string;
  targetSection: string;
  diff: string; // unified diff format
  rationale: string;
  confidence: number;
  sourceThreadId: string;
  sourceMessageIds: string[];
  agentId: string;
  status: 'pending' | 'approved' | 'rejected' | 'superseded';
  createdAt: string;
};
```

### AgentRun
```typescript
type AgentRun = {
  id: string;
  agentId: string;
  triggerEvent: string;
  inputRef: string;
  outputRef?: string;
  status: 'running' | 'completed' | 'failed' | 'skipped';
  durationMs: number;
  error?: string;
  createdAt: string;
};
```

---

## Governance rules

- No agent may write to `documents` table directly. All writes go through `patch_proposals`.
- No agent may merge a PR without passing the policy engine check.
- All agent runs are immutable once created — status updates only.
- Agents must respect `ai-edit: false` frontmatter on any document section.
- Agent prompts and system instructions are version-controlled in `packages/agents/prompts/`.
- Model routing is defined in `packages/agents/model-config.ts` — agents do not hardcode model names.

---

## Adding a new agent

1. Define the agent contract in `packages/agents/contracts/`.
2. Implement the agent in `packages/agents/src/<agent-name>/`.
3. Register trigger events in `packages/agents/registry.ts`.
4. Add agent ID and risk class to this file.
5. Write unit tests in `packages/agents/src/<agent-name>/__tests__/`.
6. Update `SKILLS.md` if new tooling or MCP connections are required.
7. Open a PR — all agent changes require human review regardless of risk tier.
