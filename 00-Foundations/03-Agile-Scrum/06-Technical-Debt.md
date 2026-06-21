---
title: "Technical Debt — Nợ kỹ thuật"
section: 00-Foundations/03-Agile-Scrum
tags: [scrum, technical-debt, refactoring, quality, foundations, fresher]
related:
  - "[[05-Product-Backlog-Refinement]]"
  - "[[03-Scrum-Artifacts-va-DoD]]"
difficulty: ⭐⭐⭐
estimated_time: 25m
source: ["Agile Practitioner with Scrum", Ward Cunningham]
---

# Technical Debt — Nợ kỹ thuật

> [!summary] TL;DR
> **Technical debt (nợ kỹ thuật)** = công việc phát sinh thêm trong tương lai do các quyết định/hoàn cảnh ở hiện tại (code vội, framework cũ, tìm ra cách tốt hơn, đổi yêu cầu, bug…). Giống nợ tài chính: để lâu sinh "lãi" — tính năng xây trên phần nợ càng khó bảo trì. Không thể **xóa hoàn toàn**, chỉ quản lý: **làm cho nó hiện rõ** với stakeholder, **xử lý sớm**, và **trộn** việc trả nợ với việc tạo giá trị nghiệp vụ (đừng dành cả Sprint chỉ trả nợ).

---

## 1. Technical debt là gì?

Ẩn dụ "nợ" (Ward Cunningham): chọn giải pháp nhanh-nhưng-không-tối-ưu hôm nay = "vay" thời gian; sau này phải "trả" bằng công sửa/viết lại, kèm "lãi" (càng để lâu càng đắt).

```mermaid
flowchart LR
    A["Quyết định nhanh hôm nay<br/>(vội, deadline, chưa biết)"] --> B["nợ kỹ thuật<br/>(code khó sửa)"]
    B --> C["để lâu = 'lãi'<br/>(sửa càng tốn kém)"]
```

```
★ Insight ─────────────────────────────────────
• Nợ kỹ thuật không phải lúc nào cũng xấu: đôi khi "vay" có chủ đích là khôn ngoan
  (ra mắt sớm để chiếm thị trường, rồi trả nợ sau). Cái xấu là nợ VÔ HÌNH & KHÔNG
  TRẢ — tích lại tới mức mọi thay đổi đều rủi ro. Quản lý nợ ≠ không bao giờ vay.
─────────────────────────────────────────────────
```

---

## 2. Các nguyên nhân (ví dụ) — hay được hỏi

| Nguyên nhân | Ví dụ |
|-------------|-------|
| **Cần refactor/rewrite** | Một component bị quá nhiều thứ phụ thuộc → sửa nó phải test lại cả hệ thống |
| **Framework/runtime cũ** | Xây bằng phiên bản cũ, bản mới có tính năng tối ưu / deprecate cái đang dùng |
| **Giải pháp ngắn hạn** | Làm tạm vì áp lực đối thủ, giờ cần viết lại cho dễ bảo trì |
| **Tìm ra cách tốt hơn** | Học được kỹ thuật mới, viết lại gọn hơn nhiều |
| **Đổi yêu cầu** | Luật thay đổi → phải sửa nhiều component |
| **Bug** | Lỗi luôn phát sinh trong mọi sản phẩm |

---

## 3. Cách xử lý technical debt

1. **Làm cho nó HIỆN RÕ (visible):** với cả stakeholder kỹ thuật lẫn nghiệp vụ — họ cần hiểu *vì sao* phải dành thời gian trả nợ. (Có thể đưa vào backlog như item rõ ràng.)
2. **Xử lý SỚM:** như nợ tài chính, để lâu thì "lãi" chồng chất — tính năng mới xây trên phần nợ càng ngày càng khó bảo trì.
3. **TRỘN với giá trị nghiệp vụ:** **đừng** dành nguyên một Sprint chỉ để trả nợ. Mỗi Sprint nên vừa trả một phần nợ vừa giao giá trị mới.

> [!warning] Nợ kỹ thuật giấu kín làm **giảm transparency** ([[01-Tong-quan-Agile-Scrum]]): nhìn bề ngoài team có vẻ tiến xa hơn thực tế (vì tính năng "xong" nhưng nợ ngầm bên dưới). Đưa nợ ra ánh sáng là khôi phục minh bạch.

```
★ Insight ─────────────────────────────────────
• Nguyên tắc "trộn, đừng dồn" gắn chặt với luật "mỗi Sprint ra Increment có giá
  trị" ([[04-Scrum-Events]]): một Sprint chỉ trả nợ = một Sprint không giao giá
  trị nghiệp vụ → giống hardening sprint (anti-pattern). Trả nợ đều tay từng chút
  qua mỗi Sprint là cách bền vững.
• Definition of Done chặt là "vắc-xin" chống nợ mới: nếu DoD bắt buộc test, review,
  không lỗi lint... thì nợ khó lọt vào ngay từ đầu ([[03-Scrum-Artifacts-va-DoD]]).
─────────────────────────────────────────────────
```

---

## 4. Tự kiểm tra

1. Technical debt là gì? *(việc phát sinh thêm sau này do quyết định/hoàn cảnh hiện tại)*
2. Có thể xóa hoàn toàn nợ kỹ thuật? *(không, chỉ quản lý)*
3. 3 cách xử lý? *(làm hiện rõ, xử lý sớm, trộn với giá trị nghiệp vụ)*
4. Vì sao không nên dành cả Sprint chỉ trả nợ? *(không giao giá trị → giống anti-pattern hardening sprint)*
5. Nợ giấu kín ảnh hưởng trụ cột nào? *(Transparency — team trông tiến xa hơn thực tế)*

## Liên quan
- [[00-MOC-Agile-Scrum|⬅ MOC Agile-Scrum]]
- Trước: [[05-Product-Backlog-Refinement]] · Kế tiếp: [[07-Cong-cu-va-Theo-doi|Công cụ & theo dõi]]
- [[03-Scrum-Artifacts-va-DoD|Definition of Done]]
