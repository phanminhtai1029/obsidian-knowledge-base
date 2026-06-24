---
title: "HTML Semantic Tags"
section: 01-HTML-CSS
tags: [html, semantic, accessibility, seo, fresher, frontend]
related:
  - "[[02-HTML-Forms]]"
  - "[[12-SEO-Co-Ban]]"
difficulty: ⭐⭐
estimated_time: 30m
source: [MDN]
---

# HTML Semantic Tags

> [!summary] TL;DR
> Semantic HTML dùng các thẻ có **ý nghĩa rõ ràng** (như `<header>`, `<nav>`, `<article>`) thay cho `<div>` chung chung, giúp trình duyệt, search engine, và screen reader hiểu cấu trúc trang. Semantic tags cải thiện SEO và accessibility đồng thời.

> [!tip] 🎯 Hiểu trong 30 giây
> **Semantic HTML = dùng thẻ "gọi đúng tên" thay vì `<div>` vô danh.** Thay vì bọc mọi thứ trong `<div>`, dùng `<header>` cho đầu trang, `<nav>` cho thanh điều hướng, `<main>` cho nội dung chính, `<article>` cho một bài, `<footer>` cho chân trang.
> - Về hiển thị thì `<div>` và `<header>` *trông y hệt*, nhưng `<header>` **mang ý nghĩa** để máy hiểu: **Google** (SEO) hiểu đâu là nội dung chính, **screen reader** (đọc cho người khiếm thị) biết "đây là vùng điều hướng" để nhảy tới. Ví von: thay vì đánh số phòng bằng "phòng 1, 2, 3" vô nghĩa, ta gắn biển "Bếp", "Phòng ngủ" → ai cũng định hướng được.
> - Lợi ích kép: **SEO tốt hơn + dễ truy cập hơn (a11y)** mà không tốn thêm công.

## 1. Khái niệm

**Semantic HTML (HTML ngữ nghĩa)** là cách dùng các thẻ HTML mô tả **ý nghĩa của nội dung**, không chỉ cách nó trông.

- **Non-semantic:** `<div>`, `<span>` — không nói gì về nội dung bên trong.
- **Semantic:** `<article>`, `<nav>`, `<header>` — mô tả rõ vai trò nội dung.

**Tại sao cần biết:**
- **SEO:** Search engine (Google) dùng semantic tags để hiểu hierarchy của trang.
- **Accessibility:** Screen reader (NVDA, JAWS) dùng landmarks để điều hướng.
- **Maintainability:** Code dễ đọc, dễ bảo trì hơn mớ `<div>` lồng nhau.

## 2. Cú pháp / API

```html
<!-- Cấu trúc trang chuẩn với semantic tags -->
<body>
  <header>          <!-- Tiêu đề trang / logo / nav chính -->
    <nav>           <!-- Menu điều hướng -->
      <ul>
        <li><a href="/">Trang chủ</a></li>
      </ul>
    </nav>
  </header>

  <main>            <!-- Nội dung chính — chỉ 1 thẻ main/trang -->
    <article>       <!-- Nội dung độc lập (bài viết, post) -->
      <section>     <!-- Nhóm nội dung liên quan trong article -->
        <h1>Tiêu đề bài</h1>
        <p>Nội dung...</p>
      </section>
      <aside>       <!-- Nội dung phụ (sidebar, liên kết liên quan) -->
        <p>Xem thêm...</p>
      </aside>
    </article>
  </main>

  <footer>          <!-- Chân trang -->
    <p>&copy; 2025 MyApp</p>
  </footer>
</body>
```

**Bảng semantic tags thông dụng:**

| Tag | Vai trò |
|-----|---------|
| `<header>` | Phần đầu trang hoặc section |
| `<nav>` | Khối điều hướng |
| `<main>` | Nội dung chính (duy nhất/trang) |
| `<article>` | Nội dung độc lập, tự hoàn chỉnh |
| `<section>` | Nhóm nội dung có heading liên quan |
| `<aside>` | Nội dung phụ / sidebar |
| `<footer>` | Phần chân trang hoặc section |
| `<figure>` | Hình ảnh, diagram kèm caption |
| `<figcaption>` | Chú thích cho `<figure>` |
| `<time>` | Ngày/giờ có thể đọc được bởi máy |
| `<mark>` | Văn bản được highlight |
| `<details>` / `<summary>` | Accordion mặc định của browser |

```
★ Insight ─────────────────────────────────────
• Quy tắc chọn nhanh: nội dung "đem đi nơi khác vẫn hiểu trọn vẹn" (bài blog, 1
  sản phẩm, 1 comment) → <article>; nội dung "chỉ là 1 mảng có heading của trang
  này" → <section>; còn lại chỉ để gom cho layout/CSS → <div>. Phần lớn tranh cãi
  article-vs-section tan biến khi hỏi "tách ra độc lập có đứng vững không?".
• Mỗi semantic tag tạo một "landmark" trong accessibility tree — screen reader có
  phím tắt nhảy thẳng tới <nav>, <main>, <header>. Đây chính là lý do semantic =
  SEO + a11y "miễn phí": bạn không thêm code, chỉ chọn đúng thẻ. Xem [[11-Accessibility-a11y]].
─────────────────────────────────────────────────
```

## 3. Ví dụ minh họa

### Ví dụ 1: Blog post layout đơn giản

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>Blog của tôi</title>
</head>
<body>
  <header>
    <h1>Blog Lập Trình</h1>
    <nav aria-label="Menu chính">
      <a href="/">Trang chủ</a>
      <a href="/about">Giới thiệu</a>
    </nav>
  </header>

  <main>
    <article>
      <header>          <!-- header lồng bên trong article — hợp lệ -->
        <h2>JavaScript là gì?</h2>
        <time datetime="2025-05-01">01 tháng 5, 2025</time>
      </header>

      <section>
        <h3>Giới thiệu</h3>
        <p>JavaScript là ngôn ngữ lập trình phía client...</p>
      </section>

      <figure>
        <img src="js-logo.png" alt="Logo JavaScript">
        <figcaption>Logo chính thức của JavaScript</figcaption>
      </figure>

      <aside>
        <h3>Bài liên quan</h3>
        <ul>
          <li><a href="/css">CSS cơ bản</a></li>
        </ul>
      </aside>
    </article>
  </main>

  <footer>
    <p>Liên hệ: <a href="mailto:dev@example.com">dev@example.com</a></p>
  </footer>
</body>
</html>
```

**Kết quả mong đợi:** Trang render bình thường. Screen reader nhận diện được `<nav>` là navigation landmark, `<main>` là vùng nội dung chính.

**Giải thích:**
- `<header>` dùng được cả ở cấp trang lẫn trong `<article>`.
- `<time datetime="2025-05-01">` giúp máy đọc được ngày theo chuẩn ISO.
- `<figure>` + `<figcaption>` liên kết hình ảnh với chú thích của nó.

### Ví dụ 2: Accordion dùng `<details>` và `<summary>` (không cần JS)

```html
<section>
  <h2>Câu hỏi thường gặp</h2>

  <details>
    <summary>JavaScript và TypeScript khác nhau thế nào?</summary>
    <p>TypeScript là superset của JavaScript, bổ sung kiểu dữ liệu tĩnh.
       TypeScript được compile sang JavaScript trước khi chạy.</p>
  </details>

  <details open>  <!-- "open" = mở sẵn khi tải trang -->
    <summary>React là gì?</summary>
    <p>React là thư viện UI của Facebook, dùng Virtual DOM...</p>
  </details>
</section>
```

**Output / Kết quả mong đợi:** Browser hiển thị accordion collapse/expand mà không cần một dòng JavaScript nào.

**Giải thích:** `<details>` là interactive disclosure widget built-in của HTML5. Thuộc tính `open` làm cho nó mở sẵn.

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Lỗi phổ biến
> - **Dùng `<section>` không có heading:** `<section>` phải luôn có `<h2>` - `<h6>`. Nếu không có heading, dùng `<div>`.
> - **Lồng `<main>` nhiều lần:** Chỉ được có **1 thẻ `<main>`** trong toàn bộ trang.
> - **Dùng `<article>` cho mọi thứ:** `<article>` chỉ dành cho nội dung **tự hoàn chỉnh** (có thể copy ra trang khác vẫn hiểu được).
> - **Bỏ qua `lang` attribute:** `<html lang="vi">` bắt buộc cho accessibility và SEO.

## 5. Câu hỏi phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Semantic HTML là gì, vì sao nên dùng thay `<div>`?"
> *"Semantic HTML là dùng các thẻ có ý nghĩa rõ ràng như header, nav, main, article, footer thay vì div chung chung. Về mặt hiển thị chúng giống div, nhưng chúng mang ngữ nghĩa để máy hiểu cấu trúc trang. Google đọc semantic tag để biết đâu là nội dung chính nên tốt cho SEO; screen reader dựa vào đó để người khiếm thị nhảy nhanh tới vùng điều hướng hay nội dung chính, tốt cho accessibility. Ngoài ra code cũng dễ đọc và bảo trì hơn. Em vẫn dùng div khi chỉ cần một container thuần để layout hay styling, còn khi khối nội dung có ý nghĩa thì ưu tiên thẻ semantic."*

> [!note] 🧠 Mẹo nhớ
> **Semantic = thẻ "gọi đúng tên" (`<header>`/`<nav>`/`<main>`/`<article>`) thay `<div>` vô danh** → máy hiểu → **SEO + a11y tốt hơn.** `<div>` chỉ để gom layout thuần.

1. **Q:** Sự khác nhau giữa `<section>` và `<div>` là gì?
   **A:** `<section>` là semantic — có ý nghĩa "nhóm nội dung liên quan có heading". `<div>` là container thuần túy không có semantic. Dùng `<section>` khi có heading kèm theo, `<div>` khi chỉ cần nhóm cho layout/styling.

2. **Q:** Khi nào dùng `<article>` thay vì `<section>`?
   **A:** `<article>` khi nội dung **độc lập và tự hoàn chỉnh** (bài blog, tin tức, comment). `<section>` khi nội dung là **một phần của trang** và cần heading để phân nhóm.

3. **Q:** Tại sao semantic HTML quan trọng với SEO?
   **A:** Search engine dùng semantic tags để xác định hierarchy (`<h1>` là main topic), tìm navigation (`<nav>`), và hiểu nội dung chính (`<main>`/`<article>`). Trang có semantic markup tốt thường được rank cao hơn.

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Viết lại cấu trúc trang web cá nhân (portfolio) chỉ dùng semantic tags — không `<div>` nào cả. Trang cần có: header với nav, main với ít nhất 2 section, aside, footer.
- [ ] **Bài 2:** Mở DevTools trên một trang web lớn (vnexpress.net), dùng Accessibility tree trong Elements panel để xem browser nhận diện các landmark như thế nào.
- [ ] **Bài 3:** Dùng `<details>/<summary>` để tạo FAQ accordion 3 câu hỏi mà không dùng JavaScript.

## 7. Liên kết

- Note liên quan: [[02-HTML-Forms]], [[12-SEO-Co-Ban]], [[04-CSS-Selectors]]
- [MDN: HTML elements reference](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)
- [MDN: Semantic HTML](https://developer.mozilla.org/en-US/docs/Glossary/Semantics#semantics_in_html)
- [WHATWG: HTML Living Standard](https://html.spec.whatwg.org/)
