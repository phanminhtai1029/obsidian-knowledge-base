---
title: "Site Reliability Engineering — Building for Reliability"
section: 06-DevOps
tags: [devops, sre, reliability, resilience, slo, sla, error-budget, circuit-breaker, twelve-factor, fresher]
related:
  - "[[11-Observability-Incident-Response]]"
  - "[[09-CI-CD-Continuous-Deployment]]"
  - "[[02-Backend/16-Production-Streaming-Scaling]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 35m
source: ["DevOps Foundations — Ernest Mueller & James Wickett (LinkedIn Learning)", "Google — Site Reliability Engineering", "Michael Nygard — Release It!", "12factor.net"]
---

# SRE — Building for Reliability

> [!summary] TL;DR
> **SRE** (Site Reliability Engineering, gốc từ Google) = phần **vận hành** của DevOps: dùng **tư duy kỹ thuật phần mềm** để tự động hoá vận hành VÀ **engineer độ tin cậy ngay từ thiết kế** (không "bolt-on" sau khi live). **Reliability** = khả năng hệ thống thực hiện đúng chức năng một cách nhất quán (gồm availability, performance, security). Hai mảng: **building for reliability** (thiết kế: stability patterns như **Circuit Breaker**, **12-factor app**, chống **cascading failure** ở integration point) và **operational feedback** (observability + incident response → note 11). Triết lý cốt lõi: **mọi hệ thống đều fail** → đừng đuổi theo "hệ hoàn hảo" mà xây **resilience** (redundancy, load balancer, auto-scaling, failover). Hệ thống là **sociotechnical** (gồm cả con người). *"You write it, you run it."*

---

## 1. SRE là gì?

> [!note] Định nghĩa
> **Engineering** = áp dụng nguyên lý lý thuyết để giải bài toán thực tế. **Reliability** = khả năng hệ thống thực hiện **đúng chức năng, nhất quán, khi được kỳ vọng** — gồm availability, performance, security. **SRE** coined ở Google cho kỹ sư lo độ tin cậy website. Đây là **phần Ops thật sự của DevOps**: IaC định nghĩa hệ thống bằng code, CI/CD build & deploy, còn SRE **monitor, quản lý, sửa** khi có sự cố.

Định nghĩa "SRE = dùng software engineering để tự động hoá vận hành" là **chưa đủ**: phải **engineer độ tin cậy từ đầu** — không thể bolt-on reliability sau khi đã live. SRE cải thiện các metric DORA: change failure rate, time to restore (MTTR), khả năng đạt uptime/performance.

---

## 2. Building for Reliability — Theory (vai trò của Dev)

Phần lớn thành công ở production đến từ **quyết định lúc thiết kế** & kiến trúc phần mềm. Ba nguồn kinh điển:

| Nguồn | Đóng góp |
|---|---|
| Michael Nygard — **Release It!** | "design patterns cho sự ổn định"; **integration point là nguyên nhân lỗi số 1**; **cascading failure** là mối đe doạ số 1 trong kiến trúc nhiều lớp |
| **12-Factor App** (12factor.net) | manifesto cho app service-ready; vd Factor 3 *Config*: tách config theo deployment ra **biến môi trường**, không bundle vào code |
| **Martin Fowler** | giải thích súc tích các thuật ngữ kiến trúc (serverless, bimodal IT…) |

> [!warning] Cascading failure & Circuit Breaker
> Trong kiến trúc nhiều lớp (nhất là **microservices** với nhiều integration point), nếu một DB layer chậm/down → app server tier cạn connection pool → nghẽn cả tầng app → lan sự cố. **Circuit Breaker** theo dõi lỗi/chậm khi gọi integration point; phát hiện lỗi bất thường thì **ngừng gọi** (thay vì dội bom hệ đang chết). Kết hợp **timeout** để chặn lan outage. Thư viện: **Resilience4j**. → liên hệ [[02-Backend/16-Production-Streaming-Scaling]] (rate limiting, backpressure).

---

## 3. Building for Reliability — Practice (resilience)

> *"Dev comes from school, but Ops comes from the street."* — SRE biến kinh nghiệm vận hành thành kỹ thuật có kỷ luật.

**Sự thật phũ phàng: mọi hệ thống đều fail.** Trong hệ phức tạp, component lỗi *liên tục* mà không phải lúc nào cũng thành outage — như **lát phô mai Thuỵ Sĩ** xếp chồng: chỉ khi các lỗ thẳng hàng thì user mới gặp sự cố (Richard Cook — *How Complex Systems Fail*: "mọi hệ phức tạp luôn chạy ở chế độ degraded").

**Số "nines" của availability:**

| Uptime | "Nines" | Downtime/năm |
|---|---|---|
| 99.9% | three nines | ~8.77 giờ |
| 99.999% | five nines | ~5.26 phút |

Thay vì đổ tiền **ngăn lỗi** (trò chơi thua), hãy theo **resilience** — khả năng duy trì/khôi phục trạng thái ổn định sau sự cố:

| Kỹ thuật resilience | Tác dụng |
|---|---|
| **Redundancy** | chạy nhiều bản sao → mất một vẫn phục vụ |
| **Load balancer** | đẩy traffic về phần khoẻ mạnh |
| **Auto-scaling** | thêm server khi tải cao (không cần server to hơn) |
| **Automated failover & recovery** | tự chuyển đổi khi component chết (vd Kubernetes: health check + failover + replicate state) |

> [!tip] Hệ thống là sociotechnical — con người là một phần
> Đừng dồn hết công sức cho "uptime hoàn hảo" mà quên rằng hệ thống **luôn hỏng một phần** và cần *chuyên gia can thiệp*. SRE nên dành **ít nhất nửa thời gian viết tool** (runbook, automation) chứ không chỉ sửa tay. Và Dev cũng là chuyên gia về code của họ → **on-call** hỗ trợ khi code lỗi: *"You write it, you run it."* (Google: dev tự support tới khi thuyết phục được SRE rằng dịch vụ đủ ổn định & có đủ monitoring).

> [!question] Phỏng vấn: "SRE là gì? Khác gì 'tự động hoá vận hành'?"
> SRE (gốc Google) là **phần vận hành của DevOps**: dùng tư duy kỹ thuật phần mềm để vận hành dịch vụ với độ observability & automation cao. Nhưng nói "SRE = tự động hoá Ops" là **thiếu** — cốt lõi là **engineer độ tin cậy ngay từ thiết kế** (stability patterns, 12-factor, resilience), không thể gắn reliability vào sau khi đã live. SRE cũng nhấn: mọi hệ đều fail → xây **resilience** thay vì đuổi theo "không bao giờ lỗi"; và **dev on-call** cho code của mình.

> [!question] Phỏng vấn: "Cascading failure là gì? Circuit Breaker chống nó thế nào?"
> **Cascading failure**: lỗi/chậm ở một **integration point** (vd DB) lan ngược làm cạn tài nguyên (connection pool) ở tầng trên rồi sập cả chuỗi — nguy hiểm nhất trong kiến trúc nhiều lớp/microservices. **Circuit Breaker** theo dõi tỷ lệ lỗi/chậm khi gọi một dependency; khi vượt ngưỡng nó **"mở mạch" — ngừng gọi** trong một khoảng, trả lỗi/fallback nhanh thay vì dội bom hệ đang chết, cho nó thời gian hồi phục. Kết hợp **timeout** để không treo chờ. (Thư viện: Resilience4j.)

```
★ Insight ─────────────────────────────────────
• Chuyển từ "ngăn mọi lỗi" sang "resilience" là cú lật tư duy của SRE: chấp nhận
  hệ luôn degraded, đầu tư vào PHỤC HỒI NHANH (MTTR) thay vì theo đuổi uptime tuyệt
  đối tốn kém.
• "Reliability phải engineer từ thiết kế" nối SRE ngược về Dev: Circuit Breaker,
  12-factor, timeout là quyết định DESIGN-TIME, không phải vá lúc cháy nhà.
• Sociotechnical: con người là component của hệ thống resilience — đó là vì sao
  incident response & blameless postmortem (note 11) thuộc về SRE, không phải phụ
  lục.
─────────────────────────────────────────────────
```

---

## 4. SLI / SLO / SLA & Error Budget

Bộ khái niệm trung tâm của SRE (Google) để **lượng hoá** độ tin cậy:

| Thuật ngữ | Nghĩa | Ví dụ |
|---|---|---|
| **SLI** (Indicator) | chỉ số **đo được** về chất lượng dịch vụ | % request < 300ms; tỷ lệ request thành công |
| **SLO** (Objective) | **mục tiêu nội bộ** cho SLI | 99.9% request thành công trong 30 ngày |
| **SLA** (Agreement) | **cam kết với khách** kèm hậu quả (bồi thường) nếu vi phạm | thường *lỏng hơn* SLO để có biên an toàn |
| **Error budget** | phần "được phép lỗi" = 100% − SLO | SLO 99.9% → budget 0.1% downtime; còn budget thì cứ deploy/đổi mới, hết budget thì *đóng băng* để ổn định |

> **Toil** = công việc vận hành thủ công, lặp lại, không tạo giá trị lâu dài → SRE tìm cách **tự động hoá** nó (đây là một dạng muda → [[05-Agile-Lean]]).

---

## 5. Tự kiểm tra

1. SRE là gì? Vì sao "tự động hoá Ops" là định nghĩa chưa đủ?
2. Integration point & cascading failure nguy hiểm ra sao? Circuit Breaker giúp gì?
3. "Mọi hệ thống đều fail" dẫn tới chiến lược resilience nào (kể 3 kỹ thuật)?
4. Phân biệt SLI / SLO / SLA. Error budget dùng để làm gì?
5. "You write it, you run it" & sociotechnical system nghĩa là gì?

---

## 6. Liên quan
- [[11-Observability-Incident-Response]] — operational feedback (nửa còn lại của SRE)
- [[09-CI-CD-Continuous-Deployment]] — deploy an toàn, giảm change failure
- [[02-Backend/16-Production-Streaming-Scaling]] — rate limiting, backpressure, observability
- [[07-IaC-Concepts]] — immutable infra hỗ trợ recovery/failover
- [[00-MOC-DevOps|MOC: DevOps]]
