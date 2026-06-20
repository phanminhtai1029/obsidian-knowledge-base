---
title: "Advanced Indexing"
section: 04-AI/02-RAG-Optimization
tags: [ai, rag, indexing, semantic-chunking, hnsw, ann, vector-db, fresher]
related:
  - "[[02-Hybrid-Search]]"
  - "[[../01-AI-Fundamentals-RAG/03-Modern-RAG-Architecture]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 35m
source: [QN26_FR_AI_01, "01-advanced-indexing.mdx"]
---

# Advanced Indexing

> [!summary] TL;DR
> Indexing "ngây thơ" = **Fixed-size Chunking** (cắt cứng mỗi 500–1000 ký tự) + **Flat Index** (so sánh brute-force với mọi vector). Khi scale lên thì gặp 2 vấn đề: **mất ngữ nghĩa** (chunk gãy ý) và **latency cao**. Hai kỹ thuật khắc phục: **Semantic Chunking** (cắt theo điểm thay đổi chủ đề, dựa trên độ tương đồng vector giữa các câu) và **HNSW** (graph nhiều tầng cho ANN search). Ba tham số HNSW phải thuộc: **`M`** (số link/node), **`ef_construction`** (độ sâu lúc build), **`ef_search`** (độ sâu lúc query — đòn bẩy trade-off tốc độ ↔ độ chính xác).

---

## 1. Vì sao Indexing cơ bản không đủ?

Cách tiếp cận đơn giản ban đầu gồm 2 thành phần:

| Thành phần | Cách làm | Vấn đề khi scale |
|---|---|---|
| **Fixed-size Chunking** | Cắt văn bản mỗi 500/1000 ký tự, bất kể câu/ý đã trọn vẹn chưa | **Loss of Semantics** — một ý hoàn chỉnh bị xé làm 2 chunk vô nghĩa → LLM mất ngữ cảnh |
| **Flat Indexing** | Lưu mọi vector vào một danh sách dài, search = so sánh với **từng** vector (brute-force) | **High Latency** — quét tuần tự hàng triệu vector quá chậm cho real-time |

> Chỉ ổn với vài trăm tài liệu. Khi hệ thống lớn lên gấp nhiều lần, cả chất lượng (chunking) lẫn tốc độ (flat index) đều sụp.

```
★ Insight ─────────────────────────────────────
• Đây là 2 trục tối ưu KHÁC NHAU: Semantic Chunking lo CHẤT LƯỢNG dữ liệu đầu
  vào (cắt khéo), HNSW lo TỐC ĐỘ tìm kiếm (search nhanh). Phỏng vấn hay gộp
  chung "tối ưu indexing" — bạn nên tách rạch ròi 2 mục tiêu này.
• Nguyên tắc nền: "garbage in, garbage out" — chunk xấu thì retrieval và
  generation đều hỏng, dù vector DB nhanh đến đâu.
─────────────────────────────────────────────────
```

---

## 2. Semantic Chunking — cắt theo ý nghĩa

**Mục tiêu:** chunk **đủ nhỏ** để search chính xác, nhưng **đủ lớn** để bao trọn một ý.

### Ý tưởng cốt lõi
- Các câu liên tiếp **cùng chủ đề** → vector của chúng **gần nhau** trong không gian biểu diễn.
- Khi nội dung **chuyển sang chủ đề mới** → vector **đổi hướng đột ngột**, tạo khoảng cách lớn.
- → Semantic Chunking phát hiện điểm thay đổi này để **cắt đúng chỗ giao nhau của 2 chủ đề**.

### Quy trình triển khai
1. **Sentence Splitting** — tách văn bản thành câu hoàn chỉnh theo dấu câu.
2. **Similarity Calculation** — tính độ tương đồng (vd cosine) giữa câu hiện tại và câu kế tiếp.
3. **Thresholding**:
   - Similarity **cao** → cùng chủ đề → gộp vào cùng chunk.
   - Similarity **tụt mạnh dưới ngưỡng** → đổi chủ đề → cắt chunk tại đây, bắt đầu chunk mới.

### Recursive vs Semantic Chunking

**Ví dụ:** một đoạn có 2 luồng thông tin tách biệt — phần đầu nói về phần cứng (GPU NVIDIA H100), phần sau nói về mô hình ngôn ngữ (Llama-3).

- **Recursive (cắt theo dấu câu)** → tách thành 4 chunk nhỏ. Hậu quả: chunk chứa "giúp giảm thời gian training" bị tách khỏi chủ ngữ "NVIDIA H100" → nếu chỉ tìm thấy chunk đó, LLM không biết đang nói về cái gì.
- **Semantic** → phát hiện điểm gãy ngữ nghĩa, cho ra **2 chunk trọn vẹn**: 1 chunk đủ về H100, 1 chunk đủ về Llama-3.

| Tiêu chí | **Recursive Chunking** | **Semantic Chunking** |
|---|---|---|
| Nguyên lý | Cắt theo dấu câu (xuống dòng, dấu chấm) + số ký tự cố định | Cắt theo **thay đổi ý nghĩa** giữa các câu |
| Ưu điểm | Cực nhanh, rẻ; giữ cấu trúc gốc | Giữ trọn ý, bám flow văn bản; tăng độ chính xác search |
| Nhược điểm | Dễ cắt ngang ý quan trọng; context nhiễu (2 nửa của 2 ý) | Tốn tài nguyên tính toán (chạy model so từng câu) |
| Hợp với | Tài liệu **cấu trúc rõ**: hợp đồng, luật, giáo trình | Tài liệu **ít cấu trúc**: biên bản họp, transcript video |

---

## 3. HNSW — Hierarchical Navigable Small World

Brute-force (tính khoảng cách tới **mọi** điểm) bất khả thi về thời gian khi dữ liệu lớn. Giải pháp chuẩn: **ANN (Approximate Nearest Neighbor)** — chấp nhận "gần đúng" để đổi lấy tốc độ. **HNSW** là thuật toán ANN phổ biến nhất, cân bằng tốt giữa tốc độ và độ chính xác.

### Cơ chế: đồ thị nhiều tầng
- **Layer 0** — chứa **toàn bộ** điểm dữ liệu, link chi tiết nhất. Tầng tìm kiếm chính xác cuối cùng.
- **Tầng trên** — bản rút gọn của Layer 0: node thưa hơn, link dài hơn, đóng vai **"đường tắt" (shortcut)** giúp nhảy nhanh qua không gian lớn.

**Quy trình search:**
1. Bắt đầu từ entry point ở **tầng cao nhất**.
2. Mỗi tầng, di chuyển tới neighbor gần query nhất đến khi chạm cực trị cục bộ của tầng đó.
3. Node tìm được trở thành entry point cho tầng dưới.
4. Lặp đến **Layer 0** → tìm cục bộ chính xác → trả kết quả cuối.

```text
   Tầng cao  ●───────────────●          (thưa, link dài → nhảy nhanh)
              \             /
   Tầng giữa   ●──●─────●──●             (dày hơn)
                \  \    /  /
   Layer 0   ●─●─●─●─●─●─●─●─●─●         (đủ điểm, link ngắn → chính xác)
   query ──► bắt đầu trên cùng, "trượt" dần xuống dưới
```

### Ba tham số phải thuộc

| Tham số | Là gì | Tăng giá trị → tác động |
|---|---|---|
| **`M`** (Max links/node) | Số link tối đa 1 node nối với neighbor → quyết định **mật độ đồ thị** | Đồ thị dày hơn, ít kẹt cực trị cục bộ → **chính xác hơn**. Đổi lại **tốn RAM** & thêm data chậm hơn |
| **`ef_construction`** (độ sâu lúc build) | Số ứng viên xét khi thêm node mới vào index | Xét nhiều ứng viên hơn → đồ thị **chất lượng cao**, search sau mượt. Đổi lại **load data chậm** |
| **`ef_search`** (độ sâu lúc query) | Số neighbor giữ trong priority queue **khi truy vấn** | ⭐ Đòn bẩy trực tiếp **tốc độ ↔ chính xác**: tăng → quét rộng, không bỏ sót, nhưng chậm; giảm → cực nhanh, nhưng dễ bỏ sót |

```
★ Insight ─────────────────────────────────────
• Phân biệt build-time vs query-time: `M` và `ef_construction` cố định khi DỰNG
  index (đổi phải re-index). `ef_search` chỉnh được LÚC CHẠY mỗi query → đây là
  núm vặn linh hoạt nhất để tune theo SLA.
• "Cực trị cục bộ" (local extrema): greedy search có thể kẹt ở một vùng không
  phải đáp án toàn cục. Tăng `M`/`ef` = cho thuật toán nhiều đường thoát hơn.
─────────────────────────────────────────────────
```

### Chiến lược cấu hình thực tế

> [!tip] Không có bộ tham số hoàn hảo cho mọi bài toán — tune theo yêu cầu nghiệp vụ
> **Case 1 — Chatbot real-time:** cần phản hồi nhanh, chấp nhận sai số nhỏ → giữ `ef_search` **thấp (50–100)** để tối ưu latency.
> **Case 2 — Hệ thống tra cứu chuyên sâu:** cần chính xác tuyệt đối, không bỏ sót → `M` **cao (64)** + `ef_search` **cao (400–500)**. Chấp nhận tốn RAM & chậm hơn để đạt độ chính xác tối đa.

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Semantic Chunking không phải lúc nào cũng tốt hơn
> Với tài liệu cấu trúc rõ (luật, hợp đồng), Recursive thường đủ và rẻ hơn nhiều. Đừng "over-engineer" mọi pipeline bằng Semantic Chunking — nó tốn 1 lượt chạy model cho từng cặp câu.

> [!warning] Nhầm HNSW là "tìm chính xác"
> HNSW là **ANN — gần đúng**. Nó đánh đổi một chút recall lấy tốc độ. Nếu bài toán cần đúng 100% (vd đối soát tài chính), phải tăng `ef_search` cao hoặc cân nhắc index khác.

> [!warning] `M`/`ef_construction` đặt thấp rồi mới phát hiện thiếu chính xác
> Hai tham số này gắn với index đã build → sửa = **re-index toàn bộ** (tốn thời gian). Cân nhắc kỹ từ đầu cho dữ liệu lớn.

---

## 5. Câu hỏi phỏng vấn thường gặp

**Q1: Vì sao cần Semantic Chunking thay vì cắt theo số ký tự?**
> Cắt cố định dễ xé một ý hoàn chỉnh thành 2 chunk vô nghĩa, làm LLM mất chủ ngữ/ngữ cảnh. Semantic Chunking cắt tại điểm vector giữa các câu đổi hướng đột ngột (đổi chủ đề) → mỗi chunk bao trọn một ý.

**Q2: HNSW giải quyết vấn đề gì, theo cơ chế nào?**
> Giải bài toán latency của brute-force trên hàng triệu vector. Cơ chế: đồ thị nhiều tầng — tầng trên thưa làm "đường tắt" nhảy nhanh, tầng dưới (Layer 0) đủ điểm để tìm chính xác. Đây là một thuật toán ANN.

**Q3: Phân biệt `ef_construction` và `ef_search`?**
> `ef_construction` áp dụng **lúc build index** (quyết định chất lượng đồ thị, đổi phải re-index). `ef_search` áp dụng **lúc query** (chỉnh runtime), là đòn bẩy trực tiếp đánh đổi tốc độ ↔ độ chính xác.

**Q4: `M` cao thì sao?**
> Đồ thị dày hơn → chính xác hơn, ít kẹt cực trị cục bộ; nhưng tốn RAM và thêm data chậm hơn.

**Q5: Khi nào cấu hình `ef_search` thấp, khi nào cao?**
> Thấp (50–100) cho chatbot real-time cần nhanh, chấp nhận sai số. Cao (400–500) cho hệ thống tra cứu cần chính xác tuyệt đối.

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Build một Semantic Chunker: tách câu → tính cosine giữa các câu liên tiếp → cắt khi tụt dưới ngưỡng. So sánh số chunk & chất lượng với Recursive trên cùng văn bản 2 chủ đề.
- [ ] **Bài 2:** Dựng vector DB (Chroma/Qdrant) với HNSW; chạy cùng query với `ef_search` = 50, 100, 400 → đo latency và recall.
- [ ] **Bài 3:** Giải thích bằng lời tại sao tăng `M` lại tốn RAM (gợi ý: số link lưu cho mỗi node).

---

## 7. Liên kết

- [[02-Hybrid-Search]] — sau khi index tốt, tối ưu cách **tìm** (Sparse + Dense + RRF)
- [[04-Post-Retrieval-Processing]] — Bi-Encoder vs Cross-Encoder & funnel
- [[../01-AI-Fundamentals-RAG/03-Modern-RAG-Architecture]] — 3 phase tổng thể; ANN/HNSW trong bức tranh lớn
- [[../01-AI-Fundamentals-RAG/02-RAG-Theoretical-Foundations#Glossary mở rộng (tra cứu nhanh — gom từ cả 4 môn AI)|Glossary mở rộng]]
- [[00-MOC-RAG-Optimization|MOC: RAG & Optimization]]
