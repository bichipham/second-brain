---
title: "Next.js: Server Components vs Client Components"
type: concept
tags: [nextjs, react, server-components, client-components, app-router, rsc, rendering]
updated: 2026-05-25
sources:
  - notes/2026-05-25-hoc-nextjs.md
---

## Definition

In [Next.js](./nextjs-overview.md) App Router, every React component is a **Server Component by default**. Server Components render on the server — they can access databases, secrets, and backend services directly, but cannot use browser APIs, React state, or side effects. **Client Components** opt in with the `'use client'` directive at the top of the file and run in the browser (hydrated from server-rendered HTML), enabling interactivity.

The boundary between server and client is explicit and declared at the file level. Everything in that file's subtree inherits the boundary unless a new `'use client'` declaration overrides it.

## Synonyms & Aliases

- RSC — React Server Components (the spec)
- SC / CC — shorthand for Server Component / Client Component
- `'use client'` component

## Decision Rule

Default to **Server Component**. Opt into Client Component only when you need:

| Need | Use Client Component |
|---|---|
| `useState` / `useReducer` | Yes |
| `useEffect` / lifecycle hooks | Yes |
| Browser APIs (`window`, `document`, `localStorage`) | Yes |
| Event handlers (`onClick`, `onChange`, `onSubmit`) | Yes |
| Third-party libraries that require a browser context | Yes |
| Direct database / secret access | No — use Server Component |
| SEO-critical content | No — use Server Component |
| Static or data-heavy rendering | No — use Server Component |

Push interactivity to **leaf nodes** of the component tree. Keep parent layouts and data-loading layers as Server Components.

## Composition Rules

- A Server Component **can** import and render a Client Component.
- A Client Component **cannot** import a Server Component (it would serialize it, losing server-only behavior).
- You can pass Server Component output as **children props** into a Client Component — this is the idiomatic pattern for mixing both in one tree.

```tsx
// ✅ Server Component wrapping a Client Component
import SearchInput from './SearchInput'; // 'use client'

export default async function Page() {
  const data = await fetch('/api/data').then(r => r.json());
  return (
    <main>
      <SearchInput />          {/* Client Component — interactive */}
      <ResultsList data={data} /> {/* Server Component — data from server */}
    </main>
  );
}
```

## Common Mistakes

- Marking every component `'use client'` out of habit — eliminates all server rendering benefits and increases bundle size.
- Fetching data in a Client Component when a Server Component could do it — adds client-side waterfall latency.
- Importing a Server Component inside a Client Component — not supported; restructure using children props instead.
- Not understanding that `'use client'` marks a boundary, not a component type — all children of a Client Component also become client-rendered unless they are passed as props from a Server Component above.

## Business Context

Proper use of the server/client boundary is the single most impactful architectural decision in a Next.js App Router project. It directly affects JavaScript bundle size, Time to First Byte, secret exposure risk, and SEO quality. Teams that default to `'use client'` everywhere lose the core value proposition of the App Router.

## Related Knowledge

- [Next.js Overview](./nextjs-overview.md)
- [Next.js Rendering Strategies](./nextjs-rendering-strategies.md)
- [Data Fetching in App Router](../../playbooks/nextjs/nextjs-data-fetching.md)
- [Create Your First Next.js App](../../playbooks/nextjs/nextjs-first-app.md)