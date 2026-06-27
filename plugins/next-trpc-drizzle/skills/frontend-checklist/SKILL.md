---
name: frontend-checklist
description: Use after agreeing a feature's UI to turn it into the `## Frontend + Integration` test list in features/<name>/checklist.md — test-backed Behavior items vs browser-checked Visual & responsive items, ready to drive build-frontend-feature. Owns the frontend checklist's file shape.
---

# Frontend Checklist

Turn the agreed UI — the described screens **in this conversation** plus the feature's already-built backend contract — into the `## Frontend + Integration` section of `features/<name>/checklist.md`. The Behavior items there are a **TDD test list** (see tdd); the Visual items are a browser-check list. **Authoring only: no components, no tests.**

## Input

The described UI in context, plus the feature's real API surface under `features/<name>/api` (so integration items match the **actual** contract, not an invented shape). If a screen or state is undecided, **ask** — don't invent UI or copy.

## Two groups (the split that matters)

- **Behavior (test-backed):** what the user observes/does, loading/error states, the integrated data path — each becomes a UI test. For what's worth asserting (and what isn't), see testing (and testing/ui.md).
- **Visual & responsive (browser-checked, NOT tests):** layout, spacing, token usage, the `md:` tablet state — built per frontend-standards and checked in the browser. jsdom has no layout engine, so **never write a Vitest test for these.** The separate group is what stops anyone trying.

## Structure

The two groups under `## Frontend + Integration`, each a flat list of unchecked boxes — Behavior ids `F<n>`, Visual ids `V<n>`. **No fixed count** in either: list as many cases as the UI actually has, and add more as you discover them.

```md
## Frontend + Integration
### Behavior (test-backed)
- [ ] F1 renders the funnel cards given funnel data
- [ ] F2 changing the period refetches and updates the cards
- [ ] F3 loading and error states render
- [ ] F4 integrated data path resolves against the real router

### Visual & responsive (browser-checked, not tests)
- [ ] V1 layout / spacing / token usage
- [ ] V2 md: tablet state
```

- Behavior items assert what the user **observes** (roles / text / outcomes), never component internals — see testing/ui.md.
- Integration items name the real procedure / `queryOptions` they wire to.
- Preserve described copy verbatim; don't invent labels or placeholder text.

## Rules

- Write only the `## Frontend + Integration` section — append it; the file title and `## Backend` come from backend-checklist.
- **Stop at authoring.** Present the section for the user to review and amend — don't start building. Building is build-frontend-feature.
