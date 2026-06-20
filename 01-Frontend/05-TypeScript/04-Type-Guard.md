---
title: "TypeScript Type Guard"
section: 05-TypeScript
tags: [typescript, type-guard, narrowing, discriminated-union, predicate, fresher, frontend]
related:
  - "[[02-Basic-Types]]"
  - "[[03-Type-Combination-Union-Intersection]]"
  - "[[06-Classes-va-Interface]]"
difficulty: ⭐⭐⭐
estimated_time: 35m
source: [TypeScript Handbook, typescriptlang.org/docs/handbook/2/narrowing.html]
---

# TypeScript Type Guard

> [!summary] TL;DR
> **Type narrowing** = TypeScript thu hẹp type trong một block code dựa trên điều kiện. Các guard có sẵn: **`typeof`** cho primitives, **`instanceof`** cho class instances, **`in`** operator cho property existence, equality narrowing (`=== null`). **Discriminated union** dùng `kind`/`type` field để TS tự narrow. **Custom type predicate** (`x is T`) để viết guard phức tạp dùng lại được.

---

## 1. Khái niệm

### Type Narrowing là gì?

TypeScript phân tích flow control để thu hẹp type của variable trong từng block:

```typescript
function printLength(value: string | number) {
  // Ở đây: value là string | number
  if (typeof value === 'string') {
    // Ở đây: TS biết value là string
    console.log(value.toUpperCase()); // OK — string method
  } else {
    // Ở đây: TS biết value là number
    console.log(value.toFixed(2));    // OK — number method
  }
  // Ở đây: value lại là string | number
}
```

```
★ Insight ─────────────────────────────────────
• Narrowing = TS ĐỌC chính các phép kiểm tra runtime của bạn (typeof, instanceof,
  in, ===) và tự suy ra type hẹp hơn trong nhánh đó. Bạn không "ép kiểu" — bạn
  viết check JS bình thường, TS theo dõi flow. Đây là cầu nối đẹp giữa kiểm tra
  lúc chạy và an toàn lúc biên dịch.
• Custom predicate `x is T` là "TIN TÔI ĐI": sau khi gọi, TS coi x là T tuyệt
  đối mà KHÔNG kiểm tra thân hàm — viết `return true` bừa là tạo lỗ hổng kiểu ngầm.
  Vì vậy predicate chỉ nên dùng khi typeof/instanceof không đủ (validate shape
  object từ API), và phải implement check ĐÚNG. Đừng quên: typeof null === 'object'.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp / API

### 2.1 `typeof` Guard — cho primitives

```typescript
type StringOrNumber = string | number | boolean;

function process(value: StringOrNumber): string {
  if (typeof value === 'string') {
    return value.toUpperCase(); // string
  }
  if (typeof value === 'number') {
    return value.toFixed(2);    // number
  }
  // TS infers: value là boolean ở đây
  return value ? 'YES' : 'NO';
}

// typeof checks: 'string' | 'number' | 'boolean' | 'bigint' | 'symbol' | 'object' | 'function' | 'undefined'
// LƯU Ý: typeof null === 'object' — phải check null riêng
function handleNullable(x: string | null) {
  if (typeof x === 'string') {  // narrowing OK — null bị loại
    return x.length;
  }
  return 0;
}
```

### 2.2 `instanceof` Guard — cho class instances

```typescript
class ApiError extends Error {
  constructor(
    message: string,
    public readonly statusCode: number,
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

class NetworkError extends Error {
  constructor(
    message: string,
    public readonly url: string,
  ) {
    super(message);
    this.name = 'NetworkError';
  }
}

function handleError(err: unknown): string {
  if (err instanceof ApiError) {
    return `API Error [${err.statusCode}]: ${err.message}`; // có statusCode
  }
  if (err instanceof NetworkError) {
    return `Network Error (${err.url}): ${err.message}`;    // có url
  }
  if (err instanceof Error) {
    return `Error: ${err.message}`;
  }
  return 'Unknown error';
}

try {
  throw new ApiError('Not found', 404);
} catch (err) {
  console.log(handleError(err)); // "API Error [404]: Not found"
}
```

### 2.3 `in` Operator Guard — property existence

```typescript
interface Fish { swim(): void; gill: string }
interface Bird { fly(): void;  wing: string }
type Animal = Fish | Bird;

function move(animal: Animal): void {
  if ('swim' in animal) {
    animal.swim(); // TS biết đây là Fish
  } else {
    animal.fly();  // TS biết đây là Bird
  }
}

// Pattern thực tế: phân biệt request types
interface GetRequest  { method: 'GET';  url: string }
interface PostRequest { method: 'POST'; url: string; body: unknown }
type Request = GetRequest | PostRequest;

function processRequest(req: Request) {
  if ('body' in req) {
    // req là PostRequest
    console.log('POST body:', req.body);
  } else {
    // req là GetRequest
    console.log('GET request to:', req.url);
  }
}
```

### 2.4 Discriminated Union — `kind`/`type` field

```typescript
// Pattern mạnh nhất cho complex union types
type LoadingState = {
  status: 'loading';
};

type SuccessState<T> = {
  status:  'success';
  data:    T;
  fetchedAt: Date;
};

type ErrorState = {
  status:  'error';
  error:   Error;
  retries: number;
};

type AsyncState<T> = LoadingState | SuccessState<T> | ErrorState;

interface User { id: number; name: string }

function renderUserState(state: AsyncState<User>): string {
  switch (state.status) {
    case 'loading':
      return 'Loading...';
    case 'success':
      return `User: ${state.data.name} (fetched at ${state.fetchedAt.toISOString()})`;
    case 'error':
      return `Error: ${state.error.message} (${state.retries} retries)`;
  }
}

// Redux-style actions — cũng là discriminated union
type Action =
  | { type: 'INCREMENT'; payload: number }
  | { type: 'DECREMENT'; payload: number }
  | { type: 'RESET' }
  | { type: 'SET_USER'; payload: User };

function reducer(state: number, action: Action): number {
  switch (action.type) {
    case 'INCREMENT': return state + action.payload;
    case 'DECREMENT': return state - action.payload;
    case 'RESET':     return 0;
    case 'SET_USER':
      console.log('User set:', action.payload.name);
      return state;
  }
}
```

### 2.5 Custom Type Predicate

```typescript
// Syntax: function guard(x: SomeType): x is SpecificType
// Dùng khi typeof/instanceof không đủ

// Simple predicate
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function isNonNullObject(value: unknown): value is Record<string, unknown> {
  return typeof value === 'object' && value !== null;
}

// Validate object shape từ API
interface ProductData {
  id:    number;
  name:  string;
  price: number;
}

function isProductData(obj: unknown): obj is ProductData {
  return (
    isNonNullObject(obj) &&
    typeof obj.id    === 'number' &&
    typeof obj.name  === 'string' &&
    typeof obj.price === 'number'
  );
}

async function fetchAndValidateProduct(id: number): Promise<ProductData | null> {
  const res  = await fetch(`/api/products/${id}`);
  const json: unknown = await res.json();

  if (isProductData(json)) {
    return json; // TS biết đây là ProductData
  }
  console.error('Invalid product data:', json);
  return null;
}

// Array filter với predicate
const values: Array<string | null | undefined> = ['hello', null, 'world', undefined, 'ts'];
const strings = values.filter((v): v is string => typeof v === 'string');
// strings: string[] — không còn null/undefined
```

### 2.6 Equality Narrowing và Truthy Narrowing

```typescript
// Equality narrowing
function processId(id: string | number | null) {
  if (id === null) {
    console.log('No ID');
    return;
  }
  // id là string | number ở đây

  if (id === '') {
    console.log('Empty string ID');
    return;
  }
  // id là non-empty string | number
}

// Truthy narrowing
function greet(name: string | null | undefined) {
  if (name) {
    // name là string (truthy — không phải '', null, undefined)
    console.log(`Hello, ${name.toUpperCase()}!`);
  } else {
    console.log('Hello, stranger!');
  }
}

// Nullish coalescing với narrowing
const value: string | null = getValue();
const safe = value ?? 'default'; // safe: string (null removed)
```

---

## 3. Ví dụ minh họa

### Ví dụ 1: Error handling với instanceof chain

```typescript
class ValidationError extends Error {
  constructor(public readonly field: string, message: string) {
    super(message);
    this.name = 'ValidationError';
  }
}

class AuthError extends Error {
  constructor(message: string, public readonly code: 'EXPIRED' | 'INVALID') {
    super(message);
    this.name = 'AuthError';
  }
}

async function callApi(endpoint: string): Promise<unknown> {
  const res = await fetch(endpoint);
  if (res.status === 401) throw new AuthError('Token expired', 'EXPIRED');
  if (res.status === 422) throw new ValidationError('email', 'Invalid email format');
  if (!res.ok)            throw new Error(`HTTP ${res.status}`);
  return res.json();
}

async function safeCallApi(endpoint: string) {
  try {
    return await callApi(endpoint);
  } catch (err: unknown) {
    if (err instanceof ValidationError) {
      return { error: `Validation: ${err.field} — ${err.message}` };
    }
    if (err instanceof AuthError) {
      if (err.code === 'EXPIRED') { /* refresh token */ }
      return { error: `Auth: ${err.message}` };
    }
    if (err instanceof Error) {
      return { error: err.message };
    }
    return { error: 'Unknown error' };
  }
}
```

### Ví dụ 2: Form field validator với discriminated union

```typescript
type FieldValid   = { valid: true;  value: string }
type FieldInvalid = { valid: false; errors: string[] }
type FieldResult  = FieldValid | FieldInvalid;

function validateEmail(value: string): FieldResult {
  const errors: string[] = [];
  if (!value) errors.push('Email không được để trống');
  if (value && !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) {
    errors.push('Email không đúng định dạng');
  }
  return errors.length > 0 ? { valid: false, errors } : { valid: true, value };
}

function processForm(email: string) {
  const result = validateEmail(email);
  if (!result.valid) {
    result.errors.forEach(e => console.error(e)); // errors: string[]
    return;
  }
  // TS biết result.valid === true → result là FieldValid
  console.log('Valid email:', result.value.toLowerCase()); // value: string
}
```

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Pitfall 1: `typeof null === 'object'` — phải check null riêng
> `typeof null` trả về `'object'` trong JS (historical bug). Nếu dùng `typeof x === 'object'` để guard → vẫn có thể là null. Luôn thêm `&& x !== null` hoặc check null trước: `if (x === null) return`.

> [!warning] Pitfall 2: Type predicate phản ánh runtime — compiler tin bạn tuyệt đối
> ```typescript
> function isFish(x: Fish | Bird): x is Fish {
>   return true; // LIE! — TS tin hoàn toàn
> }
> ```
> Custom predicate phải implement đúng logic runtime. TS không kiểm tra body của predicate function — trách nhiệm hoàn toàn thuộc về dev.

> [!tip] Sắp xếp narrow từ specific đến general
> Trong catch hoặc if chain, check type cụ thể trước (ApiError, NetworkError) rồi mới check type chung (Error, object). Check `instanceof Error` trước sẽ "nuốt" cả ApiError.

---

## 5. Câu hỏi phỏng vấn thường gặp

**Q1: Type Guard là gì? Liệt kê các loại type guard trong TypeScript.**

> Type guard là kỹ thuật/code pattern giúp TypeScript thu hẹp (narrow) type của variable trong một block. Các loại: (1) **`typeof`** guard cho primitives (`typeof x === 'string'`). (2) **`instanceof`** guard cho class instances. (3) **`in`** operator để check property existence. (4) **Equality narrowing** (`x === null`, `x === 'literal'`). (5) **Truthy narrowing** (`if (x)`). (6) **Custom type predicate** `function f(x): x is T`.

**Q2: Discriminated Union là gì? Tại sao nó hữu ích?**

> Discriminated Union là union type trong đó mỗi member có một common "discriminant" property (thường `kind`, `type`, `status`) với literal type. TS dùng discriminant để narrow tự động trong switch/if. Hữu ích vì: (1) switch exhaustive check — TS báo lỗi nếu thiếu case. (2) Không cần custom predicate. (3) Mô hình hoá state machine (loading/success/error), Redux actions. Đây là pattern cơ bản của nhiều TS codebase lớn.

**Q3: Custom type predicate `x is T` dùng khi nào?**

> Dùng khi `typeof`/`instanceof`/`in` không đủ để narrow — ví dụ: validate shape của object từ external source (API, localStorage), check nhiều properties cùng lúc, hoặc cần reusable guard function. Syntax: `function isT(x: unknown): x is T { return /* runtime check */ }`. Sau khi call predicate, TS tin hoàn toàn vào kết quả — body không được verify bởi compiler.

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Viết discriminated union `AsyncResult<T>` có 3 state: `loading`, `success` (với `data: T`), `error` (với `message: string`). Viết React component-like function `renderResult<T>(result: AsyncResult<T>, render: (data: T) => string): string` sử dụng switch exhaustive.

- [ ] **Bài 2:** Viết custom predicate `isValidUser(obj: unknown): obj is User` check đầy đủ shape của `User { id: number; name: string; email: string }`. Dùng nó trong `processApiResponse(response: unknown): User[]` filter array từ API.

---

## 7. Liên kết

- [[02-Basic-Types]] — `unknown`, `never` cần narrowing để dùng
- [[03-Type-Combination-Union-Intersection]] — Discriminated union là Union với discriminant
- [[06-Classes-va-Interface]] — `instanceof` guard dựa trên class hierarchy
- [[07-Generics]] — Generic type predicates
