---
title: "Agentic Patterns: ReAct, Multi-Expert, Reflection, Planning"
section: 04-AI/04-LangGraph-Agentic
tags: [ai, langgraph, agentic, react, multi-expert, reflection, planning, toolnode, fresher]
related:
  - "[[01-LangGraph-Foundations-State]]"
  - "[[03-Tool-Calling-Tavily]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 35m
source: [QN26_FR_AI_01, "02-agentic-patterns-reflection-planning.mdx"]
---

# Agentic Patterns: Multi-Expert Research Agent với ReAct

> [!summary] TL;DR
> Research agent ngây thơ (User → LLM → Web Search → Answer) bị giới hạn: 1 tool, không suy luận chất lượng, không kế hoạch, không tự sửa. **3 cải tiến cốt lõi:** (1) **Multi-Expert Tools** — thay web search bằng nhiều **LLM chuyên gia** (mỗi tool là 1 lần `llm.invoke` với system prompt chuyên ngành); (2) **ReAct** (Reason → Act → Observe lặp lại) — Coordinator LLM **suy luận trước khi hành động**, quan sát kết quả, lặp đến khi đủ; (3) **Advanced** — Reflection (tự đánh giá & sửa), Planning, kiểm soát iteration. Dùng **`ToolNode` prebuilt** thay vì tự viết tool execution. Nguyên tắc thiết kế then chốt: **1 node/agent không nên làm quá 2 việc** → tách Planning Agent & Synthesizer Agent ra khỏi Coordinator.

---

## 1. Từ Simple Agent đến Agentic Workflow

```
Simple:   Question → Search → Answer
ReAct:    Question → [Think → Act → Observe]* → Answer
Advanced: Question → Plan → [Execute → Reflect]* → Synthesize
```

Agent đơn giản gặp 4 vấn đề: **một tool duy nhất** (web search) → hẹp chuyên môn; **không suy luận** về chất lượng; **không lập kế hoạch** cho câu hỏi phức tạp; **không reflection** để tự cải thiện.

```
★ Insight ─────────────────────────────────────
• "Agentic" = LLM tự QUYẾT ĐỊNH chuỗi hành động thay vì chạy theo kịch bản cứng.
  Khác biệt với chain/workflow tĩnh: ở agent, SỐ BƯỚC và THỨ TỰ không biết
  trước — LLM quyết runtime dựa trên quan sát. Đó là lý do cần vòng lặp + state
  của LangGraph ([[01-LangGraph-Foundations-State]]).
─────────────────────────────────────────────────
```

---

## 2. Pattern 1 — Multi-Expert Tool Use

### Ý tưởng: LLM chuyên gia làm "tool"

Thay vì 1 web search → nhiều **LLM có kiến thức chuyên ngành**: AI Research Expert, Financial Analyst, Medical Expert, Legal Expert... Mỗi expert là **1 lần `llm.invoke`** với system prompt riêng:

```python
@tool
def ai_research_expert(query: str) -> str:
    """Chuyên gia ML/AI: papers, kiến trúc, xu hướng"""
    messages = [SystemMessage(content=AI_RESEARCH_PROMPT),
                HumanMessage(content=query)]
    return ai_research_llm.invoke(messages).content

@tool
def financial_analyst(query: str) -> str:
    """Chuyên gia tài chính: cổ phiếu, thị trường, định giá"""
    messages = [SystemMessage(content=FINANCIAL_PROMPT),
                HumanMessage(content=query)]
    return financial_llm.invoke(messages).content
```

### Vì sao LLM tool > Web search?

| | LLM Expert Tool | Web Search |
|---|---|---|
| Chất lượng | **Nhất quán**, có suy luận cấu trúc | Tuỳ kết quả search (có thể rác) |
| Loại câu hỏi mạnh | **Phân tích, tổng hợp** | Sự kiện, tin mới |
| Điểm yếu | **Knowledge cutoff** (không biết tin mới) | Nhiễu, chất lượng không đều |

> [!warning] Knowledge cutoff vẫn cần web search
> Expert là LLM nên **không biết sự kiện sau ngày train**. Câu hỏi về tin tức hiện tại vẫn cần Tavily/web search ([[03-Tool-Calling-Tavily]]). Mạnh nhất là **kết hợp** expert (phân tích) + web (dữ kiện mới).

---

## 3. ⭐ Pattern 2 — ReAct (Reason + Act + Observe)

ReAct = vòng lặp **Think → Act → Observe**, lặp đến khi đủ thông tin hoặc chạm `max_iterations`.

| Bước | Bản chất | Thể hiện trong messages |
|---|---|---|
| **Think** (Reason) | Coordinator LLM suy luận: "câu này cần chuyên môn AI" | nội dung `AIMessage` |
| **Act** | Gọi tool phù hợp | `AIMessage.tool_calls=[...]` |
| **Observe** | Nhận & phân tích kết quả expert | `ToolMessage(content=..., name=...)` |
| **Repeat** | Lặp hoặc trả lời cuối | quay lại Think, hoặc END |

### Coordinator + ToolNode + routing

```python
coordinator_with_tools = coordinator_llm.bind_tools([ai_research_expert, financial_analyst])

def coordinator_node(state: ResearchState) -> dict:
    messages = state["messages"]
    if state["current_iteration"] == 0:                      # chèn system prompt lần đầu
        messages = [SystemMessage(content=COORDINATOR_PROMPT)] + messages
    response = coordinator_with_tools.invoke(messages)        # Think + Act
    return {"messages": [response],
            "current_iteration": state["current_iteration"] + 1}   # đếm vòng lặp

def should_continue(state: ResearchState) -> str:
    last = state["messages"][-1]
    if hasattr(last, "tool_calls") and last.tool_calls:      # còn gọi tool → tiếp
        return "continue"
    if state["current_iteration"] >= state["max_iterations"]: # chạm trần → dừng
        return "end"
    return "end"                                             # có câu trả lời cuối → dừng
```

```python
from langgraph.prebuilt import ToolNode

tool_node = ToolNode([ai_research_expert, financial_analyst])   # ✨ prebuilt
workflow.add_conditional_edges("coordinator", should_continue,
                               {"continue": "tools", "end": END})
workflow.add_edge("tools", "coordinator")     # quay lại Coordinator để Observe
```

> [!tip] Dùng `ToolNode` prebuilt — ĐỪNG tự viết
> `ToolNode(tools)` tự: parse `tool_calls` từ `AIMessage` → thực thi tool → tạo `ToolMessage` (đúng `tool_call_id`) → xử lý lỗi/edge case. Battle-tested, code gọn, đúng chuẩn LangGraph. Tự viết tool execution dễ sai id và bỏ sót lỗi.

```
★ Insight ─────────────────────────────────────
• ReAct ánh xạ TRỰC TIẾP vào vòng agent↔tools ở [[01-LangGraph-Foundations-State]]:
  coordinator_node = "Think+Act", ToolNode = thực thi, edge tools→coordinator =
  "Observe rồi nghĩ tiếp". "Reasoning" của ReAct chính là phần `content` của
  AIMessage trước khi nó quyết gọi tool nào.
• max_iterations là van an toàn BẮT BUỘC: LLM có thể kẹt vòng "gọi tool mãi".
  Thực tế đặt 3-5. Hết vòng → ép END dù chưa "hoàn hảo".
─────────────────────────────────────────────────
```

### Sequential vs Parallel consultation

```python
# Tuần tự: gọi expert AI trước, rồi tài chính (2 vòng ReAct)
# Song song: một AIMessage chứa NHIỀU tool_calls cùng lúc →
tool_calls=[{"name": "ai_research_expert", ...},
            {"name": "financial_analyst", ...}]
# ToolNode thực thi tất cả, trả về tất cả ToolMessage
```

---

## 4. Pattern 3 — Reflection & Planning (Advanced)

**Reflection:** agent **tự đánh giá** chất lượng output của chính mình, refine nếu chưa đạt → bắt lỗi, cải thiện lặp.

**Quy tắc thiết kế cốt lõi:** *một node/agent không nên làm quá 2 việc*. Coordinator ReAct hiện làm **3 việc**: (1) đoán & phân tích câu hỏi, (2) gọi action, (3) review & tổng hợp câu trả lời → giảm chất lượng.

> [!tip] Tách vai trò để tăng chất lượng
> - **Planning Agent** — lo việc "đoán & phân tích câu hỏi", đưa ra **hành động + lý do** cho Coordinator làm theo.
> - **Synthesizer Agent** — lo việc sinh câu trả lời cuối khi Coordinator quyết dừng.
> → Coordinator chỉ còn tập trung điều phối (gọi tool). Mỗi agent ít việc hơn → ít lỗi hơn.

```
★ Insight ─────────────────────────────────────
• Đây là tư duy "separation of concerns" áp vào agent: giống microservices,
  mỗi agent một trách nhiệm hẹp. Planning tách RA TRƯỚC vòng lặp (lập kế hoạch),
  Synthesizer tách RA SAU (tổng hợp). Coordinator ở GIỮA chỉ thực thi kế hoạch.
• Đây cũng là cầu nối sang [[04-Multi-Agent-Collaboration]]: khi tách đủ nhiều
  vai trò, ReAct single-agent tiến hoá thành hệ multi-agent có phân cấp.
─────────────────────────────────────────────────
```

### Dynamic Expert Selection (LLM-based routing)

```python
routing_prompt = """Câu hỏi: {question}
Experts: ai_research_expert (ML/AI), financial_analyst (thị trường).
Expert nào nên trả lời? Trả về JSON list."""
```

---

## 5. Trade-offs — khi nào dùng

| | Multi-Expert ReAct | Simple Agent |
|---|---|---|
| Chất lượng | Cao (chuyên môn + suy luận cấu trúc) | Vừa |
| Loại câu hỏi | Phức tạp, đa lĩnh vực, **phân tích** | Đơn giản, 1 lĩnh vực, **sự kiện** |
| Chi phí | **3-5× LLM calls** (mỗi expert + coordinator) | Thấp |
| Latency | Cao (tuần tự nhiều vòng) | Thấp |
| Token | Lớn (lịch sử đầy đủ mỗi call + nhiều system prompt) | Nhỏ |

**Dùng Multi-Expert ReAct khi:** cần chuyên môn, nhiều góc nhìn, chất lượng > tốc độ/chi phí, phân tích > sự kiện đơn thuần.

---

## 6. Best Practices

> [!tip] 5 nguyên tắc production
> 1. **System prompt rõ vai trò** cho từng expert. 2. **Đặt tên tool mô tả** (`ai_research_expert` chứ không `tool1`). 3. **`max_iterations` 3-5** chống lặp vô hạn. 4. **Dùng `ToolNode` prebuilt**. 5. **State tối giản** — chỉ giữ `messages` + iteration.

> [!tip] Prompt engineering cho Coordinator
> Cho **few-shot examples** (Question → Thought → Action) và **chỉ định format output** (## Summary / ## AI Perspective / ## Financial / ## Recommendation) để câu trả lời cuối nhất quán.

---

## 7. Pitfalls / Bẫy thường gặp

> [!warning] Tự viết tool execution thay vì ToolNode
> Dễ sai `tool_call_id`, bỏ sót xử lý lỗi, không handle parallel tool_calls. Dùng `ToolNode(tools)`.

> [!warning] Coordinator gánh quá nhiều việc
> Một node vừa phân tích vừa gọi tool vừa tổng hợp → chất lượng tụt. Tách Planning/Synthesizer.

> [!warning] Quên knowledge cutoff của expert
> Hỏi tin mới mà chỉ dùng LLM expert → trả lời lỗi thời. Thêm web search cho dữ kiện hiện tại.

> [!warning] Không giới hạn iteration
> LLM kẹt vòng gọi tool → cháy token/tiền. Luôn `max_iterations` + ép END khi chạm trần.

---

## 8. Câu hỏi phỏng vấn thường gặp

**Q1: ReAct là gì?**
> Reason + Act + Observe lặp lại: LLM suy luận (Think) → gọi tool (Act) → quan sát kết quả (Observe) → lặp đến khi đủ thông tin hoặc chạm max_iterations. Trong LangGraph: coordinator_node (Think+Act) ↔ ToolNode (thực thi) với edge quay lại để Observe.

**Q2: Vì sao dùng LLM expert làm tool thay vì web search?**
> Cho câu hỏi phân tích/tổng hợp, LLM expert cho chất lượng nhất quán và suy luận cấu trúc, tránh nhiễu từ kết quả search. Nhưng vẫn cần web search cho tin tức mới vì expert bị knowledge cutoff.

**Q3: Vì sao nên dùng ToolNode prebuilt?**
> Nó tự parse tool_calls, thực thi tool, tạo ToolMessage đúng tool_call_id, xử lý lỗi và parallel calls — battle-tested. Tự viết dễ sai id và bỏ sót edge case.

**Q4: "1 node không quá 2 việc" — nghĩa là gì, giải quyết ra sao?**
> Coordinator ReAct làm 3 việc (phân tích, gọi action, tổng hợp) → giảm chất lượng. Tách Planning Agent (phân tích trước) và Synthesizer Agent (tổng hợp sau), để Coordinator chỉ điều phối.

**Q5: Sequential vs parallel tool consultation?**
> Sequential: gọi từng expert qua nhiều vòng ReAct. Parallel: một AIMessage chứa nhiều tool_calls cùng lúc, ToolNode thực thi tất cả và trả về tất cả ToolMessage — nhanh hơn khi các expert độc lập.

**Q6: Chi phí của Multi-Expert ReAct?**
> 3-5× số lần gọi LLM (mỗi expert + coordinator + nhiều vòng), latency cao do tuần tự, token lớn do gửi full lịch sử mỗi call. Chỉ dùng khi chất lượng > tốc độ/chi phí.

---

## 9. Bài tập tự luyện

- [ ] **Bài 1:** Dựng Multi-Expert ReAct với 2 expert (AI + Financial) + Coordinator + `ToolNode`; in messages từng vòng để thấy Think→Act→Observe.
- [ ] **Bài 2:** Hỏi câu cần cả 2 expert ("So sánh NVIDIA vs AMD cho AI") → quan sát agent lặp qua 2-3 iteration.
- [ ] **Bài 3:** Thêm expert thứ 3 (`legal_expert`) — kiểm tra Coordinator tự định tuyến.
- [ ] **Bài 4:** Tách **Synthesizer Agent** riêng để sinh câu trả lời cuối; so sánh chất lượng với bản Coordinator tự tổng hợp.

---

## 10. Liên kết

- [[01-LangGraph-Foundations-State]] — vòng agent↔tools, conditional edge (nền của ReAct)
- [[03-Tool-Calling-Tavily]] — thêm web search (Tavily) song song với expert tools
- [[04-Multi-Agent-Collaboration]] — tách vai trò → hệ multi-agent phân cấp
- [[05-Human-in-the-Loop-Persistence]] — chèn phê duyệt người vào vòng agent
- [[../03-LLMOps-Evaluation/01-Ragas-Evaluation-Metrics]] — đánh giá chất lượng agent
- [[00-MOC-LangGraph-Agentic|MOC: LangGraph & Agentic AI]]
