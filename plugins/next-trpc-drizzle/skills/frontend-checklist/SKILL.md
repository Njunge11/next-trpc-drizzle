---
name: frontend-checklist
description: Use after agreeing a feature's UI to turn it into the `## Frontend + Integration` test list in features/<name>/checklist.md — test-backed Behavior items vs browser-checked Visual items, ready to drive build-frontend-feature. Owns the checklist file shape.
---

# Frontend Checklist

Turn the UI agreed **in this conversation** (plus the feature's real API surface under `features/<name>/api`) into the `## Frontend + Integration` section of `features/<name>/checklist.md`. **Authoring only: no components, no tests.**

## How

- Read the described UI and the real backend contract from context; if a screen or state is undecided, **ask**. Preserve described copy verbatim.
- Split the section into two groups — the split is the point:
  - **Behavior (test-backed)** — `F<n>`: what the user observes/does, loading/error, the integrated data path. Each becomes a UI test (what to assert: testing).
  - **Visual & responsive (browser-checked, not tests)** — `V<n>`: layout, spacing, tokens, the `md:` tablet state (built per frontend-standards). jsdom has no layout engine — **never write a Vitest test for these.**

  ```md
  ## Frontend + Integration
  ### Behavior (test-backed)
  - [ ] F1 renders the cards given data
  - [ ] F2 changing the period updates the cards

  ### Visual & responsive (browser-checked, not tests)
  - [ ] V1 md: tablet layout
  ```

- Append only this section; the file title and `## Backend` come from backend-checklist.
- **Stop at authoring.** Present it for review; don't build — that's build-frontend-feature.
