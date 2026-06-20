---
title: "CSS Layout: Grid"
section: 01-HTML-CSS
tags: [css, grid, layout, two-dimensional, fresher, frontend]
related:
  - "[[08-CSS-Layout-Flexbox]]"
  - "[[10-CSS-Responsive-Media-Query]]"
difficulty: ⭐⭐⭐
estimated_time: 50m
source: [CSS-fundamentals.docx]
---

# CSS Layout: Grid

> [!summary] TL;DR
> CSS Grid là layout system **2 chiều** (rows + columns), cho phép đặt element chính xác vào bất kỳ ô nào. Container dùng `display: grid` + `grid-template-columns/rows`. Item dùng `grid-column` / `grid-row` để span qua nhiều ô. Dùng `fr` unit để chia không gian linh hoạt.

## 1. Khái niệm

**CSS Grid** — hệ thống layout 2 chiều đầu tiên thực sự mạnh mẽ của CSS. Khác với Flexbox (1 chiều), Grid kiểm soát cả rows và columns đồng thời.

**Thuật ngữ:**
- **Grid container:** Element có `display: grid`.
- **Grid item:** Con trực tiếp của container.
- **Grid line:** Đường kẻ chia grid (đánh số từ 1).
- **Grid track:** Khoảng giữa 2 grid line (= row hoặc column).
- **Grid cell:** Giao điểm của 1 row track và 1 column track.
- **Grid area:** Vùng gồm nhiều cells.

## 2. Cú pháp / API

```css
/* ── CONTAINER ── */
.grid {
  display: grid;

  /* Định nghĩa columns */
  grid-template-columns: 200px 1fr 1fr;       /* 3 cột: 200px cố định + 2 cột chia đều */
  grid-template-columns: repeat(3, 1fr);       /* 3 cột bằng nhau */
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); /* Responsive không cần media query */

  /* Định nghĩa rows */
  grid-template-rows: 60px 1fr auto;           /* Header + content + footer */

  gap: 1.5rem;                                 /* row-gap + column-gap */
  row-gap: 1rem;
  column-gap: 2rem;

  /* Đặt tên areas — layout trực quan */
  grid-template-areas:
    "header header header"
    "sidebar main   main"
    "footer footer footer";

  justify-items: stretch;   /* Căn item theo column axis */
  align-items: stretch;     /* Căn item theo row axis */
}

/* ── ITEM ── */
.item {
  /* Đặt vị trí theo grid line */
  grid-column: 1 / 3;       /* Từ line 1 đến line 3 (span 2 cột) */
  grid-column: 1 / -1;      /* Từ đầu đến cuối */
  grid-column: span 2;       /* Chiếm 2 cột từ vị trí hiện tại */

  grid-row: 1 / 3;           /* Span 2 rows */

  /* Hoặc đặt theo named area */
  grid-area: header;

  justify-self: center;      /* Override justify-items cho item này */
  align-self: end;
}
```

## 3. Ví dụ minh họa

### Ví dụ 1: Holy Grail layout với named areas

```html
<div class="page-layout">
  <header class="header">Header</header>
  <nav class="sidebar">Sidebar</nav>
  <main class="main-content">Main Content</main>
  <footer class="footer">Footer</footer>
</div>
```

```css
.page-layout {
  display: grid;
  grid-template-columns: 240px 1fr;     /* Sidebar cố định + content fill */
  grid-template-rows: 64px 1fr 60px;   /* Header + content + footer */
  grid-template-areas:
    "header  header"
    "sidebar main"
    "footer  footer";
  min-height: 100vh;
  gap: 0;
}

.header  { grid-area: header;  background: #0066cc; color: white; padding: 0 2rem; display: flex; align-items: center; }
.sidebar { grid-area: sidebar; background: #f8f9fa; padding: 1rem; }
.main-content { grid-area: main; padding: 2rem; }
.footer  { grid-area: footer;  background: #333; color: white; text-align: center; line-height: 60px; }
```

**Output / Kết quả mong đợi:** Layout hoàn chỉnh với header full-width, sidebar trái cố định 240px, content fill còn lại, footer full-width.

**Giải thích:**
- `grid-template-areas` dùng tên để đặt items — trực quan hơn line number.
- `1fr` fill không gian còn lại sau sidebar.

### Ví dụ 2: Responsive image gallery không cần media query

```html
<div class="gallery">
  <img src="1.jpg" alt="" class="featured">
  <img src="2.jpg" alt="">
  <img src="3.jpg" alt="">
  <img src="4.jpg" alt="">
  <img src="5.jpg" alt="">
</div>
```

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
}

.gallery img {
  width: 100%;
  height: 200px;
  object-fit: cover;
  border-radius: 8px;
}

/* Ảnh đầu tiên span 2 cột — ảnh nổi bật */
.gallery .featured {
  grid-column: span 2;
  height: 300px;
}
```

**Output / Kết quả mong đợi:** Gallery tự động co giãn số cột theo viewport. Ảnh đầu tiên rộng gấp đôi.

**Giải thích:** `repeat(auto-fit, minmax(200px, 1fr))` — "tạo nhiều cột nhất có thể với mỗi cột tối thiểu 200px, chia đều không gian còn lại". Responsive hoàn toàn không cần media query.

```
★ Insight ─────────────────────────────────────
• `repeat(auto-fit, minmax(200px, 1fr))` là "responsive 1 dòng" mạnh nhất CSS:
  trình duyệt tự tính số cột vừa với viewport, không cần một breakpoint nào. Nắm
  được idiom này, 80% card-grid/gallery khỏi cần media query.
• auto-fit vs auto-fill quyết định bởi: KHI thiếu item, auto-fit COLLAPSE cột
  trống (item nở ra full) còn auto-fill GIỮ cột trống (item giữ nguyên 200px).
  Muốn item luôn dãn đầy hàng → auto-fit; muốn bảo toàn kích thước item, chừa
  chỗ cho item sau → auto-fill.
─────────────────────────────────────────────────
```

### Bảng chọn nhanh: Flexbox hay Grid?

| Tình huống | Chọn | Vì sao |
|---|---|---|
| Nav bar, button group, hàng card 1 dòng | **Flexbox** | Bố cục 1 chiều, nội dung quyết định kích thước |
| Layout toàn trang, dashboard, gallery lưới | **Grid** | Bố cục 2 chiều, đặt item chính xác theo hàng+cột |
| Căn giữa 1 phần tử | Cả hai OK | `place-items: center` (grid) hoặc `justify/align` (flex) |
| Cần item tự xuống hàng theo nội dung | **Flexbox** `wrap` | Item tự gói theo chiều rộng còn lại |
| Cần các ô thẳng hàng cả 2 chiều | **Grid** | Track cố định giữ thẳng hàng tuyệt đối |

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Lỗi phổ biến
> - **Grid không áp dụng cho cháu:** `display: grid` chỉ tạo context cho **con trực tiếp**. Cháu muốn dùng grid phải tạo grid riêng trên parent của nó.
> - **Nhầm `auto-fit` và `auto-fill`:** `auto-fill` tạo cột trống nếu không đủ item. `auto-fit` collapse cột trống — dùng `auto-fit` cho gallery/card grid.
> - **Quên `minmax()` khi dùng `fr`:** `1fr` trên column có thể làm item bị overflow nếu nội dung quá rộng. `minmax(0, 1fr)` an toàn hơn.
> - **`gap` ảnh hưởng kích thước:** Grid gap tính THÊM vào layout — không cần tính toán thủ công như `margin`.

## 5. Câu hỏi phỏng vấn thường gặp

1. **Q:** Sự khác nhau giữa CSS Grid và Flexbox?
   **A:** **Flexbox** là layout **1 chiều** — sắp xếp items theo row hoặc column. **Grid** là layout **2 chiều** — kiểm soát cả rows và columns đồng thời. Dùng Grid cho page layout tổng thể (macro), Flexbox cho components bên trong (micro).

2. **Q:** `fr` unit là gì?
   **A:** `fr` (fraction) đại diện cho một phần của **không gian còn lại** sau khi trừ các kích thước cố định. `1fr 2fr` — cột đầu chiếm 1/3, cột sau chiếm 2/3 không gian còn lại.

3. **Q:** `auto-fit` khác `auto-fill` thế nào trong `repeat()`?
   **A:** Với 3 items và 5 cột có thể fit: `auto-fill` tạo 5 cột (2 cột trống). `auto-fit` tạo 5 cột rồi **collapse** 2 cột trống về 0 — items grow fill hết không gian. Dùng `auto-fit` cho card grid đáp ứng.

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Tạo dashboard layout: header (full width), sidebar trái 280px, main content, widget panel phải 320px, footer full width. Dùng `grid-template-areas` với named areas.
- [ ] **Bài 2:** Tạo magazine layout: cột 1 có ảnh lớn span 2 rows, cột 2 có 2 bài nhỏ xếp dọc. Hint: `grid-template-rows: 200px 200px` + `grid-row: 1 / 3` cho ảnh lớn.

## 7. Liên kết

- Note liên quan: [[08-CSS-Layout-Flexbox]], [[10-CSS-Responsive-Media-Query]]
- [MDN: CSS Grid Layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout)
- [CSS Tricks: A Complete Guide to CSS Grid](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [Grid Garden (game học Grid)](https://cssgridgarden.com/)
