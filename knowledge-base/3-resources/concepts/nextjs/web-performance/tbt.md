---
title: Web Performance — Total Blocking Time (TBT)
type: concept
tags: [performance, tbt, lab-metric, long-tasks, javascript, main-thread]
updated: 2026-05-26
sources:
  - https://web.dev/articles/tbt
  - https://web.dev/articles/optimize-long-tasks
---

# Total Blocking Time (TBT)

TBT là **lab metric** đo tổng thời gian main thread bị block sau FCP — khiến browser không thể phản hồi user input. TBT thường được dùng như **proxy metric cho INP** trong môi trường lab (vì INP khó đo chính xác trong lab).

---

## 1. TBT là gì?

TBT = tổng **blocking time** của tất cả Long Tasks xảy ra sau [FCP](https://web.dev/articles/fcp) cho đến [TTI](https://web.dev/articles/tti) (hoặc hết trace).

**Long Task** = task chạy trên main thread **> 50ms**.

**Blocking time** của 1 task = phần vượt quá 50ms.

```
Task duration = 250ms
Blocking time = 250 - 50 = 200ms   ← phần gây hại

Task duration = 35ms
Blocking time = 0                   ← không phải Long Task
```

### Ví dụ tính TBT

| Task | Duration | Blocking Time |
|---|---|---|
| Task 1 | 250ms | 200ms |
| Task 2 | 90ms | 40ms |
| Task 3 | 35ms | 0ms |
| Task 4 | 30ms | 0ms |
| Task 5 | 155ms | 105ms |
| **Tổng** | 560ms | **345ms TBT** |

---

## 2. Ngưỡng đánh giá

| Ngưỡng | Đánh giá |
|---|---|
| < 200ms | ✅ Tốt |
| 200ms – 600ms | ⚠️ Cần cải thiện |
| > 600ms | ❌ Kém |

Đo trên **thiết bị mobile trung bình** (Lighthouse dùng Moto G4 làm baseline).

---

## 3. TBT vs TTI vs INP

### TBT vs TTI

TTI coi trang là "interactive" khi main thread **free 5 giây liên tục**. Nhưng TTI không nói lên mức độ nghiêm trọng của blocking.

Ví dụ hai trang đều có TTI = 10s:
- **Trang A**: 3 tasks × 51ms mỗi cái → TBT = **3ms** (gần như không ảnh hưởng)
- **Trang B**: 1 task × 10,000ms → TBT = **9,950ms** (cực kỳ tệ)

TTI bằng nhau nhưng trải nghiệm hoàn toàn khác → TBT phản ánh thực tế tốt hơn TTI.

### TBT vs INP

| | TBT | INP |
|---|---|---|
| Môi trường | **Lab** (Lighthouse, WebPageTest) | **Field** (thực tế từ user) |
| Đo | Blocking time trong load phase | Latency của mọi interactions |
| Sau load | Không tính (mặc định) | ✅ Tính toàn bộ vòng đời trang |
| Dùng như | Proxy/indicator cho INP | Ground truth của responsiveness |

> TBT cao → khả năng INP cao. Nhưng không phải lúc nào cũng đúng: TBT có thể flag vấn đề không thật (user không interact lúc đó), hoặc bỏ sót vấn đề (interactions sau load phase).

**Ưu tiên**: Tối ưu INP trước, TBT sẽ tốt lên theo.

---

## 4. Đo TBT

### Công cụ

| Tool | Cách dùng |
|---|---|
| **Lighthouse** | Chạy audit → xem "Total Blocking Time" trong Performance score |
| **WebPageTest** | Xem TBT trong timeline |
| Chrome DevTools → Performance | Xem Long Tasks thủ công (màu đỏ trên main thread) |
| PageSpeed Insights | Lab data từ Lighthouse |

### Xem Long Tasks trong DevTools

1. Mở **Performance panel** → Record → Reload trang
2. Nhìn vào **Main thread** track
3. Tasks có **tam giác đỏ** góc trên phải = Long Tasks
4. Click vào task → xem call stack để biết nguyên nhân

### Đo Long Tasks bằng JavaScript

```javascript
// PerformanceObserver — monitor Long Tasks realtime
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log('Long Task detected:', {
      duration: entry.duration,
      blockingTime: entry.duration - 50,
      startTime: entry.startTime,
    });
  }
});

observer.observe({ type: 'longtask', buffered: true });

// Tính TBT thủ công
let tbt = 0;
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.duration > 50) {
      tbt += entry.duration - 50;
    }
  }
}).observe({ type: 'longtask', buffered: true });
```

---

## 5. Cách cải thiện TBT

TBT cải thiện = giảm Long Tasks. Có 4 hướng chính:

### 5.1 Break up Long Tasks (Yielding)

```javascript
// ❌ Một task lớn 300ms — block main thread
function processItems(items) {
  for (const item of items) {
    heavyProcess(item); // mỗi cái 3ms → 100 items = 300ms
  }
}

// ✅ Yield sau mỗi chunk — chia thành nhiều tasks nhỏ
async function processItems(items) {
  for (const item of items) {
    heavyProcess(item);
    // Nhường main thread để browser xử lý interactions
    await scheduler.yield();
    // Hoặc nếu không có scheduler.yield:
    // await new Promise(resolve => setTimeout(resolve, 0));
  }
}
```

**`scheduler.yield`** (Chrome 129+) ưu tiên hơn `setTimeout(0)` vì nó giữ task trong queue hiện tại thay vì đẩy xuống cuối:

```javascript
// Cách dùng scheduler.yield với fallback
function yieldToMain() {
  if ('scheduler' in window && 'yield' in scheduler) {
    return scheduler.yield();
  }
  return new Promise(resolve => setTimeout(resolve, 0));
}

async function processInChunks(items, chunkSize = 10) {
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    processChunk(chunk);
    await yieldToMain(); // yield sau mỗi chunk
  }
}
```

---

### 5.2 Giảm JavaScript được load khi startup

Script evaluation gây Long Tasks trong load phase — ảnh hưởng trực tiếp TBT.

```javascript
// ❌ Load tất cả lúc startup
import { featureA } from './feature-a'; // 50KB
import { featureB } from './feature-b'; // 80KB
import { featureC } from './feature-c'; // 60KB

// ✅ Chỉ load khi cần (code splitting)
const featureA = await import('./feature-a');

// React lazy loading
const HeavyChart = React.lazy(() => import('./HeavyChart'));
```

**Next.js** tự động code-split theo route. Thêm dynamic import cho heavy components:

```jsx
import dynamic from 'next/dynamic';

// Chỉ load khi component mount, không block initial render
const HeavyEditor = dynamic(() => import('./HeavyEditor'), {
  loading: () => <p>Loading editor...</p>,
  ssr: false, // nếu component dùng browser APIs
});
```

---

### 5.3 Giảm ảnh hưởng của Third-party Scripts

Third-party scripts (analytics, ads, chat widgets) thường gây Long Tasks mà bạn không kiểm soát được code.

```html
<!-- ❌ Load đồng thời với trang — cạnh tranh main thread -->
<script src="https://analytics.example.com/script.js"></script>

<!-- ✅ Load sau khi trang interactive -->
<script src="https://analytics.example.com/script.js"
        defer
        data-timeout="3000">
</script>
```

Trong Next.js:
```jsx
import Script from 'next/script';

// afterInteractive: load sau hydration
<Script src="https://analytics.example.com/script.js"
        strategy="afterInteractive" />

// lazyOnload: load khi browser idle
<Script src="https://chat-widget.example.com/widget.js"
        strategy="lazyOnload" />

// worker: load trong Web Worker (không block main thread)
<Script src="https://analytics.example.com/script.js"
        strategy="worker" />
```

**Facade pattern** — thay thế third-party widget bằng placeholder, load thật khi user interact:

```jsx
// Thay vì load YouTube embed ngay (nặng ~500KB)
// Dùng thumbnail + load player khi click
const [showPlayer, setShowPlayer] = useState(false);

if (!showPlayer) {
  return (
    <div onClick={() => setShowPlayer(true)}>
      <img src={`https://img.youtube.com/vi/${videoId}/hqdefault.jpg`} alt="Play video" />
      <PlayButton />
    </div>
  );
}

return <iframe src={`https://youtube.com/embed/${videoId}?autoplay=1`} />;
```

---

### 5.4 Dùng Web Workers cho heavy computation

```javascript
// main.js — không block main thread
const worker = new Worker('./data-processor.js');

worker.postMessage({ items: largeDataset });
worker.onmessage = (e) => {
  renderResults(e.data); // chỉ update UI trên main thread
};

// data-processor.js — chạy trên thread riêng
self.onmessage = (e) => {
  const result = e.data.items.map(heavyTransform); // thoải mái chạy lâu
  self.postMessage(result);
};
```

---

## 6. TBT trong Lighthouse Score

TBT đóng góp **30%** vào Lighthouse Performance score (cao nhất trong các metrics):

| Metric | Trọng số |
|---|---|
| FCP | 10% |
| Speed Index | 10% |
| **TBT** | **30%** |
| LCP | 25% |
| CLS | 25% |

Lighthouse chạy trên **simulated Moto G4 + slow 4G**. TBT thực tế của user trên thiết bị tốt hơn sẽ nhỏ hơn Lighthouse report.

---

## 7. Checklist TBT

### Giảm JS load phase
- [ ] Code splitting: dynamic `import()` cho features không cần ngay
- [ ] `next/dynamic` cho heavy components (Next.js)
- [ ] Mỗi JS bundle ≤ 100KB (sau gzip)
- [ ] Remove unused JavaScript (tree shaking)

### Tránh Long Tasks
- [ ] Yield trong long loops với `scheduler.yield()` hoặc `setTimeout(0)`
- [ ] Chia event handlers thành tasks nhỏ
- [ ] Move heavy computation vào Web Worker
- [ ] Debounce/throttle expensive event handlers

### Third-party scripts
- [ ] Audit third-party scripts (Lighthouse → "Reduce third-party impact")
- [ ] Dùng `defer` / `async` / `strategy="afterInteractive"` (Next.js)
- [ ] Implement facade pattern cho heavy widgets (YouTube, maps, chat)
- [ ] Xem xét remove scripts không cần thiết

### Kiểm tra
- [ ] Chạy Lighthouse → xem TBT score
- [ ] Dùng Performance panel → identify Long Tasks màu đỏ
- [ ] Test trên mobile (throttle CPU 4x trong DevTools)

---

## 8. Debug TBT trong DevTools — Step by step

1. Mở Chrome DevTools → **Performance** tab
2. Click ⚙️ → bật **CPU throttling: 4x slowdown** (simulate mobile)
3. Click **Record** → Reload trang → Stop sau 5s
4. Nhìn vào **Main** thread track:
   - Tasks có **đường đỏ** = Long Tasks
   - Hover để xem duration
5. Click vào Long Task → **Bottom-Up** tab → xem function nào tốn thời gian nhất
6. Tìm trong **Call Tree** để biết nguyên nhân (script evaluation, event handler, layout...)

---

## Xem thêm

- [INP — Interaction to Next Paint](./inp.md)
- [LCP — Largest Contentful Paint](./lcp.md)
- [Web Performance Overview](./overview.md)
- [web.dev — TBT](https://web.dev/articles/tbt)
- [web.dev — Optimize Long Tasks](https://web.dev/articles/optimize-long-tasks)
