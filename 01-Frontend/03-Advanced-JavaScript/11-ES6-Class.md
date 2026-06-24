---
title: "ES6: Class"
section: 03-Advanced-JavaScript
tags: [es6, class, OOP, extends, fresher, frontend]
related:
  - "[[09-ES6-Arrow-Function]]"
  - "[[03-Closure]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [MDN]
---

# ES6: Class

> [!summary] TL;DR
> ES6 Class là **syntactic sugar** (lớp áo cú pháp cho dễ đọc) đặt lên cơ chế **kế thừa qua prototype** sẵn có của JS — *không* phải OOP class thuần như Java/C#. `constructor()` = hàm chạy khi tạo đối tượng mới (instance) bằng `new`. `extends` = kế thừa (lớp con dùng lại lớp cha), `super()` = gọi constructor của lớp cha. `static` method = thuộc về *class* chứ không thuộc *instance* (gọi `ClassName.method()`). `#privateField` (từ 2022) = thuộc tính *thật sự riêng tư*, bên ngoài không truy cập được.

> [!tip] 🎯 Hiểu trong 30 giây
> **Class = một "khuôn bánh", còn object tạo ra (`new`) là "cái bánh".** Khuôn định nghĩa cái bánh sẽ có gì (`constructor` đổ nguyên liệu) và làm được gì (`method`); mỗi lần `new` là đúc ra một cái bánh riêng với dữ liệu riêng.
>
> Từ khóa cần nhớ:
> - `constructor` → hàm chạy khi `new`, để gán dữ liệu ban đầu.
> - `extends` + `super` → "kế thừa": lớp con dùng lại lớp cha; `super()` gọi constructor cha.
> - `static` → thuộc về *khuôn*, không thuộc *cái bánh* — gọi `ClassName.method()` (vd `Math.random()`), dùng cho factory/tiện ích.
> - `#field` → biến *thật sự* riêng tư, bên ngoài đụng vào là lỗi.
>
> **Câu hay bị hỏi:** "Class trong JS có phải OOP thật không?" → *Không hẳn*. Nó chỉ là **lớp áo cú pháp dễ đọc** đặt lên cơ chế **prototype** có sẵn của JS (`typeof Class === 'function'`). Bên dưới vẫn là prototype chain.

---

## 1. Khái niệm

### Class là syntactic sugar

```javascript
// Trước ES6 — prototype-based
function PersonOld(name, age) {
  this.name = name;
  this.age  = age;
}
PersonOld.prototype.greet = function() {
  return `Xin chào, tôi là ${this.name}`;
};

// ES6 class — cú pháp rõ ràng hơn, semantics giống nhau
class Person {
  constructor(name, age) {
    this.name = name;
    this.age  = age;
  }
  greet() {
    return `Xin chào, tôi là ${this.name}`;
  }
}

// Kết quả giống nhau:
const p1 = new PersonOld('Alice', 25);
const p2 = new Person('Bob', 30);
console.log(typeof Person); // "function" — class là function!
```

### Quan trọng: Class NOT hoisted như function declaration

Class ở trong TDZ — không thể dùng trước khi khai báo.

```
★ Insight ─────────────────────────────────────
• "Sugar trên prototype" không phải chi tiết vụn vặt: `typeof Class === 'function'`,
  method nằm trên prototype (dùng chung mọi instance, tiết kiệm bộ nhớ), extends
  chỉ là nối prototype chain. Hiểu điều này thì instanceof, super, và "vì sao
  method lại shared" trở nên hiển nhiên — class chỉ là lớp áo dễ đọc.
• Bài toán "mất this" lặp lại từ [[09-ES6-Arrow-Function]]: tách method ra
  (`const {greet}=obj`) hay truyền làm callback → this rơi về undefined. 3 lối
  thoát: bind trong constructor, arrow class field (this đóng băng = instance),
  hoặc bind tại điểm dùng. React class component sống chết với chuyện này.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp

### 2.1 Class cơ bản

```javascript
class Animal {
  // Constructor — chạy khi new Animal(...)
  constructor(name, sound) {
    this.name  = name;
    this.sound = sound;
    this.alive = true;
  }

  // Instance method
  speak() {
    return `${this.name} nói: ${this.sound}!`;
  }

  isAlive() {
    return this.alive;
  }

  // Getter
  get info() {
    return `${this.name} (${this.alive ? 'alive' : 'dead'})`;
  }

  // Setter
  set status(val) {
    this.alive = val === 'alive';
  }

  // Static method — gọi qua Class, không qua instance
  static create(name, sound) {
    return new Animal(name, sound);
  }

  // toString
  toString() {
    return `Animal(${this.name})`;
  }
}

const dog = new Animal('Rex', 'Gâu');
console.log(dog.speak());       // "Rex nói: Gâu!"
console.log(dog.info);          // "Rex (alive)"
dog.status = 'dead';
console.log(dog.info);          // "Rex (dead)"

const cat = Animal.create('Mimi', 'Meo');
console.log(cat.speak());       // "Mimi nói: Meo!"
```

### 2.2 Kế thừa với `extends` và `super`

```javascript
class Dog extends Animal {
  constructor(name, breed) {
    super(name, 'Gâu');  // Gọi constructor của Animal
    this.breed = breed;
  }

  // Override method
  speak() {
    const parentSpeech = super.speak(); // gọi method cha
    return `${parentSpeech} (${this.breed})`;
  }

  // Method riêng của Dog
  fetch(item) {
    return `${this.name} đã lấy ${item}!`;
  }
}

class GoldenRetriever extends Dog {
  constructor(name) {
    super(name, 'Golden Retriever');
    this.temperament = 'friendly';
  }

  speak() {
    return `[Golden] ${super.speak()}`;
  }
}

const rex     = new Dog('Rex', 'Labrador');
const goldie  = new GoldenRetriever('Goldie');

console.log(rex.speak());    // "Rex nói: Gâu! (Labrador)"
console.log(goldie.speak()); // "[Golden] Goldie nói: Gâu! (Golden Retriever)"
console.log(goldie.fetch('bóng')); // "Goldie đã lấy bóng!"

// instanceof chain
console.log(goldie instanceof GoldenRetriever); // true
console.log(goldie instanceof Dog);             // true
console.log(goldie instanceof Animal);          // true
```

### 2.3 Private Fields (ES2022)

```javascript
class BankAccount {
  #balance;       // private field
  #owner;

  constructor(owner, initialBalance = 0) {
    this.#owner   = owner;
    this.#balance = initialBalance;
  }

  deposit(amount) {
    if (amount <= 0) throw new Error('Số tiền phải > 0');
    this.#balance += amount;
    return this;
  }

  withdraw(amount) {
    if (amount > this.#balance) throw new Error('Số dư không đủ');
    this.#balance -= amount;
    return this;
  }

  get balance() { return this.#balance; }
  get owner()   { return this.#owner; }

  toString() {
    return `[${this.#owner}] Số dư: ${this.#balance.toLocaleString('vi-VN')}₫`;
  }
}

const acc = new BankAccount('Alice', 1000000);
acc.deposit(500000).deposit(200000).withdraw(300000);
console.log(acc.balance); // 1400000
console.log(acc.toString()); // "[Alice] Số dư: 1.400.000₫"
// console.log(acc.#balance); // SyntaxError — private!
```

### 2.4 Static members

```javascript
class MathUtils {
  static PI = 3.14159265358979;

  static circleArea(r)   { return MathUtils.PI * r * r; }
  static circumference(r){ return 2 * MathUtils.PI * r; }
  static clamp(val, min, max) { return Math.min(Math.max(val, min), max); }
  static randomInt(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
  }
}

console.log(MathUtils.circleArea(5));         // 78.539...
console.log(MathUtils.clamp(150, 0, 100));    // 100
console.log(MathUtils.randomInt(1, 10));      // random 1-10

// Không cần new MathUtils() — static dùng qua Class
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: EventEmitter pattern

```javascript
class EventEmitter {
  #events = {};

  on(event, listener) {
    if (!this.#events[event]) this.#events[event] = [];
    this.#events[event].push(listener);
    return this; // chainable
  }

  off(event, listener) {
    if (!this.#events[event]) return this;
    this.#events[event] = this.#events[event].filter(l => l !== listener);
    return this;
  }

  emit(event, ...args) {
    (this.#events[event] ?? []).forEach(listener => listener(...args));
    return this;
  }

  once(event, listener) {
    const wrapper = (...args) => {
      listener(...args);
      this.off(event, wrapper);
    };
    return this.on(event, wrapper);
  }
}

class Store extends EventEmitter {
  #state;

  constructor(initialState) {
    super();
    this.#state = initialState;
  }

  get state() { return { ...this.#state }; }

  setState(updates) {
    const prevState = { ...this.#state };
    this.#state     = { ...this.#state, ...updates };
    this.emit('change', this.#state, prevState);
  }
}

const store = new Store({ count: 0, user: null });

store.on('change', (newState, prevState) => {
  console.log('State changed:', prevState, '→', newState);
});

store.setState({ count: 1 });
// "State changed: {count: 0, user: null} → {count: 1, user: null}"
store.setState({ user: { name: 'Alice' } });
```

### Ví dụ 2: Component base class

```javascript
class Component {
  constructor(props = {}) {
    this.props = props;
    this.state = {};
    this.el    = null;
  }

  setState(updates) {
    this.state = { ...this.state, ...updates };
    this.render();
  }

  // Template method — subclass override
  template() { return ''; }

  render() {
    if (!this.el) return;
    const content       = document.createElement('div');
    content.textContent = this.template();
    this.el.replaceChildren(content);
  }

  mount(selector) {
    this.el = document.querySelector(selector);
    this.render();
    return this;
  }
}

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = { count: this.props.initial ?? 0 };
  }

  template() {
    return `Đếm: ${this.state.count}`;
  }

  increment() { this.setState({ count: this.state.count + 1 }); }
  decrement() { this.setState({ count: this.state.count - 1 }); }
}

const counter = new Counter({ initial: 5 });
// counter.mount('#app');
// counter.increment(); → cập nhật DOM
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: Quên gọi `super()` trước khi dùng `this` trong constructor con
> Trong class kế thừa (`extends`), constructor PHẢI gọi `super()` trước khi truy cập `this`. Nếu không → `ReferenceError: Must call super constructor`. Nếu không cần logic init riêng, JS tự inject `super(...args)` cho bạn.

> [!warning] Pitfall 2: Method không bound — mất `this` khi detach
> `const { greet } = person; greet()` — `this` = `undefined`. Fix: (1) bind trong constructor `this.greet = this.greet.bind(this)`, (2) dùng arrow class field `greet = () => {}`, (3) bind tại điểm dùng `greet.bind(person)`.

> [!tip] Class không bắt buộc trong JS
> JS là multi-paradigm. Class phù hợp khi cần nhiều instances với shared behavior, hoặc khi team quen OOP. Closure/factory functions và module pattern đôi khi đơn giản hơn. Đừng ép OOP vào mọi vấn đề.

---

## 5. Phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "ES6 Class có phải OOP thật không?"
> *"Class trong JavaScript chủ yếu là syntactic sugar — lớp áo cú pháp đặt lên cơ chế kế thừa qua prototype có sẵn của JS, chứ không phải class-based OOP như Java hay C#. Bằng chứng là `typeof MyClass` ra `'function'`, và các method thực ra nằm trên prototype dùng chung cho mọi instance. `extends` chỉ là nối prototype chain, `super` gọi lên cha. Class giúp code OOP dễ đọc hơn nhiều so với viết prototype thủ công, và hành vi giống OOP truyền thống ở hầu hết trường hợp, nên em vẫn dùng class khi cần nhiều instance có hành vi chung; còn việc đơn giản thì factory function với closure đôi khi gọn hơn."*

> [!note] 🧠 Mẹo nhớ
> **Class = khuôn bánh, `new` = đúc cái bánh.** Class JS = **sugar trên prototype** (`typeof` ra `'function'`). `static` thuộc khuôn, `#` là private thật, lớp con phải gọi `super()` trước khi dùng `this`.

**Q1: ES6 Class có phải OOP thực sự không?**

> ES6 Class là syntactic sugar trên prototype-based inheritance — bên dưới vẫn là prototype chain, không phải class-based OOP như Java. `typeof MyClass === 'function'`. Tuy nhiên, cú pháp class giúp code OOP dễ đọc hơn và hành vi giống OOP truyền thống ở hầu hết use cases.

**Q2: `extends` và `super` hoạt động thế nào?**

> `extends` thiết lập prototype chain: instance của subclass có prototype = instance của superclass. `super()` trong constructor gọi constructor cha để khởi tạo `this`. `super.method()` gọi method của cha. Nếu subclass có constructor, PHẢI gọi `super()` trước khi dùng `this`.

**Q3: Static method là gì? Dùng khi nào?**

> Static method thuộc **class**, không thuộc instance — gọi qua `ClassName.method()`. Không có `this` của instance. Dùng cho: factory methods (`User.create()`), utility functions liên quan đến class (`MathUtils.clamp()`), constants (`Config.MAX_RETRIES`).

---

## 6. Bài tập thực hành

- [ ] **Bài 1:** Tạo class `Stack` với methods `push(val)`, `pop()`, `peek()`, `isEmpty()`, `size`. Dùng private field `#items = []`. Add static method `from(array)` tạo Stack từ array.

- [ ] **Bài 2:** Implement inheritance: `Shape` (base, có `area()` và `perimeter()`), `Circle` (bán kính), `Rectangle` (chiều rộng, cao), `Square extends Rectangle`. Mỗi subclass override `area()` và `perimeter()`. Thêm static `compare(s1, s2)` trả về shape có area lớn hơn.

---

## 7. Liên kết

- [[09-ES6-Arrow-Function]] — Arrow class fields cho event handlers
- [[03-Closure]] — So sánh class pattern vs closure/module pattern
- [[07-ES6-Enhanced-Object-Literals]] — Method shorthand tương tự trong object
