---
title: "Keys & ràng buộc toàn vẹn"
section: 03-Database
tags: [database, primary-key, foreign-key, constraint, referential-integrity, fresher]
related:
  - "[[01-Tong-quan-CSDL-quan-he]]"
  - "[[03-Quan-he-va-mo-hinh-ER]]"
  - "[[05-Chuan-hoa-va-phi-chuan-hoa]]"
difficulty: ⭐⭐
estimated_time: 25m
source: [LinkedIn "Database Foundations" - Scott Simpson]
---

# Keys & ràng buộc toàn vẹn

> [!summary] TL;DR
> **Key (khóa)** là giá trị dùng để **xác định duy nhất** một hàng. **Primary key (PK)** là khóa chính của bảng. Khi PK của bảng này xuất hiện ở bảng khác để nối dữ liệu, nó là **Foreign key (FK)**. Nếu dữ liệu không có khóa tự nhiên, ta thêm **surrogate/synthetic key** (cột số auto-increment, hoặc UUID). Cần ≥2 cột mới đủ duy nhất → **composite key**. **Referential integrity** bắt buộc FK phải trỏ tới một PK thật sự tồn tại → DB từ chối dữ liệu "mồ côi" và có thể **cascading delete**.

---

## 1. Vì sao cần unique value & key

Tên người **không** đáng tin để làm định danh (trùng tên, trùng cả SĐT/email/ngày sinh). Muốn "lấy đúng 1 hàng" → cần một giá trị **duy nhất (unique)**. Ràng buộc `UNIQUE` khiến DB từ chối hàng có giá trị trùng ở cột đó.

## 2. Phân biệt các loại key ⭐ (hay hỏi)

| Loại key | Định nghĩa | Ví dụ |
|----------|------------|-------|
| **Primary Key (PK)** | Khóa chính, định danh duy nhất 1 hàng; **NOT NULL** + **UNIQUE** | `CustomerID` |
| **Foreign Key (FK)** | PK của bảng khác, đặt trong bảng này để **nối quan hệ** | `FavoriteDish` (là `DishID` của Dishes) |
| **Surrogate / Synthetic key** | Khóa **nhân tạo** ta tự thêm (vì không có khóa tự nhiên), thường số auto-increment | cột ID tự tăng |
| **Natural key** | Khóa có sẵn trong dữ liệu thật | mã số thuế, ISBN |
| **Composite key** | Khóa **ghép từ ≥2 cột** mới đủ duy nhất | (EventName + EventDate) |
| **Candidate key** | Cột/bộ cột **có thể** làm PK (ứng viên khóa) | (Name+Date) song song với PK |
| **Unique key** | Ràng buộc duy nhất, **có thể NULL** (khác PK) | cột Email UNIQUE |

> [!tip] PK vs Unique
> **PK**: duy nhất **và** không NULL, mỗi bảng chỉ 1. **Unique**: duy nhất nhưng **cho phép NULL**, một bảng có nhiều. Mọi PK đều là candidate key; ta *chọn* một candidate key làm PK.

## 3. Surrogate key: số tự tăng hay UUID?

| | Auto-increment (INT) | UUID |
|--|----------------------|------|
| Ưu | Ngắn, nhanh, dễ đọc | Khó đoán → an toàn hơn, sinh phân tán được |
| Nhược | **Dễ đoán** (1,2,3…) → rủi ro bảo mật | Dài, nặng hơn |

## 4. Referential integrity (toàn vẹn tham chiếu)

Khi khai báo FK, DB **biết** quan hệ và **không cho** nhập/sửa dữ liệu vi phạm:
- Không cho đặt `FavoriteDish` = món **không tồn tại** trong Dishes.
- Khi **xóa** một hàng được tham chiếu, DB có thể:
  - **Cascading delete** — xóa luôn các hàng con (xóa Customer → xóa Orders của họ).
  - **Restrict** — chặn xóa nếu còn hàng con tham chiếu.

```
★ Insight ─────────────────────────────────────
• Trong quan hệ 1-nhiều, FK luôn nằm ở phía "nhiều". Vd "1 món là yêu thích
  của NHIỀU khách" → FK DishID nằm ở bảng Customers (phía nhiều).
• Dùng KEY thay vì chép tên đầy đủ: key không đổi, đảm bảo duy nhất, tốn ít
  chỗ. Nếu chép "tên món" vào Customers, đổi tên món sẽ phải sửa khắp nơi →
  vỡ tính nhất quán. Đây chính là động cơ của chuẩn hóa ([[05-Chuan-hoa-va-phi-chuan-hoa]]).
• Cascading delete tiện nhưng nguy hiểm: xóa 1 customer có thể quét sạch
  orders/reservations. Phải cân nhắc theo business rule + compliance.
─────────────────────────────────────────────────
```

## Tự kiểm tra
1. PK khác Unique key ở điểm nào? *(PK: NOT NULL + duy nhất, 1 bảng 1 cái; Unique: cho phép NULL, nhiều cái)*
2. Foreign key là gì? *(PK của bảng khác đặt trong bảng này để nối quan hệ)*
3. Surrogate key khác natural key? *(nhân tạo ta thêm vs có sẵn trong dữ liệu)*
4. Composite key dùng khi nào? *(khi không 1 cột nào đủ duy nhất → ghép ≥2 cột)*
5. Referential integrity ngăn điều gì? *(FK trỏ tới giá trị không tồn tại; dữ liệu mồ côi)*

## Liên quan
- [[00-MOC-Database|⬅ MOC Database]]
- Trước: [[01-Tong-quan-CSDL-quan-he]] · Kế tiếp: [[03-Quan-he-va-mo-hinh-ER]]
- [[17-Thuc-hanh-SQL-AutoGarage|Thực hành: PK/FK trong schema AutoGarage]]
