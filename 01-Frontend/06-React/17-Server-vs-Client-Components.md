---
title: "Server vs Client Components (RSC)"
section: 06-React
tags: [react, nextjs, rsc, server-component, client-component, use-client, hydration, fresher, frontend]
related:
  - "[[16-NextJS-App-Router]]"
  - "[[18-Server-Actions-Data-Fetching]]"
  - "[[08-useEffect-Hook]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 40m
source: [React raw transcript, Next.js docs]
---

# Server vs Client Components (RSC)

> [!summary] TL;DR
> Trong Next.js App Router, **mặc định mọi component là Server Component** — render (và có thể cache) **trên server**, gửi HTML về trước, JavaScript nạp nền sau → nhanh, an toàn, SEO tốt, fetch data ngay tại chỗ. Nhưng Server Component **không** dùng được tương tác (sự kiện click, state) hay **browser API** (localStorage, geolocation), cũng không dùng hook như `useState`/`useEffect`. Khi cần những thứ đó, đánh dấu file bằng directive **`"use client"`** ở đầu → thành **Client Component** (render/hydrate ở trình duyệt). Mỗi page/component cần **`export default`**.

> [!tip] 🎯 Hiểu trong 30 giây
> Trong Next.js mới, component được chia 2 loại theo *nơi nó chạy*:
> - **Server Component (mặc định)** = chạy **trên máy chủ**, dựng sẵn HTML gửi về → trang hiện nhanh, tốt SEO, lấy dữ liệu ngay tại chỗ, và **giấu được code/khóa bí mật** vì không gửi xuống trình duyệt. Đổi lại: **không** có tương tác (không `onClick`), **không** dùng `useState`/`useEffect`, **không** đụng được thứ của trình duyệt (`localStorage`, `window`).
> - **Client Component** = chạy **ở trình duyệt**, có đủ tương tác/state/hook. Báo cho Next biết bằng cách viết dòng **`"use client"`** ở *đầu file*.
>
> **Hydration** (làm "sống dậy"): server gửi HTML *tĩnh* để hiện ngay, sau đó React *gắn JavaScript vào* để nút bấm, state... bắt đầu hoạt động.
>
> **Quy tắc vàng:** để mặc định là server, **chỉ dán `"use client"` ở những mảnh nhỏ thật sự cần tương tác** — đừng dán ở gốc app (sẽ biến cả cây thành client, mất hết lợi ích).

---

## 1. Mặc định: Server Component

Trong App Router, component bạn tạo **mặc định chạy trên server**:

```jsx
// app/mountain/page.js — đây là SERVER component (mặc định)
export default async function Page() {        // có thể async để fetch data
  const data = await getData();
  return <main>{/* render từ dữ liệu server */}</main>;
}
```

Lợi ích server component:
- **Nhanh**: trả HTML trước, JS nạp nền (chunk hóa).
- **Bảo mật**: code/secret/data-fetch nằm ở server, không lộ ra client.
- **Caching** & tối ưu tự động, không cần cấu hình.
- **Fetch data ngay tại component** (co-location) — xem [[18-Server-Actions-Data-Fetching]].

> **Hydration**: server gửi HTML "tĩnh" trước để hiện ngay; sau đó React "gắn" JavaScript vào để phần tương tác sống dậy.

---

## 2. Khi nào cần Client Component

Server component **không làm được** những việc cần trình duyệt. Khi đó dùng `"use client"`:

```jsx
"use client";                 // ← directive ở DÒNG ĐẦU file
import { useState } from "react";

export default function Counter() {
  const [n, setN] = useState(0);          // hook cần client
  return <button onClick={() => setN(n + 1)}>{n}</button>;  // sự kiện cần client
}
```

| Cần Client Component khi... | Ví dụ |
|---|---|
| **Tương tác** (event handler) | `onClick`, `onChange` |
| **State / hook** | `useState`, `useEffect`, `useReducer` |
| **Browser API** | `localStorage`, `geolocation`, `window` |
| **Truy cập file/tài nguyên local phía client** | `next/image` loader tùy biến... |

---

## 3. Server vs Client — bảng phân biệt

| Tiêu chí | **Server Component** (mặc định) | **Client Component** (`"use client"`) |
|---|---|---|
| Render ở | Server | Trình duyệt (sau hydrate) |
| Đánh dấu | (không cần gì) | `"use client"` đầu file |
| Dùng `useState`/`useEffect` | ❌ | ✅ |
| Event handler (onClick...) | ❌ | ✅ |
| Browser API | ❌ | ✅ |
| Fetch data trực tiếp (async) | ✅ (khuyến khích) | hạn chế (dùng effect) |
| Lộ code ra client | Không | Có (gửi JS xuống) |
| Tốt cho | Nội dung, data, SEO | Tương tác, state |

```
★ Insight ─────────────────────────────────────
• Quy tắc thực dụng: ĐỂ MẶC ĐỊNH là server, chỉ "use client" ở những lá cây thật
  sự cần tương tác. Đừng gắn "use client" lên trên cùng app — làm vậy biến cả cây
  thành client, mất hết lợi ích server (nhanh, bảo mật, SEO). Đẩy "use client"
  xuống càng sâu (component nhỏ) càng tốt.
• Lỗi kinh điển: "functions cannot be passed to client components" hoặc dùng
  useState trong server component. Nghĩa là bạn đang dùng tính năng client trong
  một file server → thêm "use client" hoặc tách phần tương tác ra component riêng.
─────────────────────────────────────────────────
```

---

## 4. Kết hợp: server bọc client

Mẫu phổ biến: server component fetch data → truyền **props** xuống client component để tương tác.

```jsx
// page.js (SERVER): fetch rồi truyền xuống
export default async function Page() {
  const hotels = await getHotels();
  return hotels.map(h => <HotelBlock key={h.id} name={h.name} />);  // HotelBlock là client
}
```
```jsx
// HotelBlock.js (CLIENT): có tương tác/ảnh local
"use client";
import Image from "next/image";
export default function HotelBlock({ name }) { ... }
```

> Server lo **dữ liệu**, client lo **tương tác** — ranh giới rõ ràng, hiệu năng tốt.

---

## 5. Q&A phỏng vấn

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Server vs Client Component khác gì, khi nào dùng `use client`?"
> *"Trong Next.js App Router, mặc định mọi component là Server Component, tức render trên server và gửi HTML về trước, nên nhanh, tốt SEO, bảo mật vì code và thao tác lấy dữ liệu nằm ở server không lộ ra client, và fetch data được ngay tại component. Nhược điểm là server component không có tương tác, không dùng được useState, useEffect hay browser API như localStorage. Khi cần những thứ đó em thêm directive use client ở dòng đầu file để biến nó thành Client Component, chạy ở trình duyệt. Nguyên tắc của em là giữ mặc định server và chỉ đánh dấu use client ở những component lá thật sự cần tương tác, không đặt ở gốc, vì đặt ở gốc sẽ biến cả cây thành client và mất hết lợi ích của server."*

> [!note] 🧠 Mẹo nhớ
> **Mặc định = Server (nhanh/SEO/bảo mật/fetch tại chỗ, nhưng KHÔNG state/event/browser API).** Cần tương tác → **`"use client"`** ở đầu file. **Đẩy `"use client"` xuống lá**, đừng đặt ở gốc.

> [!question] 1. Trong Next.js App Router, component mặc định là gì?
> **Server Component** — render (và có thể cache) trên server, gửi HTML trước, JS nạp nền. Nhanh, bảo mật, SEO tốt, fetch data ngay tại chỗ.

> [!question] 2. Khi nào phải dùng Client Component? Đánh dấu thế nào?
> Khi cần **tương tác** (event), **state/hook** (`useState`, `useEffect`), hoặc **browser API** (localStorage...). Đánh dấu bằng directive **`"use client"`** ở dòng đầu file.

> [!question] 3. Server Component không làm được gì?
> Không dùng `useState`/`useEffect`/hook, không event handler, không browser API. Những việc đó phải ở client component.

> [!question] 4. Hydration là gì?
> Server gửi HTML tĩnh để hiển thị ngay; sau đó React "gắn" JavaScript vào để phần tương tác hoạt động. Cho first paint nhanh + tương tác đầy đủ.

> [!question] 5. Vì sao không nên đặt `"use client"` ở component gốc?
> Vì nó biến **cả cây** thành client → mất lợi ích server (nhanh, bảo mật, SEO). Nên giữ mặc định server và chỉ `"use client"` ở các component lá cần tương tác.

> [!question] 6. Lỗi "functions cannot be passed to client components" nghĩa là gì?
> Bạn đang dùng tính năng client (truyền hàm/dùng hook/sự kiện) trong file đang là **server component**. Khắc phục: thêm `"use client"` hoặc tách phần tương tác ra một client component riêng.

---

## 6. Bài tập tự luyện

1. Tạo trang server component fetch danh sách (async) và render; xác nhận không dùng được `useState` trong đó.
2. Tạo `Counter` client component (`"use client"` + `useState` + `onClick`), nhúng vào trang server.
3. Mẫu server-bọc-client: server fetch data → truyền props → client component hiển thị + có nút tương tác.
4. Thử dùng `useState` trong server component, đọc lỗi, rồi sửa bằng `"use client"`.

---

## 7. Liên kết
- [[16-NextJS-App-Router]] — App Router, cấu trúc app/
- [[18-Server-Actions-Data-Fetching]] — fetch trong server component, server action
- [[08-useEffect-Hook]] — hook chỉ chạy ở client
- [[00-MOC-React|MOC: React]]
