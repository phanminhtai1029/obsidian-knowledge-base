---
title: "TypeScript Functions"
section: 05-TypeScript
tags: [typescript, function, overload, optional, default, rest, fresher, frontend]
related:
  - "[[02-Basic-Types]]"
  - "[[03-Type-Combination-Union-Intersection]]"
  - "[[07-Generics]]"
difficulty: ⭐⭐⭐
estimated_time: 35m
source: [TypeScript Handbook, typescriptlang.org/docs/handbook/2/functions.html]
---

# TypeScript Functions

> [!summary] TL;DR
> TypeScript annotation cho functions: **parameter types**, **return type** (thường có thể infer), **optional** (`name?: string`), **default** (`age = 18`), **rest** (`...args: string[]`). **Function overloads** cho phép một function có nhiều signatures với behavior khác nhau tùy input. **`void`** là return type cho hàm không return giá trị có nghĩa; **`never`** cho hàm không bao giờ return.

> [!tip] 🎯 Hiểu trong 30 giây
> Với hàm, TS cho ghi rõ **kiểu của từng tham số** và **kiểu trả về** (`function add(a: number, b: number): number`). Vài "gia vị":
> - **`name?: string`** = tham số *tùy chọn* (có thể không truyền → bên trong là `string | undefined`, phải tự kiểm tra).
> - **`age: number = 18`** = tham số *mặc định* (không truyền thì lấy 18 → bên trong luôn là `number`, khỏi kiểm tra). Thường tiện hơn optional.
> - **`...args: number[]`** = gom nhiều tham số còn lại vào mảng (rest).
> - **`void`** = hàm không trả về gì có nghĩa; **`never`** = hàm *không bao giờ trả về* (luôn throw / vòng lặp vô tận).
> - **Overload** = khai báo *nhiều chữ ký* cho cùng một hàm khi nó cư xử khác nhau tùy kiểu đầu vào.

---

## 1. Khái niệm

### Function types trong TypeScript

TypeScript xem **function signature** như một type — bao gồm param types và return type. Function type có thể assign, pass as callback, store in variable.

```typescript
// Function type — khai báo tường minh
type Predicate<T> = (value: T) => boolean;
type Mapper<T, U> = (item: T, index: number) => U;
type Handler      = (event: Event) => void;

// Sử dụng:
const isPositive: Predicate<number> = (n) => n > 0;
const double:     Mapper<number, number> = (n) => n * 2;
```

```
★ Insight ─────────────────────────────────────
• optional (`x?: T`) và default (`x: T = ...`) giống nhau ở "có thể không truyền"
  nhưng KHÁC ở kiểu BÊN TRONG hàm: optional → `T | undefined` (phải kiểm tra),
  default → `T` (đã thay undefined). Có giá trị mặc định rõ ràng thì chọn default
  để khỏi check undefined; thực sự "có thể vắng" thì dùng optional.
• Overload là HAI mặt: các signature CÔNG KHAI (cái caller thấy, có thể đặc tả
  input→output khác nhau) + một signature THỰC THI (có body, caller KHÔNG gọi
  trực tiếp được). Đây là cách diễn đạt "input loại này → ra type kia"
  (createElement('input')→HTMLInputElement) — điều union return type không làm gọn được.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp / API

### 2.1 Basic Parameter và Return Type

```typescript
// Function declaration
function greet(name: string, formal: boolean): string {
  return formal ? `Good day, ${name}.` : `Hey ${name}!`;
}

// Arrow function
const add = (a: number, b: number): number => a + b;

// Return type inference — TS có thể tự infer trong nhiều trường hợp
function multiply(a: number, b: number) {
  return a * b; // inferred: number
}

// Hàm không có ý nghĩa trả về
function logInfo(msg: string): void {
  console.log(`[INFO] ${msg}`);
}

// Hàm không bao giờ return (throw hoặc infinite loop)
function fail(message: string): never {
  throw new Error(message);
}
```

### 2.2 Optional và Default Parameters

```typescript
// Optional params — dùng ? sau tên (phải ở CUỐI params list)
function createUser(name: string, email: string, role?: string): object {
  return { name, email, role: role ?? 'user' };
}
createUser('Alice', 'alice@example.com');          // OK — role không bắt buộc
createUser('Bob',   'bob@example.com', 'admin');   // OK

// Default params — value được dùng khi không truyền hoặc truyền undefined
function paginate(
  items: number[],
  page:     number = 1,
  pageSize: number = 10,
): number[] {
  const start = (page - 1) * pageSize;
  return items.slice(start, start + pageSize);
}

paginate([1,2,3,4,5]);           // page=1, pageSize=10
paginate([1,2,3,4,5], 2);        // page=2, pageSize=10
paginate([1,2,3,4,5], 1, 3);     // page=1, pageSize=3
paginate([1,2,3,4,5], undefined, 5); // page=1 (default), pageSize=5

// Default với object destructuring
interface Options {
  timeout?: number;
  retries?: number;
  verbose?: boolean;
}

function fetchData(url: string, { timeout = 5000, retries = 3, verbose = false }: Options = {}) {
  if (verbose) console.log(`Fetching ${url}, timeout=${timeout}`);
  // fetch logic...
}

fetchData('/api/users');
fetchData('/api/users', { timeout: 10000, verbose: true });
```

### 2.3 Rest Parameters

```typescript
// Rest params — phải ở vị trí CUỐI
function sum(...nums: number[]): number {
  return nums.reduce((acc, n) => acc + n, 0);
}
console.log(sum(1, 2, 3, 4, 5)); // 15

// Kết hợp với fixed params
function log(level: 'info' | 'warn' | 'error', ...messages: string[]): void {
  const prefix = `[${level.toUpperCase()}]`;
  console.log(prefix, ...messages);
}
log('info', 'Server started', 'Port: 3000');
log('error', 'Connection failed');

// Rest với tuple types
function first<T>(...args: [T, ...unknown[]]): T {
  return args[0];
}
first(1, 'two', true); // type: number
first('hello');        // type: string
```

### 2.4 Function Overloads

```typescript
// Overloads — nhiều signatures, một implementation
// QUAN TRỌNG: implementation signature phải compatible với tất cả overload signatures

// Example 1: Overloaded function trả về type khác nhau tùy input
function formatId(id: string): string;
function formatId(id: number): string;
function formatId(id: string | number): string {
  // implementation — phải handle cả 2 cases
  return typeof id === 'string' ? `STR-${id.toUpperCase()}` : `NUM-${id.toString().padStart(6, '0')}`;
}

const a = formatId('abc'); // type: string
const b = formatId(42);    // type: string

// Example 2: Return type thay đổi theo input type
function createElement(tag: 'input'):  HTMLInputElement;
function createElement(tag: 'button'): HTMLButtonElement;
function createElement(tag: 'div'):    HTMLDivElement;
function createElement(tag: string):   HTMLElement {
  return document.createElement(tag);
}

const input  = createElement('input');   // type: HTMLInputElement
const button = createElement('button');  // type: HTMLButtonElement
input.value = 'hello';   // OK — input có .value
button.click();          // OK — button có .click()

// Example 3: Optional parameter overload
function search(query: string): Promise<string[]>;
function search(query: string, limit: number): Promise<string[]>;
function search(query: string, limit: number = 10): Promise<string[]> {
  return fetch(`/api/search?q=${encodeURIComponent(query)}&limit=${limit}`)
    .then(r => r.json());
}
```

### 2.5 Function Type Signatures và Callbacks

```typescript
// Callback types
type Comparator<T> = (a: T, b: T) => number;
type Transformer<T, U> = (value: T) => U;

function sortBy<T>(arr: T[], comparator: Comparator<T>): T[] {
  return [...arr].sort(comparator);
}

const users = [
  { name: 'Charlie', age: 30 },
  { name: 'Alice',   age: 25 },
  { name: 'Bob',     age: 28 },
];

const byAge  = sortBy(users, (a, b) => a.age - b.age);
const byName = sortBy(users, (a, b) => a.name.localeCompare(b.name));

// Generic callback
function pipe<T>(...fns: Array<Transformer<T, T>>): Transformer<T, T> {
  return (value: T) => fns.reduce((v, fn) => fn(v), value);
}

const processString = pipe(
  (s: string) => s.trim(),
  (s: string) => s.toLowerCase(),
  (s: string) => s.replace(/\s+/g, '-'),
);
console.log(processString('  Hello World  ')); // "hello-world"
```

### 2.6 `this` Parameter

```typescript
// 'this' parameter — chỉ tồn tại ở type level, bị xóa lúc compile
interface Counter {
  count: number;
  increment(this: Counter): void;
  reset(this: Counter): void;
}

const counter: Counter = {
  count: 0,
  increment() {
    this.count++; // TS biết 'this' là Counter
  },
  reset() {
    this.count = 0;
  },
};

// Hàm cần 'this' context
function validate(this: { minAge: number }, age: number): boolean {
  return age >= this.minAge;
}

const validator = { minAge: 18, validate };
console.log(validator.validate(20)); // true
// validate(20); // Error: 'this' context không đúng
```

---

## 3. Ví dụ minh họa

### Ví dụ 1: API client với overloads

```typescript
interface GetOptions  { method?: 'GET' }
interface PostOptions { method: 'POST'; body: unknown }
type RequestOptions = GetOptions | PostOptions;

// Overloads cho type-safe fetch wrapper
async function request(url: string, options?: GetOptions): Promise<unknown>;
async function request(url: string, options: PostOptions): Promise<unknown>;
async function request(url: string, options: RequestOptions = {}): Promise<unknown> {
  const fetchOptions: RequestInit = {
    method: options.method ?? 'GET',
  };

  if ('body' in options && options.method === 'POST') {
    fetchOptions.headers  = { 'Content-Type': 'application/json' };
    fetchOptions.body     = JSON.stringify(options.body);
  }

  const res = await fetch(url, fetchOptions);
  if (!res.ok) throw new Error(`HTTP ${res.status}: ${res.statusText}`);
  return res.json();
}

// Dùng
const users = await request('/api/users');
const newUser = await request('/api/users', {
  method: 'POST',
  body:   { name: 'Alice', email: 'alice@example.com' },
});
```

### Ví dụ 2: Validation library với typed errors

```typescript
type Validator<T> = (value: T) => string | null; // null = valid, string = error message

function required(value: string): string | null {
  return value.trim() ? null : 'Trường này không được để trống';
}

function minLength(min: number): Validator<string> {
  return (value) => value.length >= min ? null : `Tối thiểu ${min} ký tự`;
}

function maxLength(max: number): Validator<string> {
  return (value) => value.length <= max ? null : `Tối đa ${max} ký tự`;
}

function pattern(regex: RegExp, message: string): Validator<string> {
  return (value) => regex.test(value) ? null : message;
}

function compose<T>(...validators: Array<Validator<T>>): Validator<T> {
  return (value) => {
    for (const validate of validators) {
      const error = validate(value);
      if (error) return error;
    }
    return null;
  };
}

const validateEmail = compose(
  required,
  minLength(5),
  pattern(/^[^\s@]+@[^\s@]+\.[^\s@]+$/, 'Email không hợp lệ'),
);

const validatePassword = compose(
  required,
  minLength(8),
  maxLength(128),
  pattern(/[A-Z]/, 'Phải có ít nhất 1 chữ hoa'),
  pattern(/[0-9]/, 'Phải có ít nhất 1 chữ số'),
);

console.log(validateEmail('alice@example.com')); // null (valid)
console.log(validateEmail('not-email'));          // "Email không hợp lệ"
console.log(validatePassword('weak'));            // "Tối thiểu 8 ký tự"
```

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Pitfall 1: Overload implementation không visible với callers
> Implementation signature (signature cuối có body) **không phải** một overload — callers không thể dùng nó trực tiếp. Nếu muốn accept `string | number`, phải thêm overload signature cho union đó.

> [!warning] Pitfall 2: Optional parameter (`?`) vs Default parameter
> `name?: string` → type là `string | undefined`. `name: string = ''` → type là `string` (default xử lý `undefined` trước khi vào function body). Prefer default param khi có giá trị mặc định rõ ràng — code xử lý gọn hơn.

> [!tip] Infer return type khi có thể — chỉ annotate khi cần
> TS infer return type rất tốt. Chỉ explicit annotate khi: (1) complex function với nhiều paths, (2) public API của library cần stable return type, (3) muốn function có return type hẹp hơn inference.

---

## 5. Câu hỏi phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Optional `?` và default parameter khác nhau thế nào?"
> *"Optional parameter viết với dấu hỏi, ví dụ name hỏi chấm string, nghĩa là có thể không truyền; nhưng bên trong hàm kiểu của nó là string hoặc undefined nên em phải kiểm tra trước khi dùng. Default parameter viết với dấu bằng và một giá trị, ví dụ name bằng chuỗi rỗng, cũng cho phép không truyền, nhưng khi không truyền hoặc truyền undefined thì nó nhận giá trị mặc định, nên bên trong hàm kiểu luôn là string đã xử lý undefined. Em thường ưu tiên default parameter vì khỏi phải check undefined, code gọn và an toàn hơn. Ngoài ra TypeScript còn có function overload để một hàm có nhiều chữ ký tùy kiểu đầu vào."*

> [!note] 🧠 Mẹo nhớ
> **Optional `name?` → bên trong là `T | undefined` (phải check). Default `name = x` → luôn có giá trị (khỏi check) → ưu tiên dùng.** `void` = không trả gì; `never` = không bao giờ trả.

**Q1: Optional parameter `?` và default parameter khác nhau thế nào?**

> **Optional** `name?: string`: param có thể không truyền; bên trong function, type là `string | undefined` — phải check null/undefined trước khi dùng. **Default** `name: string = ''`: param có thể không truyền; nếu không truyền hoặc truyền `undefined`, value = `''`; bên trong function, type là `string` (đã handle undefined). Prefer default param vì không phải check undefined.

**Q2: Function overload dùng khi nào? Cấu trúc thế nào?**

> Dùng khi function có behavior **khác nhau dựa trên type của input** — ví dụ: `createElement('input')` trả về `HTMLInputElement`, `createElement('button')` trả về `HTMLButtonElement`. Cấu trúc: (1) viết **overload signatures** (không có body, chỉ có type); (2) viết **implementation signature** (có body, compatible với tất cả overloads). Callers chỉ thấy overload signatures, không thấy implementation signature.

**Q3: `void` và `never` return type khác nhau thế nào?**

> **`void`**: hàm kết thúc bình thường nhưng không return giá trị có nghĩa (`return;` hoặc không return). Thường dùng cho event handlers, side-effect functions. **`never`**: hàm **không bao giờ return** — luôn throw, hoặc infinite loop. `never` là subtype của mọi type; không có value nào có type `never`.

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Viết overloaded function `format(value: Date): string` và `format(value: number, currency: string): string` — format ngày hoặc currency. Implementation dùng `Intl.DateTimeFormat` và `Intl.NumberFormat`.

- [ ] **Bài 2:** Viết generic `memoize<T extends (...args: any[]) => any>(fn: T): T` — cache kết quả của function dựa trên arguments. Đảm bảo return type của memoized function giống fn gốc.

---

## 7. Liên kết

- [[02-Basic-Types]] — `void`, `never`, primitive types trong function signatures
- [[03-Type-Combination-Union-Intersection]] — Union types trong function params/return
- [[07-Generics]] — Generic functions `<T>` mở rộng function typing
- [[06-Classes-va-Interface]] — Method signatures trong interface và class
