---
title: "React Cheatsheet"
section: 99-Cheatsheets
tags: [cheatsheet, react, hooks, context, router, reference, fresher, frontend]
related:
  - "[[01-React-Overview]]"
  - "[[03-State-voi-useState]]"
  - "[[08-useEffect-Hook]]"
  - "[[11-Context-API]]"
  - "[[13-Context-voi-useReducer]]"
  - "[[14-React-Router]]"
difficulty: ⭐
estimated_time: 10m
source: [react.dev, reactrouter.com]
---

# React Cheatsheet

> [!summary] TL;DR
> Tra nhanh tất cả hooks, JSX rules, Context pattern và React Router v6 trước phỏng vấn. Điểm hay hỏi nhất: `useEffect` dependency array, `useState` vs `useReducer`, `useMemo` vs `useCallback`, tại sao key prop quan trọng.

---

## 1. JSX Rules

| Rule | Sai | Đúng |
|---|---|---|
| Attribute HTML | `class=` | `className=` |
| Label for | `for=` | `htmlFor=` |
| Inline style | `style="color:red"` | `style={{ color: 'red' }}` |
| Phải có 1 root | `<A/><B/>` | `<><A/><B/></>` |
| Self-closing | `<input>` | `<input />` |
| Boolean shorthand | `disabled={true}` | `disabled` |

```jsx
// Expression — bọc trong {}
function Card({ name, score, isAdmin }) {
  return (
    <div className={`card ${isAdmin ? 'card-admin' : ''}`}>
      <h2>{name}</h2>
      {score > 90 && <span>Top performer</span>}   {/* short-circuit */}
      {isAdmin ? <AdminBadge /> : <UserBadge />}    {/* ternary */}
    </div>
  );
}
```

> [!warning] `{count && <Comp />}` render số `0` khi count=0
> Dùng `{count > 0 && <Comp />}` hoặc ternary `{count ? <Comp /> : null}`.

---

## 2. Hooks — Bảng Tra Nhanh

| Hook | Mục đích | Khi nào dùng |
|---|---|---|
| `useState` | local state | giá trị đơn giản, thay đổi trigger re-render |
| `useReducer` | local state phức tạp | nhiều action types, state object nhiều fields |
| `useEffect` | side effects | fetch, DOM manipulation, subscriptions |
| `useContext` | đọc Context | tránh prop drilling |
| `useRef` | ref đến DOM / mutable value | focus, timer ID, giá trị không trigger re-render |
| `useMemo` | memoize giá trị | tính toán nặng, tránh tính lại |
| `useCallback` | memoize function | stable reference cho child component / dep array |
| `useLayoutEffect` | effect đồng bộ sau DOM update | đo lường DOM, animation |
| `useId` | tạo unique ID stable | label + input pairing trong SSR |

---

## 3. useState

```jsx
const [count, setCount] = useState(0);

// Functional update — dùng khi next state phụ thuộc prev
setCount(prev => prev + 1);

// Object state — luôn spread để immutable
const [user, setUser] = useState({ name: '', age: 0 });
setUser(prev => ({ ...prev, name: 'Alice' }));

// Array state
const [items, setItems] = useState([]);
setItems(prev => [...prev, newItem]);          // thêm
setItems(prev => prev.filter(i => i.id !== id)); // xóa
setItems(prev => prev.map(i =>                   // sửa
  i.id === id ? { ...i, done: true } : i
));
```

---

## 4. useEffect

```jsx
// 3 mode dependency array
useEffect(() => { /* chạy sau mọi render */ });
useEffect(() => { /* chạy 1 lần khi mount */ }, []);
useEffect(() => { /* chạy khi dep thay đổi */ }, [dep]);

// Cleanup — trả về function
useEffect(() => {
  const controller = new AbortController();

  fetch(url, { signal: controller.signal })
    .then(r => r.json())
    .then(setData)
    .catch(err => { if (err.name !== 'AbortError') setError(err); });

  return () => controller.abort(); // cleanup khi unmount hoặc dep thay đổi
}, [url]);
```

| Deps | Chạy khi |
|---|---|
| (không có) | sau mỗi render |
| `[]` | 1 lần sau mount |
| `[a, b]` | sau mount VÀ mỗi khi a hoặc b thay đổi |

---

## 5. useRef

```jsx
// DOM ref
const inputRef = useRef(null);
<input ref={inputRef} />
inputRef.current.focus(); // sau mount

// Mutable value — không trigger re-render
const timerRef = useRef(null);
timerRef.current = setTimeout(fn, 1000);
clearTimeout(timerRef.current);

// Previous value pattern
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => { ref.current = value; });
  return ref.current; // render trước
}
```

---

## 6. useMemo & useCallback

```jsx
// useMemo — cache giá trị tính toán
const sorted = useMemo(
  () => [...items].sort(compareFn),
  [items]  // chỉ sort lại khi items thay đổi
);

// useCallback — cache function (stable reference)
const handleSubmit = useCallback(
  (data) => { dispatch({ type: 'SUBMIT', data }); },
  [dispatch]  // dispatch từ useReducer luôn stable
);
// Dùng khi: truyền callback vào React.memo component
//           hoặc function trong dependency array của useEffect
```

---

## 7. Context API Pattern

```jsx
// 1. Tạo context + Provider + custom hook (trong 1 file)
const ThemeContext = createContext(null);

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const toggle = useCallback(() =>
    setTheme(t => t === 'light' ? 'dark' : 'light'), []);

  const value = useMemo(() => ({ theme, toggle }), [theme, toggle]);
  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}

export function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error('useTheme must be inside ThemeProvider');
  return ctx;
}

// 2. Wrap app
<ThemeProvider><App /></ThemeProvider>

// 3. Dùng trong component
const { theme, toggle } = useTheme();
```

### Context + useReducer (state phức tạp)

```jsx
// Tách 2 context để tối ưu re-render
const StateCtx    = createContext(null);
const DispatchCtx = createContext(null); // dispatch luôn stable

export function Provider({ children }) {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <DispatchCtx.Provider value={dispatch}>
      <StateCtx.Provider value={state}>
        {children}
      </StateCtx.Provider>
    </DispatchCtx.Provider>
  );
}

export const useAppState    = () => useContext(StateCtx);
export const useAppDispatch = () => useContext(DispatchCtx);
// Component chỉ dispatch → không re-render khi state thay đổi
```

---

## 8. React Router v6

```jsx
// Setup
<BrowserRouter><App /></BrowserRouter>

// Routes
<Routes>
  <Route path="/"              element={<Home />} />
  <Route path="/users/:id"     element={<UserDetail />} />
  <Route path="/dashboard"     element={<DashboardLayout />}>
    <Route index element={<Overview />} />          {/* /dashboard */}
    <Route path="settings" element={<Settings />} /> {/* /dashboard/settings */}
  </Route>
  <Route path="*" element={<NotFound />} />
</Routes>

// Trong DashboardLayout
<Outlet /> {/* children route render ở đây */}
```

| Hook | Trả về | Dùng để |
|---|---|---|
| `useParams()` | `{ id: '42' }` | lấy URL params |
| `useNavigate()` | `navigate(path)` | điều hướng programmatic |
| `useLocation()` | `{ pathname, search, state }` | đọc URL hiện tại |
| `useSearchParams()` | `[params, setParams]` | quản lý query string |

```jsx
// Lazy loading
const Dashboard = lazy(() => import('./pages/Dashboard'));
<Suspense fallback={<Spinner />}>
  <Routes>...</Routes>
</Suspense>

// Protected route
function RequireAuth({ children }) {
  const { user } = useAuth();
  const location = useLocation();
  if (!user) return <Navigate to="/login" state={{ from: location }} replace />;
  return children;
}
```

---

## 9. Component Patterns Nhanh

```jsx
// React.memo — skip re-render nếu props không đổi
const MemoList = React.memo(function List({ items }) {
  return <ul>{items.map(i => <li key={i.id}>{i.name}</li>)}</ul>;
});

// Compound component — children dùng context của parent
function Tabs({ children }) {
  const [active, setActive] = useState(0);
  return <TabsContext.Provider value={{ active, setActive }}>{children}</TabsContext.Provider>;
}
Tabs.Tab    = function Tab({ index, children }) { ... };
Tabs.Panel  = function Panel({ index, children }) { ... };

// Render prop
function MouseTracker({ render }) {
  const [pos, setPos] = useState({ x: 0, y: 0 });
  return <div onMouseMove={e => setPos({ x: e.clientX, y: e.clientY })}>
    {render(pos)}
  </div>;
}
```

---

## 10. Câu Hỏi Phỏng Vấn Thường Gặp

**Q1: `useState` vs `useReducer` — khi nào dùng cái nào?**

> `useState`: state đơn giản, ít action. `useReducer`: state phức tạp (nhiều fields liên quan), nhiều action types, cần test reducer riêng, hoặc next state phụ thuộc vào action type VÀ current state. Cú pháp: `const [state, dispatch] = useReducer(reducerFn, initialState)`.

**Q2: `useMemo` vs `useCallback` — khác nhau thế nào?**

> `useMemo(() => compute(), [deps])` → cache **giá trị** (kết quả tính toán). `useCallback(fn, [deps])` → cache **function** (stable reference). `useCallback(fn, deps)` tương đương `useMemo(() => fn, deps)`. Dùng `useCallback` khi truyền callback vào `React.memo` component hoặc dùng làm dependency.

**Q3: Tại sao `key` prop quan trọng trong list?**

> `key` giúp React nhận dạng element nào thay đổi/thêm/xóa trong reconciliation. Stable key (ID) → React reuse DOM node, update props. Không có key hoặc key=index → React không biết thứ tự đổi, re-create toàn bộ → chậm + bug (mất input state khi reorder).

**Q4: `useEffect` cleanup function — khi nào cần?**

> Cần khi effect tạo ra thứ gì đó cần dọn dẹp: subscription (WebSocket, event listener), timer, AbortController. Cleanup chạy khi: component unmount, VÀ mỗi lần trước khi effect chạy lại (dep thay đổi). Thiếu cleanup → memory leak.

---

## 11. Bài Tập Tự Kiểm Tra

- [ ] Viết `useLocalStorage(key, initialValue)` hook — sync state với localStorage, handle JSON parse error.
- [ ] Implement debounced search: input controlled + `useEffect` + `useRef` timer, chỉ fetch sau 300ms kể từ lần gõ cuối.

---

## 12. Liên Kết

- [[03-State-voi-useState]] — useState chi tiết
- [[08-useEffect-Hook]] — useEffect chi tiết
- [[11-Context-API]] — Context API
- [[13-Context-voi-useReducer]] — Context + useReducer
- [[14-React-Router]] — React Router v6
