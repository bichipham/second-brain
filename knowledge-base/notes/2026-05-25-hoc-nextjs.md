# Học Next.js - ghi chú ban đầu

Mục tiêu: có một note gốc để học Next.js từ tổng quan đến thực hành, sau đó có thể ingest vào KB thành các tài liệu kiểu concept và playbook.

## Next.js là gì

- Next.js là framework React thiên về full-stack web app.
- Nó giải quyết các phần mà React thuần không cung cấp sẵn: routing, server rendering, data fetching, API handlers, tối ưu ảnh, build, deploy.
- Mental model quan trọng: không chỉ là thư viện UI; đây là framework tổ chức cả frontend lẫn một phần backend.

## Khi nào nên dùng

- Cần SEO tốt cho trang public.
- Cần render nhanh cho landing page, blog, docs, ecommerce.
- Muốn dùng React nhưng không muốn tự ghép quá nhiều phần như router + SSR + bundling + API.
- Muốn deploy nhanh trên Vercel hoặc hạ tầng Node-compatible.

## Những khái niệm cần nắm đầu tiên

### 1. App Router

- Next.js hiện tại ưu tiên `app/` directory.
- Cấu trúc thư mục quyết định route.
- Mỗi folder có thể có `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`.
- `layout.tsx` giúp chia sẻ UI khung giữa nhiều trang.

### 2. Server Components vs Client Components

- Mặc định trong App Router, component là Server Component.
- Server Component phù hợp cho lấy dữ liệu, render trên server, không cần state phía client.
- Client Component cần thêm `'use client'` ở đầu file.
- Chỉ dùng Client Component khi cần state, effect, event handler, browser API.
- Ghi nhớ: đừng đẩy mọi thứ sang client nếu không cần.

### 3. Rendering strategies

- Static rendering: trang build sẵn, nhanh và cache tốt.
- Dynamic rendering: render theo request, phù hợp dữ liệu thay đổi theo người dùng hoặc request.
- Streaming: gửi từng phần UI khi sẵn sàng.
- ISR / revalidation: cập nhật lại trang tĩnh theo chu kỳ hoặc theo trigger.

### 4. Data Fetching

- Có thể `fetch()` trực tiếp trong Server Component.
- Next.js thêm cơ chế cache và revalidation cho `fetch`.
- Cần hiểu các option như cache mặc định, `no-store`, `revalidate`.
- Phân biệt rõ: fetch ở server thường tốt hơn fetch ở client nếu không cần tương tác realtime.

### 5. Route Handlers

- Trong App Router có thể tạo API bằng `app/api/.../route.ts`.
- Hữu ích cho webhook, proxy, logic backend đơn giản.
- Không phải lúc nào cũng thay thế backend riêng; chỉ hợp với scope vừa và nhỏ hoặc BFF layer.

### 6. Server Actions

- Cho phép submit form hoặc gọi logic server trực tiếp từ UI.
- Giảm boilerplate API endpoint trong một số use case.
- Cần hiểu tradeoff: tiện nhưng phải kiểm soát validation, auth, side effects.

## Routing - phần cần nhớ

- File-system routing: folder map thành URL.
- Dynamic route: `[id]`.
- Nested route: lồng folder.
- Route group: `(marketing)`, `(dashboard)` để nhóm logic mà không ảnh hưởng URL.
- Parallel routes và intercepting routes là tính năng nâng cao, học sau.

## Các tính năng hữu ích khác

- `next/image` để tối ưu ảnh.
- `next/link` để điều hướng client-side.
- Metadata API để set title, description, Open Graph.
- Middleware cho auth, redirect, rewrite ở edge/runtime phù hợp.
- Partial prerendering: cần theo dõi khi dùng version mới.

## Những câu hỏi cần tự trả lời khi học

- Khi nào chọn Server Component, khi nào chọn Client Component?
- Khi nào fetch ở server, khi nào fetch ở client?
- Khi nào cần dynamic rendering thay vì static?
- Có nên dùng Server Actions hay route handlers cho form submit?
- Nếu app cần auth, cache, và dashboard realtime thì kiến trúc nào hợp lý?

## Lộ trình học gợi ý

### Giai đoạn 1 - Nắm mental model

- Hiểu Next.js là framework React full-stack.
- Hiểu App Router.
- Hiểu Server vs Client Components.
- Tạo app đầu tiên và đi qua vài route đơn giản.

### Giai đoạn 2 - Data và rendering

- Học fetch trong Server Component.
- Học caching và revalidation.
- Học loading UI, error handling, not-found.
- Thử static page, dynamic page, và revalidate page.

### Giai đoạn 3 - Backend integration

- Tạo route handlers.
- Thử submit form.
- Tìm hiểu Server Actions.
- Kết nối database hoặc mock API.

### Giai đoạn 4 - Production concerns

- Metadata, SEO, image optimization.
- Auth.
- Performance profiling.
- Deploy và environment variables.

## Bài thực hành nên làm

### Bài 1 - Blog đơn giản

- Trang home.
- Trang danh sách bài viết.
- Trang chi tiết bài viết theo slug.
- Metadata cơ bản.

### Bài 2 - Dashboard task manager

- Login mock.
- Danh sách task.
- Tạo và cập nhật task.
- Loading, error, empty state.

### Bài 3 - Mini ecommerce

- Product listing.
- Product detail.
- Search/filter.
- Cart đơn giản.

## Các lỗi hoặc hiểu nhầm dễ gặp

- Nghĩ rằng mọi component đều nên là Client Component.
- Fetch dữ liệu ở client quá nhiều làm mất lợi thế server rendering.
- Không phân biệt cache mặc định của `fetch` trong Next.js.
- Trộn logic server/client thiếu ranh giới.
- Lạm dụng route handlers như backend đầy đủ trong khi domain đã phức tạp.

## Checklist học nhanh

- Tạo app Next.js mới.
- Đi qua cấu trúc `app/`.
- Tạo 3 route: `/`, `/about`, `/posts/[slug]`.
- Tạo `layout.tsx` chung.
- Tạo một Server Component fetch data.
- Tạo một Client Component có search input.
- Tạo `loading.tsx` và `error.tsx`.
- Tạo một API route handler.
- Deploy thử.

## Gợi ý tách note sau khi ingest

- Concept: Next.js overview.
- Concept: Server Components vs Client Components.
- Concept: Next.js rendering strategies.
- Playbook: Tạo app Next.js đầu tiên.
- Playbook: Data fetching trong App Router.
- Playbook: Route handlers và form submit.

## Cần đọc tiếp

- App Router conventions.
- Data fetching + caching + revalidation.
- Server Actions.
- Authentication patterns.
- Deployment and performance optimization.