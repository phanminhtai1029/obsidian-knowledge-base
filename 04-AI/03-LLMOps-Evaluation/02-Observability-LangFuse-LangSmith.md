---
title: "Observability: LangFuse & LangSmith"
section: 04-AI/03-LLMOps-Evaluation
tags: [ai, llmops, observability, langfuse, langsmith, tracing, monitoring, fresher]
related:
  - "[[01-Ragas-Evaluation-Metrics]]"
  - "[[03-Experiment-Comparison]]"
difficulty: ⭐⭐⭐
estimated_time: 30m
source: [QN26_FR_AI_01, "02-observability.mdx"]
---

# Observability: LangFuse & LangSmith

> [!summary] TL;DR
> Demo dễ, **chạy production khó**: LLM là **black box xác suất** — input vào, output ra, ở giữa không thấy gì. **Observability** là "tia X-quang" soi vào trong: **Tracing** (vẽ lại luồng chain/agent), **Cost** (đếm token/tiền theo user/feature), **Latency** (chỗ nào chậm: Vector DB hay LLM?), **Quality** (gắn điểm/feedback vào trace). 2 công cụ chính: **LangFuse** (open-source, self-host được → quyền riêng tư, miễn phí) và **LangSmith** (của LangChain, trả phí, tích hợp sâu + Playground "Edit & Re-run" mạnh). Production: **sampling** (chỉ trace 1-5%), **che PII**, **đặt alert**.

---

## 1. Vì sao cần Observability?

Phần mềm truyền thống deterministic — bug thì step-through từng dòng là ra. LLM app có **thành phần xác suất** (model AI) hoạt động như **hộp đen**. 4 lý do bắt buộc phải có observability:

1. **Black Box execution** — input vào, output ra, *ở giữa xảy ra gì?* Retriever có lấy đúng tài liệu không? LLM có phớt lờ system prompt không?
2. **Cost Management** — gọi API LLM rất đắt. Một vòng lặp lỗi có thể đốt hàng trăm đô. Cần theo dõi chi phí **real-time theo user/feature**.
3. **Latency Debugging** — response mất 10 giây: do **Vector DB search** chậm hay do **LLM generation** chậm? Phải bóc tách được.
4. **Quality Assurance** — làm sao biết prompt version mới **thực sự tốt hơn**? Cần track usage + feedback người dùng (thumbs up/down).

```
★ Insight ─────────────────────────────────────
• Observability ≠ Evaluation (Môn này có cả hai). Evaluation (Ragas) trả lời
  "CHẤT LƯỢNG bao nhiêu điểm" trên tập test. Observability trả lời "trong
  PRODUCTION thực tế điều gì đang xảy ra" trên traffic thật. Bổ trợ nhau: dùng
  observability bắt trace tệ → đưa vào dataset → chạy Ragas đánh giá.
• Khái niệm trung tâm là TRACE: một bản ghi cây (tree) toàn bộ các bước của một
  request — từ retrieve, rerank, đến từng lần gọi LLM con — kèm input/output,
  token, thời gian mỗi span. "Span" = một bước con trong trace.
─────────────────────────────────────────────────
```

---

## 2. LangFuse — Open Source Observability

[LangFuse](https://langfuse.com/) là công cụ open-source (giấy phép **MIT**), tập trung vào engineering observability. Điểm mạnh nhất: **self-host** được (Docker Compose) → **dữ liệu không rời hạ tầng của bạn**.

**Tính năng chính:**
- **Tracing** — trực quan hoá luồng chain/agent phức tạp.
- **Prompt Management** — version control prompt như một **CMS** (người không phải dev cũng sửa prompt được, không cần deploy lại code).
- **Scores** — gắn điểm chất lượng (thủ công hoặc tự động, vd điểm Ragas) vào trace.
- **Cost Tracking** — tự tính token & chi phí.

**Tích hợp LangChain** — qua `CallbackHandler`, tự động "đo đạc" mọi chain:

```python
from langfuse.callback import CallbackHandler
from langchain_openai import ChatOpenAI
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate

langfuse_handler = CallbackHandler()        # 1. khởi tạo handler

llm = ChatOpenAI()
prompt = PromptTemplate.from_template("Thủ đô của {country} là gì?")
chain = LLMChain(llm=llm, prompt=prompt)

# 2. truyền handler vào callbacks → trace tự xuất hiện trên dashboard
result = chain.invoke(
    {"country": "Vietnam"},
    config={"callbacks": [langfuse_handler]}
)
```

**Prompt Management** — thay vì hardcode prompt trong Git, lưu trên LangFuse để chỉnh không cần deploy:

```python
from langfuse import Langfuse
langfuse = Langfuse()

prompt_obj = langfuse.get_prompt("customer-support", version="production")
langchain_prompt = PromptTemplate.from_template(prompt_obj.get_langchain_prompt())
```

---

## 3. LangSmith — giải pháp Native của LangChain

[LangSmith](https://smith.langchain.com/) do chính team LangChain xây. Trả phí/thương mại, nhưng **tích hợp sâu nhất** và có tính năng **Debug** mạnh: replay & sửa từng bước trong chain.

**Tính năng chính:**
- **Deep Tracing** — hiện **từng bước**, kể cả retry và trạng thái nội bộ.
- **Playground "Edit & Re-run"** — lấy một trace **lỗi** từ production, mở trong playground, **sửa prompt ngay tại chỗ** và chạy lại xem có hết lỗi không. (Đây là điểm "ăn tiền" so với LangFuse.)
- **Datasets & Testing** — tạo dataset từ traffic production và chạy đánh giá ngay.

**Setup (Auto-Tracing)** — "magic" với người dùng LangChain: thường **không cần sửa code**, chỉ cần biến môi trường:

```bash
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY="ls-..."
export LANGCHAIN_PROJECT="my-production-app"
```

Sau khi set, **mọi thứ** tự được trace:

```python
from langchain.chat_models import ChatOpenAI
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import FAISS

# execution này tự động log lên LangSmith, không cần thêm dòng nào
vectorstore = FAISS.from_texts(["LangSmith giỏi debugging."], embedding=OpenAIEmbeddings())
retriever = vectorstore.as_retriever()
docs = retriever.get_relevant_documents("LangSmith dùng để làm gì?")
```

```
★ Insight ─────────────────────────────────────
• Khác biệt cốt lõi về cơ chế tích hợp: LangFuse dùng CALLBACK (bạn chủ động
  truyền handler vào từng invoke) → hoạt động cả với code KHÔNG dùng LangChain.
  LangSmith dùng BIẾN MÔI TRƯỜNG + auto-instrument LangChain → "không cần code"
  nhưng gắn chặt hệ sinh thái LangChain. Đây là lý do LangFuse "platform-
  agnostic" còn LangSmith "native".
─────────────────────────────────────────────────
```

---

## 4. ⭐ So sánh LangFuse vs LangSmith

| Tiêu chí | **LangFuse** | **LangSmith** |
|---|---|---|
| Open Source | ✅ Có (MIT) | ❌ Không (proprietary) |
| Self-hosting | Dễ (Docker Compose) | Chỉ Enterprise (phức tạp) |
| Giá | Free tier rộng / self-host miễn phí | Trả phí (theo trace/token) |
| Tích hợp | SDK + LangChain Callbacks | Native (biến môi trường) |
| Debugging | Tốt (xem trace) | **Xuất sắc** (Playground Edit & Re-run) |
| Prompt Management | **Xuất sắc** (kiểu CMS) | Tốt (tích hợp Hub) |

**Khuyến nghị chọn:**
- **LangFuse** khi: cần **quyền riêng tư dữ liệu** (self-host), ngân sách hạn chế, hoặc muốn giải pháp **độc lập nền tảng** (chạy tốt cả với code không phải LangChain).
- **LangSmith** khi: "all-in" vào **LangChain/LangGraph**, cần **Playground debug** mạnh, và chấp nhận phụ thuộc SaaS.

---

## 5. Production Best Practices

> [!tip] 1. Sampling — đừng trace 100% ở production
> Production có thể hàng triệu request. Trace tất cả vừa **đắt** vừa **nhiễu**.
> - **Dev/Staging:** trace **100%**.
> - **Production:** trace **1–5%** traffic, **hoặc** các trace "quan trọng cao" (lỗi, feedback tiêu cực).

> [!warning] 2. PII / Quyền riêng tư
> **Không bao giờ** log dữ liệu nhạy cảm (thẻ tín dụng, PII) lên tool observability cloud.
> - **LangFuse:** chạy hàm **PII Masking** trước khi gửi.
> - **LangSmith:** dùng tính năng enterprise redaction, hoặc self-host nếu cần tuân thủ.

> [!tip] 3. Alerts — đừng chỉ ngồi nhìn dashboard
> Đặt cảnh báo tự động:
> - **Error Rate Spike** — lỗi > 10% trong 5 phút.
> - **Latency Spike** — P99 latency > 10 giây.
> - **Cost Anomaly** — chi tiêu/ngày > $50.

```
★ Insight ─────────────────────────────────────
• "P99 latency" = ngưỡng mà 99% request nhanh hơn nó (chỉ 1% chậm hơn). Dùng
  P99/P95 thay vì TRUNG BÌNH vì trung bình che giấu các "đuôi dài" — vài request
  chậm 30s vẫn cho trung bình đẹp, nhưng đó chính là user đang bực. Đo percentile
  mới thấy trải nghiệm tệ nhất.
─────────────────────────────────────────────────
```

---

## 6. Data Drift & Model Drift — vì sao chất lượng tụt dần theo thời gian

Một hệ AI đang chạy ngon **vẫn có thể tệ dần** dù *không ai sửa code* — vì **thế giới thay đổi nhưng mô hình thì đứng yên**. Đây gọi là **drift** (trôi dạt). Observability chính là cách *phát hiện* drift.

| Loại drift | Cái gì "trôi" | Ví dụ |
|---|---|---|
| **Data drift** (input/covariate shift) | **Phân phối dữ liệu đầu vào** đổi so với lúc train/thiết lập | Người dùng bắt đầu hỏi bằng tiếng lóng mới, chủ đề mới, ngôn ngữ khác mà hệ chưa gặp |
| **Concept drift** | **Quan hệ input→output đúng** thay đổi | "Email khuyến mãi" hôm nay khác kiểu spam 2 năm trước → nhãn đúng đã đổi |
| **Model drift** (hệ quả) | **Chất lượng mô hình tụt** do data/concept drift | Accuracy/feedback giảm dần dù mô hình y nguyên |

> [!note] Drift trong hệ RAG/LLM trông như thế nào?
> - **Tài liệu nguồn cũ đi** (chính sách công ty đổi) nhưng vector store chưa cập nhật → câu trả lời *đúng cú pháp nhưng sai thực tế*. (Liên quan: phải **re-index khi đổi embedding/đổi dữ liệu** — xem [[../02-RAG-Optimization/01-Advanced-Indexing]].)
> - **Câu hỏi người dùng dịch chuyển** sang chủ đề mới mà tài liệu chưa phủ → retrieval trả context kém liên quan.
> - **Nhà cung cấp LLM âm thầm đổi version model** → output đổi tính cách/format dù prompt y nguyên.

> [!tip] Phát hiện & xử lý drift (nêu được là ăn điểm)
> - **Phát hiện:** theo dõi *phân phối* input (độ dài, ngôn ngữ, chủ đề, embedding), tỉ lệ "không tìm thấy context", và **feedback người dùng** (thumbs down tăng) qua dashboard observability + đặt **alert**.
> - **Đo trôi:** so phân phối embedding mới vs baseline (vd PSI — Population Stability Index, hoặc KL-divergence).
> - **Xử lý:** cập nhật/re-index tri thức, bổ sung tài liệu chủ đề mới, fine-tune/đổi prompt lại, **ghim version model** (pin) để tránh nhà cung cấp đổi ngầm.

```
★ Insight ─────────────────────────────────────
• Drift là lý do cốt lõi vì sao "deploy xong là xong" SAI với hệ ML/LLM. Khác
  phần mềm thường (logic cố định), chất lượng AI là hàm của DỮ LIỆU THỰC ngoài
  đời — mà dữ liệu thì luôn trôi. Vì vậy MLOps/LLMOps coi monitoring drift là
  vòng đời bắt buộc, không phải tuỳ chọn. (Liên hệ MLOps: [[../../06-DevOps/12-Modern-DevOps]].)
• Overfitting (lúc train) vs drift (lúc production) dễ lẫn: overfit là mô hình
  học vẹt dữ liệu CŨ; drift là dữ liệu MỚI khác dữ liệu cũ. Hai vấn đề ở hai
  thời điểm khác nhau của vòng đời. (Xem overfitting: [[../01-AI-Fundamentals-RAG/01-Introduction-AI-GenAI]].)
─────────────────────────────────────────────────
```

> [!note] 🧠 Mẹo nhớ
> **Data drift = đầu VÀO đổi; Concept drift = quan hệ ĐÚNG đổi; Model drift = chất lượng TỤT (hệ quả).** "Deploy xong không phải là xong" — phải monitor phân phối input + feedback để bắt drift, rồi re-index/cập nhật.

---

## 7. Pitfalls / Bẫy thường gặp

> [!warning] Trace 100% ở production
> Đốt tiền và làm dashboard nhiễu. Luôn sampling + ưu tiên trace lỗi/feedback xấu.

> [!warning] Log thẳng PII lên cloud tool
> Vi phạm bảo mật/tuân thủ (GDPR...). Mask trước khi gửi hoặc self-host.

> [!warning] Coi observability là evaluation
> Trace cho biết "đã xảy ra gì", không tự nói "tốt hay xấu". Phải gắn **scores** (Ragas/feedback) mới thành đánh giá.

> [!warning] Chọn LangSmith rồi muốn self-host
> LangSmith self-host chỉ có ở bản Enterprise và phức tạp. Nếu yêu cầu cốt lõi là self-host/privacy → chọn LangFuse từ đầu.

---

## 8. Câu hỏi phỏng vấn thường gặp

**Q1: Observability trong LLM app là gì và vì sao cần?**
> Là khả năng "soi" vào hệ thống LLM (vốn là black box xác suất) để biết điều gì xảy ra giữa input và output: tracing luồng, đo cost, debug latency, đảm bảo chất lượng. Cần vì không thể step-through model như code thường, và lỗi/chi phí ở production rất tốn kém.

**Q2: Trace là gì?**
> Bản ghi dạng cây toàn bộ các bước (span) của một request — retrieve, rerank, từng lần gọi LLM — kèm input/output, token, thời gian mỗi bước. Là đơn vị quan sát cơ bản.

**Q3: LangFuse khác LangSmith thế nào?**
> LangFuse open-source (MIT), self-host dễ, miễn phí/rẻ, tích hợp qua callback nên dùng được cả ngoài LangChain. LangSmith proprietary, trả phí, native LangChain qua biến môi trường, mạnh nhất ở Playground "Edit & Re-run" debug.

**Q4: Khi nào chọn LangFuse, khi nào LangSmith?**
> LangFuse khi cần privacy/self-host, ngân sách hạn chế, hoặc code không thuần LangChain. LangSmith khi all-in LangChain/LangGraph và cần debug playground mạnh, chấp nhận SaaS.

**Q5: Production nên trace bao nhiêu phần trăm traffic, vì sao?**
> 1–5% (cộng thêm các trace lỗi/feedback xấu) thay vì 100%, vì trace tất cả vừa tốn tiền vừa nhiễu. Dev/staging mới trace 100%.

**Q6: Observability khác Evaluation chỗ nào?**
> Evaluation (Ragas) chấm điểm chất lượng trên tập test. Observability theo dõi điều xảy ra trên traffic production thực. Kết hợp: observability bắt trace xấu → đưa vào dataset → Ragas đánh giá.

**Q7: Data drift là gì? Vì sao một hệ AI đang chạy tốt lại tệ dần?**
> **Data drift** = phân phối dữ liệu đầu vào ở production **đổi khác** so với lúc train/thiết lập (người dùng hỏi chủ đề/ngôn ngữ mới...). **Concept drift** = quan hệ input→output đúng thay đổi. Hậu quả là **model drift** — chất lượng tụt dù mô hình không đổi. Trong RAG còn do **tài liệu nguồn cũ đi** mà chưa re-index. **Phát hiện** bằng theo dõi phân phối input + feedback (thumbs down) qua observability + alert; **xử lý** bằng cập nhật/re-index dữ liệu, đổi prompt, ghim version model.

---

## 9. Bài tập tự luyện

- [ ] **Bài 1:** Tạo tài khoản LangSmith, set 3 biến môi trường, chạy một chain RAG và xem trace (các span, token, latency từng bước).
- [ ] **Bài 2:** Tích hợp LangFuse `CallbackHandler` vào cùng chain đó; so sánh giao diện trace 2 bên.
- [ ] **Bài 3:** Gắn `score` (vd điểm Faithfulness từ Ragas) vào trace trên LangFuse.
- [ ] **Bài 4:** Bật sampling 10% và mô phỏng 100 request → quan sát chỉ ~10 trace được ghi.
- [ ] **Bài 5:** Thiết kế một "drift monitor" đơn giản: log độ dài + ngôn ngữ câu hỏi mỗi ngày, vẽ biểu đồ phân phối, đặt ngưỡng cảnh báo khi tỉ lệ "không tìm thấy context" tăng vọt.

---

## 9. Liên kết

- [[01-Ragas-Evaluation-Metrics]] — điểm Ragas được gắn vào trace (Scores)
- [[03-Experiment-Comparison]] — observability thu thập trace/cost/latency cho thí nghiệm
- [[../02-RAG-Optimization/00-MOC-RAG-Optimization|MOC: RAG & Optimization]] — các bước (retrieve, rerank) chính là các span trong trace
- [[00-MOC-LLMOps-Evaluation|MOC: LLMOps & Evaluation]]
