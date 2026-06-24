---
title: "TypeScript: Union & Intersection Types"
section: 05-TypeScript
tags: [typescript, union, intersection, keyof, typeof, utility-types, fresher, frontend]
related:
  - "[[02-Basic-Types]]"
  - "[[04-Type-Guard]]"
  - "[[07-Generics]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [TypeScript Handbook, typescriptlang.org/docs/handbook/2/types-from-types.html]
---

# TypeScript: Union & Intersection Types

> [!summary] TL;DR
> **Union** (`A | B`) = value có thể là A **hoặc** B. **Intersection** (`A & B`) = value phải thỏa mãn **cả** A **và** B (merge types). **Literal types** giới hạn value về tập hữu hạn. **`keyof T`** lấy union of keys của type T. **`typeof`** ở type level lấy type của một value. **Utility types** (`Partial`, `Required`, `Pick`, `Omit`, `Record`…) là type transformations có sẵn — tránh viết lại boilerplate.

> [!tip] 🎯 Hiểu trong 30 giây
> Hai cách ghép kiểu, đọc đúng nghĩa của ký hiệu:
> - **Union `A | B`** = "**HOẶC**" — giá trị có thể là A *hoặc* B. Ví dụ `string | number` nhận cả chữ lẫn số; `'success' | 'error'` chỉ cho đúng 2 chuỗi đó (**literal type** = giới hạn vào tập giá trị cụ thể).
> - **Intersection `A & B`** = "**VÀ**" — giá trị phải có *cả* thuộc tính của A *lẫn* B (gộp 2 type lại).
> - Mẹo nhớ ngược trực giác: `|` (hoặc) cho ra type *rộng hơn* (nhiều khả năng); `&` (và) cho ra type *hẹp hơn* (đòi hỏi nhiều hơn).
>
> **Utility types** (`Partial<T>` cho tất cả thành tùy chọn, `Pick`/`Omit` chọn/bỏ field, `Record`...) là các "khuôn biến đổi type" có sẵn — đỡ viết lại bằng tay.

---

## 1. Khái niệm

### Union vs Intersection — biểu đồ Venn

```text
Union (A | B)          Intersection (A & B)
┌──────────────┐        ┌──────────────┐
│  A  │  B  │  │        │  A  ∩  B     │
│     │ or  │  │        │  (A AND B)   │
└──────────────┘        └──────────────┘
  value là A hoặc B       value là A VÀ B
  Type hẹp hơn            Type rộng hơn
  (ít properties hơn)     (nhiều properties hơn)
```

```
★ Insight ─────────────────────────────────────
• Nghịch lý "hoặc cho ít, và cho nhiều": Union (A|B) NGHE rộng nhưng dùng được
  ÍT property (chỉ phần CHUNG cả hai) → phải narrow mới chạm field riêng. Intersection
  (A&B) gộp property của cả hai nên dùng được NHIỀU. Nhớ theo "value vs property"
  thì hết lẫn. Và A&B với 2 primitive khác nhau (string & number) ra `never`.
• Discriminated union (mỗi nhánh có field hằng kind/type/status) là pattern xương
  sống của TS thực chiến: nó cho switch tự narrow + "exhaustive check" (thiếu case
  là compiler báo). Mô hình loading/success/error và Redux action đều dựa vào nó —
  xem [[04-Type-Guard]]. Utility types (Partial/Omit/Pick/Record…) là CÓ SẴN, đừng viết lại.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp / API

### 2.1 Union Types

```typescript
// Union với primitive
type StringOrNumber = string | number;
let id: StringOrNumber = 'abc'; // OK
id = 42;                        // OK
// id = true;                   // Error

// Union với literal types — rất phổ biến
type Status  = 'pending' | 'active' | 'inactive';
type Method  = 'GET' | 'POST' | 'PUT' | 'DELETE';
type Size    = 'sm' | 'md' | 'lg' | 'xl';

// Union với object types
interface Dog  { species: 'dog';  breed: string }
interface Cat  { species: 'cat';  indoor: boolean }
type Pet = Dog | Cat;

function describe(pet: Pet): string {
  // Cần narrow trước khi access field riêng
  if (pet.species === 'dog') {
    return `Dog: ${pet.breed}`; // TS biết đây là Dog
  }
  return `Cat: ${pet.indoor ? 'indoor' : 'outdoor'}`;
}

// Array với mixed types
type Mixed = Array<string | number>;
const arr: Mixed = ['hello', 42, 'world', 100];
```

### 2.2 Intersection Types

```typescript
// Intersection — merge multiple types
interface HasId    { id: number }
interface HasName  { name: string }
interface HasEmail { email: string }

type User = HasId & HasName & HasEmail;
// tương đương: { id: number; name: string; email: string }

const user: User = { id: 1, name: 'Alice', email: 'alice@example.com' };

// Thực tế: Intersection thường dùng để compose mixins
interface Serializable {
  serialize(): string;
  deserialize(data: string): void;
}

interface Timestamped {
  createdAt: Date;
  updatedAt: Date;
}

type EntityBase = HasId & Timestamped;
type Document   = EntityBase & Serializable & { content: string };
```

### 2.3 Literal Types và Discriminated Unions

```typescript
// Discriminated Union — union có "discriminant" field
type Circle    = { kind: 'circle';    radius: number }
type Rectangle = { kind: 'rectangle'; width: number; height: number }
type Triangle  = { kind: 'triangle';  base: number;  height: number }
type Shape = Circle | Rectangle | Triangle;

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':    return Math.PI * shape.radius ** 2;
    case 'rectangle': return shape.width * shape.height;
    case 'triangle':  return 0.5 * shape.base * shape.height;
    // default: never — exhaustive check
  }
}

// Template literal types (TS 4.1+)
type EventName = `on${Capitalize<string>}`; // "onClick", "onChange"...
type CSSProperty = `${string}-${string}`;   // "font-size", "margin-top"...

type UserEventMap = {
  [K in 'click' | 'focus' | 'blur']: `on${Capitalize<K>}`;
};
// { click: 'onClick'; focus: 'onFocus'; blur: 'onBlur' }
```

### 2.4 `keyof` và `typeof` ở type level

```typescript
interface Config {
  host: string;
  port: number;
  ssl:  boolean;
}

// keyof — lấy union of keys
type ConfigKey = keyof Config; // 'host' | 'port' | 'ssl'

function getConfigValue(config: Config, key: ConfigKey): Config[ConfigKey] {
  return config[key]; // type-safe property access
}

const cfg: Config = { host: 'localhost', port: 3000, ssl: false };
const host = getConfigValue(cfg, 'host');   // type: string
const port = getConfigValue(cfg, 'port');   // type: number
// getConfigValue(cfg, 'timeout');           // Error: 'timeout' not in Config

// typeof — lấy type từ value (type-level operation)
const palette = {
  primary:   '#3498db',
  secondary: '#2ecc71',
  danger:    '#e74c3c',
} as const;

type ColorKey   = keyof typeof palette;  // 'primary' | 'secondary' | 'danger'
type ColorValue = typeof palette[ColorKey]; // '#3498db' | '#2ecc71' | '#e74c3c'

// Kết hợp keyof + typeof để tạo type từ runtime value
const ROUTES = {
  home:    '/',
  about:   '/about',
  contact: '/contact',
} as const;

type Route     = typeof ROUTES[keyof typeof ROUTES]; // '/' | '/about' | '/contact'
type RouteName = keyof typeof ROUTES;                // 'home' | 'about' | 'contact'
```

### 2.5 Utility Types

```typescript
interface User {
  id:        number;
  name:      string;
  email:     string;
  password:  string;
  createdAt: Date;
}

// Partial<T> — tất cả properties trở thành optional
type UserUpdate = Partial<User>;
// { id?: number; name?: string; email?: string; ... }

// Required<T> — tất cả properties trở thành required
type StrictUser = Required<User>; // ngược Partial

// Readonly<T> — tất cả properties trở thành readonly
type FrozenUser = Readonly<User>;
// const frozen: FrozenUser = { ... };
// frozen.name = 'Bob'; // Error

// Pick<T, K> — chỉ lấy một số properties
type PublicUser = Pick<User, 'id' | 'name' | 'email'>;
// { id: number; name: string; email: string }

// Omit<T, K> — loại bỏ một số properties
type SafeUser = Omit<User, 'password'>;
// { id: number; name: string; email: string; createdAt: Date }

// Record<K, V> — object với keys là K, values là V
type UserMap   = Record<string, User>;
type StatusMap = Record<'active' | 'inactive', number>;
// { active: number; inactive: number }

// Exclude<T, U> — loại trừ từ union
type NumericId = Exclude<string | number | boolean, string | boolean>; // number

// Extract<T, U> — giữ lại từ union
type StringTypes = Extract<string | number | null | undefined, string | null>;
// string | null

// NonNullable<T> — loại bỏ null và undefined
type SafeString = NonNullable<string | null | undefined>; // string

// ReturnType<T> — lấy return type của function
function createUser(name: string, email: string) {
  return { id: Date.now(), name, email, createdAt: new Date() };
}
type CreatedUser = ReturnType<typeof createUser>;
// { id: number; name: string; email: string; createdAt: Date }

// Parameters<T> — lấy params type của function
type CreateParams = Parameters<typeof createUser>; // [name: string, email: string]
```

---

## 3. Ví dụ minh họa

### Ví dụ 1: API Error Handling với Union Types

```typescript
type ApiSuccess<T> = {
  status: 'success';
  data:   T;
};

type ApiError = {
  status:  'error';
  code:    number;
  message: string;
};

type ApiResult<T> = ApiSuccess<T> | ApiError;

interface UserData { id: number; name: string; email: string }

async function fetchUser(id: number): Promise<ApiResult<UserData>> {
  try {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) {
      return { status: 'error', code: res.status, message: res.statusText };
    }
    const data = await res.json() as UserData;
    return { status: 'success', data };
  } catch {
    return { status: 'error', code: 0, message: 'Network error' };
  }
}

async function displayUser(id: number) {
  const result = await fetchUser(id);

  if (result.status === 'error') {
    console.error(`[${result.code}] ${result.message}`);
    return;
  }

  // TS biết result là ApiSuccess<UserData> ở đây
  console.log(result.data.name.toUpperCase());
}
```

### Ví dụ 2: Type-safe config builder với Partial + Required

```typescript
interface AppConfig {
  apiUrl:    string;
  timeout:   number;
  retries:   number;
  debug:     boolean;
  logLevel:  'error' | 'warn' | 'info' | 'debug';
}

const DEFAULTS: Required<AppConfig> = {
  apiUrl:   'https://api.example.com',
  timeout:  5000,
  retries:  3,
  debug:    false,
  logLevel: 'warn',
};

function createConfig(overrides: Partial<AppConfig> = {}): AppConfig {
  return { ...DEFAULTS, ...overrides };
}

// Development config
const devConfig = createConfig({
  apiUrl:   'http://localhost:3000',
  debug:    true,
  logLevel: 'debug',
});

// Production config — dùng defaults
const prodConfig = createConfig();

console.log(devConfig.retries);  // 3 (từ default)
console.log(devConfig.debug);    // true (override)
```

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Pitfall 1: Union type — phải narrow trước khi dùng field riêng
> ```typescript
> type AB = { a: string } | { b: number };
> function fn(x: AB) {
>   // x.a; // Error — b có thể không có 'a'
>   if ('a' in x) { x.a; } // OK sau khi narrow
> }
> ```
> TS không cho phép access property chỉ có trong 1 member của union — phải dùng type guard.

> [!warning] Pitfall 2: Intersection không luôn có nghĩa với primitive
> `type Impossible = string & number` → type `never` vì không có value nào vừa là string vừa là number. Intersection với primitives thường cho ra `never`. Intersection hữu ích với object types.

> [!tip] Utility types là built-in — không cần tự viết
> TypeScript đã có sẵn: `Partial`, `Required`, `Readonly`, `Pick`, `Omit`, `Record`, `Exclude`, `Extract`, `NonNullable`, `ReturnType`, `Parameters`, `ConstructorParameters`, `InstanceType`. Tra `typescriptlang.org/docs/handbook/utility-types.html` để xem đầy đủ.

---

## 5. Câu hỏi phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Union vs Intersection?"
> *"Union viết bằng dấu gạch đứng, nghĩa là hoặc: giá trị có thể là một trong các kiểu liệt kê, ví dụ string hoặc number, hay một literal union như success hoặc error để giới hạn giá trị. Em dùng union khi hàm nhận nhiều loại đầu vào, hoặc khi mô hình hóa các trạng thái. Intersection viết bằng dấu và, nghĩa là phải thỏa tất cả: nó gộp các type lại, giá trị phải có đủ thuộc tính của mọi type thành phần, hay dùng khi compose nhiều type hoặc thêm thuộc tính. Một điểm hay nhầm là union cho ra kiểu rộng hơn vì nhiều khả năng, còn intersection cho ra kiểu hẹp hơn vì đòi hỏi nhiều hơn."*

> [!note] 🧠 Mẹo nhớ
> **`|` = HOẶC (rộng hơn) · `&` = VÀ (hẹp hơn).** Literal union (`'a' | 'b'`) để giới hạn giá trị. Utility types (`Partial`/`Pick`/`Omit`) = khuôn biến đổi type có sẵn.

**Q1: Union và Intersection types — phân biệt và khi nào dùng?**

> **Union** `A | B`: value có thể là A **hoặc** B. Dùng khi: function accept nhiều loại input, API response có thể thành công hoặc lỗi, discriminated unions cho state. **Intersection** `A & B`: value phải thỏa **cả** A **và** B — merge types. Dùng khi: compose mixins, extend type với thêm properties, combine required roles/permissions.

**Q2: `keyof` và `typeof` ở type level là gì?**

> **`keyof T`** (type operator): trả về union of all keys của type T — `keyof { a: string; b: number }` = `'a' | 'b'`. Dùng để viết generic functions type-safe. **`typeof x`** (type-level): trả về type của value `x` — không phải JS `typeof` (trả về string). Kết hợp: `keyof typeof obj` = union of keys từ runtime object.

**Q3: Kể 5 utility types và use case của từng cái?**

> **`Partial<T>`**: tất cả props optional — dùng cho partial update (PATCH requests). **`Omit<T, K>`**: loại bỏ props nhạy cảm — `Omit<User, 'password'>` cho public API. **`Pick<T, K>`**: chỉ lấy props cần — `Pick<Config, 'host' | 'port'>`. **`Record<K, V>`**: typed dictionary — `Record<UserId, User>`. **`ReturnType<F>`**: lấy return type của function — tránh duplicate type definition.

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Tạo discriminated union `Notification` có 3 variant: `'email'` (to, subject, body), `'sms'` (phone, message), `'push'` (deviceId, title, payload). Viết function `sendNotification(n: Notification): Promise<void>` với switch exhaustive.

- [ ] **Bài 2:** Viết generic function `pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K>` — trả về object chỉ với các keys được chỉ định. Test với `pick(user, ['name', 'email'])`.

---

## 7. Liên kết

- [[02-Basic-Types]] — Literal types, enum, primitives
- [[04-Type-Guard]] — Narrowing union types về specific type
- [[07-Generics]] — `keyof`, `typeof`, mapped types trong generics
- [[06-Classes-va-Interface]] — Interface extension vs Intersection types
