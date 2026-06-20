---
title: "Authorization & Current User (RBAC)"
section: 02-Backend
tags: [backend, fastapi, authorization, rbac, oauth2, get-current-user, security, fresher]
related:
  - "[[08-Authentication-OAuth2-JWT]]"
  - "[[06-Dependency-Injection]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 40m
source: [QN26_FR_AI_01, "06.Authentication (In FastAPI)"]
---

# Authorization & Current User (RBAC)

> [!summary] TL;DR
> Sau khi user đã đăng nhập và cầm JWT ([[08-Authentication-OAuth2-JWT]]), mỗi request bảo vệ sẽ gửi header `Authorization: Bearer <token>`. FastAPI dùng **`OAuth2PasswordBearer`** để **rút token** khỏi header, rồi một dependency **`get_current_user`** **verify token → tìm user → trả về** (sai thì raise **401**). Đây là **authorization (AuthZ)**: quyết định *được làm gì*. Mở rộng thành **RBAC (Role-Based Access Control)** bằng cách đọc claim `roles` trong token và một dependency `require_role("admin")` chặn ai không đủ quyền (raise **403**). Toàn bộ gắn vào endpoint qua **`Depends`** ([[06-Dependency-Injection]]) — bảo vệ route chỉ là thêm một tham số.

---

## 1. AuthN xong rồi — giờ tới AuthZ

| | Authentication (Note 8) | **Authorization (Note này)** |
|---|---|---|
| Câu hỏi | *Bạn là ai?* | *Bạn được làm gì?* |
| Khi nào | Lúc đăng nhập (`/token`) | **Mỗi request** vào route bảo vệ |
| Cơ chế | verify mật khẩu → cấp JWT | rút JWT → verify → kiểm tra quyền |
| Lỗi điển hình | 401 Unauthorized | 403 Forbidden (đã đăng nhập nhưng thiếu quyền) |

> [!note] 401 vs 403 — đừng nhầm
> **401 Unauthorized**: *chưa* xác thực hợp lệ (thiếu token / token sai / hết hạn). **403 Forbidden**: *đã* xác thực nhưng **không đủ quyền** (vd user thường gọi API admin). Recruiter hay hỏi đúng cặp này.

---

## 2. `OAuth2PasswordBearer` — rút token khỏi header

```python
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")   # trỏ tới endpoint /token
```

- `OAuth2PasswordBearer` là một **dependency**: khi dùng `Depends(oauth2_scheme)`, FastAPI **đọc header** `Authorization: Bearer <token>` và trả về phần `<token>` (chuỗi).
- `tokenUrl="token"` cho `/docs` biết lấy token ở đâu → hiện nút **Authorize**.
- Thiếu header / sai định dạng → tự trả **401**.

---

## 3. `get_current_user` — dependency cốt lõi

Đây là "trái tim" của bảo vệ route: verify token → lấy user.

```python
from fastapi import Depends, HTTPException, status
import jwt

async def get_current_user(
    token: str = Depends(oauth2_scheme),     # rút token
    session: SessionDep = ...,               # DI session ([[06-Dependency-Injection]])
):
    credentials_exc = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Không xác thực được",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        user_id = payload.get("sub")
        if user_id is None:
            raise credentials_exc
    except jwt.InvalidTokenError:
        raise credentials_exc

    user = await get_user_by_id(session, user_id)   # tìm trong DB
    if user is None:
        raise credentials_exc
    return user
```

Bảo vệ một endpoint = thêm tham số:

```python
@app.get("/me")
async def read_me(current_user = Depends(get_current_user)):
    return current_user        # tới được đây nghĩa là token hợp lệ
```

```
★ Insight ─────────────────────────────────────
• get_current_user là sub-dependency điển hình: nó tự cần token (Depends
  oauth2_scheme) + session (Depends get_session). Endpoint chỉ khai báo
  Depends(get_current_user) là có sẵn user — FastAPI giải cả cây phụ thuộc.
• Bảo vệ route trong FastAPI KHÔNG dùng if/middleware rải rác, mà bằng DI: "route
  này cần current_user" thể hiện ngay trên chữ ký hàm → đọc code biết ngay route
  nào công khai, route nào bảo vệ. Đây là khác biệt lớn với cách dùng decorator
  thủ công ở framework khác.
─────────────────────────────────────────────────
```

---

## 4. RBAC — phân quyền theo vai trò

Token mang claim `roles` (đặt lúc phát ở Note 8). Tạo dependency kiểm tra vai trò:

```python
def require_role(*allowed: str):
    async def checker(current_user = Depends(get_current_user)):
        if not set(allowed) & set(current_user.roles):
            raise HTTPException(status_code=403, detail="Không đủ quyền")
        return current_user
    return checker        # trả về một dependency "đã cấu hình"

@app.delete("/users/{uid}")
async def delete_user(uid: int, _ = Depends(require_role("admin"))):
    ...        # chỉ admin tới được đây
```

> `require_role` là một **dependency factory**: gọi với tham số (`"admin"`) → trả về dependency cụ thể. Mẫu này cho phép tái dùng cho nhiều mức quyền.

| Mô hình phân quyền | Ý tưởng | Khi nào |
|---|---|---|
| **RBAC** (Role-Based) | Quyền gắn theo **vai trò** (admin/user/editor) | Phổ biến, đơn giản |
| **ABAC** (Attribute-Based) | Quyền theo **thuộc tính** (phòng ban, giờ, sở hữu) | Phức tạp, linh hoạt |
| **Ownership check** | "Chỉ chủ sở hữu được sửa" | Tài nguyên cá nhân |

---

## 5. Bảo vệ cả nhóm route

Gắn dependency ở cấp router để áp cho mọi route con:

```python
from fastapi import APIRouter
admin_router = APIRouter(prefix="/admin", dependencies=[Depends(require_role("admin"))])
# mọi route trong admin_router đều yêu cầu admin
```

> [!tip] `dependencies=[...]` khác `Depends(...)` ở tham số
> Khi **không cần giá trị trả về** (chỉ cần "chạy để chặn"), đặt vào `dependencies=[...]` của route/router. Khi **cần dùng** kết quả (vd `current_user`), khai báo làm tham số `= Depends(...)`.

---

## 6. Q&A phỏng vấn

> [!question] 1. Sau khi có JWT, request tiếp theo xác thực thế nào?
> Client gửi header `Authorization: Bearer <token>`. FastAPI dùng `OAuth2PasswordBearer` rút token, rồi `get_current_user` decode + verify token, tìm user trong DB; hợp lệ → trả user, sai → 401.

> [!question] 2. 401 và 403 khác nhau ra sao?
> **401 Unauthorized**: chưa xác thực hợp lệ (thiếu/sai/hết hạn token). **403 Forbidden**: đã xác thực nhưng **không đủ quyền** (vd user thường gọi API admin).

> [!question] 3. `OAuth2PasswordBearer` làm gì?
> Là dependency **rút token** từ header `Authorization: Bearer`. `tokenUrl` cho Swagger biết endpoint lấy token (nút Authorize). Thiếu/sai header → tự trả 401.

> [!question] 4. Vì sao bảo vệ route bằng `Depends(get_current_user)` thay vì middleware/if?
> Vì nó **khai báo ngay trên chữ ký hàm** (đọc code biết route nào bảo vệ), **tái dùng** được, **tự giải cây phụ thuộc** (token + session), và **dễ test** (override). Middleware/if rải rác dễ sót và khó test.

> [!question] 5. RBAC là gì? Hiện thực trong FastAPI thế nào?
> Role-Based Access Control: quyền gắn theo **vai trò**. Hiện thực bằng claim `roles` trong JWT + dependency factory `require_role("admin")` kiểm tra vai trò, thiếu → 403.

> [!question] 6. Khi nào dùng `dependencies=[Depends(...)]` thay vì tham số `= Depends(...)`?
> Khi chỉ cần dependency **chạy để chặn** (không dùng giá trị) → đặt vào `dependencies=[...]` của route/router (vd áp auth cho cả nhóm). Khi cần **dùng kết quả** (như `current_user`) → khai báo làm tham số.

---

## 7. Bài tập tự luyện

1. Tạo `oauth2_scheme` + `get_current_user` (verify JWT, tìm user, 401 khi sai). Bảo vệ `GET /me`.
2. Thêm claim `roles` khi phát token; viết `require_role("admin")` và bảo vệ `DELETE /users/{id}` (user thường → 403).
3. Tạo `admin_router` với `dependencies=[Depends(require_role("admin"))]`, đặt vài route con, kiểm tra tất cả đều yêu cầu admin.
4. Thêm ownership check: `PATCH /todos/{id}` chỉ cho chủ sở hữu sửa (so `todo.owner_id == current_user.id`), khác → 403.

---

## 8. Liên quan
- [[08-Authentication-OAuth2-JWT]] — phát & verify JWT, claim `roles`
- [[06-Dependency-Injection]] — `get_current_user` là sub-dependency
- [[00-MOC-Backend|MOC: Backend]]
