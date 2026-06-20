---
title: "ES6: Enhanced Object Literals"
section: 03-Advanced-JavaScript
tags: [es6, object, shorthand, method, fresher, frontend]
related:
  - "[[08-ES6-Destructuring]]"
  - "[[06-ES6-Template-Literals]]"
difficulty: ⭐⭐
estimated_time: 25m
source: [MDN]
---

# ES6: Enhanced Object Literals

> [!summary] TL;DR
> ES6 cải tiến cú pháp object literal: **Shorthand property** (`{name}` thay `{name: name}`), **Shorthand method** (`greet() {}` thay `greet: function() {}`), **Computed property name** (`{[key]: value}`). Giảm code thừa, tăng tính đọc được, dùng rất nhiều trong React/Node.

---

## 1. Khái niệm

### Ba cải tiến chính

| Tính năng | Trước ES6 | ES6 |
|---|---|---|
| Property shorthand | `{ name: name }` | `{ name }` |
| Method shorthand | `{ greet: function() {} }` | `{ greet() {} }` |
| Computed property | `obj[key] = value` (sau khai báo) | `{ [key]: value }` |

```
★ Insight ─────────────────────────────────────
• Property shorthand `{name, age}` chỉ là `{name: name, age: age}` — nhưng nó là
  cú pháp bạn gặp NHIỀU NHẤT khi return object hoặc setState({name, email}). Đối
  xứng với object destructuring ([[08-ES6-Destructuring]]): một bên dựng, một bên tháo, cùng dạng `{name}`.
• Method shorthand `greet(){}` ≠ arrow property `greet: ()=>{}`: shorthand có
  this trỏ vào object (đúng cho method cần this), còn arrow lấy this lexical
  (thường KHÔNG phải object). Đây là lỗi tinh vi — định nghĩa method object bằng
  arrow rồi dùng this.something sẽ ra undefined. Quy tắc ở [[09-ES6-Arrow-Function]].
─────────────────────────────────────────────────
```

---

## 2. Cú pháp

### 2.1 Property Shorthand

```javascript
const name  = 'Alice';
const age   = 25;
const email = 'alice@example.com';

// Trước ES6
const userOld = { name: name, age: age, email: email };

// ES6 shorthand — tên property trùng tên biến
const user = { name, age, email };

console.log(user); // { name: 'Alice', age: 25, email: 'alice@example.com' }

// Kết hợp shorthand và tên khác
const x = 10, y = 20;
const point = { x, y, label: 'Origin' };
```

### 2.2 Method Shorthand

```javascript
// Trước ES6
const calculatorOld = {
  value: 0,
  add: function(n) { this.value += n; return this; },
  sub: function(n) { this.value -= n; return this; },
  result: function() { return this.value; },
};

// ES6 method shorthand
const calculator = {
  value: 0,
  add(n) { this.value += n; return this; },
  sub(n) { this.value -= n; return this; },
  mul(n) { this.value *= n; return this; },
  reset() { this.value = 0; return this; },
  result() { return this.value; },
};

console.log(
  calculator.add(10).add(5).sub(3).result()
); // 12
```

### 2.3 Computed Property Names

```javascript
const prefix = 'get';
const field  = 'Name';

// Trước ES6: phải tạo object trước, rồi add property
const objOld = {};
objOld[prefix + field] = function() { return 'Alice'; };

// ES6: computed trong object literal
const obj = {
  [prefix + field]() { return 'Alice'; },
  ['set' + field](v) { this._name = v; },
};

console.log(obj.getName()); // "Alice"

// Thực tế: dynamic action types (Redux pattern)
const actionTypes = { INCREMENT: 'INCREMENT', DECREMENT: 'DECREMENT' };
const actionType  = actionTypes.INCREMENT;

const handlers = {
  [actionTypes.INCREMENT]: (state) => ({ ...state, count: state.count + 1 }),
  [actionTypes.DECREMENT]: (state) => ({ ...state, count: state.count - 1 }),
};

const state   = { count: 0 };
const handler = handlers[actionType];
console.log(handler(state)); // { count: 1 }
```

### 2.4 Kết hợp tất cả

```javascript
function createUser(name, age, role) {
  const id        = Date.now();
  const createdAt = new Date().toISOString();

  return {
    id,           // shorthand
    name,         // shorthand
    age,          // shorthand
    role,         // shorthand
    createdAt,    // shorthand
    [`can${role}`]: true,   // computed: canAdmin, canUser, v.v.

    // method shorthand
    greet() {
      return `Xin chào, tôi là ${this.name} (${this.role})`;
    },
    isAdult() {
      return this.age >= 18;
    },
    toJSON() {
      return { id: this.id, name: this.name, role: this.role };
    },
  };
}

const alice = createUser('Alice', 25, 'Admin');
console.log(alice.greet());   // "Xin chào, tôi là Alice (Admin)"
console.log(alice.canAdmin);  // true
console.log(alice.toJSON());  // { id: ..., name: 'Alice', role: 'Admin' }
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: Xây dựng API service object

```javascript
const BASE_URL = 'https://api.example.com';

function createApiService(resource) {
  const url = `${BASE_URL}/${resource}`;

  return {
    // method shorthand + template literal
    async getAll(params = {}) {
      const query   = new URLSearchParams(params).toString();
      const fullUrl = query ? `${url}?${query}` : url;
      const res     = await fetch(fullUrl);
      if (!res.ok) throw new Error(`GET ${resource} failed: ${res.status}`);
      return res.json();
    },

    async getById(id) {
      const res = await fetch(`${url}/${id}`);
      if (!res.ok) throw new Error(`GET ${resource}/${id} failed: ${res.status}`);
      return res.json();
    },

    async create(data) {
      const res = await fetch(url, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      });
      if (!res.ok) throw new Error(`POST ${resource} failed: ${res.status}`);
      return res.json();
    },
  };
}

const userService = createApiService('users');
const postService = createApiService('posts');
// userService.getAll({ page: 1 })
// postService.getById(42)
```

### Ví dụ 2: Config object với computed keys

```javascript
function buildFormConfig(fields) {
  const validators = {};
  const defaults   = {};
  const labels     = {};

  fields.forEach(({ name, label, defaultValue, validate }) => {
    labels[name]     = label;
    defaults[name]   = defaultValue ?? '';
    validators[name] = validate ?? (() => '');
  });

  return { labels, defaults, validators };
}

const config = buildFormConfig([
  { name: 'username', label: 'Tên đăng nhập', defaultValue: '', validate: v => v.length < 3 ? 'Quá ngắn' : '' },
  { name: 'email',    label: 'Email',          defaultValue: '' },
  { name: 'age',      label: 'Tuổi',           defaultValue: 18 },
]);

console.log(config.labels.username);           // "Tên đăng nhập"
console.log(config.validators.username('ab')); // "Quá ngắn"
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: Method shorthand và `this`
> Method shorthand `{ greet() {} }` dùng `this` bình thường như method ES5. Tuy nhiên **arrow function** trong object `{ greet: () => {} }` không có `this` riêng — `this` sẽ là outer scope. Dùng shorthand `greet() {}` khi cần `this`, dùng arrow khi không.

> [!warning] Pitfall 2: Computed property với side effects
> Computed property `{ [expensiveCall()]: value }` được evaluate một lần khi object literal được tạo. Không nên dùng expression có side effects (API call, random) trong computed keys.

> [!tip] Property shorthand rất phổ biến trong React
> `setState({ name, email, age })` thay vì `setState({ name: name, email: email, age: age })`. Shorthand cũng dùng nhiều khi return từ function: `return { user, token, expires }`.

---

## 5. Phỏng vấn thường gặp

**Q1: Property shorthand là gì?**

> Khi tên property và tên biến giống nhau, ES6 cho phép viết `{ name }` thay vì `{ name: name }`. Compiler hiểu đó là `name: name`. Dùng rất phổ biến khi return object từ function hoặc tạo object từ nhiều biến.

**Q2: Method shorthand khác gì function property?**

> `{ greet() {} }` — method shorthand, ngắn hơn, có thể dùng `super` trong class. `{ greet: function() {} }` — function property ES5, tương đương về behavior nhưng verbose hơn. `{ greet: () => {} }` — arrow function, **không có** `this` riêng. Prefer shorthand trong code mới.

**Q3: Computed property name dùng khi nào?**

> Khi tên property chỉ biết lúc runtime (dynamic). Ví dụ: `{ [actionType]: handler }` trong Redux reducer, `{ [fieldName]: value }` khi build form state, `{ [key]: obj[key] }` khi map/transform object keys.

---

## 6. Bài tập thực hành

- [ ] **Bài 1:** Refactor object sau sang cú pháp ES6 (shorthand + method shorthand):
  ```javascript
  const x = 10, y = 20, label = 'Point A';
  const point = {
    x: x, y: y, label: label,
    toString: function() { return '(' + this.x + ', ' + this.y + ')'; },
    distanceTo: function(other) { return Math.hypot(this.x - other.x, this.y - other.y); }
  };
  ```

- [ ] **Bài 2:** Dùng computed property name xây dựng object `translations` từ mảng `[{lang: 'vi', text: 'Xin chào'}, {lang: 'en', text: 'Hello'}]`.

---

## 7. Liên kết

- [[08-ES6-Destructuring]] — Destructure object tạo ra với shorthand
- [[09-ES6-Arrow-Function]] — Phân biệt method shorthand vs arrow trong object
- [[11-ES6-Class]] — Method shorthand dùng nhiều trong class definition
