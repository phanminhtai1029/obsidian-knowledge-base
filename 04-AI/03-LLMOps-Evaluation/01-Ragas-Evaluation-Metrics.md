---
title: "Ragas Evaluation Metrics"
section: 04-AI/03-LLMOps-Evaluation
tags: [ai, llmops, ragas, evaluation, faithfulness, answer-relevancy, context-precision, context-recall, fresher]
related:
  - "[[02-Observability-LangFuse-LangSmith]]"
  - "[[03-Experiment-Comparison]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 35m
source: [QN26_FR_AI_01, "01-ragas-evaluation-metrics.mdx"]
---

# Ragas Evaluation Metrics

> [!summary] TL;DR
> Không đo được thì không tối ưu được. **Ragas** (Retrieval Augmented Generation **As**sessment) là framework **tự động** đánh giá RAG, dùng **LLM làm giám khảo** (LLM-as-a-Judge) nên **không cần ground-truth do người gán** cho phần lớn metric. 4 metric chia 2 nhóm: **Generation** (chất lượng sinh) = **Faithfulness** (bám context, chống hallucination) + **Answer Relevancy** (trả đúng câu hỏi); **Retrieval** (chất lượng tìm) = **Context Precision** (xếp hạng — đoạn liên quan có lên đầu không) + **Context Recall** (độ phủ — có tìm đủ thông tin không). Mọi điểm ∈ **[0, 1]**, càng cao càng tốt.

---

## 1. Vì sao cần Ragas?

Sau khi xây (Môn 1) và tối ưu (Môn 2) RAG, câu hỏi sống còn là: **bản mới có thực sự tốt hơn bản cũ không?** Đánh giá thủ công (người đọc từng câu trả lời) **không scale** — tốn người, chậm, chủ quan.

Cách đánh giá truyền thống của NLP (BLEU, ROUGE) so khớp **từ ngữ bề mặt** với một đáp án mẫu → không đo được "đúng ý nhưng diễn đạt khác". Ragas giải quyết bằng cách dùng **LLM mạnh (vd GPT-4) làm giám khảo**, đánh giá theo **ngữ nghĩa**, đa chiều.

```
★ Insight ─────────────────────────────────────
• "Ragas" = Retrieval Augmented Generation Assessment — bài báo gốc "Ragas:
  Automated Evaluation of Retrieval Augmented Generation" (2024).
• Triết lý cốt lõi: REFERENCE-FREE. Đa số metric KHÔNG cần đáp án chuẩn do
  người viết — LLM tự sinh câu hỏi/claim để đối chiếu. Riêng Context Recall thì
  CẦN reference answer (đáp án mẫu). Nhớ ngoại lệ này để không trả lời sai khi
  recruiter hỏi "metric nào cần ground truth?".
─────────────────────────────────────────────────
```

---

## 2. ⭐ Bức tranh tổng: RAG Triad & 2 trục đánh giá

Một câu trả lời RAG có 3 thành phần: **Question** (hỏi) — **Context** (đoạn tìm được) — **Answer** (đáp). 4 metric của Ragas đo 4 "cạnh" của tam giác này:

| Metric | Đo quan hệ giữa | Nhóm | Trả lời câu hỏi |
|---|---|---|---|
| **Faithfulness** | Answer ↔ Context | Generation | "Đáp có **bịa** ngoài context không?" |
| **Answer Relevancy** | Answer ↔ Question | Generation | "Đáp có **đúng trọng tâm** câu hỏi không?" |
| **Context Precision** | Context ↔ Question (thứ hạng) | Retrieval | "Đoạn liên quan có **lên top** không?" |
| **Context Recall** | Context ↔ Ground-truth | Retrieval | "Có **tìm đủ** thông tin cần không?" |

> [!note] Tách bạch lỗi: Generation hay Retrieval?
> Sức mạnh lớn nhất của Ragas là **chỉ ra hỏng ở đâu**. Faithfulness/Answer Relevancy thấp → lỗi ở **LLM sinh** (prompt, model). Context Precision/Recall thấp → lỗi ở **retriever** (chunking, embedding, search). Không có việc "RAG dở chung chung" — luôn quy được về một tầng.

---

## 3. Faithfulness — độ trung thành (chống hallucination)

**Định nghĩa:** đáp án có bao nhiêu phần được **chứng minh bằng context** đã tìm. Một câu trả lời "faithful" khi **mọi khẳng định (claim)** trong nó suy ra được từ context.

**Quy trình 3 bước:**
1. **Decomposition** — LLM tách câu trả lời thành các **claim** (mệnh đề) riêng lẻ.
2. **Verification** — kiểm tra **từng claim** có suy ra được từ context không.
3. **Scoring** — tỉ lệ claim được chứng minh / tổng claim.

$$ \text{Faithfulness} = \frac{\text{số claim được context hỗ trợ}}{\text{tổng số claim trong câu trả lời}} $$

> [!example] Ví dụ Faithfulness = 0.5
> **Hỏi:** "Einstein sinh ở đâu và khi nào?"
> **Context:** "Albert Einstein (sinh **14/3/1879**) là nhà vật lý lý thuyết **gốc Đức**..."
> **Đáp:** "Einstein sinh ở **Đức** ngày **20/3/1879**."
> - Claim 1: "Einstein sinh ở Đức." → ✅ Đúng (suy ra từ "gốc Đức").
> - Claim 2: "Einstein sinh ngày 20/3/1879." → ❌ Sai (context nói 14/3, không phải 20/3).
> → **Faithfulness = 1/2 = 0.5**.

> [!warning] Faithfulness ≠ Answer Correctness (bẫy kinh điển)
> Faithfulness chỉ hỏi "đáp có **bám context** không", **KHÔNG** hỏi "context có **đúng sự thật** không". Nếu retriever lấy về context sai mà LLM trung thành lặp lại → **Faithfulness vẫn = 1.0** dù câu trả lời sai bét. Đo tính đúng đối chiếu thực tế là việc của **Answer Correctness** (đối chiếu ground-truth), không phải Faithfulness.

---

## 4. Answer Relevancy — độ liên quan của câu trả lời

**Định nghĩa:** câu trả lời có **đúng trọng tâm** câu hỏi không. **Không** đánh giá đúng/sai sự thật, mà đánh giá **đầy đủ & không lan man** (thiếu thông tin hoặc thừa thông tin đều bị trừ điểm).

**Quy trình (reverse-engineer rất "mẹo"):**
1. **Reverse-engineer** — từ câu **trả lời**, bảo LLM sinh ngược ra $N$ câu **hỏi** mà câu trả lời đó có thể đáp.
2. **Embedding** — embed câu hỏi gốc và $N$ câu hỏi sinh ra.
3. **Similarity** — tính **cosine similarity trung bình** giữa câu hỏi gốc và các câu hỏi sinh ra.

$$ \text{Answer Relevancy} = \frac{1}{N}\sum_{i=1}^{N} \cos(E_{g_i}, E_o) $$

trong đó $E_{g_i}$ là embedding của câu hỏi sinh thứ $i$, $E_o$ là embedding câu hỏi gốc.

> [!example] Ví dụ Answer Relevancy
> **Hỏi:** "Nước Pháp nằm ở đâu và thủ đô là gì?"
> - **Đáp thiếu:** "Pháp nằm ở Tây Âu." → LLM sinh ngược ra "Pháp nằm ở đâu?" — **lệch** câu hỏi gốc (thiếu vế thủ đô) → similarity thấp.
> - **Đáp đủ:** "Pháp ở Tây Âu và Paris là thủ đô." → LLM sinh ngược ra đúng "Pháp ở đâu và thủ đô là gì?" → similarity ≈ 1.

```
★ Insight ─────────────────────────────────────
• Vì sao "sinh ngược câu hỏi"? Nếu một câu trả lời TỐT, ta phải tái tạo lại được
  ĐÚNG câu hỏi gốc từ nó. Đáp lan man/thiếu sẽ kéo các câu hỏi sinh ra đi lệch
  → cosine tụt. Đây là cách đo "liên quan" mà không cần đáp án mẫu.
• Answer Relevancy bị PHẠT cả khi câu trả lời THỪA thông tin lạc đề, không chỉ
  thiếu — vì câu thừa khiến LLM sinh ra câu hỏi không khớp.
─────────────────────────────────────────────────
```

---

## 5. Context Precision — độ chính xác xếp hạng retrieval

**Định nghĩa:** trong danh sách context tìm được, các đoạn **liên quan có được xếp hạng cao** (lên đầu) không. Đây là metric đo **thứ tự**, không chỉ đo "có hay không".

**Quy trình:**
1. **Determine Relevance** — LLM đánh dấu mỗi context là liên quan ($v_k=1$) hay không ($v_k=0$).
2. **Precision@k** — tại mỗi vị trí $k$, tính tỉ lệ context liên quan trong top-$k$.
3. **Weighted Average** — trung bình có trọng số của Precision@k, **chỉ cộng tại các vị trí có context liên quan**.

$$ \text{Context Precision} = \frac{\sum_{k=1}^{K} \big(\text{Precision@}k \times v_k\big)}{\text{tổng số context liên quan}} $$

> [!example] Ví dụ Context Precision ≈ 0.76
> **Hỏi:** "Lợi ích sức khoẻ của trà xanh?" — 5 context theo thứ tự:
> | k | Context | Liên quan? | Precision@k |
> |---|---|---|---|
> | 1 | Chống oxy hoá, giảm nguy cơ ung thư | ✅ $v_1{=}1$ | 1/1 = 1.0 |
> | 2 | Đồn điền trà phổ biến ở châu Á | ❌ $v_2{=}0$ | 1/2 = 0.5 |
> | 3 | Tăng trao đổi chất, giảm cân | ✅ $v_3{=}1$ | 2/3 ≈ 0.67 |
> | 4 | Lịch sử trà có từ ngàn năm | ❌ $v_4{=}0$ | 2/4 = 0.5 |
> | 5 | Cải thiện chức năng não | ✅ $v_5{=}1$ | 3/5 = 0.6 |
>
> Chỉ cộng các vị trí liên quan (k=1,3,5):
> Context Precision = (1.0 + 0.67 + 0.6) / 3 ≈ **0.76**. Điểm phản ánh việc có context vô dụng **chen vào giữa** các context tốt.

---

## 6. Context Recall — độ phủ retrieval

**Định nghĩa:** thông tin cần thiết trong **reference answer** (đáp án mẫu) có bao nhiêu phần **tìm thấy được** trong context. Đo retriever có **bỏ sót** thông tin không.

**Quy trình:**
1. **Decomposition** — tách reference answer thành các claim.
2. **Attribution** — LLM kiểm tra mỗi claim có suy ra được từ context tìm được không.
3. **Ratio** — tỉ lệ claim được context hỗ trợ / tổng claim của reference.

$$ \text{Context Recall} = \frac{\text{số claim của reference được context hỗ trợ}}{\text{tổng số claim của reference}} $$

> [!example] Ví dụ Context Recall = 0
> **Hỏi:** "Tháp Eiffel ở đâu?" — **Reference:** "Tháp Eiffel nằm ở Paris."
> **Context tìm được:** "Paris là thủ đô của Pháp."
> → Context **không hề nhắc** vị trí tháp Eiffel → claim không suy ra được → **Recall = 0/1 = 0**. Retriever đã **trượt** đoạn chứa thông tin cần.

> [!warning] Context Recall là metric DUY NHẤT cần reference answer
> 3 metric kia reference-free. Recall cần đáp án mẫu do người gán làm gốc đối chiếu "đã tìm đủ chưa". Đây là chi phí "ground-truth" của Ragas — và là điểm recruiter hay hỏi để phân biệt.

---

## 7. Precision vs Recall — cặp khái niệm phải phân biệt

| | **Context Precision** | **Context Recall** |
|---|---|---|
| Câu hỏi cốt lõi | Thứ tìm được có **sạch & xếp đúng** không? | Có **bỏ sót** gì không? |
| Phạt khi | Context rác chen lên top | Thiếu context chứa thông tin cần |
| Cần ground-truth? | Không (LLM tự chấm liên quan) | **Có** (reference answer) |
| Cải thiện bằng | Re-ranking, Cross-Encoder (Môn 2) | Tăng top-K, hybrid search, chunking tốt hơn |

```
★ Insight ─────────────────────────────────────
• Precision và Recall thường ĐÁNH ĐỔI: tăng top-K → Recall tăng (vơ được nhiều
  hơn) nhưng Precision dễ tụt (kéo theo rác). Re-ranking (Môn 2) là cách hiếm
  hoi cải thiện CẢ HAI: lấy top-K lớn cho Recall rồi Cross-Encoder lọc cho
  Precision. → Đây là cầu nối trực tiếp giữa Môn 2 và Môn 3.
─────────────────────────────────────────────────
```

---

## 8. Pitfalls / Bẫy thường gặp

> [!warning] Tin tuyệt đối điểm LLM-as-a-Judge
> Giám khảo cũng là LLM → có **variance** (chạy lại điểm hơi khác), có bias (thiên vị câu dài, văn phong nịnh). Nên chạy nhiều lần lấy trung bình, và dùng model mạnh làm judge.

> [!warning] Nhầm Faithfulness với "câu trả lời đúng"
> Như §3: Faithful = bám context, không phải đúng sự thật. Cặp đôi cần nhìn cùng nhau: Faithfulness (đáp vs context) + Context Recall/Precision (context vs thực tế cần).

> [!warning] Quên Context Recall cần reference
> Khi dựng pipeline đánh giá, nếu dataset không có `ground_truth`, Context Recall sẽ không tính được. Chuẩn bị đáp án mẫu cho tập test ngay từ đầu.

> [!warning] Tốn token/chi phí
> Mỗi metric gọi LLM nhiều lần (tách claim, sinh câu hỏi, chấm liên quan). Đánh giá 200 câu × 4 metric = rất nhiều lần gọi → cân nhắc dùng model rẻ hơn cho judge khi cần.

---

## 9. Câu hỏi phỏng vấn thường gặp

**Q1: Ragas là gì, vì sao không dùng BLEU/ROUGE?**
> Ragas là framework tự động đánh giá RAG dùng LLM-as-a-Judge, đánh giá theo ngữ nghĩa và đa chiều (4 metric), phần lớn reference-free. BLEU/ROUGE chỉ so khớp từ ngữ bề mặt với một đáp án mẫu → không bắt được "đúng ý, khác chữ" và không tách được lỗi retrieval vs generation.

**Q2: 4 metric của Ragas là gì, chia nhóm thế nào?**
> Generation: Faithfulness (đáp bám context, chống hallucination) + Answer Relevancy (đáp đúng trọng tâm câu hỏi). Retrieval: Context Precision (đoạn liên quan có lên top) + Context Recall (có tìm đủ thông tin). Mọi điểm ∈ [0,1].

**Q3: Faithfulness khác Answer Correctness ở đâu?**
> Faithfulness đo đáp có suy ra từ context không (bám nguồn). Answer Correctness đo đáp có khớp ground-truth (đúng sự thật). LLM trung thành lặp lại context sai vẫn Faithfulness = 1 nhưng Correctness thấp.

**Q4: Answer Relevancy tính bằng cách nào?**
> "Reverse-engineer": từ câu trả lời, LLM sinh ngược N câu hỏi, embed chúng và câu hỏi gốc, lấy cosine similarity trung bình. Đáp tốt thì tái tạo đúng câu hỏi gốc → similarity cao; đáp thiếu/thừa thì lệch → thấp.

**Q5: Metric nào cần ground-truth (reference answer)?**
> Chỉ **Context Recall**. 3 metric còn lại reference-free (LLM tự sinh claim/câu hỏi để đối chiếu).

**Q6: Context Precision thấp thì sửa ở đâu, Context Recall thấp thì sửa ở đâu?**
> Precision thấp → context rác lên top → thêm re-ranking/Cross-Encoder (Môn 2). Recall thấp → bỏ sót → tăng top-K, hybrid search, cải thiện chunking/embedding.

---

## 10. Bài tập tự luyện

- [ ] **Bài 1:** Cài `ragas`, dựng dataset nhỏ (question, answer, contexts, ground_truth) và chạy cả 4 metric; đọc hiểu output.
- [ ] **Bài 2:** Cố tình tạo 1 câu trả lời "trung thành với context sai" → quan sát Faithfulness cao nhưng Answer Correctness thấp.
- [ ] **Bài 3:** Giảm top-K từ 5 xuống 2 → quan sát Context Recall tụt; tăng lên 10 → quan sát Precision có tụt không.
- [ ] **Bài 4:** Tự tính tay Context Precision cho một danh sách 5 context (như ví dụ §5) rồi đối chiếu với Ragas.

---

## 11. Liên kết

- [[02-Observability-LangFuse-LangSmith]] — nơi log lại điểm Ragas theo từng trace
- [[03-Experiment-Comparison]] — dùng 4 metric này so sánh Naive/Graph/Hybrid
- [[../02-RAG-Optimization/04-Post-Retrieval-Processing]] — re-ranking cải thiện Context Precision & Recall
- [[../01-AI-Fundamentals-RAG/02-RAG-Theoretical-Foundations#Glossary mở rộng (tra cứu nhanh — gom từ cả 4 môn AI)|Glossary mở rộng]]
- [[00-MOC-LLMOps-Evaluation|MOC: LLMOps & Evaluation]]
