---
title: "CSS: Tổng quan và Cú pháp"
section: 01-HTML-CSS
tags: [css, syntax, basics, cascade, fresher, frontend]
related:
  - "[[04-CSS-Selectors]]"
  - "[[07-CSS-Cascade-Specificity-Inheritance]]"
difficulty: ⭐
estimated_time: 25m
source: [CSS-fundamentals.docx]
---

# CSS: Tổng quan và Cú pháp

> [!summary] TL;DR
> CSS (Cascading Style Sheets) là ngôn ngữ style dùng để kiểm soát **hình thức trình bày** của HTML. Một rule CSS gồm **selector** (chọn phần tử) + **declaration block** (tập hợp property: value). Có 3 cách nhúng CSS: external file, embedded `<style>`, inline `style=""`.

> [!tip] 🎯 Hiểu trong 30 giây
> Nếu HTML là *bộ khung và nội dung* của trang thì **CSS là "lớp trang điểm"** — quyết định màu sắc, font, khoảng cách, bố cục. Một quy tắc (rule) CSS gồm 2 phần: **selector** (chọn *phần tử nào*) + **declaration block** (cặp `thuộc tính: giá trị` mô tả *style gì*).
> ```css
> p { color: red; font-size: 16px; }   /* selector { property: value; } */
> ```
> 3 nơi đặt CSS: **file riêng `.css`** (nên dùng — tách bạch, tái dùng), thẻ `<style>` trong HTML, hay `style="..."` ngay trên thẻ (inline — hạn chế). "Cascading" (xếp tầng) nghĩa là nhiều rule có thể cùng áp lên một phần tử và có quy tắc *ai thắng* (xem [[07-CSS-Cascade-Specificity-Inheritance]]).

## 1. Khái niệm

**CSS** — Cascading Style Sheets — tách biệt **cấu trúc** (HTML) khỏi **trình bày** (CSS). Nhờ đó:
- Thay đổi giao diện toàn site chỉ bằng sửa 1 file CSS.
- Một trang HTML có thể có nhiều stylesheet (print, mobile, dark mode...).

**"Cascading"** nghĩa là nhiều nguồn style có thể kết hợp — khi xung đột, **rule cụ thể nhất** (specificity cao nhất) sẽ thắng.

## 2. Cú pháp / API

```css
/* Anatomy của một CSS rule */
selector {
  property: value;    /* một declaration */
  property: value;    /* declaration thứ hai */
}

/* Ví dụ cụ thể */
p {
  color: #333;
  font-size: 16px;
  line-height: 1.5;
}

/* Nhóm selector — dùng dấu phẩy */
h1, h2, h3 {
  font-family: 'Inter', sans-serif;
}

/* Comment trong CSS */
/* Đây là single-line comment */
/*
  Đây là
  multi-line comment
*/
```

**3 cách nhúng CSS:**

```html
<!-- 1. External stylesheet (tốt nhất cho production) -->
<head>
  <link rel="stylesheet" href="styles.css">
</head>

<!-- 2. Embedded / Internal (trong <head>) -->
<head>
  <style>
    body { margin: 0; }
  </style>
</head>

<!-- 3. Inline (hạn chế dùng — khó override, khó maintain) -->
<p style="color: red; font-weight: bold;">Text</p>
```

## 3. Ví dụ minh họa

### Ví dụ 1: Rule cơ bản và comment có cấu trúc

```css
/* =====================
   TYPOGRAPHY
   ===================== */
body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  font-size: 16px;
  line-height: 1.6;
  color: #212529;
}

h1 { font-size: 2rem; }
h2 { font-size: 1.5rem; }

/* =====================
   LAYOUT
   ===================== */
.container {
  max-width: 1200px;
  margin: 0 auto;       /* căn giữa theo chiều ngang */
  padding: 0 1rem;
}
```

**Output / Kết quả mong đợi:** Font hệ thống, chữ dễ đọc, container căn giữa.

**Giải thích:**
- `-apple-system, BlinkMacSystemFont` là system font stack — không cần load font ngoài, render nhanh.
- `margin: 0 auto` chỉ hoạt động với block element có width xác định.

### Ví dụ 2: Cascade — rule sau ghi đè rule trước

```css
/* Giả sử 2 rule cùng selector, cùng specificity */
p {
  color: blue;
  font-size: 14px;
}

p {
  color: red;   /* ghi đè color ở trên, font-size vẫn 14px */
}

/* HTML: <p>Text này sẽ màu đỏ, size 14px</p> */
```

**Output / Kết quả mong đợi:** Đoạn text màu đỏ — rule thứ hai **overrides** rule đầu vì xuất hiện sau trong source order.

**Giải thích:** "Cascading" hoạt động: cùng specificity → rule cuối cùng trong source thắng.

```
★ Insight ─────────────────────────────────────
• "Tách cấu trúc khỏi trình bày" không chỉ là sạch code: nó cho phép CÙNG một
  HTML khoác nhiều bộ áo (print stylesheet, dark mode, theme khách hàng) chỉ
  bằng đổi file CSS — đó là lý do inline style bị chê (trộn 2 thứ lại, mất khả
  năng tái dùng & cache).
• "Cuối cùng thắng" chỉ đúng KHI specificity ngang nhau. Đây là bậc thấp nhất
  của thuật toán cascade đầy đủ: origin/importance → specificity → source order
  (xem [[07-CSS-Cascade-Specificity-Inheritance]]). Nhớ thứ hạng này thì không
  bao giờ ngạc nhiên "sao rule viết sau mà không thắng".
─────────────────────────────────────────────────
```

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Lỗi phổ biến
> - **Quên dấu chấm phẩy `;`:** `color: red` thiếu `;` sẽ làm property tiếp theo cũng bị bỏ qua.
> - **Dùng inline style quá nhiều:** Inline style có specificity cao nhất (trừ `!important`), rất khó override sau này.
> - **`<link>` sau `<style>`:** External stylesheet đặt sau embedded style sẽ override embedded style vì cascade theo source order.
> - **Khoảng trắng trong selector có nghĩa:** `div p` (descendant) khác `divp` (lỗi cú pháp).

## 5. Câu hỏi phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "CSS là gì, khác HTML thế nào?"
> *"HTML định nghĩa cấu trúc và nội dung của trang, tức là có cái gì, còn CSS định nghĩa hình thức trình bày, tức là trông như thế nào: màu sắc, font chữ, khoảng cách, bố cục. Hai ngôn ngữ có cú pháp hoàn toàn khác nhau, HTML là markup còn CSS là style sheet. Một quy tắc CSS gồm selector để chọn phần tử và declaration block chứa các cặp thuộc tính và giá trị. Em thường viết CSS ở file riêng để tách bạch và tái sử dụng, hạn chế inline style. Cascading nghĩa là nhiều quy tắc có thể cùng áp lên một phần tử và có cơ chế xác định quy tắc nào thắng."*

> [!note] 🧠 Mẹo nhớ
> **HTML = nội dung/cấu trúc (cái gì); CSS = trình bày (trông ra sao).** Rule = **selector { property: value }**. Ưu tiên **file CSS riêng** hơn inline.

1. **Q:** CSS là gì và nó khác gì với HTML?
   **A:** HTML định nghĩa **cấu trúc và nội dung** (what), CSS định nghĩa **hình thức trình bày** (how it looks). HTML là markup language, CSS là style sheet language — cú pháp hoàn toàn khác nhau.

2. **Q:** 3 cách nhúng CSS và ưu/nhược điểm của từng cách?
   **A:** External (file `.css` riêng) — tốt nhất: cache được, dùng chung nhiều trang; Embedded (`<style>` trong `<head>`) — nhanh cho single-page; Inline (`style=""`) — ưu tiên cao nhất trong cascade nhưng khó maintain, không tái sử dụng được.

3. **Q:** "Cascading" trong CSS nghĩa là gì?
   **A:** Nhiều nguồn style (browser defaults, external, embedded, inline) có thể áp dụng cùng lúc. Khi xung đột, thuật toán cascade xét theo thứ tự: **origin + importance → specificity → source order** để chọn rule thắng.

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Tạo file `styles.css`, định nghĩa style cho: body (font-family, color, margin), h1 (color, font-size), p (line-height), a (color, text-decoration). Link vào HTML và kiểm tra.
- [ ] **Bài 2:** Thực hành cascade: trong cùng 1 file CSS, viết 3 rule cho `h1` với color khác nhau. Dự đoán màu nào sẽ thắng rồi kiểm tra trong browser.

## 7. Liên kết

- Note liên quan: [[04-CSS-Selectors]], [[05-CSS-Box-Model]], [[07-CSS-Cascade-Specificity-Inheritance]]
- [MDN: CSS reference](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference)
- [MDN: How CSS works](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps/How_CSS_works)
