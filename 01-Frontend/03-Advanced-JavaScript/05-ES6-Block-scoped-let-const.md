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
> `let` và `const` (ra mắt năm 2015) sửa 2 nhược điểm của `var`: `var` **không có block scope** (không bị giới hạn trong cặp `{}`) và **hoisting bất ngờ** (đọc trước khai báo ra `undefined` gây bug âm thầm). `const` — **không gán lại được** (reassign = trỏ biến sang giá trị khác), nhưng nếu là object/array thì nội dung bên trong vẫn **sửa được** (mutable). `let` — **gán lại được**. Quy tắc thực hành: mặc định dùng `const`, cần gán lại mới dùng `let`, không dùng `var`.

> [!tip] 🎯 Hiểu trong 30 giây
> Ví von **`const` như dán nhãn tên cố định lên một chiếc hộp**: bạn không được lấy nhãn dán sang hộp khác (không reassign), nhưng **đồ bên trong hộp vẫn thêm/bớt được** (`obj.key = ...`, `arr.push(...)` vẫn chạy). `let` thì nhãn gỡ ra dán hộp khác được (reassign). `var` là kiểu cũ, vừa thiếu kỷ luật vừa hay gây bug.
>
> **Điểm CỰC hay ra thi — bẫy "const là immutable":** Câu trả lời đúng là *KHÔNG*. `const` chỉ khóa **liên kết biến → địa chỉ**, không khóa **nội dung**. `const u = {}` thì `u.name = 'Bob'` vẫn được, chỉ `u = {}` mới lỗi. Muốn khóa luôn nội dung phải `Object.freeze()` (mà cũng chỉ khóa nông 1 tầng).
>
> **Quy tắc vàng:** mặc định `const` → cần gán lại mới đổi `let` → gần như không bao giờ `var`.

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

```
★ Insight ─────────────────────────────────────
• const khóa LIÊN KẾT (biến↔địa chỉ), không khóa NỘI DUNG. `const u = {}` nghĩa
  là "u mãi trỏ vào object NÀY", nhưng object đó vẫn sửa được (u.name='Bob' OK).
  Đây là lý do `const arr=[]; arr.push(1)` hợp lệ mà `arr=[2]` thì lỗi. Muốn
  khóa nội dung → Object.freeze (mà cũng chỉ NÔNG một tầng).
• Mặc định const là một quyết định về KHẢ ĐỌC, không phải khắt khe: đọc `const`
  là biết ngay "giá trị này không bị gán lại" → bớt một thứ phải theo dõi trong
  đầu. Chỉ đổi sang `let` khi thực sự cần reassign. var bị loại vì thiếu block
  scope + hoisting undefined âm thầm (xem [[02-Hoisting]]).
─────────────────────────────────────────────────
```

---

## 5. Phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "`const` có phải immutable không?"
> *"Không hẳn ạ. `const` chỉ ngăn việc gán lại biến — nghĩa là không thể trỏ biến sang một giá trị/đối tượng khác. Nhưng nếu giá trị là object hay array thì nội dung bên trong vẫn sửa được bình thường: `const u = {}` thì `u.name = 'Bob'` hợp lệ, `const arr = []` thì `arr.push(1)` hợp lệ, chỉ có `u = {}` hay `arr = [2]` mới báo lỗi. Em hình dung `const` như dán nhãn cố định lên cái hộp: không đổi nhãn được nhưng đồ trong hộp vẫn thêm bớt. Muốn khóa luôn nội dung thì dùng `Object.freeze`, nhưng nó chỉ khóa nông một tầng. Trong dự án em mặc định dùng `const`, chỉ đổi `let` khi cần gán lại."*

> [!note] 🧠 Mẹo nhớ
> **"`const` khóa cái NHÃN, không khóa cái RUỘT."** Thứ tự ưu tiên: **const → let → (đừng) var.**

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
