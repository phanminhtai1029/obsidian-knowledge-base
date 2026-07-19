---
title: "Azure Language: text analysis (language detection, NER, PII) bản Foundry Tools"
section: 05-Cloud/02-Azure/AI-103
tags: [azure, ai-103, azure-language, nlp, ner, pii, fresher]
---

# Note 11 — Azure Language: text analysis bản Foundry Tools

> **TL;DR:** **Azure Language in Foundry Tools** cung cấp API phân tích văn bản với 3 năng lực chính còn hiện hành: **language detection** (nhận diện ngôn ngữ + confidence score 0-1), **named entity recognition** (NER — bóc thực thể Person/Location/DateTime/Organization/Email/URL…), **PII extraction & redaction** (tìm + che thông tin cá nhân qua `redacted_text`). Các năng lực **sentiment, summarization, key phrase** nay được đánh dấu **deprecated** (giữ cho app cũ — thay đổi lớn so với AI-102!). Dùng qua `TextAnalyticsClient` (package `azure-ai-textanalytics`), auth bằng key hoặc **Entra ID** (khuyến nghị production). Giới hạn: **5.120 ký tự/document**, tối đa **1.000 document/collection**.

## 1. Vị trí trong Foundry & kết nối

- Cần **Foundry resource** trong subscription; gọi API qua **resource endpoint** (không phải project endpoint). Mẹo suy endpoint: project endpoint = resource endpoint + `/api/projects/{project_name}` → bỏ đuôi là ra resource endpoint. **Key của project và resource là một**.
- 2 cách auth:

```python
# Key-based
from azure.core.credentials import AzureKeyCredential
from azure.ai.textanalytics import TextAnalyticsClient
client = TextAnalyticsClient(endpoint="FOUNDRY_RESOURCE_ENDPOINT",
                             credential=AzureKeyCredential("FOUNDRY_RESOURCE_KEY"))

# Entra ID (khuyến nghị production)
from azure.identity import DefaultAzureCredential
client = TextAnalyticsClient(endpoint="FOUNDRY_RESOURCE_ENDPOINT",
                             credential=DefaultAzureCredential())
```

Gọi được bằng REST (JSON) hoặc SDK Python/.NET/JavaScript (pattern giống nhau).

## 2. Ba năng lực chính

### Language detection — `detect_language()`
- Trả về mỗi document: `primary_language.name`, `iso6391_name`, `confidence_score` (0→1, càng gần 1 càng chắc).
- Use case: kho nội dung không rõ ngôn ngữ; chat app tự nhận ngôn ngữ user để trả lời đúng thứ tiếng.
- Ba tình huống cần nhớ:

| Input | Kết quả |
|-------|---------|
| Đơn ngữ, rõ ràng | Ngôn ngữ đúng, confidence cao (~0.9+) |
| **Trộn nhiều ngôn ngữ** trong 1 document | Trả ngôn ngữ **chiếm tỉ trọng lớn nhất**, confidence **thấp hơn** |
| Không parse được (lỗi encoding…) | `(unknown)` + score **0** |

### Named Entity Recognition — `recognize_entities()`
- Bóc thực thể theo **category/subcategory**: Person, PersonType, Location, DateTime, Organization, Address, Email, URL…
- Use case chuẩn: index bài báo theo người/địa danh/ngày tháng được nhắc đến (đề thi: chọn NER, không phải regex + LLM, không phải PII).

### PII detection & redaction — `recognize_pii_entities()`
- Nhận diện thông tin nhạy cảm: tên, địa chỉ, phone, email, **USSocialSecurityNumber**, số thẻ… kèm confidence từng entity.
- **Redaction**: response có sẵn `doc.redacted_text` — bản văn bản đã che PII bằng dấu `*` → dùng trước khi công bố feedback khách hàng, hồ sơ y tế, tài liệu pháp lý.

```python
response = client.recognize_pii_entities(documents=documents, language="en")
for doc in response:
    print(doc.redacted_text)   # "********** works at ************. His email is ..."
```

## 3. Giới hạn & thay đổi so với đời cũ

- **5.120 ký tự / document**; **1.000 document (ID) / collection** mỗi request.
- ⚠️ **Deprecated** (chỉ còn để đỡ app cũ): **sentiment analysis, summarization, key phrase extraction** và các task tương tự — giáo trình mới định hướng các việc đó sang **GenAI/LLM**, còn Azure Language giữ vai trò các task **trích xuất chuyên biệt** (detect/NER/PII/health).
- Tên cũ: Text Analytics → Azure AI Language → **Azure Language in Foundry Tools** (SDK vẫn tên `azure-ai-textanalytics`).

`★ Insight ─────────────────────────────────────`
Câu "vì sao không dùng LLM bóc PII/entity?": Azure Language dùng model chuyên biệt → **rẻ, nhanh, deterministic, có confidence score, có sẵn redaction** — LLM đắt hơn và không bảo đảm bắt đủ (rủi ro compliance khi sót PII). Ngược lại việc *hiểu/tóm/viết* thì LLM thắng — đúng ranh giới Microsoft vẽ khi deprecate sentiment/summarization ở Language.
`─────────────────────────────────────────────────`

## Q&A phỏng vấn

**Q1. App cần index bài báo theo người, địa danh, ngày tháng — dùng gì?**
→ Azure Language **NER** (`recognize_entities`) — trả entity theo category Person/Location/DateTime. Không dùng PII (mục đích là che thông tin nhạy cảm) hay regex thủ công.

**Q2. Cần đăng trích đoạn feedback khách nhưng phải xoá thông tin cá nhân?**
→ `recognize_pii_entities` + lấy `redacted_text` — service tự che PII bằng `*`.

**Q3. Document trộn tiếng Anh + Pháp thì detect_language trả gì?**
→ Ngôn ngữ chiếm tỉ trọng lớn nhất, với confidence thấp hơn bình thường. Text không parse được thì trả `(unknown)` với score 0.

**Q4. Sentiment analysis giờ nằm đâu trong giáo trình mới?**
→ Ở Azure Language nó bị **deprecated** (giữ cho app cũ). Hướng mới: dùng GenAI/LLM cho sentiment/tóm tắt; Azure Language tập trung detect/NER/PII/health.

**Q5. Endpoint nào để gọi Azure Language — project hay resource?**
→ **Resource endpoint** (`https://{res}.services.ai.azure.com` — bỏ phần `/api/projects/...`). Key project = key resource.

## Liên quan
- [[00-MOC-AI-103]] — MOC AI-103
- [[01-Microsoft-Foundry-Tong-quan-Plan-Prepare]] — Foundry Tools tổng quan
- [[12-Language-va-Speech-MCP-Server]] — expose các năng lực này cho agent qua MCP
- [[14-Translator-Text-va-Speech]] — dịch thuật (chị em cùng nhóm NLP)
