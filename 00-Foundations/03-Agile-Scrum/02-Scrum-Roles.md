---
title: "Scrum Roles — Product Owner · Dev Team · Scrum Master"
section: 00-Foundations/03-Agile-Scrum
tags: [scrum, roles, product-owner, scrum-master, dev-team, foundations, fresher]
related:
  - "[[01-Tong-quan-Agile-Scrum]]"
  - "[[03-Scrum-Artifacts-va-DoD]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [Scrum Guide, "Agile Practitioner with Scrum"]
---

# Scrum Roles — Product Owner · Dev Team · Scrum Master

> [!summary] TL;DR
> Scrum Team gồm **3 vai trò**: **Product Owner** quyết định *cái gì* cần làm và thứ tự ưu tiên (sở hữu Product Backlog); **Development Team** quyết định *làm thế nào* và xây ra Increment (tự tổ chức, đa kỹ năng, ≤10 người); **Scrum Master** là **servant leader** — coach/facilitator giúp team theo đúng Scrum, gỡ trở ngại, không phải sếp. Cả 3 **chia sẻ** trách nhiệm quản lý; **không có** Project Manager.

---

## 1. Product Owner (PO) — "cái GÌ"

PO chịu trách nhiệm **tối đa hóa giá trị** sản phẩm. Cụ thể:

- **Sở hữu Product Backlog**: nội dung và đặc biệt là **thứ tự ưu tiên** (order). Người khác có thể đề xuất, nhưng PO **có tiếng nói cuối cùng**.
- Bảo đảm Dev Team luôn làm **việc quan trọng nhất** trước.
- Là **một người**, không phải một hội đồng (tránh xung đột quan điểm → quyết chậm).
- Có thể bị ảnh hưởng bởi stakeholder trong/ngoài team, nhưng **quyết định là của PO**.

> [!warning] PO **không phải** Project Manager, **không phải** Product Manager. Đặc điểm PO tốt: hiểu **nghiệp vụ (domain)**, hiểu **công nghệ vừa đủ** để ra quyết định & trao đổi với Dev, **giao tiếp giỏi** để align team kỹ thuật và business, được **stakeholder tôn trọng**.

> Ví dụ Agile Fitness: **Bob Jones** — 20 năm trong ngành gym, hiểu khách hàng & nhân viên, hiểu cả công nghệ lẫn dinh dưỡng, biết đối thủ làm gì, giao tiếp thuyết phục.

---

## 2. Development Team — "làm THẾ NÀO"

Dev Team **biến** một phần Product Backlog thành **Increment có thể release** (phần mềm chạy được). Hai đặc tính cốt lõi:

| Đặc tính | Nghĩa |
|----------|-------|
| **Self-organizing** (tự tổ chức) | Tự biết cách làm việc, ít cần chỉ đạo từ ngoài; PO **không** giao việc, team **tự kéo (pull)** việc |
| **Cross-functional** (đa kỹ năng) | Có **đủ** mọi kỹ năng để hoàn thành việc: design, code, test, doc, deploy… |

**Không có** sub-team (không chia "team test", "team doc") và **không có** thứ bậc (không team lead / senior / junior trong nghĩa phân cấp Scrum).

### Quy mô & kỹ năng

- **Kích thước: 10 người trở xuống** (nếu SM/PO kiêm việc dev thì tính luôn). <3 người → thiếu kỹ năng, khó cross-functional; >10 → rối giao tiếp. (Quy tắc "two-pizza" của Jeff Bezos: đội nhỏ đủ để 2 cái pizza nuôi.)
- **I-shaped** = giỏi sâu 1 mảng. **T-shaped** = sâu 1 mảng + biết rộng nhiều mảng (đáng quý hơn).
- Scrum **không cấm** chuyên môn hóa; chỉ là biết nhiều kỹ năng giúp team linh hoạt hơn.
- Làm **nhiều team cùng lúc** giảm năng suất do **context switching**.

### Feature team vs Component team

| | **Component team** | **Feature team** |
|---|---|---|
| Làm gì | Chuyên 1 lớp kỹ thuật (vd chỉ UI, hoặc chỉ DB) | Làm trọn 1 tính năng đầu-cuối (UI + logic + DB) |
| Sản phẩm | Mảnh ghép kỹ thuật | "Lát cắt dọc" có giá trị dùng được |
| Agile chuộng | | ✅ (vì giao **giá trị nghiệp vụ** trọn vẹn) |

> Ví dụ Agile Fitness Dev Team: Jim (design + code), Maria (code + từng test), Ravi (QA + hiểu nghiệp vụ), Chuck (doc + build tools), Jamal (code + test automation) → đủ kỹ năng = cross-functional.

```
★ Insight ─────────────────────────────────────
• "Tự tổ chức" + "đa kỹ năng" đi đôi: team tự quyết cách làm CHỈ KHI có đủ kỹ
  năng để làm trọn việc. Đó là lý do Scrum chuộng feature team — một đội làm được
  "lát cắt dọc" thì mới tự chủ giao giá trị, không phải chờ "team khác" làm phần
  của họ. Component team luôn bị kẹt phụ thuộc chéo.
• PO KHÔNG giao việc, Dev tự kéo việc — đây là "bottom-up intelligence": người
  làm trực tiếp hiểu rõ nhất nên tự ước lượng & nhận việc theo năng lực. Nối thẳng
  sang cách Sprint Planning vận hành ([[04-Scrum-Events]]).
─────────────────────────────────────────────────
```

---

## 3. Scrum Master (SM) — servant leader

"Master" KHÔNG phải "ông chủ" hay quản lý. SM là người **thông thạo (mastery) Scrum/Agile**, đóng các vai:

- **Coach / Mentor**: dạy team & tổ chức về Agile, *vì sao* mỗi luật tồn tại (chống "mechanical scrum").
- **Facilitator**: tổ chức, dẫn dắt các event cho hiệu quả.
- **Problem solver / Impediment remover**: gỡ trở ngại cho team (vd thuyết phục mua công cụ test).
- **Servant leader**: *phục vụ* để team giỏi hơn, **không micromanage**.

### Servant leader nghĩa là gì (ví dụ)

- KHÔNG ra lệnh "làm nhanh lên" → mà **giúp** team làm hiệu quả hơn.
- Để Dev Team **tự thực thi kế hoạch** (không quản lý vi mô).
- **Thuyết phục** (influence) management mua phần mềm team cần.
- **Tổ chức** training để giảm lỗi build.

### Xử lý xung đột

SM xử lý xung đột **theo từng tình huống (case by case)**, không làm "quan tòa": có khi để team tự giải quyết, có khi can thiệp sớm để hạ nhiệt. Tùy ngữ cảnh.

> Ví dụ Agile Fitness: **Ashley Wright** — sống với nguyên tắc agile, hiểu chiều sâu Scrum, thuyết phục được cả business lẫn kỹ thuật, là mentor cho cả công ty.

```
★ Insight ─────────────────────────────────────
• "Servant leader" là nghịch lý cố ý: lãnh đạo bằng cách PHỤC VỤ, quyền lực đến
  từ ảnh hưởng & chuyên môn chứ không từ chức vụ. SM không có quyền sa thải hay
  giao việc — sức mạnh là coach & gỡ rào. Đây là điểm hay bị hỏi để phân biệt SM
  với "team lead" hay "manager" truyền thống.
─────────────────────────────────────────────────
```

---

## 4. Bảng so sánh 3 vai trò ⭐

| | **Product Owner** | **Development Team** | **Scrum Master** |
|---|---|---|---|
| Trả lời câu hỏi | Cái **GÌ** & thứ tự ưu tiên | Làm **THẾ NÀO** | Giúp làm Scrum **ĐÚNG** |
| Sở hữu | Product Backlog | Sprint Backlog & cách làm | Quy trình Scrum |
| Số lượng | 1 người | ≤10, tự tổ chức, đa kỹ năng | 1 người |
| KHÔNG phải | Project/Product Manager | Nhóm coder thuần | Sếp / quản lý |
| Tạo ra | Định hướng giá trị | Increment (phần mềm chạy) | Môi trường để team thành công |

> [!tip] Cách nhớ nhanh: **PO = WHAT, Dev = HOW, SM = giúp làm đúng cách.** Trách nhiệm quản lý **chia sẻ** giữa cả 3.

---

## 5. Tự kiểm tra

1. Ai sở hữu Product Backlog và thứ tự ưu tiên? *(Product Owner)*
2. Hai đặc tính cốt lõi của Dev Team? *(self-organizing & cross-functional)*
3. Kích thước Dev Team khuyến nghị? *(≤10)*
4. Scrum Master là sếp của team? *(không — servant leader, coach/facilitator)*
5. T-shaped khác I-shaped? *(T: sâu 1 mảng + rộng nhiều mảng; I: chỉ sâu 1 mảng)*
6. PO khác Project Manager chỗ nào? *(PO sở hữu giá trị sản phẩm/backlog, là 1 người trong Scrum Team; PM không thuộc Scrum Team)*

## Liên quan
- [[00-MOC-Agile-Scrum|⬅ MOC Agile-Scrum]]
- Trước: [[01-Tong-quan-Agile-Scrum]] · Kế tiếp: [[03-Scrum-Artifacts-va-DoD|Artifacts & DoD]]
- [[04-Scrum-Events|Events]]
