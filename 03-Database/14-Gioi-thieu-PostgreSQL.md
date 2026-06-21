---
title: "Giới thiệu PostgreSQL"
section: 03-Database
tags: [database, postgresql, ordbms, mvcc, json, fresher]
related:
  - "[[15-PostgreSQL-vs-cac-DB-khac]]"
  - "[[16-Lam-viec-voi-PostgreSQL]]"
  - "[[06-ACID-va-Transaction]]"
difficulty: ⭐⭐
estimated_time: 25m
source: [LinkedIn "Learning PostgreSQL" - Sarah Conway]
---

# Giới thiệu PostgreSQL

> [!summary] TL;DR
> **PostgreSQL** (gọi tắt **Postgres**) là **ORDBMS** — object-relational DBMS: nền quan hệ + thêm tính năng hướng đối tượng (class, inheritance). Miễn phí & mã nguồn mở 35+ năm. Đặc điểm nổi bật: **ACID** (từ 2001), **MVCC** (đọc-ghi đồng thời không khóa lẫn nhau), nhiều **kiểu dữ liệu** (kể cả **JSON/JSONB**, UUID, array, geometric), nhiều **loại index** (BTREE mặc định, Hash, GiST, SP-GiST, GIN, BRIN), hỗ trợ view/trigger/stored procedure, bám sát chuẩn SQL. Được Uber, Netflix, Instagram, Spotify, Reddit… dùng.

---

## 1. PostgreSQL là gì

- **ORDBMS** (Object-Relational DBMS): chủ yếu là quan hệ, **cộng** vài tính năng hướng đối tượng (classes, inheritance, polymorphism) → xử lý được nhiều kiểu dữ liệu hơn DB quan hệ thuần.
- **Mã nguồn mở, miễn phí** từ Postgres95 (1994), phát triển liên tục 35+ năm, cộng đồng lớn (700+ contributor, 1.6M+ dòng C).
- Bám sát **chuẩn SQL** (PG15 đạt ≥170/179 tính năng bắt buộc SQL:2016) → biết SQL là có lợi thế ngay.

## 2. Tính năng cốt lõi ⭐

| Tính năng | Ý nghĩa |
|-----------|---------|
| **ACID** (từ 2001) | đảm bảo toàn vẹn giao dịch ([[06-ACID-va-Transaction]]) |
| **MVCC** (Multi-Version Concurrency Control) | mỗi user thấy **snapshot** nhất quán; **reader không chặn writer** và ngược lại → đa người dùng mượt |
| **Nhiều kiểu dữ liệu** | Boolean, numeric, temporal, array, UUID, **JSON/JSONB**, hstore, geometric… |
| **Nhiều loại index** | BTREE (mặc định), Hash, GiST, SP-GiST, GIN, BRIN |
| **View, Trigger, Stored procedure** | hỗ trợ đầy đủ |
| **Extension** | mở rộng gần như mọi nhu cầu ([[16-Lam-viec-voi-PostgreSQL]]) |
| **Quốc tế hóa** | full international character sets |

> [!tip] JSON/JSONB → "NoSQL bên trong Postgres"
> Từ bản 9.2 (JSON) và 9.4 (**JSONB** — dạng nhị phân, index được), Postgres lưu & truy vấn dữ liệu phi cấu trúc như document DB. Vì vậy nói Postgres "vừa SQL vừa NoSQL".

## 3. MVCC — vì sao quan trọng

Thay vì khóa hàng khi đọc/ghi (gây chờ), MVCC cho mỗi giao dịch một **bản chụp tại thời điểm**: người khác **không thấy** thay đổi cho đến khi **commit**; hàng vẫn đọc được trong lúc đang sửa. → Hiệu năng cao trong môi trường nhiều người dùng (đây là cách Postgres hiện thực chữ **Isolation** của ACID).

```
★ Insight ─────────────────────────────────────
• "Object-relational" là điểm khác biệt với MySQL (relational thuần): Postgres
  xử lý JSON/JSONB, array, custom type, geometric… ngay trong engine.
• MVCC = lý do Postgres không bị "reader chặn writer". Cái giá: sinh nhiều
  phiên bản hàng → cần VACUUM dọn rác định kỳ (kiến thức DBA nâng cao).
• Chọn loại index theo truy vấn: BTREE cho so sánh/khoảng; GIN cho JSONB/full-
  text; BRIN cho bảng lớn theo thứ tự (vd time-series).
─────────────────────────────────────────────────
```

## Tự kiểm tra
1. ORDBMS nghĩa là gì? *(object-relational — quan hệ + tính năng hướng đối tượng)*
2. MVCC giải quyết gì? *(đọc-ghi đồng thời không khóa lẫn nhau; mỗi giao dịch thấy snapshot nhất quán)*
3. JSONB cho Postgres khả năng gì? *(lưu/truy vấn dữ liệu phi cấu trúc như NoSQL, index được)*
4. Index mặc định? *(BTREE)*
5. Postgres miễn phí không? *(có — mã nguồn mở, miễn phí)*

## Liên quan
- [[00-MOC-Database|⬅ MOC Database]]
- Trước: [[13-Phan-quyen-Compliance-SQL-Injection]] · Kế tiếp: [[15-PostgreSQL-vs-cac-DB-khac]]
- [[06-ACID-va-Transaction|ACID]] · [[12-Index-Transaction-StoredProcedure|Các loại index]]
