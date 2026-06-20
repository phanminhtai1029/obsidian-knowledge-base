---
title: "ES6: let và const (Block Scoping)"
section: 03-Advanced-JavaScript
tags: [es6, let, const, block-scope, fresher, frontend]
related:
  - "[[01-Scope]]"
  - "[[02-Hoisting]]"
difficulty: ⭐⭐
estimated_time: 25m
source: [MDN]
---

# ES6: let và const (Block Scoping)

> [!summary] TL;DR
> `let` và `const` (ES2015) giải quyết 2 vấn đề của `var`: **không có block scope** và **hoisting bất ngờ**. `const` — không thể reassign (nhưng object/array bên trong vẫn mutable). `let` — cho phép reassign. Quy tắc thực hành: luôn dùng `const` mặc định, `let` khi cần, không dùng `var`.

---

## 1. Khái niệm

### Ba vấn đề của `var`

```javascript
// Vấn đề 1: Không có block scope
for (var i = 0; i < 3; i++) {}
console.log(i); // 3 — i thoát ra ngoài loop!

// Vấn đề 2: Hoisting init thành undefined gây bug ngầm
console.log(name); // undefined (không phải lỗi!)
var name = 'Alice';

// Vấn đề 3: Có thể khai báo lại
var x = 1;
var x = 2; // không lỗi!
console.log(x); // 2
```

### So sánh `var` / `let` / `const`

| Đặc điểm | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Function | Block | Block |
| Hoisting | ✅ (init `undefined`) | ✅ (TDZ) | ✅ (TDZ) |
| Khai báo lại | ✅ | ❌ | ❌ |
| Reassign | ✅ | ✅ | ❌ |
| Thuộc `window` | ✅ (global) | ❌ | ❌ |

---

## 2. Cú pháp

### 2.1 `let` — block scope, có thể reassign

```javascript
let score = 0;
score = 10;       // OK
score = score + 5; // OK

{
  let blockVar = 'tôi chỉ sống trong block này';
  console.log(blockVar); // OK
}
// console.log(blockVar); // ReferenceError

// let trong loop — mỗi iteration có binding riêng
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Output: 0, 1, 2 (đúng!)
```

### 2.2 `const` — không thể reassign

```javascript
const PI = 3.14159;
// PI = 3; // TypeError: Assignment to constant variable

const user = { name: 'Alice', age: 25 };
// user = {};  // TypeError — không thể reassign reference

// Nhưng có thể mutate nội dung!
user.name = 'Bob';    // OK
user.age  = 26;       // OK
user.email = 'bob@example.com'; // OK
console.log(user); // { name: 'Bob', age: 26, email: '...' }

const arr = [1, 2, 3];
arr.push(4);     // OK
arr[0] = 99;     // OK
// arr = [5, 6]; // TypeError
```

### 2.3 Tại sao dùng `const` mặc định?

```javascript
// const truyền đạt intent: "giá trị này không thay đổi"
const MAX_RETRIES  = 3;
const BASE_URL     = 'https://api.example.com';
const CONFIG       = { timeout: 5000, retries: MAX_RETRIES };

// Nếu cần thay đổi → dùng let
let currentPage = 1;
let isLoading   = false;

// Đọc code biết ngay: const = stable reference, let = can change
```

### 2.4 Destructuring với `const` và `let`

```javascript
const { name, age } = { name: 'Alice', age: 25 };
const [first, second] = [10, 20];

let a = 1, b = 2;
[a, b] = [b, a]; // swap — chỉ let mới swap được
console.log(a, b); // 2, 1
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: Refactor `var` → `const`/`let`

```javascript
// Trước (var — nhiều vấn đề tiềm ẩn)
function processUsersOld(users) {
  var result = [];
  var total  = 0;

  for (var i = 0; i < users.length; i++) {
    var user    = users[i];
    var doubled = user.score * 2;
    total += doubled;
    result.push({ name: user.name, score: doubled });
  }

  return { users: result, total: total };
}

// Sau (const/let — rõ ràng hơn)
function processUsers(users) {
  const result = []; // không reassign mảng, chỉ push
  let total    = 0;  // cần reassign (+=)

  for (const user of users) {       // const trong for-of OK
    const doubled = user.score * 2; // không reassign trong loop
    total += doubled;
    result.push({ name: user.name, score: doubled });
  }

  return { users: result, total };
}

const data = [
  { name: 'Alice', score: 80 },
  { name: 'Bob',   score: 90 },
];

console.log(processUsers(data));
// { users: [{name:'Alice',score:160},{name:'Bob',score:180}], total: 340 }
```

### Ví dụ 2: Object freeze cho "true const"

```javascript
// const không ngăn mutation — dùng Object.freeze để đóng băng
const CONFIG = Object.freeze({
  API_URL:  'https://api.example.com',
  TIMEOUT:  5000,
  MAX_RETRY: 3,
});

// CONFIG.API_URL = 'hack'; // Silently fails (non-strict) hoặc TypeError (strict)
console.log(CONFIG.API_URL); // "https://api.example.com" — không đổi

// Lưu ý: freeze nông (shallow) — nested objects vẫn mutable
const nested = Object.freeze({ inner: { value: 1 } });
nested.inner.value = 99; // OK — inner không bị freeze
console.log(nested.inner.value); // 99
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: `const` với object/array không phải "immutable"
> `const` ngăn **reassign reference** (`obj = {}`), không ngăn **mutation** (`obj.key = value`). Để thực sự immutable, dùng `Object.freeze()`. Với array lồng sâu dùng deep freeze hoặc thư viện Immer.

> [!warning] Pitfall 2: TDZ với `let`/`const`
> Không giống `var`, `let`/`const` không thể truy cập trước dòng khai báo. Khai báo ở đầu scope để tránh nhầm lẫn — dù không bắt buộc về mặt syntax.

> [!tip] `const` trong for-of / for-in
> `const` hoạt động trong `for...of` và `for...in` — mỗi iteration tạo binding mới. Chỉ `for (let i = 0; ...)` cần `let` vì `i` được cập nhật trong cùng binding.

---

## 5. Phỏng vấn thường gặp

**Q1: Phân biệt `let`, `const`, `var`?**

> `var`: function scope, hoist init `undefined`, có thể khai báo lại, thuộc `window` khi global. `let`: block scope, hoist vào TDZ, không khai báo lại, có thể reassign. `const`: block scope, hoist vào TDZ, không khai báo lại, không reassign (nhưng nội dung object/array vẫn mutable). Dùng `const` mặc định, `let` khi cần reassign.

**Q2: `const` có phải là immutable không?**

> Không hoàn toàn. `const` ngăn reassign biến (không thể `const x = 1; x = 2`), nhưng không ngăn mutation nội dung. `const obj = {}; obj.key = 'value'` hoàn toàn hợp lệ. Để immutable thật sự, dùng `Object.freeze()` hoặc thư viện như Immer.

**Q3: Khi nào dùng `let`, khi nào dùng `const`?**

> Mặc định dùng `const`. Chuyển sang `let` khi: cần reassign (counter, loop variable, flag thay đổi trạng thái). Không dùng `var` trong code mới. Quy tắc đơn giản: nếu không chắc, bắt đầu với `const`, compiler sẽ báo lỗi nếu bạn cần reassign.

---

## 6. Bài tập thực hành

- [ ] **Bài 1:** Refactor đoạn code sau, thay hết `var` bằng `let`/`const` thích hợp, giải thích lý do chọn từng cái:
  ```javascript
  var total = 0;
  var items = ['apple', 'banana', 'cherry'];
  for (var i = 0; i < items.length; i++) {
    var item = items[i];
    var upper = item.toUpperCase();
    total += upper.length;
  }
  ```

- [ ] **Bài 2:** Giải thích tại sao đoạn sau log ra gì và sửa nếu cần:
  ```javascript
  const arr = [1, 2, 3];
  arr.push(4);
  arr = [1, 2, 3, 4]; // ?
  ```

---

## 7. Liên kết

- [[01-Scope]] — Block scope vs Function scope
- [[02-Hoisting]] — TDZ của let/const
- [[08-ES6-Destructuring]] — const/let trong destructuring
