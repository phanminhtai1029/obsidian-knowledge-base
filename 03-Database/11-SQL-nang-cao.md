---
title: "SQL nâng cao — Subquery, CTE, Window function, CASE"
section: 03-Database
tags: [database, sql, subquery, cte, window-function, rank, case, fresher]
related:
  - "[[08-Aggregate-va-GROUP-BY]]"
  - "[[09-JOIN]]"
  - "[[17-Thuc-hanh-SQL-AutoGarage]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 35m
source: [sql-answers AutoGarage (bài tự giải), LinkedIn "Database Foundations"]
---

# SQL nâng cao — Subquery, CTE, Window function, CASE

> [!summary] TL;DR
> **Subquery** = truy vấn lồng trong truy vấn (dùng kết quả query này làm điều kiện query kia): `IN`/`NOT IN`, scalar `= (...)`, **correlated** (tham chiếu bảng ngoài). **CTE** (`WITH name AS (...)`) = bảng tạm có tên, làm câu phức dễ đọc, có thể đệ quy. **Window function** (`RANK()/ROW_NUMBER()/SUM() OVER (PARTITION BY … ORDER BY …)`) tính trên "cửa sổ" hàng mà **không gộp** mất hàng. **CASE** = if-else trong SQL. Đây là các kỹ thuật ở "Cách 2" trong bài AutoGarage của bạn — thường gọn & nhanh hơn subquery lồng nhiều tầng.

---

## 1. Subquery (truy vấn con)

| Dạng | Ví dụ | Ghi chú |
|------|-------|---------|
| **Scalar** (trả 1 giá trị) | `WHERE Fee = (SELECT MAX(Fee) FROM …)` | so sánh trực tiếp |
| **IN / NOT IN** (trả 1 cột) | `WHERE CarID IN (SELECT CarID FROM …)` | thuộc/không thuộc tập |
| **Correlated** (tham chiếu bảng ngoài) | `… WHERE c2.CarBrandCode = cb.CarBrandCode` | chạy lại cho mỗi hàng ngoài → chậm hơn |

**Câu 6 (Cách 2)** — dùng `IN`:
```sql
SELECT EmployeeID, EmployeeName, Address, PhoneNumber
FROM Employee
WHERE EmployeeID IN (
    SELECT sh.EmployeeID FROM ServiceHistory sh
    JOIN Car c ON sh.CarID = c.CarID
    JOIN CarBrand cb ON c.CarBrandCode = cb.CarBrandCode
    WHERE cb.CarBrandName = 'Hyundai' AND sh.ServiceFee > 2000
);
```

**Câu 7** — `NOT IN` (nhân viên bảo dưỡng Honda Q1 nhưng **chưa từng** đụng Mercedes):
```sql
... WHERE cb.CarBrandName = 'Honda'
    AND sh.ServiceDate BETWEEN '2024-01-01' AND '2024-03-31'
    AND e.EmployeeID NOT IN (
        SELECT sh2.EmployeeID FROM ServiceHistory sh2
        JOIN Car c2 ON sh2.CarID = c2.CarID
        JOIN CarBrand cb2 ON c2.CarBrandCode = cb2.CarBrandCode
        WHERE cb2.CarBrandName = 'Mercedes'
    );
```

## 2. CTE — `WITH` (Common Table Expression)

Bảng tạm có tên, viết **trước** câu chính → dễ đọc hơn subquery lồng sâu:
```sql
WITH BrandCount AS (
    SELECT c.CarBrandCode, COUNT(*) AS cnt,
           RANK() OVER (ORDER BY COUNT(*) DESC) AS rk
    FROM ServiceHistory sh
    JOIN Car c ON sh.CarID = c.CarID
    WHERE EXTRACT(YEAR FROM sh.ServiceDate) = 2024
    GROUP BY c.CarBrandCode
)
SELECT cb.CarBrandCode, cb.CarBrandName, bc.cnt AS ServiceNumber
FROM BrandCount bc
JOIN CarBrand cb ON bc.CarBrandCode = cb.CarBrandCode
WHERE bc.rk = 1;
```
*(Câu 8, Cách 2 — gọn hơn hẳn subquery MAX hai tầng ở [[08-Aggregate-va-GROUP-BY]].)*

## 3. Window function ⭐

`hàm() OVER (PARTITION BY … ORDER BY …)` — tính trên một **cửa sổ** hàng, **giữ nguyên** số hàng (khác `GROUP BY` gộp lại).

| Hàm | Tác dụng |
|-----|----------|
| `RANK()` | hạng, **nhảy bậc** khi đồng hạng (1,1,3) |
| `DENSE_RANK()` | hạng, **không nhảy** (1,1,2) |
| `ROW_NUMBER()` | số thứ tự duy nhất |
| `SUM()/AVG() OVER(...)` | running total / trung bình theo cửa sổ |

- `PARTITION BY` = chia cửa sổ theo nhóm (như GROUP BY nhưng không gộp hàng).
- **Câu 11 (Cách 2)** dùng `RANK() OVER (PARTITION BY c.CarBrandCode ORDER BY COUNT(*) DESC)` để tìm phụ tùng dùng nhiều nhất **theo từng hãng**.

> [!tip] Mẫu "top-1 mỗi nhóm"
> `RANK() OVER (PARTITION BY nhóm ORDER BY chỉ_số DESC)` rồi `WHERE rk = 1` → lấy hàng đứng đầu mỗi nhóm. Thay thế gọn cho subquery `MAX` lồng. Đây là khác biệt cốt lõi giữa **Cách 1 (subquery)** và **Cách 2 (CTE+window)** trong bài của bạn.

## 4. CASE — if/else trong SQL

```sql
CASE sh.ServiceNumber
     WHEN 1 THEN 'first time'
     WHEN 2 THEN 'second time'
     WHEN 3 THEN 'third time'
     WHEN 4 THEN 'fourth time'
     ELSE CONCAT(CAST(sh.ServiceNumber AS CHAR), 'th time')
END AS ServiceNumber
```
*(Câu 5 — đổi số lần bảo dưỡng thành chữ.)*

```
★ Insight ─────────────────────────────────────
• Cách 1 (subquery MAX hai tầng) vs Cách 2 (CTE + RANK): cùng kết quả nhưng
  Cách 2 dễ đọc, dễ sửa, và thường nhanh hơn vì không phải chạy subquery
  tương quan lặp lại. Trong phỏng vấn, biết CẢ HAI và giải thích trade-off
  ghi điểm hơn chỉ thuộc một.
• RANK vs DENSE_RANK vs ROW_NUMBER khác nhau ở cách xử lý đồng hạng — hay bị
  hỏi. Muốn "top 1 kể cả đồng hạng" dùng RANK; muốn đúng 1 hàng dùng ROW_NUMBER.
─────────────────────────────────────────────────
```

## Tự kiểm tra
1. Correlated subquery khác subquery thường? *(tham chiếu bảng ngoài, chạy lại mỗi hàng → chậm)*
2. CTE khai báo bằng từ khóa gì, lợi ích? *(`WITH` — dễ đọc, tái dùng, đệ quy được)*
3. Window function khác GROUP BY? *(không gộp mất hàng; tính trên cửa sổ, giữ nguyên hàng)*
4. RANK vs DENSE_RANK vs ROW_NUMBER? *(nhảy bậc / không nhảy / số thứ tự duy nhất khi đồng hạng)*
5. Mẫu "top-1 mỗi nhóm"? *(RANK() OVER(PARTITION BY nhóm ORDER BY … DESC) rồi WHERE rk=1)*

## Liên quan
- [[00-MOC-Database|⬅ MOC Database]]
- Trước: [[10-Thao-tac-du-lieu-INSERT-UPDATE-DELETE]] · Kế tiếp: [[12-Index-Transaction-StoredProcedure]]
- [[08-Aggregate-va-GROUP-BY|Cách 1 (subquery MAX)]] · [[17-Thuc-hanh-SQL-AutoGarage|Câu 8,9,11 - so sánh 2 cách]]
