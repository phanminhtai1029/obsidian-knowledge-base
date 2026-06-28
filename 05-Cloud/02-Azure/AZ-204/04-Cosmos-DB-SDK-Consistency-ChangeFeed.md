---
title: "Cosmos DB: SDK, consistency level, change feed"
section: 05-Cloud/02-Azure/AZ-204
tags: [azure, az-204, cosmos-db, nosql, consistency, change-feed, fresher]
related:
  - "[[00-MOC-AZ-204]]"
  - "[[05-Blob-Storage-SDK-Lifecycle]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 26m
source: ["_source/Microsoft/Az-204.docx (Lesson 4, Module 2)", "Microsoft Learn"]
status: 🚧 khung
---

# Cosmos DB: SDK, consistency level, change feed

> [!summary] TL;DR
> *(Chưa viết.)*

> [!todo] Cần viết (Lesson 4 "Develop solutions that use Azure Cosmos DB")
> - **Cosmos DB là gì**: NoSQL phân tán toàn cầu, multi-model (Core/SQL API, MongoDB, Cassandra, Gremlin, Table); **RU/s (Request Units)** = đơn vị throughput; provisioned vs serverless vs autoscale.
> - **Phân cấp**: Account → Database → **Container** → Item; **partition key** (chọn sao cho phân tán đều, tránh hot partition).
> - **SDK operations**: CRUD item, query SQL, point read vs query, `CreateItem/ReadItem/UpsertItem`; pagination (continuation token).
> - **5 consistency levels**: Strong → Bounded Staleness → Session (mặc định) → Consistent Prefix → Eventual — đánh đổi latency/availability/độ mới; bảng phân biệt.
> - **Change feed**: log thay đổi theo thứ tự → trigger xử lý (Functions Cosmos trigger, change feed processor); dùng cho event sourcing, materialized view, đồng bộ.

## 1. Cosmos DB tổng quan + RU/s + multi-model

## 2. Phân cấp & partition key

## 3. SDK: CRUD & query

## 4. 5 consistency levels (bảng đánh đổi)

## 5. Change feed (xử lý sự kiện thay đổi)

## Tự kiểm tra
1. *(thêm câu hỏi)*

## Liên quan
- [[00-MOC-AZ-204]]
- [[../../../03-Database/00-MOC-Database|MOC Database]] — đối chiếu SQL vs NoSQL
- [[03-Azure-Functions-Bindings-Triggers]] — Cosmos change feed trigger
