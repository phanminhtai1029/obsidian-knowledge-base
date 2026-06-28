---
title: "Event-based: Event Grid & Event Hub"
section: 05-Cloud/02-Azure/AZ-204
tags: [azure, az-204, event-grid, event-hub, event-driven, fresher]
related:
  - "[[00-MOC-AZ-204]]"
  - "[[12-Service-Bus-Queue-Storage]]"
difficulty: ⭐⭐⭐
estimated_time: 24m
source: ["_source/Microsoft/Az-204.docx (Lesson 11, Module 5)", "Microsoft Learn"]
status: 🚧 khung
---

# Event-based: Event Grid & Event Hub

> [!summary] TL;DR
> *(Chưa viết.)*

> [!todo] Cần viết (Lesson 11 "Develop event-based solutions")
> - **Event vs Message** (phân biệt nền tảng): event = thông báo "đã có chuyện xảy ra" (publisher không quan tâm ai xử lý), thường nhẹ; message = dữ liệu hợp đồng publisher kỳ vọng được xử lý.
> - **Azure Event Grid**: định tuyến event theo mô hình pub/sub; **topic / subscription / event handler**; built-in event từ Azure resource (Blob created…); **CloudEvents** schema; filter; retry & dead-letter; reactive.
> - **Azure Event Hub**: ingest **luồng dữ liệu lớn** (big data / telemetry / IoT), hàng triệu event/giây; **partition**, **consumer group**, offset/checkpoint; tích hợp Kafka protocol; Capture sang Blob.
> - **Bảng chọn**: Event Grid (định tuyến event rời rạc) vs Event Hub (streaming khối lượng lớn) vs Service Bus (message giao dịch) — khi nào dùng cái nào.
> - SDK gửi/nhận; Functions trigger Event Grid/Hub.

## 1. Event vs Message (khái niệm nền)

## 2. Azure Event Grid (pub/sub, topic, handler)

## 3. Azure Event Hub (streaming, partition, consumer group)

## 4. Event Grid vs Event Hub vs Service Bus (bảng chọn)

## Tự kiểm tra
1. *(thêm câu hỏi)*

## Liên quan
- [[00-MOC-AZ-204]]
- [[12-Service-Bus-Queue-Storage]] — message-based đối chiếu
- [[03-Azure-Functions-Bindings-Triggers]] — Functions trigger từ event
