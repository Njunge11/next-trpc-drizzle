---
name: tdd
description: Use when implementing a feature test-first or writing tests — the Canon TDD loop (test list, one test, make it pass, refactor) and the backend-first build order. Vitest stack.
---

# TDD Workflow

How we build every feature. Test-first, one test at a time (Kent Beck's Canon TDD). What to *assert* lives in the testing skill; this is the *loop*.

## The loop

1. **Write the test list.** List the behavior variants for the change — basic case, edge cases, error cases, and ways it must not break existing behavior. This is behavioral analysis only. **Don't** mix in implementation decisions; there's time for those later.
2. **Write one test.** Turn exactly one list item into a real, runnable test — setup, invocation, assertions. Work backwards from the assertions. This is where interface decisions get made.
3. **Make it pass.** Change the code until this test and all previous tests pass. Nothing more.
4. **Refactor (optional).** Now make implementation decisions — improve the design only as far as this session needs.
5. **Repeat from 2** until the list is empty. The feature is done when the list is empty.

Add tests to the list whenever you discover one mid-loop; cross them off as they pass.

## Mistakes to avoid

- **Don't** write all the tests up front, then make them pass one by one — reworking the first decision invalidates the speculative rest, and you go a long time seeing nothing green.
- **Don't** write tests without assertions just for coverage.
- **Don't** paste computed/actual values into expected values — that defeats the double-check. Expected values come from the spec or a fixture.
- **Don't** fake a pass by deleting assertions.
- **Don't** mix refactoring into the make-it-pass step — make it run, then make it right (one hat at a time).
- **Don't** abstract too soon — duplication is a hint, not a command.

## Build order per feature

We build a feature back-to-front, each layer driven by the loop above:

1. **Backend, TDD.** Work the slices bottom-up — `db` (repo, PGlite), then `api` (service with fake repo, then router with `createCaller`). Test list → one test → pass → refactor, per slice.
2. **UI tests, TDD.** Write the failing UI tests (Testing Library + MSW against canned responses) for the components and their behavior.
3. **Integrate UI → backend.** Wire the real tRPC/TanStack Query calls; make the UI tests pass against the real router.

Picking the *order* of tests within a list matters and comes with experience — a good order keeps each step small and shapes a cleaner result.
