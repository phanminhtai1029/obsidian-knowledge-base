---
title: "Bootstrap Utilities"
section: 01-HTML-CSS
tags: [bootstrap, utilities, helpers, fresher, frontend]
related:
  - "[[13-Bootstrap-Overview]]"
  - "[[14-Bootstrap-Layout-vs-Grid]]"
  - "[[15-Bootstrap-Components]]"
difficulty: ⭐⭐
estimated_time: 30m
source: [Bootstrap 5 docs]
---

# Bootstrap 5 — Utilities (Helper Classes)

> [!tip] 🎯 Hiểu trong 30 giây
> **Utility class = "class tí hon làm đúng 1 việc"**, ghép nhiều cái lại để style ngay trên HTML mà khỏi viết file CSS. Ví dụ `m-3` (margin), `p-2` (padding), `d-flex` (display flex), `text-center` (canh giữa chữ), `bg-primary` (nền xanh). Đây chính là tư tưởng *atomic CSS* (giống Tailwind).
> - Hầu hết có **biến thể responsive** theo breakpoint: `d-none d-md-block` = ẩn ở mobile, hiện từ `md` trở lên; `text-md-center`.
> - Lợi: dựng nhanh, nhất quán, không phình file CSS. Đổi lại: HTML có thể dài ra vì nhiều class.

> [!summary] TL;DR
> Bootstrap Utilities là các **atomic CSS classes** — mỗi class làm đúng 1 việc. Kết hợp chúng để style nhanh mà không cần viết CSS tùy chỉnh. Hầu hết đều có phiên bản responsive (`d-md-none`, `text-lg-center`).

---

## 1. Khái niệm

**Utility classes** (còn gọi là helper classes) là CSS classes một mục đích — mỗi class map trực tiếp tới một CSS property/value:

```text
.mt-3   →   margin-top: 1rem
.d-flex →   display: flex
.fw-bold →  font-weight: 700
```

> [!tip] Triết lý Atomic CSS
> Bootstrap Utilities giống **Tailwind CSS** về tư tưởng: thay vì viết class `.product-card { display: flex; ... }`, ta ghép `d-flex gap-3 align-items-center`. Ưu điểm: không phải đặt tên, không bị CSS conflict.

```
★ Insight ─────────────────────────────────────
• Utility class chỉ là đường tắt cho 1 dòng CSS bạn ĐÃ biết: `mt-3`=margin-top
  1rem, `d-flex`=display:flex, `text-center`=text-align:center. Học utilities
  KHÔNG phải học cái mới — chỉ là tra bảng tên gọi cho kiến thức CSS gốc. Nắm
  CSS thật rồi thì Tailwind/Bootstrap utility học trong một buổi.
• Tradeoff cốt lõi (hay hỏi phỏng vấn): utility-first khiến HTML "rậm" class
  nhưng đổi lại KHÔNG cần đặt tên, KHÔNG file CSS phình theo thời gian, và xóa
  component là xóa sạch style của nó. Component-class (.card) thì HTML sạch
  nhưng CSS dễ phình và "dead CSS" tích tụ. Không có bên nào "đúng" tuyệt đối.
─────────────────────────────────────────────────
```

### Hệ thống spacing (t-shirt sizing)

| Class suffix | Value |
|---|---|
| `0` | 0 |
| `1` | 0.25rem (4px) |
| `2` | 0.5rem (8px) |
| `3` | 1rem (16px) |
| `4` | 1.5rem (24px) |
| `5` | 3rem (48px) |
| `auto` | auto |

---

## 2. Cú pháp & Class quan trọng

### 2.1 Spacing — Margin & Padding

```html
<!-- Margin: m{side}-{size} -->
<div class="mt-3">margin-top: 1rem</div>
<div class="mb-4">margin-bottom: 1.5rem</div>
<div class="mx-auto">margin-left: auto; margin-right: auto (căn giữa block)</div>
<div class="ms-auto">margin-left: auto (đẩy sang phải trong flex)</div>
<div class="m-0">reset tất cả margin về 0</div>

<!-- Padding: p{side}-{size} -->
<div class="p-3">padding: 1rem tất cả các phía</div>
<div class="px-4 py-2">padding-x: 1.5rem, padding-y: 0.5rem</div>
<div class="ps-3 pe-0">padding-start (left): 1rem, padding-end (right): 0</div>

<!-- Side shortcuts: t=top, b=bottom, s=start(left), e=end(right), x=horizontal, y=vertical -->
```

### 2.2 Display

```html
<div class="d-none">ẩn hoàn toàn (display: none)</div>
<div class="d-block">display: block</div>
<div class="d-inline">display: inline</div>
<div class="d-inline-block">display: inline-block</div>
<div class="d-flex">display: flex</div>
<div class="d-grid">display: grid</div>

<!-- Responsive: hiện ở md trở lên, ẩn ở mobile -->
<div class="d-none d-md-block">Chỉ hiện từ md (≥768px)</div>
<div class="d-md-none">Chỉ hiện ở mobile (< md)</div>
```

### 2.3 Flex Utilities

```html
<div class="d-flex justify-content-between align-items-center">
  <span>Left</span>
  <span>Right</span>
</div>

<!-- justify-content: start | end | center | between | around | evenly -->
<!-- align-items: start | end | center | baseline | stretch -->

<div class="d-flex flex-wrap gap-3">
  <!-- flex-wrap: cho phép xuống dòng -->
  <!-- gap-3: khoảng cách giữa các item = 1rem -->
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</div>

<!-- flex-grow / flex-shrink -->
<div class="d-flex">
  <div class="flex-grow-1">Chiếm hết không gian còn lại</div>
  <div>Fixed width</div>
</div>
```

### 2.4 Text Utilities

```html
<!-- Alignment -->
<p class="text-center">Căn giữa</p>
<p class="text-end">Căn phải</p>
<p class="text-start">Căn trái (default)</p>
<p class="text-md-center">Căn giữa từ md trở lên</p>

<!-- Font weight & style -->
<span class="fw-bold">Bold (700)</span>
<span class="fw-semibold">Semibold (600)</span>
<span class="fw-normal">Normal (400)</span>
<span class="fst-italic">Italic</span>

<!-- Font size: fs-1 (2.5rem) → fs-6 (1rem) -->
<h1 class="fs-1">Heading size 1</h1>
<p class="fs-5">Paragraph size 5 (1.25rem)</p>

<!-- Text transform & decoration -->
<span class="text-uppercase">UPPERCASE</span>
<span class="text-lowercase">lowercase</span>
<span class="text-decoration-none">Bỏ underline (link)</span>
<span class="text-decoration-line-through">Gạch ngang</span>

<!-- Truncate với ellipsis -->
<p class="text-truncate" style="width: 200px;">
  Đoạn văn bản dài sẽ bị cắt với dấu ba chấm...
</p>
```

### 2.5 Color Utilities

```html
<!-- Text color -->
<p class="text-primary">text-primary (xanh)</p>
<p class="text-danger">text-danger (đỏ)</p>
<p class="text-success">text-success (xanh lá)</p>
<p class="text-warning">text-warning (vàng)</p>
<p class="text-muted">text-muted (xám nhạt)</p>
<p class="text-white bg-dark">text-white</p>

<!-- Background color -->
<div class="bg-primary text-white p-2">bg-primary</div>
<div class="bg-light text-dark p-2">bg-light</div>
<div class="bg-warning p-2">bg-warning</div>

<!-- Text opacity (Bootstrap 5.1+) -->
<p class="text-primary text-opacity-75">75% opacity</p>
<p class="text-primary text-opacity-50">50% opacity</p>
```

### 2.6 Shadow & Border

```html
<!-- Shadow -->
<div class="shadow-none p-2 mb-2">Không shadow</div>
<div class="shadow-sm p-2 mb-2">Shadow nhỏ</div>
<div class="shadow p-2 mb-2">Shadow mặc định</div>
<div class="shadow-lg p-2 mb-2">Shadow lớn</div>

<!-- Border -->
<div class="border p-2 mb-2">border: 1px solid</div>
<div class="border border-primary p-2 mb-2">border màu primary</div>
<div class="border-0 p-2 mb-2">Xóa border</div>

<!-- Border radius -->
<img class="rounded" src="..." alt="">           <!-- bo nhẹ -->
<img class="rounded-circle" src="..." alt="">    <!-- bo tròn (avatar) -->
<img class="rounded-pill" src="..." alt="">      <!-- bo pill (badge) -->
<img class="rounded-0" src="..." alt="">         <!-- vuông hoàn toàn -->
<img class="rounded-3" src="..." alt="">         <!-- bo cấp độ 3 (0→5) -->
```

### 2.7 Position Utilities

```html
<!-- Position type -->
<div class="position-relative">
  <div class="position-absolute top-0 end-0">
    <!-- Góc trên phải của parent relative -->
  </div>
</div>

<!-- top/bottom/start/end: 0 | 50 | 100 -->
<div class="position-absolute top-50 start-50 translate-middle">
  Căn giữa tuyệt đối (cả ngang lẫn dọc)
</div>

<!-- Fixed/sticky -->
<nav class="position-sticky top-0">Sticky nav bar</nav>
<footer class="position-fixed bottom-0 w-100">Fixed footer</footer>
```

### 2.8 Sizing & Overflow

```html
<!-- Width & Height -->
<div class="w-25">25%</div>
<div class="w-50">50%</div>
<div class="w-75">75%</div>
<div class="w-100">100%</div>
<div class="mw-100">max-width: 100%</div>

<div class="h-100">height: 100% (parent cần có chiều cao xác định)</div>
<div class="min-vh-100">min-height: 100vh</div>

<!-- Overflow -->
<div class="overflow-auto" style="height: 100px;">Scroll nếu tràn</div>
<div class="overflow-hidden">Ẩn phần tràn</div>
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: Profile Card dùng Utilities thuần (không CSS tùy chỉnh)

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>Profile Card</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
</head>
<body class="min-vh-100 d-flex align-items-center justify-content-center bg-light">

  <div class="card shadow p-4 text-center" style="width: 20rem;">
    <img src="https://picsum.photos/80/80?face"
         class="rounded-circle mx-auto d-block mb-3 border border-3 border-primary"
         width="80" height="80" alt="Avatar">

    <h5 class="fw-bold mb-1">Phan Minh Tài</h5>
    <p class="text-muted fs-6 mb-3">Frontend Developer Fresher</p>

    <div class="d-flex justify-content-center gap-2 mb-3">
      <span class="badge bg-primary rounded-pill">HTML</span>
      <span class="badge bg-success rounded-pill">CSS</span>
      <span class="badge bg-warning text-dark rounded-pill">Bootstrap</span>
    </div>

    <div class="d-grid gap-2">
      <a href="#" class="btn btn-primary btn-sm">Xem portfolio</a>
      <a href="#" class="btn btn-outline-secondary btn-sm">LinkedIn</a>
    </div>
  </div>

</body>
</html>
```

### Ví dụ 2: Responsive Navbar với Spacing và Display Utilities

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>Utility Navbar</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
</head>
<body>

  <!-- Notification bar — ẩn ở mobile, hiện từ md -->
  <div class="d-none d-md-flex justify-content-end align-items-center
              bg-dark text-white py-1 px-4 fs-6">
    <span class="me-3">phanminhtai1029@gmail.com</span>
    <a href="#" class="text-white text-decoration-none">Đăng xuất</a>
  </div>

  <!-- Navbar chính -->
  <nav class="navbar navbar-expand-md navbar-light bg-white shadow-sm">
    <div class="container">
      <a class="navbar-brand fw-bold text-primary fs-4" href="#">FresherAI</a>

      <button class="navbar-toggler border-0" type="button"
              data-bs-toggle="collapse" data-bs-target="#mainNav">
        <span class="navbar-toggler-icon"></span>
      </button>

      <div class="collapse navbar-collapse" id="mainNav">
        <ul class="navbar-nav mx-auto gap-2">
          <li class="nav-item"><a class="nav-link fw-semibold" href="#">HTML/CSS</a></li>
          <li class="nav-item"><a class="nav-link fw-semibold" href="#">JavaScript</a></li>
          <li class="nav-item"><a class="nav-link fw-semibold" href="#">React</a></li>
        </ul>

        <!-- Search: ẩn ở mobile, hiện từ md -->
        <div class="d-none d-md-flex gap-2">
          <input class="form-control form-control-sm" type="search" placeholder="Tìm kiếm...">
          <button class="btn btn-primary btn-sm px-3">Tìm</button>
        </div>
      </div>
    </div>
  </nav>

  <!-- Hero section với min-vh -->
  <section class="min-vh-100 d-flex align-items-center justify-content-center bg-light">
    <div class="text-center px-4">
      <h1 class="fw-bold fs-1 mb-3">Ôn tập Frontend</h1>
      <p class="text-muted fs-5 mb-4 mx-auto" style="max-width: 500px;">
        Từ HTML cơ bản đến React — tất cả trong một vault Obsidian.
      </p>
      <div class="d-flex gap-3 justify-content-center flex-wrap">
        <a href="#" class="btn btn-primary px-4 py-2">Bắt đầu học</a>
        <a href="#" class="btn btn-outline-primary px-4 py-2">Xem lộ trình</a>
      </div>
    </div>
  </section>

  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: `mx-auto` không căn giữa được
> `mx-auto` chỉ hoạt động trên **block element có width xác định** (không phải 100%). Ví dụ: `<div class="w-50 mx-auto">` — nếu không có `w-*` hoặc inline style `max-width`, phần tử đã chiếm 100% rồi thì auto margin không có hiệu lực.

> [!warning] Pitfall 2: `h-100` không có tác dụng
> `h-100` (height: 100%) yêu cầu **parent phải có chiều cao xác định**. Dùng `min-vh-100` trên parent thay vì `h-100` khi muốn full viewport height.

> [!warning] Pitfall 3: Responsive display class viết sai chiều
> Để **ẩn ở mobile, hiện từ md**: viết `d-none d-md-block` (ẩn default, rồi hiện từ md).
> Để **hiện ở mobile, ẩn từ md**: viết `d-block d-md-none`.
> Thứ tự: **mobile-first** — class không có breakpoint là mặc định (smallest screen).

> [!tip] Dùng `gap-*` thay vì margin trên flex items
> Trong flex container, dùng `gap-2` / `gap-3` trên container thay vì `me-2` trên từng item — code gọn hơn, không bị margin thừa ở item cuối.

---

## 5. Phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Utility class là gì, ẩn/hiện theo màn hình thế nào?"
> *"Utility class là các class nguyên tử, mỗi class làm đúng một việc như margin, padding, display, canh chữ hay màu nền, và mình ghép nhiều class lại để style ngay trên HTML mà không cần viết CSS riêng, đúng tư tưởng atomic CSS giống Tailwind. Hầu hết có biến thể responsive theo breakpoint. Ví dụ d-none d-md-block nghĩa là ẩn ở mobile rồi hiện từ md trở lên, dùng cho phần tử chỉ cho desktop như sidebar; ngược lại d-md-none nghĩa là hiện ở mobile nhưng ẩn từ md trở lên, dùng cho nút menu hamburger chỉ xuất hiện trên điện thoại. Ưu điểm là dựng nhanh và nhất quán, đổi lại HTML có thể dài ra vì nhiều class."*

> [!note] 🧠 Mẹo nhớ
> **Utility = class tí hon 1 việc (`m-3`, `d-flex`, `text-center`), ghép lại style nhanh.** Responsive: **`d-none d-md-block` = ẩn mobile/hiện desktop; `d-md-none` = hiện mobile/ẩn desktop.**

**Q1: Sự khác nhau giữa `d-none d-md-block` và `d-md-none`?**

> - `d-none d-md-block`: **Ẩn ở mobile** (d-none), **hiện từ md trở lên** (d-md-block) — dùng cho desktop-only element (sidebar, search bar).
> - `d-md-none`: **Hiện ở mobile** (không override), **ẩn từ md trở lên** — dùng cho mobile-only element (hamburger menu label, compact nav).

**Q2: `ms-auto` trong flex làm gì?**

> `margin-inline-start: auto` = margin-left auto trong LTR. Trong flex container, nó **đẩy phần tử về phía cuối (end)**. Dùng phổ biến để đặt button "Đăng xuất" sang phải navbar: `<button class="btn ms-auto">`.

**Q3: Tại sao nên dùng `gap-*` thay vì `me-*` trên flex items?**

> `gap` là CSS property trên **container** — áp dụng đều cho tất cả gaps giữa các items, kể cả khi xuống dòng (`flex-wrap`). `me-*` là margin trên **item** — bị thừa ở item cuối và không hoạt động đúng khi wrap.

**Q4: Làm thế nào để override Bootstrap utility?**

```css
/* Cách 1: tăng specificity */
.my-component.text-muted { color: #555 !important; }

/* Cách 2: dùng CSS custom property (Bootstrap 5) */
:root {
  --bs-secondary-color: #555;  /* text-muted dùng biến này */
}

/* Cách 3: !important (nhanh, nhưng khó debug) */
.special-text { color: #e74c3c !important; }
```

---

## 6. Bài tập thực hành

**Bài 1:** Dùng **chỉ Utilities, không viết CSS tùy chỉnh nào**, tạo layout sau:
- Header: `bg-dark text-white` với logo bên trái, nav links bên phải
- Main: 2 cột (8/4) với `gap-4` — cột trái có text, cột phải có form liên hệ
- Footer: `bg-light text-center py-4 mt-5` với copyright text

**Bài 2:** Tạo grid 4 cột card sản phẩm. Yêu cầu:
- Mobile: 1 cột, tablet (sm): 2 cột, desktop (lg): 4 cột
- Mỗi card: ảnh + tiêu đề + badge trạng thái + giá (dùng `text-danger fw-bold`)
- Card đồng đều chiều cao dùng `h-100` + `d-flex flex-column`

**Bài 3 (nâng cao):** Tái tạo giao diện notification bar: dùng Utilities tạo thanh thông báo sticky top với `position-sticky top-0`, hiện `badge bg-danger` với số đếm, ẩn ở màn hình nhỏ hơn `sm`.

---

## 7. Liên kết

- [[13-Bootstrap-Overview]] — Tổng quan Bootstrap 5, CDN setup
- [[14-Bootstrap-Layout-vs-Grid]] — Grid utilities (`col-*`, `g-*`, `offset-*`)
- [[15-Bootstrap-Components]] — Components (Button, Card, Modal, Accordion)
- [[08-CSS-Layout-Flexbox]] — Flexbox gốc — hiểu để dùng flex utilities đúng
- [[10-CSS-Responsive-Media-Query]] — Media query gốc — nền tảng responsive utilities
