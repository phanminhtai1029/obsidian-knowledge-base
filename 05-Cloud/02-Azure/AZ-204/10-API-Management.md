---
title: "API Management: instance, products, policies"
section: 05-Cloud/02-Azure/AZ-204
tags: [azure, az-204, api-management, apim, gateway, fresher]
related:
  - "[[00-MOC-AZ-204]]"
  - "[[11-Event-Grid-Event-Hub]]"
difficulty: ⭐⭐⭐
estimated_time: 24m
source: ["_source/Microsoft/Az-204.docx (Lesson 10, Module 5)", "Microsoft Learn"]
status: 🚧 khung
---

# API Management: instance, products, policies

> [!summary] TL;DR
> *(Chưa viết.)*

> [!todo] Cần viết (Lesson 10 "API Management")
> - **APIM là gì**: facade/gateway đặt trước backend API → quản lý tập trung security, throttle, version, analytics; **3 thành phần**: API gateway, management plane, developer portal.
> - **Khái niệm chính**: **API** (tập operations), **Product** (gói nhiều API + policy + subscription), **Group** (built-in: Administrators/Developers/Guests), **Subscription** (key truy cập).
> - **Tier**: Consumption / Developer / Basic / Standard / Premium (VNet, multi-region).
> - **Authentication options** cho client: **subscription key** (header `Ocp-Apim-Subscription-Key`), **OAuth 2.0 / JWT validate**, **client certificate (mTLS)**.
> - **Policies** (cốt lõi): XML inbound/backend/outbound/on-error — rate-limit/quota, caching, transform (rewrite, set-header), CORS, `validate-jwt`, mock; policy expression.
> - Tạo & document API (import OpenAPI), versioning & revisions.

## 1. APIM tổng quan (gateway, portal, management)

## 2. API / Product / Group / Subscription

## 3. Authentication (subscription key, JWT, cert)

## 4. Policies (rate-limit, cache, transform, validate-jwt)

## 5. Tạo, document, version API

## Tự kiểm tra
1. *(thêm câu hỏi)*

## Liên quan
- [[00-MOC-AZ-204]]
- [[02-App-Service-Web-Apps]] · [[03-Azure-Functions-Bindings-Triggers]] — backend đứng sau APIM
- [[../../../02-Backend/00-MOC-Backend|MOC Backend]] — REST API, auth, rate-limit
