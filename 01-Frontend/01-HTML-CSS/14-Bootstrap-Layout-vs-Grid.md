---
title: "Bootstrap Layout & Grid System"
section: 01-HTML-CSS
tags: [bootstrap, grid, layout, responsive, breakpoint, fresher, frontend]
related:
  - "[[13-Bootstrap-Overview]]"
  - "[[09-CSS-Layout-Grid]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [Bootstrap 5 docs]
---

# Bootstrap Layout & Grid System

> [!summary] TL;DR
> Bootstrap Grid dùng hệ thống **12 cột** với cú pháp `col-{breakpoint}-{size}`. Cần wrap trong `.row` (trong `.container`). **Breakpoints:** xs(<576), sm(≥576), md(≥768), lg(≥1024), xl(≥1280), xxl(≥1400). `col-md-6` = chiếm 6/12 cột từ breakpoint md trở lên, nhỏ hơn thì full width.

## 1. Khái niệm

### Container
Wrapper ngoài cùng để center content và set max-width:

| Class | Width |
|-------|-------|
| `.container` | Responsive max-width theo breakpoint |
| `.container-fluid` | 100% width luôn |
| `.container-{bp}` | 100% đến breakpoint, max-width từ đó lên |

### Row
`.row` tạo flex container, handles negative margin để offset `.container` padding.

### Column
`.col-*` là flex item. Hệ thống 12 cột — `col-4` chiếm 4/12 = 1/3.

### Breakpoints Bootstrap 5

| Name | Min-width | Class infix |
|------|-----------|-------------|
| Extra small | < 576px | (none) |
| Small | ≥ 576px | `sm` |
| Medium | ≥ 768px | `md` |
| Large | ≥ 992px | `lg` |
| Extra large | ≥ 1200px | `xl` |
| XXL | ≥ 1400px | `xxl` |

## 2. Cú pháp / API

```html
<!-- Cấu trúc cơ bản: container > row > col -->
<div class="container">
  <div class="row">
    <div class="col-md-6">Cột trái</div>
    <div class="col-md-6">Cột phải</div>
  </div>
</div>

<!-- ── CÁC CÁCH ĐỊNH NGHĨA COLUMN ── -->

<!-- 1. Auto width: chia đều -->
<div class="row">
  <div class="col">Auto 1/3</div>
  <div class="col">Auto 1/3</div>
  <div class="col">Auto 1/3</div>
</div>

<!-- 2. Số cột cụ thể: tổng phải = 12 -->
<div class="row">
  <div class="col-4">4 cột</div>
  <div class="col-8">8 cột</div>
</div>

<!-- 3. Responsive: thay đổi theo breakpoint -->
<div class="row">
  <!-- Mobile: full width | Tablet: 6/12 | Desktop: 4/12 -->
  <div class="col-12 col-md-6 col-lg-4">Card</div>
  <div class="col-12 col-md-6 col-lg-4">Card</div>
  <div class="col-12 col-md-12 col-lg-4">Card</div>
</div>

<!-- 4. Row cols (Bootstrap 5): định nghĩa trên row -->
<div class="row row-cols-1 row-cols-md-2 row-cols-lg-3 g-4">
  <div class="col"><div class="card">Card 1</div></div>
  <div class="col"><div class="card">Card 2</div></div>
  <div class="col"><div class="card">Card 3</div></div>
</div>

<!-- 5. Offset: đẩy cột sang phải -->
<div class="row">
  <div class="col-md-6 offset-md-3">Căn giữa bằng offset</div>
</div>

<!-- 6. Order: đổi thứ tự hiển thị -->
<div class="row">
  <div class="col order-md-2">Thứ nhất trong HTML, thứ 2 trên md+</div>
  <div class="col order-md-1">Thứ hai trong HTML, thứ 1 trên md+</div>
</div>

<!-- Gap giữa các col: g-{size} (0-5) -->
<div class="row g-3">          <!-- gap: 1rem -->
<div class="row gx-4 gy-2">   <!-- column-gap: 1.5rem, row-gap: 0.5rem -->
```

## 3. Ví dụ minh họa

### Ví dụ 1: Layout blog phổ biến (sidebar + content)

```html
<div class="container py-4">
  <div class="row g-4">

    <!-- Main content: 8 cột trên md+, full width trên mobile -->
    <div class="col-md-8">
      <article class="card">
        <div class="card-body">
          <h1 class="h3">Tiêu đề bài viết</h1>
          <p class="text-muted small">
            <span>01 Tháng 5, 2025</span> · <span>5 phút đọc</span>
          </p>
          <p>Nội dung bài viết dài...</p>
        </div>
      </article>
    </div>

    <!-- Sidebar: 4 cột trên md+, full width trên mobile
         Trên mobile sẽ xuất hiện SAU main content -->
    <div class="col-md-4">
      <div class="card mb-3">
        <div class="card-body">
          <h5 class="card-title">Bài viết liên quan</h5>
          <ul class="list-unstyled">
            <li><a href="#">CSS Grid cơ bản</a></li>
            <li><a href="#">Flexbox cho mọi người</a></li>
          </ul>
        </div>
      </div>
    </div>

  </div>
</div>
```

**Output / Kết quả mong đợi:**
- Mobile: content 100% + sidebar 100% (stacked)
- Desktop: content 66% + sidebar 33% (side by side)

### Ví dụ 2: Responsive card grid với `row-cols`

```html
<div class="container py-5">
  <h2 class="mb-4">Khóa học</h2>

  <!-- row-cols: định nghĩa số cột trên row, không cần đặt class trên mỗi col -->
  <div class="row row-cols-1 row-cols-sm-2 row-cols-xl-4 g-4">

    <div class="col">
      <div class="card h-100">
        <img src="course1.jpg" class="card-img-top" alt="HTML CSS">
        <div class="card-body d-flex flex-column">
          <h5 class="card-title">HTML & CSS</h5>
          <p class="card-text text-muted flex-grow-1">Nền tảng frontend...</p>
          <a href="#" class="btn btn-primary mt-auto">Xem khóa học</a>
        </div>
      </div>
    </div>

    <!-- Thêm 3 card tương tự -->

  </div>
</div>
```

**Output / Kết quả mong đợi:**
- Mobile: 1 card/hàng
- Small: 2 card/hàng
- XL+: 4 card/hàng
- Tất cả card cùng chiều cao (`h-100`) nhờ flex column + `mt-auto` trên button

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Lỗi phổ biến
> - **Quên `.row` giữa `.container` và `.col`:** `.col-*` PHẢI là con trực tiếp của `.row`. Thiếu `.row` → layout bị lệch.
> - **Tổng cột > 12:** `col-8 + col-6 = 14` → col thứ hai xuống hàng. Tổng cột trong 1 row phải = 12 hoặc dùng `col` (auto).
> - **Class responsive không cộng dồn đúng:** Bootstrap mobile-first — `col-md-6` nghĩa là "từ md trở lên chiếm 6 cột, nhỏ hơn chiếm full width (12)". Cần ghi rõ `col-12 col-md-6` nếu muốn explicit.
> - **`container-fluid` trong `container`:** Không lồng `.container` trong `.container` — chỉ dùng 1 loại container ngoài cùng.

## 5. Câu hỏi phỏng vấn thường gặp

1. **Q:** `col-md-6` có nghĩa là gì?
   **A:** Từ breakpoint `md` (≥768px) trở lên, element chiếm **6/12 cột** (50%). Nhỏ hơn md, element chiếm full width (12/12). Có thể combine: `col-12 col-md-6 col-lg-4` = mobile full, tablet half, desktop third.

2. **Q:** `.container` và `.container-fluid` khác nhau thế nào?
   **A:** `.container` có `max-width` thay đổi theo breakpoint (576→540px, 768→720px...) và `margin: 0 auto` để center. `.container-fluid` luôn `width: 100%` fill viewport. Dùng `container-fluid` cho full-bleed backgrounds.

3. **Q:** `row-cols-{n}` vs đặt `col-*` trên từng item khác gì?
   **A:** `row-cols-3` trên row: mọi item con tự động chiếm 1/3 — không cần class trên từng item. `col-4` trên từng item: cần đặt class thủ công. `row-cols` tiện hơn khi render danh sách items từ data.

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Tạo layout "magazine style" dùng Bootstrap Grid: hàng 1 có 1 ảnh lớn (`col-md-8`) + sidebar (`col-md-4`). Hàng 2 có 3 card bằng nhau. Hàng 3 có 2 cột 50/50.
- [ ] **Bài 2:** Thực hành **reorder column** trên mobile. HTML có order: Hình > Nội dung > Sidebar. Trên desktop phải hiển thị: Sidebar trái | Nội dung | Hình phải. Dùng `order-*` utilities.

## 7. Liên kết

- Note liên quan: [[13-Bootstrap-Overview]], [[15-Bootstrap-Components]], [[08-CSS-Layout-Flexbox]]
- [Bootstrap 5: Grid system](https://getbootstrap.com/docs/5.3/layout/grid/)
- [Bootstrap 5: Breakpoints](https://getbootstrap.com/docs/5.3/layout/breakpoints/)
- [Bootstrap 5: Columns](https://getbootstrap.com/docs/5.3/layout/columns/)
