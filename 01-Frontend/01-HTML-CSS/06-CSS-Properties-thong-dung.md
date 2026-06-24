---
title: "CSS Properties Thông Dụng"
section: 01-HTML-CSS
tags: [css, properties, font, color, text, background, fresher, frontend]
related:
  - "[[05-CSS-Box-Model]]"
  - "[[11-CSS-Units-Color-Border-Background]]"
difficulty: ⭐⭐
estimated_time: 40m
source: [CSS-fundamentals.docx]
---

# CSS Properties Thông Dụng

> [!summary] TL;DR
> Các CSS property hay dùng nhất: **Typography** (font-family, font-size, font-weight, line-height, color), **Background** (background-color, background-image), **Display** (block, inline, inline-block, none), **Position** (static, relative, absolute, fixed, sticky), **Visibility** (opacity, visibility). Hiểu rõ `display` và `position` là nền tảng cho mọi layout.

> [!tip] 🎯 Hiểu trong 30 giây
> Đây là "bộ thuộc tính CSS dùng hằng ngày". Hai cái *nền tảng nhất phải nắm chắc*:
> - **`display`** = phần tử "chiếm chỗ" kiểu gì: **`block`** (chiếm cả hàng, vd `<div>`), **`inline`** (nằm trong dòng, không đặt được width/height, vd `<span>`), **`inline-block`** (nằm trong dòng *nhưng* đặt được kích thước), **`none`** (ẩn hẳn, *không* chiếm chỗ).
> - **`position`** = cách định vị: **`static`** (mặc định, theo luồng), **`relative`** (dịch khỏi vị trí gốc *nhưng vẫn giữ chỗ*), **`absolute`** (thoát luồng, định vị theo cha *đã positioned*), **`fixed`** (dính theo màn hình, không cuộn theo), **`sticky`** (bình thường cho tới khi cuộn tới ngưỡng thì *dính* lại).
> - Phân biệt nhỏ hay hỏi: **`display:none`** (biến mất, không chiếm chỗ) vs **`visibility:hidden`** (ẩn nhưng *vẫn chiếm chỗ*) vs **`opacity:0`** (trong suốt, vẫn chiếm chỗ *và vẫn bấm được*).

## 1. Khái niệm

CSS có hàng trăm property. Nhóm **thông dụng nhất** mà fresher cần nắm vững:

| Nhóm | Properties chính |
|------|-----------------|
| Typography | `font-family`, `font-size`, `font-weight`, `font-style`, `line-height`, `letter-spacing`, `text-align`, `text-decoration`, `text-transform` |
| Color & Background | `color`, `background-color`, `background-image`, `background-size`, `opacity` |
| Display & Visibility | `display`, `visibility`, `overflow` |
| Sizing | `width`, `height`, `min-width`, `max-width`, `min-height`, `max-height` |
| Position | `position`, `top`, `right`, `bottom`, `left`, `z-index` |
| Cursor & Interaction | `cursor`, `pointer-events`, `user-select` |

## 2. Cú pháp / API

```css
/* ── TYPOGRAPHY ── */
p {
  font-family: 'Inter', -apple-system, sans-serif;  /* Font stack */
  font-size: 1rem;          /* 1rem = 16px (default) */
  font-weight: 400;         /* 100–900; 400=normal, 700=bold */
  font-style: italic;       /* normal | italic | oblique */
  line-height: 1.6;         /* Số không đơn vị = relative to font-size */
  letter-spacing: 0.05em;   /* Tracking */
  text-align: center;       /* left | center | right | justify */
  text-decoration: underline; /* none | underline | line-through */
  text-transform: uppercase;  /* none | uppercase | lowercase | capitalize */
}

/* ── COLOR & BACKGROUND ── */
.hero {
  color: #212529;                      /* Foreground text */
  background-color: rgba(0,0,0,0.05); /* Transparent bg */
  background-image: url('hero.jpg');
  background-size: cover;             /* contain | cover | auto | 100% 50% */
  background-position: center center;
  background-repeat: no-repeat;
  opacity: 0.9;                        /* 0 = ẩn, 1 = hiện */
}

/* ── DISPLAY ── */
.block       { display: block; }         /* Chiếm toàn bộ chiều rộng */
.inline      { display: inline; }        /* Theo luồng text, không nhận w/h */
.inline-block{ display: inline-block; }  /* Inline nhưng nhận w/h/margin */
.hidden      { display: none; }          /* Ẩn, không chiếm không gian */
.invisible   { visibility: hidden; }     /* Ẩn nhưng VẪN chiếm không gian */

/* ── POSITION ── */
.relative { position: relative; top: 10px; left: 20px; }  /* offset từ vị trí gốc */
.absolute { position: absolute; top: 0; right: 0; }       /* relative đến positioned ancestor */
.fixed    { position: fixed; bottom: 20px; right: 20px; } /* relative đến viewport */
.sticky   { position: sticky; top: 0; }                   /* scroll rồi dính */
```

## 3. Ví dụ minh họa

### Ví dụ 1: Card với typography, color, position hoàn chỉnh

```html
<div class="card">
  <span class="badge">Mới</span>
  <img src="product.jpg" alt="Sản phẩm">
  <div class="card-body">
    <h3 class="card-title">Tên sản phẩm</h3>
    <p class="card-price">299.000 ₫</p>
    <p class="card-desc">Mô tả ngắn về sản phẩm...</p>
  </div>
</div>
```

```css
.card {
  position: relative;       /* Làm anchor cho .badge */
  width: 280px;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 16px rgba(0,0,0,0.12);
  background-color: #fff;
  font-family: 'Inter', sans-serif;
}

.card img {
  width: 100%;
  height: 180px;
  object-fit: cover;        /* Cắt hình để fill box, không méo */
  display: block;
}

.badge {
  position: absolute;       /* Relative đến .card */
  top: 12px;
  left: 12px;
  background-color: #e53e3e;
  color: white;
  font-size: 0.75rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  padding: 4px 8px;
  border-radius: 4px;
}

.card-body { padding: 16px; }

.card-title {
  font-size: 1rem;
  font-weight: 600;
  color: #1a1a1a;
  margin: 0 0 4px;
}

.card-price {
  font-size: 1.25rem;
  font-weight: 700;
  color: #e53e3e;
  margin: 0 0 8px;
}

.card-desc {
  font-size: 0.875rem;
  line-height: 1.5;
  color: #666;
  margin: 0;
  /* Cắt text sau 2 dòng */
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
```

**Output / Kết quả mong đợi:** Card sản phẩm đẹp với badge overlay, ảnh fill box, text cắt sau 2 dòng.

### Ví dụ 2: Sticky header và position fixed button

```css
/* Sticky header — dính khi scroll xuống */
.site-header {
  position: sticky;
  top: 0;
  z-index: 100;             /* Hiển thị trên các element khác */
  background-color: white;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

/* Fixed "Back to top" button */
.btn-top {
  position: fixed;
  bottom: 24px;
  right: 24px;
  width: 44px;
  height: 44px;
  border-radius: 50%;
  background-color: #0066cc;
  color: white;
  border: none;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.btn-top:hover {
  background-color: #0052a3;
  transform: translateY(-2px);  /* Nhấc lên 2px khi hover */
}
```

**Output / Kết quả mong đợi:** Header dính khi scroll. Button "Lên đầu" luôn hiển thị ở góc phải bên dưới.

```
★ Insight ─────────────────────────────────────
• `position: absolute` luôn hỏi "neo vào AI?" → ancestor gần nhất có position
  KHÁC static. Nếu không tìm thấy → neo vào viewport (qua <html>). 90% bug
  "absolute nhảy lung tung" sửa bằng đúng 1 dòng: thêm `position: relative` lên
  cha mong muốn để biến nó thành điểm neo.
• `z-index` chỉ ăn khi element đã `position` (≠ static) HOẶC là flex/grid item.
  Quan trọng hơn: z-index so sánh TRONG cùng một "stacking context". Một cha có
  `opacity < 1`, `transform`, hay `filter` sẽ tạo stacking context mới — khiến
  con dù z-index 9999 vẫn không vượt được element ngoài context đó. Đây là lý do
  thật sự đằng sau "z-index to mà vẫn bị che".
─────────────────────────────────────────────────
```

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Lỗi phổ biến
> - **`position: absolute` không hoạt động đúng:** Element absolute tìm **positioned ancestor** gần nhất (có position khác `static`). Nếu không có, nó relative đến `<html>`. Fix: đặt `position: relative` trên parent.
> - **`display: none` vs `visibility: hidden`:** `none` xóa khỏi layout (element không chiếm không gian). `hidden` ẩn nhưng **vẫn chiếm không gian** — dùng khi muốn giữ layout ổn định.
> - **`opacity: 0` vs `display: none`:** `opacity: 0` ẩn visually nhưng element vẫn tương tác được (click được). Kết hợp với `pointer-events: none` để disable interaction.
> - **`z-index` không hoạt động:** `z-index` chỉ có hiệu lực khi `position` khác `static`.

## 5. Câu hỏi phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Phân biệt các giá trị `position`?"
> *"static là mặc định, phần tử nằm theo luồng bình thường, không nhận top left. relative là dịch chuyển khỏi vị trí gốc bằng top left nhưng vẫn giữ chỗ cũ, và quan trọng là nó tạo mốc cho con absolute. absolute thì thoát khỏi luồng, không giữ chỗ, và được định vị theo tổ tiên gần nhất có position khác static, nếu không có thì theo viewport. fixed cũng thoát luồng nhưng neo theo viewport nên không cuộn theo trang, hay dùng cho header dính hoặc nút nổi. sticky là lai giữa relative và fixed: phần tử nằm bình thường cho tới khi cuộn đến ngưỡng mình đặt thì nó dính lại như fixed, hay dùng cho tiêu đề cột dính khi cuộn."*

> [!note] 🧠 Mẹo nhớ
> **display: block (cả hàng) · inline (trong dòng, không kích thước) · inline-block (trong dòng + kích thước) · none (ẩn, mất chỗ).** position: **relative (giữ chỗ, làm mốc) · absolute (thoát luồng theo cha positioned) · fixed (neo viewport) · sticky (dính khi cuộn tới).** `display:none` mất chỗ ≠ `visibility:hidden`/`opacity:0` giữ chỗ.

1. **Q:** Sự khác nhau giữa `position: absolute`, `relative`, `fixed`, `sticky`?
   **A:** `relative` — offset từ vị trí gốc, vẫn chiếm không gian. `absolute` — xóa khỏi flow, relative đến positioned ancestor. `fixed` — relative đến viewport, không scroll cùng trang. `sticky` — relative cho đến khi scroll đến threshold, rồi dính như fixed.

2. **Q:** `display: none` và `visibility: hidden` khác nhau thế nào?
   **A:** `display: none` hoàn toàn xóa element khỏi layout (không chiếm không gian, không tương tác được). `visibility: hidden` chỉ ẩn visually nhưng **vẫn chiếm không gian** trong layout.

3. **Q:** Khi nào dùng `em` vs `rem` cho font-size?
   **A:** `rem` (root em) — relative đến `<html>` font-size (thường 16px) — nhất quán, dễ scale toàn site. `em` — relative đến font-size của **parent element** — hữu ích cho component tự scale theo context của nó (như `padding: 1em` trên button).

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Tạo tooltip đơn giản: `.tooltip { position: relative }`. Khi hover, hiện `::before` (nội dung tooltip) và `::after` (arrow) dùng `position: absolute`. Tooltip xuất hiện phía trên element.
- [ ] **Bài 2:** Tạo "overlay" full-screen: `position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); z-index: 1000`. Thêm nút đóng (X) ở góc phải trên bằng `position: absolute`.

## 7. Liên kết

- Note liên quan: [[05-CSS-Box-Model]], [[08-CSS-Layout-Flexbox]], [[11-CSS-Units-Color-Border-Background]]
- [MDN: CSS properties reference](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference)
- [MDN: position](https://developer.mozilla.org/en-US/docs/Web/CSS/position)
- [MDN: display](https://developer.mozilla.org/en-US/docs/Web/CSS/display)
