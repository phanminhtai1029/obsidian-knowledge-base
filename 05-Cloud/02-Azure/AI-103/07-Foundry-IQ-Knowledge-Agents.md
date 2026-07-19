---
title: "Foundry IQ — knowledge-enhanced agents (nền tảng RAG dùng chung)"
section: 05-Cloud/02-Azure/AI-103
tags: [azure, ai-103, foundry-iq, rag, knowledge-base, ai-search, fresher]
---

# Note 07 — Foundry IQ: knowledge-enhanced AI agents

> **TL;DR:** **Foundry IQ** = nền tảng tri thức managed cho AI agent, xây trên **Azure AI Search** — thay vì mỗi team tự dựng pipeline RAG (chunking, vector DB, access control) lặp đi lặp lại, bạn tạo **knowledge base** (tổ chức theo **business domain**, không theo nơi lưu trữ) **một lần**, rồi **nhiều agent cùng dùng chung** qua kết nối **MCP**. Nối được 6 loại nguồn: AI Search index, Blob Storage, **Web (Bing realtime)**, SharePoint (Remote realtime / Indexed), OneLake. Foundry IQ tự lo indexing, embedding, chọn chiến lược retrieval, ranking, **citations**. Chất lượng phụ thuộc 2 thứ bạn kiểm soát: **dữ liệu** (scoring profiles, semantic ranking, custom analyzers) và **retrieval instructions** của agent (LUÔN search KB, cite đúng format, fallback khi không tìm thấy).

## 1. Vấn đề: agent "trần" không dùng được trong enterprise

| Giới hạn | Hệ quả |
|----------|--------|
| Knowledge cutoff | Không biết tính năng/chính sách mới |
| Không truy cập dữ liệu private | Trả lời chung chung, thiếu quy trình công ty |
| Thiếu context tổ chức | Khuyên sai yêu cầu bảo mật/quy trình duyệt |
| Bịa đáp án (fabrication) | Rủi ro compliance |
| Mỗi team tự build RAG | Lặp công sức hạ tầng |

**RAG** (Retrieve → Augment → Generate) giải quyết 4 dòng đầu — cho 3 lợi ích: **real-time updates** (không retrain), **source transparency** (user thấy tài liệu nào tạo ra câu trả lời), **factual grounding** (neo vào nội dung thật). Nhưng RAG tự build vẫn dính dòng cuối → **Foundry IQ** giải nốt: RAG-as-a-platform dùng chung.

## 2. Knowledge base — tổ chức theo domain nghiệp vụ

Agent không search "SharePoint site A" hay "Blob container B" — nó search "**Product Documentation**" hay "**HR Policies**". Một KB gộp nhiều nguồn bất kể nơi lưu: spec từ SharePoint + API docs từ Blob + analytics từ OneLake + ticket từ search index → agent thấy **một nguồn tri thức thống nhất**.

Khi thêm data source: **Discovery** (quét tài liệu) → **Processing** (chunk + embed) → **Indexing** (thành searchable) → **Monitoring** (tài liệu đổi → tự reindex). Cấu hình một lần; **mọi agent nối vào hưởng ngay khi KB cải thiện** — đây là "shared knowledge advantage": Product Docs KB phục vụ cả support agent lẫn developer agent; HR KB chỉ phục vụ employee assistant.

### Nối agent vào KB (qua MCP)

```python
knowledge_tool = MCPTool(
    server_label="product-docs",
    server_url=f"{search_endpoint}/knowledgebases/product-documentation/mcp")

agent = project_client.agents.create_version(
    agent_name="product-support-agent",
    definition=PromptAgentDefinition(
        model="gpt-4o-mini",
        instructions="Answer product questions using the knowledge base. Always cite your sources.",
        tools=[knowledge_tool]))
```

## 3. Sáu loại data source

| Nguồn | Kiểu truy cập | Hợp khi |
|-------|---------------|---------|
| **Azure AI Search Index** | Indexed | Đã đầu tư AI Search; cần semantic ranking, custom scoring, facet, đa ngữ |
| **Azure Blob Storage** | Direct | File PDF/docx/txt/md/HTML trong Blob — đường thẳng từ file vào KB, không tự quản index |
| **Web (Bing)** | Real-time | Thông tin công khai, mới (giá, sự kiện) — **ít kiểm soát nguồn**, cần accuracy cao thì tránh; hay dùng làm nguồn **bổ trợ** khi KB nội bộ không có đáp án |
| **SharePoint Remote** | Real-time | Đơn giản nhất, luôn mới, **tự tôn trọng permission SharePoint**, không phải quản index |
| **SharePoint Indexed** | Indexed | Nhanh hơn, full năng lực AI Search (custom analyzer, enrichment pipeline, trộn nguồn khác) — đổi lại phải quản lịch index, permission cấu hình lúc index |
| **OneLake** | Direct | Dữ liệu phi cấu trúc trong Microsoft Fabric lakehouse (BI report, research output) |

Một KB **trộn được nhiều nguồn** (vd SharePoint nội bộ làm chính + web grounding bổ trợ).

**Ba kỹ thuật nâng chất lượng retrieval:** **scoring profiles** (boost field/attribute quan trọng lên đầu), **semantic ranking** (model AI hiểu nghĩa thay vì chỉ keyword), **custom analyzers** (xử lý nội dung đặc thù: HTML, mã sản phẩm, thuật ngữ chuyên ngành).

## 4. Retrieval instructions — hợp đồng hành vi của agent

Cùng câu hỏi "chính sách nghỉ phép?", 3 hành vi có thể xảy ra — chỉ hành vi 3 chấp nhận được:

| Hành vi | Vấn đề |
|---------|--------|
| Trả lời từ training data ("đa số công ty cho 2-3 tuần…") | Sai — không phải chính sách của bạn |
| Search nhưng không cite ("bạn được 15 ngày PTO") | Đúng nhưng **không kiểm chứng được** |
| Search + cite + grounded ("15 ngày 【doc_id:1†Employee Handbook 2024】") | ✓ chuẩn enterprise |

Instruction mơ hồ ("Answer HR questions using the knowledge base") → hành vi bất nhất. Instruction hiệu quả chỉ rõ **3 điều**:

1. **Khi nào retrieve**: "ALWAYS search the knowledge base before answering; NEVER answer from training data"
2. **Cite thế nào**: format chính xác `【doc_id:search_id†source_name】`
3. **Fallback khi không có**: "I don't have that information… contact hr@company.com" (không đoán)

Mẫu theo loại agent: **customer support** (không chắc → "để tôi nối chuyên viên", không đoán); **internal research** (tổng hợp đa nguồn, cite hết, nêu độ tin cậy); **domain expert/compliance** (chỉ trả lời đúng domain, cite đến section + effective date, câu hỏi diễn giải pháp lý → chuyển team compliance).

### Test & giám sát

Test 4 loại query: factual thẳng (→ cite trực tiếp), cần tổng hợp (→ nhiều citation), **ngoài KB** (→ fallback lịch sự), mơ hồ (→ hỏi lại). Response tốt có 4 tính chất: **grounding, citation, relevance, completeness**. Production theo dõi: tần suất citation, tần suất fallback ("I don't know" nhiều = KB thiếu nội dung), loại query phổ biến, retrieval accuracy → lặp cải thiện instructions + nội dung KB.

`★ Insight ─────────────────────────────────────`
So sánh 3 tầng RAG trong hệ Foundry để trả lời "chọn cái nào": **file_search** (vector store nhỏ, tài liệu upload trực tiếp, 1 app) → **Azure AI Search tự quản** (kiểm soát tối đa index/pipeline, tự code retrieval — xem note 19) → **Foundry IQ** (managed, KB dùng chung đa agent, nối qua MCP, tự chọn chiến lược retrieval). Từ khoá đề bài "nhiều agent", "nhiều nguồn dữ liệu", "không muốn quản hạ tầng search" → Foundry IQ.
`─────────────────────────────────────────────────`

## Q&A phỏng vấn

**Q1. Foundry IQ khác gì tự build RAG bằng AI Search?**
→ Foundry IQ là tầng managed trên AI Search: tự lo chunk/embed/index/reindex, tự chọn chiến lược retrieval theo query, trả citations; KB tổ chức theo business domain và **dùng chung cho nhiều agent** — cải thiện một lần, mọi agent hưởng. Tự build thì kiểm soát sâu hơn nhưng mỗi team lặp lại hạ tầng.

**Q2. SharePoint Remote vs SharePoint Indexed?**
→ Remote: query realtime, luôn mới, tự tôn trọng permission SharePoint, không quản index — nhưng search hạn chế, tốc độ phụ thuộc SharePoint. Indexed: preprocess vào AI Search — nhanh hơn, custom analyzer/enrichment/trộn nguồn, nhưng phải quản lịch index và permission cấu hình lúc index.

**Q3. Vì sao phải viết retrieval instructions chi tiết?**
→ Không có thì agent có thể trả lời từ training data (sai), hoặc trả lời đúng mà không cite (không kiểm chứng). Instruction phải chốt: luôn search KB, format citation, fallback khi không tìm thấy.

**Q4. Khi nào bật web grounding trong KB, khi nào tránh?**
→ Bật khi cần thông tin công khai thay đổi nhanh (giá, sự kiện) hoặc làm nguồn bổ trợ. Tránh khi accuracy + kiểm soát nguồn là bắt buộc (dựa Bing → không kiểm soát được nguồn cụ thể).

**Q5. Scoring profile khác semantic ranking?**
→ Scoring profile: boost theo **field/attribute** do bạn định nghĩa (business logic — vd ưu tiên tài liệu mới). Semantic ranking: model AI xếp lại kết quả theo **nghĩa** của query, vượt khỏi khớp keyword.

**Q6. Metric nào cho biết KB đang thiếu nội dung?**
→ **Fallback frequency** cao ("I don't have that information" nhiều) + retrieval accuracy thấp (tài liệu lấy về không chứa đáp án) → bổ sung data source/nội dung cho KB.

## Liên quan
- [[00-MOC-AI-103]] — MOC AI-103
- [[04-Toi-uu-Model-va-Responsible-GenAI]] — RAG cơ bản & grounding
- [[06-Custom-Tools-va-MCP-Tools]] — MCPTool (Foundry IQ nối qua MCP)
- [[19-Knowledge-Mining-AI-Search]] — Azure AI Search tự quản (tầng dưới của Foundry IQ)
- [[../../../04-AI/02-RAG-Optimization/00-MOC-RAG-Optimization|MOC RAG Optimization]] — kỹ thuật tối ưu RAG tổng quát
