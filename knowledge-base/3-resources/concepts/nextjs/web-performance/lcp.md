---
title: Web Performance — Largest Contentful Paint (LCP)
type: concept
tags: [performance, lcp, core-web-vitals, nextjs, images, fonts, rendering]
updated: 2026-05-26
sources:
  - https://web.dev/articles/lcp
  - https://web.dev/articles/optimize-lcp
---

# Largest Contentful Paint (LCP)

LCP đo thời gian để **phần nội dung lớn nhất** của trang render xong và hiển thị với người dùng. Đây là Core Web Vital quan trọng nhất cho **perceived load speed** — cảm giác trang load nhanh hay chậm.

---

## 1. LCP là gì?

LCP báo cáo thời gian render của **image, text block, hoặc video lớn nhất** trong viewport, tính từ lúc user bắt đầu navigate.

> **Mục tiêu**: Cho người dùng biết rằng trang đã sẵn sàng để đọc/dùng. Hero image load xong = trang có nội dung chính.

### Ngưỡng đánh giá

| Ngưỡng | Đánh giá |
|---|---|
| ≤ 2.5 giây | ✅ Tốt |
| 2.5s – 4.0s | ⚠️ Cần cải thiện |
| > 4.0 giây | ❌ Kém |

Đo tại **75th percentile** của page loads, phân theo mobile và desktop.

---

## 2. Những element nào được tính là LCP?

Browser xét các element sau:

- `<img>` (bao gồm GIF, animated PNG — dùng thời điểm first frame)
- `<image>` bên trong `<svg>`
- `<video>` (lấy thời điểm poster image load hoặc first frame, cái nào sớm hơn)
- Element có `background-image` dùng CSS `url()`
- Block-level elements chứa text nodes (heading, paragraph...)

**Bị loại trừ** (không tính là LCP candidate):
- Element có `opacity: 0`
- Element phủ toàn viewport (thường là background)
- Placeholder image, low-entropy image

### LCP thay đổi trong quá trình load

LCP không cố định — browser liên tục cập nhật khi trang load thêm content:

```
Frame 1: <h1> text render → LCP candidate = h1
Frame 2: Hero <img> load xong → LCP candidate = img (lớn hơn h1)
Frame 3: User interact → LCP chốt tại img
```

LCP chốt khi **user đầu tiên interact** (tap, scroll, keypress).

---

## 3. Các phần tạo nên LCP

```
[Navigation start]
     ↓
[TTFB - Time to First Byte]         ← Server response time
     ↓
[Resource load delay]                ← Thời gian trước khi browser biết cần fetch
     ↓
[Resource load time]                 ← Download time của ảnh/font
     ↓
[Render delay]                       ← Thời gian từ khi resource download xong đến khi render
     ↓
[LCP]
```

Mỗi phần đều có thể optimize riêng.

---

## 4. Đo LCP

### JavaScript (`web-vitals` library — khuyến nghị)
```javascript
import { onLCP } from 'web-vitals';

onLCP(({ value, rating, entries }) => {
  console.log('LCP:', value, rating); // rating: 'good' | 'needs-improvement' | 'poor'

  // Tìm xem element nào là LCP
  const lcpEntry = entries[entries.length - 1];
  console.log('LCP element:', lcpEntry.element);
});
```

### Raw PerformanceObserver
```javascript
new PerformanceObserver((list) => {
  const entries = list.getEntries();
  const lastEntry = entries[entries.length - 1];
  console.log('LCP candidate:', lastEntry.startTime, lastEntry.element);
}).observe({ type: 'largest-contentful-paint', buffered: true });
```

### Tools
| Tool | Loại | Dùng cho |
|---|---|---|
| [PageSpeed Insights](https://pagespeed.web.dev/) | Field + Lab | Quick check, CrUX data |
| Chrome DevTools → Performance | Lab | Debug chi tiết |
| Lighthouse | Lab | Audit + suggestions |
| [WebPageTest](https://webpagetest.org/) | Lab | Filmstrip, waterfall |
| Search Console | Field | Trend theo thời gian |

---

## 5. Tối ưu LCP

### 5.1 Giảm TTFB (Time to First Byte)

TTFB chậm = mọi thứ khác đều chậm theo.

```javascript
// Next.js: dùng ISR hoặc SSG thay vì SSR thuần
// pages/product/[id].tsx

// ❌ SSR — render mỗi request
export async function getServerSideProps({ params }) {
  const product = await fetchProduct(params.id);
  return { props: { product } };
}

// ✅ ISR — cache, revalidate theo interval
export async function getStaticProps({ params }) {
  const product = await fetchProduct(params.id);
  return {
    props: { product },
    revalidate: 60, // revalidate sau 60 giây
  };
}
```

Các cách khác giảm TTFB:
- Dùng CDN gần người dùng
- Edge Runtime trong Next.js (`export const runtime = 'edge'`)
- Cache database queries với Redis
- HTTP caching headers (`Cache-Control: s-maxage=...`)

---

### 5.2 Loại bỏ Render-Blocking Resources

CSS và JS trong `<head>` block render → LCP bị delay.

```html
<!-- ❌ Render-blocking CSS không critical -->
<link rel="stylesheet" href="/styles/full-bundle.css">

<!-- ✅ Inline critical CSS, defer phần còn lại -->
<style>
  /* Chỉ inline CSS cần cho above-the-fold */
  .hero { background: #fff; font-size: 2rem; }
</style>
<link rel="preload" href="/styles/below-fold.css" as="style"
      onload="this.onload=null;this.rel='stylesheet'">

<!-- ❌ JS block render -->
<script src="/app.js"></script>

<!-- ✅ Defer JS không critical -->
<script src="/app.js" defer></script>
<!-- hoặc async nếu không cần đợi DOM -->
<script src="/analytics.js" async></script>
```

Trong Next.js — Script component tự handle:
```jsx
import Script from 'next/script';

// strategy="afterInteractive" — load sau khi trang interactive
<Script src="/analytics.js" strategy="afterInteractive" />

// strategy="lazyOnload" — load khi browser idle
<Script src="/chat-widget.js" strategy="lazyOnload" />
```

---

### 5.3 Tối ưu LCP Resource (image/font)

#### Preload LCP image

```html
<!-- Báo browser fetch sớm nhất có thể -->
<link rel="preload" as="image" href="/hero.webp"
      fetchpriority="high">
```

Trong Next.js:
```jsx
import Image from 'next/image';

// priority prop = preload + fetchpriority="high"
<Image
  src="/hero.webp"
  alt="Hero image"
  width={1200}
  height={600}
  priority  // ← quan trọng cho LCP image
/>
```

#### Dùng đúng format ảnh

```html
<!-- ✅ WebP/AVIF nhỏ hơn JPEG/PNG 25-50% -->
<picture>
  <source srcset="/hero.avif" type="image/avif">
  <source srcset="/hero.webp" type="image/webp">
  <img src="/hero.jpg" alt="Hero" loading="eager">
</picture>
```

Next.js Image tự convert sang WebP/AVIF.

#### Tránh lazy loading cho LCP image

```jsx
// ❌ LCP image mà lazy load → delay LCP
<Image src="/hero.jpg" loading="lazy" />

// ✅ LCP image phải load sớm
<Image src="/hero.jpg" priority />  // Next.js
// hoặc
<img src="/hero.jpg" loading="eager" fetchpriority="high">
```

#### Font display

```css
/* ❌ font-display: block — text vô hình cho đến khi font load */
@font-face {
  font-family: 'MyFont';
  font-display: block;
}

/* ✅ font-display: swap — hiện fallback font ngay, swap khi load xong */
@font-face {
  font-family: 'MyFont';
  font-display: swap;
}
```

Preload font quan trọng:
```html
<link rel="preload" href="/fonts/heading.woff2"
      as="font" type="font/woff2" crossorigin>
```

---

### 5.4 Giảm Render Delay

Render delay = thời gian từ khi resource download xong đến khi render. Nguyên nhân:

**CSS blocking render:**
```css
/* ❌ Visibility hidden delay render */
.hero-image { visibility: hidden; }

/* JS sau đó mới show */
document.querySelector('.hero-image').style.visibility = 'visible';
```

**JS block main thread:**
Long tasks trên main thread delay paint. → Xem [INP — Optimize Long Tasks](./inp.md#5-tối-ưu-event-callbacks-processing-duration).

---

### 5.5 SSR / SSG với Next.js

Client-side rendering chậm LCP vì HTML trống, JS phải chạy mới có content.

```jsx
// ❌ Client-side only — LCP chậm
'use client';
export default function Page() {
  const [data, setData] = useState(null);
  useEffect(() => { fetchData().then(setData); }, []);
  return data ? <HeroSection data={data} /> : null;
}

// ✅ Server Component — HTML có sẵn content
export default async function Page() {
  const data = await fetchData();
  return <HeroSection data={data} />;
}
```

---

## 6. Checklist LCP

### Server & Network
- [ ] TTFB < 600ms (đo với PageSpeed Insights)
- [ ] Dùng CDN cho static assets
- [ ] HTTP caching headers đúng
- [ ] Dùng SSG/ISR thay SSR khi có thể (Next.js)

### Resources
- [ ] LCP image dùng `priority` / `fetchpriority="high"` / `<link rel="preload">`
- [ ] LCP image không dùng `loading="lazy"`
- [ ] Dùng WebP hoặc AVIF cho images
- [ ] `font-display: swap` cho web fonts
- [ ] Preload fonts critical

### Render
- [ ] Inline critical CSS (above-the-fold)
- [ ] Defer non-critical CSS và JS
- [ ] Không dùng `visibility: hidden` / `opacity: 0` cho LCP element
- [ ] Dùng Server Components cho content chính (Next.js App Router)
- [ ] Không lazy-load above-the-fold content

### Kiểm tra
- [ ] Xác định LCP element trong DevTools (Performance panel → LCP marker)
- [ ] Kiểm tra LCP trên mobile (thường chậm hơn desktop 2-3x)

---

## 7. LCP trong Next.js — Quick Reference

```jsx
// app/page.tsx — Server Component fetch data
export default async function HomePage() {
  const heroData = await getHeroContent(); // server-side

  return (
    <main>
      {/* LCP image — priority + đúng size */}
      <Image
        src={heroData.imageUrl}
        alt={heroData.alt}
        width={1200}
        height={600}
        priority
        sizes="100vw"
      />
      {/* LCP text — server rendered, không cần JS */}
      <h1>{heroData.headline}</h1>
    </main>
  );
}

// next.config.js — optimize images
/** @type {import('next').NextConfig} */
const config = {
  images: {
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920],
  },
};
```

---

## Xem thêm

- [INP — Interaction to Next Paint](./inp.md)
- [TBT — Total Blocking Time](./tbt.md)
- [Web Performance Overview](./overview.md)
- [web.dev — LCP](https://web.dev/articles/lcp)
- [web.dev — Optimize LCP](https://web.dev/articles/optimize-lcp)
