---
name: data-fetching
description: Use when writing or reviewing data-fetching or perf code — tRPC + TanStack Query v5 + Next.js App Router. Server prefetch/HydrateClient, useSuspenseQuery, caching, optimistic updates, loading/error UI, no waterfalls.
---

# Data Fetching & Performance

The playbook for making every page feel instant. Stack: **tRPC + TanStack Query v5 + Next.js App Router (Next 16)**. Patterns below are the current canonical ones from the tRPC, TanStack Query, and Next.js docs — see [Sources](#sources).

## Golden path

> Kick off the query on the **server** (no `await`), stream it to the client, and let the client component read it via `useSuspenseQuery`. The Suspense fallback (`loading.tsx` or an inner `<Suspense>`) shows meanwhile. The client never re-fetches what the server already started.

```
Server Component: prefetch(queryOptions)  ──(stream)──▶  HydrateClient
   Client Component: useSuspenseQuery(sameQueryOptions) → reads cache, no waterfall
```

This is "render as you fetch": the server starts the fetch and returns HTML immediately (good TTFB), data streams in as it resolves. A spinner that never resolves, or a *second* skeleton on top of the route skeleton, means the pattern was broken — see [Loading UI](#loading-ui).

## Wiring

The `prefetch` / `HydrateClient` / `useTRPC` setup (`@trpc/tanstack-react-query`, **not** the legacy `createHydrationHelpers`) is a **one-time per-app** install — `lib/trpc/{query-client,server,react}.tsx`. Full boilerplate in [`wiring.md`](wiring.md). Everything below assumes it's in place.

## Prefetching

In the **page (Server Component)**: `prefetch` each query (no `await`), wrap the tree in `HydrateClient`.

```tsx
export default async function Page() {
  prefetch(trpc.users.funnel.queryOptions({ period: "30d" }));   // fire — don't await
  prefetch(trpc.users.table.queryOptions({ page: 1 }));          // fires in parallel
  return (
    <HydrateClient>
      <Suspense fallback={<FunnelCardsSkeleton />}><FunnelCards /></Suspense>
      <Suspense fallback={<UsersTableSkeleton />}><UsersTable /></Suspense>
    </HydrateClient>
  );
}
```

- **No waterfalls.** `prefetch` calls don't `await`, so they all start together. Never `await` one query before starting the next unless the second genuinely depends on the first.
- **Share `queryOptions` verbatim.** Server `prefetch` and client `useSuspenseQuery` must call the identical `trpc.x.queryOptions(input)` — that's what makes the query keys match and the cache hit. Mismatched input = cache miss = refetch.
- **Don't `await` prefetch** in the page. Awaiting blocks the whole page on the slowest query and kills streaming. Let each section stream into its own Suspense boundary.

## Client components

Read with `useSuspenseQuery` — the component suspends until its (already-streaming) data resolves, and the nearest Suspense fallback covers it. No manual `isLoading` branches.

```tsx
"use client";
export function FunnelCards() {
  const trpc = useTRPC();
  const { data } = useSuspenseQuery(trpc.users.funnel.queryOptions({ period: "30d" }));
  return <Cards data={data} />;
}
```

`useQuery` picks up the streamed promise too, but **Next.js won't suspend** for it — the component renders in `pending` status and opts out of server-rendering that content. So: `useSuspenseQuery` for prefetched data; plain `useQuery` only for data fetched **after** mount (user-triggered, dependent on client state), with its own loading state.

### Several data points at once

Two strategies — pick deliberately:

1. **One aggregating endpoint (preferred for metrics).** When the points are computed together — e.g. the funnel counts (signups, onboarded, posted, applied, paid) — return them as one object from a single procedure backed by **one SQL query** (`COUNT(*) FILTER (WHERE …)` per stage). One round trip, one cache entry, no fan-out. The component does a single `useSuspenseQuery`.

2. **`useSuspenseQueries` (genuinely independent sources).** When the points come from separate queries you want in parallel. **You cannot stack `useSuspenseQuery` calls** — the first suspends before the others run. Use one `useSuspenseQueries` (fixed set) or `useQueries` (variable count); both run in parallel, return results in input order, and accept a `combine` to fold into one value:

```tsx
const trpc = useTRPC();
const [{ data: revenue }, { data: applications }] = useSuspenseQueries({
  queries: [
    trpc.revenue.summary.queryOptions({ period }),
    trpc.applications.summary.queryOptions({ period }),
  ],
});
```

Server prefetch still fires each with its own `prefetch(...)` call (parallel). Avoid duplicate query keys in the array — same key twice can share data.

**Server Components are for prefetch only.** Don't render a query's data in a Server Component *and* the same query in a Client Component — when the client revalidates after `staleTime`, the server-rendered copy goes stale and desyncs. Treat the Server Component as a place to start the fetch, nothing more (avoid `fetchQuery` + rendering its result on the server). Also: never use a Server Action as a `queryFn` — Server Actions run serially and stall React Query; fetch via tRPC (which we do).

## Caching

- **`staleTime`** (set non-zero, e.g. 30–60s) governs refetch-on-mount/focus. With prefetched data, a non-zero staleTime is what *prevents the instant re-fetch* after hydration (default `0` would refetch immediately). Raise it for reference data that rarely changes.
- **`gcTime`** (default 5m) is how long unused cache is retained before garbage-collection. Leave unless a screen needs longer.
- **Query keys are derived** from `queryOptions` — never hand-write keys. Invalidate with the matching filter: `queryClient.invalidateQueries(trpc.users.table.queryFilter())`.
- **One freshness layer.** Do not also set route/`fetch` caching in `next.config.ts` for these — TanStack owns freshness here; two layers conflict.

## Loading UI

- **`loading.tsx` per route segment** is a Next.js primitive: it auto-wraps `page.tsx` and nested layouts/children in a `<Suspense>` boundary and shows **instantly on navigation** (it's prefetched). It does **not** wrap the same-segment `layout.tsx` or `error.tsx`.
- **Make it a meaningful skeleton that matches the loaded layout** — same container, grid, column count, card sizes — so there's no layout shift when real content swaps in. A spinner is acceptable; a *mismatched* skeleton is worse than none.
- **One skeleton, not two.** The route fallback (`loading.tsx`) and the inner `<Suspense>` fallbacks are different boundaries — don't let a client component *also* render its own spinner inside an already-suspended boundary. With `useSuspenseQuery`, the component suspends; it must not branch on `isLoading` too.
- **Keep runtime data out of `layout.tsx`.** If a layout reads uncached data (`cookies()`, `headers()`, an uncached fetch), `loading.tsx` won't show a fallback for it and navigation blocks. Fetch in `page.tsx`, or wrap the layout's runtime access in its own `<Suspense>`.
- **Granular streaming:** beyond `loading.tsx`, wrap independent sections in their own `<Suspense>` so fast sections paint while slow ones stream.

## Error UI

- **`error.tsx` per segment** is the error boundary — it **must** be a Client Component (`"use client"`). It catches render + data errors in its segment.
- It receives `{ error, reset }`; wire `reset()` to a "Try again" action that re-renders the segment. Every route that fetches gets one.
- For finer control, an inner `react-error-boundary` `<ErrorBoundary>` around a specific `<Suspense>` isolates one failing section instead of failing the page.

## Optimistic updates

Two approaches — pick by scope.

**A. UI-only via mutation variables** (simplest; single component reflects the pending action). Use the mutation's `variables` + `isPending` to render the optimistic row; no cache writes, auto-reconciles on settle.

**B. Cache update via `onMutate`** (when other components/queries must reflect the change immediately):

The callbacks receive a `context` (use `context.client` instead of a closed-over QueryClient); `onMutate`'s return value arrives as `onMutateResult`:

```ts
const filter = trpc.promo.list.queryFilter();
useMutation(trpc.promo.markPaid.mutationOptions({
  onMutate: async (vars, ctx) => {
    await ctx.client.cancelQueries(filter);            // 1. stop in-flight refetch
    const prev = ctx.client.getQueryData(filter.queryKey); // 2. snapshot
    ctx.client.setQueryData(filter.queryKey, optimistic(prev, vars)); // 3. apply
    return { prev };                                   // → onMutateResult
  },
  onError: (_e, _vars, onMutateResult, ctx) =>
    ctx.client.setQueryData(filter.queryKey, onMutateResult.prev), // rollback
  onSettled: (_d, _e, _vars, _res, ctx) =>
    ctx.client.invalidateQueries(filter),              // reconcile (runs even after rollback)
}));
```

Order matters: **`cancelQueries` first** (else a slow refetch clobbers the optimistic value); **invalidate in `onSettled`** (runs even after a rollback).

## Inner CTAs & secondary surfaces

The pattern most often dropped — modals, drawers, detail panels, tab content opened from a button deserve the **same** treatment as a page:

- **Preload on hover/focus** of the trigger: `queryClient.prefetchQuery(trpc.x.queryOptions(input))`, so the panel's data is warm before the click.
- Open the panel against the **same cache / `queryOptions`**, not an ad-hoc `fetch` with a fresh spinner.
- Heavy panels: `next/dynamic` the component (keep it out of the initial bundle) **and** prefetch its data on hover — so opening feels instant despite the lazy code.

## No waterfalls (CRITICAL)

- Independent fetches run in **parallel** — fire all `prefetch`/promises, then await together (`Promise.all`) if you must await at all.
- Parallelize RSC fetches by **component composition**: sibling async components each fetch their own data instead of a parent fetching then rendering a child (which serializes them).
- Move `await` into the branch that actually uses the value; start promises early, await late.

## Per-feature checklist

- [ ] Page `prefetch`es every query it renders (no `await`, parallel), wrapped in `HydrateClient`.
- [ ] Client components use `useSuspenseQuery` with the **same `queryOptions`** as the prefetch.
- [ ] `loading.tsx` exists and its skeleton matches the loaded layout (no shift).
- [ ] No second spinner stacked inside an already-suspended boundary.
- [ ] No runtime/uncached data in `layout.tsx` blocking the fallback.
- [ ] `error.tsx` present (`"use client"`) with a working `reset()`.
- [ ] Instant-feeling mutations use an optimistic update (variables, or `onMutate` cancel→snapshot→set→rollback→invalidate).
- [ ] Inner CTAs (modals/drawers/tabs) preload data on hover and reuse the cache.
- [ ] No sequential awaits for independent data anywhere in the path.

## Sources

- [tRPC — Set up with React Server Components (TanStack React Query)](https://trpc.io/docs/client/tanstack-react-query/server-components)
- [TanStack Query — Advanced Server Rendering](https://tanstack.com/query/latest/docs/framework/react/guides/advanced-ssr)
- [TanStack Query — Server Rendering & Hydration](https://tanstack.com/query/v5/docs/framework/react/guides/ssr)
- [TanStack Query — Parallel Queries (`useQueries` / `useSuspenseQueries`, `combine`)](https://tanstack.com/query/v5/docs/framework/react/guides/parallel-queries)
- [TanStack Query — Optimistic Updates](https://tanstack.com/query/v5/docs/framework/react/guides/optimistic-updates)
- [Next.js — loading.js & instant loading states](https://nextjs.org/docs/app/api-reference/file-conventions/loading)
- [Next.js — error.js / error boundaries](https://nextjs.org/docs/app/api-reference/file-conventions/error)
- Vercel React Best Practices — `async-parallel`, `server-parallel-fetching`, `bundle-preload`
