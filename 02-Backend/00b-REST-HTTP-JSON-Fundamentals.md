---
title: "REST, HTTP & JSON — Nền tảng giao thức Web"
section: 02-Backend
tags: [backend, rest, http, json, crud, web-fundamentals, fresher]
related:
  - "[[01-FastAPI-Overview]]"
  - "[[02-Path-Query-Parameters]]"
  - "[[03-Pydantic-Data-Modeling]]"
difficulty: ⭐⭐
estimated_time: 30m
source: ["Build REST APIs with FastAPI — Miki Tebeka (LinkedIn Learning)", "MDN Web Docs — HTTP"]
---

# REST, HTTP & JSON — Nền tảng giao thức Web

> [!summary] TL;DR
> Trước khi viết FastAPI, phải nắm **3 lớp nền tảng** mà framework "giấu" đi (nhưng phỏng vấn/Quiz hay đào): **REST** là phong cách thiết kế API xoay quanh **tài nguyên (resource)** + thao tác **CRUD**, ánh xạ sang **HTTP verb** (Create→POST, Read→GET, Update→PUT/PATCH, Delete→DELETE). **HTTP** là giao thức **văn bản (text), request–response**, chạy trên socket: mỗi message gồm **dòng đầu (request/status line) → headers → dòng trống → body**; header **`Host`** là bắt buộc trong request; kết quả mang **status code** (2xx/4xx/5xx). **JSON** là định dạng trao đổi dữ liệu, ánh xạ gần như 1–1 với kiểu Python — nhưng **thiếu** datetime (mã hoá thành chuỗi ISO), binary (mã hoá **Base64**), và không có set/tuple. FastAPI lo hết các lớp này cho bạn, nhưng *"mọi abstraction đều rò rỉ"* — hiểu lớp dưới mới debug được.

---

## 1. REST & CRUD ↔ HTTP verb

**REST** = **Re**presentational **S**tate **T**ransfer: phong cách thiết kế API coi mọi thứ là **tài nguyên (resource)** có **URL** định danh (vd `/users`, `/users/42`), và dùng **HTTP verb** để thao tác. Về bản chất đây là một dạng **RPC** (Remote Procedure Call): client gửi request "làm việc X" → server trả response.

Thao tác trên tài nguyên gom thành 4 nhóm **CRUD**, ánh xạ sang verb HTTP:

| CRUD | HTTP verb | Ví dụ | Ý nghĩa |
|---|---|---|---|
| **C**reate | `POST` | `POST /users` | Tạo user mới |
| **R**ead | `GET` | `GET /users/42` | Lấy 1 user / danh sách |
| **U**pdate | `PUT` / `PATCH` | `PUT /users/42` | Thay toàn bộ / một phần |
| **D**elete | `DELETE` | `DELETE /users/42` | Xoá user |

> [!note] PUT vs PATCH — đừng nhầm
> **`PUT`** = thay **toàn bộ** tài nguyên (gửi đầy đủ các field). **`PATCH`** = cập nhật **một phần** (chỉ gửi field cần đổi). PUT mang tính **idempotent** (gọi nhiều lần kết quả như một lần); POST thì **không** (gọi 2 lần tạo 2 bản ghi).

> [!question] Phỏng vấn: "REST API là gì? CRUD ánh xạ HTTP verb thế nào?"
> REST là phong cách thiết kế API quanh **tài nguyên** định danh bằng URL, thao tác bằng **HTTP verb**, dữ liệu thường mã hoá **JSON**, chạy trên **HTTP**. CRUD ↔ verb: Create→POST, Read→GET, Update→PUT/PATCH, Delete→DELETE. Câu chốt ăn điểm: *GET phải an toàn (không đổi dữ liệu) & idempotent; POST tạo mới (không idempotent); PUT thay toàn bộ (idempotent); PATCH sửa một phần.*

---

## 2. HTTP — giao thức văn bản, request–response

HTTP (**H**yper**T**ext **T**ransfer **P**rotocol) là giao thức **dạng văn bản**, chạy trên **socket/TCP**, theo mô hình **request → response**. Client và server có thể là 2 process khác nhau, thậm chí 2 máy khác nhau.

### 2.1. Cấu trúc một HTTP **request**

```http
POST /users HTTP/1.1          ← request line: VERB + path + version
Host: api.example.com         ← header BẮT BUỘC duy nhất trong request
Content-Type: application/json
Content-Length: 38
                              ← dòng TRỐNG ngăn header với body
{"name": "An", "email": "a@x.io"}   ← body (Content-Length byte)
```

### 2.2. Cấu trúc một HTTP **response**

```http
HTTP/1.1 201 Created          ← status line: version + status code + reason
Content-Type: application/json
Content-Length: 51

{"id": 42, "name": "An", "email": "a@x.io"}
```

| Phần | Request | Response |
|---|---|---|
| Dòng đầu | `VERB path HTTP/1.1` | `HTTP/1.1 code reason` |
| Headers | metadata (Host, Content-Type…) | metadata (Content-Type, Server…) |
| Dòng trống | ngăn header ↔ body | ngăn header ↔ body |
| Body | dữ liệu gửi lên (JSON…) | dữ liệu trả về |

> `Content-Length` = số byte của **body** (không tính dòng đầu & headers). Đây là chi tiết quan trọng để chặn tấn công gửi body khổng lồ → xem [[17-Request-Response-Advanced]].

### 2.3. Status code — nhóm theo chữ số đầu

| Nhóm | Nghĩa | Code hay gặp |
|---|---|---|
| **2xx** | Thành công | 200 OK · 201 Created · 204 No Content |
| **3xx** | Chuyển hướng | 301 Moved · 304 Not Modified |
| **4xx** | **Lỗi phía client** | 400 Bad Request · 401 Unauthorized · 403 Forbidden · 404 Not Found · 422 Unprocessable |
| **5xx** | **Lỗi phía server** | 500 Internal Error · 503 Unavailable |

> [!tip] Mẹo nhớ: "4xx là lỗi của BẠN (client), 5xx là lỗi của TÔI (server)".

### 2.4. Chunked transfer encoding (streaming)

Bình thường server gửi `Content-Length` rồi gửi đủ bấy nhiêu byte. Nhưng khi **không biết trước độ dài** (vd stream token LLM, kết quả query lớn), server dùng header `Transfer-Encoding: chunked`: gửi dữ liệu thành **từng khúc (chunk)**, mỗi khúc kèm số byte (hệ hex), kết thúc bằng khúc rỗng. Đây chính là cơ chế dưới `StreamingResponse` → xem [[14-Streaming-Foundations]].

---

## 3. JSON — định dạng trao đổi dữ liệu

JSON (**J**ava**S**cript **O**bject **N**otation) là định dạng text để mã hoá dữ liệu. Ánh xạ kiểu JSON ↔ Python gần như 1–1:

| JSON | Python | Ghi chú |
|---|---|---|
| object `{}` | `dict` | |
| array `[]` | `list` | |
| string | `str` | |
| number | `int` **hoặc** `float` | JSON chỉ có **một** kiểu số |
| `true`/`false` | `True`/`False` | |
| `null` | `None` | |

### 3.1. Những kiểu JSON **không** có (bẫy hay gặp)

| Kiểu Python | Vấn đề | Cách mã hoá qua JSON |
|---|---|---|
| `datetime` | JSON không có kiểu thời gian | → **chuỗi ISO 8601** (`"2026-06-22T10:30:00Z"`); bên nhận phải tự parse lại |
| `bytes` / ảnh | JSON không có nhị phân | → **Base64** (chuỗi text) |
| `set`, `tuple` | Không tồn tại trong JSON | → `array` (mất tính "set"/cố định) |

> [!question] Phỏng vấn: "Gửi một `datetime` hay một file ảnh qua JSON thế nào?"
> JSON **không có** kiểu datetime và binary. `datetime` thường mã hoá thành **chuỗi ISO 8601** rồi bên nhận parse lại (FastAPI/Pydantic tự lo việc này). Ảnh/nhị phân mã hoá **Base64** thành chuỗi (cồng kềnh hơn ~33%) — hoặc tốt hơn là **không nhét vào JSON** mà gửi trực tiếp dạng raw body / `multipart` → xem [[17-Request-Response-Advanced]].

```
★ Insight ─────────────────────────────────────
• "All abstractions are leaky" (Joel Spolsky): FastAPI giấu HTTP/JSON cho bạn,
  nhưng khi gặp lỗi 422, CORS, chunked, Base64… kiến thức lớp dưới mới cứu bạn.
• GET có body không? Về lý thuyết HTTP cho phép nhưng quy ước REST: GET KHÔNG mang
  body — tham số đi qua path/query. Dữ liệu phức tạp → POST/PUT với JSON body.
• JSON chỉ có MỘT kiểu số → khi cần phân biệt int/float/decimal (vd tiền tệ), phải
  thoả thuận ở tầng schema (Pydantic) chứ JSON không tự giữ được sự khác biệt đó.
─────────────────────────────────────────────────
```

---

## 4. FastAPI lo gì cho bạn ở các lớp này?

| Lớp nền tảng | Bạn tự làm (thuần Python) | FastAPI làm hộ |
|---|---|---|
| HTTP parse | đọc socket, tách line/header/body | Uvicorn + Starlette parse sẵn |
| Routing CRUD↔verb | tự `if method == "POST"` | `@app.post("/users")` |
| JSON ⇄ object | `json.loads` / `json.dumps` thủ công | tự parse body & serialize response |
| datetime/Base64 | tự convert | Pydantic tự mã hoá ISO/validate |
| Status code | tự ghi status line | mặc định 200/201, `HTTPException` cho lỗi |

> Nói cách khác: note này là **"phần chìm của tảng băng"** dưới mọi note FastAPI còn lại.

---

## 5. Tự kiểm tra

1. REST là gì? CRUD ánh xạ sang HTTP verb như thế nào?
2. PUT khác PATCH ra sao? Verb nào idempotent, verb nào không?
3. Một HTTP request gồm những phần nào? Header nào bắt buộc?
4. 401, 403, 404, 422, 500 — mỗi mã nghĩa gì, lỗi của ai (client/server)?
5. Chunked transfer encoding dùng khi nào?
6. Kể 3 kiểu dữ liệu Python không có sẵn trong JSON & cách mã hoá chúng.

---

## 6. Liên quan
- [[01-FastAPI-Overview]] — kiến trúc & async (lớp trên của HTTP/REST)
- [[02-Path-Query-Parameters]] — tham số trên URL (path/query)
- [[03-Pydantic-Data-Modeling]] — JSON body ⇄ object, validate & serialize
- [[17-Request-Response-Advanced]] — Form, raw body, response không-JSON, content negotiation
- [[14-Streaming-Foundations]] — chunked encoding & streaming
- [[00-MOC-Backend|MOC: Backend]]
