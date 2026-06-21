---
title: "Index, Transaction & Stored Procedure"
section: 03-Database
tags: [database, index, transaction, stored-procedure, performance, fresher]
related:
  - "[[06-ACID-va-Transaction]]"
  - "[[13-Phan-quyen-Compliance-SQL-Injection]]"
  - "[[14-Gioi-thieu-PostgreSQL]]"
difficulty: ⭐⭐⭐
estimated_time: 25m
source: [LinkedIn "Database Foundations" - Scott Simpson]
---

# Index, Transaction & Stored Procedure

> [!summary] TL;DR
> **Index** = "mục lục" cho cột, giúp tra cứu nhanh (khỏi quét cả bảng) — nhưng **làm chậm INSERT/UPDATE** và tốn chỗ (trade-off). PK tự là một index. **Transaction** gom nhiều lệnh thành khối all-or-nothing (ACID — [[06-ACID-va-Transaction]]). **Stored procedure** = "chương trình" lưu trên server DB, chứa loạt lệnh tái sử dụng; còn dùng để **bảo vệ dữ liệu nhạy cảm** (chỉ cho truy cập qua procedure thay vì SQL trực tiếp).

---

## 1. Index — tăng tốc tra cứu

- Không index: tìm theo `LastName` → DB **quét toàn bảng (full scan)**, so từng hàng.
- Có index trên cột: DB lưu **tham chiếu giá trị → vị trí**, trả kết quả nhanh hơn nhiều (như mục lục cuối sách).
- PK **đã là** một index.

| | Lợi | Hại |
|--|-----|-----|
| Index | đọc/tra cứu **nhanh** | INSERT/UPDATE **chậm hơn** (phải cập nhật index), tốn dung lượng |

> [!tip] Trade-off
> Index không miễn phí. Tạo index cho cột **hay tra cứu/lọc/join**; đừng index bừa mọi cột. Đây là một tối ưu có đánh đổi giống denormalization ([[05-Chuan-hoa-va-phi-chuan-hoa]]).

PostgreSQL có nhiều **loại index**: **BTREE** (mặc định, hợp đa số), Hash, GiST, SP-GiST, GIN, BRIN — chọn theo kiểu truy vấn ([[14-Gioi-thieu-PostgreSQL]]).

## 2. Transaction (nhắc lại trong thực thi)

Gom query/statement thành **khối**: lỗi 1 phần → **rollback** toàn bộ, tránh để DB ở trạng thái dở dang. Cú pháp khái niệm:
```sql
BEGIN;
  UPDATE Accounts SET Balance = Balance - 100 WHERE Id = 1;
  UPDATE Accounts SET Balance = Balance + 100 WHERE Id = 2;
COMMIT;   -- hoặc ROLLBACK nếu có lỗi
```
→ Cần khi một hành động gồm **nhiều bước phải cùng thành công**. Chi tiết ACID: [[06-ACID-va-Transaction]].

## 3. Stored Procedure

= một **chương trình lưu trên server DB**, chứa loạt lệnh ta gọi lại khi cần.

| Lợi ích | Giải thích |
|---------|-----------|
| **Tái sử dụng** | khỏi viết lại query dài/phức tạp hay dùng |
| **Bảo mật** | DBA chỉ cho thao tác **qua procedure** (input kiểm soát, chạy transaction, kiểm tra kết quả) thay vì cho chạy SQL trực tiếp lên bảng/cột nhạy cảm |
| **Nhất quán** | logic nghiệp vụ tập trung một chỗ |

```
★ Insight ─────────────────────────────────────
• Index tăng tốc ĐỌC nhưng phạt GHI — DB ghi-nhiều (OLTP nặng insert) phải cân
  nhắc; DB đọc-nhiều (analytics/OLAP) hưởng lợi rõ. Luôn đo trước khi thêm.
• Stored procedure + giới hạn quyền là một LỚP phòng SQL injection: nếu app
  chỉ gọi procedure (tham số hóa) thay vì ghép chuỗi SQL, bề mặt tấn công
  giảm hẳn ([[13-Phan-quyen-Compliance-SQL-Injection]]).
─────────────────────────────────────────────────
```

## Tự kiểm tra
1. Index giúp gì và đánh đổi gì? *(tra cứu nhanh; chậm INSERT/UPDATE + tốn chỗ)*
2. PK có phải index không? *(có — PK tự là một index)*
3. Index mặc định của PostgreSQL? *(BTREE)*
4. Transaction giải quyết vấn đề gì? *(all-or-nothing; rollback khi lỗi giữa chừng)*
5. Hai công dụng của stored procedure? *(tái sử dụng query; bảo vệ dữ liệu nhạy cảm/kiểm soát truy cập)*

## Liên quan
- [[00-MOC-Database|⬅ MOC Database]]
- Trước: [[11-SQL-nang-cao]] · Kế tiếp: [[13-Phan-quyen-Compliance-SQL-Injection]]
- [[06-ACID-va-Transaction|ACID đầy đủ]] · [[14-Gioi-thieu-PostgreSQL|Các loại index Postgres]]
