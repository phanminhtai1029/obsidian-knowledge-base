---
title: "Testing với Pytest"
section: 02-Backend
tags: [backend, fastapi, testing, pytest, testclient, mock, fixture, async-test, fresher]
related:
  - "[[06-Dependency-Injection]]"
  - "[[07-CRUD-Repository-Pattern]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [QN26_FR_AI_01, "07.UnitTest"]
---

# Testing với Pytest

> [!summary] TL;DR
> Test API FastAPI bằng **`pytest`** + **`TestClient`** (gọi app **trong tiến trình**, không cần chạy server thật, không qua mạng → nhanh, hợp CI/CD). `TestClient` chạy được cả endpoint sync lẫn async. Để test **trực tiếp hàm async** dùng **`httpx.AsyncClient` + `ASGITransport`** với `@pytest.mark.asyncio`. **Mock** (`Mock`/`MagicMock`/**`AsyncMock`**) thay thế phụ thuộc thật (DB, API ngoài) để test cô lập. **Fixture** (`@pytest.fixture`) chuẩn bị dữ liệu/tài nguyên dùng chung, hỗ trợ `yield` (setup/teardown). Với DI, dùng **`app.dependency_overrides`** để thay session/auth thật bằng bản giả khi test.

---

## 1. `TestClient` vs `requests`

| | `requests` | **`pytest` + `TestClient`** |
|---|---|---|
| Mục đích | Gửi HTTP request | Framework test |
| Cần server đang chạy? | **Có** | **Không** |
| Qua mạng? | Có | Không (gọi app trực tiếp) |
| Tốc độ | Chậm | Nhanh |
| Hợp CI/CD | Kém lý tưởng | **Rất tốt** |
| Phù hợp | Integration / API thật | Unit + integration |

---

## 2. Test cơ bản với `TestClient`

```python
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)        # bọc app, không khởi động server

def test_get_existing_user():
    response = client.get("/users/alice")
    assert response.status_code == 200
    data = response.json()
    assert data["username"] == "alice"
    assert data["job"] == "Engineer"

def test_get_missing_user():
    response = client.get("/users/unknown")
    assert response.status_code == 404
    assert response.json() == {"detail": "User not found"}

def test_create_existing_user():
    response = client.post("/users", params={"username": "charlie", "job": "Manager"})
    assert response.status_code == 400
```

- Đặt file `test_*.py`, hàm `test_*`, chạy bằng `pytest`.
- Mỗi test **gọi thẳng app** và **assert** status code + body.

---

## 3. Test hàm async với `AsyncClient`

Khi muốn viết test bằng `async def` (gọi trực tiếp coroutine, dùng async fixture):

```python
import pytest
from httpx import AsyncClient, ASGITransport
from app.main import app

transport = ASGITransport(app=app)        # đẩy request thẳng vào ASGI app

@pytest.mark.asyncio                       # đánh dấu test async (pytest-asyncio)
async def test_get_existing_user():
    async with AsyncClient(transport=transport, base_url="http://test") as client:
        response = await client.get("/users/alice")
        assert response.status_code == 200
        assert response.json()["username"] == "alice"
```

> Cần `pip install httpx` và plugin `pytest-asyncio`. `ASGITransport` cho `httpx` "nói chuyện" thẳng với app ASGI mà không mở cổng mạng.

---

## 4. Mock — thay thế phụ thuộc thật

`Mock` là object **giả vờ** là object khác trong lúc test (từ `unittest.mock`). Dùng khi không muốn chạm tài nguyên thật (DB, API ngoài):

```python
from unittest.mock import Mock

def get_username(db_session):
    return db_session.fetch_user()["name"]

mock_db = Mock()
mock_db.fetch_user.return_value = {"name": "HaiHT"}   # định nghĩa hành vi giả
assert get_username(mock_db) == "HaiHT"               # không cần DB thật
```

| Loại | Dùng cho |
|---|---|
| `Mock` | object đồng bộ thông thường |
| `MagicMock` | hỗ trợ thêm `__len__`, `__iter__`, `__getitem__`... |
| **`AsyncMock`** | mock **hàm async** (gọi bằng `await`) |

```python
from unittest.mock import AsyncMock
async_mock = AsyncMock()
async_mock.fetch.return_value = "data"
await async_mock.fetch()       # dùng với await
```

```
★ Insight ─────────────────────────────────────
• Quy tắc chọn mock: phụ thuộc là async → AsyncMock; cần index/len/iter →
  MagicMock; còn lại → Mock. Mock async function bằng Mock thường sẽ trả về Mock
  chứ không phải coroutine → await lỗi. Đây là bug test hay gặp.
• Mock để test ĐƠN VỊ (unit): cô lập logic khỏi DB/mạng cho nhanh & ổn định.
  Nhưng đừng mock mọi thứ — phần "ráp nối thật" (app + DB thật) vẫn cần
  integration test, nếu không bạn chỉ đang test cái mock của chính mình.
─────────────────────────────────────────────────
```

---

## 5. Fixture — chuẩn bị thứ test cần

`Fixture` là hàm **chuẩn bị dữ liệu/tài nguyên**, được pytest **tiêm vào test qua tên tham số**:

```python
import pytest

@pytest.fixture
def number():
    return 42

def test_addition(number):       # tên tham số khớp tên fixture → được tiêm
    assert number + 1 == 43
```

**Vì sao cần:** tránh lặp setup ở mọi test.

```python
@pytest.fixture
def db_session():
    return connect_db()

def test_user_exist(db_session):   # dùng chung setup
    ...
def test_create_user(db_session):
    ...
```

**Setup/teardown bằng `yield`:**

```python
@pytest.fixture
def temp_file():
    f = open("tmp.txt", "w")    # setup
    yield f                     # trao cho test
    f.close()                   # teardown (sau test)
```

| `scope` của fixture | Tạo lại khi nào |
|---|---|
| `function` (mặc định) | mỗi test |
| `module` | mỗi file test |
| `session` | một lần cho cả lần chạy |

---

## 6. `dependency_overrides` — thay dependency khi test

Sức mạnh của DI ([[06-Dependency-Injection]]): thay session/auth thật bằng bản test mà **không đổi code app**:

```python
async def fake_session():
    # vd session SQLite in-memory, hoặc mock
    yield test_session

app.dependency_overrides[get_session] = fake_session     # ép dùng bản giả

def test_list_users():
    response = client.get("/users")
    assert response.status_code == 200

# Dọn sau khi test: app.dependency_overrides.clear()
```

> Nhờ đó test endpoint **không cần Postgres thật**: override `get_session` bằng DB test/in-memory, hoặc override `get_current_user` để giả lập user đã đăng nhập.

---

## 7. Q&A phỏng vấn

> [!question] 1. Vì sao dùng `TestClient` thay vì `requests`?
> `TestClient` gọi app **trong tiến trình**, **không cần server chạy**, không qua mạng → nhanh, ổn định, hợp CI/CD, và có sẵn assertion qua `response`. `requests` cần server thật, chậm, hợp test integration với API thật.

> [!question] 2. Làm sao test một endpoint async?
> Dùng `httpx.AsyncClient` + `ASGITransport(app=app)` trong test `async def`, đánh dấu `@pytest.mark.asyncio` (cần `pytest-asyncio`). Hoặc `TestClient` (chạy được cả async) nếu không cần await trực tiếp.

> [!question] 3. Mock là gì? Khi nào dùng `AsyncMock`?
> Mock là object **giả** thay cho phụ thuộc thật (DB/API) để test cô lập. Dùng **`AsyncMock`** khi mock **hàm async** (gọi bằng `await`); `Mock` thường sẽ không trả coroutine → await lỗi.

> [!question] 4. Fixture là gì? `yield` trong fixture để làm gì?
> Fixture là hàm chuẩn bị tài nguyên/dữ liệu cho test, tiêm qua tên tham số, tránh lặp setup. `yield` cho phép **setup trước / teardown sau** (mở rồi đóng tài nguyên). `scope` điều khiển tần suất tạo lại.

> [!question] 5. Test endpoint cần DB mà không muốn dùng Postgres thật thì sao?
> Dùng **`app.dependency_overrides`** thay `get_session` bằng session test (SQLite in-memory) hoặc mock; thay `get_current_user` để giả user đăng nhập. Code app không đổi, test chạy độc lập.

> [!question] 6. Unit test và integration test khác nhau ra sao trong bối cảnh này?
> **Unit**: cô lập logic, mock DB/mạng → nhanh. **Integration**: ráp nối thật (app + DB thật/test) để chắc các mảnh hoạt động cùng nhau. Cần cả hai; chỉ mock thì dễ "test chính cái mock".

---

## 8. Bài tập tự luyện

1. Viết `TestClient` test cho CRUD todo: tạo (201), lấy (200), lấy thiếu (404), xóa (204).
2. Viết test async bằng `AsyncClient` + `@pytest.mark.asyncio` cho `GET /todos`.
3. Mock một repository bằng `AsyncMock` (`get.return_value = ...`) và test service không cần DB.
4. Viết fixture `client` (scope module) + override `get_session` bằng SQLite in-memory; chạy full test CRUD không cần Postgres.

---

## 9. Liên quan
- [[06-Dependency-Injection]] — `dependency_overrides` nền tảng test
- [[07-CRUD-Repository-Pattern]] — đối tượng cần test
- [[00-MOC-Backend|MOC: Backend]]
