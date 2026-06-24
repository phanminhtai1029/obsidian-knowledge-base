---
title: "CSS Layout: Flexbox"
section: 01-HTML-CSS
tags: [css, flexbox, layout, responsive, fresher, frontend]
related:
  - "[[09-CSS-Layout-Grid]]"
  - "[[10-CSS-Responsive-Media-Query]]"
difficulty: ⭐⭐⭐
estimated_time: 50m
source: [CSS-fundamentals.docx]
---

# CSS Layout: Flexbox

> [!summary] TL;DR
> Flexbox là layout module **1 chiều** (row hoặc column), giúp phân bổ không gian và căn chỉnh các item trong container. Container dùng `display: flex` + `justify-content` (main axis) + `align-items` (cross axis). Item dùng `flex: grow shrink basis` để kiểm soát tỷ lệ chiếm không gian.

> [!tip] 🎯 Hiểu trong 30 giây
> **Flexbox = công cụ xếp các phần tử thành MỘT hàng hoặc MỘT cột và căn chỉnh chúng dễ dàng** (việc mà `float` ngày xưa làm rất khổ). Bạn đặt `display: flex` lên *cái khung cha*, rồi các con tự xếp thành hàng.
> - **1 chiều** = hoặc theo *hàng ngang* (`row` — mặc định), hoặc *cột dọc* (`column`), tại một thời điểm.
> - 2 núm căn chỉnh hay dùng: **`justify-content`** (căn theo *trục chính* — ngang nếu là row: dồn trái/phải/giữa/cách đều) và **`align-items`** (căn theo *trục vuông góc* — dọc nếu là row: trên/giữa/dưới). Muốn căn *giữa hoàn hảo* cả 2 chiều: `justify-content: center; align-items: center`.
> - `flex: 1` trên item = "co giãn lấp đầy phần trống còn lại" (chia đều).
>
> **Khi nào dùng (ra thi):** Flexbox cho bố cục **1 chiều** — thanh nav, hàng nút, *sidebar cố định + main co giãn*. Bố cục **2 chiều** (cả hàng lẫn cột) → dùng **Grid** ([[09-CSS-Layout-Grid]]).

## 1. Khái niệm

**Flexbox** (Flexible Box Layout) giải quyết bài toán phân bổ không gian và căn chỉnh — điều mà `float` và `inline-block` làm rất kém.

**Hai vai trò:**
- **Flex container:** Element cha — `display: flex`. Kiểm soát layout của các item.
- **Flex item:** Element con trực tiếp của container. Tự động trở thành flex item.

**Hai trục:**
- **Main axis:** Chiều chính — `flex-direction: row` (ngang, default) hoặc `column` (dọc).
- **Cross axis:** Chiều vuông góc với main axis.

## 2. Cú pháp / API

```css
/* ── CONTAINER PROPERTIES ── */
.container {
  display: flex;                /* Kích hoạt flexbox */
  flex-direction: row;          /* row | row-reverse | column | column-reverse */
  flex-wrap: nowrap;            /* nowrap | wrap | wrap-reverse */
  gap: 1rem;                    /* Khoảng cách giữa items (row-gap + column-gap) */

  /* Căn chỉnh trên MAIN AXIS */
  justify-content: flex-start;  /* flex-start | flex-end | center | space-between | space-around | space-evenly */

  /* Căn chỉnh trên CROSS AXIS */
  align-items: stretch;         /* stretch | flex-start | flex-end | center | baseline */

  /* Căn chỉnh nhiều dòng (khi flex-wrap) */
  align-content: flex-start;    /* (tương tự justify-content) */
}

/* ── ITEM PROPERTIES ── */
.item {
  flex-grow: 0;    /* Tỷ lệ chiếm thêm không gian thừa (0 = không grow) */
  flex-shrink: 1;  /* Tỷ lệ thu nhỏ khi thiếu không gian (1 = co lại) */
  flex-basis: auto;/* Kích thước ban đầu trước khi grow/shrink */
  flex: 1;         /* Shorthand: flex-grow=1, flex-shrink=1, flex-basis=0 */

  align-self: auto; /* Override align-items cho item riêng lẻ */
  order: 0;         /* Thay đổi thứ tự hiển thị (không đổi DOM order) */
}
```

## 3. Ví dụ minh họa

### Ví dụ 1: Navigation bar chuẩn với Flexbox

```html
<header class="header">
  <div class="logo">MyApp</div>
  <nav class="nav">
    <a href="/">Trang chủ</a>
    <a href="/blog">Blog</a>
    <a href="/about">Giới thiệu</a>
  </nav>
  <button class="btn-cta">Đăng nhập</button>
</header>
```

```css
.header {
  display: flex;
  align-items: center;       /* Căn giữa theo chiều dọc */
  justify-content: space-between; /* Logo trái, nav giữa, button phải */
  padding: 0 2rem;
  height: 64px;
  background: white;
  box-shadow: 0 1px 4px rgba(0,0,0,0.1);
}

.logo {
  font-size: 1.5rem;
  font-weight: 700;
  color: #0066cc;
}

.nav {
  display: flex;
  gap: 1.5rem;              /* Khoảng cách đều giữa các link */
}

.nav a {
  text-decoration: none;
  color: #333;
  font-size: 0.95rem;
}

.btn-cta {
  padding: 8px 20px;
  background: #0066cc;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}
```

**Output / Kết quả mong đợi:** Header ngang: logo bên trái, nav ở giữa, button bên phải — tất cả căn giữa theo chiều dọc.

### Ví dụ 2: Card grid responsive với flex-wrap

```html
<div class="card-grid">
  <div class="card">Card 1</div>
  <div class="card">Card 2</div>
  <div class="card">Card 3</div>
  <div class="card">Card 4</div>
  <div class="card">Card 5</div>
</div>
```

```css
.card-grid {
  display: flex;
  flex-wrap: wrap;      /* Cho phép xuống hàng */
  gap: 1.5rem;
}

.card {
  flex: 1 1 280px;      /* grow=1, shrink=1, basis=280px */
  /* Nghĩa: mỗi card chiếm ít nhất 280px, có thể grow đều nhau */
  padding: 1.5rem;
  background: white;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
}
```

**Output / Kết quả mong đợi:** Màn hình rộng: 3-4 card/hàng. Màn hình nhỏ: xuống 2 hoặc 1 card/hàng — không cần media query.

**Giải thích:** `flex: 1 1 280px` — card có kích thước tối thiểu 280px, tự động fill không gian còn lại đều nhau.

```
★ Insight ─────────────────────────────────────
• Toàn bộ sự bối rối "justify hay align?" biến mất nếu nhớ DUY NHẤT một câu:
  justify-content đi theo MAIN axis, align-items đi theo CROSS axis. Khi đổi
  flex-direction: column, main axis xoay dọc → justify giờ căn DỌC, align căn
  NGANG. Đừng học thuộc "ngang/dọc", học thuộc "main/cross" rồi suy ra.
• flex-basis thắng width: khi item là flex item, basis quyết định kích thước
  khởi điểm trước grow/shrink, lấn át cả width. Vì vậy `flex: 1 1 280px` đúng
  hơn `width: 280px` — basis "gợi ý" 280px nhưng cho phép co giãn, còn width
  thường cứng nhắc hơn trong ngữ cảnh flex.
─────────────────────────────────────────────────
```

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Lỗi phổ biến
> - **`flex: 1` không hoạt động:** Đảm bảo parent có `display: flex`. Flex chỉ áp dụng cho **con trực tiếp**.
> - **`gap` vs `margin`:** Dùng `gap` trên container thay vì `margin` trên item — `gap` không tạo khoảng cách ngoài cùng, `margin` tạo khoảng ngoài rìa container.
> - **`align-items` vs `justify-content` nhầm trục:** `justify-content` = main axis, `align-items` = cross axis. Khi `flex-direction: column`, main axis là dọc, cross axis là ngang — ngược với `row`.
> - **Quên `min-width: 0` trên flex item:** Flex item mặc định `min-width: auto` (không co xuống nhỏ hơn nội dung). Text dài hoặc `overflow: hidden` không hoạt động nếu thiếu `min-width: 0`.

## 5. Câu hỏi phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Sidebar trái cố định + main phải co giãn: dùng Flexbox hay Grid?"
> *"Với bố cục một hàng gồm sidebar cố định bên trái và main co giãn bên phải, em dùng Flexbox vì đây là layout một chiều. Em đặt display flex cho khung cha, cho sidebar một chiều rộng cố định ví dụ width 250px kèm flex-shrink 0 để nó không bị co lại, rồi cho main flex 1 để nó tự lấp hết phần không gian còn lại. Như vậy sidebar luôn giữ đúng bề rộng còn main co giãn theo màn hình. Em chọn Flexbox thay vì Grid vì bài toán chỉ trải theo một trục ngang; nếu cần kiểm soát cả hàng lẫn cột, ví dụ layout tổng thể header sidebar content footer, thì em mới dùng Grid."*
>
> ```css
> .layout { display: flex; }
> .sidebar { width: 250px; flex-shrink: 0; }  /* cố định, không co */
> .main    { flex: 1; }                        /* co giãn lấp phần còn lại */
> ```

> [!note] 🧠 Mẹo nhớ
> **Flexbox = 1 chiều (hàng HOẶC cột).** `justify-content` = trục chính, `align-items` = trục vuông góc; căn giữa = cả hai `center`. **Sidebar+main = Flexbox (`width` cố định + `flex:1`); layout 2 chiều = Grid.**

1. **Q:** `justify-content` và `align-items` khác nhau thế nào?
   **A:** `justify-content` căn chỉnh items theo **main axis** (ngang nếu `flex-direction: row`). `align-items` căn chỉnh theo **cross axis** (dọc nếu `flex-direction: row`). Khi `flex-direction: column`, chúng đảo vai trò.

2. **Q:** `flex: 1` nghĩa là gì?
   **A:** Shorthand cho `flex-grow: 1; flex-shrink: 1; flex-basis: 0%`. Các item có `flex: 1` sẽ **chia đều** không gian có sẵn trong container.

3. **Q:** Khi nào dùng Flexbox, khi nào dùng Grid?
   **A:** **Flexbox** cho layout **1 chiều** — nav bar, button group, card row. **Grid** cho layout **2 chiều** — toàn bộ trang, gallery, dashboard. Thực tế hay kết hợp: Grid cho macro layout, Flexbox cho micro layout bên trong.

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Dùng Flexbox tạo layout "Holy Grail" (header + main row gồm left sidebar, content, right sidebar + footer). Sidebar có `width: 240px`, content `flex: 1` để fill còn lại.
- [ ] **Bài 2:** Tạo pricing card row: 3 card cùng chiều cao (tự động). Card giữa có `transform: scale(1.05)` và `z-index: 1` để nổi bật. Dùng `align-items: stretch` để đảm bảo tất cả card cùng chiều cao.

## 7. Liên kết

- Note liên quan: [[09-CSS-Layout-Grid]], [[10-CSS-Responsive-Media-Query]], [[05-CSS-Box-Model]]
- [MDN: Flexbox](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout)
- [CSS Tricks: A Complete Guide to Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [Flexbox Froggy (game học Flexbox)](https://flexboxfroggy.com/)
