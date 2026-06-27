# Backend Testing — what to assert

> A test should fail because the **observable behavior** is wrong — the API response, the DB state, the side effect — not because you refactored the code.

If you can swap the ORM for raw SQL, or restructure a service, without touching the test, you're testing behavior. Assertions on internal calls (`expect(repo.save).toHaveBeenCalled()`) are implementation tests — they break on every refactor and prove nothing a user observes.

Stack: **Vitest** (the `backend` project — see the parent skill), **tRPC `createCaller`**, **fake repos** for services, **PGlite** (real schema) for repos. Tests live in each feature's `api/__tests__/` and `db/__tests__/`.

## Rule of thumb

| Assert on…                          | Good? | |
| ----------------------------------- | ----- | -------------- |
| Router/API response (data, error)   | ✅    | behavior |
| DB state after the call             | ✅    | behavior |
| Side effect happened (email queued) | ✅    | behavior |
| Authz outcome (denied / allowed)    | ✅    | behavior |
| Internal function called            | ❌    | implementation |
| Private method / intermediate var   | ❌    | implementation |
| Which ORM method ran                | ❌    | implementation |

## Layered approach (maps to slices)

**Repo (`db/__tests__/`) — PGlite, real schema.** Push the imported schema with `drizzle-kit push` (see the parent skill), so FKs and constraints are real. Assert rows in/out of the DB, not which Drizzle calls ran.
```ts
const user = await repo.createUser({ name: "Acme", email: "a@example.com" })
expect(await repo.getUserById(user.id)).toMatchObject({ name: "Acme" })
```

**Service (`api/__tests__/`) — fake repo.** Inject a fake repo plus deterministic `now()` / `uuid()`. Assert returned values and what was persisted to the fake, never that a method was called.
```ts
const svc = makeUsersService({ repo: fakeRepo, now: () => FIXED, uuid: () => "id-1" })
await svc.onboardUser({ userId: "u1" })
expect(fakeRepo.users).toContainEqual(expect.objectContaining({ id: "u1", onboardedAt: FIXED }))
```

**Router (`api/__tests__/`) — `createCaller` + session.** Drive the real router with a built context. Assert the response shape and thrown errors.
```ts
const caller = appRouter.createCaller(ctxFor(session))
expect(await caller.users.funnel({ period: "30d" })).toMatchObject({ signups: 1204 })

const anon = appRouter.createCaller(ctxFor(null)) // no session
await expect(anon.users.funnel({ period: "30d" })).rejects.toThrow(/unauthorized/)
```

## Determinism

Every service/repo takes injected `now()` and `uuid()` — never read the clock or randomness directly. Expected values come from the spec or a fixture, **never** copied from the implementation's output.

## Mocking boundary

Mock only systems **outside ours** — payment/webhook providers, email, OAuth. Never mock your own repos in router tests or your own services in router/UI tests; let them run against PGlite or a fake repo. Swapping the ORM, restructuring a service, or changing auth strategy should leave behavior tests green.

## Litmus test

*If I completely rewrote the implementation (ORM → SQL, service restructure), should this test still pass?* If no, it's an implementation test — rewrite it to assert the response, the DB state, or the side effect.
