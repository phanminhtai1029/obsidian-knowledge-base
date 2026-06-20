---
title: "CSS Workflow: DevTools, Organizing, Validation"
section: 01-HTML-CSS
tags: [css, devtools, workflow, linting, validation, caniuse, code-organization, fresher, frontend]
related:
  - "[[03-CSS-Overview-va-cu-phap]]"
  - "[[17-CSS-Reset-Normalize]]"
difficulty: ⭐⭐
estimated_time: 25m
source: [CSS raw transcript]
---

# CSS Workflow: DevTools, Organizing, Validation

> [!summary] TL;DR
> Bốn kỹ năng "vận hành" CSS hằng ngày: (1) **DevTools** (F12 / chuột phải → Inspect) — xem & **sửa CSS trực tiếp (live)** trong trình duyệt để thử nghiệm, không lưu vào file; (2) **Tổ chức code** — comment nhiều, naming convention nhất quán, format nhất quán, gom style cùng loại, tách file khi lớn; (3) **Can I Use** — tra một thuộc tính CSS được trình duyệt nào hỗ trợ; (4) **Validation/Linting** — kiểm tra CSS không lỗi cú pháp (W3C validator, CSS Lint) và làm đẹp/định dạng (beautifier, linter trong editor).

---

## 1. DevTools — sân chơi CSS

Mở: **chuột phải → Inspect** hoặc **F12**. Có sẵn trong mọi trình duyệt (không cần cài).

| Làm được gì | Lợi ích |
|---|---|
| Xem HTML + CSS đang áp | Hiểu vì sao phần tử trông thế |
| **Sửa CSS live** (đổi padding, màu...) | Thử nghiệm tức thì, không đụng file gốc |
| Bật/tắt từng property | Cô lập xem property nào gây hiệu ứng |
| Xem **computed width/box** | Debug box model |
| Hover node → highlight trên trang | Định vị nhanh phần tử |

> [!tip] Live editing không lưu — refresh là mất
> Mọi thay đổi trong DevTools chỉ ở **bộ nhớ trình duyệt**; refresh trang là trở về bản gốc. Đây là chỗ **thử** trước khi chép vào file CSS thật.

---

## 2. Tổ chức code CSS

CSS hợp lệ vẫn có thể "hỗn loạn". Giữ sạch bằng:

| Nguyên tắc | Chi tiết |
|---|---|
| **Comment nhiều** | Chia khối + mô tả; giúp người khác (và chính bạn sau này) |
| **Naming nhất quán** | Chọn dash/underscore/camelCase — quan trọng là **nhất quán** |
| **Format nhất quán** | Thụt lề/khoảng trắng đồng đều → dễ đọc |
| **Gom style cùng loại** | Heading một chỗ, header components một chỗ |
| **Tách file khi lớn** | `typography.css`, `layout.css`... dễ bảo trì |

```
★ Insight ─────────────────────────────────────
• "CSS chạy đúng" ≠ "CSS tốt". Trình duyệt không quan tâm bạn format đẹp hay xấu,
  nhưng CON NGƯỜI bảo trì thì có. Phần lớn chi phí một codebase là ĐỌC LẠI, không
  phải viết mới — nên nhất quán (naming, format) là khoản đầu tư rẻ mà lãi lớn.
─────────────────────────────────────────────────
```

---

## 3. Can I Use — kiểm tra hỗ trợ

[caniuse.com](https://caniuse.com): gõ tên thuộc tính (vd `flexbox`, `animation`) → bảng hỗ trợ theo trình duyệt + % độ phủ.

> Trạng thái spec (CR = Candidate Recommendation...) không quan trọng bằng **% usage thực tế**. Flexbox ~95%+ → dùng thoải mái dù vẫn "CR". (Liên quan: [[../02-DOM-Event/14-Cross-Browser-Compatibility]].)

---

## 4. Validation & Linting

| Công cụ | Vai trò |
|---|---|
| **W3C CSS Validator** | Kiểm tra **lỗi cú pháp / code không hợp lệ** (dán code/URL/file) |
| **CSS Lint / linter trong editor** | "Code cleaner" — bắt lỗi, cảnh báo, ngay khi gõ |
| **CSS Beautifier / formatter** | Tự **định dạng** lại (thụt lề, khoảng trắng nhất quán) |

> [!note] "Lint" nghĩa là gì?
> Không phải "bụi vải trong túi" — **linter** là công cụ **kiểm tra & dọn code** (bắt lỗi, áp quy tắc style). Tốt nhất là tích hợp **ngay trong editor** (VS Code) thay vì web tool, để bắt lỗi lúc viết.

---

## 5. Q&A phỏng vấn

> [!question] 1. DevTools dùng để làm gì khi học/làm CSS?
> Xem HTML+CSS đang áp, **sửa CSS live** để thử nghiệm (không lưu file), bật/tắt property, xem computed box, định vị phần tử. Mở bằng F12 / chuột phải → Inspect; có sẵn mọi trình duyệt.

> [!question] 2. Thay đổi CSS trong DevTools có lưu lại không?
> Không — chỉ ở bộ nhớ trình duyệt, **refresh là mất**. Là nơi thử trước khi chép vào file thật.

> [!question] 3. Vài nguyên tắc tổ chức CSS sạch?
> Comment nhiều, **naming nhất quán**, **format nhất quán**, gom style cùng loại, tách file khi dự án lớn. Mục tiêu: dễ đọc & bảo trì.

> [!question] 4. Linter và validator khác gì?
> **Validator** (W3C) kiểm tra **lỗi cú pháp/hợp lệ**. **Linter** rộng hơn: bắt lỗi + cảnh báo + áp quy tắc style, lý tưởng tích hợp trong editor. **Beautifier** lo định dạng cho đẹp.

> [!question] 5. Làm sao biết một thuộc tính CSS có dùng được rộng rãi?
> Tra **Can I Use** — xem bảng hỗ trợ theo trình duyệt và **% usage**; ưu tiên độ phủ thực tế hơn trạng thái spec.

---

## 6. Bài tập tự luyện

1. Mở DevTools trên một trang bất kỳ, đổi `background-color` và `padding` của một phần tử live, rồi refresh để thấy reset.
2. Lấy một file CSS "lộn xộn", chạy qua beautifier và sắp xếp lại theo nhóm (typography/layout/components).
3. Tra Can I Use cho `gap` (trong flexbox) và `:has()`; ghi lại % hỗ trợ.
4. Cài một CSS linter trong VS Code và sửa các cảnh báo trên một file CSS.

---

## 7. Liên kết
- [[03-CSS-Overview-va-cu-phap]] — cách viết CSS
- [[17-CSS-Reset-Normalize]] — nhất quán cross-browser
- [[../02-DOM-Event/14-Cross-Browser-Compatibility]] — Can I Use, feature detection
- [[00-MOC-HTML-CSS|MOC: HTML & CSS]]
