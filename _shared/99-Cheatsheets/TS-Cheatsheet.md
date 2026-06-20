---
title: "TypeScript Cheatsheet"
section: 99-Cheatsheets
tags: [cheatsheet, typescript, types, generics, utility-types, reference, fresher, frontend]
related:
  - "[[01-TS-Overview]]"
  - "[[02-Basic-Types]]"
  - "[[03-Type-Combination-Union-Intersection]]"
  - "[[06-Classes-va-Interface]]"
  - "[[07-Generics]]"
difficulty: ⭐
estimated_time: 10m
source: [typescriptlang.org/docs, typescript-cheatsheets.netlify.app]
---

# TypeScript Cheatsheet

> [!summary] TL;DR
> Tra nhanh basic types, interface vs type, generics và utility types trước phỏng vấn. Điểm hay hỏi nhất: `interface` vs `type alias`, `unknown` vs `any`, Utility types (`Partial`, `Pick`, `Omit`), generic constraints.

---

## 1. Basic Types

```ts
// Primitives
let name:    string  = 'Alice';
let age:     number  = 30;
let active:  boolean = true;
let nothing: null    = null;
let missing: undefined = undefined;

// Special types
let x: any;       // tắt type check — tránh dùng
let y: unknown;   // phải narrow trước khi dùng — dùng thay any
let z: never;     // không bao giờ xảy ra (throw / infinite loop)
let v: void;      // function không return (hoặc return undefined)

// Object
let obj: object;                           // bất kỳ non-primitive
let point: { x: number; y: number };      // object literal type

// Array
let nums: number[]       = [1, 2, 3];
let strs: Array<string>  = ['a', 'b'];    // generic syntax

// Tuple — array cố định độ dài và kiểu
let pair: [string, number] = ['age', 30];

// Enum
enum Direction { Up, Down, Left, Right }  // 0,1,2,3
enum Status { Active = 'ACTIVE', Inactive = 'INACTIVE' }
```

---

## 2. Interface vs Type Alias

| | `interface` | `type` |
|---|---|---|
| Extend | `extends` (OOP style) | `&` intersection |
| Merge (declaration merging) | ✅ có thể | ❌ không thể |
| Primitive alias | ❌ | ✅ `type ID = string` |
| Union | ❌ | ✅ `type Result = OK \| Err` |
| Computed properties | ❌ | ✅ |
| Khi nào dùng | Object shapes, class contracts | Union, intersection, alias |

```ts
// Interface — declaration merging (thêm được)
interface User { name: string; }
interface User { age: number; }    // OK — merge thành { name, age }

// Type alias — không merge được
type Product = { id: number; name: string; };
// type Product = { price: number }; // Error: Duplicate identifier

// Extend
interface Admin extends User { role: string; }
type AdminT = User & { role: string };  // tương đương

// Best practice: interface cho object/class, type cho union/alias
```

---

## 3. Union & Intersection & Literal Types

```ts
// Union — một trong các kiểu
type StringOrNumber = string | number;
type Status = 'loading' | 'success' | 'error';  // string literal union

// Intersection — có tất cả properties
type AdminUser = User & { permissions: string[] };

// Literal types
const direction: 'up' | 'down' | 'left' | 'right' = 'up';
function setAlign(align: 'left' | 'center' | 'right') { ... }

// Discriminated Union — pattern phổ biến
type Result<T> =
  | { status: 'success'; data: T }
  | { status: 'error';   error: string };

function handle(result: Result<User>) {
  if (result.status === 'success') {
    console.log(result.data.name); // TypeScript biết type ở đây
  } else {
    console.error(result.error);
  }
}
```

---

## 4. Type Narrowing & Type Guard

```ts
// typeof
function process(x: string | number) {
  if (typeof x === 'string') x.toUpperCase(); // string
  else x.toFixed(2);                          // number
}

// instanceof
if (err instanceof Error) console.log(err.message);

// in operator
if ('admin' in user) { /* user có property admin */ }

// Custom type guard — return type là `x is Type`
function isUser(x: unknown): x is User {
  return typeof x === 'object' && x !== null && 'name' in x;
}
```

---

## 5. Functions

```ts
// Parameter types + return type
function add(a: number, b: number): number { return a + b; }

// Optional (?) và default
function greet(name: string, greeting?: string): string {
  return `${greeting ?? 'Hello'}, ${name}`;
}

// Rest params
function sum(...nums: number[]): number {
  return nums.reduce((a, b) => a + b, 0);
}

// Function type
type Handler = (event: MouseEvent) => void;
type Fetcher = <T>(url: string) => Promise<T>;

// Overloads
function format(x: string): string;
function format(x: number): string;
function format(x: string | number): string {
  return String(x);
}
```

---

## 6. Generics

```ts
// Basic generic
function identity<T>(arg: T): T { return arg; }
identity<string>('hello');   // explicit
identity(42);                // inferred → T = number

// Generic interface
interface ApiResponse<T> {
  data:    T;
  status:  number;
  message: string;
}

// Generic constraints — T phải có property length
function logLength<T extends { length: number }>(x: T): void {
  console.log(x.length);
}

// keyof constraint — K phải là key của T
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Default generic
interface Paginated<T = unknown> {
  items:      T[];
  total:      number;
  page:       number;
}
```

---

## 7. Utility Types — Bảng Tra Nhanh

| Utility Type | Mô tả | Ví dụ |
|---|---|---|
| `Partial<T>` | tất cả properties optional | `Partial<User>` |
| `Required<T>` | tất cả properties required | `Required<Config>` |
| `Readonly<T>` | tất cả properties readonly | `Readonly<Config>` |
| `Pick<T, K>` | chỉ lấy properties K | `Pick<User, 'name' \| 'email'>` |
| `Omit<T, K>` | bỏ properties K | `Omit<User, 'password'>` |
| `Record<K, V>` | object map K → V | `Record<string, number>` |
| `Exclude<T, U>` | loại bỏ U khỏi union T | `Exclude<'a'\|'b'\|'c', 'a'>` → `'b'\|'c'` |
| `Extract<T, U>` | giữ lại U trong union T | `Extract<string\|number, string>` → `string` |
| `NonNullable<T>` | loại bỏ null/undefined | `NonNullable<string\|null>` → `string` |
| `ReturnType<T>` | kiểu return của function | `ReturnType<typeof fn>` |
| `Parameters<T>` | kiểu params của function | `Parameters<typeof fn>` |
| `Awaited<T>` | unwrap Promise type | `Awaited<Promise<string>>` → `string` |

```ts
// Ví dụ thực tế: form update (chỉ cần một số fields)
interface User {
  id:       number;
  name:     string;
  email:    string;
  password: string;
}

type UserUpdateForm = Omit<User, 'id' | 'password'>;
// → { name: string; email: string }

type UserPublic = Pick<User, 'id' | 'name' | 'email'>;
// → { id: number; name: string; email: string }

type PartialUpdate = Partial<UserUpdateForm>;
// → { name?: string; email?: string }
```

---

## 8. Classes & Interface Contract

```ts
interface Serializable {
  serialize():   string;
  deserialize(s: string): this;
}

class User implements Serializable {
  constructor(
    public  name:   string,
    private email:  string,
    readonly id:    number,
  ) {}

  serialize(): string {
    return JSON.stringify({ name: this.name, id: this.id });
  }

  deserialize(s: string): this {
    Object.assign(this, JSON.parse(s));
    return this;
  }
}

// Access modifiers
// public    — ai cũng truy cập được (default)
// private   — chỉ class này
// protected — class này + subclasses
// readonly  — chỉ đọc sau khi khởi tạo
```

---

## 9. `any` vs `unknown` — Quan Trọng!

> [!warning] Tránh `any` — dùng `unknown` thay thế
> `any` tắt hoàn toàn type check. `unknown` yêu cầu narrow type trước khi dùng — an toàn hơn.

```ts
let x: any;
x.foo();              // ✅ (không lỗi nhưng runtime có thể crash)

let y: unknown;
y.foo();              // ❌ Error: Object is of type 'unknown'
if (typeof y === 'string') y.toUpperCase(); // ✅ sau khi narrow
```

---

## 10. Câu Hỏi Phỏng Vấn Thường Gặp

**Q1: `interface` vs `type` alias — khi nào dùng cái nào?**

> `interface` cho object shapes và class contracts — hỗ trợ declaration merging (thêm properties sau). `type` alias linh hoạt hơn — dùng cho union, intersection, primitive alias, computed types. Thực tế: dùng `interface` cho domain models (User, Product), `type` cho helper types và unions.

**Q2: `unknown` vs `any` — tại sao nên dùng `unknown`?**

> `any` tắt hoàn toàn type check — không có lỗi compile nhưng có thể crash runtime. `unknown` là type-safe alternative: phải dùng type narrowing (`typeof`, `instanceof`, type guard) trước khi thao tác. Dùng `unknown` cho data từ external source (API response, JSON.parse, error objects).

**Q3: Generics dùng để làm gì? Cho ví dụ.**

> Generics tạo code tái sử dụng với type-safety — function/class/interface hoạt động với nhiều kiểu khác nhau mà vẫn an toàn. Ví dụ: `useState<User>(null)` — React biết state là User, tránh lỗi type. `ApiResponse<Product[]>` — response generic cho mọi endpoint. Khác với `any`: generic preserves type information.

---

## 11. Bài Tập Tự Kiểm Tra

- [ ] Viết generic function `groupBy<T>(arr: T[], key: keyof T): Record<string, T[]>` — group array theo key.
- [ ] Tạo `DeepPartial<T>` utility type — làm tất cả properties (kể cả nested) trở thành optional.

---

## 12. Liên Kết

- [[02-Basic-Types]] — Basic types chi tiết
- [[03-Type-Combination-Union-Intersection]] — Union, Intersection, Discriminated Union
- [[06-Classes-va-Interface]] — Classes và Interface
- [[07-Generics]] — Generics chi tiết
