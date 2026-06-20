---
title: "JavaScript Closure"
section: 03-Advanced-JavaScript
tags: [js, closure, module, counter, fresher, frontend]
related:
  - "[[01-Scope]]"
  - "[[02-Hoisting]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 45m
source: [MDN, YDKJS]
---

# JavaScript Closure

> [!summary] TL;DR
> **Closure** = function + lexical environment nơi nó được định nghĩa. Inner function "đóng gói" (close over) các biến của outer scope, giữ chúng sống ngay cả khi outer function đã return xong. Ứng dụng thực tế: counter, module pattern, partial application, data encapsulation, mô phỏng useState.

---

## 1. Khái niệm

### Định nghĩa

Closure xảy ra khi một function **truy cập biến từ outer scope** sau khi outer function đã return:

```text
function outer() {
  let x = 10;            ← biến trong outer scope
  return function inner() {
    return x;            ← inner "close over" biến x
  };
}
const fn = outer();     ← outer đã return, nhưng x vẫn sống
fn();                   → 10  ← x vẫn truy cập được
```

### Tại sao biến không bị garbage collected?

Khi `inner` function được trả ra và vẫn có reference, JS engine **không** giải phóng lexical environment của `outer`. Biến `x` vẫn sống trong bộ nhớ bởi vì `inner` cần nó.

### Closure là reference, không phải copy

Closure giữ **reference** tới biến, không phải snapshot giá trị. Nếu biến thay đổi sau này, closure thấy giá trị mới.

---

## 2. Cú pháp

### 2.1 Closure cơ bản

```javascript
function makeCounter() {
  let count = 0;   // private — bên ngoài không truy cập

  return {
    increment() { count++; },
    decrement() { count--; },
    value()     { return count; },
  };
}

const counter = makeCounter();
counter.increment();
counter.increment();
counter.increment();
counter.decrement();
console.log(counter.value()); // 2

// count hoàn toàn riêng tư
// console.log(count); // ReferenceError
```

### 2.2 Closure với tham số

```javascript
function makeAdder(x) {
  return function(y) {
    return x + y;   // x được "đóng gói" vào closure
  };
}

const add5  = makeAdder(5);
const add10 = makeAdder(10);

console.log(add5(3));   // 8
console.log(add10(3));  // 13
console.log(add5(add10(2))); // 5 + (10 + 2) = 17
```

### 2.3 Closure giữ reference (không phải copy)

```javascript
function makeAccumulator() {
  let total = 0;
  return function(amount) {
    total += amount;   // reference — total thực sự thay đổi
    return total;
  };
}

const acc = makeAccumulator();
console.log(acc(10)); // 10
console.log(acc(20)); // 30
console.log(acc(5));  // 35
```

### 2.4 Module Pattern

```javascript
const userStore = (function () {
  // Private state
  let users = [];
  let nextId = 1;

  // Private helper
  function findById(id) {
    return users.find(u => u.id === id);
  }

  // Public API
  return {
    add(name, email) {
      const user = { id: nextId++, name, email };
      users.push(user);
      return user;
    },
    remove(id) {
      users = users.filter(u => u.id !== id);
    },
    get(id) {
      return findById(id);
    },
    getAll() {
      return [...users]; // return copy — không expose mảng gốc
    },
    count() {
      return users.length;
    },
  };
})();

userStore.add('Alice', 'alice@example.com');
userStore.add('Bob',   'bob@example.com');
console.log(userStore.count());    // 2
console.log(userStore.get(1));     // { id: 1, name: 'Alice', ... }
userStore.remove(1);
console.log(userStore.getAll());   // [{ id: 2, name: 'Bob', ... }]
```

### 2.5 Mô phỏng useState với Closure

```javascript
function useState(initialValue) {
  let state = initialValue;

  function getState() {
    return state;
  }

  function setState(newValue) {
    state = typeof newValue === 'function'
      ? newValue(state)    // functional update
      : newValue;
  }

  return [getState, setState];
}

const [getCount, setCount] = useState(0);

console.log(getCount()); // 0
setCount(5);
console.log(getCount()); // 5
setCount(prev => prev + 1);
console.log(getCount()); // 6
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: Event handler factory với closure

```javascript
function createButtonHandlers() {
  let clickCount = 0;
  let lastClickTime = null;

  function handleClick(label) {
    clickCount++;
    lastClickTime = new Date().toLocaleTimeString('vi-VN');
    console.log(`[${label}] Lần click thứ ${clickCount} lúc ${lastClickTime}`);
  }

  function getStats() {
    return { clickCount, lastClickTime };
  }

  return { handleClick, getStats };
}

const btnA = createButtonHandlers();
const btnB = createButtonHandlers(); // instance riêng biệt

btnA.handleClick('Button A'); // [Button A] Lần click thứ 1
btnA.handleClick('Button A'); // [Button A] Lần click thứ 2
btnB.handleClick('Button B'); // [Button B] Lần click thứ 1 (count riêng)

console.log(btnA.getStats()); // { clickCount: 2, lastClickTime: '...' }
console.log(btnB.getStats()); // { clickCount: 1, lastClickTime: '...' }
```

### Ví dụ 2: Memoization với closure

```javascript
function memoize(fn) {
  const cache = new Map(); // cache sống trong closure

  return function(...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      console.log('Cache hit:', key);
      return cache.get(key);
    }

    const result = fn(...args);
    cache.set(key, result);
    console.log('Computed:', key);
    return result;
  };
}

function expensiveCalc(n) {
  let sum = 0;
  for (let i = 1; i <= n; i++) sum += i;
  return sum;
}

const memoCalc = memoize(expensiveCalc);

console.log(memoCalc(1000)); // Computed: [1000] → 500500
console.log(memoCalc(1000)); // Cache hit: [1000] → 500500
console.log(memoCalc(500));  // Computed: [500]  → 125250
console.log(memoCalc(1000)); // Cache hit: [1000] → 500500
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: Closure trong loop với `var`
> ```javascript
> const funcs = [];
> for (var i = 0; i < 3; i++) {
>   funcs.push(() => console.log(i)); // tất cả close over cùng 1 biến i
> }
> funcs[0](); // 3 (không phải 0!)
> funcs[1](); // 3
> funcs[2](); // 3
> ```
> Fix: dùng `let` (block scope mỗi iteration), hoặc IIFE để capture snapshot.

> [!warning] Pitfall 2: Memory leak nếu closure giữ reference lớn
> Closure giữ toàn bộ lexical environment — kể cả những biến bạn không dùng. Nếu closure tồn tại lâu (event listener không được remove), DOM node và data lớn bị giữ trong bộ nhớ. Giải phóng bằng cách set biến về `null` hoặc remove listener khi không dùng.

> [!tip] Mỗi lần gọi `makeCounter()` tạo closure riêng
> `const c1 = makeCounter()` và `const c2 = makeCounter()` có `count` hoàn toàn độc lập. Đây là tính chất quan trọng — closure encapsulate state per instance.

---

## 5. Phỏng vấn thường gặp

**Q1: Closure là gì? Cho ví dụ đơn giản.**

> Closure là function có khả năng truy cập biến từ outer scope ngay cả sau khi outer function đã return. Ví dụ: `function makeAdder(x) { return (y) => x + y; }` — inner arrow function "đóng gói" biến `x`. Mỗi lần gọi `makeAdder(5)` tạo một closure riêng với `x = 5`.

**Q2: Module Pattern là gì? Closure liên quan thế nào?**

> Module Pattern dùng IIFE tạo scope riêng, trả về object public API, giữ private state thông qua closure. Biến private không truy cập được từ ngoài, chỉ qua public methods. Đây là cách tạo encapsulation trong JS trước khi có ES6 class và ES modules.

**Q3: Làm sao tránh bug `var` trong loop với closure?**

> (1) Dùng `let` thay `var` — `let` tạo binding mới cho mỗi iteration, mỗi closure capture biến riêng. (2) Dùng IIFE: `(function(i) { ... })(i)` tạo scope mới capture snapshot `i` tại thời điểm đó. (3) Dùng `.forEach` với callback — mỗi callback nhận bản copy qua tham số hàm.

---

## 6. Bài tập thực hành

- [ ] **Bài 1:** Viết `makeTimer()` trả về object `{ start(), stop(), getElapsed() }`. `start()` ghi nhận thời điểm bắt đầu, `stop()` dừng, `getElapsed()` trả số giây đã trôi qua. State phải private.

- [ ] **Bài 2:** Implement `once(fn)` — wrapper nhận function `fn`, trả về function chỉ gọi `fn` đúng 1 lần, những lần sau trả về kết quả lần đầu tiên. Dùng closure để lưu trạng thái.

---

## 7. Liên kết

- [[01-Scope]] — Lexical scope là nền tảng của closure
- [[02-Hoisting]] — Closure + hoisting kết hợp tạo ra edge cases
- [[09-ES6-Arrow-Function]] — Arrow function và closure `this`
