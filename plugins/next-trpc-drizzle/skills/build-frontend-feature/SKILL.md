---
name: build-frontend-feature
description: Use to build the FRONTEND and integration of a feature test-first — Phase 2, after the backend is green. From a described UI to UI tests, shadcn build, and wiring the real router. Next.js + TanStack Query + shadcn/Tailwind.
---

# Feature Build — Frontend + Integration (Phase 2)

The frontend half of a feature plus wiring it to the real backend, built test-first.

References: frontend-checklist · tdd · frontend-standards · data-fetching · testing.

## The loop

0. **Restate the UI → component tree.** You'll usually get the UI as prose. Decompose it by responsibility and repetition into a tree of **shadcn components** (frontend-standards): pick the right primitives and compose them with their variants / slots / `asChild`. Don't build one monolith, and don't author from scratch what shadcn already provides.
1. **Have the `## Frontend + Integration` checklist** in `features/<name>/checklist.md` — author it with frontend-checklist if you haven't already. It splits into **Behavior** (test-backed) and **Visual & responsive** (browser-checked, never jsdom tests); that split drives the rest of this loop.
2. **Behavior → TDD it.** One case → one test (the testing skill; MSW at the network edge or seed the cache) → build to pass (frontend-standards: shadcn primitives, semantic tokens) → tick `[x]` → refactor.
3. **Integrate** — first **find the feature's backend API surface**: its router and procedures under `features/<name>/api` and the input/output types they expose. Wire against that actual contract — don't invent the shape. Then the real tRPC/TanStack Query path (data-fetching): server `prefetch` + `HydrateClient`, client `useSuspenseQuery` with the **same `queryOptions`** the backend exposes, `loading.tsx` (skeleton matching layout) + `error.tsx`. Make the integrated **behavior** tests pass against the **real** router.
4. **Visual & responsive.** Build the layout mobile-first with a deliberate `md:` state per frontend-standards; verify in the browser at ~375 / ~768 / ~1280.
5. **Done** when every **behavior** item is checked and `vitest` is green. Then **surface the proof in the transcript** — show the checklist with all behavior items `[x]` and paste the `vitest` run showing green. A transcript-only watcher (e.g. `/goal`) can't open the files; it only sees what you surface.

## Rules

- shadcn-first; semantic tokens only (no hardcoded values); mobile-first with an explicit `md:` (tablet) state.
- Behavior tests assert what the user sees/does, never component internals; mock only the network edge or seed the cache.
- **Visual/responsive correctness is a browser check, not a jsdom test** — don't write a Vitest test that pretends to assert layout.
- Hit a real backend/contract bug while integrating? Fix it — add or amend the backend tests alongside the change so the whole suite stays green.
