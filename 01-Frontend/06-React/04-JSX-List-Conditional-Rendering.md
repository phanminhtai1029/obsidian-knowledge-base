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
> **JSX** (JavaScript XML) = cú pháp cho viết "HTML" ngay trong JS; thực chất là lớp áo cho lời gọi `React.createElement`. Quy tắc: mỗi component trả về **một thẻ gốc** (hoặc bọc bằng Fragment `<>...</>`); viết `className` thay `class`, `htmlFor` thay `for`; tên thuộc tính dùng camelCase (vd `onClick`). **List** (danh sách): dùng `.map()` để render một mảng ra nhiều phần tử, mỗi phần tử **bắt buộc** có `key` (mã định danh) duy nhất và ổn định — **đừng dùng index** nếu danh sách có thể đổi thứ tự/xóa chèn. **Conditional** (hiển thị có điều kiện): `? :` (ternary) cho if/else, `&&` cho "chỉ hiện khi đúng", hoặc `return` sớm cho trường hợp phức tạp.

> [!tip] 🎯 Hiểu trong 30 giây
> **JSX = viết "HTML" ngay trong JavaScript.** Thực chất mỗi thẻ JSX bị dịch thành một lời gọi hàm trả về object, nên trong dấu `{}` bạn chỉ nhét được **biểu thức có giá trị** (biến, phép tính, ternary) — không nhét được `if`/`for`. Và vì `class` là từ khóa JS nên phải viết `className`.
>
> **List:** muốn render một mảng ra giao diện thì dùng `.map()`, và mỗi phần tử **phải có `key`** — một "mã định danh" để React biết item nào là item nào khi danh sách thay đổi.
>
> **⚠️ 2 bẫy CỰC hay ra thi:**
> 1. **`key={index}` là bad practice:** index không gắn chặt với *dữ liệu* mà gắn với *vị trí*. Khi bạn xóa/chèn/đảo phần tử ở giữa, index của các item bị xô lệch → React khớp nhầm item cũ với item mới → state cục bộ (vd nội dung ô input) **dính nhầm sang dòng khác**, UI lỗi. → Dùng `key={item.id}` (mã ổn định từ data).
> 2. **`{count && <X/>}` khi `count = 0`** sẽ in số **`0`** ra màn hình (vì `0` falsy nhưng React vẫn render số). → Viết `{count > 0 && <X/>}`.

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

```
★ Insight ─────────────────────────────────────
• JSX chỉ là JS trá hình: mỗi thẻ → một lời gọi createElement trả về OBJECT mô tả
  UI. Vì vậy trong `{}` chỉ đặt được BIỂU THỨC (có giá trị) chứ không đặt được
  câu lệnh (if/for) — và className thay class vì class là từ khoá JS. Nhớ "JSX =
  hàm trả object" thì mọi quy tắc lạ của nó tự suy ra được.
• key KHÔNG phải để "hết warning" — nó là DANH TÍNH giúp React khớp item cũ↔mới
  khi reconcile. Dùng index làm key + list reorder/xoá giữa → React gán nhầm
  state (vd ô input) sang item khác. Và bẫy kinh điển: `{count && <X/>}` khi
  count=0 sẽ IN SỐ 0 ra màn hình (0 là falsy nhưng React vẫn render) → dùng
  `count > 0 && ...`.
─────────────────────────────────────────────────
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

### 2.7 `dangerouslySetInnerHTML` & render Markdown an toàn (UI chatbot)

> [!warning] React mặc định **an toàn**, nhưng `dangerouslySetInnerHTML` mở lại cửa XSS
> Khi bạn viết `{userInput}` trong JSX, React **escape** (chuyển ký tự đặc biệt như `<` thành `&lt;`) → chuỗi `<script>` hiện ra dưới dạng **chữ**, không chạy. Đây là lớp chống **XSS (Cross-Site Scripting — chèn mã độc qua dữ liệu người dùng)** mặc định. Nhưng để chèn **HTML thật** (vd render Markdown của LLM thành đậm/nghiêng/list), bạn phải dùng `dangerouslySetInnerHTML` — React cố ý đặt tên "dangerously" để cảnh báo: lúc này **bạn tự chịu trách nhiệm** chống XSS.

Ngữ cảnh **chatbot AI**: model trả về **Markdown** (chữ đậm, khối code, link…), ta muốn render đẹp. Hai cách:

```jsx
// ❌ NGUY HIỂM: markdown → HTML rồi nhét thẳng, không lọc → dính XSS nếu nội dung chứa <script>/<img onerror>
import { marked } from "marked";
function Bubble({ markdown }) {
  return <div dangerouslySetInnerHTML={{ __html: marked.parse(markdown) }} />;
}

// ✅ AN TOÀN: SANITIZE bằng DOMPurify trước khi đưa vào DOM (loại bỏ thẻ/thuộc tính nguy hiểm)
import { marked } from "marked";
import DOMPurify from "dompurify";
function Bubble({ markdown }) {
  const clean = DOMPurify.sanitize(marked.parse(markdown)); // cắt <script>, on* handler, javascript: ...
  return <div dangerouslySetInnerHTML={{ __html: clean }} />;
}

// ✅✅ TỐT NHẤT cho React: dùng react-markdown — KHÔNG đụng dangerouslySetInnerHTML,
//      nó parse Markdown thành React element thật (đã an toàn), HTML thô mặc định bị bỏ
import Markdown from "react-markdown";
function Bubble({ markdown }) {
  return <Markdown>{markdown}</Markdown>;
}
```

> [!tip] Vì sao output LLM cũng phải coi là "không tin cậy"?
> Người dùng có thể **dẫn dụ model in ra HTML/script độc** (một dạng [[../../04-AI/04-LangGraph-Agentic/03-Tool-Calling-Tavily|prompt injection]]), hoặc nội dung lấy từ web qua RAG đã chứa mã độc. Vì vậy **đừng tin output model** — vẫn sanitize như tin người dùng.

```
★ Insight ─────────────────────────────────────
• Quy tắc vàng: text → `{value}` (React tự escape, an toàn). HTML thật →
  bắt buộc sanitize (DOMPurify) HOẶC dùng react-markdown (không render HTML thô).
• Bản chất XSS giống note DOM thuần ([[../02-DOM-Event/13-DOM-Performance]]):
  `dangerouslySetInnerHTML` của React ≈ `innerHTML` của Vanilla — cùng rủi ro.
─────────────────────────────────────────────────
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

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Vì sao `key={index}` là bad practice?"
> *"Vì key là danh tính để React khớp item cũ với item mới khi cập nhật danh sách. Nếu lấy index của mảng làm key thì key gắn với vị trí chứ không gắn với dữ liệu, mà vị trí lại thay đổi khi mình thêm, xóa hoặc đảo phần tử ở giữa. Ví dụ xóa phần tử đầu, mọi index của các phần tử sau bị dịch lên một, React tưởng đó vẫn là các item cũ ở đúng vị trí nên giữ nguyên DOM và state cũ, dẫn tới sai — kinh điển là state cục bộ như nội dung ô input bị gán nhầm sang dòng khác, hoặc checkbox tick nhầm. Cách đúng là dùng một id ổn định và duy nhất từ chính dữ liệu. Chỉ khi danh sách tĩnh, không bao giờ đổi thứ tự thì dùng index mới tạm chấp nhận được."*

> [!note] 🧠 Mẹo nhớ
> **JSX = HTML trong JS, `{}` chỉ chứa biểu thức, `class`→`className`.** **`key` theo DATA (id), đừng theo index.** **`count && <X/>` → dùng `count > 0 && <X/>`** kẻo in số 0.

**Q1: JSX là gì? Tại sao dùng `className` thay `class`?**

> JSX (JavaScript XML) là cú pháp mở rộng của JS cho phép viết HTML-like code. Trước khi chạy, Babel biên dịch JSX về `React.createElement()`. Dùng `className` vì `class` là **reserved keyword** trong JavaScript (dùng cho ES6 classes). Tương tự, `for` → `htmlFor`, `tabindex` → `tabIndex`. Một số props như `data-*` và `aria-*` vẫn giữ lowercase.

**Q2: Tại sao cần `key` prop khi render list? Index có ổn không?**

> `key` giúp React **identify** mỗi item trong list khi reconciliation — React biết item nào thêm mới, xóa, hay chỉ thay đổi vị trí. Không có key: React phải re-render toàn bộ list. Index làm key: không ổn khi list có thể **thêm/xóa ở giữa hoặc reorder** — index của một item thay đổi → React sẽ nhầm component và có thể gán state (như input value) sai. Luôn dùng **stable unique ID** từ data.

**Q3: Fragment là gì? Khi nào dùng?**

> **Fragment** (`<>...</>` hoặc `<React.Fragment>`) là wrapper ảo — cho phép return nhiều elements mà **không thêm DOM node thừa**. Dùng khi: (1) component cần return nhiều siblings, (2) thêm wrapper div sẽ phá vỡ CSS layout (ví dụ flex/grid), (3) render table row/cell cần đúng cấu trúc HTML. Dùng `<React.Fragment key={id}>` khi cần key prop trong list.

**Q4: Render câu trả lời Markdown của chatbot ra HTML thế nào cho an toàn?**

> Mặc định JSX `{value}` **escape** mọi thứ → an toàn nhưng không render được HTML (Markdown sẽ hiện ra dạng chữ thô). Muốn render HTML thật phải dùng `dangerouslySetInnerHTML` — lúc này tự chịu rủi ro **XSS**. Ba lựa chọn: (1) **tệ nhất**: `marked.parse(md)` rồi nhét thẳng → dính XSS; (2) **an toàn**: `DOMPurify.sanitize(...)` trước khi đưa vào `dangerouslySetInnerHTML`; (3) **tốt nhất trong React**: dùng `react-markdown` — nó parse Markdown thành React element thật, mặc định bỏ HTML thô nên không cần `dangerouslySetInnerHTML`. Quan trọng: **coi output của LLM như dữ liệu không tin cậy** (có thể bị prompt injection ép in mã độc) → luôn sanitize.

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
