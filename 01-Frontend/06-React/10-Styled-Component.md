---
title: "React: Styled-Components"
section: 06-React
tags: [react, styled-components, css-in-js, tagged-template, ThemeProvider, fresher, frontend]
related:
  - "[[09-CSS-in-JS-Inline-Scoped-Global]]"
  - "[[02-Component-va-Props]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [styled-components.com/docs]
---

# React: Styled-Components

> [!summary] TL;DR
> `styled-components` là CSS-in-JS library dùng **tagged template literals** để viết CSS trực tiếp trong JS. Syntax: `` const Button = styled.button`css here` ``. Hỗ trợ **props-based styling** `` ${props => props.primary ? '...' : '...'} ``, **nesting** và **pseudo-classes** như CSS bình thường. **ThemeProvider** inject theme object vào toàn bộ styled components. **`createGlobalStyle`** cho global/reset CSS.

---

## 1. Khái niệm

### Tagged Template Literals

Styled-components dùng **tagged template literals** — cú pháp JS cho phép hàm xử lý template string:

```js
// Tagged template literal — styled.button là function
const Button = styled.button`
  background: blue;
  color: white;
`;
// Tương đương:
// const Button = styled.button(["background: blue;\n  color: white;\n"]);
```

Kết quả là một **React component** với class name được auto-generate và scoped.

```
★ Insight ─────────────────────────────────────
• styled.button`...` chính là TAGGED TEMPLATE bạn học ở
  [[../03-Advanced-JavaScript/06-ES6-Template-Literals]]: hàm `styled.button` nhận
  phần chữ CSS + các hàm `${props => ...}` rồi sinh ra component + class hash. Hiểu
  "nó là tagged template" thì cú pháp lạ này hết bí ẩn — và props-based style chỉ
  là interpolation hàm chạy lại mỗi khi props đổi.
• Hai luật sống còn: (1) ĐỪNG định nghĩa styled component TRONG render — mỗi render
  tạo component mới → mất DOM/state + chậm; đặt ở cấp module. (2) ThemeProvider
  THỰC CHẤT là Context ([[11-Context-API]]): nó "phát" theme object xuống, mọi
  styled component đọc qua props.theme. Vì vậy đổi theme = đổi value Context = cả cây style cập nhật.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp / API

### 2.1 Cài đặt và cú pháp cơ bản

```bash
npm install styled-components
```

```jsx
import styled from 'styled-components';

// Tạo styled component — cú pháp: styled.htmlTag`css`
const Title = styled.h1`
  font-size: 24px;
  font-weight: bold;
  color: #1a1a2e;
  margin-bottom: 16px;
`;

const Container = styled.div`
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 16px;
`;

const Card = styled.div`
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  padding: 24px;

  /* Pseudo-classes — hoạt động như CSS thường */
  &:hover {
    box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
    transform: translateY(-2px);
    transition: all 0.2s ease;
  }
`;

function App() {
  return (
    <Container>
      <Card>
        <Title>Hello Styled-Components!</Title>
        <p>CSS-in-JS made easy.</p>
      </Card>
    </Container>
  );
}
```

### 2.2 Props-based Dynamic Styles

```jsx
// Props truyền vào styled component để điều chỉnh style
const Button = styled.button`
  padding: 8px 16px;
  border-radius: 4px;
  border: 2px solid transparent;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.2s;

  /* Conditional styles dựa trên props */
  background-color: ${props => props.primary ? '#0066cc' : 'transparent'};
  color: ${props => props.primary ? 'white' : '#0066cc'};
  border-color: ${props => props.primary ? 'transparent' : '#0066cc'};

  /* Size variants */
  font-size: ${props => {
    if (props.size === 'sm') return '12px';
    if (props.size === 'lg') return '18px';
    return '14px';
  }};

  padding: ${({ size }) => size === 'sm' ? '4px 12px' : size === 'lg' ? '12px 24px' : '8px 16px'};

  /* Disabled state */
  opacity: ${props => props.disabled ? 0.5 : 1};
  cursor: ${props => props.disabled ? 'not-allowed' : 'pointer'};

  &:hover:not(:disabled) {
    filter: brightness(0.9);
  }
`;

function App() {
  return (
    <div>
      <Button primary>Primary Button</Button>
      <Button>Outline Button</Button>
      <Button primary size="sm">Small Primary</Button>
      <Button size="lg">Large Outline</Button>
      <Button primary disabled>Disabled</Button>
    </div>
  );
}
```

### 2.3 Extending Styled Components

```jsx
// Mở rộng từ styled component đã có
const BaseButton = styled.button`
  padding: 8px 16px;
  border-radius: 4px;
  border: none;
  cursor: pointer;
  font-weight: 600;
`;

// styled(ExistingComponent)`additional css`
const PrimaryButton = styled(BaseButton)`
  background: #0066cc;
  color: white;
  &:hover { background: #0052a3; }
`;

const DangerButton = styled(BaseButton)`
  background: #dc2626;
  color: white;
  &:hover { background: #b91c1c; }
`;

// Extend bất kỳ component nào có className prop
const Link = ({ className, children, href }) => (
  <a href={href} className={className}>{children}</a>
);

const StyledLink = styled(Link)`
  color: #0066cc;
  text-decoration: none;
  &:hover { text-decoration: underline; }
`;
```

### 2.4 ThemeProvider — Global Theme

```jsx
import { ThemeProvider } from 'styled-components';

// Define theme object
const lightTheme = {
  colors: {
    primary:    '#0066cc',
    background: '#ffffff',
    surface:    '#f8f9fa',
    text:       '#1a1a2e',
    textSecondary: '#6b7280',
    error:      '#dc2626',
    success:    '#16a34a',
  },
  spacing: {
    xs: '4px',
    sm: '8px',
    md: '16px',
    lg: '24px',
    xl: '32px',
  },
  borderRadius: {
    sm: '4px',
    md: '8px',
    lg: '16px',
  },
};

const darkTheme = {
  ...lightTheme,
  colors: {
    ...lightTheme.colors,
    primary:    '#60a5fa',
    background: '#0f172a',
    surface:    '#1e293b',
    text:       '#f1f5f9',
    textSecondary: '#94a3b8',
  },
};

// Styled components access theme qua props.theme
const ThemedCard = styled.div`
  background:    ${props => props.theme.colors.surface};
  color:         ${props => props.theme.colors.text};
  border-radius: ${props => props.theme.borderRadius.md};
  padding:       ${props => props.theme.spacing.lg};
  box-shadow:    0 2px 8px rgba(0, 0, 0, 0.1);
`;

// Dùng ThemeProvider để wrap app
function App() {
  const [isDark, setIsDark] = useState(false);

  return (
    <ThemeProvider theme={isDark ? darkTheme : lightTheme}>
      <button onClick={() => setIsDark(prev => !prev)}>
        Toggle {isDark ? 'Light' : 'Dark'} Mode
      </button>
      <ThemedCard>
        <h2>Themed Content</h2>
        <p>Adapts to theme automatically.</p>
      </ThemedCard>
    </ThemeProvider>
  );
}
```

### 2.5 createGlobalStyle

```jsx
import { createGlobalStyle } from 'styled-components';

// Global styles — không tạo DOM element, chỉ inject CSS
const GlobalStyle = createGlobalStyle`
  *, *::before, *::after {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  body {
    font-family: system-ui, -apple-system, sans-serif;
    background-color: ${props => props.theme.colors.background};
    color: ${props => props.theme.colors.text};
    line-height: 1.5;
    transition: background-color 0.2s, color 0.2s;
  }

  a {
    color: ${props => props.theme.colors.primary};
    text-decoration: none;
  }
`;

// Render cùng với ThemeProvider
function App() {
  return (
    <ThemeProvider theme={lightTheme}>
      <GlobalStyle />       {/* inject global styles */}
      <MainContent />
    </ThemeProvider>
  );
}
```

### 2.6 useTheme Hook

```jsx
import { useTheme } from 'styled-components';

// Dùng theme trong logic JS (không phải CSS)
function ThemedComponent() {
  const theme = useTheme();

  const chartColors = [
    theme.colors.primary,
    theme.colors.success,
    theme.colors.error,
  ];

  return (
    <div>
      {chartColors.map((color, i) => (
        <div key={i} style={{ width: '20px', height: '20px', background: color }} />
      ))}
    </div>
  );
}
```

---

## 3. Ví dụ minh họa

### Ví dụ: Design system components

```jsx
import styled, { css } from 'styled-components';

// Reusable mixins với css helper
const flexCenter = css`
  display: flex;
  align-items: center;
  justify-content: center;
`;

const ellipsis = css`
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
`;

// Input component
const Input = styled.input`
  width: 100%;
  padding: 8px 12px;
  border: 1px solid ${({ theme, error }) => error ? theme.colors.error : '#d1d5db'};
  border-radius: ${({ theme }) => theme.borderRadius.sm};
  font-size: 14px;
  outline: none;
  transition: border-color 0.15s;

  &:focus {
    border-color: ${({ theme, error }) => error ? theme.colors.error : theme.colors.primary};
    box-shadow: 0 0 0 3px ${({ theme, error }) =>
      error ? 'rgba(220, 38, 38, 0.15)' : 'rgba(0, 102, 204, 0.15)'};
  }
`;

const Label = styled.label`
  display: block;
  font-size: 14px;
  font-weight: 500;
  color: ${({ theme }) => theme.colors.text};
  margin-bottom: 4px;
`;

const ErrorText = styled.p`
  font-size: 12px;
  color: ${({ theme }) => theme.colors.error};
  margin-top: 4px;
`;

function FormField({ label, error, ...inputProps }) {
  return (
    <div>
      {label && <Label>{label}</Label>}
      <Input error={!!error} {...inputProps} />
      {error && <ErrorText>{error}</ErrorText>}
    </div>
  );
}
```

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Pitfall 1: Không define styled components trong render function
> ```jsx
> // SAI — tạo styled component bên trong render → recreate mỗi render → performance issue
> function MyComponent() {
>   const Box = styled.div`color: red;`; // Đừng làm thế này!
>   return <Box>...</Box>;
> }
>
> // ĐÚNG — define ở module level (ngoài component)
> const Box = styled.div`color: red;`;
> function MyComponent() { return <Box>...</Box>; }
> ```

> [!warning] Pitfall 2: Bundle size — styled-components không tree-shake styles
> styled-components inject toàn bộ styles vào `<style>` tag. Không cần lo CSS unused vì chỉ styles của components được render mới được inject. Nhưng bundle JS to hơn so với CSS Modules. Đây là trade-off cần cân nhắc.

> [!tip] Khi nào chọn styled-components vs CSS Modules?
> **styled-components**: component library, design system cần strong theming, complex dynamic styles. **CSS Modules**: app thông thường, muốn gần CSS hơn, không muốn add runtime dependency. Cả hai đều tốt — chọn theo preference của team.

---

## 5. Câu hỏi phỏng vấn thường gặp

**Q1: styled-components là gì? Hoạt động như thế nào?**

> `styled-components` là CSS-in-JS library dùng **tagged template literals** để viết CSS trong JavaScript. `styled.button\`css here\`` tạo ra React component với class name được auto-generate và CSS được inject vào `<style>` tag khi component render. **Lợi ích**: scoped styling, props-based conditional styles, full CSS support (hover, media queries), TypeScript support tốt. **Hoạt động**: component render → styled-components generate unique hash → inject CSS với hash class → component nhận class name.

**Q2: ThemeProvider trong styled-components hoạt động thế nào?**

> `ThemeProvider` là Context Provider inject **theme object** vào tất cả styled components trong cây. Styled components access theme qua `props.theme`: `` background: ${props => props.theme.colors.primary} ``. Có thể nest `ThemeProvider` để override theme cho sub-tree. `useTheme()` hook cho phép access theme trong logic JS (không phải template literal).

**Q3: CSS Modules vs styled-components — chọn cái nào?**

> **CSS Modules**: zero runtime overhead, gần với CSS hơn, dễ hiểu với team CSS-heavy, bundle nhỏ hơn. **styled-components**: colocation tốt hơn (CSS và logic trong 1 file), props-based dynamic styles elegant hơn, TypeScript props typing, ThemeProvider cho global theming. Rule of thumb: CSS Modules cho apps, styled-components cho component libraries/design systems. Cả hai đều tốt — consistency quan trọng hơn là chọn "đúng".

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Tạo `Chip` component với styled-components — variant `filled` và `outlined`, màu sắc từ props. Khi click → toggle selected state, style đổi tương ứng.

- [ ] **Bài 2:** Implement dark/light theme toggle với `ThemeProvider`. Define 2 theme objects. App root dùng `ThemeProvider` với theme từ state. Tạo `GlobalStyle` reset. Tạo `Card`, `Button`, `Input` đều dùng `props.theme`.

---

## 7. Liên kết

- [[09-CSS-in-JS-Inline-Scoped-Global]] — Inline styles, CSS Modules, Global CSS
- [[03-State-voi-useState]] — Theme state toggle
- [[11-Context-API]] — ThemeProvider về bản chất là Context Provider
