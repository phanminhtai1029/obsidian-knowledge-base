---
title: "Estimation — Ước lượng (story point, Planning Poker, velocity)"
section: 00-Foundations/03-Agile-Scrum
tags: [estimation, story-point, planning-poker, velocity, t-shirt, pert, foundations, fresher]
related:
  - "[[05-Product-Backlog-Refinement]]"
  - "[[07-Cong-cu-va-Theo-doi]]"
  - "[[10-Tai-lieu-Yeu-cau-va-Dac-ta]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 40m
source: ["Agile Practitioner with Scrum", Mike Cohn - Agile Estimating]
---

# Estimation — Ước lượng

> [!summary] TL;DR
> Agile ước lượng bằng **đơn vị tương đối** (**story point**) thay vì giờ/ngày tuyệt đối, vì con người ước lượng *so sánh* tốt hơn ước lượng *con số tuyệt đối*. Story point gói cả **khối lượng + độ phức tạp + rủi ro**. Thường dùng **dãy Fibonacci** (1,2,3,5,8,13…) vì việc càng to càng khó chính xác. Kỹ thuật: **Planning Poker** (ước lượng kín rồi đồng thuận), **T-shirt sizing**, **Affinity**, **PERT/3-point**. **Velocity** = point hoàn thành mỗi Sprint → dùng để **dự báo**. Tuyệt đối **không** quy đổi point ↔ giờ cố định.

---

## 1. Vì sao dùng tương đối, không tuyệt đối?

```
★ Insight ─────────────────────────────────────
• Con người DỞ ở ước lượng tuyệt đối ("việc này mất bao nhiêu giờ?") nhưng GIỎI
  ở so sánh ("việc A to gấp đôi việc B"). Story point khai thác đúng thế mạnh đó:
  bạn chỉ cần nói item này tương đương/lớn hơn item kia, không cần đoán số giờ
  chính xác — vốn luôn sai vì phụ thuộc người làm, gián đoạn, độ quen việc.
• Point còn TÁCH ước lượng khỏi cá nhân: "8 điểm" là độ lớn của việc, không phải
  "8 giờ của bạn". Người nhanh/chậm khác nhau nhưng độ lớn việc thì chung → ước
  lượng ổn định hơn, và tránh áp lực "sao làm chậm thế".
─────────────────────────────────────────────────
```

| | **Tuyệt đối** (giờ/ngày) | **Tương đối** (story point) |
|---|---|---|
| Đơn vị | Giờ, ngày công | Point (số không thứ nguyên) |
| Gói yếu tố | Chủ yếu thời gian | **Khối lượng + phức tạp + rủi ro/bất định** |
| Phụ thuộc người làm | Cao (nhanh/chậm khác nhau) | Thấp (độ lớn việc là chung) |
| Độ ổn định | Dễ sai | Ổn định hơn qua thời gian |

> **Story point** = con số **tương đối** thể hiện *độ lớn tổng thể* của một item. Item 8 point lớn/khó hơn item 5 point — nhưng **không** map cứng "8 point = 8 giờ".

---

## 2. Vì sao dùng dãy Fibonacci?

Dãy ước lượng phổ biến: **1, 2, 3, 5, 8, 13, 21…** (Fibonacci), hoặc biến thể 0, ½, 1, 2, 3, 5, 8, 13, 20, 40, 100.

- **Khoảng cách giãn dần** phản ánh sự thật: việc càng lớn càng **khó ước lượng chính xác** → không cho phép chọn "14, 15, 16" (giả vờ chính xác). Phải chọn 13 hoặc 21.
- Buộc người ước lượng **quyết** giữa các bậc rõ rệt, tránh tranh cãi vô nghĩa "7 hay 8".
- Item quá lớn (vd 21, 40, 100) = tín hiệu **cần chẻ nhỏ** (refinement → [[05-Product-Backlog-Refinement]]).

---

## 3. Planning Poker ⭐ (kỹ thuật phổ biến nhất)

Quy trình ước lượng nhóm, mỗi người có bộ bài Fibonacci:

```text
1. PO đọc & làm rõ một story
2. Mỗi Dev chọn 1 lá bài (ƯỚC LƯỢNG KÍN — không cho người khác thấy)
3. Tất cả LẬT BÀI cùng lúc
4. Nếu KHỚP  → chốt điểm đó
   Nếu LỆCH → người CAO nhất & THẤP nhất giải thích lý do
5. Bàn lại → lặp từ bước 2 cho tới khi ĐỒNG THUẬN
```

Ví dụ:
```text
Story: "tạo hồ sơ online cho hội viên"
Vòng 1:  3   5   8   8   13     → lệch (3 và 13 giải thích)
Vòng 2:  5   8   8   8   8      → đồng thuận = 8 story points
```

### 3 nguyên tắc cốt lõi của Planning Poker

1. **Ước lượng kín rồi lật cùng lúc** → tránh **anchoring** (bị số người khác "neo" suy nghĩ).
2. **Người cao/thấp nhất giải thích** → người có ước lượng lệch thường thấy điều người khác bỏ sót (rủi ro ẩn, hoặc cách làm đơn giản hơn).
3. **Đồng thuận, KHÔNG lấy trung bình / biểu quyết đa số** → bàn tới khi cả nhóm nhất trí; "đa số thắng" bỏ sót góc nhìn quan trọng.

> [!warning] Planning Poker chỉ là **một** kỹ thuật, **không** bắt buộc bởi Scrum. Đừng tuyệt đối hóa.

---

## 4. Các kỹ thuật ước lượng khác

| Kỹ thuật | Cách làm | Khi nào |
|----------|----------|---------|
| **Planning Poker** | Bài kín → đồng thuận | Ước lượng kỹ từng story |
| **T-shirt sizing** | Gán XS/S/M/L/XL thay vì số | Ước lượng **nhanh, thô** lúc đầu, nhiều item |
| **Affinity estimation** | Nhóm các story "na ná nhau" về cùng cỡ | Backlog rất nhiều item, cần phân loại nhanh |
| **Dot voting** | Mỗi người dán điểm vào item ưu tiên/cỡ | Quyết nhanh tập thể |
| **3-point / PERT** | Tính từ 3 con số: Lạc quan(O), Khả năng nhất(M), Bi quan(P) | Khi cần con số thời gian có tính rủi ro |

### Công thức PERT (3-point estimate)

```text
Estimate = (O + 4M + P) / 6

O = Optimistic (lạc quan nhất)
M = Most likely (khả năng cao nhất)
P = Pessimistic (bi quan nhất)

Ví dụ: O=2, M=5, P=14  →  (2 + 4×5 + 14)/6 = 36/6 = 6 ngày
```

PERT cho ước lượng "có trọng số", giảm thiên lệch lạc quan.

---

## 5. Velocity — dùng ước lượng để dự báo ⭐

**Velocity** = tổng story point **HOÀN THÀNH** (đạt Definition of Done) trong một Sprint.

```text
 Sprint 1: 18 pt    Sprint 2: 22 pt    Sprint 3: 20 pt
 → Velocity trung bình ≈ 20 pt/Sprint
 → Backlog còn 100 pt  ⇒  dự báo ~5 Sprint nữa
```

- Lấy **trung bình vài Sprint** để có velocity ổn định (1 Sprint đầu chưa đáng tin).
- Dùng để **forecast**: còn bao nhiêu Sprint, Sprint tới gánh được bao nhiêu point.
- Chỉ tính point của item **thật sự done**, không tính item làm dở.

```
★ Insight ─────────────────────────────────────
• Velocity là cầu nối "tương đối → kế hoạch thực": point không là giờ, nhưng khi
  biết team làm ~20 point/Sprint thì point trở nên DỰ BÁO ĐƯỢC theo lịch. Đây là
  lý do không cần quy đổi point↔giờ — velocity tự "hiệu chỉnh" qua thực tế từng team.
• Velocity của team A KHÔNG so sánh được với team B (mỗi team chuẩn point khác
  nhau). Dùng velocity để so sánh/ép giữa các team là sai lầm kinh điển của quản lý.
─────────────────────────────────────────────────
```

> [!warning] **Không** ép point thành KPI cá nhân hay so velocity giữa các team. Point là công cụ **dự báo nội bộ**, không phải thước đo năng suất để thưởng/phạt — làm vậy team sẽ "thổi phồng" point (point inflation).

---

## 6. Bảng tổng — câu trả lời phỏng vấn nhanh

| Câu hỏi | Trả lời ngắn |
|---------|--------------|
| Story point là gì? | Đơn vị **tương đối** đo độ lớn (khối lượng + phức tạp + rủi ro) |
| Vì sao không dùng giờ? | Người ước lượng so sánh tốt hơn đoán tuyệt đối; point độc lập người làm |
| Vì sao Fibonacci? | Khoảng giãn dần phản ánh "việc to khó ước lượng chính xác" |
| Planning Poker chống gì? | Anchoring (bị neo bởi ước lượng người khác) |
| Velocity là gì? | Point hoàn thành/Sprint → dự báo tiến độ |
| PERT tính sao? | (O + 4M + P) / 6 |

## 7. Tự kiểm tra

1. Story point đo cái gì? *(độ lớn tương đối: khối lượng + phức tạp + rủi ro)*
2. Vì sao Agile dùng tương đối thay tuyệt đối? *(người so sánh tốt hơn đoán số tuyệt đối)*
3. 3 nguyên tắc Planning Poker? *(kín rồi lật cùng lúc; cao/thấp giải thích; đồng thuận không lấy trung bình)*
4. Velocity dùng để làm gì? *(dự báo số Sprint/point gánh được)*
5. Có nên so velocity giữa 2 team? *(không — chuẩn point khác nhau)*
6. Công thức PERT? *((O + 4M + P)/6)*

## Liên quan
- [[00-MOC-Agile-Scrum|⬅ MOC Agile-Scrum]]
- Trước: [[11-Wireframe-Mockup-Prototype]] · Kế tiếp: [[13-Test-Plan-va-QA|Test Plan & QA]]
- [[05-Product-Backlog-Refinement|Refinement]] · [[07-Cong-cu-va-Theo-doi|Velocity & burndown]]
