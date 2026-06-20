---
title: "TypeScript: Class & Interface"
section: 05-TypeScript
tags: [typescript, class, interface, OOP, abstract, access-modifier, fresher, frontend]
related:
  - "[[01-TS-Overview]]"
  - "[[03-Type-Combination-Union-Intersection]]"
  - "[[07-Generics]]"
difficulty: ⭐⭐⭐
estimated_time: 45m
source: [TypeScript Handbook, typescriptlang.org/docs/handbook/2/classes.html]
---

# TypeScript: Class & Interface

> [!summary] TL;DR
> **Interface** là contract thuần type — mô tả shape của object, không có implementation. **Class** có cả type và implementation. Access modifiers: `public` (default), `private` (TS-level hoặc `#` JS-level), `protected`, `readonly`. **`implements`** = class thỏa mãn interface contract. **`extends`** = kế thừa từ class/interface khác. **`abstract`** = class không thể instantiate, có method chưa implement.

---

## 1. Khái niệm

### Interface vs Type Alias vs Class

| | Interface | Type Alias | Class |
|---|---|---|---|
| Mô tả shape | ✅ | ✅ | ✅ |
| Có implementation | ❌ | ❌ | ✅ |
| Extends/merge | ✅ Declaration merging | ❌ | ✅ (class only) |
| Union/Intersection | ❌ | ✅ | ❌ |
| Computed properties | ❌ | ✅ | ✅ |
| `implements` với Class | ✅ | ✅ (object types) | ✅ |

**Rule of thumb:**
- Dùng **interface** cho public API, shapes của objects, contract giữa components
- Dùng **type alias** cho union, tuple, complex mapped types
- Dùng **class** khi cần both type + runtime behavior (methods, constructor)

```
★ Insight ─────────────────────────────────────
• Trục phân biệt cốt lõi: interface/type CHỈ tồn tại lúc biên dịch (bị xóa, 0
  runtime), còn class tồn tại CẢ hai (vừa là type, vừa là giá trị JS có method).
  Vì vậy `instanceof` chỉ chạy với class (có thật lúc runtime), không với interface.
  Chọn class chỉ khi cần hành vi/khởi tạo thật; còn lại interface/type là đủ và nhẹ.
• extends KẾ THỪA (lấy cả type + code), implements chỉ CAM KẾT thỏa contract
  (không lấy code — phải tự viết). Và TS dùng "structural typing": hợp lệ nếu
  ĐÚNG HÌNH DẠNG, không cần khai báo implements — khác Java (nominal). private
  của TS chỉ là quy ước lúc biên dịch; muốn ẩn thật lúc chạy phải dùng `#field`.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp / API

### 2.1 Interface

```typescript
// Basic interface
interface User {
  readonly id:   number;        // không thể thay đổi sau khi tạo
  name:          string;
  email:         string;
  age?:          number;        // optional
  role:          'admin' | 'user' | 'guest';
}

// Method signatures trong interface
interface Repository<T> {
  findById(id: number): Promise<T | null>;
  findAll(): Promise<T[]>;
  save(entity: T): Promise<T>;
  delete(id: number): Promise<void>;
}

// Interface extends interface (không giới hạn số lượng)
interface BaseEntity {
  id:        number;
  createdAt: Date;
  updatedAt: Date;
}

interface Product extends BaseEntity {
  name:     string;
  price:    number;
  inStock:  boolean;
}

// Declaration merging — chỉ interface mới làm được (type alias không thể)
interface Window {
  myCustomProp: string; // extend global Window interface
}

// Index signatures — dynamic keys
interface StringMap {
  [key: string]: string;
}
const headers: StringMap = { 'Content-Type': 'application/json' };
```

### 2.2 Class với TypeScript

```typescript
class Animal {
  // Property declarations (bắt buộc với strictPropertyInitialization)
  name:  string;
  sound: string;
  alive: boolean = true;

  constructor(name: string, sound: string) {
    this.name  = name;
    this.sound = sound;
  }

  speak(): string {
    return `${this.name} says: ${this.sound}!`;
  }

  get info(): string {
    return `${this.name} (${this.alive ? 'alive' : 'dead'})`;
  }

  set status(val: 'alive' | 'dead') {
    this.alive = val === 'alive';
  }
}

class Dog extends Animal {
  breed: string;

  constructor(name: string, breed: string) {
    super(name, 'Gâu');
    this.breed = breed;
  }

  override speak(): string {  // 'override' keyword (TS 4.3+)
    return `[Dog] ${super.speak()} (${this.breed})`;
  }
}

const dog = new Dog('Rex', 'Labrador');
console.log(dog.speak()); // "[Dog] Rex says: Gâu! (Labrador)"
```

### 2.3 Access Modifiers

```typescript
class BankAccount {
  // public (default) — accessible từ bất kỳ đâu
  public owner: string;

  // private — chỉ accessible trong class này (TypeScript-level)
  private balance: number;

  // readonly — gán 1 lần (trong declaration hoặc constructor)
  readonly accountNumber: string;

  // protected — accessible trong class này và subclasses
  protected transactionHistory: number[] = [];

  constructor(owner: string, initialBalance: number, accountNumber: string) {
    this.owner          = owner;
    this.balance        = initialBalance;
    this.accountNumber  = accountNumber;
  }

  deposit(amount: number): this {
    if (amount <= 0) throw new Error('Amount phải > 0');
    this.balance += amount;
    this.transactionHistory.push(amount);
    return this; // chainable
  }

  withdraw(amount: number): this {
    if (amount > this.balance) throw new Error('Insufficient funds');
    this.balance -= amount;
    this.transactionHistory.push(-amount);
    return this;
  }

  getBalance(): number {
    return this.balance;
  }
}

// Constructor shorthand — rất phổ biến trong TS
class Point {
  constructor(
    public readonly x: number,
    public readonly y: number,
    private label: string = 'Point',
  ) {}

  toString(): string {
    return `${this.label}(${this.x}, ${this.y})`;
  }

  distanceTo(other: Point): number {
    return Math.sqrt((this.x - other.x) ** 2 + (this.y - other.y) ** 2);
  }
}

const p1 = new Point(0, 0);
const p2 = new Point(3, 4, 'P2');
console.log(p1.distanceTo(p2)); // 5
// p1.x = 10; // Error: readonly
```

### 2.4 `implements` — Class thỏa mãn Interface

```typescript
interface Serializable {
  serialize(): string;
}

interface Validatable {
  validate(): boolean;
}

// Class có thể implement nhiều interfaces
class UserEntity implements Serializable, Validatable {
  constructor(
    public id:    number,
    public name:  string,
    public email: string,
  ) {}

  serialize(): string {
    return JSON.stringify({ id: this.id, name: this.name, email: this.email });
  }

  validate(): boolean {
    return (
      this.name.trim().length > 0 &&
      /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(this.email)
    );
  }
}

// Generic Repository interface + implementation
interface IRepository<T extends { id: number }> {
  findById(id: number): T | undefined;
  findAll(): T[];
  save(entity: T): T;
  delete(id: number): boolean;
}

class InMemoryRepository<T extends { id: number }> implements IRepository<T> {
  private store: Map<number, T> = new Map();

  findById(id: number): T | undefined   { return this.store.get(id); }
  findAll(): T[]                        { return Array.from(this.store.values()); }
  save(entity: T): T                    { this.store.set(entity.id, entity); return entity; }
  delete(id: number): boolean           { return this.store.delete(id); }
}

const repo = new InMemoryRepository<UserEntity>();
repo.save(new UserEntity(1, 'Alice', 'alice@example.com'));
console.log(repo.findById(1)?.name); // "Alice"
```

### 2.5 Abstract Class

```typescript
// Abstract class — không thể instantiate trực tiếp
// Cung cấp base implementation + abstract methods (subclass phải implement)
abstract class Shape {
  abstract readonly name: string;

  // Abstract method — subclass PHẢI override
  abstract getArea(): number;
  abstract getPerimeter(): number;

  // Concrete method — shared giữa tất cả subclass
  describe(): string {
    return `${this.name}: area=${this.getArea().toFixed(2)}, perimeter=${this.getPerimeter().toFixed(2)}`;
  }

  static compare(a: Shape, b: Shape): Shape {
    return a.getArea() >= b.getArea() ? a : b;
  }
}

class Circle extends Shape {
  readonly name = 'Circle';

  constructor(private radius: number) { super(); }

  getArea():      number { return Math.PI * this.radius ** 2; }
  getPerimeter(): number { return 2 * Math.PI * this.radius; }
}

class Rectangle extends Shape {
  readonly name = 'Rectangle';

  constructor(private width: number, private height: number) { super(); }

  getArea():      number { return this.width * this.height; }
  getPerimeter(): number { return 2 * (this.width + this.height); }
}

const circle = new Circle(5);
const rect   = new Rectangle(4, 6);

console.log(circle.describe()); // "Circle: area=78.54, perimeter=31.42"
console.log(rect.describe());   // "Rectangle: area=24.00, perimeter=20.00"
console.log(Shape.compare(circle, rect).name); // "Circle" (area lớn hơn)

// new Shape(); // Error: Cannot create an instance of an abstract class
```

---

## 3. Ví dụ minh họa

### Ví dụ 1: Repository Pattern với interface + class

```typescript
interface IUserRepository {
  findById(id: number): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  create(data: Omit<User, 'id' | 'createdAt'>): Promise<User>;
  update(id: number, data: Partial<Omit<User, 'id'>>): Promise<User | null>;
  delete(id: number): Promise<boolean>;
}

interface User {
  id:        number;
  name:      string;
  email:     string;
  role:      'admin' | 'user';
  createdAt: Date;
}

class MockUserRepository implements IUserRepository {
  private users: User[] = [];
  private nextId = 1;

  async findById(id: number) {
    return this.users.find(u => u.id === id) ?? null;
  }

  async findByEmail(email: string) {
    return this.users.find(u => u.email === email) ?? null;
  }

  async create(data: Omit<User, 'id' | 'createdAt'>): Promise<User> {
    const user: User = { ...data, id: this.nextId++, createdAt: new Date() };
    this.users.push(user);
    return user;
  }

  async update(id: number, data: Partial<Omit<User, 'id'>>): Promise<User | null> {
    const idx = this.users.findIndex(u => u.id === id);
    if (idx === -1) return null;
    this.users[idx] = { ...this.users[idx], ...data };
    return this.users[idx];
  }

  async delete(id: number): Promise<boolean> {
    const before = this.users.length;
    this.users = this.users.filter(u => u.id !== id);
    return this.users.length < before;
  }
}
```

### Ví dụ 2: Abstract EventEmitter base class

```typescript
abstract class BaseService {
  protected readonly name: string;
  private isStarted = false;

  constructor(name: string) {
    this.name = name;
  }

  // Template method pattern — subclass override hooks
  protected abstract onStart(): Promise<void>;
  protected abstract onStop(): Promise<void>;
  protected abstract onHealthCheck(): boolean;

  async start(): Promise<void> {
    if (this.isStarted) return;
    console.log(`[${this.name}] Starting...`);
    await this.onStart();
    this.isStarted = true;
    console.log(`[${this.name}] Started`);
  }

  async stop(): Promise<void> {
    if (!this.isStarted) return;
    console.log(`[${this.name}] Stopping...`);
    await this.onStop();
    this.isStarted = false;
  }

  healthCheck(): { service: string; healthy: boolean } {
    return { service: this.name, healthy: this.onHealthCheck() };
  }
}

class DatabaseService extends BaseService {
  private connection: null | object = null;

  constructor() { super('DatabaseService'); }

  protected async onStart(): Promise<void> {
    this.connection = {}; // giả lập connect
    console.log('DB connected');
  }

  protected async onStop(): Promise<void> {
    this.connection = null;
    console.log('DB disconnected');
  }

  protected onHealthCheck(): boolean {
    return this.connection !== null;
  }
}
```

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Pitfall 1: TypeScript `private` vs JavaScript `#private`
> `private` trong TS chỉ là type-level — bị xóa sau compile, vẫn accessible ở runtime JS. Để truly private, dùng `#field` (JS private fields, ES2022). Khi build library hoặc cần real encapsulation, prefer `#`.

> [!warning] Pitfall 2: `implements` không tự kế thừa implementation
> `class A implements IB {}` — A **thỏa mãn contract của IB** nhưng không có implementation từ IB. Muốn kế thừa implementation, dùng `extends`. `implements` chỉ check type compatibility.

> [!tip] Interface cho public API — type alias cho internal/complex
> Convention phổ biến: dùng `interface` cho props, function contracts (dễ extend/merge); `type` cho union types, mapped types, complex transformations. Không có quy tắc tuyệt đối — nhất quán trong codebase là quan trọng nhất.

---

## 5. Câu hỏi phỏng vấn thường gặp

**Q1: Interface và Type Alias khác nhau thế nào?**

> Về mặt type-checking, hầu hết tương đương. Khác biệt chính: **Interface** hỗ trợ **declaration merging** (khai báo 2 lần → merge thành 1), hỗ trợ `extends` với inheritance, không thể dùng cho union/intersection trực tiếp. **Type alias** hỗ trợ union `|`, intersection `&`, mapped types, template literal types; không có declaration merging. Rule of thumb: interface cho shapes của objects, type alias cho complex type operations.

**Q2: `implements` và `extends` khác nhau?**

> **`extends`**: kế thừa từ class/interface khác — nhận cả type VÀ implementation (với class). **`implements`**: một class cam kết thỏa mãn contract của interface — chỉ check type compatibility, không kế thừa implementation. Class có thể `implements` nhiều interfaces nhưng chỉ `extends` 1 class.

**Q3: Access modifiers trong TypeScript?**

> **`public`** (default): accessible từ bất kỳ đâu. **`private`**: chỉ trong class (TypeScript-level, bị xóa sau compile — dùng `#` cho real runtime privacy). **`protected`**: trong class và subclasses. **`readonly`**: chỉ gán trong declaration hoặc constructor. Constructor shorthand: `constructor(private name: string)` tự động tạo và gán private field.

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Tạo interface `ILogger` với methods `info`, `warn`, `error` (nhận string, return void). Implement `ConsoleLogger` và `SilentLogger` (không log gì). Viết function nhận `ILogger` như dependency injection.

- [ ] **Bài 2:** Tạo abstract class `Animal` có abstract `speak(): string`. Implement `Dog`, `Cat`, `Bird` override `speak()`. Viết function `makeThemSpeak(animals: Animal[]): string[]` dùng polymorphism.

---

## 7. Liên kết

- [[03-Type-Combination-Union-Intersection]] — Interface vs type alias, intersection types
- [[04-Type-Guard]] — `instanceof` guard với class hierarchy
- [[07-Generics]] — Generic class và interface `<T>`
- [[../03-Advanced-JavaScript/11-ES6-Class|ES6 Class]] — Class JavaScript không có type annotations
