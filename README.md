# next-trpc-drizzle

A Claude Code plugin: a test-first engineering playbook for the **Next.js + tRPC + Drizzle** stack (the T3-style stack, plus TanStack Query, shadcn/ui + Tailwind, and Vitest). It ships **9 skills** that steer a coding agent through a two-phase TDD feature pipeline and the standards for backend, frontend, data fetching, and testing.

## Install

In Claude Code, add this repo as a marketplace, install the plugin, then reload:

```text
/plugin marketplace add Njunge11/next-trpc-drizzle
/plugin install next-trpc-drizzle@next-trpc-drizzle
/reload-plugins
```

Needs a recent Claude Code (the `/plugin` command). To try it locally without installing, run `claude --plugin-dir ./plugins/next-trpc-drizzle`.

## Use

Once installed, each skill loads **automatically** when your task matches its description. You can also invoke any skill explicitly by its namespaced name:

```text
/next-trpc-drizzle:<skill>      # e.g. /next-trpc-drizzle:data-fetching
```

| Skill | Use when |
| --- | --- |
| `backend-checklist` | You've agreed a backend design and want it turned into the `## Backend` test-case checklist (Phase 1 input). |
| `build-backend-feature` | Building a feature's backend end-to-end, test-first (Phase 1) — drives the checklist to a green backend suite. |
| `frontend-checklist` | You've agreed a UI and want the `## Frontend + Integration` checklist — test-backed behavior vs browser-checked visual (Phase 2 input). |
| `build-frontend-feature` | Building the frontend + integration after the backend is done (Phase 2), test-first from a described UI. |
| `tdd` | Running the Canon TDD loop — test list → one test → make it pass → refactor. |
| `backend-standards` | Writing or reviewing backend code — layered tRPC → service → repository → Drizzle, queries, transactions, errors. |
| `frontend-standards` | Building or reviewing UI — shadcn/ui composition, semantic tokens, mobile-first responsive incl. tablet. |
| `data-fetching` | tRPC + TanStack Query v5 + Next App Router — prefetch/hydrate, `useSuspenseQuery`, caching, optimistic updates, `loading.tsx`/`error.tsx`. |
| `testing` | Configuring or writing tests — Vitest projects, PGlite repo tests, MSW vs cache-seeding, behavior-not-implementation. |

## Workflow

A **two-phase, test-first pipeline**. Each phase follows the same shape — *agree the design → author a checklist → drive the build to green* — and you stay in control at every handoff. The `*-standards`, `tdd`, `data-fetching`, and `testing` skills are always-on references the build phases pull in automatically.

### Phase 1 — Backend

1. **Discuss** the feature's data, mutations, and architecture in chat.
2. **Author the checklist** — invoke `backend-checklist`. It turns the agreed design into the `## Backend` section of `features/<name>/checklist.md` (one observable behavior per line). Review and amend it — this checklist is the definition of done.
3. **Build** — drive the phase autonomously with `/goal`:

   ```
   /goal implement features/<name>/checklist.md using build-backend-feature
   ```

   The prompt stays short because the skill carries the finish line: `build-backend-feature` runs the TDD loop (`tdd`, `backend-standards`, `testing`), and on completion surfaces the checklist (all items `[x]`) plus the green `vitest --project backend` run for `/goal` to verify.
4. **Review the backend** before moving on — it's a deliberate gate, not an automatic roll into the UI.

### Phase 2 — Frontend + integration

5. **Describe the UI**, then invoke `frontend-checklist` to author the `## Frontend + Integration` section — **Behavior** (test-backed) vs **Visual & responsive** (browser-checked). Review and amend.
6. **Build** with `/goal` + `build-frontend-feature` (behavior items only — the judge can't see layout):

   ```
   /goal implement features/<name>/checklist.md using build-frontend-feature
   ```

7. **Do the visual/responsive pass yourself** in the browser at ~375 / ~768 / ~1280px.

> **About `/goal`:** a Claude Code harness command (v2.1.139+) **you** run to keep the agent working autonomously until a condition holds — it loops after each turn until satisfied or you `/goal clear`. Its evaluator only reads the session transcript; it can't open files or run commands, so the condition must be something the agent **proves in its output**. The build skills handle that for you — they surface the checklist state and paste the test run — which is why the prompt can stay this short. Append a cap like `… or stop after 25 turns` if you want a hard turn limit.

## License

MIT
