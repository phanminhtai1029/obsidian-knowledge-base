---
title: "Implementing Streaming APIs (SSE token streaming, WebSocket chat, Connection Manager)"
section: 02-Backend
tags: [backend, fastapi, streaming, sse, websocket, connection-manager, token-streaming, async-generator, fresher, exam-focus]
related:
  - "[[14-Streaming-Foundations]]"
  - "[[16-Production-Streaming-Scaling]]"
difficulty: ⭐⭐⭐⭐⭐
estimated_time: 50m
source: [QN26_FR_AI_01, "09.Streaming-Pattern(SSE & Websocket)"]
---

# Implementing Streaming APIs

> [!summary] TL;DR
> Nền tảng kỹ thuật của mọi streaming trong FastAPI là **async generator** (`async def` + `yield`): trả từng chunk, `await` giữa các chunk để nhường event loop. Ba cách hiện thực: (1) **`StreamingResponse`** — stream HTTP thô bất kỳ; (2) **SSE** qua `EventSourceResponse` (`sse-starlette`) — chuẩn `text/event-stream`, dùng cho **token streaming kiểu OpenAI**; (3) **WebSocket** (`@app.websocket`) — hai chiều, `accept()` → vòng lặp `receive_text`/`send_text`, bắt **`WebSocketDisconnect`**. Để làm **chat nhiều người**, cần **Connection Manager** giữ danh sách kết nối và **broadcast** tin nhắn tới tất cả.

> ⭐ **Note trọng tâm thi** (mentor: SSE token streaming, WebSocket chat, Connection Manager).

---

## 1. Async generator — gốc của streaming

```python
import asyncio

async def fetching_content(path: str):
    with open(path, "r", encoding="utf-8") as f:
        while (line := f.readline()):
            yield line              # gửi từng dòng
            await asyncio.sleep(1)  # nhường event loop giữa các chunk

async for line in fetching_content("data.txt"):
    print(line)
```

- `yield` → gửi **một phần** ngay, chưa cần xong toàn bộ.
- `await` giữa các chunk → không block event loop (phục vụ request khác trong lúc chờ).

---

## 2. `StreamingResponse` — stream HTTP thô

Built-in của FastAPI, bọc một generator thành response stream:

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

app = FastAPI()

async def number_stream():
    for i in range(10):
        yield f"chunk {i}\n"
        await asyncio.sleep(0.5)

@app.get("/stream")
async def stream():
    return StreamingResponse(number_stream(), media_type="text/plain")
```

> Dùng khi muốn stream **bất kỳ định dạng** (text, NDJSON, file lớn) mà không cần chuẩn SSE.

---

## 3. SSE trong FastAPI (`sse-starlette`)

```python
# pip install sse-starlette
from sse_starlette.sse import EventSourceResponse
import asyncio

async def event_generator():
    i = 0
    while True:
        await asyncio.sleep(1)
        yield {"event": "message", "data": f"event {i}"}   # đúng khuôn SSE
        i += 1

@app.get("/stream")
async def stream():
    return EventSourceResponse(event_generator())
```

- `EventSourceResponse` tự đặt `Content-Type: text/event-stream` và format đúng chuẩn (`event:`, `data:`, dòng trống).
- Client trình duyệt dùng `new EventSource("/stream")` để nhận và **tự reconnect**.

### 3.1. SSE token streaming kiểu OpenAI (use-case AI)

Đây chính là cách ChatGPT "gõ từng chữ" — server stream **từng token** khi LLM sinh:

```python
@app.post("/chat/stream")
async def chat_stream(prompt: str):
    async def token_generator():
        # llm.astream(...) trả từng token bất đồng bộ
        async for token in llm.astream(prompt):
            yield {"event": "token", "data": token}
        yield {"event": "done", "data": "[DONE]"}     # tín hiệu kết thúc
    return EventSourceResponse(token_generator())
```

```
★ Insight ─────────────────────────────────────
• Token streaming dùng SSE chứ không phải WebSocket vì luồng MỘT CHIỀU: client
  gửi prompt một lần (POST), server stream token về. Đây là mẫu "Request Stream"
  ở [[14-Streaming-Foundations]].
• Quy ước [DONE]: SSE không có "tín hiệu hết" sẵn, nên server tự gửi một sự kiện
  kết thúc ([DONE] kiểu OpenAI) để client biết dừng. Thiếu nó client cứ chờ mãi.
─────────────────────────────────────────────────
```

---

## 4. WebSocket — hai chiều

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()                    # BẮT BUỘC: chấp nhận kết nối
    try:
        while True:
            data = await websocket.receive_text()   # nhận từ client
            await websocket.send_text(f"Echo: {data}")  # gửi lại
    except WebSocketDisconnect:
        print("Client disconnected")            # xử lý khi client ngắt
```

| Bước | API |
|---|---|
| Chấp nhận kết nối | `await websocket.accept()` |
| Nhận | `await websocket.receive_text()` / `.receive_json()` |
| Gửi | `await websocket.send_text(...)` / `.send_json(...)` |
| Ngắt kết nối | bắt `WebSocketDisconnect` |

> [!warning] Phải `accept()` và phải bắt `WebSocketDisconnect`
> Quên `accept()` → kết nối không mở. Không bắt `WebSocketDisconnect` → khi client đóng tab, vòng lặp `receive` ném exception làm rối log/treo tài nguyên. Luôn bọc `try/except`.

---

## 5. Connection Manager — chat nhiều người

WebSocket một client thì dễ; **chat phòng** cần giữ **mọi kết nối** và **broadcast**. Connection Manager là lớp quản lý đó:

```python
class ConnectionManager:
    def __init__(self):
        self.active: list[WebSocket] = []

    async def connect(self, ws: WebSocket):
        await ws.accept()
        self.active.append(ws)            # thêm vào danh sách

    def disconnect(self, ws: WebSocket):
        self.active.remove(ws)            # bỏ khỏi danh sách

    async def broadcast(self, message: str):
        for conn in self.active:          # gửi tới TẤT CẢ
            await conn.send_text(message)

manager = ConnectionManager()

@app.websocket("/ws/{user}")
async def chat(websocket: WebSocket, user: str):
    await manager.connect(websocket)
    await manager.broadcast(f"🟢 {user} joined")
    try:
        while True:
            msg = await websocket.receive_text()
            await manager.broadcast(f"{user}: {msg}")    # phát cho cả phòng
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        await manager.broadcast(f"🔴 {user} left")
```

| Method | Vai trò |
|---|---|
| `connect` | accept + thêm vào danh sách active |
| `disconnect` | xóa khỏi danh sách (khi ngắt) |
| `broadcast` | gửi message tới **mọi** kết nối |
| (mở rộng) `send_personal` | gửi riêng 1 client |

```
★ Insight ─────────────────────────────────────
• Connection Manager này giữ kết nối trong list NỘI BỘ của MỘT tiến trình. Chạy
  nhiều worker/nhiều server thì user ở worker A không broadcast tới user ở worker
  B được — vì mỗi worker có list riêng. Lời giải production: Redis Pub/Sub làm
  "event bus" chung (xem [[16-Production-Streaming-Scaling]]). Đây là giới hạn
  recruiter rất hay đào sâu sau khi bạn trình bày Connection Manager.
• broadcast lặp tuần tự gửi từng client — nếu một client chậm/treo sẽ ghìm cả
  vòng. Production cần gửi song song (asyncio.gather) + timeout + loại kết nối chết.
─────────────────────────────────────────────────
```

---

## 6. Q&A phỏng vấn

> [!question] 1. Nền tảng kỹ thuật của streaming trong FastAPI là gì?
> **Async generator** (`async def` + `yield`): trả từng chunk, `await` giữa các chunk để nhường event loop. `StreamingResponse`/`EventSourceResponse`/WebSocket đều bọc quanh ý tưởng này.

> [!question] 2. Khác nhau giữa `StreamingResponse` và `EventSourceResponse`?
> `StreamingResponse` stream HTTP **thô** (định dạng tùy ý). `EventSourceResponse` (sse-starlette) tuân chuẩn **SSE** (`text/event-stream`, format `event:`/`data:`), client dùng `EventSource` và tự reconnect — hợp token streaming.

> [!question] 3. Cách hiện thực token streaming kiểu OpenAI?
> Dùng **SSE**: endpoint POST nhận prompt, một async generator `async for token in llm.astream(...)` → `yield {"event":"token","data":token}`, kết thúc gửi `[DONE]`. Client nhận từng token và render dần.

> [!question] 4. Vòng đời một WebSocket endpoint gồm gì?
> `await accept()` → vòng lặp `receive_text/json` ⇄ `send_text/json` → bắt **`WebSocketDisconnect`** khi client ngắt để dọn dẹp. Quên accept → không mở; không bắt disconnect → lỗi/treo.

> [!question] 5. Connection Manager để làm gì?
> Quản lý **danh sách kết nối WebSocket đang mở** và cung cấp `connect`/`disconnect`/`broadcast` để gửi tin nhắn tới **tất cả** (chat phòng). Là nơi tập trung logic nhiều client.

> [!question] 6. Connection Manager in-memory có hạn chế gì khi scale?
> Nó giữ kết nối trong **một tiến trình**; nhiều worker/server thì không broadcast chéo được (mỗi worker list riêng). Cần **Redis Pub/Sub** làm event bus chung để fan-out giữa các tiến trình.

---

## 7. Bài tập tự luyện

1. Viết `/stream` dùng `StreamingResponse` phát 10 chunk cách 0.5s; rồi đổi sang `EventSourceResponse` và so sánh format nhận được.
2. Viết `/chat/stream` SSE giả lập token (tách câu thành từ, yield từng từ cách 0.1s), gửi `[DONE]` cuối; viết client `EventSource` (hoặc `requests stream`) nhận.
3. Viết WebSocket echo có `accept()` + bắt `WebSocketDisconnect`; test bằng `websockets` client.
4. Hiện thực `ConnectionManager` (connect/disconnect/broadcast) cho chat `/ws/{user}`; mở 2 client và kiểm tra broadcast. Sau đó cải tiến `broadcast` gửi song song bằng `asyncio.gather`.

---

## 8. Liên quan
- [[14-Streaming-Foundations]] — SSE vs WebSocket, khi nào dùng
- [[16-Production-Streaming-Scaling]] — Redis fan-out, rate limit, observability
- [[../04-AI/04-LangGraph-Agentic/01-LangGraph-Foundations-State|LangGraph stream]] — `astream` token từ LLM
- [[00-MOC-Backend|MOC: Backend]]
