---
title: "ES6: Template Literals"
section: 03-Advanced-JavaScript
tags: [es6, template-literal, string, fresher, frontend]
related:
  - "[[07-ES6-Enhanced-Object-Literals]]"
  - "[[08-ES6-Destructuring]]"
difficulty: ⭐
estimated_time: 20m
source: [MDN]
---

# ES6: Template Literals

> [!summary] TL;DR
> Template literals dùng backtick (`` ` ``) thay dấu nháy. Hỗ trợ **string interpolation** (`${expression}`), **multiline strings** (không cần `\n`), và **tagged templates** (hàm xử lý string — nền tảng của styled-components, gql). Biểu thức trong `${}` có thể là bất kỳ JS expression nào.

---

## 1. Khái niệm

### Trước ES6 — string concatenation

```javascript
const name = 'Alice';
const age  = 25;

// Trước ES6 — verbose, dễ sai
const msg = 'Xin chào ' + name + ', bạn ' + age + ' tuổi.';
const multiline = 'Dòng 1\n' +
                  'Dòng 2\n' +
                  'Dòng 3';
```

### Với Template Literals

```javascript
const name = 'Alice';
const age  = 25;

const msg       = `Xin chào ${name}, bạn ${age} tuổi.`;
const multiline = `Dòng 1
Dòng 2
Dòng 3`;
```

---

## 2. Cú pháp

### 2.1 String Interpolation — `${expression}`

```javascript
const product = 'Laptop';
const price   = 15000000;
const qty     = 2;

// Biểu thức đơn giản
console.log(`Sản phẩm: ${product}`);
console.log(`Giá: ${price.toLocaleString('vi-VN')}₫`);

// Biểu thức tính toán
console.log(`Tổng: ${(price * qty).toLocaleString('vi-VN')}₫`);

// Gọi function
const upper = str => str.toUpperCase();
console.log(`Tên: ${upper(product)}`);   // "Tên: LAPTOP"

// Ternary
const inStock = true;
console.log(`Trạng thái: ${inStock ? 'Còn hàng' : 'Hết hàng'}`);
```

### 2.2 Multiline Strings

```javascript
// Multiline HTML template (dễ đọc)
function createCard(name, role, avatar) {
  return `
    <div class="card">
      <img src="${avatar}" alt="${name}">
      <h3>${name}</h3>
      <p>${role}</p>
    </div>
  `.trim();
}

// Note: nhớ dùng textContent khi insert vào DOM — chỉ dùng template literal
// để BUILD HTML string với data TIN CẬY (không phải user input)
const card = createCard('Alice', 'Frontend Dev', '/images/alice.jpg');
```

### 2.3 Nested Template Literals

```javascript
const users = [
  { name: 'Alice', active: true },
  { name: 'Bob',   active: false },
  { name: 'Carol', active: true },
];

const userList = `
Danh sách người dùng:
${users.map((u, i) => `  ${i + 1}. ${u.name} — ${u.active ? 'Hoạt động' : 'Ngừng'}`).join('\n')}
Tổng: ${users.length} người
`;

console.log(userList);
```

### 2.4 Tagged Templates

```javascript
// Tagged template: hàm xử lý string trước khi trả kết quả
function highlight(strings, ...values) {
  // strings: mảng các phần string thuần
  // values:  mảng các giá trị interpolated
  return strings.reduce((result, str, i) => {
    const val = values[i - 1];
    return result + (val !== undefined ? `[${val}]` : '') + str;
  });
}

const name  = 'Alice';
const score = 95;
const msg   = highlight`Học sinh ${name} đạt ${score} điểm.`;
console.log(msg); // "Học sinh [Alice] đạt [95] điểm."

// Ứng dụng thực tế: styled-components, GraphQL, i18n
// css`color: red; font-size: ${size}px`
// gql`query { user(id: ${id}) { name } }`
```

### 2.5 Raw strings — `String.raw`

```javascript
// String.raw — không xử lý escape sequences
const path    = String.raw`C:\Users\Alice\Documents`;
const pattern = String.raw`^\d{4}-\d{2}-\d{2}$`;

console.log(path);    // "C:\Users\Alice\Documents" (không chuyển \U, \D)
console.log(pattern); // "^\d{4}-\d{2}-\d{2}$" (không chuyển \d)

// Bình thường:
console.log(`C:\Users\Alice`); // "C:" + newline + "sers" + ... (do \U không có escape)
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: Build SQL-like query string (demo tagged template)

```javascript
function sqlSafe(strings, ...values) {
  const escaped = values.map(v =>
    typeof v === 'string'
      ? v.replace(/'/g, "''")   // escape single quotes
      : v
  );
  return strings.reduce((acc, str, i) => acc + (escaped[i - 1] ?? '') + str);
}

const table = 'users';
const name  = "Alice's";

const query = sqlSafe`SELECT * FROM ${table} WHERE name = '${name}'`;
console.log(query);
// "SELECT * FROM users WHERE name = 'Alice''s'"
// (escaped single quote — SQL safe)
```

### Ví dụ 2: Logging utility với template

```javascript
const LOG_LEVELS = { INFO: 'INFO', WARN: 'WARN', ERROR: 'ERROR' };

function createLogger(prefix) {
  return {
    info(msg, ...data)  { console.log(`[${prefix}][${LOG_LEVELS.INFO}]  ${msg}`, ...data); },
    warn(msg, ...data)  { console.warn(`[${prefix}][${LOG_LEVELS.WARN}]  ${msg}`, ...data); },
    error(msg, ...data) { console.error(`[${prefix}][${LOG_LEVELS.ERROR}] ${msg}`, ...data); },
  };
}

const log = createLogger('App');
log.info('Server started on port 3000');
log.warn('Retry attempt 2/3', { endpoint: '/api/data' });
log.error('Connection failed', new Error('Timeout'));
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: Dùng template literal với user input → XSS khi insert vào DOM
> Template literal tạo string — nếu dùng để build HTML rồi set vào DOM bằng phương pháp không an toàn, vẫn có nguy cơ XSS. Chỉ dùng template literal để build HTML với dữ liệu **tin cậy** (constant, server-validated). Với user input: dùng `textContent` + `createElement`.

> [!warning] Pitfall 2: Leading whitespace trong multiline template
> Multiline template giữ nguyên indent. String bắt đầu ngay sau backtick → thêm newline và spaces không mong muốn. Dùng `.trim()` để xóa leading/trailing whitespace.

> [!tip] Template literals hỗ trợ bất kỳ expression nào trong `${}`
> Có thể dùng ternary, function call, array methods, thậm chí nested template. Nhưng giữ expression đơn giản trong `${}` — logic phức tạp nên extract ra biến riêng cho dễ đọc.

---

## 5. Phỏng vấn thường gặp

**Q1: Template literal là gì? Ưu điểm so với string concatenation?**

> Template literal dùng backtick, hỗ trợ interpolation `${}`, multiline tự nhiên, và tagged templates. Ưu điểm: code ngắn hơn, dễ đọc hơn, ít lỗi hơn (không quên space, không nhầm dấu ngoặc). Biểu thức trong `${}` được evaluate và convert sang string tự động.

**Q2: Tagged template là gì? Cho ví dụ ứng dụng?**

> Tagged template là hàm đứng trước backtick: `fn\`...\``. Hàm nhận mảng string thuần và các interpolated values, có thể xử lý tùy ý. Ứng dụng: styled-components (CSS-in-JS), GraphQL (gql\`query {...}\`), i18n translation, SQL escaping, syntax highlighting.

**Q3: Phân biệt template literal và string thông thường?**

> String thông thường (`'...'` hay `"..."`) là static. Template literal (`` `...` ``) hỗ trợ multiline tự nhiên và interpolation. Cả hai đều tạo primitive `string` value. Performance tương đương — JS engine optimize template literals.

---

## 6. Bài tập thực hành

- [ ] **Bài 1:** Viết function `formatInvoice(items)` nhận mảng `[{name, qty, price}]` và trả về chuỗi invoice đẹp dùng template literal multiline, bao gồm header, từng dòng item, và tổng cộng.

- [ ] **Bài 2:** Viết tagged template function `currency` biến số thành chuỗi tiền tệ: `` currency`Giá: ${15000000}` `` → `"Giá: 15.000.000₫"`.

---

## 7. Liên kết

- [[07-ES6-Enhanced-Object-Literals]] — Shorthand methods hay dùng với template
- [[08-ES6-Destructuring]] — Kết hợp destructuring + template literal
- [[09-ES6-Arrow-Function]] — Arrow function trong template expression
