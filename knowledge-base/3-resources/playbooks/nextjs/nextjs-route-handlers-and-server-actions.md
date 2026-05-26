---
title: Route Handlers and Server Actions in Next.js
type: playbook
tags: [nextjs, api, route-handlers, server-actions, forms, mutations, backend, app-router]
updated: 2026-05-25
sources:
  - notes/2026-05-25-hoc-nextjs.md
---

## Problem

You need to handle form submissions, webhooks, or backend logic within a [Next.js](../../concepts/nextjs/nextjs-overview.md) App Router project. Two mechanisms exist — Route Handlers and Server Actions — and choosing the wrong one adds unnecessary complexity or creates security gaps.

## Investigation Steps

Choose your mechanism based on these questions:

| Question | Route Handler | Server Action |
|---|---|---|
| Does an external client (mobile, webhook, third-party) need to call this? | Yes | No |
| Is this a form submit or UI-triggered mutation with no external caller? | No | Yes |
| Do you need to control HTTP method, status codes, and headers explicitly? | Yes | No |
| Is the domain logic complex enough to warrant proper HTTP semantics? | Yes | No |

## Resolution

### Route Handlers

Create `app/api/<path>/route.ts`. Supported HTTP methods: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, `OPTIONS`.

```ts
// app/api/posts/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET() {
  const posts = await db.posts.findMany();
  return NextResponse.json(posts);
}

export async function POST(req: NextRequest) {
  const body = await req.json();

  // Always validate at the boundary
  if (!body.title) {
    return NextResponse.json({ error: 'title required' }, { status: 400 });
  }

  const post = await db.posts.create({ data: body });
  return NextResponse.json(post, { status: 201 });
}
```

**Dynamic route params:**

```ts
// app/api/posts/[id]/route.ts
export async function GET(
  req: NextRequest,
  { params }: { params: { id: string } }
) {
  const post = await db.posts.findUnique({ where: { id: params.id } });
  if (!post) return NextResponse.json({ error: 'Not found' }, { status: 404 });
  return NextResponse.json(post);
}
```

**When to use Route Handlers:**
- Webhook endpoints (Stripe, GitHub, CMS)
- Proxying requests to internal APIs that must not be exposed to the browser
- BFF (Backend for Frontend) aggregation layer for mobile or third-party clients
- Endpoints that require explicit HTTP status codes and response headers

### Server Actions

Server Actions are async functions marked `'use server'` that run on the server and can be called directly from forms or UI event handlers — no explicit API endpoint needed.

```ts
// app/actions.ts
'use server';

import { revalidatePath } from 'next/cache';

export async function createTask(formData: FormData) {
  const title = formData.get('title') as string;

  // Always validate — Server Actions are HTTP POST requests under the hood
  if (!title?.trim()) throw new Error('Title is required');

  // Check auth before mutating
  const session = await getSession();
  if (!session) throw new Error('Unauthorized');

  await db.tasks.create({ data: { title, userId: session.userId } });
  revalidatePath('/tasks');
}
```

Use in a Server Component form (no JavaScript required for basic submit):

```tsx
// app/tasks/new/page.tsx
import { createTask } from '../actions';

export default function NewTaskPage() {
  return (
    <form action={createTask}>
      <input name="title" required />
      <button type="submit">Create</button>
    </form>
  );
}
```

Use programmatically from a Client Component:

```tsx
'use client';
import { createTask } from '../actions';
import { useTransition } from 'react';

export default function TaskForm() {
  const [isPending, startTransition] = useTransition();

  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    startTransition(() => createTask(formData));
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="title" />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating…' : 'Create'}
      </button>
    </form>
  );
}
```

### Comparison

| Dimension | Route Handler | Server Action |
|---|---|---|
| Primary use case | External clients, webhooks, BFF | Internal forms, UI mutations |
| Boilerplate | More (explicit endpoint file) | Less (function-as-action) |
| Auth enforcement | Standard HTTP (bearer token, session cookie) | Must be explicit inside function |
| Testability | Easy — standard HTTP requests | Harder — requires test harness or integration test |
| Error surface | HTTP status codes | Thrown exceptions, `useFormState` |
| External callability | Yes — any HTTP client | No — Next.js internal only |

## Prevention

- **Validate all inputs** — Server Actions are HTTP POST requests; missing validation is a common security gap.
- **Always check auth before mutating** — Server Actions do not automatically inherit auth context from middleware.
- **Do not use Server Actions as a full backend** — for complex domains with many entity types and business rules, use Route Handlers or a dedicated backend service.
- **Do not use Route Handlers for simple form-only flows** — that is Server Actions' job; route handlers add unnecessary boilerplate there.
- **Avoid heavy side effects without error boundaries** — wrap Server Action calls in `try/catch` and use `useFormState` to surface errors to users.

## Related Knowledge

- [Next.js Overview](../../concepts/nextjs/nextjs-overview.md)
- [Data Fetching in App Router](./nextjs-data-fetching.md)
- [Server Components vs Client Components](../../concepts/nextjs/nextjs-server-vs-client-components.md)
- [Create Your First Next.js App](./nextjs-first-app.md)