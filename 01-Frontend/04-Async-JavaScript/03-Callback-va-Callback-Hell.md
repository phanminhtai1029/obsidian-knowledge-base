---
title: "Callback & Callback Hell"
section: 04-Async-JavaScript
tags: [async, callback, hell, pyramid, pattern, fresher, frontend]
related:
  - "[[01-Async-Overview]]"
  - "[[02-Event-Loop]]"
  - "[[04-Promise]]"
difficulty: ⭐⭐⭐
estimated_time: 35m
source: [javascript.info, MDN]
---

# Callback & Callback Hell

> [!summary] TL;DR
> **Callback** là function được truyền vào function khác để gọi lại sau. Là nền tảng của async JS thời ES5. **Callback Hell** (Pyramid of Doom) xảy ra khi lồng nhiều async callbacks — code thụt lề thành hình tháp, khó đọc và khó xử lý lỗi. Giải pháp: named functions để flatten, hoặc chuyển sang **Promise/async-await**.

> [!tip] 🎯 Hiểu trong 30 giây
> **Callback = "xong việc thì gọi lại số này".** Bạn đưa cho một hàm *một hàm khác* và dặn: "làm xong thì chạy hàm này giúp tôi". Giống như **để lại số điện thoại ở tiệm sửa xe** — sửa xong họ gọi lại, bạn không phải đứng đợi.
>
> **Callback Hell** xảy ra khi việc B phải đợi A xong, C đợi B xong... → callback lồng trong callback lồng trong callback, code **thụt lề thành hình kim tự tháp** rất khó đọc, mỗi tầng lại phải tự kiểm tra lỗi. Ví von: *"sửa xong gọi tôi, rồi tôi mới đặt lốp, lốp xong gọi tôi, rồi tôi mới..."* — chồng chất rối tung.
>
> **Vì sao quan trọng:** đây là lý do Promise và async/await ra đời (note sau). Callback **không lỗi thời** — `addEventListener`, `map`, `forEach` vẫn là callback; chỉ là không dùng nó để *xâu chuỗi nhiều việc async* nữa.

---

## 1. Khái niệm

### Callback là gì?

**Callback** = function truyền vào function khác dưới dạng argument, và được gọi sau (call**back**) khi sự kiện xảy ra hoặc tác vụ hoàn thành.

```javascript
// Callback đồng bộ (sync callback) — phổ biến nhất
[1, 2, 3].forEach(function(item) {
  console.log(item); // function này là callback
});

// Callback bất đồng bộ (async callback)
setTimeout(function() {
  console.log('Chạy sau 1 giây');
}, 1000);
```

### Error-first Callback Convention (Node.js)

Quy ước chuẩn trong Node.js: **tham số đầu tiên luôn là error**, tham số tiếp theo là result.

```javascript
// Quy ước: callback(error, result)
fs.readFile('data.json', 'utf8', function(err, data) {
  if (err) {
    console.error('Lỗi đọc file:', err.message);
    return; // thoát sớm khi lỗi
  }
  console.log('Nội dung:', data);
});
```

### Callback Hell là gì?

Khi cần thực hiện nhiều async operations theo thứ tự (output của cái này là input của cái kia), nếu dùng callback → lồng nhau → "Pyramid of Doom":

```text
asyncOp1(function() {
    asyncOp2(function() {
        asyncOp3(function() {
            asyncOp4(function() {
                // đây là đỉnh "kim tự tháp ngược"
            });
        });
    });
});
```

---

## 2. Cú pháp / API

### 2.1 Callback cơ bản

```javascript
// Callback sync trong array methods
const numbers = [1, 2, 3, 4, 5];

const doubled = numbers.map(function(n) { return n * 2; });
const evens   = numbers.filter(function(n) { return n % 2 === 0; });

// Modern: arrow function callback
const doubled2 = numbers.map(n => n * 2);
const evens2   = numbers.filter(n => n % 2 === 0);

// Event listener callback
document.querySelector('button').addEventListener('click', function(event) {
  console.log('Clicked!', event.target);
});
```

### 2.2 Async Callback Pattern

```javascript
// Giả lập async operations với setTimeout
function getUser(id, callback) {
  setTimeout(() => {
    if (id <= 0) return callback(new Error('Invalid ID'));
    callback(null, { id, name: 'Alice', role: 'admin' });
  }, 300);
}

function getPermissions(role, callback) {
  setTimeout(() => {
    const perms = { admin: ['read', 'write', 'delete'], user: ['read'] };
    callback(null, perms[role] ?? []);
  }, 200);
}

function logActivity(userId, action, callback) {
  setTimeout(() => {
    console.log(`User ${userId}: ${action}`);
    callback(null, { logged: true });
  }, 100);
}

// Dùng độc lập — vẫn ổn
getUser(1, function(err, user) {
  if (err) return console.error(err);
  console.log(user.name);
});
```

### 2.3 Callback Hell — khi cần chain

```javascript
// Yêu cầu: lấy user → lấy permissions → log activity → làm gì đó
// Đây là Callback Hell:
getUser(1, function(err, user) {
  if (err) return console.error('Error 1:', err);

  getPermissions(user.role, function(err, permissions) {
    if (err) return console.error('Error 2:', err);

    logActivity(user.id, 'login', function(err, log) {
      if (err) return console.error('Error 3:', err);

      // Có thể tiếp tục lồng sâu hơn...
      console.log(user.name, permissions, log);
      // Nếu cần thêm 1 bước nữa → thụt lề thêm 1 cấp
    });
  });
});
```

**Vấn đề của Callback Hell:**
1. **Indentation** — code thụt lề thành hình tháp, khó đọc
2. **Error handling lặp lại** — `if (err) return ...` ở mỗi cấp
3. **Khó refactor** — extract function ra khó vì context lồng nhau
4. **Không composable** — không thể tái sử dụng flow

```
★ Insight ─────────────────────────────────────
• Vấn đề SÂU nhất của callback không phải thụt lề — mà là "inversion of control":
  bạn TRAO hàm của mình cho thư viện và TIN nó gọi đúng 1 lần, đúng lúc, đúng
  tham số. Nó gọi 2 lần? quên gọi? gọi với lỗi không báo? Bạn chịu. Promise lấy
  lại quyền kiểm soát đó (settle đúng 1 lần, theo spec). Đây là lý do triết học
  Promise ra đời, không chỉ vì thẩm mỹ.
• Hai bug callback kinh điển — gọi cb NHIỀU LẦN và QUÊN return sau cb(err) — chính
  là thứ Promise xoá sổ theo thiết kế. Nhận diện được chúng giải thích vì sao cả
  hệ sinh thái chuyển sang Promise/async-await ([[04-Promise]]).
─────────────────────────────────────────────────
```

### 2.4 Giải pháp 1: Named Functions (flatten)

```javascript
// Tách thành các named functions — flat thay vì lồng
function handleUser(err, user) {
  if (err) return console.error('Error getting user:', err);
  getPermissions(user.role, handlePermissions.bind(null, user));
}

function handlePermissions(user, err, permissions) {
  if (err) return console.error('Error getting permissions:', err);
  logActivity(user.id, 'login', handleLog.bind(null, user, permissions));
}

function handleLog(user, permissions, err, log) {
  if (err) return console.error('Error logging:', err);
  console.log(user.name, permissions, log);
}

// Gọi — phẳng hơn nhiều
getUser(1, handleUser);
```

### 2.5 Giải pháp 2: Promisify — convert callback sang Promise

```javascript
// Wrapper tự viết
function getUserPromise(id) {
  return new Promise((resolve, reject) => {
    getUser(id, (err, user) => {
      if (err) reject(err);
      else resolve(user);
    });
  });
}

// Node.js có util.promisify tự động
const { promisify } = require('util');
// const readFile = promisify(fs.readFile);

// Sau khi promisify: dùng async/await — clean hoàn toàn
async function loadAll() {
  try {
    const user        = await getUserPromise(1);
    // const perms    = await getPermissionsPromise(user.role);
    console.log(user.name);
  } catch (err) {
    console.error(err);
  }
}
loadAll();
```

---

## 3. Ví dụ minh họa

### Ví dụ 1: Real-world — Sequential API calls

```javascript
// Giả lập: login → load dashboard → fetch notifications
const api = {
  login: (creds, cb) =>
    setTimeout(() => cb(null, { token: 'abc123', userId: 42 }), 300),
  getDashboard: (userId, token, cb) =>
    setTimeout(() => cb(null, { stats: { sales: 150 } }), 400),
  getNotifications: (userId, cb) =>
    setTimeout(() => cb(null, [{ msg: 'New order' }]), 200),
};

// --- Callback Hell ---
api.login({ username: 'alice', password: '***' }, function(err, session) {
  if (err) return console.error('Login failed:', err);
  api.getDashboard(session.userId, session.token, function(err, dashboard) {
    if (err) return console.error('Dashboard failed:', err);
    api.getNotifications(session.userId, function(err, notifs) {
      if (err) return console.error('Notifications failed:', err);
      console.log('Sales:', dashboard.stats.sales);
      console.log('Notifs:', notifs.length);
    });
  });
});

// --- Promisified + async/await ---
const login    = creds         => new Promise((res, rej) => api.login(creds, (e, d) => e ? rej(e) : res(d)));
const getDash  = (uid, token)  => new Promise((res, rej) => api.getDashboard(uid, token, (e, d) => e ? rej(e) : res(d)));
const getNotifs = uid          => new Promise((res, rej) => api.getNotifications(uid, (e, d) => e ? rej(e) : res(d)));

async function loadApp() {
  const session  = await login({ username: 'alice', password: '***' });
  const [dash, notifs] = await Promise.all([
    getDash(session.userId, session.token),
    getNotifs(session.userId),
  ]);
  console.log('Sales:', dash.stats.sales, '| Notifs:', notifs.length);
}
loadApp().catch(console.error);
```

### Ví dụ 2: Custom event system với callbacks

```javascript
class EventEmitterSimple {
  #listeners = {};

  on(event, callback) {
    if (!this.#listeners[event]) this.#listeners[event] = [];
    this.#listeners[event].push(callback);
    return this;
  }

  emit(event, ...args) {
    (this.#listeners[event] ?? []).forEach(cb => cb(...args));
  }

  off(event, callback) {
    if (!this.#listeners[event]) return;
    this.#listeners[event] = this.#listeners[event].filter(cb => cb !== callback);
  }
}

const emitter = new EventEmitterSimple();

const onData = (data) => console.log('Data received:', data);
emitter.on('data', onData);
emitter.on('error', (err) => console.error('Error:', err));

emitter.emit('data', { id: 1, name: 'Alice' }); // "Data received: {id: 1, name: 'Alice'}"
emitter.off('data', onData);
emitter.emit('data', { id: 2 }); // không có gì — listener đã bị xóa
```

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Pitfall 1: Gọi callback nhiều lần
> ```javascript
> function badAsync(cb) {
>   setTimeout(() => cb('first call'), 100);
>   setTimeout(() => cb('second call'), 200); // callback gọi 2 lần!
> }
> ```
> Callback nên được gọi **đúng 1 lần**. Promise giải quyết vấn đề này bằng cách settle chỉ 1 lần (resolve/reject lần sau bị ignore).

> [!warning] Pitfall 2: Quên thoát sớm sau khi xử lý error
> ```javascript
> function process(data, cb) {
>   if (!data) {
>     cb(new Error('No data')); // BUG: không return → chạy tiếp!
>     // thiếu return ở đây
>   }
>   cb(null, processData(data)); // gọi cb lần 2 nếu !data
> }
> ```
> Luôn `return callback(err)` chứ không phải chỉ `callback(err)`.

> [!tip] Async callback không có nghĩa là "chạy ngay sau khi return"
> `setTimeout(fn, 0)` và `Promise.resolve().then(fn)` đều async — fn không chạy ngay, mà được enqueue. Code sau dòng đăng ký chạy trước.

---

## 5. Câu hỏi phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Callback Hell là gì, tránh thế nào?"
> *"Callback là hàm mình truyền vào hàm khác để được gọi lại khi tác vụ xong. Callback Hell là khi nhiều tác vụ async phụ thuộc nhau, mình lồng callback trong callback nhiều tầng, code thụt lề thành hình kim tự tháp, rất khó đọc và mỗi tầng lại phải xử lý lỗi riêng. Để tránh, em có ba hướng: tách thành các named function cho phẳng ra; promisify để biến hàm callback thành Promise; và tốt nhất là dùng async/await để code async đọc như đồng bộ. Ngoài ra Promise còn đảm bảo chỉ settle đúng một lần, khắc phục được bug callback bị gọi nhiều lần hoặc quên gọi của callback thuần."*

> [!note] 🧠 Mẹo nhớ
> **"Để lại số, xong gọi lại."** Lồng nhiều tầng = **kim tự tháp khổ đau (Pyramid of Doom)** → gỡ bằng **named function → promisify → async/await.** Callback vẫn sống ở `addEventListener`/`map`.

**Q1: Callback Hell là gì? Làm thế nào để tránh?**

> Callback Hell (Pyramid of Doom) là tình trạng các async callbacks lồng nhau nhiều cấp, tạo thành hình tháp thụt lề, khó đọc, khó xử lý lỗi, và khó refactor. Giải pháp: (1) **Named functions** để flatten — tách mỗi step thành function có tên riêng. (2) **Promisify** — convert callback-based functions sang Promise. (3) **async/await** — code có dạng synchronous, dễ đọc nhất.

**Q2: Error-first callback convention là gì?**

> Quy ước chuẩn trong Node.js: tham số đầu tiên của callback luôn là `error` (null nếu thành công), tham số tiếp theo là result. Ví dụ: `callback(null, data)` hoặc `callback(new Error('...'))`. Quy ước này giúp xử lý lỗi nhất quán, nhưng phải check `if (err)` ở mỗi cấp — đây là một trong những bất tiện của callback pattern.

**Q3: Khi nào nên dùng callback thay vì Promise?**

> Callback vẫn phù hợp cho: (1) **Synchronous callbacks** (array methods: map/filter/forEach) — không cần async overhead. (2) **Event listeners** (`addEventListener`) — có thể fire nhiều lần, Promise chỉ settle 1 lần. (3) **Performance-critical code** — callback có overhead thấp hơn Promise trong một số trường hợp. Với async chaining (sequential operations) → dùng Promise/async-await.

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Viết function `readFileAsync(path, callback)` giả lập đọc file (dùng `setTimeout` để giả lập delay). Rồi viết phiên bản `readFilePromise(path)` dùng Promise wrapper.

- [ ] **Bài 2:** Refactor đoạn callback hell sau dùng async/await:
  ```javascript
  login(creds, (err, user) => {
    if (err) return handleError(err);
    loadProfile(user.id, (err, profile) => {
      if (err) return handleError(err);
      loadSettings(user.id, (err, settings) => {
        if (err) return handleError(err);
        renderDashboard(user, profile, settings);
      });
    });
  });
  ```

---

## 7. Liên kết

- [[01-Async-Overview]] — Tổng quan async, 3 patterns tiến hóa
- [[02-Event-Loop]] — Cơ chế async callbacks hoạt động thế nào
- [[04-Promise]] — Giải pháp hiện đại thay thế callback chaining
- [[05-Async-Await]] — Syntactic sugar trên Promise
