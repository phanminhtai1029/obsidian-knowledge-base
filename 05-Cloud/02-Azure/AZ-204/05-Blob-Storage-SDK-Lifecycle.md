---
title: "Blob Storage: SDK, metadata, lifecycle"
section: 05-Cloud/02-Azure/AZ-204
tags: [azure, az-204, blob-storage, sdk, lifecycle, fresher]
related:
  - "[[00-MOC-AZ-204]]"
  - "[[04-Cosmos-DB-SDK-Consistency-ChangeFeed]]"
difficulty: ⭐⭐⭐
estimated_time: 22m
source: ["_source/Microsoft/Az-204.docx (Lesson 5, Module 2)", "Microsoft Learn"]
status: 🚧 khung
---

# Blob Storage: SDK, metadata, lifecycle

> [!summary] TL;DR
> *(Chưa viết.)*

> [!todo] Cần viết (Lesson 5 "Develop solutions that use Blob Storage")
> - **Cấu trúc**: Storage Account → Container → Blob; 3 loại blob: **Block** (file/object), **Append** (log), **Page** (disk VHD).
> - **SDK operations**: upload/download/list/delete blob (`BlobServiceClient` → `ContainerClient` → `BlobClient`); streaming, SAS để client upload trực tiếp.
> - **Properties & metadata**: system properties vs user-defined metadata (cặp key-value); set/get qua SDK.
> - **Access tiers**: Hot / Cool / Cold / Archive (đánh đổi giá lưu vs giá truy cập + rehydrate Archive).
> - **Lifecycle management policy**: rule tự động chuyển tier / xoá blob theo tuổi (last modified / last accessed) → tối ưu chi phí.
> - **Data protection**: soft delete, versioning, snapshot, immutability.

## 1. Storage Account, container, loại blob

## 2. SDK: upload/download/list

## 3. Properties & metadata

## 4. Access tiers (Hot/Cool/Cold/Archive)

## 5. Lifecycle management policy

## Tự kiểm tra
1. *(thêm câu hỏi)*

## Liên quan
- [[00-MOC-AZ-204]]
- [[../AZ-900/09-Storage-Blob-Disk-Files]] — Storage ở góc AZ-900 (tier, redundancy)
- [[../AI-Azure/17-Azure-AI-Search]] — Blob làm nguồn dữ liệu RAG
