---
title: "Caching: Azure Cache for Redis & CDN"
section: 05-Cloud/02-Azure/AZ-204
tags: [azure, az-204, redis, cache, cdn, performance, fresher]
related:
  - "[[00-MOC-AZ-204]]"
  - "[[09-Application-Insights]]"
difficulty: ⭐⭐⭐
estimated_time: 22m
source: ["_source/Microsoft/Az-204.docx (Lesson 8, Module 4)", "Microsoft Learn"]
status: 🚧 khung
---

# Caching: Azure Cache for Redis & CDN

> [!summary] TL;DR
> *(Chưa viết.)*

> [!todo] Cần viết (Lesson 8 "Implement caching for solutions")
> - **Vì sao cache**: giảm latency, giảm tải DB/backend, tăng throughput.
> - **Azure Cache for Redis**: in-memory data store; tier Basic/Standard/Premium/Enterprise; dùng làm cache, session store, pub/sub, distributed lock.
> - **Cache patterns**: **cache-aside (lazy loading)**, write-through, read-through; **expiration / TTL**; eviction policy; **invalidation** (vấn đề khó nhất).
> - **Cấu hình tối ưu**: data sizing, connection pooling (đừng tạo connection mỗi request), encryption in-transit, clustering, key naming.
> - **Azure CDN**: cache nội dung tĩnh ở edge gần user; **endpoint & profile**; caching rules, TTL, purge; origin = Blob/App Service; so với Front Door.
> - Liên hệ connection pool (Backend) & cache trong RAG.

## 1. Vì sao cache + Azure Cache for Redis

## 2. Cache patterns (cache-aside, TTL, invalidation)

## 3. Cấu hình tối ưu Redis (sizing, connection)

## 4. Azure CDN (endpoint, profile, edge)

## Tự kiểm tra
1. *(thêm câu hỏi)*

## Liên quan
- [[00-MOC-AZ-204]]
- [[../AZ-900/08-Networking-VNet-VPN-ExpressRoute]] — CDN/edge ở góc mạng
- [[../AI-Azure/17-Azure-AI-Search]] — caching trong pipeline RAG
