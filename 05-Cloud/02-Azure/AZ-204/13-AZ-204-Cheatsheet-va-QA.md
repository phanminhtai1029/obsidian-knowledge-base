---
title: "AZ-204 Cheatsheet + Q&A phỏng vấn"
section: 05-Cloud/02-Azure/AZ-204
tags: [azure, az-204, cheatsheet, qa, on-thi, fresher]
related:
  - "[[00-MOC-AZ-204]]"
difficulty: ⭐⭐
estimated_time: 24m
source: ["_source/Microsoft/Az-204.docx (tổng hợp 5 module)", "Microsoft Learn"]
status: ✅
---

# AZ-204 Cheatsheet + Q&A phỏng vấn

> [!summary] TL;DR
> Tờ tra nhanh tổng hợp toàn bộ AZ-204 (góc **developer**): bảng **chọn dịch vụ theo nhu cầu** (compute/storage/messaging), các **cặp dễ nhầm**, **glossary** ~30 thuật ngữ, **Q&A phỏng vấn**, **CLI** hay gặp và **checklist ôn nhanh**. Dùng để rà soát lần cuối trước thi/phỏng vấn.

---

## 1. Bảng chọn dịch vụ (tra nhanh)

**Compute — chạy code ở đâu:**
| Nhu cầu | Chọn |
|---|---|
| Web/API truyền thống, deploy nhanh, slot blue-green | **App Service** |
| Hàm theo sự kiện, trả tiền theo lần chạy | **Azure Functions** |
| Microservice container, scale-to-0, revisions | **Container Apps** |
| 1 container lẻ, job ngắn one-off | **ACI** |
| Cần Kubernetes đầy đủ | **AKS** |

**Storage — lưu dữ liệu ở đâu:**
| Nhu cầu | Chọn |
|---|---|
| NoSQL phân tán toàn cầu, low-latency | **Cosmos DB** |
| File/object phi cấu trúc (ảnh/video/backup) | **Blob Storage** |
| Bí mật (secret/key/cert) | **Key Vault** |
| Cấu hình tập trung + feature flag | **App Configuration** |
| Cache in-memory, session | **Cache for Redis** |

**Messaging — truyền tin giữa thành phần:**
| Nhu cầu | Chọn |
|---|---|
| Định tuyến **event** rời rạc (reactive) | **Event Grid** |
| **Stream** dữ liệu lớn (IoT/telemetry/log) | **Event Hub** |
| **Message** giao dịch, ordering, pub/sub, DLQ | **Service Bus** |
| Queue đơn giản, rẻ, dung lượng cực lớn | **Queue Storage** |
| Gateway quản lý/bảo vệ API | **API Management** |

---

## 2. Các cặp dễ nhầm

| Cặp | Khác biệt cốt lõi |
|---|---|
| **ACI vs Container Apps vs AKS** | 1 container lẻ · microservice serverless (KEDA scale-0, revisions) · K8s đầy đủ |
| **App Service vs Functions** | Web app chạy liên tục (theo plan) · hàm theo sự kiện (theo lần chạy) |
| **Scale up vs scale out** | Đổi tier mạnh hơn (dọc) · thêm instance (ngang, autoscale) |
| **Deployment slot swap** | Hoán đổi staging↔production đã warm-up → gần zero-downtime + rollback |
| **Trigger vs binding** | Trigger = cái kích hoạt (1 cái) · binding = kết nối dữ liệu (input/output, n cái) |
| **Cosmos consistency** | Strong→Bounded→**Session (mặc định)**→Consistent Prefix→Eventual (mới ↔ nhanh) |
| **Point read vs query** | Point read (id+partition key) ~1 RU, rẻ nhất · query quét tốn nhiều RU |
| **Blob tier** | Hot/Cool/Cold/Archive (lưu rẻ ↔ đọc đắt; Archive offline phải rehydrate) |
| **Key Vault vs App Configuration** | Bí mật · cấu hình thường + feature flag (nhạy cảm → Key Vault reference) |
| **System vs user-assigned MI** | Gắn vòng đời resource (1-1) · độc lập, dùng chung nhiều resource |
| **SAS vs account key** | Chìa hẹp có hạn (user delegation an toàn nhất) · chìa vạn năng vĩnh viễn |
| **AuthN vs AuthZ** | Ai (OIDC/ID token) · được làm gì (OAuth2/access token) |
| **MSAL flow** | Auth Code (có user) · Client Credentials (app-to-app) · OBO (gọi hộ user) |
| **Event vs message** | Báo "đã xảy ra" (Event Grid/Hub) · "hãy xử lý" (Service Bus/Queue) |
| **Event Grid vs Event Hub** | Định tuyến event rời rạc · ingest stream lớn (partition/consumer group) |
| **Service Bus vs Queue Storage** | Đầy đủ tính năng (topic/session/DLQ/transaction) · đơn giản, rẻ, lớn |
| **Peek-lock vs receive-and-delete** | Khóa→complete (không mất việc) · xóa ngay (nhanh, rủi ro mất) |
| **Cache-aside** | Đọc cache→miss→đọc DB→nạp cache+TTL; khó nhất là invalidation |

---

## 3. Glossary thuật ngữ

| Thuật ngữ | Nghĩa nhanh |
|---|---|
| **Image / Dockerfile** | Ảnh bất biến app+lib · công thức build image theo layer |
| **ACR Tasks** | Build image trên cloud (`az acr build`), tự rebuild khi base image vá |
| **KEDA** | Event-driven autoscaling, scale tới 0 (Container Apps) |
| **Dapr** | Sidecar runtime cho microservice (invoke/pub-sub/state) |
| **Revision** | Snapshot bất biến của Container App → traffic split/rollback |
| **App Service Plan** | Compute đứng sau web app (tier + instance); tính tiền theo plan |
| **Deployment slot** | Bản sao môi trường để swap blue-green |
| **Cold start** | Trễ lần gọi đầu sau khi instance bị thu hồi (Consumption) |
| **Trigger / Binding** | Cái kích hoạt function (1) · kết nối dữ liệu in/out (n) |
| **Durable Functions** | Orchestration có trạng thái (chaining, fan-out/in) |
| **RU/s** | Request Unit/giây — throughput Cosmos |
| **Partition key** | Khóa phân tán dữ liệu Cosmos; sai → hot partition (429) |
| **Change feed** | Log thay đổi Cosmos theo thứ tự → trigger xử lý |
| **SAS** | Token cấp quyền giới hạn/tạm thời tới Storage |
| **Managed Identity** | Danh tính Entra do Azure quản → truy cập không cần secret |
| **DefaultAzureCredential** | Tự chọn cách lấy token (MI trên Azure, CLI ở local) |
| **Feature flag** | Bật/tắt tính năng runtime (App Configuration) |
| **TTL / invalidation** | Hạn sống entry cache · giữ cache không lỗi thời |
| **Policy (APIM)** | XML inbound/backend/outbound/on-error |
| **Subscription key** | Khóa client gọi product trong APIM |
| **DLQ** | Dead-letter queue — chứa message lỗi/quá hạn |
| **Consumer group** | Khung nhìn độc lập trên stream Event Hub |
| **Operation ID** | Correlation ID xâu chuỗi distributed tracing |
| **KQL** | Kusto Query Language — query telemetry/log |
| **Sampling** | Lấy mẫu telemetry để giảm chi phí ingest |

---

## 4. Q&A phỏng vấn (tổng hợp)

> [!question] "Microservice HTTP, rảnh không tốn tiền, tải cao tự scale, deploy chia traffic thử nghiệm?"
> **Container Apps**: KEDA scale-to-0 + revisions (traffic split/rollback).

> [!question] "Deploy không downtime + rollback nhanh trên App Service?"
> **Deployment slot**: deploy staging → warm-up → **swap**; lỗi thì swap ngược.

> [!question] "Phân biệt trigger và binding?"
> Trigger = cái kích hoạt hàm (đúng 1); binding = kết nối khai báo input/output (n).

> [!question] "Chọn partition key sai gây gì?"
> **Hot partition** → throttle 429 dù tổng RU dư; logical partition trần 20GB.

> [!question] "Khi nào dùng Managed Identity thay secret?"
> Mọi khi resource Azure truy cập dịch vụ Azure khác → bỏ secret khỏi code, Azure tự xoay khóa.

> [!question] "Key Vault vs App Configuration?"
> Bí mật vs cấu hình thường + feature flag; nhạy cảm trong App Config dùng Key Vault reference.

> [!question] "Event vs message?"
> Event báo "đã xảy ra" (Event Grid/Hub); message "hãy xử lý" (Service Bus/Queue).

> [!question] "Truy nguyên request chậm qua nhiều service?"
> **Distributed tracing** theo operation ID + Application Map (App Insights).

> [!question] "Rate-limit + validate JWT + ẩn backend không sửa code?"
> **APIM policies** ở inbound (rate-limit/validate-jwt/rewrite).

---

## 5. CLI cheatsheet

```bash
# Container
az acr build --registry myreg --image app:1.0 .
az container create -g rg -n job --image app:1.0 --restart-policy OnFailure
az containerapp create -g rg -n app --image app:1.0 --environment env

# App Service
az webapp up --name myapp --runtime "PYTHON:3.12"
az webapp deployment slot create -g rg -n myapp --slot staging
az webapp deployment slot swap -g rg -n myapp --slot staging

# Functions
az functionapp create -g rg -n fn --consumption-plan-location eastus \
  --runtime python --storage-account sa

# Storage / Cosmos
az storage blob upload --account-name sa -c images -f photo.jpg -n photo.jpg
az cosmosdb create -g rg -n cosmos

# Key Vault
az keyvault secret set --vault-name kv --name db-pw --value "***"
az webapp identity assign -g rg -n myapp          # bật system-assigned MI
```

---

## 6. Checklist ôn nhanh

- [ ] Thuộc bảng **chọn compute/storage/messaging** theo nhu cầu.
- [ ] **5 consistency level** Cosmos theo thứ tự + mặc định **Session**.
- [ ] **Trigger (1) vs binding (n)**; hosting plan + cold start.
- [ ] **Deployment slot swap**; scale up vs out.
- [ ] **Managed Identity** (system vs user) + **DefaultAzureCredential**; Key Vault vs App Configuration.
- [ ] **SAS** (user delegation an toàn nhất) vs account key.
- [ ] **APIM policy** 4 giai đoạn; subscription key vs JWT.
- [ ] **Event vs message**; Event Grid vs Event Hub vs Service Bus.
- [ ] **Peek-lock vs receive-and-delete**; DLQ.
- [ ] **Distributed tracing** + KQL + sampling.

---

## Liên quan
- [[00-MOC-AZ-204]]
- [[../AZ-900/15-AZ-900-Cheatsheet]] — cheatsheet AZ-900 đối chiếu
- [[../AI-102/13-AI-102-Cheatsheet-va-QA]] — cheatsheet AI-102
