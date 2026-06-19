# Atlas Catalog Skill

**Trigger:** `/atlas-catalog`  
**Scope:** Use when working with catalog entities, domain schema, or data model questions

---

## Core entities

Use these exact names and field names everywhere. They are the canonical domain model.

### App

```typescript
type App = {
  id: string;           // uuid
  slug: string;         // url-safe, unique
  name: string;
  description: string;
  ownerTeamId: string;
  repoId?: string;
  patternId?: string;
  primaryEnvironmentId?: string;
  lifecycle: 'experimental' | 'active' | 'deprecated' | 'archived';
  runtime?: string;     // e.g. 'node', 'python', 'static'
  visibility: 'org' | 'team' | 'restricted';
  supportContact?: string;
  tags: string[];
  createdAt: string;
  updatedAt: string;
};
```

### Pattern

```typescript
type Pattern = {
  id: string;
  slug: string;
  name: string;
  description: string;
  costProfile: 'zero' | 'low' | 'medium' | 'high';
  scaleToZero: boolean;
  constraints: string[];
  templateRef?: string;
  tags: string[];
};
```

### Environment

```typescript
type Environment = {
  id: string;
  name: string;
  type: 'sandpit' | 'test' | 'staging' | 'production';
  costNotes?: string;
  constraints?: string[];
  provider?: string;    // e.g. 'azure'
};
```

### Document

```typescript
type Document = {
  id: string;
  title: string;
  entityType: 'app' | 'pattern' | 'environment' | 'global';
  entityId?: string;
  docType: 'overview' | 'setup' | 'deploy' | 'rollback' | 'troubleshooting' | 'cost' | 'ownership' | 'adr' | 'other';
  status: 'draft' | 'review' | 'published';
  content: string;     // Markdown/MDX
  aiEdit: boolean;     // false = AI cannot patch
  sourceRef?: string;  // git path if docs-as-code
  ownerId: string;
  createdAt: string;
  updatedAt: string;
};
```

### Thread

```typescript
type Thread = {
  id: string;
  title: string;
  entityType: 'app' | 'pattern' | 'environment' | 'document';
  entityId: string;
  status: 'open' | 'resolved';
  resolvedAt?: string;
  tags: string[];
  createdBy: string;
  createdAt: string;
};
```

---

## Mandatory app metadata

Before an app can be set to `lifecycle: 'active'`, it must have:

- `name`
- `slug`
- `ownerTeamId`
- All 7 required documents (see `/atlas-docs`)
- `visibility` set
- `supportContact` set

---

## Relationships

- App `uses` Pattern (many-to-one)
- App `owned_by` Team (many-to-one)
- App `sourced_from` Repository (many-to-one)
- App `deployed_to` Environment (many-to-many)
- Document `belongs_to` App | Pattern | Environment
- Thread `belongs_to` App | Pattern | Environment | Document
- Thread `yields` Insight (one-to-one)
- Insight `produces` PatchProposal (one-to-many)

---

## When to use

- Implementing catalog CRUD routes
- Designing queries that join catalog entities
- Validating entity shapes in service functions
- Explaining the domain model to new contributors
