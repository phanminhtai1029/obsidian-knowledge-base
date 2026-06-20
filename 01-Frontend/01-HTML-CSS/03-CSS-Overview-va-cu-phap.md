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

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Lỗi phổ biến
> - **Quên dấu chấm phẩy `;`:** `color: red` thiếu `;` sẽ làm property tiếp theo cũng bị bỏ qua.
> - **Dùng inline style quá nhiều:** Inline style có specificity cao nhất (trừ `!important`), rất khó override sau này.
> - **`<link>` sau `<style>`:** External stylesheet đặt sau embedded style sẽ override embedded style vì cascade theo source order.
> - **Khoảng trắng trong selector có nghĩa:** `div p` (descendant) khác `divp` (lỗi cú pháp).

## 5. Câu hỏi phỏng vấn thường gặp

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
