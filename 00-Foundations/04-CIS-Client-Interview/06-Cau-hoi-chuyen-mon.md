---
title: "Câu hỏi chuyên môn (Technical) — Concept · Application · Example"
section: 00-Foundations/04-CIS-Client-Interview
tags: [cis, interview, technical, concept-example, soft-skill, fresher]
related:
  - "[[02-Ba-loai-cau-hoi-va-cau-truc]]"
  - "[[07-Cau-hoi-quan-diem-PREP]]"
  - "[[../02-Git/00-MOC-Git]]"
difficulty: ⭐⭐⭐
estimated_time: 30m
source: ["9. Answer in structure - Template.docx", Source-CIS.docx]
---

# Câu hỏi chuyên môn (Technical)

> [!summary] TL;DR
> Câu hỏi technical kiểm tra bạn **hiểu sâu & dùng được** kiến thức. Trả lời theo **Concept → Application → Example**: định nghĩa/giải thích khái niệm cho đúng → nói nó dùng để làm gì / vì sao quan trọng → ví dụ thật từ dự án của bạn. Trong PV khách hàng, đừng "đọc định nghĩa sách giáo khoa" — hãy giải thích bằng lời của mình rồi **gắn vào kinh nghiệm thực tế**.

---

## 1. Cấu trúc Concept – Application – Example ⭐⭐⭐

| Bước | Trả lời gì | Bí quyết |
|------|------------|----------|
| **Concept** | Định nghĩa/giải thích rõ, đúng | Nói bằng lời của mình, không học vẹt |
| **Application** | Dùng để làm gì, vì sao quan trọng | Cho thấy bạn hiểu *mục đích*, không chỉ thuộc lòng |
| **Example** | Ví dụ thật từ kinh nghiệm | Biến lý thuyết thành "tôi đã dùng nó" |

```
★ Insight ─────────────────────────────────────
• Ví dụ (Example) là thứ phân biệt fresher "học thuộc" với fresher "làm thật".
  Khách hàng PV để vào dự án → câu trả lời technical có ví dụ từ dự án bạn làm
  ăn điểm gấp đôi định nghĩa suông ([[01-Tong-quan-CIS]]).
• Nếu không chắc câu hỏi, hãy CLARIFY trước khi trả lời ("Do you mean X in
  context of Y?") — vừa tránh trả lời lạc đề, vừa thể hiện kỹ năng giao tiếp
  ([[../05-LEAP-Communication/05-Paraphrase|LEAP-Paraphrase]]).
• Đừng "bịa" khi không biết. Nói thật + cách bạn sẽ tìm hiểu ("I haven't used
  it in production, but I understand the concept as… and I'd learn it by…")
  ăn điểm thái độ hơn là trả lời sai.
─────────────────────────────────────────────────
```

---

## 2. Bài mẫu của bạn — "version control" (Source-CIS.docx)

**Câu hỏi:** *"Can you explain the concept of version control and why it's important?"*

> **[Concept]** *"Version control is a system that tracks changes to source code over time, allowing multiple developers to collaborate while maintaining a history of modifications."*
> **[Application]** *"It helps teams manage parallel development, resolve conflicts, and revert to previous stable versions when necessary."*
> **[Example]** *"When using Git in a team project, each developer works on separate branches. After completing a feature, changes are merged into main through pull requests, ensuring code review and controlled integration. Without version control, tracking changes or restoring previous versions would be significantly harder."*

→ ✅ Đủ 3 phần; Example dùng đúng thuật ngữ thật (branch, PR, merge, code review). Liên hệ kiến thức nền: [[../02-Git/00-MOC-Git|MOC Git]].

---

## 3. Các câu technical hay gặp (chuẩn bị sẵn)

| Câu hỏi | Liên kết ôn |
|---------|-------------|
| Version control là gì & vì sao quan trọng? | [[../02-Git/01-How-Git-Works]] |
| **Synchronous vs Asynchronous** processing? | (ôn: blocking vs non-blocking, đợi vs không đợi) |
| Giải thích **scalability** trong hệ thống phần mềm? | (vertical vs horizontal scaling) |
| **Scrum vs Waterfall** khác nhau? | [[../03-Agile-Scrum/01-Tong-quan-Agile-Scrum]] |

> [!tip] Với fresher AI, chuẩn bị thêm các khái niệm sát JID của bạn: **RAG**, **AI agent**, **embedding/vector DB**, **prompt engineering**, **fine-tuning vs RAG** — mỗi cái theo Concept-Application-Example với ví dụ từ 3 dự án của bạn.

---

## 4. Tự kiểm tra

1. Cấu trúc trả lời technical? *(Concept → Application → Example)*
2. Vì sao Example quan trọng trong PV khách hàng? *(chứng minh làm thật, không chỉ học thuộc)*
3. Khi không chắc câu hỏi nên làm gì? *(clarify/xác nhận lại trước khi trả lời)*
4. Khi không biết một khái niệm thì sao? *(nói thật + cách sẽ tìm hiểu, không bịa)*
5. "Sync vs async" cốt lõi khác nhau ở đâu? *(đợi/blocking vs không đợi/non-blocking)*

## Liên quan
- [[00-MOC-CIS|⬅ MOC CIS]]
- Trước: [[05-Cau-hoi-hanh-vi-STAR]] · Kế tiếp: [[07-Cau-hoi-quan-diem-PREP|Câu hỏi quan điểm (PREP)]]
- [[../02-Git/00-MOC-Git|MOC Git]] · [[../03-Agile-Scrum/00-MOC-Agile-Scrum|MOC Scrum]] (kho kiến thức technical)
