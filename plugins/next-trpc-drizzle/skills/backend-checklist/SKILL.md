---
name: backend-checklist
description: Use after agreeing a feature's backend design to turn the discussion into the `## Backend` section of features/<name>/checklist.md — one observable behavior case per line (repo/service/router), ready to drive build-backend-feature. Owns the backend checklist structure.
---

# Backend Checklist

Turn the agreed backend design — the architecture and implementation details settled **in this conversation** — into the `## Backend` section of `features/<name>/checklist.md`. That file is the living test list the backend phase burns down (build-backend-feature). **Authoring only: no code, no tests.**

## Input

The agreed design already in context — data points, mutations, endpoints, authz rules. Read it; don't re-derive it. If a case needs a decision that wasn't made, **ask** — don't invent behavior.

## What each line is

- **One observable behavior** (the Canon TDD test list — see tdd): an API response, DB state, a side effect, or an authz outcome. `router rejects unauthenticated request` ✅ — `add authMiddleware to the router` ❌ (that's implementation).
- The architecture you discussed decides *which* cases exist; each line still states **behavior**, never how it's built. Litmus: if you could swap the ORM or restructure a service without changing the line, it's a good line.

## Structure

```md
# <name> — feature checklist

## Backend
- [ ] B1 repo: <behavior>                 # e.g. funnel counts computed in ONE query
- [ ] B2 service: <behavior>              # e.g. 7d / 30d / all periods bucket correctly
- [ ] B3 router: <behavior> for an authed session
- [ ] B4 router: unauthenticated request rejected

## Frontend + Integration
- [ ] (added in phase 2 — see frontend-checklist)
```

- One behavior per line, prefixed **B<n>** so each item is referenceable ("amend B3").
- Tag the layer (`repo` / `service` / `router`); each line maps to exactly one test.
- Cover the basics: happy path, edges, error cases, authz, and "must not break `<X>`".
- Expected values come from the agreed spec — never from imagined implementation output.

## Rules

- Leave `## Frontend + Integration` stubbed; filling it is frontend-checklist's job.
- **Stop at authoring.** Present the section for the user to review and amend — don't start building. Building is build-backend-feature.
