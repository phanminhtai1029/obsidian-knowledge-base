---
title: "Traversing DOM"
section: 02-DOM-Event
tags: [dom, traversal, parent, child, fresher, frontend]
related:
  - "[[02-Selecting-DOM-Elements]]"
  - "[[05-Creating-Adding-Managing-Nodes]]"
difficulty: ⭐⭐
estimated_time: 30m
source: [Javascript.docx]
---

# Traversing DOM (Duyệt cây DOM)

> [!summary] TL;DR
> Traversal = điều hướng từ element này sang element liên quan (parent/child/sibling) mà không select lại từ đầu. Ưu tiên `parentElement`, `children`, `firstElementChild`, `nextElementSibling` (chỉ trả về Element nodes). `closest()` và `contains()` đặc biệt hữu ích trong event handling.

---

## 1. Khái niệm

Sau khi có một element, bạn có thể **di chuyển** sang các element liên quan mà không cần gọi lại `querySelector`. Đây gọi là **DOM traversal**.

### Sơ đồ property traversal

```text
                    parentElement
                         ↑
previousElementSibling ← element → nextElementSibling
                    ↓
              children / firstElementChild / lastElementChild
```

---

## 2. Cú pháp & Property

### 2.1 Parent

```javascript
const child = document.getElementById('child');

// parentElement: trả về Element (dùng cái này)
const parent = child.parentElement;
console.log(parent.tagName); // "DIV", "SECTION"...

// parentNode: trả về Node bất kỳ (có thể là Document)
// Dùng khi cần check <html> → document
```

### 2.2 Children

```javascript
const container = document.getElementById('container');

// children: HTMLCollection — chỉ Element nodes (KHÔNG có Text/Comment)
const first = container.firstElementChild;
const last  = container.lastElementChild;
const all   = Array.from(container.children);

// childNodes: NodeList — tất cả nodes (kể cả Text "\n  ")
// Tránh dùng trừ khi thực sự cần text nodes
```

### 2.3 Siblings

```javascript
const current = document.querySelector('.active');

// Chỉ Element — dùng cái này
const next = current.nextElementSibling;
const prev = current.previousElementSibling;

// nextSibling / previousSibling — trả về cả Text node (thường là "\n  ")
// Tránh trừ khi cần
```

### 2.4 closest() — leo lên tìm ancestor

```javascript
// Tìm lên trên cho đến khi match CSS selector (bao gồm chính element)
const newsContent = document.getElementById('news-content');
const article = newsContent.closest('.article');   // ancestor.article
const section = newsContent.closest('section');    // ancestor <section>

// Trả về null nếu không tìm thấy lên đến gốc
const self = newsContent.closest('#news-content'); // chính nó nếu match
```

### 2.5 contains() — kiểm tra quan hệ cha-con

```javascript
const main = document.getElementById('main');
const inner = document.getElementById('inner');

console.log(main.contains(inner)); // true nếu inner là descendant của main
console.log(inner.contains(main)); // false
console.log(main.contains(main));  // true (self-inclusive)
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: Tree Traversal

```html
<!DOCTYPE html>
<html lang="vi">
<head><meta charset="UTF-8"><title>DOM Traversal</title></head>
<body>
  <section id="departments">
    <div class="dept" id="engineering">
      <h3>Kỹ thuật</h3>
      <ul>
        <li class="employee">Alice</li>
        <li class="employee">Bob</li>
        <li class="employee">Charlie</li>
      </ul>
    </div>
    <div class="dept" id="marketing">
      <h3>Marketing</h3>
      <ul>
        <li class="employee">Diana</li>
        <li class="employee">Eve</li>
      </ul>
    </div>
  </section>

  <script>
    const alice = document.querySelector('.employee'); // Alice
    console.log('Alice:', alice.textContent);

    // Parent: <ul>
    const list = alice.parentElement;
    console.log('Parent tag:', list.tagName); // "UL"

    // Grandparent: <div.dept>
    const dept = list.parentElement;
    console.log('Dept:', dept.querySelector('h3').textContent); // "Kỹ thuật"

    // Siblings
    console.log('Next:', alice.nextElementSibling?.textContent); // "Bob"
    console.log('Prev:', alice.previousElementSibling);          // null

    // Tất cả employees trong list
    const names = Array.from(list.children).map(li => li.textContent);
    console.log('Employees:', names); // ["Alice", "Bob", "Charlie"]

    // closest() — tìm dept chứa Charlie
    const charlie = document.querySelector('.employee:last-child');
    const charlieDept = charlie.closest('.dept');
    console.log("Charlie's dept:", charlieDept.id); // "engineering"

    // contains() — kiểm tra
    const section = document.getElementById('departments');
    console.log('Section contains div:', section.contains(dept)); // true
  </script>
</body>
</html>
```

### Ví dụ 2: closest() trong Event Delegation

```javascript
// Xóa todo item khi click nút xóa (kể cả click vào icon bên trong nút)
document.getElementById('todoList').addEventListener('click', (e) => {
  // closest() xử lý trường hợp click vào <i class="icon"> bên trong nút
  const deleteBtn = e.target.closest('.btn-delete');
  if (!deleteBtn) return;

  const todoItem = deleteBtn.closest('.todo-item');
  todoItem.remove();
});
```

### Ví dụ 3: Lấy tất cả siblings trong khi loop

```javascript
function getAllSiblings(element) {
  const parent = element.parentElement;
  return Array.from(parent.children).filter(child => child !== element);
}

const activeItem = document.querySelector('li.active');
const otherItems = getAllSiblings(activeItem);
otherItems.forEach(item => item.classList.remove('active'));
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: `nextSibling` trả về Text node
> ```javascript
> // HTML: <li>A</li>\n<li>B</li>
> const li = document.querySelector('li');
> li.nextSibling;          // Text "\n" — khoảng trắng!
> li.nextElementSibling;   // <li>B</li> — đúng
> ```

> [!warning] Pitfall 2: Loop và xóa `children` cùng lúc
> `children` là **live** — khi xóa item trong lúc loop, index bị nhảy:
> ```javascript
> // Lỗi: bỏ sót item
> for (let i = 0; i < container.children.length; i++) {
>   container.children[i].remove(); // length co lại, bỏ sót
> }
>
> // Fix: snapshot trước
> Array.from(container.children).forEach(child => child.remove());
> ```

> [!tip] closest() là công cụ mạnh trong event delegation
> `e.target.closest('.btn')` cho phép click vào element con bất kỳ (icon, span) mà vẫn tìm đúng button wrapper.

```
★ Insight ─────────────────────────────────────
• Quy luật chung của traversal: mọi property có chữ "Element" (parentElement,
  children, firstElementChild, nextElementSibling) chỉ thấy ELEMENT; bản không có
  chữ "Element" (parentNode, childNodes, nextSibling) thấy cả Text/Comment — và
  thường vớ phải "\n  " (khoảng trắng giữa các thẻ). Mặc định chọn bản "Element".
• closest() đi NGƯỢC hướng querySelector: querySelector tìm XUỐNG con cháu,
  closest() leo LÊN tổ tiên đến khi khớp selector (tính cả chính nó). Cặp này
  giải quyết 2 nửa của mọi bài toán điều hướng — nhớ chúng là một cặp đối xứng.
─────────────────────────────────────────────────
```

---

## 5. Phỏng vấn thường gặp

**Q1: `parentNode` vs `parentElement` — khác nhau thế nào?**

> `parentNode` trả về **node bất kỳ** (có thể là Document — `<html>` element có `parentNode = document`). `parentElement` trả về **Element** — trả về `null` nếu parent là Document. Thực tế luôn dùng `parentElement`.

**Q2: Khi nào dùng `closest()` thay vì `parentElement`?**

> `parentElement` đi lên 1 bậc. `closest()` **leo lên cho đến khi match selector** — hữu ích khi không biết có bao nhiêu bậc. Ví dụ trong list lồng nhiều level: `e.target.closest('li')` luôn tìm đúng `<li>` dù click vào con bao nhiêu cấp.

**Q3: `children` vs `childNodes` — nên dùng cái nào?**

> Thực tế luôn dùng `children` (chỉ Element nodes) vì `childNodes` kể cả text nodes (khoảng trắng). Chỉ dùng `childNodes` khi cần thao tác text node trực tiếp.

---

## 6. Bài tập thực hành

**Bài 1:** Tạo cây gia đình HTML (`<ul>` lồng). Viết JS: tìm parent của `<li>` cụ thể, đếm siblings, liệt kê cousins.

**Bài 2:** Viết `highlightPath(element)` — tô màu tất cả ancestors của element đến `<body>`, mỗi bậc một màu khác nhau.

---

## 7. Liên kết

- [[02-Selecting-DOM-Elements]] — Select element trước khi traverse
- [[04-Modifying-DOM-Elements]] — Sửa đổi sau khi traverse
- [[05-Creating-Adding-Managing-Nodes]] — Tạo và chèn nodes
- [[07-Event-Propagation-Delegation]] — closest() trong event delegation
