---
title: "Bảo mật, giám sát & vận hành dịch vụ AI"
section: 05-Cloud/02-Azure/AI-102
tags: [azure, ai-102, security, monitoring, managed-identity, fresher]
related:
  - "[[00-MOC-AI-102]]"
  - "[[../AZ-900/10-Identity-Security-AzureAD-RBAC]]"
  - "[[../AZ-900/14-Monitoring-Advisor-Monitor]]"
difficulty: ⭐⭐⭐
estimated_time: 22m
source: ["_source/Microsoft/AI-102.docx (Lesson 3, 19, Tim Warner)", "Microsoft Learn"]
status: 🚧 khung
---

# Bảo mật, giám sát & vận hành dịch vụ AI

> [!summary] TL;DR
> *(Chưa viết.)*

> [!todo] Cần viết (Lesson 3 "Manage and Secure AI Solutions" + Lesson 19 "Monitor and Optimize AI Solutions")
> - **Xác thực**: API key vs **Entra ID + Managed Identity** (vì sao managed identity an toàn hơn — không lộ key), key rotation, Key Vault.
> - **Mạng**: public endpoint vs **private endpoint / VNet integration**, network ACL, firewall.
> - **Phân quyền**: RBAC role cho Cognitive Services (Cognitive Services User/Contributor) → liên kết [[../AZ-900/10-Identity-Security-AzureAD-RBAC]].
> - **Giám sát**: Azure Monitor, diagnostic logs, metrics, alert; chi phí & quota/rate-limit (429), throttling.
> - **Tối ưu**: chọn tier/region, caching, batch, autoscale; chi phí token (với OpenAI).

## 1. Xác thực: API key vs Entra ID / Managed Identity

## 2. Bảo mật mạng (private endpoint, firewall)

## 3. RBAC cho dịch vụ AI

## 4. Giám sát & alert (Azure Monitor, diagnostic logs)

## 5. Tối ưu chi phí & hiệu năng (quota, 429, caching)

## Tự kiểm tra
1. *(thêm câu hỏi)*

## Liên quan
- [[00-MOC-AI-102]]
- [[02-Bao-mat-Giam-sat-Van-hanh-AI]]
