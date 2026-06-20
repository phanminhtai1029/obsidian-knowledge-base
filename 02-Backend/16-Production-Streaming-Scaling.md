---
title: "Production Streaming & Scaling (Async pipeline, Redis event bus, Rate limiting, Observability, uvloop)"
section: 02-Backend
tags: [backend, fastapi, streaming, production, async-pipeline, redis, pubsub, rate-limiting, observability, uvloop, scaling, fresher, exam-focus]
related:
  - "[[14-Streaming-Foundations]]"
  - "[[15-Implementing-Streaming-APIs]]"
  - "[[12-Deployment-Uvicorn]]"
difficulty: ⭐⭐⭐⭐⭐
estimated_time: 55m
source: [QN26_FR_AI_01, "09.Streaming-Pattern(SSE & Websocket)"]
---

# Production Streaming & Scaling

> [!summary] TL;DR
> Đưa streaming lên production cần giải 5 bài toán mentor dặn: (1) **Async pipeline** — tách *sản xuất* và *tiêu thụ* qua `asyncio.Queue` có **giới hạn** để xử lý **backpressure**; (2) **Fan-out + Redis event bus** — `ConnectionManager` in-memory không broadcast được giữa nhiều worker, nên dùng **Redis Pub/Sub** làm bus chung; (3) **Rate limiting** — chặn lạm dụng bằng **token bucket / sliding window**, distributed qua Redis; (4) **Observability** — đo *active connections, tokens/sec, latency P99*, structured log + tracing (nối với [[../04-AI/03-LLMOps-Evaluation/00-MOC-LLMOps-Evaluation|LLMOps]]); (5) **uvloop** — thay event loop mặc định bằng libuv để nhanh hơn 2–4×. Mục tiêu: phục vụ **hàng triệu kết nối** ổn định.

> ⭐ **Note trọng tâm thi** — phủ trọn 5 từ khóa mentor: *async pipeline, connection manager (scaling), redis event bus, rate limiting, observability, uvloop optimization*.

---

## 1. Token streaming ở production

AI API hiện đại stream token *ngay khi sinh* (xem [[15-Implementing-Streaming-APIs#3.1|SSE token streaming]]). Ở production còn thêm: cancel khi client đóng, timeout, đo throughput, rate limit theo user — các mục dưới.

---

## 2. Async pipeline & backpressure

**Vấn đề:** server **sản xuất** dữ liệu (token, sự kiện) nhanh hơn client **tiêu thụ** → nếu cứ đẩy, bộ nhớ phình → OOM. Cần **tách** producer ↔ consumer qua một **hàng đợi có giới hạn**.

```python
import asyncio

async def producer(queue: asyncio.Queue):
    async for token in llm.astream(prompt):
        await queue.put(token)      # nếu queue ĐẦY → tự CHỜ (backpressure)
    await queue.put(None)           # sentinel báo hết

async def consumer(queue: asyncio.Queue, ws):
    while (token := await queue.get()) is not None:
        await ws.send_text(token)

queue = asyncio.Queue(maxsize=100)  # GIỚI HẠN → ghìm producer khi đầy
await asyncio.gather(producer(queue), consumer(queue, ws))
```

| Khái niệm | Ý nghĩa |
|---|---|
| **Producer** | bên tạo dữ liệu (LLM, sự kiện) |
| **Consumer** | bên gửi/ghi dữ liệu (client, DB) |
| **Bounded queue** | `Queue(maxsize=N)` — hàng đợi giới hạn |
| **Backpressure** | queue đầy → `put` *chờ* → producer chậm lại theo consumer |

```
★ Insight ─────────────────────────────────────
• maxsize chính là "van" backpressure. maxsize=0 (vô hạn) là bẫy: producer nhanh
  sẽ nhồi RAM tới khi sập. Đặt giới hạn = producer tự động khớp tốc độ consumer.
• Pipeline còn cho phép chèn bước xử lý giữa producer→consumer (lọc, gộp token,
  kiểm duyệt nội dung) mà không khóa hai đầu — đây là "streaming middleware".
─────────────────────────────────────────────────
```

---

## 3. Fan-out & Redis event bus (scaling Connection Manager)

**Fan-out** = *một* sự kiện cần tới *nhiều* client (vd tin nhắn phòng chat, giá cổ phiếu).

**Vấn đề scaling:** `ConnectionManager` ([[15-Implementing-Streaming-APIs#5|Note 15]]) giữ kết nối trong **một tiến trình**. Khi chạy nhiều worker/nhiều máy (cần thiết để chịu tải), user ở worker A **không** broadcast tới user ở worker B được — mỗi worker một danh sách riêng.

```
Worker A: [user1, user2]      user1 gửi tin
Worker B: [user3, user4]      → user3, user4 KHÔNG nhận được ❌
```

**Lời giải: Redis Pub/Sub** làm **event bus** chung cho mọi worker:

```python
import redis.asyncio as redis
r = redis.from_url("redis://localhost")

# Khi nhận tin từ client: PUBLISH lên kênh chung (mọi worker đang subscribe)
async def on_message(msg: str):
    await r.publish("chat_room", msg)

# Mỗi worker SUBSCRIBE kênh, nhận tin → broadcast cho client CỦA MÌNH
async def redis_listener(manager: ConnectionManager):
    pubsub = r.pubsub()
    await pubsub.subscribe("chat_room")
    async for message in pubsub.listen():
        if message["type"] == "message":
            await manager.broadcast(message["data"].decode())
```

```
        Worker A ──publish──┐                 ┌──subscribe── Worker A → client A
user1 ─▶ (broadcast local)  ├──▶  REDIS  ──▶ ─┤
        Worker B ──publish──┘   (Pub/Sub)     └──subscribe── Worker B → client B,C
```

| Vai trò | Ai làm |
|---|---|
| **Publisher** | worker nhận tin từ client → `PUBLISH` |
| **Event bus** | **Redis** (kênh Pub/Sub) |
| **Subscriber** | mọi worker `SUBSCRIBE` → broadcast tới client local |

> Nhờ Redis, broadcast **xuyên tiến trình/máy**. Đây là kiến trúc chuẩn để chat/realtime scale ngang.

---

## 4. Rate limiting

**Vì sao:** chặn lạm dụng (spam, brute-force, tốn tiền token LLM), bảo vệ tài nguyên, đảm bảo công bằng giữa user.

| Thuật toán | Ý tưởng | Đặc điểm |
|---|---|---|
| **Fixed window** | Đếm request trong mỗi khung thời gian cố định | Đơn giản; "burst" ở ranh giới khung |
| **Sliding window** | Cửa sổ trượt theo thời gian | Mượt hơn, chính xác hơn |
| **Token bucket** | Mỗi user một "xô token", request tốn 1 token, token nạp đều theo thời gian | Cho phép **burst** có kiểm soát — phổ biến nhất |

```python
# Ví dụ dùng slowapi (giới hạn theo IP)
from slowapi import Limiter
from slowapi.util import get_remote_address
limiter = Limiter(key_func=get_remote_address)

@app.get("/chat")
@limiter.limit("5/minute")        # tối đa 5 request/phút mỗi IP
async def chat(request: Request):
    ...
```

> [!warning] Rate limit phải **distributed** khi nhiều worker
> Đếm trong bộ nhớ mỗi worker → user gọi worker khác nhau sẽ vượt giới hạn thật. Production lưu counter ở **Redis** (atomic `INCR`+`EXPIRE` hoặc token bucket trên Redis) để giới hạn **toàn cụm**. Vượt giới hạn → trả **429 Too Many Requests** (kèm header `Retry-After`).

---

## 5. Observability cho streaming

Streaming là kết nối **sống lâu** → khó quan sát hơn request/response. Cần đo:

| Loại tín hiệu | Chỉ số streaming cần theo dõi |
|---|---|
| **Metrics** | số **active connections**, **messages/tokens per second**, độ dài kết nối, tỉ lệ ngắt |
| **Latency** | **time-to-first-token (TTFT)**, độ trễ giữa các token, **P99** (không chỉ trung bình) |
| **Logs** | structured log mỗi connect/disconnect/error (kèm request-id) |
| **Tracing** | trace xuyên suốt: request → LLM → stream (OpenTelemetry) |

> [!tip] Đo P99, không chỉ mean
> Trung bình che giấu đuôi xấu. Một số ít kết nối trễ cao (P99) phá trải nghiệm. Cùng tư duy với observability ở môn LLMOps → xem [[../04-AI/03-LLMOps-Evaluation/02-Observability-LangFuse-LangSmith|Observability (LLMOps)]] để nối backend metrics với trace LLM.

```python
# Ví dụ đếm kết nối đang mở (ý tưởng — đẩy vào Prometheus)
active_connections = 0

@app.websocket("/ws")
async def ws(websocket: WebSocket):
    global active_connections
    await websocket.accept(); active_connections += 1
    try:
        ...
    finally:
        active_connections -= 1        # luôn giảm khi đóng (kể cả lỗi)
```

---

## 6. uvloop — tối ưu event loop

**uvloop** là cài đặt thay thế cho event loop mặc định của `asyncio`, viết trên **libuv** (cùng nền Node.js). Nhanh hơn **~2–4×** ở I/O — giảm latency, tăng throughput, **không sửa code**.

```python
# Cách 1: cài uvicorn[standard] → Uvicorn TỰ dùng uvloop nếu có
# uvicorn app.main:app  (tự chọn loop="uvloop")

# Cách 2: ép tường minh
import uvicorn
uvicorn.run("app.main:app", loop="uvloop")

# Cách 3: set policy thủ công (script asyncio thuần)
import asyncio, uvloop
asyncio.set_event_loop_policy(uvloop.EventLoopPolicy())
```

| | asyncio mặc định | **uvloop** |
|---|---|---|
| Nền tảng | Python thuần | **libuv** (C) |
| Hiệu năng I/O | cơ bản | **~2–4× nhanh hơn** |
| Cần sửa code | — | Không (chỉ đổi loop) |
| Nền tảng | mọi OS | Linux/macOS (không Windows) |

> [!note] uvloop hợp với streaming/nhiều kết nối
> Streaming = nhiều kết nối I/O đồng thời sống lâu → đúng "sở trường" của uvloop. Đây là tối ưu **rẻ nhất** (đổi một dòng cấu hình). Liên kết deploy: [[12-Deployment-Uvicorn]].

```
★ Insight ─────────────────────────────────────
• Thứ tự tối ưu nên theo: (1) uvloop (gần như free) → (2) nhiều worker (dùng hết
  core) → (3) async pipeline + backpressure (ổn định bộ nhớ) → (4) Redis fan-out
  (scale ngang) → (5) rate limit + observability (bảo vệ & nhìn thấy). Đừng nhảy
  vào tối ưu vi mô trước khi làm mấy bước "to" này.
• "Hàng triệu kết nối" KHÔNG đạt được bằng một process to hơn, mà bằng: nhiều
  process/máy (scale ngang) + event bus chung (Redis) + kết nối nhẹ (SSE/WebSocket
  async, không thread mỗi kết nối). Đây là khác biệt tư duy sync-thread vs async.
─────────────────────────────────────────────────
```

---

## 7. Q&A phỏng vấn

> [!question] 1. Async pipeline & backpressure giải quyết vấn đề gì?
> Khi producer nhanh hơn consumer, dữ liệu dồn → tràn RAM. Tách hai bên qua **`asyncio.Queue(maxsize=N)`**: queue đầy thì `put` chờ → producer tự chậm lại khớp consumer (**backpressure**). `maxsize` là "van" điều tiết.

> [!question] 2. Vì sao Connection Manager in-memory không scale? Khắc phục?
> Nó giữ kết nối trong **một tiến trình**; nhiều worker/máy thì không broadcast chéo (mỗi worker list riêng). Khắc phục bằng **Redis Pub/Sub**: worker `PUBLISH` lên kênh chung, mọi worker `SUBSCRIBE` rồi broadcast tới client local → fan-out xuyên tiến trình.

> [!question] 3. Các thuật toán rate limiting? Cái nào phổ biến và vì sao cần distributed?
> **Fixed window / sliding window / token bucket**. **Token bucket** phổ biến (cho burst có kiểm soát). Khi nhiều worker phải đếm ở **Redis** (atomic) để giới hạn toàn cụm; vượt → **429** kèm `Retry-After`.

> [!question] 4. Observability cho streaming cần đo gì khác request/response?
> Vì kết nối sống lâu: đo **active connections**, **tokens/messages per second**, **time-to-first-token**, độ trễ giữa token, tỉ lệ ngắt — và xem **P99** chứ không chỉ mean. Kèm structured log connect/disconnect + tracing.

> [!question] 5. uvloop là gì? Bật thế nào? Lợi gì?
> Event loop thay thế dựa trên **libuv**, nhanh hơn asyncio ~2–4× ở I/O. Bật bằng `uvicorn[standard]` (tự dùng) hoặc `loop="uvloop"` / set policy. Không phải sửa code logic → tối ưu rẻ nhất, hợp streaming nhiều kết nối.

> [!question] 6. Làm sao một hệ thống chịu được hàng triệu kết nối streaming?
> **Scale ngang** (nhiều process/máy) + **event bus chung** (Redis Pub/Sub) cho fan-out + **kết nối async nhẹ** (không thread mỗi kết nối) + uvloop + backpressure + rate limit. Không phải một process to hơn.

> [!question] 7. Token streaming bị client đóng giữa chừng thì nên làm gì?
> Phát hiện ngắt (WebSocketDisconnect / client đóng SSE) → **hủy** tác vụ sinh token (cancel coroutine/queue), giải phóng tài nguyên & **dừng tính phí** LLM. Đừng để producer chạy tiếp vô ích.

---

## 8. Bài tập tự luyện

1. Viết pipeline producer→`Queue(maxsize=10)`→consumer cho token; cho producer nhanh, consumer chậm, quan sát backpressure ghìm producer.
2. Mở rộng `ConnectionManager` chat dùng **Redis Pub/Sub**; chạy 2 instance app (2 cổng) và kiểm tra tin nhắn broadcast chéo.
3. Thêm rate limit `5/minute` cho endpoint chat (slowapi), trả 429 khi vượt; rồi phác cách lưu counter ở Redis cho nhiều worker.
4. Thêm metric `active_connections` + đo time-to-first-token; bật uvloop và so sánh throughput trước/sau.

---

## 9. Liên quan
- [[14-Streaming-Foundations]] — nền tảng SSE/WebSocket
- [[15-Implementing-Streaming-APIs]] — Connection Manager (bản in-memory)
- [[12-Deployment-Uvicorn]] — uvloop, workers, deploy
- [[../04-AI/03-LLMOps-Evaluation/02-Observability-LangFuse-LangSmith|Observability (LLMOps)]] — đo P99, trace
- [[00-MOC-Backend|MOC: Backend]]
