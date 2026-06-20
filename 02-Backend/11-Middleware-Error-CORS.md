---
title: "Middleware, Error Handling & CORS"
section: 02-Backend
tags: [backend, fastapi, middleware, cors, error-handling, exception-handler, http-status, fresher]
related:
  - "[[03-Pydantic-Data-Modeling]]"
  - "[[06-Dependency-Injection]]"
  - "[[12-Deployment-Uvicorn]]"
difficulty: ⭐⭐
estimated_time: 35m
source: [QN26_FR_AI_01, "02.Building Async Web Services", "04.CRUD-DI-The Application"]
---

# Middleware, Error Handling & CORS

> [!summary] TL;DR
> **Middleware** là lớp **bọc quanh mọi request/response** — chạy *trước* khi vào endpoint và *sau* khi endpoint trả về (logging, đo thời gian, gắn header, auth thô). **Error handling**: ném **`HTTPException`** để trả lỗi có status code; đăng ký **`@app.exception_handler`** để **đồng bộ định dạng lỗi** toàn app (vd đổi 422 → 400, gói `{detail, code}`). **CORS** giải quyết việc trình duyệt **chặn** request từ origin khác (same-origin policy): thêm **`CORSMiddleware`** khai báo origin/method/header được phép. Phân biệt: **middleware** áp cho *mọi* request một cách chung; **dependency** ([[06-Dependency-Injection]]) áp *có chọn lọc* cho route cụ thể và trả về giá trị.

---

## 1. Middleware — lớp bọc quanh request

```python
import time
from fastapi import FastAPI, Request

app = FastAPI()

@app.middleware("http")
async def add_process_time(request: Request, call_next):
    start = time.time()
    response = await call_next(request)        # gọi tiếp chuỗi → endpoint
    response.headers["X-Process-Time"] = str(time.time() - start)
    return response
```

- Code **trước `call_next`**: chạy *trước* endpoint (vd ghi log, kiểm tra).
- `await call_next(request)`: chuyển tiếp cho lớp/endpoint kế.
- Code **sau `call_next`**: chạy *sau* endpoint (vd gắn header, đo thời gian).

| Use case middleware | Ví dụ |
|---|---|
| Logging / tracing | ghi mỗi request + thời gian |
| Đo hiệu năng | header `X-Process-Time` |
| Gắn header chung | security headers, request-id |
| CORS, GZip | dùng middleware có sẵn |

> [!note] Thứ tự middleware
> Middleware xếp **chồng**: cái thêm sau "bọc ngoài" cái thêm trước. Request đi vào theo thứ tự ngoài→trong, response đi ra theo trong→ngoài (như các lớp vỏ hành).

```
★ Insight ─────────────────────────────────────
• Middleware vs Dependency — câu hay hỏi:
  - Middleware: áp cho MỌI request, không trả giá trị cho endpoint, thấy được cả
    request/response thô. Hợp việc xuyên suốt (log, CORS, timing).
  - Dependency: áp CHỌN LỌC cho route khai báo nó, TRẢ giá trị (session, user),
    tích hợp validate & docs. Hợp việc theo route (auth, DB session).
  Cần "user hiện tại" → dependency. Cần "log mọi request" → middleware.
─────────────────────────────────────────────────
```

---

## 2. Error Handling

### 2.1. `HTTPException` — trả lỗi có status code

```python
from fastapi import HTTPException

@app.get("/todos/{id}")
async def get_todo(id: int):
    todo = await repo.get(id)
    if todo is None:
        raise HTTPException(status_code=404, detail="Todo not found")
    return todo
```

### 2.2. Exception handler — đồng bộ định dạng lỗi

Đăng ký handler để **mọi lỗi cùng một khuôn** (frontend dễ xử lý):

```python
from fastapi import Request
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse

@app.exception_handler(RequestValidationError)
async def validation_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=400,        # đổi 422 mặc định → 400 theo yêu cầu dự án
        content={"detail": "Invalid request parameters",
                 "code": "BAD_REQUEST", "errors": exc.errors()},
    )
```

| Loại lỗi | Bắt bằng | Mặc định |
|---|---|---|
| Lỗi validate input | `RequestValidationError` | 422 |
| Lỗi nghiệp vụ chủ động | `HTTPException` | status bạn đặt |
| Lỗi tùy chỉnh | class exception riêng + handler | bạn định nghĩa |

> Chi tiết luồng validation & ví dụ gói `{success, error}` xem [[03-Pydantic-Data-Modeling#7. Quy trình validation & xử lý lỗi tập trung|Note 3]].

### 2.3. Các status code hay dùng

| Nhóm | Code | Ý nghĩa |
|---|---|---|
| 2xx (thành công) | 200 / 201 / 204 | OK / Created / No Content |
| 4xx (lỗi client) | 400 / 401 / 403 / 404 / 422 | Bad Request / Unauthorized / Forbidden / Not Found / Validation |
| 5xx (lỗi server) | 500 / 503 | Internal Error / Service Unavailable |

---

## 3. CORS — Cross-Origin Resource Sharing

### Vấn đề: Same-Origin Policy

Trình duyệt **chặn** JavaScript gọi API ở **origin khác** (khác scheme/host/port) vì lý do bảo mật. Ví dụ frontend `http://localhost:3000` gọi API `http://localhost:8000` → **bị chặn** nếu server không cho phép.

> **Origin** = scheme + host + port. `http://a.com` ≠ `https://a.com` ≠ `http://a.com:8080`.

### Giải pháp: `CORSMiddleware`

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],   # origin frontend được phép
    allow_credentials=True,                    # cho gửi cookie/Authorization
    allow_methods=["*"],                       # GET, POST, ...
    allow_headers=["*"],
)
```

| Tham số | Ý nghĩa |
|---|---|
| `allow_origins` | danh sách origin được phép gọi |
| `allow_methods` | method HTTP được phép |
| `allow_headers` | header được phép gửi |
| `allow_credentials` | cho gửi cookie/`Authorization` (khi True **không** dùng `allow_origins=["*"]`) |

> [!warning] CORS là cơ chế của **trình duyệt**, không phải bảo mật server
> CORS chỉ kiểm soát **trình duyệt** có cho JS đọc response hay không. Công cụ như `curl`/Postman **không** bị CORS. ⇒ Đừng coi CORS là lớp bảo mật — vẫn cần auth. Và `allow_origins=["*"]` cùng `allow_credentials=True` là **cấu hình sai** (trình duyệt sẽ từ chối).

> [!tip] Preflight `OPTIONS`
> Với request "không đơn giản" (có header tùy chỉnh, method PUT/DELETE...), trình duyệt gửi trước một request **`OPTIONS`** (preflight) để hỏi server có cho phép không. `CORSMiddleware` tự trả lời preflight này.

---

## 4. Q&A phỏng vấn

> [!question] 1. Middleware là gì? Chạy khi nào?
> Là lớp **bọc quanh mọi request/response**: code trước `call_next` chạy *trước* endpoint, code sau chạy *sau*. Dùng cho việc xuyên suốt: logging, timing, gắn header, CORS, GZip.

> [!question] 2. Middleware khác Dependency thế nào?
> **Middleware**: áp cho **mọi** request, không trả giá trị cho endpoint, thấy request/response thô (việc xuyên suốt). **Dependency**: áp **chọn lọc** theo route, **trả giá trị** (session/user), tích hợp validate & docs. "Log mọi request" → middleware; "lấy user hiện tại" → dependency.

> [!question] 3. CORS là gì? Vì sao cần?
> CORS cho phép server **khai báo origin nào** được trình duyệt cho gọi API, vượt qua **same-origin policy** (trình duyệt mặc định chặn request khác origin). Cấu hình bằng `CORSMiddleware`.

> [!question] 4. CORS có phải là bảo mật server không?
> Không. CORS chỉ kiểm soát **trình duyệt**; `curl`/Postman bỏ qua nó. Nó tránh JS web khác lạm dụng API trong ngữ cảnh trình duyệt, nhưng **không thay** xác thực/phân quyền.

> [!question] 5. Làm sao đồng bộ định dạng lỗi toàn app?
> Đăng ký `@app.exception_handler(...)` cho `RequestValidationError`/`HTTPException`/exception tùy chỉnh, trả `JSONResponse` theo khuôn chung (vd `{detail, code, errors}`), kể cả đổi status mặc định (422→400).

> [!question] 6. Preflight request là gì?
> Request `OPTIONS` trình duyệt tự gửi **trước** request "không đơn giản" để hỏi server có cho phép method/header đó không. `CORSMiddleware` tự trả lời.

---

## 5. Bài tập tự luyện

1. Viết middleware đo thời gian xử lý, gắn header `X-Process-Time`, và log method + path + status.
2. Đăng ký exception handler cho `RequestValidationError` đổi 422→400 với khuôn `{detail, code, errors}`.
3. Cấu hình `CORSMiddleware` chỉ cho `http://localhost:3000`, `allow_credentials=True`; thử gọi từ origin khác và quan sát bị chặn.
4. Tạo exception class `NotFoundError` riêng + handler trả 404 `{code:"NOT_FOUND"}`, dùng trong repository/route.

---

## 6. Liên quan
- [[03-Pydantic-Data-Modeling]] — `RequestValidationError`, validate input
- [[06-Dependency-Injection]] — so sánh với middleware
- [[12-Deployment-Uvicorn]] — cấu hình production (CORS chặt, secret)
- [[00-MOC-Backend|MOC: Backend]]
