---
title: "Next.js & App Router"
section: 06-React
tags: [react, nextjs, app-router, file-based-routing, layout, link, ssr, fresher, frontend]
related:
  - "[[14-React-Router]]"
  - "[[17-Server-vs-Client-Components]]"
  - "[[01-React-Overview]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 40m
source: [React raw transcript, Next.js docs]
---

# Next.js & App Router

> [!summary] TL;DR
> **Next.js** là framework React phổ biến nhất, **được team React khuyến nghị** — nó "nướng sẵn" nhiều best practice (server rendering, routing, tối ưu) mà không phải tự cấu hình. Tạo dự án: `npx create-next-app@latest`. **App Router** dùng **routing theo file/thư mục**: mỗi **thư mục** trong `app/` là một route, file **`page.js`** bên trong là nội dung hiển thị. **`layout.js`** bọc quanh các page (đặt header/nav dùng chung mọi trang). Điều hướng giữa các route bằng component **`<Link>`** (`next/link`) thay cho thẻ `<a>` thường. Mỗi page **bắt buộc `export default`**.

---

## 1. Next.js là gì, vì sao dùng

| Vấn đề khi dùng React "trần" | Next.js giải quyết |
|---|---|
| Tự dựng routing | **File-based routing** sẵn |
| Tự cấu hình SSR/tối ưu | **Server rendering** mặc định |
| Tự lo cấu trúc/best practice | "Nướng sẵn" theo chuẩn |
| Data fetching rời rạc | Fetch ngay trong server component |

Tạo dự án (wizard hỏi TypeScript, ESLint, Tailwind, `src/`, **App Router**):
```sh
npx create-next-app@latest
npm run dev        # chạy ở localhost:3000
```

> So với **Vite** (chỉ dựng React SPA thuần, nhẹ): Next.js là **framework đầy đủ** (routing, SSR, server components, server actions). Vite hợp app nhỏ/SPA; Next.js hợp app production cần SEO/SSR.

---

## 2. App Router — routing theo file

Mọi file nằm trong thư mục **`app/`** (đôi khi dưới `src/app/`). Quy tắc:

```
app/
├── layout.js          → khung chung (HTML, header) bọc mọi trang
├── page.js            → route "/" (trang chủ)
├── mountain/
│   └── page.js        → route "/mountain"
└── hotels/
    └── page.js        → route "/hotels"
```

| File/thư mục | Vai trò |
|---|---|
| **thư mục** | tạo một **route** (segment URL) |
| **`page.js`** | nội dung hiển thị tại route đó |
| **`layout.js`** | khung bọc dùng chung (header/nav/footer) |

```jsx
// app/mountain/page.js  → hiển thị tại /mountain
export default function Page() {        // BẮT BUỘC export default
  return (
    <main>
      <h1>Lift Status Info</h1>
    </main>
  );
}
```

```
★ Insight ─────────────────────────────────────
• Khác biệt lớn với React Router ([[14-React-Router]]): React Router khai báo route
  bằng CODE (<Route path=... element=.../>); App Router suy ra route từ CẤU TRÚC
  THƯ MỤC. Tạo folder = tạo route, không cần đăng ký. Ít boilerplate, nhưng cấu
  trúc thư mục giờ chính là "bản đồ" ứng dụng.
• Tên file là quy ước có ý nghĩa: page.js (nội dung route), layout.js (khung bọc),
  loading.js (UI khi tải), error.js (UI khi lỗi). Next.js nhận diện các tên này
  tự động — đặt sai tên là route không hoạt động.
─────────────────────────────────────────────────
```

---

## 3. `layout.js` — khung dùng chung

Đặt phần tử xuất hiện trên **mọi trang** (header, nav) một lần, khỏi import lặp:

```jsx
// app/layout.js
import Link from "next/link";

export default function RootLayout({ children }) {   // children = page hiện tại
  return (
    <html><body>
      <header>
        <nav>
          <Link href="/">Snowtooth Mountain</Link>
          <Link href="/mountain">Mountain Info</Link>
          <Link href="/hotels">Hotels</Link>
        </nav>
      </header>
      {children}        {/* nội dung page được "tiêm" vào đây */}
    </body></html>
  );
}
```

> `children` chính là `page.js` của route hiện tại — giống cách ta tiêm React app vào `<body>`.

---

## 4. `<Link>` — điều hướng client-side

```jsx
import Link from "next/link";
<Link href="/hotels">Hotels</Link>
```

| | `<a href>` thường | **`<Link>`** (next/link) |
|---|---|---|
| Tải lại trang | **Có** (full reload) | **Không** (chuyển client-side, mượt) |
| Giữ state app | Mất | Giữ |
| Prefetch | Không | Có (Next tự prefetch) |

> Dùng `<Link>` để chuyển giữa các route trong app; `<a>` chỉ cho link ra ngoài.

---

## 5. Q&A phỏng vấn

> [!question] 1. Next.js là gì? Khác React thuần và Vite ra sao?
> Next.js là **framework React** (team React khuyến nghị) nướng sẵn routing, SSR, server components, tối ưu. React thuần chỉ là thư viện UI; **Vite** dựng SPA nhẹ. Next.js hợp app production cần SSR/SEO.

> [!question] 2. App Router định tuyến thế nào?
> **Theo file/thư mục**: mỗi thư mục trong `app/` là một route, file **`page.js`** là nội dung. Tạo folder = tạo route, không cần đăng ký bằng code.

> [!question] 3. `layout.js` để làm gì? `children` là gì?
> `layout.js` là **khung bọc dùng chung** (header/nav) cho mọi trang con. `children` là nội dung **page hiện tại** được tiêm vào layout.

> [!question] 4. `<Link>` khác `<a>` thế nào?
> `<Link>` (next/link) chuyển trang **client-side** (không full reload, giữ state, có prefetch) → mượt. `<a>` tải lại cả trang. Dùng `<Link>` để điều hướng nội bộ.

> [!question] 5. So sánh App Router (Next.js) với React Router?
> React Router khai báo route bằng **code** (`<Route>`). App Router suy ra route từ **cấu trúc thư mục** (file-based). App Router ít boilerplate + tích hợp SSR/server components.

---

## 6. Bài tập tự luyện

1. `create-next-app`, tạo route `/mountain` và `/hotels` bằng thư mục + `page.js`.
2. Thêm header có `<Link>` vào `layout.js` để hiện trên mọi trang; kiểm tra chuyển trang không reload.
3. Thử đặt sai tên file (`index.js` thay `page.js`) và quan sát route không hoạt động.
4. So sánh: cùng app này viết bằng React Router thì cần khai báo gì.

---

## 7. Liên kết
- [[14-React-Router]] — routing kiểu code-based để so sánh
- [[17-Server-vs-Client-Components]] — component mặc định là server
- [[18-Server-Actions-Data-Fetching]] — fetch data & xử lý form
- [[00-MOC-React|MOC: React]]
