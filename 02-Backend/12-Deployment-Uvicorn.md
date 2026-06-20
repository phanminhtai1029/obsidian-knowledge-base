---
title: "Deployment & Config"
section: 02-Backend
tags: [backend, fastapi, deployment, uvicorn, gunicorn, pydantic-settings, config, env, fresher]
related:
  - "[[01-FastAPI-Overview]]"
  - "[[08-Authentication-OAuth2-JWT]]"
  - "[[16-Production-Streaming-Scaling]]"
difficulty: ⭐⭐
estimated_time: 35m
source: [QN26_FR_AI_01, "03.Database Setup", "02.Building Async Web Services"]
---

# Deployment & Config

> [!summary] TL;DR
> Cấu hình app (chuỗi DB, `SECRET_KEY`, CORS...) **đọc từ biến môi trường / `.env`** bằng **`pydantic-settings`** (`BaseSettings`) — **không hardcode bí mật trong code**. Chạy app bằng **Uvicorn** (ASGI server). Dev: `uvicorn app.main:app --reload`. Production: nhiều tiến trình **worker** (`--workers N`) hoặc **Gunicorn + Uvicorn workers**, **tắt `--reload`**, tắt `echo`/`debug`. Đóng gói bằng **Docker**. Tối ưu vòng lặp sự kiện bằng **uvloop** (xem [[16-Production-Streaming-Scaling]]).

---

## 1. Config với `pydantic-settings`

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    database_url: str
    sync_database_url: str | None = None
    secret_key: str
    access_token_expire_minutes: int = 30
    cors_origins: list[str] = ["http://localhost:3000"]

    model_config = SettingsConfigDict(env_file=".env")   # tự đọc .env

settings = Settings()      # tự nạp từ biến môi trường / .env
```

`.env`:
```
DATABASE_URL=postgresql+asyncpg://user:pass@localhost:5432/mydb
SECRET_KEY=...
```

| Lợi ích | Giải thích |
|---|---|
| **Tách config khỏi code** | Đổi môi trường (dev/stag/prod) không sửa code |
| **Validate config** | `BaseSettings` ép kiểu & báo lỗi nếu thiếu/sai |
| **Bí mật an toàn** | Secret nằm ở env, **không commit** vào git |

> [!warning] Không hardcode bí mật & không commit `.env`
> `SECRET_KEY`, mật khẩu DB, API key **phải** đến từ env, không nhúng trong code (rò rỉ khi đẩy git). Thêm `.env` vào `.gitignore`. Đây cũng là nguyên tắc của **12-Factor App** (config qua môi trường).

> [!note] Pydantic v2: `BaseSettings` tách gói
> Trong Pydantic v2, `BaseSettings` chuyển sang gói riêng **`pydantic-settings`** (`pip install pydantic-settings`), không còn trong `pydantic` lõi như v1.

### Tiêm settings qua DI (cache)

```python
from functools import lru_cache
from fastapi import Depends

@lru_cache                      # đọc & dựng Settings MỘT lần
def get_settings() -> Settings:
    return Settings()

@app.get("/info")
async def info(cfg: Settings = Depends(get_settings)):
    return {"token_ttl": cfg.access_token_expire_minutes}
```

---

## 2. Chạy app với Uvicorn

**Uvicorn** là **ASGI server** (xem [[01-FastAPI-Overview#2. Tầng phục vụ|tầng phục vụ]]) — nó lắng nghe mạng và gọi vào app FastAPI.

```sh
# Dev: auto-reload khi đổi code
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000

# Production: nhiều worker, KHÔNG reload
uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```

| Cờ | Ý nghĩa | Môi trường |
|---|---|---|
| `--reload` | tự nạp lại khi sửa code | **chỉ dev** (đừng dùng prod) |
| `--workers N` | N tiến trình xử lý song song | production |
| `--host 0.0.0.0` | nghe mọi network interface | production/container |

> `app.main:app` = "trong module `app/main.py`, lấy biến tên `app`".

### Gunicorn + Uvicorn workers (cổ điển cho production)

```sh
gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

> Gunicorn làm **process manager** (quản lý, tự restart worker chết), mỗi worker chạy Uvicorn. `-w 4` ≈ `2×số_core + 1` là điểm khởi đầu thường gặp.

```
★ Insight ─────────────────────────────────────
• Worker = TIẾN TRÌNH riêng (parallelism), khác với async (concurrency trong 1
  tiến trình). Kết hợp cả hai: N worker, mỗi worker một event loop phục vụ nhiều
  request đồng thời → tận dụng cả nhiều core lẫn I/O concurrency. Đây là lý do
  "async rồi sao còn cần nhiều worker": async lo I/O, worker lo nhiều CPU core.
• --reload theo dõi file thay đổi → tốn tài nguyên & không an toàn. CHỈ dùng dev.
  Production phải tắt; bật nhầm là một lỗi deploy kinh điển.
─────────────────────────────────────────────────
```

---

## 3. Đóng gói bằng Docker (tối giản)

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

> Container hóa giúp môi trường nhất quán (dev = prod). Postgres thường chạy container riêng (`docker compose`), app trỏ tới qua `DATABASE_URL`.

---

## 4. uvloop — tăng tốc event loop (xem trước)

**uvloop** là cài đặt event loop nhanh hơn `asyncio` mặc định (viết trên libuv). Cài `uvicorn[standard]` thì Uvicorn **tự dùng uvloop** nếu có. Chi tiết & benchmark để ở [[16-Production-Streaming-Scaling]].

---

## 5. Checklist production

| Hạng mục | Dev | Production |
|---|---|---|
| `--reload` | ✅ | ❌ tắt |
| `echo=True` (SQL log) | ✅ debug | ❌ tắt (ồn, lộ query) |
| `SECRET_KEY` | tạm | từ env, đủ mạnh |
| CORS | rộng | **chặt** (origin cụ thể) |
| Workers | 1 | nhiều (`-w`) |
| HTTPS | không | có (qua reverse proxy: Nginx/Traefik) |
| Migration | tay | `alembic upgrade head` khi deploy ([[05-Alembic-Migrations]]) |

---

## 6. Q&A phỏng vấn

> [!question] 1. Vì sao dùng `pydantic-settings` thay vì hardcode config?
> Để **tách config khỏi code** (đổi môi trường không sửa code), **validate** kiểu config, và **giữ bí mật ở env** (không commit). Theo nguyên tắc 12-Factor App.

> [!question] 2. Uvicorn là gì? Khác FastAPI ra sao?
> Uvicorn là **ASGI server** lắng nghe mạng và gọi vào app. **FastAPI** chỉ là **framework** (app), không tự nghe cổng. Uvicorn chạy FastAPI.

> [!question] 3. `--reload` dùng khi nào? Vì sao không dùng ở production?
> Chỉ **dev** — tự nạp lại khi sửa code. Production tắt vì nó theo dõi file (tốn tài nguyên), không ổn định và không cần thiết.

> [!question] 4. Đã async rồi, sao còn cần nhiều worker?
> Async lo **I/O concurrency** trong **một** tiến trình/event loop (một core). **Worker** là nhiều **tiến trình** → tận dụng **nhiều core** (parallelism). Kết hợp cả hai để dùng hết CPU lẫn lợi ích async.

> [!question] 5. Gunicorn + UvicornWorker để làm gì?
> Gunicorn làm **process manager** (quản lý, restart worker chết), mỗi worker là một Uvicorn chạy app async. Cho production ổn định với nhiều tiến trình.

> [!question] 6. Vài thứ phải đổi khi đưa app từ dev lên production?
> Tắt `--reload` và `echo`, `SECRET_KEY` mạnh từ env, CORS chặt theo origin cụ thể, nhiều worker, bật HTTPS qua reverse proxy, chạy migration (`alembic upgrade head`).

---

## 7. Bài tập tự luyện

1. Viết `Settings(BaseSettings)` đọc `DATABASE_URL`, `SECRET_KEY`, `CORS_ORIGINS` từ `.env`; tiêm qua `get_settings` có `lru_cache`.
2. Chạy app dev với `--reload`, rồi production-style với `--workers 4`; quan sát số tiến trình.
3. Viết `Dockerfile` + `docker compose` (app + Postgres), build & chạy, gọi `/docs`.
4. Liệt kê 5 thứ bạn sẽ kiểm tra trong checklist trước khi deploy và giải thích vì sao.

---

## 8. Liên quan
- [[01-FastAPI-Overview]] — ASGI server, tầng phục vụ, async vs parallelism
- [[08-Authentication-OAuth2-JWT]] — `SECRET_KEY` từ env
- [[05-Alembic-Migrations]] — migration khi deploy
- [[16-Production-Streaming-Scaling]] — uvloop, scaling sâu hơn
- [[00-MOC-Backend|MOC: Backend]]
