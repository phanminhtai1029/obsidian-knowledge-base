---
title: "Language MCP server & Speech MCP server cho agents"
section: 05-Cloud/02-Azure/AI-103
tags: [azure, ai-103, mcp, azure-language, azure-speech, agents, fresher]
---

# Note 12 — Azure Language MCP server & Azure Speech MCP server

> **TL;DR:** Azure cung cấp sẵn 2 **MCP server** (public preview) biến năng lực Foundry Tools thành tool cho agent — không phải viết tích hợp riêng từng API. **Language MCP server** (`…cognitiveservices.azure.com/language/mcp?api-version=…`) expose: language detection, NER, **PII redaction**, **Text Analytics for Health**. **Speech MCP server** (`…cognitiveservices.azure.com/speech/mcp`) expose: **speech-to-text (Recognize)** + **text-to-speech (Synthesize)** — vì làm việc với file audio nên **bắt buộc có Azure Storage** (blob container + **SAS URL** với quyền read/add/create/write/list; SAS phải coi như secret). Cả hai nối vào agent qua Tools page trong portal (auth **key-based** — header `Ocp-Apim-Subscription-Key`) hoặc bằng code với `MCPTool`. Agent **tự chọn tool** theo mô tả (dynamic tool discovery), gọi nhiều tool một lượt được; client app gọi agent qua Responses API với **`agent_reference`** trong `extra_body`.

## 1. Nhắc lại kiến trúc MCP (host – client – server)

| Thành phần | Vai trò |
|-----------|---------|
| **Host** | App chạy agent (Microsoft Foundry hoặc custom app) |
| **Client** | Thành phần trong host quản lý kết nối tới MCP server |
| **Server** | Chương trình expose tools/resources/prompts cho agent khám phá & gọi |

Agent nối server → nhận **catalog tool + mô tả** → **tự chọn tool** theo prompt (dynamic tool discovery — không cần routing logic). Tool thêm/sửa/xoá ở server, agent không đổi.

## 2. Azure Language MCP server

**Endpoint:** `https://{foundry-resource-name}.cognitiveservices.azure.com/language/mcp?api-version=2025-11-15-preview` (có cả bản local tự host).

| Tool | Làm gì |
|------|--------|
| Language Detection | Nhận diện ngôn ngữ văn bản |
| Named Entity Recognition | Bóc thực thể (people, places, orgs, dates, quantities) |
| PII Redaction | Tìm + che thông tin cá nhân |
| **Text Analytics for Health** | Bóc **thực thể y khoa** (chẩn đoán, thuốc, triệu chứng) từ văn bản lâm sàng |

- **Multi-task một lượt:** prompt "cho biết bài này viết tiếng gì và nhắc đến ai" → agent gọi cả `detect_language_from_text` lẫn `extract_named_entities_from_text` rồi tổng hợp một câu trả lời.
- Lần đầu dùng tool sẽ có **approval prompt** (duyệt 1 lần hoặc "Always approve"); pane **Logs** cho xem từng tool call (input + kết quả).

### Kết nối (2 cách)
1. **Portal**: Tools page → Connect a tool → "Azure Language in Foundry Tools" → điền Foundry resource name + auth **Key-based** (credential = **`Ocp-Apim-Subscription-Key`** — key của project) → Use in an agent. Sau đó **nhớ sửa instructions** của agent: "Use the Azure Language tool to perform text analysis tasks."
2. **Code**:
```python
from azure.ai.projects.models import MCPTool
mcp_tool = MCPTool(
    server_label="azure-language",
    server_url="https://{res}.cognitiveservices.azure.com/language/mcp?api-version=2025-11-15-preview",
    require_approval="always")   # + allowed_tools=[...] để giới hạn tool
```

### Client app gọi agent

```python
response = openai_client.responses.create(
    input=[{"role": "user", "content": user_prompt}],
    extra_body={"agent_reference": {"name": "Text-Analysis-Agent", "type": "agent_reference"}})
print(response.output_text)   # response.model_dump_json() để soi tool nào đã được gọi
```

## 3. Azure Speech MCP server

**Endpoint:** `https://{foundry-resource-name}.cognitiveservices.azure.com/speech/mcp`

| Tool | Làm gì | Tuỳ chọn (ghi ngay trong prompt) |
|------|--------|----------------------------------|
| **Speech-to-text (Recognize)** | Audio file → text (WAV/MP3/OGG/FLAC/MP4/M4A/AAC…) | Ngôn ngữ, **phrase hints** (thuật ngữ domain tăng độ chính xác), profanity filtering (mask/remove/raw), output detailed/simple |
| **Text-to-speech (Synthesize)** | Text → audio file giọng neural | Voice (vd `en-GB-SoniaNeural`), ngôn ngữ, format WAV/MP3 |

### Yêu cầu Storage — điểm khác biệt với Language MCP

Vì làm việc với **file audio**:
- **TTS**: file audio sinh ra được lưu vào **Blob container**; response trả **link** tới file.
- **STT**: transcribe từ URL công khai hoặc từ blob qua **SAS URL**.
- Khi connect phải cung cấp **SAS URL** của container với quyền **Read, Add, Create, Write, List** (header `X-Blob-Container-Url`).

> ⚠️ **SAS URL là secret**: expiry ngắn nhất có thể, scope một container duy nhất, không nhét vào source code/agent prompt/chat transcript; lưu secret store, rotate định kỳ và rotate ngay nếu nghi lộ.

**Prerequisites**: subscription + Foundry resource & project (role Contributor/Owner trên resource group) + Storage account + SAS URL.

### Test trong playground
- TTS: `Generate "To be or not to be…" as speech` → nhận link audio trong blob. Chỉ định giọng ngay trong prompt: `…using the voice "en-GB-SoniaNeural"`.
- STT: `Transcribe the file at https://…/meeting-recording.wav` → nhận text.

`★ Insight ─────────────────────────────────────`
Mẫu chung của cả hai server (và là ý nghĩa chiến lược của chúng): **Foundry Tools "agent hoá" qua MCP** — thay vì dev viết code SDK từng dịch vụ (note 11, 13), agent gọi thẳng dịch vụ như tool bằng ngôn ngữ tự nhiên. Điểm phân hoá khi thi: Language MCP chỉ cần key; **Speech MCP cần thêm Storage + SAS URL** vì có file audio vào/ra.
`─────────────────────────────────────────────────`

## Q&A phỏng vấn

**Q1. Vì sao dùng Language MCP server thay vì gọi SDK `azure-ai-textanalytics` trực tiếp?**
→ Một kết nối tool duy nhất cho mọi năng lực; agent tự chọn tool theo prompt (không viết routing); tool cập nhật ở server không đụng agent; user tương tác bằng ngôn ngữ tự nhiên. SDK trực tiếp hợp khi app cần kiểm soát chi tiết/không có agent.

**Q2. Speech MCP server cần gì mà Language MCP không cần? Vì sao?**
→ **Azure Storage account + SAS URL** (read/add/create/write/list) — vì tool làm việc với file audio: TTS ghi file ra container, STT đọc file từ URL/blob. Text thuần thì không cần.

**Q3. Auth khi connect 2 MCP server này trong portal?**
→ **Key-based**: header `Ocp-Apim-Subscription-Key` = key của Foundry project; Speech thêm `X-Blob-Container-Url` = SAS URL.

**Q4. Client app tham chiếu agent thế nào khi gọi Responses API?**
→ Qua `extra_body={"agent_reference": {"name": "<tên agent>", "type": "agent_reference"}}` — không phải truyền endpoint agent vào `model`.

**Q5. Prompt yêu cầu 2 việc (phát hiện ngôn ngữ + bóc entity) thì agent xử lý sao?**
→ Gọi **nhiều tool trong một lượt** — mỗi call đi qua MCP server độc lập, agent tổng hợp kết quả thành một câu trả lời.

**Q6. Text Analytics for Health là gì?**
→ Tool trong Language MCP server bóc **thực thể y khoa** (chẩn đoán, thuốc, triệu chứng) từ văn bản lâm sàng — use case healthcare đặc thù.

## Liên quan
- [[00-MOC-AI-103]] — MOC AI-103
- [[06-Custom-Tools-va-MCP-Tools]] — MCP & MCPTool tổng quát
- [[11-Azure-Language-Text-Analysis]] — các năng lực Language phía sau server
- [[13-Speech-GenAI-va-Voice-Live-API]] — Azure Speech SDK & Voice Live
