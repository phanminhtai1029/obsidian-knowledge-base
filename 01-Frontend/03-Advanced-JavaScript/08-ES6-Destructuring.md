---
title: "ES6: Destructuring"
section: 03-Advanced-JavaScript
tags: [es6, destructuring, array, object, fresher, frontend]
related:
  - "[[07-ES6-Enhanced-Object-Literals]]"
  - "[[10-ES6-Rest-Spread]]"
difficulty: ⭐⭐⭐
estimated_time: 35m
source: [MDN]
---

# ES6: Destructuring

> [!summary] TL;DR
> Destructuring cho phép **unpack** giá trị từ array/object vào biến riêng lẻ. **Array destructuring** theo vị trí (index). **Object destructuring** theo tên key — có thể đổi tên (`{name: alias}`), đặt default (`{age = 18}`), và lồng sâu. Dùng rất nhiều trong React (props, useState, useEffect deps).

---

## 1. Khái niệm

### Trước ES6

```javascript
const user = { name: 'Alice', age: 25, role: 'admin' };

// Phải viết lặp lại
const name = user.name;
const age  = user.age;
const role = user.role;

const arr    = [10, 20, 30];
const first  = arr[0];
const second = arr[1];
```

### ES6 Destructuring

```javascript
const { name, age, role } = user;     // object
const [first, second, third] = arr;   // array
```

---

## 2. Cú pháp

### 2.1 Object Destructuring cơ bản

```javascript
const config = {
  host:     'localhost',
  port:     3000,
  database: 'mydb',
  ssl:      false,
};

// Cơ bản
const { host, port, database } = config;
console.log(host, port, database); // "localhost" 3000 "mydb"

// Đổi tên (alias)
const { host: serverHost, port: serverPort } = config;
console.log(serverHost, serverPort); // "localhost" 3000

// Giá trị mặc định
const { host: h, timeout = 5000 } = config;
console.log(timeout); // 5000 (config không có timeout)

// Kết hợp alias + default
const { ssl: useSSL = true } = config;
console.log(useSSL); // false (có trong config → dùng giá trị config)
```

### 2.2 Array Destructuring cơ bản

```javascript
const rgb = [255, 128, 0];

const [r, g, b] = rgb;
console.log(r, g, b); // 255 128 0

// Bỏ qua phần tử
const [, second, , fourth] = [10, 20, 30, 40];
console.log(second, fourth); // 20 40

// Default value
const [x = 0, y = 0, z = 0] = [5, 10];
console.log(x, y, z); // 5 10 0

// Swap biến
let a = 1, b = 2;
[a, b] = [b, a];
console.log(a, b); // 2 1
```

### 2.3 Nested Destructuring

```javascript
const employee = {
  id: 1,
  name: 'Alice',
  address: {
    city: 'Hà Nội',
    district: 'Đống Đa',
    zipcode: '100000',
  },
  scores: [85, 92, 78],
};

// Nested object
const { name, address: { city, district } } = employee;
console.log(city, district); // "Hà Nội" "Đống Đa"
// Lưu ý: `address` không được tạo thành biến — chỉ `city` và `district`

// Nested array
const { scores: [s1, s2, s3] } = employee;
console.log(s1, s2, s3); // 85 92 78
```

### 2.4 Destructuring trong Function Parameters

```javascript
// Trước ES6
function greetOld(user) {
  console.log('Xin chào, ' + user.name + ' (' + user.age + ' tuổi)');
}

// ES6 — destructure trực tiếp trong params
function greet({ name, age, role = 'User' }) {
  console.log(`Xin chào, ${name} (${age} tuổi) — ${role}`);
}

greet({ name: 'Alice', age: 25, role: 'Admin' });
// "Xin chào, Alice (25 tuổi) — Admin"
greet({ name: 'Bob', age: 30 });
// "Xin chào, Bob (30 tuổi) — User"

// Array params
function first([head, ...tail]) {
  return { head, tail };
}
console.log(first([1, 2, 3, 4])); // { head: 1, tail: [2, 3, 4] }
```

### 2.5 Destructuring với Rest

```javascript
// Object rest
const { name, age, ...rest } = { name: 'Alice', age: 25, role: 'admin', dept: 'Dev' };
console.log(rest); // { role: 'admin', dept: 'Dev' }

// Array rest
const [first, second, ...remaining] = [1, 2, 3, 4, 5];
console.log(remaining); // [3, 4, 5]
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: Destructuring API response

```javascript
async function fetchUserProfile(userId) {
  const res  = await fetch(`https://api.example.com/users/${userId}`);
  const data = await res.json();

  // Destructure + rename + default
  const {
    id,
    name,
    email,
    avatar: avatarUrl = '/images/default-avatar.png',
    role   = 'user',
    settings: {
      theme     = 'light',
      language  = 'vi',
      notifications: { email: emailNotif = true } = {},
    } = {},
  } = data;

  return { id, name, email, avatarUrl, role, theme, language, emailNotif };
}
```

### Ví dụ 2: useState-like pattern + destructuring

```javascript
// Destructuring useState return value (React pattern)
function useState(initialValue) {
  let state = initialValue;
  const getState = () => state;
  const setState = (v) => { state = typeof v === 'function' ? v(state) : v; };
  return [getState, setState];
}

// Array destructuring với rename
const [count, setCount] = useState(0);
const [name,  setName]  = useState('');

setCount(prev => prev + 1);
setName('Alice');

console.log(count()); // 1
console.log(name());  // 'Alice'
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: Destructure property không tồn tại → `undefined`
> `const { missing } = {}` — `missing` là `undefined`, không throw. Nếu sau đó dùng `missing.something` → `TypeError`. Luôn set default value khi destructure key có thể vắng mặt: `const { key = defaultValue } = obj`.

> [!warning] Pitfall 2: Nested destructuring với nullish trung gian
> `const { a: { b } } = { a: null }` → `TypeError: Cannot destructure property 'b' of null`. Nếu nested value có thể null/undefined, cần default cho tầng trung gian: `const { a: { b } = {} } = { a: null }`.

> [!warning] Pitfall 3: `address` không tạo biến khi dùng nested destructuring
> `const { address: { city } } = user` — chỉ tạo biến `city`, không tạo `address`. Nếu cần cả hai: `const { address, address: { city } } = user`.

> [!tip] Destructuring trong function params giúp tự-document API
> `function render({ title, subtitle = '', theme = 'light', onClose })` — nhìn vào signature biết ngay component nhận props gì và default là gì. Không cần đọc body.

---

## 5. Phỏng vấn thường gặp

**Q1: Destructuring là gì? Phân biệt array và object destructuring?**

> Destructuring unpack giá trị từ array/object vào biến riêng. **Array destructuring** theo **vị trí** (index): `const [a, b] = [1, 2]`. **Object destructuring** theo **tên key**: `const { a, b } = { a: 1, b: 2 }` — thứ tự không quan trọng.

**Q2: Làm thế nào đổi tên khi destructure object?**

> Dùng `:` sau key: `const { originalKey: newName } = obj`. Ví dụ: `const { user_name: username } = response` — tạo biến `username` với giá trị của `response.user_name`. Có thể kết hợp với default: `const { user_name: username = 'guest' } = response`.

**Q3: Swap 2 biến bằng destructuring?**

> `let a = 1, b = 2; [a, b] = [b, a];` — array destructuring unpack `[b, a]` (tức `[2, 1]`) vào `[a, b]`, kết quả `a = 2, b = 1`. Không cần biến temp. Đây là idiom ES6 rất phổ biến.

---

## 6. Bài tập thực hành

- [ ] **Bài 1:** Cho mảng của màu RGB: `[[255,0,0], [0,255,0], [0,0,255]]`. Dùng destructuring trong `map` để trả về array `[{r,g,b}, ...]`.

- [ ] **Bài 2:** Viết function `extractConfig(config)` nhận object phức tạp có nested keys, dùng nested destructuring với defaults để trả về object phẳng gồm tất cả config values cần thiết.

---

## 7. Liên kết

- [[10-ES6-Rest-Spread]] — Rest trong destructuring (`...rest`)
- [[07-ES6-Enhanced-Object-Literals]] — Tạo object mới từ destructured values
- [[09-ES6-Arrow-Function]] — Arrow function với destructured params
