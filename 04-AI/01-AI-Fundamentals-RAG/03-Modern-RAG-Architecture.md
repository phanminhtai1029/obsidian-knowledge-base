---
title: "Modern RAG Architecture"
section: 04-AI/01-AI-Fundamentals-RAG
tags: [ai, rag, indexing, retrieval, generation, chunking, hybrid-search, reranking, prompt-engineering, fresher]
related:
  - "[[02-RAG-Theoretical-Foundations]]"
  - "[[04-LangChain-Framework-Core]]"
  - "[[../02-RAG-Optimization/00-MOC-RAG-Optimization]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 60m
source: [QN26_FR_AI_01, "03-modern-rag-architecture.mdx"]
---

# Modern RAG Architecture

> [!summary] TL;DR
> RAG hiện đại = **3 phase**: **(1) Indexing** (ETL: Load → Chunking → Embedding → Vector Store), **(2) Retrieval** (Query processing → Similarity search → Hybrid search → Re-ranking), **(3) Generation** (Context preparation → Prompt engineering → Generation + citation). Nhớ: chunk nhỏ tốt cho **search**, chunk lớn tốt cho **LLM** → giải bằng **Parent-Child indexing**. Retrieval quyết định thành bại; dùng **Hybrid (Dense+Sparse) + RRF + Cross-Encoder re-rank** theo chiến lược "funnel". Generation cần chống **"Lost in the Middle"** bằng **context reordering (U-shape)** + **citation**.

---

## 1. Tổng quan: 3 phase

```text
            ┌───────────── INDEXING (offline) ─────────────┐
Raw data →  Load → Chunking → Embedding → Vector Store
            └───────────────────────────────────────────────┘
                                   │
User query ─────────────────────────────────────────────────┐
            ┌───────────── RETRIEVAL ──────────────┐         │
            Query processing → Similarity/Hybrid →  Re-rank   │
            └───────────────────────────────────────┘         │
                                   │                           │
            ┌───────────── GENERATION ─────────────┐           │
            Context prep → Prompt → LLM → Answer + citation  ◄─┘
            └────────────────────────────────────────┘
```

---

## 2. Phase 1 — Indexing (giống ETL)

Mục tiêu: biến raw data đủ định dạng về dạng chuẩn để search được.

### 2.1 Document Loading
- **Extract content**: bóc bỏ font/màu/layout, giữ lại **plain text**.
- **Collect metadata**: topic, số trang, ngày publish, tác giả… → cực kỳ quan trọng cho **Pre-filtering**.

> [!example] Pre-filtering bằng metadata
> User hỏi "Doanh thu 2024" → hệ thống lọc đúng tài liệu có metadata năm 2024 thay vì search toàn bộ kho.

### 2.2 Text Splitting (Chunking)
Vì sao cần chunk?
- **(a) Context window limit** — không nhồi cả tài liệu vào model được.
- **(b) Search accuracy** — vector của 1 đoạn ngắn tập trung 1 ý sẽ biểu diễn ngữ nghĩa tốt hơn vector trung bình của cả trang lẫn lộn nhiều chủ đề.

| Kiểu chunking | Mô tả | Đánh giá |
|---------------|-------|----------|
| **Fixed-size** | Cắt theo số ký tự/token cố định (vd mỗi 500 ký tự) | Đơn giản, dễ **mất ngữ nghĩa** nếu cắt giữa câu |
| **Recursive** | Cắt theo cấu trúc tự nhiên: `\n\n` → `\n` → dấu câu → space | **Phổ biến nhất**, giữ cấu trúc câu/đoạn |

- **Chunk Overlap**: cho 2 chunk liền kề chồng lấn (thường 10–20% độ dài chunk) → "cầu nối" ngữ cảnh. VD: Chunk 1 kết ở từ 100, Chunk 2 bắt đầu từ từ 80.

### 2.3 Advanced Indexing (Small-to-Big)
Mâu thuẫn: chunk nhỏ tốt cho **search**, chunk lớn tốt cho **LLM** (nhiều context hơn). Giải pháp:

- **Parent-Child Indexing**: cắt thành **Parent** lớn (vd 1000 token) và **Child** nhỏ (vd 200 token). **Search trên Child** (chính xác) nhưng **trả Parent cho LLM** (đủ context).
- **Summary Indexing**: dùng LLM tóm tắt chunk, index bản tóm tắt cô đọng nhưng trả bản gốc chi tiết khi cần.

### 2.4 Embedding
Dùng **Embedding model** biến chunk thành **dense vector**; text có ngữ nghĩa gần nhau → vector gần nhau trong không gian.

### 2.5 Vector Store
Lưu vector + ID + metadata vào DB vector chuyên dụng để phục vụ retrieval.

---

## 3. Phase 2 — Retrieval (quyết định thành bại)

> Nếu khâu này lấy sai/thiếu, LLM không đủ dữ liệu để trả lời đúng.

### 3.1 Query Processing
Câu hỏi user thường ngắn, thiếu ngữ cảnh, đa nghĩa. Cải thiện:

- **Multi-Query**: dùng LLM sinh 3–5 biến thể câu hỏi, search hết rồi gộp kết quả.
  > VD input "db connection error" → sinh các query về timeout / access denied / firewall port 5432 → không bỏ sót tài liệu.
- **HyDE (Hypothetical Document Embeddings)**: nhờ LLM **viết câu trả lời giả định**, rồi dùng vector của câu trả lời đó để search → thu hẹp khoảng cách ngữ nghĩa giữa "câu hỏi" và "tài liệu chứa đáp án".

### 3.2 Similarity Search
- **Dense Retrieval**: tính tương đồng giữa query vector `q` và document vector `d`.
  - Metric: **Cosine Similarity** hoặc **Euclidean Distance**.
  - Thuật toán: **ANN (Approximate Nearest Neighbor)** như **HNSW** thay vì brute-force → nhanh khi dữ liệu lớn.
- **Nhược điểm**: mạnh ngữ nghĩa nhưng yếu **keyword chính xác**. VD "iPhone 14" và "iPhone 15" vector rất gần, nhưng user cần đúng bản 15.

### 3.3 Hybrid Search
Kết hợp 2 luồng song song:

| Luồng | Cơ chế | Thế mạnh |
|-------|--------|----------|
| **Dense (Vector)** | Tương đồng ngữ nghĩa | "giá xe" ↔ "chi phí mua ô tô" |
| **Sparse (Keyword)** | BM25 / TF-IDF, đếm tần suất từ khoá | Tên riêng, thuật ngữ, mã sản phẩm |

- **Reciprocal Rank Fusion (RRF)**: thuật toán gộp & re-rank kết quả 2 luồng.
  - Vấn đề: score của Vector và BM25 khác thang → cộng trực tiếp vô lý.
  - Giải pháp: RRF **bỏ qua score thô**, chuẩn hoá theo **rank**. Công thức: $\text{score} = \sum_i \frac{1}{k + r_i}$ với $r_i$ là rank, hằng số **k thường = 60** (làm mượt, tránh Top 1 áp đảo).
  - Kết quả: tài liệu xếp hạng cao **ở cả hai** luồng được tổng điểm cao hơn tài liệu chỉ top 1 ở một luồng nhưng vắng ở luồng kia.

### 3.4 Re-ranking
Sau khi có tập candidate (vd Top 50), thứ tự chưa chắc đúng vì vector là dạng nén thông tin.

- **Cross-Encoder**: model chấm lại độ liên quan giữa câu hỏi và từng tài liệu.

| | **Bi-Encoder** (Indexing/Retrieval) | **Cross-Encoder** (Re-ranking) |
|---|---|---|
| Cách mã hoá | Hỏi & tài liệu thành **2 vector riêng** | Đưa **cả hỏi + tài liệu cùng lúc** vào model |
| Tốc độ | Rất nhanh | Chậm hơn nhiều |
| Độ chính xác | Mất quan hệ ngữ pháp/ngữ nghĩa phức tạp | Bắt được phủ định, nhân-quả phức tạp |

- **Chiến lược "Funnel" (cái phễu)**: tên gọi xuất phát từ **hình cái phễu** — rộng ở trên, hẹp ở dưới. Ý tưởng: **lọc dần** từ nhiều xuống ít, dùng phương pháp **rẻ/nhanh ở đầu** để bắt được nhiều ứng viên (recall cao), rồi **đắt/chính xác ở cuối** để giữ lại tinh hoa (precision cao):

```text
        ┌─────────────────────────────┐
  rộng  │ Retrieve Many: 50 docs       │  ← Bi-Encoder (nhanh, rẻ) → recall cao
        └──────────────┬──────────────┘
                ┌───────┴───────┐
  hẹp dần       │ Re-rank: 5    │          ← Cross-Encoder (chậm, chính xác) → precision cao
                └───────┬───────┘
                    ┌───┴───┐
  hẹp nhất          │ →LLM  │              ← chỉ 5 doc tinh nhất vào prompt
                    └───────┘
```

  → Vì chạy Cross-Encoder trên cả 50k doc thì quá chậm, ta chỉ chạy nó trên **50 ứng viên** mà Bi-Encoder đã lọc. Đây là cách **cân bằng tốc độ & độ chính xác** kinh điển của RAG.

> [!example] Re-ranking sửa thứ tự
> Query "Sao tôi không nhận được email thông báo?" — Vector search xếp "hướng dẫn tạo chữ ký email" lên Rank 1 (sai intent). Cross-Encoder hiểu "không nhận được" → đẩy "khắc phục email vào spam" lên Rank 1.

---

## 4. Phase 3 — Generation

### 4.1 Context Preparation
- **Context Stuffing**: gộp toàn bộ Top-K vào prompt. Vấn đề: **cost & latency** tăng, **noise** làm LLM lạc đề.
- **Context Reordering** — chống hiện tượng **"Lost in the Middle"**: LLM chú ý tốt nhất ở **đầu & cuối** prompt, hay bỏ quên phần **giữa**.
  > **U-shape Optimization**: đặt tài liệu quan trọng nhất ở **hai đầu**, ít quan trọng ở giữa.
- **Context Compression**: dùng LLM nhỏ/NLP tóm tắt ý chính trước khi feed LLM chính.
  > VD: context thô 300 token về phí huỷ hợp đồng → nén còn "Điều 7.2: Phí huỷ = 02 tháng phí sử dụng" (20 token) → trả lời nhanh, ít hallucinate.

### 4.2 Prompt Engineering
Context tốt + prompt dở vẫn ra đáp án sai.

- **Zero-shot**: template cố định, không ví dụ mẫu.

```text
System: You are an assistant for question-answering tasks. Use the following pieces of
context to answer the question. If you don't know the answer, just say that you don't know.

Context:
{context}

Question:
{question}

Answer:
```

- **Few-shot**: thêm 1–2 ví dụ mẫu (bộ Context–Question–Answer) để LLM học style & format.
- **Chain-of-Thought (CoT)**: yêu cầu model "Let's think step by step" → chính xác hơn với câu hỏi cần logic ([arXiv:2201.11903](https://arxiv.org/abs/2201.11903)).

#### So sánh 3 kỹ thuật cốt lõi

| Kỹ thuật | Có ví dụ mẫu? | Bắt model suy luận? | Dùng khi nào | Đánh đổi |
|----------|:---:|:---:|--------------|----------|
| **Zero-shot** | ❌ | ❌ | Task đơn giản, rõ ràng | Rẻ nhất, nhưng kém với task khó/đặc thù |
| **Few-shot** | ✅ (1–n) | ❌ | Cần đúng **format/style** cụ thể | Tốn token cho ví dụ; chọn ví dụ phải khéo |
| **Chain-of-Thought** | tuỳ (zero/few-shot CoT) | ✅ | Câu hỏi cần **logic nhiều bước**, tính toán, so sánh | Output dài hơn, chậm & tốn token hơn |

> [!tip] Mẹo nhớ
> **Zero-shot** = "làm đi". **Few-shot** = "làm giống mấy ví dụ này". **CoT** = "nghĩ từng bước rồi mới trả lời". CoT có thể kết hợp với few-shot ("few-shot CoT": ví dụ mẫu kèm cả lời giải từng bước).

#### Kỹ thuật bổ sung (hay gặp thực tế)
- **Role / Persona prompting**: gán vai ("Bạn là trợ lý pháp lý…") để định hướng văn phong & chuyên môn.
- **Output format constraint**: ép định dạng đầu ra (JSON, bảng, bullet) → dễ parse bằng Output Parser.
- **Guardrail "không bịa"**: chỉ thị rõ *"Nếu context không có thông tin, hãy trả lời 'Tôi không biết'"* → giảm hallucination (đặc biệt quan trọng trong RAG).

### 4.3 Generation & Attribution
- **Citation** là lợi thế lớn của RAG so với chatbot thường: yêu cầu LLM "mọi thông tin phải kèm ID tài liệu nguồn".
  > VD: "Làm thêm ngày thường hưởng 150% lương cơ bản **[Employee Handbook, P.12]**, ngày lễ 300% **[Luật LĐ 2019, Điều 98]**." → user click `[..]` mở tài liệu gốc đối chiếu.
- **Lợi ích**: dễ kiểm chứng, tăng độ tin cậy, giảm rủi ro bịa thông tin.

---

## 5. Vai trò từng component (phân tích "nếu bỏ đi")

| Bỏ component | Hệ thống trở thành | Giới hạn cốt lõi |
|--------------|--------------------|------------------|
| **Embedding Model** | Lexical Retrieval-driven RAG | Mất query ngữ nghĩa, khó bắt từ đồng nghĩa, dễ sót thông tin (về BM25/TF-IDF) |
| **Vector Store / ANN Index** | Unscalable Prototype | Latency tăng mạnh O(N) khi dữ liệu lớn, khó vận hành |
| **LLM** | Semantic Search / Retrieval System | Không tổng hợp/diễn giải/trả lời hội thoại — user phải tự đọc excerpt |

---

## 6. Pitfalls / Bẫy thường gặp

> [!warning] Chunk quá to hoặc quá nhỏ
> Chunk to → vector nhoè ngữ nghĩa, search kém. Chunk nhỏ → thiếu context cho LLM. Cân bằng bằng overlap + Parent-Child.

> [!warning] Chỉ dùng Vector Search thuần
> Sẽ sót tên riêng/mã sản phẩm/thuật ngữ. Production gần như luôn cần **Hybrid + RRF**.

> [!warning] Nhồi hết Top-K vào prompt
> Tốn token, tăng latency, gây "Lost in the Middle". Dùng reordering + compression.

---

## 7. Câu hỏi phỏng vấn thường gặp

**Q1: 3 phase của RAG hiện đại? Phase nào quyết định thành bại?**
> Indexing → Retrieval → Generation. **Retrieval** quyết định: lấy sai/thiếu thì LLM không cứu được.

**Q2: Vì sao cần Hybrid Search? RRF làm gì?**
> Vector mạnh ngữ nghĩa nhưng yếu keyword chính xác; Sparse (BM25) bắt keyword. RRF gộp 2 luồng dựa trên **rank** (không phải score thô), hằng k≈60, ưu tiên doc xếp cao ở cả hai.

**Q3: Bi-Encoder vs Cross-Encoder?**
> Bi-Encoder mã hoá hỏi & doc thành 2 vector riêng → nhanh, dùng ở retrieval. Cross-Encoder đưa cả cặp vào model → chậm nhưng chính xác, dùng ở re-rank theo chiến lược funnel.

**Q4: "Lost in the Middle" và cách khắc phục?**
> LLM bỏ quên thông tin ở giữa prompt. Khắc phục bằng context reordering U-shape (quan trọng ở 2 đầu) + compression.

**Q5: Parent-Child indexing giải quyết vấn đề gì?**
> Mâu thuẫn chunk nhỏ (search tốt) vs chunk lớn (LLM tốt): search trên Child nhỏ, trả Parent lớn cho LLM.

---

## 8. Bài tập tự luyện

- [ ] **Bài 1:** Cho kho 10k tài liệu policy. Thiết kế pipeline indexing đầy đủ (loader, chunk size/overlap, Parent-Child, metadata). Giải thích lựa chọn.
- [ ] **Bài 2:** Cài đặt Hybrid Search (vector + BM25) + RRF (k=60) trên 1 dataset nhỏ, so sánh kết quả Top-5 với vector-only.
- [ ] **Bài 3:** Viết prompt RAG zero-shot có yêu cầu citation và xử lý trường hợp "không biết".

---

## 9. Liên kết

- [[02-RAG-Theoretical-Foundations]] — nền tảng lý thuyết
- [[04-LangChain-Framework-Core]] — hiện thực từng component
- [[../02-RAG-Optimization/00-MOC-RAG-Optimization|MOC: RAG & Optimization]] — đào sâu indexing/hybrid/rerank
- [[00-MOC-AI-Fundamentals-RAG|MOC: AI Fundamentals & RAG]]
