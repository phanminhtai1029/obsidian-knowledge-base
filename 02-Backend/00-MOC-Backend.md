---
title: "MOC: Backend - FastAPI"
section: 02-Backend
tags: [moc, backend, fastapi, python, pydantic, sqlalchemy, fresher]
module: L3_PP_FASTAPI
---

# MOC: Backend — Building High-Performance APIs with FastAPI

> Module `L3_PP_FASTAPI`. Thi: Part 1 Theory (Quiz) + Part 2 Practice (Coding) + Final Project.
> ✅ Đã viết đủ 16 note — bám học liệu gốc (`_shared/_source/02-Backend-FastAPI`), bổ sung GraphQL + cụm Streaming/Production theo mentor dặn.

## Note

### Phần A — Nền tảng & Web cơ bản
| # | Note | Nội dung | Độ khó | Trạng thái |
|---|------|----------|--------|------------|
| 1 | [[01-FastAPI-Overview\|FastAPI Overview, Backend Architecture & Async]] | **Tier vs Layer vs Component**, 3-tier, N-layer, ASGI/WSGI, **sync vs async**, event loop, FastAPI intro | ⭐⭐⭐ | ✅ |
| 2 | [[02-Path-Query-Parameters\|Path & Query Parameters]] | Path/query params, type conversion, validation tự động | ⭐⭐ | ✅ |
| 3 | [[03-Pydantic-Data-Modeling\|Data Modeling (Pydantic)]] | BaseModel, nested, response_model, validator, Pydantic v2 | ⭐⭐⭐ | ✅ |

### Phần B — Database & Ứng dụng
| # | Note | Nội dung | Độ khó | Trạng thái |
|---|------|----------|--------|------------|
| 4 | [[04-SQLAlchemy-Database\|Database Setup (SQLAlchemy Async)]] | Async SQLAlchemy 2.0, AsyncSession, Mapped, asyncpg | ⭐⭐⭐⭐ | ✅ |
| 5 | [[05-Alembic-Migrations\|Alembic Migrations]] | Migration, autogenerate, revision | ⭐⭐⭐ | ✅ |
| 6 | [[06-Dependency-Injection\|Dependency Injection]] | `Depends()`, `get_db_session`, yield-dependency | ⭐⭐⭐ | ✅ |
| 7 | [[07-CRUD-Repository-Pattern\|CRUD & Repository Pattern]] | CRUD async, repository pattern, paging | ⭐⭐⭐ | ✅ |

### Phần C — Bảo mật & Kiểm thử
| # | Note | Nội dung | Độ khó | Trạng thái |
|---|------|----------|--------|------------|
| 8 | [[08-Authentication-OAuth2-JWT\|Authentication (JWT & OAuth2)]] | OAuth2 Password/Google flow, JWT, passlib | ⭐⭐⭐⭐ | ✅ |
| 9 | [[09-Authorization-RBAC\|Authorization & Current User]] | `OAuth2PasswordBearer`, `get_current_user`, RBAC | ⭐⭐⭐⭐ | ✅ |
| 10 | [[10-Testing-Pytest\|Testing với Pytest]] | Pytest, TestClient, async test, fixtures, mock DB | ⭐⭐⭐ | ✅ |
| 11 | [[11-Middleware-Error-CORS\|Middleware, Error Handling & CORS]] | Middleware, exception handler, CORS | ⭐⭐ | ✅ |
| 12 | [[12-Deployment-Uvicorn\|Deployment & Config]] | pydantic-settings, env vars, Uvicorn/Gunicorn | ⭐⭐ | ✅ |

### Phần D — GraphQL
| # | Note | Nội dung | Độ khó | Trạng thái |
|---|------|----------|--------|------------|
| 13 | [[13-GraphQL-Strawberry\|GraphQL với Strawberry]] | REST vs GraphQL, schema/type/resolver, query/mutation, N+1 | ⭐⭐⭐ | ✅ |

### Phần E — Streaming & Production *(trọng tâm thi — mentor dặn)*
| # | Note | Nội dung | Độ khó | Trạng thái |
|---|------|----------|--------|------------|
| 14 | [[14-Streaming-Foundations\|Streaming Foundations & Protocols]] | Kiến trúc streaming, pull vs push, **SSE vs WebSocket**, backpressure | ⭐⭐⭐⭐ | ✅ |
| 15 | [[15-Implementing-Streaming-APIs\|Implementing Streaming APIs]] | `StreamingResponse`, **SSE token streaming** (LLM), **WebSocket chat**, **Connection Manager** | ⭐⭐⭐⭐⭐ | ✅ |
| 16 | [[16-Production-Streaming-Scaling\|Production Streaming & Scaling]] | **Async pipeline**, **Redis event bus**, **Rate limiting**, **Observability**, **uvloop** | ⭐⭐⭐⭐⭐ | ✅ |

## Liên quan
- [[../03-Database/00-MOC-Database|MOC: Database]] — SQLAlchemy nối với DB
- [[../04-AI/03-LLMOps-Evaluation/00-MOC-LLMOps-Evaluation|MOC: LLMOps]] — Observability liên kết chéo (Note 16)
- [[../00-INDEX|🏠 Index tổng]]
