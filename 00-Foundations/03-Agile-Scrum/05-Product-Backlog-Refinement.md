---
title: "Product Backlog & Refinement (Grooming)"
section: 00-Foundations/03-Agile-Scrum
tags: [scrum, backlog, refinement, grooming, planning-poker, estimation, foundations, fresher]
related:
  - "[[03-Scrum-Artifacts-va-DoD]]"
  - "[[04-Scrum-Events]]"
  - "[[12-Estimation]]"
difficulty: ⭐⭐⭐
estimated_time: 30m
source: [Scrum Guide, "Agile Practitioner with Scrum"]
---

# Product Backlog & Refinement (Grooming)

> [!summary] TL;DR
> **Product Backlog Refinement** (còn gọi **grooming**) = hoạt động **liên tục** giữ cho Product Backlog **chi tiết, ưu tiên rõ, sẵn sàng** cho các Sprint tới. PO + Dev Team cùng làm; **không nên quá 10%** sức (capacity) của Dev Team. Việc gồm: thêm chi tiết, xếp lại ưu tiên, thêm/bớt item, **chẻ item to thành nhỏ**, và **ước lượng (estimate)**. Refinement tốt → Sprint Planning & thực thi mượt hơn.

---

## 1. Refinement là gì?

Trong Scrum Guide, refinement được định nghĩa **lỏng** nhưng là hoạt động then chốt. Khác các event timeboxed cố định, refinement có thể làm **một buổi** hoặc **rải nhiều buổi** trong Sprint.

- **Ai làm:** Product Owner + Development Team (bắt buộc cả hai).
- **Giới hạn:** ≤ **10%** capacity Dev Team. *(Ví dụ Sprint 2 tuần = 80 giờ → ≤ 8 giờ refinement.)*
- **Bản chất:** hoạt động liên tục, **không** phải 1 trong 5 event chính thức ([[04-Scrum-Events]]).

---

## 2. Refinement làm những gì?

| Việc | Mô tả |
|------|-------|
| **Thêm chi tiết** | Bổ sung mô tả, acceptance criteria cho item |
| **Xếp lại ưu tiên** | Cập nhật thứ tự theo giá trị/độ rủi ro mới |
| **Thêm / bớt item** | Đưa item mới vào, bỏ item không còn cần (giữ backlog sạch) |
| **Chẻ nhỏ (split)** | Tách **epic / story lớn** thành nhiều story nhỏ, làm được trong 1 Sprint |
| **Ước lượng (estimate)** | Gán estimate (story point) cho item — chi tiết [[12-Estimation]] |

> Ví dụ Agile Fitness: với story "xem video tập luyện", Dev hỏi → PO làm rõ. Story "thanh toán online" được chẻ thành 3 (thẻ tín dụng / e-check / thanh toán định kỳ); rồi hạ ưu tiên (vì đa số hội viên thanh toán tại phòng gym). Thêm 2 story mới, ước lượng vài item cho Sprint tương lai.

```
★ Insight ─────────────────────────────────────
• Refinement giải bài toán "đỉnh backlog nhỏ & sẵn, đáy to & mơ hồ" ([[03-Scrum-Artifacts-va-DoD]]):
  nó liên tục "kéo" item từ đáy lên, chẻ nhỏ & làm rõ để khi tới Sprint Planning
  thì item ở đỉnh ĐÃ sẵn — Planning chỉ việc chọn, không sa lầy phân tích.
• Trần 10% capacity là cố ý: refinement quan trọng nhưng KHÔNG được nuốt thời gian
  làm sản phẩm. Đủ để backlog luôn "sẵn 1–2 Sprint tới", không hơn.
─────────────────────────────────────────────────
```

---

## 3. "Ready" — đối xứng với "Done"

Nhiều team đặt **Definition of Ready (DoR)**: tiêu chí để một item đủ rõ để **được kéo vào Sprint** (đã có mô tả, acceptance criteria, estimate, không phụ thuộc lớn). Đối xứng với **Definition of Done** (khi nào item *xong*).

```text
   Backlog item:  [thô] ──refinement──▶ [READY] ──Sprint──▶ [DONE]
                          (đủ rõ để làm)        (đạt DoD)
```

> [!tip] **INVEST** là bộ tiêu chí cho user story tốt (rất hợp dùng khi refinement & chẻ story): **I**ndependent, **N**egotiable, **V**aluable, **E**stimable, **S**mall, **T**estable. Chi tiết ở [[10-Tai-lieu-Yeu-cau-va-Dac-ta]].

---

## 4. Planning Poker (giới thiệu)

Một kỹ thuật **ước lượng** phổ biến trong refinement: cả Dev Team dùng bộ bài số (thường dãy Fibonacci), **mỗi người chọn kín** rồi **lật cùng lúc**; nếu lệch nhau thì người cao nhất & thấp nhất giải thích, bàn lại, lặp tới khi **đồng thuận**.

```text
  Story: "tạo hồ sơ online cho hội viên"
  Lật bài:  3   5   8   8   13      → lệch → bàn → lật lại
            5   8   8   8   8       → đồng thuận = 8 story points
```

> [!warning] Đây chỉ là **giới thiệu**. Toàn bộ về **story point, tương đối vs tuyệt đối, T-shirt sizing, PERT/3-point, velocity** — xem note chuyên sâu [[12-Estimation]].

---

## 5. Lợi ích của refinement tốt

- Sprint Planning nhanh & nhẹ hơn (item đã sẵn).
- Thực thi Sprint mượt (ít "ồ, item này chưa rõ").
- Dev Team thấy **bức tranh lớn**/roadmap, đỡ "cắm đầu" vào task hằng ngày.
- Backlog luôn sạch, ưu tiên đúng.

---

## 6. Tự kiểm tra

1. Refinement là 1 trong 5 event chính thức? *(không — là hoạt động liên tục)*
2. Giới hạn thời gian refinement? *(≤10% capacity Dev Team)*
3. Ai bắt buộc tham gia refinement? *(PO + Dev Team)*
4. 5 việc chính của refinement? *(thêm chi tiết, xếp ưu tiên, thêm/bớt, chẻ nhỏ, estimate)*
5. INVEST viết tắt gì? *(Independent, Negotiable, Valuable, Estimable, Small, Testable)*

## Liên quan
- [[00-MOC-Agile-Scrum|⬅ MOC Agile-Scrum]]
- Trước: [[04-Scrum-Events]] · Kế tiếp: [[06-Technical-Debt|Technical Debt]]
- [[12-Estimation|Estimation chi tiết]] · [[10-Tai-lieu-Yeu-cau-va-Dac-ta|INVEST & User story]]
