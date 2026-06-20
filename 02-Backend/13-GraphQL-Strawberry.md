---
title: "GraphQL với Strawberry"
section: 02-Backend
tags: [backend, fastapi, graphql, strawberry, query, mutation, resolver, n+1, fresher]
related:
  - "[[01-FastAPI-Overview]]"
  - "[[07-CRUD-Repository-Pattern]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [QN26_FR_AI_01, "08.GraphQL"]
---

# GraphQL với Strawberry

> [!summary] TL;DR
> **GraphQL** là một *query language cho API*: client **tự khai báo cần field nào** và nhận đúng từng đó — tránh **over-fetching** (lấy thừa) / **under-fetching** (phải gọi nhiều lần) của REST. Toàn bộ API đi qua **một endpoint** (`/graphql`), được mô tả bởi một **schema** gồm các **type**, **Query** (đọc) và **Mutation** (ghi); mỗi field có một **resolver** (hàm trả dữ liệu). Trong Python dùng **Strawberry** (`@strawberry.type/field/mutation/input`) + `GraphQLRouter` gắn vào FastAPI. Bẫy hay gặp: Strawberry **tự đổi `snake_case` → `camelCase`**; GraphQL **không trả mã HTTP 404/400** mà gom lỗi vào mảng **`errors`**.

---

## 1. REST vs GraphQL

| Tiêu chí | **REST** | **GraphQL** |
|---|---|---|
| Endpoint | Nhiều (`/users`, `/users/{id}/posts`...) | **Một** (`/graphql`) |
| Lấy dữ liệu | Server quyết định trả gì | **Client chọn** field cần |
| Over-fetching | Hay bị (trả thừa field) | Không (chỉ field yêu cầu) |
| Under-fetching | Hay bị (phải gọi nhiều API) | Không (gộp trong 1 query) |
| Schema/kiểu | Tùy (OpenAPI) | **Bắt buộc**, mạnh (typed schema) |
| HTTP method | GET/POST/PUT/DELETE | Hầu hết **POST** `/graphql` |
| Mã lỗi | HTTP status (404, 400...) | Luôn 200 + mảng **`errors`** |

> [!example] Client chọn đúng field cần
> ```graphql
> { user(cc: 1) { name role } }   # chỉ lấy name + role, không lấy field khác
> ```
> Trả về: `{"data": {"user": {"name": "HaiHT", "role": "Trainer"}}}`

---

## 2. Khái niệm cốt lõi

| Khái niệm | Là gì |
|---|---|
| **Schema** | Bản mô tả toàn bộ API: có type gì, query/mutation nào |
| **Type** | Cấu trúc dữ liệu (vd `Book` có `id, title, author`) |
| **Query** | Thao tác **đọc** dữ liệu (như GET) |
| **Mutation** | Thao tác **thay đổi** dữ liệu (thêm/sửa/xóa) |
| **Resolver** | Hàm thực thi cho một field — trả về dữ liệu |
| **Input type** | Type dùng cho **tham số đầu vào** của mutation |

---

## 3. Dựng GraphQL API với Strawberry

```python
from fastapi import FastAPI
import strawberry
from strawberry.fastapi import GraphQLRouter

@strawberry.type                      # định nghĩa TYPE
class User:
    cc: int
    name: str
    role: str

fake_db = {1: User(cc=1, name="HaiHT", role="Trainer")}

@strawberry.type                      # gom các field ĐỌC vào Query
class Query:
    @strawberry.field
    def hello(self) -> str:           # resolver đơn giản
        return "Hello GraphQL"

    @strawberry.field
    def user(self, cc: int) -> User | None:   # resolver có tham số
        return fake_db.get(cc)

schema = strawberry.Schema(query=Query)       # dựng schema
graphql_router = GraphQLRouter(schema)

app = FastAPI()
app.include_router(graphql_router, prefix="/graphql")   # gắn vào /graphql
```

Mở `http://127.0.0.1:8000/graphql` có sẵn **GraphQL Playground** để gõ query. Gọi bằng client:

```python
import requests
requests.post("http://localhost:8000/graphql",
              json={"query": "{ user(cc: 1) { name role } }"}).json()
# {'data': {'user': {'name': 'HaiHT', 'role': 'Trainer'}}}
```

### Mutation + Input type

```python
@strawberry.input                     # type cho ĐẦU VÀO
class BookInput:
    id: int
    title: str
    author: str
    published_year: int

@strawberry.type
class Mutation:
    @strawberry.mutation
    def add_book(self, book: BookInput) -> Book:
        if book.id in store:
            raise ValueError(f"Book id {book.id} đã tồn tại.")   # → vào "errors"
        ...

schema = strawberry.Schema(query=Query, mutation=Mutation)
```

```graphql
mutation {
  addBook(book: { id: 5, title: "New Book", author: "Someone", publishedYear: 2023 }) {
    id title
  }
}
```

---

## 4. Ba bẫy thực chiến (rút từ bài làm thật)

> [!warning] 1. `snake_case` → `camelCase` tự động
> Strawberry đổi tên field Python `published_year` thành GraphQL **`publishedYear`**. Trên Playground phải gõ `publishedYear`, gõ `published_year` sẽ báo "field không tồn tại".

> [!warning] 2. GraphQL không trả mã HTTP lỗi
> Khác REST (404/400), GraphQL hầu như luôn trả **HTTP 200**, lỗi nằm trong mảng **`errors`** của response. Cách xử lý: trong resolver/`data` `raise ValueError("...")` → Strawberry tự đưa message vào `errors`.

> [!warning] 3. Query vs Mutation đặt đúng chỗ
> Đọc dữ liệu → `@strawberry.field` (trong `Query`). Thay đổi dữ liệu (thêm/sửa/xóa) → `@strawberry.mutation` (trong `Mutation`). Để hàm "thêm/xóa" nhầm vào Query là lỗi hay gặp.

**Tổ chức code 3 lớp** (giống N-layer):
```
data.py     # kho dữ liệu + CRUD (validate id, raise lỗi)
schema.py   # type / Query / Mutation (resolver gọi xuống data.py)
main.py     # gắn GraphQLRouter vào FastAPI tại /graphql
```

---

## 5. Vấn đề N+1 trong GraphQL

Vì client tự chọn quan hệ lồng nhau, GraphQL **rất dễ dính N+1 query**:

```graphql
{ allBooks { title author { name } } }   # 1 query lấy books + N query lấy author từng cuốn
```

→ 1 truy vấn books + N truy vấn author = **N+1** lần chạm DB.

> [!tip] DataLoader gom truy vấn
> Giải pháp chuẩn là **DataLoader**: gom các yêu cầu lẻ trong cùng một "tick" thành **một** truy vấn batch (`WHERE id IN (...)`) → từ N+1 còn 2. Strawberry có hỗ trợ `DataLoader`.

```
★ Insight ─────────────────────────────────────
• GraphQL dời "quyền quyết định lấy gì" từ SERVER sang CLIENT. Lợi: frontend linh
  hoạt, một endpoint. Hại: server khó kiểm soát chi phí query — client có thể yêu
  cầu cấu trúc lồng sâu gây N+1 hoặc query nặng. Vì thế production cần DataLoader
  + giới hạn độ sâu/độ phức tạp query.
• REST không "thua" GraphQL — chúng đánh đổi khác nhau. REST: cache HTTP dễ, đơn
  giản, status code rõ. GraphQL: linh hoạt field, hợp UI nhiều dạng dữ liệu. Chọn
  theo bài toán, không theo trend.
─────────────────────────────────────────────────
```

---

## 6. Q&A phỏng vấn

> [!question] 1. GraphQL khác REST ở điểm cốt lõi nào?
> Client **tự chọn field cần** qua **một endpoint** `/graphql`, tránh over/under-fetching. REST nhiều endpoint, server quyết định trả gì. GraphQL có schema typed bắt buộc; lỗi nằm trong `errors` thay vì HTTP status.

> [!question] 2. Query và Mutation khác nhau thế nào?
> **Query** = đọc dữ liệu (`@strawberry.field`). **Mutation** = thay đổi dữ liệu — thêm/sửa/xóa (`@strawberry.mutation`). Đặt thao tác ghi vào Query là sai.

> [!question] 3. Resolver là gì?
> Hàm thực thi cho một field — nhận tham số và **trả dữ liệu** cho field đó. Trong Strawberry là method gắn `@strawberry.field`/`@strawberry.mutation`.

> [!question] 4. Vì sao gõ `published_year` trên Playground báo lỗi?
> Strawberry tự đổi `snake_case` (Python) → **`camelCase`** (GraphQL), nên phải gõ `publishedYear`.

> [!question] 5. GraphQL xử lý lỗi ra sao (so với REST 404/400)?
> Thường trả HTTP **200**, lỗi nằm trong mảng **`errors`**. Trong resolver `raise` exception (vd `ValueError`) → message vào `errors`. Không dựa vào HTTP status như REST.

> [!question] 6. Vấn đề N+1 trong GraphQL là gì? Khắc phục?
> Truy vấn quan hệ lồng nhau gây 1 + N lần chạm DB (vd lấy N book rồi lấy author từng cuốn). Khắc phục bằng **DataLoader** (gom thành truy vấn batch `IN (...)`), và giới hạn độ sâu/độ phức tạp query.

---

## 7. Bài tập tự luyện

1. Dựng `Book` API (Strawberry + FastAPI): `allBooks(offset, limit)`, `bookById(id)`, `booksByAuthor(author)`.
2. Thêm mutation `addBook(book: BookInput!)`, `updateBook`, `deleteBook`; trùng id / không tồn tại → `raise` để hiện trong `errors`.
3. Chạy 6 truy vấn (CRUD + verify) trên Playground; thử thêm trùng id và đọc mảng `errors`.
4. Giải thích một query lồng nhau có thể gây N+1 thế nào và DataLoader giảm xuống bao nhiêu lần truy vấn.

---

## 8. Liên quan
- [[01-FastAPI-Overview]] — REST nền tảng để so sánh
- [[07-CRUD-Repository-Pattern]] — CRUD tương ứng trong REST
- [[00-MOC-Backend|MOC: Backend]]
