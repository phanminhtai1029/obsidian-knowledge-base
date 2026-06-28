---
title: "Message-based: Service Bus & Queue Storage"
section: 05-Cloud/02-Azure/AZ-204
tags: [azure, az-204, service-bus, queue-storage, messaging, fresher]
related:
  - "[[00-MOC-AZ-204]]"
  - "[[11-Event-Grid-Event-Hub]]"
difficulty: ⭐⭐⭐
estimated_time: 24m
source: ["_source/Microsoft/Az-204.docx (Lesson 12, Module 5)", "Microsoft Learn"]
status: 🚧 khung
---

# Message-based: Service Bus & Queue Storage

> [!summary] TL;DR
> *(Chưa viết.)*

> [!todo] Cần viết (Lesson 12 "Develop message-based solutions")
> - **Vì sao hàng đợi**: decoupling producer/consumer, load leveling (đệm spike), độ tin cậy (retry), async.
> - **Azure Service Bus** (enterprise messaging): **queue** (point-to-point) vs **topic/subscription** (pub/sub, filter rule); tính năng nâng cao — FIFO qua **sessions**, **dead-letter queue (DLQ)**, duplicate detection, scheduled message, transaction, peek-lock vs receive-and-delete; tier Standard/Premium.
> - **Azure Queue Storage**: hàng đợi đơn giản trên Storage Account; dung lượng rất lớn, rẻ; ít tính năng (không topic/session/transaction); message ≤ 64KB, TTL.
> - **Bảng chọn Service Bus vs Queue Storage**: khi nào cần tính năng giao dịch/ordering (Service Bus) vs đơn giản & dung lượng lớn (Queue Storage).
> - SDK gửi/nhận; Functions trigger; liên hệ BackgroundTasks vs message queue (Backend).

## 1. Vì sao hàng đợi (decoupling, load leveling)

## 2. Azure Service Bus (queue, topic, DLQ, session)

## 3. Azure Queue Storage (đơn giản, dung lượng lớn)

## 4. Service Bus vs Queue Storage (bảng chọn)

## Tự kiểm tra
1. *(thêm câu hỏi)*

## Liên quan
- [[00-MOC-AZ-204]]
- [[11-Event-Grid-Event-Hub]] — event-based đối chiếu
- [[../../../02-Backend/00-MOC-Backend|MOC Backend]] — async task & queue
