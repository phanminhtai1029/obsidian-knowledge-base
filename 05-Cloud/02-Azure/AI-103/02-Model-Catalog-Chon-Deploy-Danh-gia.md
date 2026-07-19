---
title: "Model catalog: chọn (benchmark), deploy, đánh giá Foundry Models"
section: 05-Cloud/02-Azure/AI-103
tags: [azure, ai-103, microsoft-foundry, model-catalog, benchmark, evaluation, fresher]
---

# Note 02 — Model catalog: chọn, deploy & đánh giá model

> **TL;DR:** **Foundry Models catalog** có **1.900+ model** chia 2 nhóm: *sold directly by Azure* (bill thẳng vào subscription — gồm Azure OpenAI) và *partners & community* (license/giá riêng, có thể cần Marketplace subscription). Chọn model bằng **benchmark 4 chiều**: quality (Quality index), safety (HarmBench/ToxiGen/WMDP), cost (giá per 1M token, tỉ lệ ước tính input:output = 3:1) và performance (latency, TTFT, throughput). Deploy chủ yếu theo kiểu **Global Standard** (pay-per-token, quota cao nhất); cần throughput ổn định thì dùng **Provisioned (PTU)**; job bất đồng bộ lớn thì **Batch** (giảm 50%, xong trong 24h); cần chủ quyền dữ liệu thì **Data Zone/Regional**. Sau deploy: test ở **playground** → đánh giá bằng metric thủ công + tự động (AI-assisted: groundedness/relevance/coherence/fluency; NLP: F1/BLEU/METEOR/ROUGE).

## 1. Catalog: hai nhóm model & cách lọc

- **Foundry Models sold directly by Azure** — bill qua Azure subscription: model Azure OpenAI, Microsoft và một số provider.
- **Foundry Models from partners and community** — provider tự định license + giá (Meta, Mistral, Cohere, Hugging Face…); khi deploy có thể phải **Agree** điều khoản Azure Marketplace.

Bộ lọc catalog: **Collection** (nhóm nguồn), **Capabilities** (reasoning, tool calling, multimodal), **Source** (OpenAI/Microsoft/Meta/Anthropic…), **Inference task** (text gen, dịch, sinh ảnh…), **Fine-tuning methods**, **Industry** (model chuyên ngành y/luật…). Mỗi model có **model card**: provider, năng lực, benchmark, cân nhắc Responsible AI, kiểu deploy hỗ trợ.

## 2. Phân loại model

| Loại | Đặc điểm | Ví dụ |
|------|----------|-------|
| **LLM** (Large Language Model) | Suy luận sâu, context lớn, tốn tài nguyên | GPT-5, Mistral Large, Llama 3 70B |
| **SLM** (Small Language Model) | Rẻ, nhanh, chạy được edge/hardware yếu | Phi-4, Llama 3 8B |
| **Chat completion** | Sinh text hội thoại thông thường | Đa số model trong catalog |
| **Reasoning model** | Tự suy luận từng bước bên trong — mạnh toán/code/logic | Claude Opus, o-series |
| **Embedding** | Text → vector số phục vụ semantic search / RAG | Ada, Cohere embed |
| **Image / video generation** | Sinh ảnh, video từ text | GPT-image-1, Sora 2 |
| **Image analysis (multimodal input)** | Nhận text + ảnh, trả lời bằng text | GPT-4.1 |
| **TTS / STT** | Text↔speech | GPT-4o-tts, GPT-4o-transcribe |

`★ Insight ─────────────────────────────────────`
Cặp phân biệt hay thi: **LLM vs SLM** (trí tuệ vs chi phí/tốc độ — SLM chạy được on-edge) và **chat model vs reasoning model** (reasoning model *tự* chain-of-thought bên trong nên KHÔNG cần prompt "think step by step" — prompt kiểu đó chỉ dành cho model thường).
`─────────────────────────────────────────────────`

## 3. Benchmark — 4 nhóm chỉ số chọn model

### Quality (chất lượng)
- **Quality index**: điểm trung bình chuẩn hoá 0→1 trên nhiều dataset.
- Dataset tiêu biểu: **Arena-Hard** (QA đối kháng), **BIG-Bench Hard** (suy luận), **GPQA** (câu hỏi sau đại học), **HumanEval+ / MBPP+** (code), **MATH** (toán), **MMLU-Pro** (kiến thức tổng quát), **IFEval** (tuân thủ chỉ dẫn).

### Safety (an toàn)
| Benchmark | Đo gì | Đọc điểm |
|-----------|-------|----------|
| **HarmBench** | Kháng sinh nội dung có hại (3 nhóm: hành vi hại chuẩn, hại theo ngữ cảnh, vi phạm bản quyền) | **ASR** (Attack Success Rate) — càng **thấp** càng an toàn |
| **ToxiGen** | Phát hiện hate speech ngầm/đối kháng | **F1** — càng **cao** càng tốt |
| **WMDP** | Lượng kiến thức nguy hiểm (sinh học/mạng/hoá học) | Điểm cao = model *biết nhiều* kiến thức nhạy cảm |

### Cost (chi phí)
- Giá tính trên **1 triệu token** input và output riêng.
- **Estimated cost** gộp theo tỉ lệ điển hình **3 input : 1 output** → một con số để so sánh.

### Performance (hiệu năng)
- **Latency**: mean, P50/P90/P95/P99 (percentile — P90 = 90% request xong nhanh hơn mốc này).
- **TTFT** (Time To First Token): thời gian tới token đầu tiên khi streaming — quyết định cảm giác "nhanh".
- **Throughput**: **GTPS** (generated tokens/s), **TTPS** (tổng token/s), time between tokens.
- Leaderboard tóm tắt bằng *mean TTFT (thấp tốt)* + *mean GTPS (cao tốt)*.

Công cụ so sánh trong portal: **Model leaderboard** (xếp hạng 4 chiều), **scenario leaderboards** (theo use case: reasoning/code/math/QA/groundedness), **trade-off charts** (2 trục, model gần góc trên-phải là tốt cả hai), **side-by-side comparison** (chọn 2-3 model so chi tiết).

![[model-leaderboard.png]]
*Ảnh: Microsoft Learn — Model leaderboard trong Foundry portal (Discover): panel Quality (index cao tốt) cạnh panel Safety (attack success rate — THẤP tốt), cuộn xuống còn Throughput và Estimated cost. Minh hoạ trực quan "4 chiều benchmark" và chiều đọc điểm ngược nhau giữa Quality và Safety.*

## 4. Deployment types

| Kiểu | Vùng | Tính tiền | Dùng khi |
|------|------|-----------|----------|
| **Global Standard** | Mọi region Azure | Pay-per-token | **Mặc định** — workload chung, **quota cao nhất** |
| **Global Provisioned** | Mọi region | **PTU** (Provisioned Throughput Unit — đơn vị throughput đặt trước) | Cần throughput cao ổn định, dự đoán được |
| **Global Batch** | Mọi region | Pay-per-token **−50%** | Job bất đồng bộ lớn, chấp nhận xong trong **24h** |
| **Data Zone** (Standard/Provisioned/Batch) | Trong **data zone** (EU/US) | Như trên | Cần compliance dữ liệu ở lại EU/US |
| **Standard** (regional) | **1 region** | Pay-per-token | Data residency theo region, volume thấp |
| **Regional Provisioned** | 1 region | PTU | Throughput ổn định + residency |
| **Developer** | Mọi region | Pay-per-token | **Chỉ để đánh giá model đã fine-tune** |

Quy trình deploy: model card → **Deploy** (default/custom settings) → đặt **deployment name** (code sẽ truyền tên này vào tham số `model`) → với managed compute chọn thêm VM SKU + instance count → kiểm tra status **Succeeded**. Quản lý sau deploy ở mục **Build → Models**: endpoint URL, key, metrics.

## 5. Playground & tích hợp vào code

- **Playground**: test model ngay không cần code — thử prompt, chỉnh **system message**, tham số (**temperature** — độ ngẫu nhiên, **max tokens**, **top-p**); tab **Code** sinh sẵn code mẫu (Python/C#/JS) đã điền endpoint + deployment name.
- Tích hợp app cần đủ bộ ba: **endpoint URL** (project endpoint hoặc OpenAI v1 endpoint) + **auth** (key, hoặc **Entra ID** — khuyến nghị production) + **deployment name**.

## 6. Đánh giá model (evaluation)

### Thủ công (manual)
- **Interactive testing** trong playground (kể cả **test 2 model side-by-side** cùng system prompt).

![[playground-compare-models.png]]
*Ảnh: Microsoft Learn — Compare models trong playground: gpt-4.1-mini và gpt-4.1 trả lời CÙNG một prompt cạnh nhau, mỗi bên có Setup/Chat/Code riêng và nút "Save as agent".*
- **Structured review**: bộ test case + người chấm theo thang (1-5) trên tiêu chí *relevance, informativeness, engagement, accuracy, safety*.
- **User studies**: người dùng thật phản hồi.

### Tự động (automated)
**AI-assisted metrics** (chỉ định một model GPT làm "giám khảo" — evaluator model):

| Nhóm | Metric |
|------|--------|
| Generation quality | **Groundedness** (trả lời bám context hay bịa — bản *Groundedness Pro* chấm nhị phân), **Relevance**, **Coherence** (mạch lạc), **Fluency** (đúng ngữ pháp, tự nhiên) |
| Risk & safety | Self-harm, hateful/unfair, violent, sexual, protected material (nội dung bản quyền), **indirect attack/jailbreak**. Kết quả gộp thành **defect rate** = % response vượt ngưỡng severity (thường Medium) |

**NLP metrics** (toán học, cần **ground truth** — đáp án chuẩn để so):

| Metric | Đặc trưng | Hợp với |
|--------|-----------|---------|
| **F1-score** | Tỉ lệ từ trùng, cân bằng precision/recall | Phân loại, information retrieval |
| **BLEU** | So n-gram với bản tham chiếu | Dịch máy |
| **METEOR** | BLEU + synonym/stem/paraphrase | Dịch máy (mềm dẻo hơn) |
| **ROUGE** | Thiên về recall | **Tóm tắt** |
| **GLEU** | BLEU cấp câu | Đánh giá từng câu |

### Chạy evaluation trong portal
- Đối tượng: **Model** (tự sinh output khi chạy) / **Agent** / **Dataset** (output có sẵn).
- Dữ liệu test: upload CSV/JSONL, dùng dataset có sẵn, hoặc **generate synthetic dataset** (mô tả topic → hệ thống tự sinh).
- **Evaluator library**: xem/quản lý evaluator Microsoft-curated + custom, có version management.
- Điểm thấp → cải thiện theo thang: **prompt engineering → đổi model → RAG → fine-tuning** (độ phức tạp & chi phí tăng dần — xem [[04-Toi-uu-Model-va-Responsible-GenAI]]).

`★ Insight ─────────────────────────────────────`
Phân biệt 2 tầng đánh giá: **benchmark** (điểm công khai, so sánh model *trước khi chọn*) vs **evaluation** (chạy trên *dữ liệu & prompt của bạn sau khi deploy*). Benchmark tốt không bảo đảm hợp use case — nên thi hay hỏi "chọn model xong rồi làm gì để biết nó đạt yêu cầu?" → chạy evaluation với dataset riêng.
`─────────────────────────────────────────────────`

## Q&A phỏng vấn

**Q1. Kiểu deployment nào nên dùng mặc định và vì sao?**
→ **Global Standard**: pay-per-token, dùng mọi region, quota cao nhất, đủ cho đa số workload. Chỉ chuyển Provisioned khi cần throughput bảo đảm (PTU), Batch khi job lớn không cần realtime (rẻ hơn 50%), Data Zone/Regional khi có yêu cầu compliance dữ liệu.

**Q2. Groundedness đo cái gì, khác Relevance thế nào?**
→ Groundedness: câu trả lời có **bám vào context được cung cấp** không hay tự bịa (chống hallucination). Relevance: câu trả lời có **đúng câu hỏi** không. Một câu trả lời có thể relevant nhưng không grounded (đúng chủ đề nhưng bịa số liệu).

**Q3. Khi nào dùng ROUGE thay vì BLEU?**
→ ROUGE thiên recall → hợp **tóm tắt** (quan trọng là phủ đủ ý chính). BLEU so n-gram thiên precision → hợp **dịch máy**. Cả hai đều cần ground truth, không hợp bài sinh nội dung mở.

**Q4. ASR trong HarmBench nghĩa là gì, cao hay thấp thì tốt?**
→ Attack Success Rate — tỉ lệ tấn công khiến model sinh nội dung có hại thành công. **Thấp** thì model an toàn hơn.

**Q5. PTU là gì?**
→ Provisioned Throughput Unit — đơn vị throughput **đặt trước** trả tiền cố định, đổi lại throughput cao và ổn định (không bị nghẽn theo quota chia sẻ như pay-per-token). Dùng cho production traffic lớn, đều.

**Q6. Điểm evaluation thấp thì cải thiện theo thứ tự nào?**
→ Prompt engineering (rẻ nhất) → thử model khác → thêm RAG (khi thiếu kiến thức domain) → fine-tune (khi cần độ nhất quán mà prompt không đạt). Safety kém thì thêm content filters, prompt hardening, output validation.

## Liên quan
- [[00-MOC-AI-103]] — MOC AI-103
- [[01-Microsoft-Foundry-Tong-quan-Plan-Prepare]] — kiến trúc Foundry
- [[03-Chat-App-Foundry-SDK-va-Tools]] — dùng model đã deploy trong code
- [[04-Toi-uu-Model-va-Responsible-GenAI]] — tối ưu khi evaluation chưa đạt
