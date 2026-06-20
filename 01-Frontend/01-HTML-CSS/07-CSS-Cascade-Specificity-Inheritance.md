---
title: "CSS Cascade, Specificity & Inheritance"
section: 01-HTML-CSS
tags: [css, cascade, specificity, inheritance, important, fresher, frontend]
related:
  - "[[03-CSS-Overview-va-cu-phap]]"
  - "[[04-CSS-Selectors]]"
difficulty: ⭐⭐⭐
estimated_time: 45m
source: [CSS-fundamentals.docx]
---

# CSS Cascade, Specificity & Inheritance

> [!summary] TL;DR
> Khi nhiều CSS rule áp dụng cho cùng một element, thuật toán **cascade** quyết định rule nào thắng theo thứ tự: **Origin & Importance → Specificity → Source Order**. **Specificity** tính theo (ID, Class, Element) — ví dụ `#id .class p` = (1,1,1). **Inheritance** là cơ chế con thừa kế một số property của cha (font, color) nhưng không phải tất cả (margin, padding).

## 1. Khái niệm

### Cascade (Thác nước)
Thuật toán CSS cascade xử lý xung đột theo 4 yếu tố theo thứ tự ưu tiên giảm dần:

1. **Origin + Importance:** `!important` từ user agent > `!important` từ author > author styles > user styles > user agent (browser default).
2. **Specificity:** Điểm số dựa trên loại selector.
3. **Source Order:** Rule xuất hiện sau trong file thắng nếu cùng specificity.
4. **Inheritance:** Giá trị thừa kế từ cha nếu không có rule nào áp dụng.

### Specificity (Độ cụ thể)
Tính theo hệ `(a, b, c)`:
- `a` = số ID selectors (`#id`)
- `b` = số class, attribute, pseudo-class selectors (`.class`, `[attr]`, `:hover`)
- `c` = số type, pseudo-element selectors (`p`, `::before`)

### Inheritance (Thừa kế)
Một số property **thừa kế**: `color`, `font-*`, `line-height`, `text-align`, `visibility`...
Một số property **không thừa kế**: `margin`, `padding`, `border`, `width`, `height`, `display`...

## 2. Cú pháp / API

```css
/* Specificity examples */
p              { color: red; }    /* (0,0,1) = 1 điểm */
.text          { color: blue; }   /* (0,1,0) = 10 điểm */
#title         { color: green; }  /* (1,0,0) = 100 điểm */
#title.special { color: purple; } /* (1,1,0) = 110 điểm */

/* !important — bỏ qua specificity, chỉ so với !important khác */
p { color: red !important; }  /* Thắng mọi rule không có !important */

/* inherit — ép thừa kế */
.child { color: inherit; }

/* initial — về giá trị default của browser */
.reset { color: initial; }

/* unset — inherit nếu property được thừa kế, ngược lại initial */
.unset { color: unset; }
```

## 3. Ví dụ minh họa

### Ví dụ 1: Cascade và specificity quyết định màu sắc

```html
<div id="container" class="wrapper">
  <p class="text" style="color: purple;">Màu này là gì?</p>
</div>
```

```css
/* (0,0,1) = 1 điểm */
p { color: red; }

/* (0,1,0) = 10 điểm */
.text { color: blue; }

/* (1,0,0) = 100 điểm */
#container p { color: green; }  /* (1,0,1) = 101 */

/* Inline style = (1,0,0,0) — luôn thắng specificity thông thường */
/* style="color: purple" */
```

**Thứ tự thắng (cao → thấp):**
1. `style="color: purple"` — **Inline style** (highest specificity)
2. `#container p` (1,0,1) = 101
3. `.text` (0,1,0) = 10
4. `p` (0,0,1) = 1

**Output / Kết quả mong đợi:** Text màu **purple** (inline style thắng).

### Ví dụ 2: Inheritance — font-family lan xuống toàn trang

```css
/* Đặt font ở body, tất cả con thừa kế */
body {
  font-family: 'Inter', sans-serif;
  font-size: 16px;
  color: #212529;
  line-height: 1.6;
}

/* h1 KHÔNG thừa kế font-size từ body — browser có default style cho h1 */
/* Nhưng THỪA KẾ font-family và color */
h1 {
  /* font-family: 'Inter' — thừa kế từ body */
  font-size: 2rem;  /* override browser default (2em) */
}

/* Forced inheritance */
.icon {
  color: inherit;   /* Icon màu theo text của parent */
}
```

**Giải thích:**
- `color` và `font-family` được thừa kế từ `body` xuống tất cả con cháu.
- `font-size: 2rem` của `h1` **không** thừa kế từ body — đây là browser default UA stylesheet.
- `color: inherit` trên `.icon` ép nó lấy màu từ element cha ngay trên.

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Lỗi phổ biến
> - **Lạm dụng `!important`:** Một khi dùng, phải dùng `!important` để override `!important` khác — tạo ra "arms race" specificity, debug rất khó. Chỉ dùng khi không có cách nào khác (override third-party CSS).
> - **Overspecific selectors:** `div.container > ul.nav > li.nav-item > a.link` — specificity (0,3,3) = 33 điểm, gần như không override được. Ưu tiên dùng class đơn.
> - **`color` thừa kế nhưng `border` không:** Nhiều người nhầm tưởng tất cả property đều thừa kế. Kiểm tra MDN để xác nhận.

> [!tip] Mẹo debug specificity
> Dùng DevTools (F12) > Elements > Computed: CSS rules được liệt kê theo specificity, rule bị gạch chân = bị override.

## 5. Câu hỏi phỏng vấn thường gặp

1. **Q:** Tính specificity của: `div#app .nav > a:hover`
   **A:** ID: 1, Class+PseudoClass: 2 (`.nav` + `:hover`), Element: 2 (`div` + `a`) → **(1, 2, 2)** = 122 điểm.

2. **Q:** Khi nào nên dùng `!important`?
   **A:** Chỉ khi cần override CSS từ nguồn bên ngoài không kiểm soát được (third-party widget, library). Trong code tự viết, **tránh hoàn toàn** — là dấu hiệu của specificity problem cần refactor.

3. **Q:** Property nào thừa kế (inherited), property nào không?
   **A:** **Thừa kế:** `color`, `font-*`, `line-height`, `letter-spacing`, `text-align`, `visibility`, `cursor`. **Không thừa kế:** `margin`, `padding`, `border`, `width`, `height`, `display`, `position`, `background`. Có thể ép thừa kế với `inherit`.

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Tính specificity của các selector sau, xếp theo thứ tự từ thấp đến cao: `*`, `p`, `.card`, `#main`, `div.card p`, `#main .nav > a:hover`, `style=""`.
- [ ] **Bài 2:** Tạo HTML có conflict giữa class selector và element selector. Dùng DevTools xem Computed styles để thấy rule nào bị override. Sau đó thêm `!important` vào rule thua và quan sát thay đổi.

## 7. Liên kết

- Note liên quan: [[04-CSS-Selectors]], [[03-CSS-Overview-va-cu-phap]], [[08-CSS-Layout-Flexbox]]
- [MDN: Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity)
- [MDN: Cascade](https://developer.mozilla.org/en-US/docs/Web/CSS/Cascade)
- [MDN: Inheritance](https://developer.mozilla.org/en-US/docs/Web/CSS/Inheritance)
- [Specificity Calculator](https://specificity.keegan.st/)
