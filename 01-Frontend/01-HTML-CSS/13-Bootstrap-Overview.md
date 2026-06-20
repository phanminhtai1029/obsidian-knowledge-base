---
title: "Bootstrap: Tổng quan"
section: 01-HTML-CSS
tags: [bootstrap, css-framework, responsive, fresher, frontend]
related:
  - "[[14-Bootstrap-Layout-vs-Grid]]"
  - "[[10-CSS-Responsive-Media-Query]]"
difficulty: ⭐⭐
estimated_time: 30m
source: [Bootstrap 5 docs]
---

# Bootstrap: Tổng quan

> [!summary] TL;DR
> Bootstrap 5 là CSS framework phổ biến nhất, cung cấp hệ thống Grid 12 cột, 30+ UI components (Button, Modal, Navbar...), và hàng trăm utility classes. Dùng Bootstrap để prototype nhanh và đảm bảo responsive cross-browser mà không cần viết CSS từ đầu.

## 1. Khái niệm

**Bootstrap** là CSS framework mã nguồn mở — tập hợp CSS/JS pre-built cho typography, layout, components, và utilities.

**Phiên bản:** Bootstrap 5 (2021+) — loại bỏ jQuery dependency, dùng Vanilla JS, CSS custom properties, nâng cấp utilities.

**Khi nào dùng Bootstrap:**
- Prototype nhanh, admin panel, internal tool.
- Team không có designer — Bootstrap cho giao diện decent mặc định.
- Cross-browser compatibility là ưu tiên.

**Khi nào KHÔNG dùng:**
- Brand identity độc đáo cần custom design system.
- Bundle size là vấn đề (full Bootstrap ~22KB CSS min+gzip).
- Đã có design system riêng (Material, Ant Design...).

## 2. Cú pháp / API

```html
<!-- CDN — Không cần cài đặt, nhanh nhất để test -->
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Bootstrap Demo</title>
  <!-- CSS -->
  <link
    href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
    rel="stylesheet"
  >
</head>
<body>
  <!-- Content với Bootstrap classes -->

  <!-- JS bundle (Popper.js included — cần cho dropdown, tooltip, modal) -->
  <script
    src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js">
  </script>
</body>
</html>
```

**Cài đặt qua npm:**
```bash
npm install bootstrap@5.3.3
```

```js
// main.js (Vite/Webpack)
import 'bootstrap/dist/css/bootstrap.min.css'
import 'bootstrap/dist/js/bootstrap.bundle.min.js'
```

## 3. Ví dụ minh họa

### Ví dụ 1: Trang đầy đủ Bootstrap 5 trong 20 dòng HTML

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
  <title>Demo Bootstrap</title>
</head>
<body>
  <!-- Navbar -->
  <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
    <div class="container">
      <a class="navbar-brand fw-bold" href="#">FresherAI</a>
    </div>
  </nav>

  <!-- Hero -->
  <div class="py-5 bg-light text-center">
    <div class="container">
      <h1 class="display-4 fw-bold">Học Frontend nhanh hơn</h1>
      <p class="lead text-muted">Vault Obsidian với 70+ note có code minh họa</p>
      <a href="#" class="btn btn-primary btn-lg me-2">Bắt đầu</a>
      <a href="#" class="btn btn-outline-secondary btn-lg">Xem demo</a>
    </div>
  </div>

  <!-- Cards -->
  <div class="container py-5">
    <div class="row g-4">
      <div class="col-md-4">
        <div class="card h-100 shadow-sm">
          <div class="card-body">
            <h5 class="card-title">HTML & CSS</h5>
            <p class="card-text text-muted">Semantic HTML, Flexbox, Grid, Responsive.</p>
            <a href="#" class="btn btn-sm btn-outline-primary">Học ngay</a>
          </div>
        </div>
      </div>
      <div class="col-md-4">
        <div class="card h-100 shadow-sm">
          <div class="card-body">
            <h5 class="card-title">JavaScript</h5>
            <p class="card-text text-muted">Scope, Closure, ES6+, Async/Await.</p>
            <a href="#" class="btn btn-sm btn-outline-primary">Học ngay</a>
          </div>
        </div>
      </div>
      <div class="col-md-4">
        <div class="card h-100 shadow-sm">
          <div class="card-body">
            <h5 class="card-title">React</h5>
            <p class="card-text text-muted">useState, useEffect, Context, Router.</p>
            <a href="#" class="btn btn-sm btn-outline-primary">Học ngay</a>
          </div>
        </div>
      </div>
    </div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

**Output / Kết quả mong đợi:** Trang responsive hoàn chỉnh với navbar, hero, 3 card — mobile: stack 1 cột, desktop: 3 cột.

### Ví dụ 2: Typography utilities

```html
<!-- Heading styles -->
<p class="h1">Heading 1 style (không phải h1 tag)</p>
<p class="display-1">Display 1 — cực lớn</p>
<p class="lead">Lead text — to hơn paragraph bình thường</p>

<!-- Text utilities -->
<p class="text-primary">Màu primary (blue)</p>
<p class="text-success">Màu success (green)</p>
<p class="text-danger">Màu danger (red)</p>
<p class="text-muted">Text mờ (gray)</p>
<p class="fw-bold">In đậm</p>
<p class="fst-italic">In nghiêng</p>
<p class="text-center">Căn giữa</p>
<p class="text-uppercase">Chữ hoa</p>
<p class="text-truncate" style="width:200px">Text dài bị cắt với ellipsis...</p>
```

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Lỗi phổ biến
> - **Override Bootstrap với CSS thường:** Bootstrap classes có specificity thấp — dùng class riêng hoặc `!important` khi cần override. Hoặc import custom CSS SAU Bootstrap trong stylesheet.
> - **Quên `container`:** Grid và nhiều components cần wrapper `.container` để có max-width và centering.
> - **Load Bootstrap JS không có Popper:** Dropdown, tooltip, popover cần Popper.js. Dùng `bootstrap.bundle.min.js` (đã bundle Popper) thay vì `bootstrap.min.js`.
> - **Dùng Bootstrap 4 class trong BS5:** `ml-*`/`mr-*` → `ms-*`/`me-*` (logical properties), `float-left` → `float-start`, `text-left` → `text-start`.

## 5. Câu hỏi phỏng vấn thường gặp

1. **Q:** Bootstrap 5 khác Bootstrap 4 điều gì quan trọng nhất?
   **A:** BS5 loại bỏ **jQuery dependency** (dùng Vanilla JS). Thêm CSS custom properties (`--bs-*`). Đổi từ left/right sang start/end (logical properties cho RTL support). Cải thiện Grid với `xxl` breakpoint và row cols. Thêm nhiều utility class hơn.

2. **Q:** Khi nào nên dùng Bootstrap, khi nào tự viết CSS?
   **A:** Bootstrap: prototype nhanh, admin panel, team không có designer. Tự viết: design system riêng của công ty, performance critical (không muốn load unused CSS), cần kiểm soát hoàn toàn HTML class structure.

3. **Q:** Làm sao giảm bundle size khi dùng Bootstrap?
   **A:** Dùng **Sass với custom import** (chỉ import component cần thiết). Hoặc dùng **PurgeCSS** để loại bỏ class không dùng. Bootstrap 5 hỗ trợ tree-shaking khi import qua npm module system.

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Tạo trang landing page chỉ dùng Bootstrap CDN (không viết CSS tùy chỉnh): navbar responsive, hero section, 3 feature cards, pricing table (3 cột), footer. Responsive hoàn toàn.
- [ ] **Bài 2:** Override màu primary của Bootstrap bằng CSS custom property: `:root { --bs-primary: #e63946; --bs-primary-rgb: 230, 57, 70; }`. Kiểm tra tất cả `.btn-primary`, `.bg-primary`, `.text-primary` đã đổi màu chưa.

## 7. Liên kết

- Note liên quan: [[14-Bootstrap-Layout-vs-Grid]], [[15-Bootstrap-Components]], [[16-Bootstrap-Utilities]]
- [Bootstrap 5 Official Docs](https://getbootstrap.com/docs/5.3/)
- [Bootstrap CDN](https://www.bootstrapcdn.com/)
