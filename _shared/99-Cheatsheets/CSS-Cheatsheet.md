---
title: "CSS Cheatsheet"
section: 99-Cheatsheets
tags: [cheatsheet, css, flexbox, grid, reference, fresher, frontend]
related:
  - "[[05-CSS-Box-Model]]"
  - "[[08-CSS-Layout-Flexbox]]"
  - "[[09-CSS-Layout-Grid]]"
  - "[[10-CSS-Responsive-Media-Query]]"
  - "[[11-CSS-Units-Color-Border-Background]]"
difficulty: ⭐
estimated_time: 10m
source: [MDN Web Docs, css-tricks.com/snippets/css/a-guide-to-flexbox]
---

# CSS Cheatsheet

> [!summary] TL;DR
> Tra nhanh Flexbox, Grid, Box Model, Selectors và Media Query trước phỏng vấn. Điểm hay hỏi nhất: Flexbox axis, Grid template areas, specificity, `position` stacking context.

---

## 1. Box Model

```text
┌──────────────────────────────┐
│           margin             │
│  ┌────────────────────────┐  │
│  │        border          │  │
│  │  ┌──────────────────┐  │  │
│  │  │     padding      │  │  │
│  │  │  ┌────────────┐  │  │  │
│  │  │  │  content   │  │  │  │
│  │  │  └────────────┘  │  │  │
│  │  └──────────────────┘  │  │
│  └────────────────────────┘  │
└──────────────────────────────┘
```

| Property | Mô tả |
|---|---|
| `box-sizing: content-box` | (default) width = content only |
| `box-sizing: border-box` | width = content + padding + border |

> [!tip] Luôn đặt `* { box-sizing: border-box }` trong reset CSS — tính toán layout dễ hơn nhiều.

---

## 2. Flexbox — Container Properties

```css
.container {
  display: flex;                      /* hoặc inline-flex */
  flex-direction: row;                /* row | row-reverse | column | column-reverse */
  flex-wrap: nowrap;                  /* nowrap | wrap | wrap-reverse */
  justify-content: flex-start;        /* căn theo main axis */
  align-items: stretch;               /* căn theo cross axis */
  align-content: stretch;             /* khi có nhiều dòng (wrap) */
  gap: 16px;                          /* khoảng cách giữa items */
}
```

### justify-content (main axis)

| Giá trị | Mô tả |
|---|---|
| `flex-start` | dồn về đầu |
| `flex-end` | dồn về cuối |
| `center` | căn giữa |
| `space-between` | khoảng cách đều, 2 đầu dính sát |
| `space-around` | khoảng cách đều, 2 đầu có nửa gap |
| `space-evenly` | khoảng cách đều tất cả (kể cả 2 đầu) |

### align-items (cross axis)

| Giá trị | Mô tả |
|---|---|
| `stretch` | kéo dài theo cross axis (default) |
| `flex-start` | dồn về đầu cross axis |
| `flex-end` | dồn về cuối cross axis |
| `center` | căn giữa cross axis |
| `baseline` | căn theo baseline text |

## 3. Flexbox — Item Properties

```css
.item {
  flex-grow: 0;    /* hệ số mở rộng khi có không gian thừa (default: 0) */
  flex-shrink: 1;  /* hệ số thu nhỏ khi thiếu không gian (default: 1) */
  flex-basis: auto;/* kích thước ban đầu trước khi grow/shrink */

  /* Shorthand: grow shrink basis */
  flex: 0 1 auto;  /* default */
  flex: 1;         /* = flex: 1 1 0 — item chiếm đều không gian */
  flex: auto;      /* = flex: 1 1 auto */
  flex: none;      /* = flex: 0 0 auto — không co không dãn */

  align-self: auto;    /* override align-items cho item này */
  order: 0;            /* thứ tự hiển thị (default: 0, thấp hơn = trước) */
}
```

---

## 4. CSS Grid — Container Properties

```css
.grid {
  display: grid;
  grid-template-columns: 200px 1fr 1fr;        /* 3 cột */
  grid-template-columns: repeat(3, 1fr);       /* 3 cột bằng nhau */
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); /* responsive */
  grid-template-rows: auto 1fr auto;           /* header, main, footer */
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  gap: 16px;                                   /* row-gap + column-gap */
}
```

## 5. CSS Grid — Item Properties

```css
.item {
  grid-column: 1 / 3;           /* từ line 1 đến line 3 (chiếm 2 cột) */
  grid-column: 1 / span 2;      /* từ line 1, span 2 cột */
  grid-row: 2 / 4;              /* từ line 2 đến line 4 */
  grid-area: header;            /* dùng với grid-template-areas */
}
```

```css
/* Ví dụ layout với named areas */
.layout {
  display: grid;
  grid-template-areas:
    "nav  nav"
    "side main"
    "foot foot";
  grid-template-columns: 240px 1fr;
  grid-template-rows: 60px 1fr 40px;
  min-height: 100vh;
}
.nav  { grid-area: nav; }
.side { grid-area: side; }
.main { grid-area: main; }
.foot { grid-area: foot; }
```

---

## 6. Selectors & Specificity

| Selector | Ví dụ | Specificity |
|---|---|---|
| Inline style | `style="..."` | 1,0,0,0 |
| ID | `#header` | 0,1,0,0 |
| Class / pseudo-class / attr | `.btn`, `:hover`, `[type]` | 0,0,1,0 |
| Element / pseudo-element | `div`, `::before` | 0,0,0,1 |
| Universal | `*` | 0,0,0,0 |

```css
/* Combinators */
div p         /* descendant — p con cháu của div */
div > p       /* child — p con trực tiếp */
div + p       /* adjacent sibling — p ngay sau div */
div ~ p       /* general sibling — tất cả p sau div cùng cha */
```

> [!warning] Tránh dùng `!important` — phá vỡ cascade, khó debug. Nếu cần tăng specificity, thêm class thay vì `!important`.

---

## 7. Position

| Giá trị | Mô tả |
|---|---|
| `static` | (default) theo flow bình thường |
| `relative` | offset từ vị trí gốc, vẫn chiếm không gian |
| `absolute` | relative đến ancestor có `position` khác static |
| `fixed` | relative đến viewport, không scroll |
| `sticky` | relative + fixed khi scroll qua threshold |

```css
/* Pattern hay dùng: absolute centering */
.parent { position: relative; }
.child  {
  position: absolute;
  top: 50%; left: 50%;
  transform: translate(-50%, -50%);
}
```

---

## 8. Units

| Unit | Loại | So sánh với |
|---|---|---|
| `px` | absolute | pixel |
| `%` | relative | parent element |
| `em` | relative | font-size của **element hiện tại** |
| `rem` | relative | font-size của **root (html)** |
| `vw` | relative | 1% chiều rộng viewport |
| `vh` | relative | 1% chiều cao viewport |
| `ch` | relative | chiều rộng ký tự "0" |
| `fr` | relative | chỉ dùng trong Grid |

```css
/* Pattern chuẩn: rem cho font-size, px cho border/shadow */
html { font-size: 16px; }
body { font-size: 1rem; }  /* = 16px */
h1   { font-size: 2rem; }  /* = 32px */
```

---

## 9. Media Queries

```css
/* Mobile-first (khuyến nghị) */
.container { padding: 16px; }

@media (min-width: 768px) {
  .container { padding: 24px; }    /* tablet trở lên */
}

@media (min-width: 1024px) {
  .container { padding: 32px; }   /* desktop trở lên */
}

/* Breakpoints phổ biến */
/* sm: 640px | md: 768px | lg: 1024px | xl: 1280px */
```

---

## 10. CSS Custom Properties (Variables)

```css
:root {
  --color-primary: #0066cc;
  --spacing-base: 8px;
  --font-size-base: 16px;
}

.button {
  background: var(--color-primary);
  padding: calc(var(--spacing-base) * 2);
  font-size: var(--font-size-base);
}

/* Override trong scope */
.dark-theme {
  --color-primary: #4da6ff;
}
```

---

## 11. Câu Hỏi Phỏng Vấn Thường Gặp

**Q1: Flexbox vs Grid — dùng khi nào?**

> **Flexbox**: 1 chiều — layout theo hàng ngang hoặc cột dọc, item tự điều chỉnh. Dùng cho navbar, card row, button group. **Grid**: 2 chiều — layout theo cả hàng lẫn cột cùng lúc. Dùng cho page layout, gallery, dashboard. Thực tế: kết hợp cả hai — Grid cho page layout, Flexbox cho components bên trong.

**Q2: `em` vs `rem` khác nhau thế nào?**

> `em` relative đến `font-size` của **chính element** (hoặc parent nếu dùng cho font-size). `rem` (root em) luôn relative đến `font-size` của `html`. `rem` dễ dự đoán hơn vì không bị compound: `html { font-size: 16px }` → `1rem = 16px` mọi nơi. Dùng `rem` cho typography, `em` cho spacing trong components nhỏ.

**Q3: Specificity tính thế nào?**

> 4 số `a,b,c,d`: a=inline style, b=ID selector, c=class/pseudo-class/attr, d=element/pseudo-element. So sánh từ trái sang phải. `#id .class p` = 0,1,1,1; `.class .class` = 0,0,2,0; `#id` thắng vì b=1 > b=0.

---

## 12. Bài Tập Tự Kiểm Tra

- [ ] Tạo layout 3 cột bằng Grid với cột giữa rộng gấp đôi 2 cột còn lại, responsive xuống 1 cột ở mobile.
- [ ] Căn giữa 1 element (cả ngang lẫn dọc) trong container bằng 3 cách khác nhau: Flexbox, Grid, absolute+transform.

---

## 13. Liên Kết

- [[08-CSS-Layout-Flexbox]] — Flexbox chi tiết
- [[09-CSS-Layout-Grid]] — Grid chi tiết
- [[05-CSS-Box-Model]] — Box model
- [[10-CSS-Responsive-Media-Query]] — Responsive design
