---
title: "Blob Storage: SDK, metadata, lifecycle"
section: 05-Cloud/02-Azure/AZ-204
tags: [azure, az-204, blob-storage, sdk, lifecycle, fresher]
related:
  - "[[00-MOC-AZ-204]]"
  - "[[04-Cosmos-DB-SDK-Consistency-ChangeFeed]]"
difficulty: ⭐⭐⭐
estimated_time: 24m
source: ["_source/Microsoft/Az-204.docx (Lesson 5, Module 2)", "Microsoft Learn"]
status: ✅
---

# Blob Storage: SDK, metadata, lifecycle

> [!summary] TL;DR
> **Azure Blob Storage** lưu **object phi cấu trúc** (ảnh/video/file/backup) theo phân cấp **Storage Account → Container → Blob**. 3 loại blob: **Block** (file/object thường) · **Append** (tối ưu ghi nối, vd log) · **Page** (random-access, lưu VHD cho VM). Thao tác qua **SDK** theo chuỗi client `BlobServiceClient → ContainerClient → BlobClient` (upload/download/list/delete); cấp **SAS** để client upload thẳng không qua server. Mỗi blob có **system properties** (do Azure đặt) và **user-defined metadata** (cặp key-value tùy bạn). Tối ưu chi phí bằng **access tier** **Hot / Cool / Cold / Archive** (lưu càng nguội → lưu rẻ, đọc đắt; Archive offline phải **rehydrate**) và **lifecycle management policy** (luật tự chuyển tier / xóa blob theo tuổi). Bảo vệ dữ liệu: **soft delete, versioning, snapshot, immutability**.

---

## 1. Storage Account, container, loại blob

```
Storage Account  →  Container (≈ thư mục gốc)  →  Blob (object)
```

| Loại blob | Tối ưu cho | Ví dụ |
|---|---|---|
| **Block blob** | File/object đọc-ghi nguyên khối | Ảnh, video, document, backup |
| **Append blob** | **Ghi nối tiếp** vào cuối | Log, audit trail |
| **Page blob** | Đọc/ghi **random** theo trang 512B | VHD của Azure VM (OS/data disk) |

- **Storage account** loại khuyến nghị chung: **Standard general-purpose v2**. Đặt tên blob theo "thư mục ảo" (`2026/06/file.jpg`) — chỉ là quy ước tên, blob thực ra phẳng trong container.

---

## 2. SDK: upload / download / list

Chuỗi client phân cấp đúng theo phân cấp tài nguyên:
```python
from azure.storage.blob import BlobServiceClient
svc = BlobServiceClient(account_url, credential)      # nên dùng Managed Identity
container = svc.get_container_client("images")
blob = container.get_blob_client("2026/photo.jpg")

with open("photo.jpg", "rb") as f:
    blob.upload_blob(f, overwrite=True)               # upload
data = blob.download_blob().readall()                 # download
for b in container.list_blobs(name_starts_with="2026/"):   # list (lọc prefix)
    print(b.name)
blob.delete_blob()                                    # delete
```
- **SAS (Shared Access Signature)**: chuỗi cấp quyền **tạm thời, giới hạn** (thời hạn + quyền read/write + IP) để **client upload/download trực tiếp** vào blob, **không phải đẩy qua server backend** → giảm tải server. (Chi tiết SAS ở [[06-AuthN-AuthZ-Identity-Entra-MSAL-SAS-Graph]].)

---

## 3. Properties & metadata

| | **System properties** | **User-defined metadata** |
|---|---|---|
| Ai đặt | Azure tự đặt | Bạn đặt |
| Ví dụ | `Content-Type`, `ETag`, `Last-Modified`, size | `{"author": "tai", "project": "x"}` |
| Sửa được? | Một số (vd Content-Type) | Có, cặp key-value tùy ý |

```python
blob.set_http_headers(content_settings=ContentSettings(content_type="image/jpeg"))
blob.set_blob_metadata({"author": "tai", "reviewed": "true"})
props = blob.get_blob_properties()   # đọc cả properties + metadata
```
> Metadata hữu ích để gắn nhãn/lọc blob mà không cần DB phụ; nhưng **không query được như index** — muốn tìm kiếm phải có hệ khác (vd AI Search).

---

## 4. Access tiers (Hot / Cool / Cold / Archive)

| Tier | Dành cho | Phí lưu | Phí truy cập | Lưu tối thiểu |
|---|---|---|---|---|
| **Hot** | Truy cập thường xuyên | Cao | Thấp | — |
| **Cool** | Ít truy cập | Thấp hơn | Cao hơn | 30 ngày |
| **Cold** | Hiếm truy cập, vẫn online | Thấp hơn nữa | Cao hơn | 90 ngày |
| **Archive** | Lưu trữ dài hạn, **offline** | **Thấp nhất** | **Cao nhất** | 180 ngày |

- **Archive** ở trạng thái **offline** → phải **rehydrate** (chuyển về Hot/Cool, mất tới vài giờ) mới đọc được; có thể chọn priority Standard/High.
- Xóa/đổi tier **sớm hơn** mức tối thiểu → phí phạt prorated. (So với AZ-900: AZ-900 nêu Hot/Cool/Archive; AZ-204 thêm **Cold** và góc SDK.)

---

## 5. Lifecycle management policy

- Là tập **rule JSON** ở cấp Storage Account, chạy định kỳ, **tự động** hành động trên blob theo tuổi:
  - chuyển Hot → Cool sau N ngày **không sửa/không truy cập**,
  - chuyển → Archive sau M ngày,
  - **xóa** blob (hoặc version/snapshot cũ) sau K ngày.
- Lọc theo **prefix** (container/thư mục ảo) và **blob index tag**.

```json
{ "rules": [{ "name": "cool-then-delete", "enabled": true,
  "type": "Lifecycle",
  "definition": { "filters": { "blobTypes": ["blockBlob"], "prefixMatch": ["logs/"] },
    "actions": { "baseBlob": {
      "tierToCool":    { "daysAfterModificationGreaterThan": 30 },
      "tierToArchive": { "daysAfterModificationGreaterThan": 90 },
      "delete":        { "daysAfterModificationGreaterThan": 365 } } } } }] }
```

**Data protection (nêu nhanh):** **soft delete** (khôi phục blob/container đã xóa trong N ngày) · **versioning** (giữ phiên bản cũ tự động) · **snapshot** (ảnh chụp read-only thủ công) · **immutability/WORM** (khóa không cho sửa/xóa — tuân thủ pháp lý).

> [!question] Phỏng vấn: "Cho client upload ảnh lên Blob mà không để lộ key và không dồn tải lên backend — làm sao?"
> Backend cấp **SAS** giới hạn (chỉ quyền write, thời hạn ngắn, đúng blob) → client dùng SAS **upload thẳng** vào Blob. Không lộ account key, không phải stream file qua server. Bảo mật hơn nữa thì dùng **user delegation SAS** ký bằng Entra ID/Managed Identity.

> [!question] Phỏng vấn: "Log cũ chiếm chi phí — tự động hạ tier rồi xóa thế nào?"
> Đặt **lifecycle management policy**: rule chuyển blob `logs/` sang Cool/Archive sau số ngày không sửa và **delete** sau hạn lưu → tối ưu chi phí tự động, không cần script.

---

```
★ Insight ─────────────────────────────────────
• Chuỗi client BlobServiceClient→ContainerClient→BlobClient phản chiếu
  đúng phân cấp tài nguyên — nhớ phân cấp là nhớ luôn cách gọi SDK.
• SAS là "chìa khóa dùng một lần có hạn": chuyển việc upload/download
  ra thẳng client, giảm tải backend mà vẫn kiểm soát quyền + thời hạn.
• Lifecycle policy = FinOps tự động: bài toán tier ở note AZ-900/09
  giờ được tự thực thi bằng luật theo tuổi blob, không đụng tay.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. Phân biệt **block / append / page blob**, mỗi loại hợp gì?
2. Viết chuỗi client SDK để upload 1 blob. **SAS** giúp gì khi client upload?
3. **System properties** vs **user-defined metadata** khác nhau ra sao? Metadata có query được không?
4. So sánh **Hot/Cool/Cold/Archive** về phí & độ sẵn sàng. **Rehydrate** là gì?
5. **Lifecycle policy** làm được những hành động nào? Cho ví dụ rule.

---

## Liên quan
- [[00-MOC-AZ-204]]
- [[04-Cosmos-DB-SDK-Consistency-ChangeFeed]] — storage NoSQL của module 2
- [[06-AuthN-AuthZ-Identity-Entra-MSAL-SAS-Graph]] — SAS chi tiết
- [[../AZ-900/09-Storage-Blob-Disk-Files]] — Storage góc AZ-900 (tier, redundancy)
- [[../AI-Azure/17-Azure-AI-Search]] — Blob làm nguồn dữ liệu RAG
