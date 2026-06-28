---
title: "AuthN/AuthZ: Identity platform, Entra ID, MSAL, SAS, Graph"
section: 05-Cloud/02-Azure/AZ-204
tags: [azure, az-204, identity, entra-id, msal, sas, microsoft-graph, fresher]
related:
  - "[[00-MOC-AZ-204]]"
  - "[[07-Key-Vault-App-Configuration-Managed-Identity]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 26m
source: ["_source/Microsoft/Az-204.docx (Lesson 6, Module 3)", "Microsoft Learn"]
status: 🚧 khung
---

# AuthN/AuthZ: Identity platform, Entra ID, MSAL, SAS, Graph

> [!summary] TL;DR
> *(Chưa viết.)*

> [!todo] Cần viết (Lesson 6 "Implement user authentication and authorization")
> - **Microsoft Identity platform**: identity provider (OAuth 2.0 / OpenID Connect); **app registration** (client ID, tenant, redirect URI, secret/cert), scopes & permissions (delegated vs application).
> - **AuthN vs AuthZ**: xác thực (bạn là ai) vs phân quyền (bạn được làm gì) — phân biệt rõ.
> - **MSAL (Microsoft Authentication Library)**: lấy & cache token; các luồng — Authorization Code, Client Credentials (app-to-app), On-Behalf-Of, Device Code; **access token / ID token / refresh token**.
> - **Entra ID** (tên cũ Azure AD): tenant, user/group, service principal, conditional access.
> - **Shared Access Signature (SAS)**: token cấp quyền giới hạn (thời gian, quyền, IP) tới Storage — user delegation SAS vs account SAS vs service SAS; vì sao an toàn hơn account key.
> - **Microsoft Graph**: API thống nhất truy cập dữ liệu M365 (user, mail, calendar…); gọi qua SDK với token.

## 1. Microsoft Identity platform & app registration

## 2. AuthN vs AuthZ (OAuth2/OIDC)

## 3. MSAL & các token flow

## 4. Shared Access Signature (SAS)

## 5. Microsoft Graph

## Tự kiểm tra
1. *(thêm câu hỏi)*

## Liên quan
- [[00-MOC-AZ-204]]
- [[../AZ-900/10-Identity-Security-AzureAD-RBAC]] — Entra ID/RBAC ở góc AZ-900
- [[07-Key-Vault-App-Configuration-Managed-Identity]] — Managed Identity (không cần secret)
