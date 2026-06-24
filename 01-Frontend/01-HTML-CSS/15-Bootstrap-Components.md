---
title: "Bootstrap Components"
section: 01-HTML-CSS
tags: [bootstrap, components, ui, fresher, frontend]
related:
  - "[[13-Bootstrap-Overview]]"
  - "[[14-Bootstrap-Layout-vs-Grid]]"
  - "[[16-Bootstrap-Utilities]]"
difficulty: ⭐⭐
estimated_time: 40m
source: [Bootstrap 5 docs]
---

# Bootstrap 5 — Components (UI Components)

> [!summary] TL;DR
> Bootstrap Components là các UI widget sẵn dùng (Button, Card, Navbar, Modal, Accordion, Tabs). Phần lớn chỉ cần HTML + class; một số cần `bootstrap.bundle.min.js` để hoạt động (Modal, Accordion, Tabs collapse).

> [!tip] 🎯 Hiểu trong 30 giây
> **Component = các "khối UI lắp sẵn"** của Bootstrap: Button, Card, Navbar, Modal (hộp thoại), Accordion (xếp gập), Tabs... Bạn chỉ copy cấu trúc HTML + gắn đúng class là có ngay.
> - Phần lớn chỉ cần **HTML + class** (Button, Card). Nhưng các thành phần *có tương tác/động* (Modal, Accordion, Tabs, Dropdown) cần nạp **`bootstrap.bundle.min.js`** thì mới chạy.
> - Điều khiển bằng thuộc tính `data-bs-*` (cách *khai báo*, vd `data-bs-toggle="modal"`) hoặc gọi **JS API** khi cần *chủ động* (vd đóng modal sau khi gọi API thành công).

---

## 1. Khái niệm

Bootstrap Components là tập hợp các **UI widget sẵn dùng** — mỗi component có cấu trúc HTML riêng, CSS riêng, và đôi khi cần JavaScript (qua `data-bs-*` attributes hoặc BS JS API).

| Component | Cần JS? | Dùng khi |
|-----------|---------|----------|
| Button | ❌ | Hành động, form submit |
| Badge | ❌ | Số thông báo, nhãn trạng thái |
| Alert | ✅ (dismiss) | Thông báo tạm thời |
| Card | ❌ | Hiển thị content block |
| Navbar | ✅ (collapse) | Thanh điều hướng responsive |
| Modal | ✅ | Dialog/popup overlay |
| Accordion | ✅ | Toggle nội dung xếp chồng |
| Tabs | ✅ | Chuyển đổi panel nội dung |

> [!tip] Data attributes vs JS API
> Bootstrap 5 ưu tiên `data-bs-toggle`, `data-bs-target` — không cần viết JS. Dùng JS API khi cần kiểm soát lập trình: `const modal = new bootstrap.Modal(el)`.

```
★ Insight ─────────────────────────────────────
• Chia component Bootstrap làm 2 nhóm: "CSS thuần" (Button, Card, Badge — chỉ
  cần class) và "cần JS" (Modal, Accordion, Tabs, Navbar collapse — cần bundle
  JS + data-bs-*). Quên include `bootstrap.bundle.min.js` thì nhóm 2 "im lặng"
  không báo lỗi — đây là bug số 1 của người mới.
• Bootstrap JS thao tác DOM trực tiếp → XUNG ĐỘT với React/Vue (vốn tự quản DOM
  qua Virtual DOM). Trong dự án React, KHÔNG dùng bootstrap.js mà dùng wrapper
  như react-bootstrap để component sống trong vòng đời React. Đây là lý do quan
  trọng khi phỏng vấn hỏi "dùng Bootstrap với React thế nào?".
─────────────────────────────────────────────────
```

---

## 2. Cú pháp & Class quan trọng

### 2.1 Buttons

```html
<!-- Variants màu -->
<button class="btn btn-primary">Primary</button>
<button class="btn btn-outline-danger">Outline Danger</button>
<button class="btn btn-secondary btn-sm">Small</button>
<button class="btn btn-success btn-lg">Large</button>

<!-- Disabled -->
<button class="btn btn-primary" disabled>Disabled</button>

<!-- Loading state với spinner -->
<button class="btn btn-primary" disabled>
  <span class="spinner-border spinner-border-sm me-1"></span>
  Loading...
</button>
```

### 2.2 Cards

```html
<div class="card" style="width: 18rem;">
  <img src="thumbnail.jpg" class="card-img-top" alt="Thumbnail">
  <div class="card-body">
    <h5 class="card-title">Tiêu đề card</h5>
    <p class="card-text">Mô tả ngắn gọn về card này.</p>
    <a href="#" class="btn btn-primary">Xem thêm</a>
  </div>
  <div class="card-footer text-muted">2 ngày trước</div>
</div>
```

### 2.3 Navbar

```html
<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
  <div class="container">
    <a class="navbar-brand" href="#">MyApp</a>

    <!-- Hamburger toggle (hiện ở mobile) -->
    <button class="navbar-toggler" type="button"
            data-bs-toggle="collapse" data-bs-target="#navMenu">
      <span class="navbar-toggler-icon"></span>
    </button>

    <!-- Menu collapsible -->
    <div class="collapse navbar-collapse" id="navMenu">
      <ul class="navbar-nav ms-auto">
        <li class="nav-item"><a class="nav-link active" href="#">Home</a></li>
        <li class="nav-item"><a class="nav-link" href="#">About</a></li>
        <li class="nav-item"><a class="nav-link" href="#">Contact</a></li>
      </ul>
    </div>
  </div>
</nav>
```

### 2.4 Modal

```html
<!-- Trigger button -->
<button class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#confirmModal">
  Xác nhận xóa
</button>

<!-- Modal HTML — đặt trực tiếp trong body, không lồng sâu -->
<div class="modal fade" id="confirmModal" tabindex="-1">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title">Xác nhận</h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
      </div>
      <div class="modal-body">
        Bạn có chắc muốn xóa mục này không?
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Hủy</button>
        <button type="button" class="btn btn-danger" id="confirmDelete">Xóa</button>
      </div>
    </div>
  </div>
</div>
```

### 2.5 Alert & Badge

```html
<!-- Alert có nút đóng -->
<div class="alert alert-success alert-dismissible fade show" role="alert">
  <strong>Thành công!</strong> Dữ liệu đã được lưu.
  <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
</div>

<!-- Badge trên button -->
<button class="btn btn-primary">
  Thông báo <span class="badge bg-danger rounded-pill">4</span>
</button>
```

### 2.6 Accordion

```html
<div class="accordion" id="faqAccordion">
  <div class="accordion-item">
    <h2 class="accordion-header">
      <button class="accordion-button" type="button"
              data-bs-toggle="collapse" data-bs-target="#faq1">
        Bootstrap là gì?
      </button>
    </h2>
    <div id="faq1" class="accordion-collapse collapse show"
         data-bs-parent="#faqAccordion">
      <div class="accordion-body">
        Bootstrap là CSS framework phổ biến nhất thế giới...
      </div>
    </div>
  </div>

  <div class="accordion-item">
    <h2 class="accordion-header">
      <button class="accordion-button collapsed" type="button"
              data-bs-toggle="collapse" data-bs-target="#faq2">
        Bootstrap 5 khác Bootstrap 4 thế nào?
      </button>
    </h2>
    <div id="faq2" class="accordion-collapse collapse"
         data-bs-parent="#faqAccordion">
      <div class="accordion-body">
        BS5 bỏ jQuery, dùng data-bs-* thay data-*, thêm nhiều utilities mới...
      </div>
    </div>
  </div>
</div>
```

### 2.7 Tabs

```html
<!-- Tab navigation -->
<ul class="nav nav-tabs" id="myTab">
  <li class="nav-item">
    <button class="nav-link active" data-bs-toggle="tab" data-bs-target="#tabHome">Home</button>
  </li>
  <li class="nav-item">
    <button class="nav-link" data-bs-toggle="tab" data-bs-target="#tabProfile">Profile</button>
  </li>
</ul>

<!-- Tab content -->
<div class="tab-content mt-3">
  <div class="tab-pane fade show active" id="tabHome">
    Nội dung tab Home...
  </div>
  <div class="tab-pane fade" id="tabProfile">
    Nội dung tab Profile...
  </div>
</div>
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: Product Card với Badge "Sale" và đồng đều chiều cao

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>Product Cards</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
  <style>
    .card-img-wrapper { position: relative; }
    .sale-badge { position: absolute; top: 10px; left: 10px; }
  </style>
</head>
<body class="p-4 bg-light">
  <div class="row row-cols-1 row-cols-md-3 g-4" style="max-width: 900px; margin: auto;">
    <div class="col">
      <div class="card h-100 shadow-sm">
        <div class="card-img-wrapper">
          <img src="https://picsum.photos/300/200?1" class="card-img-top" alt="Sản phẩm">
          <span class="badge bg-danger sale-badge fs-6">-20%</span>
        </div>
        <div class="card-body d-flex flex-column">
          <h5 class="card-title">Áo thun nam</h5>
          <p class="card-text text-muted flex-grow-1">Chất liệu cotton cao cấp, thoáng mát.</p>
          <div class="d-flex justify-content-between align-items-center mb-2">
            <span class="fw-bold text-danger fs-5">200.000đ</span>
            <span class="text-decoration-line-through text-muted">250.000đ</span>
          </div>
        </div>
        <div class="card-footer d-grid">
          <button class="btn btn-primary">Thêm vào giỏ</button>
        </div>
      </div>
    </div>
  </div>
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

### Ví dụ 2: Navbar + Modal đăng xuất + Alert động bằng DOM API

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>Navbar + Modal</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
</head>
<body>
  <nav class="navbar navbar-expand-md navbar-dark bg-primary">
    <div class="container">
      <a class="navbar-brand fw-bold" href="#">Dashboard</a>
      <button class="navbar-toggler" type="button"
              data-bs-toggle="collapse" data-bs-target="#nav">
        <span class="navbar-toggler-icon"></span>
      </button>
      <div class="collapse navbar-collapse" id="nav">
        <ul class="navbar-nav me-auto">
          <li class="nav-item"><a class="nav-link active" href="#">Trang chủ</a></li>
          <li class="nav-item"><a class="nav-link" href="#">Báo cáo</a></li>
        </ul>
        <button class="btn btn-outline-light"
                data-bs-toggle="modal" data-bs-target="#logoutModal">
          Đăng xuất
        </button>
      </div>
    </div>
  </nav>

  <div class="container mt-4" id="alertArea"></div>

  <!-- Modal đăng xuất -->
  <div class="modal fade" id="logoutModal" tabindex="-1">
    <div class="modal-dialog modal-sm">
      <div class="modal-content">
        <div class="modal-header border-0">
          <h5 class="modal-title">Đăng xuất?</h5>
          <button class="btn-close" data-bs-dismiss="modal"></button>
        </div>
        <div class="modal-body py-0 text-muted">Bạn sẽ cần đăng nhập lại.</div>
        <div class="modal-footer border-0">
          <button class="btn btn-secondary btn-sm" data-bs-dismiss="modal">Hủy</button>
          <button class="btn btn-danger btn-sm" id="btnLogout">Xác nhận</button>
        </div>
      </div>
    </div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
  <script>
    document.getElementById('btnLogout').addEventListener('click', () => {
      // Đóng modal
      bootstrap.Modal.getInstance(document.getElementById('logoutModal')).hide();

      // Tạo Alert bằng DOM API (tránh innerHTML với biến động)
      const alertEl = document.createElement('div');
      alertEl.className = 'alert alert-info alert-dismissible fade show';
      alertEl.setAttribute('role', 'alert');

      const msg = document.createElement('span');
      msg.textContent = 'Đã đăng xuất thành công!';

      const closeBtn = document.createElement('button');
      closeBtn.type = 'button';
      closeBtn.className = 'btn-close';
      closeBtn.setAttribute('data-bs-dismiss', 'alert');

      alertEl.append(msg, closeBtn);
      document.getElementById('alertArea').appendChild(alertEl);

      // Tự đóng sau 3 giây
      setTimeout(() => {
        bootstrap.Alert.getOrCreateInstance(alertEl).close();
      }, 3000);
    });
  </script>
</body>
</html>
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: Quên `bootstrap.bundle.min.js`
> Modal, Accordion, Tabs **không hoạt động** nếu chỉ include CSS. File `bootstrap.bundle.min.js` đã gộp Popper.js — không cần thêm riêng.

> [!warning] Pitfall 2: Lồng Modal sâu trong DOM
> Đặt Modal HTML trong phần tử có `overflow: hidden` hoặc `transform` sẽ làm sai z-index và backdrop. **Luôn đặt Modal HTML trực tiếp trong `<body>`**.

> [!warning] Pitfall 3: Accordion `data-bs-parent` sai ID
> `data-bs-parent="#faqAccordion"` phải khớp **chính xác** với `id` của wrapper. Nếu sai, nhiều item sẽ mở đồng thời thay vì tự đóng item cũ.

> [!tip] Card height đồng đều trong grid
> Dùng `h-100` trên `.card` + `d-flex flex-column` trên `.card-body` + `flex-grow-1` trên `.card-text` → button luôn ở cuối card dù content dài ngắn khác nhau.

---

## 5. Phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Component nào cần JS, điều khiển bằng data-bs vs JS API?"
> *"Phần lớn component của Bootstrap chỉ cần HTML và class như Button hay Card. Nhưng các component có tương tác động như Modal, Accordion, Tabs, Dropdown thì cần nạp file bootstrap bundle js mới hoạt động. Có hai cách điều khiển: cách khai báo bằng thuộc tính data-bs, ví dụ data-bs-toggle bằng modal và data-bs-dismiss bằng modal, Bootstrap tự xử lý khi người dùng click, rất nhanh gọn; và cách dùng JS API khi cần chủ động trong code, ví dụ lấy instance modal rồi gọi hide sau khi một request API thành công trong callback. Em dùng data-bs cho thao tác đơn giản, JS API khi cần điều khiển theo logic."*

> [!note] 🧠 Mẹo nhớ
> **Component lắp sẵn; loại tương tác (Modal/Accordion/Tabs) cần `bootstrap.bundle.js`.** Điều khiển: **`data-bs-*` (khai báo) vs JS API (chủ động).**

**Q1: Sự khác nhau giữa `data-bs-dismiss="modal"` và đóng Modal bằng JavaScript?**

> `data-bs-dismiss="modal"` là **declarative** — Bootstrap tự xử lý khi click. JS API (`Modal.getInstance(el).hide()`) dùng khi cần đóng **programmatically** — ví dụ sau khi gọi API thành công trong `.then()` callback.

**Q2: Tại sao Bootstrap 5 bỏ jQuery?**

> jQuery thêm ~87KB (minified) chỉ để wrap DOM API sẵn có. Bootstrap 5 dùng Vanilla JS — nhỏ hơn, nhanh hơn, không conflict với React/Vue vốn quản lý DOM theo cách riêng.

**Q3: `navbar-expand-lg` nghĩa là gì? Khi nào dùng `navbar-expand-md`?**

> Navbar **expand** (hiện full menu ngang) từ breakpoint đó trở lên. `navbar-expand-lg` = ≥992px expand, dưới đó hamburger. Dùng `navbar-expand-md` (≥768px) khi site ít menu items và cần expand sớm hơn.

**Q4: Làm thế nào mở Modal lập trình (programmatically)?**

```javascript
const modalEl = document.getElementById('confirmModal');
const modal = new bootstrap.Modal(modalEl, {
  backdrop: 'static', // click ngoài không đóng
  keyboard: false     // ESC không đóng
});
modal.show();

// Đóng sau khi xử lý xong:
modal.hide();
```

---

## 6. Bài tập thực hành

**Bài 1:** Xây dựng trang FAQ dùng Accordion gồm 5 câu hỏi. Yêu cầu: chỉ mở 1 câu tại một thời điểm (`data-bs-parent`), câu đầu tiên mặc định mở.

**Bài 2:** Tạo trang portfolio nhỏ với:
- Navbar responsive (collapse ở mobile) gồm 4 nav link
- Section "Projects" gồm 3 Card dạng grid (ảnh, tiêu đề, mô tả, nút "Demo")
- Nút "Liên hệ" trên Navbar mở Modal có form (họ tên + email + nút gửi)

**Bài 3 (nâng cao):** Sau khi submit form trong Modal, đóng modal và tạo Alert `alert-success` bằng DOM API (createElement). Alert tự đóng sau 3 giây dùng `setTimeout` + `bootstrap.Alert.getOrCreateInstance(el).close()`.

---

## 7. Liên kết

- [[13-Bootstrap-Overview]] — Tổng quan Bootstrap 5, CDN setup, typography
- [[14-Bootstrap-Layout-vs-Grid]] — Grid system, responsive columns
- [[16-Bootstrap-Utilities]] — Spacing, display, flex, text utilities
- [[06-CSS-Properties-thong-dung]] — CSS `position` context (cần cho badge overlay trên card)
- [[08-CSS-Layout-Flexbox]] — Flexbox nền tảng của Bootstrap flex utilities
