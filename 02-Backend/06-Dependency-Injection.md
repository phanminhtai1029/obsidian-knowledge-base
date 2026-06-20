---
title: "Dependency Injection"
section: 02-Backend
tags: [backend, fastapi, dependency-injection, depends, yield, annotated, session, fresher]
related:
  - "[[04-SQLAlchemy-Database]]"
  - "[[07-CRUD-Repository-Pattern]]"
  - "[[09-Authorization-RBAC]]"
difficulty: ⭐⭐⭐
estimated_time: 35m
source: [QN26_FR_AI_01, "03.Database Setup"]
---

# Dependency Injection

> [!summary] TL;DR
> **Dependency Injection (DI)** = thay vì endpoint **tự tạo** thứ nó cần (session DB, user hiện tại, config), nó **khai báo** "tôi cần cái này" và để FastAPI **cung cấp**. Trong FastAPI, DI làm qua **`Depends(callable)`**: FastAPI gọi `callable`, lấy kết quả, tiêm vào tham số. Dependency dạng **`yield`** cho phép **setup trước / teardown sau** (mở session → `yield` → đóng/rollback) — lý tưởng cho session DB. Gói lại bằng **`Annotated[AsyncSession, Depends(...)]`** để tái dùng gọn. DI là cách FastAPI hiện thực ranh giới giữa các lớp ([[01-FastAPI-Overview|N-layer]]) và làm code **dễ test** (thay dependency bằng bản giả).

---

## 1. DI là gì? Vì sao cần?

**Không DI** — endpoint tự lo mọi thứ (khó test, lặp code):

```python
@app.get("/users")
async def get_users():
    async with AsyncSessionLocal() as session:   # tự mở session mỗi nơi
        ...
```

**Có DI** — endpoint chỉ *khai báo nhu cầu*:

```python
@app.get("/users")
async def get_users(session: AsyncSession = Depends(get_session)):
    ...        # FastAPI lo tạo & dọn session
```

| Lợi ích | Giải thích |
|---|---|
| **Tách bạch** | Endpoint không quan tâm session được tạo thế nào |
| **Tái sử dụng** | Một `get_session` dùng cho mọi endpoint |
| **Dễ test** | Test thay `get_session` bằng session giả (`dependency_overrides`) |
| **Quản lý vòng đời** | Tự mở/đóng tài nguyên đúng lúc (yield) |

---

## 2. `Depends()` cơ bản

```python
from fastapi import Depends

def common_params(skip: int = 0, limit: int = 10):
    return {"skip": skip, "limit": limit}

@app.get("/items/")
async def list_items(params: dict = Depends(common_params)):
    return params      # FastAPI gọi common_params(), tiêm kết quả vào params
```

FastAPI nhìn `Depends(common_params)` → gọi `common_params` (tự bóc query `skip`, `limit`) → đưa kết quả vào `params`.

---

## 3. Dependency dạng `yield` — setup & teardown

Đây là dạng quan trọng nhất cho **tài nguyên cần dọn dẹp** (session, file, kết nối):

```python
async def make_async_session_db():
    async with AsyncSessionLocal() as session:   # SETUP: mở session
        try:
            yield session                        # trao session cho endpoint
        except Exception:
            await session.rollback()             # lỗi → hoàn tác
            raise
        # (ra khỏi `async with` → session tự đóng): TEARDOWN
```

- Code **trước `yield`** chạy *trước khi* endpoint xử lý (setup).
- Giá trị `yield` được **tiêm vào** endpoint.
- Code **sau `yield`** chạy *sau khi* endpoint trả về (teardown) — kể cả khi có lỗi.

> [!warning] ĐỪNG bọc dependency yield trong `@asynccontextmanager`
> FastAPI **tự hiểu** hàm có `yield` là dependency cần setup/teardown. Nếu bạn bọc thêm `@asynccontextmanager`, dependency sẽ trả về *object context manager* thay vì session → sai. Để hàm `yield` "trần", FastAPI lo phần còn lại. (Khác với `lifespan` của app — cái đó *mới* cần `@asynccontextmanager`.)

```python
@app.get("/users")
async def get_users(session = Depends(make_async_session_db)):
    result = await session.execute(select(DemoTestUser))
    return result.scalars().all()
```

---

## 4. Gọn hơn với `Annotated` (kiểu tái dùng)

Lặp `= Depends(...)` ở mọi endpoint dài dòng. Gói thành một **type alias**:

```python
from typing import Annotated
from sqlalchemy.ext.asyncio import AsyncSession

MyDBSession = Annotated[AsyncSession, Depends(make_async_session_db)]

@app.get("/users")
async def get_users(session: MyDBSession):        # không cần "= Depends(...)"
    return await _query_users(session)

@app.post("/users/init")
async def init_users(session: MyDBSession):       # tái dùng y hệt
    await _insert_demo_users(session); return {"status": "ok"}
```

`Annotated[Kiểu, Depends(...)]` vừa cho **type hint đúng** (IDE/autocomplete), vừa nhúng luôn dependency.

```
★ Insight ─────────────────────────────────────
• Mẫu Annotated[AsyncSession, Depends(get_session)] là cách viết "chuẩn" hiện đại
  của FastAPI: kiểu và dependency đi cùng nhau, khai báo một lần, dùng nhiều nơi.
  Đổi cách tạo session chỉ sửa MỘT chỗ (hàm get_session) — mọi endpoint hưởng theo.
• DI chính là "hiện thực" của N-layer: endpoint (API layer) KHÔNG tự new session
  hay repository; nó nhận chúng qua Depends. Nhờ đó lớp trên phụ thuộc vào "hợp
  đồng" chứ không phụ thuộc cách tạo cụ thể → dễ thay, dễ test.
─────────────────────────────────────────────────
```

---

## 5. Sub-dependency (dependency lồng nhau)

Dependency có thể **phụ thuộc dependency khác** → tạo thành cây. FastAPI giải toàn bộ cây:

```python
async def get_current_user(session: MyDBSession, token: str = Depends(oauth2_scheme)):
    # dùng session + token để tìm user
    ...

@app.get("/me")
async def me(user = Depends(get_current_user)):   # get_current_user lại cần session + token
    return user
```

> Đây là nền cho **xác thực**: `get_current_user` là một dependency, gắn vào endpoint nào cần đăng nhập (xem [[09-Authorization-RBAC]]).

| Use case phổ biến của DI | Dependency |
|---|---|
| Session DB | `get_session` / `make_async_session_db` |
| User hiện tại (auth) | `get_current_user` |
| Cấu hình app | `get_settings` (pydantic-settings) |
| Gom tham số phân trang | `PaginationParams = Depends()` |
| Phân quyền | `require_role("admin")` |

---

## 6. Vài điểm nâng cao

- **Caching trong 1 request:** mặc định, cùng một dependency được gọi nhiều nơi trong **một request** thì chỉ chạy **một lần** (kết quả cache lại). Tắt bằng `Depends(fn, use_cache=False)`.
- **Dependency cấp router/app:** gắn `dependencies=[Depends(verify_token)]` vào `APIRouter`/`include_router` để áp cho mọi route con (vd bắt buộc auth).
- **Override khi test:** `app.dependency_overrides[get_session] = fake_session` để thay bằng bản giả (xem [[10-Testing-Pytest]]).

---

## 7. Q&A phỏng vấn

> [!question] 1. Dependency Injection là gì? FastAPI hiện thực nó bằng gì?
> DI là kỹ thuật **cung cấp** thứ một thành phần cần từ bên ngoài, thay vì để nó tự tạo. FastAPI hiện thực qua **`Depends(callable)`**: khai báo nhu cầu ở tham số, FastAPI gọi callable và tiêm kết quả vào. Lợi ích: tách bạch, tái dùng, dễ test.

> [!question] 2. Dependency dạng `yield` khác dependency thường ra sao?
> Dependency `yield` có **setup (trước yield)** và **teardown (sau yield)** — dùng cho tài nguyên cần dọn dẹp (session, file). Code sau `yield` luôn chạy khi request kết thúc, kể cả khi lỗi → đảm bảo đóng/rollback session.

> [!question] 3. Vì sao KHÔNG bọc dependency session trong `@asynccontextmanager`?
> Vì FastAPI **tự** quản lý vòng đời của hàm `yield`. Bọc thêm `@asynccontextmanager` khiến dependency trả về *object context manager* thay vì giá trị `yield` (session) → endpoint nhận sai. Chỉ `lifespan` của app mới cần `@asynccontextmanager`.

> [!question] 4. `Annotated[AsyncSession, Depends(get_session)]` lợi gì?
> Gói **kiểu + dependency** thành một alias tái dùng: type hint đúng (IDE hiểu), khai báo một lần dùng nhiều endpoint, và đổi cách tạo session chỉ sửa một chỗ.

> [!question] 5. DI giúp việc test dễ hơn thế nào?
> Có thể **override** dependency lúc test: `app.dependency_overrides[get_session] = fake`. Endpoint không đổi nhưng chạy với session/test double giả → test nhanh, không cần DB thật.

> [!question] 6. Sub-dependency là gì? Cho ví dụ.
> Dependency phụ thuộc dependency khác, tạo thành cây mà FastAPI tự giải. Ví dụ `get_current_user` cần `session` + `token` (mỗi cái lại là dependency); endpoint chỉ cần `Depends(get_current_user)`.

---

## 8. Bài tập tự luyện

1. Viết dependency `get_session` dạng `yield` có rollback khi lỗi, gói thành `SessionDep = Annotated[AsyncSession, Depends(get_session)]`, dùng cho 2 endpoint.
2. Viết dependency `pagination(limit=20, offset=0)` trả dict và inject vào `GET /items`.
3. Viết `get_current_user` (sub-dependency cần `session` + token giả) và gắn vào `GET /me`.
4. Trong test, dùng `app.dependency_overrides` để thay `get_session` bằng session SQLite in-memory; chạy một test `GET /items` không cần Postgres.

---

## 9. Liên quan
- [[04-SQLAlchemy-Database]] — session được tiêm qua DI
- [[07-CRUD-Repository-Pattern]] — repository nhận session qua DI
- [[09-Authorization-RBAC]] — `get_current_user` là một dependency
- [[10-Testing-Pytest]] — `dependency_overrides` khi test
- [[00-MOC-Backend|MOC: Backend]]
