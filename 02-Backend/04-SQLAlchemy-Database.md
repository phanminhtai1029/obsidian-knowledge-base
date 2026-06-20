---
title: "Database Setup (SQLAlchemy Async)"
section: 02-Backend
tags: [backend, fastapi, sqlalchemy, async, orm, asyncpg, postgres, database, fresher]
related:
  - "[[03-Pydantic-Data-Modeling]]"
  - "[[05-Alembic-Migrations]]"
  - "[[06-Dependency-Injection]]"
  - "[[07-CRUD-Repository-Pattern]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 45m
source: [QN26_FR_AI_01, "03.Database Setup"]
---

# Database Setup (SQLAlchemy Async)

> [!summary] TL;DR
> Để FastAPI nói chuyện với PostgreSQL **bất đồng bộ**, cần 2 lớp: **driver async** `asyncpg` (gửi/nhận byte với DB) và **ORM** `SQLAlchemy` (ánh xạ class Python ↔ bảng, lo session/transaction). Khởi tạo gồm: chuỗi `DATABASE_URL` dạng `postgresql+asyncpg://...`, **engine** (`create_async_engine` — giữ thông tin & pool kết nối), **session factory** (`async_sessionmaker`), và **model ORM** kế thừa `DeclarativeBase` với `Mapped`/`mapped_column`. Mọi truy vấn chạy trong **`AsyncSession`** (`session.add`, `await session.commit()`, `await session.execute(select(...))`). Phân biệt cốt lõi: **SQLAlchemy model = bảng trong DB**, còn **Pydantic schema = hình dạng JSON qua API** — hai thứ khác nhau, nối bằng `from_attributes`.

---

## 1. Hai cách nói chuyện với Postgres: `asyncpg` vs `SQLAlchemy`

| | **asyncpg** | **SQLAlchemy (ORM)** |
|---|---|---|
| Bản chất | Driver async **thô** cho Postgres | Toolkit ORM đầy đủ |
| Viết gì | **SQL tay** (`SELECT ...`) | Code Python (class, object) |
| ORM model | ❌ | ✅ class ↔ bảng |
| Transaction tự động | ❌ thủ công | ✅ |
| Migration (Alembic) | ❌ | ✅ |
| Tích hợp FastAPI | thủ công | **first-class** |

> SQLAlchemy **ngồi trên** `asyncpg`: bạn vẫn cài `asyncpg` làm driver, nhưng viết code qua SQLAlchemy. Driver thô nhanh hơn ở hot-path, nhưng ORM năng suất & an toàn hơn cho dự án vừa/lớn.

### SQLAlchemy có 2 tầng: Core vs ORM

| Tầng | Mức độ | Cách dùng |
|---|---|---|
| **Core** (SQL Expression) | Thấp, gần SQL | Dựng query bằng `Table`, `Column`, `select()` |
| **ORM** | Cao | Map **class ↔ bảng**, **object ↔ dòng** |

Note này (và hầu hết dự án FastAPI) dùng **ORM**.

---

## 2. Kết nối: URL → Engine → Session factory

```python
from urllib.parse import quote
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker

# 1) Chuỗi kết nối: chú ý "+asyncpg" để dùng driver async
DATABASE_URL = "postgresql+asyncpg://{user}:{password}@{host}:{port}/{database}".format(
    user="haiht", password=quote("abc@123"),   # quote() để escape ký tự đặc biệt trong mật khẩu
    host="localhost", port=5432, database="fsa_pgdb",
)

# 2) Engine: giữ thông tin kết nối + connection pool (dùng chung toàn app)
engine = create_async_engine(DATABASE_URL, echo=True)   # echo=True: log SQL ra console

# 3) Session factory: "khuôn" tạo ra các AsyncSession
AsyncSessionLocal = async_sessionmaker(engine, expire_on_commit=False)
```

| Thành phần | Vai trò |
|---|---|
| `DATABASE_URL` | Dialect+driver (`postgresql+asyncpg`) + thông tin đăng nhập |
| **`engine`** | Quản lý **pool kết nối**, tạo **một lần** dùng chung cả app |
| **`async_sessionmaker`** | Factory sinh ra `AsyncSession` cho từng request |
| `expire_on_commit=False` | Sau `commit` object vẫn dùng được (không bị "hết hạn") |
| `echo=True` | In SQL sinh ra — tiện debug, **tắt ở production** |

> [!note] Engine ≠ Session
> **Engine** sống suốt vòng đời app (1 cái duy nhất), quản lý pool. **Session** là *phiên làm việc ngắn* — mở cho mỗi request, đóng khi xong. Đừng dùng chung một session cho nhiều request.

---

## 3. Định nghĩa model ORM (class ↔ bảng)

```python
from datetime import datetime
from sqlalchemy import String, func
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column

class Base(DeclarativeBase):     # lớp cơ sở chung cho mọi model
    pass

class DemoTestUser(Base):
    __tablename__ = "demo_test_users"        # tên bảng trong DB

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String, nullable=False)
    created_at: Mapped[datetime] = mapped_column(server_default=func.now())
```

| Cú pháp | Ý nghĩa |
|---|---|
| `DeclarativeBase` | Lớp gốc; SQLAlchemy gom metadata mọi bảng kế thừa nó |
| `__tablename__` | Tên bảng vật lý |
| `Mapped[int]` | Type hint kiểu cột (SQLAlchemy 2.0 style) |
| `mapped_column(primary_key=True)` | Khai báo chi tiết cột (PK, nullable, default...) |
| `server_default=func.now()` | DB tự điền thời gian khi insert |

### Tạo bảng từ model

```python
async def create_tables():
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)   # ORM → CREATE TABLE
```

> `create_all` tiện cho demo/dev. **Production dùng [[05-Alembic-Migrations|Alembic]]** để quản lý thay đổi schema có lịch sử/rollback.

---

## 4. Vòng đời app: `lifespan` (startup / shutdown)

Hook để chạy việc khởi tạo/dọn dẹp đúng lúc app bật/tắt:

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI

@asynccontextmanager
async def lifespan(app: FastAPI):
    # --- startup: tạo bảng nếu chưa có ---
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield
    # --- shutdown: dọn dẹp (vd drop bảng khi test) ---
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)

app = FastAPI(title="Demo Async API", lifespan=lifespan)
```

> `lifespan` thay cho `@app.on_event("startup"/"shutdown")` cũ. Phần trước `yield` chạy khi **khởi động**, phần sau chạy khi **tắt**.

---

## 5. Làm việc với dữ liệu qua `AsyncSession`

```python
from sqlalchemy import select

# INSERT
async def insert_demo_users():
    async with AsyncSessionLocal() as session:     # mở session (context manager)
        session.add_all([DemoTestUser(name="HaiHT"), DemoTestUser(name="ConCo")])
        await session.commit()                     # PHẢI await commit

# SELECT
async def query_users():
    async with AsyncSessionLocal() as session:
        result = await session.execute(select(DemoTestUser))   # await execute
        users = result.scalars().all()             # lấy list object (không phải Row)
        return users
```

| Thao tác | Code |
|---|---|
| Thêm 1 / nhiều | `session.add(obj)` / `session.add_all([...])` |
| Lưu xuống DB | `await session.commit()` |
| Hoàn tác | `await session.rollback()` |
| Truy vấn | `await session.execute(select(Model))` |
| Lấy object | `result.scalars().all()` / `.scalar_one()` |

> [!warning] Async = nhớ `await`
> Mọi thao tác chạm DB trong bản async đều phải `await`: `await session.commit()`, `await session.execute(...)`. Quên `await` → trả về coroutine chưa chạy, lỗi khó hiểu. (Xem cơ chế ở [[01-FastAPI-Overview#3. Sync vs Async — dùng khi nào?|sync vs async]].)

---

## 6. Nối SQLAlchemy model ↔ Pydantic schema

API trả JSON, nhưng query trả về **object SQLAlchemy**. Cần Pydantic schema bật `from_attributes` để đọc thuộc tính object:

```python
from pydantic import BaseModel, ConfigDict
from typing import List

class UserOut(BaseModel):
    id: int
    name: str
    created_at: datetime
    model_config = ConfigDict(from_attributes=True)   # cho phép đọc từ object (Pydantic v2)
    # Pydantic v1 cũ: class Config: orm_mode = True

@app.get("/users", response_model=List[UserOut])
async def get_users():
    async with AsyncSessionLocal() as session:
        result = await session.execute(select(DemoTestUser))
        return result.scalars().all()    # object ORM → FastAPI serialize qua UserOut
```

### SQLAlchemy model ≠ Pydantic schema (rất hay bị hỏi)

| | **SQLAlchemy model** | **Pydantic schema** |
|---|---|---|
| Đại diện cho | **Bảng trong DB** | **Hình dạng dữ liệu qua API** (JSON) |
| Kế thừa | `DeclarativeBase` | `BaseModel` |
| Mục đích | Lưu trữ, truy vấn | Validate input / format output |
| Ví dụ | `DemoTestUser(Base)` | `UserOut(BaseModel)` |
| Nối với nhau bằng | — | `from_attributes=True` |

```
★ Insight ─────────────────────────────────────
• Đừng trả thẳng object SQLAlchemy ra API "cho nhanh". Vì (1) object ORM có thể
  chứa field nhạy cảm (hashed_password) và quan hệ lazy-load gây lỗi/N+1; (2) lớp
  lưu trữ và lớp API nên TÁCH để đổi cái này không vỡ cái kia. response_model +
  Pydantic schema chính là ranh giới an toàn đó.
• "+asyncpg" trong URL là mấu chốt: thiếu nó, SQLAlchemy dùng driver SYNC
  (psycopg2) → gọi trong async endpoint sẽ BLOCK event loop. Async ORM đòi async
  driver tương ứng.
─────────────────────────────────────────────────
```

---

## 7. Session như một Dependency (bản xem trước)

Thực tế không mở session thủ công trong mỗi endpoint, mà **tiêm (inject)** qua `Depends` — một factory lo mở/đóng/rollback tập trung:

```python
async def make_async_session_db():       # KHÔNG bọc @asynccontextmanager
    async with AsyncSessionLocal() as session:
        try:
            yield session
        except Exception as e:
            await session.rollback()
            raise
```

> Chi tiết cơ chế `Depends`, `Annotated[AsyncSession, Depends(...)]` để ở [[06-Dependency-Injection]].

---

## 8. Q&A phỏng vấn

> [!question] 1. asyncpg và SQLAlchemy khác nhau thế nào? Có cần cả hai không?
> `asyncpg` là **driver async thô** (viết SQL tay, không ORM). `SQLAlchemy` là **ORM** map class↔bảng, lo session/transaction/migration. Có — SQLAlchemy **chạy trên** asyncpg: asyncpg làm tầng truyền dữ liệu, SQLAlchemy làm tầng mô hình hóa.

> [!question] 2. Engine và Session khác nhau ra sao?
> **Engine** tạo **một lần**, sống suốt app, quản lý **pool kết nối**. **Session** là **phiên làm việc ngắn**, mở cho từng request, đóng khi xong. Không dùng chung một session cho nhiều request.

> [!question] 3. SQLAlchemy model và Pydantic schema khác nhau gì?
> SQLAlchemy model = **bảng trong DB** (kế thừa `DeclarativeBase`). Pydantic schema = **hình dạng JSON qua API** (kế thừa `BaseModel`). Tách để bảo mật & độc lập tầng; nối bằng `from_attributes=True` (đọc thuộc tính từ object ORM).

> [!question] 4. Vì sao chuỗi kết nối phải có `+asyncpg`? Quên thì sao?
> Nó chỉ định **driver async**. Thiếu → SQLAlchemy dùng driver **sync** (psycopg2), khi gọi trong endpoint `async def` sẽ **block event loop**, mất hết lợi ích async. Async ORM đòi async driver.

> [!question] 5. `expire_on_commit=False` để làm gì?
> Mặc định sau `commit`, các object bị "expire" (lần truy cập sau sẽ reload từ DB → cần await, dễ lỗi ngoài session). Đặt `False` để object vẫn đọc được thuộc tính sau commit.

> [!question] 6. `lifespan` dùng để làm gì?
> Chạy code **khi app khởi động** (trước `yield`: tạo bảng, mở kết nối, nạp model) và **khi app tắt** (sau `yield`: dọn dẹp). Thay cho `on_event("startup"/"shutdown")` cũ.

> [!question] 7. Vì sao không nên dùng `create_all` ở production?
> `create_all` chỉ tạo bảng còn thiếu, **không xử lý thay đổi schema** (đổi/ thêm cột trên bảng đã có), không có lịch sử/rollback. Production cần **Alembic migration** để versioning schema an toàn.

---

## 9. Bài tập tự luyện

1. Dựng Postgres bằng `docker compose` (image `postgres`), viết `DATABASE_URL` đúng và test kết nối bằng `SELECT 1`.
2. Định nghĩa model `Product(id, name, price, created_at)` với `DeclarativeBase`/`Mapped`, tạo bảng qua `lifespan` startup.
3. Viết 2 endpoint: `POST /products` (insert) và `GET /products` (`response_model=List[ProductOut]` với `from_attributes=True`). Quan sát SQL khi bật `echo=True`.
4. Giải thích điều gì xảy ra nếu bỏ `+asyncpg` khỏi URL và gọi endpoint — thử và đọc lỗi.

---

## 10. Liên quan
- [[03-Pydantic-Data-Modeling]] — schema vs model
- [[05-Alembic-Migrations]] — quản lý thay đổi schema ở production
- [[06-Dependency-Injection]] — tiêm session qua `Depends`
- [[07-CRUD-Repository-Pattern]] — gói thao tác DB thành repository
- [[00-MOC-Backend|MOC: Backend]]
