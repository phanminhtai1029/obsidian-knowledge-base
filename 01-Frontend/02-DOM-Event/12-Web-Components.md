---
title: "Web Components & Custom Elements"
section: 02-DOM-Event
tags: [dom, web-components, custom-elements, shadow-dom, html-template, fresher, frontend]
related:
  - "[[05-Creating-Adding-Managing-Nodes]]"
  - "[[../06-React/01-React-Overview]]"
difficulty: ⭐⭐⭐
estimated_time: 35m
source: [JS-DOM raw transcript, MDN]
---

# Web Components & Custom Elements

> [!summary] TL;DR
> **Web Components** là **chuẩn của trình duyệt** (không cần framework) để tạo **thẻ HTML tái sử dụng** của riêng bạn, theo nguyên tắc DRY. Gồm 3 phần: (1) **Custom Elements** — định nghĩa thẻ riêng (`class extends HTMLElement` + `customElements.define`); (2) **Shadow DOM** — "DOM ẩn" đóng gói style/script, **không rò rỉ** ra ngoài (`attachShadow`); (3) **HTML Templates** — khối markup `<template>` được parse nhưng **chưa render** cho tới khi clone. Khác React/Vue (cần thư viện), Web Components chạy thuần trình duyệt — nhưng hệ sinh thái (state, data binding) yếu hơn.

---

## 1. Vì sao có Web Components

Khi cần một phần giao diện **lặp lại** nhiều nơi/nhiều trang, copy HTML mãi là vi phạm DRY (*Don't Repeat Yourself*). Web Components cho phép gói nó thành **một thẻ tái sử dụng** như `<user-card>`.

| 3 trụ cột | Vai trò |
|---|---|
| **Custom Elements** | Định nghĩa thẻ HTML của riêng bạn |
| **Shadow DOM** | Đóng gói (encapsulation) style & cấu trúc, cô lập khỏi trang |
| **HTML Templates** | Khối markup parse sẵn nhưng chưa render, clone khi cần |

---

## 2. Custom Element — tạo thẻ riêng

```javascript
class UserCard extends HTMLElement {          // 1) kế thừa HTMLElement
  constructor() {
    super();
    this.attachShadow({ mode: "open" });      // 2) gắn Shadow DOM
    this.shadowRoot.innerHTML = `
      <style>p { color: blue; }</style>        <!-- style cô lập -->
      <p>User Card Component</p>
    `;
  }
}
window.customElements.define("user-card", UserCard);  // 3) đăng ký tên thẻ
```
```html
<user-card></user-card>   <!-- dùng như thẻ HTML bình thường -->
```

> Tên custom element **bắt buộc có dấu gạch ngang** (`user-card`) để không đụng thẻ HTML chuẩn.

---

## 3. Shadow DOM — đóng gói, chống rò rỉ style

Shadow DOM như một "tài liệu ẩn riêng" gắn vào element, **vô hình với bên ngoài**. CSS bên trong **không leak** ra global, và CSS global **không lọt** vào trong.

```javascript
const shadow = element.attachShadow({ mode: "open" });   // "open" = JS ngoài truy cập được shadowRoot
```

| `mode` | Ý nghĩa |
|---|---|
| `open` | `element.shadowRoot` truy cập được từ ngoài |
| `closed` | Ẩn hoàn toàn, ngoài không lấy được `shadowRoot` |

```
★ Insight ─────────────────────────────────────
• Shadow DOM giải đúng nỗi đau CSS toàn cục: trong dự án lớn, class .button ở
  component A vô tình đè .button component B. Style trong Shadow DOM bị "nhốt" lại
  → hết xung đột. Đây là cách trình duyệt làm "scoped CSS" mà framework phải mô
  phỏng bằng hashing (CSS Modules, styled-components — xem [[../06-React/10-Styled-Component]]).
─────────────────────────────────────────────────
```

---

## 4. HTML Template — markup parse sẵn, render sau

```html
<template id="card-tpl">
  <div class="card"><h3></h3><p></p></div>   <!-- parse nhưng CHƯA render -->
</template>
```
```javascript
const tpl = document.getElementById("card-tpl");
const clone = tpl.content.cloneNode(true);    // clone khi cần
clone.querySelector("h3").textContent = "Alice";
document.body.appendChild(clone);             // giờ mới render
```

> `<template>` không hiện trên trang cho tới khi bạn **clone** và chèn vào DOM — lý tưởng để tái dùng cấu trúc phức tạp (xem [[05-Creating-Adding-Managing-Nodes]]).

---

## 5. Web Components vs Framework (React/Vue/Angular)

| Tiêu chí | **Web Components** | **React/Vue/Angular** |
|---|---|---|
| Cần thư viện? | **Không** (chuẩn trình duyệt) | Có |
| Đóng gói style | Shadow DOM (native) | CSS Modules / scoped / CSS-in-JS |
| State management | Tự làm (yếu) | **Mạnh** (hooks, store) |
| Data binding | Thủ công | Tự động |
| Hệ sinh thái/tool | Hạn chế | **Phong phú** |
| Tái dùng giữa framework | **Tốt** (chạy mọi nơi) | Khóa vào framework |

> Web Components hợp để tạo **UI element tái dùng độc lập** (design system dùng chéo nhiều framework). App lớn có state phức tạp vẫn thường chọn React/Vue. Chúng **cộng tác được** — có thể nhúng web component trong app React.

---

## 6. Q&A phỏng vấn

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Web Components là gì, gồm những phần nào?"
> *"Web Components là chuẩn của trình duyệt cho phép tạo thẻ HTML tái sử dụng của riêng mình mà không cần framework. Nó gồm ba phần. Custom Elements là định nghĩa thẻ riêng bằng một class kế thừa HTMLElement rồi đăng ký với customElements.define, tên thẻ bắt buộc có dấu gạch ngang. Shadow DOM là một DOM ẩn gắn vào phần tử để đóng gói style và cấu trúc, giúp style bên trong không rò rỉ ra ngoài và không bị global đè, giống scoped CSS native. HTML Template là khối markup trong thẻ template, được parse sẵn nhưng chưa render cho tới khi clone ra dùng. So với React hay Vue thì Web Components chạy thuần trình duyệt nhưng hệ sinh thái về state và data binding yếu hơn."*

> [!note] 🧠 Mẹo nhớ
> **Web Components = thẻ HTML tự chế, chuẩn trình duyệt.** 3 phần: **Custom Elements** (thẻ riêng) + **Shadow DOM** (đóng gói style, không rò rỉ) + **Template** (markup chờ clone).

> [!question] 1. Web Components gồm những gì?
> 3 phần: **Custom Elements** (thẻ riêng), **Shadow DOM** (đóng gói style/cấu trúc), **HTML Templates** (markup parse sẵn, render khi clone). Là chuẩn trình duyệt, không cần framework.

> [!question] 2. Tạo một custom element thế nào?
> Định nghĩa `class extends HTMLElement`, (tùy chọn) `attachShadow` + set nội dung trong constructor, rồi `customElements.define("ten-the", Class)`. Tên **phải có dấu gạch ngang**.

> [!question] 3. Shadow DOM giải quyết vấn đề gì?
> **Đóng gói (encapsulation)**: style & cấu trúc bên trong **không rò rỉ** ra global và không bị global đè → hết xung đột CSS. Là "scoped CSS" native của trình duyệt.

> [!question] 4. `<template>` khác `<div>` ẩn (`display:none`) ra sao?
> `<template>` được **parse nhưng KHÔNG render** (không tạo node hiển thị, script/ảnh bên trong không chạy/không tải) cho tới khi clone. `<div hidden>` vẫn nằm trong DOM và tài nguyên vẫn được xử lý.

> [!question] 5. Khi nào chọn Web Components, khi nào chọn React?
> Web Components: UI element **tái dùng độc lập**, dùng chéo nhiều framework, không muốn phụ thuộc thư viện. React: app lớn cần **state management/data binding** mạnh và hệ sinh thái phong phú. Có thể kết hợp.

---

## 7. Bài tập tự luyện

1. Tạo `<user-card>` hiển thị `name`/`role` (đọc từ attribute), style trong Shadow DOM màu xanh; chứng minh CSS global không đè được.
2. Dùng `<template>` + `cloneNode` render danh sách 3 card từ một mảng.
3. Thử `mode: "closed"` và kiểm tra `element.shadowRoot` trả `null`.
4. So sánh: viết cùng một "card" bằng Web Component và bằng React component, nhận xét ưu/nhược.

---

## 8. Liên kết
- [[05-Creating-Adding-Managing-Nodes]] — `createElement`, clone node
- [[../06-React/01-React-Overview]] — component model của React
- [[../06-React/10-Styled-Component]] — scoped CSS kiểu framework
- [[00-MOC-DOM|MOC: DOM & Events]]
