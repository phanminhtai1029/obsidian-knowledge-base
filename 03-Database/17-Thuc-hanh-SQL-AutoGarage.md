---
title: "⭐ Thực hành SQL — AutoGarage (14 câu + lời giải của bạn)"
section: 03-Database
tags: [database, sql, practice, practical-test, autogarage, fresher]
related:
  - "[[07-SQL-co-ban]]"
  - "[[09-JOIN]]"
  - "[[11-SQL-nang-cao]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 60m
source: [sql-answers/ — bài tự giải của bạn (AutoGarage)]
---

# ⭐ Thực hành SQL — AutoGarage

> [!summary] TL;DR
> Bộ đề thực hành kiểu **Final Practical Test**: CSDL **AutoGarage** (gara ô tô) gồm 6 bảng. 14 câu phủ kín: **DDL** (CREATE), **DML seed** (INSERT), truy vấn `LIKE`/ngày, **JOIN** nhiều bảng, **GROUP BY/HAVING**, **subquery** (IN/NOT IN/MAX), **CTE + window `RANK()`**, **CASE**, **UPDATE/DELETE có subquery**. Đây là **bài tự giải của bạn** — nhiều câu có **Cách 1 (subquery)** và **Cách 2 (CTE + window)**; phần dưới phân tích kỹ thuật & bẫy từng câu. Học thuộc các *mẫu* (pattern) ở đây là đủ tự tin thi thực hành.

> [!tip] Cách dùng note này
> Che phần lời giải, tự viết SQL từ đề → so với bài của bạn → đọc phần "Kỹ thuật" để hiểu *vì sao*. Liên kết lý thuyết: [[07-SQL-co-ban]] · [[08-Aggregate-va-GROUP-BY]] · [[09-JOIN]] · [[10-Thao-tac-du-lieu-INSERT-UPDATE-DELETE]] · [[11-SQL-nang-cao]].

---

## 1. Lược đồ (schema) — 6 bảng

```
CarBrand (CarBrandCode PK, CarBrandName)
   ▲ 1
   │ M
Car (CarID PK, LicensePlate, Owner, Address, District, PhoneNumber, CarBrandCode FK→CarBrand)
   ▲ 1
   │ M
ServiceHistory (ServiceCode PK, CarID FK→Car, EmployeeID FK→Employee,
                ServiceNumber, ServiceFee, ServiceDate)
   ▲ 1                                   ▲ M
   │ M                                   │ 1
ServiceDetails (DetailCode PK,        Employee (EmployeeID PK, EmployeeName,
   ServiceCode FK→ServiceHistory,       Gender, DateofBirth, Address, PhoneNumber)
   PartNumber FK→Part, ServiceDescription)
   ▼ M
   │ 1
Part (PartNumber PK, PartName)
```

**Quan hệ:** CarBrand 1—M Car · Car 1—M ServiceHistory · Employee 1—M ServiceHistory · ServiceHistory 1—M ServiceDetails · Part 1—M ServiceDetails. (ServiceDetails là **bảng nối** giữa ServiceHistory và Part — xem [[03-Quan-he-va-mo-hinh-ER]].)

---

## 2. Câu 1 — DDL: tạo database & 6 bảng

```sql
CREATE DATABASE AutoGarage;

CREATE TABLE CarBrand (
    CarBrandCode VARCHAR(10) PRIMARY KEY,
    CarBrandName VARCHAR(50)
);
CREATE TABLE Car (
    CarID VARCHAR(10) PRIMARY KEY,
    LicensePlate VARCHAR(20), Owner VARCHAR(50), Address VARCHAR(100),
    District VARCHAR(50), PhoneNumber VARCHAR(15),
    CarBrandCode VARCHAR(10),
    FOREIGN KEY (CarBrandCode) REFERENCES CarBrand(CarBrandCode)
);
CREATE TABLE Employee (
    EmployeeID VARCHAR(10) PRIMARY KEY,
    EmployeeName VARCHAR(50), Gender VARCHAR(10), DateofBirth DATE,
    Address VARCHAR(100), PhoneNumber VARCHAR(15)
);
CREATE TABLE Part (
    PartNumber VARCHAR(10) PRIMARY KEY, PartName VARCHAR(50)
);
CREATE TABLE ServiceHistory (
    ServiceCode VARCHAR(10) PRIMARY KEY,
    CarID VARCHAR(10), EmployeeID VARCHAR(10),
    ServiceNumber INT, ServiceFee DECIMAL(10,2), ServiceDate DATE,
    FOREIGN KEY (CarID) REFERENCES Car(CarID),
    FOREIGN KEY (EmployeeID) REFERENCES Employee(EmployeeID)
);
CREATE TABLE ServiceDetails (
    DetailCode VARCHAR(10) PRIMARY KEY,
    ServiceCode VARCHAR(10), PartNumber VARCHAR(10), ServiceDescription VARCHAR(100),
    FOREIGN KEY (ServiceCode) REFERENCES ServiceHistory(ServiceCode),
    FOREIGN KEY (PartNumber) REFERENCES Part(PartNumber)
);
```
**Kỹ thuật:** thứ tự tạo bảng phải theo phụ thuộc FK — bảng được tham chiếu (CarBrand, Part, Employee) tạo **trước**; `ServiceDetails` tạo **cuối** vì trỏ tới 2 bảng. `DECIMAL(10,2)` cho phí, `DATE` cho ngày. → lý thuyết: [[04-Kieu-du-lieu-va-thiet-ke-bang]].

## 3. Câu 2 — Seed data (INSERT)

Chèn nhiều hàng bằng một `INSERT … VALUES (…),(…)`. *(Bộ dữ liệu mẫu: 5 hãng, nhiều xe, 7 nhân viên, 5 phụ tùng, 16 lịch sử dịch vụ + chi tiết.)*
```sql
INSERT INTO CarBrand VALUES
('H001','Hyundai'),('H002','Mercedes'),('H003','Toyota'),('H004','Honda'),('H005','Chevrolet');
-- … (Car, Employee, Part, ServiceHistory, ServiceDetails tương tự)
```
**Kỹ thuật:** insert theo đúng thứ tự FK (CarBrand trước Car…). → [[10-Thao-tac-du-lieu-INSERT-UPDATE-DELETE]].

## 4. Câu 3 — Lọc `LIKE` + nhiều điều kiện

> **Đề:** Liệt kê xe có biển bắt đầu `93A` và thuộc quận `Cam Le`.
```sql
SELECT * FROM Car
WHERE LicensePlate LIKE '93A%' AND District = 'Cam Le';
```
**Kỹ thuật:** `LIKE '93A%'` khớp tiền tố; `AND` ghép điều kiện. → [[07-SQL-co-ban]].

## 5. Câu 4 — Lọc theo tháng/năm + JOIN (2 cách)

> **Đề:** Thông tin các lần bảo dưỡng trong **tháng 2/2024** (kèm xe + nhân viên).
```sql
-- Cách 1: tách tháng & năm
WHERE EXTRACT(MONTH FROM sh.ServiceDate) = 2
  AND EXTRACT(YEAR  FROM sh.ServiceDate) = 2024;

-- Cách 2: dùng khoảng ngày
WHERE sh.ServiceDate BETWEEN '2024-02-01' AND '2024-02-29';
```
**Kỹ thuật:** `EXTRACT` rõ ràng nhưng **không dùng được index** trên cột ngày; `BETWEEN` khoảng thì tận dụng index → nhanh hơn trên bảng lớn. (Lưu ý 2024 nhuận nên 29 ngày.) → [[07-SQL-co-ban]], [[09-JOIN]].

## 6. Câu 5 — LEFT JOIN + CASE + sắp nhiều khóa

> **Đề:** Liệt kê **mọi xe** (kể cả chưa bảo dưỡng), đổi `ServiceNumber` thành chữ ('first time'…).
```sql
SELECT c.CarID, c.LicensePlate, cb.CarBrandName,
       CASE sh.ServiceNumber
            WHEN 1 THEN 'first time' WHEN 2 THEN 'second time'
            WHEN 3 THEN 'third time' WHEN 4 THEN 'fourth time'
            ELSE CONCAT(CAST(sh.ServiceNumber AS CHAR), 'th time')
       END AS ServiceNumber
FROM Car c
JOIN CarBrand cb ON c.CarBrandCode = cb.CarBrandCode
LEFT JOIN ServiceHistory sh ON c.CarID = sh.CarID
ORDER BY c.CarID ASC, c.LicensePlate DESC;
```
**Kỹ thuật:** **LEFT JOIN** giữ xe chưa có lịch sử (nếu không sẽ mất); `CASE` ánh xạ giá trị; `ORDER BY` 2 khóa khác chiều. → [[09-JOIN]], [[11-SQL-nang-cao]].

## 7. Câu 6 — JOIN chuỗi 4 bảng (2 cách: JOIN vs IN)

> **Đề:** Nhân viên từng bảo dưỡng xe **Hyundai** với phí **> 2000**.
```sql
-- Cách 1: JOIN + DISTINCT
SELECT DISTINCT e.EmployeeID, e.EmployeeName, e.Address, e.PhoneNumber
FROM Employee e
JOIN ServiceHistory sh ON e.EmployeeID = sh.EmployeeID
JOIN Car c ON sh.CarID = c.CarID
JOIN CarBrand cb ON c.CarBrandCode = cb.CarBrandCode
WHERE cb.CarBrandName = 'Hyundai' AND sh.ServiceFee > 2000;

-- Cách 2: subquery IN
SELECT EmployeeID, EmployeeName, Address, PhoneNumber FROM Employee
WHERE EmployeeID IN ( SELECT sh.EmployeeID FROM ServiceHistory sh
    JOIN Car c ON sh.CarID = c.CarID
    JOIN CarBrand cb ON c.CarBrandCode = cb.CarBrandCode
    WHERE cb.CarBrandName = 'Hyundai' AND sh.ServiceFee > 2000 );
```
**Kỹ thuật:** JOIN cần `DISTINCT` vì 1 nhân viên có thể khớp nhiều bản ghi → trùng. Subquery `IN` tự khử trùng. → [[09-JOIN]], [[11-SQL-nang-cao]].

## 8. Câu 7 — NOT IN (loại trừ tập)

> **Đề:** Nhân viên bảo dưỡng xe **Honda** trong **Q1/2024** nhưng **chưa từng** bảo dưỡng xe **Mercedes**.
```sql
SELECT DISTINCT e.*
FROM Employee e
JOIN ServiceHistory sh ON e.EmployeeID = sh.EmployeeID
JOIN Car c ON sh.CarID = c.CarID
JOIN CarBrand cb ON c.CarBrandCode = cb.CarBrandCode
WHERE cb.CarBrandName = 'Honda'
  AND sh.ServiceDate BETWEEN '2024-01-01' AND '2024-03-31'
  AND e.EmployeeID NOT IN (
      SELECT sh2.EmployeeID FROM ServiceHistory sh2
      JOIN Car c2 ON sh2.CarID = c2.CarID
      JOIN CarBrand cb2 ON c2.CarBrandCode = cb2.CarBrandCode
      WHERE cb2.CarBrandName = 'Mercedes' );
```
**Kỹ thuật:** mẫu "có X nhưng không có Y" = điều kiện dương + `NOT IN (tập Y)`. ⚠️ bẫy NULL của `NOT IN` (xem [[10-Thao-tac-du-lieu-INSERT-UPDATE-DELETE]]); ở đây an toàn vì EmployeeID NOT NULL.

## 9. Câu 8 — Nhóm có COUNT lớn nhất (2 cách) ⭐

> **Đề:** Hãng xe có **số lần bảo dưỡng nhiều nhất** năm 2024.
```sql
-- Cách 1: subquery MAX của các count
... GROUP BY cb.CarBrandCode, cb.CarBrandName
HAVING COUNT(*) = ( SELECT MAX(cnt) FROM (
    SELECT COUNT(*) AS cnt FROM ServiceHistory sh2
    JOIN Car c2 ON sh2.CarID = c2.CarID
    WHERE EXTRACT(YEAR FROM sh2.ServiceDate)=2024
    GROUP BY c2.CarBrandCode ) AS t );

-- Cách 2: CTE + RANK()
WITH BrandCount AS (
    SELECT c.CarBrandCode, COUNT(*) AS cnt,
           RANK() OVER (ORDER BY COUNT(*) DESC) AS rk
    FROM ServiceHistory sh JOIN Car c ON sh.CarID=c.CarID
    WHERE EXTRACT(YEAR FROM sh.ServiceDate)=2024
    GROUP BY c.CarBrandCode )
SELECT cb.CarBrandCode, cb.CarBrandName, bc.cnt AS ServiceNumber
FROM BrandCount bc JOIN CarBrand cb ON bc.CarBrandCode=cb.CarBrandCode
WHERE bc.rk = 1;
```
**Kỹ thuật:** không lồng `MAX(COUNT())` trực tiếp được → Cách 1 gộp 2 tầng; Cách 2 `RANK()` gọn & dễ đọc, **giữ cả đồng hạng**. → [[08-Aggregate-va-GROUP-BY]], [[11-SQL-nang-cao]].

## 10. Câu 9 — SUM lớn nhất theo nhân viên (2 cách)

> **Đề:** Nhân viên có **tổng phí dịch vụ cao nhất** tháng 12/2024.
```sql
-- Cách 1: HAVING SUM = (SELECT MAX(total) …)
-- Cách 2: CTE + RANK() OVER (ORDER BY SUM(ServiceFee) DESC), WHERE rk=1
```
**Kỹ thuật:** y như Câu 8 nhưng đổi `COUNT` → `SUM(ServiceFee)`. Cùng một *mẫu* "top-1 theo chỉ số gộp".

## 11. Câu 10 — HAVING với điều kiện đặc biệt

> **Đề:** Nhân viên **chỉ** bảo dưỡng đúng **một hãng** xe, và hãng đó là **Toyota**.
```sql
... GROUP BY e.EmployeeID, e.EmployeeName
HAVING COUNT(DISTINCT cb.CarBrandName) = 1
   AND MAX(cb.CarBrandName) = 'Toyota';
```
**Kỹ thuật:** `COUNT(DISTINCT …) = 1` ép "chỉ một hãng"; vì chỉ có 1 giá trị nên `MAX(name)='Toyota'` xác định đó là Toyota. Mẹo hay: dùng `MAX` để lấy giá trị duy nhất của nhóm trong `HAVING`.

## 12. Câu 11 — Window theo nhóm (PARTITION BY)

> **Đề:** Với **mỗi hãng**, phụ tùng được dùng **nhiều nhất**.
```sql
WITH PartCount AS (
    SELECT c.CarBrandCode, sd.PartNumber, COUNT(*) AS cnt,
           RANK() OVER (PARTITION BY c.CarBrandCode ORDER BY COUNT(*) DESC) AS rk
    FROM ServiceDetails sd
    JOIN ServiceHistory sh ON sd.ServiceCode=sh.ServiceCode
    JOIN Car c ON sh.CarID=c.CarID
    GROUP BY c.CarBrandCode, sd.PartNumber )
SELECT cb.CarBrandCode, cb.CarBrandName, p.PartNumber, p.PartName
FROM PartCount pc
JOIN CarBrand cb ON pc.CarBrandCode=cb.CarBrandCode
JOIN Part p ON pc.PartNumber=p.PartNumber
WHERE pc.rk = 1
ORDER BY cb.CarBrandCode;
```
**Kỹ thuật:** **`PARTITION BY`** = xếp hạng **trong từng hãng** (top-1 mỗi nhóm). Đây là sức mạnh window vượt subquery thường. → [[11-SQL-nang-cao]].

## 13. Câu 12 — GROUP BY + HAVING đếm

> **Đề:** Xe đã bảo dưỡng **từ 2 lần trở lên**.
```sql
SELECT c.CarID, c.LicensePlate, COUNT(*) AS ServiceNumber
FROM Car c JOIN ServiceHistory sh ON c.CarID = sh.CarID
GROUP BY c.CarID, c.LicensePlate
HAVING COUNT(*) >= 2;
```
**Kỹ thuật:** điều kiện trên **aggregate** → dùng `HAVING`, không phải `WHERE`. → [[08-Aggregate-va-GROUP-BY]].

## 14. Câu 13 — UPDATE có subquery lồng

> **Đề:** Đặt phí = 500 cho lịch sử ngày `2024-11-10` của xe **hãng Hyundai**.
```sql
UPDATE ServiceHistory SET ServiceFee = 500
WHERE ServiceDate = '2024-11-10'
  AND CarID IN ( SELECT CarID FROM Car
      WHERE CarBrandCode = ( SELECT CarBrandCode FROM CarBrand
                             WHERE CarBrandName = 'Hyundai' ) );
```
**Kỹ thuật:** subquery 2 tầng (CarBrand→Car→điều kiện UPDATE). Luôn `SELECT` thử mệnh đề `WHERE` trước khi UPDATE thật. → [[10-Thao-tac-du-lieu-INSERT-UPDATE-DELETE]].

## 15. Câu 14 — DELETE có subquery (NOT IN)

> **Đề:** Xóa nhân viên **nữ** **chưa từng** thực hiện bảo dưỡng.
```sql
DELETE FROM Employee
WHERE Gender = 'Female'
  AND EmployeeID NOT IN ( SELECT DISTINCT EmployeeID FROM ServiceHistory );
```
**Kỹ thuật:** mẫu "xóa hàng không xuất hiện ở bảng khác". Thói quen của bạn: **comment thử `INSERT`/`SELECT`** kiểm tra trước khi chạy DELETE — rất tốt (không có undo). ⚠️ cân nhắc `NOT EXISTS` nếu cột có thể NULL.

---

```
★ Insight ─────────────────────────────────────
• Cả đề xoay quanh ~6 MẪU: (1) lọc LIKE/ngày, (2) JOIN chuỗi nhiều bảng,
  (3) GROUP BY + HAVING, (4) "top-1/nhiều nhất" = subquery MAX HOẶC RANK(),
  (5) "có X không có Y" = NOT IN/NOT EXISTS, (6) UPDATE/DELETE + subquery.
  Thuộc 6 mẫu này là giải được hầu hết biến thể.
• "Cách 1 vs Cách 2": subquery chạy mọi nơi (kể cả MySQL cũ); CTE+window cần
  bản hỗ trợ (Postgres OK từ lâu) nhưng gọn/nhanh hơn. Trong phỏng vấn, trình
  bày được CẢ HAI + nói trade-off là điểm cộng lớn.
─────────────────────────────────────────────────
```

## 16. Cheatsheet nhanh (tra khi thi)

| Cần | Cú pháp |
|-----|---------|
| Lọc tiền tố | `WHERE col LIKE 'abc%'` |
| Lọc khoảng ngày | `WHERE d BETWEEN '2024-02-01' AND '2024-02-29'` |
| Lấy tháng/năm | `EXTRACT(MONTH FROM d)`, `EXTRACT(YEAR FROM d)` |
| Đếm theo nhóm + lọc | `GROUP BY … HAVING COUNT(*) >= n` |
| Top-1 mỗi nhóm | `RANK() OVER (PARTITION BY g ORDER BY x DESC)` → `WHERE rk=1` |
| Có X không Y | `… WHERE … AND id NOT IN (SELECT … )` |
| If/else | `CASE WHEN … THEN … ELSE … END` |
| Bảng tạm | `WITH t AS ( … ) SELECT … FROM t` |
| psql xem dọc | `\x` · trợ giúp `\h SELECT` · thoát `\q` |

## Tự kiểm tra
1. Thứ tự CREATE/INSERT các bảng theo gì? *(theo phụ thuộc FK — bảng được tham chiếu trước)*
2. "Hãng bảo dưỡng nhiều nhất" có mấy cách? *(subquery MAX của count; hoặc CTE + RANK)*
3. "Top-1 mỗi nhóm" dùng gì? *(RANK() OVER (PARTITION BY … ORDER BY … DESC), WHERE rk=1)*
4. "Chỉ một hãng và là Toyota" viết thế nào? *(HAVING COUNT(DISTINCT brand)=1 AND MAX(brand)='Toyota')*
5. Trước khi UPDATE/DELETE nên làm gì? *(SELECT thử mệnh đề WHERE; comment thử; cân nhắc transaction)*

## Liên quan
- [[00-MOC-Database|⬅ MOC Database]]
- Trước: [[16-Lam-viec-voi-PostgreSQL]] · Quay lại: [[00-MOC-Database|MOC Database]]
- Lý thuyết: [[07-SQL-co-ban]] · [[08-Aggregate-va-GROUP-BY]] · [[09-JOIN]] · [[10-Thao-tac-du-lieu-INSERT-UPDATE-DELETE]] · [[11-SQL-nang-cao]]
