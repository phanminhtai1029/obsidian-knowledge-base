---
title: "Modifying DOM Elements"
section: 02-DOM-Event
tags: [dom, modify, textContent, innerHTML, fresher, frontend]
related:
  - "[[02-Selecting-DOM-Elements]]"
  - "[[05-Creating-Adding-Managing-Nodes]]"
difficulty: ⭐⭐
estimated_time: 30m
source: [Javascript.docx]
---

# Modifying DOM Elements

> [!summary] TL;DR
> Sửa đổi DOM element qua: `textContent` (text an toàn), `innerHTML` (HTML tĩnh — không dùng với user input), `.value`/`.checked` (form inputs), `getAttribute`/`setAttribute` (attributes), `classList.add/remove/toggle` (classes), `element.style.propName` (inline CSS camelCase).

---

## 1. Khái niệm

Sau khi select element, có 5 nhóm thay đổi:

| Nhóm | Methods/Properties | Ghi chú |
|------|-------------------|---------|
| Nội dung text | `textContent`, `innerText`, `innerHTML` | Ưu tiên `textContent` |
| Giá trị form | `.value`, `.checked` | Input luôn là string |
| Attributes | `getAttribute`, `setAttribute` | `dataset` cho data-* |
| Classes | `classList.add/remove/toggle/contains` | Ưu tiên hơn `style` |
| Inline style | `element.style.propertyName` (camelCase) | Specificity cao nhất |

---

## 2. Cú pháp & Chi tiết

### 2.1 Text Content

```javascript
const el = document.getElementById('message');

// textContent: tất cả text (kể cả hidden), không parse HTML — AN TOÀN
el.textContent = 'Hello World';

// innerText: chỉ text hiển thị (theo layout) — chậm hơn vì trigger reflow
el.innerText = 'Line 1\nLine 2';

// innerHTML: đọc/ghi HTML tags
// Chỉ dùng với nội dung tĩnh, KHÔNG dùng với user input
el.innerHTML = '<strong>Bold text</strong>'; // nội dung tĩnh — OK
```

> [!warning] innerHTML và bảo mật
> Gán user input trực tiếp vào `innerHTML` có thể dẫn đến **XSS (Cross-Site Scripting)** — kẻ tấn công inject script độc hại vào trang. **Luôn dùng `textContent` cho text từ người dùng**. Với HTML động, dùng `createElement` + `textContent` hoặc thư viện sanitizer như DOMPurify.

### 2.2 Cách an toàn render user input có HTML

```javascript
// NGUY HIỂM — không làm thế này với user input
// el.innerHTML = `Xin chào ${userNameFromInput}!`;

// AN TOÀN — dùng textContent
const greeting = document.createElement('p');
greeting.textContent = `Xin chào, ${userNameFromInput}!`;
container.appendChild(greeting);

// Hoặc nếu chỉ update text node:
el.textContent = `Xin chào, ${userNameFromInput}!`;
```

### 2.3 Form Input Values

```javascript
// Text input — .value trả về string
const nameInput = document.getElementById('nameInput');
console.log(nameInput.value); // "Maaike" (string)
nameInput.value = 'John Doe';

// Number input — VẪN là string!
const amount = document.getElementById('amount');
const num = parseInt(amount.value, 10); // parse sang number

// SELECT dropdown
const dept = document.getElementById('deptSelect');
console.log(dept.value); // value của option đang chọn
dept.value = 'marketing'; // chuyển sang option có value="marketing"

// Radio button — cần .checked
const radios = document.querySelectorAll('input[name="status"]');
radios.forEach(r => { if (r.checked) console.log('Selected:', r.value); });

// Checkbox
const newsletter = document.getElementById('newsletter');
console.log(newsletter.checked); // true/false
newsletter.checked = true;
```

### 2.4 Attributes

```javascript
const img = document.getElementById('hero-img');

// Đọc/ghi attribute
const src = img.getAttribute('src');
img.setAttribute('src', 'new.jpg');
img.setAttribute('alt', 'Updated photo');

// Shortcut (direct property)
img.src = 'photo.jpg';
img.alt = 'Photo';

// data-* attributes → dataset (camelCase)
const card = document.querySelector('.card');
card.setAttribute('data-product-id', '42');
console.log(card.dataset.productId); // "42"
card.dataset.productId = '99';
```

### 2.5 classList

```javascript
const el = document.getElementById('text');

el.classList.add('highlight');
el.classList.add('large', 'bold');         // nhiều class cùng lúc
el.classList.remove('highlight');
el.classList.toggle('active');             // thêm/xóa xen kẽ
el.classList.replace('old-cls', 'new-cls');
const isOn = el.classList.contains('active'); // boolean
```

### 2.6 Inline Styles — camelCase

```javascript
const box = document.getElementById('box');

box.style.backgroundColor = 'yellow';  // background-color → camelCase
box.style.fontSize        = '18px';
box.style.marginTop       = '20px';
box.style.display         = 'none';    // ẩn element
box.style.display         = '';        // reset về CSS rule

// Đọc computed style (kể cả từ CSS file)
const computed = window.getComputedStyle(box);
console.log(computed.backgroundColor); // "rgb(255, 255, 0)"
```

> [!tip] classList > inline style
> `classList.toggle('dark')` tốt hơn `style.backgroundColor = '#000'` — style trong CSS file dễ maintain, hỗ trợ media query, override được.

```
★ Insight ─────────────────────────────────────
• textContent vs innerHTML là ranh giới AN TOÀN, không chỉ tiện nghi: textContent
  coi mọi thứ là CHỮ (kể cả "<script>" cũng chỉ hiện ra dưới dạng văn bản),
  innerHTML PARSE chuỗi thành HTML thật → nhét chuỗi từ user vào đó = mở cửa
  XSS. Quy tắc cứng: dữ liệu người dùng → luôn textContent. Sâu hơn ở [[13-DOM-Performance]].
• classList vs style lặp lại đúng triết lý "tách hành vi khỏi trình bày": JS chỉ
  nên BẬT/TẮT một class (trạng thái), còn HÌNH THỨC để CSS lo. Lợi ích kép — CSS
  hỗ trợ media query/hover/transition mà inline style trong JS không có, và sau
  này đổi giao diện chỉ sửa file .css, không phải lùng trong code JS.
─────────────────────────────────────────────────
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: Profile Editor

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8"><title>Profile Editor</title>
  <style>
    .error   { border: 2px solid red; }
    .success { border: 2px solid green; }
    .online  { background: green; color: white; padding: 2px 8px; border-radius: 4px; }
    .offline { background: gray; color: white; padding: 2px 8px; border-radius: 4px; }
  </style>
</head>
<body>
  <div id="profile">
    <h2 id="username">Chưa đặt tên</h2>
    <p id="bio">Chưa có bio</p>
    <span id="badge" class="offline">offline</span>
  </div>
  <form id="editForm">
    <input id="nameInput" type="text" placeholder="Tên của bạn">
    <input id="bioInput" type="text" placeholder="Bio">
    <button type="submit">Cập nhật</button>
  </form>

  <script>
    document.addEventListener('DOMContentLoaded', () => {
      const form      = document.getElementById('editForm');
      const nameInput = document.getElementById('nameInput');
      const username  = document.getElementById('username');
      const bio       = document.getElementById('bio');
      const badge     = document.getElementById('badge');

      form.addEventListener('submit', (e) => {
        e.preventDefault();
        const name = nameInput.value.trim();

        if (!name) {
          nameInput.classList.add('error');
          return;
        }
        nameInput.classList.remove('error');
        nameInput.classList.add('success');

        // textContent — an toàn với user input
        username.textContent = name;
        bio.textContent = document.getElementById('bioInput').value.trim() || 'Chưa có bio';

        badge.textContent = 'online';
        badge.classList.remove('offline');
        badge.classList.add('online');
      });
    });
  </script>
</body>
</html>
```

### Ví dụ 2: Dark Mode với localStorage

```javascript
const toggle = document.getElementById('darkModeBtn');

toggle.addEventListener('click', () => {
  document.documentElement.classList.toggle('dark');
  const isDark = document.documentElement.classList.contains('dark');
  toggle.textContent = isDark ? 'Light Mode' : 'Dark Mode';
  localStorage.setItem('darkMode', String(isDark));
});

// Restore preference khi load
if (localStorage.getItem('darkMode') === 'true') {
  document.documentElement.classList.add('dark');
}
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: innerHTML với user input — dùng textContent thay thế
> Không bao giờ render text từ người dùng qua `innerHTML`. Dùng `textContent` hoặc `createElement` + `textContent` để text được escape tự động, ngăn chặn script injection.

> [!warning] Pitfall 2: Input value luôn là string
> ```javascript
> const a = document.getElementById('numA').value; // "5" (string)
> const b = document.getElementById('numB').value; // "3" (string)
> console.log(a + b);                               // "53" — sai!
> console.log(parseInt(a, 10) + parseInt(b, 10));   // 8 — đúng
> ```

> [!warning] Pitfall 3: CSS property viết camelCase trong JS
> ```javascript
> el.style.backgroundColor = 'red'; // Đúng
> el.style['font-size']    = '16px'; // Đúng (bracket notation)
> // el.style.background-color — SyntaxError
> ```

---

## 5. Phỏng vấn thường gặp

**Q1: `textContent` vs `innerText` vs `innerHTML` — dùng khi nào?**

> `textContent` — text thuần, an toàn, không phụ thuộc layout — **dùng mặc định**. `innerText` — text hiển thị theo layout (chậm hơn vì trigger reflow). `innerHTML` — đọc/ghi HTML tags, **chỉ dùng với nội dung tĩnh**, không bao giờ với user input.

**Q2: Tại sao nên dùng `classList` thay vì set `style` trực tiếp?**

> Style trong CSS file dễ maintain, hỗ trợ media query, dễ override. `classList.toggle('active')` giữ logic trong CSS. Nếu sau này đổi màu chỉ sửa CSS, không phải hunt trong JS code.

**Q3: Đọc computed style (style từ CSS file) bằng cách nào?**

> `window.getComputedStyle(element).propertyName` — trả về giá trị thực tế sau cascade. Ví dụ: `window.getComputedStyle(btn).backgroundColor`.

---

## 6. Bài tập thực hành

**Bài 1:** "Text Transformer" — input nhập text, các button để in hoa, in thường, đảo ngược, toggle bold/italic/underline bằng classList. Dùng `textContent` để update output.

**Bài 2:** Form đổi thông tin ảnh (`src`, `alt`, `width`, `height`). Dùng `setAttribute` để cập nhật. Validate width/height là số nguyên dương.

---

## 7. Liên kết

- [[02-Selecting-DOM-Elements]] — Select element trước khi modify
- [[05-Creating-Adding-Managing-Nodes]] — Tạo element mới thêm vào DOM
- [[06-Event-Handling]] — Kích hoạt modification khi user action
- [[10-Form-Validation]] — Validate form input values
