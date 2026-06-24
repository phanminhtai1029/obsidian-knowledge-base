---
title: "TypeScript Decorators"
section: 05-TypeScript
tags: [typescript, decorator, metadata, class, method, property, fresher, frontend]
related:
  - "[[06-Classes-va-Interface]]"
  - "[[07-Generics]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 45m
source: [TypeScript Handbook, typescriptlang.org/docs/handbook/decorators.html]
---

# TypeScript Decorators

> [!summary] TL;DR
> **Decorator** là hàm đặc biệt được gọi để *annotate* hoặc *modify* class, method, property, hoặc parameter. Cú pháp: `@decorator` đặt trước target. Có **2 spec**: **Legacy decorators** (`experimentalDecorators: true`, dùng trong Angular/NestJS) và **ECMAScript Stage 3** (TS 5.0+, không cần flag). Decorator factory cho phép truyền config: `@log({ level: 'verbose' })`. Thứ tự áp dụng: class decorators last, method/property decorators bottom-up.

> [!tip] 🎯 Hiểu trong 30 giây
> **Decorator = một cái "nhãn dán" `@tên` đặt ngay trên class/method/property để *thêm hành vi* cho nó mà không sửa code bên trong.** Thực chất `@log` là *một hàm* chạy lúc class được định nghĩa, nhận target rồi bọc/ghi chú thêm.
> - Ví von: như **dán tem "dễ vỡ", "ưu tiên" lên kiện hàng** — bản thân kiện hàng không đổi, nhưng hệ thống xử lý nó khác đi.
> - Hay gặp ở framework: NestJS/Angular dùng `@Injectable`, `@Controller`; TypeORM dùng `@Entity`, `@Column`. Fresher chỉ cần hiểu *nó là hàm chạy lúc khai báo để thêm logic (log, validate, đánh dấu để inject...)* và **phải bật cấu hình** mới dùng được (`experimentalDecorators` cho bản cũ).

---

## 1. Khái niệm

### Decorator Pattern là gì?

**Decorator** là hàm nhận target (class, method, v.v.) và có thể:
- **Observe** — đọc metadata, logging
- **Modify** — thay đổi behavior, add functionality
- **Replace** — thay thế target hoàn toàn

```
★ Insight ─────────────────────────────────────
• Decorator chạy lúc ĐỊNH NGHĨA class (khi file load), KHÔNG phải lúc gọi method.
  Nó "gói" hoặc đăng ký target một lần; phần logic chạy mỗi lần gọi là code BÊN
  TRONG wrapper mà decorator gắn vào. Hiểu mốc thời gian này thì @Get/@Entity/
  @Injectable hết ma thuật — chúng chỉ ghi metadata lúc khởi động để framework đọc.
• Đây chính là nền của "magic" trong NestJS/Angular/TypeORM mà bạn gặp ở backend:
  @Controller/@Get gom route, @Injectable đánh dấu cho DI, @Entity/@Column map
  ORM — liên hệ trực tiếp với phần FastAPI/DI bên Backend. Phân biệt @log (plain)
  vs @retry(3) (factory: hàm TRẢ VỀ decorator để nhận tham số) là câu hỏi hay gặp.
─────────────────────────────────────────────────
```

```typescript
// Decorator cơ bản — function nhận target
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class BankAccount { /* ... */ }
// Object.seal ngăn thêm properties mới
```

### Legacy vs Stage 3

| | Legacy (`experimentalDecorators`) | Stage 3 (TS 5.0+) |
|---|---|---|
| tsconfig | `experimentalDecorators: true` | Không cần flag |
| Dùng trong | Angular, NestJS, TypeORM | Code mới, React tooling |
| Method decorator args | (target, key, descriptor) | (value, context) |
| Class decorator return | void hoặc constructor | void hoặc class |

> [!warning] Hai spec không tương thích
> Legacy decorators và Stage 3 có API khác nhau hoàn toàn. Khi dùng Angular/NestJS/TypeORM → bắt buộc dùng legacy. Code mới có thể dùng Stage 3. Note này tập trung **legacy** vì phổ biến hơn trong fresher interviews.

---

## 2. Cú pháp / API (Legacy Decorators)

### 2.1 Bật Legacy Decorators trong tsconfig

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

### 2.2 Class Decorator

```typescript
// Class decorator — nhận constructor function
function Injectable(constructor: Function) {
  console.log(`Registering ${constructor.name} for DI`);
}

function Singleton<T extends new (...args: any[]) => {}>(constructor: T) {
  let instance: InstanceType<T>;
  return class extends constructor {
    constructor(...args: any[]) {
      if (!instance) {
        instance = super(...args);
      }
      return instance;
    }
  };
}

// Decorator factory — trả về decorator (cho phép truyền params)
function Component(config: { selector: string; template: string }) {
  return function(constructor: Function) {
    constructor.prototype.__selector = config.selector;
    constructor.prototype.__template = config.template;
    console.log(`Component registered: ${config.selector}`);
  };
}

@Component({ selector: 'app-user', template: '<div>{{name}}</div>' })
class UserComponent {
  name = 'Alice';
}

@Singleton
class DatabaseConnection {
  private id = Math.random();
  getId() { return this.id; }
}

const db1 = new DatabaseConnection();
const db2 = new DatabaseConnection();
console.log(db1.getId() === db2.getId()); // true — singleton
```

### 2.3 Method Decorator

```typescript
// Method decorator — nhận (target, propertyKey, descriptor)
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  descriptor.value = function(...args: unknown[]) {
    console.log(`Calling ${propertyKey}(${args.join(', ')})`);
    const result = original.apply(this, args);
    console.log(`${propertyKey} returned:`, result);
    return result;
  };
  return descriptor;
}

function readonly(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  descriptor.writable = false;
  return descriptor;
}

// Decorator factory với config
function validate(schema: { [key: string]: (v: unknown) => boolean }) {
  return function(target: any, key: string, descriptor: PropertyDescriptor) {
    const original = descriptor.value;
    descriptor.value = function(...args: unknown[]) {
      Object.entries(schema).forEach(([paramName, validator], index) => {
        if (!validator(args[index])) {
          throw new Error(`Invalid argument '${paramName}' at position ${index}`);
        }
      });
      return original.apply(this, args);
    };
    return descriptor;
  };
}

function retry(times: number, delay: number = 0) {
  return function(target: any, key: string, descriptor: PropertyDescriptor) {
    const original = descriptor.value;
    descriptor.value = async function(...args: unknown[]) {
      for (let attempt = 0; attempt < times; attempt++) {
        try {
          return await original.apply(this, args);
        } catch (err) {
          if (attempt === times - 1) throw err;
          if (delay > 0) await new Promise(r => setTimeout(r, delay));
          console.warn(`Retry ${attempt + 1}/${times} for ${key}`);
        }
      }
    };
    return descriptor;
  };
}

class UserService {
  @log
  @retry(3, 500)
  async fetchUser(id: number): Promise<object> {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return res.json();
  }

  @readonly
  greet(name: string): string {
    return `Hello, ${name}!`;
  }

  @validate({ name: (v) => typeof v === 'string' && v.length > 0 })
  createUser(name: string): object {
    return { id: Date.now(), name };
  }
}
```

### 2.4 Property Decorator

```typescript
// Property decorator — nhận (target, propertyKey)
// KHÔNG nhận descriptor — dùng để metadata hoặc định nghĩa accessor
function minLength(min: number) {
  return function(target: object, propertyKey: string) {
    let value: string;

    Object.defineProperty(target, propertyKey, {
      get() { return value; },
      set(newVal: string) {
        if (newVal.length < min) {
          throw new Error(`${propertyKey} must be at least ${min} characters`);
        }
        value = newVal;
      },
      enumerable:   true,
      configurable: true,
    });
  };
}

function format(formatFn: (val: string) => string) {
  return function(target: object, propertyKey: string) {
    let value: string;
    Object.defineProperty(target, propertyKey, {
      get() { return value; },
      set(raw: string) { value = formatFn(raw); },
      enumerable:   true,
      configurable: true,
    });
  };
}

class User {
  @minLength(2)
  name: string = '';

  @format(s => s.trim().toLowerCase())
  email: string = '';
}

const user = new User();
user.email = '  Alice@Example.COM  ';
console.log(user.email); // "alice@example.com"

// user.name = 'A'; // Error: name must be at least 2 characters
```

### 2.5 Parameter Decorator

```typescript
// Parameter decorator — nhận (target, methodName, parameterIndex)
const REQUIRED_PARAMS = Symbol('required');

function required(target: object, methodName: string, paramIndex: number) {
  const existing: number[] = (Reflect as any).getMetadata(REQUIRED_PARAMS, target, methodName) ?? [];
  existing.push(paramIndex);
  (Reflect as any).defineMetadata(REQUIRED_PARAMS, existing, target, methodName);
}

// Thường kết hợp với emitDecoratorMetadata + reflect-metadata
// Phổ biến nhất trong NestJS:
// @Get('/users/:id')
// async getUser(@Param('id') id: string, @Body() dto: CreateUserDto) { ... }
```

### 2.6 Thứ tự áp dụng Decorators

```typescript
function outer(target: any, key: string, desc: PropertyDescriptor) {
  console.log('outer');
  return desc;
}

function inner(target: any, key: string, desc: PropertyDescriptor) {
  console.log('inner');
  return desc;
}

class Example {
  @outer      // evaluated second, applied second (after inner)
  @inner      // evaluated first, applied first
  method() {}
}
// Evaluation order: outer → inner (top to bottom)
// Application order: inner → outer (bottom to top — like function composition)
// Với nhiều decorators trên cùng 1 element: @outer @inner → inner(outer(element))
```

---

## 3. Ví dụ minh họa

### Ví dụ 1: NestJS-style controller decorators

```typescript
// Giả lập NestJS-style routing decorators
const routes: Array<{ method: string; path: string; handler: string; controller: any }> = [];

function Controller(basePath: string) {
  return function(constructor: Function) {
    constructor.prototype.__basePath = basePath;
  };
}

function Get(path: string) {
  return function(target: any, key: string) {
    routes.push({ method: 'GET', path, handler: key, controller: target.constructor });
  };
}

function Post(path: string) {
  return function(target: any, key: string) {
    routes.push({ method: 'POST', path, handler: key, controller: target.constructor });
  };
}

@Controller('/api/users')
class UserController {
  @Get('/')
  async listUsers() {
    return [{ id: 1, name: 'Alice' }];
  }

  @Get('/:id')
  async getUser() { /* id từ params */ }

  @Post('/')
  async createUser() { /* body từ request */ }
}

console.log(routes);
// [
//   { method: 'GET',  path: '/',    handler: 'listUsers',  ... },
//   { method: 'GET',  path: '/:id', handler: 'getUser',    ... },
//   { method: 'POST', path: '/',    handler: 'createUser', ... },
// ]
```

### Ví dụ 2: Memoize decorator

```typescript
function memoize(target: any, key: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  const cache    = new Map<string, unknown>();

  descriptor.value = function(...args: unknown[]) {
    const cacheKey = JSON.stringify(args);
    if (cache.has(cacheKey)) {
      console.log(`Cache hit for ${key}(${cacheKey})`);
      return cache.get(cacheKey);
    }
    const result = original.apply(this, args);
    cache.set(cacheKey, result);
    return result;
  };

  return descriptor;
}

class Calculator {
  @memoize
  fibonacci(n: number): number {
    if (n <= 1) return n;
    return this.fibonacci(n - 1) + this.fibonacci(n - 2);
  }

  @memoize
  factorial(n: number): number {
    if (n <= 1) return 1;
    return n * this.factorial(n - 1);
  }
}

const calc = new Calculator();
console.log(calc.fibonacci(10)); // 55 (computed)
console.log(calc.fibonacci(10)); // 55 (cache hit)
console.log(calc.factorial(5));  // 120
```

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Pitfall 1: Decorator execution order — dễ nhầm
> Decorators trên cùng 1 method: `@A @B method()` → evaluate A, B top-to-bottom; apply B trước rồi đến A (bottom-up). Class decorator chạy **sau** method/property decorators. Khi kết hợp nhiều decorators, test kỹ order.

> [!warning] Pitfall 2: Legacy vs Stage 3 — không tương thích
> Nếu migrate từ `experimentalDecorators` sang Stage 3, tất cả decorators cần rewrite. Angular đang ở legacy; NestJS hỗ trợ cả hai. Kiểm tra docs của framework trước khi chọn.

> [!tip] Decorators phổ biến trong ecosystem
> **Angular**: `@Component`, `@Injectable`, `@NgModule`, `@Input`, `@Output`. **NestJS**: `@Controller`, `@Get/@Post/@Put`, `@Injectable`, `@Body`, `@Param`. **TypeORM/Prisma**: `@Entity`, `@Column`, `@PrimaryGeneratedColumn`. **class-validator**: `@IsEmail()`, `@IsNotEmpty()`, `@MinLength()`.

---

## 5. Câu hỏi phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Decorator là gì, dùng để làm gì?"
> *"Decorator là một hàm đặc biệt được gọi lúc class, method hoặc property được định nghĩa, không phải lúc tạo instance. Mình đặt nó bằng cú pháp a còng tên decorator ngay trước mục tiêu, để thêm hoặc thay đổi hành vi mà không phải sửa trực tiếp code bên trong, giống như dán nhãn lên một đối tượng. Công dụng phổ biến gồm logging để bọc method ghi lại lời gọi, validation để kiểm tra tham số, đánh dấu metadata cho dependency injection như trong Angular hay NestJS, và mapping ORM như Entity hay Column trong TypeORM. Nó chính là pattern Decorator trong OOP. Lưu ý có hai spec, bản legacy cần bật experimentalDecorators còn bản chuẩn ECMAScript mới thì TypeScript 5 hỗ trợ sẵn."*

> [!note] 🧠 Mẹo nhớ
> **Decorator = nhãn `@tên` (một hàm) gắn lên class/method để thêm hành vi, không sửa ruột.** Hay thấy ở NestJS/Angular/TypeORM. Cần bật cấu hình (`experimentalDecorators` cho bản cũ).

**Q1: Decorator là gì trong TypeScript? Dùng để làm gì?**

> Decorator là **hàm đặc biệt** được gọi lúc class/method/property được định nghĩa (không phải lúc instantiate). Cú pháp `@decorator` trước target. Dùng để: (1) **Logging** — wrap method để log calls/returns. (2) **Validation** — validate params trước khi chạy. (3) **DI metadata** — Angular/NestJS đánh dấu class để inject. (4) **ORM mapping** — TypeORM `@Entity`, `@Column`. (5) **Caching** — memoize method. Pattern Decorator trong OOP.

**Q2: Class decorator và method decorator khác nhau thế nào?**

> **Class decorator**: nhận `constructor` function, return void hoặc new constructor. Chạy một lần khi class định nghĩa. Dùng để: modify prototype, register class, implement Singleton, add metadata. **Method decorator**: nhận `(target, propertyKey, descriptor)`, return descriptor (hoặc void). Dùng để: wrap method với pre/post logic, thay đổi behavior, enforce validation. Cả hai đều chạy lúc class load, không phải lúc runtime method call (ngoại trừ wrapper logic bên trong).

**Q3: Decorator factory là gì? Tại sao cần?**

> **Decorator factory** là hàm return decorator — cho phép truyền config vào decorator: `@retry(3)` thay vì `@retry`. Không có factory: `@log` — không có tham số. Có factory: `@log({ level: 'debug' })` — truyền config. Cú pháp: `function factory(config) { return function(target, key, desc) { /* dùng config */ }; }`.

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Viết class decorator `@Autobind` — tự động bind tất cả method instances (fix vấn đề mất `this` khi detach method). Dùng `Object.getOwnPropertyNames(prototype)` để lấy tất cả method names.

- [ ] **Bài 2:** Viết method decorator factory `@debounce(delay: number)` — wrap method với debounce logic (chỉ chạy sau `delay` ms kể từ lần gọi cuối). Dùng `setTimeout`/`clearTimeout`.

---

## 7. Liên kết

- [[06-Classes-va-Interface]] — Class TypeScript là target chính của decorators
- [[07-Generics]] — Decorator với generic types
- [[../03-Advanced-JavaScript/11-ES6-Class|ES6 Class]] — Class JavaScript base cho TS decorators
