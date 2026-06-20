---
title: "Async JavaScript: Tổng quan"
section: 04-Async-JavaScript
tags: [async, concurrency, js, single-thread, fresher, frontend]
related:
  - "[[02-Event-Loop]]"
  - "[[04-Promise]]"
difficulty: ⭐⭐
estimated_time: 25m
source: [javascript.info, MDN]
---

# Async JavaScript: Tổng quan

> [!summary] TL;DR
> JavaScript là **single-threaded** — chỉ có 1 call stack, không thể làm 2 việc cùng lúc. **Async** cho phép defer các tác vụ chậm (network, timer, I/O) để không block UI thread. Ba pattern tiến hóa: **Callback** → **Promise** → **async/await**, mỗi cái giải quyết nhược điểm của cái trước.

---

## 1. Khái niệm

### JavaScript là single-threaded

JS chỉ có **1 thread** thực thi — không giống Java/Python có thể chạy multi-thread song song. Điều này có nghĩa:
- Tại mỗi thời điểm, chỉ 1 đoạn code đang chạy trên Call Stack
- Nếu 1 task tốn 5 giây → toàn bộ trang web bị "đóng băng" 5 giây
- Trạng thái đó gọi là **blocking** (đồng bộ — synchronous)

### Sync vs Async

```javascript
// Synchronous — mỗi dòng chờ dòng trước hoàn thành
console.log('1');
console.log('2');  // chạy sau khi dòng 1 xong
console.log('3');  // chạy sau khi dòng 2 xong
// Output: 1 → 2 → 3 (luôn theo thứ tự)

// Asynchronous — không đợi, tiếp tục chạy code kế tiếp
console.log('1');
setTimeout(() => console.log('2'), 1000); // đăng ký: chạy sau 1 giây
console.log('3');
// Output: 1 → 3 → 2  ("3" không đợi timer!)
```

### Tại sao cần Async?

Các tác vụ chậm (slow operations) thường gặp trong web:
- **Network requests** — `fetch()`, gọi API (có thể mất 200ms đến vài giây)
- **Timers** — `setTimeout`, `setInterval`, animation
- **File I/O** — đọc/ghi file (chủ yếu trong Node.js)
- **User interaction** — click, scroll, keyboard events

Nếu làm sync → UI bị lock → scroll không được, click không phản hồi → UX tệ. Async cho phép JS "đăng ký" xử lý khi tác vụ xong rồi tiếp tục làm việc khác.

```
★ Insight ─────────────────────────────────────
• Async ≠ multi-thread. JS vẫn chạy 1 thread; cái chạy song song là WEB API của
  trình duyệt (timer, network ở thread C++ riêng), còn callback của bạn vẫn xếp
  hàng chờ về 1 thread JS. Đây là "concurrency" (xen kẽ) chứ không phải
  "parallelism" (thật sự cùng lúc) — phân biệt này hay bị hỏi. Cơ chế ở [[02-Event-Loop]].
• 3 pattern là một dòng TIẾN HÓA giải cùng một bài toán: callback (callback hell)
  → Promise (then phẳng, có combinator) → async/await (đọc như sync). async/await
  chỉ là VỎ ĐƯỜNG trên Promise — không hiểu Promise thì không debug được stack
  lỗi. Và `await` tuần tự KHÔNG tự song song: nhiều việc độc lập → Promise.all.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp / API

### 2.1 Ba pattern Async chính

#### Pattern 1: Callback (ES5 — cũ)

```javascript
function fetchUser(id, callback) {
  setTimeout(() => {
    const user = { id, name: 'Alice' };
    callback(null, user); // error-first convention: (err, result)
  }, 500);
}

fetchUser(1, function(err, user) {
  if (err) { console.error(err); return; }
  console.log(user.name); // "Alice"
});

console.log('Đang chờ...'); // chạy TRƯỚC callback
// Output: "Đang chờ..." → "Alice" (0.5s sau)
```

#### Pattern 2: Promise (ES6)

```javascript
function fetchUser(id) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve({ id, name: 'Alice' });
      // reject(new Error('Not found')); // khi thất bại
    }, 500);
  });
}

fetchUser(1)
  .then(user   => console.log(user.name))
  .catch(err   => console.error(err))
  .finally(()  => console.log('Done'));
```

#### Pattern 3: Async/Await (ES2017 — hiện đại nhất)

```javascript
async function loadUser(id) {
  try {
    const user = await fetchUser(id); // trông như sync nhưng KHÔNG block
    console.log(user.name);
  } catch (err) {
    console.error(err);
  } finally {
    console.log('Done');
  }
}

loadUser(1);
```

### 2.2 JS Runtime Environment

```text
┌─────────────────────────────────────────┐
│           JS Engine (V8)                │
│  ┌────────────┐  ┌─────────────────┐    │
│  │ Call Stack │  │  Memory Heap    │    │
│  │ (LIFO)     │  │  (objects/vars) │    │
│  └────────────┘  └─────────────────┘    │
└─────────────────────────────────────────┘
              ↕  Event Loop  ↕
┌─────────────────────────────────────────┐
│         Web APIs (Browser/Node)         │
│  setTimeout | fetch | DOM Events | I/O  │
└─────────────────────────────────────────┘
              ↓ push callbacks
┌──────────────────┐  ┌───────────────────┐
│   Task Queue     │  │  Microtask Queue  │
│  (Macrotask)     │  │  (Promise.then)   │
│  setTimeout/I/O  │  │  queueMicrotask   │
└──────────────────┘  └───────────────────┘
```

**Quy tắc ưu tiên:** Microtask Queue được xử lý hết **trước** khi Event Loop lấy task tiếp theo từ Task Queue.

---

## 3. Ví dụ minh họa

### Ví dụ 1: Tại sao blocking là vấn đề

```javascript
// Giả lập tác vụ nặng sync — cực kỳ tệ trong browser
function heavyComputation() {
  const start = Date.now();
  while (Date.now() - start < 3000) { /* block 3 giây */ }
  return 'Done';
}

const btn = document.querySelector('button');
btn.addEventListener('click', () => {
  console.log('Bắt đầu tính...');
  const result = heavyComputation(); // BLOCK: không click/scroll được 3 giây!
  console.log(result);
});

// Solution: Web Worker (cho CPU-intensive) hoặc defer bằng setTimeout(fn, 0)
```

### Ví dụ 2: So sánh 3 patterns với cùng một luồng dữ liệu

```javascript
// Setup: giả lập API calls
const delay = (ms, val) => new Promise(res => setTimeout(() => res(val), ms));
const getUser   = id     => delay(300, { id, name: 'Alice' });
const getOrders = userId => delay(300, [{ id: 1, total: 250000 }]);

// --- Callback (cồng kềnh khi cần chain) ---
function loadWithCallback(userId, onDone) {
  getUser(userId).then(user => { // dùng Promise bên trong cho gọn demo
    getOrders(user.id).then(orders => onDone(null, { user, orders }));
  });
}
loadWithCallback(1, (err, data) => console.log('[CB]', data.user.name));

// --- Promise chain (phẳng hơn, composable) ---
getUser(1)
  .then(user => getOrders(user.id).then(orders => ({ user, orders })))
  .then(data => console.log('[P]', data.user.name));

// --- Async/Await (đọc như sync, dễ debug nhất) ---
async function loadAll(userId) {
  const user   = await getUser(userId);
  const orders = await getOrders(user.id);
  console.log('[AA]', user.name, orders.length, 'đơn hàng');
}
loadAll(1);
```

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Pitfall 1: Đọc biến async trước khi nó sẵn sàng
> ```javascript
> let userData = null;
> fetch('/api/me').then(r => r.json()).then(d => { userData = d; });
> console.log(userData); // null — fetch chưa xong!
> ```
> Mọi code cần `userData` phải nằm trong `.then()` hoặc sau `await`.

> [!warning] Pitfall 2: `async` không tự động song song hóa
> ```javascript
> // Sequential: tốn 300ms + 300ms = 600ms
> const user   = await getUser(1);
> const orders = await getOrders(1);
> ```
> Muốn parallel (chỉ tốn 300ms): `const [user, orders] = await Promise.all([getUser(1), getOrders(1)])`.

> [!tip] Callback vẫn quan trọng — không "lỗi thời"
> Event listeners (`addEventListener`), array methods (`forEach`, `map`) đều dùng callback. Callback pattern không biến mất — chỉ không dùng callback cho **async chaining** nữa vì Promise/async/await làm tốt hơn.

---

## 5. Câu hỏi phỏng vấn thường gặp

**Q1: JavaScript có phải multi-threaded không? Async có nghĩa là multi-thread không?**

> Không — JS là **single-threaded**, chỉ có 1 call stack. Async **không** có nghĩa là multi-thread. Browser cung cấp **Web APIs** (chạy trong C++ threads riêng) để xử lý setTimeout, fetch, DOM events. Khi xong, chúng enqueue callback vào Task Queue; Event Loop đưa callback lên Call Stack khi Stack rỗng. Kết quả là cảm giác concurrent nhưng JS code vẫn chạy trên 1 thread.

**Q2: Phân biệt Synchronous và Asynchronous?**

> **Synchronous:** Code thực thi theo thứ tự từ trên xuống, mỗi dòng đợi dòng trước hoàn thành. Nếu 1 operation chậm → toàn bộ thread bị block.
> **Asynchronous:** Tác vụ chậm được "đăng ký" với Web APIs; JS tiếp tục thực thi code kế tiếp; khi tác vụ hoàn thành, callback được enqueue để xử lý sau. Non-blocking — UI thread không bị đóng băng.

**Q3: Tại sao cần 3 pattern? Chỉ dùng async/await có đủ không?**

> Về mặt code hàng ngày, async/await là đủ trong hầu hết trường hợp. Nhưng cần hiểu cả 3 vì: (1) **Callback** còn phổ biến trong event listeners và legacy code. (2) **Promise** là nền tảng của async/await — khi debug, error stack vẫn là Promise chain; cần `Promise.all/race/allSettled` cho parallel scenarios. (3) **async/await** là syntactic sugar — không hiểu Promise thì không debug được.

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Viết function `delay(ms)` trả về Promise resolve sau `ms` milliseconds. Dùng nó để in "Loading...", "Processing...", "Done!" với khoảng cách 500ms mỗi bước bằng async/await.

- [ ] **Bài 2:** Convert đoạn callback lồng nhau sau sang Promise chain rồi sang async/await:
  ```javascript
  getUser(1, (err, user) => {
    if (err) return handleError(err);
    getProfile(user.id, (err, profile) => {
      if (err) return handleError(err);
      console.log(user.name, profile.bio);
    });
  });
  ```

---

## 7. Liên kết

- [[02-Event-Loop]] — Cơ chế bên dưới: Call Stack, Web APIs, Task/Microtask Queue
- [[03-Callback-va-Callback-Hell]] — Pattern cũ nhất và vấn đề callback hell
- [[04-Promise]] — State machine, chaining, combinators
- [[05-Async-Await]] — Syntactic sugar trên Promise
- [[../02-DOM-Event/09-Fetch-API|Fetch API]] — Async trong thực tế với DOM
