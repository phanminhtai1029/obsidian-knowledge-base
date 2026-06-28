---
title: "Container: image, ACR, ACI, Container Apps"
section: 05-Cloud/02-Azure/AZ-204
tags: [azure, az-204, container, acr, aci, container-apps, fresher]
related:
  - "[[00-MOC-AZ-204]]"
  - "[[02-App-Service-Web-Apps]]"
difficulty: ⭐⭐⭐
estimated_time: 24m
source: ["_source/Microsoft/Az-204.docx (Lesson 1, Module 1)", "Microsoft Learn"]
status: 🚧 khung
---

# Container: image, ACR, ACI, Container Apps

> [!summary] TL;DR
> *(Chưa viết.)*

> [!todo] Cần viết (Lesson 1 "Implement containerized solutions")
> - **Container là gì** (so với VM): chia sẻ kernel, nhẹ, khởi động nhanh; **image** vs **container instance**; Dockerfile, layer.
> - **Azure Container Registry (ACR)**: managed registry (dựa trên Docker Registry); tier Basic/Standard/Premium; **ACR Tasks** (build image trên cloud — `az acr build`); geo-replication; webhook.
> - **Azure Container Instance (ACI)**: chạy 1 container đơn lẻ, serverless, trả tiền theo giây; container group; restart policy; không tự scale → hợp burst/job ngắn.
> - **Azure Container Apps**: nền tảng serverless chạy microservice container, **autoscale (KEDA)** kể cả scale-to-zero, **Dapr**, revision/traffic-split (blue-green); chạy trên AKS quản lý sẵn.
> - **So sánh ACI vs Container Apps vs AKS** (khi nào dùng cái nào).
> - Lệnh CLI tiêu biểu: `az acr create/build`, `az container create`, `az containerapp create`.

## 1. Container & image cơ bản (vs VM)

## 2. Azure Container Registry (ACR) + ACR Tasks

## 3. Azure Container Instance (ACI)

## 4. Azure Container Apps (KEDA, Dapr, revision)

## 5. So sánh ACI / Container Apps / AKS

## Tự kiểm tra
1. *(thêm câu hỏi)*

## Liên quan
- [[00-MOC-AZ-204]]
- [[../AZ-900/07-Compute-VM-Container-Functions]] — góc nhìn khái niệm AZ-900
- [[../../../06-DevOps/13-Docker-Practical|Docker thực hành]]
