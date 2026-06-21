---
title: "React: Tổng quan"
section: 06-React
tags: [react, overview, virtual-dom, vite, compiler, fresher, frontend]
related:
  - "[[02-Component-va-Props]]"
  - "[[03-State-voi-useState]]"
difficulty: ⭐⭐
estimated_time: 30m
source: [React.docx, react.dev]
---

# React: Tổng quan

> [!summary] TL;DR
> **React** là JavaScript **library** (không phải framework) do Facebook tạo ra, open source năm 2013, dùng để build **UI theo kiến trúc component**. Cơ chế hoạt động: JSX → React Elements → **Virtual DOM** → diff với real DOM → chỉ update những gì thay đổi. Khởi tạo project: `npm create vite@latest my-app -- --template react`. **React 19** giới thiệu React Compiler tự động optimize, không cần `useMemo`/`useCallback` thủ công.

---

## 1. Khái niệm

### React là gì?

React là **JavaScript library** để build user interfaces, ra đời tại Facebook (2013), open source từ đó. Hiện được dùng bởi Netflix, Airbnb, Microsoft, PayPal.

**Library vs Framework:**
- **Library**: React chỉ lo phần View — bạn tự chọn routing, state management, data fetching
- **Framework**: Angular/Next.js đã tích hợp sẵn router, DI, HTTP client, ...

### Virtual DOM

React không update real DOM trực tiếp mà dùng **Virtual DOM** — cây object JS mô tả UI:

```mermaid
flowchart LR
    A["State/Props thay đổi"] --> B["React tạo Virtual DOM mới"]
    B --> C["Diff với Virtual DOM cũ<br/>(Reconciliation)"]
    C --> D["Chỉ patch phần thay đổi<br/>vào Real DOM"]
```

Lợi ích: tránh re-render không cần thiết, DOM operations tốn kém nhất được tối thiểu hóa.

```
★ Insight ─────────────────────────────────────
• Virtual DOM giải đúng bài toán bạn đã thấy ở DOM thủ công: thao tác DOM thật =
  reflow tốn kém. React "gom" thay đổi vào cây JS, diff, rồi patch MỘT lần phần
  khác biệt — cùng tinh thần DocumentFragment ([[13-DOM-Performance]]) nhưng tự
  động hoá. Đây là lý do React "declarative": bạn mô tả UI THEO state, React lo phần "làm sao update DOM".
• "Library, không phải framework" là câu trả lời phỏng vấn quan trọng: React chỉ
  lo View → bạn TỰ LẮP routing (React Router), state toàn cục (Redux/Zustand),
  data fetching, SSR (Next.js). Một luật xuyên suốt: data chảy XUỐNG qua props,
  sự kiện chảy LÊN qua callback props ("one-way data flow").
─────────────────────────────────────────────────
```

### React Compiler (React 19)

React 19 ra mắt **React Compiler** (code name "React Forget") — tự động tối ưu hóa code tại compile time, loại bỏ nhu cầu dùng `useMemo`/`useCallback` thủ công:

```mermaid
flowchart LR
    A["JSX component"] --> B["React Compiler"] --> C["Optimized JS cho browser"]
```
> Giống compiled language — compiler hiểu khi nào cần re-render.

---

## 2. Cú pháp / API

### 2.1 Khởi tạo project với Vite

```bash
# Tạo project React (Vite template)
npm create vite@latest my-app -- --template react

# Di chuyển vào project và chạy
cd my-app
npm install
npm run dev   # → http://localhost:5173
```

**Cấu trúc project:**

```text
my-app/
├── index.html          # entry HTML — có <div id="root">
├── vite.config.js
├── package.json
└── src/
    ├── main.jsx        # entry point — ReactDOM.createRoot + render
    ├── App.jsx         # root component
    └── assets/
```

### 2.2 Entry point

```jsx
// src/main.jsx
import { createRoot } from 'react-dom/client';
import App from './App.jsx';
import './index.css';

// Tìm <div id="root"> trong index.html, inject component App vào đó
createRoot(document.getElementById('root')).render(<App />);
```

### 2.3 Component đầu tiên

```jsx
// src/App.jsx
function App() {
  return (
    <div>
      <h1>Hello React!</h1>
    </div>
  );
}

export default App;
```

### 2.4 React createElement (under the hood)

JSX là cú pháp sugar — compiler chuyển về `React.createElement`:

```jsx
// JSX — developer viết:
const element = <h1 className="title">Hello</h1>;

// Sau khi biên dịch — JavaScript thực chạy:
const element = React.createElement('h1', { className: 'title' }, 'Hello');
// → { type: 'h1', props: { className: 'title', children: 'Hello' } }
```

### 2.5 React Developer Tools

Cài extension **React Developer Tools** (Chrome/Firefox):
- Tab **Components**: xem component tree, props, state hiện tại
- Tab **Profiler**: đo performance, phát hiện slow renders

---

## 3. Ví dụ minh họa

### Ví dụ 1: App đơn giản với component lồng nhau

```jsx
// Header component
function Header({ title }) {
  return (
    <header>
      <h1>{title}</h1>
    </header>
  );
}

// Main component
function Main({ items }) {
  return (
    <main>
      <ul>
        {items.map((item, i) => (
          <li key={i}>{item}</li>
        ))}
      </ul>
    </main>
  );
}

// Root App
function App() {
  const menuItems = ['Pho', 'Bun bo', 'Com tam'];
  return (
    <>
      <Header title="My Restaurant" />
      <Main items={menuItems} />
    </>
  );
}

export default App;
```

### Ví dụ 2: Flow dữ liệu trong React

```text
App (root)
├── state: { user, theme, ... }          ← state tập trung ở trên cao
├── Header (props: user.name)            ← nhận data qua props
├── Sidebar (props: theme)
└── Main
    └── ProductList (props: products)    ← tiếp tục pass down
        └── ProductCard (props: product)
```

Nguyên tắc: **data flows down** (props), **events flow up** (callback props).

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Pitfall 1: React là library, không phải framework
> React chỉ handle UI layer. Cần routing → thêm React Router. Cần state management phức tạp → Redux/Zustand/Context. Cần SSR/SSG → Next.js. Đừng nhầm React với Next.js khi trả lời phỏng vấn.

> [!warning] Pitfall 2: `import React from 'react'` không còn bắt buộc
> Từ React 17+, không cần import React để dùng JSX nữa (JSX Transform tự động). Chỉ import khi cần dùng `React.memo`, `React.lazy`, v.v.

> [!tip] Virtual DOM ≠ Shadow DOM
> **Virtual DOM** là khái niệm của React — cây object JS. **Shadow DOM** là Web standard để encapsulate DOM trong Web Components. Hai khái niệm hoàn toàn khác nhau.

---

## 5. Câu hỏi phỏng vấn thường gặp

**Q1: React là gì? Tại sao dùng React thay vì Vanilla JS?**

> React là **JavaScript library** để build UI theo kiến trúc **component**. So với Vanilla JS: (1) **Component reuse** — viết 1 lần, dùng nhiều nơi. (2) **Declarative** — mô tả UI "nên trông như thế nào", React lo update DOM. (3) **Virtual DOM** — diff và chỉ patch những gì thay đổi, hiệu quả hơn DOM manipulation thủ công. (4) **Ecosystem** lớn: React Router, Redux, Next.js, React Native. Nhược điểm: learning curve với JSX/hooks, cần thêm thư viện cho routing/state.

**Q2: Virtual DOM là gì? Reconciliation là gì?**

> **Virtual DOM** là biểu diễn lightweight của Real DOM dưới dạng JS object. Khi state/props thay đổi: (1) React tạo **cây Virtual DOM mới**. (2) **Diffing**: so sánh với cây cũ, tìm ra sự khác biệt (O(n) nhờ heuristics). (3) **Reconciliation**: chỉ update những DOM nodes thực sự thay đổi. Lợi ích: tránh layout thrashing, batch multiple updates, hiệu suất tốt hơn.

**Q3: React library vs React framework (Next.js) khác nhau thế nào?**

> **React (library)**: chỉ handle UI, bạn tự chọn routing/state/SSR, dùng Vite để bundle. **Next.js (framework)**: bao gồm React + file-based routing, SSR/SSG, API routes, Image optimization, caching. Khi nào dùng Next.js: cần SEO, performance tốt nhất, full-stack app. Khi nào dùng Vite+React: SPA đơn giản, CRA replacement, học cơ bản.

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Khởi tạo project Vite+React, xóa code mặc định, tạo component `Header` nhận prop `title` và hiển thị trong `<h1>`. Render từ `App`.

- [ ] **Bài 2:** Cài React Developer Tools. Mở Components tab, inspect cây component của một trang React bất kỳ (ví dụ react.dev). Mô tả những gì bạn thấy.

---

## 7. Liên kết

- [[02-Component-va-Props]] — Component anatomy và cách truyền props
- [[03-State-voi-useState]] — useState hook, quản lý state
- [[../03-Advanced-JavaScript/11-ES6-Class|ES6 Class]] — class syntax (legacy React class components)
- [[../04-Async-JavaScript/01-Async-Overview|Async Overview]] — cần cho data fetching trong React
