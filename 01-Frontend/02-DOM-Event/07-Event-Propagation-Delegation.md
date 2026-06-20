---
title: "Event Propagation & Delegation"
section: 02-DOM-Event
tags: [dom, event, bubbling, delegation, fresher, frontend]
related:
  - "[[06-Event-Handling]]"
  - "[[03-Traversing-DOM]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [Javascript.docx]
---

# Event Propagation & Delegation

> [!summary] TL;DR
> Event propagation có 2 pha: **Bubbling** (mặc định — event nổi lên từ target đến gốc) và **Capturing** (từ gốc xuống target). `stopPropagation()` dừng lan truyền. **Event Delegation** — gắn 1 listener trên parent, dùng `e.target` + `closest()` để xác định element được click — hiệu quả hơn nhiều so với gắn listener riêng từng item.

---

## 1. Khái niệm

### Event Propagation — Hai pha

Khi click vào `<button>` bên trong `<div>` bên trong `<section>`, event không chỉ xảy ra ở button — nó **lan truyền**:

```text
Phase 1 — CAPTURING (từ trên xuống):
  document → html → body → section → div → button

Phase 2 — BUBBLING (từ dưới lên) [DEFAULT]:
  button → div → section → body → html → document
```

`addEventListener(event, fn)` mặc định là **bubbling** (phase 2). Truyền `true` làm tham số thứ 3 để bắt capturing.

```
★ Insight ─────────────────────────────────────
• Event Delegation KHÔNG phải kỹ thuật riêng — nó chỉ là "cưỡi lên" bubbling:
  vì click ở <li> tự nổi lên <ul>, ta đặt 1 listener ở <ul> rồi hỏi e.target là
  ai. Hiểu bubbling = tự suy ra được delegation, không cần học thuộc tách rời.
• Delegation thắng ở 2 điểm khó thay thế: (1) 1 listener thay vì N → nhẹ bộ nhớ
  với list dài; (2) tự động áp dụng cho item THÊM SAU khi page load — listener
  gắn trực tiếp lên item cũ thì "mù" với item mới. Đây là lý do framework (React)
  cũng gắn 1 listener ở gốc thay vì mỗi phần tử một cái.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp

### 2.1 Bubbling — mặc định

```javascript
const btn         = document.getElementById('btn');
const innerDiv    = document.getElementById('innerDiv');
const outerDiv    = document.getElementById('outerDiv');

btn.addEventListener('click',      () => console.log('Button clicked'));
innerDiv.addEventListener('click', () => console.log('Inner div clicked'));
outerDiv.addEventListener('click', () => console.log('Outer div clicked'));

// Click button → log theo thứ tự:
// "Button clicked" → "Inner div clicked" → "Outer div clicked"
// (bubbles up)
```

### 2.2 Capturing — truyền `true`

```javascript
btn.addEventListener('click',      () => console.log('Button'), true);
innerDiv.addEventListener('click', () => console.log('Inner'), true);
outerDiv.addEventListener('click', () => console.log('Outer'), true);

// Click button → log theo thứ tự:
// "Outer" → "Inner" → "Button"
// (captures down)
```

### 2.3 stopPropagation()

```javascript
btn.addEventListener('click', (e) => {
  console.log('Button clicked');
  e.stopPropagation(); // dừng tại đây — không bubble lên div
});

innerDiv.addEventListener('click', () => {
  console.log('Inner div — không bao giờ chạy nếu click vào button');
});
```

### 2.4 preventDefault()

```javascript
// Ngăn hành vi mặc định của browser
document.getElementById('resourceLink').addEventListener('click', (e) => {
  e.preventDefault(); // không navigate đến href
  console.log('Link clicked nhưng không navigate');
});

document.getElementById('myForm').addEventListener('submit', (e) => {
  e.preventDefault(); // không reload trang
  console.log('Form submitted — xử lý bằng JS');
  // Gọi API, validate...
});
```

### 2.5 Event Delegation

```javascript
// Thay vì gắn listener trên từng <li> (không efficient):
// document.querySelectorAll('li').forEach(li => li.addEventListener('click', ...))

// Gắn 1 listener trên parent
const agendaList = document.getElementById('agendaList');

agendaList.addEventListener('click', (e) => {
  // e.target = element thực sự được click
  if (e.target.tagName === 'LI') {
    e.target.remove(); // xóa item được click
  }
});

// Với closest() — xử lý click vào element con bên trong li
agendaList.addEventListener('click', (e) => {
  const item = e.target.closest('li');
  if (!item) return; // click vào gap, không phải li
  item.classList.toggle('done');
});
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: Bubbling demo trực quan

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8"><title>Event Propagation</title>
  <style>
    .outer { background: #ffcccc; padding: 30px; cursor: pointer; }
    .inner { background: #ccffcc; padding: 20px; }
    button { padding: 10px 20px; }
    #log   { margin-top: 10px; font-family: monospace; }
  </style>
</head>
<body>
  <div class="outer" id="outer">
    Outer div
    <div class="inner" id="inner">
      Inner div
      <button id="btn">Button</button>
    </div>
  </div>
  <div id="log"></div>
  <button onclick="document.getElementById('log').textContent=''">Clear</button>

  <script>
    const log = document.getElementById('log');
    function addLog(msg) {
      log.textContent += msg + '\n';
    }

    // Bubbling (mặc định)
    document.getElementById('btn').addEventListener('click', () => addLog('▶ Button'));
    document.getElementById('inner').addEventListener('click', () => addLog('▶ Inner div'));
    document.getElementById('outer').addEventListener('click', () => addLog('▶ Outer div'));

    // Click nút → thấy thứ tự bubble từ trong ra ngoài
  </script>
</body>
</html>
```

### Ví dụ 2: Event Delegation — Dynamic Todo

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8"><title>Event Delegation</title>
  <style>
    .todo-item { display: flex; align-items: center; gap: 8px; padding: 8px; border: 1px solid #ddd; margin: 4px; }
    .todo-item.done span { text-decoration: line-through; color: #999; }
  </style>
</head>
<body>
  <input id="newTodo" type="text" placeholder="Nhiệm vụ mới...">
  <button id="addBtn">Thêm</button>

  <ul id="todoList"></ul>

  <script>
    document.addEventListener('DOMContentLoaded', () => {
      const list   = document.getElementById('todoList');
      const input  = document.getElementById('newTodo');
      const addBtn = document.getElementById('addBtn');

      // Thêm item
      function addTodo(text) {
        const li = document.createElement('li');
        li.className = 'todo-item';

        const checkbox = document.createElement('input');
        checkbox.type = 'checkbox';
        checkbox.className = 'todo-check';

        const span = document.createElement('span');
        span.textContent = text; // textContent — an toàn

        const delBtn = document.createElement('button');
        delBtn.textContent = 'Xóa';
        delBtn.className = 'btn-delete';

        li.append(checkbox, span, delBtn);
        list.appendChild(li);
      }

      addBtn.addEventListener('click', () => {
        const text = input.value.trim();
        if (!text) return;
        addTodo(text);
        input.value = '';
      });

      // EVENT DELEGATION — 1 listener cho toàn bộ list
      list.addEventListener('click', (e) => {
        const item = e.target.closest('.todo-item');
        if (!item) return;

        // Click checkbox → toggle done
        if (e.target.classList.contains('todo-check')) {
          item.classList.toggle('done');
        }

        // Click nút xóa
        if (e.target.classList.contains('btn-delete')) {
          item.remove();
        }
      });

      // Seed data
      ['Đọc tài liệu', 'Làm bài tập', 'Review PR'].forEach(addTodo);
    });
  </script>
</body>
</html>
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: Không hiểu tại sao parent cũng fire event
> Click vào `<button>` trong `<div>` → `div.onclick` cũng chạy vì **bubbling**. Nếu chỉ muốn button handle event, dùng `e.stopPropagation()` hoặc kiểm tra `if (e.target === btn)`.

> [!warning] Pitfall 2: Event delegation với dynamic children
> Nếu thêm element vào DOM sau khi page load, listener gắn trực tiếp trên element cũ không hoạt động với element mới. **Event delegation trên parent** xử lý được cả elements tạo ra sau:
> ```javascript
> // Không tốt — chỉ hoạt động với items hiện có
> document.querySelectorAll('.item').forEach(item => item.addEventListener('click', fn));
>
> // Tốt — hoạt động với items thêm sau
> document.getElementById('list').addEventListener('click', (e) => {
>   if (e.target.closest('.item')) fn(e);
> });
> ```

> [!tip] e.target.closest() trong delegation
> `e.target` có thể là element con (icon, span) bên trong item bạn quan tâm. `e.target.closest('.item')` leo lên tìm wrapper — xử lý được click mọi nơi trong item.

---

## 5. Phỏng vấn thường gặp

**Q1: Event Bubbling là gì? Làm thế nào dừng nó?**

> Bubbling = event lan truyền từ target element lên các ancestor trong DOM tree. Dừng bằng `e.stopPropagation()` — event không bubble lên parent nữa. Tuy nhiên, dùng cẩn thận vì có thể break event delegation của component khác.

**Q2: Event Delegation là gì? Lợi ích?**

> Thay vì gắn listener riêng từng item, gắn **1 listener trên parent** và dùng `e.target` để xác định item nào được click. Lợi ích: (1) ít listeners = ít memory, (2) hoạt động với elements thêm dynamically sau page load, (3) code gọn hơn cho lists dài.

**Q3: `e.preventDefault()` vs `e.stopPropagation()` — khác nhau?**

> `preventDefault()` — ngăn **hành vi mặc định** của browser (navigate link, reload form, checkbox toggle). `stopPropagation()` — ngăn **event lan truyền** lên/xuống DOM tree. Hai cái độc lập — có thể dùng một hoặc cả hai.

---

## 6. Bài tập thực hành

**Bài 1:** Tạo nested `<div>` 3 cấp, mỗi cấp có màu khác nhau và click listener. Click vào div trong cùng và quan sát thứ tự bubbling. Sau đó thêm `stopPropagation()` ở cấp 2 và kiểm tra lại.

**Bài 2:** Xây dựng shopping cart dùng event delegation: list sản phẩm, mỗi item có nút "+", "-", "Xóa". Chỉ dùng **1 listener** trên container để handle tất cả 3 loại nút.

---

## 7. Liên kết

- [[06-Event-Handling]] — addEventListener cơ bản
- [[03-Traversing-DOM]] — closest() để tìm element trong delegation
- [[10-Form-Validation]] — preventDefault() trong form submit
- [[09-Fetch-API]] — Gọi API sau khi prevent default
