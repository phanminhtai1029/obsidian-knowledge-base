---
title: "LangChain Framework & Core Components"
section: 04-AI/01-AI-Fundamentals-RAG
tags: [ai, langchain, langgraph, runnable, lcel, document-loader, embeddings, vectorstore, retriever, output-parser, fresher]
related:
  - "[[03-Modern-RAG-Architecture]]"
  - "[[05-Building-RAG-Agent]]"
  - "[[../04-LangGraph-Agentic/00-MOC-LangGraph-Agentic]]"
difficulty: ⭐⭐⭐
estimated_time: 55m
source: [QN26_FR_AI_01, "04-langchain-framework-and-core-components.mdx", "docs.langchain.com"]
---

# LangChain Framework & Core Components

> [!summary] TL;DR
> **LangChain** nối **data ↔ logic ↔ model** để dựng app LLM/RAG. Component cốt lõi: **Document & Loaders** (`page_content` + `metadata` + `id`), **Embeddings**, **Vector Stores** (`add_documents`, `similarity_search`), **Retrievers**, **Prompt Templates**, **Output Parsers**. Tất cả đều là **Runnable** — interface chuẩn có `invoke/batch/stream` (+ bản async) → ghép bằng toán tử **`|` (LCEL)** thành **chain**: `prompt | model | parser`. ⚠️ Điểm phỏng vấn: **LangChain chain là TUYẾN TÍNH (DAG, một chiều, không vòng lặp)**; muốn vòng lặp/điều kiện/state (agent) → dùng **LangGraph (đồ thị có chu trình)**.

---

## 1. Khái niệm

LangChain đóng vai trò "link" nối **data – logic xử lý – model**. Trong RAG, nó giải quyết sự **đứt gãy giữa nguồn dữ liệu và năng lực LLM**, hỗ trợ toàn bộ vòng đời ứng dụng RAG mà không phải tự nối tay phức tạp.

📚 Docs: https://docs.langchain.com/oss/python/langchain/overview

---

## 2. Core Component 1 — Documents & Document Loaders

Đơn vị cơ bản biểu diễn thông tin là object **`Document`**, gồm 3 thuộc tính:

- `page_content`: chuỗi nội dung text.
- `metadata`: dict thông tin bổ sung (nguồn, số trang, ngày…). → phục vụ **pre-filtering**.
- `id`: định danh document.

### Loading & Splitting

```python
from langchain_community.document_loaders import PyPDFLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter

loader = PyPDFLoader("../AI.pdf")
doc = loader.load()

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,     # kích thước mỗi chunk
    chunk_overlap=200,   # 200 token chồng lấn giữa 2 chunk liền kề
)
all_splits = text_splitter.split_documents(doc)
```

---

## 3. Core Component 2 — Embeddings

Biến text thành vector số thực; nghĩa gần nhau → vector gần nhau (đo bằng cosine similarity).

```python
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings(model="text-embedding-3-large")
vector_1 = embeddings.embed_query(all_splits[0].page_content)
print(len(vector_1))   # 1536 — số chiều vector
```

> [!tip] Đề thi dùng AWS Bedrock
> Học liệu mẫu dùng `OpenAIEmbeddings`, nhưng đề thi dùng **Bedrock** → đổi sang `langchain-aws` (`BedrockEmbeddings`, `ChatBedrock`). Xem [[../../05-Cloud/01-AWS-Bedrock/00-MOC-AWS-Bedrock|Cloud/AWS Bedrock]].

---

## 4. Core Component 3 — Vector Stores

Lưu `Document` + vector và cung cấp truy vấn:
- **Indexing**: `add_documents`.
- **Querying**: `similarity_search`.

Loại: in-memory (`FAISS`, `InMemoryVectorStore`) → DB chuyên dụng (`Chroma`, `Pinecone`, `Postgres`/pgvector).

```python
from langchain_core.vectorstores import InMemoryVectorStore

vector_store = InMemoryVectorStore(embeddings)
vector_store.add_documents(documents=all_splits)
results = vector_store.similarity_search("How many distribution centers does Nike have?")
```

---

## 5. Core Component 4 — Retrievers

VectorStore để **lưu**, Retriever để **truy vấn**. Tạo từ VectorStore qua `.as_retriever()`:

| `search_type` | Ý nghĩa |
|---------------|---------|
| `similarity` | Tương đồng mặc định |
| `mmr` (Maximum Marginal Relevance) | Cân bằng **tương đồng** và **đa dạng** kết quả (tránh chunk gần trùng) |
| `similarity_score_threshold` | Lọc bỏ kết quả dưới ngưỡng score |

```python
retriever = vector_store.as_retriever(search_type="similarity", search_kwargs={"k": 4})
docs = retriever.invoke("When was Nike founded?")
```

---

## 6. Core Component 5 — Prompt Templates & Output Parsers

- **PromptTemplate / ChatPromptTemplate**: khuôn prompt có placeholder `{biến}`, điền lúc chạy.
- **Output Parser**: ép output LLM về định dạng mong muốn (string, JSON, Pydantic object…). Phổ biến: `StrOutputParser`, `JsonOutputParser`, `PydanticOutputParser`.

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_template(
    "Trả lời dựa trên context.\n\nContext:\n{context}\n\nCâu hỏi: {question}"
)
parser = StrOutputParser()
```

---

## 7. ⭐ Runnable & LCEL — ghép chain bằng `|`

Đây là **trái tim của LangChain** và là phần hay bị hỏi.

### Runnable là gì?
**Runnable** là một **interface chuẩn (protocol)** mà *hầu hết mọi component* LangChain đều implement: `PromptTemplate`, `ChatModel`/`LLM`, `OutputParser`, `Retriever`, `Tool`, thậm chí cả function thường (bọc bằng `RunnableLambda`). Vì cùng "hình dạng", chúng **ghép nối được với nhau**.

Mọi Runnable có bộ method thống nhất:

| Method | Công dụng |
|--------|-----------|
| `invoke(input)` | Chạy 1 input → 1 output |
| `batch([inputs])` | Chạy nhiều input **song song** |
| `stream(input)` | **Stream** từng token/chunk khi sinh |
| `ainvoke / abatch / astream` | Bản **async** tương ứng |

### LCEL — LangChain Expression Language
Vì tất cả là Runnable, ta nối chúng bằng toán tử **`|` (pipe)**: output của cái trước thành input của cái sau → tạo thành **chain**. Cú pháp này gọi là **LCEL**.

```python
from langchain_core.runnables import RunnablePassthrough

# RAG chain kinh điển, đọc từ trái sang phải:
chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt          # điền context + question vào khuôn
    | llm             # LLM sinh câu trả lời
    | StrOutputParser()  # ép về string
)

answer = chain.invoke("Nike được thành lập năm nào?")
# stream cũng "miễn phí" nhờ Runnable:
for token in chain.stream("..."):
    print(token, end="")
```

### Runnable "primitives" hay dùng
- **`RunnablePassthrough`** — đẩy input đi thẳng không đổi (vd giữ nguyên câu hỏi gốc).
- **`RunnableParallel`** (dict) — chạy nhiều nhánh **song song**, gộp kết quả thành dict.
- **`RunnableLambda`** — bọc 1 function Python thường thành Runnable.

```
★ Insight ─────────────────────────────────────
• Lợi ích "miễn phí" khi viết bằng LCEL: streaming, async, batching, retry,
  fallback, và khả năng trace/observe — không phải code tay.
• `{"context": retriever, "question": RunnablePassthrough()}` thực chất là một
  RunnableParallel: nhánh context gọi retriever, nhánh question giữ nguyên input.
• Đọc chain như đường ống Unix: dữ liệu chảy trái → phải qua từng "trạm".
─────────────────────────────────────────────────
```

---

## 8. ⭐ LangChain (tuyến tính) vs LangGraph (đồ thị) — RẤT hay hỏi

> Câu phỏng vấn thực tế: *"LangChain có tuyến tính không? Khác LangGraph chỗ nào?"*

| Tiêu chí | **LangChain (LCEL chain)** | **LangGraph** |
|----------|----------------------------|---------------|
| Cấu trúc | **Tuyến tính / DAG** (đồ thị có hướng, **không chu trình**) | **Đồ thị có state**, cho phép **chu trình (cycle)** |
| Luồng dữ liệu | Một chiều A → B → C | Có **vòng lặp** & **nhánh điều kiện** (conditional edges) |
| State | Truyền tuần tự qua từng bước | **State object** chia sẻ toàn graph, cập nhật qua từng node |
| Hợp với | Pipeline **xác định** (RAG chain cố định, prompt→model→parser) | **Agent** (Reason→Act→Observe lặp), **multi-agent**, **human-in-the-loop** |
| Vòng lặp | Không tự nhiên (phải tự hack) | Native (loop tới khi điều kiện dừng) |

**Chốt để trả lời phỏng vấn:**
> "**Có** — LangChain (LCEL) về bản chất là luồng **tuyến tính/DAG**, dữ liệu chảy một chiều, **không biểu diễn vòng lặp** một cách tự nhiên. Khi cần vòng lặp, rẽ nhánh theo điều kiện, hoặc state phức tạp (đặc trưng của agent), ta dùng **LangGraph** — mô hình hoá luồng thành **đồ thị có chu trình** với node/edge và một **state** dùng chung."

→ Đào sâu ở [[../04-LangGraph-Agentic/00-MOC-LangGraph-Agentic|Môn 4: LangGraph & Agentic AI]].

---

## 9. Pitfalls / Bẫy thường gặp

> [!warning] Quên metadata khi load
> Không gắn metadata → mất pre-filtering. Luôn thu metadata ngay từ loader.

> [!warning] Số chiều embedding phải khớp Vector Store
> Đổi embedding model (1536 → 1024 chiều) mà không re-index → query sai. Phải re-embed.

> [!warning] Tưởng LangChain làm được vòng lặp agent "ngon"
> LCEL chain tuyến tính; ép nó làm agent loop sẽ rối. Đúng công cụ: LangGraph.

> [!warning] `k` quá nhỏ
> `k=1` dễ thiếu context. Production thường k=4–8 rồi re-rank.

---

## 10. Câu hỏi phỏng vấn thường gặp

**Q1: Runnable là gì? Vì sao ghép được bằng `|`?**
> Runnable là interface chuẩn (có `invoke/batch/stream` + async) mà mọi component LangChain implement. Cùng "hình dạng" nên nối được bằng `|` (LCEL): output bước trước = input bước sau, tạo thành chain.

**Q2: LangChain có tuyến tính không? Khác LangGraph thế nào?**
> Có, LCEL chain là tuyến tính/DAG một chiều, không vòng lặp tự nhiên. LangGraph là đồ thị có chu trình + state dùng chung + nhánh điều kiện → hợp cho agent, multi-agent, human-in-the-loop.

**Q3: VectorStore vs Retriever?**
> VectorStore lo lưu trữ + search vector. Retriever là lớp truy vấn (Runnable) ghép chain dễ; tạo từ VectorStore bằng `.as_retriever()`.

**Q4: `mmr` khác `similarity`?**
> `similarity` lấy gần nhất theo vector; `mmr` cân bằng liên quan + **đa dạng** → giảm trùng lặp.

**Q5: LCEL mang lại gì so với viết tay?**
> Streaming, async, batch, retry, fallback, trace — đều có sẵn nhờ chuẩn Runnable.

---

## 11. Bài tập tự luyện

- [ ] **Bài 1:** Dựng RAG chain LCEL hoàn chỉnh `retriever | prompt | llm | StrOutputParser` và chạy `.invoke()` + `.stream()`.
- [ ] **Bài 2:** Dùng `RunnableParallel` chạy song song 2 nhánh (vd: vừa lấy context vừa giữ câu hỏi gốc), in ra dict kết quả.
- [ ] **Bài 3:** Viết 2 câu trả lời mẫu cho câu phỏng vấn "LangChain có tuyến tính không?" — 1 ngắn gọn, 1 chi tiết kèm ví dụ vòng lặp agent.
- [ ] **Bài 4:** Đổi embedding sang `BedrockEmbeddings` (langchain-aws) và chạy lại chain.

---

## 12. Liên kết

- [[03-Modern-RAG-Architecture]] — lý thuyết các component này hiện thực
- [[05-Building-RAG-Agent]] — ghép thành ReAct agent
- [[../04-LangGraph-Agentic/00-MOC-LangGraph-Agentic|Môn 4: LangGraph]] — đồ thị có chu trình & state
- [[../../05-Cloud/01-AWS-Bedrock/00-MOC-AWS-Bedrock|Cloud/AWS Bedrock]] — đổi sang langchain-aws
- [[00-MOC-AI-Fundamentals-RAG|MOC: AI Fundamentals & RAG]]
