---
name: ci
description: Use when configuring CI or GitHub Actions or deciding how tests run across a Turborepo monorepo — turbo run test --affected (changed packages plus dependents), fetch-depth 0, and PR vs merge run shapes.
---

# CI (Turborepo monorepo)

**Don't run all tests.** Turbo's dependency graph runs only what a change affects.

## The command

```bash
turbo run test:run --affected
```

`--affected` runs tasks only for packages changed since the base branch (`main`) **plus everything that depends on them** (Turbo 2.9+). Two layers keep CI cheap:

- **`--affected`** prunes the task graph to changed packages + dependents — the biggest lever.
- **Caching** restores unchanged packages' results instantly; a cacheable `test:run` is keyed on inputs.

## Why graph-aware, not "test the folder I touched"

When an app imports a shared package (e.g. a schema or UI package), that dependency edge matters:

- Change **only the app** → only the app's tests run; unrelated packages are skipped.
- Change the **shared package** → `--affected` automatically also runs every package that depends on it.

Hand-rolling "only test what I changed" misses that second case. Let the graph decide.

## Gotchas

1. **Full git history required.** With a shallow clone Turbo can't compute the diff and treats *everything* as changed (fails safe, but slow). Use `actions/checkout` with `fetch-depth: 0`.
2. **Protected branch = safety net.** Use `--affected` on PRs, but run the **full** suite on merge to `main`/release — pruning is accurate, but a full run is cheap insurance and mostly cache hits.

## Recommended shape

- **Per package:** a `test:run` script.
- **PR CI:** `turbo run lint test:run --affected` with `fetch-depth: 0`.
- **Merge / release CI:** `turbo run lint test:run` (full, no `--affected`).
- **Later:** add Turborepo Remote Cache so the cache is shared across runners, not per-machine.

### Sketch — PR workflow

```yaml
# .github/workflows/ci.yml
on: pull_request
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 } # full history → correct --affected diff
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: pnpm }
      - run: pnpm install --frozen-lockfile
      - run: pnpm turbo run lint test:run --affected
```

## Sources

- [Turborepo — Constructing CI](https://turborepo.dev/docs/crafting-your-repository/constructing-ci)
- [Turborepo — `run` reference (`--affected`, `--filter`)](https://turborepo.dev/docs/reference/run)
- [Using the `--affected` flag in CI](https://rebeccamdeprey.com/blog/using-the-turborepo---affected-flag-in-ci)
