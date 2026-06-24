---
title: "React: CSS-in-JS (Inline/Scoped/Global)"
section: 06-React
tags: [react, css-in-js, inline-style, css-modules, global-css, classnames, fresher, frontend]
related:
  - "[[10-Styled-Component]]"
  - "[[02-Component-va-Props]]"
difficulty: ⭐⭐⭐
estimated_time: 35m
source: [tự bổ sung — react.dev, MDN]
---

# React: CSS-in-JS (Inline/Scoped/Global)

> [!summary] TL;DR
> React có 3 cách tô style chính: **Inline styles** (viết style bằng *object JS*, tên thuộc tính camelCase; tiện cho giá trị động nhưng *không* viết được `:hover`/pseudo-class/`@media`), **CSS Modules** (file đặt tên `.module.css` — bundler **tự đổi tên class thành duy nhất** nên không đụng nhau giữa các component; tốt nhất để style component), **Global CSS** (import file `.css` dùng chung — hợp cho reset/typography/biến CSS). Ghép nhiều class động bằng thư viện `clsx` hoặc template literal. Inline style viết 2 cặp ngoặc `style={{ color: 'red' }}` — ngoặc ngoài là "đây là biểu thức JS", ngoặc trong là object.

> [!tip] 🎯 Hiểu trong 30 giây
> Có 3 nơi để "tô màu" cho component, chọn theo **phạm vi** và **sức mạnh**:
> - **Inline style** = dán style thẳng lên 1 thẻ bằng object JS (`style={{ color:'red' }}`). Cực cục bộ, hợp giá trị *động* (màu lấy từ API). Nhược: **không** dùng được `:hover`, `@media`, animation.
> - **CSS Modules** (file `.module.css`) = viết CSS bình thường nhưng tên class được **tự "dán nhãn" thành duy nhất** → hai component đặt tên `.button` giống nhau cũng *không đụng nhau*. Đây là lựa chọn mặc định cho component, giữ đủ sức mạnh CSS.
> - **Global CSS** = file dùng chung toàn app, hợp cho reset, font, biến màu — nhưng dễ trùng tên gây đè nhau.
>
> **Bẫy nhỏ:** `style={{...}}` tạo *object mới mỗi lần render* → nếu truyền xuống con đã `React.memo` thì phá memo. Khắc phục: khai báo object style *ngoài* component.

---

## 1. Khái niệm

### 3 Cách Style trong React

```text
1. Inline Styles      → JS object, không cần file .css
2. CSS Modules        → Scoped CSS, không clash, Vite support sẵn
3. Global CSS         → import file .css, cần đảm bảo không conflict
```

| Phương pháp | Scope | Hover/Media | Dynamic | Dùng khi |
|---|---|---|---|---|
| Inline | Cực local (1 element) | ❌ | Dễ | Style dynamic 1 element |
| CSS Modules | Component scope | ✅ | Với className | Component styling |
| Global CSS | Global | ✅ | Với className | Reset, typography, variables |
| Styled-components | Component scope | ✅ | Với props | Xem note tiếp theo |

```
★ Insight ─────────────────────────────────────
• Trục lựa chọn là PHẠM VI + KHẢ NĂNG: inline = cực cục bộ nhưng MẤT hover/media/
  keyframes (vì nó chỉ là thuộc tính style của 1 element); CSS Modules = tự đổi
  tên class thành duy nhất → diệt xung đột global mà GIỮ trọn sức mạnh CSS; global
  CSS = mạnh nhưng dễ đụng tên. Mặc định CSS Modules cho component, inline chỉ cho
  giá trị THỰC SỰ động (màu từ API, width tính toán).
• Bẫy nhỏ nhưng hay gặp: `style={{...}}` tạo OBJECT MỚI mỗi render → nếu truyền
  xuống component bọc React.memo sẽ phá memo (khác reference). Khắc phục: khai báo
  style object NGOÀI component. Cùng một bài học "tham chiếu mới mỗi render" với
  deps của useEffect và value của Context.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp / API

### 2.1 Inline Styles

```jsx
// style nhận JS OBJECT — không phải CSS string
// Property names là camelCase (không phải kebab-case)
function InlineExamples() {
  const primaryColor = '#0066cc';
  const isError = true;

  return (
    <div>
      {/* Hard-coded inline style — 2 cặp {} */}
      <p style={{ color: 'red', fontSize: 16, marginTop: '8px' }}>
        Red text
      </p>

      {/* Dynamic — từ variables */}
      <button style={{ backgroundColor: primaryColor, color: 'white' }}>
        Primary Button
      </button>

      {/* Conditional style */}
      <span style={{ color: isError ? '#dc2626' : '#16a34a' }}>
        {isError ? 'Error' : 'Success'}
      </span>

      {/* Style object tách riêng — clean hơn khi phức tạp */}
      <Card />
    </div>
  );
}

// Style object tách ra ngoài component (tránh tạo mới mỗi render)
const cardStyle = {
  padding: '16px',
  borderRadius: '8px',
  boxShadow: '0 2px 8px rgba(0,0,0,0.1)',
  backgroundColor: '#fff',
};

function Card({ children }) {
  return <div style={cardStyle}>{children}</div>;
}
```

**Hạn chế của inline styles:**
- Không viết được `&:hover`, `&:focus`, `@media`, `@keyframes`
- Không có CSS variables
- Không cascade/inherit

### 2.2 CSS Modules

```css
/* Button.module.css */
.button {
  padding: 8px 16px;
  border-radius: 4px;
  border: none;
  cursor: pointer;
  font-size: 14px;
  transition: opacity 0.2s;
}

.button:hover {
  opacity: 0.85;
}

.primary {
  background-color: #0066cc;
  color: white;
}

.secondary {
  background-color: #f0f0f0;
  color: #333;
}

.danger {
  background-color: #dc2626;
  color: white;
}

.disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

```jsx
// Button.jsx
import styles from './Button.module.css';

function Button({ children, variant = 'primary', disabled = false, onClick }) {
  // styles.button → "Button_button__abc123" (auto-scoped)
  // Combine classes bằng template literal hoặc array.filter.join
  const className = [
    styles.button,
    styles[variant],            // styles.primary / styles.secondary / styles.danger
    disabled && styles.disabled,
  ].filter(Boolean).join(' ');

  return (
    <button className={className} disabled={disabled} onClick={onClick}>
      {children}
    </button>
  );
}

// Sử dụng — class names tự động scoped, không conflict với Button.css của project khác
function App() {
  return (
    <div>
      <Button variant="primary">Save</Button>
      <Button variant="secondary">Cancel</Button>
      <Button variant="danger" disabled>Delete</Button>
    </div>
  );
}
```

### 2.3 Global CSS

```css
/* src/index.css — global styles */

/* CSS Reset / Normalize */
*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

/* CSS Custom Properties (variables) */
:root {
  --color-primary: #0066cc;
  --color-error:   #dc2626;
  --color-success: #16a34a;
  --border-radius: 4px;
  --font-family: system-ui, -apple-system, sans-serif;
}

body {
  font-family: var(--font-family);
  line-height: 1.5;
  color: #333;
}

/* Utility classes */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip: rect(0,0,0,0);
}
```

```jsx
// src/main.jsx — import global CSS ở entry point
import './index.css';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';

createRoot(document.getElementById('root')).render(<App />);
```

### 2.4 Dynamic className Patterns

```jsx
// 1. Template literal
function Alert({ type, message }) {
  return (
    <div className={`alert alert-${type}`}>
      {message}
    </div>
  );
}

// 2. Array.filter(Boolean).join — cho nhiều conditional classes
function Input({ hasError, isFocused, isDisabled }) {
  const classes = [
    'input',
    hasError   && 'input--error',
    isFocused  && 'input--focused',
    isDisabled && 'input--disabled',
  ].filter(Boolean).join(' ');

  return <input className={classes} disabled={isDisabled} />;
}

// 3. Thư viện clsx (nhẹ hơn classnames)
// npm install clsx
import clsx from 'clsx';

function Tag({ label, active, size = 'md' }) {
  return (
    <span
      className={clsx(
        'tag',
        `tag--${size}`,      // selalu ada
        { 'tag--active': active } // conditional
      )}
    >
      {label}
    </span>
  );
}
```

### 2.5 Combining CSS Modules + CSS Variables

```jsx
// Tận dụng cả hai: CSS Modules cho scoping + CSS variables cho theming

/* theme.css */
/* :root { --primary: #0066cc; } */
/* [data-theme="dark"] { --primary: #60a5fa; } */

/* Card.module.css */
/* .card { border: 1px solid var(--primary); } */

function Card({ children }) {
  return <div className={styles.card}>{children}</div>;
}

// Toggle theme ở App level
function App() {
  const [theme, setTheme] = useState('light');
  return (
    <div data-theme={theme}>
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
      <Card>Content</Card>
    </div>
  );
}
```

---

## 3. Ví dụ minh họa

### Ví dụ 1: Responsive grid với CSS Modules

```css
/* Grid.module.css */
.grid {
  display: grid;
  gap: 16px;
}

.cols-1 { grid-template-columns: 1fr; }
.cols-2 { grid-template-columns: repeat(2, 1fr); }
.cols-3 { grid-template-columns: repeat(3, 1fr); }
.cols-4 { grid-template-columns: repeat(4, 1fr); }

@media (max-width: 768px) {
  .cols-2, .cols-3, .cols-4 { grid-template-columns: 1fr; }
}
```

```jsx
import styles from './Grid.module.css';
import clsx from 'clsx';

function Grid({ children, cols = 3 }) {
  return (
    <div className={clsx(styles.grid, styles[`cols-${cols}`])}>
      {children}
    </div>
  );
}
```

### Ví dụ 2: Status badge với inline + CSS Module combo

```jsx
/* Badge.module.css */
/* .badge { padding: 2px 8px; border-radius: 999px; font-size: 12px; font-weight: 600; } */

import styles from './Badge.module.css';

const STATUS_COLORS = {
  active:   { bg: '#dcfce7', color: '#166534' },
  inactive: { bg: '#f3f4f6', color: '#4b5563' },
  pending:  { bg: '#fef9c3', color: '#854d0e' },
  error:    { bg: '#fee2e2', color: '#991b1b' },
};

function StatusBadge({ status, label }) {
  const colors = STATUS_COLORS[status] ?? STATUS_COLORS.inactive;

  return (
    <span
      className={styles.badge}
      style={{ backgroundColor: colors.bg, color: colors.color }}
    >
      {label ?? status}
    </span>
  );
}
```

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Pitfall 1: Inline style object tạo mới mỗi render
> `<div style={{ color: 'red' }}>` — literal object `{ color: 'red' }` tạo **new reference mỗi render**. Đây thường không phải vấn đề lớn, nhưng nếu truyền qua props xuống component được `React.memo`, nó sẽ bypass memo. Fix: khai báo constant ngoài component `const style = { color: 'red' }`.

> [!warning] Pitfall 2: Global CSS class name conflicts
> Import nhiều CSS files với cùng tên class → styles override nhau tùy thứ tự import. **CSS Modules giải quyết vấn đề này** bằng cách auto-scope. Nếu dùng global CSS, dùng naming convention: BEM (`block__element--modifier`) hoặc namespace (`Button-primary`).

> [!tip] Inline style cho dynamic values, CSS Modules cho static structure
> Best practice: CSS Modules xử lý base styles, layout, responsive. Inline style chỉ cho values thực sự dynamic (màu từ API, width từ calculation). Tránh dùng inline style cho styles có thể viết trong CSS.

---

## 5. Câu hỏi phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "CSS Modules là gì, hơn global CSS ở đâu?"
> *"CSS Modules là cách viết CSS mà tên class được bundler như Vite tự đổi thành tên duy nhất khi build, ví dụ class button trở thành Button_button_abc123. Mình import file module css vào component rồi dùng `styles.button`. Lợi ích lớn nhất là tránh xung đột tên class: với global CSS, hai component cùng đặt tên button sẽ đè style của nhau tùy thứ tự import, rất khó lần; còn CSS Modules tự cô lập theo component nên không bao giờ đụng, mà vẫn giữ trọn sức mạnh CSS như hover, media query, animation. Nên em mặc định dùng CSS Modules cho component, để global CSS cho reset, font và biến màu, còn inline style chỉ cho giá trị thực sự động."*

> [!note] 🧠 Mẹo nhớ
> **Inline = động nhưng yếu (không hover/media). CSS Modules = tự cô lập tên class, mặc định cho component. Global = dùng chung, dễ đụng tên.** `style={{}}` tạo object mới mỗi render → khai báo ngoài component nếu cần.

**Q1: CSS Modules là gì? Tại sao nên dùng thay vì global CSS?**

> **CSS Modules** là CSS file mà class names được **tự động scoped** bởi bundler (Vite/Webpack) — `.button` trong `Button.module.css` trở thành `Button_button__abc123` trong output. Import vào component: `import styles from './Button.module.css'`, dùng: `className={styles.button}`. **Lợi ích**: không bao giờ có class name conflicts dù đặt tên giống nhau ở các components khác nhau. **Giữ được** tất cả tính năng CSS: hover, media queries, animations.

**Q2: Inline style trong React khác HTML như thế nào?**

> HTML: `style="color: red; font-size: 16px"` (CSS string). React: `style={{ color: 'red', fontSize: 16 }}` (JS object). Khác biệt: (1) **Camelcase property names**: `fontSize` không phải `font-size`, `backgroundColor` không phải `background-color`. (2) **JS values**: số không cần đơn vị `px` cho pixel values (fontSize: 16 = 16px), strings cần đơn vị. (3) **Dynamic**: có thể dùng variables, expressions. Hạn chế: không support `:hover`, `@media`, `@keyframes`.

**Q3: Khi nào dùng inline style, CSS Modules, hay Styled-components?**

> **Inline style**: giá trị hoàn toàn dynamic (màu từ data, width từ calculation, transform animated). **CSS Modules**: component styling với pseudo-classes, media queries, animations — recommended cho hầu hết trường hợp. **Styled-components**: thích CSS-in-JS workflow, theming phức tạp, component library. **Global CSS**: CSS reset, base typography, CSS variables, utility classes.

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Tạo `ProgressBar` component với CSS Modules. Props: `value` (0-100), `color` (string, hex). Inline style cho width (`${value}%`) và backgroundColor. CSS Module cho base styles (height, border-radius, transition).

- [ ] **Bài 2:** Tạo theme toggle (light/dark). Global CSS với `:root` và `[data-theme="dark"]` cho CSS variables. App component toggle `data-theme` attribute trên `document.body`. Các components dùng `var(--bg)`, `var(--text)`.

---

## 7. Liên kết

- [[10-Styled-Component]] — CSS-in-JS library với tagged template literals
- [[02-Component-va-Props]] — Component props dùng trong dynamic styling
- [[03-State-voi-useState]] — State điều khiển dynamic className
