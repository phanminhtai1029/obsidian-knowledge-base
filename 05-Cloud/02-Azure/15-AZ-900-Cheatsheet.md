---
title: "AZ-900 Cheatsheet & Glossary"
section: 05-Cloud/02-Azure
tags: [azure, az-900, cheatsheet, glossary, reference, fresher]
related:
  - "[[04-Cloud-Service-Types-IaaS-PaaS-SaaS]]"
  - "[[09-Storage-Blob-Disk-Files]]"
  - "[[00-MOC-Azure]]"
difficulty: ⭐⭐
estimated_time: 20m
source: ["Microsoft AZ-900 — Jim Cheshire", "Microsoft Learn"]
---

# AZ-900 Cheatsheet & Glossary

> [!summary] TL;DR
> Trang ôn nhanh trước thi AZ-900 / phỏng vấn cloud: **dịch vụ theo nhóm**, **bảng phân biệt các cặp dễ nhầm**, **storage tier/redundancy**, **glossary ~30 thuật ngữ**, và mẹo đi thi. Câu vàng phân biệt: *fault vs disaster*, *AZ vs region pair*, *RBAC vs Lock*, *IaaS/PaaS/SaaS*.

---

## 1. Dịch vụ theo nhóm

| Nhóm | Dịch vụ chính |
|---|---|
| **Compute** | Virtual Machines (IaaS), ACI, Functions (serverless), AKS, Virtual Desktop |
| **Networking** | VNet, Subnet, VNet Peering, VPN Gateway, ExpressRoute, Azure DNS, Load Balancer |
| **Storage** | Blob, Disks, Files, Queue, Table; AzCopy, Storage Explorer, Data Box |
| **Identity** | Entra ID (Azure AD), AD DS, MFA, Conditional Access, RBAC |
| **App hosting** | App Service (PaaS), AKS, VM |
| **Management** | Portal, CLI/PowerShell, Cloud Shell, ARM templates, Azure Arc |
| **Governance** | Blueprints, Policy, Resource Locks, Tags |
| **Monitoring** | Advisor, Service Health, Azure Monitor (App Insights, Log Analytics/KQL) |
| **Cost** | Pricing Calculator, TCO Calculator, Cost Management + Billing |

---

## 2. Service types & shared responsibility

| | IaaS | PaaS | SaaS |
|---|---|---|---|
| Bạn lo | OS↑ | App & data | Data & cách dùng |
| Ví dụ | Azure VM | App Service, AKS, Functions | Microsoft 365, Outlook.com |
| Trách nhiệm / kiểm soát | cao nhất | trung bình | thấp nhất |

> Luôn của bạn: **data, identity, quản lý quyền truy cập**.

---

## 3. Cặp dễ nhầm (đề thi hay cài bẫy)

| Cặp | Khác biệt cốt lõi |
|---|---|
| **Fault vs Disaster** | nhỏ/cục bộ (→ AZ) vs lớn/cả vùng (→ region pair, BCDR) |
| **Availability Zone vs Region Pair** | HA trong region vs DR đa region |
| **Availability Set vs Availability Zone** | rack/update domain trong DC vs nhiều DC riêng |
| **Vertical vs Horizontal scaling** | máy mạnh hơn vs nhiều máy hơn |
| **Scalability vs Elasticity** | mở rộng được vs **tự động** co giãn |
| **AuthN vs AuthZ** | bạn là ai vs bạn được làm gì |
| **RBAC vs Resource Lock** | theo role/principal vs áp cho **mọi user** |
| **Policy vs Lock vs Blueprint** | ép luật vs chống xoá/đổi vs gói triển khai |
| **VNet Peering vs VPN Gateway** | backbone riêng (không mã hoá) vs Internet (mã hoá) |
| **VPN Gateway vs ExpressRoute** | qua Internet ~1.25Gbps vs đường riêng 10–100Gbps |
| **Pricing vs TCO Calculator** | ước giá mới vs so sánh tiết kiệm khi di trú |
| **Advisor vs Service Health vs Monitor** | cấu hình của bạn vs sự cố Azure vs metrics/log |

---

## 4. Storage tier & redundancy

| Tier | Phí lưu | Phí đọc | Min |
|---|---|---|---|
| Hot | cao | thấp | — |
| Cool | thấp | cao | 30 ngày |
| Archive | thấp nhất | cao nhất (hydrate) | 180 ngày |

| Redundancy | Phạm vi | Chống |
|---|---|---|
| LRS | 3 bản, 1 datacenter | lỗi đĩa/server |
| ZRS | 3 AZ | mất 1 datacenter (fault) |
| GRS | + region xa (LRS) | disaster |
| GZRS | ZRS + region xa | fault + disaster (bền nhất) |

---

## 5. Glossary (~30 thuật ngữ)

| Thuật ngữ | Nghĩa |
|---|---|
| **Shared Responsibility** | Trách nhiệm chia giữa bạn & provider, đổi theo IaaS/PaaS/SaaS |
| **Consumption-based** | Trả cho tài nguyên được **cấp phát** (kể cả không dùng) |
| **High Availability** | Sẵn sàng ≥99% |
| **Scalability / Elasticity** | Mở rộng được / **tự động** co giãn |
| **Agility** | Triển khai & thay đổi nhanh |
| **Fault tolerance / BCDR** | Chịu lỗi nhỏ / kế hoạch chống thảm hoạ |
| **Geography / Region / AZ / Datacenter** | Vùng pháp lý / vị trí / zone trong region / toà nhà |
| **Region Pair** | Cặp region (≥300mi) cho DR & update tuần tự |
| **Resource Group** | Hộp chứa logic của resource |
| **Subscription / Management Group** | Đơn vị thanh toán / tổ chức nhiều subscription |
| **IaaS / PaaS / SaaS** | 3 loại dịch vụ theo mức provider cấp |
| **VM / ACI / Functions / AKS** | Máy ảo / container nhanh / serverless / Kubernetes |
| **Availability Set** | Fault domain + update domain cho VM |
| **VMSS** | Scale set — autoscale nhiều VM giống nhau |
| **App Service / App Service Plan** | PaaS host web app / đơn vị scale & tier |
| **VNet / Subnet** | Mạng ảo / phân đoạn mạng |
| **Peering / VPN Gateway / ExpressRoute** | Nối VNet / nối lai qua Internet / đường riêng |
| **Public / Private endpoint** | IP từ Internet / chỉ mạng riêng |
| **Blob / Disk / Files** | File phi cấu trúc / đĩa VM / chia sẻ SMB |
| **LRS/ZRS/GRS/GZRS** | Các mức redundancy theo phạm vi |
| **Entra ID (Azure AD)** | Dịch vụ định danh đám mây |
| **Service principal / Managed identity** | App trong Entra ID / identity đại diện resource |
| **SSO / MFA / Passwordless** | Đăng nhập 1 lần / đa yếu tố / không mật khẩu |
| **Conditional Access** | Áp chính sách theo tín hiệu khi truy cập |
| **RBAC** | Security principal + role + scope (additive) |
| **Zero Trust / Defense in Depth** | Không tin mặc định / phòng thủ nhiều lớp |
| **Defender for Cloud** | Giám sát & bảo vệ security posture |
| **Blueprint / Policy / Lock** | Gói triển khai / ép luật / chống xoá-đổi |
| **ARM / ARM template** | Engine quản resource / file JSON declarative |
| **Azure Arc** | Mở rộng quản trị ra ngoài Azure |
| **Advisor / Service Health / Monitor** | Khuyến nghị / sự cố Azure / metrics-log |
| **Reservations / Spot VM / Hybrid Benefit** | Cam kết dài hạn / dung lượng dư rẻ / mang license |
| **Tags** | Cặp name-value phân loại & lên hoá đơn |
| **KQL** | Kusto Query Language cho Log Analytics |

```
★ Insight ─────────────────────────────────────
• 3 câu chốt AZ-900: (1) fault→AZ vs disaster→region pair; (2) ranh
  giới shared responsibility theo IaaS/PaaS/SaaS; (3) RBAC (theo role)
  vs Lock (mọi user) vs Policy (ép luật).
• Đề AZ-900 = nhận diện đúng dịch vụ cho tình huống + phân biệt cặp
  thuật ngữ na ná. Học theo "khi nào dùng cái nào" hơn là học vẹt.
─────────────────────────────────────────────────
```

---

## Liên quan
- [[04-Cloud-Service-Types-IaaS-PaaS-SaaS]] · [[05-Kien-truc-vat-ly-Regions-AZ]] · [[10-Identity-Security-AzureAD-RBAC]]
- [[00-MOC-Azure]] — quay lại MOC
