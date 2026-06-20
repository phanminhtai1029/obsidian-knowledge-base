---
title: "Path & Query Parameters"
section: 02-Backend
tags: [backend, fastapi, path-parameters, query-parameters, validation, http, fresher]
related:
  - "[[01-FastAPI-Overview]]"
  - "[[03-Pydantic-Data-Modeling]]"
difficulty: ⭐⭐
estimated_time: 30m
source: [QN26_FR_AI_01, "02.Building Async Web Services"]
---

# Path & Query Parameters

> [!summary] TL;DR
> FastAPI lấy dữ liệu đầu vào từ URL theo 2 cách: **Path parameter** (nằm *trong* đường dẫn: `/items/42`) và **Query parameter** (nằm *sau dấu `?`*: `/items?skip=0&limit=10`). FastAPI **tự đọc type hint** để (1) chuyển kiểu chuỗi → kiểu Python, (2) validate, (3) sinh docs. Quy tắc phân biệt: **tên tham số trùng với `{...}` trong path → là path param; còn lại → là query param**. Dùng `Path(...)` và `Query(...)` để thêm ràng buộc (`gt`, `min_length`, `alias`...). Sai ràng buộc → FastAPI trả **422 Unprocessable Entity** kèm chi tiết lỗi, bạn không phải viết tay.

---

## 1. Path Parameter — biến nằm trong đường dẫn

**Path parameter** là biến là *một phần của URL path*. Ví dụ `/items/42` → bắt được `42`.

```python
from fastapi import FastAPI
app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int):     # FastAPI tự ép "42" (str) → 42 (int)
    return {"item_id": item_id}
```

- `{item_id}` trong path khai báo có một path param tên `item_id`.
- `item_id: int` → FastAPI **tự chuyển kiểu** và validate. Gọi `/items/abc` → **422** (không ép được sang int).

### Nhiều path parameter

```python
@app.get("/users/{user_id}/items/{item_id}")
async def read_user_item(user_id: int, item_id: str):
    return {"user_id": user_id, "item_id": item_id}
```

### Bắt cả dấu `/` với converter `:path`

Mặc định path param **không** nuốt dấu `/`. Muốn bắt cả đường dẫn con (file path), dùng `:path`:

```python
@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
# /files/docs/readme.txt  ->  file_path = "docs/readme.txt"
```

### Bảng ánh xạ URL → hàm

| URL Pattern | Signature | Ví dụ URL | Kết quả |
|---|---|---|---|
| `/items/{item_id}` | `read_item(item_id: int)` | `/items/42` | `{"item_id": 42}` |
| `/users/{user_id}/orders/{order_id}` | `get_order(user_id: int, order_id: str)` | `/users/7/orders/abc` | `{"user_id": 7, "order_id": "abc"}` |
| `/files/{file_path:path}` | `read_file(file_path: str)` | `/files/docs/a.txt` | `{"file_path": "docs/a.txt"}` |
| `/blog/{year}/{month}/{slug}` | `read_blog(year: int, month: int, slug: str)` | `/blog/2025/10/x` | `{"year": 2025, "month": 10, "slug": "x"}` |

### Validation cho path param với `Path()`

```python
from fastapi import Path

@app.get("/products/{product_id}")
async def get_product(
    product_id: int = Path(..., gt=0, description="ID phải > 0")
):
    return {"product_id": product_id}
```

- `...` (Ellipsis) = **bắt buộc** (required).
- `gt=0` = greater than 0. Gọi `/products/0` → 422 với message `Input should be greater than 0`.

| Ràng buộc | Ý nghĩa | Áp dụng cho |
|---|---|---|
| `gt` / `ge` | lớn hơn / lớn hơn hoặc bằng | số |
| `lt` / `le` | nhỏ hơn / nhỏ hơn hoặc bằng | số |
| `min_length` / `max_length` | độ dài tối thiểu/tối đa | chuỗi |
| `pattern` (regex) | khớp biểu thức chính quy | chuỗi |

---

## 2. Query Parameter — biến sau dấu `?`

Mọi tham số hàm **không** khớp `{...}` trong path → FastAPI hiểu là **query parameter**.

### 2.1. Giá trị mặc định ⇒ không bắt buộc

```python
fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):   # skip, limit là query param
    return fake_items_db[skip : skip + limit]
```

| URL | Giá trị |
|---|---|
| `/items/` | `skip=0, limit=10` (dùng mặc định) |
| `/items/?skip=20` | `skip=20, limit=10` |

> Đây là pattern **phân trang (paging)** kinh điển: `skip` + `limit`.

### 2.2. Optional — đặt mặc định `None`

```python
@app.get("/items/{item_id}")
async def read_item(item_id: str, q: str | None = None):  # q optional
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}
```

### 2.3. Required — không đặt giá trị mặc định

```python
@app.get("/items/{item_id}")
async def read_item(item_id: str, needy: str):   # needy BẮT BUỘC (không có default)
    return {"item_id": item_id, "needy": needy}
```

| URL | Kết quả |
|---|---|
| `/items/foo` | ❌ 422 `Field required` cho `needy` |
| `/items/foo?needy=abc` | ✅ OK |

### 2.4. Tự chuyển kiểu Boolean

FastAPI nhận nhiều cách viết `True`: `1`, `true`, `True`, `on`, `yes` (mọi biến thể hoa/thường).

```python
@app.get("/items/{item_id}")
async def read_item(item_id: str, short: bool = False):
    item = {"item_id": item_id}
    if not short:
        item["description"] = "mô tả dài..."
    return item
# /items/foo?short=yes  ->  short = True
```

### 2.5. `Query()` — validation & alias

```python
from fastapi import Query

@app.get("/search")
async def search(
    sort_by: str = Query("name", alias="sortBy", max_length=20)
):
    return {"sort_by": sort_by}
# Client gửi ?sortBy=price  ->  trong code biến tên sort_by = "price"
```

> `alias` cho phép **tên public (URL) khác tên biến Python** — hữu ích khi frontend dùng `camelCase` còn Python dùng `snake_case`.

---

## 3. Path vs Query — bảng phân biệt

| Tiêu chí | **Path parameter** | **Query parameter** |
|---|---|---|
| Vị trí trong URL | Trong đường dẫn: `/items/42` | Sau `?`: `/items?skip=0` |
| Khai báo | Phải có `{...}` trong path | Không có trong path |
| Bản chất | **Định danh tài nguyên** (cái nào) | **Lọc/tùy chọn** (lấy thế nào) |
| Bắt buộc? | Luôn bắt buộc (là phần URL) | Tùy: có default → optional |
| Validation | `Path(...)` | `Query(...)` |
| Ví dụ điển hình | `GET /users/7` | `GET /users?role=admin&limit=10` |

```
★ Insight ─────────────────────────────────────
• FastAPI quyết định path vs query KHÔNG dựa vào thứ tự tham số, mà dựa vào việc
  tên tham số có khớp {placeholder} trong chuỗi path hay không. Vì thế thứ tự
  khai báo trong hàm không quan trọng — bạn để query trước path cũng được.
• Quy ước REST: path = "tài nguyên nào" (danh từ định danh), query = "lọc/sắp xếp
  /phân trang". Đừng nhét bộ lọc vào path (vd /items/active/sorted) — đó là mùi
  thiết kế API tồi; hãy dùng /items?status=active&sort=price.
─────────────────────────────────────────────────
```

> [!warning] 422 Unprocessable Entity là gì?
> Khi dữ liệu vào **sai kiểu hoặc vi phạm ràng buộc**, FastAPI tự trả mã **422** kèm JSON chỉ rõ trường nào sai, lý do gì (`loc`, `msg`, `type`). Bạn **không phải tự viết code kiểm tra** — đây là sức mạnh của validation dựa trên type hint + Pydantic. Phân biệt với **404** (không tìm thấy route/tài nguyên) và **400** (request hỏng nói chung).

---

## 4. Q&A phỏng vấn

> [!question] 1. Path parameter và query parameter khác nhau thế nào? Khi nào dùng cái nào?
> **Path** nằm trong đường dẫn (`/users/7`), dùng để **định danh tài nguyên cụ thể**, luôn bắt buộc. **Query** nằm sau `?` (`/users?role=admin`), dùng để **lọc/sắp xếp/phân trang**, có thể optional. Quy tắc REST: path = "cái nào", query = "lấy thế nào".

> [!question] 2. FastAPI làm sao biết tham số nào là path, nào là query?
> Dựa vào việc tên tham số **có khớp `{placeholder}` trong chuỗi path** hay không — khớp thì là path param, còn lại là query param. Không phụ thuộc thứ tự khai báo trong hàm.

> [!question] 3. Làm sao để một query parameter là bắt buộc / optional?
> **Bắt buộc:** không đặt giá trị mặc định (`needy: str`). **Optional:** đặt mặc định (`q: str | None = None` hoặc `skip: int = 0`). Có default ⇒ không bắt buộc.

> [!question] 4. Khi client gửi `/items/abc` mà bạn khai báo `item_id: int` thì sao?
> FastAPI không ép được `"abc"` sang `int` → trả **422 Unprocessable Entity** với chi tiết lỗi. Không cần bạn tự kiểm tra.

> [!question] 5. `alias` trong `Query()` để làm gì?
> Cho phép tên tham số trên URL khác tên biến Python (vd URL `sortBy` ↔ biến `sort_by`). Hữu ích khi frontend dùng camelCase còn backend dùng snake_case.

> [!question] 6. Làm sao bắt một path param chứa dấu `/` (như đường dẫn file)?
> Dùng converter `:path`: `@app.get("/files/{file_path:path}")` → `file_path` sẽ nuốt cả dấu `/`.

---

## 5. Bài tập tự luyện

1. Viết endpoint `GET /users/{user_id}/orders/{order_id}` với `user_id: int` (`gt=0`) và `order_id: str` (`min_length=3`). Thử URL hợp lệ và không hợp lệ, quan sát 422.
2. Viết endpoint phân trang `GET /products` với `skip: int = 0`, `limit: int = 10` (thêm `le=100`), `q: str | None = None`. Trả lát danh sách giả.
3. Thêm tham số `sort_by` với `alias="sortBy"` và chỉ chấp nhận `"name"` hoặc `"price"` (gợi ý: `pattern`/regex). Test bằng `/docs`.
4. Giải thích vì sao `/items/active` là thiết kế kém nếu "active" là bộ lọc, và viết lại đúng chuẩn REST.

---

## 6. Liên quan
- [[01-FastAPI-Overview]] — FastAPI, ASGI, async nền tảng
- [[03-Pydantic-Data-Modeling]] — validate **body** (request payload) bằng Pydantic
- [[00-MOC-Backend|MOC: Backend]]
