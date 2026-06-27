# next-trpc-drizzle

A Claude Code plugin: an engineering playbook (as auto-discovered skills) for the **Next.js + tRPC + Drizzle** stack — the T3-style stack, extended with TanStack Query, shadcn/ui + Tailwind, Vitest, and Turborepo.

The skills steer a coding agent toward fast, correct, test-first feature work: a two-phase TDD pipeline plus standards for the backend, frontend, data fetching, testing, and CI.

## Skills

| Skill | Use when |
| --- | --- |
| `build-backend-feature` | Build a feature's backend end-to-end, test-first (Phase 1). Drives a living checklist until the backend suite is green. Pairs with `/goal`. |
| `build-frontend-feature` | Build the frontend + integration after the backend is done (Phase 2), test-first from a described UI. Pairs with `/goal`. |
| `tdd` | The Canon TDD loop — test list → one test → make it pass → refactor. |
| `backend-standards` | Layered architecture (tRPC router → service → repo → Drizzle), query/perf, transactions, error handling. |
| `frontend-standards` | shadcn/ui-first, semantic design tokens (no hardcoded values), mobile-first responsive incl. tablet. |
| `data-fetching` | tRPC + TanStack Query v5 + Next App Router: prefetch/hydrate, `useSuspenseQuery(ies)`, caching, optimistic updates, `loading.tsx`/`error.tsx`. |
| `testing` | Vitest projects, PGlite repo tests, MSW vs cache-seeding, behavior-not-implementation. |
| `ci` | Turborepo `--affected` test runs. |

Once installed, Claude loads each skill automatically when your task matches its description, or you can invoke it explicitly as `/next-trpc-drizzle:<skill>` (e.g. `/next-trpc-drizzle:data-fetching`).

## Install

```text
/plugin marketplace add Njunge11/next-trpc-drizzle
/plugin install next-trpc-drizzle@next-trpc-drizzle
```

(Replace `Njunge11/next-trpc-drizzle` with the GitHub repo path once pushed. To try it locally before publishing: `/plugin marketplace add /path/to/next-trpc-drizzle`.)

## License

MIT
