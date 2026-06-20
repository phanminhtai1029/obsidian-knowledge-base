---
title: "LangGraph Foundations & State Management"
section: 04-AI/04-LangGraph-Agentic
tags: [ai, langgraph, state-management, stategraph, messages, reducer, fresher]
related:
  - "[[02-Agentic-Patterns]]"
  - "[[05-Human-in-the-Loop-Persistence]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 40m
source: [QN26_FR_AI_01, "01-langgraph-foundations-state-management.mdx"]
---

# LangGraph Foundations & State Management

> [!summary] TL;DR
> **LangChain chains chỉ chạy thẳng** (A → B → C), không vòng lặp, không rẽ nhánh — không đủ để xây agent. **LangGraph** là framework điều phối LLM dạng **đồ thị có hướng (StateGraph)**: cho phép **vòng lặp**, **rẽ nhánh có điều kiện**, và **state tường minh** chia sẻ giữa các bước. Nguyên tắc cốt lõi: **messages-centric** — mọi I/O giữa các node đi qua trường `messages` (kiểu `Annotated[List[AnyMessage], add_messages]`), còn dữ liệu cấu hình/theo dõi để ở **context fields** riêng. 3 khối xây dựng: **Node** (hàm xử lý, trả về *state delta*), **Edge** (nối node — thường hoặc có điều kiện), **State** (TypedDict). `add_messages` là **reducer** tự gộp message mới vào lịch sử (append, không ghi đè).

---

## 1. LangGraph là gì? Vì sao cần?

LangGraph là **orchestration framework cho LLM**, do chính team LangChain xây, **kế thừa** các thành phần LangChain (dùng lại `AIMessage`, `HumanMessage`...), nhưng bổ sung khả năng quản lý **state** và luồng phức tạp.

**Vấn đề của LangChain chain:** chỉ hỗ trợ luồng **tuyến tính**:

```python
chain = prompt | llm | output_parser   # chỉ A → B → C, một chiều
```

Agent thực tế cần nhiều hơn:

| Nhu cầu | LangGraph giải quyết bằng |
|---|---|
| **Luồng phức tạp** | `add_conditional_edges` — rẽ nhánh theo trạng thái |
| **Vòng lặp** (retry, ReAct) | Cạnh quay ngược: `add_edge("tools", "agent")` |
| **Human-in-the-loop** | Tạm dừng đồ thị, chèn `HumanMessage` |
| **Stateful** (nhớ hội thoại) | `messages` lưu lịch sử tự nhiên |
| **Multi-agent** | Nhiều node chia sẻ chung `messages` |

### LangChain vs LangGraph

| Khía cạnh | **LangChain** | **LangGraph** |
|---|---|---|
| Kiểu luồng | Tuyến tính, tuần tự | **Có chu trình**, có điều kiện |
| State | Ngầm (implicit) | **Tường minh** qua `messages` |
| Lịch sử message | Chỉ trong chain | **Bền** trong state (+ checkpointer) |
| Vòng lặp | Không hỗ trợ | **Hỗ trợ gốc** |
| Rẽ nhánh | Hạn chế | Linh hoạt |
| Dùng cho | Pipeline đơn giản | Agent phức tạp, multi-turn |

```
★ Insight ─────────────────────────────────────
• Đừng nghĩ LangGraph "thay thế" LangChain — nó ĐỨNG TRÊN LangChain. Node của
  bạn vẫn gọi `llm.invoke()`, vẫn dùng message types của LangChain. LangGraph
  chỉ thêm tầng "điều phối có state + vòng lặp" mà chain không làm được.
• Điểm khác biệt triết lý: chain là HÀM (input→output, không nhớ). Graph là MÁY
  TRẠNG THÁI (state machine) — mỗi node đọc state, trả về phần thay đổi của
  state. Đây là vì sao agent (cần nhớ, cần lặp) phải dùng graph.
─────────────────────────────────────────────────
```

---

## 2. StateGraph & State — mô hình messages-centric

`StateGraph` là đồ thị có hướng để điều phối luồng LLM. Khai báo kèm **kiểu state**:

```python
from langgraph.graph import StateGraph, END
workflow = StateGraph(AgentState)   # AgentState là TypedDict mô tả state
```

### State: `messages` (I/O) vs context (metadata)

> 🔑 **Nguyên tắc vàng:** `messages` là **kênh I/O chính** cho MỌI node; các trường khác là **context** (cấu hình, theo dõi, không phải I/O hội thoại).

```python
from typing import TypedDict, List, Annotated
from langchain_core.messages import AnyMessage
from langgraph.graph import add_messages

class AgentState(TypedDict):
    # CORE: kênh I/O — bắt buộc
    messages: Annotated[List[AnyMessage], add_messages]

    # CONTEXT: metadata, KHÔNG phải I/O
    user_id: str
    session_id: str
    max_iterations: int
    current_iteration: int
```

**Khi nào dùng context fields?** Cấu hình (`max_iterations`, timeout), metadata (`user_id`, timestamp), theo dõi (đếm vòng lặp, metric), dữ liệu phi-hội-thoại (đường dẫn file, API key) — và khi muốn **truyền context từ ngoài vào** cho tool.

> [!tip] Vì sao messages là "core"?
> 1. **I/O chuẩn hoá** — mọi node đọc/ghi cùng một kênh. 2. **Tương thích LangChain** — LLM/tool/agent đều "ăn" messages. 3. **Tự tích luỹ lịch sử** hội thoại. 4. **Type-safe** — phân biệt `AIMessage`/`HumanMessage`/`SystemMessage`/`ToolMessage`.

### Các loại Message

```python
from langchain_core.messages import (
    AIMessage,      # LLM trả lời
    HumanMessage,   # người dùng nhập
    SystemMessage,  # system prompt (vai trò, hướng dẫn)
    ToolMessage,    # kết quả tool trả về
    FunctionMessage # function call (deprecated)
)
```

| Loại | Ai tạo ra | Vai trò |
|---|---|---|
| `SystemMessage` | Lập trình viên | Đặt vai trò/luật cho LLM, thường ở đầu |
| `HumanMessage` | Người dùng | Câu hỏi/đầu vào |
| `AIMessage` | LLM | Câu trả lời — **có thể chứa `tool_calls`** |
| `ToolMessage` | Hệ thống | Kết quả tool, gắn `tool_call_id` khớp với lời gọi |

---

## 3. ⭐ `add_messages` Reducer — vì sao quan trọng

`add_messages` là **reducer** đặc biệt cho trường messages. **Reducer** = hàm quyết định state mới được **gộp** thế nào với delta mà node trả về.

```python
messages: Annotated[List[AnyMessage], add_messages]
#                                     └── reducer: APPEND, không ghi đè
```

**Hành vi:**
- **Append** message mới vào list (không thay thế list cũ).
- Xử lý **ID & khử trùng lặp** (dedup).
- Gộp message thông minh.

```python
# Node 1 trả về {"messages": [AIMessage("Hello")]}
# → state: [AIMessage("Hello")]
# Node 2 trả về {"messages": [HumanMessage("Hi")]}
# → state: [AIMessage("Hello"), HumanMessage("Hi")]   ← TÍCH LUỸ, không mất cái cũ
```

```
★ Insight ─────────────────────────────────────
• Đây là chỗ nhiều người mới NGỠ NGÀNG: node chỉ trả về phần MỚI (`return
  {"messages": [response]}`), KHÔNG trả về toàn bộ list. Reducer `add_messages`
  tự cộng dồn. Nếu KHÔNG có reducer (chỉ `messages: List[AnyMessage]`), giá trị
  trả về sẽ GHI ĐÈ list cũ → mất sạch lịch sử. Reducer chính là "phép cộng"
  của state.
• Với các trường context thường (vd `current_iteration`), không có reducer →
  hành vi mặc định là GHI ĐÈ. Đó là lý do node ReAct trả về
  `current_iteration + 1` (đọc cũ, ghi mới) thay vì mong nó tự tăng.
─────────────────────────────────────────────────
```

---

## 4. Nodes & Edges

### Node — hàm nhận state, trả về *state delta*

```python
def my_node(state: AgentState) -> dict:
    messages = state["messages"]          # 1. đọc state
    response = llm.invoke(messages)        # 2. xử lý (gọi LLM/tool)
    return {"messages": [response]}        # 3. trả về PHẦN THAY ĐỔI
```

Node LLM cơ bản và node Tool:

```python
def llm_node(state: AgentState) -> dict:
    return {"messages": [llm.invoke(state["messages"])]}

def tool_node(state: AgentState) -> dict:
    tool_call = state["messages"][-1].tool_calls[0]   # lấy lời gọi tool
    result = execute_tool(tool_call)
    return {"messages": [ToolMessage(content=str(result),
                                     tool_call_id=tool_call["id"])]}
```

> [!warning] `tool_call_id` phải khớp
> `ToolMessage` bắt buộc gắn `tool_call_id` trùng với id trong `tool_calls` của `AIMessage` trước đó. LLM dùng id này để biết kết quả nào ứng với lời gọi nào. Sai/thiếu id → lỗi hoặc LLM hiểu nhầm.

### Edge — nối các node

```python
# Cạnh thường: luôn đi A → B
workflow.add_edge("node_a", "node_b")

# Cạnh có điều kiện: hàm router trả về tên nhánh
def should_continue(state: AgentState) -> str:
    last = state["messages"][-1]
    if hasattr(last, "tool_calls") and last.tool_calls:   # LLM muốn gọi tool?
        return "tools"
    if state["current_iteration"] >= state["max_iterations"]:
        return "end"
    return "continue"

workflow.add_conditional_edges("agent", should_continue, {
    "tools": "tool_node",
    "continue": "agent",
    "end": END,
})
```

```
★ Insight ─────────────────────────────────────
• `add_conditional_edges(source, router, mapping)`: router là một HÀM THƯỜNG
  (không phải LLM) đọc state và trả về một KHOÁ; mapping ánh xạ khoá → node đích.
  Đây là "bộ não điều hướng" — quyết định khi nào lặp, khi nào dừng.
• Mẫu kinh điển của agent: agent → (có tool_calls?) → tools → quay lại agent →
  (hết tool_calls) → END. Vòng lặp này CHÍNH LÀ ReAct (xem [[02-Agentic-Patterns]]).
─────────────────────────────────────────────────
```

---

## 5. Dựng graph đầu tiên + chạy

```python
workflow = StateGraph(ChatState)
workflow.add_node("chatbot", chatbot_node)   # thêm node
workflow.set_entry_point("chatbot")           # node bắt đầu
workflow.add_edge("chatbot", END)             # node kết thúc

checkpointer = MemorySaver()                   # bộ nhớ trạng thái (tuỳ chọn)
app = workflow.compile(checkpointer=checkpointer)   # COMPILE → app chạy được
```

**Chạy với `thread_id`** (định danh hội thoại — nhờ checkpointer mà lần gọi sau nhớ lần trước):

```python
config = {"configurable": {"thread_id": "alice-chat"}}

app.invoke({"messages": [HumanMessage("LangGraph là gì?")], "user_name": "Alice"},
           config=config)
# Lần 2: cùng thread_id → vẫn nhớ ngữ cảnh "Alice" + câu hỏi trước
app.invoke({"messages": [HumanMessage("Vì sao tốt hơn chain?")]}, config=config)
```

### `invoke` vs `stream`

| | `app.invoke(...)` | `app.stream(...)` |
|---|---|---|
| Trả về | **Kết quả cuối cùng** (state sau khi chạy xong) | **Từng bước** (event mỗi node) |
| Dùng khi | Cần kết quả gọn | Hiển thị tiến trình real-time, debug |

---

## 6. Tool Calling Pattern (vòng lặp agent hoàn chỉnh)

```python
@tool
def search_web(query: str) -> str:
    """Tìm thông tin trên web"""
    return f"Kết quả cho: {query}"

llm_with_tools = llm.bind_tools([search_web])   # gắn tool vào LLM

def agent_node(state):  return {"messages": [llm_with_tools.invoke(state["messages"])]}

def tool_node(state):
    msgs = []
    for tc in state["messages"][-1].tool_calls:
        result = search_web.invoke(tc["args"])
        msgs.append(ToolMessage(content=result, tool_call_id=tc["id"]))
    return {"messages": msgs}

workflow = StateGraph(AgentState)
workflow.add_node("agent", agent_node)
workflow.add_node("tools", tool_node)
workflow.set_entry_point("agent")
workflow.add_conditional_edges("agent", should_continue, {"tools": "tools", "end": END})
workflow.add_edge("tools", "agent")     # ← VÒNG LẶP: tool xong quay lại agent
app = workflow.compile()
```

> Đây là khung xương của mọi ReAct agent. Chi tiết hoá ở [[02-Agentic-Patterns]] (dùng `ToolNode` prebuilt thay vì tự viết `tool_node`).

---

## 7. Checkpointer (giới thiệu) & Visualize

**Checkpointer** = thành phần **lưu & khôi phục state** giữa các lần chạy. Không có nó, mỗi lần `invoke` là **stateless** (quên sạch). `MemorySaver` là bản đơn giản nhất (lưu trong RAM, mất khi restart — chỉ dùng dev/test). Chi tiết SQLite/Postgres ở [[05-Human-in-the-Loop-Persistence]].

> [!note] Vì sao checkpointer "hợp" với messages-first?
> `messages` **append-only & xác định** (deterministic), mỗi node trả về **delta**, nên toàn bộ lịch sử **tái dựng được** → graph trở nên **replayable & debuggable**. Đây là lý do sâu xa buộc mọi I/O hội thoại phải nằm trong `messages`.

**Visualize đồ thị:**

```python
from IPython.display import Image, display
display(Image(app.get_graph().draw_mermaid_png()))   # vẽ sơ đồ Mermaid
```

---

## 8. Best Practices

> [!tip] 1. Luôn dùng `messages` cho I/O
> ✅ `messages: Annotated[List[AnyMessage], add_messages]` rồi `return {"messages": [response]}`.
> ❌ `input: str / output: str` rồi tự xử lý — mất tích hợp LangChain, mất lịch sử, mất khả năng checkpoint.

> [!tip] 2. Tách bạch concern
> messages (I/O hội thoại) | user_id/preferences (context người dùng) | max_iterations/current_step (điều khiển luồng) | sources/scores (theo dõi kết quả).

> [!tip] 3. Gắn metadata cho message
> `AIMessage(content=..., name="research_agent", additional_kwargs={"confidence": 0.95})` — đặc biệt hữu ích để **phân biệt agent** trong multi-agent (xem [[04-Multi-Agent-Collaboration]]).

> [!tip] 4. Quản lý lịch sử dài (TrimMessage)
> Hội thoại dài → vượt context window. Cần cắt bớt (trim) message cũ hoặc tóm tắt. (Liên hệ [[03-LLMOps-Evaluation/02-Observability-LangFuse-LangSmith|cost/token]].)

---

## 9. Pitfalls / Bẫy thường gặp

> [!warning] Quên reducer `add_messages`
> Khai báo `messages: List[AnyMessage]` (không `Annotated[..., add_messages]`) → mỗi node ghi đè list → **mất lịch sử**. Đây là lỗi #1 của người mới.

> [!warning] Node trả về cả list messages cũ
> Node chỉ cần trả **delta** (`[response]`). Nếu trả lại toàn bộ `state["messages"] + [response]`, kết hợp với reducer append → **nhân đôi** message.

> [!warning] Quên checkpointer khi cần nhớ multi-turn
> Không compile với checkpointer → mỗi `invoke` stateless, "Alice" và câu hỏi trước biến mất. Multi-turn chat **bắt buộc** checkpointer + `thread_id`.

> [!warning] Vòng lặp không có điều kiện dừng
> `add_edge("tools", "agent")` mà router không bao giờ trả "end" → **lặp vô hạn**. Luôn có `max_iterations` hoặc điều kiện thoát.

---

## 10. Câu hỏi phỏng vấn thường gặp

**Q1: LangGraph khác LangChain ở đâu?**
> LangChain chain tuyến tính (A→B→C), state ngầm, không vòng lặp/rẽ nhánh. LangGraph là state machine dạng đồ thị: vòng lặp, rẽ nhánh có điều kiện, state tường minh qua messages, đứng trên LangChain. Dùng cho agent phức tạp/multi-turn.

**Q2: "Messages-centric" nghĩa là gì?**
> Mọi I/O hội thoại giữa node đi qua trường `messages`; dữ liệu khác (cấu hình, theo dõi) để ở context fields. Giúp chuẩn hoá I/O, tương thích LangChain, tự tích luỹ lịch sử, và cho phép checkpoint/replay.

**Q3: `add_messages` reducer làm gì? Không có nó thì sao?**
> Là reducer append message mới vào lịch sử (kèm dedup theo id), nên node chỉ trả delta. Không có nó, giá trị trả về ghi đè list → mất toàn bộ lịch sử hội thoại.

**Q4: Node trả về gì?**
> Một dict là **state delta** — chỉ phần thay đổi (vd `{"messages": [response]}`), không phải toàn bộ state. LangGraph dùng reducer để gộp delta vào state hiện tại.

**Q5: Conditional edge hoạt động thế nào?**
> `add_conditional_edges(source, router, mapping)`: router là hàm thường đọc state trả về một khoá; mapping ánh xạ khoá → node đích. Dùng để quyết định lặp tiếp (gọi tool) hay dừng (END).

**Q6: invoke vs stream?**
> invoke trả kết quả cuối (state sau khi chạy xong); stream trả từng event theo bước — dùng để hiển thị tiến trình real-time hoặc debug.

---

## 11. Bài tập tự luyện

- [ ] **Bài 1:** Dựng "Hello World Graph" 1 node, compile, chạy `invoke`, rồi `draw_mermaid_png()` để xem sơ đồ.
- [ ] **Bài 2:** Cố tình bỏ `add_messages` → quan sát lịch sử bị mất; thêm lại → quan sát tích luỹ.
- [ ] **Bài 3:** Thêm checkpointer + `thread_id`, chạy 2 lượt hỏi nối tiếp, kiểm tra agent nhớ lượt trước.
- [ ] **Bài 4:** Dựng vòng agent↔tools với `should_continue` + `max_iterations`; thử bỏ điều kiện dừng để thấy lặp vô hạn.

---

## 12. Liên kết

- [[02-Agentic-Patterns]] — biến vòng agent↔tools thành ReAct + multi-expert
- [[03-Tool-Calling-Tavily]] — chi tiết tool calling, `bind_tools`, Tavily
- [[04-Multi-Agent-Collaboration]] — nhiều node/agent chia sẻ state + dialog stack
- [[05-Human-in-the-Loop-Persistence]] — checkpointer (SQLite/Postgres), interrupt, time-travel
- [[../03-LLMOps-Evaluation/03-Experiment-Comparison]] — hướng agentic giải bài toán analytical
- [[00-MOC-LangGraph-Agentic|MOC: LangGraph & Agentic AI]]
