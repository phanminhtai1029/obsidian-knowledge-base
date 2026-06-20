---
title: "List & Table Rendering"
section: 02-DOM-Event
tags: [dom, list, table, render, fresher, frontend]
related:
  - "[[02-Selecting-DOM-Elements]]"
  - "[[06-Event-Handling]]"
difficulty: ⭐⭐
estimated_time: 35m
source: [Javascript.docx]
---

# List & Table Rendering

> [!summary] TL;DR
> Render list dùng `createElement` + `appendChild`/`DocumentFragment`. Render table chuyên dụng: `createTHead()`, `createTBody()`, `insertRow(-1)`, `insertCell()`. Khi render từ mảng JSON, dùng `DocumentFragment` hoặc `replaceChildren()` để giảm reflow — append toàn bộ fragment 1 lần thay vì từng item.

---

## 1. Khái niệm

### Dynamic List Rendering

Thay vì hardcode HTML, ta dùng JS để render danh sách từ dữ liệu:
- Tạo từng `<li>` bằng `createElement`
- Đặt nội dung bằng `textContent` (an toàn, không XSS)
- Gắn vào `<ul>` hoặc dùng `DocumentFragment` cho batch lớn

### Dynamic Table Rendering

Browser cung cấp API chuyên dụng cho table:

| Method | Mục đích |
|---|---|
| `table.createTHead()` | Tạo/lấy `<thead>` |
| `table.createTBody()` | Tạo/lấy `<tbody>` |
| `table.createTFoot()` | Tạo/lấy `<tfoot>` |
| `tbody.insertRow(-1)` | Thêm `<tr>` vào cuối (`-1` = cuối) |
| `row.insertCell(-1)` | Thêm `<td>` vào cuối row |
| `table.deleteRow(index)` | Xóa row theo index |

> [!tip] Dùng `insertRow(-1)` thay `appendChild`
> `insertRow(-1)` là API table chuyên dụng — tự xử lý vị trí cuối, tương thích cả thead/tbody/tfoot, không cần tạo element riêng.

### DocumentFragment

`DocumentFragment` là node ảo — không nằm trong DOM thật. Thêm nhiều element vào fragment, rồi append fragment 1 lần → **chỉ 1 reflow** thay vì N reflow.

```
★ Insight ─────────────────────────────────────
• "Tạo data → render ra DOM" là bài toán mà cả React sinh ra để giải. Khi tự làm
  bằng tay, 3 nguyên tắc vàng lặp lại: (1) textContent cho mọi giá trị data (chặn
  XSS), (2) gom vào DocumentFragment rồi append 1 lần (1 reflow), (3) 1 listener
  delegation ở container thay vì mỗi item một listener. Nắm 3 cái này = nắm "DOM
  rendering thủ công" đủ để hiểu vì sao framework tự động hóa chúng.
• replaceChildren() > xóa bằng innerHTML="": innerHTML rỗng buộc trình duyệt
  parse lại chuỗi và làm "mồ côi" listener gắn trực tiếp; replaceChildren() xóa
  sạch node con một cách tường minh, an toàn hơn. Còn listener đặt theo delegation
  ở container thì luôn sống sót qua mọi lần render lại.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp

### 2.1 Render danh sách đơn giản

```javascript
const tasks = ['Đọc docs', 'Viết code', 'Review PR'];
const ul    = document.getElementById('taskList');

tasks.forEach(text => {
  const li       = document.createElement('li');
  li.textContent = text;      // textContent — tránh XSS
  ul.appendChild(li);
});
```

### 2.2 Render với DocumentFragment (hiệu quả hơn)

```javascript
const data     = ['Mục 1', 'Mục 2', 'Mục 3', 'Mục 4', 'Mục 5'];
const ul       = document.getElementById('list');
const fragment = document.createDocumentFragment();

data.forEach(text => {
  const li       = document.createElement('li');
  li.textContent = text;
  fragment.appendChild(li);
});

ul.appendChild(fragment); // 1 lần append = 1 reflow
```

### 2.3 Thêm / Xóa item trong list

```javascript
const input  = document.getElementById('taskInput');
const addBtn = document.getElementById('addBtn');
const ul     = document.getElementById('taskList');

addBtn.addEventListener('click', () => {
  const text = input.value.trim();
  if (!text) return;

  const li      = document.createElement('li');
  const span    = document.createElement('span');
  span.textContent = text;

  const delBtn       = document.createElement('button');
  delBtn.textContent = 'Xóa';
  delBtn.className   = 'del-btn';

  li.append(span, delBtn);
  ul.appendChild(li);
  input.value = '';
});

// Event Delegation — 1 listener xử lý tất cả nút Xóa
ul.addEventListener('click', (e) => {
  if (e.target.classList.contains('del-btn')) {
    e.target.closest('li').remove();
  }
});
```

### 2.4 Render bảng từ mảng JSON

```javascript
function renderTable(data, containerId) {
  const container = document.getElementById(containerId);
  container.replaceChildren(); // xóa nội dung cũ an toàn

  if (!data.length) {
    const p       = document.createElement('p');
    p.textContent = 'Không có dữ liệu.';
    container.appendChild(p);
    return;
  }

  const table = document.createElement('table');
  table.className = 'data-table';

  // Tạo header từ keys của object đầu tiên
  const thead = table.createTHead();
  const headerRow = thead.insertRow();
  Object.keys(data[0]).forEach(key => {
    const th       = document.createElement('th');
    th.textContent = key;
    headerRow.appendChild(th);
  });

  // Tạo body
  const tbody = table.createTBody();
  data.forEach(item => {
    const row = tbody.insertRow(-1); // -1 = thêm vào cuối
    Object.values(item).forEach(val => {
      const cell       = row.insertCell(-1);
      cell.textContent = val;         // textContent — an toàn
    });
  });

  container.appendChild(table);
}

// Dùng
const employees = [
  { name: 'An',   dept: 'Dev',    salary: 15000000 },
  { name: 'Bình', dept: 'Design', salary: 12000000 },
  { name: 'Châu', dept: 'QA',     salary: 13000000 },
];
renderTable(employees, 'tableContainer');
```

### 2.5 Xóa row trong bảng

```javascript
// Xóa row bằng index (0-based, tính từ đầu table)
const table = document.getElementById('myTable');
table.deleteRow(1); // xóa row thứ 2 trong table

// Event delegation — xóa row khi click nút trong row
table.addEventListener('click', (e) => {
  const btn = e.target.closest('.btn-delete-row');
  if (!btn) return;
  e.target.closest('tr').remove();
});
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: Task Manager hoàn chỉnh (add / complete / delete)

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8"><title>Task Manager</title>
  <style>
    .task-item { display: flex; gap: 8px; align-items: center; padding: 6px; border-bottom: 1px solid #eee; }
    .task-item.done span { text-decoration: line-through; color: #999; }
    .task-item span { flex: 1; }
  </style>
</head>
<body>
  <input id="inp" type="text" placeholder="Nhiệm vụ mới...">
  <button id="addBtn">Thêm</button>
  <ul id="list" style="list-style:none;padding:0"></ul>

  <script>
    const inp    = document.getElementById('inp');
    const addBtn = document.getElementById('addBtn');
    const list   = document.getElementById('list');

    function createItem(text) {
      const li      = document.createElement('li');
      li.className  = 'task-item';

      const cb      = document.createElement('input');
      cb.type       = 'checkbox';
      cb.className  = 'cb-done';

      const span       = document.createElement('span');
      span.textContent = text;

      const del        = document.createElement('button');
      del.textContent  = '✕';
      del.className    = 'btn-del';

      li.append(cb, span, del);
      return li;
    }

    addBtn.addEventListener('click', () => {
      const text = inp.value.trim();
      if (!text) return;
      list.appendChild(createItem(text));
      inp.value = '';
      inp.focus();
    });

    inp.addEventListener('keydown', (e) => {
      if (e.key === 'Enter') addBtn.click();
    });

    // 1 listener dùng Event Delegation
    list.addEventListener('click', (e) => {
      const item = e.target.closest('.task-item');
      if (!item) return;

      if (e.target.classList.contains('cb-done')) {
        item.classList.toggle('done', e.target.checked);
      }
      if (e.target.classList.contains('btn-del')) {
        item.remove();
      }
    });

    // Seed data
    ['Đọc tài liệu DOM', 'Viết bài tập', 'Review code'].forEach(t => list.appendChild(createItem(t)));
  </script>
</body>
</html>
```

### Ví dụ 2: Render bảng nhân viên từ JSON

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8"><title>Employee Table</title>
  <style>
    table { border-collapse: collapse; width: 100%; }
    th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
    th { background: #4a90d9; color: white; }
    tr:hover { background: #f5f5f5; }
    .btn-del { background: #e74c3c; color: white; border: none; cursor: pointer; padding: 2px 8px; }
  </style>
</head>
<body>
  <div id="tableWrap"></div>

  <script>
    const employees = [
      { id: 1, name: 'Nguyễn An',  dept: 'Frontend', salary: 15000000 },
      { id: 2, name: 'Trần Bình',  dept: 'Backend',  salary: 18000000 },
      { id: 3, name: 'Lê Châu',    dept: 'Design',   salary: 13000000 },
    ];

    function renderEmployeeTable(data) {
      const wrap  = document.getElementById('tableWrap');
      wrap.replaceChildren(); // xóa nội dung cũ

      const table = document.createElement('table');
      const thead = table.createTHead();
      const hRow  = thead.insertRow();

      ['ID', 'Tên', 'Bộ phận', 'Lương', 'Hành động'].forEach(h => {
        const th       = document.createElement('th');
        th.textContent = h;
        hRow.appendChild(th);
      });

      const tbody = table.createTBody();
      data.forEach(emp => {
        const row    = tbody.insertRow(-1);
        row.dataset.id = emp.id;

        [emp.id, emp.name, emp.dept, emp.salary.toLocaleString('vi-VN') + '₫'].forEach(val => {
          const cell       = row.insertCell(-1);
          cell.textContent = val;
        });

        const actionCell  = row.insertCell(-1);
        const delBtn      = document.createElement('button');
        delBtn.textContent = 'Xóa';
        delBtn.className  = 'btn-del';
        actionCell.appendChild(delBtn);
      });

      // Event delegation trên tbody
      tbody.addEventListener('click', (e) => {
        if (e.target.classList.contains('btn-del')) {
          e.target.closest('tr').remove();
        }
      });

      wrap.appendChild(table);
    }

    renderEmployeeTable(employees);
  </script>
</body>
</html>
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: Reset container làm mất event listeners
> Gán chuỗi rỗng vào `innerHTML` để clear container sẽ xóa mọi event listeners đã gắn trực tiếp lên child elements. Thay vào đó dùng `element.replaceChildren()` (modern) hoặc vòng lặp `while (el.firstChild) el.removeChild(el.firstChild)`. Event Delegation trên container thì an toàn vì listener ở container vẫn còn.

> [!warning] Pitfall 2: Hiển thị dữ liệu người dùng bằng DOM insertion method không an toàn
> Nếu giá trị từ user input hoặc API chứa thẻ script ác ý, việc chèn vào DOM qua các method không an toàn sẽ cho phép execute code. Luôn dùng `textContent` khi hiển thị dữ liệu không tin cậy — nó tự escape HTML entities.

> [!tip] `insertRow(-1)` vs `createElement` + `appendChild`
> `insertRow(-1)` là API của `HTMLTableSectionElement` — tạo `<tr>`, chèn vào cuối, trả về element trong 1 dòng. Không cần `createElement('tr')` riêng. Ngắn gọn và intent rõ ràng hơn với table.

---

## 5. Phỏng vấn thường gặp

**Q1: DocumentFragment là gì? Khi nào dùng?**

> `DocumentFragment` là container nhẹ không thuộc DOM thật. Thêm nhiều element vào fragment trước, rồi append fragment 1 lần vào DOM → browser chỉ reflow 1 lần. Dùng khi render nhiều item (> 10-20) cùng lúc để tránh layout thrashing.

**Q2: Sự khác biệt giữa `insertRow()` và `createElement('tr')`?**

> `insertRow(-1)` là table API chuyên dụng: tự tạo `<tr>`, chèn vào đúng vị trí trong thead/tbody/tfoot, và trả về element đó. `createElement('tr')` tạo element orphan, phải tự `appendChild` vào đúng nơi. `insertRow` ngắn hơn và ít lỗi hơn với table.

**Q3: Làm thế nào render danh sách hiệu quả với nhiều item?**

> (1) Dùng `DocumentFragment` — tập hợp tất cả element, append 1 lần. (2) Tránh loop read-modify-DOM (gây reflow mỗi vòng). (3) Với list rất dài (> 1000 items) xem xét Virtual Scrolling. (4) Luôn dùng `textContent` thay vì các method chèn HTML khi hiển thị dữ liệu người dùng để tránh XSS.

---

## 6. Bài tập thực hành

**Bài 1:** Tạo danh sách mua sắm: thêm item (tên + số lượng), đánh dấu đã mua (gạch chân), xóa item. Dùng Event Delegation cho tất cả interactions.

**Bài 2:** Render bảng từ mảng JSON 10 học sinh (`{ name, score, grade }`). Tính grade tự động (≥8: A, ≥6.5: B, ≥5: C, còn lại: F). Thêm nút "Xóa" mỗi row. Sort theo điểm khi click header cột "Score".

---

## 7. Liên kết

- [[05-Creating-Adding-Managing-Nodes]] — createElement, DocumentFragment, insertBefore
- [[07-Event-Propagation-Delegation]] — Event Delegation pattern
- [[09-Fetch-API]] — Render data từ API vào table
- [[06-Event-Handling]] — addEventListener cơ bản
