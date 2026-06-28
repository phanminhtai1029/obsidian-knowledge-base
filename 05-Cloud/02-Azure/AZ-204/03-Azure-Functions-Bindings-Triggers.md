---
title: "Azure Functions: binding & trigger"
section: 05-Cloud/02-Azure/AZ-204
tags: [azure, az-204, functions, serverless, binding, trigger, fresher]
related:
  - "[[00-MOC-AZ-204]]"
  - "[[02-App-Service-Web-Apps]]"
difficulty: ⭐⭐⭐
estimated_time: 24m
source: ["_source/Microsoft/Az-204.docx (Lesson 3, Module 1)", "Microsoft Learn"]
status: 🚧 khung
---

# Azure Functions: binding & trigger

> [!summary] TL;DR
> *(Chưa viết.)*

> [!todo] Cần viết (Lesson 3 "Implement Azure Functions")
> - **Vì sao Functions**: serverless, event-driven, trả tiền theo lần chạy; **hosting plan** Consumption vs Premium vs Dedicated (App Service) — cold start, timeout, VNet.
> - **Trigger** = cái gì kích hoạt hàm chạy (mỗi function đúng **1 trigger**): HTTP, Timer (CRON), Blob, Queue, Event Grid/Hub, Service Bus, Cosmos DB change feed, webhook.
> - **Binding** = kết nối khai báo (declarative) tới resource, **input** (đọc) / **output** (ghi) — giảm code boilerplate; cấu hình qua `function.json` / decorator / attribute.
> - **Durable Functions** (orchestration: chaining, fan-out/in, human-interaction) — nêu khái niệm.
> - So sánh Functions vs Container Apps vs App Service (khi nào chọn serverless).
> - Liên hệ FastAPI BackgroundTasks vs hàng đợi (note Backend).

## 1. Functions & hosting plan (Consumption/Premium)

## 2. Trigger (HTTP, Timer, Blob, Queue, Event…)

## 3. Binding input/output (declarative)

## 4. Durable Functions (orchestration patterns)

## 5. Khi nào dùng Functions

## Tự kiểm tra
1. *(thêm câu hỏi)*

## Liên quan
- [[00-MOC-AZ-204]]
- [[../AI-Azure/18-Azure-App-Service-Functions-deploy]] — deploy Functions
- [[11-Event-Grid-Event-Hub]] · [[12-Service-Bus-Queue-Storage]] — nguồn trigger
