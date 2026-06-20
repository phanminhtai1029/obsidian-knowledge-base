---
title: "CRUD & Repository Pattern"
section: 02-Backend
tags: [backend, fastapi, crud, repository-pattern, paging, concurrency, transaction, fresher]
related:
  - "[[04-SQLAlchemy-Database]]"
  - "[[06-Dependency-Injection]]"
  - "[[03-Pydantic-Data-Modeling]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [QN26_FR_AI_01, "03.Database Setup", "04.CRUD-DI-The Application"]
---

# CRUD & Repository Pattern

> [!summary] TL;DR
> **CRUD** = 4 thao tác cơ bản với dữ liệu: **C**reate / **R**ead / **U**pdate / **D**elete, ánh xạ sang HTTP `POST` / `GET` / `PATCH|PUT` / `DELETE`. **Repository Pattern** gom mọi câu truy vấn DB vào một nơi (`repository.py`) để **API layer không viết SQL trực tiếp** — đúng tinh thần N-layer ([[01-FastAPI-Overview|Tier/Layer]]). Một dự án FastAPI chuẩn tách: `models.py` (bảng) · `schemas.py` (Pydantic) · `repository.py` (truy vấn) · `routes/` (endpoint). Hai chủ đề "thực chiến" hay hỏi: **phân trang** (`limit/offset` + đếm `total` bằng subquery) và **chống lost-update** khi toggle đồng thời bằng `SELECT ... FOR UPDATE`.

---

## 1. CRUD ↔ HTTP method ↔ Status code

| CRUD | HTTP method | Ý nghĩa | Status thành công |
|---|---|---|---|
| **Create** | `POST /todos` | Tạo mới | **201** Created |
| **Read** (list) | `GET /todos` | Liệt kê | **200** OK |
| **Read** (one) | `GET /todos/{id}` | Lấy 1 | 200 / **404** nếu không có |
| **Update** | `PATCH /todos/{id}` | Sửa một phần | 200 / 404 |
| **Delete** | `DELETE /todos/{id}` | Xóa | **204** No Content / 404 |

> [!note] `PUT` vs `PATCH`
> `PUT` = thay **toàn bộ** tài nguyên (gửi đủ mọi field). `PATCH` = sửa **một phần** (chỉ gửi field cần đổi). Với form "sửa todo" thường dùng `PATCH`.

---

## 2. Cấu trúc dự án theo lớp

```
app/
├── main.py                 # khởi tạo FastAPI, lifespan, include_router
├── core/config.py          # cấu hình (.env) qua pydantic-settings
├── db/
│   ├── session.py          # engine + AsyncSession (get_session)
│   ├── models.py           # SQLAlchemy model (BẢNG)
│   ├── schemas.py          # Pydantic: TodoCreate / TodoUpdate / TodoRead
│   └── repository.py       # CRUD: mọi truy vấn DB gom ở đây
└── api/routes/todos.py     # endpoint /todos (APIRouter) — chỉ gọi repository
```

| File | Lớp (layer) | Trách nhiệm |
|---|---|---|
| `routes/todos.py` | API/Presentation | nhận request, validate, gọi repo, trả status |
| `repository.py` | Data Access | đóng gói truy vấn (`select`, `add`, `delete`) |
| `models.py` | Domain/Model | định nghĩa bảng |
| `schemas.py` | Schema | hình dạng input/output JSON |

> Dự án lớn chèn thêm **`services/`** (Business layer) giữa route và repository khi nghiệp vụ phức tạp (nhiều repo, transaction, quy tắc).

---

## 3. Repository Pattern — gom truy vấn một chỗ

**Vấn đề nếu không có repository:** SQL/ORM rải khắp các endpoint → trùng lặp, khó đổi, khó test.

**Repository** = lớp đóng gói thao tác DB, nhận `session` từ ngoài (qua DI):

```python
from sqlalchemy import select, delete
from sqlalchemy.ext.asyncio import AsyncSession

class TodoRepository:
    def __init__(self, session: AsyncSession):
        self.session = session

    async def create(self, data: dict) -> Todo:
        todo = Todo(**data)
        self.session.add(todo)
        await self.session.commit()
        await self.session.refresh(todo)      # nạp lại id/created_at do DB sinh
        return todo

    async def get(self, todo_id: int) -> Todo | None:
        return await self.session.get(Todo, todo_id)   # tìm theo PK

    async def list(self, limit: int, offset: int) -> list[Todo]:
        result = await self.session.execute(
            select(Todo).order_by(Todo.id).limit(limit).offset(offset)
        )
        return result.scalars().all()

    async def update(self, todo: Todo, changes: dict) -> Todo:
        for k, v in changes.items():
            setattr(todo, k, v)
        await self.session.commit()
        return todo

    async def delete(self, todo: Todo) -> None:
        await self.session.delete(todo)
        await self.session.commit()
```

Endpoint mỏng, chỉ điều phối:

```python
@router.post("/todos", status_code=201, response_model=TodoRead)
async def create_todo(payload: TodoCreate, session: SessionDep):
    repo = TodoRepository(session)
    return await repo.create(payload.model_dump())

@router.get("/todos/{todo_id}", response_model=TodoRead)
async def get_todo(todo_id: int, session: SessionDep):
    todo = await TodoRepository(session).get(todo_id)
    if todo is None:
        raise HTTPException(status_code=404, detail="Todo not found")
    return todo
```

```
★ Insight ─────────────────────────────────────
• Repository nhận session qua tham số (DI), KHÔNG tự tạo session bên trong. Nhờ
  đó nhiều repository có thể chia sẻ CÙNG một session → cùng một transaction
  (service gọi 2 repo, commit một lần). Đây là lý do session được tiêm từ tầng
  trên xuống thay vì mỗi repo tự mở.
• 404 là việc của API layer, không phải repository. Repository trả None khi
  không thấy; route mới quyết định dịch None → HTTPException(404). Tách như vậy
  để repository tái dùng được ở chỗ "không thấy" là chuyện bình thường.
─────────────────────────────────────────────────
```

---

## 4. Phân trang, lọc, sắp xếp

### 4.1. Phân trang `limit/offset` + đếm `total`

```python
from sqlalchemy import func, select

async def list_paged(self, limit: int, offset: int, completed: bool | None, search: str | None):
    stmt = select(Todo)
    if completed is not None:
        stmt = stmt.where(Todo.completed == completed)
    if search:
        stmt = stmt.where(Todo.title.ilike(f"%{search}%"))   # ilike = không phân biệt hoa/thường

    # total = đếm SỐ BẢN GHI KHỚP FILTER (không phụ thuộc limit/offset)
    total = await self.session.scalar(select(func.count()).select_from(stmt.subquery()))

    rows = await self.session.execute(stmt.limit(limit).offset(offset))
    return {"total": total, "limit": limit, "offset": offset, "items": rows.scalars().all()}
```

> [!tip] Vì sao đếm `total` bằng subquery?
> Bọc câu query **đã áp filter** thành subquery rồi `COUNT(*)` → `total` phản ánh đúng số bản ghi khớp điều kiện, **độc lập** với `limit/offset`. Nhờ vậy frontend biết tổng số trang.

### 4.2. Sắp xếp an toàn (chống SQL injection)

```python
SORTABLE = {"id": Todo.id, "title": Todo.title, "created_at": Todo.created_at}

def apply_sort(stmt, sort: str):
    desc = sort.startswith("-")           # "-created_at" = giảm dần
    col = SORTABLE[sort.lstrip("-")]      # CHỈ chấp nhận cột trong whitelist
    return stmt.order_by(col.desc() if desc else col.asc())
```

> [!warning] Không ghép thẳng tên cột từ input vào query
> `sort` đến từ người dùng. Nếu nối chuỗi trực tiếp vào SQL → nguy cơ injection. Dùng **whitelist** (`SORTABLE`) và/hoặc khai báo `sort: Literal["id","-id",...]` để chặn giá trị lạ ngay ở tầng validate.

---

## 5. Concurrency: chống "lost update" với `FOR UPDATE`

**Vấn đề:** endpoint toggle đọc `completed` rồi ghi giá trị đảo. Nếu 2 request chạy song song, cả hai đọc cùng giá trị cũ (`false`), cùng tính `true`, cùng ghi `true` → **2 lần bấm chỉ đổi 1 lần** (mất một cập nhật).

**Giải pháp:** khóa dòng khi đọc bằng `with_for_update()`:

```python
async def toggle_locked(self, todo_id: int) -> Todo | None:
    result = await self.session.execute(
        select(Todo).where(Todo.id == todo_id).with_for_update()   # SELECT ... FOR UPDATE
    )
    todo = result.scalar_one_or_none()
    if todo is None:
        return None
    todo.completed = not todo.completed
    await self.session.commit()      # commit mới nhả khóa
    return todo
```

```
Request A: khóa dòng → đọc completed → đảo → commit → NHẢ KHÓA
Request B: muốn đọc cùng dòng → PHẢI CHỜ A commit → đọc giá trị mới → đảo đúng
```

Mỗi request nằm trong **một transaction riêng** (session async); commit xong mới nhả khóa → 2 lần toggle đổi đúng 2 lần.

> Đây là khóa **bi quan (pessimistic lock)** ở mức dòng. Phương án khác: **optimistic lock** (cột `version`, kiểm tra khi update).

---

## 6. Q&A phỏng vấn

> [!question] 1. CRUD là gì và ánh xạ sang HTTP method/status nào?
> Create/Read/Update/Delete → `POST` (201) / `GET` (200) / `PATCH|PUT` (200) / `DELETE` (204). Không tìm thấy tài nguyên → 404.

> [!question] 2. Repository Pattern là gì? Giải quyết vấn đề gì?
> Là lớp **đóng gói mọi truy vấn DB** vào một nơi, để API layer không viết SQL trực tiếp. Giải quyết: tránh trùng lặp truy vấn, dễ đổi cách lưu trữ, dễ test (mock repository), và giữ ranh giới N-layer rõ ràng.

> [!question] 3. Vì sao repository nhận session từ ngoài thay vì tự tạo?
> Để nhiều repository **chia sẻ cùng một session/transaction** (service gọi 2 repo rồi commit một lần). Tự tạo session bên trong sẽ mỗi repo một transaction, không nguyên tử được. Session tiêm qua DI ([[06-Dependency-Injection]]).

> [!question] 4. PUT khác PATCH thế nào?
> `PUT` thay **toàn bộ** tài nguyên (gửi đủ field), `PATCH` sửa **một phần** (chỉ field cần đổi).

> [!question] 5. Phân trang đếm `total` thế nào cho đúng?
> Bọc query **đã áp filter** thành subquery rồi `COUNT(*)` → `total` là số bản ghi khớp filter, **không** bị ảnh hưởng bởi `limit/offset`.

> [!question] 6. "Lost update" là gì? Khắc phục ra sao?
> Hai request đọc cùng giá trị cũ rồi cùng ghi → một cập nhật bị mất. Khắc phục bằng **`SELECT ... FOR UPDATE`** (`with_for_update()`) khóa dòng cho tới khi commit, buộc request sau đọc giá trị mới nhất (pessimistic lock); hoặc optimistic lock bằng cột `version`.

> [!question] 7. Làm sao chống SQL injection ở tham số `sort`?
> Không nối chuỗi input vào query. Dùng **whitelist** cột hợp lệ và khai báo `sort: Literal[...]` để chặn giá trị lạ ngay tầng validate.

---

## 7. Bài tập tự luyện

1. Viết `TodoRepository` đủ 5 method (create/get/list/update/delete), endpoint trả đúng status (201/200/204/404).
2. Thêm `GET /todos?limit&offset&completed&search&sort` trả `{total, limit, offset, items}`; đếm `total` bằng subquery; sort qua whitelist.
3. Hiện thực `toggle_locked` với `with_for_update()`. Mô phỏng 2 request song song (2 task asyncio) và kiểm chứng trạng thái đổi đúng 2 lần.
4. Tách thêm lớp `TodoService` gọi repository, đặt quy tắc "title không rỗng & ≤ 200 ký tự" ở service, giải thích vì sao đặt ở service chứ không ở route.

---

## 8. Liên quan
- [[04-SQLAlchemy-Database]] — session, model, `select`
- [[06-Dependency-Injection]] — tiêm session vào repository
- [[03-Pydantic-Data-Modeling]] — schema Create/Update/Read
- [[00-MOC-Backend|MOC: Backend]]
