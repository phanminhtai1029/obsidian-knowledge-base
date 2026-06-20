---
title: "Event Handling"
section: 02-DOM-Event
tags: [dom, event, addEventListener, fresher, frontend]
related:
  - "[[07-Event-Propagation-Delegation]]"
  - "[[10-Form-Validation]]"
difficulty: ⭐⭐
estimated_time: 35m
source: [Javascript.docx]
---

# Event Handling

> [!summary] TL;DR
> Events = tín hiệu JS nhận khi user tương tác (click, input, submit...). Ba cách gắn handler: HTML `onclick=`, JS property `.onclick =`, `addEventListener` (khuyên dùng — cho phép nhiều handler + dễ remove). Luôn wrap code trong `DOMContentLoaded`. Event object chứa thông tin sự kiện (`target`, `key`, `preventDefault`...).

---

## 1. Khái niệm

**Event** = tín hiệu nhỏ xảy ra khi có hành động: click, hover, nhập bàn phím, submit form, trang load xong...

**Event Handler** = function được gọi khi event xảy ra.

**Event Listener** = cơ chế "lắng nghe" event trên element và gọi handler tương ứng.

### Ba cách gắn event handler — so sánh

| Cách | Cú pháp | Ưu điểm | Nhược điểm |
|------|---------|---------|-----------|
| HTML attribute | `onclick="fn()"` | Nhanh để test | Trộn HTML/JS, không scalable |
| JS property | `el.onclick = fn` | Đơn giản | Chỉ 1 handler per event |
| `addEventListener` | `el.addEventListener('click', fn)` | Nhiều handlers, dễ remove | Verbose hơn đôi chút |

> [!tip] addEventListener là chuẩn
> Luôn dùng `addEventListener` trong code thực tế. `.onclick =` tiện nhưng chỉ cho phép 1 handler — handler mới sẽ ghi đè handler cũ.

---

## 2. Cú pháp

### 2.1 addEventListener

```javascript
const btn = document.getElementById('logBtn');

// Cách 1: named function (dễ remove sau này)
function handleClick() {
  console.log("Don't click me!");
}
btn.addEventListener('click', handleClick);

// Cách 2: arrow function (ngắn gọn, nhưng không remove được)
btn.addEventListener('click', () => {
  console.log("Don't click me!");
});

// Cách 3: thêm nhiều handler cho cùng 1 event
btn.addEventListener('click', () => console.log('Handler 1'));
btn.addEventListener('click', () => console.log('Handler 2'));
// Cả hai đều chạy khi click

// removeEventListener — cần CÙNG reference function
btn.removeEventListener('click', handleClick); // xóa được vì là named fn
```

### 2.2 Event object

```javascript
btn.addEventListener('click', (e) => {
  console.log(e.type);           // "click"
  console.log(e.target);         // element được click
  console.log(e.currentTarget);  // element mà listener được gắn
  console.log(e.timeStamp);      // thời điểm xảy ra event
});

// Keyboard event
document.addEventListener('keydown', (e) => {
  console.log(e.key);            // "a", "Enter", "ArrowUp"
  console.log(e.code);           // "KeyA", "Enter", "ArrowUp"
  console.log(e.ctrlKey);        // true/false
  console.log(e.shiftKey);       // true/false
});

// Mouse event
document.addEventListener('mousemove', (e) => {
  console.log(e.clientX, e.clientY); // vị trí con trỏ
});
```

### 2.3 Common DOM Events

```javascript
document.addEventListener('DOMContentLoaded', () => {
  // DOM sẵn sàng — tất cả code nên wrap trong đây

  // Click
  document.getElementById('clickBtn').addEventListener('click', () => {
    console.log('Button clicked');
  });

  // Hover
  const hoverArea = document.getElementById('hoverArea');
  hoverArea.addEventListener('mouseover', () => console.log('Mouse over'));
  hoverArea.addEventListener('mouseout', () => console.log('Mouse out'));

  // Focus / Blur (input)
  const input = document.getElementById('emailInput');
  input.addEventListener('focus', () => console.log('Input focused'));
  input.addEventListener('blur', () => console.log('Input lost focus'));

  // Input — mỗi lần gõ phím
  input.addEventListener('input', (e) => {
    console.log('Current value:', e.target.value);
  });

  // Keydown — phím được nhấn
  document.addEventListener('keydown', (e) => {
    console.log('Key pressed:', e.key);
  });

  // Form submit
  document.getElementById('myForm').addEventListener('submit', (e) => {
    e.preventDefault(); // ngăn trang refresh
    console.log('Form submitted');
  });
});
```

### 2.4 removeEventListener — one-time handler

```javascript
const btn = document.getElementById('actionBtn');

function handleActionOnce() {
  console.log('Action performed — chỉ 1 lần!');
  btn.removeEventListener('click', handleActionOnce); // self-remove
}

btn.addEventListener('click', handleActionOnce);
// Click lần 2 → không có gì xảy ra
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: Interactive Dashboard với nhiều events

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8"><title>Event Demo</title>
  <style>
    #hoverBox { padding: 20px; background: #eee; margin: 10px 0; transition: background 0.3s; }
    #hoverBox.hover { background: #c8e6c9; }
  </style>
</head>
<body>
  <button id="clickBtn">Click tôi</button>
  <div id="hoverBox">Hover vào đây</div>
  <input id="liveSearch" type="text" placeholder="Gõ để tìm...">
  <p id="output"></p>
  <form id="searchForm">
    <input id="searchInput" type="text" placeholder="Tìm kiếm...">
    <button type="submit">Tìm</button>
  </form>

  <script>
    document.addEventListener('DOMContentLoaded', () => {
      const output = document.getElementById('output');

      // Click event
      document.getElementById('clickBtn').addEventListener('click', (e) => {
        output.textContent = `Clicked tại (${e.clientX}, ${e.clientY})`;
      });

      // Hover events
      const hoverBox = document.getElementById('hoverBox');
      hoverBox.addEventListener('mouseover', () => hoverBox.classList.add('hover'));
      hoverBox.addEventListener('mouseout',  () => hoverBox.classList.remove('hover'));

      // Live search (input event)
      document.getElementById('liveSearch').addEventListener('input', (e) => {
        output.textContent = `Đang tìm: "${e.target.value}"`;
      });

      // Form submit với preventDefault
      document.getElementById('searchForm').addEventListener('submit', (e) => {
        e.preventDefault();
        const query = document.getElementById('searchInput').value.trim();
        output.textContent = query
          ? `Kết quả tìm kiếm: "${query}"`
          : 'Vui lòng nhập từ khóa';
      });

      // Keyboard shortcut: Ctrl+K focus search
      document.addEventListener('keydown', (e) => {
        if (e.ctrlKey && e.key === 'k') {
          e.preventDefault();
          document.getElementById('liveSearch').focus();
        }
      });
    });
  </script>
</body>
</html>
```

### Ví dụ 2: Counter với one-time event

```javascript
document.addEventListener('DOMContentLoaded', () => {
  let count = 0;
  const countDisplay = document.getElementById('count');
  const addBtn = document.getElementById('addBtn');
  const resetBtn = document.getElementById('resetBtn');

  addBtn.addEventListener('click', () => {
    count++;
    countDisplay.textContent = count;

    // Khi đạt 5, disable button sau 1 lần nữa
    if (count === 5) {
      function onceMore() {
        count++;
        countDisplay.textContent = count + ' (max!)';
        addBtn.disabled = true;
        addBtn.removeEventListener('click', onceMore);
      }
      addBtn.removeEventListener('click', arguments.callee); // nếu arrow — không dùng được
    }
  });

  resetBtn.addEventListener('click', () => {
    count = 0;
    countDisplay.textContent = 0;
    addBtn.disabled = false;
  });
});
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: Gắn handler khi DOM chưa load
> ```javascript
> // Lỗi — chạy ngay khi script load, DOM chưa có #btn
> document.getElementById('btn').addEventListener('click', fn);
>
> // Fix — wrap trong DOMContentLoaded
> document.addEventListener('DOMContentLoaded', () => {
>   document.getElementById('btn').addEventListener('click', fn);
> });
> ```

> [!warning] Pitfall 2: `.onclick = fn` ghi đè handler cũ
> ```javascript
> btn.onclick = () => console.log('Handler 1');
> btn.onclick = () => console.log('Handler 2'); // ghi đè — chỉ Handler 2 chạy!
>
> // Fix: dùng addEventListener
> btn.addEventListener('click', () => console.log('Handler 1'));
> btn.addEventListener('click', () => console.log('Handler 2')); // cả hai chạy
> ```

> [!warning] Pitfall 3: Arrow function không remove được
> ```javascript
> // Không remove được — mỗi lần là anonymous function mới
> btn.addEventListener('click', () => doSomething());
> btn.removeEventListener('click', () => doSomething()); // KHÔNG hoạt động!
>
> // Fix: dùng named function
> function handler() { doSomething(); }
> btn.addEventListener('click', handler);
> btn.removeEventListener('click', handler); // hoạt động
> ```

---

## 5. Phỏng vấn thường gặp

**Q1: Sự khác nhau giữa `onclick` và `addEventListener('click', ...)`?**

> `onclick` chỉ cho phép **1 handler** — gán lại sẽ ghi đè. `addEventListener` cho phép **nhiều handlers** cho cùng event và dễ remove bằng `removeEventListener`. Trong code thực tế luôn dùng `addEventListener`.

**Q2: `e.target` vs `e.currentTarget` — khác nhau thế nào?**

> `e.target` — element **thực sự bị click** (có thể là element con). `e.currentTarget` — element mà event **listener được gắn**. Trong event delegation: click vào `<li>` bên trong `<ul>`, `target` = `<li>`, `currentTarget` = `<ul>`.

**Q3: Tại sao cần `DOMContentLoaded`?**

> `<script>` trong `<head>` chạy khi DOM chưa load xong → `getElementById` trả về `null`. `DOMContentLoaded` fires khi DOM đã parse xong (không cần chờ images/stylesheets). Cách khác: đặt `<script>` cuối `<body>`.

---

## 6. Bài tập thực hành

**Bài 1:** Tạo "Pomodoro timer" đơn giản: nút Start/Stop/Reset, hiện số giây đếm ngược từ 25 phút. Dùng `setInterval`/`clearInterval` bên trong event handlers.

**Bài 2:** Color Picker — 6 ô màu khác nhau, click vào ô nào thì đổi màu nền body. Dùng `data-color` attribute và 1 event listener duy nhất (event delegation trên container).

---

## 7. Liên kết

- [[07-Event-Propagation-Delegation]] — Bubbling, capturing, delegation
- [[04-Modifying-DOM-Elements]] — Thay đổi DOM trong event handler
- [[10-Form-Validation]] — Submit event + preventDefault
- [[09-Fetch-API]] — Gọi API trong event handler
