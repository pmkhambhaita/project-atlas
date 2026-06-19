# ADR 0002 — AI Agent Architecture

**Date:** 2025-07-13  
**Status:** Accepted  
**Deciders:** Platform team  
**Tags:** ai, agents, architecture

---

## Context

Project Atlas is AI-native by design. The platform must support contextual chat anchored to catalog items and documentation pages, as well as the ability to propose, review, and apply doc patches autonomously or semi-autonomously. We need a principled approach to agent design that is safe, auditable, and incrementally autonomous.

Key requirements:
- Context-aware chat responses grounded in catalog/docs data
- AI-generated patch proposals with full provenance
- Tiered autonomy: not all agents act with equal authority
- Audit trail for every agent action
- Graceful degradation when models are unavailable or rate-limited

---

## Decision

We adopt a **multi-agent pipeline architecture** with the following named agents, each with a defined role, input, output, and risk tier.

### Agent Roster

| Agent | Role | Risk Tier | Autonomy |
|---|---|---|---|
| `ContextAgent` | Retrieves relevant catalog/docs context for a query | Low | Fully autonomous |
| `ConversationDistillerAgent` | Summarises threads into structured patch proposals | Low | Fully autonomous |
| `DocPatchAgent` | Drafts Markdown diffs against existing docs | Medium | Requires human review |
| `IndexingAgent` | Keeps search index in sync with doc/catalog changes | Low | Fully autonomous |
| `GovernanceAgent` | Evaluates patch risk tier and routes for approval | Medium | Autonomous classification, human approval |
| `PRAgent` | Opens GitHub PRs for approved patches | High | Requires explicit human approval |

### Risk Tiers

**Low** — Read-only or reversible. No human approval required.  
**Medium** — Writes draft content. Requires at least one human reviewer before merge.  
**High** — Creates external artefacts (PRs, issues, webhooks). Requires explicit approval from a maintainer.

### Agent Communication

Agents communicate via an internal **job queue** (BullMQ / Redis). Each job includes:

```ts
type AgentJob = {
  agentId: string
  traceId: string          // propagated across the pipeline
  input: Record<string, unknown>
  riskTier: 'low' | 'medium' | 'high'
  createdBy: string        // userId or 'system'
  createdAt: string
}
```

### Provenance Model

Every patch proposal carries a provenance record:

```ts
type PatchProvenance = {
  sourceThreadId: string
  sourceMessageIds: string[]
  generatedBy: string      // agent name + model version
  traceId: string
  humanReviewerId?: string
  approvedAt?: string
}
```

This provenance is stored in the database and included as a YAML front-matter comment in committed Markdown files.

### Context Retrieval Strategy

1. Query is embedded using the configured embedding model (default: `text-embedding-3-small`)
2. Top-K chunks retrieved from Postgres `pgvector` index (K=8 by default)
3. Chunks re-ranked by recency and catalog-item relevance score
4. Final context window assembled: system prompt + catalog metadata + top chunks + user query
5. Response streamed back to UI via SSE

### Guardrails

- Agents may not write directly to `main`; all patches go via PR
- Agents may not access secrets or environment variables beyond their declared scope
- All LLM calls are logged with token counts for cost tracking
- Rate limits per user: 20 AI requests/min (configurable)
- PII scrubbing applied before any content is sent to external LLM APIs

---

## Consequences

### Positive

- Clear separation of concerns between agents makes testing and replacement straightforward
- Risk tiers provide a principled escalation path without blocking low-risk automation
- Provenance model ensures full auditability and supports compliance requirements
- Queue-based design allows retry, dead-letter handling, and horizontal scaling

### Negative / trade-offs

- Redis adds infrastructure complexity (mitigated: single shared instance with tech-stack)
- Multi-agent pipelines are harder to debug than single-function calls; traceId propagation is essential
- Medium/High tier actions require human availability; async approval UX must be polished

### Neutral

- Model provider is abstracted behind a `LLMClient` interface; switching from OpenAI to Anthropic or a local model requires only a config change
- Agent roster is expected to grow; new agents follow the same job/provenance contract

---

## Alternatives considered

| Approach | Reason not chosen |
|---|---|
| Single monolithic AI endpoint | No separation of concerns; harder to apply per-action risk tiers |
| LangChain / LlamaIndex framework | Adds abstraction overhead; Atlas needs tight control over prompts and context assembly |
| Fully autonomous agents (no human approval) | Too risky for doc patch application in MVP; autonomy should be earned iteratively |
| Webhook-only (no queue) | No retry, no dead-letter, no backpressure — insufficient for production |
