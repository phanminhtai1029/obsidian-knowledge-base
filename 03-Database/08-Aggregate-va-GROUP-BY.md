---
title: "Aggregate functions & GROUP BY"
section: 03-Database
tags: [database, sql, aggregate, group-by, having, count, sum, fresher]
related:
  - "[[07-SQL-co-ban]]"
  - "[[09-JOIN]]"
  - "[[11-SQL-nang-cao]]"
difficulty: ⭐⭐⭐
estimated_time: 25m
source: [LinkedIn "Database Foundations" - Scott Simpson, sql-answers AutoGarage]
---

# Aggregate functions & GROUP BY

> [!summary] TL;DR
> **Aggregate function** gộp nhiều hàng thành **một giá trị**: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`. `GROUP BY cột` chia dữ liệu thành **nhóm** rồi áp hàm gộp cho từng nhóm. **`HAVING`** lọc **sau khi gộp** (lọc trên kết quả aggregate), khác **`WHERE`** lọc **trước khi gộp** (lọc trên hàng thô). `COUNT(col)` chỉ đếm hàng **không NULL** ở cột đó; `COUNT(*)` đếm mọi hàng.

---

## 1. Các hàm gộp

| Hàm | Trả về |
|-----|--------|
| `COUNT(col)` | số hàng có giá trị (không NULL) ở `col` |
| `COUNT(*)` | tổng số hàng |
| `SUM(col)` | tổng |
| `AVG(col)` | trung bình |
| `MIN(col)` / `MAX(col)` | nhỏ nhất / lớn nhất |

```sql
SELECT COUNT(*) FROM Customers WHERE State = 'CA';  -- 16
SELECT SUM(Price), AVG(Price), MIN(Price), MAX(Price) FROM Dishes;
```

## 2. GROUP BY — gộp theo nhóm

```sql
SELECT State, COUNT(*) AS Num
FROM Customers
GROUP BY State;
```
→ mỗi `State` một hàng kết quả, kèm số khách của bang đó.

> [!tip] Quy tắc vàng
> Mọi cột trong `SELECT` **không** nằm trong hàm gộp thì **phải** xuất hiện trong `GROUP BY`. (Vd `SELECT State, COUNT(*)` → `GROUP BY State`.)

## 3. HAVING vs WHERE ⭐ (cực hay hỏi)

| | `WHERE` | `HAVING` |
|--|---------|----------|
| Lọc | **hàng thô** (trước gộp) | **nhóm** (sau gộp) |
| Dùng được aggregate? | ❌ không | ✅ có |
| Thứ tự thực thi | trước `GROUP BY` | sau `GROUP BY` |

```sql
SELECT c.CarID, c.LicensePlate, COUNT(*) AS ServiceNumber
FROM Car c
JOIN ServiceHistory sh ON c.CarID = sh.CarID
GROUP BY c.CarID, c.LicensePlate
HAVING COUNT(*) >= 2;          -- lọc nhóm có ≥2 lần bảo dưỡng
```
*(Câu 12 trong bài AutoGarage của bạn.)*

## 4. 📌 Mẫu "tìm nhóm có giá trị lớn nhất" (từ bài của bạn)

Bài AutoGarage có dạng câu hay gặp: **"hãng xe bảo dưỡng nhiều nhất 2024"** (Câu 8), **"nhân viên doanh thu cao nhất tháng 12"** (Câu 9). Mẫu **subquery + MAX của nhóm**:
```sql
SELECT cb.CarBrandCode, cb.CarBrandName, COUNT(*) AS ServiceNumber
FROM ServiceHistory sh
JOIN Car c ON sh.CarID = c.CarID
JOIN CarBrand cb ON c.CarBrandCode = cb.CarBrandCode
WHERE EXTRACT(YEAR FROM sh.ServiceDate) = 2024
GROUP BY cb.CarBrandCode, cb.CarBrandName
HAVING COUNT(*) = (                       -- chỉ giữ nhóm = giá trị lớn nhất
    SELECT MAX(cnt) FROM (
        SELECT COUNT(*) AS cnt
        FROM ServiceHistory sh2
        JOIN Car c2 ON sh2.CarID = c2.CarID
        WHERE EXTRACT(YEAR FROM sh2.ServiceDate) = 2024
        GROUP BY c2.CarBrandCode
    ) AS t
);
```
→ "Cách 2" của bạn dùng **window function `RANK()`** gọn hơn — xem [[11-SQL-nang-cao]].

```
★ Insight ─────────────────────────────────────
• "Lấy max của các count" KHÔNG viết được bằng MAX(COUNT(*)) lồng trực tiếp →
  phải gộp 2 lần (subquery con đếm theo nhóm, rồi MAX bên ngoài). Đây là lý do
  window function ra đời để viết gọn hơn.
• COUNT(col) bỏ qua NULL — nên khi đếm "số bản ghi", đếm trên cột NOT NULL
  (hoặc khóa) để không hụt; hoặc dùng COUNT(*).
─────────────────────────────────────────────────
```

## Tự kiểm tra
1. Kể 5 hàm gộp? *(COUNT, SUM, AVG, MIN, MAX)*
2. WHERE khác HAVING? *(WHERE lọc hàng trước gộp, không aggregate; HAVING lọc nhóm sau gộp, có aggregate)*
3. Cột nào bắt buộc vào GROUP BY? *(mọi cột SELECT không nằm trong hàm gộp)*
4. `COUNT(col)` vs `COUNT(*)`? *(bỏ NULL ở col / đếm mọi hàng)*
5. Vì sao cần subquery để lấy "nhóm count lớn nhất"? *(không lồng MAX(COUNT()) trực tiếp được → gộp 2 lần)*

## Liên quan
- [[00-MOC-Database|⬅ MOC Database]]
- Trước: [[07-SQL-co-ban]] · Kế tiếp: [[09-JOIN]]
- [[11-SQL-nang-cao|RANK() thay subquery MAX]] · [[17-Thuc-hanh-SQL-AutoGarage|Câu 8,9,12]]
