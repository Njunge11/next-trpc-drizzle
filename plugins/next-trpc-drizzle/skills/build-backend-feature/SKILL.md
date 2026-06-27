---
name: build-backend-feature
description: Use to build the BACKEND of a feature test-first — Phase 1, before any frontend. Drives a feature's checklist + TDD loop to a green backend suite. tRPC + Drizzle + Vitest/PGlite.
context: fork
agent: backend-feature-builder
---

# Feature Build — Backend (Phase 1)

Build a feature's backend test-first, driven by its checklist. This runs in a forked subagent with the `tdd`, `testing`, `backend-standards`, and `data-fetching` skills preloaded — so the checklist file and the repo are your source of truth (no chat history). Author the checklist first, in-session, with backend-checklist.

**Checklist:** `$ARGUMENTS` — the path to the feature's `features/<name>/checklist.md`. If none was given, locate the feature's checklist under `features/`.

## The loop

1. Read the `## Backend` section of the checklist — the test list to burn down.
2. **Pick one unchecked case** → write one real test for it (setup / invoke / assert).
3. **Make it pass** — minimum code, layer order db → service → router.
4. **Refactor** if needed, then **tick the item `[x]`**. Add any newly-discovered cases to the list.
5. Repeat until **every `## Backend` item is `[x]` and `vitest --project backend` is green**. Then **return the proof**: show the checklist with all `## Backend` items `[x]` and paste the `vitest --project backend` run showing green, so a transcript-only watcher (e.g. `/goal`) can verify it.

## Rules

- **Stop at the backend.** Don't touch frontend/UI files — that's a separate phase and your review gate.
