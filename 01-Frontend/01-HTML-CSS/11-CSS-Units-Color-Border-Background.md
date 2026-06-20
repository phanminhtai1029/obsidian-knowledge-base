---
title: "CSS Units, Color, Border & Background"
section: 01-HTML-CSS
tags: [css, units, color, border, background, rem, rgb, hsl, fresher, frontend]
related:
  - "[[05-CSS-Box-Model]]"
  - "[[06-CSS-Properties-thong-dung]]"
difficulty: ⭐⭐
estimated_time: 35m
source: [CSS-fundamentals.docx]
---

# CSS Units, Color, Border & Background

> [!summary] TL;DR
> **Units:** Dùng `rem` cho typography, `%`/`fr` cho layout, `px` cho border/shadow. **Color:** Ưu tiên `hsl()` (trực quan nhất), `rgba()` khi cần transparency. **Border:** Shorthand `border: width style color`. **Background:** `background-size: cover` + `background-position: center` để fill ảnh đẹp.

## 1. Khái niệm

### CSS Units (Đơn vị đo)

| Loại | Units | Relative to |
|------|-------|-------------|
| **Absolute** | `px` | Pixel vật lý |
| **Relative to font** | `em` | Font-size của parent |
| | `rem` | Font-size của `<html>` (default 16px) |
| **Relative to viewport** | `vw` | 1% viewport width |
| | `vh` | 1% viewport height |
| | `vmin/vmax` | Min/max của vw và vh |
| **Percentage** | `%` | Phụ thuộc property (width→parent width) |
| **Grid** | `fr` | Fraction of remaining space |

### Color Formats

| Format | Ví dụ | Ghi chú |
|--------|-------|---------|
| Keyword | `red`, `transparent` | 148 màu chuẩn |
| Hex | `#FF5733`, `#f57` | Shorthand 3 chữ số khi pair giống nhau |
| RGB | `rgb(255, 87, 51)` | 0-255 mỗi kênh |
| RGBA | `rgba(255, 87, 51, 0.5)` | Alpha: 0=transparent, 1=opaque |
| HSL | `hsl(14, 100%, 60%)` | Hue(0-360°), Saturation%, Lightness% |
| HSLA | `hsla(14, 100%, 60%, 0.8)` | + Alpha |

## 2. Cú pháp / API

```css
/* ── UNITS ── */
html { font-size: 16px; }    /* Root = 16px → 1rem = 16px */

body { font-size: 1rem; }    /* 16px — scale với user preference */
h1   { font-size: 2.5rem; }  /* 40px */
p    { font-size: 1rem; line-height: 1.6; }

.hero {
  height: 100vh;             /* Full viewport height */
  padding: 5vw;              /* 5% viewport width */
}

.sidebar { width: 30%; }     /* 30% của parent */

/* ── COLOR ── */
.element {
  color: #212529;                      /* Hex dark */
  background-color: hsl(210, 40%, 96%); /* HSL: màu xanh nhạt */
  border-color: rgba(0, 0, 0, 0.15);  /* RGBA: đen 15% opacity */
}

/* HSL rất tiện để tạo color scale */
:root {
  --blue-100: hsl(214, 100%, 97%);    /* Rất nhạt */
  --blue-500: hsl(214, 85%, 55%);     /* Chuẩn */
  --blue-900: hsl(214, 90%, 20%);     /* Rất đậm */
}

/* ── BORDER ── */
.box {
  /* Shorthand: width style color */
  border: 2px solid #dee2e6;
  border-top: 4px solid #0066cc;         /* Override một cạnh */
  border-radius: 8px;                    /* Bo tất cả góc */
  border-radius: 4px 8px 12px 16px;     /* TL TR BR BL */
  border-radius: 50%;                    /* Hình tròn (với width=height) */
}

/* ── BACKGROUND ── */
.hero {
  background: url('hero.jpg') center/cover no-repeat;  /* Shorthand */
  background-color: #f0f4f8;             /* Fallback nếu ảnh không load */
}

.gradient {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}
```

## 3. Ví dụ minh họa

### Ví dụ 1: Color system với CSS custom properties

```css
/* Design token — định nghĩa một lần, dùng khắp nơi */
:root {
  --color-primary:   hsl(220, 90%, 55%);   /* Blue */
  --color-success:   hsl(142, 71%, 45%);   /* Green */
  --color-danger:    hsl(  0, 84%, 60%);   /* Red */
  --color-warning:   hsl( 38, 92%, 55%);   /* Yellow */
  --color-text:      hsl(220, 20%, 15%);   /* Near black */
  --color-muted:     hsl(220, 10%, 55%);   /* Gray */
  --color-bg:        hsl(220, 30%, 97%);   /* Near white */

  /* Spacing scale với rem */
  --space-1: 0.25rem;   /* 4px */
  --space-2: 0.5rem;    /* 8px */
  --space-4: 1rem;      /* 16px */
  --space-8: 2rem;      /* 32px */
}

.btn-primary {
  background-color: var(--color-primary);
  color: white;
  padding: var(--space-2) var(--space-4);
  border-radius: 6px;
  border: none;
}

.btn-primary:hover {
  /* Dùng HSLA để tối màu khi hover */
  background-color: hsl(220, 90%, 45%);  /* Lightness giảm từ 55% xuống 45% */
}

.alert-danger {
  background-color: hsl(0, 84%, 95%);   /* Rất nhạt — cùng hue với danger */
  border: 1px solid var(--color-danger);
  color: hsl(0, 84%, 35%);              /* Đậm hơn để readable */
}
```

**Giải thích:** HSL rất tiện để tạo color scale — chỉ thay đổi Lightness % trong khi giữ nguyên Hue và Saturation.

### Ví dụ 2: Hero section với background ảnh + gradient overlay

```html
<section class="hero">
  <div class="hero-content">
    <h1>Tiêu đề lớn</h1>
    <p>Mô tả ngắn</p>
    <a href="#" class="hero-cta">Bắt đầu ngay</a>
  </div>
</section>
```

```css
.hero {
  position: relative;
  min-height: 80vh;
  display: flex;
  align-items: center;

  /* Gradient phủ lên ảnh để text dễ đọc */
  background:
    linear-gradient(
      to right,
      rgba(0, 0, 0, 0.7) 0%,
      rgba(0, 0, 0, 0.3) 100%
    ),
    url('hero-bg.jpg') center/cover no-repeat;
}

.hero-content {
  max-width: 600px;
  padding: 2rem;
  color: white;
}

.hero-content h1 {
  font-size: clamp(2rem, 5vw, 4rem); /* Fluid typography: min 2rem, tối đa 4rem */
  line-height: 1.2;
  margin-bottom: 1rem;
}

.hero-cta {
  display: inline-block;
  padding: 0.875rem 2rem;
  background: hsl(38, 92%, 55%);
  color: hsl(220, 20%, 10%);
  border-radius: 50px;         /* Pill shape */
  text-decoration: none;
  font-weight: 700;
  transition: transform 0.2s, box-shadow 0.2s;
}

.hero-cta:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(0,0,0,0.25);
}
```

**Output / Kết quả mong đợi:** Hero section ảnh nền mờ gradient, text trắng dễ đọc, CTA button vàng, font size tự scale theo viewport.

```
★ Insight ─────────────────────────────────────
• rem vs em là chuyện "neo vào đâu": rem LUÔN neo vào <html> (cố định, không
  cộng dồn), em neo vào font-size CHA (cộng dồn → "snowball" khi lồng nhau). Quy
  tắc thực dụng: rem cho font-size & spacing toàn cục (nhất quán); em cho padding
  TRONG component để nó tự co theo cỡ chữ của chính component đó.
• `clamp(min, ưa-thích, max)` là "responsive typography 1 dòng": thay vì 3 media
  query đổi font-size, viết `clamp(2rem, 5vw, 4rem)` — chữ co giãn mượt theo
  viewport nhưng không bao giờ nhỏ hơn 2rem hay lớn hơn 4rem. Cùng tinh thần
  "1 dòng thay nhiều breakpoint" như `auto-fit/minmax` bên Grid.
─────────────────────────────────────────────────
```

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Lỗi phổ biến
> - **`em` cascades — gây bất ngờ:** Nếu font-size của parent là 20px và child cũng dùng `font-size: 1.2em`, con = 24px, cháu = 28.8px... Dùng `rem` để tránh vòng lặp này.
> - **`vh` trên mobile browser:** Mobile browser thay đổi `vh` khi scroll (do thanh toolbar). Dùng `dvh` (dynamic viewport height) thay `vh` cho mobile layouts.
> - **Contrast ratio của màu:** Đảm bảo text/background đạt tỷ lệ contrast ≥ 4.5:1 (WCAG AA). Dùng [coolors.co/contrast-checker](https://coolors.co/contrast-checker).
> - **`background-size: cover` bị crop:** Ảnh sẽ bị cắt để fill container. Dùng `object-position` (với `<img>`) hoặc `background-position` để kiểm soát phần nào được giữ lại.

## 5. Câu hỏi phỏng vấn thường gặp

1. **Q:** Sự khác nhau giữa `em` và `rem`?
   **A:** `em` — relative đến font-size của **element cha** (cascades, có thể gây snowball effect). `rem` — relative đến font-size của `<html>` (mặc định 16px, nhất quán). Khuyến nghị: dùng `rem` cho font-size, `em` cho padding/margin trong component (tự scale theo font component đó).

2. **Q:** Tại sao dùng HSL thay vì Hex?
   **A:** HSL trực quan hơn khi làm việc với màu sắc: dễ tạo color variants (chỉ đổi Lightness%), dễ thấy màu sẽ trông như thế nào khi nghe tên hue (120° = xanh lá, 240° = xanh dương). Hex không có thông tin trực quan.

3. **Q:** Làm sao tạo ảnh cover background không bị méo?
   **A:** Cho `<img>`: `object-fit: cover; width: 100%; height: 100%`. Cho background image: `background-size: cover; background-position: center`. Cả hai giữ aspect ratio và fill container, cắt phần thừa.

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Tạo color palette cho theme sáng/tối dùng CSS Custom Properties. Define 10 màu (primary, secondary, success, danger, text, bg, muted... cho 2 theme). Switch theme bằng class `.dark` trên `<body>`.
- [ ] **Bài 2:** Tạo card với badge overlay: badge dùng `background: linear-gradient(...)`, card image dùng `background-size: cover`, card title dùng fluid typography `font-size: clamp(1rem, 2.5vw, 1.5rem)`.

## 7. Liên kết

- Note liên quan: [[05-CSS-Box-Model]], [[06-CSS-Properties-thong-dung]], [[10-CSS-Responsive-Media-Query]]
- [MDN: CSS units](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units)
- [MDN: CSS colors](https://developer.mozilla.org/en-US/docs/Web/CSS/color)
- [Coolors Contrast Checker](https://coolors.co/contrast-checker)
