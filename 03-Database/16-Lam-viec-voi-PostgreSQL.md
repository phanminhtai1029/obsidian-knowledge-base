---
title: "Làm việc với PostgreSQL — psql, pgAdmin, deploy"
section: 03-Database
tags: [database, postgresql, psql, pgadmin, deploy, extension, fresher]
related:
  - "[[14-Gioi-thieu-PostgreSQL]]"
  - "[[15-PostgreSQL-vs-cac-DB-khac]]"
  - "[[17-Thuc-hanh-SQL-AutoGarage]]"
difficulty: ⭐⭐
estimated_time: 25m
source: [LinkedIn "Learning PostgreSQL" - Sarah Conway]
---

# Làm việc với PostgreSQL — psql, pgAdmin, deploy

> [!summary] TL;DR
> **Truy cập nhanh không cài đặt**: Postgres Playground (Crunchy Data), Postgres WASM (Supabase/Snaplet) chạy ngay trên trình duyệt. **Cài đầy đủ**: tải từ postgresql.org cho mọi OS, hoặc dùng container. **psql** = CLI mặc định: meta-command bắt đầu bằng `\` (`\?` trợ giúp, `\h LỆNH` cú pháp SQL, `\x` chế độ hiển thị dọc, `\q` thoát, `\conninfo` kiểm tra kết nối); chạy file `-f`, lệnh đơn `-c`. **pgAdmin 4** = GUI phổ biến. **Deploy**: on-prem / cloud (AWS/GCP/Azure) / hybrid / multi-cloud; VM hoặc container (Docker/k8s). **Extension** mở rộng tính năng (PostGIS…).

---

## 1. Truy cập PostgreSQL

| Cách | Mô tả |
|------|-------|
| **Postgres Playground** (Crunchy Data) | terminal Postgres ngay trên browser, kèm tutorial |
| **Postgres WASM** (Supabase/Snaplet) | instance chạy trong VM trên browser, hoàn toàn mã nguồn mở |
| **Cài local / on-prem** | tải gói/installer từ postgresql.org (Linux, macOS, Windows, BSD, Solaris) |
| **Container** | Docker/Kubernetes |

## 2. psql — CLI mặc định ⭐

Cài kèm Postgres, không cần setup thêm. **Meta-command** (gõ trong psql, bắt đầu `\`):

| Lệnh | Tác dụng |
|------|----------|
| `\?` | liệt kê meta-command |
| `\h` / `\h SELECT` | trợ giúp **cú pháp SQL** |
| `\conninfo` | kiểm tra đang nối DB nào, user nào |
| `\x` | bật/tắt **expanded display** (hiển thị dọc — dễ đọc bản ghi dài) |
| `\l` / `\dt` | liệt kê database / bảng |
| `\q` (hoặc `Ctrl+D`) | thoát |

**Từ shell hệ điều hành:**
```bash
psql --version                 # kiểm tra cài đặt
createdb linkedin              # tạo DB
psql -d linkedin -f init.sql   # chạy file SQL (-f)
psql -d linkedin -c "SELECT * FROM resources;"   # chạy lệnh đơn (-c)
```

> [!tip] Kết quả khó đọc → `\x`
> Khi `SELECT *` ra bản ghi rộng bị wrap rối, bật `\x` để xem dạng **dọc** (mỗi cột một dòng). CLI khác: `pgcli` (syntax highlight + autocomplete), `psql` vẫn là chuẩn.

## 3. pgAdmin & GUI khác

- **pgAdmin 4** — GUI mã nguồn mở phổ biến nhất; desktop hoặc server mode; xem dashboard (sessions, TPS, tuples, block I/O), duyệt cây Schemas → public → Tables → View/Edit Data, và có **psql tool** tích hợp.
- GUI khác: **DBeaver**, **Beekeeper Studio**, **pgweb** (mã nguồn mở); DataGrip, Postico 2 (thương mại).

## 4. Deploy & vận hành

| Khía cạnh | Lựa chọn |
|-----------|----------|
| **Nơi chạy** | on-premise · cloud (AWS, GCP, Azure) · **hybrid** (vừa on-prem vừa cloud) · **multi-cloud** |
| **Hạ tầng** | server trực tiếp · VM (VirtualBox, VMware, Vagrant) · container (Docker Swarm, Kubernetes, OpenShift, Rancher) |
| **Tự động hóa** | Jenkins, Puppet, Chef, Terraform, Ansible (thay scripting thủ công dễ lỗi) |

## 5. Extension

Postgres mở rộng gần như mọi nhu cầu qua **extension** (PostgreSQL Extension Network): hiệu năng, monitoring/auditing, backup/restore, replication, partitioning, bảo mật (SCRAM auth, mã hóa, row-level security). Forks/bản dựng: TimescaleDB (time-series), FerretDB, EDB Postgres Advanced Server…

```
★ Insight ─────────────────────────────────────
• Phân biệt 2 nhóm lệnh trong psql: SQL (SELECT, CREATE… kết thúc bằng `;`) vs
  meta-command (`\x`, `\dt`… KHÔNG có `;`). Người mới hay nhầm gõ `\dt;` → lỗi.
• Trên cloud thường dùng "managed Postgres" (AWS RDS/Aurora, Azure Database for
  PostgreSQL, Cloud SQL) — nhà cung cấp lo backup/HA/patch; bạn tập trung vào
  schema & query ([[../05-Cloud/00-MOC-Cloud]]).
─────────────────────────────────────────────────
```

## Tự kiểm tra
1. 2 cách dùng Postgres không cần cài? *(Postgres Playground, Postgres WASM trên browser)*
2. Meta-command xem trợ giúp cú pháp SQL? *(`\h` / `\h SELECT`)*
3. `\x` để làm gì? *(bật expanded display — xem bản ghi dạng dọc)*
4. Cờ chạy file SQL và lệnh đơn trong psql? *(`-f` file, `-c` lệnh)*
5. Hybrid cloud nghĩa là gì? *(triển khai vừa on-premise vừa trên cloud)*

## Liên quan
- [[00-MOC-Database|⬅ MOC Database]]
- Trước: [[15-PostgreSQL-vs-cac-DB-khac]] · Kế tiếp: [[17-Thuc-hanh-SQL-AutoGarage]]
- [[14-Gioi-thieu-PostgreSQL|Tính năng Postgres]] · [[../05-Cloud/00-MOC-Cloud|Managed DB trên cloud]]
