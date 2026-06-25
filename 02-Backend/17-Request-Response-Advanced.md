---
title: "Request & Response nâng cao — Form, Raw, File/CSV, Content Negotiation, Serializer"
section: 02-Backend
tags: [backend, fastapi, request, response, form, content-negotiation, serializer, security, fresher]
related:
  - "[[00b-REST-HTTP-JSON-Fundamentals]]"
  - "[[03-Pydantic-Data-Modeling]]"
  - "[[11-Middleware-Error-CORS]]"
  - "[[15-Implementing-Streaming-APIs]]"
difficulty: ⭐⭐⭐
estimated_time: 35m
source: ["Build REST APIs with FastAPI — Miki Tebeka (LinkedIn Learning)", "FastAPI Docs"]
---

# Request & Response nâng cao

> [!summary] TL;DR
> Ngoài JSON body (đã học ở [[03-Pydantic-Data-Modeling]]), FastAPI còn nhận/trả nhiều dạng khác. **Nhận:** dữ liệu từ **HTML form** (`Form()`), **file upload** (`UploadFile`), và **raw body** (`await request.body()` cho ảnh/nhị phân — **luôn chặn kích thước** chống DoS). **Trả:** không phải lúc nào cũng JSON — có thể trả **file/ảnh** (`Response(media_type=...)`), **CSV**, hay **stream** (`StreamingResponse`). **Custom serializer** (`@field_serializer`) kiểm soát cách một field được mã hoá ra JSON (vd `timedelta` → số giây). **Content negotiation**: nhìn header **`Accept`** của client để trả JSON *hoặc* CSV từ cùng một endpoint.

---

## 1. Các nguồn dữ liệu đầu vào (input) của FastAPI

| Nguồn | Khai báo | Khi nào dùng |
|---|---|---|
| Path param | `/{id}` + tham số | định danh tài nguyên trên URL |
| Query param | tham số có default | lọc/sắp xếp/phân trang |
| **JSON body** | tham số kiểu `BaseModel` | payload có cấu trúc ([[03-Pydantic-Data-Modeling]]) |
| **Form** | `= Form(...)` | submit từ HTML form (`application/x-www-form-urlencoded`) |
| **File** | `UploadFile` | upload file (`multipart/form-data`) |
| **Raw body** | `Request` + `await .body()` | nhị phân/ảnh, hoặc tự kiểm soát parse |

### 1.1. Nhận dữ liệu Form

```python
from typing import Annotated
from fastapi import FastAPI, Form

app = FastAPI()

@app.post("/survey")
async def survey(
    name: Annotated[str, Form()],
    happy: Annotated[int, Form()],
):
    return {"name": name, "happy": happy}
```

> Form khác JSON body: dữ liệu đến dưới dạng `key=value&...` (content-type `x-www-form-urlencoded`), không phải JSON. Phải khai báo `Form()` rõ ràng để FastAPI biết đọc từ form chứ không phải body JSON.

> [!question] Phỏng vấn: "Khi nào dùng Form, khi nào dùng JSON body, khi nào dùng UploadFile?"
> **JSON body** (Pydantic model) cho API "máy gọi máy" — payload có cấu trúc, kiểu mặc định của REST. **Form** (`Form()`) khi dữ liệu đến từ **HTML form** của trình duyệt (`x-www-form-urlencoded`). **UploadFile** (`multipart/form-data`) khi cần **upload file** — file đọc theo luồng, không nuốt hết vào RAM như raw body. Mấu chốt: ba thứ này khác nhau ở **content-type**, nên phải khai báo đúng để FastAPI parse đúng nguồn.

### 1.2. Nhận raw body + **chặn kích thước** (chống DoS)

```python
from fastapi import Request, HTTPException, status

MAX_SIZE = 5 * 1024 * 1024        # 5 MB

@app.post("/size")
async def image_size(request: Request):
    # 1) Chặn SỚM qua Content-Length (chưa đọc body)
    length = int(request.headers.get("content-length", 0))
    if length == 0 or length > MAX_SIZE:
        raise HTTPException(status.HTTP_413_REQUEST_ENTITY_TOO_LARGE,
                            detail="File quá lớn hoặc thiếu Content-Length")
    data: bytes = await request.body()   # đọc toàn bộ body thô (bytes)
    return {"bytes": len(data)}
```

> [!warning] LUÔN validate kích thước input
> "Đừng tin bất cứ gì client gửi lên." Nếu đọc body mà không giới hạn, kẻ tấn công gửi vài GB → **cạn RAM, sập server (DoS)**. Kiểm `Content-Length` **trước** khi đọc là tuyến phòng thủ rẻ nhất. (Liên hệ OWASP & security mindset ở [[09-Authorization-RBAC#6. OWASP Top 10 & tư duy bảo mật|Note 9]].)

---

## 2. Các dạng response (không chỉ JSON)

Trả `dict`/`BaseModel` → FastAPI tự thành JSON. Nhưng đôi khi cần dạng khác:

| Muốn trả | Dùng | media_type |
|---|---|---|
| JSON (mặc định) | `dict` / `BaseModel` / `JSONResponse` | `application/json` |
| Ảnh/file nhị phân | `Response(content=bytes, media_type=...)` | `image/png`, `application/pdf`… |
| CSV / text thô | `Response(content=str, media_type="text/csv")` | `text/csv` |
| File trên đĩa | `FileResponse(path)` | tự suy ra |
| Stream (không rõ độ dài) | `StreamingResponse(generator)` | tuỳ ([[15-Implementing-Streaming-APIs]]) |

```python
from fastapi import Response

@app.get("/report.csv")
async def report():
    csv = "id,name\n1,An\n2,Bình\n"
    return Response(content=csv, media_type="text/csv")
```

---

## 3. Custom serializer — `@field_serializer`

Mặc định Pydantic tự mã hoá field ra JSON, nhưng đôi khi cần **kiểm soát cách hiển thị**. Ví dụ `timedelta` mặc định ra chuỗi ISO `"PT1M25S"` — khó dùng; ta đổi thành **số giây**:

```python
from datetime import timedelta
from pydantic import BaseModel, field_serializer

class TimeResponse(BaseModel):
    delta: timedelta

    @field_serializer("delta")
    def serialize_delta(self, v: timedelta) -> int:
        return int(v.total_seconds())     # "PT1M25S" → 85
```

| Decorator | Vai trò | Chiều |
|---|---|---|
| `@field_validator` | kiểm tra/chuẩn hoá dữ liệu **vào** | input → object ([[03-Pydantic-Data-Modeling]]) |
| **`@field_serializer`** | định dạng dữ liệu **ra** | object → JSON |

> [!tip] Validator vs Serializer — câu hay hỏi
> **Validator** chạy khi *nhận* dữ liệu (ép kiểu, kiểm tra hợp lệ). **Serializer** chạy khi *trả* dữ liệu (định dạng output). Một lo cổng vào, một lo cổng ra.

---

## 4. Content negotiation — một endpoint, nhiều định dạng

Client báo "tôi muốn nhận định dạng nào" qua header **`Accept`**. Server đọc header đó để trả JSON *hoặc* CSV từ **cùng một endpoint**:

```python
from fastapi import Request, Response
from fastapi.responses import JSONResponse

@app.get("/logs")
async def get_logs(request: Request):
    records = [{"level": "INFO", "msg": "started"}]
    accept = request.headers.get("accept", "application/json")

    if "text/csv" in accept:
        csv = "level,msg\n" + "\n".join(f"{r['level']},{r['msg']}" for r in records)
        return Response(content=csv, media_type="text/csv")
    return JSONResponse(content={"count": len(records), "logs": records})
```

| Header `Accept` của client | Server trả |
|---|---|
| `application/json` (hoặc `*/*`) | JSON |
| `text/csv` | CSV |
| định dạng không hỗ trợ | 406 Not Acceptable |

> [!question] Phỏng vấn: "Cùng một URL trả được cả JSON lẫn CSV không?"
> Có — gọi là **content negotiation**. Server đọc header **`Accept`** của client và chọn định dạng tương ứng (JSON/CSV/XML…). Nếu không hỗ trợ định dạng client yêu cầu thì trả **406 Not Acceptable**. Đây là cách giữ một endpoint sạch mà phục vụ nhiều loại consumer.

```
★ Insight ─────────────────────────────────────
• "Trả JSON" chỉ là MẶC ĐỊNH, không phải luật. Response = bytes + media_type +
  status. JSON, CSV, ảnh, stream đều là cùng một cơ chế với media_type khác nhau.
• Ranh giới bảo mật nằm ở ĐẦU VÀO: form/file/raw body là nơi dữ liệu lạ tràn vào
  → giới hạn kích thước, kiểm content-type, không tin Content-Length một cách mù
  quáng (kẻ gian có thể khai man → vẫn phải giới hạn khi đọc thực tế).
• field_validator ↔ field_serializer là cặp đối xứng vào/ra: hiểu một cái là suy
  ra cái kia. Cùng triết lý "schema kiểm soát biên giới dữ liệu".
─────────────────────────────────────────────────
```

---

## 5. BackgroundTasks — việc chạy *sau khi* đã trả response

Có việc *không cần* client đợi: gửi email xác nhận, ghi log/analytics, dọn file tạm, gọi webhook. FastAPI có **`BackgroundTasks`** — trả response cho client **ngay**, rồi mới chạy việc đó **ở hậu trường** (cùng tiến trình).

```python
from fastapi import BackgroundTasks, FastAPI

app = FastAPI()

def send_welcome_email(email: str):     # việc chạy hậu trường (có thể chậm)
    # ... gọi SMTP/email API ...
    print(f"Đã gửi mail cho {email}")

@app.post("/register")
async def register(email: str, bg: BackgroundTasks):
    # ... lưu user vào DB ...
    bg.add_task(send_welcome_email, email)   # XẾP LỊCH, không chạy ngay
    return {"status": "ok"}                   # client nhận response NGAY,
    #                                          # email gửi SAU khi response đã đi
```

> [!note] BackgroundTasks chạy *khi nào* và *ở đâu*?
> Task được chạy **sau khi response đã gửi xong**, ngay **trong cùng tiến trình** (process) của app. Phù hợp việc **nhẹ, nhanh, không sống còn**. Vì cùng process: app **crash/restart là mất task chưa chạy**, và task nặng vẫn **ngốn tài nguyên** của chính server đó.

> [!warning] Khi nào KHÔNG dùng BackgroundTasks → cần Celery / hàng đợi
> `BackgroundTasks` **không** phải hệ job thực thụ: không retry tự động, không bền (mất khi restart), không chạy trên máy worker riêng, không xem được tiến độ. Việc **nặng / lâu / quan trọng** (xử lý video, train, gửi hàng loạt, phải đảm bảo chạy) → dùng **task queue** như **Celery / RQ / Dramatiq + Redis/RabbitMQ**: có **retry, bền (persisted), worker riêng để scale, theo dõi trạng thái**.

| Tiêu chí | **`BackgroundTasks`** (FastAPI) | **Celery** (task queue) |
|---|---|---|
| Chạy ở đâu | **Cùng process** với app | **Worker riêng** (máy/tiến trình khác) |
| Độ bền | Mất khi app restart/crash | **Bền** (lưu trong broker Redis/RabbitMQ) |
| Retry | ❌ Tự lo | ✅ Có sẵn |
| Theo dõi/scale | Không | ✅ Có (scale worker độc lập) |
| Hợp việc | Nhẹ, nhanh, "fire-and-forget" | Nặng, lâu, quan trọng, định kỳ |

```
★ Insight ─────────────────────────────────────
• BackgroundTasks ≠ async concurrency. Nó KHÔNG làm app "đa luồng hơn"; nó chỉ
  hoãn một hàm tới sau khi response đi. Nếu hàm đó blocking nặng, nó vẫn chiếm
  tài nguyên server (và có thể chặn event loop nếu là sync nặng).
• Quy tắc chọn: cần client KHÔNG phải đợi + việc NHẸ + lỡ mất cũng được →
  BackgroundTasks. Cần ĐẢM BẢO chạy / nặng / scale riêng → Celery.
─────────────────────────────────────────────────
```

---

## 6. Tự kiểm tra

1. Kể các nguồn input của FastAPI; Form khác JSON body ở điểm nào?
2. Vì sao phải chặn kích thước khi đọc raw body? Chặn ở đâu là rẻ nhất?
3. Trả một file ảnh PNG về client bằng FastAPI như thế nào?
4. `@field_validator` khác `@field_serializer` ra sao?
5. Content negotiation là gì? Dựa vào header nào? Trả mã gì khi không hỗ trợ?
6. `BackgroundTasks` chạy khi nào? Khi nào phải chuyển sang Celery?

---

## 7. Bài tập tự luyện

1. Viết `POST /upload` nhận `UploadFile`, từ chối file > 2MB (kiểm `content-length` rồi kiểm lại độ dài thực).
2. Viết model có field `created_at: datetime` và `@field_serializer` trả về Unix timestamp (int).
3. Viết `GET /export` trả JSON nếu `Accept: application/json`, trả CSV nếu `Accept: text/csv`, ngược lại 406.
4. Viết `GET /qr/{text}` sinh ảnh QR (bytes) và trả `Response(media_type="image/png")`.
5. Viết `POST /register` trả `{"status":"ok"}` ngay và dùng `BackgroundTasks` để "gửi mail" (giả lập bằng `time.sleep` + ghi log); quan sát response về trước khi task xong.

---

## 8. Liên quan
- [[00b-REST-HTTP-JSON-Fundamentals]] — HTTP body/headers/Accept, JSON & Base64
- [[03-Pydantic-Data-Modeling]] — JSON body, `@field_validator`
- [[11-Middleware-Error-CORS]] — `Response`, `JSONResponse`, status code
- [[15-Implementing-Streaming-APIs]] — `StreamingResponse` (response không rõ độ dài)
- [[09-Authorization-RBAC]] — OWASP & validate input chống tấn công
- [[00-MOC-Backend|MOC: Backend]]
