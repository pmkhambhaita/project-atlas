# Product Requirements Document — Atlas MVP

**Version:** 0.1.0  
**Status:** Draft  
**Owner:** Platform team  
**Last updated:** 2025-07-13

---

## 1. Purpose

This document defines the requirements for the Atlas Minimum Viable Product (MVP). Atlas is an AI-native internal developer platform combining a Backstage-style service catalog with Confluence-style documentation, contextual AI chat, and AI-generated patch proposals.

The MVP validates the core loop: **register an app → attach docs → ask a question → AI proposes a doc update → human approves → change is committed**.

---

## 2. Problem statement

Engineering teams suffer from:

1. **Catalog sprawl** — services, APIs, and environments exist in spreadsheets, Notion pages, and tribal knowledge
2. **Stale documentation** — docs are written once and never updated as the system evolves
3. **Context loss** — decisions made in Slack threads or stand-ups are never captured in a discoverable place
4. **High onboarding friction** — new engineers spend days finding the right docs, owners, and runbooks

Atlas solves this by making the catalog and docs a living system, continuously updated by AI agents acting on real conversations.

---

## 3. Goals and non-goals

### Goals (MVP)

- [ ] Register catalog items (apps, APIs, libraries, environments, teams)
- [ ] Attach TechDocs-style Markdown documentation to catalog items
- [ ] Chat with AI in context of a specific catalog item or doc page
- [ ] AI proposes Markdown patch from a chat thread (with full provenance)
- [ ] Human reviews and approves/rejects patch proposals in the UI
- [ ] Approved patches committed to the repo via GitHub PR
- [ ] Basic search across catalog and docs (Postgres FTS)
- [ ] OIDC-based authentication (GitHub OAuth for MVP)
- [ ] Azure sandpit deployment via single command

### Non-goals (MVP)

- Autonomous PR creation without human approval
- Repo scanning and auto-catalog population
- Meilisearch integration
- Mobile-optimised UI
- Multi-tenant SaaS mode
- RBAC beyond owner/viewer roles
- Slack / Teams integration

---

## 4. Users and personas

| Persona | Description | Primary jobs-to-be-done |
|---|---|---|
| **Platform Engineer** | Owns Atlas deployment and catalog governance | Register services, maintain catalog schema, approve high-risk patches |
| **Backend Engineer** | Registers their services and writes runbooks | Find API docs quickly, update docs when things change |
| **New Joiner** | Onboarding to a new team or codebase | Discover what exists, understand architecture, ask questions without interrupting colleagues |
| **Tech Lead** | Responsible for architecture quality and doc health | Review AI patch proposals, ensure docs reflect reality |

---

## 5. User stories

### Catalog

- As a **Backend Engineer**, I can register a new service with a name, description, owner team, and tech stack tags so that it appears in the catalog.
- As a **Platform Engineer**, I can define catalog item types (App, API, Library, Environment, Team) so that the catalog schema is consistent.
- As a **New Joiner**, I can search the catalog by name, tag, or team so that I can find relevant services quickly.
- As a **Tech Lead**, I can see which catalog items have stale documentation (not updated in 30+ days) so that I can prioritise doc maintenance.

### Documentation

- As a **Backend Engineer**, I can create and edit Markdown documentation attached to a catalog item so that my runbooks and API guides live next to the service definition.
- As any user, I can navigate from a catalog item to its documentation in one click.
- As any user, I can see the last-updated date and editor for any doc page.

### AI Chat

- As any user, I can open a chat thread anchored to a catalog item or doc page and ask questions in natural language.
- As any user, I receive AI responses grounded in the catalog metadata and doc content for that item.
- As any user, I can see which sources (doc chunks) the AI used to generate its response.

### Patch proposals

- As a **Tech Lead**, I can trigger the AI to summarise a chat thread into a doc patch proposal.
- As a **Tech Lead**, I can review a diff view of the proposed changes before approving.
- As a **Tech Lead**, I can approve a patch, which opens a GitHub PR containing the changes and provenance metadata.
- As a **Tech Lead**, I can reject a patch with a comment, which closes the proposal and notifies the requester.
- As a **Platform Engineer**, I can see an audit log of all patch proposals, approvals, and rejections.

### Authentication

- As any user, I can sign in with my GitHub account via OAuth.
- As a **Platform Engineer**, I can restrict access to specific GitHub organisations or teams.

---

## 6. Functional requirements

### 6.1 Catalog

| ID | Requirement | Priority |
|---|---|---|
| CAT-01 | Catalog items have: name, slug, type, description, owner (team), tags, status, createdAt, updatedAt | Must |
| CAT-02 | Supported item types: App, API, Library, Environment, Team | Must |
| CAT-03 | Items can be created, read, updated, and deleted via API and UI | Must |
| CAT-04 | Catalog search by name, tag, type, and owner using Postgres FTS | Must |
| CAT-05 | Each catalog item has a canonical URL slug | Must |
| CAT-06 | Items display "last updated" and link to most recent doc page | Should |
| CAT-07 | Import catalog items from a YAML manifest file | Could |

### 6.2 Documentation

| ID | Requirement | Priority |
|---|---|---|
| DOC-01 | Doc pages are Markdown, stored in the database with version history | Must |
| DOC-02 | Each doc page belongs to a catalog item | Must |
| DOC-03 | Doc pages render in a read view with syntax highlighting | Must |
| DOC-04 | Doc pages have an inline edit mode (Markdown editor) | Must |
| DOC-05 | Doc changes create a new version (append-only history) | Must |
| DOC-06 | Doc pages display provenance metadata when AI-generated | Must |
| DOC-07 | Doc pages are indexed in Postgres for FTS and pgvector for semantic search | Must |

### 6.3 AI chat

| ID | Requirement | Priority |
|---|---|---|
| AI-01 | Chat is anchored to a catalog item or doc page | Must |
| AI-02 | Chat context includes catalog metadata + top-8 doc chunks by semantic similarity | Must |
| AI-03 | AI responses stream via SSE | Must |
| AI-04 | Chat messages are persisted in a thread | Must |
| AI-05 | Source chunks cited in each AI response are displayed | Must |
| AI-06 | Users can rate AI responses (thumbs up/down) | Should |
| AI-07 | Rate limit: 20 AI requests/user/minute | Must |

### 6.4 Patch proposals

| ID | Requirement | Priority |
|---|---|---|
| PATCH-01 | Users can trigger a patch proposal from a thread | Must |
| PATCH-02 | Patch proposal shows a Markdown diff (before/after) | Must |
| PATCH-03 | Patch proposal includes provenance: source thread ID, message IDs, agent version | Must |
| PATCH-04 | Approving a patch opens a GitHub PR with the diff and provenance YAML | Must |
| PATCH-05 | Rejecting a patch archives the proposal with a rejection comment | Must |
| PATCH-06 | Patch proposals have status: draft, pending_review, approved, rejected, merged | Must |
| PATCH-07 | Audit log records all proposal lifecycle events with actor and timestamp | Must |

---

## 7. Non-functional requirements

| Category | Requirement |
|---|---|
| Performance | Catalog search returns in < 200ms (p95) |
| Performance | AI streaming first token latency < 1.5s (p95) |
| Performance | Doc page load < 500ms (p95) |
| Reliability | API uptime > 99% in sandpit; no SLA in MVP |
| Security | All endpoints require authentication |
| Security | No PII sent to external LLM APIs without scrubbing |
| Security | Secrets managed via Azure Key Vault / environment variables; never in code |
| Observability | Structured JSON logging with traceId on all API requests |
| Observability | LLM token usage logged per request for cost tracking |
| Cost | Sandpit total cost < £50/month |

---

## 8. MVP success metrics

| Metric | Target | Measurement method |
|---|---|---|
| Time to register a new app in catalog | < 5 minutes | Manual test with new engineer |
| Time for first AI patch proposal end-to-end | < 15 minutes from chat to PR | Manual test |
| Patch proposal acceptance rate (human-approved) | > 60% | Audit log analysis |
| Doc search result relevance (top-1 accuracy) | > 70% on test query set | Offline eval |
| New engineer onboarding time to find runbook | < 3 minutes | Manual test |

---

## 9. Phased delivery

### Phase 1 — Foundation (Weeks 1–2)
- Monorepo scaffold, CI, lint, typecheck, test pipeline
- Database schema: catalog items, doc pages, users
- Auth: GitHub OAuth via Better Auth
- Catalog CRUD API + basic UI

### Phase 2 — Docs + Search (Weeks 3–4)
- Doc page CRUD API + Markdown editor UI
- Postgres FTS for catalog + docs
- pgvector embedding pipeline (IndexingAgent)

### Phase 3 — AI Chat (Weeks 5–6)
- ContextAgent + LLMClient
- Chat UI with SSE streaming
- Source citation display

### Phase 4 — Patch Proposals (Weeks 7–8)
- ConversationDistillerAgent + DocPatchAgent
- Patch proposal UI (diff view, approve/reject)
- GovernanceAgent risk classification
- PRAgent + GitHub PR creation
- Audit log UI

### Phase 5 — Hardening + Sandpit (Weeks 9–10)
- Azure sandpit Terraform module
- `atlas sandpit:create` CLI command
- Cost budget alerts
- Performance testing and observability
- MVP acceptance criteria sign-off

---

## 10. Open questions

| # | Question | Owner | Due |
|---|---|---|---|
| 1 | Should patch proposals auto-merge if no reviewer acts within 72h? | Platform team | Phase 4 |
| 2 | Which embedding model to use for semantic search? | Platform team | Phase 2 |
| 3 | Should the catalog support custom fields per item type? | Platform team | Phase 1 |
| 4 | What is the GitHub org/team restriction strategy for auth? | Platform team | Phase 1 |
