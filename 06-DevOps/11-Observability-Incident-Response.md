---
title: "Operational Feedback — Observability & Incident Response"
section: 06-DevOps
tags: [devops, sre, observability, monitoring, apm, rum, incident-response, postmortem, blameless, fresher]
related:
  - "[[10-SRE-Reliability]]"
  - "[[04-DevOps-Culture]]"
  - "[[02-Backend/16-Production-Streaming-Scaling]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 35m
source: ["DevOps Foundations — Ernest Mueller & James Wickett (LinkedIn Learning)", "Google SRE Book"]
---

# Operational Feedback — Observability & Incident Response

> [!summary] TL;DR
> Nửa thứ hai của SRE: lấy thông tin từ production để **biết hệ đang ở trạng thái nào** và **feedback về dev** (Way 2 & 3). **Observability** = mức độ suy ra được *trạng thái bên trong* từ *output bên ngoài*. Năm vùng nên đo: **synthetic/health checks**, **system & app metrics** (CPU/RAM + custom metrics), **end-user performance** (**APM** ở tầng code, **RUM** ở trình duyệt, **tracing** xuyên service), **logs**, **security monitoring**. Dùng **build-measure-learn** để xây observability vừa đủ (monitoring không dùng = waste). **Incident response**: 3 việc — troubleshooting, automation, communication; học từ ICS (incident command của lính cứu hoả). **Postmortem blameless**: không có "single root cause", **không đổ lỗi**, minh bạch — vì *nếu một người mắc lỗi mà gây outage lớn thì chính hệ thống mới là thứ tệ*.

---

## 1. Observability vs Monitoring

> [!note] Định nghĩa
> **Observability** = mức độ ta **suy ra được trạng thái nội bộ** của hệ thống từ **output bên ngoài** (metrics, logs). Nói cách khác: nhìn vào những gì đang monitor, ta có thật sự biết chuyện gì đang xảy ra không? Monitoring là *thu thập*; observability là *khả năng hiểu*.

### Năm vùng observability nên đo

| # | Vùng | Trả lời câu hỏi | Công cụ/ý |
|---|---|---|---|
| 1 | **Synthetic / health check** | "Nó có chạy không?" | hit endpoint theo lịch (không phải traffic thật) |
| 2 | **System & app metrics** | "Service/host/process có bình thường?" | system: CPU/RAM (time-series, Graphite); **custom app metrics**: thời gian một function, số login/giờ, error count |
| 3 | **End-user performance** | "Khách *thực sự* trải nghiệm gì?" | **APM** (tầng code: mỗi function/API/query mất bao lâu); **RUM** (JS tag ở trình duyệt); **tracing** (theo 1 request xuyên nhiều service) |
| 4 | **Logs** | "Cái gì / khi nào / ở đâu / liên quan ai?" | text chi tiết; dùng cho troubleshoot, audit, capacity, forensics |
| 5 | **Security monitoring** | "Có bị tấn công không?" | indicator of compromise: spike login fail, injection, IP xấu, request dị thường |

> [!tip] APM vs RUM
> **APM** (Application Performance Monitoring) = instrument **tầng code**, cho biết mỗi function/API call/DB query mất bao lâu. **RUM** (Real User Monitoring) = instrument **front-end** (JS page tag), đo đúng cái **user thật** trải nghiệm. Kết hợp với **tracing** để theo một request lan qua nhiều microservice. → liên hệ observability LLM: [[04-AI/03-LLMOps-Evaluation/00-MOC-LLMOps-Evaluation]].

### Xây observability theo build-measure-learn

Đừng đoán cần gì; dùng Lean: **build** stack monitoring tối thiểu cho cả 5 vùng → **measure** (bắt đầu thu metric) → **learn** (xem chỗ nào cần sâu hơn) → lặp. *Monitoring không dùng là waste* (có team phát hiện 30% tải hệ thống đến từ chính monitoring của họ!). Và **Sharing**: dev dùng error/perf để cải thiện app, product manager cần biết để ra quyết định.

---

## 2. Incident Response

Dù thiết kế/test/monitor tốt đến đâu, **vẫn sẽ hỏng**. Giỏi xử lý sự cố (incident) là một phần công việc. Ba năng lực:

| Năng lực | Nội dung |
|---|---|
| **Troubleshooting** | hiểu hệ đủ sâu để chẩn đoán & khắc phục |
| **Automation** | có sẵn tool để thu thập thông tin & remediate **nhanh, an toàn** |
| **Communication** | điều phối nhóm chuyên gia + báo stakeholder & end user |

> [!note] ICS — học từ lính cứu hoả
> **Incident Command System** (1968, điều phối cháy rừng California; nay là chuẩn quốc tế được Liên Hợp Quốc khuyến nghị) dạy cách xử lý khẩn cấp lớn, nhiều bên. IT incident management thừa hưởng nhiều từ ICS: định nghĩa rõ **cách phát hiện, báo cáo, phối hợp** → sửa nhanh thay vì tạo thêm hỗn loạn. Không có "một cách đúng duy nhất" — process tuỳ tổ chức/team/product.

---

## 3. Postmortem (Incident Retrospective) — Blameless

Sửa xong **không** đi tiếp — failure là **cơ hội học** (Way 3). Postmortem = quy trình chính thức phân tích sự cố để học.

> [!warning] "Root cause analysis" truyền thống là cái bẫy
> RCA kiểu cũ thường là *màn tìm người để đổ lỗi*. Nhưng: **nếu một người mắc lỗi mà gây outage lớn, thì chính HỆ THỐNG của bạn mới tệ & thiếu resilience** — cần cải thiện. Ai cũng mắc lỗi; việc của kỹ sư là làm hệ thống *vẫn chạy được dù vậy*.

Ba nguyên tắc postmortem hiệu quả:

| # | Nguyên tắc | Vì sao |
|---|---|---|
| 1 | **Không có single root cause** | một change *kích hoạt* outage, nhưng thường có thiếu sót ở nhiều tầng (test, monitoring, process) cho phép nó thành sự cố — xét tất cả |
| 2 | **Bỏ tư duy đổ lỗi (blameless)** | người ta hành động vì *nghĩ* đó là đúng; bỏ cognitive bias, nhìn từ góc người vận hành để hiểu *cách* sai xảy ra |
| 3 | **Minh bạch** | trong & sau sự cố, truyền thông thật với stakeholder/user → xây niềm tin (người ta biết khi bị giấu/nói dối) |

> [!question] Phỏng vấn: "Observability gồm những gì? APM khác RUM thế nào?"
> Observability = khả năng **suy ra trạng thái nội bộ từ output bên ngoài**. Năm vùng đo: synthetic/health check, system & app metrics (gồm custom metrics), end-user performance, logs, security monitoring. **APM** instrument **tầng code** (function/API/query mất bao lâu); **RUM** instrument **front-end/trình duyệt**, đo trải nghiệm **user thật**. Kết hợp **tracing** để theo một request xuyên nhiều service. Nguyên tắc: xây vừa đủ theo build-measure-learn — *monitoring không dùng là waste*.

> [!question] Phỏng vấn: "Blameless postmortem là gì? Vì sao không truy 'root cause' để quy trách nhiệm?"
> Blameless postmortem là phân tích sự cố **không đổ lỗi cá nhân**, tập trung học & cải thiện hệ thống. Lý do bỏ "single root cause + quy trách nhiệm": nếu một sai sót của *một người* đủ gây outage lớn thì **hệ thống thiếu resilience** mới là vấn đề thật; hơn nữa hiếm khi chỉ có một nguyên nhân — thường có lỗ hổng ở test, monitoring, process cùng cho phép sự cố xảy ra. Đổ lỗi còn phá **Westrum generative culture**: người ta sẽ giấu tin xấu → mất feedback loop. → [[04-DevOps-Culture]].

```
★ Insight ─────────────────────────────────────
• Observability là Measurement (CAMS) ở tầng vận hành — và là nhiên liệu cho cả
  ba Ways: thấy flow, đóng feedback loop, cung cấp dữ liệu để học.
• Blameless không phải "tử tế cho vui": nó là điều kiện KỸ THUẬT để có dữ liệu sự
  cố trung thực. Văn hoá đổ lỗi = mù thông tin = hệ kém tin cậy hơn.
• "Monitoring không dùng là waste" áp Lean vào observability: đừng dựng dashboard
  cho đẹp; dựng cái giúp ra quyết định, đo, rồi mới đào sâu chỗ cần.
─────────────────────────────────────────────────
```

---

## 4. Tự kiểm tra

1. Observability khác monitoring thế nào? Kể 5 vùng observability.
2. APM vs RUM vs tracing — mỗi cái đo gì?
3. Vì sao nên xây observability theo build-measure-learn?
4. Ba năng lực cốt lõi của incident response? ICS là gì?
5. Ba nguyên tắc của blameless postmortem? Vì sao "single root cause" là sai lầm?

---

## 5. Liên quan
- [[10-SRE-Reliability]] — building for reliability (nửa đầu của SRE)
- [[04-DevOps-Culture]] — Westrum generative, học từ thất bại (nền của blameless)
- [[02-Backend/16-Production-Streaming-Scaling]] — observability/metrics trong app thật
- [[04-AI/03-LLMOps-Evaluation/00-MOC-LLMOps-Evaluation]] — observability cho LLM
- [[02-CAMS-CALMS-Values]] — Measurement (chữ M)
- [[00-MOC-DevOps|MOC: DevOps]]
