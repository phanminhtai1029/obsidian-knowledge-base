---
title: "Azure OpenAI Service (đối chiếu AWS Bedrock)"
section: 05-Cloud/02-Azure
tags: [azure, ai-azure, azure-openai, llm, deployment, bedrock, fresher]
related:
  - "[[17-Azure-AI-Search]]"
  - "[[10-Identity-Security-AzureAD-RBAC]]"
  - "[[../../01-AWS-Bedrock/00-MOC-AWS-Bedrock]]"
difficulty: ⭐⭐⭐
estimated_time: 25m
source: ["Microsoft Learn — Azure OpenAI", "tham khảo web"]
---

# Azure OpenAI Service

> [!summary] TL;DR
> **Azure OpenAI Service** là dịch vụ PaaS cho phép gọi các model OpenAI (GPT-4o, GPT-4, embeddings…) **qua hạ tầng Azure** — kèm bảo mật doanh nghiệp, mạng riêng, compliance. Khái niệm cốt lõi: phải tạo một **deployment** (đặt tên cho một model cụ thể) rồi gọi qua **endpoint + API key** *hoặc* **Entra ID / managed identity** (an toàn hơn). Đây là **tương đương Azure của AWS Bedrock** — cùng vai trò "cổng gọi foundation model có quản trị". 🎯 Đề thi FresherAI dùng **Bedrock**; phần này học thêm để đối chiếu.

---

## 1. Azure OpenAI vs OpenAI vs Bedrock

| | OpenAI API (openai.com) | Azure OpenAI | AWS Bedrock |
|---|---|---|---|
| Hạ tầng | OpenAI | **Azure** (region, VNet, RBAC) | **AWS** |
| Model | OpenAI | OpenAI (qua Azure) | Nhiều hãng (Anthropic, Meta, Amazon…) |
| Bảo mật DN | cơ bản | Entra ID, private endpoint, compliance | IAM, VPC |
| Khái niệm gọi | model name | **deployment name** | model ID |
| Vai trò | API LLM | "Bedrock của Azure" | "Azure OpenAI của AWS" |

---

## 2. Quy trình dùng

1. Tạo **Azure OpenAI resource** (trong resource group, chọn region).
2. Tạo **deployment**: chọn model (vd `gpt-4o`) → đặt **deployment name**.
3. Lấy **endpoint** + **API key** (hoặc dùng **managed identity** — khỏi lưu key, an toàn hơn → [[10-Identity-Security-AzureAD-RBAC]]).
4. Gọi qua SDK.

```python
# SDK openai >=1.x, dùng AzureOpenAI
from openai import AzureOpenAI

client = AzureOpenAI(
    azure_endpoint="https://<resource>.openai.azure.com/",
    api_key="<API_KEY>",            # hoặc dùng Entra ID token
    api_version="2024-10-21",
)
resp = client.chat.completions.create(
    model="<deployment_name>",       # LƯU Ý: tên DEPLOYMENT, không phải tên model
    messages=[{"role": "user", "content": "Xin chào"}],
)
print(resp.choices[0].message.content)
```

> [!question] Phỏng vấn: "Azure OpenAI khác gọi OpenAI trực tiếp ở đâu?"
> Azure OpenAI chạy **trên hạ tầng Azure** → thêm bảo mật doanh nghiệp (Entra ID/managed identity, private endpoint, RBAC, compliance khu vực) và **SLA**. Khác biệt kỹ thuật rõ nhất: gọi bằng **deployment name** (bạn tự đặt khi deploy model) thay vì tên model trực tiếp.

> [!question] Phỏng vấn: "Đối chiếu Azure OpenAI với AWS Bedrock?"
> Cùng vai trò: **cổng gọi foundation model có quản trị** của mỗi cloud. Bedrock đa nhà cung cấp model (Anthropic Claude, Meta Llama, Amazon Titan…), Azure OpenAI tập trung model OpenAI. Cả hai đều tích hợp IAM/identity, VPC/VNet, và LangChain (`langchain-aws` ↔ `langchain-openai`/`AzureChatOpenAI`).

---

```
★ Insight ─────────────────────────────────────
• "deployment name ≠ model name" là cái bẫy số 1 khi mới dùng Azure
  OpenAI — gọi bằng tên deployment bạn tự đặt, không phải "gpt-4o".
• Ưu tiên managed identity hơn API key: không có secret nằm trong code/
  env → giảm hẳn rủi ro lộ khoá (đúng tinh thần Zero Trust).
• Map kiến thức sang Bedrock: deployment↔model ID, endpoint+key↔IAM
  credential, AzureChatOpenAI↔ChatBedrock. Hiểu một bên là suy ra bên kia.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. "Deployment" trong Azure OpenAI là gì, gọi model bằng tên gì?
2. Hai cách xác thực khi gọi Azure OpenAI? Cách nào an toàn hơn, vì sao?
3. Azure OpenAI khác OpenAI API trực tiếp ở những điểm nào?
4. Map các khái niệm Azure OpenAI ↔ AWS Bedrock.

---

## Liên quan
- [[17-Azure-AI-Search]] — vector store cho RAG trên Azure
- [[10-Identity-Security-AzureAD-RBAC]] — managed identity gọi service an toàn
- [[../../01-AWS-Bedrock/00-MOC-AWS-Bedrock]] — nhánh bám đề thi (đối chiếu)
