---
title: "Công cụ Scrum & theo dõi tiến độ"
section: 00-Foundations/03-Agile-Scrum
tags: [scrum, jira, board, kanban, burndown, velocity, tracking, foundations, fresher]
related:
  - "[[04-Scrum-Events]]"
  - "[[12-Estimation]]"
difficulty: ⭐⭐⭐
estimated_time: 30m
source: ["Agile Practitioner with Scrum", Jira docs]
---

# Công cụ Scrum & theo dõi tiến độ

> [!summary] TL;DR
> Công cụ (**Jira, Trello, Azure DevOps**…) **không** làm bạn agile — agility là tư duy & văn hóa. Nhưng công cụ **hỗ trợ** khi: team phân tán địa lý, cần tự động hóa quy trình, cần **analytics/visibility**, hoặc nhiều team chung một sản phẩm. **Scrum board** (To Do · In Progress · Done) trực quan hóa luồng việc giống Kanban. Theo dõi tiến độ qua **burndown chart**, **information radiator**, **WIP limit**, và **velocity**.

---

## 1. "Công cụ không làm bạn agile"

> Agility là **mindset & culture**. Dùng Jira/Trello **một mình không** biến tổ chức/cá nhân thành agile. Khi mới bắt đầu, **đừng** sa đà vào công cụ — tập trung dùng những gì đang có để ra **kết quả** hiệu quả nhất.

### Nhưng khi nào công cụ thật sự giúp?

| Tình huống | Công cụ giúp gì |
|------------|-----------------|
| Team **phân tán địa lý** | Cải thiện giao tiếp, chia sẻ thông tin |
| Cần **tự động hóa** | Tự động quy trình, deploy nhanh (nối [[../02-Git/12-CI-CD-la-gi|CI/CD]]) |
| Cần **nhìn thấy số liệu** | Analytics có sẵn → tăng visibility quy trình |
| **Nhiều team** chung 1 sản phẩm | Gom & chia sẻ thông tin giữa các team |

```
★ Insight ─────────────────────────────────────
• Câu "tool ≠ agile" hay được hỏi để bẫy: nhiều người nghĩ cài Jira + họp standup
  = agile. Tool chỉ KHUẾCH ĐẠI cách làm việc sẵn có — quy trình tốt thì tool giúp
  tốt hơn, quy trình dở thì tool chỉ làm cái dở chạy nhanh hơn. Tư duy trước, tool sau.
─────────────────────────────────────────────────
```

---

## 2. Scrum board (Agile board)

Board sắp xếp **Sprint Backlog item** theo **trạng thái hoàn thành**, dạng các cột (swim lane). Giống **Kanban board** (Kanban = framework để *trực quan hóa & quản lý* luồng việc).

```text
   TO DO            IN PROGRESS         DONE
 ┌──────────┐     ┌──────────────┐   ┌──────────┐
 │ Story #4 │     │ Story #2  ⚠️ │   │ Story #1 │
 │ Story #5 │     │ Bug #7       │   │ Story #3 │
 │ Bug #8   │     │              │   │          │
 └──────────┘     └──────────────┘   └──────────┘
                   ↑ WIP limit = 4 (vượt → cảnh báo)
```

- Số cột/trạng thái **tùy chỉnh** theo team (có thể thêm "Code Review", "Testing"…).
- Item **di chuyển** giữa cột khi đổi trạng thái → tăng **transparency**.
- Giúp **phát hiện nút thắt cổ chai (bottleneck)** (vd cột "Testing" ùn item).

### WIP limit (Work In Progress)
Đặt **trần** số item ở một cột (vd "In Progress" ≤ 4). Vượt trần → công cụ **cảnh báo trực quan**. Mục đích: buộc team **làm xong rồi mới nhận việc mới**, tránh "ôm đồm" nhiều việc dở.

```
★ Insight ─────────────────────────────────────
• WIP limit chống ảo giác năng suất: 10 việc "đang làm dở" trông bận rộn nhưng
  giá trị giao = 0 cho tới khi có cái DONE. Giới hạn WIP ép dòng chảy "ít việc,
  làm xong, giao" → giảm context switching (đã gặp ở [[02-Scrum-Roles]]) & lộ
  bottleneck sớm. Đây là tinh thần cốt lõi Kanban nhúng vào Scrum.
─────────────────────────────────────────────────
```

---

## 3. Theo dõi tiến độ (tracking)

### Burndown chart
Biểu đồ cho thấy **lượng việc còn lại** giảm dần theo ngày trong Sprint. Trục X = ngày, trục Y = công việc còn lại (story point/giờ). Đường lý tưởng đi xuống đều về 0 cuối Sprint.

```text
 còn lại │\
  (point)│ \___        ← thực tế: chững lại = có vấn đề
         │     \__
         │        \___
         │            \____
         └────────────────── ngày →  (lý tưởng: đường thẳng xuống 0)
```

### Information radiator
Bảng (điện tử hoặc vật lý) **hiển thị nổi bật** thông tin tiến độ cho mọi người **liếc là thấy**: activity stream, burndown, "sprint health", biểu đồ created-vs-resolved… Mục đích: minh bạch, ai cũng nắm trạng thái mà không cần hỏi.

### Subscription / query
Người không dự Daily (vd CIO, BA cấp cao) **đăng ký nhận email** theo query (vd "các story chưa bắt đầu") → tự cập nhật mà không làm phiền team.

### Velocity (vận tốc)
**Velocity** = tổng story point team **hoàn thành** mỗi Sprint. Trung bình vài Sprint → **dự báo** lượng việc Sprint tới gánh được. (Chi tiết & cách dùng để forecast: [[12-Estimation]].)

---

## 4. Bảng tổng — công cụ theo dõi

| Công cụ | Cho biết |
|---------|----------|
| **Scrum board** | Trạng thái từng item (To Do/In Progress/Done) |
| **WIP limit** | Cảnh báo ôm quá nhiều việc dở |
| **Burndown chart** | Việc còn lại giảm thế nào trong Sprint |
| **Information radiator** | Tổng quan tiến độ, ai cũng thấy |
| **Velocity** | Năng lực gánh việc → dự báo Sprint sau |

> [!tip] Một số công cụ phổ biến: **Jira** (mạnh, doanh nghiệp), **Trello** (đơn giản, Kanban), **Azure DevOps**, **Asana**, **GitHub Projects**. Khái niệm board/burndown giống nhau, khác giao diện.

---

## 5. Tự kiểm tra

1. Dùng Jira có làm team agile không? *(không — agile là tư duy; tool chỉ hỗ trợ)*
2. Scrum board 3 cột cơ bản? *(To Do, In Progress, Done)*
3. WIP limit để làm gì? *(giới hạn việc dở, tránh ôm đồm, lộ bottleneck)*
4. Burndown chart thể hiện gì? *(việc còn lại giảm dần trong Sprint)*
5. Velocity là gì, dùng làm gì? *(point hoàn thành/Sprint; dự báo năng lực Sprint sau)*

## Liên quan
- [[00-MOC-Agile-Scrum|⬅ MOC Agile-Scrum]]
- Trước: [[06-Technical-Debt]] · Kế tiếp: [[08-Certifications|Certifications]]
- [[12-Estimation|Velocity & Estimation]] · [[../02-Git/12-CI-CD-la-gi|Tự động hóa CI/CD]]
