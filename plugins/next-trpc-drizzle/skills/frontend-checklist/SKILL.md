---
name: frontend-checklist
description: Use after agreeing a feature's UI to turn it into the `## Frontend + Integration` section of features/<name>/checklist.md — test-backed Behavior items vs browser-checked Visual & responsive items, ready to drive build-frontend-feature. Owns the frontend checklist structure.
---

# Frontend Checklist

Turn the agreed UI — the described screens **in this conversation** plus the feature's already-built backend contract — into the `## Frontend + Integration` section of `features/<name>/checklist.md`. That file is the living test list the frontend phase burns down (build-frontend-feature). **Authoring only: no components, no tests.**

## Input

The described UI in context, plus the feature's real API surface under `features/<name>/api` (so integration items match the **actual** contract, not an invented shape). If a screen or state is undecided, **ask** — don't invent UI or copy.

## Two groups (the split that matters)

- **Behavior (test-backed):** what the user sees/does, loading/error states, the integrated data path. Each maps to a UI test.
- **Visual & responsive (browser-checked, NOT tests):** layout, spacing, token usage, the `md:` tablet state. jsdom has no layout engine — **never write a Vitest test for these.** Keeping them in a separate group is what stops anyone from trying.

## Structure

```md
## Frontend + Integration
### Behavior (test-backed)
- [ ] F1 renders <…> given <data>
- [ ] F2 <interaction> → <outcome>
- [ ] F3 loading / error states
- [ ] F4 integrated data path against the real router

### Visual & responsive (browser-checked, not tests)
- [ ] V1 layout / spacing / token usage
- [ ] V2 md: tablet state
```

- Behavior items prefixed **F<n>**, visual items **V<n>** — each referenceable.
- Behavior items assert what the user **observes** (roles / text / outcomes), never component internals.
- Integration items name the real procedure / `queryOptions` they wire to.
- Preserve described copy verbatim; don't invent labels or placeholder text.

## Rules

- One observable behavior per Behavior line; one visual concern per Visual line.
- **Stop at authoring.** Present the section for the user to review and amend — don't start building. Building is build-frontend-feature.
