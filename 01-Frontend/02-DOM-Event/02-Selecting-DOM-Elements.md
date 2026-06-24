---
title: "Selecting DOM Elements"
section: 02-DOM-Event
tags: [dom, query, selector, fresher, frontend]
related:
  - "[[01-DOM-Tree-va-Node-Types]]"
  - "[[03-Traversing-DOM]]"
difficulty: ⭐⭐
estimated_time: 30m
source: [Javascript.docx]
---

# Selecting DOM Elements

> [!summary] TL;DR
> 5 method chính: `getElementById` (1 kết quả, theo ID), `getElementsByClassName`/`getElementsByTagName` (HTMLCollection live), `querySelector` (first match, CSS selector), `querySelectorAll` (static NodeList). Ưu tiên `querySelector`/`querySelectorAll` vì linh hoạt và nhất quán.

> [!tip] 🎯 Hiểu trong 30 giây
> Muốn JS sửa một phần tử thì trước hết phải **"tóm" được nó** từ cây DOM. Cách dễ nhớ nhất: **`querySelector('...')`** — dùng *đúng cú pháp CSS selector* bạn đã quen (`#id`, `.class`, `div p`), trả về **phần tử đầu tiên** khớp; còn **`querySelectorAll`** trả về **mọi** phần tử khớp. Các hàm cũ như `getElementById` vẫn chạy nhưng mỗi hàm một kiểu cú pháp; `querySelector` gộp tất cả vào một cách viết → cứ ưu tiên nó.
> *(Lưu ý nhỏ ra thi: `getElementsByClassName` trả về danh sách "sống" — live, tự cập nhật khi DOM đổi; `querySelectorAll` trả về danh sách "tĩnh" — chụp tại thời điểm gọi.)*

---

## 1. Khái niệm

Trước khi thay đổi DOM, bạn cần **chọn** element. JavaScript cung cấp nhiều method — mỗi cái có use case khác nhau.

| Method | Trả về | Input | Ghi chú |
|--------|--------|-------|---------|
| `getElementById(id)` | Element \| null | ID (không có #) | Nhanh nhất |
| `getElementsByClassName(cls)` | HTMLCollection (live) | Class (không có .) | Tự cập nhật khi DOM đổi |
| `getElementsByTagName(tag)` | HTMLCollection (live) | Tag name | Live |
| `querySelector(sel)` | Element \| null | CSS selector | **First match** |
| `querySelectorAll(sel)` | NodeList (static) | CSS selector | Snapshot tại thời điểm gọi |

> [!tip] HTMLCollection vs NodeList
> **HTMLCollection** (từ `getElementsBy*`) là **live** — phản ánh DOM hiện tại, nhưng không có `.forEach()` trực tiếp. **NodeList** (từ `querySelectorAll`) là **static** và có `.forEach()`. Trong thực tế, `querySelectorAll` linh hoạt hơn.

```
★ Insight ─────────────────────────────────────
• "Live" nghe tiện nhưng là cái bẫy kinh điển: lặp qua HTMLCollection live mà
  vừa xóa/thêm element thì collection TỰ ĐỔI ngay giữa vòng lặp → bỏ sót phần
  tử. NodeList static (querySelectorAll) chụp 1 ảnh cố định, lặp an toàn. Đây
  là lý do thực dụng để mặc định dùng querySelectorAll.
• Một selector duy nhất cho mọi nhu cầu: querySelector dùng CÚ PHÁP CSS giống hệt
  file .css (`#id`, `.class`, `ul > li`, `[type=email]`) → không phải nhớ 4 hàm
  getElementsBy* riêng. Học CSS selector ([[04-CSS-Selectors]]) một lần, dùng
  được cả cho styling lẫn cho JS.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp & Ví dụ

### 2.1 getElementById

```javascript
// Trả về element có id="mainHeader" (hoặc null)
const header = document.getElementById('mainHeader');
console.log(header.textContent); // "Company Dashboard"

// Nếu có 2 element cùng id — chỉ trả về cái đầu tiên
```

### 2.2 getElementsByClassName

```javascript
// Trả về HTMLCollection — tất cả elements có class="product-info"
const products = document.getElementsByClassName('product-info');

// Phải truy cập từng item bằng index
console.log(products[0].textContent);

// Loop — cần Array.from() để có forEach
Array.from(products).forEach(el => console.log(el.textContent));
```

### 2.3 getElementsByTagName

```javascript
const paragraphs = document.getElementsByTagName('p');
for (let i = 0; i < paragraphs.length; i++) {
  console.log(paragraphs[i].textContent);
}
```

### 2.4 querySelector — first match

```javascript
const header   = document.querySelector('#dashboardHeader');  // by ID
const firstP   = document.querySelector('p');                 // first <p>
const hl       = document.querySelector('.highlight');        // first .highlight

// CSS selector phức tạp
const navLink  = document.querySelector('ul#nav > a.active'); // direct child
const emailIn  = document.querySelector('input[type="email"]'); // by attribute
```

### 2.5 querySelectorAll — all matches

```javascript
const highlights = document.querySelectorAll('.highlight');
highlights.forEach(el => console.log(el.textContent)); // có forEach trực tiếp

// Selector phức tạp
const emailInputs  = document.querySelectorAll('input[type="email"]');
const thirdItems   = document.querySelectorAll('ul li:nth-child(3n)');
const parasAfterH2 = document.querySelectorAll('h2 + p');
```

### 2.6 matches() — kiểm tra không select

```javascript
// Kiểm tra một element đã có có match selector không
const items = document.querySelectorAll('.content *');
items.forEach(el => {
  if (el.matches('.active')) {
    console.log('Active element:', el);
  }
});

// Thường dùng trong event delegation
document.addEventListener('click', e => {
  if (e.target.matches('button.btn-delete')) {
    // xử lý xóa
  }
});
```

---

## 3. CSS Selectors quan trọng với querySelector

| Selector | Cú pháp | Ví dụ |
|----------|---------|-------|
| Tag | `tag` | `'p'`, `'div'` |
| Class | `.class` | `'.active'` |
| ID | `#id` | `'#header'` |
| Child trực tiếp | `A > B` | `'ul > li'` |
| Descendant | `A B` | `'div p'` |
| Adjacent sibling | `A + B` | `'h2 + p'` |
| Attribute | `[attr=val]` | `'input[type="email"]'` |
| Nth child | `:nth-child(n)` | `'li:nth-child(3n)'` |
| First child | `:first-child` | `'section > p:first-child'` |

---

## 4. Ví dụ thực tế

### Ví dụ 1: Populate và query dashboard

```html
<!DOCTYPE html>
<html lang="vi">
<head><meta charset="UTF-8"><title>Dashboard</title></head>
<body>
  <h1 id="mainHeader">Company Dashboard</h1>
  <section class="stats">
    <p class="stat-item">Nhân viên: <span class="value">120</span></p>
    <p class="stat-item">Dự án: <span class="value">15</span></p>
    <p class="stat-item highlight">Doanh thu: <span class="value">5 tỷ</span></p>
  </section>
  <ul id="departments">
    <li>Kỹ thuật</li>
    <li>Marketing</li>
    <li>Kế toán</li>
  </ul>

  <script>
    // 1. Select by ID
    const header = document.getElementById('mainHeader');
    console.log('Header:', header.textContent);

    // 2. Tất cả stat values
    const values = document.querySelectorAll('.stat-item .value');
    values.forEach((el, i) => console.log(`Stat ${i + 1}:`, el.textContent));

    // 3. Highlight item đặc biệt
    const highlighted = document.querySelector('.highlight');
    highlighted.style.fontWeight = 'bold';
    highlighted.style.color = 'darkgreen';

    // 4. Departments list
    const depts = document.querySelectorAll('#departments li');
    const names = Array.from(depts).map(li => li.textContent);
    console.log('Departments:', names);

    // Lưu reference — tránh select nhiều lần
    const list = document.getElementById('departments');
    const newItem = document.createElement('li');
    newItem.textContent = 'HR';
    list.appendChild(newItem);
  </script>
</body>
</html>
```

### Ví dụ 2: getFormData helper

```javascript
// Lấy toàn bộ data từ form theo id
function getFormData(formId) {
  const form = document.getElementById(formId);
  const fields = form.querySelectorAll('input, select, textarea');
  const data = {};
  fields.forEach(field => {
    if (!field.name) return;
    if (field.type === 'checkbox') {
      data[field.name] = field.checked;
    } else if (field.type === 'radio') {
      if (field.checked) data[field.name] = field.value;
    } else {
      data[field.name] = field.value;
    }
  });
  return data;
}

console.log(getFormData('registerForm'));
// { name: "Alice", email: "alice@example.com", newsletter: true }
```

---

## 5. Pitfalls thường gặp

> [!warning] Pitfall 1: Quên `#` hoặc `.` trong querySelector
> ```javascript
> querySelector('header')   // thẻ <header> (tag selector)
> querySelector('#header')  // id="header"
> querySelector('.header')  // class="header"
>
> // getElementById KHÔNG dùng #:
> getElementById('header')  // đúng
> getElementById('#header') // SAI — trả về null
> ```

> [!warning] Pitfall 2: HTMLCollection không có `.forEach()`
> ```javascript
> // TypeError: HTMLCollection.forEach is not a function
> document.getElementsByClassName('item').forEach(el => ...);
>
> // Fix:
> Array.from(document.getElementsByClassName('item')).forEach(el => ...);
> // Hoặc đơn giản hơn:
> document.querySelectorAll('.item').forEach(el => ...);
> ```

> [!warning] Pitfall 3: Select nhiều lần trong loop — tốn tài nguyên
> ```javascript
> // Không tốt — select DOM mỗi iteration
> items.forEach(item => document.getElementById('list').appendChild(item));
>
> // Tốt — lưu reference trước
> const list = document.getElementById('list');
> items.forEach(item => list.appendChild(item));
> ```

---

## 6. Phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "`querySelector` khác `getElementById` thế nào?"
> *"`getElementById` chỉ tìm theo id, truyền tên id không cần dấu thăng, và nhanh hơn chút. `querySelector` thì dùng cú pháp CSS selector nên linh hoạt hơn nhiều, tìm được theo class, theo thuộc tính, theo quan hệ cha con, và trả về phần tử đầu tiên khớp; muốn lấy hết thì dùng querySelectorAll. Em ưu tiên querySelector để codebase nhất quán một kiểu cú pháp, kể cả khi tìm theo id thì viết querySelector thăng id. Lưu ý querySelectorAll trả về NodeList tĩnh có forEach, còn getElementsByClassName trả về HTMLCollection sống tự cập nhật nhưng không có forEach."*

> [!note] 🧠 Mẹo nhớ
> **`querySelector` = tìm bằng CSS selector (linh hoạt, ưu tiên dùng); `querySelectorAll` = lấy tất cả (NodeList tĩnh).** `getElementsByClassName` = danh sách *sống*.

**Q1: Sự khác nhau giữa `querySelector` và `getElementById`?**

> `getElementById` nhanh hơn, chỉ tìm theo ID (không cần `#`). `querySelector` dùng **CSS selector** — linh hoạt hơn, hỗ trợ class, attribute, combinator. Cho codebase nhất quán, nhiều team dùng `querySelector('#id')` thay vì `getElementById`.

**Q2: `querySelectorAll` trả về gì? Khác `getElementsByClassName` thế nào?**

> `querySelectorAll` trả về **static NodeList** (snapshot, có `.forEach()`). `getElementsByClassName` trả về **live HTMLCollection** (tự cập nhật khi DOM thay đổi, không có `.forEach()`). Dùng `querySelectorAll` cho nhất quán.

**Q3: Khi nào dùng `matches()`?**

> Khi đã có element (ví dụ `e.target`) và muốn kiểm tra nó có match selector không — không cần tìm element mới. Phổ biến trong event delegation: `if (e.target.matches('.btn-delete')) { ... }`.

---

## 7. Bài tập thực hành

**Bài 1:** Tạo trang product list (mỗi item có class `product`, giá có class `price`, nút xóa có class `btn-delete`). JS để: log tên sản phẩm, highlight giá > 100k, đếm tổng.

**Bài 2:** Viết `getFormData(formId)` như ví dụ 2 trên, xử lý đủ input/select/textarea/checkbox/radio.

---

## 8. Liên kết

- [[01-DOM-Tree-va-Node-Types]] — DOM tree, node types cơ bản
- [[03-Traversing-DOM]] — Duyệt parent/child sau khi select
- [[04-Modifying-DOM-Elements]] — Thay đổi element sau khi select
- [[06-Event-Handling]] — addEventListener kết hợp với selection
