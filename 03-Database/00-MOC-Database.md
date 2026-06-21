---
title: "MOC: Database"
section: 03-Database
tags: [moc, database, sql, postgresql, fresher]
module: L1_AL_NLS_DF_NLS
---

# MOC: Database Fundamentals & PostgreSQL

> Module `L1_AL_NLS_DF_NLS`. Thi: **Final Theory Test** + **Final Practical Test**.
> Nguồn: LinkedIn *Database Foundations* (Scott Simpson) + *Learning PostgreSQL* (Sarah Conway) + bộ thực hành **AutoGarage** (bài tự giải).

## Cụm A — Nền tảng CSDL quan hệ (Theory)

| # | Note | Nội dung | Trạng thái |
|---|------|----------|------------|
| 1 | [[01-Tong-quan-CSDL-quan-he\|Tổng quan CSDL & mô hình quan hệ]] | entity/relation/schema, DBMS vs RDBMS, các loại DB | ✅ |
| 2 | [[02-Keys-va-rang-buoc\|Keys & ràng buộc toàn vẹn]] | PK/FK/surrogate/composite/candidate, referential integrity | ✅ |
| 3 | [[03-Quan-he-va-mo-hinh-ER\|Quan hệ & mô hình ER]] | 1:M, M:N (linking table), 1:1, ER diagram, naming | ✅ |
| 4 | [[04-Kieu-du-lieu-va-thiet-ke-bang\|Kiểu dữ liệu & thiết kế bảng]] | CHAR/VARCHAR, DATE, DECIMAL, NULL, CREATE TABLE | ✅ |
| 5 | [[05-Chuan-hoa-va-phi-chuan-hoa\|Chuẩn hóa & phi chuẩn hóa]] | 1NF/2NF/3NF, denormalization, trade-off | ✅ |
| 6 | [[06-ACID-va-Transaction\|ACID & Transaction]] | Atomic/Consistent/Isolated/Durable, transaction | ✅ |

## Cụm B — SQL truy vấn & thao tác (Theory + Practical)

| # | Note | Nội dung | Trạng thái |
|---|------|----------|------------|
| 7 | [[07-SQL-co-ban\|SQL cơ bản]] | DDL/DML/DCL, CRUD, SELECT/WHERE/LIKE, ORDER BY | ✅ |
| 8 | [[08-Aggregate-va-GROUP-BY\|Aggregate & GROUP BY]] | COUNT/SUM/AVG, GROUP BY, HAVING vs WHERE | ✅ |
| 9 | [[09-JOIN\|JOIN]] | INNER/LEFT/RIGHT/FULL, join nhiều bảng | ✅ |
| 10 | [[10-Thao-tac-du-lieu-INSERT-UPDATE-DELETE\|INSERT/UPDATE/DELETE]] | thao tác an toàn (WHERE, SELECT trước), subquery | ✅ |
| 11 | [[11-SQL-nang-cao\|SQL nâng cao]] | subquery, CTE, window RANK(), CASE | ✅ |

## Cụm C — Quản trị & bảo mật (Theory)

| # | Note | Nội dung | Trạng thái |
|---|------|----------|------------|
| 12 | [[12-Index-Transaction-StoredProcedure\|Index, Transaction & Stored Procedure]] | index trade-off, transaction, stored proc | ✅ |
| 13 | [[13-Phan-quyen-Compliance-SQL-Injection\|Phân quyền, Compliance & SQL Injection]] | GRANT, PII/HIPAA/GDPR, injection & phòng chống | ✅ |

## Cụm D — PostgreSQL (Theory)

| # | Note | Nội dung | Trạng thái |
|---|------|----------|------------|
| 14 | [[14-Gioi-thieu-PostgreSQL\|Giới thiệu PostgreSQL]] | ORDBMS, MVCC, JSONB, index types | ✅ |
| 15 | [[15-PostgreSQL-vs-cac-DB-khac\|PostgreSQL vs các DB khác]] | relational vs NoSQL, scaling, replication | ✅ |
| 16 | [[16-Lam-viec-voi-PostgreSQL\|Làm việc với PostgreSQL]] | psql, pgAdmin, deploy, extension | ✅ |

## ⭐ Cụm E — Thực hành (Practical Test)

| # | Note | Nội dung | Trạng thái |
|---|------|----------|------------|
| 17 | [[17-Thuc-hanh-SQL-AutoGarage\|⭐ Thực hành SQL — AutoGarage]] | 14 câu + lời giải của bạn (Cách 1 vs Cách 2) + cheatsheet | ✅ |

## Liên quan
- [[../02-Backend/00-MOC-Backend|MOC: Backend (FastAPI)]] — SQLAlchemy ORM dùng DB này
- [[../05-Cloud/00-MOC-Cloud|MOC: Cloud]] — managed PostgreSQL trên cloud
- [[../00-INDEX|🏠 Index tổng]]
