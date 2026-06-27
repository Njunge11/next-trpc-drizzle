---
name: build-backend-feature
description: Use to build the BACKEND of a feature end-to-end test-first (Phase 1, before frontend) — decide the feature, write a living test-case checklist at features/<name>/checklist.md, write tests, make them pass until the backend suite is green. tRPC + Drizzle + Vitest/PGlite. Pairs with /goal.
---

# Feature Build — Backend (Phase 1)

The backend half of a feature, built test-first. This is a **standalone phase**: finish and review it before touching the frontend (the build-frontend-feature skill is a separate phase with its own manual gate).

References: tdd · backend-standards · testing · data-fetching.

## The loop

0. **Decide the feature.** What data points / mutations it needs; one aggregating endpoint vs several.
1. **Write the test-case checklist** → `features/<name>/checklist.md`, under `## Backend`. Each line is one behavior case (repo / service / router), unchecked. This file is the **living test list** — it's the artifact, version-controlled with the feature.
2. **Pick one unchecked case** → write one real test for it (setup / invoke / assert; see the testing skill).
3. **Make it pass** — minimum code, layer order db → service → router, backend-standards.
4. **Refactor** if needed, then **tick the item `[x]`**. Add any newly-discovered cases to the list.
5. Repeat until **every `## Backend` item is checked and the backend suite is green**.

## Checklist format

`features/users/checklist.md`:

```md
# users — feature checklist

## Backend
- [ ] B1 repo: funnel counts computed in ONE query (signups/onboarded/posted/applied/paid)
- [ ] B2 service: period math (7d / 30d / all) buckets correctly
- [ ] B3 router: funnel({period}) returns the funnel object for an authed session
- [ ] B4 router: unauthenticated request is rejected

## Frontend + Integration
- [ ] (added in phase 2)
```

## Finish line — `/goal`

`/goal`'s evaluator only reads the transcript; it can't run commands or files. So the condition must be something the agent **proves in its output** — it has to paste the test run and show the checklist. Run it with auto mode so each turn is unattended:

```
/goal Every item under "## Backend" in features/<name>/checklist.md is
checked [x], and `vitest --project backend` for this feature passes — paste the run
output showing all tests green. Do not modify any frontend/UI files. Stop after 25
turns if not met.
```

That has the four parts of a good condition: a **measurable end state** (all items checked + suite green), a **stated check** (paste the `vitest` output), a **constraint** (don't touch the frontend), and a **cap** (25 turns).

## Rules

- One test at a time; expected values come from the spec/checklist, never pasted from implementation output.
- One hat: make it pass, then refactor.
- **Stop at the backend.** The frontend is a separate phase and your review gate — don't roll into it.
