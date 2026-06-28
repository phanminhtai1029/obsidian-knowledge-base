---
title: "Kiến trúc vật lý — Regions, Region Pairs, Availability Zones"
section: 05-Cloud/02-Azure
tags: [azure, az-900, regions, region-pairs, availability-zones, datacenter, fresher]
related:
  - "[[03-Loi-ich-dam-may]]"
  - "[[06-To-chuc-tai-nguyen-Resource-Group-Management-Group]]"
  - "[[09-Storage-Blob-Disk-Files]]"
difficulty: ⭐⭐⭐
estimated_time: 25m
source: ["Microsoft AZ-900 — Jim Cheshire", "Microsoft Learn"]
---

# Kiến trúc vật lý của Azure

> [!summary] TL;DR
> Azure chia thế giới thành **Geographies** (theo biên giới quốc gia/châu lục — cho compliance) → mỗi geography có nhiều **Regions** (vị trí vật lý chứa datacenter). Mỗi region ghép với một **Region Pair** (cách ≥300 dặm, dùng cho **disaster recovery** & cập nhật lần lượt). Trong region có thể có **Availability Zones** (≥3 AZ, mỗi AZ là vị trí vật lý riêng — chống **datacenter failure / fault**, KHÔNG chống disaster vì cùng region). Ngoài ra có **Sovereign Regions** đặc biệt (Azure Government, Germany, China). **Datacenter** = toà nhà vật lý, mỗi region có ≥2.

---

## 1. Cây phân cấp địa lý

```
Geography (US, UK, Europe…)  ── biên giới pháp lý / compliance
│
├── Region (vị trí vật lý, vd "Central US")
│   ├── Availability Zone 1 ── ≥1 datacenter
│   ├── Availability Zone 2 ── ≥1 datacenter
│   └── Availability Zone 3 ── ≥1 datacenter
│
└── Region Pair (region khác, cách ≥300 dặm) ── dùng cho DR
```

---

## 2. Region & Region Pair

- **Region:** vị trí vật lý chứa các datacenter + hạ tầng vận hành Azure.
- **Region Pair:** mỗi region nối trực tiếp với một region khác làm **cặp**:
  - **Disaster recovery:** nếu cả region gặp thảm hoạ, có thể fail over sang region pair (cần bạn **đã replicate** tài nguyên ở cả hai).
  - **Cập nhật tuần tự:** khi Microsoft cập nhật Azure, chỉ update **một** region trong cặp; xong & ổn định mới update region còn lại → không gián đoạn vì update.
  - Khoảng cách **≥ 300 dặm** để một thảm hoạ khó ảnh hưởng cả hai.

> [!question] Phỏng vấn: "Availability Zone có dùng cho disaster recovery được không?"
> **Không.** AZ chống **fault / datacenter failure** trong **cùng một region**. Vì các AZ đều nằm trong một region (một vị trí địa lý), một **disaster** quét cả region có thể xoá sổ mọi AZ. Để DR phải dùng **region pairs** (đa vùng). Đây là phân biệt fault↔disaster ở [[03-Loi-ich-dam-may]].

---

## 3. Availability Zones

- Vị trí vật lý **riêng biệt** trong một region; region "AZ-enabled" có **≥3 AZ**. (Không phải region nào cũng có AZ.)
- Hai kiểu dịch vụ tận dụng AZ:

| | Zonal service | Zone-redundant service |
|---|---|---|
| Ví dụ | Azure Virtual Machines | Azure Storage |
| Cách phân bổ | **Bạn** chọn deploy vào nhiều AZ | **Azure tự** nhân bản qua nhiều AZ |

## 4. Datacenter & Sovereign Regions

- **Datacenter:** toà nhà vật lý trong region (mỗi region ≥2). Có điện/máy phát/làm mát/mạng **độc lập** → không datacenter nào phụ thuộc datacenter khác (fault tolerance).
- **Sovereign Regions** (tách biệt khỏi Azure công cộng):
  - **Azure Government** — dữ liệu ở Mỹ, chỉ công dân Mỹ truy cập, có phần đạt chuẩn DoD.
  - **Azure Germany** — tuân GDPR, vận hành bởi data trustee (T-Systems).
  - **Azure China** — vận hành bởi 21Vianet, tuân quy định Trung Quốc.

```
★ Insight ─────────────────────────────────────
• Bộ ba chống lỗi tăng dần: Datacenter độc lập → Availability Zone
  (chống mất 1 datacenter) → Region Pair (chống mất cả region/thảm hoạ).
• Region pair = DR; AZ = HA/fault tolerance. Một câu hỏi tình huống
  "bảo vệ khỏi thảm hoạ tự nhiên" → đáp án LUÔN là region pairs/đa vùng.
• Sovereign region sinh ra vì COMPLIANCE, không vì hiệu năng — nhớ gắn
  chúng với "chủ quyền dữ liệu/quy định".
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. Thứ tự phân cấp: geography, region, AZ, datacenter — cái nào chứa cái nào?
2. Region pair cách nhau tối thiểu bao xa và dùng để làm gì?
3. AZ chống loại sự cố nào, KHÔNG chống loại nào? Vì sao?
4. Zonal vs zone-redundant service khác nhau ở chỗ ai phân bổ qua AZ?
5. Kể 3 sovereign region và lý do tồn tại của chúng.

---

## Liên quan
- [[03-Loi-ich-dam-may]] — fault (→AZ) vs disaster (→region pair)
- [[09-Storage-Blob-Disk-Files]] — redundancy LRS/ZRS/GRS bám đúng AZ/region
- [[06-To-chuc-tai-nguyen-Resource-Group-Management-Group]] — tổ chức *logic* tài nguyên
