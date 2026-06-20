---
title: "MOC: AWS Bedrock"
section: 05-Cloud/01-AWS-Bedrock
tags: [moc, cloud, aws, bedrock, s3-vectors, langchain-aws, fresher]
---

# MOC: AWS Bedrock (bám đề thi)

> Hạ tầng AI mà khung chương trình thực tế dùng. Lab setup: "get the API key from Bedrock",
> `langchain-aws`, `Amazon S3 Vectors`.
> ⚠️ Khung — note chi tiết viết sau.

## Note dự kiến

| # | Note | Nội dung dự kiến | Trạng thái |
|---|------|------------------|------------|
| 1 | [[01-AWS-Bedrock-Overview\|AWS Bedrock Overview]] | Bedrock là gì, các foundation model, region | 🔜 |
| 2 | [[02-Bedrock-Setup-API-Key\|Setup & API Key]] | IAM, lấy API key, credentials | 🔜 |
| 3 | [[03-Bedrock-LLM-APIs\|Bedrock LLM APIs]] | Gọi model qua boto3 / langchain-aws | 🔜 |
| 4 | [[04-S3-Vectors-VectorStore\|Amazon S3 Vectors]] | Vector store trên S3, `AmazonS3Vectors` | 🔜 |
| 5 | [[05-langchain-aws-Integration\|langchain-aws Integration]] | ChatBedrock, embeddings, tích hợp RAG | 🔜 |

## Tham khảo
- https://docs.langchain.com/oss/python/integrations/providers/aws
- https://reference.langchain.com/python/langchain-aws/vectorstores/s3_vectors

## Liên quan
- [[../00-MOC-Cloud|MOC: Cloud]]
- [[../../04-AI/00-MOC-AI|MOC: AI]]
