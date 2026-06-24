---
title: "React: Tối ưu re-render — memo, useMemo, useCallback"
section: 06-React
tags: [react, performance, memo, useMemo, useCallback, re-render, fresher, frontend]
related:
  - "[[03-State-voi-useState]]"
  - "[[08-useEffect-Hook]]"
  - "[[02-Component-va-Props]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 35m
source: [react.dev, React.docx]
---

# React: Tối ưu re-render — `React.memo`, `useMemo`, `useCallback`

> [!summary] TL;DR
> Mặc định, khi component **cha re-render** thì **mọi component con cũng re-render** (dù props không đổi). 3 công cụ ghi nhớ (memoization) để cắt re-render thừa: **`React.memo(Component)`** bọc *component* — bỏ qua re-render nếu props không đổi (so sánh nông). **`useMemo(fn, deps)`** ghi nhớ *giá trị* tính toán nặng giữa các render. **`useCallback(fn, deps)`** ghi nhớ *function* để giữ nguyên reference (đặc biệt khi truyền callback xuống con đã `memo`). Tất cả so sánh props/deps bằng **reference** → vì sao `useMemo`/`useCallback` thường đi kèm `React.memo`. **Đừng lạm dụng** — bản thân việc so sánh cũng tốn chi phí; React 19 Compiler đang dần tự lo việc này.

> [!tip] 🎯 Hiểu trong 30 giây
> Mặc định React khá "chăm chỉ thái quá": **cha vẽ lại thì con cũng vẽ lại theo**, kể cả khi dữ liệu của con y nguyên. Đa số trường hợp điều này *rẻ* và không sao. Nhưng khi con render nặng, ta muốn nói: *"props không đổi thì khỏi vẽ lại"*.
>
> 3 công cụ, nhớ theo "ghi nhớ cái gì":
> - **`React.memo`** → ghi nhớ **cả COMPONENT**: cha re-render mà props của con *không đổi* thì con **bỏ qua** lần render đó. (Ví von: *"đề bài y hệt lần trước thì nộp lại bài cũ, khỏi làm lại"*.)
> - **`useMemo`** → ghi nhớ **một GIÁ TRỊ** tính toán tốn kém (lọc/sắp xếp mảng lớn): chỉ tính lại khi nguyên liệu (`deps`) đổi.
> - **`useCallback`** → ghi nhớ **một FUNCTION** để nó *giữ nguyên địa chỉ* qua các render. (Thực chất là `useMemo` chuyên cho function.)
>
> **Mắt xích quan trọng (rất hay hỏi):** React so sánh props bằng **địa chỉ (reference)**. Mỗi render, object/array/function viết thẳng trong component là **vật mới khác địa chỉ** → con đã `memo` vẫn tưởng "props đổi" và vẫn re-render. Vì vậy khi truyền callback/object xuống con `memo`, phải bọc `useCallback`/`useMemo` để giữ địa chỉ → `memo` mới phát huy.

---

## 1. Khái niệm

### Vì sao cần? — Re-render lan từ cha xuống con

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>+{count}</button>
      {/* ExpensiveChild re-render MỖI lần count đổi, dù props của nó không đổi */}
      <ExpensiveChild data={someStaticData} />
    </div>
  );
}
```

Mỗi lần bấm nút, `Parent` re-render → `ExpensiveChild` cũng re-render dù `data` không đổi. Nếu `ExpensiveChild` render nặng → lãng phí.

### So sánh 3 công cụ

| Công cụ | Ghi nhớ cái gì | Trả về | Dùng khi |
|---|---|---|---|
| **`React.memo(Comp)`** | Cả **component** | Component mới (memoized) | Con render nặng + cha re-render thường xuyên + props ít đổi |
| **`useMemo(fn, deps)`** | **Giá trị** trả về của `fn` | Giá trị đã cache | Tính toán nặng (filter/sort/map mảng lớn), hoặc tạo object/array ổn định để làm deps/props |
| **`useCallback(fn, deps)`** | Chính **function** | Function đã cache | Truyền callback xuống con đã `memo`, hoặc callback làm dependency của `useEffect` |

> `useCallback(fn, deps)` ≡ `useMemo(() => fn, deps)`. Một cái nhớ *function*, một cái nhớ *giá trị*.

```
★ Insight ─────────────────────────────────────
• Cả 3 đều dựa trên một sự thật: React so sánh bằng THAM CHIẾU (Object.is). Object/
  array/function tạo trong thân component là vật MỚI mỗi render → "khác" theo so
  sánh nông. React.memo so sánh nông props; useMemo/useCallback giữ reference ổn
  định để phép so sánh đó trả về "giống". Thiếu một mắt xích (memo con NHƯNG không
  useCallback callback) → tối ưu vô hiệu.
• "Tối ưu" không miễn phí: memo phải lưu props cũ + so sánh mỗi render; useMemo/
  useCallback lưu cache + so deps. Với component nhẹ, chi phí này có thể LỚN HƠN
  cái nó tiết kiệm. Quy tắc: đo trước (Profiler), tối ưu sau — đừng bọc bừa. React
  19 Compiler tự chèn các tối ưu này, giảm dần nhu cầu làm tay.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp / API

### 2.1 `React.memo` — bỏ qua re-render khi props không đổi

```jsx
import { memo } from 'react';

const ExpensiveChild = memo(function ExpensiveChild({ data }) {
  console.log('ExpensiveChild render');
  return <div>{/* render nặng từ data */}</div>;
});

// Bây giờ: cha re-render mà `data` (cùng reference) không đổi → con KHÔNG re-render
```

`React.memo` so sánh **nông** (shallow) từng prop. Muốn so sánh tùy biến → truyền hàm thứ 2:

```jsx
const Row = memo(
  function Row({ user }) { return <li>{user.name}</li>; },
  (prevProps, nextProps) => prevProps.user.id === nextProps.user.id // true = bỏ qua re-render
);
```

### 2.2 `useMemo` — ghi nhớ giá trị tính toán

```jsx
function ProductList({ products, query }) {
  // KHÔNG memo: filter chạy lại MỖI render (kể cả khi chỉ state khác đổi)
  // const filtered = products.filter(p => p.name.includes(query));

  // CÓ memo: chỉ filter lại khi products hoặc query đổi
  const filtered = useMemo(
    () => products.filter(p => p.name.includes(query)),
    [products, query]
  );

  return <ul>{filtered.map(p => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

### 2.3 `useCallback` — giữ ổn định reference của function

```jsx
import { useCallback, memo } from 'react';

const Child = memo(function Child({ onAction }) {
  console.log('Child render');
  return <button onClick={onAction}>Action</button>;
});

function Parent() {
  const [count, setCount] = useState(0);

  // ❌ Không useCallback: handleAction là FUNCTION MỚI mỗi render
  //    → Child (dù memo) vẫn re-render vì prop onAction "đổi"
  // const handleAction = () => console.log('action');

  // ✅ useCallback: giữ nguyên reference giữa các render (deps rỗng)
  const handleAction = useCallback(() => console.log('action'), []);

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>+{count}</button>
      <Child onAction={handleAction} />
    </>
  );
}
```

---

## 3. Ví dụ minh họa

### Bộ ba kết hợp đúng cách

```jsx
const ExpensiveTable = memo(function ExpensiveTable({ rows, onRowClick }) {
  console.log('Table render'); // chỉ in khi rows/onRowClick thực sự đổi
  return (
    <table><tbody>
      {rows.map(r => (
        <tr key={r.id} onClick={() => onRowClick(r.id)}><td>{r.name}</td></tr>
      ))}
    </tbody></table>
  );
});

function Dashboard({ allRows }) {
  const [count, setCount] = useState(0);
  const [filter, setFilter] = useState('');

  // useMemo: chỉ lọc lại khi allRows/filter đổi (không phải khi count đổi)
  const rows = useMemo(
    () => allRows.filter(r => r.name.includes(filter)),
    [allRows, filter]
  );

  // useCallback: giữ reference ổn định để ExpensiveTable (memo) không re-render thừa
  const handleRowClick = useCallback((id) => console.log('clicked', id), []);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count {count}</button>
      <input value={filter} onChange={e => setFilter(e.target.value)} />
      {/* Bấm Count → Dashboard re-render, nhưng rows & handleRowClick giữ nguyên
          → ExpensiveTable KHÔNG re-render */}
      <ExpensiveTable rows={rows} onRowClick={handleRowClick} />
    </div>
  );
}
```

---

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Pitfall 1: `React.memo` con nhưng truyền props "luôn mới" → vô hiệu
> Truyền `style={{...}}`, `data={[...]}`, hay `onClick={() => ...}` viết thẳng inline → mỗi render là reference mới → `memo` so sánh thấy "khác" → con vẫn re-render. Phải `useMemo`/`useCallback` cho các props đó.

> [!warning] Pitfall 2: Lạm dụng — tối ưu non-nớt (premature optimization)
> Bọc `useMemo`/`useCallback`/`memo` ở mọi nơi làm code rối và **chậm hơn** với component nhẹ (chi phí so sánh + bộ nhớ cache > lợi ích). Chỉ tối ưu khi Profiler chỉ ra điểm nóng thật.

> [!warning] Pitfall 3: Thiếu/sai deps → giá trị cũ (stale)
> `useMemo`/`useCallback` với deps thiếu → cache giá trị/closure cũ, không cập nhật khi biến liên quan đổi. Tuân thủ ESLint `exhaustive-deps` như với `useEffect` ([[08-useEffect-Hook]]).

> [!tip] React 19 Compiler — tự động memoize
> React Compiler (React 19) phân tích code và tự chèn memoization tại compile time → giảm nhu cầu dùng `useMemo`/`useCallback` thủ công. Vẫn nên *hiểu* cơ chế để debug và làm việc với codebase cũ.

---

## 5. Câu hỏi phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "`React.memo` và `useMemo` khác nhau gì, vai trò tối ưu?"
> *"`React.memo` bọc một component: nó ghi nhớ kết quả render và bỏ qua việc render lại nếu props không đổi, so sánh nông. Còn `useMemo` là hook ghi nhớ một giá trị tính toán bên trong component, chỉ tính lại khi dependency đổi, thường dùng cho phép lọc hay sắp xếp mảng lớn. Nói ngắn gọn, `React.memo` ngăn re-render cả component con, `useMemo` ngăn tính lại một giá trị tốn kém. Hai cái hay đi cùng nhau, vì React so sánh props bằng tham chiếu, nên khi truyền object hoặc function xuống một con đã memo, em phải dùng `useMemo` hoặc `useCallback` để giữ tham chiếu ổn định, nếu không memo sẽ vô hiệu. Em chỉ dùng khi đo thấy có vấn đề hiệu năng thật, vì lạm dụng còn làm chậm hơn."*

> [!example] 🗣️ Trả lời mẫu — "Lạm dụng useMemo/useCallback ở mọi nơi thì tốt hơn hay tệ đi?"
> *"Tệ đi trong đa số trường hợp. Mỗi useMemo/useCallback đều có chi phí: React phải lưu cache và so sánh dependency mỗi lần render, còn React.memo phải lưu và so sánh props. Với component nhẹ, chi phí này thường lớn hơn cái nó tiết kiệm, lại làm code rối và khó đọc. Re-render trong React đa phần là rẻ, nên em chỉ tối ưu khi React Profiler chỉ ra điểm nóng cụ thể: component render nặng, render nhiều lần không cần thiết. Đó là nguyên tắc tránh tối ưu non-nớt. Thêm nữa React 19 có Compiler tự chèn memoization nên nhu cầu làm tay đang giảm."*

> [!example] 🗣️ Trả lời mẫu — "Làm sao ngăn component con re-render khi props không đổi?"
> *"Em bọc component con bằng `React.memo`. Khi đó nếu cha re-render mà props của con không đổi thì con được bỏ qua. Nhưng phải đảm bảo props thực sự ổn định về tham chiếu: nếu em truyền callback hay object tạo mới mỗi render thì memo vô hiệu, nên callback em bọc `useCallback`, còn giá trị object/array tính toán em bọc `useMemo`. Bộ ba memo + useCallback + useMemo phối hợp mới cho hiệu quả."*

> [!note] 🧠 Mẹo nhớ
> **memo = nhớ cả COMPONENT · useMemo = nhớ GIÁ TRỊ · useCallback = nhớ FUNCTION.** Tất cả so sánh bằng **địa chỉ** → memo con thì callback/object phải `useCallback`/`useMemo`. **Đo trước, tối ưu sau** — đừng bọc bừa.

**Q1: `useMemo` vs `useCallback` khác nhau gì?**

> `useMemo(fn, deps)` ghi nhớ **giá trị trả về** của `fn`. `useCallback(fn, deps)` ghi nhớ **chính function** `fn`. Thực chất `useCallback(fn, deps)` ≡ `useMemo(() => fn, deps)`. Dùng `useMemo` cho tính toán nặng/giá trị ổn định; `useCallback` cho function truyền xuống con đã `memo` hoặc làm deps của `useEffect`.

**Q2: Vì sao `React.memo` đôi khi "không có tác dụng"?**

> Vì props vẫn "đổi" theo so sánh nông: object/array/function inline tạo reference mới mỗi render. Giải pháp: bọc các props đó bằng `useMemo`/`useCallback` để giữ reference ổn định. `React.memo` chỉ hữu ích khi props thực sự ổn định.

**Q3: Khi nào KHÔNG nên dùng memoization?**

> Khi component nhẹ và re-render đã rẻ — chi phí so sánh/cache có thể lớn hơn lợi ích. Khi deps thay đổi gần như mỗi render (memo vô ích). Nguyên tắc: profile trước (React DevTools Profiler), tối ưu điểm nóng thật sự, không bọc đại trà.

---

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Tạo `Parent` có `count` (state) và con `List` render 1000 item. Bọc `List` bằng `React.memo`, truyền props `items` ổn định bằng `useMemo`. Mở Profiler, bấm tăng `count` và xác nhận `List` không re-render.

- [ ] **Bài 2:** Tạo con `Button` đã `memo` nhận `onClick`. Chứng minh nếu truyền `onClick={() => ...}` inline thì `Button` vẫn re-render; sửa bằng `useCallback` để nó ngừng re-render.

---

## 7. Liên kết

- [[03-State-voi-useState]] — re-render bắt nguồn từ state/props đổi; so sánh reference
- [[08-useEffect-Hook]] — `useCallback` thường dùng để ổn định deps của effect
- [[02-Component-va-Props]] — props read-only, one-way flow; cha re-render → con re-render
- [[01-React-Overview]] — Virtual DOM diffing và React 19 Compiler
