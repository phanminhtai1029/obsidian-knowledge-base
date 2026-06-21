---
title: "Phân quyền, Compliance & SQL Injection"
section: 03-Database
tags: [database, security, access-control, pii, gdpr, sql-injection, fresher]
related:
  - "[[12-Index-Transaction-StoredProcedure]]"
  - "[[10-Thao-tac-du-lieu-INSERT-UPDATE-DELETE]]"
  - "[[../02-Backend/00-MOC-Backend]]"
difficulty: ⭐⭐⭐
estimated_time: 25m
source: [LinkedIn "Database Foundations" - Scott Simpson]
---

# Phân quyền, Compliance & SQL Injection

> [!summary] TL;DR
> **Access control**: cấp quyền theo cấp độ (đọc/ghi/sửa schema) cho từng user, tới mức bảng hoặc **từng cột** (`GRANT`/`REVOKE`). **Compliance**: dữ liệu cá nhân (**PII**) bị quản chặt — **HIPAA** (y tế, Mỹ), **GDPR** (EU) quy định cách lưu/truy cập/xóa; vi phạm rất tốn kém. **SQL Injection**: kẻ tấn công nhét đoạn SQL vào ô nhập để **chiếm quyền điều khiển câu truy vấn**. Phòng: **prepared statement / tham số hóa**, validate input, giới hạn quyền, stored procedure — và đừng bao giờ **ghép chuỗi** dữ liệu người dùng vào SQL.

---

## 1. Access control (kiểm soát truy cập)

DBMS cho cấp **mức truy cập khác nhau**:
- User A: CRUD dữ liệu nhưng **không** được đổi schema.
- User B: **chỉ đọc** (read-only).
- Có thể giới hạn tới **bảng** hoặc thậm chí **từng cột**.

```sql
GRANT SELECT, INSERT ON Customers TO app_user;
REVOKE DELETE ON Customers FROM app_user;
```
→ Nguyên tắc **least privilege**: cấp đúng tối thiểu cần thiết.

## 2. Compliance & PII

- **PII** (Personally Identifiable Information) = thông tin định danh cá nhân (tên, email, ngày sinh, SĐT…).
- Luật quản chặt cách **xử lý / lưu / truy cập / xóa**:

| Luật | Phạm vi |
|------|---------|
| **HIPAA** | dữ liệu y tế (Mỹ) |
| **GDPR** | dữ liệu cá nhân (EU) |

→ Khi thiết kế DB phải đảm bảo đáp ứng các quy định mình chịu sự điều chỉnh. Không tuân thủ = phạt nặng. (Liên hệ: tách PII ra bảng riêng — quan hệ 1:1, [[03-Quan-he-va-mo-hinh-ER]].)

## 3. SQL Injection ⭐

**Cơ chế**: app ghép thẳng giá trị người dùng vào câu SQL. Kẻ tấn công nhập một **đoạn SQL** thay vì dữ liệu thường → đổi ý nghĩa câu truy vấn.

Ví dụ ô `LastName` nhập:
```
Smith'; DROP TABLE Customers; --
```
Câu SQL ghép chuỗi trở thành:
```sql
INSERT INTO Customers (LastName) VALUES ('Smith'); DROP TABLE Customers; --');
```
→ lệnh `DROP TABLE` chạy, `--` biến phần còn lại thành comment. **Mất sạch bảng.**

### Cách phòng (nhiều lớp)
| Biện pháp | Vì sao hiệu quả |
|-----------|-----------------|
| **Prepared statement / parameterized query** | dữ liệu được truyền tách khỏi câu lệnh → không bị hiểu là SQL (lớp phòng **chính**) |
| **Validate / sanitize input** | loại ký tự/định dạng bất thường |
| **Least privilege** | user app không có quyền DROP/ALTER → giảm thiệt hại |
| **Stored procedure** | truy cập qua procedure tham số hóa thay vì SQL ghép tay |
| Công cụ quét tự động | phát hiện lỗ hổng injection cơ bản |

> [!warning] Bảo mật không "set & forget"
> Hầu như tuần nào cũng có vụ rò rỉ dữ liệu vì SQL injection. Bảo mật phải là một phần của **mọi bước** thiết kế–xây dựng–bảo trì, không phải thêm vào sau cùng.

```
★ Insight ─────────────────────────────────────
• Gốc rễ injection là TRỘN dữ liệu với mã lệnh. Parameterized query tách hẳn
  hai thứ — đây là lý do nó mạnh hơn việc "lọc ký tự xấu" (luôn có cách lách).
• Ở backend FastAPI, dùng ORM/parameter binding của SQLAlchemy là đã tham số
  hóa sẵn — đừng tự ghép f-string vào câu SQL ([[../02-Backend/00-MOC-Backend]]).
─────────────────────────────────────────────────
```

## Tự kiểm tra
1. Access control cấp quyền tới mức nào? *(database/bảng/từng cột; theo loại thao tác)*
2. PII là gì, 2 luật điển hình? *(thông tin định danh cá nhân; HIPAA, GDPR)*
3. SQL injection hoạt động thế nào? *(nhét đoạn SQL vào input để đổi ý nghĩa câu truy vấn)*
4. Biện pháp phòng injection mạnh nhất? *(prepared statement / parameterized query — tách dữ liệu khỏi lệnh)*
5. Nguyên tắc least privilege? *(cấp đúng quyền tối thiểu cần thiết)*

## Liên quan
- [[00-MOC-Database|⬅ MOC Database]]
- Trước: [[12-Index-Transaction-StoredProcedure]] · Kế tiếp: [[14-Gioi-thieu-PostgreSQL]]
- [[10-Thao-tac-du-lieu-INSERT-UPDATE-DELETE|DELETE/UPDATE thận trọng]] · [[../02-Backend/00-MOC-Backend|ORM tham số hóa]]
