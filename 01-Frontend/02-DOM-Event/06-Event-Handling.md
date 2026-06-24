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

> [!tip] 🎯 Hiểu trong 30 giây
> **Event (sự kiện) = tín hiệu trình duyệt báo cho JS biết "người dùng vừa làm gì đó"** (bấm chuột, gõ phím, gửi form). Bạn "đăng ký lắng nghe" rồi bảo nó *làm gì khi xảy ra* — gọi là **event handler**.
> - Cách nên dùng: **`el.addEventListener('click', handler)`** — cho phép gắn *nhiều* handler và *gỡ* được (`removeEventListener`), gọn hơn 2 cách cũ (`onclick=` trong HTML hay `.onclick =` ghi đè lẫn nhau).
> - Hàm handler nhận một **event object** (`e`) chứa thông tin: `e.target` (phần tử bị tác động), `e.key` (phím nào), `e.preventDefault()` (chặn hành vi mặc định)...
> - Nên bọc code trong **`DOMContentLoaded`** để chắc chắn HTML đã dựng xong DOM rồi mới gắn listener (gắn quá sớm sẽ không tìm thấy phần tử).

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

```
★ Insight ─────────────────────────────────────
• `removeEventListener` đòi CÙNG MỘT reference hàm đã add — đây là lý do arrow
  inline (`() => ...`) gỡ KHÔNG được: mỗi lần viết ra là một hàm mới khác địa chỉ.
  Quy tắc: cần gỡ về sau → tách thành named function; không cần gỡ → arrow inline
  cho gọn. Hoặc dùng `{ once: true }` để browser tự gỡ.
• 3 cách gắn handler thực ra là 3 thế hệ: onclick="" (HTML, trộn markup+logic),
  el.onclick= (1 slot duy nhất), addEventListener (nhiều slot + capture/once/
  passive). Bản chất giống "inline style vs class" bên CSS — cùng một xu hướng
  tách hành vi khỏi cấu trúc. addEventListener thắng vì mở rộng được.
─────────────────────────────────────────────────
```

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

  // named function để có thể removeEventListener
  function handleAdd() {
    count++;
    countDisplay.textContent = count;
    if (count >= 5) {
      countDisplay.textContent = count + ' (max!)';
      addBtn.disabled = true;
      addBtn.removeEventListener('click', handleAdd); // gỡ listener khi đạt max
    }
  }
  addBtn.addEventListener('click', handleAdd);

  resetBtn.addEventListener('click', () => {
    count = 0;
    countDisplay.textContent = 0;
    addBtn.disabled = false;
    addBtn.addEventListener('click', handleAdd); // gắn lại sau reset
  });
});
```

> [!tip] Option `{ once: true }`
> Nếu chỉ cần handler chạy đúng 1 lần rồi tự gỡ, không cần tự `removeEventListener`:
> `btn.addEventListener('click', fn, { once: true })` — browser tự gỡ sau lần đầu.

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

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "`onclick` khác `addEventListener` thế nào? `e.target` vs `e.currentTarget`?"
> *"onclick gán qua thuộc tính chỉ cho phép một handler, gán cái mới sẽ ghi đè cái cũ. addEventListener cho phép gắn nhiều handler cho cùng một sự kiện và gỡ được bằng removeEventListener, nên em luôn dùng addEventListener. Về event object, e.target là phần tử thực sự bị tác động, có thể là phần tử con; còn e.currentTarget là phần tử mà mình gắn listener vào. Trong event delegation thì rõ nhất: gắn listener ở ul, click vào một li bên trong thì target là li đó còn currentTarget là ul."*

> [!note] 🧠 Mẹo nhớ
> **`addEventListener` (nhiều handler, gỡ được) > `onclick` (1 handler, đè nhau).** **`e.target` = nơi bị click; `e.currentTarget` = nơi gắn listener.** Bọc code trong `DOMContentLoaded`.

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
