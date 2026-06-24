---
title: "Accessibility (a11y) trong DOM"
section: 02-DOM-Event
tags: [dom, accessibility, a11y, aria, screen-reader, semantic-html, fresher, frontend]
related:
  - "[[01-DOM-Tree-va-Node-Types]]"
  - "[[05-Creating-Adding-Managing-Nodes]]"
  - "[[../01-HTML-CSS/01-HTML-Semantic-Tags]]"
difficulty: ⭐⭐⭐
estimated_time: 35m
source: [JS-DOM raw transcript, WAI-ARIA]
---

# Accessibility (a11y) trong DOM

> [!summary] TL;DR
> **Accessibility (a11y)** = làm cho web dùng được với **mọi người, kể cả người khuyết tật** (khiếm thị, không dùng được chuột...). Ẩn dụ: thay vì chỉ có cầu thang, hãy làm thêm **đường dốc & thang máy**. Khi sinh DOM bằng JS phải nhớ: `alt` cho ảnh, `label` cho input, **semantic HTML** (`<header>` thay vì `<div>`), **keyboard accessible** (`tabindex` + xử lý phím Enter/Space), **ARIA roles** cho widget phức tạp, và **đủ tương phản màu**. Điểm JS quan trọng nhất: **screen reader chỉ đọc phần đang focus** — nếu JS thay đổi nội dung chỗ khác, người dùng screen reader **không biết** trừ khi bạn dùng **`aria-live`**.

---

## 1. Vì sao a11y quan trọng

Screen reader (trình đọc màn hình) giúp người khiếm thị "nghe" trang web, thường **chỉ dùng bàn phím**. Nó đọc to **phần tử đang focus**. Web không a11y = bỏ rơi nhóm người dùng này (và nhiều nơi còn là yêu cầu pháp lý).

---

## 2. Checklist khi sinh HTML bằng JavaScript

| Việc cần làm | Vì sao |
|---|---|
| Thêm `alt` khi tạo `<img>` | Screen reader đọc mô tả ảnh |
| Thêm `<label>` cho input | Người dùng biết ô nhập là gì |
| Dùng **semantic HTML** (`<header>`, `<main>`, `<nav>`) | Đừng tạo `<div>` đóng vai header |
| **Keyboard accessible** | Gán `tabindex` + xử lý phím Enter/Space để kích hoạt |
| **ARIA roles/properties** | Bổ nghĩa cho widget phức tạp không có thẻ HTML tương đương |
| **Tương phản màu** đủ | Người thị lực kém đọc được |

```javascript
// ❌ Không a11y
const img = document.createElement("img");
img.src = "puppy.jpg";

// ✅ Có a11y
const img = document.createElement("img");
img.src = "puppy.jpg";
img.alt = "Chú cún golden đang chạy trên cỏ";   // screen reader đọc được
```

---

## 3. Điểm cốt lõi cho JS: `aria-live` cho nội dung động

**Vấn đề:** screen reader chỉ đọc **phần đang focus**. Người sáng mắt thấy ngay thông báo "Gửi thành công!" hiện ở góc, nhưng người dùng screen reader **không hề biết** vì focus của họ vẫn ở nút Submit.

**Giải pháp:** đánh dấu vùng nội dung động bằng **`role`** + **`aria-live`** để screen reader tự đọc khi nội dung đổi:

```html
<!-- Vùng thông báo: screen reader sẽ đọc khi nội dung thay đổi -->
<div role="status" aria-live="polite" id="msg"></div>
```
```javascript
document.getElementById("msg").textContent = "Cảm ơn bạn đã gửi!"; // tự được đọc lên
```

| `aria-live` | Khi nào đọc | Dùng cho |
|---|---|---|
| `polite` | Đọc khi người dùng **rảnh** (không cắt ngang) | Thông báo thường: "Đã lưu", "Có tin mới" |
| `assertive` | Đọc **ngay lập tức** (cắt ngang) | Cảnh báo khẩn: lỗi nghiêm trọng |
| `off` | Không tự đọc | (mặc định cho phần tử thường) |

```
★ Insight ─────────────────────────────────────
• Bẫy lớn nhất của SPA/JS-heavy app với a11y: bạn cập nhật DOM bằng JS ở MỘT chỗ,
  màn hình đổi, nhưng screen reader "đứng yên" ở focus cũ → người mù không biết gì
  vừa xảy ra. aria-live là cây cầu báo cho họ. Không có nó, mọi cập nhật động đều
  "vô hình" với screen reader.
• Semantic HTML là a11y "miễn phí": dùng <button> thay <div onclick> thì tự được
  focus bằng Tab, tự kích hoạt bằng Enter/Space, screen reader tự biết "đây là
  nút". Tự chế bằng <div> thì phải thêm tabindex + role + key handler để bù lại —
  nhiều việc hơn mà dễ sót.
─────────────────────────────────────────────────
```

---

## 4. Phần tử tương tác tự tạo phải focus được

Nếu buộc phải dùng `<div>`/`<span>` làm phần tử tương tác (nên tránh), phải bù:

```javascript
const widget = document.createElement("div");
widget.setAttribute("role", "button");   // báo "đây là nút"
widget.setAttribute("tabindex", "0");     // cho phép Tab tới (focus được)
widget.addEventListener("keydown", (e) => {
  if (e.key === "Enter" || e.key === " ") activate();  // kích hoạt bằng phím
});
```

> [!tip] Ưu tiên thẻ gốc (native) hơn ARIA
> Quy tắc số 1 của ARIA: *"No ARIA is better than bad ARIA"*. Dùng `<button>`, `<a>`, `<input>` thật trước; chỉ dùng `tabindex`/`role` khi không có thẻ tương đương. Thẻ native đã a11y sẵn.

---

## 5. So sánh: `textContent` vs `innerHTML` (góc a11y & bảo mật)

| | `textContent` | `innerHTML` |
|---|---|---|
| Chèn được HTML | Không (chỉ text) | Có |
| Rủi ro XSS | An toàn | **Nguy hiểm** với dữ liệu người dùng |
| a11y | Đơn giản, an toàn | Phải tự đảm bảo cấu trúc đúng |

> Khi chỉ cần thêm chữ (như thông báo) → dùng `textContent` (an toàn). Xem thêm [[13-DOM-Performance]].

---

## 6. Q&A phỏng vấn

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "a11y là gì, khi sinh DOM bằng JS cần lưu ý gì?"
> *"Accessibility, viết tắt a11y, là làm cho web dùng được với mọi người, kể cả người khuyết tật, ví dụ người khiếm thị dùng screen reader hay người không dùng được chuột. Nó quan trọng vì tính bao trùm, cải thiện trải nghiệm cho tất cả, tốt cho SEO và nhiều nơi còn là yêu cầu pháp lý. Khi sinh DOM bằng JS em chú ý: ảnh có alt, input có label, dùng semantic HTML như header nav main thay cho div, đảm bảo thao tác được bằng bàn phím với tabindex và xử lý phím Enter Space, dùng ARIA role cho widget phức tạp, và đủ tương phản màu. Điểm JS quan trọng nhất là screen reader chỉ đọc phần đang focus, nên khi JS cập nhật nội dung ở chỗ khác, ví dụ thông báo hay kết quả tìm kiếm, em phải đặt vùng đó là aria-live thì screen reader mới đọc tự động."*

> [!note] 🧠 Mẹo nhớ
> **a11y = web cho MỌI người (cả người khuyết tật).** Checklist khi sinh DOM: **alt, label, semantic HTML, bàn phím (tabindex), ARIA, tương phản.** Nội dung JS đổi động → **`aria-live`** mới được screen reader đọc.

> [!question] 1. Accessibility (a11y) là gì và vì sao quan trọng?
> Là làm web dùng được với mọi người **kể cả người khuyết tật** (khiếm thị dùng screen reader, người không dùng chuột...). Quan trọng vì tính bao trùm, trải nghiệm tốt hơn cho tất cả, SEO, và thường là yêu cầu pháp lý.

> [!question] 2. Khi sinh DOM bằng JS cần lưu ý gì cho a11y?
> `alt` cho ảnh, `label` cho input, **semantic HTML**, keyboard accessible (`tabindex` + Enter/Space), **ARIA roles** cho widget phức tạp, đủ tương phản màu.

> [!question] 3. Vì sao screen reader không "thấy" nội dung JS cập nhật động? Khắc phục?
> Vì screen reader chỉ đọc **phần đang focus**; cập nhật ở chỗ khác nằm ngoài focus. Khắc phục bằng **`aria-live`** (`polite`/`assertive`) + `role` trên vùng động để được đọc tự động.

> [!question] 4. `aria-live="polite"` khác `assertive` thế nào?
> `polite` đợi người dùng rảnh mới đọc (không cắt ngang) — cho thông báo thường. `assertive` đọc ngay lập tức (cắt ngang) — cho cảnh báo khẩn.

> [!question] 5. Vì sao nên ưu tiên `<button>` thay vì `<div onclick>`?
> `<button>` tự focus được bằng Tab, tự kích hoạt bằng Enter/Space, screen reader hiểu là nút. `<div>` phải thêm `tabindex` + `role="button"` + xử lý phím để bù — nhiều việc, dễ sót.

---

## 7. Bài tập tự luyện

1. Audit một form: thêm `alt` cho ảnh, `label` cho input, đổi `<div>` header → `<header>`/`<main>`.
2. Thêm `<div role="status" aria-live="polite">` và cập nhật `textContent` khi submit; kiểm tra bằng screen reader (NVDA/VoiceOver).
3. Tạo "nút" bằng `<div>` rồi bổ sung `tabindex`, `role`, xử lý Enter/Space để dùng được bằng bàn phím.
4. So sánh trải nghiệm Tab qua form trước/sau khi dùng semantic HTML.

---

## 8. Liên kết
- [[../01-HTML-CSS/01-HTML-Semantic-Tags]] — semantic HTML là nền của a11y
- [[05-Creating-Adding-Managing-Nodes]] — sinh phần tử có a11y
- [[13-DOM-Performance]] — `textContent` vs `innerHTML`
- [[00-MOC-DOM|MOC: DOM & Events]]
