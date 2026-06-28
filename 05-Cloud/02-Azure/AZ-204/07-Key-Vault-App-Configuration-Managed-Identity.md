---
title: "Key Vault, App Configuration & Managed Identity"
section: 05-Cloud/02-Azure/AZ-204
tags: [azure, az-204, key-vault, app-configuration, managed-identity, secrets, fresher]
related:
  - "[[00-MOC-AZ-204]]"
  - "[[06-AuthN-AuthZ-Identity-Entra-MSAL-SAS-Graph]]"
difficulty: ⭐⭐⭐
estimated_time: 24m
source: ["_source/Microsoft/Az-204.docx (Lesson 7, Module 3)", "Microsoft Learn"]
status: 🚧 khung
---

# Key Vault, App Configuration & Managed Identity

> [!summary] TL;DR
> *(Chưa viết.)*

> [!todo] Cần viết (Lesson 7 "Implement secure Azure solutions")
> - **Vấn đề**: secret/credential/cert hardcode trong code/config là rủi ro lớn (repo public, leak).
> - **Azure Key Vault**: lưu **secrets / keys / certificates**; access qua RBAC hoặc access policy; soft delete + purge protection; rotation; audit log; HSM-backed (Premium).
> - **App Configuration**: quản lý cấu hình tập trung (key-value), **feature flags**, point-in-time snapshot; tham chiếu Key Vault cho giá trị nhạy cảm.
> - **Managed Identity** (điểm cốt lõi): danh tính Entra do Azure quản lý cho resource → **không cần lưu secret**; **System-assigned** (gắn vòng đời resource) vs **User-assigned** (độc lập, dùng chung). Code dùng `DefaultAzureCredential` để lấy token tự động.
> - **Luồng chuẩn**: App (Managed Identity) → token → đọc secret từ Key Vault, không có key nào trong code.
> - Liên hệ secrets trong CI/CD (DevOps).

## 1. Vì sao cần Key Vault (rủi ro hardcode secret)

## 2. Azure Key Vault (secret/key/cert)

## 3. App Configuration & feature flags

## 4. Managed Identity (system vs user-assigned)

## 5. Luồng chuẩn: Managed Identity → Key Vault

## Tự kiểm tra
1. *(thêm câu hỏi)*

## Liên quan
- [[00-MOC-AZ-204]]
- [[../../../06-DevOps/09-CI-CD-Continuous-Deployment]] — secrets trong CI/CD
- [[06-AuthN-AuthZ-Identity-Entra-MSAL-SAS-Graph]] — token & Entra ID
