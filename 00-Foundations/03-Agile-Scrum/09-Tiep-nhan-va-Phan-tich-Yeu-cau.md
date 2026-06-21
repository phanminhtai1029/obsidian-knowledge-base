---
title: "Tiếp nhận & phân tích yêu cầu (Requirement Analysis)"
section: 00-Foundations/03-Agile-Scrum
tags: [requirement, ba, scope, elicitation, srs, foundations, fresher]
related:
  - "[[10-Tai-lieu-Yeu-cau-va-Dac-ta]]"
  - "[[11-Wireframe-Mockup-Prototype]]"
  - "[[05-Product-Backlog-Refinement]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 40m
source: [BABOK, IEEE 830, tổng hợp thực hành]
---

# Tiếp nhận & phân tích yêu cầu (Requirement Analysis)

> [!summary] TL;DR
> Khi vào dự án có yêu cầu từ khách hàng, chuỗi việc điển hình: **tiếp nhận → đọc & làm rõ (clarify) → rã scope (in/out-of-scope) → phân loại yêu cầu → viết tài liệu → khách duyệt → chẻ thành backlog**. **4 loại requirement** cần phân biệt: **Business** (vì sao/mục tiêu), **User** (người dùng cần làm gì), **Functional** (hệ thống *phải làm gì*), **Non-functional** (hệ thống phải *tốt thế nào* — hiệu năng, bảo mật…). Yêu cầu mơ hồ là nguồn gốc số 1 của dự án thất bại → **làm rõ ngay từ đầu**.

---

## 1. Toàn cảnh: từ yêu cầu khách → code

```text
 Khách hàng          BA / team                              Dev Team
 ─────────           ─────────                              ────────
 nêu nhu cầu  ─▶ 1.Tiếp nhận ─▶ 2.Làm rõ ─▶ 3.Rã scope ─▶ 4.Phân loại
                                                               │
                              ┌────────────────────────────────┘
                              ▼
                 5.Viết tài liệu (BRD→SRS→Spec)  ─▶ 6.Khách DUYỆT (sign-off)
                              │
                              ▼
                 7.Chẻ thành Epic→User Story→Task ─▶ 8.Estimate ─▶ vào Sprint
```

> Tài liệu chi tiết (BRD/SRS/Spec): [[10-Tai-lieu-Yeu-cau-va-Dac-ta]]. Chẻ story & estimate: [[05-Product-Backlog-Refinement]], [[12-Estimation]].

```
★ Insight ─────────────────────────────────────
• Trong Agile, chuỗi này KHÔNG làm một lần đầu dự án rồi xong (như Waterfall) mà
  lặp lại liên tục, làm rõ dần (progressive elaboration): chỉ đào sâu yêu cầu cho
  1–2 Sprint sắp tới, phần xa để thô. "Tài liệu vừa đủ" — đúng tinh thần Manifesto
  ([[01-Tong-quan-Agile-Scrum]]), không phải bỏ tài liệu.
─────────────────────────────────────────────────
```

---

## 2. Bước 1–2: Tiếp nhận & làm rõ (elicitation + clarify)

**Elicitation** (khai thác yêu cầu) — các kỹ thuật lấy yêu cầu từ stakeholder:

| Kỹ thuật | Mô tả |
|----------|-------|
| **Phỏng vấn** (interview) | Hỏi trực tiếp stakeholder/khách |
| **Workshop** | Họp nhóm nhiều bên cùng thống nhất |
| **Khảo sát** (survey) | Lấy ý kiến số đông |
| **Quan sát** (observation) | Xem người dùng làm việc thực tế |
| **Phân tích tài liệu** | Đọc tài liệu nghiệp vụ, hệ thống cũ |
| **Prototype** | Dựng mẫu để khách phản hồi → [[11-Wireframe-Mockup-Prototype]] |

**Làm rõ (clarify):** yêu cầu khách thường mơ hồ/mâu thuẫn/thiếu. Phải:
- Đặt câu hỏi đào sâu ("hệ thống cần chịu bao nhiêu user cùng lúc?").
- Ghi lại **giả định (assumptions)** & **ràng buộc (constraints)**.
- Phát hiện yêu cầu **ngầm** (implicit) chưa nói ra.

> [!warning] **Yêu cầu mơ hồ = rủi ro lớn nhất.** "Hệ thống phải nhanh" là vô nghĩa — phải định lượng: "trang chủ tải < 2 giây với 1000 user đồng thời". Yêu cầu tốt phải: **rõ ràng, đo được, khả thi, không mâu thuẫn, kiểm thử được**.

---

## 3. Bước 3: Rã scope (xác định phạm vi) ⭐

**Scope** = ranh giới dự án: cái gì **làm** và cái gì **không làm**. Rã scope = chia mục tiêu lớn thành các phần việc rõ ràng & vạch ranh giới.

### In-scope vs Out-of-scope

```text
┌─────────── DỰ ÁN ───────────┐
│  IN-SCOPE (làm)             │     OUT-OF-SCOPE (KHÔNG làm trong phạm vi này)
│  • Đăng ký / đăng nhập      │     • Tích hợp ví điện tử (giai đoạn 2)
│  • Đặt lịch tập             │     • App Apple Watch
│  • Thanh toán thẻ           │     • Báo cáo BI nâng cao
└─────────────────────────────┘
```

> [!tip] **Ghi rõ out-of-scope** quan trọng ngang in-scope: nó chặn **"scope creep"** (yêu cầu phình dần ngoài kiểm soát) và tránh tranh cãi "tưởng cái này có trong hợp đồng".

### WBS (Work Breakdown Structure)
Chẻ phạm vi thành cây phân cấp việc nhỏ dần đến mức ước lượng/giao được:

```text
Hệ thống quản lý phòng gym
├─ 1. Quản lý hội viên
│   ├─ 1.1 Đăng ký
│   ├─ 1.2 Đăng nhập / quên mật khẩu
│   └─ 1.3 Hồ sơ cá nhân
├─ 2. Đặt lịch tập
│   ├─ 2.1 Xem lịch lớp
│   └─ 2.2 Đăng ký lớp
└─ 3. Thanh toán
    ├─ 3.1 Thẻ tín dụng
    └─ 3.2 Thanh toán định kỳ
```

```
★ Insight ─────────────────────────────────────
• Rã scope (BA) và chẻ epic→story (Scrum) là cùng một việc ở hai mức: WBS định
  ranh giới & cấu trúc tổng; refinement chẻ tiếp tới story làm-được-trong-1-Sprint
  ([[05-Product-Backlog-Refinement]]). Một bên "vẽ bản đồ", một bên "chia chặng đi".
• "Out-of-scope" là vũ khí chống scope creep — kẻ giết deadline thầm lặng. Viết
  ra thứ KHÔNG làm cũng là một quyết định, và nó bảo vệ cả team lẫn khách.
─────────────────────────────────────────────────
```

---

## 4. Bước 4: Phân loại yêu cầu — 4 loại ⭐⭐ (rất hay hỏi)

```text
   BUSINESS requirement   "Vì sao làm?" — mục tiêu tổ chức
          │  (tăng 20% hội viên online)
          ▼
   USER requirement       "Người dùng cần làm gì?"
          │  (hội viên muốn đặt lịch qua app)
          ▼
   FUNCTIONAL requirement "Hệ thống PHẢI LÀM GÌ?"
          │  (cho phép chọn lớp, xác nhận chỗ trống, gửi email)
          ▼
   NON-FUNCTIONAL         "Hệ thống phải TỐT THẾ NÀO?"
              (tải < 2s, 1000 user đồng thời, mã hóa mật khẩu)
```

| Loại | Trả lời | Ví dụ |
|------|---------|-------|
| **Business** | Vì sao / mục tiêu kinh doanh | Tăng 20% đăng ký online trong 6 tháng |
| **User (stakeholder)** | Người dùng cần làm được gì | Hội viên muốn đặt lịch lớp qua app |
| **Functional (FR)** | Hệ thống **phải làm gì** | Hệ thống cho chọn lớp, kiểm tra chỗ trống, gửi email xác nhận |
| **Non-functional (NFR)** | Hệ thống phải **tốt thế nào** | Trang tải < 2s; hỗ trợ 1000 user; mật khẩu mã hóa bcrypt |

### Functional vs Non-functional (cặp hay nhầm nhất)

| | **Functional** | **Non-functional** |
|---|---|---|
| Mô tả | **Hành vi** — làm *gì* | **Chất lượng** — làm *tốt ra sao* |
| Ví dụ | "Cho phép đăng nhập bằng email" | "Đăng nhập phản hồi < 1s" |
| Nhóm NFR phổ biến | — | Performance, Security, Usability, Reliability, Scalability, Maintainability |

> [!tip] Mẹo nhớ NFR bằng "**-ilities**": scalab**ility**, secur**ity**, usab**ility**, reliab**ility**, maintainab**ility**, availab**ility**… NFR thường bị bỏ quên nhưng là thứ làm hệ thống "dùng được thật" — phỏng vấn rất hay khai thác.

---

## 5. Truy vết & duyệt (bước 5–6)

- **Sign-off:** khách hàng **duyệt** tài liệu yêu cầu → mốc cam kết chính thức (chống "tôi đâu có yêu cầu thế").
- **Traceability (truy vết):** mỗi requirement nối được tới design → code → test, để biết "yêu cầu này đã làm & test chưa". Chi tiết: [[13-Test-Plan-va-QA]] (traceability matrix).

---

## 6. Tự kiểm tra

1. 4 loại requirement? *(Business, User, Functional, Non-functional)*
2. Functional vs Non-functional khác gì? *(làm gì vs làm tốt thế nào)*
3. Vì sao phải ghi rõ out-of-scope? *(chống scope creep, tránh tranh cãi)*
4. WBS là gì? *(chẻ phạm vi thành cây việc nhỏ dần)*
5. Yêu cầu "hệ thống phải nhanh" sai chỗ nào? *(mơ hồ, không đo được — phải định lượng)*
6. Vài nhóm NFR? *(performance, security, usability, reliability, scalability)*

## Liên quan
- [[00-MOC-Agile-Scrum|⬅ MOC Agile-Scrum]]
- Trước: [[08-Certifications]] · Kế tiếp: [[10-Tai-lieu-Yeu-cau-va-Dac-ta|Tài liệu & đặc tả]]
- [[11-Wireframe-Mockup-Prototype]] · [[05-Product-Backlog-Refinement|Chẻ thành backlog]]
