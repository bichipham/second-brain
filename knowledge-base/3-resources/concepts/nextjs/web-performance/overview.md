---
title: Web Performance — Core Web Vitals Overview
type: concept
tags: [performance, core-web-vitals, inp, lcp, tbt, cls, nextjs]
updated: 2026-05-26
sources:
  - https://web.dev/articles/vitals
  - https://web.dev/explore/how-to-optimize-inp
---

# Web Performance — Core Web Vitals Overview

Tổng quan về các metrics quan trọng nhất để đo và tối ưu hiệu năng web.

---

## Core Web Vitals là gì?

**Core Web Vitals** là bộ 3 metrics do Google định nghĩa, đại diện cho 3 khía cạnh quan trọng nhất của user experience:

| Metric | Đo cái gì | Ngưỡng tốt |
|---|---|---|
| **[LCP](./lcp.md)** — Largest Contentful Paint | Tốc độ load nội dung chính | ≤ 2.5s |
| **[INP](./inp.md)** — Interaction to Next Paint | Tốc độ phản hồi interaction | ≤ 200ms |
| **CLS** — Cumulative Layout Shift | Ổn định layout (không nhảy layout) | ≤ 0.1 |

> Google dùng Core Web Vitals làm **ranking signal** cho Search từ 2021.

---

## Metric bổ sung quan trọng

| Metric | Đo cái gì | Loại |
|---|---|---|
| **[TBT](./tbt.md)** — Total Blocking Time | Tổng thời gian main thread bị block | Lab proxy cho INP |
| **FCP** — First Contentful Paint | Thời gian để render bất kỳ content nào | Load |
| **TTFB** — Time to First Byte | Thời gian server response | Server |
| **TTI** — Time to Interactive | Thời gian trang fully interactive | Load |

---

## Field vs Lab Data

| | Field Data | Lab Data |
|---|---|---|
| **Nguồn** | Real users (Chrome UX Report) | Synthetic test (Lighthouse) |
| **Metrics** | LCP, INP, CLS (+ FCP, TTFB) | LCP, TBT, CLS, FCP, Speed Index |
| **Tool** | PageSpeed Insights, Search Console | Lighthouse, WebPageTest, DevTools |
| **Dùng để** | Biết user thực tế trải nghiệm gì | Debug, tìm nguyên nhân |
| **INP** | ✅ Chính xác | ⚠️ Khó đo (dùng TBT làm proxy) |

---

## Mối quan hệ giữa các Metrics

```
Server responds nhanh (TTFB thấp)
        ↓
Content render sớm (FCP thấp)
        ↓
Nội dung chính hiện ra (LCP thấp)   ← Core Web Vital
        ↓
Trang interactive, JS load xong
(TBT thấp → ít Long Tasks)
        ↓
User interact → browser phản hồi nhanh (INP thấp)  ← Core Web Vital
        ↓
Layout không nhảy (CLS thấp)        ← Core Web Vital
```

---

## Quick Wins theo từng Metric

### LCP (Load Speed)
- Dùng `priority` cho hero image (`<Image priority>` trong Next.js)
- `font-display: swap` cho web fonts
- Dùng SSG/ISR thay SSR (giảm TTFB)
- Inline critical CSS

### INP (Responsiveness)
- Break up Long Tasks với `scheduler.yield()`
- Defer non-critical work sau `requestAnimationFrame`
- Giảm DOM size (< 1,400 nodes)
- Move heavy computation vào Web Worker

### TBT (Lab proxy của INP)
- Code splitting: `next/dynamic`, dynamic `import()`
- Third-party scripts: dùng `strategy="afterInteractive"` (Next.js)
- Facade pattern cho heavy widgets

### CLS (Visual Stability)
- Set `width` và `height` cho images (tránh layout shift khi load)
- `aspect-ratio` cho responsive media
- Tránh insert content above existing content
- `font-display: optional` nếu muốn tránh FOUT hoàn toàn

---

## Tools tổng hợp

| Tool | Dùng cho | Link |
|---|---|---|
| PageSpeed Insights | Field + Lab, quick check | [pagespeed.web.dev](https://pagespeed.web.dev/) |
| Chrome DevTools Performance | Debug chi tiết Long Tasks, LCP | Built-in |
| Lighthouse | Lab audit, TBT, opportunities | Built-in / CLI |
| WebPageTest | Filmstrip, waterfall, advanced | [webpagetest.org](https://webpagetest.org/) |
| Search Console | Field data trend theo thời gian | [search.google.com/search-console](https://search.google.com/search-console) |
| `web-vitals` library | Đo INP/LCP/CLS trong code | `npm i web-vitals` |

```javascript
// Đo tất cả Core Web Vitals trong một chỗ
import { onINP, onLCP, onCLS } from 'web-vitals';

function sendToAnalytics({ name, value, rating }) {
  fetch('/api/analytics', {
    method: 'POST',
    body: JSON.stringify({ metric: name, value, rating }),
  });
}

onINP(sendToAnalytics);
onLCP(sendToAnalytics);
onCLS(sendToAnalytics);
```

---

## Tài liệu chi tiết

- [INP — Interaction to Next Paint](./inp.md) — responsiveness, input delay, long tasks, web workers
- [LCP — Largest Contentful Paint](./lcp.md) — load speed, images, fonts, SSR/SSG
- [TBT — Total Blocking Time](./tbt.md) — lab metric, long tasks, script evaluation, third-party
