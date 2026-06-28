---
title: "MOC: Azure"
section: 05-Cloud/02-Azure
tags: [moc, cloud, azure, az-900, azure-openai, fresher]
---

# MOC: Azure

> 📚 Học thêm theo nguyện vọng cá nhân (đề thi FresherAI dùng AWS). Bám giáo trình **AZ-900 Microsoft Azure Fundamentals** (3 module đúng outline thi) + cụm AI-Azure để đối chiếu với nhánh Bedrock.
> Nguồn: `_source/Microsoft/AZ-900.md` (khoá Jim Cheshire) + tham khảo web cho cụm AI.

## Cụm A — Cloud Concepts (AZ-900 Module 1)

| # | Note | Trạng thái |
|---|------|------------|
| 1 | [[01-Tong-quan-Cloud-Shared-Responsibility\|Tổng quan Cloud & Shared Responsibility]] | ✅ |
| 2 | [[02-Cloud-Models-Consumption\|Cloud Models & Consumption-based]] | ✅ |
| 3 | [[03-Loi-ich-dam-may\|Lợi ích đám mây (HA/Scale/Reliability/Security)]] | ✅ |
| 4 | [[04-Cloud-Service-Types-IaaS-PaaS-SaaS\|Cloud Service Types: IaaS/PaaS/SaaS]] | ✅ |

## Cụm B — Azure Architecture & Services (AZ-900 Module 2)

| # | Note | Trạng thái |
|---|------|------------|
| 5 | [[05-Kien-truc-vat-ly-Regions-AZ\|Kiến trúc vật lý: Regions, Region Pairs, AZ]] | ✅ |
| 6 | [[06-To-chuc-tai-nguyen-Resource-Group-Management-Group\|Tổ chức tài nguyên: RG, Subscription, MG]] | ✅ |
| 7 | [[07-Compute-VM-Container-Functions\|Compute: VM, Container, Functions, App hosting]] | ✅ |
| 8 | [[08-Networking-VNet-VPN-ExpressRoute\|Networking: VNet, Peering, VPN, ExpressRoute]] | ✅ |
| 9 | [[09-Storage-Blob-Disk-Files\|Storage: Blob, Disks, Files, Tiers, Redundancy]] | ✅ |
| 10 | [[10-Identity-Security-AzureAD-RBAC\|Identity & Security: Entra ID, MFA, RBAC, Zero Trust]] | ✅ |

## Cụm C — Management & Governance (AZ-900 Module 3)

| # | Note | Trạng thái |
|---|------|------------|
| 11 | [[11-Quan-ly-chi-phi\|Quản lý chi phí: Calculators, Cost Management, Tags]] | ✅ |
| 12 | [[12-Governance-Blueprints-Policy-Locks\|Governance: Blueprints, Policy, Locks]] | ✅ |
| 13 | [[13-Cong-cu-quan-ly-CLI-ARM-Arc\|Công cụ: Portal, CLI/PowerShell, Arc, ARM]] | ✅ |
| 14 | [[14-Monitoring-Advisor-Monitor\|Monitoring: Advisor, Service Health, Monitor]] | ✅ |

## Cụm D — Tra cứu

| # | Note | Trạng thái |
|---|------|------------|
| 15 | [[15-AZ-900-Cheatsheet\|AZ-900 Cheatsheet & Glossary]] | ✅ |

## Cụm E — AI-Azure (đối chiếu Bedrock)

| # | Note | Trạng thái |
|---|------|------------|
| 16 | [[16-Azure-OpenAI-Service\|Azure OpenAI Service]] (↔ Bedrock) | ✅ |
| 17 | [[17-Azure-AI-Search\|Azure AI Search]] (↔ S3 Vectors) | ✅ |
| 18 | [[18-Azure-App-Service-Functions-deploy\|Deploy FastAPI: App Service & Functions]] | ✅ |

## Cụm F — AI-102 (Azure AI Engineer Associate)

> Cert AI-102 chuyên sâu AI-trên-Azure — bám trọng tâm thi FresherAI. Xem MOC riêng:

| Note | Trạng thái |
|------|------------|
| [[AI-102/00-MOC-AI-102\|MOC: AI-102]] (13 note, 7 cụm) | 🚧 khung dựng xong |

## Lộ trình

```mermaid
flowchart LR
    A["A. Cloud Concepts<br/>(1-4)"] --> B["B. Architecture & Services<br/>(5-10)"]
    B --> C["C. Management & Governance<br/>(11-14)"]
    C --> D["D. Cheatsheet (15)"]
    B -.nền cho.-> E["E. AI-Azure<br/>(16-18)"]
    E -.đối chiếu.-> AWS["AWS Bedrock"]
```

## Liên quan
- [[../00-MOC-Cloud|MOC: Cloud]]
- [[../01-AWS-Bedrock/00-MOC-AWS-Bedrock|MOC: AWS Bedrock]] — nhánh bám đề thi (so sánh tương đương)
- [[../../04-AI/00-MOC-AI|MOC: AI]] — RAG/LLM dùng Azure OpenAI + AI Search
