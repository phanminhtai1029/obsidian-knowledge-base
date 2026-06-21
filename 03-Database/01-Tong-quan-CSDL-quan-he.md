---
title: "Tổng quan CSDL & mô hình quan hệ"
section: 03-Database
tags: [database, rdbms, relational-model, schema, fresher]
related:
  - "[[02-Keys-va-rang-buoc]]"
  - "[[03-Quan-he-va-mo-hinh-ER]]"
  - "[[14-Gioi-thieu-PostgreSQL]]"
difficulty: ⭐⭐
estimated_time: 20m
source: [LinkedIn "Database Foundations" - Scott Simpson]
---

# Tổng quan CSDL & mô hình quan hệ

> [!summary] TL;DR
> **Database** = nơi lưu dữ liệu có **cấu trúc, nhất quán, tin cậy, tìm kiếm được**. **Relational database (CSDL quan hệ)** tổ chức dữ liệu thành **bảng (table)** gồm **hàng (row/record)** và **cột (column/field)**; mỗi bảng là một **relation**. Cấu trúc toàn bộ các bảng gọi là **schema**. **DBMS** là *phần mềm* quản trị (SQL Server, MySQL, PostgreSQL…), còn **database** là *dữ liệu* được quản trị — đừng nhầm hai khái niệm. Quan hệ là loại DB phổ biến nhất, nhưng còn document (MongoDB), graph (Neo4j), object, key-value…

---

## 1. Vì sao cần database (thay vì spreadsheet)?

Câu chuyện kinh điển: từ **note card** (phi cấu trúc) → **spreadsheet** (có cấu trúc cơ bản: hàng = khách hàng, cột = thuộc tính) → **database** (nhiều bảng + quan hệ + ràng buộc).

Database cho ta thứ spreadsheet không có:
- **Nhiều bảng + quan hệ** giữa chúng (Customers ↔ Orders ↔ Dishes).
- **Ràng buộc (rules)**: bắt buộc không thiếu dữ liệu, đúng kiểu, duy nhất.
- **Bảo mật**: phân quyền ai được xem/sửa gì.
- **Tính toàn vẹn**: chỉ commit thay đổi khi mọi bước liên quan thành công (xem [[06-ACID-va-Transaction]]).
- **Giảm dư thừa (redundancy)** dữ liệu.

---

## 2. Các khái niệm cốt lõi của mô hình quan hệ

| Thuật ngữ | Là gì | Ví dụ (bảng Customers) |
|-----------|-------|------------------------|
| **Entity** (thực thể) | Loại "thứ" ta lưu | Customer, Dish, Order |
| **Instance** (thể hiện) | Một cá thể cụ thể của entity = **1 row** | Khách "Taylor Jenkins" |
| **Attribute** (thuộc tính) | Đặc điểm mô tả entity = **1 column** | FirstName, Email, Phone |
| **Row / Record** | Một bản ghi (1 instance) | một hàng khách hàng |
| **Column / Field** | Một thuộc tính, có **kiểu dữ liệu** cố định | cột Email kiểu VARCHAR |
| **Relation** | Tập các cột = **một bảng** | bảng Customers |
| **Table** | Khối xây dựng cơ bản của DB | Customers, Dishes |
| **Schema** | Cấu trúc toàn bộ bảng + quan hệ | sơ đồ của cả DB |

> [!tip] "Relational" không phải vì các bảng "liên kết với nhau"
> Tên gọi **relational** đến từ chữ **relation** (một bảng = một tập cột có quan hệ với nhau), chứ KHÔNG phải vì các bảng nối với nhau. Đây là câu dễ bị hỏi mẹo.

---

## 3. DBMS vs Database vs RDBMS — đừng nhầm

| Khái niệm | Nghĩa |
|-----------|-------|
| **Database** | Bản thân **dữ liệu** được tổ chức |
| **DBMS** (Database Management System) | **Phần mềm** để tạo/quản trị/tương tác với database (SQL Server, MySQL, Access, PostgreSQL) |
| **RDBMS** (Relational DBMS) | DBMS dành riêng cho CSDL quan hệ, hỗ trợ SQL |

---

## 4. Các loại database (không chỉ có quan hệ)

| Loại | Tổ chức dữ liệu | Ví dụ |
|------|-----------------|-------|
| **Relational** (phổ biến nhất) | Bảng (rows/columns), quan hệ | PostgreSQL, MySQL, SQL Server |
| **Document** | Tài liệu JSON-like | MongoDB, CouchDB |
| **Graph** | Node + cạnh (quan hệ) | Neo4j |
| **Object** | Đối tượng | Realm, Objectivity/DB |
| **Key-Value / NoSQL khác** | cặp khóa-giá trị, stream… | Redis, Cassandra |

```
★ Insight ─────────────────────────────────────
• Mỗi cột có MỘT kiểu dữ liệu cố định (number/string/date/boolean/binary).
  Giá trị từng hàng khác nhau, nhưng "loại thông tin" của cột là bất biến —
  đó là điều khiến DB lưu trữ hiệu quả và cung cấp tính năng theo kiểu.
• "Schema" trong CSDL quan hệ là schema CỨNG (fixed): phải định nghĩa cấu
  trúc trước khi nhét dữ liệu. NoSQL ngược lại — schema động ([[15-PostgreSQL-vs-cac-DB-khac]]).
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. "Relational" trong relational database bắt nguồn từ đâu? *(từ "relation" = một bảng/tập cột có quan hệ, không phải vì các bảng nối nhau)*
2. Phân biệt Database, DBMS, RDBMS? *(dữ liệu / phần mềm quản trị / phần mềm quản trị CSDL quan hệ)*
3. Row và Column còn gọi là gì? *(record / field)*
4. Kể 3 loại database ngoài quan hệ? *(document, graph, object/key-value…)*
5. Schema là gì? *(cấu trúc toàn bộ bảng + quan hệ trong DB)*

## Liên quan
- [[00-MOC-Database|⬅ MOC Database]]
- Kế tiếp: [[02-Keys-va-rang-buoc|Keys & ràng buộc toàn vẹn]]
- [[14-Gioi-thieu-PostgreSQL|PostgreSQL — một RDBMS cụ thể]] · [[../02-Backend/00-MOC-Backend|Backend dùng DB này qua ORM]]
