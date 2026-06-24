---
title: "CSS Responsive & Media Query"
section: 01-HTML-CSS
tags: [css, responsive, media-query, mobile-first, breakpoint, fresher, frontend]
related:
  - "[[08-CSS-Layout-Flexbox]]"
  - "[[09-CSS-Layout-Grid]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [CSS-fundamentals.docx]
---

# CSS Responsive & Media Query

> [!summary] TL;DR
> [!tip] 🎯 Hiểu trong 30 giây
> **Responsive = web tự co giãn/sắp xếp lại cho vừa MỌI màn hình** (điện thoại, tablet, desktop) thay vì làm nhiều trang riêng. Công cụ chính là **Media Query** — "CSS có điều kiện": *nếu màn rộng ≥ 768px thì áp dụng style này*.
> - **Mobile-First** (cách được khuyến nghị): viết CSS *cho điện thoại trước* (đơn giản nhất), rồi dùng `@media (min-width: ...)` **thêm dần** style cho màn lớn hơn. Ngược lại là Desktop-First (viết cho desktop trước rồi `max-width` thu nhỏ dần).
> - **Vì sao Mobile-First tốt hơn (ra thi):** đa số người dùng vào bằng điện thoại; CSS nền nhẹ rồi *bồi thêm* cho màn lớn thì sạch hơn là viết phức tạp rồi *gỡ bớt* cho mobile; tránh tải style thừa trên thiết bị yếu.
>
> Responsive Web Design là thiết kế web thích nghi với mọi kích thước màn hình. Ba kỹ thuật chính: **Media Queries** (áp dụng CSS theo điều kiện viewport), **Flexible layouts** (Flexbox/Grid), **Fluid images** (`max-width: 100%`). Cách tiếp cận **mobile-first** (viết CSS cho mobile trước, thêm breakpoint cho màn hình lớn) được khuyến nghị.

## 1. Khái niệm

**Responsive Web Design (RWD)** — thiết kế 1 codebase HTML/CSS hoạt động tốt trên mọi thiết bị (điện thoại, tablet, desktop, TV...).

**3 trụ cột:**
1. **Fluid layouts:** Dùng `%`, `fr`, `auto`, Flexbox, Grid — không dùng px cố định cho width.
2. **Flexible images/media:** `max-width: 100%; height: auto` — hình ảnh co lại theo container.
3. **Media queries:** Áp dụng CSS khác nhau theo điều kiện (viewport width, orientation, resolution...).

**Viewport meta tag — bắt buộc:**
```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```
Không có tag này, mobile browser sẽ zoom out trang desktop → responsive CSS không hoạt động.

## 2. Cú pháp / API

```css
/* Cú pháp media query */
@media media-type and (media-feature) {
  /* CSS áp dụng khi điều kiện đúng */
}

/* Phổ biến nhất: screen width */
@media screen and (min-width: 768px) { /* tablet + desktop */ }
@media screen and (max-width: 767px) { /* mobile */ }

/* Mobile-first approach (khuyến nghị) */
/* Viết CSS base cho mobile, override lên cho màn lớn hơn */

/* Base styles (mobile) */
.container { padding: 1rem; }
.nav       { display: none; } /* Ẩn nav trên mobile */

/* Tablet: min-width: 768px */
@media (min-width: 768px) {
  .container { padding: 2rem; }
  .nav { display: flex; } /* Hiện nav trên tablet+ */
}

/* Desktop: min-width: 1024px */
@media (min-width: 1024px) {
  .container { max-width: 1200px; margin: 0 auto; }
}

/* Orientation */
@media (orientation: landscape) { }
@media (orientation: portrait)  { }

/* Dark mode */
@media (prefers-color-scheme: dark) {
  body { background: #121212; color: #e0e0e0; }
}

/* Print */
@media print {
  .nav, .footer, .ads { display: none; }
  body { font-size: 12pt; color: black; }
}

/* Hover capability (touch vs mouse) */
@media (hover: hover) {
  button:hover { background: lightblue; }  /* Chỉ áp dụng khi device có hover */
}
```

**Breakpoints phổ biến (dùng với mobile-first):**

| Breakpoint | `min-width` | Thiết bị |
|-----------|------------|---------|
| sm | 576px | Landscape phone |
| md | 768px | Tablet |
| lg | 1024px | Desktop |
| xl | 1280px | Large desktop |
| 2xl | 1536px | Extra large |

## 3. Ví dụ minh họa

### Ví dụ 1: Responsive card grid — mobile-first

```css
/* BASE (mobile) — 1 cột */
.card-grid {
  display: grid;
  gap: 1rem;
  grid-template-columns: 1fr;
}

/* Tablet — 2 cột */
@media (min-width: 768px) {
  .card-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 1.5rem;
  }
}

/* Desktop — 3 cột */
@media (min-width: 1024px) {
  .card-grid {
    grid-template-columns: repeat(3, 1fr);
    gap: 2rem;
  }
}
```

**Output / Kết quả mong đợi:**
- Mobile (< 768px): 1 card/hàng
- Tablet (768px-1023px): 2 card/hàng
- Desktop (≥ 1024px): 3 card/hàng

### Ví dụ 2: Responsive navigation

```html
<header>
  <div class="logo">Brand</div>
  <button class="hamburger" aria-label="Menu">☰</button>
  <nav class="nav">
    <a href="/">Trang chủ</a>
    <a href="/blog">Blog</a>
  </nav>
</header>
```

```css
/* Mobile: nav ẩn, hamburger hiện */
header     { display: flex; align-items: center; justify-content: space-between; padding: 1rem; }
.hamburger { display: block; font-size: 1.5rem; background: none; border: none; cursor: pointer; }
.nav       { display: none; }

/* Desktop: nav hiện, hamburger ẩn */
@media (min-width: 768px) {
  .hamburger { display: none; }
  .nav {
    display: flex;
    gap: 2rem;
  }
  .nav a { text-decoration: none; color: #333; }
}
```

**Output / Kết quả mong đợi:** Mobile hiện hamburger button, desktop hiện nav links ngang.

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Lỗi phổ biến
> - **Quên viewport meta tag:** Responsive CSS không hoạt động trên mobile nếu thiếu `<meta name="viewport">`.
> - **Desktop-first với `max-width`:** Khó maintain, thêm override liên tục. Chuyển sang **mobile-first** (`min-width`) để code ngắn hơn.
> - **Dùng px cho layout:** `width: 960px` không responsive. Dùng `max-width: 960px` + `width: 100%`.
> - **Ảnh không responsive:** Ảnh mặc định không co theo container. Thêm `img { max-width: 100%; height: auto; }` vào global CSS.

> [!tip] Tip kiểm tra responsive
> Chrome DevTools: F12 → Toggle Device Toolbar (Ctrl+Shift+M) — giả lập kích thước màn hình và touch events.

```
★ Insight ─────────────────────────────────────
• Mobile-first không chỉ là "thẩm mỹ": với min-width, CSS base là phiên bản
  ĐƠN GIẢN NHẤT (1 cột, ít tính năng) rồi BỔ SUNG dần khi màn rộng ra → mỗi
  query chỉ thêm, không gỡ. Desktop-first (max-width) buộc bạn LIÊN TỤC undo
  những gì màn lớn đã set → CSS phình và dễ vỡ. Ít override = ít bug.
• Bước tiến mới: `@container` (container query) căn theo kích thước CỦA CHA
  chứa nó, không phải viewport. Nhờ vậy một card component "tự responsive" dù
  nằm ở sidebar hẹp hay main rộng — điều media query (chỉ biết viewport) không
  làm được. Đây là hướng thay thế nhiều media query ở cấp component.
─────────────────────────────────────────────────
```

## 5. Câu hỏi phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Mobile-First nghĩa là gì, vì sao tối ưu hơn code desktop trước?"
> *"Mobile-First nghĩa là em viết CSS cho màn hình điện thoại trước, lấy đó làm nền, rồi dùng media query với min-width để bổ sung dần style cho màn hình lớn hơn như tablet và desktop. Cách này tối ưu hơn desktop-first vì vài lý do. Thứ nhất, lưu lượng truy cập từ mobile hiện chiếm phần lớn nên ưu tiên mobile là hợp lý. Thứ hai, CSS nền cho mobile thường đơn giản, ít tính năng, rồi mình thêm vào cho màn lớn, sạch và dễ bảo trì hơn là viết bố cục phức tạp cho desktop rồi phải override ngược lại để thu nhỏ cho mobile. Thứ ba, thiết bị yếu không phải tải và ghi đè nhiều style thừa. Cùng tinh thần đó, các framework như Bootstrap hay Tailwind cũng mặc định mobile-first, class không breakpoint áp cho mọi cỡ, thêm tiền tố md hay lg cho màn lớn hơn."*

> [!note] 🧠 Mẹo nhớ
> **Mobile-First = viết cho điện thoại trước, `@media (min-width)` THÊM DẦN cho màn lớn.** Tốt hơn vì: traffic mobile lớn + CSS nền nhẹ rồi bồi thêm (thay vì override ngược). Bootstrap/Tailwind cũng mobile-first.

1. **Q:** Mobile-first vs Desktop-first — cái nào tốt hơn?
   **A:** **Mobile-first** (`min-width`) được khuyến nghị vì: mobile chiếm >60% traffic, CSS base đơn giản hơn (ít tính năng), thêm dần cho màn lớn thay vì override ngược. Desktop-first yêu cầu nhiều override hơn trên mobile.

2. **Q:** Media query có thể kiểm tra điều kiện nào ngoài width?
   **A:** `width/height` (viewport), `orientation` (landscape/portrait), `prefers-color-scheme` (dark/light), `prefers-reduced-motion` (animation preference), `hover` (touch vs mouse), `resolution` (retina display), `print` (in ấn).

3. **Q:** Làm sao tạo layout responsive mà không cần media query?
   **A:** `flexbox` + `flex-wrap: wrap` + `flex: 1 1 280px`, hoặc `grid` + `repeat(auto-fit, minmax(280px, 1fr))` — tự động co giãn số cột theo viewport width mà không cần breakpoint.

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Tạo trang portfolio responsive: mobile (1 cột), tablet (2 cột), desktop (3 cột). Dùng **CSS Grid + media queries**. Thêm dark mode với `@media (prefers-color-scheme: dark)`.
- [ ] **Bài 2:** Implement hamburger menu responsive hoàn chỉnh: mobile hiện icon ☰, click toggle class `.open` bằng JavaScript, `.open .nav { display: flex; flex-direction: column }`. Desktop: nav ngang, hamburger ẩn.

## 7. Liên kết

- Note liên quan: [[08-CSS-Layout-Flexbox]], [[09-CSS-Layout-Grid]], [[13-Bootstrap-Overview]]
- [MDN: Responsive design](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Responsive_Design)
- [MDN: Using media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries)
- [web.dev: Responsive design](https://web.dev/learn/design/)
