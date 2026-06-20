---
title: "Experiment Comparison: Naive, Advanced, Graph, Hybrid"
section: 04-AI/03-LLMOps-Evaluation
tags: [ai, llmops, experiment, comparison, naive-rag, advanced-rag, graphrag, hybrid, fresher]
related:
  - "[[01-Ragas-Evaluation-Metrics]]"
  - "[[02-Observability-LangFuse-LangSmith]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 35m
source: [QN26_FR_AI_01, "03-experiment-comparison.mdx"]
---

# Experiment Comparison: Naive vs Advanced vs Graph vs Hybrid RAG

> [!summary] TL;DR
> Đây là note "tổng hợp toàn bộ chương trình": so sánh **4 kiến trúc RAG** trên cùng một bộ test, dùng metric Ragas (Môn 3) + cost/latency. **Naive** (chỉ vector, top-K) — rẻ, nhanh, dở quan hệ. **Advanced** (hybrid search + HyDE + rerank + MMR — chính là Môn 2) — cân bằng, **lựa chọn mặc định cho production**. **GraphRAG** (Neo4j, entity, multi-hop) — vô địch câu hỏi **quan hệ/nhiều bước**. **Hybrid** (gộp tất cả + adaptive routing) — chất lượng cao nhất nhưng đắt & phức tạp nhất. Kết luận thực dụng: **Hybrid mạnh nhất, nhưng Advanced mới là lựa chọn đúng cho đa số đội** (dễ xây, rẻ, vẫn chất lượng). Dùng Graph chỉ khi **thật sự** cần hiểu quan hệ phức tạp.

---

## 1. Mục tiêu & câu hỏi nghiên cứu

Để so sánh **công bằng & khoa học**, phải dựng môi trường có kiểm soát (cùng dataset, cùng metric). 4 câu hỏi nghiên cứu:

1. **GraphRAG so với Naive RAG** ra sao, đặc biệt với truy vấn **multi-hop** (nhiều bước)?
2. **Hybrid tối ưu** thế nào — gộp vector + graph traversal có cho kết quả tốt hơn?
3. **Khi nào dùng kiến trúc nào** — có ranh giới rõ theo độ phức tạp dữ liệu không?
4. **Đánh đổi** chất lượng ↔ chi phí ↔ độ trễ — cái giá thêm của GraphRAG có đáng không?

```
★ Insight ─────────────────────────────────────
• "Multi-hop" = câu hỏi cần NỐI nhiều mẩu sự thật rải ở các tài liệu khác nhau:
  "Cho X, điều gì kéo theo Z?" trong khi X→Y ở tài liệu A, Y→Z ở tài liệu B.
  Vector RAG xử dở vì mỗi chunk là một "ốc đảo" — không có cạnh nối. Graph xử
  tốt vì có thể "nhảy" A→B→C theo quan hệ. Đây là trục phân biệt cốt lõi của
  cả thí nghiệm.
─────────────────────────────────────────────────
```

---

## 2. ⭐ Bốn hệ thống đem so sánh

| Hệ thống | Bản chất | Kỹ thuật chính | Map về môn |
|---|---|---|---|
| **Naive RAG** (baseline) | Chỉ vector search, top-K, feed thẳng | Fixed-size chunking, embedding chuẩn, top-k=5 | Môn 1 |
| **Advanced RAG** | Tối ưu retrieval toàn diện | Hybrid (vector+BM25), HyDE, Cross-Encoder rerank, MMR | **Môn 2** |
| **GraphRAG** | Knowledge graph + vector | Entity/relation extraction, Neo4j, traversal, community detection | Môn 2 (GraphRAG) |
| **Hybrid** | Gộp tất cả + định tuyến | Query classifier, adaptive routing, result fusion (RRF) | Tổng hợp |

### Chi tiết Pros/Cons

**Naive RAG**
- Components: fixed-size chunking (500–1000 ký tự), OpenAI `text-embedding-3-small`, ChromaDB/FAISS, GPT-4, top-k=5.
- ✅ Đơn giản (xong trong một buổi chiều), nhanh, rẻ.
- ❌ **Không quan hệ** (mỗi chunk là ốc đảo), bỏ sót nếu từ khoá không khớp, **không multi-hop**.

**Advanced RAG**
- Components: semantic chunking, HNSW, hybrid search (vector+BM25), HyDE, Cross-Encoder rerank, MMR.
- ✅ Retrieval tốt hơn hẳn, xử lý được câu phức tạp, precision cao (rerank lọc nhiễu).
- ❌ Phức tạp hơn (nhiều index), latency cao hơn (bước rerank), tốn hơn.

**GraphRAG**
- Components: trích `(Subject, Predicate, Object)` bằng LLM, Neo4j, entity-based search, **community detection (thuật toán Leiden)**, subgraph retrieval.
- ✅ Quan hệ tường minh, **multi-hop reasoning** (A→B→C), ít hallucinate về quan hệ.
- ❌ Setup phức tạp (định nghĩa schema khó), chất lượng phụ thuộc **entity extraction** ("garbage in garbage out"), khó cập nhật/bảo trì graph.

**Hybrid**
- Components: tất cả ở trên + **query classifier** (câu này có tính quan hệ không?) + adaptive routing + result fusion (RRF).
- ✅ Chất lượng cao nhất, linh hoạt, xử lý **mọi loại** câu hỏi.
- ❌ Phức tạp nhất (kỹ thuật hệ phân tán), đắt nhất (trả tiền cho mọi pipeline), gánh nặng bảo trì.

```
★ Insight ─────────────────────────────────────
• "Community detection / Leiden algorithm": gom các node liên quan thành "cộng
  đồng" để trả lời câu hỏi TỔNG QUÁT ("tóm tắt xu hướng...") — thay vì chỉ tra
  một entity, GraphRAG có thể tóm tắt cả một cụm. Đây là điểm GraphRAG hiện đại
  (Microsoft GraphRAG) vượt note GraphRAG cơ bản ở Môn 2.
• "Adaptive routing" trong Hybrid = một CLASSIFIER quyết định gửi câu hỏi đi
  đường nào: câu quan hệ → graph, câu factual → vector. Chất lượng Hybrid phụ
  thuộc nặng vào router này — router sai thì gộp pipeline cũng vô ích.
─────────────────────────────────────────────────
```

---

## 3. Thiết kế thí nghiệm

### Dataset
- **Domain:** bài báo khoa học (Computer Science).
- **Quy mô:** 100 tài liệu, 200 câu hỏi.
- **Phân loại câu hỏi** (rất quan trọng — kết quả phân theo loại này):
  - **Factual (40%):** "X là gì?"
  - **Relational (30%):** "X liên quan Y thế nào?"
  - **Multi-hop (20%):** "Cho X, điều gì kéo theo Z?"
  - **Analytical (10%):** "Tóm tắt xu hướng..."

### Ground Truth
- Đáp án do **chuyên gia gán**, có **nguồn tham chiếu** (biết chunk nào chứa đáp án), đã kiểm tra để **không có câu mơ hồ**.

### Dataset Split
- **Train:** 20 câu (tinh chỉnh prompt).
- **Test:** 180 câu (giữ riêng để đánh giá — held-out).

> [!note] Vì sao tách Train/Test?
> Nếu tinh chỉnh prompt trên chính tập đem chấm điểm → "học vẹt đề thi", điểm ảo cao. Giữ test held-out đảm bảo đo đúng năng lực tổng quát. Đây là nguyên tắc ML kinh điển áp vào đánh giá RAG.

---

## 4. Bộ metric đầy đủ (4 nhóm)

| Nhóm | Metric | Ý nghĩa |
|---|---|---|
| **Retrieval** | Context Precision, Context Recall, Retrieval Latency, số API calls | Tìm có sạch/đủ/nhanh/đắt không (→ Ragas, Môn 3) |
| **Generation** | Faithfulness, Answer Relevance, **Answer Correctness**, Generation Latency | Sinh có trung thực/đúng trọng tâm/đúng sự thật không |
| **Cost** | Embedding calls, LLM calls, cost/query, cost/token | Tiền thực tế mỗi truy vấn |
| **User-Centric** | End-to-end latency, Answer completeness, User satisfaction (LLM-as-a-Judge) | Trải nghiệm người dùng cảm nhận |

> [!tip] Đánh giá đa chiều — đừng chỉ nhìn accuracy
> Một hệ "điểm chất lượng cao nhất" nhưng **đắt gấp 5 lần và chậm gấp 3** có thể là lựa chọn **tệ** cho sản phẩm thực. Luôn cân chất lượng **cùng** cost & latency. Đây chính là tinh thần LLMOps.

### Code đánh giá (rút gọn)

```python
from ragas import evaluate
from ragas.metrics import (faithfulness, answer_relevance,
                           context_precision, context_recall)

def evaluate_system(system, dataset):
    outputs = []
    for item in dataset:
        result = system.query(item["question"])
        outputs.append({
            "question": item["question"],
            "answer": result["answer"],
            "contexts": result["contexts"],
            "ground_truth": item["ground_truth"],   # cần cho Context Recall
        })
    scores = evaluate(outputs, metrics=[faithfulness, answer_relevance,
                                        context_precision, context_recall])
    scores["avg_latency"] = measure_latency(system)
    scores["avg_cost"]    = calculate_cost(system)
    return scores
```

> [!warning] Statistical significance — T-Test
> Chênh lệch điểm nhỏ có thể chỉ là **nhiễu ngẫu nhiên**. Dùng **T-Test** kiểm tra khác biệt giữa 2 hệ có **thực sự** (statistically significant) hay không trước khi kết luận "hệ A tốt hơn hệ B".

---

## 5. ⭐ Kết quả & phát hiện chính

> [!note] Lưu ý về số liệu
> Bảng kết quả trong học liệu gốc để **trống** (template để học viên tự chạy điền). Phần dưới là **xu hướng định tính** mà học liệu kết luận — đây mới là phần recruiter hỏi.

**Phát hiện tổng quát:**
- **Chất lượng cao nhất → Hybrid** — đứng đầu mọi mặt nhưng **giá cao nhất**.
- **Hiệu quả chi phí nhất → Naive** — vô địch cho tác vụ đơn giản.
- **Multi-hop tốt nhất → Graph** — thống trị recall ở câu hỏi phức tạp.
- **Cân bằng → Advanced** — lựa chọn **thực dụng cho production**.

### Hiệu năng theo loại câu hỏi

| Loại câu hỏi | Naive | Advanced | Graph | Hybrid |
|---|---|---|---|---|
| **Factual** ("X là gì") | Khá | Tốt hơn | Tương đương (thừa) | **Tốt nhất** |
| **Relational** ("X liên hệ Y") | **Kém** (mất liên kết) | Tốt | **Xuất sắc** (sinh ra để làm việc này) | Xuất sắc |
| **Multi-hop** ("X kéo theo Z") | Kém | Tạm | **Xuất sắc** | **Tốt nhất** |
| **Analytical** ("tóm tắt xu hướng") | Khó cho mọi hệ | Khó | Khó | Nhỉnh hơn (context bao quát) |

```
★ Insight ─────────────────────────────────────
• Đọc bảng này theo cột "loại câu hỏi", không theo "hệ nào mạnh nhất": với câu
  FACTUAL, Graph là OVERKILL (tốn công mà chỉ ngang Naive). Bài học: kiến trúc
  phải KHỚP loại dữ liệu/câu hỏi, không phải "càng phức tạp càng tốt".
• Câu ANALYTICAL khó cho TẤT CẢ — vì cần tổng hợp toàn corpus, vượt khả năng
  retrieve-then-read. Đây là biên giới hiện tại của RAG, dẫn tới hướng agentic
  (Môn 4): để agent lặp nhiều vòng truy vấn thay vì một lần.
─────────────────────────────────────────────────
```

---

## 6. Khuyến nghị: khi nào dùng kiến trúc nào?

| Dùng | Khi |
|---|---|
| **Naive RAG** | Câu factual đơn giản; nhạy chi phí; yêu cầu latency thấp; tập tài liệu nhỏ (< 50 docs) |
| **Advanced RAG** | Câu phức tạp; ứng dụng đòi chất lượng; tập tài liệu lớn → **mặc định cho production** |
| **GraphRAG** | Dữ liệu quan hệ cao (gian lận, chuỗi cung ứng, mạng lưới); multi-hop là yêu cầu cứng; domain có entity rõ |
| **Hybrid** | Câu hỏi đủ loại (user hỏi bất kỳ thứ gì); cần chất lượng cao nhất bất kể giá; có ngân sách; domain phức tạp (pháp lý, y tế) |

### Ưu tiên tối ưu cho từng hệ
- **Naive:** cải thiện chunking & embedding model.
- **Advanced:** tinh chỉnh Reranker & Query Transformation.
- **Graph:** tập trung **chất lượng entity extraction** (prompt engineering).
- **Hybrid:** cải thiện **logic routing** (classifier).

> [!tip] Kết luận thực dụng (câu chốt hay được hỏi)
> Hybrid mạnh nhất, **nhưng Advanced RAG mới là lựa chọn tốt nhất cho đa số đội** vì dễ xây hơn, rẻ hơn khi vận hành, mà vẫn chất lượng cao. **Chỉ dùng GraphRAG khi bạn thực sự cần hiểu quan hệ phức tạp** giữa các mẩu dữ liệu.

---

## 7. Pitfalls / Bẫy thường gặp

> [!warning] "Phức tạp hơn = tốt hơn"
> Sai. Graph trên câu factual là overkill (ngang Naive nhưng tốn gấp bội). Chọn kiến trúc theo loại câu hỏi & dữ liệu, không theo độ "xịn".

> [!warning] Kết luận từ chênh lệch nhỏ không kiểm định
> 0.82 vs 0.84 có thể là nhiễu. Phải T-Test; nếu không significant thì coi như ngang nhau.

> [!warning] Chỉ tối ưu accuracy, bỏ qua cost/latency
> Hệ điểm cao nhưng đắt/chậm có thể không dùng được trong sản phẩm. LLMOps đo đa chiều.

> [!warning] Tinh chỉnh prompt trên tập test
> Gây "rò rỉ" → điểm ảo. Luôn giữ test held-out, tinh chỉnh trên train.

---

## 8. Câu hỏi phỏng vấn thường gặp

**Q1: 4 kiến trúc RAG khác nhau thế nào?**
> Naive: chỉ vector + top-K, đơn giản/rẻ/dở quan hệ. Advanced: hybrid search + HyDE + rerank + MMR (toàn bộ Môn 2), cân bằng. GraphRAG: Neo4j + entity + traversal, mạnh multi-hop/quan hệ. Hybrid: gộp tất cả + adaptive routing, chất lượng cao nhất nhưng đắt/phức tạp nhất.

**Q2: Khi nào GraphRAG vượt Naive rõ nhất?**
> Ở câu hỏi relational và multi-hop — cần nối nhiều mẩu sự thật rải rác qua quan hệ. Naive coi mỗi chunk là ốc đảo nên trượt; Graph traverse A→B→C nên thắng. Với câu factual đơn giản thì Graph là overkill.

**Q3: Nếu chỉ chọn một kiến trúc cho production, chọn gì, vì sao?**
> Advanced RAG — chất lượng cao, dễ xây và rẻ hơn Hybrid, xử lý tốt phần lớn truy vấn thực tế. Chỉ nâng lên Graph/Hybrid khi có yêu cầu quan hệ phức tạp rõ ràng và ngân sách.

**Q4: Vì sao đánh giá phải đo cả cost & latency, không chỉ chất lượng?**
> Vì sản phẩm thực bị ràng buộc tiền và tốc độ. Một hệ điểm cao nhất nhưng đắt gấp 5/chậm gấp 3 có thể không khả thi. LLMOps yêu cầu cân bằng đa chiều.

**Q5: Vì sao cần T-Test khi so sánh hệ thống?**
> Để biết chênh lệch điểm là thực hay chỉ nhiễu ngẫu nhiên. Khác biệt không statistically significant thì không được kết luận hệ này hơn hệ kia.

**Q6: Loại câu hỏi nào khó cho mọi kiến trúc, vì sao?**
> Analytical (tóm tắt xu hướng) — cần tổng hợp toàn corpus, vượt mô hình retrieve-then-read một lần. Đây là động lực cho hướng agentic (Môn 4): lặp nhiều vòng truy vấn/suy luận.

---

## 9. Bài tập tự luyện

- [ ] **Bài 1:** Dựng `ExperimentRunner` với ít nhất Naive vs Advanced; chạy trên ~20 câu, điền bảng kết quả §5.
- [ ] **Bài 2:** Phân loại câu hỏi test thành Factual/Relational/Multi-hop và đối chiếu hệ nào thắng từng loại (kiểm chứng bảng §5).
- [ ] **Bài 3:** Đo cost & latency mỗi hệ; vẽ biểu đồ chất lượng ↔ chi phí để thấy đánh đổi.
- [ ] **Bài 4:** Chạy T-Test giữa Advanced và Hybrid trên Faithfulness → khác biệt có significant không?

---

## 10. Liên kết

- [[01-Ragas-Evaluation-Metrics]] — bộ metric dùng để chấm 4 hệ thống
- [[02-Observability-LangFuse-LangSmith]] — thu thập trace/cost/latency cho thí nghiệm
- [[../02-RAG-Optimization/00-MOC-RAG-Optimization|MOC: RAG & Optimization]] — "Advanced RAG" = toàn bộ Môn 2
- [[../02-RAG-Optimization/05-GraphRAG-Implementation]] — chi tiết kiến trúc GraphRAG
- [[../04-LangGraph-Agentic/00-MOC-LangGraph-Agentic|MOC: LangGraph & Agentic]] — hướng agentic giải bài toán analytical
- [[00-MOC-LLMOps-Evaluation|MOC: LLMOps & Evaluation]]
