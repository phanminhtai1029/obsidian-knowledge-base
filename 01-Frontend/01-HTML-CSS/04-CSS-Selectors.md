---
title: "CSS Selectors"
section: 01-HTML-CSS
tags: [css, selector, specificity, combinator, pseudo-class, fresher, frontend]
related:
  - "[[03-CSS-Overview-va-cu-phap]]"
  - "[[07-CSS-Cascade-Specificity-Inheritance]]"
difficulty: ⭐⭐
estimated_time: 35m
source: [CSS-fundamentals.docx]
---

# CSS Selectors

> [!summary] TL;DR
> CSS Selector xác định **phần tử nào** trong HTML sẽ được style. Các loại chính: type (`p`), class (`.btn`), ID (`#app`), attribute (`[type="email"]`), pseudo-class (`:hover`), pseudo-element (`::before`), và combinator (` `, `>`, `+`, `~`). Specificity của selector quyết định rule nào thắng khi xung đột.

> [!tip] 🎯 Hiểu trong 30 giây
> **Selector = "cách chỉ tay vào đúng phần tử" cần tô style.** Vài loại chính:
> - **type** `p` (mọi thẻ p), **class** `.btn` (phần tử có class btn — *dùng nhiều nhất*), **id** `#app` (đúng 1 phần tử), **attribute** `[type="email"]`.
> - **pseudo-class** `:hover` (trạng thái — khi rê chuột), **pseudo-element** `::before` (tạo phần tử ảo).
> - **combinator** (quan hệ): `A B` (B nằm *bất kỳ đâu* trong A), `A > B` (B là *con trực tiếp* của A), `A + B` (B *ngay sau* A).
>
> Khi nhiều rule chọi nhau, cái nào **specificity** (độ "mạnh") cao hơn thì thắng — id mạnh hơn class, class mạnh hơn type (xem [[07-CSS-Cascade-Specificity-Inheritance]]).

## 1. Khái niệm

Selector là phần đầu của CSS rule, trả lời câu hỏi **"áp dụng style này cho phần tử nào?"**

**Phân loại chính:**
- **Basic selectors:** type, class, ID, universal (`*`), attribute.
- **Combinator selectors:** mô tả quan hệ giữa các phần tử.
- **Pseudo-class:** trạng thái đặc biệt (`:hover`, `:nth-child`).
- **Pseudo-element:** phần tử ảo (`::before`, `::after`, `::first-line`).

## 2. Cú pháp / API

```css
/* === BASIC SELECTORS === */
p        { }   /* Type: tất cả <p> */
.card    { }   /* Class: phần tử có class="card" */
#app     { }   /* ID: phần tử có id="app" — chỉ dùng 1 lần/trang */
*        { }   /* Universal: tất cả phần tử */
[href]   { }   /* Attribute: có thuộc tính href */
[type="submit"] { }  /* Attribute value chính xác */
[class^="btn"]  { }  /* Attribute bắt đầu bằng "btn" */
[href$=".pdf"]  { }  /* Attribute kết thúc bằng ".pdf" */
[class*="icon"] { }  /* Attribute chứa "icon" */

/* === COMBINATOR SELECTORS === */
div p      { }  /* Descendant: <p> bất kỳ trong <div> */
div > p    { }  /* Child: <p> là con trực tiếp của <div> */
h2 + p     { }  /* Adjacent sibling: <p> liền sau <h2> */
h2 ~ p     { }  /* General sibling: mọi <p> cùng cấp sau <h2> */

/* === PSEUDO-CLASS === */
a:hover        { }  /* Khi hover chuột */
input:focus    { }  /* Khi element được focus */
li:first-child { }  /* Con đầu tiên */
li:last-child  { }  /* Con cuối cùng */
li:nth-child(2n) { }  /* Con chẵn (2, 4, 6...) */
li:nth-child(odd){ }  /* Con lẻ */
input:valid    { }  /* Input pass validation */
input:invalid  { }  /* Input fail validation */
p:not(.special){ }  /* <p> KHÔNG có class "special" */

/* === PSEUDO-ELEMENT === */
p::first-line  { }  /* Dòng đầu tiên của <p> */
p::first-letter{ }  /* Chữ cái đầu tiên */
.tooltip::before { content: "★ "; }  /* Thêm nội dung trước element */
.clearfix::after { content: ""; display: block; clear: both; }
```

## 3. Ví dụ minh họa

### Ví dụ 1: Styling navigation với combinators và pseudo-class

```html
<nav class="main-nav">
  <ul>
    <li><a href="/">Trang chủ</a></li>
    <li><a href="/blog" class="active">Blog</a></li>
    <li><a href="/contact">Liên hệ</a></li>
  </ul>
</nav>
```

```css
/* Reset list style */
.main-nav ul {
  list-style: none;
  margin: 0;
  padding: 0;
  display: flex;
  gap: 1rem;
}

/* Type + Class combinator: <a> trong .main-nav */
.main-nav a {
  text-decoration: none;
  color: #333;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  transition: background-color 0.2s;
}

/* Pseudo-class: hover state */
.main-nav a:hover {
  background-color: #f0f0f0;
}

/* Class selector: trang hiện tại */
.main-nav a.active {
  background-color: #0066cc;
  color: white;
}
```

**Output / Kết quả mong đợi:** Nav ngang với link hover effect và active state rõ ràng.

**Giải thích:**
- `.main-nav a` là descendant combinator — chọn `<a>` bất kỳ trong `.main-nav`.
- `a:hover` là pseudo-class — chỉ áp dụng khi người dùng hover chuột.
- `a.active` là compound selector — `<a>` có cả hai class (không có khoảng trắng giữa `a` và `.active`).

### Ví dụ 2: Zebra striping bảng dùng `nth-child`

```html
<table>
  <tbody>
    <tr><td>JavaScript</td><td>Frontend</td></tr>
    <tr><td>Python</td><td>Backend</td></tr>
    <tr><td>SQL</td><td>Database</td></tr>
    <tr><td>Docker</td><td>DevOps</td></tr>
  </tbody>
</table>
```

```css
table { border-collapse: collapse; width: 100%; }
td    { padding: 8px 12px; border: 1px solid #ddd; }

/* Dòng chẵn màu khác */
tbody tr:nth-child(even) {
  background-color: #f8f9fa;
}

/* Hover highlight */
tbody tr:hover {
  background-color: #e3f2fd;
}

/* Cột đầu tiên in đậm */
td:first-child {
  font-weight: bold;
}
```

**Output / Kết quả mong đợi:** Bảng alternating màu, hover highlight, cột đầu in đậm.

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Lỗi phổ biến
> - **Khoảng trắng quan trọng:** `div p` (descendant) ≠ `div> p` (cú pháp lỗi) ≠ `div>p` (child — đúng).
> - **Overuse ID selector:** ID (`#app`) có specificity rất cao (100 điểm), khó override. Ưu tiên class.
> - **Selector quá cụ thể:** `header nav ul li a:hover` — 5 cấp lồng nhau, rất khó override sau. Giới hạn 3 cấp.
> - **`::before`/`::after` cần `content`:** Pseudo-element không render nếu thiếu `content: ""`.

> [!tip] Mẹo
> Dùng [Specificity Calculator](https://specificity.keegan.st/) để tính điểm specificity nhanh khi debug.

```
★ Insight ─────────────────────────────────────
• 4 combinator chỉ khác nhau ở "khoảng cách quan hệ": (space)=hậu duệ bất kỳ
  cấp, `>`=con TRỰC TIẾP, `+`=anh em LIỀN sau, `~`=mọi anh em sau. Đọc selector
  từ PHẢI sang TRÁI (trình duyệt cũng vậy): `div > p` = "tìm mọi p, giữ lại p
  nào có cha trực tiếp là div" — tư duy này giúp đọc selector phức tạp không loạn.
• Bộ ba hiện đại đáng nhớ: `:is(a, b)` gom nhóm cho gọn (lấy specificity cao nhất
  trong nhóm), `:where(...)` y hệt nhưng specificity = 0 (dễ override), và `:has()`
  — selector "cha" đầu tiên của CSS: `.card:has(img)` chọn card NÀO chứa ảnh.
  `:has()` mở ra rất nhiều thứ trước đây buộc phải dùng JS.
─────────────────────────────────────────────────
```

## 5. Câu hỏi phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Class selector vs ID selector?"
> *"Class selector viết với dấu chấm, ví dụ chấm btn, có thể dùng lại trên nhiều phần tử trong cùng một trang, specificity là mười. ID selector viết với dấu thăng, ví dụ thăng app, chỉ nên dùng đúng một lần mỗi trang vì id phải duy nhất, specificity là một trăm nên rất mạnh. Trong styling em ưu tiên class vì tái sử dụng được và dễ override, còn id specificity quá cao dễ gây khó ghi đè về sau, nên em hạn chế dùng id để style, id chủ yếu để JS truy cập hoặc làm anchor. Khi nhiều rule chọi nhau thì cái có specificity cao hơn thắng, nên giữ specificity thấp và nhất quán giúp CSS dễ bảo trì."*

> [!note] 🧠 Mẹo nhớ
> **Class `.btn` = dùng nhiều lần (specificity 10, ưu tiên style); ID `#app` = 1 lần/trang (specificity 100, mạnh → khó override, hạn chế dùng style).** Combinator: `A B` (hậu duệ) vs `A > B` (con trực tiếp).

1. **Q:** Sự khác biệt giữa class selector và ID selector?
   **A:** Class (`.btn`) có thể dùng nhiều lần trên một trang, specificity = 10. ID (`#app`) chỉ dùng **một lần** mỗi trang, specificity = 100. Trong CSS, ID hiếm khi dùng vì specificity quá cao gây khó override.

2. **Q:** `div > p` và `div p` khác nhau thế nào?
   **A:** `div p` (descendant) chọn **mọi** `<p>` bên trong `<div>`, dù lồng sâu bao nhiêu cấp. `div > p` (child) chỉ chọn `<p>` là **con trực tiếp** của `<div>`.

3. **Q:** Pseudo-class và pseudo-element khác nhau thế nào?
   **A:** **Pseudo-class** (`:hover`, `:focus`, `:nth-child`) — trạng thái hoặc vị trí của phần tử **thực**. **Pseudo-element** (`::before`, `::after`, `::first-line`) — tạo ra **phần tử ảo** không tồn tại trong DOM. Pseudo-element dùng `::` (2 dấu hai chấm) theo CSS3.

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Viết CSS cho một card list: mỗi card `.card` có border, padding. Card thứ nhất (`:first-child`) có border màu vàng. Card khi hover có `box-shadow`. Dùng `::before` của card để hiển thị số thứ tự từ `counter`.
- [ ] **Bài 2:** Style một form: input `:valid` có border xanh, `:invalid` có border đỏ, `:focus` có `outline` rõ ràng. Label liền trước input lỗi (`:invalid + label` hoặc dùng JS class) đổi màu đỏ.

## 7. Liên kết

- Note liên quan: [[03-CSS-Overview-va-cu-phap]], [[07-CSS-Cascade-Specificity-Inheritance]], [[05-CSS-Box-Model]]
- [MDN: CSS selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_selectors)
- [CSS Tricks: A Complete Guide to CSS Selectors](https://css-tricks.com/almanac/selectors/)
- [Specificity Calculator](https://specificity.keegan.st/)
