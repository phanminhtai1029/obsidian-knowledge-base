---
title: "TypeScript Basic Types"
section: 05-TypeScript
tags: [typescript, types, primitive, enum, tuple, unknown, never, fresher, frontend]
related:
  - "[[01-TS-Overview]]"
  - "[[03-Type-Combination-Union-Intersection]]"
difficulty: ⭐⭐
estimated_time: 35m
source: [TypeScript Handbook, typescriptlang.org/docs/handbook/2/everyday-types.html]
---

# TypeScript Basic Types

> [!summary] TL;DR
> TypeScript cung cấp đủ types cho mọi giá trị JS: **primitive** (`string`, `number`, `boolean`), **array** (`T[]`), **tuple** (fixed-length typed array), **enum** (named constants). **Special types**: `any` (opt-out checking), `unknown` (type-safe any — phải narrow trước), `never` (unreachable / exhaustive check), `void` (không return). `strictNullChecks: true` bắt buộc handle `null`/`undefined` riêng.

> [!tip] 🎯 Hiểu trong 30 giây
> Đây là "bộ nhãn kiểu" cơ bản: `string`, `number`, `boolean`, mảng `number[]`, **tuple** (mảng *cố định độ dài & kiểu từng vị trí*, vd `[string, number]`), **enum** (tập hằng số có tên).
> **Ba "kiểu đặc biệt" hay ra thi — phân biệt cho rõ:**
> - **`any`** = "tắt kiểm tra kiểu", muốn làm gì cũng được → **nguy hiểm**, mất hết lợi ích của TS.
> - **`unknown`** = "chưa biết kiểu nhưng vẫn an toàn": gán gì vào cũng được, *nhưng trước khi dùng phải kiểm tra (narrow) bằng `typeof`/`instanceof`*. Đây là phiên bản *an toàn* của `any`.
> - **`never`** = "không bao giờ có giá trị" (hàm luôn throw, hoặc nhánh không thể xảy ra).
>
> Bật `strictNullChecks` thì `null`/`undefined` phải xử lý riêng (không lẫn vào kiểu khác) → tránh lỗi "cannot read property of undefined".

---

## 1. Khái niệm

### Hệ thống types trong TypeScript

```text
TypeScript Types
├── Primitive:    string, number, boolean, bigint, symbol
├── Object:       object, array T[], tuple, function
├── Special:      any, unknown, never, void
├── Nullish:      null, undefined (tách biệt khi strictNullChecks)
├── Literal:      "hello", 42, true, null, undefined
└── Composite:    union |, intersection &, enum
```

```
★ Insight ─────────────────────────────────────
• any vs unknown là cặp đối lập về an toàn: any TẮT kiểm tra (dùng được mọi thứ,
  crash runtime), unknown CHẶN cho tới khi bạn narrow (typeof/guard). Quy tắc:
  dữ liệu "chưa rõ kiểu" (JSON.parse, catch err) → để unknown rồi thu hẹp, đừng
  bao giờ để any cho qua. any là "lỗ thoát" tạm thời, unknown là cách an toàn.
• never = "không thể xảy ra", và mẹo exhaustive-check biến nó thành lưới an toàn:
  gán biến union vào `const _: never = x` ở default của switch → thêm thành viên
  union mà quên xử lý là COMPILER BÁO LỖI ngay. Cộng với `as const` (giữ literal
  thay vì widen về string), đây là 2 kỹ thuật làm type "biết nói" hộ bạn.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp / API

### 2.1 Primitive Types

```typescript
// string
let firstName: string = 'Alice';
let greeting: string  = `Hello, ${firstName}!`;

// number — không phân biệt int/float
let age:  number = 25;
let pi:   number = 3.14;
let hex:  number = 0xff;
let big:  number = 1_000_000; // numeric separators

// boolean
let isActive:   boolean = true;
let isLoggedIn: boolean = false;

// null và undefined (với strictNullChecks)
let nothing: null      = null;
let notSet:  undefined = undefined;

// strictNullChecks: true → phải handle riêng
let name: string | null = null;
if (name !== null) {
  console.log(name.toUpperCase()); // safe — TS biết name là string ở đây
}

// Optional chaining + nullish coalescing
const display = name?.toUpperCase() ?? 'Anonymous';
```

### 2.2 Array và Tuple

```typescript
// Array — 2 cú pháp tương đương
const nums:   number[]      = [1, 2, 3];
const strs:   Array<string> = ['a', 'b', 'c'];
const matrix: number[][]    = [[1, 2], [3, 4]];

// Tuple — fixed-length, mỗi position có type riêng
const pair:  [string, number]  = ['Alice', 25];
const point: [number, number]  = [10, 20];

// Destructuring tuple
const [name, ageVal] = pair; // name: string, ageVal: number
console.log(name.toUpperCase()); // "ALICE"
console.log(ageVal.toFixed(0));  // "25"

// Named tuple (TS 4.0+) — dễ đọc hơn
type Point3D = [x: number, y: number, z: number];
const p: Point3D = [1, 2, 3];

// Optional tuple element
type Config = [host: string, port: number, ssl?: boolean];
const c1: Config = ['localhost', 3000];        // OK
const c2: Config = ['localhost', 3000, true];  // OK
```

### 2.3 Enum

```typescript
// Numeric enum (default: 0, 1, 2...)
enum Direction {
  Up,    // 0
  Down,  // 1
  Left,  // 2
  Right, // 3
}
const dir: Direction = Direction.Up;
console.log(dir);           // 0
console.log(Direction[0]);  // "Up" (reverse mapping)

// Numeric enum với giá trị custom
enum HttpStatus {
  OK          = 200,
  NotFound    = 404,
  ServerError = 500,
}

// String enum — KHÔNG có reverse mapping — thường prefer hơn
enum Color {
  Red   = 'RED',
  Green = 'GREEN',
  Blue  = 'BLUE',
}
const c: Color = Color.Red;
console.log(c); // "RED"

// Const enum — inline lúc compile, không tạo JS object
const enum Size {
  Small  = 'S',
  Medium = 'M',
  Large  = 'L',
}
const mySize: Size = Size.Medium; // biên dịch thành: const mySize = 'M'
```

### 2.4 Special Types: any, unknown, never, void

```typescript
// any — opt-out type checking (tránh dùng)
let data: any = 42;
data = 'hello';        // OK
data.toUpperCase();    // OK compile — nhưng có thể crash runtime
data.nonexistent.deep; // OK compile — crash runtime!

// unknown — type-safe any (phải narrow trước khi dùng)
let parsed: unknown = JSON.parse('{"name":"Alice"}');
// parsed.name;               // Error: Object is of type 'unknown'
if (typeof parsed === 'object' && parsed !== null && 'name' in parsed) {
  console.log((parsed as { name: string }).name); // safe sau khi narrow
}

// void — hàm không return giá trị có nghĩa
function logMessage(msg: string): void {
  console.log(msg);
  // không return, hoặc return; (trả về undefined)
}

// never — function không bao giờ return (throw hoặc infinite loop)
function throwError(msg: string): never {
  throw new Error(msg);
}

// never trong exhaustive check — cực kỳ hữu ích
type Shape = 'circle' | 'square' | 'triangle';

function getArea(shape: Shape): number {
  switch (shape) {
    case 'circle':   return Math.PI * 25;
    case 'square':   return 25;
    case 'triangle': return 12.5;
    default:
      const _exhaustive: never = shape; // Error nếu thiếu case ở trên
      throw new Error(`Unknown shape: ${_exhaustive}`);
  }
}
// Nếu thêm 'pentagon' vào Shape mà quên thêm case → TS báo lỗi ngay
```

### 2.5 Literal Types và Type Assertions

```typescript
// Literal types — giá trị cụ thể làm type
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH';
type DiceRoll   = 1 | 2 | 3 | 4 | 5 | 6;

let method: HttpMethod = 'GET';
// method = 'CONNECT'; // Error

// as const — prevent widening từ string → literal
const config = {
  method: 'GET' as const,  // type: 'GET', không phải string
  url: '/api/users',       // type: string (widened — OK cho URL)
} as const;
// config.method = 'POST'; // Error — readonly

// Lấy union type từ array
const methods = ['GET', 'POST', 'PUT'] as const;
type AllMethods = typeof methods[number]; // 'GET' | 'POST' | 'PUT'

// Type assertions — "compiler, tôi biết type này"
const input = document.getElementById('email') as HTMLInputElement;
console.log(input.value); // string — safe vì đã assert

// Non-null assertion operator (!)
const btn = document.querySelector('button')!; // assert không null
btn.addEventListener('click', () => {});
```

---

## 3. Ví dụ minh họa

### Ví dụ 1: API Response typing với unknown

```typescript
interface Product {
  id:       number;
  name:     string;
  price:    number;
  category: 'electronics' | 'clothing' | 'food';
  inStock:  boolean;
  tags?:    string[];
}

// Type guard để validate dữ liệu từ API
function isProduct(obj: unknown): obj is Product {
  return (
    typeof obj === 'object' && obj !== null &&
    typeof (obj as any).id    === 'number' &&
    typeof (obj as any).name  === 'string' &&
    typeof (obj as any).price === 'number'
  );
}

async function fetchProduct(id: number): Promise<Product> {
  const res  = await fetch(`/api/products/${id}`);
  const json: unknown = await res.json();

  if (!isProduct(json)) {
    throw new Error('Invalid product data from API');
  }
  return json; // TS biết đây là Product sau khi guard
}
```

### Ví dụ 2: Enum trong State Machine

```typescript
const enum OrderStatus {
  Pending   = 'PENDING',
  Confirmed = 'CONFIRMED',
  Shipped   = 'SHIPPED',
  Delivered = 'DELIVERED',
  Cancelled = 'CANCELLED',
}

interface Order {
  id:     number;
  status: OrderStatus;
  total:  number;
}

const STATUS_LABELS: Record<OrderStatus, string> = {
  [OrderStatus.Pending]:   'Chờ xác nhận',
  [OrderStatus.Confirmed]: 'Đã xác nhận',
  [OrderStatus.Shipped]:   'Đang giao hàng',
  [OrderStatus.Delivered]: 'Đã giao',
  [OrderStatus.Cancelled]: 'Đã hủy',
};

const CANCELLABLE: Set<OrderStatus> = new Set([
  OrderStatus.Pending,
  OrderStatus.Confirmed,
]);

function canCancel(order: Order): boolean {
  return CANCELLABLE.has(order.status);
}

const order: Order = { id: 1, status: OrderStatus.Pending, total: 500000 };
console.log(STATUS_LABELS[order.status]); // "Chờ xác nhận"
console.log(canCancel(order));            // true
```

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Pitfall 1: Dùng `any` cho dữ liệu từ API
> `JSON.parse()` và response từ `fetch()` trả về `any` (hoặc nên cast sang `unknown`). Dùng `any` → mất type safety ngay. Dùng `unknown` + type guard/Zod → validate và lấy inferred type an toàn.

> [!warning] Pitfall 2: Numeric enum có reverse mapping gây confusion
> `enum E { A = 0 }` → `E[0] === 'A'` là true (reverse mapping). String enum không có reverse mapping. `const enum` bị inline — không dùng được trong dynamic lookup như `someObject[someVar]`.

> [!tip] Prefer `as const` + union type thay vì enum
> `const COLORS = { Red: 'RED', Blue: 'BLUE' } as const; type Color = typeof COLORS[keyof typeof COLORS]` — thuần JS, không có runtime overhead, không cần compile. Nhiều team TS hiện đại prefer pattern này hơn enum.

---

## 5. Câu hỏi phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "any vs unknown, vì sao API trả về nên dùng unknown?"
> *"Cả any và unknown đều nhận được mọi giá trị gán vào, nhưng khác nhau ở mức an toàn. any tắt hoàn toàn type checking, em dùng nó như thế nào cũng được mà không bị cảnh báo, nên rất dễ lọt lỗi, coi như mất lợi ích của TypeScript. unknown thì an toàn hơn: gán gì vào cũng được nhưng trước khi dùng em buộc phải thu hẹp kiểu bằng typeof, instanceof hay type predicate, nếu không TypeScript sẽ báo lỗi. Vì vậy với dữ liệu từ API trả về, em không kiểm soát được hình dạng thật, dùng unknown được coi là best practice: nó ép em phải kiểm tra và ép kiểu cẩn thận trước khi truy cập, tránh giả định sai. Tương tự catch error trong TS mới cũng nên để kiểu unknown."*

> [!note] 🧠 Mẹo nhớ
> **`any` = tắt kiểm tra (nguy hiểm); `unknown` = an toàn, phải narrow trước khi dùng.** Dữ liệu API/catch → **`unknown`**. `never` = không bao giờ có giá trị.

**Q1: Phân biệt `any` và `unknown`?**

> Cả hai accept mọi giá trị gán vào. Khác biệt: `any` **tắt type checking** — có thể dùng bất kỳ operation nào không cần check, rất nguy hiểm. `unknown` **an toàn** — trước khi dùng phải **narrow** bằng `typeof`, `instanceof`, type predicate. `unknown` là "type-safe version của `any`" — dùng khi không biết type nhưng vẫn muốn type safety (ví dụ: `catch (err: unknown)` trong TS 4.0+).

**Q2: `never` type dùng khi nào?**

> `never` là type của code **không thể reach** hoặc function **không bao giờ return**: (1) Function luôn throw: `function fail(): never { throw new Error() }`. (2) Infinite loop. (3) **Exhaustive check** trong switch: nếu `default` case nhận biến type `never`, compiler báo lỗi nếu có union member chưa handle — safety net tuyệt vời khi mở rộng union type.

**Q3: Tuple khác Array ở điểm gì?**

> **Array** `T[]`: tất cả elements cùng type, độ dài linh hoạt. **Tuple** `[T1, T2]`: số elements và type của mỗi position được cố định. Tuple hay dùng cho: function return nhiều value `[data, error]`, coordinate `[x, y, z]`, key-value pairs. Named tuples `[x: number, y: number]` (TS 4.0+) giúp code tự document hơn.

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Định nghĩa type `RGB` là `[r: number, g: number, b: number]`. Viết `rgbToHex(rgb: RGB): string` trả về hex string dạng `#RRGGBB`. Đảm bảo mỗi channel trong range 0-255 (dùng `Math.min(Math.max(v, 0), 255)`).

- [ ] **Bài 2:** Tạo string enum `Permission` (`READ`, `WRITE`, `DELETE`, `ADMIN`). Viết function `hasPermission(userPerms: Permission[], required: Permission): boolean`. Thêm exhaustive switch trong `getPermissionLabel(p: Permission): string` để test.

---

## 7. Liên kết

- [[01-TS-Overview]] — Cài đặt TS, tsconfig, type inference
- [[03-Type-Combination-Union-Intersection]] — Union, intersection, keyof, typeof
- [[04-Type-Guard]] — Narrowing từ broad types (unknown, union) sang specific type
- [[07-Generics]] — Generic types dùng basic types làm type params
