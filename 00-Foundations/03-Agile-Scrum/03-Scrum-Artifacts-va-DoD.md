---
title: "Scrum Artifacts + Definition of Done"
section: 00-Foundations/03-Agile-Scrum
tags: [scrum, artifacts, product-backlog, sprint-backlog, increment, definition-of-done, foundations, fresher]
related:
  - "[[02-Scrum-Roles]]"
  - "[[04-Scrum-Events]]"
  - "[[05-Product-Backlog-Refinement]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [Scrum Guide, "Agile Practitioner with Scrum"]
---

# Scrum Artifacts + Definition of Done

> [!summary] TL;DR
> Scrum có **3 vật phẩm (artifacts)**: **Product Backlog** (danh sách *tất cả* việc cần làm cho sản phẩm, do PO sắp xếp ưu tiên) → **Sprint Backlog** (tập con việc chọn cho Sprint hiện tại **+ kế hoạch** của Dev Team) → **Increment** (phần mềm chạy được, gộp cả giá trị Sprint này + các Sprint trước). **Definition of Done (DoD)** là tiêu chí thống nhất xác định khi nào một việc được coi là "xong" — bảo đảm minh bạch.

---

## 1. Bức tranh: 3 artifacts liên kết với nhau

```text
 PRODUCT BACKLOG            SPRINT BACKLOG               INCREMENT
 (toàn bộ việc,    ──pull──▶ (việc của Sprint này   ──build──▶ (phần mềm chạy được,
  PO ưu tiên)      cho Sprint  + kế hoạch Dev Team)            đạt Definition of Done)
                                                              = giá trị Sprint này
                                                                + mọi Sprint trước
```

| Artifact | Là gì | Ai sở hữu |
|----------|-------|-----------|
| **Product Backlog** | Danh sách *mọi thứ* cần cho sản phẩm, sắp theo ưu tiên | Product Owner |
| **Sprint Backlog** | Tập con việc cho Sprint + **kế hoạch** thực hiện | Development Team |
| **Increment** | Tổng giá trị "có thể release" tới hiện tại | Development Team |

---

## 2. Product Backlog

Danh sách **tất cả** công việc cho sản phẩm. Đặc điểm:

- **Living document** (sống & tiến hóa): không bao giờ "hoàn thành", tồn tại chừng nào sản phẩm còn.
- **Một** sản phẩm = **một** Product Backlog = **một** PO quản lý.
- Thuộc tính mỗi item thường có: **title, description, order (thứ tự), estimate (ước lượng)**.
- Chứa nhiều **loại item**: user story, bug, enhancement, feature request, research (spike), cải tiến quy trình, thậm chí chỉ là ý tưởng.
- Item trên đỉnh: **fine-grained** (chi tiết, nhỏ, sẵn làm); item dưới đáy: **coarse-grained** (to, mơ hồ — vd **epic**, cần chẻ nhỏ sau).
- Nên **review định kỳ** để bỏ item sẽ không bao giờ làm.

```text
Product Backlog (đỉnh = ưu tiên cao, đã chi tiết & nhỏ):
 ▲ #1 User story: "Là user, tôi muốn xem lịch hoạt động..."   ← nhỏ, sẵn làm
 │ #2 Bug: menu hiển thị lỗi trên Safari
 │ #3 Enhancement: ...
 │ ...
 ▼ #N Epic: "Sinh các báo cáo tăng trưởng doanh thu"          ← to, cần chẻ nhỏ
```

> [!tip] **User story** thường viết: *"Là một &lt;vai trò&gt;, tôi muốn &lt;mục tiêu&gt; để &lt;lợi ích&gt;."* (As a … I want … so that …). **Epic** = story quá lớn, chưa làm ngay được, phải chẻ thành nhiều story nhỏ — xem [[05-Product-Backlog-Refinement]].

---

## 3. Sprint Backlog

Khi vào một Sprint, Dev Team cùng PO **kéo (pull)** một tập con item từ đỉnh Product Backlog. **Sprint Backlog = tập con item đó + kế hoạch của Dev Team để làm chúng.**

- **Hai thành phần:** (1) các item được chọn cho Sprint; (2) **kế hoạch** thực hiện (Dev Team tự lập theo cách họ muốn — vd tách mỗi story thành task UI/design/DB/code/test/doc).
- Dev Team **tự do thay đổi** kế hoạch trong Sprint khi học được điều mới.
- Là **bản đồ công việc** của riêng Sprint hiện tại.

> [!warning] Phân biệt: **Product Backlog** = toàn bộ, dài hạn, PO sở hữu. **Sprint Backlog** = lát cắt cho 1 Sprint + kế hoạch, **Dev Team** sở hữu. Đừng nhầm hai cái này — câu hỏi phỏng vấn rất hay.

---

## 4. Increment

**Increment** = phần phần mềm **chạy được** mà Scrum Team tạo ra. Quan trọng:

- Là **tổng cộng dồn**: giá trị Sprint hiện tại **+ tất cả** Sprint trước, gộp lại và **hoạt động cùng nhau**.
- Phải đạt **Definition of Done** → "**potentially releasable**" (sẵn sàng để release).
- PO **có thể release ngay hoặc chưa**, tùy quyết định — nhưng nó phải đủ tốt để release được.

> Ví dụ Agile Fitness: các Sprint trước đã xây trang login, landing page, tìm địa điểm. Sprint này thêm hồ sơ huấn luyện viên, lịch sử thanh toán. **Increment** = tất cả các phần đó hoạt động cùng nhau, không chỉ phần delta mới.

```
★ Insight ─────────────────────────────────────
• "Cộng dồn" là điểm hay quên: Increment KHÔNG phải chỉ phần mới của Sprint này,
  mà là TOÀN BỘ sản phẩm tích lũy tới giờ và phần mới phải tích hợp êm với phần
  cũ. Đây là vì sao mỗi Sprint phải cho ra "lát cắt dọc" dùng được, không phải
  mảnh rời rạc (nối sang định nghĩa Sprint ở [[04-Scrum-Events]]).
─────────────────────────────────────────────────
```

---

## 5. Definition of Done (DoD) ⭐

**DoD** = bộ tiêu chí **thống nhất, ai cũng hiểu giống nhau**, xác định khi nào một artifact (thường là một backlog item) được coi là "**xong**".

### Tính chất

- Áp dụng cho: **product backlog item**, hoặc **increment**, hoặc **release**.
- Thường do **tổ chức** định nghĩa (áp cho cả công ty). Nếu tổ chức chưa có, **team** tự định nghĩa.
- Nhiều team có DoD riêng nhưng **không được mâu thuẫn** nhau; gộp lại vẫn phải "potentially releasable".
- Nên **bắt đầu đơn giản**, rồi **siết chặt dần** khi quy trình trưởng thành.
- Tăng **transparency** (một trong 3 trụ cột → [[01-Tong-quan-Agile-Scrum]]): mọi người hiểu tiến độ theo cùng một chuẩn.

### Ví dụ DoD (cấp user story) của Agile Fitness

```text
☑ Unit test viết & pass
☑ Behavior-driven test (BDD) pass
☑ Code coverage đạt ngưỡng
☑ Product Owner review & chấp nhận
☑ Class documentation đầy đủ
☑ Code analysis (lint/quality) không lỗi
```

> [!warning] **DoD vs Acceptance Criteria** (rất hay nhầm):
> - **Definition of Done**: chuẩn **chung**, áp cho **mọi** item (vd "phải có test, phải pass CI").
> - **Acceptance Criteria**: điều kiện **riêng** của **một** story cụ thể (vd "user nhập sai mật khẩu 3 lần thì khóa 5 phút").
> → DoD = chất lượng chung; AC = đặc tả hành vi của từng story. Chi tiết AC ở [[10-Tai-lieu-Yeu-cau-va-Dac-ta]].

```
★ Insight ─────────────────────────────────────
• DoD là "hợp đồng minh bạch" chống bệnh "90% done": khi chưa có DoD rõ, mỗi
  người hiểu "xong" một kiểu → review cãi nhau, tiến độ ảo. DoD biến "xong" thành
  một checklist khách quan ai cũng kiểm được → đúng tinh thần Transparency.
• DoD nên siết dần, không cầu toàn từ đầu: đặt DoD quá khắt khe lúc team còn non
  sẽ chẳng item nào "done". Bắt đầu vừa sức rồi nâng chuẩn khi trưởng thành.
─────────────────────────────────────────────────
```

---

## 6. Bảng tổng 3 artifacts

| | Product Backlog | Sprint Backlog | Increment |
|---|---|---|---|
| Phạm vi | Toàn sản phẩm | 1 Sprint | Tích lũy tới hiện tại |
| Sở hữu | Product Owner | Dev Team | Dev Team |
| Cam kết (commitment) gắn kèm | Product Goal | Sprint Goal | Definition of Done |
| Tính chất | Sống, luôn tiến hóa | Cố định trong Sprint (kế hoạch điều chỉnh được) | Phải "releasable" |

---

## 7. Tự kiểm tra

1. 3 artifacts của Scrum? *(Product Backlog, Sprint Backlog, Increment)*
2. Product Backlog vs Sprint Backlog khác gì? *(toàn bộ/PO vs lát cắt 1 Sprint + kế hoạch/Dev Team)*
3. Increment chỉ là phần mới của Sprint? *(không — cộng dồn cả phần cũ, phải hoạt động cùng nhau)*
4. Definition of Done để làm gì? *(chuẩn "xong" thống nhất → minh bạch)*
5. DoD khác Acceptance Criteria? *(DoD chung mọi item; AC riêng từng story)*
6. Epic là gì? *(story quá lớn, phải chẻ nhỏ)*

## Liên quan
- [[00-MOC-Agile-Scrum|⬅ MOC Agile-Scrum]]
- Trước: [[02-Scrum-Roles]] · Kế tiếp: [[04-Scrum-Events|Scrum Events]]
- [[05-Product-Backlog-Refinement]] · [[10-Tai-lieu-Yeu-cau-va-Dac-ta|Acceptance Criteria]]
