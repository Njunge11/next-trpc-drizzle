---
name: testing
description: Use when configuring or writing tests — Vitest projects (node and jsdom), PGlite for repo tests, fake repo for services, createCaller for routers, MSW vs cache-seeding for UI, and the behavior-not-implementation rule for what to assert.
---

# Test Setup

The harness for both test suites. *What to assert* is in backend.md / ui.md (this skill's supporting files); this is *how the runner and mocks are wired*.

## One Vitest config, two projects

Use Vitest's **`projects`** (the `workspace` file is deprecated since Vitest 3.2). One root `vitest.config.ts` defines two projects that differ only in environment + file globs:

```ts
// vitest.config.ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    projects: [
      {
        extends: true,
        test: {
          name: "backend",
          environment: "node",
          include: ["features/**/{api,db}/__tests__/**/*.test.ts"],
          setupFiles: ["./test/setup.backend.ts"],
        },
      },
      {
        extends: true,
        test: {
          name: "ui",
          environment: "jsdom", // happy-dom is faster; jsdom is more complete — default to jsdom
          include: ["features/**/ui/__tests__/**/*.test.tsx"],
          setupFiles: ["./test/setup.ui.ts"],
        },
      },
    ],
  },
});
```

Run all, or one: `vitest --project backend`, `vitest --project ui`.

## Backend project

- **Environment `node`.** No DOM.
- **Repo tests → PGlite.** Real Postgres compiled to WASM, in-process — no Docker, fast, parallel-friendly. This is the current recommended way to test Drizzle against Postgres (Drizzle ships first-class PGlite support); reserve Testcontainers for cases needing a specific Postgres version or extensions PGlite lacks.
  - **Load the schema with `drizzle-kit push`, not a migration folder.** Push the imported schema objects straight into the PGlite instance at suite start — tests don't need migration files, and this decouples them from any other app's `drizzle/` directory.
  - **Isolate each test with a transaction that rolls back.** Wrap the test body in a transaction and discard it; ~2–4 ms/test vs ~40–60 ms for truncate-and-reseed, and no sequence resets. Repos take an injected `db`/`tx` handle (see backend-standards), so the test passes the tx in:

    ```ts
    await db.transaction(async (tx) => {
      await runTest(makeRepo(tx)); // assertions here
      throw ROLLBACK;              // discard — next test starts clean
    });
    ```
  - **Alternative — PGlite snapshots.** When a test must actually commit (e.g. asserting across committed transactions), snapshot the freshly-pushed empty DB once and restore the snapshot per test instead of rolling back.
- **Service tests → in-memory fake repo.** No DB; inject the fake plus deterministic `now()`/`uuid()`.
- **Router tests → tRPC `createCaller`.** The official server-side calling primitive: build a context with a test session, call procedures, assert the response/error.

`test/setup.backend.ts` holds the shared PGlite bootstrap + `push`.

## UI project

- **Environment `jsdom`** + **React Testing Library** + **`@testing-library/user-event`**.
- `test/setup.ui.ts` imports `@testing-library/jest-dom/vitest` and starts the MSW server (below).

### The mocking decision (the part that trips people up)

Components read data through `useSuspenseQuery(trpc.x.queryOptions(...))` via `useTRPC()`. Two valid levels — **default to MSW**, drop to cache-seeding only for trivial render tests:

| Goal | Tool |
| --- | --- |
| Loading → success transition, **error** states, **mutations** / optimistic round-trips | **MSW** at the network edge (`msw-trpc` for typed handlers) |
| Pure "renders correct UI given this data" / interaction outcome, no fetch behavior under test | **Seed the cache** (`queryClient.setQueryData`), no MSW |

Both render through the **same provider wrapper** — a test QueryClient (`retry: false`) + the tRPC provider:

```tsx
// test/render.tsx
export function renderWithProviders(ui, { queryClient = makeTestClient() } = {}) {
  const trpcClient = makeTRPCClient(); // points at the URL MSW intercepts
  return render(
    <QueryClientProvider client={queryClient}>
      <TRPCProvider trpcClient={trpcClient} queryClient={queryClient}>{ui}</TRPCProvider>
    </QueryClientProvider>
  );
}
const makeTestClient = () =>
  new QueryClient({ defaultOptions: { queries: { retry: false } } }); // no retries → fast error tests
```

**MSW path** — handlers are type-safe with `msw-trpc`, server lifecycle in setup:

```ts
// test/msw.ts
import { setupServer } from "msw/node";
import { createTRPCMsw } from "msw-trpc";
export const trpcMsw = createTRPCMsw<AppRouter>();
export const server = setupServer();

// test/setup.ui.ts
beforeAll(() => server.listen({ onUnhandledRequest: "error" }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// in a test
server.use(trpcMsw.users.funnel.query(() => ({ signups: 1204, /* … */ })));
```

**Cache-seed path** — no network at all; `useSuspenseQuery` reads the warm cache and renders immediately:

```tsx
queryClient.setQueryData(trpc.users.funnel.queryOptions({ period: "30d" }).queryKey, mockFunnel);
renderWithProviders(<FunnelCards />, { queryClient });
```

> Don't mock the `useTRPC`/`useQuery` hooks themselves — that tests the mock, not the component. Mock at the network (MSW) or seed the cache.

## Dev dependencies

Verified current (June 2026): `msw-trpc@2.0.1` peers `@trpc/server@^11` + `msw@^2`; `msw@2.14.x`.

```
vitest  @vitest/coverage-v8  jsdom
@testing-library/react  @testing-library/user-event  @testing-library/jest-dom
msw  msw-trpc
@electric-sql/pglite          # backend repo tests
```

## Sources

- [Vitest — Test Projects](https://vitest.dev/guide/projects) (workspace deprecated in 3.2)
- [TanStack Query — Testing](https://tanstack.com/query/v5/docs/framework/react/guides/testing) (test QueryClient, `retry: false`)
- [TkDodo — Testing React Query (MSW approach)](https://tkdodo.eu/blog/testing-react-query)
- [msw-trpc](https://www.npmjs.com/package/msw-trpc) — typed tRPC handlers for MSW (v2.0.1, peers tRPC v11 + msw v2)
- [Drizzle — PGlite](https://orm.drizzle.team/docs/connect-pglite) & [PGlite](https://pglite.dev/) — in-process Postgres for repo tests
- [Fun & Sane Node TDD: Postgres with PGlite, Drizzle & Vitest](https://nikolamilovic.com/posts/fun-sane-node-tdd-postgres-pglite-drizzle-vitest/) — push-not-migrate + snapshotting
- [From 4 Minutes to 3 Seconds: transaction-rollback test isolation](https://dev.to/miry/from-4-minutes-to-3-seconds-how-database-transaction-rollback-revolutionized-test-suite-4olh)
