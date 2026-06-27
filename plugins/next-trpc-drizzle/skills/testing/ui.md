# UI Testing — what to assert

> A test should fail because the **user-visible behavior** is wrong, not because you refactored the code.

If you can rewrite the component's internals (state, hooks, handlers, library) without touching the test, you're testing behavior. If every refactor breaks tests, you're testing implementation.

Stack: **Vitest + Testing Library + `@testing-library/user-event` + MSW**. Tests live in each feature's `ui/__tests__/`.

## Rule of thumb

| Assert on…                         | Good? | |
| ---------------------------------- | ----- | -------------- |
| What the user sees (text, roles)   | ✅    | behavior |
| What the user can do (click, type) | ✅    | behavior |
| What appears/disappears after      | ✅    | behavior |
| React state / hooks / refs         | ❌    | implementation |
| Handler called (`toHaveBeenCalled`)| ❌    | implementation |
| CSS classes (unless user-visible)  | ❌    | implementation |

## How to query

Prefer accessible queries — `getByRole`, `getByLabelText`, `getByText`. Use `findBy*` for async (it waits); `queryBy*` to assert absence. Avoid `getByTestId` unless there's no accessible handle.

## Examples

**Loading → loaded** — assert what's on screen, not `isLoading`.
```ts
expect(screen.getByText("Loading…")).toBeVisible()
expect(await screen.findByText("1,204 signups")).toBeVisible()
```

**Interaction** — assert the outcome, not the handler.
```ts
// ❌ expect(handleApply).toHaveBeenCalled()
await user.selectOptions(screen.getByRole("combobox", { name: /period/i }), "30d")
expect(await screen.findByText("Last 30 days")).toBeVisible()
```

**Search / filter** — assert what's shown and what's gone.
```ts
await user.type(screen.getByRole("searchbox"), "acme")
expect(screen.getByText("Acme Ltd")).toBeVisible()
expect(screen.queryByText("Other Co")).not.toBeInTheDocument()
```

**Modal / dialog** — assert the dialog, not `setOpen`.
```ts
await user.click(screen.getByRole("button", { name: /mark paid/i }))
expect(screen.getByRole("dialog")).toBeVisible()
```

## Mocking boundary

Mock only the network edge — the tRPC/HTTP calls — with **MSW**. Never mock your own components, hooks, or business logic. The component should run for real against canned server responses, so swapping TanStack Query for anything else leaves the test green.

## Litmus test

Before committing a test: *if I rewrote this component's internals (state lib, data lib, markup), should it still pass?* If no, you're testing implementation — rewrite it to assert what the user observes.
