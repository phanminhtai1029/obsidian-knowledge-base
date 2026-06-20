---
title: "MOC: RAG & Optimization"
section: 04-AI/02-RAG-Optimization
tags: [moc, ai, rag, hnsw, bm25, hyde, graphrag, neo4j, fresher]
module: L2_AI_RAGO
---

# MOC: AI Advanced — RAG & Optimization

> Module `L2_AI_RAGO`. Thi: Part 1 Theory + Part 2 Practical (Coding).
> ✅ Đã viết nội dung — bám sát học liệu gốc (`_shared/_source/04-AI/02-RAG-Optimization`).

## Note

| # | Note | Nội dung | Độ khó | Trạng thái |
|---|------|----------|--------|------------|
| 1 | [[01-Advanced-Indexing\|Advanced Indexing]] | Semantic vs Recursive Chunking; HNSW (graph nhiều tầng) & 3 tham số **M / ef_construction / ef_search** | ⭐⭐⭐⭐ | ✅ |
| 2 | [[02-Hybrid-Search\|Hybrid Search]] | BM25 (TF Saturation, IDF, Length Norm) vs TF-IDF; Sparse+Dense; **RRF** (rank, k≈60) | ⭐⭐⭐⭐ | ✅ |
| 3 | [[03-Query-Transformation\|Query Transformation]] | **HyDE** (bất đối xứng ngữ nghĩa), **Query Decomposition**, Multi-Query, Step-back | ⭐⭐⭐ | ✅ |
| 4 | [[04-Post-Retrieval-Processing\|Post-Retrieval Processing]] | **Bi vs Cross-Encoder**, **Funnel** (50→5), **MMR** (relevance ↔ diversity, λ) | ⭐⭐⭐⭐ | ✅ |
| 5 | [[05-GraphRAG-Implementation\|GraphRAG Implementation]] | Pydantic `with_structured_output`, Neo4j MERGE, Cypher, **GraphCypherQAChain** (Practical/coding) | ⭐⭐⭐⭐⭐ | ✅ |

> 📒 Thuật ngữ tra cứu nhanh: [[../01-AI-Fundamentals-RAG/02-RAG-Theoretical-Foundations#Glossary mở rộng (tra cứu nhanh — gom từ cả 4 môn AI)|Glossary mở rộng]].

## Lab/Assignment trong khung
- Build Semantic Chunker & HNSW Vector DB
- Combine Vector Search with BM25 & Apply RRF
- Build HyDE Prompts & Query Decomposition Loop
- Cross-Encoder Pipeline & MMR Retrieval
- FSoft_HR.pdf Extraction & GraphCypherQAChain

## Liên quan
- [[../01-AI-Fundamentals-RAG/00-MOC-AI-Fundamentals-RAG|MOC: AI Fundamentals & RAG]]
- [[../00-MOC-AI|MOC: AI]]
