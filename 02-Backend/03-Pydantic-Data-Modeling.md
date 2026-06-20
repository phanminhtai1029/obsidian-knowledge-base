---
title: "Data Modeling với Pydantic"
section: 02-Backend
tags: [backend, fastapi, pydantic, basemodel, validation, response-model, schema, fresher]
related:
  - "[[02-Path-Query-Parameters]]"
  - "[[04-SQLAlchemy-Database]]"
  - "[[11-Middleware-Error-CORS]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [QN26_FR_AI_01, "02.Building Async Web Services"]
---

# Data Modeling với Pydantic

> [!summary] TL;DR
> **Pydantic** là thư viện định nghĩa cấu trúc dữ liệu bằng **class kế thừa `BaseModel`** + type hint, rồi **tự validate & ép kiểu** khi nhận dữ liệu. Trong FastAPI, Pydantic lo **request body** (payload JSON gửi lên): FastAPI đọc JSON → dựng thành object Pydantic → nếu sai thì trả **422**. `Field(...)` thêm ràng buộc khai báo (`gt`, `min_length`, `alias`); `@field_validator` viết luật tùy chỉnh. **`response_model`** kiểm soát & lọc dữ liệu *trả ra* (ẩn field nhạy cảm như `password`). Pydantic là **lớp Model/schema** trong kiến trúc N-layer ([[01-FastAPI-Overview]]).

---

## 1. Pydantic & `BaseModel` là gì?

Pydantic cho phép mô tả "dữ liệu phải trông như thế nào" bằng một class:

```python
from pydantic import BaseModel, Field

class Item(BaseModel):
    name: str = Field(..., min_length=3, max_length=50)
    description: str | None = None        # optional
    price: float = Field(..., gt=0)       # bắt buộc, > 0
    in_stock: bool = Field(default=True)  # mặc định True
```

- Kế thừa `BaseModel` → Pydantic tự sinh code validate, ép kiểu, serialize.
- `...` (Ellipsis) = trường **bắt buộc**; có default hoặc `| None = None` = **optional**.

> [!note] Pydantic v2 — phiên bản hiện hành
> FastAPI hiện dùng **Pydantic v2**. Vài đổi tên cần nhớ: `@validator` → **`@field_validator`**; `.dict()` → **`.model_dump()`**; `.json()` → **`.model_dump_json()`**; `parse_obj()` → **`.model_validate()`**; `class Config` → **`model_config = ConfigDict(...)`**; `BaseSettings` tách sang gói riêng **`pydantic-settings`** (xem [[12-Deployment-Uvicorn]]).

---

## 2. Request Body — dữ liệu gửi lên trong thân request

Khác với path/query (nằm trên URL), **body** là khối JSON gửi kèm (thường với `POST`/`PUT`). Khai báo tham số kiểu một `BaseModel` → FastAPI hiểu đó là body:

```python
@app.post("/items/")
async def create_item(item: Item):     # item là request body
    return {"received": item}
```

**Điều xảy ra bên trong:**
1. FastAPI đọc & parse JSON body.
2. Dựng thành object Pydantic `Item` (ép kiểu theo type hint).
3. Chạy ràng buộc `Field` + `@field_validator`.
4. Sai bất kỳ bước nào → tự trả **422** kèm JSON chỉ rõ field lỗi.

| Loại input | Nằm ở đâu | Khai báo bằng |
|---|---|---|
| Path param | trong URL path | kiểu cơ bản + `{...}` trong path |
| Query param | sau `?` trên URL | kiểu cơ bản có/không default |
| **Body** | thân request (JSON) | **`BaseModel`** |

---

## 3. `Field` — ràng buộc & metadata khai báo

`Field` gắn luật validate ngay trên thuộc tính, không cần viết `if`:

```python
from pydantic import BaseModel, Field

class ItemQueryParams(BaseModel):
    limit: int = Field(10, ge=1, le=100)                       # 1 ≤ limit ≤ 100
    search: str | None = Field(None, min_length=3, max_length=50)
    order_by: str = Field("created_at", alias="order-by")      # URL: order-by ↔ code: order_by
```

| Tham số `Field` | Ý nghĩa |
|---|---|
| `gt`/`ge`/`lt`/`le` | so sánh số (>, ≥, <, ≤) |
| `min_length`/`max_length` | độ dài chuỗi/list |
| `pattern` | regex cho chuỗi |
| `alias` | tên ngoài (JSON/URL) khác tên biến Python |
| `default` / `...` | giá trị mặc định / bắt buộc |
| `description` | mô tả (hiện trong OpenAPI `/docs`) |

---

## 4. `@field_validator` — luật validate tùy chỉnh

Khi ràng buộc khai báo không đủ (cần logic riêng, danh sách hợp lệ, chuẩn hóa dữ liệu):

```python
from pydantic import BaseModel, field_validator

class ItemQueryParams(BaseModel):
    category: str
    priority: int

    @field_validator("category")
    @classmethod
    def category_must_be_valid(cls, v: str) -> str:
        valid = {"electronics", "clothing", "food"}
        if v.lower() not in valid:
            raise ValueError(f"Category must be one of {valid}")
        return v.lower()      # trả giá trị đã CHUẨN HÓA (lowercase)
```

> [!tip] Validator vừa kiểm tra vừa **biến đổi**
> Giá trị `return` từ validator sẽ **thay thế** giá trị gốc — nên dùng để chuẩn hóa (trim khoảng trắng, lowercase, ép định dạng). `raise ValueError(...)` → Pydantic gom thành lỗi validation → FastAPI trả 422.

---

## 5. Nhóm query param bằng model + `Depends()`

Khi có nhiều query param liên quan, gom vào một model cho gọn & tái sử dụng:

```python
from fastapi import Depends

class ItemQueryParams(BaseModel):
    skip: int = 0
    limit: int = 10
    search: str | None = None

@app.get("/items/")
def get_items(params: ItemQueryParams = Depends()):  # FastAPI bóc query → validate
    return params
```

**Lợi ích:** code endpoint sạch, validation tái dùng được, tự sinh docs. (`Depends()` xem kỹ ở [[06-Dependency-Injection]].)

```
★ Insight ─────────────────────────────────────
• Cùng là Pydantic model nhưng vai trò khác nhau tùy CÁCH FastAPI nhận diện:
  - Tham số kiểu BaseModel "trần" → FastAPI coi là REQUEST BODY (đọc từ JSON).
  - Tham số kiểu BaseModel + Depends() → FastAPI coi là gom QUERY PARAMS.
  Cùng một class, ngữ cảnh khác → nguồn dữ liệu khác.
─────────────────────────────────────────────────
```

---

## 6. `response_model` — kiểm soát & lọc dữ liệu trả ra

`response_model` khai báo "đầu ra phải đúng khuôn này". FastAPI sẽ **lọc bỏ field thừa** và validate output:

```python
from datetime import datetime

class Item(BaseModel):
    name: str

class ItemResponse(BaseModel):
    name: str
    date_modified: datetime

@app.post("/items/", response_model=ItemResponse)
async def create_item(item: Item):
    return ItemResponse(name=item.name, date_modified=datetime.now())
```

> [!warning] Vì sao tách model Input và Output?
> Model **input** và **output** thường KHÁC nhau. Ví dụ user gửi lên `password` (input) nhưng response **tuyệt đối không** trả `password` ra. `response_model` đảm bảo dữ liệu thừa/nhạy cảm bị **cắt bỏ tự động**, kể cả khi bạn lỡ trả nguyên object DB. Đây là một lớp bảo vệ rò rỉ dữ liệu.

| Loại model | Vai trò | Ví dụ field |
|---|---|---|
| `*Create` / `*In` | dữ liệu nhận vào | `password`, `name` |
| `*Response` / `*Out` | dữ liệu trả ra | `id`, `name` (KHÔNG có `password`) |

---

## 7. Quy trình validation & xử lý lỗi tập trung

Luồng khi một request body sai:
```
1. Request đến
2. FastAPI parse body (JSON)
3. Pydantic chạy validation (Field + @field_validator)
4. validator raise ValueError
5. Pydantic gom thành lỗi có cấu trúc
6. FastAPI raise RequestValidationError
7. (Mặc định) trả 422 kèm chi tiết
```

Muốn **đồng bộ định dạng lỗi toàn app**, bắt `RequestValidationError` ở một chỗ:

```python
from fastapi import Request
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=422,
        content={
            "success": False,
            "error": {"code": "VALUE_ERROR", "message": str(exc)},
        },
    )
```

> Nhờ vậy mọi lỗi validation trả về cùng một "khuôn" (vd `{success, error:{code,message}}`) — frontend dễ xử lý. Chi tiết exception handler & middleware ở [[11-Middleware-Error-CORS]].

---

## 8. Q&A phỏng vấn

> [!question] 1. Pydantic là gì và FastAPI dùng nó để làm gì?
> Pydantic là thư viện **định nghĩa cấu trúc dữ liệu + validate dựa trên type hint**. FastAPI dùng nó để: validate & ép kiểu **request body**, lọc **response** (`response_model`), và **tự sinh tài liệu OpenAPI**. Nó đóng vai trò lớp Model/schema.

> [!question] 2. Khác nhau giữa path/query parameter và request body?
> Path/query nằm **trên URL** (kiểu cơ bản). Body nằm trong **thân request** dưới dạng JSON, khai báo bằng **Pydantic `BaseModel`**, dùng cho `POST`/`PUT` khi cần gửi dữ liệu phức tạp/nhiều trường.

> [!question] 3. `Field()` và `@field_validator` khác nhau khi nào dùng?
> `Field()` cho **ràng buộc khai báo đơn giản** (`gt`, `min_length`, `alias`, default). `@field_validator` cho **logic tùy chỉnh** (danh sách hợp lệ, chuẩn hóa giá trị, kiểm tra liên trường) — và có thể **biến đổi** giá trị qua `return`.

> [!question] 4. `response_model` để làm gì? Vì sao quan trọng cho bảo mật?
> Nó định khuôn & **lọc dữ liệu trả ra**, cắt bỏ field thừa/nhạy cảm (vd `password`, `hashed_password`). Ngay cả khi bạn lỡ trả nguyên object DB, `response_model` vẫn chỉ xuất các field được khai báo → tránh rò rỉ dữ liệu.

> [!question] 5. Vì sao nên tách model Input và Output (Create vs Response)?
> Vì dữ liệu nhận vào và trả ra thường khác nhau: input có `password`, output có `id` + `created_at` nhưng không có `password`. Tách model giúp rõ ràng, an toàn, và tránh "một model gánh mọi việc".

> [!question] 6. Khi validation thất bại, FastAPI trả mã gì? Làm sao đổi định dạng lỗi?
> Mặc định trả **422 Unprocessable Entity** với JSON chi tiết. Đổi định dạng bằng cách đăng ký `@app.exception_handler(RequestValidationError)` trả về `JSONResponse` theo khuôn của bạn.

> [!question] 7. (Pydantic v2) Vài thay đổi tên cần nhớ so với v1?
> `@validator`→`@field_validator`, `.dict()`→`.model_dump()`, `.json()`→`.model_dump_json()`, `parse_obj()`→`.model_validate()`, `class Config`→`model_config = ConfigDict()`, `BaseSettings` chuyển sang gói `pydantic-settings`.

---

## 9. Bài tập tự luyện

1. Tạo model `UserCreate` (có `email`, `password ≥ 8 ký tự`, `age` từ 13–120) và `UserResponse` (chỉ `id`, `email`, `created_at` — **không** password). Viết `POST /users` dùng `response_model=UserResponse`.
2. Viết `@field_validator("email")` kiểm tra email chứa `@` và chuẩn hóa về lowercase.
3. Gom các query lọc sản phẩm (`min_price`, `max_price`, `category`, `limit`) thành một model dùng `Depends()`, và thêm validator đảm bảo `min_price <= max_price` (gợi ý: `@model_validator`).
4. Đăng ký một exception handler cho `RequestValidationError` trả về `{success:false, error:{code,message}}`, rồi gửi request sai để kiểm tra.

---

## 10. Liên quan
- [[02-Path-Query-Parameters]] — input từ URL (path/query)
- [[04-SQLAlchemy-Database]] — phân biệt Pydantic schema vs SQLAlchemy model
- [[06-Dependency-Injection]] — `Depends()` gom query param
- [[11-Middleware-Error-CORS]] — exception handler tập trung
- [[00-MOC-Backend|MOC: Backend]]
