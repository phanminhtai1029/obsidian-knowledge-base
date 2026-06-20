---
title: "Tool Calling & Tavily Search"
section: 04-AI/04-LangGraph-Agentic
tags: [ai, langgraph, tool-calling, function-calling, tavily, bind-tools, fresher]
related:
  - "[[02-Agentic-Patterns]]"
  - "[[04-Multi-Agent-Collaboration]]"
difficulty: ⭐⭐⭐
estimated_time: 30m
source: [QN26_FR_AI_01, "03-tool-calling-tavily-search.mdx"]
---

# Tool Calling & Tavily Search

> [!summary] TL;DR
> **Tool Calling (= Function Calling)** là khả năng LLM **tự quyết định** khi nào gọi hàm ngoài, gọi với **tham số đúng schema (JSON)**, nhận kết quả, rồi **suy luận tiếp** để trả lời. Nó vá điểm yếu của LLM: knowledge cutoff, không truy cập dữ liệu real-time, không thực hiện hành động. Luồng: *Query → LLM phân tích → quyết gọi tool → thực thi → kết quả → LLM tổng hợp*. Trong LangChain dùng decorator **`@tool`** + **`llm.bind_tools([...])`**. **Tavily** là search engine **tối ưu cho AI/RAG**: trả kết quả đã làm sạch + có cả `answer` tổng hợp, tham số `search_depth`, `include_answer`, `include_domains`... Production: bảo mật API key (env/secret), tool description rõ ràng, caching, rate limit, parallel calls.

---

## 1. Tool Calling là gì?

4 năng lực LLM có được nhờ tool calling:
1. **LLM tự quyết khi nào dùng tool** — dựa trên câu hỏi.
2. **Gọi tool có cấu trúc** — tham số theo **JSON schema** chuẩn.
3. **Parse kết quả** tool trả về.
4. **Suy luận tiếp** với thông tin mới → câu trả lời cuối.

```
User Query → LLM phân tích → quyết dùng tool → gọi tool(params)
→ Tool chạy → trả kết quả → LLM xử lý → câu trả lời cuối
```

**Vì sao cần?** Vượt **knowledge cutoff**; truy cập **dữ liệu real-time** (thời tiết, giá cổ phiếu, tin); **thực hiện hành động** (gửi email, tạo ticket, update DB); **tích hợp API** bên thứ ba.

### Function Calling vs Tool Use

| **Function Calling** | **Tool Use** |
|---|---|
| Thuật ngữ OpenAI | Thuật ngữ LangChain/Anthropic |
| JSON schema cho function | Tool interface kèm description |
| Trả về function call object | Trả về tool invocation |
| Gắn với model OpenAI | **Framework-agnostic** |

```
★ Insight ─────────────────────────────────────
• ĐIỂM HAY: LLM KHÔNG tự chạy tool. Nó chỉ SINH RA "ý định gọi tool" (tên + args
  dạng JSON) trong AIMessage.tool_calls. Code của BẠN (hoặc ToolNode) mới thực
  sự thực thi. Đây là ranh giới an toàn: LLM đề xuất, hệ thống định đoạt — nền
  tảng cho human-in-the-loop ([[05-Human-in-the-Loop-Persistence]]).
• "LLM tự quyết" thực chất nhờ TOOL DESCRIPTION: model đọc mô tả + schema để
  match câu hỏi với tool. Vì vậy description rõ ràng = chất lượng tool calling.
─────────────────────────────────────────────────
```

---

## 2. OpenAI Function Calling (cách "thô")

Định nghĩa function bằng JSON schema rồi gọi với `function_call="auto"`:

```python
functions = [{
    "name": "search_web",
    "description": "Tìm thông tin hiện tại trên web về một chủ đề",
    "parameters": {
        "type": "object",
        "properties": {
            "query": {"type": "string", "description": "Câu truy vấn"},
            "num_results": {"type": "integer", "default": 5}
        },
        "required": ["query"]
    }
}]

response = openai.ChatCompletion.create(model="gpt-4", messages=[...],
                                        functions=functions, function_call="auto")
# response.choices[0].message.function_call → {"name": ..., "arguments": '{"query": "..."}'}
```

Sau đó tự parse, thực thi, rồi **gửi kết quả về model** (role `function`) để nó trả lời cuối. → Nhiều bước thủ công. LangChain gói gọn lại.

---

## 3. LangChain Tools (cách hiện đại)

### Decorator `@tool` (phổ biến nhất)

```python
from langchain.tools import tool

@tool
def calculator(expression: str) -> str:
    """Tính toán biểu thức toán học. Input là biểu thức Python hợp lệ."""
    try:
        return f"Result: {eval(expression)}"
    except Exception as e:
        return f"Error: {e}"
```

> **Docstring chính là tool description** mà LLM đọc để quyết gọi — viết rõ ràng, nêu *khi nào dùng* và *input/output*.

### Tool interface & built-in tools

```python
from langchain.tools import Tool
search_tool = Tool(name="WebSearch", func=search_function,
                   description="Tìm web cho thông tin hiện tại. Input: chuỗi truy vấn.")

# Built-in: DuckDuckGoSearchResults, WikipediaQueryRun, PythonREPLTool,
#           ReadFileTool / WriteFileTool ...
```

### Gắn tool vào LLM

```python
llm_with_tools = llm.bind_tools([calculator, search_tool])
# Giờ LLM có thể sinh tool_calls trỏ tới các tool này
```

---

## 4. ⭐ Tavily Search API

**Tavily** = search engine **tối ưu cho AI**: kết quả đã **format sẵn cho LLM**, thiết kế cho **RAG**, lọc nhiễu, real-time.

```python
from tavily import TavilyClient
client = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))

response = client.search(
    query="climate change solutions",   # bắt buộc
    search_depth="advanced",            # "basic" | "advanced"
    max_results=10,
    include_domains=["edu", "gov"],     # lọc domain
    exclude_domains=["example.com"],
    include_answer=True,                # ⭐ trả thẳng câu trả lời tổng hợp
    include_raw_content=False,
    include_images=True,
)
```

**Cấu trúc response** (điểm khác search thường — có sẵn `answer` + `score`):

```json
{
  "query": "...",
  "answer": "Several effective climate change solutions include...",
  "results": [
    {"title": "...", "url": "...", "content": "Bản tóm tắt sạch...", "score": 0.98}
  ],
  "response_time": 1.23
}
```

### Tích hợp LangChain

```python
from langchain_community.tools.tavily_search import TavilySearchResults
search = TavilySearchResults(max_results=5, search_depth="advanced",
                             include_answer=True)
result = search.invoke("latest AI developments")
```

```
★ Insight ─────────────────────────────────────
• Vì sao Tavily hợp RAG hơn Google API thô: nó trả `content` ĐÃ LÀM SẠCH (bỏ
  HTML/quảng cáo/menu) + trường `answer` tổng hợp + `score` độ liên quan. LLM ăn
  thẳng được, đỡ phải scrape & clean. `include_answer=True` ≈ một bước generation
  miễn phí ngay trong search.
• `search_depth="advanced"` tốn hơn nhưng kết quả sâu/liên quan hơn — đánh đổi
  chất lượng↔chi phí giống ef_search ở HNSW ([[../02-RAG-Optimization/01-Advanced-Indexing]]).
─────────────────────────────────────────────────
```

---

## 5. Advanced patterns

### Tool Chaining (output tool này → input tool kia)

```python
# company name → search_company → trích ticker → get_stock_price
```

### Parallel calls
Một `AIMessage` chứa nhiều `tool_calls` → `ToolNode` thực thi đồng thời. (Bài tập của môn: thêm Tavily **gọi song song** cùng các expert tool ở [[02-Agentic-Patterns]].)

### Error handling, caching, rate limiting

```python
from tenacity import retry, stop_after_attempt, wait_exponential
@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=2, max=10))
def call_tavily_with_retry(query): return client.search(query)

from functools import lru_cache
@lru_cache(maxsize=100)
def cached_search(query): return client.search(query)   # hoặc cache có TTL

from ratelimit import limits, sleep_and_retry
@sleep_and_retry
@limits(calls=10, period=60)        # tối đa 10 call/phút
def rate_limited_search(query): return client.search(query)
```

### Custom tool (BaseTool + Pydantic schema)

```python
from langchain.tools import BaseTool
from pydantic import BaseModel, Field

class SearchInput(BaseModel):
    query: str = Field(description="Câu truy vấn")
    domain: str = Field(description="Domain cụ thể")

class DomainSearchTool(BaseTool):
    name = "domain_search"
    description = "Tìm trong một domain cụ thể"
    args_schema = SearchInput
    def _run(self, query, domain):  return client.search(f"site:{domain} {query}")
    async def _arun(self, query, domain): return await client.search_async(...)
```

---

## 6. Security & Best Practices

> [!warning] KHÔNG hardcode API key
> ```python
> client = TavilyClient(api_key="tvly-xxxxx")   # ❌
> client = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))   # ✅ env var
> # ✅ tốt hơn: secret manager (Azure Key Vault...)
> ```
> (Liên hệ [[../03-LLMOps-Evaluation/02-Observability-LangFuse-LangSmith#5. Production Best Practices|PII/secret ở observability]].)

> [!tip] Tool description rõ ràng = tool calling chính xác
> ```python
> @tool
> def search(query: str) -> str:
>     """Tìm web cho thông tin hiện tại bằng Tavily.
>     Dùng cho: tin mới, sự kiện hiện tại, dữ kiện thực tế.
>     Input: câu truy vấn cụ thể. Output: top 5 kết quả kèm tóm tắt."""
> ```
> Mô tả mơ hồ ("Search the web") → LLM gọi sai lúc hoặc bỏ sót tool.

---

## 7. Pitfalls / Bẫy thường gặp

> [!warning] Tưởng LLM tự chạy tool
> LLM chỉ sinh `tool_calls` (ý định). Code/ToolNode mới thực thi. Quên bước thực thi → tool "không chạy".

> [!warning] Docstring/description sơ sài
> Đây là thứ DUY NHẤT LLM dựa vào để chọn tool. Sơ sài → routing sai. Đầu tư viết mô tả.

> [!warning] Hardcode hoặc log API key
> Repo public hoặc trace observability lộ key. Dùng env/secret manager, mask khi log.

> [!warning] Không cache/rate-limit search đắt
> Mỗi `search_depth="advanced"` tốn tiền + latency. Cache truy vấn lặp, giới hạn tần suất.

---

## 8. Câu hỏi phỏng vấn thường gặp

**Q1: Tool/Function Calling hoạt động thế nào?**
> LLM phân tích câu hỏi, quyết định gọi tool nào với tham số JSON (sinh ra trong AIMessage.tool_calls); hệ thống thực thi tool, trả kết quả (ToolMessage); LLM suy luận tiếp để trả lời. LLM chỉ đề xuất, không tự chạy tool.

**Q2: Function Calling khác Tool Use?**
> Cùng cơ chế, khác thuật ngữ: Function Calling (OpenAI, JSON schema cho function) vs Tool Use (LangChain/Anthropic, tool interface kèm description, framework-agnostic).

**Q3: Vì sao Tavily hợp với agent/RAG hơn search thường?**
> Trả kết quả đã làm sạch cho LLM, có trường `answer` tổng hợp và `score` liên quan, real-time, lọc nhiễu. LLM ăn trực tiếp, đỡ scrape/clean.

**Q4: `bind_tools` làm gì?**
> Gắn danh sách tool vào LLM để model biết các tool khả dụng (tên + schema + description) và có thể sinh tool_calls trỏ tới chúng.

**Q5: Vì sao tool description quan trọng?**
> Đó là thông tin LLM dùng để quyết định gọi tool nào và khi nào. Mô tả rõ (khi nào dùng, input/output) → tool calling chính xác; mô tả mơ hồ → gọi sai/bỏ sót.

**Q6: Bảo mật API key cho tool thế nào?**
> Không hardcode; dùng biến môi trường (`os.getenv`) hoặc secret manager (Key Vault); không log key lên observability; mask dữ liệu nhạy cảm.

---

## 9. Bài tập tự luyện

- [ ] **Bài 1:** Viết 2 tool bằng `@tool` (calculator + get_current_time), `bind_tools`, hỏi câu cần tính toán → quan sát `tool_calls`.
- [ ] **Bài 2:** Đăng ký Tavily, gọi `client.search` với `include_answer=True`; so sánh `answer` vs `results`.
- [ ] **Bài 3:** Thêm `TavilySearchResults` vào Research Agent (môn 02) cho IT/web request, cho phép **gọi song song** với expert tools.
- [ ] **Bài 4:** Bọc search bằng cache (TTL 1h) + rate limit 10/phút; đo số call tiết kiệm khi lặp truy vấn.

---

## 10. Liên kết

- [[02-Agentic-Patterns]] — tool calling là "Act" trong ReAct; thêm web search vào multi-expert
- [[01-LangGraph-Foundations-State]] — `bind_tools`, `ToolMessage`, vòng agent↔tools
- [[04-Multi-Agent-Collaboration]] — phân phối tool theo từng agent chuyên biệt
- [[05-Human-in-the-Loop-Persistence]] — chèn xác nhận trước khi thực thi tool nhạy cảm
- [[00-MOC-LangGraph-Agentic|MOC: LangGraph & Agentic AI]]
