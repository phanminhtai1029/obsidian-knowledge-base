---
title: "Test Plan & QA — kế hoạch kiểm thử & đảm bảo chất lượng"
section: 00-Foundations/03-Agile-Scrum
tags: [test-plan, test-case, qa, qc, test-levels, uat, traceability, foundations, fresher]
related:
  - "[[10-Tai-lieu-Yeu-cau-va-Dac-ta]]"
  - "[[03-Scrum-Artifacts-va-DoD]]"
  - "[[09-Tiep-nhan-va-Phan-tich-Yeu-cau]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 40m
source: [ISTQB, IEEE 829, tổng hợp thực hành]
---

# Test Plan & QA — kế hoạch kiểm thử & đảm bảo chất lượng

> [!summary] TL;DR
> **Test Plan** = tài liệu **chiến lược** kiểm thử (phạm vi, cách tiếp cận, tài nguyên, lịch, tiêu chí pass/fail) — "test cái gì, thế nào, ai làm, khi nào". **Test Case** = một kịch bản kiểm thử **cụ thể** (bước, dữ liệu, kết quả mong đợi). **4 cấp test (test levels)**: **Unit → Integration → System → Acceptance (UAT)**. **QA vs QC**: QA phòng ngừa (quy trình), QC phát hiện (kiểm sản phẩm). **Acceptance Criteria → test**; **Traceability matrix** nối requirement ↔ test để biết đã phủ hết chưa. Test phải thỏa **Definition of Done**.

---

## 1. Test Plan vs Test Case ⭐ (hay nhầm)

| | **Test Plan** | **Test Case** |
|---|---|---|
| Cấp độ | **Chiến lược** (tổng thể) | **Chi tiết** (1 kịch bản) |
| Trả lời | Test *cái gì, thế nào, ai, khi nào, tiêu chí* | Nhập *gì*, làm *bước nào*, mong đợi *kết quả gì* |
| Số lượng | 1 (cho dự án/release) | Rất nhiều |
| Ai viết | Test Lead / QA Lead | Tester / QC |

**Test Plan gồm (IEEE 829, rút gọn):** phạm vi (in/out scope test), **cách tiếp cận** (manual/automation, loại test), **tài nguyên** (người, môi trường, công cụ), **lịch**, **tiêu chí vào/ra** (entry/exit criteria), **rủi ro**.

**Test Case mẫu:**
```text
ID: TC-LOGIN-03
Tiền điều kiện: user đã có tài khoản
Bước:  1. Mở trang login
       2. Nhập email đúng, mật khẩu SAI
       3. Bấm "Đăng nhập"
Dữ liệu: email=a@x.com, pass=wrong123
Kết quả mong đợi: hiện thông báo "Sai mật khẩu", KHÔNG đăng nhập
Kết quả thực tế: ___   Trạng thái: Pass/Fail
```

---

## 2. Bốn cấp test (Test Levels) ⭐⭐ — "kim tự tháp test"

```text
        ▲ ít, chậm, đắt
        │      ┌───────────────┐
        │      │  Acceptance    │  UAT — khách/PO xác nhận đúng nhu cầu
        │      ├───────────────┤
        │      │    System      │  test toàn hệ thống end-to-end
        │      ├───────────────┤
        │      │  Integration   │  test các module ghép với nhau
        │      ├───────────────┤
        │      │     Unit       │  test từng hàm/đơn vị nhỏ
        ▼      └───────────────┘
          nhiều, nhanh, rẻ (đáy)
```

| Cấp | Test cái gì | Ai làm thường | Ví dụ |
|-----|-------------|---------------|-------|
| **Unit** | Một hàm/lớp/đơn vị nhỏ, cô lập | Dev | Hàm `validateEmail()` trả đúng true/false |
| **Integration** | Nhiều module **ghép** với nhau | Dev / QA | API gọi DB lưu user thành công |
| **System** | **Toàn** hệ thống, end-to-end | QA | Đăng ký → nhận email → đăng nhập → đặt lịch |
| **Acceptance (UAT)** | Đúng **nhu cầu khách** chưa | Khách / PO | Khách dùng thử, xác nhận nghiệm thu |

> [!tip] **Test Pyramid:** nên **nhiều** unit test (đáy: nhanh, rẻ, chạy mỗi commit trong CI), **ít** test cấp cao (đỉnh: chậm, đắt, dễ vỡ). Ngược lại (nhiều E2E, ít unit) gọi là "ice-cream cone" — anti-pattern.

```
★ Insight ─────────────────────────────────────
• 4 cấp test khớp với chuỗi tài liệu ([[10-Tai-lieu-Yeu-cau-va-Dac-ta]]): Unit/
  Integration verify "xây ĐÚNG cách" (theo SDD/spec — verification); Acceptance/
  UAT validate "xây ĐÚNG thứ khách cần" (theo BRD/requirement — validation).
  V&V: Verification = "build the thing right", Validation = "build the right thing".
• Test pyramid là hệ quả của "shift left" + CI ([[../02-Git/12-CI-CD-la-gi]]):
  unit test rẻ nên nhét đầy vào pipeline chạy mỗi commit; E2E đắt nên ít & chạy
  thưa. Cân bằng sai (nhiều E2E) làm CI chậm & hay "đỏ giả".
─────────────────────────────────────────────────
```

---

## 3. QA vs QC vs Testing (phân biệt thuật ngữ hay hỏi)

| | **QA** (Quality Assurance) | **QC** (Quality Control) |
|---|---|---|
| Mục tiêu | **Phòng ngừa** lỗi | **Phát hiện** lỗi |
| Tập trung | **Quy trình** (làm đúng cách để ít lỗi) | **Sản phẩm** (kiểm cái đã làm ra) |
| Ví dụ | Định nghĩa quy trình review, chuẩn coding, DoD | Chạy test case, tìm bug |
| Tính chất | Chủ động, suốt vòng đời | Bị động, sau khi có sản phẩm |

→ **Testing** là một phần của **QC** (hoạt động kiểm để tìm lỗi). QA bao trùm rộng hơn (cả quy trình).

---

## 4. Các loại test (test types) — bonus phân biệt

| Loại | Kiểm gì |
|------|---------|
| **Functional test** | Chức năng làm đúng không (theo FR) |
| **Non-functional test** | Performance, **Load/Stress**, Security, Usability (theo NFR → [[09-Tiep-nhan-va-Phan-tich-Yeu-cau]]) |
| **Regression test** | Thay đổi mới có làm hỏng cái cũ không |
| **Smoke test** | Kiểm nhanh "build có chạy nổi không" trước khi test sâu |
| **Manual vs Automation** | Tay vs script tự động (automation hợp regression, CI) |
| **Black-box vs White-box** | Không biết / có biết code bên trong khi test |

---

## 5. Acceptance Criteria → Test (cầu nối requirement)

AC viết kiểu **Given-When-Then** ([[10-Tai-lieu-Yeu-cau-va-Dac-ta]]) **dịch thẳng** thành test case:

```gherkin
# Acceptance Criteria               →   # Test case
Given user nhập sai mật khẩu 3 lần      Bước: nhập sai 3 lần rồi lần 4
When nhập lần thứ 4                      Dữ liệu: pass sai
Then khóa tài khoản 5 phút              Mong đợi: tài khoản bị khóa 5 phút
```

→ AC tốt = test rõ ràng. Đây là tinh thần **BDD** (Behavior-Driven Development).

---

## 6. Traceability Matrix (ma trận truy vết) ⭐

Bảng nối **requirement ↔ design ↔ test case** để bảo đảm **mọi yêu cầu đều được phủ test** (không sót, không thừa).

| Req ID | Mô tả | Design | Test Case | Trạng thái |
|--------|-------|--------|-----------|-----------|
| FR-01 | Đăng nhập email | SDD-§3.1 | TC-LOGIN-01..05 | ✅ Pass |
| FR-02 | Khóa sau 3 lần sai | SDD-§3.2 | TC-LOGIN-06 | ✅ Pass |
| NFR-01 | Login < 1s | SDD-§5 | TC-PERF-02 | ⚠️ chưa test |

```
★ Insight ─────────────────────────────────────
• Traceability matrix là "lưới an toàn" chống rớt yêu cầu: nó trả lời được hai
  câu sống còn — "yêu cầu này đã có test chưa?" (coverage) và "test này phục vụ
  yêu cầu nào?" (không test thừa). Khi requirement đổi, matrix chỉ ngay test nào
  cần cập nhật → nối thẳng với traceability ở [[09-Tiep-nhan-va-Phan-tich-Yeu-cau]].
─────────────────────────────────────────────────
```

---

## 7. Test trong Scrum & Definition of Done

- Test **không** là "giai đoạn cuối" — trong Scrum, test nằm **trong** mỗi Sprint, là một phần của **Definition of Done** ([[03-Scrum-Artifacts-va-DoD]]): item chưa test xong = chưa "done".
- Dev viết unit test (thường gắn CI — chạy tự động mỗi push, [[../02-Git/13-GitHub-Actions]]).
- **UAT** thường diễn ra quanh **Sprint Review** (khách/PO nghiệm thu Increment).
- "Shift-left testing": test càng sớm càng rẻ — viết test cùng lúc (hoặc trước) code (**TDD**).

---

## 8. Tự kiểm tra

1. Test Plan vs Test Case khác gì? *(plan: chiến lược tổng; case: 1 kịch bản chi tiết)*
2. 4 cấp test từ thấp lên cao? *(Unit → Integration → System → Acceptance/UAT)*
3. QA vs QC? *(QA phòng ngừa/quy trình; QC phát hiện/sản phẩm)*
4. Verification vs Validation? *(build the thing right vs build the right thing)*
5. Traceability matrix để làm gì? *(nối requirement ↔ test, đảm bảo phủ hết)*
6. Test nằm ở đâu trong Scrum? *(trong mỗi Sprint, là phần của Definition of Done)*
7. Test Pyramid khuyên gì? *(nhiều unit ở đáy, ít E2E ở đỉnh)*

## Liên quan
- [[00-MOC-Agile-Scrum|⬅ MOC Agile-Scrum]]
- Trước: [[12-Estimation]] · Quay lại: [[00-MOC-Agile-Scrum|MOC]]
- [[03-Scrum-Artifacts-va-DoD|Definition of Done]] · [[10-Tai-lieu-Yeu-cau-va-Dac-ta|AC → Test]] · [[../02-Git/13-GitHub-Actions|CI tự chạy test]]
