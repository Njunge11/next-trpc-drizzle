---
name: build-backend-feature
description: Use to build the BACKEND of a feature test-first — Phase 1, before any frontend. Drives a living checklist + TDD loop to a green backend suite. tRPC + Drizzle + Vitest/PGlite.
---

# Feature Build — Backend (Phase 1)

The backend half of a feature, built test-first. This is a **standalone phase**: finish and review it before touching the frontend (the build-frontend-feature skill is a separate phase with its own manual gate).

References: backend-checklist · tdd · backend-standards · testing · data-fetching.

## The loop

0. **Decide the feature.** What data points / mutations it needs; one aggregating endpoint vs several.
1. **Have the `## Backend` checklist** at `features/<name>/checklist.md` — author it with backend-checklist if you haven't already. It's the **living test list**: the version-controlled artifact you burn down here.
2. **Pick one unchecked case** → write one real test for it (setup / invoke / assert; see the testing skill).
3. **Make it pass** — minimum code, layer order db → service → router, backend-standards.
4. **Refactor** if needed, then **tick the item `[x]`**. Add any newly-discovered cases to the list.
5. Repeat until **every `## Backend` item is checked and the backend suite is green**. Then **surface the proof in the transcript** — show the checklist with all `## Backend` items `[x]` and paste the `vitest --project backend` run showing green. A transcript-only watcher (e.g. `/goal`) can't open the files; it only sees what you surface.

## Rules

- One test at a time; expected values come from the spec/checklist, never pasted from implementation output.
- One hat: make it pass, then refactor.
- **Stop at the backend.** The frontend is a separate phase and your review gate — don't roll into it.
