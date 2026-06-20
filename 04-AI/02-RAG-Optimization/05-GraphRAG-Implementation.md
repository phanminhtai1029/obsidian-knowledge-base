---
title: "GraphRAG Implementation"
section: 04-AI/02-RAG-Optimization
tags: [ai, rag, graphrag, neo4j, cypher, knowledge-graph, entity-extraction, fresher]
related:
  - "[[04-Post-Retrieval-Processing]]"
  - "[[../01-AI-Fundamentals-RAG/04-LangChain-Framework-Core]]"
difficulty: ⭐⭐⭐⭐⭐
estimated_time: 45m
source: [QN26_FR_AI_01, "05-graph-rag-implementation.mdx"]
---

# GraphRAG Implementation

> [!summary] TL;DR
> **GraphRAG** kết hợp **graph database (Neo4j)** với retrieval để biểu diễn tri thức **có cấu trúc** — trả lời được câu hỏi mà RAG vector-similarity thuần **không** làm được (đếm, gộp, truy quan hệ nhiều bước). Pipeline 4 bước: **Entity Extraction** (LLM + Pydantic `with_structured_output` rút entity/relationship từ PDF) → **Graph Storage** (MERGE node + relationship vào Neo4j) → **Cypher Generation** (`GraphCypherQAChain` dịch câu hỏi tự nhiên → Cypher) → **Answer Generation**. Mạnh ở: tri thức có cấu trúc, truy vết quan hệ, compliance audit. Đây là phần **thi Practical (coding)**.

---

## 1. GraphRAG là gì & khi nào dùng?

RAG vector thuần tìm theo **độ tương đồng ngữ nghĩa** — rất tốt cho "tìm đoạn nói về X". Nhưng nó **bó tay** với câu hỏi cần **quan hệ tường minh**, ví dụ: *"Có bao nhiêu chính sách ảnh hưởng tới Nhân viên?"* — không đoạn văn nào "giống" câu này về ngữ nghĩa; câu trả lời nằm ở **cấu trúc liên kết** giữa các thực thể.

**GraphRAG** lưu tri thức dưới dạng **đồ thị**: node = thực thể, edge = quan hệ. Khi đó truy vấn = **duyệt quan hệ** (graph traversal), không phải đo khoảng cách vector.

| Tiêu chí | **Vector RAG** | **GraphRAG** |
|---|---|---|
| Biểu diễn | Vector embedding (phi cấu trúc) | Node + Relationship (có cấu trúc) |
| Tìm bằng | Độ tương đồng ngữ nghĩa | Duyệt quan hệ (Cypher traversal) |
| Mạnh ở | "Tìm đoạn nói về X" | Đếm/gộp/quan hệ nhiều bước, audit |
| Yếu ở | Câu hỏi quan hệ, đếm, multi-hop | Cần extract cấu trúc trước (tốn công) |

```
★ Insight ─────────────────────────────────────
• Câu thần chú để nhớ: vector trả lời "CÁI GÌ giống thế này?", graph trả lời
  "CÁI GÌ LIÊN QUAN tới cái kia, qua quan hệ nào?". Multi-hop ("chính sách nào
  ảnh hưởng stakeholder bị ràng buộc bởi luật Y") là sân nhà của graph.
• Thực tế production hay dùng HYBRID: vector tìm đoạn + graph truy quan hệ — note
  gốc cũng ghi chú đây mới là hướng tối ưu (xem mục NOTE cuối).
─────────────────────────────────────────────────
```

---

## 2. Kiến trúc GraphRAG — 4 thành phần

1. **Entity Extraction** — nhận diện thực thể & quan hệ then chốt từ tài liệu.
2. **Graph Storage** — lưu dữ liệu có cấu trúc vào Neo4j.
3. **Semantic Search** — LLM hiểu câu hỏi ngôn ngữ tự nhiên.
4. **Graph Traversal** — tận dụng quan hệ để trả lời theo ngữ cảnh.

> Dữ liệu mẫu trong khoá: **`FSoft_HR.pdf`** — chính sách HR, cam kết nhân viên, stakeholder, yêu cầu tuân thủ.

---

## 3. Bước 1 — Setup & định nghĩa schema (Pydantic)

```python
from langchain_neo4j import Neo4jGraph, GraphCypherQAChain
from langchain_openai import ChatOpenAI
from langchain_text_splitters import RecursiveCharacterTextSplitter
from docling.document_converter import DocumentConverter
from pydantic import BaseModel, Field
from typing import List
from enum import Enum

# Kết nối Neo4j
graph = Neo4jGraph(url="bolt://localhost:7687", username="neo4j", password="<your-password>")
```

Định nghĩa **Pydantic models** làm **schema validation** cho structured output của LLM — đảm bảo dữ liệu rút ra nhất quán:

```python
class ConstraintUnit(str, Enum):
    hours = "hours"; dong = "dong"; percent = "percent"; other = "other"

class ConstraintPeriod(str, Enum):
    month = "month"; year = "year"; none = "none"

class Constraint(BaseModel):
    """Giới hạn/yêu cầu đo lường được"""
    metric: str
    value: float
    unit: ConstraintUnit
    period: ConstraintPeriod

class Commitment(BaseModel):
    """Nghĩa vụ / cam kết trong chính sách"""
    description: str
    measurable: bool
    constraints: List[Constraint] = []

class PolicyClauseExtraction(BaseModel):
    """Cấu trúc rút trích cấp cao nhất"""
    clause_title: str
    clause_text: str
    stakeholders: List[str] = []
    regulations: List[str] = []
    commitments: List[Commitment] = []
```

> [!tip] Schema phải khớp domain của bạn
> Mẫu trên dành cho chính sách HR. Tài liệu y tế thì đổi sang `Symptoms`, `Diagnoses`, `Treatments`, `MedicationConstraints`... Định nghĩa Pydantic model theo: loại tài liệu, thông tin muốn rút, quan hệ & ràng buộc của use case.

---

## 4. Bước 2 — Rút trích có cấu trúc

```python
# Load + split PDF
doc = DocumentConverter().convert("FSoft_HR.pdf").document
markdown_text = doc.export_to_markdown()
chunks = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200).split_text(markdown_text)

# LLM với structured output ← mấu chốt
model = ChatOpenAI(model="gpt-4-1-nano", temperature=0.1, max_tokens=1000)
model_with_structure = model.with_structured_output(PolicyClauseExtraction)

EXTRACTION_PROMPT = """You are an information extraction engine.
From the policy text chunk, extract structured policy information.
Rules:
1. A clause is a policy topic unit.
2. A commitment is a clear promise/obligation/prohibition.
3. If a commitment has measurable numeric limits, extract them as constraints.
4. Extract stakeholders explicitly mentioned (Employee, Partner, Board, Government...).
5. Extract legal/regulatory references explicitly mentioned.
6. Do NOT invent information.   7. If something doesn't exist, return empty list.
Text:\n{chunk}"""

all_extractions = []
for chunk in chunks:
    resp = model_with_structure.invoke(EXTRACTION_PROMPT.format(chunk=chunk))
    all_extractions.append(resp)
```

```
★ Insight ─────────────────────────────────────
• `with_structured_output(PydanticModel)` ép LLM trả về ĐÚNG schema (qua function
  calling/JSON mode) → không phải parse text tự do. Đây là cây cầu nối "văn bản
  tự do → dữ liệu có cấu trúc" — trái tim của GraphRAG ingestion.
• `temperature=0.1` thấp có chủ đích: extraction cần ỔN ĐỊNH, không sáng tạo.
  Rule "Do NOT invent" + "return empty list" là guardrail chống bịa entity.
─────────────────────────────────────────────────
```

---

## 5. Bước 3 — Nạp vào Neo4j (Cypher MERGE)

**Schema đồ thị:** node `PolicyClause`, `Stakeholder`, `Regulation`, `Commitment`, `Constraint`; quan hệ `AFFECTS`, `REFERENCES`, `CONTAINS`, `HAS_CONSTRAINT`.

```python
for ext in all_extractions:
    # Tạo node PolicyClause
    graph.query("""
        MERGE (clause:PolicyClause {title: $title})
        SET clause.text = $text
    """, {"title": ext.clause_title, "text": ext.clause_text})

    # Stakeholder + quan hệ AFFECTS
    for s in ext.stakeholders:
        graph.query("""
            MERGE (s:Stakeholder {name: $name})
            WITH s MATCH (c:PolicyClause {title: $ct})
            MERGE (c)-[:AFFECTS]->(s)
        """, {"name": s, "ct": ext.clause_title})

    # Commitment (CONTAINS) + Constraint (HAS_CONSTRAINT) ... tương tự
```

> [!note] Vì sao dùng `MERGE` chứ không `CREATE`?
> **`MERGE`** = "tìm thấy thì dùng, không thì tạo" → **chống node trùng lặp**. Nếu "Employee" xuất hiện ở 10 clause, ta muốn **1 node** Stakeholder nối tới 10 clause, chứ không phải 10 node "Employee". `CREATE` sẽ tạo trùng.

---

## 6. Bước 4 — Truy vấn bằng ngôn ngữ tự nhiên

`GraphCypherQAChain` tự **dịch câu hỏi tự nhiên → Cypher**, chạy trên Neo4j, rồi diễn giải kết quả:

```python
chain = GraphCypherQAChain.from_llm(
    ChatOpenAI(temperature=0),
    graph=graph, verbose=True,
    allow_dangerous_requests=True,
)
response = chain.invoke("how many policies that affect Employees?")
```

**Cách hoạt động:** (1) LLM hiểu câu hỏi → (2) sinh Cypher phù hợp → (3) Neo4j duyệt đồ thị → (4) chuyển kết quả thành câu trả lời.

> [!warning] `allow_dangerous_requests=True` — hiểu rủi ro
> Cho phép chain chạy Cypher do LLM sinh ra trực tiếp lên DB → nguy cơ query phá hoại/đọc lậu nếu prompt bị lạm dụng. Production cần kiểm soát: validate Cypher trước khi chạy, quyền read-only, allowlist.

> [!tip] Khi cần kiểm soát hơn — dùng custom agent / LangGraph
> Bọc `GraphCypherQAChain` hoặc dùng **LangGraph agent** để: validate & tinh chỉnh Cypher trước khi chạy, áp rule tối ưu domain, fallback cho query phức tạp, cache, log & monitor hiệu năng.

---

## 7. Lợi ích & giới hạn

**Lợi ích:**
- **Structured Knowledge** — quan hệ tường minh → trả lời được câu mà vector-similarity chịu thua.
- **Context-Aware Retrieval** — duyệt đồ thị cho ngữ cảnh phong phú.
- **Compliance Tracking** — dễ audit "cam kết nào ảnh hưởng stakeholder nào".
- **Complex Reasoning** — kết hợp traversal + LLM reasoning.

**Giới hạn (NOTE gốc):** GraphRAG phụ thuộc dữ liệu có cấu trúc cụ thể; cách trên *chưa chắc tối ưu*. Nên thử **hybrid retrieval** — kết hợp **vector search + graph exploration**.

---

## 8. Pitfalls / Bẫy thường gặp

> [!warning] Dùng `CREATE` thay `MERGE` → node trùng
> Sẽ tạo nhiều node "Employee" rời rạc, phá vỡ khả năng đếm/gộp — đúng thứ GraphRAG sinh ra để làm.

> [!warning] Extraction bịa entity
> Không có guardrail "Do NOT invent / return empty list" → LLM bịa stakeholder/constraint → đồ thị nhiễu. Giữ temperature thấp + rule rõ ràng.

> [!warning] Tin tưởng Cypher do LLM sinh mà không kiểm soát
> LLM có thể sinh Cypher sai hoặc nguy hiểm. Đừng để `allow_dangerous_requests=True` lên production trần trụi.

---

## 9. Câu hỏi phỏng vấn thường gặp

**Q1: GraphRAG khác RAG vector thường ở đâu?**
> RAG vector tìm theo độ tương đồng ngữ nghĩa (tốt cho "tìm đoạn về X"). GraphRAG lưu tri thức thành đồ thị node–quan hệ, truy vấn bằng traversal (Cypher) → trả lời được câu cần quan hệ, đếm, gộp, multi-hop mà vector không làm được.

**Q2: Vai trò của `with_structured_output` + Pydantic trong extraction?**
> Ép LLM trả về đúng schema đã định nghĩa (qua function calling/JSON) → biến văn bản tự do thành dữ liệu có cấu trúc nhất quán để nạp vào graph, khỏi parse text thủ công.

**Q3: Vì sao dùng `MERGE` trong Cypher?**
> MERGE = match-or-create, chống tạo node trùng. Cùng một thực thể (vd "Employee") xuất hiện nhiều nơi chỉ có 1 node, các quan hệ cùng trỏ về nó → mới đếm/gộp đúng.

**Q4: `GraphCypherQAChain` làm gì?**
> Dịch câu hỏi ngôn ngữ tự nhiên thành Cypher, chạy trên Neo4j, rồi diễn giải kết quả thành câu trả lời. `allow_dangerous_requests=True` để cho phép thực thi Cypher sinh động — cần kiểm soát ở production.

**Q5: Khi nào chọn GraphRAG?**
> Khi câu hỏi xoay quanh quan hệ/đếm/tổng hợp/audit trên dữ liệu có cấu trúc rõ (chính sách, hồ sơ, hệ thống compliance). Tốt nhất kết hợp hybrid với vector search.

---

## 10. Bài tập tự luyện

- [ ] **Bài 1:** Định nghĩa Pydantic schema cho một loại tài liệu khác (vd hợp đồng) + viết extraction prompt có guardrail "không bịa".
- [ ] **Bài 2:** Nạp dữ liệu vào Neo4j bằng `MERGE`; viết Cypher đếm số clause `AFFECTS` một stakeholder.
- [ ] **Bài 3:** Chạy `GraphCypherQAChain` với 5 câu hỏi quan hệ; in Cypher (`verbose=True`) và nhận xét cái nào LLM sinh sai.
- [ ] **Bài 4 (nâng cao):** Phác thảo pipeline hybrid: vector tìm đoạn liên quan + graph truy quan hệ, rồi gộp context.

---

## 11. Liên kết

- [[04-Post-Retrieval-Processing]] — re-rank trong nhánh vector RAG (so sánh hướng tiếp cận)
- [[../01-AI-Fundamentals-RAG/04-LangChain-Framework-Core]] — Output Parser / structured output, Runnable
- [[../04-LangGraph-Agentic/00-MOC-LangGraph-Agentic]] — custom agent kiểm soát Cypher generation
- [[../01-AI-Fundamentals-RAG/02-RAG-Theoretical-Foundations#Glossary mở rộng (tra cứu nhanh — gom từ cả 4 môn AI)|Glossary mở rộng]]
- [[00-MOC-RAG-Optimization|MOC: RAG & Optimization]]
