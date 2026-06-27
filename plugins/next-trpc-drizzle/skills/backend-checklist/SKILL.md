---
name: backend-checklist
description: Use after agreeing a feature's backend design to turn it into the `## Backend` test list in features/<name>/checklist.md — one observable behavior per line, ready to drive build-backend-feature. Owns the backend checklist's file shape.
---

# Backend Checklist

Turn the agreed backend design — settled **in this conversation** — into the `## Backend` section of `features/<name>/checklist.md`. That section is a **TDD test list** (see tdd): the behaviors you'll drive out one test at a time in build-backend-feature. **Authoring only: no code, no tests.**

## Input

The agreed design already in context — data points, mutations, endpoints, authz rules. Read it; don't re-derive it. If a case needs a decision that wasn't made, **ask** — don't invent behavior.

## What each line is

One **observable backend behavior** — the kind of thing the testing skill says to assert: an API response, DB state, a side effect, an authz outcome, at the repo / service / router layer. Behavior, never implementation. For the litmus and what's worth asserting, see testing (and testing/backend.md); for how a test list is written and grows, see tdd.

## Structure

The file opens with a title, then a flat list of behavior cases under `## Backend` — each an unchecked box with a `B<n>` id. **No fixed count and no one-per-layer shape**: list as many cases as the feature actually has (could be three repo cases and one router case), and add more as you discover them mid-build.

```md
# <name> — feature checklist

## Backend
- [ ] B1 funnel counts computed in one query (signups/onboarded/posted/applied/paid)
- [ ] B2 period buckets: 7d / 30d / all bucket correctly
- [ ] B3 funnel({period}) returns the funnel for an authed session
- [ ] B4 unauthenticated request is rejected
```

- Cover the behavior space (tdd): happy path, edges, error cases, authz, and "must not break `<X>`".
- Expected values come from the agreed spec — never from imagined implementation output.

## Rules

- Write only the title and the `## Backend` section. The frontend section is frontend-checklist's job in Phase 2 — don't author it here.
- **Stop at authoring.** Present the section for the user to review and amend — don't start building. Building is build-backend-feature.
