---
title: "Hybrid Search"
section: 04-AI/02-RAG-Optimization
tags: [ai, rag, hybrid-search, bm25, tf-idf, rrf, dense-sparse, fresher]
related:
  - "[[01-Advanced-Indexing]]"
  - "[[03-Query-Transformation]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 35m
source: [QN26_FR_AI_01, "02-hybrid-search.mdx"]
---

# Hybrid Search

> [!summary] TL;DR
> Vector Search mạnh ngữ nghĩa nhưng **yếu khi cần khớp chính xác từ khoá** (tên riêng, mã sản phẩm, mã lỗi "503"). **BM25** (bản nâng cấp của TF-IDF) bù đúng điểm yếu đó với 3 nguyên lý: **TF Saturation** (chống spam từ khoá), **IDF** (thưởng từ hiếm, phạt từ phổ biến), **Length Normalization** (phạt văn bản dài loãng). **Hybrid Search** = chạy song song **Sparse (BM25)** + **Dense (Vector)** rồi gộp. Vì 2 thang điểm khác nhau, ta gộp bằng **RRF** — chỉ dựa trên **rank**, công thức `Σ 1/(k + rank)` với **k≈60**.

---

## 1. Giới hạn của Vector Search → vai trò BM25

Ví dụ: người dùng tìm `Error 503 Service Unavailable`.

- **Vector Search** — tìm đoạn **nghĩa tương đương**: "server quá tải", "mất kết nối mạng". Đôi khi **bỏ sót con số "503"** vì trong không gian vector các con số không mang nhiều ngữ nghĩa riêng.
- **Keyword Search** — tìm **đúng** chuỗi "Error" và "503". Đây là lúc cần thuật toán dựa trên **tần suất từ khoá**, tiêu biểu là **BM25**.

```
★ Insight ─────────────────────────────────────
• Vector = "hiểu ý"; BM25 = "khớp chữ". Chúng bù trừ: tên riêng/mã/thuật ngữ
  hiếm là điểm CHẾT của vector nhưng là điểm MẠNH của BM25, và ngược lại với
  câu hỏi diễn đạt bằng từ đồng nghĩa.
• Đây chính là lý do "Hybrid" tồn tại — không phải vì BM25 cũ kỹ bị bỏ, mà vì
  nó vẫn vô địch ở khớp từ khoá chính xác.
─────────────────────────────────────────────────
```

---

## 2. Thuật toán BM25 (Best Matching 25)

BM25 là **gold standard** của information retrieval cổ điển — bản tinh chỉnh của **TF-IDF**, khắc phục nhược điểm về **spam từ khoá** và **độ dài văn bản**. Chấm điểm liên quan dựa trên 3 nguyên lý:

| Nguyên lý | TF-IDF | BM25 cải tiến |
|---|---|---|
| **1. TF Saturation** | Từ xuất hiện nhiều → điểm tăng **tuyến tính vô hạn** (100 lần → cao gấp 100) | Áp **bão hoà**: lần 2, 3 còn tăng đáng kể; nhưng đã 100 lần thì lần 101 hầu như không thêm điểm → **chống spam** |
| **2. IDF** (Inverse Document Frequency) | — | **Phạt nặng từ phổ biến, thưởng lớn từ hiếm**. Vd "viêm màng não" quan trọng hơn nhiều chữ "của" |
| **3. Length Normalization** | — | Từ khoá xuất hiện 1 lần trong đoạn **ngắn** được chấm cao hơn 1 lần trong tiểu thuyết **dài** (vì thông tin bị loãng) |

> [!tip] Vì sao cần "Saturation"? — ví dụ "Galaxy"
> **Doc A:** "Galaxy Galaxy Galaxy..." (lặp 1000 lần để spam top). **Doc B:** "Samsung vừa ra mắt điện thoại Galaxy mới..." (xuất hiện 5 lần, ngữ cảnh hợp lý).
> - **TF-IDF:** Score(A) >> Score(B) — Doc A thắng áp đảo nhờ 1000 từ khoá → **kết quả méo mó**.
> - **BM25:** sau ~10–20 lần, điểm Doc A không tăng nữa; Doc B được cộng điểm nhờ độ dài hợp lý → **đánh giá công bằng**, triệt tiêu lợi thế spam.

---

## 3. Hybrid Search — quy trình

Không phải thuật toán mới, mà là **quy trình gộp** kết quả từ 2 luồng search song song:

| Khái niệm | Định nghĩa |
|---|---|
| **Sparse Retriever** | Tìm theo từ khoá thống kê (BM25/TF-IDF). Vector "thưa" — đa số chiều = 0 |
| **Dense Retriever** | Tìm theo vector ngữ nghĩa (embedding). Vector "đặc" — mọi chiều có giá trị |

**Quy trình:**
1. **Parallel Execution** — gửi query đồng thời cho cả 2 engine:
   - BM25 → list A (khớp từ khoá chính xác).
   - Vector → list B (khớp ngữ cảnh & đồng nghĩa).
2. **Fusion** — gộp 2 list thành một.
3. **Ranking** — sắp lại thứ tự ưu tiên → Top-K tốt nhất cho LLM.

---

## 4. ⭐ Reciprocal Rank Fusion (RRF)

**Vấn đề:** 2 thang điểm không cùng đơn vị → **không cộng trực tiếp được**:
- Vector dùng cosine similarity → điểm trong khoảng **[0, 1]**.
- BM25 dùng công thức thống kê → điểm là **số dương bất kỳ**.

**Giải pháp RRF:** bỏ qua **score**, chỉ quan tâm **rank**. Giả định: tài liệu đứng hạng cao ở **cả 2** list thì chắc chắn quan trọng.

$$ RRF(d) = \sum_{r \in R} \frac{1}{k + rank_r(d)} $$

- `rank_r(d)` — hạng của tài liệu `d` trong list `r`, bắt đầu từ 1.
- `k` — hằng số làm mượt, thường **k = 60**. Nó **giảm chênh lệch điểm giữa các hạng rất cao** (Top1 vs Top2) → công bằng hơn.

> [!tip] Sức mạnh RRF — ví dụ k=60
> - **Doc A:** Rank 1 (Vector, đúng ngữ nghĩa) nhưng Rank 10 (BM25, thiếu từ khoá chính xác).
> - **Doc B:** Rank 2 (Vector) và Rank 3 (BM25) — khá tốt cả hai.
>
> Score(A) = 1/61 + 1/70 ≈ 0.0164 + 0.0143 = **0.0307**
> Score(B) = 1/62 + 1/63 ≈ 0.0161 + 0.0159 = **0.0320**
>
> → **Doc B thắng** (0.0320 > 0.0307). Dù Doc A là Top1 Vector, RRF ưu tiên Doc B vì **đồng thuận cao từ cả 2** thuật toán — chọn tài liệu "hài hoà" thay vì tin một phía.

```
★ Insight ─────────────────────────────────────
• k=60 KHÔNG phải con số ma thuật bắt buộc — nó là hyperparameter làm mượt. k
  lớn → san phẳng ảnh hưởng của các hạng đầu (mọi rank gần như ngang nhau); k
  nhỏ → top-rank áp đảo. 60 là mặc định kinh nghiệm phổ biến.
• RRF thắng vì RANK ổn định hơn SCORE: score thô của BM25 và cosine không cùng
  phân phối, chuẩn hoá khó; rank thì luôn là 1,2,3... so sánh được ngay.
─────────────────────────────────────────────────
```

---

## 5. Bảng so sánh tổng kết

| Phương pháp | Ưu điểm | Nhược điểm |
|---|---|---|
| **Vector Search** | Hiểu ngữ nghĩa sâu, tìm đồng nghĩa, đa ngôn ngữ | Kém khớp từ khoá chính xác, khó giải thích kết quả |
| **BM25** | Khớp từ khoá chính xác, mạnh ở domain chuyên ngành, nhanh | Không hiểu ngữ cảnh, hỏng nếu user dùng từ đồng nghĩa khác văn bản |
| **Hybrid Search** | Cân bằng: không bỏ sót từ khoá quan trọng + vẫn hiểu ngữ cảnh | Triển khai phức tạp hơn, tốn tài nguyên (chạy 2 luồng song song) |

---

## 6. Pitfalls / Bẫy thường gặp

> [!warning] Cộng thẳng cosine + BM25 score
> Sai cơ bản: 2 thang điểm khác đơn vị/phân phối. Phải chuẩn hoá (min-max) hoặc — tốt hơn — dùng **RRF dựa trên rank**.

> [!warning] Bỏ BM25 vì nghĩ "vector là đủ"
> Với mã lỗi, SKU, tên riêng, thuật ngữ hiếm → vector hay trượt. Production-grade RAG gần như luôn cần Hybrid.

> [!warning] Quên BM25 không hiểu đồng nghĩa
> Nếu user hỏi bằng từ khác hẳn văn bản (vd "xe điện" vs "EV"), BM25 trượt — đó là lúc luồng Dense gánh.

---

## 7. Câu hỏi phỏng vấn thường gặp

**Q1: Hybrid Search là gì, gồm những gì?**
> Chạy song song Sparse Retriever (BM25 — khớp từ khoá) và Dense Retriever (Vector — khớp ngữ nghĩa), rồi gộp kết quả (thường bằng RRF) để vừa không bỏ sót từ khoá vừa hiểu ngữ cảnh.

**Q2: BM25 khác TF-IDF ở đâu?**
> BM25 thêm 3 cải tiến: TF Saturation (điểm bão hoà, chống spam), IDF mạnh hơn (thưởng từ hiếm), và Length Normalization (phạt văn bản dài loãng). TF-IDF tăng điểm tuyến tính vô hạn theo tần suất → dễ bị spam.

**Q3: Vì sao dùng RRF thay vì cộng score?**
> Vì cosine ∈ [0,1] còn BM25 là số dương bất kỳ — không cùng thang. RRF bỏ score, chỉ dùng rank (`1/(k+rank)`), nên gộp được công bằng; ưu tiên tài liệu đồng thuận ở nhiều luồng.

**Q4: k trong RRF để làm gì, vì sao thường là 60?**
> k là hằng số làm mượt, giảm chênh lệch điểm giữa các hạng rất cao (Top1 vs Top2). 60 là mặc định kinh nghiệm; k lớn càng san phẳng ảnh hưởng top-rank.

**Q5: Dense vs Sparse vector khác nhau thế nào?**
> Sparse (BM25/TF-IDF): vector phần lớn = 0, mỗi chiều ứng với một từ trong vocab. Dense (embedding): mọi chiều có giá trị, mã hoá ngữ nghĩa nén.

---

## 8. Bài tập tự luyện

- [ ] **Bài 1:** Cài BM25 (rank_bm25) + một vector retriever; chạy cùng query "Error 503" → so sánh top-5 của từng bên.
- [ ] **Bài 2:** Tự code RRF gộp 2 list rank với k=60; thử k=10 và k=200 xem thứ hạng đổi thế nào.
- [ ] **Bài 3:** Tìm 3 loại truy vấn mà BM25 thắng vector, và 3 loại ngược lại.

---

## 9. Liên kết

- [[01-Advanced-Indexing]] — index tốt (Semantic Chunking + HNSW) trước khi tối ưu search
- [[03-Query-Transformation]] — cải thiện chính câu hỏi (HyDE, decomposition)
- [[04-Post-Retrieval-Processing]] — sau khi gộp, re-rank bằng Cross-Encoder
- [[../01-AI-Fundamentals-RAG/03-Modern-RAG-Architecture]] — Hybrid/RRF trong phase Retrieval
- [[00-MOC-RAG-Optimization|MOC: RAG & Optimization]]
