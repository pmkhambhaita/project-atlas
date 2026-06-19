# Atlas UI Skill

**Trigger:** `/atlas-ui`  
**Scope:** Use when working in `apps/web/` or `packages/ui/`

---

## Stack

- **Framework:** Next.js (App Router) + React + TypeScript
- **Components:** shadcn/ui — always check here before building from scratch
- **Styling:** Tailwind CSS — utility-first, no custom CSS unless unavoidable
- **Data fetching:** TanStack Query (`useQuery`, `useMutation`) or Server Components
- **Forms:** React Hook Form + Zod resolvers
- **Icons:** Lucide React

---

## Component rules

1. Every component must handle 4 states: **default, loading, error, empty**.
2. Use shadcn/ui primitives first. Extend, don't replace.
3. WCAG 2.2 AA is the minimum. Every interactive element needs keyboard support and a visible focus state.
4. Dark mode support is required. Use Tailwind `dark:` variants.
5. No hardcoded colours — use design tokens from `packages/ui/tokens.ts`.
6. Components are co-located with their stories/tests.
7. No `useEffect` for data fetching. Use TanStack Query or Server Components.
8. Server components by default. Add `'use client'` only when interactivity requires it.

---

## Folder structure

```
apps/web/
  app/                  # Next.js App Router pages
    (portal)/           # Main portal layout
      catalog/          # Catalog pages
      docs/             # Docs pages
      threads/          # Thread pages
      admin/            # Admin pages
  components/           # App-specific components
  lib/                  # Client utilities

packages/ui/
  components/           # Shared components (shadcn base + extensions)
  tokens.ts             # Design tokens
  index.ts
```

---

## Key design principles

- **Information density over whitespace** — this is a dev tool, not a marketing site
- **Context always visible** — app name, owner, and pattern should be above the fold on every app page
- **Status is a first-class element** — every entity should show lifecycle/status clearly
- **Search is the primary nav** — keyboard-first search experience
- **AI provenance is visible** — AI-generated content is clearly badged, not disguised

---

## Required page states

For every new page or major component, implement:

- `Loading` — skeleton or spinner appropriate to content type
- `Error` — friendly message + retry action
- `Empty` — helpful prompt, not just "no data found"
- `Default` — the happy path

---

## When to use

- Building new portal pages in `apps/web/`
- Creating shared components in `packages/ui/`
- Reviewing UI code for accessibility or state coverage gaps
- Designing new screens before implementation
