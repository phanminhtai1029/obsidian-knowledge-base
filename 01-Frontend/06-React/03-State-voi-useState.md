---
title: "React: State với useState"
section: 06-React
tags: [react, useState, hook, state, lift-state, useReducer, fresher, frontend]
related:
  - "[[02-Component-va-Props]]"
  - "[[08-useEffect-Hook]]"
  - "[[13-Context-voi-useReducer]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [React.docx, react.dev]
---

# React: State với useState

> [!summary] TL;DR
> **State** là dữ liệu thay đổi theo thời gian — khi state thay đổi, React **re-render** component đó và tất cả children. `useState` trả về tuple `[value, setter]` qua **array destructuring**. `setter(newValue)` thay thế state hoàn toàn — không merge như `setState` của class. **Lift state up**: đưa state lên ancestor chung khi nhiều components cần dùng cùng state. **`useReducer`** thay thế `useState` khi state phức tạp, nhiều transitions liên quan nhau.

> [!tip] 🎯 Hiểu trong 30 giây
> **State = "trí nhớ" của component.** `props` là thứ *cha đưa cho* (không sửa được), còn `state` là thứ *component tự giữ và tự đổi*. Mỗi khi state đổi, React **vẽ lại (re-render)** component đó để màn hình khớp dữ liệu mới. `useState(0)` đưa cho bạn 2 thứ: **giá trị hiện tại** + **cái nút để đổi nó** → `const [count, setCount] = useState(0)`.
>
> **2 điều khiến người mới "trật" (và hay ra thi):**
> 1. **State giống ảnh chụp mỗi lần render, không phải biến sống.** Trong một lần xử lý click, `count` bị "đóng băng". Nên gọi `setCount(count + 1)` ba lần chỉ tăng **1** (cả ba đều đọc cùng một `count` cũ rồi React **gộp lại — batching**). Muốn tăng đúng **3** phải dùng *hàm cập nhật*: `setCount(prev => prev + 1)` — React đưa giá trị mới nhất vào từng lần.
> 2. **Phải tạo dữ liệu MỚI, đừng sửa tại chỗ.** React phát hiện thay đổi bằng so sánh *địa chỉ*. `arr.push(x)` rồi `setArr(arr)` → cùng địa chỉ → **không re-render**. Phải `setArr([...arr, x])` (tạo mảng mới). Đây là lý do kỹ thuật spread cực quan trọng trong React.

---

## 1. Khái niệm

### State vs Props

| | Props | State |
|---|---|---|
| Nguồn gốc | Từ parent truyền xuống | Tạo và quản lý trong chính component |
| Mutability | Read-only (không sửa được) | Có thể cập nhật qua setter |
| Khi thay đổi | Parent re-render truyền props mới | Component tự re-render |
| Mục đích | Cấu hình component từ ngoài | Dữ liệu nội bộ thay đổi theo thời gian |

### Khi nào cần State?

Cần state khi UI cần phản ứng với một sự kiện/thay đổi:
- Toggle (dark/light mode, show/hide, open/close)
- Form input values
- Data fetched từ API
- Pagination, filtering, sorting

```
★ Insight ─────────────────────────────────────
• State là "ảnh chụp" cho mỗi lần render, KHÔNG phải biến sống: trong một handler,
  `count` đóng băng theo giá trị lúc render đó → `setCount(count+1)` ba lần chỉ +1.
  Vì vậy khi state mới phụ thuộc state cũ, dùng updater hàm `setCount(prev=>prev+1)`
  — React đưa giá trị mới nhất vào. Hiểu điều này = hiểu vì sao đọc state ngay sau
  set vẫn ra giá trị cũ ([[../03-Advanced-JavaScript/03-Closure]] là gốc rễ).
• React phát hiện đổi state bằng so sánh THAM CHIẾU, nên mutate trực tiếp
  (push/gán field) → cùng địa chỉ → KHÔNG re-render. Luôn tạo object/array MỚI
  (spread). Đây chính là lý do kỹ thuật spread shallow-copy ([[../03-Advanced-JavaScript/10-ES6-Rest-Spread]])
  là kỹ năng sống còn của React. State liên quan nhiều/phức tạp → cân nhắc useReducer.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp / API

### 2.1 useState cơ bản

```jsx
import { useState } from 'react';

function Counter() {
  // Array destructuring — [currentValue, updaterFunction]
  const [count, setCount] = useState(0); // 0 là initial state

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
      <button onClick={() => setCount(count - 1)}>-</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

**Quy ước đặt tên:** `[noun, setNoun]` — ví dụ: `[status, setStatus]`, `[isOpen, setIsOpen]`, `[users, setUsers]`.

### 2.2 useState với function updater

```jsx
// Khi new state phụ thuộc vào old state — dùng function updater để tránh stale closure
function Counter() {
  const [count, setCount] = useState(0);

  // CÁCH SAI (có thể stale trong async context):
  // setCount(count + 1);

  // CÁCH ĐÚNG — function updater nhận prev state:
  const increment = () => setCount(prev => prev + 1);
  const decrement = () => setCount(prev => prev - 1);

  // Đặc biệt quan trọng khi gọi nhiều lần liên tiếp:
  const incrementThrice = () => {
    setCount(prev => prev + 1); // 0 → 1
    setCount(prev => prev + 1); // 1 → 2
    setCount(prev => prev + 1); // 2 → 3
    // ✅ Đúng: mỗi call nhận prev state mới nhất
  };

  return (
    <div>
      <p>{count}</p>
      <button onClick={increment}>+1</button>
      <button onClick={incrementThrice}>+3</button>
    </div>
  );
}
```

### 2.3 Toggle State (Boolean)

```jsx
function ToggleButton() {
  const [isOpen, setIsOpen] = useState(false);

  const toggle = () => setIsOpen(prev => !prev); // flip boolean

  return (
    <div>
      <button onClick={toggle}>
        {isOpen ? 'Close Menu' : 'Open Menu'}
      </button>
      {isOpen && (
        <nav>
          <a href="/">Home</a>
          <a href="/about">About</a>
        </nav>
      )}
    </div>
  );
}
```

### 2.4 Object State — phải spread

```jsx
function UserForm() {
  const [user, setUser] = useState({ name: '', email: '', age: 0 });

  // SAIT — setter THAY THẾ toàn bộ state, không merge
  // setUser({ name: 'Alice' }); → { name: 'Alice' } — mất email và age!

  // ĐÚNG — spread object cũ trước, rồi override field cần thay đổi
  const updateName = (name) => setUser(prev => ({ ...prev, name }));
  const updateEmail = (email) => setUser(prev => ({ ...prev, email }));

  return (
    <form>
      <input
        value={user.name}
        onChange={e => updateName(e.target.value)}
        placeholder="Name"
      />
      <input
        value={user.email}
        onChange={e => updateEmail(e.target.value)}
        placeholder="Email"
      />
    </form>
  );
}
```

### 2.5 Array State — immutable updates

```jsx
function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', done: false },
  ]);

  const addTodo = (text) => {
    const newTodo = { id: Date.now(), text, done: false };
    setTodos(prev => [...prev, newTodo]);  // spread, rồi thêm
  };

  const toggleTodo = (id) => {
    setTodos(prev =>
      prev.map(todo => todo.id === id ? { ...todo, done: !todo.done } : todo)
    );
  };

  const removeTodo = (id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));  // filter out
  };

  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.done}
            onChange={() => toggleTodo(todo.id)}
          />
          <span style={{ textDecoration: todo.done ? 'line-through' : 'none' }}>
            {todo.text}
          </span>
          <button onClick={() => removeTodo(todo.id)}>✕</button>
        </li>
      ))}
    </ul>
  );
}
```

### 2.6 Lift State Up

```jsx
// Khi 2 sibling components cần cùng state → lift up lên parent chung

function TemperatureInput({ scale, temp, onTempChange }) {
  return (
    <label>
      {scale === 'C' ? 'Celsius' : 'Fahrenheit'}:
      <input value={temp} onChange={e => onTempChange(e.target.value)} />
    </label>
  );
}

// State ở parent — pass xuống cả 2 children
function Calculator() {
  const [celsius, setCelsius] = useState('');

  const fahrenheit = celsius !== '' ? (parseFloat(celsius) * 9/5 + 32).toFixed(1) : '';

  return (
    <div>
      <TemperatureInput scale="C" temp={celsius} onTempChange={setCelsius} />
      <TemperatureInput scale="F" temp={fahrenheit} onTempChange={() => {}} />
      {celsius && <p>{celsius}°C = {fahrenheit}°F</p>}
    </div>
  );
}
```

### 2.7 useReducer — khi state phức tạp hơn

```jsx
import { useReducer } from 'react';

// Reducer function: (currentState, action) → newState
function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT': return { count: state.count + (action.amount ?? 1) };
    case 'DECREMENT': return { count: state.count - 1 };
    case 'RESET':     return { count: 0 };
    default:          return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+1</button>
      <button onClick={() => dispatch({ type: 'INCREMENT', amount: 5 })}>+5</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-1</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
    </div>
  );
}
```

---

## 3. Ví dụ minh họa

### Ví dụ 1: Accordion component

```jsx
function AccordionItem({ title, children }) {
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <div className="accordion-item">
      <button
        className="accordion-header"
        onClick={() => setIsExpanded(prev => !prev)}
        aria-expanded={isExpanded}
      >
        {title}
        <span>{isExpanded ? '▲' : '▼'}</span>
      </button>
      {isExpanded && (
        <div className="accordion-body">{children}</div>
      )}
    </div>
  );
}

function FAQ() {
  return (
    <div>
      <AccordionItem title="What is React?">
        <p>React is a JavaScript library for building user interfaces.</p>
      </AccordionItem>
      <AccordionItem title="What are Hooks?">
        <p>Hooks let you use state and other React features in function components.</p>
      </AccordionItem>
    </div>
  );
}
```

### Ví dụ 2: Shopping cart với useReducer

```jsx
const cartReducer = (state, action) => {
  switch (action.type) {
    case 'ADD': {
      const existing = state.find(item => item.id === action.item.id);
      if (existing) {
        return state.map(item =>
          item.id === action.item.id ? { ...item, qty: item.qty + 1 } : item
        );
      }
      return [...state, { ...action.item, qty: 1 }];
    }
    case 'REMOVE':
      return state.filter(item => item.id !== action.id);
    case 'CLEAR':
      return [];
    default:
      return state;
  }
};

function Cart() {
  const [items, dispatch] = useReducer(cartReducer, []);
  const total = items.reduce((sum, item) => sum + item.price * item.qty, 0);

  return (
    <div>
      <button onClick={() => dispatch({ type: 'ADD', item: { id: 1, name: 'Book', price: 15 } })}>
        Add Book
      </button>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.name} × {item.qty} = ${(item.price * item.qty).toFixed(2)}
            <button onClick={() => dispatch({ type: 'REMOVE', id: item.id })}>✕</button>
          </li>
        ))}
      </ul>
      <p>Total: ${total.toFixed(2)}</p>
      <button onClick={() => dispatch({ type: 'CLEAR' })}>Clear Cart</button>
    </div>
  );
}
```

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Pitfall 1: State updates là bất đồng bộ (asynchronous)
> `setCount(count + 1); console.log(count);` → `count` vẫn là giá trị CŨ! State chỉ thực sự update ở lần render tiếp theo. Dùng function updater `setCount(prev => prev + 1)` khi cần reliable updates.

> [!warning] Pitfall 2: Không mutate state trực tiếp
> ```jsx
> // SAI — mutate array/object trực tiếp
> todos.push(newTodo);    setTodos(todos); // React không detect thay đổi!
> user.name = 'Bob';      setUser(user);   // Cùng reference — không re-render!
>
> // ĐÚNG — tạo object/array mới
> setTodos([...todos, newTodo]);
> setUser({ ...user, name: 'Bob' });
> ```

> [!tip] useState vs useReducer — khi nào dùng cái nào?
> **useState**: state đơn giản, ít transitions, các trường độc lập nhau. **useReducer**: nhiều sub-values liên quan (form phức tạp), next state phụ thuộc vào nhiều action types, logic update phức tạp cần test. Nếu thấy mình viết nhiều `setX` cùng lúc → xem xét chuyển sang `useReducer`.

---

## 5. Câu hỏi phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Gọi `setCount(count+1)` 3 lần thì count tăng 1 hay 3?"
> *"Chỉ tăng 1 ạ. Lý do là hai chuyện kết hợp: thứ nhất, trong một lần xử lý sự kiện, biến `count` là giá trị đóng băng của lần render hiện tại, nên cả ba lệnh đều đọc cùng một `count` cũ và tính ra cùng một kết quả. Thứ hai là State Batching — React gộp nhiều lệnh setState trong cùng một event lại và chỉ re-render một lần với giá trị cuối cùng, nên ba lệnh `setCount(count+1)` đều thành 'đặt count = giá trị cũ + 1'. Muốn nó tăng đúng 3 thì em dùng dạng hàm cập nhật: `setCount(prev => prev + 1)`, vì khi đó React truyền vào giá trị mới nhất sau mỗi lần, ba lần liên tiếp sẽ ra cộng dồn. Batching là tính năng tốt vì giảm số lần render thừa."*

> [!example] 🗣️ Trả lời mẫu — "useRef khác useState ở đâu, khi nào bắt buộc dùng useRef?"
> *"Điểm khác cốt lõi: đổi `useState` thì component re-render, còn đổi `useRef` thì KHÔNG re-render — ref là một hộp giữ giá trị tồn tại xuyên các lần render mà không ảnh hưởng giao diện. Vì vậy em dùng useState cho dữ liệu cần hiển thị lên UI, còn useRef cho những giá trị 'hậu trường' không cần vẽ lại: ví dụ giữ tham chiếu tới một DOM node để focus input, lưu id của setInterval/setTimeout để clear, hay đếm số lần render để debug. Trường hợp bắt buộc dùng useRef điển hình là truy cập trực tiếp DOM element qua `ref` rồi gọi `.focus()`, dùng useState ở đây vừa thừa vừa gây render lặp."*

> [!note] 🧠 Mẹo nhớ
> **State = trí nhớ, đổi → re-render.** `setCount(count+1)` ×3 = **+1** (batching + ảnh chụp) → muốn +3 dùng **`prev => prev+1`**. **Đừng mutate, hãy tạo mới (spread).** **useRef = giữ giá trị nhưng KHÔNG re-render** (DOM, timer id).

**Q1: useState là gì? Giải thích cú pháp `const [count, setCount] = useState(0)`.**

> `useState` là React Hook cho phép function component có **state**. `useState(0)` trả về **array 2 phần tử**: `[currentState, setterFunction]`. Ta dùng **array destructuring** để lấy ra và đặt tên. `0` là initial state. `setCount(newValue)` cập nhật state và trigger re-render. Convention đặt tên: `[noun, setNoun]`.

**Q2: Tại sao phải dùng setter function, không được mutate state trực tiếp?**

> React xác định khi nào cần re-render bằng cách so sánh reference. Nếu mutate array/object trực tiếp → reference không đổi → React không biết state thay đổi → không re-render. Setter function: (1) tạo **reference mới**, (2) **schedule re-render**, (3) **batch** nhiều updates cùng nhau để tối ưu performance.

**Q3: Lift state up là gì? Khi nào cần dùng?**

> Lift state up là pattern: **di chuyển state lên ancestor component chung** khi nhiều sibling components cần đọc/cập nhật cùng state. Ví dụ: tab A và tab B cùng hiển thị theme (dark/light) → đặt `theme` state ở parent của cả A và B, pass xuống qua props. Khi cần chia sẻ state ở nhiều nơi hơn nữa → dùng Context API.

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Tạo component `Tabs` nhận `tabs: Array<{label, content}>`. State `activeTab` track tab đang active. Click vào tab label → hiển thị content tương ứng. Style active tab khác với inactive.

- [ ] **Bài 2:** Dùng `useReducer` xây dựng traffic light (`red → green → yellow → red`). Mỗi lần click "Next" → dispatch action `NEXT` → reducer tính state tiếp theo.

---

## 7. Liên kết

- [[02-Component-va-Props]] — Props vs State
- [[08-useEffect-Hook]] — useState + useEffect (data fetching pattern)
- [[13-Context-voi-useReducer]] — useReducer kết hợp Context cho global state
- [[12-Context-voi-useState]] — useState kết hợp Context
