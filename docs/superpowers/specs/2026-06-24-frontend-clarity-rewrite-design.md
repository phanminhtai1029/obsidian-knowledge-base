# Frontend Clarity Rewrite + Gap Supplement — Design & Progress

**Ngày bắt đầu:** 2026-06-24
**Mục tiêu:** (1) Viết lại toàn bộ ~74 note Frontend dễ hiểu/dễ nhớ hơn mà GIỮ NGUYÊN kiến thức; (2) Đối chiếu 18 bộ câu hỏi phỏng vấn (`_shared/_source/example_question/`) với vault → tự bổ sung kiến thức thiếu.

## "Tầng Trực Giác" (Clarity Layer) — chèn vào mỗi note, không xoá gì cũ

1. **🎯 Hiểu trong 30 giây** (ngay sau TL;DR): ví von đời thường + "vấn đề nó giải quyết" bằng tiếng Việt thường.
2. **Chú thích code dễ thở** — comment "đang xảy ra gì" ở các đoạn code chính.
3. **🗣️ Trả lời phỏng vấn mẫu** — đoạn văn nói thành lời tự nhiên (kịch bản để thuộc & nói ra được).
4. **🧠 Mẹo nhớ** — 1 câu chốt/mnemonic.

Giữ nguyên: TL;DR, các section kiến thức, ★ Insight, bảng phân biệt, Pitfalls, Q&A gạch đầu dòng, Bài tập, Liên kết.

## Thứ tự cụm (commit + push từng cụm)

- [x] **Cụm 1 — Advanced JS** (03-Advanced-JavaScript, 11 note): Scope, Hoisting, Closure, Coercion, ES6 let/const, Template Literals, Enhanced Object Literals, Destructuring, Arrow Function, Rest/Spread, Class ✅ (commit 2026-06-24)
- [x] **Cụm 2 — Async JS** (04-Async-JavaScript, 5 note nội dung): Overview, Event Loop, Callback/Hell, Promise, Async-Await ✅ (commit 2026-06-24) — MOC là index, không cần tầng trực giác
- [x] **Cụm 3 — React/Next.js** (06-React, 18 note + note MỚI 19) ✅ HOÀN TẤT (commit 2026-06-24). Tất cả 18 note gốc + 19-Performance(memo/useMemo/useCallback). TL;DR đã Việt hoá thuật ngữ theo [[note-jargon-gloss-feedback]].
- [ ] **Cụm 4 — DOM-Event** (02-DOM-Event, 14 note)
- [ ] **Cụm 5 — TypeScript** (05-TypeScript, 8 note)
- [ ] **Cụm 6 — HTML-CSS + Bootstrap** (01-HTML-CSS, 18 note)

## Gap analysis — kiến thức thiếu cần bổ sung (đối chiếu đề)

Sẽ cập nhật trong khi làm; mỗi gap ghi rõ note đích + commit.

**Đã bổ sung:**
- [x] `React.memo`/`useMemo`/`useCallback` (tối ưu re-render) → note MỚI `06-React/19-Performance-memo-useMemo-useCallback.md` + thêm vào MOC. (đề NamPV26 Q2, TinNA2 Q2, TriLHD2 Q2)
- [x] **Debouncing & Throttling** (khái niệm) → bổ sung mục mới vào `02-DOM-Event/13-DOM-Performance.md` (đề NhaNTT60 Q2). Trước đó chỉ có ví dụ rải rác, chưa giải thích khái niệm.

**Ứng viên ban đầu (chưa xử lý hết):**
- FE: State Batching, Virtual DOM (chi tiết reconciliation), `key={index}` bad practice, SPA refresh 404, render Markdown an toàn + XSS, Debouncing/Throttling, Shallow vs Deep copy, `Promise.allSettled`, Optional chaining/Nullish, Lazy loading/code splitting.
- Backend: SSE vs WebSocket cho chatbot streaming, Rate limiting, Connection pool exhausted, 401 vs 403 (rõ ràng), BackgroundTasks.
- AI/RAG: đồng bộ dữ liệu RAG bị "lạc hậu" (stale embedding pipeline), Circuit Breaker chống agent loop vô hạn, RBAC trong RAG, abbreviation handling, Self-Querying/Query Routing, CoT vì sao tăng accuracy, Tool token security (không lộ token cho LLM).
- DevOps: secrets trong CI/CD, alpine image, docker network/service name.

## Kỳ vọng

Khối lượng lớn → làm cuốn chiếu nhiều phiên. File này là nguồn theo dõi tiến độ.
