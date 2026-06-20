---
title: "JavaScript Hoisting"
section: 03-Advanced-JavaScript
tags: [js, hoisting, TDZ, var, let, fresher, frontend]
related:
  - "[[01-Scope]]"
  - "[[05-ES6-Block-scoped-let-const]]"
difficulty: ⭐⭐⭐
estimated_time: 35m
source: [MDN, YDKJS]
---

# JavaScript Hoisting

> [!summary] TL;DR
> **Hoisting** — JS engine di chuyển khai báo lên đầu scope trước khi chạy code. `var` được hoist và init thành `undefined`. `function declaration` được hoist hoàn toàn (gọi được trước khi khai báo). `let`/`const` được hoist nhưng ở **TDZ** (Temporal Dead Zone) — truy cập trước khai báo → `ReferenceError`.

---

## 1. Khái niệm

### Hoisting là gì?

Trước khi thực thi code, JS engine có 2 phase:
1. **Compilation phase:** scan qua code, tìm tất cả khai báo (`var`, `function`, `let`, `const`), tạo binding trong scope tương ứng.
2. **Execution phase:** chạy code từng dòng.

"Hoisting" mô tả hiệu ứng: khai báo được "kéo lên đầu" scope — nhưng thực ra là compiler đã xử lý chúng trước khi execution bắt đầu.

```
★ Insight ─────────────────────────────────────
• Không có gì "bay lên" thật cả — "2 phase" mới là mô hình đúng: phase quét đăng
  ký TÊN trước, phase chạy mới GÁN giá trị. Mọi khác biệt var/let/function quy về
  một câu hỏi: "lúc đăng ký tên, nó được khởi tạo thành gì?" var→undefined (đọc
  được), let/const→chưa khởi tạo (TDZ, đọc là lỗi), function→cả thân hàm (gọi được).
• TDZ là TÍNH NĂNG, không phải phiền toái: nó biến lỗi "dùng trước khai báo" từ
  bug âm thầm (var trả undefined rồi sai ngầm) thành lỗi NỔ NGAY (ReferenceError).
  Đây là một lý do cốt lõi để mặc định const/let, bỏ var — xem [[05-ES6-Block-scoped-let-const]].
─────────────────────────────────────────────────
```

### So sánh hoisting theo từ khóa

| Từ khóa | Hoist không? | Được init thành gì? | Truy cập trước khai báo |
|---|---|---|---|
| `var` | ✅ Có | `undefined` | Được — trả `undefined` |
| `let` | ✅ Có (vào TDZ) | Không init | ❌ `ReferenceError` |
| `const` | ✅ Có (vào TDZ) | Không init | ❌ `ReferenceError` |
| `function declaration` | ✅ Có (đầy đủ) | Toàn bộ function | ✅ Gọi được |
| `function expression` | Chỉ variable hoist | `undefined` (nếu `var`) | ❌ `TypeError` |
| `class` | ✅ Có (vào TDZ) | Không init | ❌ `ReferenceError` |

### Temporal Dead Zone (TDZ)

TDZ là khoảng thời gian từ lúc scope bắt đầu đến lúc gặp dòng khai báo `let`/`const`. Trong TDZ, biến **tồn tại** nhưng **không thể truy cập**.

```text
|-- scope bắt đầu (TDZ bắt đầu) --|-- khai báo let x = 5 --|-- TDZ kết thúc -->
          ↑
    ReferenceError nếu truy cập x ở đây
```

---

## 2. Cú pháp

### 2.1 `var` hoisting

```javascript
console.log(name); // undefined — không phải ReferenceError
var name = 'Alice';
console.log(name); // "Alice"

// JS engine thực sự làm:
// var name;          ← hoist lên đầu, init = undefined
// console.log(name); // undefined
// name = 'Alice';
// console.log(name); // "Alice"
```

### 2.2 `let`/`const` TDZ

```javascript
// console.log(age); // ReferenceError: Cannot access 'age' before initialization
let age = 25;
console.log(age); // 25

// TDZ cũng xảy ra bên trong block
{
  // TDZ bắt đầu cho 'value'
  // console.log(value); // ReferenceError
  const value = 42;    // TDZ kết thúc
  console.log(value);  // 42
}
```

### 2.3 Function Declaration — hoist hoàn toàn

```javascript
// Gọi trước khi khai báo — hoạt động bình thường
greet('Bob'); // "Hello, Bob!"

function greet(name) {
  console.log('Hello, ' + name + '!');
}

// JS engine thực sự làm:
// function greet(name) { ... }  ← hoist lên đầu hoàn toàn
// greet('Bob');
```

### 2.4 Function Expression — KHÔNG hoist function body

```javascript
// sayHi(); // TypeError: sayHi is not a function
// (var sayHi được hoist = undefined, gọi undefined() → TypeError)

var sayHi = function() {
  console.log('Hi!');
};

sayHi(); // "Hi!" — chỉ hoạt động sau khi gán

// Với let/const:
// greetUser(); // ReferenceError (TDZ)
const greetUser = () => console.log('Hello!');
```

### 2.5 Hoisting trong function scope

```javascript
function processOrder() {
  console.log(status); // undefined (var hoist trong function scope)
  // console.log(amount); // ReferenceError (TDZ)

  var status = 'processing';
  let amount = 100;

  console.log(status); // "processing"
  console.log(amount); // 100
}

processOrder();
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: Bug kinh điển với var hoisting

```javascript
// Bug: showMessage được gọi trước khi isLoggedIn gán
function initApp() {
  showMessage(); // chạy nhưng thấy isLoggedIn = undefined!

  var isLoggedIn = checkAuth();

  function showMessage() {
    if (isLoggedIn) {       // undefined → falsy
      console.log('Chào mừng!');
    } else {
      console.log('Vui lòng đăng nhập.'); // luôn chạy vào đây
    }
  }
}

function checkAuth() { return true; }
initApp();
// Output: "Vui lòng đăng nhập." — dù checkAuth() trả true!
```

**Cách sửa:** Dùng `let`/`const` + gọi sau khi gán, hoặc dùng function expression.

```javascript
function initAppFixed() {
  const isLoggedIn = checkAuth(); // khai báo trước

  const showMessage = () => {    // function expression với const
    if (isLoggedIn) {
      console.log('Chào mừng!');
    } else {
      console.log('Vui lòng đăng nhập.');
    }
  };

  showMessage(); // "Chào mừng!"
}

initAppFixed();
```

### Ví dụ 2: Hoisting trong các block lồng nhau

```javascript
let x = 'outer';

function demonstrate() {
  // TDZ cho x (let) trong function scope này bắt đầu
  console.log(typeof x); // "undefined" nếu là var, nhưng với let:

  // Nếu có khai báo let x phía dưới, x ở TDZ ngay từ đầu function
  // console.log(x); // ReferenceError — x trong TDZ, không tìm outer x

  let x = 'inner'; // TDZ kết thúc
  console.log(x);  // "inner"
}

// demonstrate(); // ReferenceError nếu bỏ comment dòng console.log(x)
console.log(x); // "outer"
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: Gọi function expression trước khi gán
> `var sayHi = function() {}` — `sayHi` được hoist là `undefined`. Gọi `sayHi()` trước dòng gán → `TypeError: sayHi is not a function`. Với `const`/`let` → `ReferenceError` vì TDZ. Luôn khai báo function trước khi dùng.

> [!warning] Pitfall 2: `typeof` với TDZ
> `typeof undeclaredVar` trả `"undefined"` (không throw). Nhưng `typeof letVar` khi `letVar` đang trong TDZ → `ReferenceError`. `typeof` không bypass TDZ.

> [!tip] Quy tắc thực hành để tránh hoisting bugs
> (1) Dùng `const` mặc định, `let` khi cần reassign, tránh `var`. (2) Khai báo function và biến ở đầu scope để code dễ đọc. (3) Dùng function expression với `const` thay vì function declaration nếu muốn enforce "khai báo trước dùng".

---

## 5. Phỏng vấn thường gặp

**Q1: Hoisting là gì? Giải thích bằng ví dụ.**

> Hoisting là cơ chế JS engine di chuyển khai báo biến và function lên đầu scope trước khi thực thi. `var` hoist và init thành `undefined`, nên có thể đọc được trước dòng khai báo (trả `undefined`). Function declaration hoist hoàn toàn, gọi được trước khai báo. `let`/`const` hoist vào TDZ — truy cập trước khai báo throw `ReferenceError`.

**Q2: TDZ (Temporal Dead Zone) là gì?**

> TDZ là vùng thời gian từ đầu block scope đến dòng `let`/`const` khai báo. Trong TDZ, biến tồn tại (đã được compiler ghi nhận) nhưng chưa được khởi tạo — truy cập throw `ReferenceError`. Mục đích: bắt lỗi "dùng trước khai báo" thay vì để bug âm thầm (`undefined`).

**Q3: Phân biệt function declaration và function expression về mặt hoisting?**

> Function declaration (`function foo() {}`) hoist hoàn toàn — cả tên lẫn body, gọi được trước khi khai báo. Function expression (`const foo = function() {}` hay `const foo = () => {}`) — chỉ phần khai báo biến được hoist (vào TDZ nếu `let`/`const`, hoặc `undefined` nếu `var`); body không hoist. Kết quả: gọi trước dòng gán → `TypeError` (var) hoặc `ReferenceError` (let/const).

---

## 6. Bài tập thực hành

- [ ] **Bài 1:** Dự đoán output của đoạn code sau mà không chạy, rồi verify:
  ```javascript
  console.log(a);
  console.log(b);
  var a = 1;
  let b = 2;
  ```

- [ ] **Bài 2:** Refactor đoạn code dùng `var` và function declaration sang `const`/`let` + function expression để loại bỏ mọi hoisting side effects tiềm ẩn.

---

## 7. Liên kết

- [[01-Scope]] — Hoisting xảy ra trong scope nào
- [[05-ES6-Block-scoped-let-const]] — let/const và TDZ chi tiết
- [[03-Closure]] — Closure + hoisting tạo ra nhiều bug tinh vi
