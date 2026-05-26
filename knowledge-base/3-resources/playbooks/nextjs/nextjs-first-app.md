---
title: Create Your First Next.js App
type: playbook
tags: [nextjs, react, setup, app-router, onboarding, quickstart]
updated: 2026-05-25
sources:
  - notes/2026-05-25-hoc-nextjs.md
---

## Problem

You need to bootstrap a [Next.js](../../concepts/nextjs/nextjs-overview.md) project, understand its directory structure, and have a working app with routing, data fetching, and both server and client components.

## Investigation Steps

Before writing code, confirm:
- Node.js ≥ 18 is installed (`node -v`)
- You are using the **App Router** (`app/` directory) — this is the current standard, not the legacy Pages Router (`pages/`)
- You understand the [Server vs Client Component boundary](../../concepts/nextjs/nextjs-server-vs-client-components.md)

## Resolution

### Step 1 — Bootstrap the project

```bash
npx create-next-app@latest my-app --typescript --eslint --tailwind --app
cd my-app
npm run dev
```

Open `http://localhost:3000` to confirm the app is running.

### Step 2 — Understand the `app/` directory

```
app/
├── layout.tsx       # Root layout — wraps every page
├── page.tsx         # Home route: /
├── globals.css
└── favicon.ico
```

Special files Next.js recognizes per route segment:

| File | Purpose |
|---|---|
| `page.tsx` | Route UI — makes a segment publicly accessible |
| `layout.tsx` | Shared wrapper that persists across navigations |
| `loading.tsx` | Shown while page data loads (implicit Suspense boundary) |
| `error.tsx` | Error boundary for the route and its children |
| `not-found.tsx` | 404 handler for the segment |

### Step 3 — Create three routes

```
app/
├── page.tsx                   # /
├── about/
│   └── page.tsx               # /about
└── posts/
    └── [slug]/
        └── page.tsx           # /posts/:slug  (dynamic route)
```

Access the dynamic param via `params`:

```tsx
// app/posts/[slug]/page.tsx
export default function PostPage({ params }: { params: { slug: string } }) {
  return <h1>Post: {params.slug}</h1>;
}
```

### Step 4 — Add a shared layout

```tsx
// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <nav>/* navigation */</nav>
        {children}
      </body>
    </html>
  );
}
```

### Step 5 — Server Component with data fetch

```tsx
// app/posts/page.tsx  (Server Component — no 'use client')
async function getPosts() {
  const res = await fetch('https://jsonplaceholder.typicode.com/posts', {
    next: { revalidate: 60 },
  });
  return res.json();
}

export default async function PostsPage() {
  const posts = await getPosts();
  return (
    <ul>
      {posts.slice(0, 10).map((p: { id: number; title: string }) => (
        <li key={p.id}>{p.title}</li>
      ))}
    </ul>
  );
}
```

### Step 6 — Client Component with interactivity

```tsx
// components/SearchInput.tsx
'use client';
import { useState } from 'react';

export default function SearchInput() {
  const [query, setQuery] = useState('');
  return (
    <input
      value={query}
      onChange={e => setQuery(e.target.value)}
      placeholder="Search…"
    />
  );
}
```

### Step 7 — Add `loading.tsx` and `error.tsx`

```tsx
// app/posts/loading.tsx
export default function Loading() {
  return <p>Loading posts…</p>;
}
```

```tsx
// app/posts/error.tsx
'use client';
export default function Error({ reset }: { reset: () => void }) {
  return (
    <div>
      <p>Something went wrong.</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

### Step 8 — Add an API route handler

```ts
// app/api/hello/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  return NextResponse.json({ message: 'Hello from the API' });
}
```

## Recommended Practice Projects

| Project | Concepts covered |
|---|---|
| Simple blog | Static routes, dynamic `[slug]`, metadata, SSG |
| Task manager dashboard | Client state, form submission, loading/error states, Server Actions |
| Mini ecommerce | Product listing, search/filter, ISR, cart state |

## Prevention

- Always add `loading.tsx` and `error.tsx` from the start — retrofitting them later is error-prone.
- Keep `'use client'` at leaf components; do not pollute parent layouts with client-side concerns.
- Do not fetch data in Client Components when a Server Component can do it instead.

## Related Knowledge

- [Next.js Overview](../../concepts/nextjs/nextjs-overview.md)
- [Server Components vs Client Components](../../concepts/nextjs/nextjs-server-vs-client-components.md)
- [Next.js Rendering Strategies](../../concepts/nextjs/nextjs-rendering-strategies.md)
- [Data Fetching in App Router](./nextjs-data-fetching.md)
- [Route Handlers and Server Actions](./nextjs-route-handlers-and-server-actions.md)