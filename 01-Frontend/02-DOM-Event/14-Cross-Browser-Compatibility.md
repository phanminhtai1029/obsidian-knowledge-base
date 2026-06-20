---
title: "Cross-Browser Compatibility"
section: 02-DOM-Event
tags: [dom, cross-browser, compatibility, caniuse, feature-detection, polyfill, fresher, frontend]
related:
  - "[[09-Fetch-API]]"
  - "[[13-DOM-Performance]]"
difficulty: ⭐⭐
estimated_time: 25m
source: [JS-DOM raw transcript]
---

# Cross-Browser Compatibility

> [!summary] TL;DR
> **Cross-browser compatibility** = đảm bảo web chạy mượt trên **nhiều trình duyệt và nhiều phiên bản** khác nhau (Chrome, Firefox, Safari, Edge...). Mỗi trình duyệt có "quirk" và mức hỗ trợ tính năng khác nhau. Hai công cụ chính: (1) tra **Can I Use** xem một tính năng (Flexbox, fetch...) được trình duyệt nào hỗ trợ; (2) **code phòng thủ (feature detection)** — kiểm tra tính năng có tồn tại trước khi dùng, nếu không thì báo lỗi/fallback. Mục tiêu: trải nghiệm **nhất quán** cho mọi người dùng (may thay, không còn phải lo Internet Explorer).

---

## 1. Vấn đề

Cùng một đoạn code có thể chạy khác nhau (hoặc lỗi) trên các trình duyệt/phiên bản khác nhau, vì mỗi trình duyệt hỗ trợ tính năng ở mức độ khác nhau. Ẩn dụ: như tổ chức tiệc cho khách có khẩu vị khác nhau — phải "chiều" được mọi trình duyệt.

---

## 2. Tra cứu: Can I Use

[caniuse.com](https://caniuse.com) cho biết một tính năng (CSS/JS API) được **trình duyệt nào & phiên bản nào** hỗ trợ, kèm thống kê độ phủ. Ví dụ tra "Flexbox", "fetch", "Grid" → thấy ngay bảng xanh/đỏ theo trình duyệt.

> Dùng trước khi áp dụng một API mới: nếu độ phủ thấp, cần fallback hoặc polyfill.

---

## 3. Code phòng thủ: Feature Detection

Kiểm tra tính năng **có tồn tại** trước khi dùng — tránh lỗi vỡ trang:

```javascript
// ✅ Feature detection: hỏi "trình duyệt có hỗ trợ không?"
if ("fetch" in window) {
  fetch("/api/data").then(/* ... */);
} else {
  showMessage("Trình duyệt của bạn không hỗ trợ fetch.");  // fallback
}
```

```
★ Insight ─────────────────────────────────────
• Feature detection (hỏi "có tính năng này không?") khác BROWSER SNIFFING (đoán
  theo tên/User-Agent "đây là Chrome thì..."). Sniffing dễ sai và mong manh (UA
  giả mạo được, trình duyệt mới ra). Luôn detect TÍNH NĂNG, đừng đoán TRÌNH DUYỆT.
• Polyfill = đoạn code "vá" tính năng thiếu bằng cách tự cài đặt lại API trên
  trình duyệt cũ; còn transpile (Babel) thì dịch cú pháp mới → cú pháp cũ. Hai
  thứ khác nhau: polyfill thêm API thiếu, transpile đổi cú pháp.
─────────────────────────────────────────────────
```

| Khái niệm | Là gì |
|---|---|
| **Feature detection** | Kiểm tra tính năng tồn tại (`"fetch" in window`) rồi mới dùng |
| **Browser sniffing** | Đoán theo User-Agent (❌ nên tránh — mong manh) |
| **Polyfill** | Code tự cài đặt API còn thiếu trên trình duyệt cũ |
| **Transpile** (Babel) | Dịch cú pháp JS mới → cú pháp cũ tương thích |

---

## 4. Thực hành tốt

- Tra **Can I Use** trước khi dùng API mới.
- **Feature detection** thay vì sniffing.
- Cung cấp **fallback** hoặc **polyfill** cho tính năng quan trọng.
- Test trên nhiều trình duyệt (ít nhất Chrome + Firefox + Safari).
- Không còn cần hỗ trợ **Internet Explorer** (đã ngừng) — giảm gánh nặng.

> [!tip] Tiến bộ tăng dần (Progressive Enhancement)
> Xây nền chạy được ở mọi trình duyệt trước, rồi **thêm tính năng nâng cao** cho trình duyệt hỗ trợ. Ngược với "graceful degradation" (làm bản đầy đủ rồi hạ cấp cho trình duyệt yếu).

---

## 5. Q&A phỏng vấn

> [!question] 1. Cross-browser compatibility là gì?
> Đảm bảo web chạy **nhất quán** trên nhiều trình duyệt và phiên bản khác nhau, vốn có mức hỗ trợ tính năng và "quirk" khác nhau.

> [!question] 2. Feature detection khác browser sniffing thế nào? Nên dùng cái nào?
> **Feature detection** kiểm tra *tính năng có tồn tại* (`"fetch" in window`). **Browser sniffing** đoán theo User-Agent. Nên dùng **feature detection** vì sniffing mong manh, dễ sai (UA giả được, trình duyệt mới).

> [!question] 3. Polyfill và transpile khác nhau ra sao?
> **Polyfill** tự cài đặt **API còn thiếu** trên trình duyệt cũ. **Transpile** (Babel) dịch **cú pháp** JS mới sang cú pháp cũ. Một cái thêm API, một cái đổi cú pháp.

> [!question] 4. Làm sao biết một tính năng có được trình duyệt hỗ trợ không?
> Tra **Can I Use** (caniuse.com) — xem trình duyệt/phiên bản nào hỗ trợ + độ phủ; rồi quyết định dùng trực tiếp, fallback, hay polyfill.

> [!question] 5. Progressive enhancement là gì?
> Xây nền tảng chạy ở mọi trình duyệt trước, rồi **bổ sung tính năng nâng cao** cho trình duyệt hỗ trợ — đảm bảo ai cũng dùng được phần cốt lõi.

---

## 6. Bài tập tự luyện

1. Tra Can I Use cho 3 tính năng (`fetch`, CSS Grid, `dialog`) và ghi lại độ phủ.
2. Viết feature detection cho `fetch` với fallback (báo lỗi hoặc dùng `XMLHttpRequest`).
3. Giải thích vì sao `if (isChrome)` (sniffing) là cách làm dở, viết lại bằng feature detection.
4. Tìm 1 polyfill (vd `Array.prototype.includes`) và giải thích nó "vá" gì.

---

## 7. Liên kết
- [[09-Fetch-API]] — `fetch` và feature detection
- [[13-DOM-Performance]] — tối ưu DOM
- [[00-MOC-DOM|MOC: DOM & Events]]
