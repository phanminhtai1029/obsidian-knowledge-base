---
title: "React: Context API"
section: 06-React
tags: [react, context, createContext, Provider, useContext, prop-drilling, fresher, frontend]
related:
  - "[[12-Context-voi-useState]]"
  - "[[13-Context-voi-useReducer]]"
  - "[[03-State-voi-useState]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [react.dev/learn/passing-data-deeply-with-context]
---

# React: Context API

> [!summary] TL;DR
> **Context** giải quyết **prop drilling** — truyền data qua nhiều cấp component trung gian không cần dùng data đó. 3 bước: (1) `createContext(defaultValue)` tạo context, (2) `<Context.Provider value={...}>` wrap subtree cần access, (3) `useContext(Context)` đọc giá trị trong component con. **Context không phải state management** — nó chỉ là channel phân phối giá trị. State (useState/useReducer) vẫn cần nằm ở đâu đó.

> [!tip] 🎯 Hiểu trong 30 giây
> **Prop drilling** = phải "chuyền tay" một món đồ qua 5 người chỉ để đưa cho người cuối, dù 4 người ở giữa chẳng dùng tới. Trong React là truyền 1 prop xuống qua nhiều tầng component trung gian không cần nó — mệt và dễ sai.
>
> **Context = "loa phát thanh tòa nhà".** Đặt giá trị vào `Provider` ở trên cao một lần, rồi **bất kỳ component con nào ở sâu bên dưới** cũng "nghe" được bằng `useContext` — khỏi chuyền tay qua từng tầng. 3 bước: `createContext` (lắp loa) → `<Provider value={...}>` (phát) → `useContext` (nghe).
>
> **2 điều hay bị hỏi:**
> 1. **Context KHÔNG phải Redux/state management.** Nó chỉ là *đường ống phân phối*; state thật vẫn nằm ở `useState`/`useReducer` đâu đó. Đừng nói "Context là quản lý state".
> 2. **Khi nào KHÔNG nên dùng Context:** với dữ liệu *đổi liên tục* (vd giá trị gõ từng phím, dữ liệu real-time). Vì `value` đổi là **mọi consumer re-render** → chậm. Context hợp cho thứ *ít đổi*: theme, ngôn ngữ, user đăng nhập.

---

## 1. Khái niệm

### Vấn đề: Prop Drilling

```text
App (state: theme)
└── Layout (props: theme — chỉ pass down, không dùng)
    └── Sidebar (props: theme — chỉ pass down, không dùng)
        └── MenuItem (props: theme — chỉ pass down, không dùng)
            └── MenuItemIcon (props: theme — DÙNG ở đây!)
```

5 cấp chỉ để pass `theme` xuống từ `App` đến `MenuItemIcon` — mọi component ở giữa đều phải nhận và forward prop mà chúng không dùng.

### Context giải quyết như thế nào?

```text
App
└── ThemeContext.Provider (value: theme)
    └── Layout
        └── Sidebar
            └── MenuItem
                └── MenuItemIcon → useContext(ThemeContext) → trực tiếp lấy theme
```

Không cần truyền qua các component trung gian.

```
★ Insight ─────────────────────────────────────
• Context là ĐƯỜNG ỐNG phân phối, KHÔNG phải kho state: nó không thay Redux/
  Zustand (không selector, không batch tối ưu, không middleware). State thật vẫn
  nằm ở useState/useReducer đâu đó; Context chỉ "phát" giá trị đó xuống sâu mà
  khỏi khoan qua từng tầng props. Câu phỏng vấn hay gặp — đừng nói "Context là
  state management".
• Cái giá: value đổi → MỌI consumer re-render, kể cả đứa chỉ dùng một mẩu. Hai
  cách giảm đau: useMemo gói object value (tránh tạo reference mới mỗi render →
  re-render thừa) và TÁCH context (data đọc tách khỏi hàm dispatch ổn định). Vì
  vậy chỉ để vào Context thứ ÍT đổi (theme/auth/locale), không nhồi state đổi liên tục.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp / API

### 2.1 Tạo Context

```jsx
import { createContext } from 'react';

// createContext(defaultValue) — defaultValue dùng khi component
// không có Provider ancestor nào trong cây
const ThemeContext = createContext('light'); // default = 'light'

// Thường export context để import ở các file khác
export default ThemeContext;

// Convention: đặt tên Context trong file riêng
// src/contexts/ThemeContext.js
```

### 2.2 Provider — Cung cấp giá trị

```jsx
import ThemeContext from './contexts/ThemeContext';

function App() {
  const [theme, setTheme] = useState('light');

  return (
    // Provider wrap các components cần access context
    // value = giá trị được broadcast xuống toàn bộ subtree
    <ThemeContext.Provider value={theme}>
      <Header />
      <Main />
      <Footer />
    </ThemeContext.Provider>
  );
}
// Mọi component bên trong Provider đều có thể dùng ThemeContext
```

### 2.3 useContext — Đọc giá trị

```jsx
import { useContext } from 'react';
import ThemeContext from './contexts/ThemeContext';

// Component bất kỳ trong cây có thể đọc context — không cần props!
function MenuItemIcon({ icon }) {
  const theme = useContext(ThemeContext); // 'light' hoặc 'dark'

  return (
    <span style={{ color: theme === 'dark' ? '#fff' : '#333' }}>
      {icon}
    </span>
  );
}

// Không cần truyền theme qua Layout → Sidebar → MenuItem → MenuItemIcon
```

### 2.4 Ví dụ đầy đủ: Theme Context

```jsx
// contexts/ThemeContext.js
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext({
  theme: 'light',
  toggleTheme: () => {},
});

// Provider component bao gồm cả state
export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => setTheme(prev => prev === 'light' ? 'dark' : 'light');

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Custom hook — đóng gói useContext + validation
export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// App.jsx
import { ThemeProvider } from './contexts/ThemeContext';

function App() {
  return (
    <ThemeProvider>
      <Header />
      <Main />
    </ThemeProvider>
  );
}

// Header.jsx — không cần biết về ThemeProvider ở đâu
import { useTheme } from './contexts/ThemeContext';

function Header() {
  const { theme, toggleTheme } = useTheme();
  return (
    <header className={`header header-${theme}`}>
      <button onClick={toggleTheme}>
        {theme === 'light' ? '🌙 Dark' : '☀️ Light'}
      </button>
    </header>
  );
}
```

### 2.5 Multiple Contexts

```jsx
// Có thể có nhiều Context providers - nest lại với nhau
function App() {
  return (
    <AuthProvider>
      <ThemeProvider>
        <LocaleProvider>
          <Router>
            <AppContent />
          </Router>
        </LocaleProvider>
      </ThemeProvider>
    </AuthProvider>
  );
}

// Component đọc từ nhiều context
function UserMenu() {
  const { user }         = useAuth();    // AuthContext
  const { theme }        = useTheme();   // ThemeContext
  const { locale, t }    = useLocale();  // LocaleContext

  return (
    <menu className={`user-menu user-menu-${theme}`}>
      <li>{t('greeting', { name: user.name })}</li>
    </menu>
  );
}
```

### 2.6 Default Value vs Provider Value

```jsx
// defaultValue trong createContext — dùng khi component NGOÀI Provider tree
const UserContext = createContext(null);

function ProfileCard() {
  const user = useContext(UserContext);
  // Nếu không có UserContext.Provider ancestor: user = null (default)
  // Nếu trong Provider: user = value của Provider

  if (!user) return <p>Not logged in</p>;
  return <p>{user.name}</p>;
}

// Render WITHOUT Provider → dùng default value
// <ProfileCard />  → user = null → "Not logged in"

// Render WITH Provider
// <UserContext.Provider value={{ name: 'Alice' }}>
//   <ProfileCard />   → user = { name: 'Alice' }
// </UserContext.Provider>
```

---

## 3. Ví dụ minh họa

### Ví dụ 1: Auth Context

```jsx
// contexts/AuthContext.jsx
import { createContext, useContext, useState } from 'react';

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = async (credentials) => {
    const res = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify(credentials),
    });
    const data = await res.json();
    setUser(data.user);
    return data;
  };

  const logout = () => {
    setUser(null);
    localStorage.removeItem('token');
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, isLoggedIn: !!user }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth must be inside AuthProvider');
  return ctx;
}

// Sử dụng trong component
function Navbar() {
  const { user, logout, isLoggedIn } = useAuth();
  return (
    <nav>
      {isLoggedIn ? (
        <>
          <span>Hi, {user.name}</span>
          <button onClick={logout}>Logout</button>
        </>
      ) : (
        <a href="/login">Login</a>
      )}
    </nav>
  );
}
```

### Ví dụ 2: Toast/Notification Context

```jsx
const ToastContext = createContext(null);

export function ToastProvider({ children }) {
  const [toasts, setToasts] = useState([]);

  const addToast = (message, type = 'info', duration = 3000) => {
    const id = Date.now();
    setToasts(prev => [...prev, { id, message, type }]);
    setTimeout(() => removeToast(id), duration);
  };

  const removeToast = (id) => {
    setToasts(prev => prev.filter(t => t.id !== id));
  };

  return (
    <ToastContext.Provider value={{ addToast }}>
      {children}
      {/* Toast container render ở đây — không cần truyền toasts xuống */}
      <div className="toast-container">
        {toasts.map(toast => (
          <div key={toast.id} className={`toast toast-${toast.type}`}>
            {toast.message}
            <button onClick={() => removeToast(toast.id)}>✕</button>
          </div>
        ))}
      </div>
    </ToastContext.Provider>
  );
}

export const useToast = () => {
  const ctx = useContext(ToastContext);
  if (!ctx) throw new Error('useToast must be inside ToastProvider');
  return ctx;
};

// Dùng ở bất kỳ đâu trong app
function SaveButton({ onSave }) {
  const { addToast } = useToast();

  const handleSave = async () => {
    try {
      await onSave();
      addToast('Saved successfully!', 'success');
    } catch {
      addToast('Failed to save', 'error');
    }
  };

  return <button onClick={handleSave}>Save</button>;
}
```

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Pitfall 1: Context thay đổi → toàn bộ subtree re-render
> Khi `value` prop của Provider thay đổi, **mọi component** dùng `useContext` với context đó sẽ re-render — kể cả những component chỉ dùng một phần của value. Fix: split context (AuthContext riêng, ThemeContext riêng), dùng `useMemo` để memoize value object.

> [!warning] Pitfall 2: Context không phải Redux/Zustand
> Context chỉ là **delivery channel** — không có selectors, không batch updates tối ưu, không middleware. Dùng cho data hiếm khi thay đổi (theme, locale, auth user). Không nên dùng cho state thay đổi thường xuyên (shopping cart items count, real-time data) — sẽ gây unnecessary re-renders.

> [!tip] Custom hook wrapper là best practice
> Thay vì export `Context` và `useContext(Context)`, export custom hook `useMyContext()` — (1) ẩn implementation detail, (2) throw meaningful error nếu dùng ngoài Provider, (3) dễ thay đổi implementation sau này.

---

## 5. Câu hỏi phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Prop drilling là gì, Context giải quyết ra sao, khi nào KHÔNG nên dùng?"
> *"Prop drilling là khi mình phải truyền một prop xuống qua nhiều tầng component trung gian dù các tầng đó không dùng tới nó, chỉ để đưa xuống component sâu nhất. Việc này khiến code rườm rà và khó bảo trì. Context giải quyết bằng cách cho phép đặt giá trị ở một Provider trên cao, rồi component con ở bất kỳ độ sâu nào cũng đọc trực tiếp qua useContext mà không cần chuyền qua từng tầng. Tuy nhiên em không nên dùng Context cho state thay đổi thường xuyên, vì mỗi khi value của Provider đổi thì tất cả component đang dùng context đó đều re-render, kể cả những cái chỉ dùng một phần nhỏ, gây chậm. Context hợp với dữ liệu ít đổi như theme, ngôn ngữ, thông tin user đăng nhập. Với state đổi liên tục và cần tối ưu re-render thì em dùng thư viện như Redux hoặc Zustand. Và nhớ Context chỉ là kênh phân phối, không phải bản thân state."*

> [!note] 🧠 Mẹo nhớ
> **Prop drilling = chuyền tay qua nhiều tầng. Context = loa phát thanh (Provider phát → useContext nghe).** Context **không phải** state management; **đừng dùng cho data đổi liên tục** (value đổi → mọi consumer re-render).

**Q1: Context API là gì? Giải quyết vấn đề gì?**

> Context API giải quyết **prop drilling** — phải truyền data qua nhiều cấp component trung gian không cần dùng data đó. Context là "broadcast" mechanism: `Provider` đặt data, mọi `useContext(Context)` trong subtree đều nhận được, không cần truyền qua props trung gian. Gồm: `createContext(default)`, `Context.Provider value={...}`, `useContext(Context)`.

**Q2: Context vs Props — khi nào dùng cái nào?**

> **Props**: data flow bình thường giữa parent-child trực tiếp, khi relationship rõ ràng và component tree không quá sâu. **Context**: data cần ở nhiều nơi trong app (theme, auth, locale), tránh prop drilling qua nhiều cấp, global/semi-global data. **Không nên dùng Context cho**: state update thường xuyên (performance), state local của component.

**Q3: Tại sao Context re-render không cần thiết? Cách tối ưu?**

> Khi `Provider value` thay đổi → mọi consumer re-render. Nếu value là object literal `value={{ user, logout }}` → tạo mới mỗi render → consumers luôn re-render. Fix: (1) `useMemo` cho value: `useMemo(() => ({ user, logout }), [user])`. (2) Tách Context: `UserContext` (đọc) + `UserDispatchContext` (write functions, ổn định hơn). (3) Dùng thư viện như Zustand nếu re-render là bottleneck.

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Tạo `LocaleContext` với `locale` ('en'/'vi') và `t(key)` translation function. Hard-code translations object. Wrap app với `LocaleProvider`. Tạo `LanguageSwitcher` component dùng `useLocale()` hook.

- [ ] **Bài 2:** Tạo `CartContext` với `items` array và `totalCount` derived. Tạo `addItem(product)`, `removeItem(id)`, `clearCart()`. Dùng pattern: Provider + useContext + custom hook `useCart()`.

---

## 7. Liên kết

- [[12-Context-voi-useState]] — Pattern cụ thể: Context + useState
- [[13-Context-voi-useReducer]] — Context + useReducer cho complex state
- [[03-State-voi-useState]] — State vẫn cần — Context chỉ là delivery channel
