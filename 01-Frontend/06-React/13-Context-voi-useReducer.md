---
title: "React: Context với useReducer"
section: 06-React
tags: [react, context, useReducer, dispatch, reducer, action, fresher, frontend]
related:
  - "[[11-Context-API]]"
  - "[[12-Context-voi-useState]]"
  - "[[03-State-voi-useState]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 45m
source: [react.dev/learn/scaling-up-with-reducer-and-context]
---

# React: Context với useReducer

> [!summary] TL;DR
> Pattern **Context + useReducer**: dùng `useReducer` (thay `useState`) trong Provider khi state phức tạp, có nhiều kiểu thay đổi (action types). **Tối ưu hiệu năng**: tách làm 2 context riêng — `StateContext` (giá trị state, *đổi khi state đổi*) và `DispatchContext` (chứa hàm `dispatch` để gửi lệnh — **luôn ổn định**, không đổi reference → không gây re-render). Component nào chỉ cần *gửi lệnh* (không cần đọc state) thì lấy từ `DispatchContext` → **không bị re-render** khi state đổi. Đây là pattern được tài liệu React chính thức khuyến nghị.

> [!tip] 🎯 Hiểu trong 30 giây
> Khi state toàn cục **phức tạp** (giỏ hàng, form nhiều bước), thay `useState` bằng **`useReducer`**: bạn không tự sửa state lung tung mà **"gửi lệnh"** (`dispatch({ type: 'ADD', ... })`), một hàm `reducer` nhận lệnh và *tính ra state mới* — giống như **gửi yêu cầu cho quầy lễ tân thay vì tự vào kho lấy đồ**. Mọi cách thay đổi state gom về một chỗ (reducer) nên dễ kiểm soát và test.
>
> **Mẹo tối ưu hay được hỏi:** tách **2 context** — một cho *state* (đọc), một cho *dispatch* (gửi lệnh). Vì `dispatch` không bao giờ đổi, component nào chỉ bấm nút gửi lệnh (vd nút "Thêm") thì **không bị vẽ lại** mỗi khi state đổi → đỡ re-render thừa.

---

## 1. Khái niệm

### Tại sao useReducer thay useState?

```text
useState phù hợp khi:
  - Ít trạng thái, đơn giản
  - Ít action types
  - State transitions độc lập

useReducer phù hợp khi:
  - State object phức tạp (nhiều fields liên quan)
  - Nhiều action types (ADD, REMOVE, UPDATE, RESET, ...)
  - Next state phụ thuộc vào action type VÀ current state
  - Cần test reducer logic riêng
```

### Tách StateContext và DispatchContext

```text
Vấn đề với 1 context: value={{ state, dispatch }}
  → Khi state thay đổi → object mới → TẤT CẢ consumers re-render
  → Kể cả components chỉ dispatch (AddButton, DeleteButton)

Giải pháp: 2 contexts riêng
  StateContext.Provider value={state}       → thay đổi khi state đổi
  DispatchContext.Provider value={dispatch} → STABLE, không bao giờ thay đổi

  AddButton dùng useDispatch() → chỉ subscribe DispatchContext
           → không re-render khi state thay đổi
```

```
★ Insight ─────────────────────────────────────
• Mẹo tách 2 context dựa trên một sự thật: `dispatch` từ useReducer là reference
  ỔN ĐỊNH suốt đời component. Vì vậy component chỉ-gửi-lệnh (AddButton) subscribe
  DispatchContext sẽ KHÔNG re-render khi state đổi — chỉ component đọc state mới
  re-render. Đây là tối ưu re-render chính thức React docs khuyên, "mini Redux".
• Reducer = HÀM THUẦN `(state, action) => newState` — đúng họ với Array.reduce.
  Vì thuần (không async, không mutate, deterministic) nên TEST ĐƯỢC RIÊNG, không
  cần mount React. Đây là lý do chọn useReducer cho logic phức tạp: tách "luật
  cập nhật state" khỏi UI. Async vẫn nằm ở component rồi dispatch kết quả vào.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp / API

### 2.1 Pattern đầy đủ với 2 Contexts

```jsx
// contexts/TasksContext.jsx
import { createContext, useContext, useReducer } from 'react';

// 1. Tạo 2 contexts riêng biệt
const TasksStateContext    = createContext(null);
const TasksDispatchContext = createContext(null);

// 2. Reducer function — pure function, dễ test riêng
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'ADD_TASK':
      return [...tasks, {
        id: Date.now(),
        text: action.text,
        done: false,
      }];

    case 'TOGGLE_TASK':
      return tasks.map(task =>
        task.id === action.id ? { ...task, done: !task.done } : task
      );

    case 'UPDATE_TASK':
      return tasks.map(task =>
        task.id === action.id ? { ...task, ...action.updates } : task
      );

    case 'DELETE_TASK':
      return tasks.filter(task => task.id !== action.id);

    case 'CLEAR_DONE':
      return tasks.filter(task => !task.done);

    default:
      throw new Error(`Unknown action type: ${action.type}`);
  }
}

// 3. Provider — bọc cả 2 context
export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, []);

  return (
    <TasksDispatchContext.Provider value={dispatch}>
      <TasksStateContext.Provider value={tasks}>
        {children}
      </TasksStateContext.Provider>
    </TasksDispatchContext.Provider>
  );
}

// 4. Custom hooks — export hooks, không export contexts trực tiếp
export function useTasks() {
  const ctx = useContext(TasksStateContext);
  if (ctx === null) throw new Error('useTasks must be within TasksProvider');
  return ctx;
}

export function useTasksDispatch() {
  const ctx = useContext(TasksDispatchContext);
  if (ctx === null) throw new Error('useTasksDispatch must be within TasksProvider');
  return ctx;
}
```

### 2.2 Sử dụng trong Components

```jsx
// App.jsx
import { TasksProvider } from './contexts/TasksContext';

function App() {
  return (
    <TasksProvider>
      <AddTaskForm />    {/* chỉ cần dispatch */}
      <TaskList />       {/* cần cả state lẫn dispatch */}
      <TaskSummary />    {/* chỉ cần state */}
    </TasksProvider>
  );
}

// AddTaskForm.jsx — CHỈ dùng dispatch, không subscribe state
// → Không re-render khi tasks array thay đổi!
import { useTasksDispatch } from './contexts/TasksContext';

function AddTaskForm() {
  const dispatch = useTasksDispatch();
  const [text, setText] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!text.trim()) return;
    dispatch({ type: 'ADD_TASK', text: text.trim() });
    setText('');
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={text} onChange={e => setText(e.target.value)} placeholder="Add task..." />
      <button type="submit">Add</button>
    </form>
  );
}

// TaskList.jsx — dùng cả hai
import { useTasks, useTasksDispatch } from './contexts/TasksContext';

function TaskList() {
  const tasks    = useTasks();
  const dispatch = useTasksDispatch();

  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <input
            type="checkbox"
            checked={task.done}
            onChange={() => dispatch({ type: 'TOGGLE_TASK', id: task.id })}
          />
          <span style={{ textDecoration: task.done ? 'line-through' : 'none' }}>
            {task.text}
          </span>
          <button onClick={() => dispatch({ type: 'DELETE_TASK', id: task.id })}>
            ✕
          </button>
        </li>
      ))}
    </ul>
  );
}

// TaskSummary.jsx — chỉ đọc state
import { useTasks } from './contexts/TasksContext';

function TaskSummary() {
  const tasks    = useTasks();
  const done     = tasks.filter(t => t.done).length;
  const total    = tasks.length;

  return (
    <p>{done}/{total} tasks completed</p>
  );
}
```

### 2.3 Action Types với TypeScript-style constants

```jsx
// Định nghĩa action types như constants để avoid typos
const TASK_ACTIONS = {
  ADD:        'ADD_TASK',
  TOGGLE:     'TOGGLE_TASK',
  UPDATE:     'UPDATE_TASK',
  DELETE:     'DELETE_TASK',
  CLEAR_DONE: 'CLEAR_DONE',
};

// Action creators — functions tạo action objects
const createAddAction    = (text)    => ({ type: TASK_ACTIONS.ADD, text });
const createToggleAction = (id)      => ({ type: TASK_ACTIONS.TOGGLE, id });
const createDeleteAction = (id)      => ({ type: TASK_ACTIONS.DELETE, id });
const createUpdateAction = (id, updates) => ({ type: TASK_ACTIONS.UPDATE, id, updates });

// Sử dụng với dispatch
dispatch(createAddAction('Learn React'));
dispatch(createToggleAction(taskId));
```

### 2.4 Kết hợp với Async Actions

```jsx
// useReducer không support async natively
// Async logic sống trong components, sau đó dispatch kết quả

function TaskProvider({ children }) {
  const [state, dispatch] = useReducer(reducer, {
    tasks:   [],
    loading: false,
    error:   null,
  });

  return (
    <DispatchContext.Provider value={dispatch}>
      <StateContext.Provider value={state}>
        {children}
      </StateContext.Provider>
    </DispatchContext.Provider>
  );
}

// Component xử lý async, dispatch khi xong
function LoadTasksButton() {
  const dispatch = useDispatch();

  const loadTasks = async () => {
    dispatch({ type: 'FETCH_START' });
    try {
      const res   = await fetch('/api/tasks');
      const tasks = await res.json();
      dispatch({ type: 'FETCH_SUCCESS', tasks });
    } catch (err) {
      dispatch({ type: 'FETCH_ERROR', error: err.message });
    }
  };

  return <button onClick={loadTasks}>Load Tasks</button>;
}
```

---

## 3. Ví dụ minh họa

### Ví dụ: Shopping Cart với Context + useReducer

```jsx
// contexts/CartContext.jsx
import { createContext, useContext, useReducer, useMemo } from 'react';

const CartStateContext    = createContext(null);
const CartDispatchContext = createContext(null);

function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existing = state.items.find(i => i.id === action.item.id);
      const items = existing
        ? state.items.map(i => i.id === action.item.id ? { ...i, qty: i.qty + 1 } : i)
        : [...state.items, { ...action.item, qty: 1 }];
      return { ...state, items };
    }
    case 'REMOVE_ITEM':
      return { ...state, items: state.items.filter(i => i.id !== action.id) };
    case 'UPDATE_QTY': {
      if (action.qty <= 0) {
        return { ...state, items: state.items.filter(i => i.id !== action.id) };
      }
      return {
        ...state,
        items: state.items.map(i => i.id === action.id ? { ...i, qty: action.qty } : i),
      };
    }
    case 'CLEAR':
      return { ...state, items: [] };
    default:
      return state;
  }
}

const initialState = { items: [] };

export function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, initialState);

  // Derived state
  const cartState = useMemo(() => ({
    ...state,
    totalItems: state.items.reduce((s, i) => s + i.qty, 0),
    totalPrice: state.items.reduce((s, i) => s + i.price * i.qty, 0),
  }), [state]);

  return (
    <CartDispatchContext.Provider value={dispatch}>
      <CartStateContext.Provider value={cartState}>
        {children}
      </CartStateContext.Provider>
    </CartDispatchContext.Provider>
  );
}

export const useCart = () => {
  const ctx = useContext(CartStateContext);
  if (!ctx) throw new Error('useCart must be within CartProvider');
  return ctx;
};

export const useCartDispatch = () => {
  const ctx = useContext(CartDispatchContext);
  if (!ctx) throw new Error('useCartDispatch must be within CartProvider');
  return ctx;
};

// Action creators exported cho convenience
export const cartActions = {
  addItem:    (item)       => ({ type: 'ADD_ITEM', item }),
  removeItem: (id)         => ({ type: 'REMOVE_ITEM', id }),
  updateQty:  (id, qty)    => ({ type: 'UPDATE_QTY', id, qty }),
  clear:      ()           => ({ type: 'CLEAR' }),
};
```

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Pitfall 1: Throw Error trong reducer khi action không hợp lệ
> ```jsx
> default: throw new Error(`Unknown action: ${action.type}`)
> ```
> Tốt hơn là silently return state — phát hiện typo sớm trong development. Nếu dùng TypeScript, union type action cho phép compile-time check.

> [!warning] Pitfall 2: Mutate state trong reducer
> Reducer phải là **pure function** — không mutate state. Giống như useState, phải tạo new reference:
> ```jsx
> // SAI
> state.items.push(newItem); return state;
> // ĐÚNG
> return { ...state, items: [...state.items, newItem] };
> ```
> Immer library giúp viết "mutating" code nhưng thực chất tạo immutable updates.

> [!tip] Reducer dễ test độc lập
> `tasksReducer(initialState, { type: 'ADD_TASK', text: 'Learn' })` → test không cần React, không cần mount component. Đây là lý do chính nên dùng useReducer cho business logic phức tạp.

---

## 5. Câu hỏi phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Vì sao tách StateContext và DispatchContext?"
> *"Vì để tránh re-render thừa. Hàm dispatch của useReducer là một tham chiếu ổn định, không bao giờ đổi qua các render. Nếu em gộp chung value là object chứa cả state lẫn dispatch thì mỗi render object đó là mới, làm mọi component đọc context đều re-render. Khi tách làm hai context, component nào chỉ cần gửi lệnh, ví dụ nút Thêm hay Xóa, thì lấy từ DispatchContext và sẽ không re-render khi state đổi vì dispatch không đổi; chỉ những component thực sự đọc state mới re-render khi state thay đổi. Đây là pattern được tài liệu React chính thức khuyến nghị cho state toàn cục phức tạp."*

> [!note] 🧠 Mẹo nhớ
> **useReducer = gửi lệnh (`dispatch`) cho reducer tính state mới**, gom mọi thay đổi một chỗ. **Tách StateContext (đọc) và DispatchContext (gửi)** → component chỉ gửi lệnh thì không re-render thừa.

**Q1: Tại sao tách StateContext và DispatchContext riêng?**

> `dispatch` function từ `useReducer` là **stable reference** — không bao giờ thay đổi. Nếu để chung `value={{ state, dispatch }}` → object literal tạo mới mỗi render → mọi consumers re-render. Tách ra: components chỉ cần dispatch (AddButton, DeleteButton) subscribe `DispatchContext` — chúng **không re-render** khi state thay đổi. Chỉ components dùng state mới re-render khi state đổi. Pattern này từ React docs chính thức.

**Q2: useReducer + Context có phải là alternative cho Redux không?**

> Có thể thay thế Redux cho apps **không quá phức tạp**. Giống nhau: reducer, actions, dispatch. Redux có thêm: DevTools (time-travel debugging), middleware (redux-thunk/saga cho async), selectors tối ưu (reselect). useReducer + Context phù hợp: app medium, team không quen Redux, muốn giữ dependencies ít. Dùng Redux/Zustand khi: app lớn, nhiều developers, cần DevTools mạnh, performance critical.

**Q3: Reducer trong React và reducer trong functional programming?**

> Cùng concept: `(currentState, action) → newState`. Từ Array.prototype.reduce: `(accumulator, item) => nextAccumulator`. Reducer trong React: (1) phải **pure** — không side effects, không mutate, không async; (2) **deterministic** — cùng input luôn cho cùng output; (3) **exhaustive** — handle tất cả action types (dùng `default: throw` để catch unknown actions). Chính vì pure, reducer dễ test và debug.

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Viết tests cho `cartReducer` (không cần React). Test: ADD_ITEM tăng qty nếu item đã tồn tại; REMOVE_ITEM xóa đúng item; CLEAR trả về empty items; UPDATE_QTY(id, 0) xóa item.

- [ ] **Bài 2:** Tạo `UndoableContext` — state có thể undo/redo. Reducer dùng `past: []`, `present`, `future: []`. Actions: `DO(action)`, `UNDO`, `REDO`. `DO` push present vào past, clear future.

---

## 7. Liên kết

- [[11-Context-API]] — Context API concepts
- [[12-Context-voi-useState]] — Phiên bản đơn giản hơn với useState
- [[03-State-voi-useState]] — useState vs useReducer comparison
