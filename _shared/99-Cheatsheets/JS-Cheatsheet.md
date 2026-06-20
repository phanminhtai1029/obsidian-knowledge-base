---
title: "JavaScript Cheatsheet"
section: 99-Cheatsheets
tags: [cheatsheet, javascript, es6, async, reference, fresher, frontend]
related:
  - "[[01-Scope]]"
  - "[[03-Closure]]"
  - "[[05-ES6-Block-scoped-let-const]]"
  - "[[08-ES6-Destructuring]]"
  - "[[09-ES6-Arrow-Function]]"
  - "[[10-ES6-Rest-Spread]]"
difficulty: ⭐
estimated_time: 10m
source: [MDN Web Docs, javascript.info]
---

# JavaScript Cheatsheet

> [!summary] TL;DR
> Tra nhanh syntax ES6+, array methods, async/await trước phỏng vấn. Các mục hay bị hỏi nhất: closure, hoisting, `this`, Promise chain, `async/await` error handling.

---

## 1. Biến & Scope

| | `var` | `let` | `const` |
|---|---|---|---|
| Scope | function | block | block |
| Hoisting | có (undefined) | có (TDZ) | có (TDZ) |
| Re-assign | ✅ | ✅ | ❌ |
| Re-declare | ✅ | ❌ | ❌ |

```js
// TDZ (Temporal Dead Zone) — let/const không dùng được trước khai báo
console.log(x); // ReferenceError
let x = 1;

// var bị hoisted, giá trị là undefined
console.log(y); // undefined (không lỗi)
var y = 2;
```

---

## 2. ES6+ Syntax Nhanh

### Arrow Function

```js
// Truyền thống
function add(a, b) { return a + b; }

// Arrow — ngắn gọn, không có `this` riêng
const add = (a, b) => a + b;

// Nhiều dòng cần {}  và return
const process = (x) => {
  const result = x * 2;
  return result;
};
```

### Destructuring

```js
// Array
const [first, second, ...rest] = [1, 2, 3, 4, 5];
// first=1, second=2, rest=[3,4,5]

// Object — rename + default
const { name, age = 18, role: userRole } = user;

// Trong parameter
function greet({ name, age }) { return `${name}, ${age}`; }
```

### Spread / Rest

```js
// Spread — mở rộng iterable
const merged = { ...obj1, ...obj2 };        // object merge
const copy   = [...arr1, ...arr2];          // array copy + merge
fn(...args);                                // spread vào call

// Rest — gom phần còn lại
const sum = (...nums) => nums.reduce((a, b) => a + b, 0);
const { id, ...rest } = obj;               // object rest
```

### Template Literals

```js
const msg = `Xin chào ${name}, bạn ${age} tuổi`;
const multiLine = `
  Dòng 1
  Dòng 2
`;
```

### Optional Chaining & Nullish Coalescing

```js
const city = user?.address?.city;          // không throw nếu address là null
const name = user?.name ?? 'Anonymous';   // ?? chỉ fallback khi null/undefined (khác ||)
```

---

## 3. Array Methods — Bảng Tra Nhanh

| Method | Mục đích | Trả về | Mutate? |
|---|---|---|---|
| `map(fn)` | biến đổi mỗi phần tử | array mới | ❌ |
| `filter(fn)` | lọc theo điều kiện | array mới | ❌ |
| `reduce(fn, init)` | gom thành 1 giá trị | bất kỳ | ❌ |
| `find(fn)` | phần tử đầu tiên thỏa | phần tử / undefined | ❌ |
| `findIndex(fn)` | index đầu tiên thỏa | number / -1 | ❌ |
| `some(fn)` | ít nhất 1 phần tử thỏa | boolean | ❌ |
| `every(fn)` | tất cả phần tử thỏa | boolean | ❌ |
| `includes(val)` | kiểm tra tồn tại | boolean | ❌ |
| `flat(depth)` | flatten mảng lồng | array mới | ❌ |
| `flatMap(fn)` | map rồi flat 1 cấp | array mới | ❌ |
| `forEach(fn)` | lặp, không trả giá trị | undefined | ❌ |
| `sort(fn)` | sắp xếp **in-place** | array gốc | ✅ |
| `splice(i,n)` | xóa/thêm **in-place** | phần tử xóa | ✅ |
| `push/pop` | thêm/xóa cuối | new length / phần tử | ✅ |
| `shift/unshift` | xóa/thêm đầu | phần tử / new length | ✅ |

```js
const nums = [1, 2, 3, 4, 5];

// map + filter + reduce — chuỗi phổ biến
const result = nums
  .filter(n => n % 2 === 0)     // [2, 4]
  .map(n => n * n)               // [4, 16]
  .reduce((sum, n) => sum + n, 0); // 20

// sort — LUÔN dùng comparator
['b', 'a', 'c'].sort((a, b) => a.localeCompare(b)); // ['a','b','c']
[10, 9, 2].sort((a, b) => a - b);                   // [2, 9, 10]
```

> [!warning] sort() không copy — nó mutate array gốc
> Nếu không muốn mutate: `[...arr].sort(fn)` hoặc `arr.toSorted(fn)` (ES2023).

---

## 4. Object Methods

```js
const obj = { a: 1, b: 2, c: 3 };

Object.keys(obj)    // ['a', 'b', 'c']
Object.values(obj)  // [1, 2, 3]
Object.entries(obj) // [['a',1], ['b',2], ['c',3]]

// Tạo object từ entries
Object.fromEntries([['a', 1], ['b', 2]]); // { a: 1, b: 2 }

// Kiểm tra key
'a' in obj          // true
obj.hasOwnProperty('a') // true (không kế thừa prototype)
```

---

## 5. Async JavaScript

### Promise

```js
// Tạo Promise
const p = new Promise((resolve, reject) => {
  if (ok) resolve(data);
  else reject(new Error('Failed'));
});

// Tiêu thụ
p.then(data => console.log(data))
 .catch(err  => console.error(err))
 .finally(()  => console.log('Done'));

// Parallel
Promise.all([p1, p2, p3])        // chờ tất cả, 1 fail → reject
Promise.allSettled([p1, p2])     // chờ tất cả, trả kết quả từng cái
Promise.race([p1, p2])           // resolve/reject theo cái nhanh nhất
Promise.any([p1, p2])            // resolve theo cái thành công đầu tiên
```

### async / await

```js
async function fetchUser(id) {
  try {
    const res  = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const data = await res.json();
    return data;
  } catch (err) {
    console.error('Fetch failed:', err.message);
    throw err; // re-throw nếu caller cần handle
  }
}

// Parallel trong async/await
async function loadDashboard() {
  const [users, posts] = await Promise.all([
    fetchUsers(),
    fetchPosts(),
  ]);
  return { users, posts };
}
```

---

## 6. Closure & `this` — Hay Hỏi Trong Phỏng Vấn

```js
// Closure — function nhớ scope nơi nó được tạo
function makeCounter() {
  let count = 0;
  return {
    increment: () => ++count,
    get:       () => count,
  };
}
const counter = makeCounter();
counter.increment(); // 1
counter.increment(); // 2
counter.get();       // 2

// `this` trong arrow function — kế thừa từ outer scope
const obj = {
  name: 'Alice',
  greet() {
    const inner = () => console.log(this.name); // `this` = obj ✅
    inner();
  },
  greetBroken: function() {
    function inner() { console.log(this.name); } // `this` = undefined (strict) ❌
    inner();
  },
};
```

| Cách gọi | `this` là |
|---|---|
| `fn()` | undefined (strict) / global |
| `obj.fn()` | `obj` |
| `new Fn()` | instance mới |
| `fn.call(ctx)` | `ctx` |
| Arrow function | `this` của outer scope |

---

## 7. Câu Hỏi Phỏng Vấn Thường Gặp

**Q1: Closure là gì? Cho ví dụ thực tế.**

> Closure là function có thể truy cập biến của scope bên ngoài sau khi scope đó đã thoát. Ví dụ thực tế: debounce, counter factory, module pattern (ẩn private state).

**Q2: `==` vs `===` khác nhau thế nào?**

> `===` (strict equality): so sánh cả giá trị lẫn kiểu — không ép kiểu. `==` (loose equality): ép kiểu trước khi so sánh — `'0' == false` là `true`, gây khó lường. Luôn dùng `===` trừ khi cần check `null || undefined`: `val == null`.

**Q3: `Promise.all` vs `Promise.allSettled` khác gì?**

> `Promise.all` reject ngay khi 1 promise thất bại (fail-fast). `Promise.allSettled` chờ tất cả kết thúc và trả về mảng `{ status, value/reason }` — dùng khi cần biết kết quả mọi request dù có lỗi.

**Q4: Giải thích event loop.**

> Call stack xử lý synchronous code. Khi gặp async (setTimeout, fetch...), callback được đẩy vào Web APIs. Khi xong, callback vào task queue (macrotask) hoặc microtask queue (Promise). Event loop lấy từ microtask trước (Promise.then), rồi mới macrotask (setTimeout). Vì vậy Promise.then chạy trước setTimeout(fn, 0).

---

## 8. Bài Tập Tự Kiểm Tra

- [ ] Viết hàm `debounce(fn, delay)` dùng closure — gọi fn sau delay ms kể từ lần gọi cuối cùng.
- [ ] Viết `flatDeep(arr)` — flatten mảng lồng nhiều cấp mà không dùng `Array.flat()`.

---

## 9. Liên Kết

- [[03-Closure]] — Closure chi tiết
- [[09-ES6-Arrow-Function]] — Arrow function và `this`
- [[08-ES6-Destructuring]] — Destructuring patterns
- [[10-ES6-Rest-Spread]] — Rest/Spread
