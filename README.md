# next-trpc-drizzle

A Claude Code plugin: an engineering playbook (as auto-discovered skills) for the **Next.js + tRPC + Drizzle** stack — the T3-style stack, extended with TanStack Query, shadcn/ui + Tailwind, Vitest, and Turborepo.

The skills steer a coding agent toward fast, correct, test-first feature work: a two-phase TDD pipeline plus standards for the backend, frontend, data fetching, testing, and CI.

## Skills

| Skill | Use when |
| --- | --- |
| `backend-checklist` | Turn an agreed backend design into the `## Backend` test-case checklist (Phase 1 input). |
| `build-backend-feature` | Build a feature's backend end-to-end, test-first (Phase 1). Drives the living checklist until the backend suite is green. Pairs with `/goal`. |
| `frontend-checklist` | Turn an agreed UI into the `## Frontend + Integration` checklist — test-backed behavior vs browser-checked visual (Phase 2 input). |
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

## Workflow

A **two-phase, test-first pipeline**. Each phase follows the same shape — *agree the design → author a checklist → drive the build to green* — and you stay in control at every handoff. The `*-standards`, `tdd`, `data-fetching`, and `testing` skills are always-on references the build phases pull in automatically.

### Phase 1 — Backend

1. **Discuss** the feature's data, mutations, and architecture in chat.
2. **Author the checklist** — invoke `backend-checklist`. It turns the agreed design into the `## Backend` section of `features/<name>/checklist.md` (one observable behavior per line). Review and amend it — this checklist is the definition of done.
3. **Build** — drive the phase autonomously with `/goal` (see below):

   ```
   /goal implement features/<name>/checklist.md using build-backend-feature —
   every "## Backend" item checked [x] and `vitest --project backend` green,
   paste the run as proof. Stop after 25 turns.
   ```

   `build-backend-feature` runs the TDD loop (`tdd`, `backend-standards`, `testing`) until the suite is green.
4. **Review the backend** before moving on — it's a deliberate gate, not an automatic roll into the UI.

### Phase 2 — Frontend + integration

5. **Describe the UI**, then invoke `frontend-checklist` to author the `## Frontend + Integration` section — **Behavior** (test-backed) vs **Visual & responsive** (browser-checked). Review and amend.
6. **Build** with `/goal` + `build-frontend-feature` — behavior items only, since the judge can't see layout:

   ```
   /goal implement features/<name>/checklist.md using build-frontend-feature —
   every BEHAVIOR item checked [x] and `vitest` green, paste the run. Stop after 30 turns.
   ```

7. **Do the visual/responsive pass yourself** in the browser at ~375 / ~768 / ~1280px.

> **About `/goal`:** a Claude Code harness command (v2.1.139+) **you** run to keep the agent working autonomously until a condition holds — it loops after each turn until satisfied or you `/goal clear`. Its evaluator only reads the session transcript; it can't open files or run commands, so the condition must be something the agent **proves in its output**. That's why the build skills surface the checklist state and paste the test run.

## License

MIT
