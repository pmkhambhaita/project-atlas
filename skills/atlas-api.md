# Atlas API Skill

**Trigger:** `/atlas-api`  
**Scope:** Use when working in `apps/api/` or defining API contracts

---

## Stack

- **Framework:** Fastify v4 + TypeScript
- **Validation:** Zod (all inputs and outputs)
- **ORM:** Drizzle ORM
- **Auth:** JWT from Better Auth / Auth.js (validated on every request)
- **Error format:** Shared shape from `packages/domain/errors.ts`

---

## Route conventions

1. Every route has a Zod schema for: `params`, `querystring`, `body`, `reply`
2. No business logic in route handlers — delegate to service functions in `src/services/`
3. Use `fastify.withTypeProvider<ZodTypeProvider>()` — full type safety through
4. HTTP status codes: 200 success, 201 created, 400 validation error, 401 unauthenticated, 403 forbidden, 404 not found, 409 conflict, 422 unprocessable, 500 server error
5. All list endpoints support `limit` + `cursor` pagination. No offset pagination.
6. All timestamps in responses are ISO 8601 UTC strings

---

## Error response shape

```typescript
// packages/domain/errors.ts
type ApiError = {
  error: {
    code: string;       // machine-readable e.g. 'NOT_FOUND', 'VALIDATION_ERROR'
    message: string;    // human-readable
    details?: unknown;  // optional field-level details
    requestId: string;  // for log correlation
  };
};
```

---

## Route file structure

```
apps/api/src/
  routes/
    apps/
      index.ts          # GET /apps, POST /apps
      [id].ts           # GET /apps/:id, PATCH /apps/:id
    docs/
    threads/
    patterns/
    search/
    agent-runs/
  services/             # Business logic
  plugins/              # Fastify plugins (auth, db, etc.)
  index.ts
```

---

## Auth rules

- Every route requires authentication unless marked `{ auth: false }` in schema
- Permission checks happen in service layer, not route handlers
- Use `request.user` (typed) for the authenticated user
- Never trust client-provided user IDs for ownership checks

---

## Database rules

- Use Drizzle query builder only — no raw SQL strings
- All DB access goes through service functions — never in route handlers
- Use transactions for multi-table writes
- Timestamps: `timestamptz` in DB, ISO 8601 UTC strings in API responses

---
 
## When to use

- Adding new API routes in `apps/api/src/routes/`
- Writing service functions in `apps/api/src/services/`
- Reviewing API code for schema, auth, or error handling gaps
- Defining request/response types for new features
