---
title: "Kiểu dữ liệu & thiết kế bảng"
section: 03-Database
tags: [database, data-types, create-table, null, constraint, fresher]
related:
  - "[[03-Quan-he-va-mo-hinh-ER]]"
  - "[[02-Keys-va-rang-buoc]]"
  - "[[07-SQL-co-ban]]"
difficulty: ⭐⭐
estimated_time: 25m
source: [LinkedIn "Database Foundations" - Scott Simpson]
---

# Kiểu dữ liệu & thiết kế bảng

> [!summary] TL;DR
> Mỗi cột phải khai báo **data type** để DB lưu hiệu quả và cung cấp tính năng theo kiểu. Nhóm chính: **chuỗi** (`CHAR` cố định / `VARCHAR` biến thiên / `TEXT` dài), **ngày giờ** (`DATE` / `DATETIME` / `TIMESTAMP`), **số** (`INT` / `DECIMAL` / `FLOAT`/double). **NULL** = *vắng giá trị* (không phải 0 hay false, không phải chữ "null"). Khi tạo bảng bằng `CREATE TABLE`, ta gắn thêm **constraint**: `PRIMARY KEY`, `NOT NULL`, `AUTO_INCREMENT`, `FOREIGN KEY … REFERENCES`.

---

## 1. Vì sao phải chọn đúng kiểu

DB cần biết "loại thông tin" để **lưu hiệu quả** và **mở khóa tính năng**: sắp xếp ngày đúng thứ tự thời gian, tính toán trên số, ràng buộc độ dài chuỗi. Lưu tất cả thành chuỗi cũng được, nhưng sẽ phải làm thêm việc thủ công (sort ngày, cộng tiền…).

## 2. Các nhóm kiểu dữ liệu

| Nhóm | Kiểu | Khi dùng |
|------|------|----------|
| **Chuỗi** | `CHAR(n)` — **cố định** n ký tự | dữ liệu luôn đúng độ dài (mã 2 ký tự…) |
| | `VARCHAR(n)` — **biến thiên**, tối đa n | tên, địa chỉ (vd VARCHAR(200)) |
| | `TEXT` — dài | mô tả, bài viết |
| **Ngày giờ** | `DATE` | chỉ ngày (Birthday) |
| | `DATETIME` | ngày + giờ (Reservations, Events) |
| | `TIMESTAMP` | tự ghi thời điểm tạo/sửa (khi đặt order) |
| **Số** | `INT` / integer | khóa, đếm |
| | `DECIMAL(p,s)` | tiền (vd `DECIMAL(5,2)` = tối đa 999.99) |
| | `FLOAT` / double | số thực gần đúng |
| **Khác** | `BOOLEAN`, `BINARY`/BLOB, UUID, geospatial… | true/false, file, định danh… |

> [!warning] CHAR vs VARCHAR
> `CHAR(n)` **luôn** chiếm n ký tự dù dữ liệu ngắn → chỉ dùng khi độ dài cố định. `VARCHAR(n)` tiết kiệm chỗ với dữ liệu dài ngắn khác nhau. Trên hàng triệu hàng × trăm cột, chênh lệch này rất đáng kể.

> [!tip] Số điện thoại KHÔNG phải số
> SĐT lưu bằng **VARCHAR** (vd 20 ký tự), không phải INT — vì có số 0 đầu, dấu `+`, ký tự đặc biệt, đuôi extension, và ta không làm toán với nó.

## 3. NULL — giá trị "vắng mặt"

- NULL = **không có giá trị** (khác `0`, khác `false`, khác chuỗi `"null"`).
- NULL **không phải kiểu dữ liệu**, mà là một **điều kiện**. Ta nói một ô *is null / is not null*, **không** nói *= null*.
- Khi định nghĩa cột, ta chọn cột đó **NULLable hay NOT NULL**, và **default value**.

## 4. CREATE TABLE + constraint

```sql
CREATE TABLE Customers (
    CustomerID  INT          NOT NULL AUTO_INCREMENT,
    FirstName   VARCHAR(200) NOT NULL,
    LastName    VARCHAR(200) NOT NULL,
    Phone       VARCHAR(20)  NOT NULL,
    Birthday    DATE,
    FavoriteDish INT,
    PRIMARY KEY (CustomerID),
    FOREIGN KEY (FavoriteDish) REFERENCES Dishes(DishID)
);
```

Các **constraint** thường gặp: `PRIMARY KEY`, `FOREIGN KEY … REFERENCES`, `NOT NULL`, `UNIQUE`, `DEFAULT`, `AUTO_INCREMENT` (PostgreSQL: `SERIAL`/`GENERATED … AS IDENTITY`).

> Constraint = cách để **business rule sống trong DB** (vd "khách phải có SĐT" → `Phone NOT NULL`). DB tự chặn dữ liệu sai, ta không phải viết logic kiểm tra thủ công.

```
★ Insight ─────────────────────────────────────
• Chọn DECIMAL (không phải FLOAT) cho tiền: FLOAT là số thực gần đúng → lỗi
  làm tròn khi cộng tiền. DECIMAL(p,s) chính xác đến từng xu.
• NOT NULL + AUTO_INCREMENT cho PK: PK không được NULL (vô nghĩa), và auto-
  increment đảm bảo mỗi hàng một giá trị duy nhất, tăng dần.
─────────────────────────────────────────────────
```

## Tự kiểm tra
1. CHAR khác VARCHAR? *(CHAR cố định độ dài luôn chiếm n; VARCHAR biến thiên, tiết kiệm)*
2. Vì sao SĐT lưu VARCHAR? *(có số 0 đầu/ký tự đặc biệt/extension, không làm toán)*
3. NULL nghĩa là gì, viết điều kiện thế nào? *(vắng giá trị; dùng IS NULL / IS NOT NULL, không = null)*
4. Kiểu nào cho tiền và vì sao? *(DECIMAL — chính xác, tránh lỗi làm tròn của FLOAT)*
5. Kể 4 constraint trong CREATE TABLE? *(PRIMARY KEY, FOREIGN KEY, NOT NULL, UNIQUE/DEFAULT/AUTO_INCREMENT)*

## Liên quan
- [[00-MOC-Database|⬅ MOC Database]]
- Trước: [[03-Quan-he-va-mo-hinh-ER]] · Kế tiếp: [[05-Chuan-hoa-va-phi-chuan-hoa]]
- [[07-SQL-co-ban|SQL: DDL tạo bảng]] · [[17-Thuc-hanh-SQL-AutoGarage|Thực hành: CREATE TABLE AutoGarage]]
