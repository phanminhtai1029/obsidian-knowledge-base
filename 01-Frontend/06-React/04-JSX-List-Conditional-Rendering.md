---
title: "React: JSX, List & Conditional Rendering"
section: 06-React
tags: [react, jsx, map, key, conditional, fragment, fresher, frontend]
related:
  - "[[01-React-Overview]]"
  - "[[02-Component-va-Props]]"
  - "[[03-State-voi-useState]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [React.docx, react.dev]
---

# React: JSX, List & Conditional Rendering

> [!summary] TL;DR
> **JSX** là JavaScript XML — cú pháp sugar cho `React.createElement`. Quy tắc: một root element (hoặc Fragment `<>...</>`), dùng `className` thay `class`, `htmlFor` thay `for`, camelCase cho attributes. **List**: dùng `.map()` để render array, mỗi item **bắt buộc** có `key` prop duy nhất và ổn định (không dùng index nếu list có thể thay đổi thứ tự). **Conditional**: ternary `? :` cho if/else, `&&` cho if-only, early return cho phức tạp hơn.

---

## 1. Khái niệm

### JSX là gì?

JSX (JavaScript XML) là **cú pháp mở rộng** của JavaScript cho phép viết HTML-like code trong file JS/JSX. Trước khi chạy, Babel/Vite **compile** JSX về `React.createElement()`:

```jsx
// Developer viết (JSX):
const el = <button className="btn" onClick={handleClick}>Click me</button>;

// Sau khi compile (plain JS):
const el = React.createElement(
  'button',
  { className: 'btn', onClick: handleClick },
  'Click me'
);
```

---

## 2. Cú pháp / API

### 2.1 JSX Rules

```jsx
// 1. Mỗi component phải return 1 root element
// SAI:
function Bad() {
  return (
    <h1>Title</h1>
    <p>Paragraph</p>  // Error: Adjacent JSX elements must be wrapped
  );
}

// ĐÚNG: wrap trong div
function Good1() {
  return (
    <div>
      <h1>Title</h1>
      <p>Paragraph</p>
    </div>
  );
}

// ĐÚNG: dùng Fragment (không thêm DOM node)
function Good2() {
  return (
    <>
      <h1>Title</h1>
      <p>Paragraph</p>
    </>
  );
}

// Fragment tường minh (khi cần key prop):
function Good3({ items }) {
  return items.map(item => (
    <React.Fragment key={item.id}>
      <dt>{item.term}</dt>
      <dd>{item.definition}</dd>
    </React.Fragment>
  ));
}
```

### 2.2 JSX Attribute Differences

```jsx
// HTML vs JSX — các điểm khác nhau quan trọng
function AttributeExamples() {
  return (
    <div>
      {/* class → className */}
      <div className="container">...</div>

      {/* for → htmlFor (label) */}
      <label htmlFor="email">Email:</label>
      <input id="email" type="email" />

      {/* style nhận object, không phải string; camelCase property names */}
      <p style={{ color: 'red', fontSize: 16, marginTop: '8px' }}>
        Styled text
      </p>

      {/* Event handlers — camelCase */}
      <button onClick={handleClick} onMouseEnter={handleHover}>
        Hover or Click
      </button>

      {/* Boolean attributes */}
      <input disabled />           {/* disabled={true} */}
      <input disabled={false} />   {/* not disabled */}
      <input readOnly />           {/* không phải readonly */}

      {/* Custom data attributes — vẫn là lowercase */}
      <div data-testid="user-card" data-user-id="123">...</div>
    </div>
  );
}
```

### 2.3 JSX Expressions `{}`

```jsx
function DynamicContent() {
  const user = { name: 'Alice', age: 25 };
  const isLoggedIn = true;
  const items = ['apple', 'banana', 'cherry'];

  return (
    <div>
      {/* Biến */}
      <p>Hello, {user.name}!</p>

      {/* Expression (bất kỳ biểu thức JS nào) */}
      <p>Age: {user.age * 2} (doubled)</p>
      <p>Name: {user.name.toUpperCase()}</p>

      {/* Template literal */}
      <p>{`Today is ${new Date().toLocaleDateString()}`}</p>

      {/* Toán tử */}
      <p>Items: {items.length}</p>

      {/* Không thể dùng statements (if, for) trong {} */}
      {/* Chỉ expressions */}
    </div>
  );
}
```

### 2.4 List Rendering với `.map()`

```jsx
// Luôn cần key prop
function FruitList({ fruits }) {
  return (
    <ul>
      {fruits.map(fruit => (
        <li key={fruit.id}>  {/* key phải unique trong list này */}
          {fruit.name}
        </li>
      ))}
    </ul>
  );
}

// Ví dụ thực tế với object array
const products = [
  { id: 'p1', name: 'Laptop',  price: 999 },
  { id: 'p2', name: 'Phone',   price: 599 },
  { id: 'p3', name: 'Tablet',  price: 399 },
];

function ProductTable() {
  return (
    <table>
      <thead>
        <tr><th>Name</th><th>Price</th></tr>
      </thead>
      <tbody>
        {products.map(({ id, name, price }) => (
          <tr key={id}>
            <td>{name}</td>
            <td>${price}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

### 2.5 Key Prop — Rules

```jsx
// KEY phải:
// 1. Duy nhất trong danh sách đó (không cần global unique)
// 2. Ổn định giữa các lần render (không thay đổi nếu data không thay đổi)
// 3. Không dùng index nếu list có thể reorder/insert/delete ở giữa

// SAIT (khi list có thể thay đổi):
const bad = items.map((item, index) => <li key={index}>{item}</li>);

// ĐÚNG (stable unique ID từ data):
const good = items.map(item => <li key={item.id}>{item.name}</li>);

// Nếu data không có ID, dùng combination của stable values:
const tags = ['react', 'typescript', 'vite'];
const tagEls = tags.map(tag => <span key={tag} className="tag">{tag}</span>);
// tag value là unique và stable → OK
```

### 2.6 Conditional Rendering

```jsx
function ConditionalExamples({ isLoggedIn, user, notifications, role }) {
  return (
    <div>
      {/* 1. Ternary — if/else */}
      <p>{isLoggedIn ? `Welcome, ${user.name}!` : 'Please log in'}</p>

      {/* 2. && — chỉ render khi true */}
      {notifications.length > 0 && (
        <span className="badge">{notifications.length}</span>
      )}

      {/* 3. Nullish coalescing cho default content */}
      <p>{user.bio ?? 'No bio provided'}</p>

      {/* 4. Conditional className */}
      <button className={`btn ${isLoggedIn ? 'btn-primary' : 'btn-secondary'}`}>
        {isLoggedIn ? 'Logout' : 'Login'}
      </button>
    </div>
  );
}

// 5. Early return — cho logic phức tạp hơn
function UserProfile({ user, isLoading, error }) {
  if (isLoading) return <Spinner />;
  if (error)     return <ErrorMessage message={error} />;
  if (!user)     return <p>No user found</p>;

  // Happy path — biết chắc user tồn tại
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// 6. Switch-like với object lookup
const STATUS_COMPONENTS = {
  loading: () => <Spinner />,
  error:   ({ message }) => <p className="error">{message}</p>,
  empty:   () => <p>No data found</p>,
  success: ({ data }) => <DataTable data={data} />,
};

function DataView({ status, data, error }) {
  const Component = STATUS_COMPONENTS[status];
  return Component ? <Component data={data} message={error} /> : null;
}
```

---

## 3. Ví dụ minh họa

### Ví dụ 1: Filterable list

```jsx
function FilterableList({ items }) {
  const [filter, setFilter] = useState('');

  const filtered = items.filter(item =>
    item.name.toLowerCase().includes(filter.toLowerCase())
  );

  return (
    <div>
      <input
        type="search"
        value={filter}
        onChange={e => setFilter(e.target.value)}
        placeholder="Search..."
      />

      {filtered.length === 0 ? (
        <p>No results for "{filter}"</p>
      ) : (
        <ul>
          {filtered.map(item => (
            <li key={item.id}>
              {item.name}
              {item.featured && <span className="badge">Featured</span>}
            </li>
          ))}
        </ul>
      )}

      <p>{filtered.length} of {items.length} items</p>
    </div>
  );
}
```

### Ví dụ 2: Notification list với status indicators

```jsx
const SEVERITY_STYLES = {
  info:    { icon: 'ℹ️', className: 'notification-info'    },
  success: { icon: '✅', className: 'notification-success' },
  warning: { icon: '⚠️', className: 'notification-warning' },
  error:   { icon: '🚨', className: 'notification-error'   },
};

function NotificationItem({ id, message, severity, onDismiss }) {
  const { icon, className } = SEVERITY_STYLES[severity] ?? SEVERITY_STYLES.info;

  return (
    <div className={`notification ${className}`}>
      <span>{icon}</span>
      <p>{message}</p>
      <button onClick={() => onDismiss(id)} aria-label="Dismiss">✕</button>
    </div>
  );
}

function NotificationCenter({ notifications, onDismiss }) {
  if (notifications.length === 0) {
    return <p className="empty-state">No notifications</p>;
  }

  return (
    <div className="notification-center">
      {notifications.map(n => (
        <NotificationItem
          key={n.id}
          {...n}
          onDismiss={onDismiss}
        />
      ))}
    </div>
  );
}
```

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Pitfall 1: `&&` với số 0 — render unexpected
> `{count && <Component />}` — khi `count = 0`, JavaScript render số `0` lên DOM! Vì `0` là falsy nhưng React render nó như text node. Fix: dùng ternary `{count > 0 ? <Component /> : null}` hoặc explicit boolean `{!!count && <Component />}` hoặc `{Boolean(count) && <Component />}`.

> [!warning] Pitfall 2: Index làm key khi list có thể reorder
> Dùng index làm key: khi item bị xóa ở giữa, React sẽ nhầm và re-render sai component. Input state (typing) có thể bị gán nhầm cho item khác sau khi reorder. **Luôn dùng stable ID từ data**.

> [!tip] Nested ternary — khó đọc, nên tách thành early return
> ```jsx
> // Khó đọc:
> return loading ? <Spinner /> : error ? <Error /> : data ? <Data /> : <Empty />;
>
> // Dễ đọc hơn:
> if (loading) return <Spinner />;
> if (error) return <Error />;
> if (!data) return <Empty />;
> return <Data />;
> ```

---

## 5. Câu hỏi phỏng vấn thường gặp

**Q1: JSX là gì? Tại sao dùng `className` thay `class`?**

> JSX (JavaScript XML) là cú pháp mở rộng của JS cho phép viết HTML-like code. Trước khi chạy, Babel biên dịch JSX về `React.createElement()`. Dùng `className` vì `class` là **reserved keyword** trong JavaScript (dùng cho ES6 classes). Tương tự, `for` → `htmlFor`, `tabindex` → `tabIndex`. Một số props như `data-*` và `aria-*` vẫn giữ lowercase.

**Q2: Tại sao cần `key` prop khi render list? Index có ổn không?**

> `key` giúp React **identify** mỗi item trong list khi reconciliation — React biết item nào thêm mới, xóa, hay chỉ thay đổi vị trí. Không có key: React phải re-render toàn bộ list. Index làm key: không ổn khi list có thể **thêm/xóa ở giữa hoặc reorder** — index của một item thay đổi → React sẽ nhầm component và có thể gán state (như input value) sai. Luôn dùng **stable unique ID** từ data.

**Q3: Fragment là gì? Khi nào dùng?**

> **Fragment** (`<>...</>` hoặc `<React.Fragment>`) là wrapper ảo — cho phép return nhiều elements mà **không thêm DOM node thừa**. Dùng khi: (1) component cần return nhiều siblings, (2) thêm wrapper div sẽ phá vỡ CSS layout (ví dụ flex/grid), (3) render table row/cell cần đúng cấu trúc HTML. Dùng `<React.Fragment key={id}>` khi cần key prop trong list.

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Tạo component `DataGrid({ columns, rows })` — `columns: string[]`, `rows: object[]`. Render table với `<thead>` từ columns và `<tbody>` từ rows. Key phải stable. Xử lý empty state "No data".

- [ ] **Bài 2:** Tạo component `ConditionalCard({ status })` — status có thể là `'loading'`, `'error'`, `'empty'`, `'success'` (với data). Dùng early return pattern cho mỗi case. Test với các giá trị khác nhau.

---

## 7. Liên kết

- [[02-Component-va-Props]] — Component cơ bản, props
- [[03-State-voi-useState]] — State điều khiển conditional rendering
- [[05-Event-Handling]] — onClick, onChange trong JSX
- [[06-Form-Handling]] — Form elements trong JSX
