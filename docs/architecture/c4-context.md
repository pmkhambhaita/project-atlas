# Architecture — C4 Model

This document describes the Atlas system architecture using the [C4 model](https://c4model.com/): Context, Containers, Components, and Code. Diagrams are written in [Mermaid](https://mermaid.js.org/) for inline rendering on GitHub.

---

## Level 1 — System Context

> Who uses Atlas and what external systems does it interact with?

```mermaid
C4Context
  title System Context — Project Atlas

  Person(engineer, "Engineer", "Registers services, writes docs, asks questions")
  Person(techlead, "Tech Lead", "Reviews and approves AI patch proposals")
  Person(newjoiner, "New Joiner", "Discovers services and reads documentation")

  System(atlas, "Project Atlas", "AI-native internal developer platform: catalog + docs + contextual AI chat + patch proposals")

  System_Ext(github, "GitHub", "Source of truth for code; OAuth provider; PR target")
  System_Ext(llm, "LLM Provider", "OpenAI / Anthropic — generates AI responses and patch proposals")
  System_Ext(azure, "Microsoft Azure", "Cloud hosting: Container Apps, PostgreSQL, Redis, Blob Storage, Key Vault")

  Rel(engineer, atlas, "Registers catalog items, writes docs, chats with AI")
  Rel(techlead, atlas, "Reviews patch proposals, approves/rejects changes")
  Rel(newjoiner, atlas, "Searches catalog, reads docs, asks questions")
  Rel(atlas, github, "OAuth login, opens PRs for approved patches")
  Rel(atlas, llm, "Sends context + query, receives streamed response")
  Rel(atlas, azure, "Runs on Azure Container Apps, stores data in PostgreSQL + Redis")
```

---

## Level 2 — Container Diagram

> What deployable units make up Atlas?

```mermaid
C4Container
  title Container Diagram — Project Atlas

  Person(user, "User", "Engineer, Tech Lead, or New Joiner")

  Container(ui, "Web UI", "Next.js 14 / React", "Serves the catalog browser, doc editor, chat interface, and patch review UI")
  Container(api, "API Server", "Fastify / TypeScript", "REST + SSE endpoints for catalog, docs, chat, patches, and auth")
  Container(workers, "Background Workers", "BullMQ / TypeScript", "Runs AI agent jobs: indexing, patch drafting, PR creation")
  ContainerDb(db, "PostgreSQL", "PostgreSQL 16 + pgvector", "Stores catalog items, doc pages, chat threads, patch proposals, audit log; vector index for semantic search")
  ContainerDb(cache, "Redis", "Redis 7", "Job queue (BullMQ), rate limiting, session cache")
  Container(storage, "Blob Storage", "Azure Blob Storage", "Stores doc attachments and patch diffs")

  System_Ext(github, "GitHub", "OAuth + PR API")
  System_Ext(llm, "LLM Provider", "OpenAI / Anthropic")

  Rel(user, ui, "Uses", "HTTPS")
  Rel(ui, api, "API calls", "HTTPS / SSE")
  Rel(api, db, "Reads/writes", "Drizzle ORM")
  Rel(api, cache, "Queue jobs, rate limit", "Redis protocol")
  Rel(api, github, "OAuth, PR creation", "HTTPS")
  Rel(api, llm, "Chat completions", "HTTPS")
  Rel(workers, db, "Reads/writes", "Drizzle ORM")
  Rel(workers, cache, "Dequeues jobs", "BullMQ")
  Rel(workers, llm, "Embedding + patch generation", "HTTPS")
  Rel(workers, storage, "Stores diffs", "Azure SDK")
  Rel(workers, github, "Opens PRs", "HTTPS")
```

---

## Level 3 — Component Diagram (API Server)

> What are the major components inside the API server?

```mermaid
C4Component
  title Component Diagram — API Server

  Container_Boundary(api, "API Server") {
    Component(authRouter, "Auth Router", "Fastify plugin", "GitHub OAuth flow via Better Auth; issues session tokens")
    Component(catalogRouter, "Catalog Router", "Fastify plugin", "CRUD for catalog items; search via Postgres FTS")
    Component(docsRouter, "Docs Router", "Fastify plugin", "CRUD for doc pages; version history; provenance display")
    Component(chatRouter, "Chat Router", "Fastify plugin + SSE", "Creates threads, streams AI responses, persists messages")
    Component(patchRouter, "Patch Router", "Fastify plugin", "Patch proposal lifecycle: create, review, approve, reject")
    Component(contextAgent, "ContextAgent", "TypeScript service", "Retrieves semantic context chunks for a query from pgvector")
    Component(llmClient, "LLMClient", "TypeScript service", "Abstracts OpenAI / Anthropic; handles streaming, retries, token logging")
    Component(jobQueue, "Job Queue Client", "BullMQ", "Enqueues background agent jobs with traceId propagation")
    Component(auditLog, "Audit Logger", "TypeScript service", "Writes structured audit events for all patch lifecycle actions")
  }

  Rel(authRouter, db, "Read/write users and sessions")
  Rel(catalogRouter, db, "CRUD catalog items")
  Rel(docsRouter, db, "CRUD doc pages + versions")
  Rel(chatRouter, contextAgent, "Fetch context chunks")
  Rel(chatRouter, llmClient, "Stream chat response")
  Rel(chatRouter, db, "Persist messages")
  Rel(patchRouter, jobQueue, "Enqueue patch/PR jobs")
  Rel(patchRouter, db, "Read/write patch proposals")
  Rel(patchRouter, auditLog, "Log lifecycle events")
  Rel(contextAgent, db, "pgvector similarity search")
```

---

## Level 3 — Component Diagram (Background Workers)

> What agents run as background workers?

```mermaid
C4Component
  title Component Diagram — Background Workers

  Container_Boundary(workers, "Background Workers") {
    Component(indexAgent, "IndexingAgent", "BullMQ worker", "Chunks doc pages, generates embeddings, upserts to pgvector")
    Component(distillerAgent, "ConversationDistillerAgent", "BullMQ worker", "Summarises chat thread into a structured patch proposal draft")
    Component(patchAgent, "DocPatchAgent", "BullMQ worker", "Generates Markdown diff against the current doc page")
    Component(govAgent, "GovernanceAgent", "BullMQ worker", "Classifies patch risk tier; routes for appropriate approval flow")
    Component(prAgent, "PRAgent", "BullMQ worker", "Creates GitHub PR with diff and provenance YAML front-matter")
  }

  Rel(indexAgent, db, "Upsert embeddings", "pgvector")
  Rel(indexAgent, llm, "Generate embeddings", "HTTPS")
  Rel(distillerAgent, db, "Read thread messages, write proposal draft")
  Rel(distillerAgent, llm, "Summarise thread", "HTTPS")
  Rel(patchAgent, db, "Read doc page, write patch diff")
  Rel(patchAgent, llm, "Generate Markdown patch", "HTTPS")
  Rel(govAgent, db, "Read proposal, write risk classification")
  Rel(prAgent, github, "Create PR", "GitHub API")
  Rel(prAgent, db, "Update proposal status to approved/PR-opened")
  Rel(prAgent, auditLog, "Log PR creation event")
```

---

## Data flow — AI patch proposal (happy path)

```mermaid
sequenceDiagram
  actor User
  participant UI
  participant API
  participant Queue as Job Queue
  participant Distiller as ConversationDistillerAgent
  participant PatchAgent as DocPatchAgent
  participant Gov as GovernanceAgent
  participant PR as PRAgent
  participant GitHub

  User->>UI: Click "Propose patch from thread"
  UI->>API: POST /patches {threadId, docPageId}
  API->>Queue: Enqueue distill job {threadId, traceId}
  API-->>UI: 202 Accepted {proposalId}

  Queue->>Distiller: Process distill job
  Distiller->>API: Read thread messages
  Distiller->>LLM: Summarise thread
  Distiller->>DB: Write proposal draft {status: draft}

  Queue->>PatchAgent: Process patch job
  PatchAgent->>DB: Read current doc page
  PatchAgent->>LLM: Generate Markdown diff
  PatchAgent->>DB: Write patch diff {status: pending_review}

  Queue->>Gov: Process governance job
  Gov->>DB: Classify risk tier
  Gov->>DB: Update proposal {riskTier, status: pending_review}
  Gov->>API: Notify reviewer (in-app)

  User->>UI: Review diff, click Approve
  UI->>API: POST /patches/{id}/approve
  API->>Queue: Enqueue PR job {proposalId, traceId}
  API->>DB: Update proposal {status: approved}

  Queue->>PR: Process PR job
  PR->>GitHub: Create PR with diff + provenance YAML
  PR->>DB: Update proposal {status: pr-opened, prUrl}
  PR->>AuditLog: Log PR created event

  UI-->>User: Show PR link
```

---

## Deployment architecture (Azure sandpit)

```mermaid
graph TB
  subgraph Azure Resource Group [atlas-sandpit-rg]
    subgraph ACA [Azure Container Apps Environment]
      UI[Web UI\nNext.js Static / ACA]
      API[API Server\nFastify ACA]
      Workers[Background Workers\nBullMQ ACA Jobs]
    end
    DB[(PostgreSQL Flexible Server\nB1ms burstable)]
    Redis[(Azure Cache for Redis\nBasic C0)]
    ACR[Azure Container Registry\nBasic]
    Blob[Azure Blob Storage\nLRS hot]
    KV[Azure Key Vault\nManaged Identity]
  end

  GH[GitHub\nOAuth + PR API] -->|HTTPS| API
  LLM[LLM Provider\nOpenAI / Anthropic] -->|HTTPS| API
  LLM -->|HTTPS| Workers

  UI --> API
  API --> DB
  API --> Redis
  Workers --> DB
  Workers --> Redis
  Workers --> Blob
  Workers --> GH
  API --> KV
  Workers --> KV
  ACR --> ACA
```
