---
title: "Streaming Foundations & Protocols (SSE vs WebSocket)"
section: 02-Backend
tags: [backend, fastapi, streaming, sse, websocket, realtime, push-pull, backpressure, fresher, exam-focus]
related:
  - "[[15-Implementing-Streaming-APIs]]"
  - "[[16-Production-Streaming-Scaling]]"
  - "[[01-FastAPI-Overview]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 40m
source: [QN26_FR_AI_01, "09.Streaming-Pattern(SSE & Websocket)"]
---

# Streaming Foundations & Protocols (SSE vs WebSocket)

> [!summary] TL;DR
> Mô hình **request/response** cổ điển: client hỏi → server trả **một lần** → đóng. Không hợp với chat, dashboard real-time, hay **AI stream từng token**. **Streaming** giữ **kết nối mở (persistent)** và gửi dữ liệu **tăng dần (incremental)**. Có 2 kiểu kiến trúc: **Pull** (client liên tục hỏi — polling, lãng phí, trễ cao) vs **Push** (server chủ động đẩy khi có sự kiện — trễ thấp). Hai giao thức web phổ biến nhất: **SSE** (Server→Client một chiều, trên HTTP, tự reconnect — hợp **AI token streaming**, dashboard) và **WebSocket** (hai chiều full-duplex, trên TCP — hợp **chat**, game, collaborative). Chọn đúng giao thức theo **chiều dữ liệu** là câu hỏi thi cốt lõi.

> ⭐ **Cụm trọng tâm thi** (mentor dặn): Note 14–16. Bắt đầu từ nền tảng ở đây.

---

## 1. Vì sao request/response không đủ

```
Request/Response:  Client → request → Server → response (MỘT lần) → đóng kết nối
```

Không đáp ứng được:
- **AI token streaming**: muốn hiện chữ *ngay khi* model sinh, không chờ trả hết.
- **Live dashboard / metrics**: số liệu cập nhật liên tục.
- **Chat / game**: hai bên gửi qua lại bất cứ lúc nào.

### Khái niệm streaming cốt lõi

| Khái niệm | Ý nghĩa |
|---|---|
| **Persistent connection** | Kết nối **giữ mở** lâu, không đóng sau 1 response |
| **Incremental response** | Trả dữ liệu **từng phần** (chunk) thay vì trọn gói |
| **Backpressure** | Khi client tiêu thụ chậm hơn server sản xuất → cần cơ chế "ghìm" lại để không tràn bộ nhớ |
| **Connection lifecycle** | Vòng đời: connect → trao đổi → disconnect (xử lý ngắt giữa chừng) |

```python
# Ý tưởng streaming: yield từng chunk thay vì return một cục
async def stream_response():
    for chunk in ["Hello", "this", "is", "streaming"]:
        await asyncio.sleep(1)
        yield chunk        # gửi ngay từng phần
```

---

## 2. Pull vs Push

| | **Pull** (client hỏi) | **Push** (server đẩy) |
|---|---|---|
| Ai khởi tạo dữ liệu | Client | **Server** |
| Độ trễ | Cao (chờ lần hỏi sau) | **Thấp** |
| Dùng băng thông | Lãng phí (hỏi cả khi không có gì mới) | Hiệu quả |
| Real-time | Kém | **Tốt** |
| Ví dụ | **Polling** (lặp `GET` mỗi giây) | **WebSocket, SSE**, gRPC stream, Kafka |

```python
# Pull / Polling (lãng phí): hỏi đi hỏi lại
while True:
    result = requests.get(".../products")   # hỏi liên tục dù chưa chắc có dữ liệu mới
```

> Push **đảo chiều**: thay vì client hỏi mãi, server gửi *khi có sự kiện* → trễ thấp, đỡ lãng phí.

---

## 3. Ba mẫu giao tiếp real-time

| Mẫu | Mô tả | Dùng cho |
|---|---|---|
| **Request Stream** | Client hỏi 1 lần, server **stream kết quả** về | **AI token streaming**, stream kết quả tìm kiếm |
| **Event Stream** | Server **đẩy sự kiện liên tục** | Live dashboard, metrics monitoring |
| **Bidirectional Stream** | **Cả hai** cùng stream | Chat, multiplayer game, collaborative editing |

> Ánh xạ nhanh: Request Stream / Event Stream → **SSE**; Bidirectional → **WebSocket**.

---

## 4. Các giao thức streaming

| Protocol | Chiều | Transport |
|---|---|---|
| **HTTP Streaming** | Server → Client | HTTP |
| **Server-Sent Events (SSE)** | Server → Client | HTTP (`text/event-stream`) |
| **WebSocket** | **Hai chiều** | TCP |
| **gRPC Streaming** | Hai chiều | HTTP/2 |

Web app thường dùng 2 cái giữa: **SSE** và **WebSocket**.

### 4.1. SSE — Server-Sent Events

Server **đẩy sự kiện qua HTTP**, kết nối giữ mở.

| Đặc điểm | SSE |
|---|---|
| Chiều | **Một chiều** (Server → Client) |
| Transport | HTTP (không cần giao thức mới) |
| Content-Type | `text/event-stream` |
| Hỗ trợ trình duyệt | Native (`EventSource`) |
| Reconnect | **Tự động** |

Định dạng sự kiện (text đơn giản):
```
event: message
data: Hello world

event: update
data: {"value": 42}
```
> Mỗi sự kiện kết thúc bằng **một dòng trống**. `data:` bắt buộc, `event:` tùy chọn.

### 4.2. WebSocket — full-duplex

Kết nối **TCP bền**, hai bên gửi bất cứ lúc nào:
```
Client ⇄ WebSocket ⇄ Server   (cả hai chiều, đồng thời)
```
Dùng cho: chat, game nhiều người, collaborative editing, trading.

---

## 5. SSE vs WebSocket — bảng phân biệt (câu thi cốt lõi)

| Tiêu chí | **SSE** | **WebSocket** |
|---|---|---|
| Chiều | **Một chiều** (Server→Client) | **Hai chiều** (full-duplex) |
| Transport | **HTTP** | **TCP** (socket riêng) |
| Độ phức tạp | Đơn giản | Phức tạp hơn |
| Tự reconnect | **Có** (built-in) | Không (tự code lại) |
| Qua proxy/firewall | Dễ (là HTTP) | Đôi khi vướng |
| Trình duyệt | `EventSource` native | `WebSocket` API |
| Use case | **AI streaming**, dashboard, notifications | **Chat**, game, collaborative, trading |

```
★ Insight ─────────────────────────────────────
• Quy tắc chọn 1 giây: dữ liệu chỉ ĐI MỘT CHIỀU server→client (stream token AI,
  thông báo, dashboard)? → SSE (đơn giản, tự reconnect, là HTTP nên dễ deploy).
  Cần CẢ HAI CHIỀU tương tác liên tục (chat, game)? → WebSocket.
• Nhiều người mặc định WebSocket cho mọi thứ real-time — đó là over-engineer.
  Stream câu trả lời ChatGPT chỉ cần SSE: client gửi câu hỏi 1 lần (POST), server
  stream token về. Không cần socket hai chiều. Biết tiết chế là điểm cộng phỏng vấn.
• SSE chạy trên HTTP nên hưởng mọi hạ tầng HTTP sẵn có (auth header, proxy, load
  balancer). WebSocket "nâng cấp" (upgrade) từ HTTP lên TCP socket → một số proxy/
  LB cần cấu hình riêng để giữ kết nối.
─────────────────────────────────────────────────
```

---

## 6. Q&A phỏng vấn

> [!question] 1. Vì sao request/response cổ điển không hợp real-time?
> Vì nó trả **một lần rồi đóng** kết nối — không giữ được luồng dữ liệu liên tục cho chat, dashboard, hay stream token AI. Streaming giữ **kết nối mở** và gửi **tăng dần**.

> [!question] 2. Pull và Push khác nhau thế nào?
> **Pull (polling)**: client liên tục hỏi → trễ cao, lãng phí băng thông. **Push**: server chủ động đẩy khi có sự kiện → trễ thấp, hiệu quả. SSE/WebSocket là push.

> [!question] 3. SSE và WebSocket khác nhau ở điểm cốt lõi nào? Khi nào dùng cái nào?
> **SSE**: một chiều (Server→Client), trên HTTP, tự reconnect — hợp **AI token streaming**, dashboard, notification. **WebSocket**: hai chiều full-duplex, trên TCP — hợp **chat**, game, collaborative. Chọn theo **chiều dữ liệu**.

> [!question] 4. Stream câu trả lời của LLM nên dùng SSE hay WebSocket?
> **SSE** là đủ và hợp lý nhất: client gửi prompt một lần, server stream token về một chiều. WebSocket là thừa (over-engineer) cho ca này.

> [!question] 5. Backpressure là gì?
> Khi server **sản xuất nhanh hơn** client **tiêu thụ**, dữ liệu dồn lại → cần cơ chế "ghìm" (buffer giới hạn, chờ) để không tràn bộ nhớ. Chi tiết async pipeline ở [[16-Production-Streaming-Scaling]].

> [!question] 6. Ba mẫu giao tiếp real-time là gì?
> **Request Stream** (hỏi 1 lần, stream kết quả — AI token), **Event Stream** (server đẩy sự kiện liên tục — dashboard), **Bidirectional Stream** (hai bên stream — chat/game).

---

## 7. Bài tập tự luyện

1. Viết async generator `stream_response()` yield 4 chunk cách nhau 1s, tiêu thụ bằng `async for` và in kèm timestamp.
2. So sánh polling vs push: mô tả một tính năng "thông báo đơn mới" và lập luận chọn SSE hay WebSocket.
3. Lập bảng: với mỗi tính năng (chat 1-1, stream trả lời AI, dashboard CPU, game realtime) chọn giao thức và giải thích theo *chiều dữ liệu*.
4. Giải thích vì sao SSE dễ deploy qua proxy/LB hơn WebSocket.

---

## 8. Liên quan
- [[15-Implementing-Streaming-APIs]] — code SSE/WebSocket trong FastAPI
- [[16-Production-Streaming-Scaling]] — backpressure, fan-out, scaling
- [[01-FastAPI-Overview]] — ASGI hỗ trợ streaming/WebSocket (vì sao cần async)
- [[00-MOC-Backend|MOC: Backend]]
