---
title: "Authentication (JWT & OAuth2)"
section: 02-Backend
tags: [backend, fastapi, authentication, jwt, oauth2, security, password-hashing, fresher]
related:
  - "[[09-Authorization-RBAC]]"
  - "[[06-Dependency-Injection]]"
  - "[[11-Middleware-Error-CORS]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 45m
source: [QN26_FR_AI_01, "05.JWT-OAuth2", "06.Authentication (In FastAPI)"]
---

# Authentication (JWT & OAuth2)

> [!summary] TL;DR
> Phân biệt 4 khái niệm hay lẫn: **Authentication** (bạn *là ai*) ≠ **Authorization** (bạn *được làm gì*); **JWT** là *định dạng token* chứa claims; **OAuth2** là *framework cấp quyền truy cập*. Luồng phổ biến: user đăng nhập → server **băm mật khẩu** (bcrypt) để kiểm tra → cấp **JWT** ký bằng secret → client gửi JWT ở header `Authorization: Bearer <token>` cho request sau → server **verify chữ ký** thay vì lưu session (**stateless**). JWT gồm `header.payload.signature` (Base64URL) — **chỉ encode chứ KHÔNG mã hóa**, nên *tuyệt đối không* để mật khẩu/bí mật trong payload. FastAPI hỗ trợ **OAuth2 Password flow** qua endpoint `/token` + `OAuth2PasswordRequestForm`.

---

## 1. Bốn khái niệm nền tảng

| Khái niệm | Trả lời câu hỏi | Ví dụ |
|---|---|---|
| **Authentication** (AuthN) | *Bạn là ai?* | Đăng nhập email/mật khẩu |
| **Authorization** (AuthZ) | *Bạn được làm gì?* | "App được đọc Google Drive của bạn" |
| **JWT** | *Định dạng token* lưu claims | `{"sub":"123","role":"admin"}` |
| **OAuth2** | *Framework cấp quyền* | Đăng nhập bằng Google/GitHub |

> [!note] OAuth2 không phải giao thức xác thực
> OAuth2 lo **authorization** (cấp quyền truy cập tài nguyên), **không** tự xác nhận danh tính. Muốn "đăng nhập bằng Google" (xác thực) phải thêm tầng **OpenID Connect (OIDC)** trên OAuth2. JWT chỉ là *định dạng* — OAuth2 thường dùng JWT nhưng không bắt buộc.

---

## 2. JWT — JSON Web Token

JWT (RFC 7519) là chuỗi `header.payload.signature`, mỗi phần **Base64URL encode**:

| Phần | Nội dung |
|---|---|
| **Header** | loại token (JWT) + thuật toán ký (vd `HS256`) |
| **Payload** | các **claims** (dữ liệu): user id, roles, hạn dùng... |
| **Signature** | hash của header+payload bằng **secret key** → chống sửa đổi |

```
eyJhbGciOi...   .   eyJ1c2VyX2lk...   .   LwjJAa1U5bj...
   header              payload              signature
```

### Claims chuẩn hay gặp

| Claim | Ý nghĩa |
|---|---|
| `sub` | subject — định danh user |
| `exp` | expiration — thời điểm hết hạn |
| `iat` | issued at — thời điểm phát |
| `iss` | issuer — ai phát token |
| `aud` | audience — token dành cho ai |

> [!warning] JWT **được encode, KHÔNG được mã hóa** — đừng để bí mật trong payload
> Header & payload chỉ **Base64URL** (đảo ngược được) — ai cũng đọc được nội dung trên jwt.io mà không cần key. Chữ ký chỉ **chống sửa đổi**, *không che giấu*. ⇒ **Tuyệt đối không** để mật khẩu, thông tin nhạy cảm trong payload. (Học liệu có ví dụ đặt `password` vào token để minh họa — đó là **anti-pattern**, đừng làm thật.) Muốn che dữ liệu phải dùng **JWE** (JSON Web Encryption).

### Phát & xác minh JWT (PyJWT)

```python
import jwt
from datetime import datetime, timedelta, timezone

SECRET_KEY = "..."        # ĐỌC TỪ ENV, không hardcode (xem [[12-Deployment-Uvicorn]])
ALGORITHM = "HS256"

def create_jwt(user_id: str) -> str:
    now = datetime.now(timezone.utc)
    payload = {
        "sub": user_id,
        "iat": int(now.timestamp()),
        "exp": int((now + timedelta(minutes=15)).timestamp()),   # hạn ngắn
        "iss": "your-app", "aud": "your-client",
        "roles": ["user"],
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

def verify_jwt(token: str) -> dict:
    try:
        return jwt.decode(token, SECRET_KEY, algorithms=["HS256"],
                          audience="your-client", issuer="your-app")
    except jwt.ExpiredSignatureError:
        raise ValueError("Token has expired.")
    except jwt.InvalidTokenError as e:
        raise ValueError(f"Invalid token: {e}")
```

> `decode` **tự kiểm tra** chữ ký, `exp`, `aud`, `iss` → sai bất kỳ cái nào sẽ raise. (Thư viện: **PyJWT** hoặc **python-jose** — cú pháp tương tự.)

### HS256 vs RS256 (thuật toán ký)

| | **HS256** (đối xứng) | **RS256** (bất đối xứng) |
|---|---|---|
| Loại khóa | **1 secret** dùng chung | **private** ký / **public** verify |
| Ai verify được | Người có secret | Bất kỳ ai có public key |
| Xoay khóa | Khó | Dễ |
| Microservices | Rủi ro (phải chia sẻ secret) | **Ưu tiên** |
| Tốc độ | Nhanh hơn | Chậm hơn chút |

> Một dịch vụ đơn lẻ → HS256 đủ. Nhiều service cùng verify token (chỉ một nơi phát) → **RS256**: nơi phát giữ private key, các service khác chỉ cần public key.

```
★ Insight ─────────────────────────────────────
• "Stateless" là điểm mạnh & cũng là điểm yếu của JWT. Mạnh: server không lưu
  session, scale ngang dễ. Yếu: KHÔNG thu hồi được token đã phát trước hạn (logout
  tức thì khó). Cách chữa: hạn (exp) NGẮN + refresh token, hoặc thêm blacklist/
  token version trong DB (đánh đổi mất tính thuần stateless).
• Chữ ký ≠ mã hóa. Rất nhiều người tưởng JWT "an toàn nên nhét gì cũng được".
  Đúng là không SỬA được (signature), nhưng ĐỌC được hết (chỉ Base64). Quy tắc:
  payload chỉ chứa định danh + quyền hạn không nhạy cảm.
─────────────────────────────────────────────────
```

---

## 3. Băm mật khẩu (password hashing) — bcrypt

**Không bao giờ lưu mật khẩu dạng thô.** Lưu **hash** (một chiều) bằng `passlib` + `bcrypt`:

```python
from passlib.context import CryptContext
pwd = CryptContext(schemes=["bcrypt"], deprecated="auto")

hashed = pwd.hash("user-plain-password")      # khi đăng ký: lưu hashed vào DB
ok = pwd.verify("user-plain-password", hashed) # khi đăng nhập: so khớp
```

| Nguyên tắc | Vì sao |
|---|---|
| Lưu **hash**, không lưu plaintext | Lộ DB cũng không lộ mật khẩu gốc |
| Dùng **bcrypt/argon2**, không MD5/SHA1 | Chậm có chủ đích → chống brute-force |
| Có **salt** (bcrypt tự thêm) | 2 user cùng mật khẩu → hash khác nhau |

---

## 4. OAuth2 Password flow trong FastAPI

Luồng đơn giản nhất: client gửi username/password tới `/token`, nhận lại access token.

```python
from fastapi import FastAPI, Depends
from fastapi.security import OAuth2PasswordRequestForm

app = FastAPI()

@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = authenticate_user(form_data.username, form_data.password)  # verify hash
    if not user:
        raise HTTPException(status_code=401, detail="Sai tài khoản/mật khẩu")
    token = create_jwt(user.id)
    return {"access_token": token, "token_type": "bearer"}   # khuôn chuẩn OAuth2
```

- `OAuth2PasswordRequestForm = Depends()` → FastAPI bóc `username`/`password` từ **form-data** (chuẩn OAuth2), tự hiện ô nhập trong `/docs`.
- Trả về **đúng khuôn** `{"access_token", "token_type": "bearer"}`.
- Client các request sau gửi header: `Authorization: Bearer <access_token>` (xử lý ở [[09-Authorization-RBAC]]).

---

## 5. OAuth2 framework đầy đủ (Google/GitHub login)

Khi "đăng nhập bằng Google", có 4 vai trò & luồng cấp quyền:

| Vai trò | Là ai |
|---|---|
| **Resource Owner** | Người dùng (chủ dữ liệu) |
| **Client** | App của bạn (xin quyền) |
| **Authorization Server** | Nơi phát token (Google) |
| **Resource Server** | Nơi giữ tài nguyên (API Google Drive) |

**Core flow (Authorization Code):**
```
1. Authorization Request  : app chuyển user sang trang đồng ý của Google
2. Authorization Grant     : user đồng ý → app nhận "code"
3. Access Token Exchange   : app đổi code + client_secret lấy access token
4. API Access              : app dùng token gọi API tài nguyên
```

> Không bao giờ để `client_secret` lộ ra phía client (browser); việc đổi code → token phải làm ở **backend**.

---

## 6. Q&A phỏng vấn

> [!question] 1. Authentication và Authorization khác nhau thế nào?
> **Authentication** xác nhận *bạn là ai* (đăng nhập). **Authorization** quyết định *bạn được làm gì* (quyền truy cập). AuthN trước, AuthZ sau.

> [!question] 2. JWT gồm những phần nào? Có được mã hóa không?
> 3 phần `header.payload.signature` (Base64URL). **Không mã hóa** — chỉ encode (đọc được) + ký (chống sửa). Vì vậy không để dữ liệu nhạy cảm trong payload; muốn che phải dùng JWE.

> [!question] 3. Vì sao JWT là "stateless"? Ưu/nhược điểm?
> Server **không lưu session** — chỉ verify chữ ký bằng secret/public key. Ưu: scale ngang dễ. Nhược: **khó thu hồi token trước hạn** (logout tức thì khó) → chữa bằng exp ngắn + refresh token hoặc blacklist.

> [!question] 4. HS256 và RS256 khác gì? Khi nào dùng RS256?
> HS256 dùng **một secret chung** (ký & verify cùng khóa). RS256 dùng **private ký / public verify**. Dùng RS256 khi **nhiều service cùng verify** token (chỉ một nơi phát) — không phải chia sẻ secret, dễ xoay khóa.

> [!question] 5. Vì sao không lưu mật khẩu dạng thô? Dùng gì?
> Lộ DB sẽ lộ toàn bộ mật khẩu. Lưu **hash một chiều** bằng **bcrypt/argon2** (chậm có chủ đích, tự thêm salt) — đăng nhập thì `verify(plain, hash)`. Không dùng MD5/SHA1 (nhanh, dễ brute-force).

> [!question] 6. OAuth2 có phải giao thức xác thực không?
> Không. OAuth2 lo **cấp quyền** (authorization). Muốn xác thực danh tính ("login with Google") phải dùng **OpenID Connect** trên nền OAuth2.

> [!question] 7. `OAuth2PasswordRequestForm` và khuôn `{access_token, token_type}` để làm gì?
> `OAuth2PasswordRequestForm` bóc `username`/`password` từ form-data theo chuẩn OAuth2 (và tích hợp nút Authorize trong `/docs`). Trả về `{"access_token", "token_type":"bearer"}` là **khuôn chuẩn** để client/Swagger hiểu và tự gắn `Authorization: Bearer`.

---

## 7. Bài tập tự luyện

1. Viết `create_jwt`/`verify_jwt` (HS256) với claims `sub`, `exp` (15 phút), `roles`. Decode token hết hạn và bắt `ExpiredSignatureError`.
2. Dùng `passlib[bcrypt]` băm mật khẩu khi "đăng ký" và `verify` khi "đăng nhập"; chứng minh 2 user cùng mật khẩu cho hash khác nhau.
3. Viết endpoint `/token` dùng `OAuth2PasswordRequestForm`, trả `{access_token, token_type}`; sai mật khẩu → 401.
4. Giải thích (bằng lời) vì sao đặt `password` vào payload JWT là sai, và token có thể bị đọc ở đâu.

---

## 8. Liên quan
- [[09-Authorization-RBAC]] — `OAuth2PasswordBearer`, `get_current_user`, phân quyền
- [[06-Dependency-Injection]] — auth gắn vào endpoint qua `Depends`
- [[12-Deployment-Uvicorn]] — `SECRET_KEY` đọc từ env
- [[00-MOC-Backend|MOC: Backend]]
