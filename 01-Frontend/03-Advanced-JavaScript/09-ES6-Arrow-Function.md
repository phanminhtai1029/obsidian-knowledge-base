---
title: "ES6: Arrow Function"
section: 03-Advanced-JavaScript
tags: [es6, arrow, this, function, fresher, frontend]
related:
  - "[[03-Closure]]"
  - "[[07-ES6-Enhanced-Object-Literals]]"
difficulty: ⭐⭐⭐
estimated_time: 35m
source: [MDN]
---

# ES6: Arrow Function

> [!summary] TL;DR
> Arrow function (`=>`) là cú pháp ngắn gọn cho function. Điểm khác biệt quan trọng nhất: **không có `this` riêng** — `this` được kế thừa từ outer lexical scope (lexical `this`). Không có `arguments` object, không dùng làm constructor. Thích hợp cho callbacks, array methods; không thích hợp cho object methods cần `this`.

---

## 1. Khái niệm

### Hai điểm khác biệt với function thường

1. **Lexical `this`** — Arrow không có `this` riêng, dùng `this` của scope bao ngoài tại thời điểm định nghĩa. Không bị ảnh hưởng bởi `call`, `apply`, `bind`.
2. **Không có `arguments`** — Không có object `arguments` tự động. Dùng rest params `...args` thay thế.

### Các dạng cú pháp

```javascript
// Multi params + body
const add = (a, b) => {
  return a + b;
};

// Multi params + implicit return (không có {})
const add = (a, b) => a + b;

// Single param — không cần ()
const double = n => n * 2;

// Không có param
const greet = () => 'Hello!';

// Trả về object — cần () bên ngoài {}
const makeUser = (name, age) => ({ name, age });
```

---

## 2. Cú pháp

### 2.1 Cú pháp rút gọn

```javascript
// Trước ES6
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(function(n) { return n * 2; });
const evens   = numbers.filter(function(n) { return n % 2 === 0; });
const sum     = numbers.reduce(function(acc, n) { return acc + n; }, 0);

// ES6 arrow
const doubled = numbers.map(n => n * 2);
const evens   = numbers.filter(n => n % 2 === 0);
const sum     = numbers.reduce((acc, n) => acc + n, 0);

console.log(doubled); // [2, 4, 6, 8, 10]
console.log(evens);   // [2, 4]
console.log(sum);     // 15
```

### 2.2 Lexical `this` — giải quyết vấn đề callback

```javascript
// Vấn đề với function thường
function TimerOld() {
  this.count = 0;

  setInterval(function() {
    this.count++; // 'this' ở đây là window (hoặc undefined trong strict mode)
    console.log(this.count); // NaN
  }, 1000);
}

// ES6: Arrow function — this = Timer instance
function Timer() {
  this.count = 0;

  setInterval(() => {
    this.count++; // 'this' = Timer instance (lexical)
    console.log(this.count); // 1, 2, 3...
  }, 1000);
}

const t = new Timer();
```

### 2.3 `this` trong class method với arrow

```javascript
class Button {
  constructor(label) {
    this.label = label;
    this.clicks = 0;
  }

  // Method shorthand — this phụ thuộc vào cách gọi
  handleClickMethod() {
    this.clicks++;
    console.log(`${this.label}: ${this.clicks} clicks`);
  }

  // Arrow function field — this luôn là instance
  handleClickArrow = () => {
    this.clicks++;
    console.log(`${this.label}: ${this.clicks} clicks`);
  };

  bindListeners(element) {
    // Method shorthand cần bind
    element.addEventListener('click', this.handleClickMethod.bind(this));

    // Arrow không cần bind — this đã đúng
    element.addEventListener('click', this.handleClickArrow);
  }
}
```

### 2.4 Không có `arguments` — dùng rest params

```javascript
// Function thường có arguments
function sumOld() {
  let total = 0;
  for (let i = 0; i < arguments.length; i++) total += arguments[i];
  return total;
}

// Arrow không có arguments — dùng rest
const sumArrow = (...nums) => nums.reduce((acc, n) => acc + n, 0);

console.log(sumArrow(1, 2, 3, 4, 5)); // 15

// Trong arrow, arguments thuộc outer function
function outer() {
  const inner = () => {
    console.log(arguments[0]); // arguments của outer, không phải inner
  };
  inner();
}
outer('hello'); // "hello"
```

### 2.5 Không dùng làm constructor

```javascript
const Person = (name) => {
  this.name = name; // this không phải instance
};

// new Person('Alice'); // TypeError: Person is not a constructor
// Arrow function không có [[Construct]] slot
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: Chaining array methods với arrow functions

```javascript
const orders = [
  { id: 1, product: 'Laptop', price: 15000000, qty: 1, status: 'delivered' },
  { id: 2, product: 'Mouse',  price: 500000,   qty: 2, status: 'pending' },
  { id: 3, product: 'Keyboard', price: 800000, qty: 1, status: 'delivered' },
  { id: 4, product: 'Monitor',  price: 5000000, qty: 2, status: 'cancelled' },
];

const report = orders
  .filter(o => o.status === 'delivered')
  .map(o => ({
    id:      o.id,
    product: o.product,
    total:   o.price * o.qty,
  }))
  .reduce((acc, o) => {
    acc.items.push(o);
    acc.grandTotal += o.total;
    return acc;
  }, { items: [], grandTotal: 0 });

console.log(report);
// { items: [{id:1,...,total:15000000},{id:3,...,total:800000}], grandTotal: 15800000 }
```

### Ví dụ 2: Debounce với arrow + closure

```javascript
function debounce(fn, delay) {
  let timerId = null;

  return (...args) => {    // arrow: this từ outer (fn context)
    clearTimeout(timerId);
    timerId = setTimeout(() => {
      fn(...args);          // arrow: this từ outer
    }, delay);
  };
}

const handleSearch = debounce((query) => {
  console.log('Searching for:', query);
  // fetch(`/api/search?q=${encodeURIComponent(query)}`)
}, 300);

// Gọi nhiều lần nhanh — chỉ execute sau 300ms kể từ lần gọi cuối
handleSearch('a');
handleSearch('al');
handleSearch('ali');
handleSearch('alice'); // chỉ cái này thực sự search sau 300ms
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: Arrow function trong object method cần `this`
> ```javascript
> const obj = {
>   name: 'Alice',
>   greet: () => `Hello, ${this.name}` // this = window/undefined, không phải obj!
> };
> console.log(obj.greet()); // "Hello, undefined"
> ```
> Dùng method shorthand `greet() {}` khi cần `this` của object.

> [!warning] Pitfall 2: `call/apply/bind` không thay đổi `this` của arrow
> `arrowFn.call({ name: 'Bob' })` — `this` trong arrow vẫn là lexical `this`, không phải `{ name: 'Bob' }`. Arrow function `this` là cố định tại thời điểm định nghĩa.

> [!tip] Khi nào dùng arrow, khi nào dùng function?
> **Arrow:** callbacks (forEach, map, filter, setTimeout), khi cần giữ `this` của outer scope, functional programming.
> **Function declaration/expression:** object methods, constructors, generator functions, khi cần `arguments`, khi cần thay đổi `this` qua call/apply/bind.

---

## 5. Phỏng vấn thường gặp

**Q1: Arrow function khác function thường điểm gì?**

> (1) **Lexical `this`** — arrow không có `this` riêng, dùng `this` của scope bao ngoài. (2) **Không có `arguments` object** — dùng rest params thay. (3) **Không thể làm constructor** — không có `new.target`, throw `TypeError` khi dùng `new`. (4) **Không có `prototype` property**. (5) Cú pháp ngắn gọn hơn.

**Q2: Tại sao class event handler hay dùng arrow function?**

> Khi pass method làm callback (`element.addEventListener('click', this.handleClick)`), `this` trong `handleClick` bị mất context — trở thành `undefined` (strict) hoặc `window`. Arrow function class field `handleClick = () => {}` capture `this` = class instance tại thời điểm định nghĩa, không cần `.bind(this)`.

**Q3: `() => ({key: value})` — tại sao cần ngoặc ngoài?**

> Arrow function với implicit return: `() => expression`. Nếu expression là object literal `{key: value}`, parser nhầm `{` là bắt đầu function body. Bọc trong `()` để disambiguate: `() => ({key: value})` — trả về object, không phải block.

---

## 6. Bài tập thực hành

- [ ] **Bài 1:** Refactor class sau để dùng arrow function cho event handlers, bỏ tất cả `.bind(this)`:
  ```javascript
  class SearchBox {
    constructor() {
      this.query = '';
      this.results = [];
      this.input = document.getElementById('search');
      this.input.addEventListener('input', this.handleInput.bind(this));
    }
    handleInput(e) { this.query = e.target.value; this.search(); }
    search() { console.log('Searching:', this.query); }
  }
  ```

- [ ] **Bài 2:** Viết `pipe(...fns)` nhận nhiều functions và trả về function mới áp dụng từng function theo thứ tự (output của fn1 là input của fn2). Dùng arrow + reduce.

---

## 7. Liên kết

- [[03-Closure]] — Arrow function và lexical scope
- [[07-ES6-Enhanced-Object-Literals]] — Phân biệt arrow vs method shorthand
- [[11-ES6-Class]] — Arrow class fields cho event handlers
