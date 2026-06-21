---
title: "Thao tác dữ liệu — INSERT, UPDATE, DELETE"
section: 03-Database
tags: [database, sql, insert, update, delete, dml, fresher]
related:
  - "[[07-SQL-co-ban]]"
  - "[[02-Keys-va-rang-buoc]]"
  - "[[13-Phan-quyen-Compliance-SQL-Injection]]"
difficulty: ⭐⭐
estimated_time: 20m
source: [LinkedIn "Database Foundations" - Scott Simpson, sql-answers AutoGarage]
---

# Thao tác dữ liệu — INSERT, UPDATE, DELETE

> [!summary] TL;DR
> `INSERT INTO bảng (cột…) VALUES (…)` thêm hàng (cột bỏ trống → NULL/default; vi phạm NOT NULL → lỗi). `UPDATE bảng SET cột=giá_trị WHERE …` sửa hàng. `DELETE FROM bảng WHERE …` xóa hàng. **Nguyên tắc sống còn**: UPDATE/DELETE **luôn có `WHERE`** và nên **`SELECT` kiểm tra trước** — DB **không có nút undo**; quên `WHERE` sẽ sửa/xóa **toàn bộ bảng**. Nhắm mục tiêu bằng **khóa** (CustomerID), đừng dựa vào tên (dễ trúng nhiều hàng).

---

## 1. INSERT — thêm hàng (Create)

```sql
INSERT INTO Employee (EmployeeID, EmployeeName, Gender, DateofBirth, Address, PhoneNumber)
VALUES ('ED007', 'Nguyen Thi Lan', 'Female', '1992-02-14', '10 Bach Dang', '0901122334');
```
- Cột không cung cấp → nhận **NULL** hoặc **default**.
- Nếu thiếu cột có ràng buộc `NOT NULL` → DB **từ chối** (constraint bảo vệ dữ liệu).
- Insert nhiều hàng: nhiều bộ `VALUES (...), (...)` (xem seed data Câu 2 AutoGarage).

## 2. UPDATE — sửa hàng (Update)

```sql
-- BƯỚC 1: luôn SELECT kiểm tra mình nhắm đúng hàng
SELECT * FROM Customers WHERE CustomerID = 1;
-- BƯỚC 2: update theo KHÓA (không theo tên)
UPDATE Customers
SET Email = 'tjenkins@landonhotel.com'
WHERE CustomerID = 1;
```

> [!warning] Quên WHERE = thảm họa
> `UPDATE Customers SET Email='x'` (không WHERE) đổi email **mọi khách**. Không undo. Luôn kèm `WHERE` thật cụ thể, ưu tiên khóa duy nhất.

## 3. DELETE — xóa hàng (Delete)

```sql
DELETE FROM Customers WHERE CustomerID = 26;
```
- Xóa theo **khóa** để không trúng nhầm (xóa theo tên "Taylor Jenkins" có thể xóa 2 người).
- Nếu có `FOREIGN KEY` + **cascading delete** → xóa hàng cha sẽ tự xóa hàng con liên quan ([[02-Keys-va-rang-buoc]]).

## 4. 📌 UPDATE/DELETE có subquery (bài của bạn)

**Câu 13** — đặt phí = 500 cho lịch sử ngày 2024-11-10 của xe **hãng Hyundai**:
```sql
UPDATE ServiceHistory
SET ServiceFee = 500
WHERE ServiceDate = '2024-11-10'
  AND CarID IN (
      SELECT CarID FROM Car
      WHERE CarBrandCode = (
          SELECT CarBrandCode FROM CarBrand WHERE CarBrandName = 'Hyundai'
      )
  );
```

**Câu 14** — xóa nhân viên **nữ** chưa từng thực hiện bảo dưỡng:
```sql
DELETE FROM Employee
WHERE Gender = 'Female'
  AND EmployeeID NOT IN (
      SELECT DISTINCT EmployeeID FROM ServiceHistory
  );
```
→ Mẹo của bạn: **comment dòng `SELECT`/`INSERT` thử nghiệm** ở trên trước khi chạy DELETE thật — đúng tinh thần "kiểm tra trước khi sửa".

```
★ Insight ─────────────────────────────────────
• NOT IN + subquery NULL là cái bẫy: nếu subquery trả về dù chỉ một NULL, NOT
  IN có thể ra rỗng. Ở Câu 14 an toàn vì EmployeeID trong ServiceHistory
  NOT NULL; nói chung nên cân nhắc NOT EXISTS để tránh bẫy NULL.
• Bọc UPDATE/DELETE nhiều bước trong TRANSACTION ([[06-ACID-va-Transaction]])
  để rollback được nếu sai — an toàn hơn chạy lệnh trần.
─────────────────────────────────────────────────
```

## Tự kiểm tra
1. INSERT thiếu cột NOT NULL thì sao? *(DB từ chối — constraint chặn)*
2. Vì sao UPDATE/DELETE phải có WHERE? *(không có undo; thiếu WHERE sửa/xóa toàn bảng)*
3. Nên nhắm hàng bằng gì? *(khóa duy nhất, không phải tên)*
4. Cascading delete làm gì? *(xóa hàng cha tự xóa hàng con tham chiếu)*
5. Thói quen an toàn trước khi UPDATE/DELETE? *(SELECT kiểm tra trước; bọc transaction)*

## Liên quan
- [[00-MOC-Database|⬅ MOC Database]]
- Trước: [[09-JOIN]] · Kế tiếp: [[11-SQL-nang-cao]]
- [[06-ACID-va-Transaction|Bọc trong transaction]] · [[17-Thuc-hanh-SQL-AutoGarage|Câu 13,14]]
