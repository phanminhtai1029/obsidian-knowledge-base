---
title: "AI-102 Cheatsheet + Q&A phỏng vấn"
section: 05-Cloud/02-Azure/AI-102
tags: [azure, ai-102, cheatsheet, interview, exam, fresher]
related:
  - "[[00-MOC-AI-102]]"
difficulty: ⭐⭐
estimated_time: 18m
source: ["_source/Microsoft/AI-102.docx (Lesson 20, Tim Warner)", "Microsoft Learn"]
status: ✅
---

# AI-102 Cheatsheet + Q&A phỏng vấn

> [!summary] TL;DR
> Tờ tra nhanh tổng hợp toàn bộ AI-102: **bảng chọn dịch vụ theo tình huống**, các **cặp dễ nhầm** (Vision vs Custom Vision vs Document Intelligence; CLU vs QnA vs OpenAI; RAG vs fine-tune; key vs Managed Identity), **glossary** thuật ngữ, **mẹo thi & lỗi hay gặp** (region/quota/tier), và **Q&A phỏng vấn**. Dùng rà soát lần cuối trước thi/phỏng vấn AI engineer.

---

## 1. Bảng tra dịch vụ ↔ use case

| Gặp tình huống… | Chọn dịch vụ |
|---|---|
| Chatbot LLM, tóm tắt, sinh text | **Azure OpenAI** (note 12) |
| Hỏi-đáp trên tài liệu nội bộ (cập nhật, dẫn nguồn) | **AI Search + OpenAI (RAG)** (note 10,12) |
| Mô tả ảnh / tag / object dựng sẵn | **AI Vision** (note 4) |
| Đọc chữ trong ảnh (text thô) | **AI Vision – Read (OCR)** (note 4) |
| Trích trường từ hoá đơn/biểu mẫu (có cấu trúc) | **Document Intelligence** (note 11) |
| Phân loại/đếm vật thể đặc thù domain | **Custom Vision** (note 5) |
| Sentiment / NER / **ẩn PII** | **AI Language** prebuilt (note 6) |
| Hiểu **ý định** câu người dùng | **CLU** (note 6,9) |
| Hỏi-đáp FAQ cố định | **Custom Question Answering** (note 9) |
| Dịch text/tài liệu đa ngữ | **Translator** (note 7) |
| Giọng nói ↔ văn bản, đọc văn bản | **AI Speech** (note 8) |
| Phân tích nội dung video | **Video Indexer** (note 4) |
| Kiểm duyệt nội dung độc / chống prompt injection | **Content Safety** (note 3) |

---

## 2. Bảng phân biệt khái niệm dễ nhầm

| Cặp | Khác biệt cốt lõi |
|---|---|
| **AI Vision Read vs Document Intelligence** | Text thô + vị trí · **dữ liệu có cấu trúc** (key-value/bảng/trường) |
| **AI Vision vs Custom Vision** | Prebuilt gọi-ngay · **train model ảnh riêng** (classify/detect) |
| **Classification vs Object detection** | Nhãn cả ảnh · nhiều vật thể + **bounding box** (đánh giá mAP) |
| **CLU vs Question Answering** | Intent+entity để **hành động** · trả lời FAQ từ **KB** |
| **CLU/QnA bot vs OpenAI chat** | Intent-based, kiểm soát chặt · generative linh hoạt (cần grounding) |
| **RAG vs Fine-tuning** | Nạp **kiến thức mới** (đổi dữ liệu) · đổi **phong cách/định dạng** |
| **Fine-tuning vs Prompt engineering** | Train trên JSONL (tốn) · chỉ chỉnh prompt (thử trước) |
| **Function calling vs RAG** | Model **hành động** (gọi API) · model có **ngữ cảnh** để trả lời |
| **API key vs Managed Identity** | Secret tĩnh dễ lộ · token tự động, **không secret** (an toàn nhất) |
| **Public vs Private endpoint** | Qua internet · trong VNet, **không ra internet** |
| **Translate vs Transliterate** | Đổi **nghĩa** · đổi **chữ viết** (kanji→romaji) |
| **Text vs Document Translation** | Chuỗi text · **nguyên file giữ định dạng** (batch/Blob) |
| **STT real-time vs batch** | Stream trả ngay · hàng loạt async qua Blob |
| **Custom template vs neural (Doc Intel)** | Mẫu cố định, ít data, nhanh · mẫu biến thiên, nhiều data, chính xác |
| **Multi vs single-service resource** | 1 key/endpoint chung (tiện) · tách billing/quyền/region (kiểm soát) |
| **Built-in vs custom skill (AI Search)** | Cấu hình sẵn · gọi Function/Web API riêng |

---

## 3. Mẹo thi & lỗi hay gặp

- **Deployment name ≠ model name**: gọi Azure OpenAI theo **tên deployment** bạn đặt, không phải "gpt-4o".
- **Translator cần `Ocp-Apim-Subscription-Region`** trong header (khác đa số dịch vụ).
- **429 = quota/TPM**, không phải bug → retry+backoff, tăng quota, scale (note 2).
- **Fine-tune không nạp kiến thức mới** — cần kiến thức mới thì RAG. Bẫy kinh điển.
- **Region availability**: không phải model/dịch vụ nào cũng có ở mọi region (Azure OpenAI giới hạn region) → check trước khi chọn region cho compliance.
- **Face identify & Custom Neural Voice = Limited Access** (phải đăng ký, Responsible AI).
- **OCR/Read cho text, Document Intelligence cho biểu mẫu** — đọc kỹ "có cấu trúc" trong đề.
- **Managed Identity** luôn là đáp án ưu tiên cho "truy cập an toàn không lộ key".

---

## 4. Q&A phỏng vấn

> [!question] "Hỏi-đáp trên tài liệu công ty, dữ liệu hay cập nhật — kiến trúc nào?"
> **RAG**: index tài liệu vào **AI Search** (hybrid + semantic), **Azure OpenAI** truy hồi rồi sinh câu trả lời **có dẫn nguồn**. Cập nhật = reindex, không train lại. Bọc **Content Safety**.

> [!question] "Vì sao không fine-tune để bot biết dữ liệu mới?"
> Fine-tune dạy **phong cách/định dạng**, không hợp nạp kiến thức hay đổi (phải train lại mỗi lần, không dẫn nguồn). Kiến thức mới → **RAG**.

> [!question] "Truy cập dịch vụ AI an toàn, không nhúng key — làm sao?"
> **Managed Identity** + `DefaultAzureCredential`: Azure cấp token tự động, **không secret trong code**, audit được, tự xoay. Nếu buộc dùng key thì cất **Key Vault**.

> [!question] "Đọc hoá đơn lấy tổng tiền & nhà cung cấp?"
> **Document Intelligence** prebuilt **invoice** — trả trường nghiệp vụ có cấu trúc, không chỉ text như OCR.

> [!question] "Phân biệt nội dung độc hại & chống prompt injection cho chatbot?"
> **Content Safety**: severity 0/2/4/6 cho 4 nhóm hại (text+image) + **Prompt Shields** (input) + **Groundedness** (output).

> [!question] "6 nguyên tắc Responsible AI?"
> Fairness, Reliability & Safety, Privacy & Security, Inclusiveness, Transparency, Accountability — Transparency & Accountability là hai nguyên tắc bao trùm.

> [!question] "Train model nhận diện lỗi sản phẩm riêng trên dây chuyền — dùng gì?"
> **Custom Vision** (object detection nếu cần vị trí/đếm; classification nếu chỉ phân loại đạt/lỗi); compact + ONNX để chạy **edge** tại nhà máy.

> [!question] "Một bot vừa trả lời FAQ vừa thực hiện yêu cầu — thiết kế?"
> **Bot Service** + **Orchestration**: route câu hỏi FAQ → **Question Answering**, route yêu cầu hành động → **CLU** (intent+entity) → gọi API.

---

## Tự kiểm tra

1. Cho 5 tình huống bất kỳ ở bảng mục 1, đọc đáp án dịch vụ rồi tự giải thích vì sao.
2. Đọc cột "khác biệt cốt lõi" mục 2, che đi và tự nói lại từng cặp.
3. Liệt kê 5 lỗi/bẫy hay gặp trong mục 3.
4. Trả lời lại 8 câu Q&A mà không nhìn đáp án.
5. RAG vs fine-tune vs function calling — mỗi cái cho bài toán nào?

---

## Liên quan
- [[00-MOC-AI-102]]
- [[../AZ-204/13-AZ-204-Cheatsheet-va-QA]] — cheatsheet AZ-204 đối chiếu
- [[../AZ-900/15-AZ-900-Cheatsheet]] — cheatsheet AZ-900 nền tảng
