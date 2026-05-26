---
title: Next.js Overview
type: concept
tags: [nextjs, react, framework, full-stack, ssr, frontend]
updated: 2026-05-25
sources:
  - notes/2026-05-25-hoc-nextjs.md
---

## Definition

Next.js is a full-stack React framework that extends React beyond pure UI rendering. While React only handles the view layer, Next.js adds routing, server-side rendering, data fetching, API handlers, image optimization, and build/deploy tooling out of the box.

Key mental model: Next.js is not just a UI library — it organizes both frontend and parts of the backend in a single opinionated structure. The primary entry point is the `app/` directory (App Router), which replaces the older `pages/` directory approach.

## Synonyms & Aliases

- "Next" — common shorthand
- React meta-framework
- React full-stack framework

## When to Use

- Public pages requiring good SEO (landing pages, blogs, docs, ecommerce product pages)
- Fast initial render without manually composing SSR + router + bundling
- Teams who want the React ecosystem without gluing together routing, SSR, and API layers
- Projects targeting Vercel or any Node-compatible hosting infrastructure

## When Not to Use

- Purely client-side SPAs with no SEO requirements where a Vite + React setup already exists
- The backend domain is complex enough to warrant a dedicated service — Next.js [Route Handlers](../../playbooks/nextjs/nextjs-route-handlers-and-server-actions.md) are not a full backend replacement for complex domains

## Core Capabilities

| Feature | What it provides |
|---|---|
| App Router | File-system routing via `app/` directory |
| Server Components | Components that render on the server by default |
| Data Fetching | Extended `fetch` with built-in caching and revalidation |
| Route Handlers | HTTP API endpoints inside `app/api/` |
| Server Actions | Server-side mutations callable from UI |
| `next/image` | Automatic image optimization and lazy loading |
| `next/link` | Client-side navigation with prefetching |
| Metadata API | `<head>` management for SEO and Open Graph |
| Middleware | Edge-runtime logic for auth, redirects, rewrites |

## Business Context

Next.js is the default choice for modern React-based web products requiring public SEO, fast time-to-first-byte, and a unified full-stack developer experience. It removes infrastructure decisions that slow down early-stage product teams and scales well into production with ISR, streaming, and edge deployment.

## Related Knowledge

- [Server Components vs Client Components](./nextjs-server-vs-client-components.md)
- [Next.js Rendering Strategies](./nextjs-rendering-strategies.md)
- [Create Your First Next.js App](../../playbooks/nextjs/nextjs-first-app.md)
- [Data Fetching in App Router](../../playbooks/nextjs/nextjs-data-fetching.md)
- [Route Handlers and Server Actions](../../playbooks/nextjs/nextjs-route-handlers-and-server-actions.md)