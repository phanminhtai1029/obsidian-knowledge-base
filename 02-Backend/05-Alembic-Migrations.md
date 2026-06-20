---
title: "Alembic Migrations"
section: 02-Backend
tags: [backend, fastapi, alembic, migration, sqlalchemy, schema, database, fresher]
related:
  - "[[04-SQLAlchemy-Database]]"
  - "[[07-CRUD-Repository-Pattern]]"
  - "[[12-Deployment-Uvicorn]]"
difficulty: ⭐⭐⭐
estimated_time: 35m
source: [QN26_FR_AI_01, "04.CRUD-DI-The Application"]
---

# Alembic Migrations

> [!summary] TL;DR
> SQLAlchemy `create_all()` chỉ **tạo bảng mới** — nó **không** biết `ALTER TABLE` khi bạn thêm/sửa cột trên bảng đã có. **Alembic** (do chính tác giả SQLAlchemy viết) là công cụ **quản lý thay đổi schema theo phiên bản**: mỗi thay đổi là một **revision** chứa `upgrade()` (tiến tới) và `downgrade()` (lùi lại). `--autogenerate` so sánh **model Python** với **DB thật** để sinh script (luôn phải **review tay**), rồi `alembic upgrade head` áp dụng. Điểm bẫy: app chạy driver **async** (`asyncpg`) nhưng **Alembic chạy driver sync** (`psycopg2`) → cần **2 chuỗi kết nối riêng**.

---

## 1. Vấn đề: vì sao `create_all` không đủ

`Base.metadata.create_all()` giống **xây nhà trên đất trống** — chỉ phát `CREATE TABLE` cho bảng *chưa tồn tại*. Nếu bạn sửa model (thêm cột `address`) rồi chạy lại, **bảng cũ không được cập nhật** → model và DB **lệch nhau**.

```python
class User(Base):
    __tablename__ = "users"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    email: Mapped[str]
    address: Mapped[str]    # ← thêm cột mới: create_all KHÔNG thêm vào bảng đã có
```

| | **`create_all()`** | **Alembic** |
|---|---|---|
| Ẩn dụ | Xây nhà từ bản vẽ (đất trống) | Quản lý **giấy phép sửa nhà** theo thời gian |
| Tạo bảng mới | ✅ | ✅ |
| Sửa bảng đã có (`ALTER`) | ❌ | ✅ |
| Lịch sử thay đổi | ❌ | ✅ (revision chain) |
| Rollback | ❌ | ✅ (`downgrade`) |
| Đồng bộ nhiều môi trường | ❌ dễ lệch | ✅ về cùng revision |

---

## 2. Alembic làm gì?

| Chức năng | Mô tả |
|---|---|
| **Schema evolution** | Quản lý thay đổi *tăng dần*: add column, đổi kiểu, tạo/xóa index... |
| **Revision script** | Mỗi migration là 1 file có `upgrade()` (tiến) và `downgrade()` (lùi) |
| **Autogenerate** | So sánh **model metadata** vs **DB hiện tại** → sinh script nháp (`--autogenerate`) |
| **Apply** | `alembic upgrade head` áp mọi migration còn thiếu lên DB |
| **Async support** | Cấu hình được với async engine/driver |

> [!warning] Autogenerate KHÔNG hoàn hảo — luôn review
> Autogenerate không bắt được mọi thay đổi (đổi tên cột, một số ràng buộc, dữ liệu...). Nó tạo **bản nháp** — bạn **phải đọc lại & sửa** file revision trước khi apply, nhất là khi có data migration (vd thêm cột NOT NULL cần default cho dữ liệu cũ).

---

## 3. Thiết lập Alembic

```sh
pip install alembic
alembic init alembic        # tạo thư mục alembic/ + file alembic.ini
```

Sinh ra:
```
alembic.ini              # cấu hình (connection, logging)
alembic/
├── env.py               # script chạy migration (nạp metadata + URL)
├── script.py.mako       # template cho file revision
└── versions/            # chứa các file revision
```

### 3.1. Cấu hình kết nối — Alembic dùng driver SYNC

```ini
# alembic.ini  — CHÚ Ý: dùng driver SYNC, không phải asyncpg
sqlalchemy.url = postgresql+psycopg2://user:pass@localhost:5432/mydb
```

```sh
pip install psycopg2     # driver SYNC cho Alembic
```

> [!note] App async, Alembic sync — 2 URL riêng
> App FastAPI gọi DB qua `postgresql+asyncpg://...` (async). Nhưng Alembic chạy như **script CLI một lần**, không cần async → dùng `postgresql+psycopg2://...` (sync). Thực tế tách trong `.env`: `DATABASE_URL` (app) và `SYNC_DATABASE_URL` (Alembic). *(Alembic cũng hỗ trợ chạy async, nhưng cấu hình sync đơn giản hơn và là cách dạy trong học liệu.)*

### 3.2. Cho Alembic "thấy" model — sửa `env.py`

Autogenerate cần `target_metadata` trỏ tới `Base.metadata` của bạn:

```python
# alembic/env.py
from models import Base                 # import model của bạn
target_metadata = Base.metadata         # để autogenerate so sánh
```

---

## 4. Offline vs Online migration

| Đặc điểm | **Online** | **Offline** |
|---|---|---|
| Kết nối DB | ✅ Có | ❌ Không |
| Thực thi SQL trực tiếp | ✅ | ❌ chỉ xuất text SQL |
| Autogenerate được | ✅ | ❌ |
| Dùng khi | Thông thường (`alembic upgrade head`) | Cần xuất SQL cho DBA review/chạy tay |
| Lệnh ví dụ | `alembic upgrade head` | `alembic upgrade head --sql > script.sql` |

---

## 5. Quy trình & lệnh thường dùng

```sh
# 1) Sinh revision tự động (so model vs DB)
alembic revision --autogenerate -m "add address column"

# 2) (BẮT BUỘC) mở file trong alembic/versions/ đọc & sửa lại

# 3) Áp dụng lên DB
alembic upgrade head

# 4) Kiểm tra
alembic current     # revision hiện tại của DB
alembic history     # toàn bộ chuỗi revision
alembic check       # "No new upgrade operations" = model khớp DB

# Lùi 1 bước / về revision cụ thể
alembic downgrade -1
alembic downgrade <revision_id>
```

> `head` = revision mới nhất trong chuỗi. `upgrade head` = chạy mọi migration còn thiếu để DB lên bản mới nhất.

### Cấu trúc file revision

```python
revision = "305b785a8fad"     # id của bản này
down_revision = "0001"        # bản ngay trước (tạo thành CHUỖI có thứ tự)

def upgrade():
    op.add_column("users", sa.Column("address", sa.String(250), nullable=True))
    op.create_index("ix_users_email", "users", ["email"])

def downgrade():              # đảo ngược chính xác upgrade
    op.drop_index("ix_users_email", "users")
    op.drop_column("users", "address")
```

```
★ Insight ─────────────────────────────────────
• revision + down_revision tạo thành DANH SÁCH LIÊN KẾT (linked list) các phiên
  bản schema. Alembic biết DB đang ở đâu (bảng alembic_version) và "đi bộ" theo
  chuỗi này để upgrade/downgrade tới đích. Đây là lý do migration phải commit
  vào git: cả team đi cùng một chuỗi.
• downgrade() phải là ĐẢO NGƯỢC chính xác của upgrade(). Viết cẩu thả ở đây =
  không rollback được khi production sự cố. Autogenerate sinh sẵn cả hai, nhưng
  vẫn phải kiểm tra cặp này khớp nhau.
─────────────────────────────────────────────────
```

---

## 6. Q&A phỏng vấn

> [!question] 1. Vì sao cần Alembic khi đã có `create_all()`?
> `create_all()` chỉ **tạo bảng mới**, không xử lý `ALTER TABLE` cho bảng đã tồn tại → model và DB lệch nhau khi bạn sửa schema. Alembic quản lý **thay đổi tăng dần có phiên bản**, có `upgrade`/`downgrade`, lịch sử và đồng bộ được mọi môi trường.

> [!question] 2. `upgrade()` và `downgrade()` trong revision để làm gì?
> `upgrade()` áp dụng thay đổi tiến tới (thêm cột, tạo index...). `downgrade()` đảo ngược để rollback về bản trước. Cặp này cho phép tiến/lùi schema an toàn.

> [!question] 3. `--autogenerate` hoạt động ra sao? Có tin tưởng tuyệt đối được không?
> Nó **so sánh `Base.metadata` (model Python) với schema DB thật** rồi sinh script nháp. **Không** tin tuyệt đối: nó bỏ sót đổi tên cột, một số ràng buộc, và không lo data migration → **phải review & sửa tay** trước khi apply.

> [!question] 4. Vì sao app dùng asyncpg nhưng Alembic dùng psycopg2?
> Alembic là **script CLI chạy một lần**, không hưởng lợi từ async → dùng driver **sync** (`psycopg2`) cho đơn giản. App phục vụ nhiều request đồng thời nên cần **async** (`asyncpg`). Vì thế tách 2 chuỗi kết nối (`DATABASE_URL` vs `SYNC_DATABASE_URL`).

> [!question] 5. `alembic upgrade head` nghĩa là gì?
> `head` là revision mới nhất. Lệnh này chạy **mọi migration còn thiếu** theo chuỗi để đưa DB lên phiên bản mới nhất. Alembic theo dõi vị trí hiện tại qua bảng `alembic_version`.

> [!question] 6. Online vs offline migration khác gì?
> **Online**: kết nối DB, chạy SQL trực tiếp, autogenerate được (dùng thường ngày). **Offline**: không kết nối, chỉ **xuất file SQL** để DBA review/chạy tay (`--sql`).

---

## 7. Bài tập tự luyện

1. `alembic init`, cấu hình `sqlalchemy.url` sync, trỏ `target_metadata` về `Base.metadata`, rồi `alembic revision --autogenerate -m "initial"` và `upgrade head`.
2. Thêm cột `priority: int` và `due_date` (nullable) vào model `Todo`, autogenerate revision mới, **đọc kỹ** file sinh ra, rồi apply. Chạy `alembic check` để xác nhận khớp.
3. Tạo 2 index (trên `completed`, `created_at`) bằng `op.create_index`, viết `downgrade` đảo ngược, rồi thử `downgrade -1` và `upgrade head`.
4. Giải thích bằng lời điều gì xảy ra nếu bạn quên review và autogenerate bỏ sót việc đổi tên cột.

---

## 8. Liên quan
- [[04-SQLAlchemy-Database]] — model & engine mà Alembic quản lý schema
- [[07-CRUD-Repository-Pattern]] — paging, filter, `SELECT ... FOR UPDATE` (concurrency)
- [[12-Deployment-Uvicorn]] — chạy migration khi deploy
- [[00-MOC-Backend|MOC: Backend]]
