---
title: "Văn hoá DevOps — Communication, Collaboration, Continuous Learning"
section: 06-DevOps
tags: [devops, culture, communication, collaboration, conway, westrum, kaizen, gemba, fresher]
related:
  - "[[03-The-Three-Ways]]"
  - "[[02-CAMS-CALMS-Values]]"
  - "[[05-Agile-Lean]]"
difficulty: ⭐⭐⭐
estimated_time: 30m
source: ["DevOps Foundations — Ernest Mueller & James Wickett (LinkedIn Learning)", "Ron Westrum — organizational culture model", "Masaaki Imai — Kaizen"]
---

# Văn hoá DevOps

> [!summary] TL;DR
> Văn hoá là phần khó nhất nhưng quan trọng nhất của DevOps (chữ **C** đứng đầu CAMS). Ba trụ: **(1) Communication & trust** — thông tin chảy tốt cần *kênh, thời điểm, chuẩn* rõ ràng; mô hình **Westrum** chia tổ chức thành *pathological / bureaucratic / generative*, generative (tập trung vào sứ mệnh) cho information flow tốt nhất; xây niềm tin bằng minh bạch, *assume good faith*. **(2) Collaboration** — phá silo bằng **cross-functional team**, self-service, và căn chỉnh mục tiêu; **Conway's Law**: hệ thống bạn thiết kế sẽ phản chiếu ranh giới giao tiếp của tổ chức. **(3) Continuous learning** — **kaizen** (cải tiến liên tục), **gemba** (đến tận nơi thật), vòng **PDCA** (plan-do-check-act).

---

## 1. Communication & Trust

Giao tiếp tốt **không tự xảy ra** — cần kế hoạch: có **kênh**, **thời điểm**, và **chuẩn published** (repo nào chứa thông tin khách hàng, channel nào báo downtime, email alias nào cho release). Minh bạch không chỉ truyền thông tin mà còn **xây niềm tin**.

**Mô hình Westrum** (nhà xã hội học Ron Westrum) — 3 kiểu tổ chức theo cách thông tin chảy:

| Kiểu tổ chức | Đặc điểm | Information flow |
|---|---|---|
| **Pathological** (bệnh lý) | ai cũng lo cho mình; tin xấu bị giấu/đổ lỗi | Kém |
| **Bureaucratic** (quan liêu) | bảo vệ địa bàn, vai trò cứng nhắc | Trung bình |
| **Generative** (sinh sản) | tập trung vào **sứ mệnh chung** | **Tốt nhất** |

Tổ chức **generative** đón nhận tin xấu, người báo tin được celebrate, dùng để học & cải thiện → an toàn, hiệu suất, năng suất cao hơn (đúng trong y tế, hàng không, tech).

> [!tip] Assume good faith — chống hiểu lầm
> Rào cản lớn nhất với niềm tin là **thiếu ngữ cảnh** gây hiểu lầm; rất hiếm khi ai đó hành động vì ác ý. Hãy **giả định thiện chí** khi thiếu thông tin, và cho người khác thấy ngữ cảnh của bạn (cho họ vào chat, wiki, code, monitoring, ticket tracker).

---

## 2. Collaboration — phá silo

Gốc của wall of confusion **không** phải kỹ năng con người kém, mà do **tổ chức tạo động lực đối nghịch** (Dev: tốc độ; Ops: ổn định) → local optimization.

> [!note] Conway's Law
> *"Hệ thống bạn thiết kế sẽ luôn căn theo ranh giới giao tiếp của bạn."* Ranh giới tổ chức là một loại ranh giới giao tiếp. ⇒ Muốn hệ thống căn theo **value stream** thì cấu trúc tổ chức cũng phải vậy. Chỉ **đổi tên** một team thành "DevOps" không thay đổi cấu trúc giao tiếp → không cải thiện gì.

Ba bước tăng cộng tác:

1. **Giảm số team trong value chain** — lập **cross-functional team** (vd nhúng một ops engineer vào team dev, mọi task ops & dev vào chung backlog).
2. **Self-service** — biến team khác thành "ảo": tự lấy tài nguyên (vd portal tự tạo cloud server) thay vì log ticket chờ đợi.
3. **Căn chỉnh mục tiêu** ở các team còn lại — Dev chịu trách nhiệm build/deploy & on-call; Ops/QA chuyển sang cung cấp **platform self-service** thay vì làm hộ.

---

## 3. Continuous Learning — Kaizen, Gemba, PDCA

Hiện thực hoá **Way 3**. Nhiều thuật ngữ Nhật vì Lean ăn sâu ở Nhật:

| Thuật ngữ | Nghĩa | Áp dụng |
|---|---|---|
| **Kaizen** | "thay đổi để tốt hơn" = cải tiến liên tục | thay đổi nhỏ mỗi ngày → kết quả lớn theo thời gian |
| **Gemba** | "nơi thật" | đến tận chỗ tạo giá trị / nơi có vấn đề mà *xem*, không dựa báo cáo. *"Gọi bạn về một bệnh nhân → đi xem bệnh nhân"* (Cook's Rule #6) |
| **PDCA** (kata) | Plan-Do-Check-Act | một dạng phương pháp khoa học tactical, làm nhỏ mỗi ngày, baseline mới nếu tốt hơn |

5 nguyên lý kaizen: biết khách hàng · workflow mượt · **gemba** (đến nơi thật) · trao quyền cho người · minh bạch. Toyota gọi đây là *"building people before building cars"* — PDCA dạy tư duy phản biện, không chỉ tạo cải tiến.

> [!question] Phỏng vấn: "Conway's Law là gì và liên quan gì tới DevOps?"
> Conway's Law nói **kiến trúc hệ thống sẽ phản chiếu cấu trúc giao tiếp của tổ chức** tạo ra nó. Hệ quả cho DevOps: nếu tổ chức chia silo Dev/Ops/QA thì hệ thống & quy trình cũng bị chia cắt theo. Muốn hệ thống căn theo value stream, phải tổ chức con người theo value stream (cross-functional team). Đây là lý do **chỉ đổi tên team thành "DevOps" mà không đổi cấu trúc giao tiếp thì không cải thiện gì**.

> [!question] Phỏng vấn: "Gemba nghĩa là gì? Vì sao quan trọng?"
> **Gemba** = "nơi thật" — kaizen yêu cầu **đến tận nơi** tạo giá trị hoặc nơi có vấn đề để *xem trực tiếp*, không dựa vào báo cáo/metric/giả định. Vào họp dự án, đọc code, tự dùng hệ thống đang lỗi. Nó đảm bảo quyết định dựa trên thực tế chứ không phải thông tin tay hai — cốt lõi của văn hoá học hỏi (Way 3).

```
★ Insight ─────────────────────────────────────
• "Đổi tên Ops thành DevOps" là antipattern hàng đầu: Conway's Law nói cấu trúc
  giao tiếp (không phải cái tên) mới quyết định kết quả hệ thống.
• Self-service là vũ khí mở rộng (scale) văn hoá: nó "xoá" team khác khỏi đường
  găng (critical path) mà không cần gộp mọi người vào một team khổng lồ.
• Westrum generative + blameless là tiền đề cho mọi thực hành SRE sau này: nếu
  tin xấu bị trừng phạt, sẽ không ai báo cáo sự cố sớm → mất feedback loop (Way 2).
─────────────────────────────────────────────────
```

---

## 4. Tự kiểm tra

1. Ba kiểu tổ chức của Westrum là gì? Kiểu nào có information flow tốt nhất?
2. Phát biểu Conway's Law. Vì sao "đổi tên team thành DevOps" không đủ?
3. Kể 3 bước tăng cộng tác (phá silo).
4. Kaizen, gemba, PDCA — mỗi cái nghĩa là gì?
5. "Assume good faith" giúp ích gì cho việc xây niềm tin?

---

## 5. Liên quan
- [[03-The-Three-Ways]] — Way 3 (learning) mà chương này hiện thực hoá
- [[02-CAMS-CALMS-Values]] — Culture & Sharing trong CAMS
- [[05-Agile-Lean]] — kaizen, value stream (process)
- [[11-Observability-Incident-Response]] — blameless postmortem (văn hoá học từ sự cố)
- [[00-Foundations/03-Agile-Scrum/01-Tong-quan-Agile-Scrum]] — Agile mindset
- [[00-MOC-DevOps|MOC: DevOps]]
