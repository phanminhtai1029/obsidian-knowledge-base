---
title: "ES6: Rest & Spread Operator"
section: 03-Advanced-JavaScript
tags: [es6, rest, spread, ..., fresher, frontend]
related:
  - "[[08-ES6-Destructuring]]"
  - "[[09-ES6-Arrow-Function]]"
difficulty: ⭐⭐
estimated_time: 30m
source: [MDN]
---

# ES6: Rest & Spread Operator

> [!summary] TL;DR
> Cả hai đều dùng `...` nhưng ngược nhau: **Rest** (`...params` trong function/destructuring) — **gom** nhiều giá trị vào 1 mảng. **Spread** (`...arr` khi gọi function/tạo array/object) — **trải** iterable ra thành các giá trị riêng lẻ. Spread tạo **shallow copy** — không deep clone.

> [!tip] 🎯 Hiểu trong 30 giây
> Cùng dấu `...` nhưng làm hai việc ngược nhau, phân biệt theo **vị trí**:
> - **Spread = "đổ hết ra".** Hình dung dốc ngược một hộp đồ ra bàn: `[...a, ...b]` đổ cả hai mảng vào một mảng mới; `{...user, age: 26}` đổ user ra rồi sửa `age`. Dùng khi *gọi hàm* hoặc *dựng array/object*.
> - **Rest = "gom hết lại".** Ngược lại, hốt mọi thứ còn dư vào một mảng: `function sum(...nums)` gom mọi đối số thành mảng `nums`. Dùng ở *tham số hàm* hoặc *vế trái destructuring*.
> - Câu nhớ: **"Dùng thì trải (spread), nhận thì gom (rest)."**
>
> **Bẫy CỰC hay ra thi — Shallow vs Deep copy:** `const b = {...a}` chỉ **copy nông 1 tầng**. Object/array *lồng bên trong* vẫn dùng chung địa chỉ → sửa `b.address.city` là `a.address.city` đổi theo! Đây là gốc bug "sao state gốc bị thay đổi?". Muốn copy sâu hoàn toàn dùng **`structuredClone(a)`** (cách hiện đại) hoặc `JSON.parse(JSON.stringify(a))` (cũ, mất Date/function). Đó cũng là lý do update state trong React phải spread từng tầng đụng tới.

---

## 1. Khái niệm

### Rest vs Spread — cùng ký hiệu, ngược chiều

```text
REST: nhiều giá trị → 1 mảng
  function fn(...args)    → args = [a, b, c, ...]
  const [head, ...tail]   → tail = [b, c, d, ...]

SPREAD: 1 iterable → nhiều giá trị
  fn(...arr)              → fn(arr[0], arr[1], ...)
  [...arr1, ...arr2]      → mảng mới gộp 2 mảng
  {...obj1, ...obj2}      → object mới merge 2 object
```

```
★ Insight ─────────────────────────────────────
• Cùng `...`, phân biệt bằng VỊ TRÍ: ở nơi NHẬN (tham số hàm, vế trái destructure)
  → REST gom lại; ở nơi DÙNG (gọi hàm, dựng array/object literal) → SPREAD trải
  ra. "Nhận thì gom, dùng thì trải" — một câu nhớ cho cả hai.
• Spread chỉ COPY NÔNG một tầng — đây là gốc rễ bug "sao state gốc bị đổi?":
  `{...state}` sao chép tầng ngoài nhưng object/array LỒNG bên trong vẫn chung
  địa chỉ. Vì vậy update immutable phải spread TỪNG TẦNG đụng tới
  (`{...s, user:{...s.user, age}}`). Cần copy sâu thật → structuredClone(). Đây
  chính là quy tắc cập nhật state trong React/Redux.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp

### 2.1 Rest Parameters

```javascript
// Thay thế arguments object — rõ ràng và array thực sự
function sum(...nums) {
  return nums.reduce((acc, n) => acc + n, 0);
}
console.log(sum(1, 2, 3, 4, 5)); // 15

// Kết hợp params thường + rest
function logWithPrefix(prefix, ...messages) {
  messages.forEach(msg => console.log(`[${prefix}] ${msg}`));
}
logWithPrefix('INFO', 'Server started', 'Port 3000', 'Ready');

// Rest PHẢI ở cuối
// function bad(a, ...b, c) {} // SyntaxError
```

### 2.2 Rest trong Destructuring

```javascript
// Array rest
const [first, second, ...rest] = [1, 2, 3, 4, 5];
console.log(first);  // 1
console.log(second); // 2
console.log(rest);   // [3, 4, 5]

// Object rest
const { id, name, ...otherProps } = {
  id: 1, name: 'Alice', age: 25, role: 'admin', dept: 'Dev'
};
console.log(otherProps); // { age: 25, role: 'admin', dept: 'Dev' }

// Dùng để tạo object không có một số keys (immutable update)
const original = { id: 1, name: 'Alice', password: 'secret', age: 25 };
const { password, ...safeUser } = original;
console.log(safeUser); // { id: 1, name: 'Alice', age: 25 }
```

### 2.3 Spread trong Function Call

```javascript
const nums = [3, 1, 4, 1, 5, 9, 2, 6];

// Trước ES6
const max = Math.max.apply(null, nums); // cồng kềnh

// ES6 spread
const max2 = Math.max(...nums);           // 9
const min  = Math.min(...nums);           // 1

function createDate(year, month, day) {
  return new Date(year, month - 1, day);
}
const dateParts = [2026, 5, 2];
const date      = createDate(...dateParts); // spread array vào params
```

### 2.4 Spread trong Array

```javascript
const fruits = ['apple', 'banana'];
const vegs   = ['carrot', 'broccoli'];

// Copy array (shallow)
const fruitsCopy = [...fruits];

// Merge arrays
const food = [...fruits, ...vegs];
console.log(food); // ['apple', 'banana', 'carrot', 'broccoli']

// Thêm phần tử ở đầu/cuối/giữa
const withMango  = [...fruits, 'mango'];
const withGrape  = ['grape', ...fruits];
const combined   = ['start', ...fruits, 'middle', ...vegs, 'end'];

// Convert iterable thành array
const nodeList  = [...document.querySelectorAll('li')]; // NodeList → Array
const charArray = [..."hello"];                         // ['h','e','l','l','o']
const setToArr  = [...new Set([1,2,2,3,3,3])];         // [1,2,3] (dedup)
```

### 2.5 Spread trong Object

```javascript
const defaults = { theme: 'light', lang: 'vi', notifications: true };
const userPrefs = { theme: 'dark', fontSize: 16 };

// Merge — sau ghi đè trước
const config = { ...defaults, ...userPrefs };
console.log(config);
// { theme: 'dark', lang: 'vi', notifications: true, fontSize: 16 }

// Thứ tự quan trọng!
const config2 = { ...userPrefs, ...defaults };
console.log(config2.theme); // 'light' (defaults ghi đè userPrefs)

// Update object immutably (React state pattern)
const user    = { id: 1, name: 'Alice', age: 25 };
const updated = { ...user, age: 26, lastLogin: new Date() };
// user không thay đổi, updated là object mới
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: Immutable state updates (React pattern)

```javascript
// State ban đầu
const state = {
  user:     { name: 'Alice', age: 25 },
  settings: { theme: 'light', notifications: true },
  cart:     [{ id: 1, name: 'Laptop', qty: 1 }],
};

// Update user age — không mutate state gốc
function updateUserAge(state, newAge) {
  return {
    ...state,
    user: { ...state.user, age: newAge },
  };
}

// Add item to cart
function addToCart(state, item) {
  return {
    ...state,
    cart: [...state.cart, item],
  };
}

// Remove item from cart
function removeFromCart(state, itemId) {
  return {
    ...state,
    cart: state.cart.filter(item => item.id !== itemId),
  };
}

const state2 = updateUserAge(state, 26);
const state3 = addToCart(state2, { id: 2, name: 'Mouse', qty: 1 });
console.log(state.user.age);  // 25 — gốc không đổi
console.log(state3.user.age); // 26
console.log(state3.cart.length); // 2
```

### Ví dụ 2: Utility functions với rest/spread

```javascript
// compose — áp dụng functions từ phải sang trái
const compose = (...fns) => x => fns.reduceRight((v, fn) => fn(v), x);

// pipe — áp dụng functions từ trái sang phải
const pipe = (...fns) => x => fns.reduce((v, fn) => fn(v), x);

const double   = x => x * 2;
const addTen   = x => x + 10;
const toString = x => `Kết quả: ${x}`;

const transform = pipe(double, addTen, toString);
console.log(transform(5)); // "Kết quả: 20" (5*2=10, 10+10=20)

// mergeDeep — merge 2 object với nested spread
function mergeDeep(target, source) {
  const result = { ...target };
  for (const key of Object.keys(source)) {
    if (source[key] && typeof source[key] === 'object' && !Array.isArray(source[key])) {
      result[key] = mergeDeep(target[key] ?? {}, source[key]);
    } else {
      result[key] = source[key];
    }
  }
  return result;
}

const base    = { a: 1, nested: { x: 1, y: 2 } };
const overlay = { b: 2, nested: { y: 99, z: 3 } };
console.log(mergeDeep(base, overlay));
// { a: 1, b: 2, nested: { x: 1, y: 99, z: 3 } }
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: Spread tạo shallow copy — không deep clone
> `const copy = { ...obj }` — chỉ copy 1 level. Nested objects vẫn là cùng reference. Thay đổi `copy.nested.value` sẽ ảnh hưởng `obj.nested.value`. Deep clone: `JSON.parse(JSON.stringify(obj))` (giới hạn: mất function, Date, undefined) hoặc `structuredClone(obj)` (ES2022+).

> [!warning] Pitfall 2: Spread object không clone methods
> `const copy = { ...instance }` — chỉ copy enumerable own properties, không copy prototype methods. Dùng `Object.create(Object.getPrototypeOf(obj))` + `Object.assign` cho full clone class instance.

> [!tip] `[...new Set(arr)]` để dedup mảng nhanh
> Tạo Set từ array (Set loại bỏ duplicate), rồi spread thành array mới. `[...new Set([1,2,2,3,3,3])]` → `[1,2,3]`. Chỉ work với primitive values (reference equality với objects).

---

## 5. Phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Shallow vs Deep copy, `{...obj}` là loại nào?"
> *"`const newObj = { ...oldObj }` là shallow copy — sao chép nông. Nó tạo object mới và copy các giá trị ở tầng ngoài cùng, nhưng nếu bên trong có object hoặc array lồng nhau thì nó chỉ copy tham chiếu, tức là hai bên vẫn trỏ chung vào cùng một object con. Hậu quả là sửa dữ liệu lồng ở bản copy thì bản gốc cũng bị đổi theo. Muốn deep copy — sao chép sâu, độc lập hoàn toàn — em dùng `structuredClone(oldObj)` là cách hiện đại và đúng nhất, hoặc `JSON.parse(JSON.stringify(oldObj))` cho dữ liệu đơn giản nhưng cách này mất Date, function và undefined. Đây cũng là lý do trong React em phải spread từng tầng khi update state lồng nhau."*

> [!note] 🧠 Mẹo nhớ
> **"Dùng thì TRẢI (spread), nhận thì GOM (rest)."** Và: **`{...obj}` = copy NÔNG; deep copy = `structuredClone()`.**

**Q1: Rest và Spread operator — giải thích và phân biệt?**

> Cả hai dùng `...`. **Rest** thu thập nhiều giá trị vào 1 mảng: xuất hiện trong function params `fn(...args)` hoặc destructuring `[head, ...tail]`. **Spread** trải iterable thành các giá trị riêng: xuất hiện khi gọi function `fn(...arr)`, tạo array mới `[...a, ...b]`, merge object `{...a, ...b}`.

**Q2: Làm thế nào copy array/object không mutate gốc?**

> Array: `const copy = [...original]`. Object: `const copy = { ...original }`. Cả hai là shallow copy — nested references vẫn chung. Deep clone: `structuredClone(obj)` (modern) hoặc `JSON.parse(JSON.stringify(obj))` (có giới hạn). Với array lồng: `original.map(item => ({ ...item }))`.

**Q3: Spread trong object — key sau ghi đè key trước?**

> Đúng. `{ ...obj1, ...obj2 }` — nếu cùng key, obj2 thắng. `{ theme: 'light', ...userPrefs }` — userPrefs.theme (nếu có) ghi đè. Thứ tự quan trọng: luôn đặt defaults trước, specific values sau.

---

## 6. Bài tập thực hành

- [ ] **Bài 1:** Viết function `mergeConfig(defaults, ...overrides)` nhận 1 config mặc định và nhiều override objects, merge theo thứ tự (sau ghi đè trước). Xử lý undefined keys.

- [ ] **Bài 2:** Viết function `groupBy(arr, key)` nhận mảng objects và 1 key, trả về object gom các items theo giá trị của key. Dùng spread + rest.

---

## 7. Liên kết

- [[08-ES6-Destructuring]] — Rest trong destructuring pattern
- [[09-ES6-Arrow-Function]] — Rest params trong arrow function
- [[11-ES6-Class]] — Spread để clone/extend objects từ class
