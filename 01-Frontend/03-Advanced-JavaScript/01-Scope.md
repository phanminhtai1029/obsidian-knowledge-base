---
title: "JavaScript Scope"
section: 03-Advanced-JavaScript
tags: [js, scope, lexical, chain, fresher, frontend]
related:
  - "[[02-Hoisting]]"
  - "[[03-Closure]]"
  - "[[05-ES6-Block-scoped-let-const]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [MDN, YDKJS]
---

# JavaScript Scope

> [!summary] TL;DR
> **Scope** xác định nơi một biến có thể được truy cập. JS có 3 loại: **Global** (toàn chương trình), **Function** (`var` khai báo trong function), **Block** (`let`/`const` bên trong `{}`). **Scope chain** — khi không tìm thấy biến, JS leo lên scope cha cho đến Global; nếu không có thì `ReferenceError`.

---

## 1. Khái niệm

### Ba loại Scope

| Loại | Tạo bởi | Từ khóa |
|---|---|---|
| **Global Scope** | Ngoài mọi function/block | `var`, `let`, `const` |
| **Function Scope** | Bên trong `function () {}` | `var`, `let`, `const` |
| **Block Scope** | Bên trong `{ }` (if/for/while…) | `let`, `const` (KHÔNG phải `var`) |

### Lexical Scope (Static Scope)

JS dùng **lexical scoping** — scope của một function được xác định tại **nơi nó được định nghĩa** (trong source code), không phải nơi nó được gọi.

```javascript
const x = 'global';

function outer() {
  const x = 'outer';

  function inner() {
    // inner nhìn thấy x = 'outer' (nơi inner được định nghĩa)
    console.log(x); // "outer"
  }

  inner();
}

outer();
```

### Scope Chain

Khi JS gặp một biến, nó tìm theo thứ tự:
1. Scope hiện tại
2. Scope của function bao ngoài
3. ... (leo lên tiếp)
4. Global scope
5. `ReferenceError` nếu không tìm thấy

---

## 2. Cú pháp

### 2.1 Global Scope

```javascript
var globalVar     = 'tôi là global';    // thuộc window.globalVar
let globalLet     = 'tôi cũng global';  // KHÔNG thuộc window
const globalConst = 'tôi cũng global';  // KHÔNG thuộc window

function anyFunction() {
  console.log(globalVar);   // "tôi là global" — truy cập được
  console.log(globalLet);   // "tôi cũng global" — truy cập được
}
```

### 2.2 Function Scope — `var` bị giam trong function

```javascript
function myFunction() {
  var funcVar  = 'chỉ sống trong function';
  let funcLet  = 'cũng vậy';
  const funcConst = 'cũng vậy';
}

myFunction();
// console.log(funcVar);  // ReferenceError: funcVar is not defined
```

### 2.3 Block Scope — `let`/`const` bị giam trong block

```javascript
{
  var blockVar   = 'var KHÔNG bị block';   // escape ra ngoài block!
  let blockLet   = 'let BỊ block';
  const blockConst = 'const BỊ block';
}

console.log(blockVar);    // "var KHÔNG bị block" — var rò ra ngoài
// console.log(blockLet); // ReferenceError
```

### 2.4 Scope Chain thực tế

```javascript
const level1 = 'level 1';

function scopeLevel2() {
  const level2 = 'level 2';

  function scopeLevel3() {
    const level3 = 'level 3';

    // Có thể truy cập tất cả level trên
    console.log(level1); // "level 1"
    console.log(level2); // "level 2"
    console.log(level3); // "level 3"
  }

  // Không thể truy cập level3
  // console.log(level3); // ReferenceError
  scopeLevel3();
}

scopeLevel2();
// console.log(level2); // ReferenceError
```

### 2.5 Vấn đề kinh điển: `var` trong vòng lặp

```javascript
// Vấn đề với var — chỉ có 1 biến i cho cả vòng lặp
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Output: 3, 3, 3 (không phải 0, 1, 2!)

// Giải pháp 1: dùng let — mỗi iteration có block scope riêng
for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 100);
}
// Output: 0, 1, 2

// Giải pháp 2: IIFE (trước ES6)
for (var k = 0; k < 3; k++) {
  (function(captured) {
    setTimeout(() => console.log(captured), 100);
  })(k);
}
// Output: 0, 1, 2
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: Module pattern dùng closure + scope

```javascript
const counterModule = (function () {
  let count = 0; // private — bên ngoài không truy cập được

  return {
    increment() { count++; },
    decrement() { count--; },
    getCount()  { return count; },
    reset()     { count = 0; },
  };
})();

counterModule.increment();
counterModule.increment();
counterModule.increment();
counterModule.decrement();
console.log(counterModule.getCount()); // 2
// console.log(count);   // ReferenceError — count là private
```

**Giải thích:** IIFE tạo function scope riêng. `count` chỉ tồn tại trong scope đó. Object trả về (public API) giữ reference tới scope đó qua closure.

### Ví dụ 2: Scope chain trong thực tế — config lookup

```javascript
const appConfig = { env: 'production', timeout: 5000 };

function createRequestHandler(endpoint) {
  const baseUrl = 'https://api.example.com';   // function scope

  return function handleRequest(params) {
    const fullUrl = baseUrl + endpoint;   // truy cập scope cha
    const timeout = appConfig.timeout;    // truy cập global scope

    console.log(`[${appConfig.env}] GET ${fullUrl}`);
    console.log('Timeout:', timeout, 'ms');
    return { url: fullUrl, params, timeout };
  };
}

const getUsers  = createRequestHandler('/users');
const getPosts  = createRequestHandler('/posts');

getUsers({ page: 1 });
// "[production] GET https://api.example.com/users"
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: `var` trong loop với async callback
> `var` không có block scope — mọi callback trong loop dùng chung 1 biến `i`. Luôn dùng `let` trong for loops. Đây là lý do `let` được tạo ra.

> [!warning] Pitfall 2: Vô tình tạo global variable
> Gán biến không khai báo (`x = 5` thay vì `let x = 5`) sẽ tạo **global variable** — ngay cả bên trong function. Trong strict mode (`'use strict'`), điều này throw `ReferenceError`. Luôn dùng `let`/`const`.

> [!tip] Lexical scope vs Dynamic scope
> JS dùng **lexical scope** (tĩnh, xác định khi viết code). Một số ngôn ngữ khác (Perl, Bash) dùng dynamic scope (xác định khi chạy). Arrow function `this` cũng là lexical — `this` của arrow function là `this` của nơi nó được định nghĩa, không phải nơi được gọi.

```
★ Insight ─────────────────────────────────────
• "Lexical" = scope khóa chặt theo VỊ TRÍ VIẾT trong source, không phải nơi gọi.
  Đọc code là biết ngay biến nào thấy được biến nào — không cần chạy thử. Đây là
  nền của [[03-Closure]]: hàm "nhớ" được scope nơi nó SINH RA, mang theo cả khi
  bị gọi ở chỗ khác.
• Scope chain chỉ tra cứu MỘT CHIỀU: trong nhìn ra ngoài được, ngoài KHÔNG nhìn
  vào trong. Hệ quả thực tế: biến private (count trong module pattern) an toàn vì
  global không có đường "chui vào" function scope. let trong for tạo binding MỚI
  mỗi vòng → mỗi closure chộp đúng giá trị, còn var chỉ 1 binding dùng chung → bug 3,3,3.
─────────────────────────────────────────────────
```

---

## 5. Phỏng vấn thường gặp

**Q1: Phân biệt Function Scope và Block Scope?**

> Function scope: biến khai báo bằng `var` bên trong function chỉ sống trong function đó. Block scope: biến khai báo bằng `let`/`const` bên trong `{}` (if, for, while, …) chỉ sống trong block đó. `var` không có block scope — nó escape ra function scope gần nhất.

**Q2: Scope chain là gì? JS tìm biến như thế nào?**

> Scope chain là chuỗi các scope lồng nhau. Khi gặp biến, JS tìm từ scope hiện tại, leo lên scope cha, ông, ... cho đến global. Nếu không tìm thấy ở đâu → `ReferenceError`. Đây là cơ chế cho phép inner function truy cập biến outer function.

**Q3: Tại sao `var` trong for loop với setTimeout cho kết quả sai?**

> `var` chỉ có function scope, không có block scope. Mọi iteration dùng chung 1 biến `i`. Khi setTimeout callback chạy (sau khi loop xong), `i` đã bằng giá trị cuối. Dùng `let` tạo binding riêng cho mỗi iteration → mỗi callback capture đúng giá trị.

---

## 6. Bài tập thực hành

- [ ] **Bài 1:** Viết function `createMultiplier(x)` trả về function nhận `y` và trả về `x * y`. Gọi `const double = createMultiplier(2)` rồi dùng `double(5)`, `double(10)`. Giải thích scope chain hoạt động.

- [ ] **Bài 2:** Tìm bug trong đoạn code sau và sửa:
  ```javascript
  const buttons = document.querySelectorAll('button');
  for (var i = 0; i < buttons.length; i++) {
    buttons[i].addEventListener('click', function() {
      console.log('Button ' + i + ' clicked');
    });
  }
  ```

---

## 7. Liên kết

- [[02-Hoisting]] — var/let/const hoisting liên quan trực tiếp đến scope
- [[03-Closure]] — Closure là ứng dụng thực tế nhất của lexical scope
- [[05-ES6-Block-scoped-let-const]] — let/const giải quyết vấn đề var scope
- [[09-ES6-Arrow-Function]] — Arrow function và lexical `this`
