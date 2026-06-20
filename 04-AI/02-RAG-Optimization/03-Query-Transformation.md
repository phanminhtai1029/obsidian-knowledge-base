---
title: "Query Transformation"
section: 04-AI/02-RAG-Optimization
tags: [ai, rag, query-transformation, hyde, query-decomposition, multi-query, fresher]
related:
  - "[[02-Hybrid-Search]]"
  - "[[04-Post-Retrieval-Processing]]"
difficulty: ⭐⭐⭐
estimated_time: 25m
source: [QN26_FR_AI_01, "03-query-transformation.mdx"]
---

# Query Transformation

> [!summary] TL;DR
> Câu hỏi thực tế thường **ngắn, thiếu ngữ cảnh, hỏi nhiều ý cùng lúc** → vector của câu hỏi không khớp vector tài liệu. **Query Transformation** dùng LLM **viết lại / mở rộng / phân rã** câu hỏi trước khi search. Hai kỹ thuật chính: **HyDE** (sinh một câu trả lời *giả định* rồi dùng vector của nó để tìm — khắc phục **bất đối xứng ngữ nghĩa** giữa hỏi ngắn và đáp dài) và **Query Decomposition** (tách câu hỏi đa ý thành các sub-question đơn ý, search riêng từng cái rồi tổng hợp).

---

## 1. Vấn đề: câu hỏi của user không hoàn hảo

Giả định "câu hỏi luôn rõ ràng, đủ nghĩa, khớp tài liệu" hiếm khi đúng. User hay gõ cụt lủn: thay vì "Quy định làm việc từ xa áp dụng từ ngày nào?" họ chỉ gõ **"remote work"**. Nhồi câu thô này vào RAG → kết quả tệ vì vector câu hỏi không khớp vector văn bản pháp lý chi tiết.

→ **Query Transformation = "biên tập viên thông minh"**: chỉnh và định hướng lại câu hỏi trước khi gửi xuống bộ tra cứu.

```
★ Insight ─────────────────────────────────────
• Đây là tối ưu ở PHÍA TRUY VẤN (pre-retrieval), khác với re-ranking là tối ưu
  PHÍA KẾT QUẢ (post-retrieval). Một pipeline mạnh tối ưu cả hai đầu.
• Đánh đổi: mỗi transform = thêm 1+ lần gọi LLM → tăng latency & chi phí. Dùng
  khi chất lượng retrieval đáng giá hơn độ trễ thêm vào.
─────────────────────────────────────────────────
```

---

## 2. HyDE — Hypothetical Document Embeddings

**Vấn đề gốc:** *bất đối xứng ngữ nghĩa* — câu hỏi thường **ngắn, dạng nghi vấn**; tài liệu thì **dài, dạng khẳng định/mô tả**. Vector của 2 dạng này không gần nhau.

**Ý tưởng HyDE:** thay vì search bằng câu hỏi, bảo LLM viết một **câu trả lời giả định**, rồi dùng vector của câu trả lời đó để tìm tài liệu thật.

**Cơ chế 3 bước:**
1. **Generate** — LLM viết một đoạn trả lời giả định. *Thông tin có thể SAI sự thật*, nhưng **văn phong & từ vựng chuyên ngành** giống tài liệu thật.
2. **Encode** — đưa đoạn giả định qua embedding model → tạo vector.
3. **Retrieve** — dùng vector "câu trả lời giả" này để search. Vì nó gần vector "câu trả lời thật" hơn là vector "câu hỏi" → kết quả chính xác hơn.

> [!example] Ví dụ HyDE
> **Câu hỏi:** "Cách xử lý lỗi màn hình xanh." (quá ngắn → vector dễ khớp nhầm tài liệu về *màu sắc* màn hình)
> **HyDE sinh:** "Để sửa lỗi Blue Screen of Death (BSOD) trên Windows, bạn cần khởi động lại máy, kiểm tra stop code, cập nhật driver card đồ hoạ, hoặc vào Safe Mode để gỡ phần mềm xung đột..."
> **Kết quả:** nhờ các từ kỹ thuật "BSOD", "driver", "Safe Mode" trong bản nháp, hệ thống dễ dàng tìm đúng tài liệu hướng dẫn kỹ thuật.

> [!note] HyDE chấp nhận "ảo giác có kiểm soát"
> Điểm tinh tế: ta cố ý để LLM **bịa** một đáp án — nhưng chỉ dùng nó làm **vector tìm kiếm**, không đưa cho user. Sai sự thật không sao, miễn **văn phong/từ khoá** đúng domain để kéo vector về đúng vùng.

---

## 3. Query Decomposition — phân rã câu hỏi

Hữu ích cho **câu hỏi phức tạp** mà một đoạn văn bản không chứa đủ thông tin để trả lời. Nếu câu hỏi cần **so sánh/tổng hợp** từ nhiều nguồn, vector câu hỏi sẽ "lửng lơ giữa các chủ đề" → search trượt.

**Chiến lược 3 bước:**
1. **Breakdown** — LLM tách câu đa-ý (multi-intent) thành chuỗi sub-question đơn-ý (single-intent) độc lập.
2. **Retrieval** — search riêng cho **từng** sub-question → mỗi lần tìm có mục tiêu rõ, chính xác cao.
3. **Synthesis** — gộp các đoạn tìm được, đưa LLM trả lời **câu hỏi gốc**.

> [!example] Ví dụ Query Decomposition
> **Câu hỏi:** "So sánh doanh thu iPhone 15 và Samsung S24 trong Q1 2024." (không tài liệu nào chứa sẵn bảng so sánh này)
> - **Sub-query 1:** "Doanh thu iPhone 15 Q1 2024?" → tìm trong báo cáo Apple.
> - **Sub-query 2:** "Doanh thu Samsung S24 Q1 2024?" → tìm trong báo cáo Samsung.
> **Final Generation:** LLM nhận cả 2 con số → tự tổng hợp thành câu trả lời so sánh hoàn chỉnh.

---

## 4. Thuật ngữ liên quan (mở rộng — hay hỏi thêm)

| Kỹ thuật                | Bản chất                                                                        | Khi dùng                               |
| ----------------------- | ------------------------------------------------------------------------------- | -------------------------------------- |
| **HyDE**                | Sinh đáp án giả → search bằng vector của nó                                     | Bất đối xứng hỏi ngắn / đáp dài        |
| **Query Decomposition** | Tách 1 câu nhiều ý thành nhiều sub-question                                     | Câu hỏi so sánh / tổng hợp nhiều nguồn |
| **Multi-Query**         | LLM sinh **nhiều câu diễn đạt** của *cùng* câu hỏi, search hết rồi gộp (vd RRF) | Câu hỏi mơ hồ, nhiều cách diễn đạt     |
| **Step-back Prompting** | Lùi về câu hỏi **khái quát hơn** để lấy ngữ cảnh nền rồi mới trả lời cụ thể     | Câu hỏi chi tiết cần kiến thức nền     |

```
★ Insight ─────────────────────────────────────
• Đừng nhầm Multi-Query với Query Decomposition: Multi-Query sinh nhiều cách
  HỎI CÙNG MỘT Ý (paraphrase); Decomposition tách thành nhiều Ý KHÁC NHAU
  (sub-problems). Mục tiêu khác nhau: tăng recall vs xử lý câu nhiều ý.
─────────────────────────────────────────────────
```

---

## 5. Pitfalls / Bẫy thường gặp

> [!warning] HyDE có thể "kéo lệch" nếu domain quá lạ
> Nếu LLM không có kiến thức nền về lĩnh vực, đáp án giả định có thể sai cả văn phong/từ khoá → vector lệch hẳn. HyDE mạnh nhất khi LLM ít nhất "nói đúng giọng" domain.

> [!warning] Decomposition làm tăng số lần search & gọi LLM
> Mỗi sub-question = thêm 1 lượt retrieval (+ 1 lượt LLM để tách, + 1 để tổng hợp). Latency và chi phí tăng theo số sub-question.

> [!warning] Tách sai → tổng hợp sai
> Nếu LLM phân rã thiếu/sai sub-question, bước Synthesis dù tốt cũng không cứu được. Chất lượng phụ thuộc bước Breakdown.

---

## 6. Câu hỏi phỏng vấn thường gặp

**Q1: HyDE giải quyết vấn đề gì và hoạt động ra sao?**
> Giải bất đối xứng ngữ nghĩa giữa câu hỏi ngắn và tài liệu dài. Cơ chế: LLM sinh một đáp án giả định (có thể sai sự thật nhưng đúng văn phong/từ khoá domain) → embed nó → dùng vector đó search, vì nó gần vector đáp án thật hơn vector câu hỏi.

**Q2: Vì sao đáp án giả định của HyDE sai sự thật vẫn dùng được?**
> Vì ta chỉ dùng nó làm vector tìm kiếm, không trả cho user. Cái cần là từ vựng/văn phong đúng domain để kéo vector về đúng vùng tài liệu thật.

**Q3: Query Decomposition khác Multi-Query thế nào?**
> Decomposition tách câu nhiều ý thành các sub-question khác ý nhau (xử lý câu so sánh/tổng hợp). Multi-Query sinh nhiều cách diễn đạt của cùng một ý (tăng recall cho câu mơ hồ).

**Q4: Khi nào nên dùng Query Decomposition?**
> Khi câu hỏi cần so sánh/tổng hợp thông tin nằm rải rác ở nhiều tài liệu mà không có đoạn nào chứa đủ — tách ra search riêng rồi tổng hợp.

---

## 7. Bài tập tự luyện

- [ ] **Bài 1:** Viết prompt HyDE; với một câu hỏi ngắn, in ra đoạn giả định + so sánh top-5 retrieval giữa "search bằng câu hỏi" và "search bằng HyDE".
- [ ] **Bài 2:** Dựng vòng Query Decomposition cho câu "So sánh A và B": tách sub-query, search từng cái, tổng hợp.
- [ ] **Bài 3:** Thêm Multi-Query (sinh 3 paraphrase) + gộp bằng RRF; đối chiếu với single-query.

---

## 8. Liên kết

- [[02-Hybrid-Search]] — kết hợp transform với Hybrid + RRF
- [[04-Post-Retrieval-Processing]] — sau khi tìm, lọc/sắp lại (Cross-Encoder, MMR)
- [[../01-AI-Fundamentals-RAG/03-Modern-RAG-Architecture]] — query transformation trong phase Retrieval
- [[00-MOC-RAG-Optimization|MOC: RAG & Optimization]]
