---
title: "App Service Web Apps: deploy, config, autoscale"
section: 05-Cloud/02-Azure/AZ-204
tags: [azure, az-204, app-service, deploy, autoscale, fresher]
related:
  - "[[00-MOC-AZ-204]]"
  - "[[03-Azure-Functions-Bindings-Triggers]]"
difficulty: ⭐⭐⭐
estimated_time: 24m
source: ["_source/Microsoft/Az-204.docx (Lesson 2, Module 1)", "Microsoft Learn"]
status: 🚧 khung
---

# App Service Web Apps: deploy, config, autoscale

> [!summary] TL;DR
> *(Chưa viết.)*

> [!todo] Cần viết (Lesson 2 "Develop App Service web apps")
> - **App Service Web App**: PaaS host ứng dụng HTTP (web/API/mobile backend); **App Service Plan** (tier F1/B/S/P, quyết định compute + tính năng); chạy được .NET/Node/Python/Java/PHP + custom container.
> - **Deploy code**: ZIP deploy, `az webapp up`, Git/GitHub Actions, container; **deployment slots** (staging → swap blue-green, warm-up, rollback).
> - **Cấu hình**: App settings = biến môi trường; **connection strings**; **TLS/SSL** (custom domain, cert, HTTPS-only); CORS; API settings.
> - **Diagnostics logging**: app log, web server log, failed request tracing, **Log stream**; App Insights tích hợp.
> - **Autoscaling**: scale up (vertical, đổi tier) vs scale out (horizontal, +instance); **autoscale rule** theo metric (CPU/queue) + scheduled; so với Container Apps/Functions.

## 1. App Service & App Service Plan (tier)

## 2. Deploy code & deployment slots (swap)

## 3. App settings, connection strings, TLS

## 4. Diagnostics logging & log stream

## 5. Autoscaling (scale up vs scale out)

## Tự kiểm tra
1. *(thêm câu hỏi)*

## Liên quan
- [[00-MOC-AZ-204]]
- [[../AI-Azure/18-Azure-App-Service-Functions-deploy]] — deploy FastAPI lên App Service/Functions
- [[../AZ-900/07-Compute-VM-Container-Functions]] — App Service ở góc AZ-900
