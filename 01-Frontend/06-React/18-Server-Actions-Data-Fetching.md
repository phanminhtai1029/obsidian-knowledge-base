---
title: "Data Fetching & Server Actions (Next.js)"
section: 06-React
tags: [react, nextjs, data-fetching, server-action, form, mutation, fresher, frontend]
related:
  - "[[17-Server-vs-Client-Components]]"
  - "[[16-NextJS-App-Router]]"
  - "[[06-Form-Handling]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 40m
source: [React raw transcript, Next.js docs]
---

# Data Fetching & Server Actions (Next.js)

> [!summary] TL;DR
> Trong Next.js, **cách tốt nhất để lấy dữ liệu là fetch ngay trong Server Component** (co-location: fetch và render cùng chỗ) — vì fetch ở server nhanh hơn và an toàn hơn fetch ở client. Component để `async`, `await fetch(...)`, rồi render. Để **xử lý form / thay đổi dữ liệu (mutation)**, dùng **Server Action**: một hàm `async` đánh dấu **`"use server"`**, gắn vào form qua thuộc tính **`action={...}`**; khi submit, browser xử lý `formData` và gọi hàm này **trên server** (log/ghi DB hiện ở terminal server, không phải console trình duyệt).

---

## 1. Fetch data trong Server Component

```jsx
// app/mountain/page.js (SERVER component)
async function getData() {
  const res = await fetch("https://snowtooth-api-rest.fly.dev");  // gọi REST API
  return res.json();
}

export default async function Page() {          // component async
  const data = await getData();                 // fetch + render CÙNG chỗ (co-location)
  return (
    <table>
      <tbody>
        {data.map(lift => (
          <tr key={lift.id}><td>{lift.name}</td><td>{lift.status}</td></tr>
        ))}
      </tbody>
    </table>
  );
}
```

| Đặc điểm | Vì sao tốt |
|---|---|
| **Co-location** | fetch & render cùng một file → dễ đọc, dễ bảo trì |
| Chạy ở **server** | nhanh hơn (gần data), giấu API key/secret |
| Không cần `useEffect` | server component fetch trực tiếp bằng `await` |

```
★ Insight ─────────────────────────────────────
• Đây là khác biệt lớn so với React cổ điển: trước đây fetch phải nằm trong
  useEffect (chạy SAU khi render, ở client) → "loading flicker" + lộ request ở
  client. Server component fetch TRƯỚC khi gửi HTML → người dùng thấy data ngay,
  không flicker, không lộ key. useEffect-fetch giờ chỉ dùng cho client component
  khi buộc phải fetch phía trình duyệt.
• Mẹo debug data: stringify rồi nhìn trước cấu trúc (`{JSON.stringify(data)}`) để
  biết key đúng (data.allLifts? data trực tiếp?) trước khi map — tránh lỗi
  "cannot read undefined".
─────────────────────────────────────────────────
```

---

## 2. Truyền data xuống component con (props)

Server component fetch → truyền xuống component con như props (con có thể là client component nếu cần tương tác):

```jsx
export default async function Page() {
  const hotels = await getData();
  return hotels.map(h => <HotelBlock key={h.id} name={h.name} capacity={h.capacity} />);
}

function HotelBlock({ name, capacity }) {       // nhận props (destructure)
  return <div><h2>{name}</h2><p>{capacity}</p></div>;
}
```

> Nhớ: dùng `className` (không phải `class`) trong JSX. Component con đụng tương tác/ảnh local → tách ra file riêng + `"use client"` ([[17-Server-vs-Client-Components]]).

---

## 3. Server Action — xử lý form & mutation

**Server Action** = hàm `async` chạy **trên server**, gọi được từ form để xử lý submit / thay đổi dữ liệu.

```jsx
// app/contact/page.js
export default function ContactPage() {
  async function submitForm(formData) {       // nhận formData
    "use server";                              // ← đánh dấu chạy trên SERVER
    const fields = {
      email: formData.get("email"),
      message: formData.get("message"),
    };
    console.log(fields);                       // hiện ở TERMINAL server, không phải browser
    // TODO: gửi xuống DB / API ở đây
    return fields;
  }

  return (
    <form action={submitForm}>                {/* gắn action = server action */}
      <input id="email" name="email" type="email" required />
      <textarea id="message" name="message" rows={4} required />
      <button type="submit">Send message</button>
    </form>
  );
}
```

| Điểm mấu chốt | Giải thích |
|---|---|
| `"use server"` | đánh dấu hàm là server action (gọi từ client, chạy ở server) |
| `formData.get("name")` | đọc field theo thuộc tính `name` — **dùng form gốc**, không cần state cho từng ô |
| `action={submitForm}` | gắn action vào `<form>`; submit là gọi hàm |
| Log ở đâu? | **Terminal server** (vì chạy server), không phải console trình duyệt |

```
★ Insight ─────────────────────────────────────
• Server Action xóa bỏ rất nhiều boilerplate so với cách cũ: không cần useState
  cho từng input, không cần tự viết handler gọi fetch('/api/...'), không cần tạo
  API route riêng. Browser tự gom formData, Next tự nối client→server. Đây là
  hướng "form chạy được cả khi JS chưa load" (progressive enhancement).
• "use server" (đánh dấu server ACTION) ≠ "use client" (đánh dấu client COMPONENT).
  Dễ lẫn: use client ở ĐẦU FILE để cả component thành client; use server ở TRONG
  HÀM để hàm đó chạy trên server. Hai directive, hai mục đích.
─────────────────────────────────────────────────
```

---

## 4. Q&A phỏng vấn

> [!question] 1. Nơi tốt nhất để fetch data trong Next.js là đâu? Vì sao?
> Trong **Server Component** (co-location: fetch & render cùng chỗ). Vì fetch ở server **nhanh hơn** (gần data), **an toàn hơn** (giấu key/secret), và không cần `useEffect` → không loading flicker.

> [!question] 2. Server Component fetch khác fetch trong `useEffect` thế nào?
> Server component `await fetch` **trước khi** gửi HTML → user thấy data ngay, không flicker, không lộ request. `useEffect` chạy **sau render, ở client** → có flicker và lộ request; chỉ dùng khi buộc fetch phía trình duyệt.

> [!question] 3. Server Action là gì?
> Hàm `async` đánh dấu **`"use server"`**, chạy **trên server**, gọi được từ form (`action={...}`) để xử lý submit / mutation (ghi DB, gọi API) mà không cần tự tạo API route.

> [!question] 4. Server Action đọc dữ liệu form thế nào?
> Qua `formData.get("name")` — dùng form gốc của trình duyệt, không cần `useState` cho từng input. Gắn hàm vào `<form action={fn}>`.

> [!question] 5. `"use server"` khác `"use client"` ra sao?
> `"use client"` đặt **đầu file** → biến component thành **client**. `"use server"` đặt **trong hàm** → đánh dấu hàm là **server action** (chạy ở server). Hai directive khác mục đích.

> [!question] 6. Log trong server action hiện ở đâu?
> Ở **terminal server** (nơi `npm run dev` chạy), không phải console trình duyệt — vì nó chạy phía server.

---

## 5. Bài tập tự luyện

1. Viết server component fetch danh sách từ một REST API và render thành bảng (nhớ `key`).
2. Tách HotelBlock nhận props (`name`, `capacity`) và map data xuống; sửa `class`→`className`.
3. Viết form `/contact` với server action `"use server"` đọc `email`/`message` qua `formData`, log ra terminal.
4. Giải thích vì sao server action giảm boilerplate so với tự tạo API route + `useState` + fetch thủ công.

---

## 6. Liên kết
- [[17-Server-vs-Client-Components]] — server vs client, `"use client"`
- [[16-NextJS-App-Router]] — App Router
- [[06-Form-Handling]] — form trong React cổ điển (controlled component) để so sánh
- [[00-MOC-React|MOC: React]]
