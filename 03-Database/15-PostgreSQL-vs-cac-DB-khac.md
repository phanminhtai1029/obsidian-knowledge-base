---
title: "PostgreSQL vs các loại DB khác"
section: 03-Database
tags: [database, postgresql, nosql, scaling, replication, fresher]
related:
  - "[[14-Gioi-thieu-PostgreSQL]]"
  - "[[01-Tong-quan-CSDL-quan-he]]"
  - "[[06-ACID-va-Transaction]]"
difficulty: ⭐⭐⭐
estimated_time: 25m
source: [LinkedIn "Learning PostgreSQL" - Sarah Conway]
---

# PostgreSQL vs các loại DB khác

> [!summary] TL;DR
> **Relational (SQL)** vs **Non-relational (NoSQL)**: quan hệ có **schema cứng**, **ACID**, hợp truy vấn phức tạp, mặc định **scale dọc (vertical)**; NoSQL **schema động**, **scale ngang (horizontal)** sẵn, hợp dữ liệu lớn/đơn giản nhưng yếu với query phức tạp & không chuẩn hóa. Postgres là **relational/ORDBMS theo chuẩn SQL**, vốn scale dọc, nhưng nay **scale ngang được** nhờ **logical replication** (từ PG10) + container (k8s). Chọn Postgres khi cần độ tin cậy, truy vấn phức tạp, đa kiểu dữ liệu.

---

## 1. Relational vs Non-relational ⭐

| Tiêu chí | Relational (SQL) | Non-relational (NoSQL) |
|----------|------------------|------------------------|
| **Schema** | cứng, định nghĩa trước | động, thay đổi theo dữ liệu |
| **Chuẩn hóa** | có chuẩn (SQL) | không chuẩn, biến thể nhiều |
| **Giao dịch** | **ACID** | thường **BASE** (ưu tiên availability) |
| **Truy vấn phức tạp** | mạnh, tin cậy | yếu, có thể trả kết quả thiếu/sai |
| **Scale mặc định** | **dọc (vertical)** | **ngang (horizontal)** |
| **Cộng đồng/công cụ** | nhiều, chuẩn hóa | phân mảnh theo từng DB |
| Ví dụ | PostgreSQL, MySQL | MongoDB, Cassandra, Redis |

> [!tip] ACID vs BASE
> Relational thiên **Consistency** (ACID). Nhiều NoSQL thiên **Availability** (BASE — Basically Available, Soft state, Eventual consistency): dữ liệu nhân bản nhiều node, sẵn sàng cao nhưng **không đảm bảo nhất quán tức thì**.

## 2. Vertical vs Horizontal scaling

| | Vertical (scale-up) | Horizontal (scale-out) |
|--|---------------------|------------------------|
| Cách | thêm CPU/RAM/disk vào **1 server** | thêm **nhiều node** vào cụm |
| Giới hạn | có trần phần cứng | gần như không trần |
| Relational truyền thống | mặc định | khó (cần orchestration) |

**Postgres giải bài toán scale ngang** bằng:
- **Logical replication** (từ **PG10**): mô hình **publish/subscribe** — node publisher tạo "publication" từ bảng, các subscriber nhận cập nhật → phân tán đọc trên nhiều node.
- **Container** (Docker/Kubernetes): chạy cụm Postgres dạng microservices.

## 3. Vì sao chọn PostgreSQL

- Quan hệ + **bám chuẩn SQL** → dễ học, tương thích, cộng đồng lớn.
- Tin cậy, hiệu năng, **ORDBMS** xử lý đa kiểu (JSON/JSONB, XML, geospatial, time-series, graph qua extension).
- Đủ cho cả app nhỏ lẫn enterprise; được Uber, Netflix, Instagram, Spotify, Instacart, Reddit dùng.
- Nhiều năm liền là "DBMS of the Year" (DB-Engines 2017, 2018, 2020).

```
★ Insight ─────────────────────────────────────
• "SQL vs NoSQL" không phải tốt/xấu mà là ĐÁNH ĐỔI: cần giao dịch chặt + query
  phức tạp → relational; cần ghi cực lớn + schema linh hoạt + scale ngang dễ →
  NoSQL. Postgres + JSONB là điểm giữa hấp dẫn.
• Logical replication ≠ physical replication: logical nhân theo từng bảng
  (publish/subscribe, chọn lọc), physical nhân toàn bộ cluster byte-level.
─────────────────────────────────────────────────
```

## Tự kiểm tra
1. Khác biệt schema giữa SQL và NoSQL? *(cứng/định nghĩa trước vs động)*
2. ACID vs BASE? *(nhất quán mạnh vs ưu tiên sẵn sàng, nhất quán cuối)*
3. Vertical vs horizontal scaling? *(mạnh hơn 1 server vs thêm nhiều node)*
4. Postgres scale ngang nhờ gì? *(logical replication PG10 + container/k8s)*
5. Vì sao chọn Postgres? *(quan hệ chuẩn SQL, tin cậy, ORDBMS đa kiểu, cộng đồng lớn)*

## Liên quan
- [[00-MOC-Database|⬅ MOC Database]]
- Trước: [[14-Gioi-thieu-PostgreSQL]] · Kế tiếp: [[16-Lam-viec-voi-PostgreSQL]]
- [[01-Tong-quan-CSDL-quan-he|Các loại DB]] · [[06-ACID-va-Transaction|ACID]] · [[../05-Cloud/00-MOC-Cloud|Managed DB trên cloud]]
