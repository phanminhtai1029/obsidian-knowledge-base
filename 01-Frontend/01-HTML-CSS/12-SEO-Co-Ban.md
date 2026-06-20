---
title: "SEO Cơ Bản"
section: 01-HTML-CSS
tags: [seo, html, meta, og, semantic, fresher, frontend]
related:
  - "[[01-HTML-Semantic-Tags]]"
  - "[[02-HTML-Forms]]"
difficulty: ⭐⭐
estimated_time: 25m
source: [MDN, tự bổ sung]
---

# SEO Cơ Bản

> [!summary] TL;DR
> SEO (Search Engine Optimization) cho frontend tập trung vào: **Semantic HTML** (giúp crawler hiểu cấu trúc), **Meta tags** (`title`, `description`, Open Graph), **Performance** (Core Web Vitals), và **Accessibility**. Frontend developer kiểm soát on-page SEO qua HTML structure và meta tags.

## 1. Khái niệm

**SEO (Search Engine Optimization)** — tối ưu hóa website để xuất hiện cao hơn trên kết quả tìm kiếm.

**Phân loại (scope frontend):**
- **On-page SEO:** HTML structure, meta tags, content — frontend kiểm soát trực tiếp.
- **Technical SEO:** Page speed, mobile-friendly, structured data — frontend + backend.
- **Off-page SEO:** Backlinks, domain authority — ngoài tầm kiểm soát của frontend.

**Search engine crawling:**
- Googlebot đọc HTML của trang.
- Phân tích heading hierarchy (`h1` → `h2` → `h3`).
- Đọc `alt` text của hình ảnh.
- Kiểm tra internal/external links.
- Đo Core Web Vitals (LCP, FID, CLS).

## 2. Cú pháp / API

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">

  <!-- ── PRIMARY META TAGS ── -->
  <title>Học CSS Flexbox từ cơ bản đến nâng cao | FresherAI</title>
  <!-- Tối đa 60 ký tự, chứa keyword chính -->

  <meta name="description" content="Hướng dẫn học CSS Flexbox toàn diện: container properties, item properties, responsive layouts. Kèm ví dụ thực tế và bài tập.">
  <!-- 150-160 ký tự, tóm tắt hấp dẫn -->

  <meta name="robots" content="index, follow">
  <!-- index: cho phép index trang | follow: theo link -->

  <link rel="canonical" href="https://fresherai.vn/css-flexbox">
  <!-- Tránh duplicate content -->

  <!-- ── OPEN GRAPH (Facebook, LinkedIn share) ── -->
  <meta property="og:type"        content="article">
  <meta property="og:title"       content="Học CSS Flexbox từ cơ bản đến nâng cao">
  <meta property="og:description" content="Hướng dẫn học CSS Flexbox toàn diện...">
  <meta property="og:image"       content="https://fresherai.vn/img/flexbox-og.jpg">
  <meta property="og:url"         content="https://fresherai.vn/css-flexbox">
  <meta property="og:locale"      content="vi_VN">

  <!-- ── TWITTER CARD ── -->
  <meta name="twitter:card"        content="summary_large_image">
  <meta name="twitter:title"       content="Học CSS Flexbox từ cơ bản đến nâng cao">
  <meta name="twitter:description" content="Hướng dẫn học CSS Flexbox toàn diện...">
  <meta name="twitter:image"       content="https://fresherai.vn/img/flexbox-og.jpg">

  <!-- ── FAVICON ── -->
  <link rel="icon" href="/favicon.ico" sizes="any">
  <link rel="icon" href="/favicon.svg" type="image/svg+xml">
  <link rel="apple-touch-icon" href="/apple-touch-icon.png">
</head>
<body>
  <!-- Chỉ 1 thẻ h1 chứa keyword chính -->
  <h1>CSS Flexbox: Hướng dẫn từ cơ bản đến nâng cao</h1>

  <!-- Heading hierarchy đúng thứ tự -->
  <h2>Flexbox là gì?</h2>
    <h3>Flex container</h3>
    <h3>Flex item</h3>
  <h2>Cách sử dụng Flexbox</h2>
</body>
</html>
```

## 3. Ví dụ minh họa

### Ví dụ 1: Structured Data (JSON-LD) cho bài blog

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "CSS Flexbox: Hướng dẫn toàn diện",
  "description": "Học CSS Flexbox từ cơ bản...",
  "author": {
    "@type": "Person",
    "name": "Tâm Dev",
    "url": "https://fresherai.vn/author/tamdev"
  },
  "datePublished": "2025-05-01",
  "dateModified": "2025-05-01",
  "image": "https://fresherai.vn/img/flexbox-og.jpg",
  "publisher": {
    "@type": "Organization",
    "name": "FresherAI",
    "logo": {
      "@type": "ImageObject",
      "url": "https://fresherai.vn/logo.png"
    }
  }
}
</script>
```

**Output / Kết quả mong đợi:** Google hiển thị rich result (star rating, author, date) trong SERP — tăng click-through rate.

**Giải thích:** JSON-LD là định dạng structured data Google ưa thích — không can thiệp vào HTML visible, dễ maintain.

### Ví dụ 2: Image SEO best practices

```html
<!-- ❌ Sai: thiếu alt, tên file không có nghĩa -->
<img src="img001.jpg">

<!-- ✅ Đúng: alt mô tả nội dung, tên file có keyword -->
<img
  src="css-flexbox-container-properties.jpg"
  alt="Sơ đồ minh họa Flexbox container với flex-direction: row"
  width="800"
  height="450"
  loading="lazy"
>

<!-- Decorative image: alt="" để screen reader bỏ qua -->
<img src="divider.svg" alt="" role="presentation">
```

**Giải thích:**
- `alt` mô tả nội dung ảnh — Googlebot "nhìn" ảnh qua alt text.
- `width` + `height` ngăn Layout Shift (cải thiện CLS — Core Web Vitals).
- `loading="lazy"` trì hoãn load ảnh ngoài viewport — cải thiện LCP.

```
★ Insight ─────────────────────────────────────
• SEO của frontend phần lớn TRÙNG với accessibility & performance làm đúng: cùng
  một `alt`, heading hierarchy, semantic tag vừa giúp screen reader vừa giúp
  Googlebot; cùng `width/height` + lazy-load vừa cải thiện trải nghiệm vừa nâng
  Core Web Vitals. Làm tốt a11y + perf ≈ làm tốt on-page SEO "miễn phí".
• Phân biệt vai trò khi phỏng vấn: `<title>` LÀ ranking factor trực tiếp;
  `meta description` thì KHÔNG — nó chỉ ảnh hưởng CTR (đoạn snippet hấp dẫn →
  nhiều click). Nhồi keyword vào description vô ích; viết title chuẩn mới đáng.
─────────────────────────────────────────────────
```

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Lỗi phổ biến
> - **Nhiều hơn 1 thẻ `<h1>`:** Mỗi trang chỉ nên có 1 `<h1>` chứa keyword chính.
> - **Bỏ qua `lang` attribute:** `<html lang="vi">` giúp Google hiểu ngôn ngữ, tránh nhầm lẫn với Vietnamese.
> - **`title` quá dài/ngắn:** < 30 ký tự quá ngắn, > 60 ký tự bị truncate trong SERP.
> - **Alt text nhồi keyword:** `alt="css flexbox css layout css grid css tutorial"` bị Google penalize. Mô tả tự nhiên.

## 5. Câu hỏi phỏng vấn thường gặp

1. **Q:** Frontend developer có thể làm gì để cải thiện SEO?
   **A:** Semantic HTML (heading hierarchy, landmark tags), meta tags đúng (title, description, OG), alt text cho ảnh, Performance (lazy loading, optimize assets), structured data (JSON-LD), canonical URL, mobile-friendly responsive design.

2. **Q:** `<title>` và `<meta name="description">` khác nhau thế nào về SEO?
   **A:** `<title>` là yếu tố ranking trực tiếp — Google dùng để hiểu topic của trang, hiển thị trong tab browser và SERP title. `<meta name="description">` **không phải** ranking factor trực tiếp nhưng ảnh hưởng click-through rate (CTR) vì hiển thị làm snippet trong SERP.

3. **Q:** Core Web Vitals là gì và frontend ảnh hưởng thế nào?
   **A:** **LCP** (Largest Contentful Paint): optimize ảnh hero, preload font. **FID/INP** (Interaction to Next Paint): giảm JavaScript blocking. **CLS** (Cumulative Layout Shift): luôn đặt `width/height` cho ảnh, tránh inject content động trên trang.

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Audit SEO trang cá nhân (hoặc trang bất kỳ) bằng [Lighthouse](https://developers.google.com/web/tools/lighthouse) (F12 > Lighthouse tab). Đọc báo cáo SEO và Accessibility, implement ít nhất 3 gợi ý.
- [ ] **Bài 2:** Thêm đầy đủ meta tags (OG + Twitter Card) vào một trang HTML demo. Test kết quả trên [Facebook Sharing Debugger](https://developers.facebook.com/tools/debug/) và Twitter Card Validator.

## 7. Liên kết

- Note liên quan: [[01-HTML-Semantic-Tags]], [[10-CSS-Responsive-Media-Query]]
- [Google: Search Central Documentation](https://developers.google.com/search/docs)
- [MDN: SEO](https://developer.mozilla.org/en-US/docs/Glossary/SEO)
- [Schema.org](https://schema.org/) — structured data reference
