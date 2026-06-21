---
title: "SQL cơ bản — SELECT, WHERE, ORDER BY"
section: 03-Database
tags: [database, sql, select, where, order-by, crud, ddl, dml, fresher]
related:
  - "[[04-Kieu-du-lieu-va-thiet-ke-bang]]"
  - "[[08-Aggregate-va-GROUP-BY]]"
  - "[[09-JOIN]]"
difficulty: ⭐⭐
estimated_time: 25m
source: [LinkedIn "Database Foundations" - Scott Simpson, sql-answers AutoGarage]
---

# SQL cơ bản — SELECT, WHERE, ORDER BY

> [!summary] TL;DR
> **SQL** (Structured Query Language) là ngôn ngữ giao tiếp với RDBMS. Ba vai trò: **DDL** (định nghĩa cấu trúc: CREATE/ALTER/DROP), **DML** (thao tác dữ liệu: SELECT/INSERT/UPDATE/DELETE), **DCL** (kiểm soát truy cập: GRANT/REVOKE). Bốn thao tác dữ liệu cơ bản = **CRUD** (Create-Read-Update-Delete). Truy vấn đọc: `SELECT cột FROM bảng WHERE điều_kiện ORDER BY cột`. Quy ước viết HOA từ khóa cho dễ đọc. Tránh `SELECT *` trên bảng lớn ở production.

---

## 1. SQL: DDL / DML / DCL

| Vai trò | Đầy đủ | Lệnh tiêu biểu |
|---------|--------|----------------|
| **DDL** | Data Definition Language | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| **DML** | Data Manipulation Language | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| **DCL** | Data Control Language | `GRANT`, `REVOKE` |

**CRUD** = Create (`INSERT`) · Read (`SELECT`) · Update (`UPDATE`) · Delete (`DELETE`) — nền tảng mọi tương tác với dữ liệu. Dù bạn viết app cấp cao, cuối cùng vẫn dịch ra SQL.

## 2. Cấu trúc một câu truy vấn

Câu lệnh (statement) gồm các **clause** (mệnh đề), mỗi clause có **keyword** + đối tượng tác động; có thể chứa **expression** và **predicate** (điều kiện).

```sql
SELECT FirstName, LastName, Email   -- clause SELECT: chọn cột
FROM   Customers                    -- clause FROM: từ bảng nào
WHERE  State = 'CA'                  -- clause WHERE: lọc hàng
ORDER  BY LastName ASC;             -- clause ORDER BY: sắp xếp
```

- `SELECT *` = lấy **mọi cột** (wildcard). Tiện khi khám phá, **nhưng** tốn tài nguyên trên bảng lớn → production nên liệt kê cột.
- Kết thúc bằng `;`.

## 3. WHERE — lọc hàng

| Toán tử | Ý nghĩa | Ví dụ |
|---------|---------|-------|
| `=`, `<>`, `<`, `>`, `<=`, `>=` | so sánh | `WHERE State = 'CA'` |
| `AND`, `OR`, `NOT` | logic | `WHERE State='CA' OR State='CO'` |
| `LIKE` + `%` / `_` | khớp mẫu (`%`=nhiều ký tự, `_`=1 ký tự) | `WHERE State LIKE 'C%'` |
| `IN (…)` | thuộc tập | `WHERE State IN ('CA','CO')` |
| `BETWEEN … AND …` | trong khoảng | `WHERE Date BETWEEN '2024-01-01' AND '2024-01-31'` |
| `IS NULL` / `IS NOT NULL` | kiểm tra NULL | `WHERE Phone IS NOT NULL` |

> [!warning] Lọc theo ngày khi cột là DATETIME
> Nếu cột là `DATETIME` (có phần giờ), `WHERE Date = '2024-02-06'` sẽ **trượt** (vì có giờ). Dùng **khoảng**: `WHERE Date >= '2024-02-06' AND Date < '2024-02-07'`. Nếu cột là `DATE` thuần thì `=` mới đúng.

## 4. ORDER BY — sắp xếp

```sql
SELECT * FROM Dishes ORDER BY Name ASC;    -- A→Z (mặc định)
SELECT * FROM Dishes ORDER BY Price DESC;  -- cao→thấp
```
`ASC` (mặc định) tăng dần, `DESC` giảm dần. Sắp xếp được trên mọi kiểu có thứ tự (số, ngày, chuỗi). Có thể sắp theo nhiều cột.

## 5. 📌 Ví dụ từ bài của bạn (AutoGarage)

**Câu 3** — Xe biển bắt đầu `93A` ở quận `Cam Le`:
```sql
SELECT * FROM Car
WHERE LicensePlate LIKE '93A%' AND District = 'Cam Le';
```
→ `LIKE '93A%'` khớp tiền tố biển số; kết hợp `AND` hai điều kiện.

**Câu 5** — dùng nhiều khóa sắp xếp + `CASE` đổi số lần thành chữ (xem [[11-SQL-nang-cao]]):
```sql
... ORDER BY c.CarID ASC, c.LicensePlate DESC;
```
→ sắp tăng theo `CarID`, trong cùng CarID thì giảm theo `LicensePlate`.

```
★ Insight ─────────────────────────────────────
• Thứ tự THỰC THI khác thứ tự VIẾT: DB chạy FROM → WHERE → GROUP BY → HAVING
  → SELECT → ORDER BY. Vì vậy alias đặt ở SELECT thường chưa dùng được trong
  WHERE (xem [[08-Aggregate-va-GROUP-BY]]).
• 'CA' (nháy đơn) là chuỗi; "CA" (nháy kép) trong SQL chuẩn là ĐỊNH DANH cột.
  Luôn dùng nháy đơn cho giá trị chuỗi.
─────────────────────────────────────────────────
```

## Tự kiểm tra
1. DDL/DML/DCL khác nhau gì? *(định nghĩa cấu trúc / thao tác dữ liệu / kiểm soát truy cập)*
2. CRUD ↔ lệnh SQL? *(INSERT/SELECT/UPDATE/DELETE)*
3. `LIKE 'C%'` khớp gì? *(chuỗi bắt đầu bằng C; % = nhiều ký tự)*
4. Vì sao lọc DATETIME bằng `=` ngày dễ trượt? *(có phần giờ → dùng khoảng >= … < ngày sau)*
5. ASC vs DESC? *(tăng dần / giảm dần)*

## Liên quan
- [[00-MOC-Database|⬅ MOC Database]]
- Trước: [[06-ACID-va-Transaction]] · Kế tiếp: [[08-Aggregate-va-GROUP-BY]]
- [[17-Thuc-hanh-SQL-AutoGarage|Thực hành đầy đủ 14 câu]]
