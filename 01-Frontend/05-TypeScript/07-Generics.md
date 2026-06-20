---
title: "TypeScript Generics"
section: 05-TypeScript
tags: [typescript, generics, T, constraint, extends, infer, fresher, frontend]
related:
  - "[[02-Basic-Types]]"
  - "[[05-TS-Functions]]"
  - "[[06-Classes-va-Interface]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 45m
source: [TypeScript Handbook, typescriptlang.org/docs/handbook/2/generics.html]
---

# TypeScript Generics

> [!summary] TL;DR
> **Generics** (`<T>`) là "placeholder type" được điền khi sử dụng — giúp viết code reusable mà vẫn type-safe. **Constraint** `T extends SomeType` giới hạn T phải là subtype của SomeType. **Default type** `<T = string>` dùng khi không truyền type arg. **Conditional types** `T extends U ? X : Y` cho logic type-level. **`infer`** extract type từ conditional. Utility types built-in (`Partial`, `Pick`, `ReturnType`) đều viết bằng generics + mapped types.

---

## 1. Khái niệm

### Tại sao cần Generics?

```typescript
// Không có generics — phải viết lại cho mỗi type
function getFirstNumber(arr: number[]): number { return arr[0]; }
function getFirstString(arr: string[]): string { return arr[0]; }

// Với generics — 1 function, type-safe với mọi loại
function getFirst<T>(arr: T[]): T {
  return arr[0];
}

const firstNum = getFirst([1, 2, 3]);     // type: number — inferred!
const firstStr = getFirst(['a', 'b']);    // type: string — inferred!
const firstObj = getFirst([{id: 1}]);     // type: {id: number}

// Hoặc explicit type argument
const explicit = getFirst<boolean>([true, false]); // type: boolean
```

```
★ Insight ─────────────────────────────────────
• Bản chất generic là GIỮ MỐI LIÊN HỆ giữa input và output, thứ mà `any` đánh
  mất. `getFirst<T>(arr: T[]): T` nói "trả về CÙNG kiểu phần tử mảng" → gọi với
  number[] ra number. Dùng any thì ra any (mất hết). Nếu một type param chỉ xuất
  hiện 1 chỗ, không nối input với output → có lẽ bạn không cần generic.
• `extends` trong generic = RÀNG BUỘC (khác extends kế thừa của class): `T extends
  {id:number}` nghĩa "T phải có id". Ràng buộc làm generic dùng được (chạm
  property an toàn). `infer` thì NGƯỢC lại — RÚT type ra từ một khuôn
  (`T extends Promise<infer U> ? U : T`). Utility types (ReturnType, Awaited…)
  chính là generic + conditional + infer.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp / API

### 2.1 Generic Functions

```typescript
// Một type param
function identity<T>(value: T): T {
  return value;
}

// Multiple type params
function pair<A, B>(first: A, second: B): [A, B] {
  return [first, second];
}
const p = pair('hello', 42); // type: [string, number]

// Generic với array methods
function last<T>(arr: T[]): T | undefined {
  return arr[arr.length - 1];
}

function take<T>(arr: T[], count: number): T[] {
  return arr.slice(0, count);
}

// Inferred:
const nums = [1, 2, 3, 4, 5];
const lastNum = last(nums);        // number | undefined
const firstTwo = take(nums, 2);    // number[]

// Type params trong arrow functions (JSX files cần trailing comma: <T,>)
const map = <T, U>(arr: T[], fn: (item: T, i: number) => U): U[] =>
  arr.map(fn);
```

### 2.2 Generic Interfaces và Types

```typescript
// Generic interface
interface Container<T> {
  value:     T;
  transform: <U>(fn: (val: T) => U) => Container<U>;
}

// Generic type alias
type Nullable<T>   = T | null;
type Optional<T>   = T | null | undefined;
type Awaited_<T>   = T extends Promise<infer U> ? U : T;
type ArrayElement<T> = T extends (infer E)[] ? E : never;

// Usage:
type UserId   = Nullable<number>;     // number | null
type MaybeStr = Optional<string>;     // string | null | undefined

// Generic type with constraints
type StringKeys<T> = {
  [K in keyof T]: T[K] extends string ? K : never;
}[keyof T];

interface User { id: number; name: string; email: string; age: number }
type UserStringKeys = StringKeys<User>; // 'name' | 'email'
```

### 2.3 Constraints (`extends`)

```typescript
// Constraint: T phải có property 'id' dạng number
function findById<T extends { id: number }>(items: T[], id: number): T | undefined {
  return items.find(item => item.id === id);
}

const users = [{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }];
const alice = findById(users, 1); // type: { id: number; name: string } | undefined

// keyof constraint — type-safe property access
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

interface Config { host: string; port: number; ssl: boolean }
const cfg: Config = { host: 'localhost', port: 3000, ssl: false };

const host = getProperty(cfg, 'host');  // type: string — safe!
const port = getProperty(cfg, 'port');  // type: number
// getProperty(cfg, 'timeout');          // Error: not in Config

// Multiple constraints
function merge<T extends object, U extends object>(target: T, source: U): T & U {
  return { ...target, ...source };
}

const merged = merge({ id: 1 }, { name: 'Alice' });
// type: { id: number } & { name: string }
```

### 2.4 Generic Class

```typescript
class Stack<T> {
  private items: T[] = [];

  push(item: T): void         { this.items.push(item); }
  pop(): T | undefined        { return this.items.pop(); }
  peek(): T | undefined       { return this.items[this.items.length - 1]; }
  isEmpty(): boolean          { return this.items.length === 0; }
  get size(): number          { return this.items.length; }
  toArray(): T[]              { return [...this.items]; }

  static from<T>(items: T[]): Stack<T> {
    const stack = new Stack<T>();
    items.forEach(item => stack.push(item));
    return stack;
  }
}

const numStack = new Stack<number>();
numStack.push(1); numStack.push(2); numStack.push(3);
console.log(numStack.pop());   // 3
console.log(numStack.size);    // 2

const strStack = Stack.from(['a', 'b', 'c']);
console.log(strStack.peek()); // 'c'

// Generic class với constraint
class Repository<T extends { id: number }> {
  private store: Map<number, T> = new Map();

  save(entity: T): T {
    this.store.set(entity.id, entity);
    return entity;
  }

  findById(id: number): T | undefined { return this.store.get(id); }
  findAll(): T[]  { return Array.from(this.store.values()); }
  count(): number { return this.store.size; }
}
```

### 2.5 Default Type Parameters

```typescript
// Default type — dùng khi không truyền type arg
interface ApiResponse<T = unknown> {
  data:    T;
  status:  number;
  message: string;
}

// Không có default:
type Response1 = ApiResponse;          // Error (nếu không có default)

// Có default:
type Response2 = ApiResponse;          // data: unknown
type Response3 = ApiResponse<string>;  // data: string
type Response4 = ApiResponse<User[]>;  // data: User[]

// Function với default
function createArray<T = string>(length: number, fill: T): T[] {
  return Array(length).fill(fill);
}

const strs = createArray(3, 'x');    // T inferred as string
const nums = createArray(3, 0);      // T inferred as number
const bools = createArray<boolean>(3, true); // explicit
```

### 2.6 Conditional Types và `infer`

```typescript
// Conditional types: T extends U ? X : Y
type IsString<T> = T extends string ? true : false;
type IsArray<T>  = T extends any[]  ? true : false;

type CheckStr  = IsString<string>;  // true
type CheckNum  = IsString<number>;  // false
type CheckArr  = IsArray<number[]>; // true

// infer — extract type từ pattern
type UnpackPromise<T> = T extends Promise<infer U> ? U : T;
type UnpackArray<T>   = T extends (infer E)[]      ? E : never;
type ReturnType_<T>   = T extends (...args: any[]) => infer R ? R : never;
type Parameters_<T>   = T extends (...args: infer P) => any   ? P : never;

type AsyncUserData = UnpackPromise<Promise<User>>;  // User
type UserArray     = UnpackArray<User[]>;            // User
type UnwrapArr     = UnpackArray<string[]>;          // string
type NotArray      = UnpackArray<string>;            // never

// Distributive conditional types (phân phối qua union)
type ToArray<T> = T extends any ? T[] : never;
type StringOrNum = ToArray<string | number>; // string[] | number[]

// Non-distributive: bọc T trong []
type ToArrayND<T> = [T] extends [any] ? T[] : never;
type Combined = ToArrayND<string | number>; // (string | number)[]
```

---

## 3. Ví dụ minh họa

### Ví dụ 1: Generic Result type (Railway-oriented programming)

```typescript
type Ok<T>  = { ok: true;  value: T }
type Err<E> = { ok: false; error: E }
type Result<T, E = Error> = Ok<T> | Err<E>;

function ok<T>(value: T):   Ok<T>  { return { ok: true, value }; }
function err<E>(error: E):  Err<E> { return { ok: false, error }; }

function divide(a: number, b: number): Result<number, string> {
  if (b === 0) return err('Division by zero');
  return ok(a / b);
}

function sqrt(n: number): Result<number, string> {
  if (n < 0) return err('Cannot sqrt negative number');
  return ok(Math.sqrt(n));
}

// Chain results
function divideAndSqrt(a: number, b: number): Result<number, string> {
  const divResult = divide(a, b);
  if (!divResult.ok) return divResult;         // propagate error

  const sqrtResult = sqrt(divResult.value);
  if (!sqrtResult.ok) return sqrtResult;

  return ok(sqrtResult.value);
}

const result = divideAndSqrt(16, 4);
if (result.ok) {
  console.log('Result:', result.value); // 2
} else {
  console.error('Error:', result.error);
}
```

### Ví dụ 2: Generic cache với TTL

```typescript
interface CacheEntry<T> {
  value:     T;
  expiresAt: number;
}

class TimedCache<K, V> {
  private store: Map<K, CacheEntry<V>> = new Map();

  constructor(private defaultTTL: number = 60_000) {}

  set(key: K, value: V, ttl: number = this.defaultTTL): void {
    this.store.set(key, {
      value,
      expiresAt: Date.now() + ttl,
    });
  }

  get(key: K): V | undefined {
    const entry = this.store.get(key);
    if (!entry) return undefined;
    if (Date.now() > entry.expiresAt) {
      this.store.delete(key);
      return undefined;
    }
    return entry.value;
  }

  has(key: K): boolean  { return this.get(key) !== undefined; }
  delete(key: K): void  { this.store.delete(key); }
  clear(): void         { this.store.clear(); }
  get size(): number    { return this.store.size; }
}

const cache = new TimedCache<string, object>(30_000); // 30s TTL
cache.set('user:1', { id: 1, name: 'Alice' });
cache.set('session', { token: 'abc' }, 5_000); // 5s TTL

console.log(cache.get('user:1')); // { id: 1, name: 'Alice' }
```

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Pitfall 1: `<T>` trong JSX/TSX file bị hiểu là JSX tag
> Trong `.tsx` files, `const fn = <T>(x: T) => x` → TS nhầm `<T>` là JSX. Fix: thêm trailing comma `<T,>` hoặc constraint `<T extends unknown>`.

> [!warning] Pitfall 2: Quá generic — mất đi lợi ích của type system
> `function fn<T>(x: T): T` — nếu không có constraint nào, T có thể là bất cứ thứ gì. Nếu chỉ dùng 1 type, viết type cụ thể còn tốt hơn generic. Generic hữu ích khi có **relationship** giữa input và output types.

> [!tip] Type inference với generics — không cần explicit type arg
> TS infer type args từ usage: `identity(42)` → T inferred as `number`. Chỉ cần explicit `identity<number>(42)` khi inference không chính xác hoặc khi muốn constrain hơn inference.

---

## 5. Câu hỏi phỏng vấn thường gặp

**Q1: Generics là gì? Tại sao cần?**

> Generics là **type parameter** — placeholder type điền vào lúc sử dụng. Cho phép viết code **reusable** mà vẫn **type-safe**. Không có generics → phải viết lại function/class cho mỗi type, hoặc dùng `any` (mất type safety). Với `<T>`, TypeScript infer hoặc accept explicit type argument để cung cấp exact typing.

**Q2: Constraint `T extends SomeType` dùng khi nào?**

> Khi function/class cần **guarantee** rằng T có certain properties hoặc methods. Ví dụ: `T extends { id: number }` — đảm bảo T có field `id`; `T extends keyof U` — T phải là valid key của U. Constraint làm generic hữu ích hơn: (1) tránh `any` để access props; (2) TS infer đúng type; (3) IDE autocomplete chính xác.

**Q3: `infer` keyword trong conditional types dùng để làm gì?**

> `infer U` trong `T extends Promise<infer U>` cho phép **extract** (capture) type từ một vị trí trong pattern. TS sẽ điền `U` = type thực tế ở vị trí đó nếu T match pattern. Dùng để implement utility types: `ReturnType<F>` extract return type, `Parameters<F>` extract param types, `Awaited<T>` unwrap Promise. Chỉ dùng được trong `extends` clause của conditional type.

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Viết generic `Queue<T>` với methods `enqueue(item: T): void`, `dequeue(): T | undefined`, `peek(): T | undefined`, `isEmpty(): boolean`, `size: number`. Thêm static `from<T>(items: T[]): Queue<T>`.

- [ ] **Bài 2:** Viết utility type `DeepPartial<T>` làm tất cả properties của T (kể cả nested objects) trở thành optional. Test với `DeepPartial<{ user: { name: string; address: { city: string } } }>`.

---

## 7. Liên kết

- [[05-TS-Functions]] — Generic functions, type param inference
- [[06-Classes-va-Interface]] — Generic class và interface
- [[03-Type-Combination-Union-Intersection]] — Utility types được implement bằng generics
- [[04-Type-Guard]] — Generic type predicates
