---
title: Data Fetching in Next.js App Router
type: playbook
tags: [nextjs, data-fetching, cache, revalidation, server-components, app-router, fetch]
updated: 2026-05-25
sources:
  - notes/2026-05-25-hoc-nextjs.md
---

## Problem

You need to fetch data in a [Next.js](../../concepts/nextjs/nextjs-overview.md) App Router application and choose the right caching and revalidation strategy to balance freshness, performance, and server load.

## Investigation Steps

Before writing fetch code, answer these questions:

1. **Who does this data belong to?** Same for all users (cacheable) or user-specific (dynamic)?
2. **How often does the data change?** Never (static), on a schedule (ISR), always (dynamic)?
3. **Does the component need user interaction after load?** If yes, consider client-side fetching for that slice.

The answers map directly to a `fetch` cache option — see the table in Resolution.

## Resolution

### Fetch in a Server Component (preferred default)

[Server Components](../../concepts/nextjs/nextjs-server-vs-client-components.md) can `fetch` directly with `async/await`. Next.js extends the native `fetch` API with caching and revalidation controls.

```tsx
// app/products/page.tsx  (Server Component)

// Cached indefinitely (static — default behavior)
const data = await fetch('https://api.example.com/config');

// Always fresh — forces dynamic rendering for this route
const data = await fetch('https://api.example.com/cart', { cache: 'no-store' });

// Revalidate every 60 seconds (ISR-style)
const data = await fetch('https://api.example.com/products', {
  next: { revalidate: 60 },
});

// On-demand revalidation via tag
const data = await fetch('https://api.example.com/posts', {
  next: { tags: ['posts'] },
});
```

### Cache option reference

| Option | Behavior | [Rendering strategy](../../concepts/nextjs/nextjs-rendering-strategies.md) | Use when |
|---|---|---|---|
| Default (no option) | Cached indefinitely | Static | Config, rarely-changing content |
| `cache: 'no-store'` | No cache, fetched every request | Dynamic | Per-user data, session-dependent content |
| `next: { revalidate: N }` | Re-fetch after N seconds | ISR | Product listings, news, content that changes on a schedule |
| `next: { tags: ['tag'] }` | Cache until `revalidateTag('tag')` is called | ISR (on-demand) | CMS content, admin-triggered invalidation |

### Parallel fetching (avoid waterfalls)

```tsx
// ✅ Both requests fire simultaneously
const [user, posts] = await Promise.all([
  fetch('/api/user').then(r => r.json()),
  fetch('/api/posts').then(r => r.json()),
]);

// ❌ Sequential — second request waits for first unnecessarily
const user = await fetch('/api/user').then(r => r.json());
const posts = await fetch('/api/posts').then(r => r.json());
```

### On-demand cache invalidation

```ts
// app/api/revalidate/route.ts
import { revalidateTag, revalidatePath } from 'next/cache';
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  const { tag } = await req.json();
  revalidateTag(tag);          // invalidate all fetches tagged with this value
  // or: revalidatePath('/posts'); // invalidate a specific route
  return NextResponse.json({ revalidated: true });
}
```

Call this endpoint from a CMS webhook or admin action to clear specific cached data without a full rebuild.

### Client-side fetching (when necessary)

Use `useEffect` + `fetch`, SWR, or React Query **only** when:
- Data requires user interaction to trigger (e.g., search-as-you-type)
- Data needs real-time updates (polling, WebSocket)
- The component must stay interactive and show updated data without navigation

```tsx
'use client';
import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(r => r.json());

export default function LiveStatus() {
  const { data, error } = useSWR('/api/status', fetcher, { refreshInterval: 5000 });
  if (error) return <p>Error</p>;
  if (!data) return <p>Loading…</p>;
  return <p>Status: {data.status}</p>;
}
```

## Prevention

- Do not mix `fetch` with `cache: 'no-store'` and a default-cached `fetch` in the same component and expect predictable rendering mode — Next.js infers the mode from all fetches; `no-store` anywhere makes the whole route dynamic.
- Always add `loading.tsx` when fetching asynchronously to prevent blank pages during slow responses.
- Avoid deeply nested `await` chains — prefer `Promise.all` for independent parallel requests.
- Do not fetch data in Client Components if a Server Component can do it — client fetching ships more JS, adds latency, and exposes API calls to the browser.

## Related Knowledge

- [Next.js Rendering Strategies](../../concepts/nextjs/nextjs-rendering-strategies.md)
- [Server Components vs Client Components](../../concepts/nextjs/nextjs-server-vs-client-components.md)
- [Route Handlers and Server Actions](./nextjs-route-handlers-and-server-actions.md)
- [Create Your First Next.js App](./nextjs-first-app.md)