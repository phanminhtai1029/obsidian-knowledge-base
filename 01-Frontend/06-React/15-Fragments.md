---
title: "React Fragments"
section: 06-React
tags: [react, fragment, jsx, dom, fresher, frontend]
related:
  - "[[04-JSX-List-Conditional-Rendering]]"
  - "[[02-Component-va-Props]]"
difficulty: ⭐⭐
estimated_time: 15m
source: [React raw transcript]
---

# React Fragments

> [!summary] TL;DR
> JSX yêu cầu **mỗi component chỉ trả về MỘT phần tử gốc** — trả về 2 thẻ cạnh nhau sẽ lỗi *"Adjacent JSX elements must be wrapped in an enclosing tag"*. Cách cũ là bọc trong một `<div>` thừa → làm **rác DOM** (thừa node lồng nhau). **Fragment** là "thẻ bọc ảo" gom nhiều phần tử mà **không tạo node thật trong DOM**. Viết `<React.Fragment>...</React.Fragment>` hoặc gọn hơn là cú pháp rỗng **`<>...</>`**.

---

## 1. Vấn đề: chỉ được trả về một gốc

```jsx
// ❌ Lỗi: 2 phần tử cạnh nhau, không có gốc chung
function App() {
  return (
    <h2>Welcome</h2>
    <main>Content</main>
  );
}
// Error: Adjacent JSX elements must be wrapped in an enclosing tag
```

Cách cũ — bọc `<div>`:
```jsx
function App() {
  return (
    <div>            {/* div thừa, chỉ để gom */}
      <h2>Welcome</h2>
      <main>Content</main>
    </div>
  );
}
```
→ DOM bị "rác" bởi các `<div>` lồng vô nghĩa (div soup).

---

## 2. Giải pháp: Fragment

```jsx
import React from "react";

function App() {
  return (
    <React.Fragment>     {/* gom nhưng KHÔNG tạo node DOM */}
      <h2>Welcome</h2>
      <main>Content</main>
    </React.Fragment>
  );
}
```

Cú pháp rút gọn (phổ biến, **không cần import React**):
```jsx
function App() {
  return (
    <>
      <h2>Welcome</h2>
      <main>Content</main>
    </>
  );
}
```

```
★ Insight ─────────────────────────────────────
• Fragment khác <div> ở chỗ: nó BIẾN MẤT khỏi DOM thật. Quan trọng khi cấu trúc
  cha kén con — vd <tr> chỉ chấp nhận <td>, hay flex/grid layout mà một <div>
  bọc thừa sẽ phá bố cục. Fragment cho bạn gom phần tử mà không thêm cấp lồng.
• Khi nào PHẢI dùng dạng dài <React.Fragment>? Khi cần truyền key (render list
  fragment): <React.Fragment key={id}>. Cú pháp <> </> không nhận được key.
─────────────────────────────────────────────────
```

---

## 3. `<>` vs `<React.Fragment>` vs `<div>`

| | `<div>` | `<React.Fragment>` | `<>...</>` |
|---|---|---|---|
| Tạo node DOM | **Có** (thừa) | Không | Không |
| Nhận `key` (render list) | Có | **Có** | Không |
| Cần import React | Không | Có (bản cũ) | Không |
| Khi dùng | Khi thật sự cần thẻ bọc | Khi cần `key` trong list | Mặc định, gọn nhất |

---

## 4. Q&A phỏng vấn

> [!question] 1. Vì sao JSX báo lỗi "Adjacent JSX elements must be wrapped"?
> Vì component chỉ được **trả về một phần tử gốc**. Hai thẻ cạnh nhau không có gốc chung → lỗi. Phải bọc trong một thẻ hoặc dùng **Fragment**.

> [!question] 2. Fragment khác `<div>` bọc thế nào?
> Fragment gom phần tử **không tạo node DOM thật** (không "rác" DOM, không thêm cấp lồng), còn `<div>` tạo một node thừa. Hữu ích khi cấu trúc cha kén con (`<tr>`/`<td>`) hoặc layout nhạy với thẻ bọc.

> [!question] 3. Khi nào phải dùng `<React.Fragment>` thay vì `<>`?
> Khi cần truyền **`key`** (render danh sách fragment): `<React.Fragment key={id}>`. Cú pháp rút gọn `<>` không nhận `key`.

---

## 5. Bài tập tự luyện

1. Viết component trả về `<h2>` + `<main>` cạnh nhau; sửa lỗi bằng `<>...</>`.
2. Render một list mà mỗi item là 2 thẻ (`<dt>`+`<dd>`) dùng `<React.Fragment key={...}>`.
3. So sánh DOM tree (DevTools) khi bọc bằng `<div>` và bằng Fragment.

---

## 6. Liên kết
- [[04-JSX-List-Conditional-Rendering]] — JSX, render list, `key`
- [[02-Component-va-Props]] — component trả về JSX
- [[00-MOC-React|MOC: React]]
