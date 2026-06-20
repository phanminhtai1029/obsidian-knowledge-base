---
title: "MOC: LangGraph & Agentic AI"
section: 04-AI/04-LangGraph-Agentic
tags: [moc, ai, langgraph, agentic, agent, tool-calling, multi-agent, fresher]
module: L2_AI_LGAA
---

# MOC: LangGraph and Agentic AI

> Module `L2_AI_LGAA`. Thi: Part 1 Theory + Part 2 Practical (Coding) + Audit.
> ✅ Đã viết nội dung — bám sát học liệu gốc (`_shared/_source/04-AI/04-LangGraph-Agentic`).

## Note

| # | Note | Nội dung | Độ khó | Trạng thái |
|---|------|----------|--------|------------|
| 1 | [[01-LangGraph-Foundations-State\|LangGraph Foundations & State Management]] | StateGraph, **messages-centric**, `add_messages` reducer, Node/Edge, conditional routing, invoke vs stream | ⭐⭐⭐⭐ | ✅ |
| 2 | [[02-Agentic-Patterns\|Agentic Patterns]] | **ReAct** (Reason+Act+Observe), Multi-Expert tools, `ToolNode` prebuilt, Reflection/Planning, "1 node ≤ 2 việc" | ⭐⭐⭐⭐ | ✅ |
| 3 | [[03-Tool-Calling-Tavily\|Tool Calling & Tavily Search]] | Tool/Function calling, `@tool` + `bind_tools`, **Tavily** (AI-optimized search), parallel/cache/security | ⭐⭐⭐ | ✅ |
| 4 | [[04-Multi-Agent-Collaboration\|Multi-Agent Collaboration]] | 4 mẫu phối hợp; **Hierarchical/Supervisor**, **dialog stack** (push/pop), context injection, **CompleteOrEscalate**, safe vs sensitive tools | ⭐⭐⭐⭐⭐ | ✅ |
| 5 | [[05-Human-in-the-Loop-Persistence\|Human-in-the-Loop & Persistence]] | **Interrupt** (before/after), checkpointer (**Memory/SQLite/Postgres**), thread_id, time-travel, short vs long-term memory | ⭐⭐⭐⭐ | ✅ |

> 📒 Thuật ngữ tra cứu nhanh: [[../01-AI-Fundamentals-RAG/02-RAG-Theoretical-Foundations#Glossary mở rộng (tra cứu nhanh — gom từ cả 4 môn AI)|Glossary mở rộng]].

## Lab/Assignment trong khung
- Hello World Graph + visualize (Mermaid / print_ascii)
- Multi-Expert ReAct Agent
- Tool Calling
- Multi-Agent System
- Persistent & HITL Agent

## Liên quan
- [[../03-LLMOps-Evaluation/00-MOC-LLMOps-Evaluation|MOC: LLMOps & Evaluation]]
- [[../00-MOC-AI|MOC: AI]]
