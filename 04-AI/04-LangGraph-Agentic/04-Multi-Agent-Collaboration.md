---
title: "Multi-Agent Collaboration"
section: 04-AI/04-LangGraph-Agentic
tags: [ai, langgraph, multi-agent, supervisor, dialog-stack, context-injection, complete-or-escalate, fresher]
related:
  - "[[02-Agentic-Patterns]]"
  - "[[05-Human-in-the-Loop-Persistence]]"
difficulty: ⭐⭐⭐⭐⭐
estimated_time: 45m
source: [QN26_FR_AI_01, "04-multi-agent-collaboration.mdx"]
---

# Multi-Agent Collaboration

> [!summary] TL;DR
> Khi một agent "ôm" quá nhiều việc, ta chia thành **nhiều agent chuyên biệt** phối hợp — lợi ích: **chuyên môn hoá**, **xử lý song song**, **modular** (dễ bảo trì), **mở rộng** (thêm agent không ảnh hưởng cái cũ). 4 mẫu phối hợp: **Sequential** (pipeline), **Hierarchical/Supervisor** (Primary Assistant định tuyến tới agent con — phổ biến nhất), **Network** (peer-to-peer), **Competitive**. Trọng tâm môn: **Hierarchical** với **shared state** + **dialog stack** (push khi vào agent con, pop khi quay lại), **context injection** (tự bơm `user_id`/`email` vào tool call), và **CompleteOrEscalate** (agent con tự quyết xong việc hay trả quyền về Primary). Tool chia **safe** (read-only) vs **sensitive** (ghi) để kiểm soát + fallback chống lỗi.

---

## 1. Vì sao cần nhiều agent?

| Lợi ích | Giải thích |
|---|---|
| **Specialization** | Mỗi agent giỏi một lĩnh vực → chính xác hơn |
| **Parallel processing** | Nhiều agent xử lý task độc lập song song → nhanh hơn |
| **Modularity** | Mỗi agent là module độc lập → dễ bảo trì/mở rộng |
| **Scalability** | Thêm agent mới không ảnh hưởng agent cũ |

**Ví dụ:** đội phát triển phần mềm (PM → Architect → Developer → Tester), chăm sóc khách hàng (FAQ / Ticket / IT / Booking agent), pipeline nội dung (Research → Writer → Editor).

```
★ Insight ─────────────────────────────────────
• Đây là sự tiến hoá trực tiếp từ quy tắc "1 node ≤ 2 việc" ở
  [[02-Agentic-Patterns]]: khi tách đủ vai trò, single-agent ReAct trở thành
  hệ multi-agent. Tư duy giống microservices — mỗi agent một bounded context.
─────────────────────────────────────────────────
```

---

## 2. ⭐ 4 mẫu phối hợp

| Mẫu | Cấu trúc | Đặc điểm |
|---|---|---|
| **1. Sequential (Pipeline)** | A → B → C | Output agent trước = input agent sau; tuyến tính |
| **2. Hierarchical (Supervisor)** | Primary → {các agent con} | Supervisor điều phối & định tuyến — **trọng tâm môn** |
| **3. Network (Peer-to-Peer)** | Agent ↔ Agent | Giao tiếp trực tiếp, không cần supervisor |
| **4. Competitive** | Nhiều agent cùng giải | Chọn lời giải tốt nhất |

**Ví dụ Hierarchical (FPT Chatbot):**
```
Primary Assistant (Supervisor)
    ├── FAQ Agent (RAG)
    ├── Ticket Support Agent
    ├── IT Support Agent (Tavily)
    └── Booking Agent
```

---

## 3. ⭐ Hierarchical System — shared state + Dialog Stack

Mọi agent **chia sẻ chung một state**, nhưng cần biết "đang ở agent nào" → **dialog stack**.

```python
class AgenticState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]           # chia sẻ chung
    dialog_state: Annotated[                                       # ngăn xếp agent
        list[Literal["primary_assistant","ticket_agent","it_agent","booking_agent"]],
        update_dialog_stack,
    ]
    conversation_id: Annotated[str, "Conversation ID"]
    user_id: Annotated[str, "User ID"]
    email: Annotated[Optional[EmailStr], "Email (optional)"]
```

### Dialog Stack — push / pop

```python
def update_dialog_stack(left: list[str], right: Optional[str]) -> list[str]:
    if right is None:    return left
    if right == "pop":   return left[:-1]    # POP: quay về agent trước
    return left + [right]                     # PUSH: vào agent mới
```

```python
dialog_state = ["primary_assistant"]                       # bắt đầu
dialog_state = ["primary_assistant", "ticket_agent"]       # PUSH vào Ticket
dialog_state = ["primary_assistant"]                       # POP về Primary
```

```
★ Insight ─────────────────────────────────────
• Dialog stack chính là một REDUCER khác (giống add_messages nhưng cho stack):
  `update_dialog_stack` quyết định gộp delta thế nào. Trả "pop" → bỏ phần tử
  cuối; trả tên agent → đẩy vào. Đây là cách LangGraph "nhớ" mình đang lồng sâu
  bao nhiêu cấp agent — y như call stack của hàm.
• Vì sao cần stack chứ không chỉ một biến current_agent? Vì có thể lồng nhiều
  cấp và cần quay lại đúng cấp cha khi agent con xong. Stack đảm bảo đúng thứ tự.
─────────────────────────────────────────────────
```

---

## 4. Context Injection — tự bơm thông tin người dùng

**Vấn đề:** agent con cần `user_id`/`email` cho mọi tool call, nhưng bắt agent nhớ & truyền mỗi lần thì rườm rà và dễ sai.

**Giải pháp:** lưu context trong state, **tự động inject** vào tool call:

```python
def inject_user_info(state: AgenticState, result):
    """Tự bơm user_id & email vào các tool call cần context"""
    if hasattr(result, "tool_calls") and result.tool_calls:
        for tc in result.tool_calls:
            if tc["name"] in ["ToTicketAssistant","ToITAssistant","ToBookingAssistant"]:
                tc["args"]["user_id"] = state["user_id"]
                if state.get("email"):
                    tc["args"]["email"] = state["email"]
    return result
```

Tool khai báo các trường này **Optional** (có thể inject từ context, hoặc user override):

```python
class CreateTicket(BaseModel):
    content: str = Field(..., description="Nội dung ticket")
    description: Optional[str] = None
    email: Optional[EmailStr] = Field(None, description="Có thể inject từ context")
    user_id: Optional[str] = Field(None, description="Tự inject")
```

> [!tip] Email OPTIONAL xuyên suốt
> Theo yêu cầu FPT Chatbot: email hoàn toàn tuỳ chọn. Có context → tự inject; user nhập trực tiếp → override; không có → tool **vẫn chạy bình thường**, chỉ không gửi email thông báo.

---

## 5. ⭐ State Transitions: Entry Node, Routing, CompleteOrEscalate

### Entry Node — cửa vào agent con (push stack)

```python
def create_entry_node(assistant_name: str, new_dialog_state: str):
    def entry_node(state: AgenticState) -> dict:
        tool_call_id = state["messages"][-1].tool_calls[0]["id"]
        return {
            "messages": [ToolMessage(
                content=f"Giờ bạn là {assistant_name}. Phản ánh hội thoại trên, "
                        f"dùng tool hỗ trợ user. Xong thì gọi CompleteOrEscalate "
                        f"để trả quyền về.",
                tool_call_id=tool_call_id)],
            "dialog_state": new_dialog_state,   # PUSH
        }
    return entry_node
```

### Routing từ Primary Assistant

```python
def route_primary_assistant(state: AgenticState):
    last = state["messages"][-1]
    if not getattr(last, "tool_calls", None):  return END
    routing_map = {
        "ToTicketAssistant": "enter_ticket_node",
        "ToITAssistant":     "enter_it_node",
        "ToBookingAssistant":"enter_booking_node",
        "rag_agent":         "rag_agent_node",
    }
    return routing_map.get(last.tool_calls[0]["name"], END)
```

### ⭐ CompleteOrEscalate — agent con tự quyết: xong hay trả quyền

```python
class CompleteOrEscalate(BaseModel):
    """Cho agent quyết định kết thúc / trả quyền về Primary"""
    cancel: bool = True   # True = escalate (trả quyền), False = cần thêm info
    reason: str
```

| Khi nào **END** (xong hẳn) | Khi nào **Escalate** (về Primary) |
|---|---|
| Task hoàn thành | Task ngoài phạm vi agent |
| Trả lời đầy đủ | Cần năng lực chỉ Primary có |
| Không cần thêm gì từ user | User đổi ý / huỷ |
| | Cần clarify nhưng agent không tự xử được |

```python
def route_ticket_agent(state: AgenticState):
    route = tools_condition(state)
    if route == END: return END
    tool_calls = state["messages"][-1].tool_calls
    names = [tc["name"] for tc in tool_calls]

    if "CompleteOrEscalate" in names:                  # agent muốn trả quyền
        for tc in tool_calls:
            if tc["name"] == "CompleteOrEscalate" and tc["args"].get("cancel", True):
                return "leave_skill"                    # → pop_dialog_state

    safe = [t.name for t in ticket_safe_tools]
    if all(tc["name"] in safe for tc in tool_calls):
        return "ticket_safe_tools"                      # read-only
    return "ticket_sensitive_tools"                     # ghi (create/update/cancel)
```

```
★ Insight ─────────────────────────────────────
• CompleteOrEscalate là một TOOL ĐẶC BIỆT (không làm gì ngoài đời) — nó chỉ là
  "tín hiệu có cấu trúc" để agent con nói "tôi xong rồi" hoặc "vượt tầm tôi".
  Đẩy quyết định kết thúc vào TOOL CALL (thay vì parse text) khiến luồng tường
  minh & đáng tin. `pop_dialog_state` chèn ToolMessage "resuming with host" rồi
  trả "pop".
• safe vs sensitive tools: tách read-only (track) khỏi ghi (create/update/cancel)
  để có thể chèn HUMAN-IN-THE-LOOP chỉ cho tool nguy hiểm
  ([[05-Human-in-the-Loop-Persistence]]) — không bắt user xác nhận mọi thao tác.
─────────────────────────────────────────────────
```

### Pop về Primary

```python
def pop_dialog_state(state) -> dict:
    messages = []
    if state["messages"][-1].tool_calls:
        messages.append(ToolMessage(
            content="Resuming dialog with the host assistant...",
            tool_call_id=state["messages"][-1].tool_calls[0]["id"]))
    return {"dialog_state": "pop", "messages": messages}
```

---

## 6. Tool Call Fallback — chống lỗi gọi tool

Khi tool lỗi (network/DB/validation), cần xử lý mượt thay vì sập cả graph:

```python
def create_tool_node_with_fallback(tools: list):
    def log_tool_result(state):
        try:
            return ToolNode(tools).invoke(state)
        except Exception as e:
            tcid = state["messages"][-1].tool_calls[0]["id"]
            return {"messages": [ToolMessage(
                content=f"Lỗi thực thi tool: {e}. Vui lòng thử lại hoặc liên hệ hỗ trợ.",
                tool_call_id=tcid)]}
    return RunnableLambda(log_tool_result).with_fallbacks([...], exception_key="error")
```

---

## 7. ⭐ So sánh: ReAct vs Hierarchical vs Supervisor

| Tiêu chí | **ReAct Agent** | **Hierarchical Multi-Agent** | **Supervisor Pattern** |
|---|---|---|---|
| Kiến trúc | 1 agent + vòng lặp | Nhiều agent chuyên biệt, phân cấp | 1 supervisor định tuyến tới workers |
| Số agent | 1 | 3-10+ | 1 supervisor + N |
| Độ phức tạp | Thấp | **Cao** | Trung bình |
| State | Đơn giản | Shared state + **dialog stack** | Tập trung |
| Tool access | Tất cả ở 1 agent | Phân theo agent | Nhóm theo specialist |
| Mở rộng | Hạn chế (1 agent) | Tốt (dễ thêm agent) | Tốt |
| Latency / Cost | Thấp | Trung bình-Cao / Cao | Trung bình |
| Hợp nhất với | FAQ, automation đơn giản | Hệ enterprise, workflow nhiều bước | CSKH đa lĩnh vực, triage |

---

## 8. Pitfalls / Bẫy thường gặp

> [!warning] Quên pop dialog stack
> Vào agent con (push) mà không bao giờ pop → kẹt mãi trong agent con, Primary không lấy lại quyền. Luôn có `CompleteOrEscalate` → `leave_skill`.

> [!warning] Không tách safe vs sensitive tools
> Gộp hết → hoặc bắt user xác nhận cả thao tác read-only (phiền), hoặc để tool ghi chạy không kiểm soát (nguy hiểm). Tách ra để HITL đúng chỗ.

> [!warning] Bắt agent tự nhớ user_id/email
> Truyền thủ công mỗi tool call → dễ quên/sai. Dùng context injection tự bơm từ state.

> [!warning] Tool lỗi làm sập cả graph
> Không có fallback → một lỗi DB/network giết cả hội thoại. Bọc `create_tool_node_with_fallback`.

> [!warning] Lạm dụng multi-agent cho task đơn giản
> Hệ phân cấp đắt & phức tạp (cost cao, latency cao). Task đơn giản/1 lĩnh vực → ReAct single-agent là đủ.

---

## 9. Câu hỏi phỏng vấn thường gặp

**Q1: Vì sao dùng nhiều agent thay vì một?**
> Chuyên môn hoá (mỗi agent giỏi 1 lĩnh vực, chính xác hơn), xử lý song song, modular (dễ bảo trì), dễ mở rộng (thêm agent không ảnh hưởng cái cũ). Đánh đổi: phức tạp, cost & latency cao hơn.

**Q2: 4 mẫu phối hợp multi-agent?**
> Sequential (pipeline A→B→C), Hierarchical/Supervisor (Primary định tuyến tới agent con), Network (peer-to-peer), Competitive (nhiều agent cùng giải, chọn tốt nhất).

**Q3: Dialog stack là gì, push/pop khi nào?**
> Là ngăn xếp theo dõi agent đang active. Push khi Primary chuyển vào agent con (entry node), pop khi agent con gọi CompleteOrEscalate trả quyền về. Giống call stack của hàm, dùng reducer `update_dialog_stack`.

**Q4: Context injection giải quyết gì?**
> Tự bơm user_id/email từ state vào các tool call thay vì bắt agent nhớ và truyền thủ công. Tool khai báo các trường này Optional; có context thì inject, user nhập thì override, không có vẫn chạy.

**Q5: CompleteOrEscalate là gì?**
> Một tool đặc biệt để agent con tín hiệu "xong việc" hoặc "vượt tầm/đổi ý" → trả quyền (escalate) về Primary. Đẩy quyết định kết thúc vào tool call có cấu trúc thay vì parse text, giúp luồng tường minh.

**Q6: Safe tools vs sensitive tools?**
> Safe = read-only (track/xem); sensitive = ghi (create/update/cancel). Tách ra để chèn human-in-the-loop xác nhận chỉ cho thao tác nguy hiểm, không làm phiền với thao tác đọc.

**Q7: Khi nào ReAct, khi nào Hierarchical?**
> ReAct cho task đơn giản, 1 lĩnh vực, ưu tiên latency/cost thấp. Hierarchical cho hệ enterprise, workflow nhiều bước, nhiều lĩnh vực — chấp nhận phức tạp & chi phí cao đổi lấy chuyên môn hoá & mở rộng.

---

## 10. Bài tập tự luyện (FPT Customer Chatbot)

- [ ] **Bài 1:** Dựng hierarchical: Primary + Ticket/IT/Booking agent + FAQ (RAG). Định nghĩa `AgenticState` với dialog stack.
- [ ] **Bài 2:** Cài context injection email/user_id; kiểm tra tool `create_ticket` chạy được **cả khi không có email**.
- [ ] **Bài 3:** Cài `CompleteOrEscalate` + `pop_dialog_state`; test luồng: hỏi FAQ → tạo ticket → quay lại Primary → đặt phòng.
- [ ] **Bài 4:** Tách safe (`track_ticket`) vs sensitive (`create/update/cancel`) tools; thêm `create_tool_node_with_fallback`.
- [ ] **Bài 5 (nâng cao):** Thêm HITL xác nhận trước các tool sensitive ([[05-Human-in-the-Loop-Persistence]]).

> Tham khảo bắt buộc: LangGraph customer-support tutorial — https://langchain-ai.github.io/langgraph/tutorials/customer-support/customer-support/

---

## 11. Liên kết

- [[02-Agentic-Patterns]] — "1 node ≤ 2 việc" → tách thành multi-agent
- [[03-Tool-Calling-Tavily]] — IT Agent dùng Tavily; phân phối tool theo agent
- [[05-Human-in-the-Loop-Persistence]] — HITL cho sensitive tools; checkpointer cho shared state
- [[01-LangGraph-Foundations-State]] — shared state, reducer, conditional edges
- [[../03-LLMOps-Evaluation/03-Experiment-Comparison]] — Hybrid RAG dùng routing tương tự supervisor
- [[00-MOC-LangGraph-Agentic|MOC: LangGraph & Agentic AI]]
