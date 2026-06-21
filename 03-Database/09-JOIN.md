---
title: "JOIN — truy vấn xuyên các bảng"
section: 03-Database
tags: [database, sql, join, inner-join, left-join, fresher]
related:
  - "[[03-Quan-he-va-mo-hinh-ER]]"
  - "[[08-Aggregate-va-GROUP-BY]]"
  - "[[11-SQL-nang-cao]]"
difficulty: ⭐⭐⭐
estimated_time: 25m
source: [LinkedIn "Database Foundations" - Scott Simpson, sql-answers AutoGarage]
---

# JOIN — truy vấn xuyên các bảng

> [!summary] TL;DR
> `JOIN` nối hàng từ ≥2 bảng dựa trên điều kiện `ON` (thường FK = PK). **INNER JOIN** chỉ giữ hàng **khớp ở cả hai** bảng; **LEFT JOIN** giữ **mọi hàng bảng trái** (bên phải không khớp → NULL); **RIGHT JOIN** ngược lại; **FULL JOIN** giữ cả hai phía. Không có `ON` đúng sẽ ra **tích Descartes** (mỗi hàng A × mỗi hàng B). Dùng **alias** (`Car c`) cho gọn khi join nhiều bảng.

---

## 1. Vì sao cần JOIN

Bảng Customers lưu `FavoriteDish = 15` (chỉ là số). Muốn thấy **tên món** → nối sang Dishes nơi `DishID = 15`:
```sql
SELECT c.FirstName, c.LastName, d.Name
FROM Customers c
JOIN Dishes d ON c.FavoriteDish = d.DishID;
```
- `ON` = điều kiện khớp (FK ↔ PK).
- **Thiếu `ON`** → mỗi hàng Customers ghép với **mọi** hàng Dishes (vô nghĩa, tích Descartes).

## 2. Các loại JOIN ⭐

| Loại | Giữ lại | Sơ đồ |
|------|---------|-------|
| **INNER JOIN** | chỉ hàng khớp **cả hai** bảng | `A ∩ B` |
| **LEFT (OUTER) JOIN** | **mọi** hàng trái + khớp phải (không khớp → NULL) | `A` (toàn bộ) |
| **RIGHT (OUTER) JOIN** | **mọi** hàng phải + khớp trái | `B` (toàn bộ) |
| **FULL (OUTER) JOIN** | mọi hàng cả hai phía | `A ∪ B` |
| **CROSS JOIN** | tích Descartes (mọi tổ hợp) | `A × B` |

> [!tip] Khi nào LEFT JOIN
> Khi muốn giữ **cả hàng không có bản ghi liên quan**. Vd "liệt kê **mọi xe**, kể cả xe **chưa từng** bảo dưỡng" → `LEFT JOIN ServiceHistory` (xe chưa bảo dưỡng vẫn hiện, cột service = NULL). `INNER JOIN` sẽ **loại** các xe đó.

## 3. JOIN nhiều bảng (chuỗi quan hệ)

Bài AutoGarage nối chuỗi `Employee → ServiceHistory → Car → CarBrand`:
```sql
SELECT DISTINCT e.EmployeeID, e.EmployeeName, e.Address, e.PhoneNumber
FROM Employee e
JOIN ServiceHistory sh ON e.EmployeeID = sh.EmployeeID
JOIN Car c           ON sh.CarID = c.CarID
JOIN CarBrand cb     ON c.CarBrandCode = cb.CarBrandCode
WHERE cb.CarBrandName = 'Hyundai' AND sh.ServiceFee > 2000;
```
*(Câu 6 — nhân viên từng bảo dưỡng xe Hyundai với phí > 2000.)* `DISTINCT` loại trùng khi 1 nhân viên có nhiều bản ghi khớp.

## 4. 📌 LEFT JOIN trong bài của bạn (Câu 5)

```sql
SELECT c.CarID, c.LicensePlate, cb.CarBrandName, ...
FROM Car c
JOIN CarBrand cb ON c.CarBrandCode = cb.CarBrandCode
LEFT JOIN ServiceHistory sh ON c.CarID = sh.CarID    -- giữ cả xe chưa bảo dưỡng
ORDER BY c.CarID ASC, c.LicensePlate DESC;
```
→ Dùng `LEFT JOIN` để **không bỏ sót** xe chưa có lịch sử bảo dưỡng.

```
★ Insight ─────────────────────────────────────
• Lỗi kinh điển: dùng INNER JOIN khi đề yêu cầu "tất cả X (kể cả chưa có Y)"
  → mất hàng. Hễ thấy chữ "tất cả/kể cả/chưa từng" → nghĩ LEFT JOIN.
• Điều kiện đặt ở ON vs WHERE khác nhau với OUTER JOIN: lọc ở WHERE trên cột
  bảng phải có thể vô tình biến LEFT JOIN thành INNER JOIN (loại hàng NULL).
─────────────────────────────────────────────────
```

## Tự kiểm tra
1. INNER vs LEFT JOIN? *(INNER: chỉ hàng khớp cả hai; LEFT: mọi hàng trái, phải không khớp = NULL)*
2. Thiếu ON gây ra gì? *(tích Descartes / CROSS JOIN)*
3. Khi nào bắt buộc LEFT JOIN? *(khi cần giữ hàng không có bản ghi liên quan — "tất cả/kể cả")*
4. DISTINCT trong câu JOIN để làm gì? *(loại hàng trùng khi 1 hàng trái khớp nhiều hàng phải)*
5. FULL JOIN giữ gì? *(mọi hàng cả hai phía, A ∪ B)*

## Liên quan
- [[00-MOC-Database|⬅ MOC Database]]
- Trước: [[08-Aggregate-va-GROUP-BY]] · Kế tiếp: [[10-Thao-tac-du-lieu-INSERT-UPDATE-DELETE]]
- [[03-Quan-he-va-mo-hinh-ER|Quan hệ là nền của JOIN]] · [[17-Thuc-hanh-SQL-AutoGarage|Câu 4,5,6,7,10]]
