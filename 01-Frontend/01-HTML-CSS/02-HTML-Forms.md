---
title: "HTML Forms"
section: 01-HTML-CSS
tags: [html, form, input, validation, accessibility, fresher, frontend]
related:
  - "[[01-HTML-Semantic-Tags]]"
  - "[[10-Form-Validation]]"
difficulty: ⭐⭐
estimated_time: 35m
source: [MDN]
---

# HTML Forms

> [!summary] TL;DR
> HTML Form là cơ chế thu thập dữ liệu từ người dùng qua các `<input>`, `<select>`, `<textarea>`. Mỗi control cần `<label>` liên kết đúng cách để đảm bảo accessibility. Thuộc tính `type` của `<input>` quyết định loại dữ liệu và validation tích hợp sẵn.

> [!tip] 🎯 Hiểu trong 30 giây
> **Form = "tờ đơn" để người dùng điền và gửi dữ liệu lên server.** Các ô là `<input>` (ô nhập), `<select>` (chọn từ danh sách), `<textarea>` (ô nhập nhiều dòng), bọc trong `<form>`.
> - Mỗi ô nên có **`<label>` gắn đúng** (qua `for`/`id`): bấm vào nhãn là con trỏ nhảy vào ô, và screen reader đọc đúng tên ô → tốt cho accessibility.
> - Thuộc tính **`type`** của `<input>` quyết định *loại dữ liệu + bàn phím + kiểm tra sẵn*: `type="email"` (tự kiểm định dạng email), `type="number"`, `type="password"` (ẩn ký tự), `type="date"`...
> - Khi gửi, `method="GET"` (dữ liệu gắn vào URL — cho tìm kiếm) hay `method="POST"` (dữ liệu trong thân request — cho đăng nhập/đăng ký).

## 1. Khái niệm

**HTML Form** là tập hợp các phần tử tương tác cho phép người dùng nhập và gửi dữ liệu đến server (hoặc xử lý bằng JavaScript).

**Các thành phần chính:**
- `<form>` — Container chứa toàn bộ form, định nghĩa `action` (URL đích) và `method` (GET/POST).
- `<input>` — Phần tử đa năng nhất, thay đổi theo `type`.
- `<label>` — Nhãn liên kết với input — **bắt buộc** cho accessibility.
- `<select>` + `<option>` — Dropdown list.
- `<textarea>` — Ô nhập văn bản nhiều dòng.
- `<button>` — Nút submit/reset/button.
- `<fieldset>` + `<legend>` — Nhóm các control liên quan.

## 2. Cú pháp / API

```html
<form action="/submit" method="POST" novalidate>
  <!-- Liên kết label ↔ input qua thuộc tính "for" và "id" -->
  <label for="email">Email:</label>
  <input
    type="email"
    id="email"
    name="email"
    placeholder="you@example.com"
    required
    autocomplete="email"
  >

  <!-- Select -->
  <label for="level">Trình độ:</label>
  <select id="level" name="level">
    <option value="">-- Chọn trình độ --</option>
    <option value="junior">Junior</option>
    <option value="mid"  selected>Mid-level</option>
    <option value="senior">Senior</option>
  </select>

  <!-- Textarea -->
  <label for="bio">Giới thiệu:</label>
  <textarea id="bio" name="bio" rows="4" maxlength="500"></textarea>

  <!-- Submit -->
  <button type="submit">Gửi</button>
  <button type="reset">Xóa</button>
</form>
```

**Bảng `<input type="...">` thông dụng:**

| type | Dùng cho | Built-in validation |
|------|----------|---------------------|
| `text` | Văn bản ngắn | minlength, maxlength |
| `email` | Địa chỉ email | Kiểm tra format @ |
| `password` | Mật khẩu (ẩn ký tự) | — |
| `number` | Số | min, max, step |
| `date` | Ngày tháng | Date picker |
| `checkbox` | Chọn nhiều | — |
| `radio` | Chọn một trong nhiều | — |
| `file` | Upload file | accept |
| `hidden` | Gửi dữ liệu ẩn | — |
| `submit` | Nút gửi form | — |

## 3. Ví dụ minh họa

### Ví dụ 1: Form đăng ký tài khoản hoàn chỉnh

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>Đăng ký</title>
</head>
<body>
  <form action="/register" method="POST">
    <fieldset>
      <legend>Thông tin cá nhân</legend>

      <div>
        <label for="fullname">Họ và tên: <span aria-hidden="true">*</span></label>
        <input
          type="text"
          id="fullname"
          name="fullname"
          required
          minlength="2"
          maxlength="100"
          autocomplete="name"
        >
      </div>

      <div>
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required autocomplete="email">
      </div>

      <div>
        <label for="password">Mật khẩu:</label>
        <input
          type="password"
          id="password"
          name="password"
          required
          minlength="8"
          pattern="(?=.*\d)(?=.*[a-z])(?=.*[A-Z]).{8,}"
          title="Ít nhất 8 ký tự, có chữ hoa, chữ thường và số"
        >
      </div>

      <div>
        <label for="dob">Ngày sinh:</label>
        <input type="date" id="dob" name="dob" max="2007-01-01">
      </div>
    </fieldset>

    <fieldset>
      <legend>Giới tính</legend>
      <label><input type="radio" name="gender" value="male"> Nam</label>
      <label><input type="radio" name="gender" value="female"> Nữ</label>
      <label><input type="radio" name="gender" value="other"> Khác</label>
    </fieldset>

    <div>
      <label>
        <input type="checkbox" name="terms" required>
        Tôi đồng ý với <a href="/terms">điều khoản sử dụng</a>
      </label>
    </div>

    <button type="submit">Đăng ký</button>
  </form>
</body>
</html>
```

**Output / Kết quả mong đợi:** Form với validation tích hợp — khi submit mà chưa điền đủ, browser hiển thị tooltip lỗi mà không cần JavaScript.

**Giải thích:**
- `<fieldset>` + `<legend>` nhóm radio buttons — screen reader đọc "Giới tính" trước khi đọc từng option.
- `pattern` + `title` cung cấp custom validation message.
- `max="2007-01-01"` chặn chọn ngày sinh dưới 18 tuổi.

### Ví dụ 2: Form tìm kiếm với datalist (autocomplete)

```html
<form action="/search" method="GET">
  <label for="tech">Tìm công nghệ:</label>
  <input
    type="search"
    id="tech"
    name="q"
    list="tech-list"
    placeholder="Nhập tên công nghệ..."
    autocomplete="off"
  >
  <datalist id="tech-list">
    <option value="React">
    <option value="Vue">
    <option value="Angular">
    <option value="TypeScript">
    <option value="Node.js">
  </datalist>
  <button type="submit">Tìm</button>
</form>
```

**Output / Kết quả mong đợi:** Khi gõ vào ô search, browser gợi ý danh sách từ `<datalist>` — tính năng autocomplete native không cần JS.

```
★ Insight ─────────────────────────────────────
• `name` là "khóa", `id` là "neo cho label" — hai thứ khác mục đích, hay bị nhầm.
  Server/JS nhận data theo `name` (input thiếu name = KHÔNG được gửi đi, dù vẫn
  hiển thị). `<label for>` trỏ tới `id`. Radio buttons cùng một nhóm phải CHUNG
  một `name` thì mới "chọn 1 trong nhiều"; khác name = mỗi cái một nhóm riêng.
• Form HTML có sẵn cả một "validation engine" miễn phí: required/pattern/min/max
  + type (email/number/date) → trình duyệt tự chặn submit và báo lỗi. JS chỉ nên
  BỔ SUNG (thông báo đẹp hơn, kiểm tra chéo 2 ô), không thay thế. Đây là nền cho
  [[10-Form-Validation]] (Constraint Validation API).
─────────────────────────────────────────────────
```

## 4. Pitfalls / Bẫy thường gặp

> [!warning] Lỗi phổ biến
> - **`<label>` không liên kết với input:** Dùng `for="id"` hoặc wrap input trong label. Nếu không, click vào label text không focus vào input.
> - **Dùng `<div>` thay `<button>` cho submit:** `<div onclick="...">` không trigger form submit, không nhận phím Enter, không accessible.
> - **Quên `name` attribute:** Input không có `name` sẽ **không được gửi** trong form data.
> - **Dùng `method="GET"` cho dữ liệu nhạy cảm:** GET data hiển thị trên URL và được lưu trong browser history — luôn dùng POST cho password, thông tin cá nhân.

## 5. Câu hỏi phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "GET vs POST trong form? Vì sao cần `<label>`?"
> *"GET đính dữ liệu vào URL dưới dạng query string nên hiển thị công khai, bookmark và cache được, hợp cho tìm kiếm và lọc. POST gửi dữ liệu trong thân request, không nằm trên URL, không lưu vào history, hợp cho đăng nhập, đăng ký, hay thao tác thay đổi dữ liệu. Về label, mỗi control nên có một label liên kết qua thuộc tính for trỏ tới id của input, vì bấm vào label sẽ focus vào ô tương ứng và quan trọng hơn là screen reader đọc đúng tên ô cho người khiếm thị, đảm bảo accessibility. Ngoài ra em dùng đúng type của input để tận dụng bàn phím phù hợp và validation tích hợp sẵn của trình duyệt."*

> [!note] 🧠 Mẹo nhớ
> **GET = dữ liệu trên URL (tìm kiếm, bookmark được); POST = dữ liệu trong body (login/đăng ký, kín).** Mỗi ô cần **`<label for>`** (bấm nhãn focus ô + screen reader đọc đúng). `type` quyết định kiểu nhập + validation.

1. **Q:** Sự khác nhau giữa `method="GET"` và `method="POST"` trong form?
   **A:** GET đính data vào URL query string (hiển thị, bookmark được, cached — dùng cho tìm kiếm). POST gửi data trong HTTP body (ẩn, không lưu history — dùng cho login, đăng ký, thay đổi dữ liệu).

2. **Q:** Tại sao cần `<label>` và cách liên kết đúng?
   **A:** Label cải thiện accessibility (screen reader đọc label khi input được focus) và UX (click label cũng focus input). Liên kết bằng `<label for="input-id">` hoặc wrap: `<label>Text <input ...></label>`.

3. **Q:** `required`, `pattern`, `min/max` hoạt động như thế nào?
   **A:** Đây là HTML5 Constraint Validation API — browser kiểm tra trước khi submit, hiển thị tooltip lỗi mặc định. Có thể tắt bằng `novalidate` trên `<form>` để tự xử lý validation bằng JS.

## 6. Bài tập tự luyện

- [ ] **Bài 1:** Tạo form "Thêm sản phẩm" có: tên (text, required), giá (number, min=0), danh mục (select với 5 option), mô tả (textarea), hình ảnh (file, accept="image/*"), nút Submit. Dùng `<fieldset>` để nhóm thông tin cơ bản và thông tin chi tiết.
- [ ] **Bài 2:** Tạo form login với email + password. Thêm validation: email phải đúng format, password ít nhất 8 ký tự. Style các input khi `:valid` (border xanh) và `:invalid` (border đỏ) bằng CSS pseudo-class.

## 7. Liên kết

- Note liên quan: [[01-HTML-Semantic-Tags]], [[10-Form-Validation]], [[06-Form-Handling]]
- [MDN: `<form>` element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form)
- [MDN: Client-side form validation](https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation)
- [MDN: `<input>` types](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#input_types)
