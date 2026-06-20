---
title: "CSS Reset vs Normalize"
section: 01-HTML-CSS
tags: [css, reset, normalize, user-agent-styles, cross-browser, fresher, frontend]
related:
  - "[[03-CSS-Overview-va-cu-phap]]"
  - "[[07-CSS-Cascade-Specificity-Inheritance]]"
difficulty: ⭐⭐
estimated_time: 20m
source: [CSS raw transcript, Eric Meyer Reset, Normalize.css]
---

# CSS Reset vs Normalize

> [!summary] TL;DR
> Mỗi trình duyệt có **user-agent styles** (style mặc định) khác nhau → cùng một HTML hiển thị lệch nhau giữa các trình duyệt. Hai cách "thuần hóa": (1) **Reset CSS** (Eric Meyer) — **xóa sạch** mọi style mặc định, cho bạn "tờ giấy trắng" nhưng phải tự style lại tất cả; (2) **Normalize.css** — **giữ** style hợp lý, chỉ **sửa những chỗ không nhất quán** giữa trình duyệt (vd chỉ reset `h1`, các heading khác đã nhất quán thì để yên). Reset = mạnh tay, Normalize = nhẹ nhàng hiện đại. Chọn cái hợp dự án.

---

## 1. Vấn đề: user-agent styles

Trình duyệt tự áp **style mặc định** cho thẻ HTML (margin của `<body>`, cỡ chữ `<h1>`, gạch chân link...). Mỗi trình duyệt làm hơi khác → trang **lệch nhau** nếu không "thuần hóa".

---

## 2. Reset vs Normalize

| Tiêu chí | **Reset CSS** (Eric Meyer) | **Normalize.css** |
|---|---|---|
| Cách làm | **Xóa sạch** mọi style mặc định | **Giữ** style tốt, chỉ sửa chỗ không nhất quán |
| Kết quả | "Tờ giấy trắng" hoàn toàn | Khác biệt nhỏ, gần style mặc định |
| Công sức sau đó | **Nhiều** (style lại từ đầu) | **Ít** (chỉ chỉnh thêm) |
| Triết lý | Đồng nhất bằng cách bỏ hết | Đồng nhất bằng cách vá khác biệt |
| Bảo trì | Tĩnh | Phát triển liên tục |

```css
/* Reset (ý tưởng): bỏ margin/padding, box-sizing chuẩn */
* { margin: 0; padding: 0; box-sizing: border-box; }
```

```
★ Insight ─────────────────────────────────────
• Reset đảm bảo bạn KHÔNG bị bất ngờ bởi style mặc định, nhưng đánh đổi: mất luôn
  cả những mặc định hữu ích (heading đậm, list có dấu chấm) → phải dựng lại. Normalize
  giữ cái tốt, chỉ sửa cái lệch → phù hợp hầu hết dự án hiện đại.
• Hỏi mẹo: "có nên dùng reset không?" — không có đáp án tuyệt đối. Nhiều CSS
  framework (Bootstrap, Tailwind) đã NHÚNG sẵn một bản reset/normalize riêng
  (Bootstrap dùng "Reboot"). Dùng framework rồi thì thường không cần thêm reset.
─────────────────────────────────────────────────
```

---

## 3. Khi nào dùng cái nào

- **Normalize**: hầu hết dự án — muốn nền nhất quán nhưng giữ mặc định hợp lý.
- **Reset**: khi cần kiểm soát tuyệt đối từng pixel (design system riêng), chấp nhận style lại tất cả.
- **Dùng framework** (Bootstrap/Tailwind): thường đã có sẵn reset riêng, không cần thêm.

> Liên quan cross-browser: reset/normalize chính là một phần của việc đảm bảo nhất quán giữa trình duyệt — xem [[../02-DOM-Event/14-Cross-Browser-Compatibility]].

---

## 4. Q&A phỏng vấn

> [!question] 1. User-agent styles là gì? Vì sao cần reset/normalize?
> Là **style mặc định** trình duyệt tự áp cho thẻ HTML. Mỗi trình duyệt khác nhau → trang hiển thị lệch. Reset/Normalize "thuần hóa" để có nền **nhất quán** giữa các trình duyệt.

> [!question] 2. Reset CSS và Normalize.css khác nhau thế nào?
> **Reset** xóa sạch mọi style mặc định (tờ giấy trắng, phải style lại tất cả). **Normalize** giữ style hợp lý, chỉ sửa những chỗ không nhất quán giữa trình duyệt (nhẹ nhàng hơn).

> [!question] 3. Nên chọn cái nào?
> Tùy dự án: **Normalize** cho đa số (giữ mặc định tốt). **Reset** khi cần kiểm soát tuyệt đối. Nếu đã dùng framework (Bootstrap "Reboot", Tailwind preflight) thì thường không cần thêm.

---

## 5. Bài tập tự luyện

1. Dán cùng một HTML vào CodePen, bật Reset rồi bật Normalize, so sánh khác biệt hiển thị.
2. Viết một reset tối giản (`margin`, `padding`, `box-sizing`) và quan sát ảnh hưởng.
3. Mở `normalize.css`, tìm chỗ chỉ reset `h1` và giải thích vì sao các heading khác không cần.

---

## 6. Liên kết
- [[03-CSS-Overview-va-cu-phap]] — user-agent styles, cách viết CSS
- [[../02-DOM-Event/14-Cross-Browser-Compatibility]] — nhất quán giữa trình duyệt
- [[13-Bootstrap-Overview]] — framework đã nhúng reset riêng
- [[00-MOC-HTML-CSS|MOC: HTML & CSS]]
