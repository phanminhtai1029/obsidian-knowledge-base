---
title: "Async/Await"
section: 04-Async-JavaScript
tags: [async, await, try-catch, parallel, sequential, fresher, frontend]
related:
  - "[[02-Event-Loop]]"
  - "[[04-Promise]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [MDN, javascript.info]
---

# Async/Await

> [!summary] TL;DR
> **`async function`** luôn trả về Promise. **`await`** tạm dừng async function tại chỗ, đợi Promise resolve, rồi tiếp tục — trông như sync nhưng **không block** main thread. Xử lý lỗi với `try/catch`. **Sequential** (từng `await` một) vs **Parallel** (`Promise.all` + `await`) — chọn đúng để tránh mất hiệu năng. **Top-level await** (ES2022) dùng được ở module scope mà không cần wrap.

> [!tip] 🎯 Hiểu trong 30 giây
> **`async/await` = cách viết code "chờ đợi" mà nhìn như code thường, khỏi lồng `.then()`.** `await` nghĩa là *"đứng đây đợi cái này xong rồi mới đi tiếp"* — nhưng **chỉ đợi trong hàm này thôi**, phần còn lại của trang vẫn chạy bình thường (không treo UI).
>
> Nó chỉ là **lớp áo đường trên Promise** — bên dưới vẫn là Promise. `await` ≈ `.then()`, còn bắt lỗi thì dùng **`try/catch`** quen thuộc thay cho `.catch()`.
>
> **Bẫy hiệu năng SỐ 1 (hay ra thi):** `await` *tuần tự* những việc *không phụ thuộc nhau* → cộng dồn thời gian. 3 việc mỗi việc 500ms mà `await` lần lượt = 1500ms. Nếu chúng độc lập → khởi động hết rồi `await Promise.all([...])` = chỉ 500ms. Quy tắc: **chỉ `await` tuần tự khi việc sau CẦN kết quả việc trước.**

---

## 1. Khái niệm

### `async` function

```javascript
// async function LUÔN trả về Promise
async function greet(name) {
  return `Hello, ${name}!`;
}

greet('Alice').then(msg => console.log(msg)); // "Hello, Alice!"
// Tương đương:
// function greet(name) { return Promise.resolve(`Hello, ${name}!`); }
```

### `await` expression

- Chỉ dùng được bên trong `async function` (hoặc top-level module)
- `await expression` tạm dừng hàm, đợi Promise settle
- Nếu fulfilled → return value
- Nếu rejected → throw error (bắt bằng `try/catch`)

```javascript
async function loadUser(id) {
  const response = await fetch(`/api/users/${id}`); // đợi Promise
  const user     = await response.json();           // đợi Promise tiếp theo
  return user;
}
// Tương đương Promise chain:
// function loadUser(id) {
//   return fetch(`/api/users/${id}`)
//     .then(res => res.json());
// }
```

---

## 2. Cú pháp / API

### 2.1 Cú pháp cơ bản

```javascript
// 1. Function declaration
async function fetchData(url) {
  const data = await fetch(url).then(r => r.json());
  return data;
}

// 2. Arrow function
const fetchData = async (url) => {
  const data = await fetch(url).then(r => r.json());
  return data;
};

// 3. Method trong object/class
class ApiService {
  async getUser(id) {
    return fetch(`/api/users/${id}`).then(r => r.json());
  }
}

// 4. IIFE để dùng await ở module scope (pre-ES2022)
(async () => {
  const data = await fetchData('/api/config');
  console.log(data);
})();

// 5. Top-level await (ES2022 — chỉ trong module)
// const config = await fetchData('/api/config'); // OK trong .mjs hoặc type="module"
```

### 2.2 Error Handling với `try/catch/finally`

```javascript
async function loadProfile(userId) {
  try {
    const user    = await fetch(`/api/users/${userId}`).then(r => {
      if (!r.ok) throw new Error(`HTTP ${r.status}`);
      return r.json();
    });
    const profile = await fetch(`/api/profiles/${userId}`).then(r => r.json());
    return { user, profile };
  } catch (err) {
    // Bắt mọi lỗi trong try block — cả network error lẫn JS errors
    console.error('Lỗi:', err.message);
    return null; // hoặc rethrow: throw err
  } finally {
    console.log('Loading complete'); // luôn chạy
  }
}

// Gọi
const data = await loadProfile(1);
if (data) {
  console.log(data.user.name);
}
```

### 2.3 Sequential vs Parallel — rất quan trọng!

```javascript
const delay = (ms, val) => new Promise(res => setTimeout(() => res(val), ms));
const getUser   = () => delay(500, { name: 'Alice' });
const getOrders = () => delay(500, [{ id: 1 }]);
const getPromos = () => delay(500, ['SALE50']);

// --- SEQUENTIAL: tốn 1500ms (3 x 500ms) ---
async function loadSequential() {
  const user   = await getUser();    // đợi 500ms
  const orders = await getOrders();  // đợi thêm 500ms
  const promos = await getPromos();  // đợi thêm 500ms
  return { user, orders, promos };
}

// --- PARALLEL với Promise.all: chỉ tốn 500ms! ---
async function loadParallel() {
  const [user, orders, promos] = await Promise.all([
    getUser(),    // bắt đầu NGAY
    getOrders(),  // bắt đầu NGAY (song song)
    getPromos(),  // bắt đầu NGAY (song song)
  ]);
  return { user, orders, promos };
}

// --- PARALLEL khi cần handle error riêng ---
async function loadParallelSafe() {
  const [userResult, ordersResult] = await Promise.allSettled([
    getUser(),
    getOrders(),
  ]);
  const user   = userResult.status   === 'fulfilled' ? userResult.value   : null;
  const orders = ordersResult.status === 'fulfilled' ? ordersResult.value : [];
  return { user, orders };
}
```

> [!warning] Anti-pattern: await trong loop
> ```javascript
> // SAI — sequential, tốn N * latency
> for (const id of userIds) {
>   const user = await getUser(id); // đợi từng cái
>   process(user);
> }
>
> // ĐÚNG — parallel, tốn 1 * latency
> const users = await Promise.all(userIds.map(id => getUser(id)));
> users.forEach(user => process(user));
> ```

```
★ Insight ─────────────────────────────────────
• Lỗi hiệu năng số 1 với async/await: await TUẦN TỰ những việc ĐỘC LẬP. Mỗi
  `await` dừng hàm tới khi xong → 3 việc 500ms thành 1500ms. Nếu chúng không phụ
  thuộc nhau → khởi động hết rồi Promise.all (chỉ 500ms). Quy tắc: "await tuần tự
  CHỈ khi việc sau CẦN kết quả việc trước".
• Bẫy tinh vi: `await` bên trong `forEach`/`map` KHÔNG được vòng lặp chờ — forEach
  kệ Promise trả về, chạy tiếp ngay. Muốn tuần tự thật → for...of; muốn song song
  → `Promise.all(arr.map(async ...))`. Và nhớ: async/await chỉ là VỎ trên Promise
  ([[04-Promise]]) — quên await một async fn = nuốt luôn lỗi của nó.
─────────────────────────────────────────────────
```

### 2.4 Async Iteration (for await...of)

```javascript
// Dùng với async iterators (streams, pagination)
async function* paginate(baseUrl) {
  let page = 1;
  while (true) {
    const res  = await fetch(`${baseUrl}?page=${page}`);
    const data = await res.json();
    if (data.items.length === 0) return;
    yield data.items;
    page++;
  }
}

async function loadAllPages() {
  for await (const items of paginate('/api/products')) {
    console.log('Page:', items.length, 'items');
    // render items...
  }
}
```

### 2.5 Lỗi thường gặp và cách xử lý

```javascript
// Pattern 1: Catch mỗi await riêng
async function robustLoad(id) {
  const user = await getUser(id).catch(() => null); // fail → null, không throw
  if (!user) return { error: 'user not found' };

  const orders = await getOrders(user.id).catch(() => []); // fail → empty array
  return { user, orders };
}

// Pattern 2: Rethrow với context
async function loadWithContext(id) {
  let user;
  try {
    user = await getUser(id);
  } catch (err) {
    throw new Error(`Failed to load user ${id}: ${err.message}`);
  }
  return user;
}

// Pattern 3: Centralized error handling wrapper
function withErrorHandling(asyncFn) {
  return async (...args) => {
    try {
      return [null, await asyncFn(...args)];
    } catch (err) {
      return [err, null];
    }
  };
}

const safeGetUser = withErrorHandling(getUser);
const [err, user] = await safeGetUser(1);
if (err) console.error(err.message);
else console.log(user.name);
```

---

## 3. Ví dụ minh họa

### Ví dụ 1: Dashboard loading với parallel + error isolation

```javascript
async function loadDashboard(userId) {
  // Khởi động TẤT CẢ requests song song
  const [statsRes, notifRes, feedRes] = await Promise.allSettled([
    fetch(`/api/stats/${userId}`).then(r => r.json()),
    fetch(`/api/notifications/${userId}`).then(r => r.json()),
    fetch(`/api/feed/${userId}`).then(r => r.json()),
  ]);

  // Mỗi phần có fallback riêng nếu fail
  const stats         = statsRes.status  === 'fulfilled' ? statsRes.value   : { sales: 0, visits: 0 };
  const notifications = notifRes.status  === 'fulfilled' ? notifRes.value   : [];
  const feed          = feedRes.status   === 'fulfilled' ? feedRes.value    : [];

  return { stats, notifications, feed };
}

// Render dashboard — không bị block dù 1 section fail
async function renderDashboard(userId) {
  const listEl = document.querySelector('#feed-list');
  const statsEl = document.querySelector('#stats');

  try {
    const data = await loadDashboard(userId);

    statsEl.textContent = `Sales: ${data.stats.sales}`;

    data.feed.forEach(item => {
      const li = document.createElement('li');
      li.textContent = item.title;
      listEl.appendChild(li);
    });
  } catch (err) {
    const p = document.createElement('p');
    p.textContent = 'Không thể tải dashboard.';
    document.body.appendChild(p);
  }
}
```

### Ví dụ 2: Retry với exponential backoff

```javascript
async function fetchWithRetry(url, { retries = 3, baseDelay = 300 } = {}) {
  for (let attempt = 0; attempt <= retries; attempt++) {
    try {
      const res = await fetch(url);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return await res.json();
    } catch (err) {
      if (attempt === retries) throw err; // hết lần thử → rethrow

      const delay = baseDelay * Math.pow(2, attempt); // 300 → 600 → 1200ms
      console.warn(`Attempt ${attempt + 1} failed. Retrying in ${delay}ms...`);
      await new Promise(res => setTimeout(res, delay)); // đợi
    }
  }
}

// Dùng
try {
  const data = await fetchWithRetry('/api/flaky-endpoint', { retries: 3, baseDelay: 500 });
  console.log(data);
} catch (err) {
  console.error('All retries exhausted:', err.message);
}
```

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Pitfall 1: `await` trong vòng lặp — tình cờ sequential
> ```javascript
> // CHẬM: await trong forEach — forEach không chờ Promise
> userIds.forEach(async id => {
>   const user = await getUser(id); // chạy "song song" nhưng forEach không await!
>   process(user); // thứ tự không đảm bảo
> });
>
> // ĐÚNG nếu muốn parallel:
> await Promise.all(userIds.map(id => getUser(id).then(process)));
>
> // ĐÚNG nếu cần sequential (có lý do):
> for (const id of userIds) { await getUser(id).then(process); }
> ```

> [!warning] Pitfall 2: Không `await` async function — bỏ sót lỗi
> ```javascript
> async function saveData() { /* ... */ }
>
> saveData(); // Không await → lỗi trong saveData bị ignore silently!
>
> await saveData(); // ĐÚNG — bắt được lỗi
> ```

> [!warning] Pitfall 3: `async/await` trong event listener không automatic error handling
> ```javascript
> button.addEventListener('click', async () => {
>   const data = await fetchData(); // lỗi ở đây bị "swallowed"!
> });
>
> // FIX: wrap trong try/catch
> button.addEventListener('click', async () => {
>   try {
>     const data = await fetchData();
>     render(data);
>   } catch (err) {
>     showError(err.message);
>   }
> });
> ```

> [!tip] `async/await` là syntactic sugar — vẫn là Promise bên dưới
> `async function` trả về Promise. `await p` = `p.then(v => ...)`. Khi debug error stack, sẽ thấy Promise chain. Hiểu Promise giúp debug async/await tốt hơn.

---

## 5. Câu hỏi phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Bắt lỗi với async/await, khác gì `.catch()`?"
> *"Với async/await em bọc đoạn code có await trong khối `try/catch`: nếu một Promise bị reject thì `await` sẽ throw lỗi và rơi vào `catch`, y như bắt lỗi đồng bộ. Còn với Promise truyền thống thì em nối `.catch()` vào cuối chuỗi. Về bản chất hai cách tương đương vì async/await chỉ là lớp áo trên Promise, nhưng `try/catch` đọc tự nhiên hơn, gom được nhiều `await` trong cùng một khối, và dùng được `finally` để dọn dẹp. Một lưu ý là nếu em gọi một async function mà quên `await` thì lỗi bên trong sẽ bị nuốt mất, nên em luôn await hoặc gắn `.catch()`."*

> [!note] 🧠 Mẹo nhớ
> **`await` = "đợi xong mới đi tiếp" (chỉ trong hàm này).** Bắt lỗi bằng **`try/catch`** (≈ `.catch()`). Bẫy: **việc độc lập đừng await tuần tự → `Promise.all`.** Quên `await` = **nuốt lỗi.**

**Q1: `async/await` là gì? Khác gì với Promise?**

> `async/await` là syntactic sugar trên Promise, giúp viết async code có dạng synchronous — dễ đọc, dễ debug. `async function` luôn trả về Promise. `await` tạm dừng function tại điểm đó, đợi Promise settle, rồi tiếp tục với giá trị hoặc throw error. **Bên dưới vẫn là Promise** — mỗi `await` tương đương một `.then()`. Không block main thread.

**Q2: Làm thế nào chạy nhiều async operations song song với async/await?**

> Dùng `await Promise.all([op1(), op2(), op3()])` — khởi động tất cả cùng lúc, rồi await tất cả. Sai lầm phổ biến: `await op1(); await op2();` — sequential, tốn N lần latency. `Promise.all` song song = chỉ tốn thời gian của operation chậm nhất.

**Q3: Làm thế nào handle lỗi với async/await?**

> Dùng `try/catch/finally` — gần giống error handling đồng bộ. `try` block chứa code có thể throw (bao gồm rejected Promise từ `await`). `catch` nhận error object. `finally` luôn chạy dù success hay fail. Có thể: (1) catch ở mức cao nhất, (2) `.catch()` ngay trên từng `await fetchX().catch(() => fallback)`, (3) wrapper pattern trả về `[error, data]` tuple.

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Viết `asyncPool(tasks, limit)` — chạy mảng async functions với tối đa `limit` concurrent tasks. Khi 1 task xong mới khởi động task kế tiếp (không dùng `Promise.all` toàn bộ một lúc).

- [ ] **Bài 2:** Viết `useFetch(url)` hook-like function trả về `{ data, loading, error }`. Dùng async/await, xử lý loading state trước khi gọi, error state khi fail, data state khi thành công.

---

## 7. Liên kết

- [[02-Event-Loop]] — await hoạt động thế nào với Microtask Queue
- [[04-Promise]] — Promise là nền tảng của async/await
- [[03-Callback-va-Callback-Hell]] — Tại sao async/await tốt hơn callback
- [[../02-DOM-Event/09-Fetch-API|Fetch API]] — async/await với fetch trong DOM context
