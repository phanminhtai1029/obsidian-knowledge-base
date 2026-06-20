---
title: "JavaScript Type Coercion"
section: 03-Advanced-JavaScript
tags: [js, coercion, ==, ===, ToPrimitive, fresher, frontend]
related:
  - "[[05-ES6-Block-scoped-let-const]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 45m
source: [YDKJS, javascript.info]
---

# JavaScript Type Coercion

> [!summary] TL;DR
> **Coercion** = JS tự động chuyển đổi kiểu dữ liệu. **Implicit coercion** xảy ra ngầm (`1 + '2'` → `'12'`). **Explicit coercion** là bạn chủ động convert (`Number('42')`). `==` dùng Abstract Equality (có coerce), `===` không coerce. Luôn dùng `===` trong production. Hiểu các WAT classic để tránh bug ngầm.

---

## 1. Khái niệm

### Implicit vs Explicit Coercion

```javascript
// Explicit — bạn chủ động chuyển đổi
Number('42')     // 42
String(100)      // "100"
Boolean(0)       // false
parseInt('3.7')  // 3

// Implicit — JS tự động chuyển đổi
'5' - 2         // 3  (string → number vì toán tử -)
'5' + 2         // "52"  (number → string vì toán tử + với string)
!!''            // false
if (1) {}       // 1 coerce sang boolean
```

### ToPrimitive — thuật toán nội bộ

Khi JS cần chuyển object sang primitive, nó gọi **ToPrimitive(hint)**:
- `hint = "number"`: thử `valueOf()` trước, rồi `toString()`
- `hint = "string"`: thử `toString()` trước, rồi `valueOf()`
- `hint = "default"`: thường như `"number"`

### Falsy Values (7 giá trị)

```javascript
// Chỉ 7 giá trị falsy trong JS:
false, 0, -0, 0n, '', null, undefined, NaN

// Mọi thứ khác đều truthy — kể cả:
[], {}, 'false', '0', -1, Infinity
```

```
★ Insight ─────────────────────────────────────
• Toán tử `+` có "thiên vị chuỗi": chỉ cần MỘT toán hạng là string, + thành nối
  chuỗi; mọi toán tử số học khác (- * / %) lại ép về number. Đây là gốc của bug
  kinh điển "price + qty = '1000003'" — input form luôn là string. Quy tắc sống
  còn: Number()/parseInt() TRƯỚC khi tính. Liên hệ [[04-Modifying-DOM-Elements]].
• Học falsy như một DANH SÁCH ĐÓNG (đúng 7 giá trị) thay vì đoán: false, 0, -0,
  0n, '', null, undefined, NaN. [] và {} TRUTHY (bẫy hay gặp). Và phân biệt
  `||` (rơi vào fallback với MỌI falsy, nuốt nhầm 0 và '') vs `??` (chỉ rơi khi
  null/undefined) — chọn `??` khi 0 hoặc '' là giá trị HỢP LỆ.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp

### 2.1 String conversion — toán tử `+`

```javascript
// + với string → string concatenation
'Hello ' + 42       // "Hello 42"   (number → string)
'Result: ' + true   // "Result: true"
'Value: ' + null    // "Value: null"
'Value: ' + undefined // "Value: undefined"

// + với 2 number → addition
1 + 2               // 3

// + unary → to number
+'42'     // 42
+''       // 0
+true     // 1
+false    // 0
+null     // 0
+undefined // NaN
+[]       // 0   ([] → '' → 0)
+{}       // NaN  ({} → '[object Object]' → NaN)
```

### 2.2 Number conversion — toán tử `-`, `*`, `/`

```javascript
'6' - 2     // 4   (string → number)
'6' * '2'   // 12
'6' / '3'   // 2
true  + 1   // 2   (true → 1)
false + 1   // 1   (false → 0)
null  + 1   // 1   (null → 0)
undefined + 1 // NaN (undefined → NaN)
```

### 2.3 `==` Abstract Equality — các quy tắc coercion

```javascript
// Quy tắc chính của ==:
// 1. Nếu 2 bên cùng kiểu → so sánh trực tiếp (như ===)
// 2. null == undefined → true (và ngược lại)
// 3. number == string → convert string sang number
// 4. boolean == * → convert boolean sang number (true→1, false→0)
// 5. object == number/string → ToPrimitive(object)

null == undefined  // true  ← quy tắc đặc biệt
null == 0          // false ← null chỉ == undefined/null
undefined == false // false ← undefined chỉ == null/undefined

1 == '1'           // true  ← '1' → 1
0 == false         // true  ← false → 0
0 == ''            // true  ← '' → 0
0 == '0'           // true  ← '0' → 0
'' == false        // true  ← false → 0, '' → 0

[] == false        // true  ← false → 0, [] → '' → 0
[] == ![]          // true  ← ![] = false → 0, [] → 0
```

### 2.4 `===` Strict Equality — không coerce

```javascript
1 === '1'          // false — khác kiểu
0 === false        // false
null === undefined // false
[] === []          // false — khác reference

// Chỉ true khi cùng kiểu VÀ cùng giá trị
1 === 1            // true
'hi' === 'hi'      // true
null === null      // true
undefined === undefined // true
```

### 2.5 Explicit conversion functions

```javascript
// To Number
Number('42')         // 42
Number('')           // 0
Number(null)         // 0
Number(undefined)    // NaN
Number(true)         // 1
Number(false)        // 0
Number('3.14abc')    // NaN
parseInt('3.14')     // 3
parseFloat('3.14px') // 3.14
+'42'                // 42

// To String
String(42)           // "42"
String(null)         // "null"
String(undefined)    // "undefined"
String(true)         // "true"
(42).toString()      // "42"
(255).toString(16)   // "ff"

// To Boolean
Boolean(0)           // false
Boolean('')          // false
Boolean(null)        // false
Boolean(undefined)   // false
Boolean(NaN)         // false
Boolean([])          // true  ← mảng rỗng là truthy!
Boolean({})          // true  ← object rỗng là truthy!
!!value              // shorthand to boolean
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: WAT — các kết quả "kỳ lạ" của JS

```javascript
// Classic WAT cases
console.log([] + []);       // ""    — [] → '', '' + '' = ''
console.log([] + {});       // "[object Object]" — [] → '', {} → '[object Object]'
console.log({} + []);       // "[object Object]" nếu trong expression, hoặc 0 nếu standalone

console.log(+'');            // 0     — '' → 0
console.log(+[]);            // 0     — [] → '' → 0
console.log(+{});            // NaN   — {} → '[object Object]' → NaN

console.log([] == false);    // true  — [] → '' → 0, false → 0
console.log([] == ![]);      // true  — ![] = false, [] == false = true
console.log({} == !{});      // false — {} != false (object equality khác)

console.log(1 + '2' + 3);   // "123" — left to right: 1+'2'='12', '12'+3='123'
console.log(1 + 2 + '3');   // "33"  — 1+2=3, 3+'3'='33'

// Thứ tự toán tử quan trọng!
typeof null === 'object'  // true — JS bug lịch sử, null không phải object
```

### Ví dụ 2: Coercion trong thực tế — form input

```javascript
// Form inputs luôn trả chuỗi — hay gây bug khi tính toán
function calculateTotal(form) {
  const priceEl = document.getElementById('price');
  const qtyEl   = document.getElementById('quantity');

  const priceStr = priceEl.value;    // e.g. "100000"
  const qtyStr   = qtyEl.value;      // e.g. "3"

  // Bug: string concatenation thay vì addition
  const buggyTotal = priceStr + qtyStr;      // "1000003" — SAI!

  // Đúng: explicit convert trước khi tính
  const price    = Number(priceStr);
  const quantity = Number(qtyStr);
  const total    = price * quantity;          // 300000 — ĐÚNG

  // Validate NaN
  if (isNaN(price) || isNaN(quantity)) {
    const errorEl = document.getElementById('error');
    errorEl.textContent = 'Vui lòng nhập số hợp lệ';
    return null;
  }

  return total;
}
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: Dùng `==` với `false` / `0` / `''`
> Ba giá trị này đều `==` nhau qua coercion. `if (value == false)` bắt được cả `0`, `''`, `false`, `null` sẽ cho kết quả không mong đợi. Dùng `if (!value)` hoặc check cụ thể từng trường hợp.

> [!warning] Pitfall 2: `null == 0` là `false`
> `null` chỉ `==` với `undefined` và chính nó. `null == 0` là `false`, `null == ''` là `false`. Điều này ngược lại với intuition vì `null` falsy, `0` falsy nhưng `null == 0` lại false. Luôn dùng `=== null` hoặc `=== undefined` để kiểm tra.

> [!warning] Pitfall 3: `typeof NaN === 'number'`
> `NaN` (Not a Number) có kiểu `"number"`. `NaN !== NaN` (NaN không bằng chính nó). Kiểm tra NaN bằng `Number.isNaN(value)` hoặc `isNaN(value)` (nhưng `isNaN` coerce trước).

> [!tip] Quy tắc thực hành
> (1) Luôn dùng `===` thay `==`. (2) Convert explicit trước khi dùng input từ form/API: `Number()`, `parseInt()`. (3) Check `isNaN()` sau khi convert. (4) Dùng `?? nullish coalescing` thay `||` khi cần tránh coerce `0`/`''` thành falsy.

---

## 5. Phỏng vấn thường gặp

**Q1: `==` và `===` khác nhau thế nào?**

> `==` (Abstract Equality) cho phép coercion kiểu trước khi so sánh — JS chuyển đổi 2 vế về cùng kiểu rồi so sánh. `===` (Strict Equality) không coerce — nếu khác kiểu luôn `false`. Trong production luôn dùng `===` để tránh bug ngầm từ coercion bất ngờ.

**Q2: Giải thích tại sao `[] == false` là `true`?**

> Theo Abstract Equality: (1) `false` là boolean → convert sang number: `false → 0`. (2) Bây giờ so sánh `[] == 0`: `[]` là object → ToPrimitive → `[].toString()` → `''` → `Number('')` → `0`. (3) `0 == 0` → `true`. Chuỗi coercion: `[] == false` → `[] == 0` → `'' == 0` → `0 == 0` → `true`.

**Q3: Các falsy values trong JavaScript?**

> Chỉ có đúng 7: `false`, `0`, `-0`, `0n` (BigInt zero), `''` (chuỗi rỗng), `null`, `undefined`, `NaN`. Mọi giá trị khác đều truthy — kể cả mảng rỗng `[]`, object rỗng `{}`, chuỗi `'false'`, số `-1`, và `Infinity`.

---

## 6. Bài tập thực hành

- [ ] **Bài 1:** Dự đoán output trước khi chạy:
  ```javascript
  console.log(1 + '1');
  console.log(+'1' + 1);
  console.log(null + 1);
  console.log(undefined + 1);
  console.log([] + []);
  console.log(true + true);
  ```

- [ ] **Bài 2:** Viết function `safeToNumber(value)` nhận bất kỳ giá trị nào, trả về `null` nếu không convert được thành số hữu hạn, trả về số nếu convert được. Xử lý `null`, `undefined`, `''`, `[]`, `{}`, chuỗi số, chuỗi không phải số.

---

## 7. Liên kết

- [[05-ES6-Block-scoped-let-const]] — `const` giúp tránh accidental reassign liên quan coercion
- [[01-Scope]] — Understanding execution context
