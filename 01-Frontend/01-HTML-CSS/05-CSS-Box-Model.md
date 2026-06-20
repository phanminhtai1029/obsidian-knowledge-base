---
title: "CSS Box Model"
section: 01-HTML-CSS
tags: [css, box-model, margin, padding, border, box-sizing, fresher, frontend]
related:
  - "[[04-CSS-Selectors]]"
  - "[[08-CSS-Layout-Flexbox]]"
difficulty: ⭐⭐
estimated_time: 35m
source: [CSS-fundamentals.docx]
---

# CSS Box Model

> [!summary] TL;DR
> Mọi phần tử HTML đều là một **hình chữ nhật** gồm 4 lớp từ trong ra ngoài: **content → padding → border → margin**. Mặc định `box-sizing: content-box` — width chỉ tính content. Đổi thành `box-sizing: border-box` để width bao gồm cả padding và border — dễ tính toán hơn nhiều.

## 1. Khái niệm

**CSS Box Model** là quy tắc mô tả cách browser tính toán kích thước và khoảng cách của mọi phần tử HTML.

**4 lớp (từ trong ra ngoài):**

```text
┌──────────────────────────────────────────┐ ← margin (không màu)
│  ┌────────────────────────────────────┐  │
│  │  border (có thể có màu, nét)       │  │
│  │  ┌──────────────────────────────┐  │  │
│  │  │  padding (trong suốt)        │  │  │
│  │  │  ┌────────────────────────┐  │  │  │
│  │  │  │   content (text/img)   │  │  │  │
│  │  │  └────────────────────────┘  │  │  │
│  │  └──────────────────────────────┘  │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

- **Content:** Vùng chứa text, hình ảnh — kích thước do `width`/`height` hoặc nội dung quyết định.
- **Padding:** Khoảng trống trong suốt giữa content và border. Có màu nền theo background của element.
- **Border:** Đường viền hiển thị được, có thể chỉnh style/width/color.
- **Margin:** Khoảng trống ngoài border, **trong suốt**, không tính vào tổng width của element.

## 2. Cú pháp / API

```css
/* Shorthand cho padding và margin: top right bottom left (clockwise) */
.box {
  padding: 10px 20px 10px 20px;  /* 4 giá trị: T R B L */
  padding: 10px 20px;            /* 2 giá trị: TB  LR */
  padding: 10px;                 /* 1 giá trị: tất cả bằng nhau */

  margin: 0 auto;                /* Căn giữa block element */
  margin-top: 16px;              /* Từng cạnh riêng lẻ */
}

/* Border shorthand: width style color */
.box {
  border: 2px solid #333;
  border-top: 4px dashed red;    /* Chỉ một cạnh */
  border-radius: 8px;            /* Bo góc */
}

/* box-sizing — reset mặc định cho toàn bộ site */
*, *::before, *::after {
  box-sizing: border-box;        /* width/height = content + padding + border */
}

.card {
  width: 300px;
  padding: 20px;
  border: 2px solid #ccc;
  /* với border-box: tổng width = 300px (không thêm padding/border) */
  /* với content-box (default): tổng width = 300+40+4 = 344px */
}
```

## 3. Ví dụ minh họa

### Ví dụ 1: So sánh content-box và border-box

```html
<div class="content-box-demo">content-box (default)</div>
<div class="border-box-demo">border-box</div>
```

```css
/* Cả hai đều có width: 200px, padding: 20px, border: 5px */

.content-box-demo {
  box-sizing: content-box;  /* default */
  width: 200px;
  padding: 20px;
  border: 5px solid blue;
  background: lightblue;
  /* Tổng chiều rộng thực = 200 + 20*2 + 5*2 = 250px */
}

.border-box-demo {
  box-sizing: border-box;
  width: 200px;
  padding: 20px;
  border: 5px solid green;
  background: lightgreen;
  /* Tổng chiều rộng thực = 200px (padding và border nằm TRONG 200px) */
  /* Content width thực = 200 - 20*2 - 5*2 = 150px */
}
```

**Output / Kết quả mong đợi:** Box đầu rộng 250px, box thứ hai rộng 200px dù cùng `width: 200px`.

**Giải thích:** `border-box` trực quan hơn vì "width là width" — padding và border không làm tràn layout.

```
★ Insight ─────────────────────────────────────
• "width là width" là lý do gần như mọi codebase mở đầu bằng
  `*, *::before, *::after { box-sizing: border-box }`. Với content-box, mỗi lần
  thêm padding bạn phải tự trừ vào width → toán nhẩm liên tục và dễ tràn grid.
  border-box biến width thành con số bạn THẤY, không phải con số phải TÍNH.
• Padding ăn màu background còn margin trong suốt — đây là cách phân biệt nhanh
  trong DevTools: vùng có màu nền nằm trong border là padding, vùng trống ngoài
  border (highlight cam) là margin. Nhớ mẹo này thì không bao giờ nhầm 2 lớp.
─────────────────────────────────────────────────
```

### Ví dụ 2: Margin collapsing

```html
<p class="top">Đoạn 1 — margin-bottom: 30px</p>
<p class="bottom">Đoạn 2 — margin-top: 20px</p>
```

```css
.top    { margin-bottom: 30px; background: lightyellow; }
.bottom { margin-top: 20px;   background: lightpink;   }

/* Khoảng cách giữa 2 đoạn = max(30, 20) = 30px — KHÔNG PHẢI 50px */
```

**Output / Kết quả mong đợi:** Khoảng cách giữa 2 đoạn là 30px (margin collapse).

**Giải thích:** **Margin collapsing** xảy ra với vertical margins của block elements liền kề — hai margin "sáp nhập" thành margin lớn nhất, không cộng lại.

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Lỗi phổ biến
> - **Quên `box-sizing: border-box`:** `width: 100%` + `padding: 20px` = tràn container. Luôn apply `border-box` toàn site với `* { box-sizing: border-box }`.
> - **Margin collapsing bất ngờ:** Hai paragraph liền nhau không có khoảng cách bằng tổng margin — margin lớn hơn sẽ "thắng". Fix: dùng `padding`, `flexbox`, hoặc `overflow: hidden` trên parent.
> - **Margin không áp dụng cho inline elements:** `<span>`, `<a>` không nhận `margin-top/bottom`. Đổi thành `display: inline-block` hoặc `block`.
> - **Padding có màu background:** Padding lấy màu nền của element, không transparent như margin.

## 5. Câu hỏi phỏng vấn thường gặp

1. **Q:** Sự khác nhau giữa `padding` và `margin`?
   **A:** `padding` là khoảng cách **bên trong** border (giữa content và border) — có màu nền, tính vào kích thước element. `margin` là khoảng cách **ngoài** border (giữa element và element khác) — transparent, không tính vào width của element.

2. **Q:** `box-sizing: border-box` làm gì và tại sao nên dùng?
   **A:** Với `content-box` (default): `width` chỉ tính content — padding và border cộng thêm vào. Với `border-box`: `width` bao gồm cả content + padding + border. Dùng `border-box` vì trực quan hơn, tránh overflow khi thêm padding.

3. **Q:** Margin collapsing là gì? Khi nào xảy ra?
   **A:** Khi hai block elements liền nhau có vertical margin, thay vì cộng lại, chúng **sáp nhập** thành margin lớn nhất. Xảy ra giữa: siblings dọc, parent-child (khi không có border/padding ngăn cách), empty block. Không xảy ra trong flex/grid container.

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Tạo card component: `width: 320px`, `padding: 24px`, `border: 1px solid #e0e0e0`, `border-radius: 12px`, `box-shadow: 0 2px 8px rgba(0,0,0,0.1)`. Kiểm tra kích thước thực trong DevTools với cả `content-box` và `border-box`.
- [ ] **Bài 2:** Tạo 3 `<p>` liền nhau với margin-bottom: 40px. Quan sát margin collapsing trong DevTools (tab Computed > Box Model). Sau đó wrap chúng trong `display: flex; flex-direction: column` — collapsing biến mất.

## 7. Liên kết

- Note liên quan: [[04-CSS-Selectors]], [[06-CSS-Properties-thong-dung]], [[08-CSS-Layout-Flexbox]]
- [MDN: Box model](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/The_box_model)
- [MDN: box-sizing](https://developer.mozilla.org/en-US/docs/Web/CSS/box-sizing)
- [MDN: Mastering margin collapsing](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_box_model/Mastering_margin_collapsing)
