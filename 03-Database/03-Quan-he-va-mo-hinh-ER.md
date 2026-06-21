---
title: "Quan hệ & mô hình hóa ER"
section: 03-Database
tags: [database, relationships, er-diagram, data-modeling, fresher]
related:
  - "[[02-Keys-va-rang-buoc]]"
  - "[[04-Kieu-du-lieu-va-thiet-ke-bang]]"
  - "[[05-Chuan-hoa-va-phi-chuan-hoa]]"
difficulty: ⭐⭐⭐
estimated_time: 25m
source: [LinkedIn "Database Foundations" - Scott Simpson]
---

# Quan hệ & mô hình hóa ER

> [!summary] TL;DR
> Ba loại quan hệ: **one-to-many (1:M)** — phổ biến nhất, FK nằm ở phía "nhiều"; **many-to-many (M:N)** — phải tạo **linking/associative table** (gồm 2 FK); **one-to-one (1:1)** — hiếm, thường để tách dữ liệu nhạy cảm. Trước khi tạo bảng, ta **mô hình hóa** bằng **ER diagram** (Entity-Relationship): bắt đầu từ **needs analysis** → xác định entity/attribute → vẽ quan hệ. Đặt tên bảng số nhiều viết hoa đầu (Customers), cột số ít UpperCamelCase (FirstName), tránh dấu cách & ký tự đặc biệt.

---

## 1. Ba loại quan hệ ⭐

| Quan hệ | Ý nghĩa | Cách hiện thực | Ví dụ |
|---------|---------|----------------|-------|
| **One-to-Many (1:M)** | 1 hàng A ↔ nhiều hàng B | **FK ở phía "nhiều"** trỏ về PK phía "một" | 1 Dish là favorite của nhiều Customers; 1 Customer có nhiều Reservations |
| **Many-to-Many (M:N)** | nhiều A ↔ nhiều B | **Linking table** = 2 quan hệ 1:M + bảng trung gian (2 FK) | Orders ↔ Dishes qua `OrdersDishes` |
| **One-to-One (1:1)** | 1 hàng A ↔ đúng 1 hàng B | FK + ràng buộc UNIQUE; tách bảng | tách Customers thành (Name) và (PII nhạy cảm) |

> [!tip] "Many-to-one" không tồn tại như loại riêng
> 1:M nhìn từ chiều ngược lại trông như "nhiều-một", nhưng vẫn là **một** loại quan hệ 1:M. Chỉ là góc nhìn.

## 2. Many-to-many cần linking table

DBMS **không** mô hình trực tiếp M:N. Ta tạo **bảng nối (linking/associative table)** — đặt tên ghép hai bảng (`OrdersDishes`), chứa tối thiểu 2 FK (`OrderID`, `DishID`). Mỗi món trong một đơn = 1 hàng.

→ Lợi: bảng Orders gọn; biết một đơn có những món gì; biết một món xuất hiện trong bao nhiêu đơn. (Ví dụ khác: `DishesIngredients`, `DishesAllergens`, `CustomersEvents`.)

## 3. One-to-one: khi nào dùng

Hiếm — vì "1 hàng ↔ 1 hàng" thường gợi ý nên gộp làm 1 bảng. Dùng khi:
- **Bảo mật**: tách PII (birthday, email) ra bảng riêng, chỉ ai có quyền mới xem.
- **Gán tài nguyên 1-1**: 1 iPad ↔ 1 nhân viên.

→ Đây là một dạng **denormalization** ([[05-Chuan-hoa-va-phi-chuan-hoa]]).

## 4. Quy trình mô hình hóa & ER diagram

1. **Needs analysis** — DB cần lưu gì? (Customers, Dishes, Events, Orders, Reservations…)
2. Với mỗi entity → liệt kê **attribute** (như nội dung 1 note card).
3. Xác định **key** ([[02-Keys-va-rang-buoc]]).
4. Vẽ **quan hệ** (đường nối; ký hiệu **crow's foot** = phía "nhiều").
5. Lặp lại — thiết kế là quá trình **iterative**, không cần đúng ngay lần đầu.

## 5. Quy ước đặt tên (best practice)

| Đối tượng | Quy ước | Ví dụ |
|-----------|---------|-------|
| **Tên bảng** | số nhiều, viết hoa đầu, không dấu cách | `Customers`, `Dishes` |
| **Tên cột** | số ít, **UpperCamelCase** | `FirstName`, `Birthday` |
| **Tên key** | TênBảng + ID | `CustomerID`, `DishID` |
| Tránh | dấu cách, ký tự đặc biệt | ❌ `First Name` |

```
★ Insight ─────────────────────────────────────
• Dấu hiệu cần linking table: khi bạn định "thêm cột FavoriteDish1,
  FavoriteDish2…" hay nhét nhiều giá trị vào 1 ô → đó là M:N, hãy tách bảng
  nối. (Cũng là vi phạm 1NF — xem [[05-Chuan-hoa-va-phi-chuan-hoa]].)
• ER diagram chỉ là MÔ HÌNH (cách nghĩ về dữ liệu). Quan hệ chỉ thực sự được
  DB "ép buộc" khi ta khai báo FK + referential integrity ([[02-Keys-va-rang-buoc]]).
─────────────────────────────────────────────────
```

## Tự kiểm tra
1. Quan hệ nào phổ biến nhất và FK nằm ở đâu? *(1:M; FK ở phía "nhiều")*
2. Hiện thực M:N thế nào? *(linking table = 2 FK, tách thành 2 quan hệ 1:M)*
3. Khi nào dùng 1:1? *(tách dữ liệu nhạy cảm/PII; gán tài nguyên 1-1)*
4. Quy ước đặt tên bảng vs cột? *(bảng số nhiều viết hoa; cột số ít UpperCamelCase)*
5. ER diagram là gì? *(sơ đồ entity-attribute-relationship để thiết kế schema)*

## Liên quan
- [[00-MOC-Database|⬅ MOC Database]]
- Trước: [[02-Keys-va-rang-buoc]] · Kế tiếp: [[04-Kieu-du-lieu-va-thiet-ke-bang]]
- [[09-JOIN|JOIN — truy vấn xuyên các quan hệ]] · [[17-Thuc-hanh-SQL-AutoGarage|Thực hành: ER của AutoGarage]]
