---
title: "ACID & Transaction"
section: 03-Database
tags: [database, acid, transaction, integrity, fresher]
related:
  - "[[05-Chuan-hoa-va-phi-chuan-hoa]]"
  - "[[12-Index-Transaction-StoredProcedure]]"
  - "[[14-Gioi-thieu-PostgreSQL]]"
difficulty: ⭐⭐⭐
estimated_time: 20m
source: [LinkedIn "Database Foundations" - Scott Simpson]
---

# ACID & Transaction

> [!summary] TL;DR
> **Transaction** = một nhóm thao tác phải **hoàn thành trọn vẹn**; nếu bất kỳ bước nào lỗi → **rollback** toàn bộ, không để DB ở trạng thái dở dang. Transaction tuân theo **ACID**: **Atomicity** (bất khả phân — all-or-nothing), **Consistency** (giữ DB ở trạng thái hợp lệ, không phá ràng buộc), **Isolation** (các transaction không giẫm lên nhau khi chạy đồng thời), **Durability** (đã commit thì dữ liệu được ghi bền, không mất). DBMS lo ACID giúp ta — chỉ cần báo "đây là transaction".

---

## 1. Vì sao cần transaction

Thao tác đơn (thêm/xóa 1 hàng) thì đơn giản. Nhưng thao tác **nhiều bước** — vd **chuyển tiền** giữa 2 tài khoản:
1. Kiểm tra số dư A → 2. Trừ tiền A → 3. Kiểm tra số dư B → 4. Cộng tiền B.

Nếu **mất điện** giữa bước 2 và 4 → A bị trừ nhưng B chưa được cộng → tiền "bốc hơi". Transaction đảm bảo: hoặc **cả 4 bước cùng xảy ra**, hoặc **không bước nào** xảy ra (cái dở dang bị undo).

## 2. ACID ⭐ (4 tính chất)

| Chữ | Tính chất | Nghĩa |
|-----|-----------|-------|
| **A** | **Atomicity** (nguyên tử) | Transaction **bất khả phân** — không tách rời; all-or-nothing |
| **C** | **Consistency** (nhất quán) | Sau transaction, DB ở trạng thái **hợp lệ**, không phá integrity rules |
| **I** | **Isolation** (cô lập) | Khi transaction đang chạy, **không gì khác** được sửa dữ liệu liên quan; request thứ 2 phải chờ |
| **D** | **Durability** (bền vững) | Khi báo "hoàn thành" → dữ liệu **đã thực sự ghi**, không mất dù sự cố sau đó |

> [!tip] Transaction không chỉ cho tài chính
> Đặt bàn nhà hàng (tránh 2 người chiếm 1 bàn), kiểm tra tồn kho (khóa không cho người khác sửa khi đang xem)… Bất cứ khi nào một việc gồm **nhiều bước phải cùng xảy ra**, hoặc cần **độc quyền truy cập** tạm thời → dùng transaction.

## 3. DBMS lo ACID giúp ta

Khả năng tuân thủ ACID **đã có sẵn** trong DBMS. Ta không phải tự viết logic đảm bảo — chỉ cần khai báo "đây là một transaction" (cú pháp tùy DBMS, vd `BEGIN … COMMIT / ROLLBACK`). Phần thực thi index/transaction/stored procedure xem [[12-Index-Transaction-StoredProcedure]].

```
★ Insight ─────────────────────────────────────
• ACID là "hợp đồng" của RDBMS — lý do ngân hàng/đặt vé tin tưởng CSDL quan hệ
  hơn nhiều NoSQL (vốn thường theo mô hình BASE, ưu tiên availability hơn
  consistency — xem [[15-PostgreSQL-vs-cac-DB-khac]]).
• Isolation là chữ "tốn kém" nhất: cô lập càng chặt càng an toàn nhưng càng
  chậm (phải khóa/chờ). PostgreSQL dùng MVCC để cho phép đọc-ghi đồng thời mà
  không khóa lẫn nhau ([[14-Gioi-thieu-PostgreSQL]]).
─────────────────────────────────────────────────
```

## Tự kiểm tra
1. Transaction là gì? *(nhóm thao tác all-or-nothing; lỗi 1 bước → rollback hết)*
2. ACID viết tắt của gì? *(Atomicity, Consistency, Isolation, Durability)*
3. Atomicity nghĩa là gì? *(bất khả phân — cả nhóm cùng chạy hoặc không bước nào)*
4. Durability đảm bảo gì? *(đã commit thì dữ liệu ghi bền, không mất)*
5. Transaction chỉ dùng cho tiền? *(không — đặt bàn, kiểm kho, bất cứ việc nhiều bước cần đồng thời)*

## Liên quan
- [[00-MOC-Database|⬅ MOC Database]]
- Trước: [[05-Chuan-hoa-va-phi-chuan-hoa]] · Kế tiếp: [[07-SQL-co-ban]]
- [[12-Index-Transaction-StoredProcedure|Transaction thực thi]] · [[14-Gioi-thieu-PostgreSQL|MVCC trong Postgres]]
