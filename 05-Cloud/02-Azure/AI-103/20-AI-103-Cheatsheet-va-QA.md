---
title: "AI-103 Cheatsheet: tra nhanh toàn bộ + Q&A + gì mới so với AI-102"
section: 05-Cloud/02-Azure/AI-103
tags: [azure, ai-103, cheatsheet, microsoft-foundry, exam, fresher]
---

# Note 20 — AI-103 Cheatsheet & Q&A tổng hợp

> **TL;DR:** Note ôn nước rút: (1) bảng **"gì mới so với AI-102"** — nắm để không trả lời theo giáo trình cũ; (2) **bản đồ chọn dịch vụ** theo bài toán; (3) các **bảng phân biệt** hay ra đề gom về một chỗ; (4) **số liệu phải thuộc**; (5) Q&A xuyên cụm. Chi tiết từng chủ đề xem note 01-19.

## 1. Gì mới so với AI-102 (giáo trình cũ) — bảng "đổi não"

| AI-102 cũ | AI-103 (Foundry era) | Note |
|-----------|----------------------|------|
| Azure AI Studio / Azure AI Foundry (hub-based) | **Microsoft Foundry**: resource → projects → assets; hub-based bị gọi là "classic" | 01 |
| Azure AI Services (Cognitive Services) | **Foundry Tools** (Language, Speech, Translator, Document Intelligence, **Content Understanding**) | 01 |
| ChatCompletions là chuẩn | **Responses API** là chuẩn mới (stateful, `previous_response_id`, tools); ChatCompletions cho model chưa hỗ trợ | 03 |
| Không có agent chính thống (Bot Framework) | **Foundry Agent Service** + declarative/hosted agents, Deploy vs Publish | 05 |
| — | **MCP** (agent ↔ tool) & **A2A** (agent ↔ agent), Language/Speech MCP server | 06, 10, 12 |
| — | **Foundry IQ**: knowledge base RAG dùng chung cho agent | 07 |
| Semantic Kernel + AutoGen (2 framework rời) | **Microsoft Agent Framework** (hợp nhất cả hai, 5 pattern orchestration) | 09 |
| Language: sentiment, summarization, key phrase là nội dung thi | Các task đó **deprecated** ở Language → đẩy sang GenAI/LLM; Language giữ detect/NER/PII/health | 11 |
| Speech SDK batch/turn-based | + **Voice Live API** (WebSocket speech-to-speech realtime, avatar) | 13 |
| DALL-E sinh ảnh | **gpt-image-1 / FLUX** + **Sora 2 sinh video** | 16 |
| Form Recognizer/Document Intelligence là cửa duy nhất cho trích xuất | + **Content Understanding**: schema đa phương thức (doc+ảnh+audio+video), extract/classify/generate | 17-18 |
| Custom Vision, Video Indexer, Bot Framework, QnA Maker, CLU | **Bị loại khỏi giáo trình** | — |

## 2. Bản đồ chọn dịch vụ theo bài toán (câu tình huống)

| Đề bài nói… | Chọn | Note |
|-------------|------|------|
| Chat/sinh text, cần tools (function, web/file search, code) | Responses API + Foundry Models | 03 |
| So sánh model theo chất lượng/giá/tốc độ | Model catalog **benchmarks**; đánh giá bằng AI-assisted metrics | 02 |
| Tự động hoá tác vụ, gọi hệ thống ngoài | **Agent** + custom tool (function) / **MCP tool** | 05-06 |
| RAG cho NHIỀU agent, quản lý tập trung theo domain nghiệp vụ | **Foundry IQ** knowledge base | 07 |
| Nhiều agent phối hợp / agent của bên khác | **Agent Framework** (nội bộ) / **A2A** (liên bên) | 09-10 |
| Nhận diện ngôn ngữ, bóc entity, che PII | **Azure Language** (KHÔNG dùng LLM — rẻ, deterministic, compliance) | 11 |
| Sentiment / tóm tắt văn bản | **GenAI/LLM** (Language đã deprecate) | 11 |
| Voice agent realtime, ngắt lời được | **Voice Live API** | 13 |
| Dịch văn bản giữ format tài liệu / chuyển hệ chữ viết | Translator: `translate` / `transliterate` | 14 |
| Hỏi đáp tự do về một tấm ảnh | **Multimodal model** (gpt-4.1, Phi-4-multimodal) — message multi-part | 15 |
| Sinh ảnh / sinh video | gpt-image-1, FLUX / **Sora 2** | 16 |
| Trích field CÓ SCHEMA từ ảnh/audio/video/doc hỗn hợp | **Content Understanding** | 17 |
| Form chuẩn ngành (invoice, W-2, hộ chiếu…), layout in cố định | **Document Intelligence** (prebuilt / custom template) | 18 |
| Tìm kiếm full-text + đào insight kho tài liệu | **Azure AI Search** (indexer + skillset + knowledge store) | 19 |
| Chặn nội dung độc hại, chống jailbreak | Guardrails: content filters + **prompt shields** | 04 |

## 3. Các bảng phân biệt "ăn điểm" gom về một chỗ

| Cặp | Mấu chốt | Note |
|-----|----------|------|
| Responses API vs ChatCompletions | stateful (`previous_response_id`) + tools vs stateless; `input_text/input_image` vs `text/image_url` | 03, 15 |
| Deploy vs Publish (agent) | Deploy = chạy trong project; **Publish = Agent Application, identity Entra MỚI → gán lại RBAC** | 05 |
| MCP vs A2A | agent ↔ **tool** vs agent ↔ **agent** (Agent Card `/.well-known/agent-card.json`) | 06, 10 |
| translate vs transliterate | đổi **ngôn ngữ** vs đổi **hệ chữ viết** (спасибо→spasibo) | 14 |
| Manual vs event-based synthesis (dịch nói) | nhiều ngôn ngữ đích vs **chỉ 1:1** (voice_name + event `synthesizing`) | 14 |
| extract vs classify vs generate (CU) | đọc cái CÓ SẴN vs chọn từ enum vs SINH mới | 17 |
| read vs layout (DI) | text + ngôn ngữ vs + **bảng + selection marks** (+key-value) | 18 |
| Custom template vs neural (DI) | layout **cố định**, train phút, 100+ ngôn ngữ vs layout **biến thiên**, chính xác hơn | 18 |
| Document Intelligence vs Content Understanding | form chuyên sâu, train bằng label vs đa phương thức, schema GenAI | 18 |
| Content Understanding vs Vision chat (15) | schema + confidence + grounding vs hỏi đáp tự do không cấu trúc | 17 |
| Language MCP vs Speech MCP server | chỉ cần key vs **+ Storage + SAS URL** (file audio) | 12 |
| Foundry IQ vs tự cắm AI Search vào từng agent | KB tập trung theo **domain nghiệp vụ**, nhiều nguồn, retrieval instructions | 07 |
| searchMode Any vs All | chứa 1 term vs phải đủ MỌI term | 19 |

## 4. Số liệu phải thuộc

| Con số | Của cái gì |
|--------|-----------|
| **4 / 8 / 12 giây**; 1280×720 / 720×1280; **2 job** song song; giữ **24h** | Sora 2 | 
| **5.120 ký tự**/document; **1.000 document**/collection | Azure Language |
| Confidence **0.9+** tự động / 0.7-0.9 cân nhắc / **<0.7** người duyệt | Content Understanding |
| **≥5-6 form mẫu** + ocr.json + fields.json + labels.json | Train custom DI |
| **500 MB** (free 4 MB); 50×50→10.000×10.000 px; PDF ≤ A3, không password | Input DI |
| **≤20 document** nếu không gắn Foundry Tools resource (cùng region) | Built-in skills AI Search |
| GPT-4.1 + GPT-4.1-mini + **text-embedding-3-large** phải deploy trước | Content Understanding API |
| 4 severity × 5 category (thêm task-adherence) | Content filters |
| Batch deployment **−50%**, trả kết quả trong **24h** | Deployment types |

## 5. Mẹo "đọc đề đoán đáp án"

- Thấy **"agent tự chọn tool theo mô tả"** → dynamic tool discovery của **MCP**.
- Thấy **"schema"** + nhiều loại content → Content Understanding; thấy **"train với label"** → Document Intelligence custom.
- Thấy **"stateful/hội thoại tiếp nối"** → Responses API (`previous_response_id`), không phải tự nối messages.
- Thấy **"Entra ID, không lưu key"** → `DefaultAzureCredential` (mọi SDK Foundry).
- Thấy **"realtime, ngắt lời, độ trễ thấp"** với voice → Voice Live API (WebSocket), không phải Speech SDK turn-based.
- Thấy **"compliance/che thông tin cá nhân"** → Azure Language PII (`redacted_text`), KHÔNG chọn LLM.
- Thấy **"một endpoint route nhiều loại form"** → composed model / custom classifier.
- Thấy **"kết quả bất đồng bộ, operation ID, poll"** → đúng pattern của Sora video, Content Understanding, DI — trả lời theo flow create → poll → get result.

## Q&A tổng hợp xuyên cụm

**Q1. Kiến trúc Microsoft Foundry một câu?**
→ Một **Foundry resource** chứa nhiều **project**; project chứa asset (Models/Agents/Tools/Knowledge); portal cho low-code, SDK (`azure-ai-projects`) cho code; Foundry Tools = các dịch vụ AI chuyên biệt gắn kèm. Hub-based là kiến trúc classic cũ.

**Q2. Một pipeline "nhận đơn hàng scan → bóc field → lưu → tìm kiếm được" dùng chuỗi nào?**
→ **Document Intelligence** (prebuilt invoice / custom model) bóc field → lưu DB/blob → **AI Search indexer** (DI làm custom skill nếu bóc trong pipeline) index cho tìm kiếm; nếu đơn hàng lẫn ảnh/audio → cân nhắc **Content Understanding** một cửa.

**Q3. Vì sao Microsoft deprecate sentiment/summarization ở Language nhưng vẫn giữ NER/PII?**
→ Việc *hiểu/tóm/cảm xúc* LLM làm tốt hơn; việc *trích xuất chuyên biệt* cần **deterministic + confidence + redaction + giá rẻ** cho compliance thì model chuyên biệt thắng — ranh giới "LLM vs specialized model" là triết lý xuyên suốt giáo trình mới.

**Q4. Agent cần: tra chính sách nội bộ + đặt lịch qua API công ty + nhờ agent đối tác báo giá. Ba kết nối?**
→ Chính sách: **Foundry IQ** knowledge base (RAG); API nội bộ: **custom tool / MCP tool**; agent đối tác: **A2A protocol** (khám phá qua Agent Card).

**Q5. App voice dịch realtime Anh→Pháp phát loa, chọn stack nào giữa TranslationRecognizer và Voice Live?**
→ Dịch 1:1 kèm phát tiếng: **TranslationRecognizer + event-based synthesis** đủ và rẻ; cần hội thoại agent hai chiều, ngắt lời, độ trễ thấp: **Voice Live API**.

**Q6. Điểm chung của video Sora, Content Understanding, Document Intelligence về mặt API?**
→ Đều **bất đồng bộ**: submit → nhận ID → poll status → lấy kết quả; SDK giấu vòng poll sau poller/`result()`.

**Q7. Publish agent lên M365 thì cái gì hay bị quên nhất?**
→ Agent Application được cấp **Entra identity mới** → phải **gán lại RBAC** cho identity đó trên các resource (Foundry, Storage, Search…) — quyền của identity cũ không tự chuyển.

**Q8. Nhà tuyển dụng hỏi: "AI-102 với giáo trình Foundry mới khác gì?"**
→ Trả lời theo bảng mục 1: đổi nền tảng (Foundry resource/project), đổi chuẩn API (Responses), thêm hẳn tầng agent (Agent Service, MCP, A2A, Foundry IQ, Agent Framework), Content Understanding thay cách tiếp cận từng-service-một, và cắt Custom Vision/Video Indexer/Bot Framework.

## Liên quan
- [[00-MOC-AI-103]] — MOC AI-103 (đường về mọi note)
- [[../AZ-204/13-AZ-204-Cheatsheet-va-QA|AZ-204 Cheatsheet]] — cheatsheet nhánh developer Azure
- [[../../01-AWS-Bedrock/00-MOC-AWS-Bedrock|MOC: AWS Bedrock]] — đối chiếu hệ sinh thái AWS
- [[../../../04-AI/00-MOC-AI|MOC: AI]] — nền tảng RAG/agent framework-agnostic
