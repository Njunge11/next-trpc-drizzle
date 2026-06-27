---
name: backend-standards
description: Layered backend architecture — tRPC router → service → repository → Drizzle. Use when writing or reviewing backend code: routers, services, repositories, queries, transactions, or migrations.
---

# Backend Standards

Rules for backend code (the `api/` and `db/` slices of a feature). Each rule has one home; the [Review checklist](#review-checklist) is a fast index, not a re-explanation.

## Principles

- Follow the layered architecture; never bypass a layer.
- One responsibility per layer.
- Use framework / ORM (tRPC, Drizzle) primitives before writing custom code.
- Minimize database round trips.

## Layers

```
tRPC Router  →  Service  →  Repository  →  Drizzle / Postgres
```

Each layer talks only to the next. A router never reaches a repository or the DB directly.

### Router — `api/*.router.ts`

The tRPC procedure. **Does:** validate input (`.input(zod)`), confirm the request is authenticated, call **exactly one** service method, shape the response/error for the client.
**Never:** business logic, transactions, SQL, ORM/Drizzle queries, repository imports.

### Service — `api/*.service.ts`

**Does:** all business logic; coordinate repositories and external services; own transaction boundaries; transform domain objects; throw domain errors. Takes injected dependencies (repo, `now()`, `uuid()`) for determinism.
**Never:** HTTP/tRPC concerns, status codes, SQL, ORM queries, re-validating input the router already validated.

### Repository — `db/*.repo.ts`

**Does:** read/write the DB; compose Drizzle queries; map rows to domain shapes. Accepts an injected `db`/`tx` handle.
**Never:** business logic, validation, HTTP concerns, external API calls, starting its own transaction.

## Queries & performance

This is where round trips are won or lost. Before writing a query, ask: *can this be one query? one transaction? loaded together? batched? run concurrently? done by the DB instead of in JS?*

**Always**
- Use Drizzle primitives; prefer them over raw SQL wherever Drizzle supports the operation.
- Select only the columns you need — **never `SELECT *`**.
- One query with `JOIN`/relations instead of N+1 (a query per row).
- Set-based writes: bulk insert; `UPDATE … WHERE` instead of read-modify-write; `DELETE … WHERE` instead of read-then-delete.
- Run independent operations concurrently:
  ```ts
  const [user, orders, notifications] = await Promise.all([
    getUser(), getOrders(), getNotifications(),
  ]);
  ```
- Paginate every list endpoint.
- Index frequently-queried columns; enforce constraints in the schema.
- Keep queries deterministic (stable ordering for pagination).

**Never**
- Queries inside loops, or duplicate queries for the same data.
- Sequential `await`s for independent operations (use `Promise.all`).
- Unbounded list queries, or full table scans where an index should exist.

## Transactions

- Use one when multiple writes must succeed together, or reads and writes must stay consistent (state transitions, money, anything multi-table).
- The **service** opens the transaction and passes the `tx` handle into repository calls. Repositories never start one.
- Don't wrap independent read-only operations in a transaction.

## Error handling

Errors bubble up: Repository → Service → Router → global handler. The global handler logs, maps the error, and returns the response.

- Don't wrap every function in `try/catch`.
- Catch **only** to: add context, convert an infrastructure error into a domain error, or recover from an expected failure.
- Never swallow an error — let it propagate.

## Migrations

- The schema is the single source of truth.
- Generate migrations with Drizzle tooling (`db:generate`); never hand-write a migration file.
- Never edit an applied migration — add a new one for every schema change.

## Reuse

- Reuse existing code only if it already follows these standards; otherwise refactor it, don't copy it.
- Never duplicate business logic, queries, validation, or utilities.
- Extract shared logic once duplication is real — not preemptively.

## Review checklist

Reject the change if any is true:

- [ ] Router touches the DB, imports a repository, or contains business logic.
- [ ] Service contains HTTP/tRPC concerns or SQL/ORM queries.
- [ ] Repository contains business logic, validation, or starts a transaction.
- [ ] A layer boundary is bypassed.
- [ ] Atomicity is required but no transaction wraps the writes.
- [ ] N+1 queries, queries in a loop, or duplicate queries.
- [ ] Independent `await`s run sequentially instead of via `Promise.all`.
- [ ] `SELECT *`, or raw SQL where Drizzle has an equivalent.
- [ ] A list endpoint lacks pagination, or a hot column lacks an index.
- [ ] A migration file was hand-written or an applied one edited.
- [ ] Repeated `try/catch`, or an error is swallowed instead of propagated.
- [ ] Duplicated business logic, query, or validation.
