---
title: Next.js Rendering Strategies
type: concept
tags: [nextjs, rendering, ssr, ssg, isr, streaming, static, dynamic, cache]
updated: 2026-05-25
sources:
  - notes/2026-05-25-hoc-nextjs.md
---

## Definition

[Next.js](./nextjs-overview.md) supports four rendering strategies that control **when** and **where** HTML is generated. The strategy for a given route is inferred from how `fetch` is called inside its [Server Components](./nextjs-server-vs-client-components.md) — no explicit mode flag is required.

| Strategy | When HTML is produced | Cache behavior | Typical use case |
|---|---|---|---|
| **Static (SSG)** | At build time | Long-lived CDN cache | Blogs, docs, marketing pages |
| **Dynamic (SSR)** | Per request, on the server | No cache by default | Personalized dashboards, auth-gated content |
| **ISR** (Incremental Static Regeneration) | At build + re-generated on interval or trigger | Stale-while-revalidate | Ecommerce product pages, news feeds |
| **Streaming** | Progressively as data resolves | Partial cache per `Suspense` boundary | Pages with mixed fast/slow data dependencies |

## Synonyms & Aliases

- SSG — Static Site Generation
- SSR — Server-Side Rendering
- ISR — Incremental Static Regeneration
- PPR — Partial Prerendering (experimental, combines static shell + dynamic streaming)

## When to Use Each

**Static**: Content does not change per user and is known at build time. Highest performance — served from CDN edge with no server compute per request.

**Dynamic**: Content varies per request (user identity, cookies, headers, real-time data). Slightly higher TTFB but always fresh. Triggered when any `fetch` in the route uses `cache: 'no-store'` or when `cookies()` / `headers()` are accessed.

**ISR / Revalidation**: Content changes occasionally but not per request. Balances freshness and performance. Configure via `next: { revalidate: N }` in `fetch` options, or via on-demand `revalidateTag()` / `revalidatePath()`.

**Streaming**: Some parts of the page are ready quickly; others depend on slow queries. Wrap slow subtrees in `<Suspense>` to stream UI progressively. `loading.tsx` files are implicit `Suspense` boundaries at the route level.

## Advantages

- Static and ISR pages are served from CDN with near-zero server load at request time.
- Streaming eliminates full-page blocking — users see fast content while slow data loads.
- Mix of strategies is supported within the same app — each route picks its own mode independently.
- ISR with `revalidateTag` enables CMS-driven on-demand cache invalidation without a full rebuild.

## Tradeoffs

- Dynamic rendering incurs full server cost per request — don't use it for content that could be static.
- ISR revalidation windows mean slightly stale data is shown until revalidation completes.
- Streaming requires intentional `Suspense` placement — missing boundaries cause the entire page to block on the slowest query.
- Partial Prerendering (PPR) mixes static shell and dynamic streaming but is experimental as of 2026.

## Common Mistakes

- Not distinguishing `cache: 'no-store'` (always dynamic) from the default cache behavior — the default is aggressive caching, which surprises developers expecting fresh data.
- Using dynamic rendering for pages that are actually static — wastes server capacity.
- Forgetting `loading.tsx` when streaming — users see blank content instead of a skeleton.
- Nesting parallel `fetch` calls sequentially with `await` instead of `Promise.all`, creating unnecessary waterfalls.

## Related Knowledge

- [Next.js Overview](./nextjs-overview.md)
- [Server Components vs Client Components](./nextjs-server-vs-client-components.md)
- [Data Fetching in App Router](../../playbooks/nextjs/nextjs-data-fetching.md)