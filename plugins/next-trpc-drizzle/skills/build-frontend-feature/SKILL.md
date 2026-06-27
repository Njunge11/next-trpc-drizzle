---
name: build-frontend-feature
description: Use to build the FRONTEND and integration of a feature after its backend is done (Phase 2) test-first from a described UI — extend the checklist, write UI tests, build with shadcn and tokens and responsive incl. tablet, then wire to the real router. Next.js + TanStack Query + shadcn/Tailwind. Pairs with /goal.
---

# Feature Build — Frontend + Integration (Phase 2)

The frontend half plus wiring it to the real backend, built test-first. **Start only after the backend phase is green and you've reviewed it** (the build-backend-feature skill).

References: tdd · frontend-standards · data-fetching · testing.

## The loop

0. **Restate the UI.** You'll usually get the UI as prose — break it into components.
1. **Extend the checklist** → same `features/<name>/checklist.md`, under `## Frontend + Integration`. Cases for rendering, interactions, **responsive incl. a `md:` tablet state**, loading/error, and the integrated data path. Unchecked.
2. **One case → one test** (the testing skill; MSW at the network edge or seed the cache).
3. **Build to pass** — frontend-standards: shadcn primitives, semantic tokens (no hardcoded values), mobile-first with a deliberate `md:` state.
4. **Tick the item `[x]`**; refactor.
5. **Integrate** — wire the real tRPC/TanStack Query path (data-fetching): server `prefetch` + `HydrateClient`, client `useSuspenseQuery` with the same `queryOptions`, `loading.tsx` (skeleton matching layout) + `error.tsx`. Make the integrated tests pass against the **real** router.
6. Repeat until **every `## Frontend + Integration` item is checked and the UI suite is green**.

## Finish line — `/goal`

The evaluator only sees the transcript, so keep the condition test-backed (paste the run); the responsive/token rules are enforced by the checklist items + frontend-standards, not by the judge. Run with auto mode:

```
/goal Every item under "## Frontend + Integration" in
features/<name>/checklist.md is checked [x], and `vitest --project ui`
passes — paste the run output showing all tests green. Do not change any backend
files (features/<name>/api, features/<name>/db). Stop after 30 turns if not met.
```

## Rules

- shadcn-first; semantic tokens only; every responsive layout has an explicit `md:` (tablet) state; verify at ~375 / ~768 / ~1280.
- Assert what the user sees/does, never component internals; mock only the network edge or seed the cache.
- **Don't reopen the backend.** If the API contract is wrong, stop and flag it — that's your gate, not a silent backend edit.
