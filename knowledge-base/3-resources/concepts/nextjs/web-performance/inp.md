---
title: Web Performance — Interaction to Next Paint (INP)
type: concept
tags: [performance, inp, core-web-vitals, nextjs, javascript, rendering, web-workers]
updated: 2026-05-26
sources:
  - https://web.dev/explore/how-to-optimize-inp
  - https://web.dev/articles/inp
  - https://web.dev/articles/optimize-inp
  - https://web.dev/articles/optimize-input-delay
  - https://web.dev/articles/script-evaluation-and-long-tasks
  - https://web.dev/articles/dom-size-and-interactivity
  - https://web.dev/articles/off-main-thread
---

# Web Performance — Interaction to Next Paint (INP)

Tài liệu tổng hợp từ [web.dev/explore/how-to-optimize-inp](https://web.dev/explore/how-to-optimize-inp) — toàn bộ kiến thức về INP từ cơ bản đến thực chiến.

---

## 1. INP là gì?

**Interaction to Next Paint (INP)** là một **Core Web Vital** metric đo lường khả năng phản hồi của trang web với user interaction. INP thay thế **First Input Delay (FID)** từ tháng 3/2024.

INP quan sát **tất cả** các lần click, tap, và keystroke trong suốt vòng đời của trang, rồi báo cáo một giá trị duy nhất đại diện cho trải nghiệm chung. Giá trị đó là **interaction tệ nhất** (bỏ qua outlier).

> Chrome data: 90% thời gian người dùng ở trên một trang là **sau khi trang load xong** → INP quan trọng hơn FID rất nhiều.

### Ngưỡng đánh giá

| Ngưỡng | Đánh giá |
|---|---|
| ≤ 200ms | ✅ Tốt |
| 201ms – 500ms | ⚠️ Cần cải thiện |
| > 500ms | ❌ Kém |

Đo tại **75th percentile** của tất cả page loads, phân theo mobile và desktop.

### FID vs INP

| | FID | INP |
|---|---|---|
| Đo | Input delay của interaction **đầu tiên** | **Tất cả** interactions |
| Bao gồm | Chỉ input delay | Input delay + processing + presentation |
| Loại | Load responsiveness | Overall responsiveness |

---

## 2. Anatomy của một Interaction

Mỗi interaction gồm **3 phần**:

```
[User taps] → [Input Delay] → [Event Handlers run] → [Presentation Delay] → [Frame painted]
              ←───────────────────── Total INP ──────────────────────────→
```

### 2.1 Input Delay
Thời gian từ lúc user tương tác đến khi event callbacks bắt đầu chạy. Nguyên nhân phổ biến:
- Long tasks đang chạy trên main thread
- `setInterval` / `setTimeout` callbacks chặn main thread
- Script evaluation đang xảy ra

### 2.2 Processing Duration
Thời gian để tất cả event callbacks chạy xong. Đây là phần code của bạn viết.

### 2.3 Presentation Delay
Thời gian từ sau khi callbacks xong đến khi browser paint frame mới. Nguyên nhân:
- DOM quá lớn → layout/style recalculation tốn kém
- Layout thrashing
- Client-side HTML rendering

### Interaction types được đo (3 loại)
- 🖱️ Click bằng chuột
- 👆 Tap trên touchscreen
- ⌨️ Nhấn phím (keyboard/onscreen)

> Hover, scroll, zoom **không được đo** trong INP.

---

## 3. Đo INP

### Trong thực tế (Field data)
Cách tốt nhất: **Real User Monitoring (RUM)** — cung cấp INP value + context (interaction nào gây ra, xảy ra lúc load hay sau, loại interaction gì).

Quick start với [PageSpeed Insights](https://pagespeed.web.dev/) → dùng Chrome UX Report (CrUX):
- Data level: origin hoặc URL
- Hạn chế: không có context chi tiết như RUM

### Đo bằng JavaScript (`web-vitals` library)
```javascript
import { onINP } from 'web-vitals';

// Đo và gửi INP trong mọi trường hợp cần report
onINP(console.log);
```

> Gửi metric khi trang bị background (`visibilitychange` event), không chỉ khi unload — vì mobile OS thường không chạy unload callback.

### Trong lab (DevTools)
- Dùng **Performance panel** của Chrome DevTools
- Interact với trang trong lúc load (main thread bận nhất)
- Follow user flows thông thường
- **Total Blocking Time (TBT)** có thể dùng như proxy metric nếu chưa có field data

---

## 4. Tối ưu Input Delay

### 4.1 Tránh recurring timers gây main thread work

`setInterval` đặc biệt nguy hiểm vì chạy liên tục, dễ xung đột với interactions.

```javascript
// ❌ setInterval chạy mỗi 100ms — có thể block interaction
setInterval(() => {
  doHeavyWork(); // 50ms work
}, 100);

// ✅ Nếu cần polling — dùng setTimeout đệ quy với early return
function poll() {
  if (shouldStop) return;
  doLightWork();
  setTimeout(poll, 100); // Nhường main thread giữa các lần
}
```

### 4.2 Tránh Long Tasks

Long task = task chạy > 50ms trên main thread. Mục tiêu: chia nhỏ chúng.

```javascript
// ❌ Một task lớn block main thread
function processAllData(items) {
  items.forEach(item => processItem(item)); // có thể > 50ms
}

// ✅ Yield giữa các chunk
async function processAllData(items) {
  for (const item of items) {
    processItem(item);
    // Yield sau mỗi item để browser xử lý interactions
    await scheduler.yield(); // hoặc: await new Promise(r => setTimeout(r, 0))
  }
}
```

### 4.3 Cẩn thận với Interaction Overlap

Khi user tương tác nhanh (ví dụ: gõ phím liên tục vào autocomplete field):

```javascript
// ✅ Debounce: giới hạn số lần callback chạy
import { debounce } from 'lodash';

const handleInput = debounce(async (value) => {
  const results = await fetchSuggestions(value);
  renderResults(results);
}, 300);

// ✅ AbortController: cancel request cũ khi có request mới
let currentController = null;

async function fetchSuggestions(query) {
  if (currentController) currentController.abort();
  currentController = new AbortController();

  try {
    const res = await fetch(`/api/suggest?q=${query}`, {
      signal: currentController.signal,
    });
    return res.json();
  } catch (e) {
    if (e.name === 'AbortError') return []; // ignore
    throw e;
  }
}
```

---

## 5. Tối ưu Event Callbacks (Processing Duration)

### 5.1 Yield thường xuyên với `scheduler.yield`

```javascript
button.addEventListener('click', async () => {
  // Việc quan trọng — cập nhật UI ngay
  updateUIImmediately();

  // Yield để browser paint frame trước
  await scheduler.yield();

  // Sau đó làm việc nặng hơn
  await doExpensiveWork();
});
```

### 5.2 Yield để rendering chạy sớm hơn

Pattern: chỉ giữ lại critical UI work trong sync code, defer phần còn lại.

```javascript
textBox.addEventListener('input', (e) => {
  // Chỉ update text box — critical, phải xảy ra trước frame tiếp theo
  updateTextBox(e);

  // Defer tất cả non-critical work ra sau frame
  requestAnimationFrame(() => {
    setTimeout(() => {
      const text = textBox.textContent;
      updateWordCount(text);   // non-critical
      checkSpelling(text);     // non-critical
      saveChanges(text);       // non-critical
    }, 0);
  });
});
```

Tại sao `requestAnimationFrame` + `setTimeout(, 0)`?
- `rAF`: đảm bảo chạy trước lần paint tiếp theo
- `setTimeout(, 0)` bên trong `rAF`: defer sang task mới **sau** khi frame đã paint
- Kết quả: user thấy visual feedback ngay, heavy work chạy sau

### 5.3 Tránh Layout Thrashing

Layout thrashing = đọc layout properties ngay sau khi thay đổi styles → browser phải tính toán layout synchronously.

```javascript
// ❌ Layout thrashing — đọc sau khi write
element.style.width = '100px';
const height = element.offsetHeight; // Force sync layout!
element.style.height = height + 'px';

// ✅ Tách read và write
// Read trước
const height = element.offsetHeight;
// Write sau
requestAnimationFrame(() => {
  element.style.width = '100px';
  element.style.height = height + 'px';
});
```

Properties hay gây layout thrashing: `offsetWidth`, `offsetHeight`, `getBoundingClientRect()`, `scrollTop`, `clientHeight`...

---

## 6. Tối ưu Presentation Delay

### 6.1 DOM Size

**Lighthouse cảnh báo** khi DOM > 800 nodes, **fail** khi > 1400 nodes.

Đo DOM size trong console:
```javascript
document.querySelectorAll('*').length;
```

DOM lớn ảnh hưởng INP vì:
1. **Initial render**: CSSOM phức tạp hơn → layout/paint tốn kém
2. **Sau interactions**: mọi DOM mutation trigger layout recalculation
3. **Memory**: `querySelectorAll` trên DOM lớn tốn RAM

Cách giảm DOM:
- Flatten DOM structure (bỏ div wrapper thừa)
- Dùng **fragments** trong React/Vue/Svelte:
  ```jsx
  // ❌ Wrapper div thừa
  return <div><ComponentA /><ComponentB /></div>;

  // ✅ Fragment — không thêm DOM node
  return <><ComponentA /><ComponentB /></>;
  ```
- **Additive approach**: chỉ render DOM khi user cần, thay vì render hết rồi hide

### 6.2 CSS `content-visibility`

Lazy render off-screen elements — browser skip rendering cho đến khi element gần viewport:

```css
/* Các section dài dưới fold */
.below-fold-section {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px; /* ước tính height để tránh layout shift */
}
```

### 6.3 CSS Selector Complexity

CSS selector phức tạp → browser phải traverse DOM nhiều hơn mỗi khi recalculate styles.

```css
/* ❌ Phức tạp — traverse DOM sâu */
.nav > ul > li:nth-child(2) > a.active ~ span.badge {}

/* ✅ Flat, specific */
.nav-badge--active {}
```

---

## 7. Script Evaluation và Long Tasks

### Vấn đề
Mỗi JS file được load → browser phải: **parse → compile → execute**. Trên device yếu, bước này tạo long tasks block main thread.

### Giải pháp: Chia nhỏ scripts

Mỗi `<script>` element = 1 evaluation task riêng. Script nhỏ → task nhỏ → ít block main thread hơn.

```html
<!-- ❌ Một bundle lớn = một long task -->
<script src="bundle.js"></script> <!-- 500KB -->

<!-- ✅ Nhiều script nhỏ = nhiều task nhỏ hơn -->
<script src="core.js"></script>      <!-- 100KB -->
<script src="features.js"></script>  <!-- 100KB -->
<script src="analytics.js"></script> <!-- 50KB -->
```

**Target**: mỗi script ≤ 100KB (sau khi nén).

### Code Splitting với Dynamic `import()`

```javascript
// ❌ Load hết ngay lúc startup
import { heavyFeature } from './heavy-feature.js';

// ✅ Chỉ load khi cần
button.addEventListener('click', async () => {
  const { heavyFeature } = await import('./heavy-feature.js');
  heavyFeature();
});
```

Bundler (webpack, Rollup, esbuild) tự động tách dynamic import thành file riêng.

### Trade-offs cần cân nhắc

| Yếu tố | Script ít, lớn | Script nhiều, nhỏ |
|---|---|---|
| Compression | Tốt hơn | Kém hơn |
| Cache invalidation | Xấu (cả bundle invalid) | Tốt (chỉ file thay đổi invalid) |
| Network requests | Ít | Nhiều hơn |
| Main thread blocking | Nhiều | Ít hơn |

---

## 8. Web Workers — Off Main Thread

### Vấn đề
Browser có 1 main thread duy nhất per tab. Mọi JS, layout, paint đều cạnh tranh trên đó.

### Giải pháp: Web Workers
Chạy JS trên thread riêng biệt — không block main thread, không có DOM access.

```javascript
// main.js
const worker = new Worker('./worker.js');

// Gửi data sang worker
worker.postMessage({ type: 'PROCESS', data: largeDataset });

// Nhận kết quả về
worker.addEventListener('message', (event) => {
  renderResults(event.data);
});

// worker.js
addEventListener('message', (event) => {
  if (event.data.type === 'PROCESS') {
    const result = heavyComputation(event.data.data);
    postMessage(result);
  }
});
```

### Dùng Comlink để code gọn hơn

```javascript
// worker.js
import { expose } from 'comlink';

expose({
  async processData(items) {
    return items.map(expensiveTransform);
  }
});

// main.js
import { wrap } from 'comlink';

const worker = new Worker('./worker.js', { type: 'module' });
const api = wrap(worker);

// Gọi như function thường — trả về Promise
const results = await api.processData(largeDataset);
```

### Nên move gì vào Web Worker?

✅ Phù hợp:
- Data processing (sort, filter, transform lớn)
- Crypto operations
- Image processing
- State management logic (Redux store)
- Autocomplete / search indexing

❌ Không thể (vì cần DOM):
- DOM manipulation
- Web Audio API
- WebRTC
- UI framework rendering

### Lợi ích OMT (Off-Main-Thread)

- Giảm INP: main thread rảnh hơn → phản hồi interaction nhanh hơn
- Giảm LCP: bớt long tasks chặn render LCP element
- Tăng độ tin cậy trên thiết bị yếu (feature phones)

---

## 9. Case Studies Thực Tế

| Công ty | Cải thiện INP | Kết quả kinh doanh |
|---|---|---|
| QuintoAndar | Giảm 80% | Tăng 36% conversions |
| Disney+ Hotstar | Giảm 61% | Tăng 100% weekly card views |
| redBus | Cải thiện INP | Tăng 7% sales |
| Trendyol | Giảm 50% | Tăng 1% click-through rate |
| Economic Times | Significant | Better engagement |
| PubConsent CMP | Giảm 64% | Tăng 1.5% ad viewability |

---

## 10. Checklist Tối Ưu INP

### Giảm Input Delay
- [ ] Kiểm tra và giảm `setInterval` callbacks
- [ ] Break up long tasks (> 50ms) thành tasks nhỏ
- [ ] Dùng `scheduler.yield()` hoặc `setTimeout(fn, 0)` để yield
- [ ] Debounce input handlers tốn kém
- [ ] Dùng `AbortController` để cancel fetch request cũ

### Giảm Processing Duration
- [ ] Yield sau UI update để rendering chạy sớm hơn
- [ ] Tách critical UI work ra khỏi non-critical work
- [ ] Dùng `requestAnimationFrame` + `setTimeout(, 0)` pattern
- [ ] Tránh layout thrashing (tách read/write DOM)
- [ ] Move heavy computation vào Web Worker

### Giảm Presentation Delay
- [ ] Giảm DOM size (< 1400 nodes)
- [ ] Flatten DOM structure, dùng fragments
- [ ] Dùng `content-visibility: auto` cho off-screen sections
- [ ] Đơn giản hoá CSS selectors
- [ ] Tránh render large HTML từ JavaScript

### Script Loading
- [ ] Code splitting: chia bundle thành nhiều file nhỏ (≤ 100KB mỗi file)
- [ ] Dynamic `import()` cho features không cần ngay
- [ ] Dùng `modulepreload` cho ES modules quan trọng
- [ ] Đảm bảo bundle có hash trong filename (cache invalidation)

---

## 11. Tools để đo và debug INP

| Tool | Dùng cho |
|---|---|
| [PageSpeed Insights](https://pagespeed.web.dev/) | Field data nhanh từ CrUX |
| Chrome DevTools → Performance panel | Lab profiling, tìm slow interactions |
| Chrome DevTools → Performance Monitor | Xem DOM size realtime |
| `web-vitals` library | Đo INP trong code |
| Lighthouse | Audit DOM size, TBT |
| [PerformanceObserver](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver) | Custom RUM |

```javascript
// Measure INP với web-vitals + gửi về analytics
import { onINP } from 'web-vitals';

onINP(({ value, rating, entries }) => {
  // Gửi về analytics
  fetch('/analytics', {
    method: 'POST',
    body: JSON.stringify({
      metric: 'INP',
      value,
      rating,          // 'good' | 'needs-improvement' | 'poor'
      interaction: entries[0]?.target?.tagName,
    }),
  });
});
```

---

## Xem thêm

- [TypeScript Nâng Cao](./typescript-advanced.md) — typing cho performance utilities
- [Next.js Server vs Client Components](./nextjs-server-vs-client-components.md) — giảm JS client-side
- [Next.js Overview](./nextjs-overview.md)
- [web.dev — Optimize INP collection](https://web.dev/explore/how-to-optimize-inp)
