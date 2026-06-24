---
title: "Form Validation"
section: 02-DOM-Event
tags: [form, validation, constraint, fresher, frontend]
related:
  - "[[02-Selecting-DOM-Elements]]"
  - "[[06-Event-Handling]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [MDN, tự bổ sung]
---

# Form Validation

> [!summary] TL;DR
> Validate form bằng 2 cách: **JS thuần** (kiểm tra `value` trong `submit` listener + `preventDefault()`) và **HTML5 Constraint Validation API** (`required`, `pattern`, `minlength`, `setCustomValidity()`). Kết hợp cả hai: HTML5 xử lý cơ bản, JS xử lý logic phức tạp. Luôn validate lại phía server — client-side validation chỉ là UX.

> [!tip] 🎯 Hiểu trong 30 giây
> **Validate form = kiểm tra người dùng nhập đúng chưa trước khi gửi đi** (email có hợp lệ không, mật khẩu đủ dài chưa). Hai mức:
> - **HTML5 sẵn có**: chỉ cần thêm thuộc tính vào thẻ — `required` (bắt buộc), `type="email"`, `minlength`, `pattern` (regex) — trình duyệt tự chặn và báo. Nhanh, ít code.
> - **JS thuần**: trong handler `submit`, gọi `e.preventDefault()` (chặn gửi đi), tự kiểm tra `value` rồi hiện lỗi. Dùng khi cần logic phức tạp (mật khẩu nhập lại phải khớp, kiểm tra chéo nhiều ô).
>
> **Câu chốt CỰC quan trọng (hay bị hỏi):** validate ở client **chỉ để trải nghiệm tốt** (báo lỗi sớm), **KHÔNG bảo mật** — kẻ xấu bỏ qua được dễ dàng. **Server LUÔN phải validate lại.**

---

## 1. Khái niệm

### Hai loại form validation

| Loại | Cách thực hiện | Ưu điểm | Nhược điểm |
|---|---|---|---|
| **HTML5 Constraint** | `required`, `pattern`, `min`, `max`, `type` | Browser tự handle UI | Giới hạn, không customize |
| **JavaScript** | `submit` listener + kiểm tra logic | Linh hoạt hoàn toàn | Phải tự build UI error |
| **Kết hợp** | HTML5 + JS bổ sung | Tốt nhất trong thực tế | — |

### Constraint Validation API

Browser cung cấp interface tích hợp:
- `element.validity` — `ValidityState` object (`valid`, `valueMissing`, `typeMismatch`, `patternMismatch`, `tooShort`...)
- `element.checkValidity()` — trả về `boolean`, fire `invalid` event nếu không hợp lệ
- `element.reportValidity()` — như `checkValidity()` nhưng hiển thị browser tooltip
- `element.setCustomValidity(msg)` — set lỗi custom (rỗng để clear)

> [!tip] `setCustomValidity('')` để clear lỗi
> Khi gọi `setCustomValidity('Lỗi gì đó')`, field đó bị đánh dấu invalid. Sau khi user sửa, gọi `setCustomValidity('')` để clear — nếu không, field vẫn invalid dù value đúng.

```
★ Insight ─────────────────────────────────────
• Ranh giới sống còn: client-side validation chỉ là UX, KHÔNG phải bảo mật. User
  tắt JS, sửa HTML trong DevTools, hay gọi thẳng API bằng curl đều bỏ qua được
  hết. Vì vậy server LUÔN phải validate lại — client validate để phản hồi nhanh
  & đỡ round-trip, server validate để giữ dữ liệu đúng. Phỏng vấn rất hay gặp câu này.
• Đừng tự viết lại cái trình duyệt đã cho: Constraint Validation API (required,
  pattern, type, validity.*) lo phần cơ bản. JS chỉ nên xử lý cái HTML KHÔNG làm
  được — so khớp 2 field (password match), gọi API check trùng username, thông
  báo lỗi đẹp. Dùng `novalidate` để tắt tooltip mặc định khi muốn tự kiểm soát UI.
─────────────────────────────────────────────────
```

---

## 2. Cú pháp

### 2.1 JS Validation cơ bản

```javascript
const form = document.getElementById('myForm');

form.addEventListener('submit', (e) => {
  e.preventDefault(); // ngăn reload trang

  const name  = document.getElementById('name').value.trim();
  const email = document.getElementById('email').value.trim();
  const errors = [];

  if (!name) {
    errors.push('Tên không được để trống');
  }

  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!email) {
    errors.push('Email không được để trống');
  } else if (!emailRegex.test(email)) {
    errors.push('Email không hợp lệ');
  }

  if (errors.length > 0) {
    displayErrors(errors);
    return;
  }

  // Pass — submit data
  submitForm({ name, email });
});

function displayErrors(errors) {
  const errorEl = document.getElementById('errorList');
  errorEl.replaceChildren();

  errors.forEach(msg => {
    const li       = document.createElement('li');
    li.textContent = msg;
    errorEl.appendChild(li);
  });
}
```

### 2.2 HTML5 Constraint Validation

```html
<form id="signupForm">
  <input id="username" type="text"
    required
    minlength="3"
    maxlength="20"
    pattern="[a-zA-Z0-9_]+"
    title="Chỉ dùng chữ, số, và gạch dưới">

  <input id="email" type="email" required>

  <input id="age" type="number" min="18" max="99" required>

  <input id="password" type="password" required minlength="8">

  <button type="submit">Đăng ký</button>
</form>
```

```javascript
// Browser tự validate khi submit — hiển thị tooltip mặc định
// Để ngăn browser UI và tự handle:
document.getElementById('signupForm').addEventListener('submit', (e) => {
  if (!e.target.checkValidity()) {
    e.preventDefault();
    e.target.reportValidity(); // hiện browser tooltip
    return;
  }
  // Form valid — tiếp tục
});
```

### 2.3 Custom Validity với `setCustomValidity`

```javascript
const passwordEl = document.getElementById('password');
const confirmEl  = document.getElementById('confirmPassword');

function validatePasswordMatch() {
  if (passwordEl.value !== confirmEl.value) {
    confirmEl.setCustomValidity('Mật khẩu không khớp');
  } else {
    confirmEl.setCustomValidity(''); // clear lỗi
  }
}

passwordEl.addEventListener('input', validatePasswordMatch);
confirmEl.addEventListener('input', validatePasswordMatch);
```

### 2.4 Real-time validation với inline error messages

```javascript
function setupFieldValidation(fieldId, errorId, validator) {
  const field   = document.getElementById(fieldId);
  const errorEl = document.getElementById(errorId);

  field.addEventListener('blur', () => {
    const msg = validator(field.value);
    errorEl.textContent = msg;
    field.classList.toggle('is-invalid', !!msg);
    field.classList.toggle('is-valid',   !msg);
  });

  field.addEventListener('input', () => {
    if (field.classList.contains('is-invalid')) {
      const msg = validator(field.value);
      errorEl.textContent = msg;
      if (!msg) {
        field.classList.replace('is-invalid', 'is-valid');
      }
    }
  });
}

// Dùng
setupFieldValidation('username', 'usernameError', value => {
  if (!value.trim())        return 'Tên không được để trống';
  if (value.trim().length < 3) return 'Tên tối thiểu 3 ký tự';
  return ''; // hợp lệ
});

setupFieldValidation('email', 'emailError', value => {
  if (!value.trim())                        return 'Email không được để trống';
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) return 'Email không hợp lệ';
  return '';
});
```

---

## 3. Ví dụ thực tế

### Ví dụ 1: Form đăng ký hoàn chỉnh với real-time validation

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8"><title>Đăng ký</title>
  <style>
    .field-group { margin-bottom: 16px; }
    label { display: block; margin-bottom: 4px; font-weight: bold; }
    input { width: 100%; padding: 8px; border: 2px solid #ccc; border-radius: 4px; font-size: 14px; }
    input.is-invalid { border-color: #e74c3c; }
    input.is-valid   { border-color: #2ecc71; }
    .error-msg { color: #e74c3c; font-size: 12px; margin-top: 4px; min-height: 16px; }
    button[type=submit] { padding: 10px 24px; background: #3498db; color: white; border: none; cursor: pointer; border-radius: 4px; }
    button[type=submit]:hover { background: #2980b9; }
    #formSuccess { color: #27ae60; font-weight: bold; display: none; }
  </style>
</head>
<body>
  <form id="registerForm" novalidate>
    <div class="field-group">
      <label for="fullname">Họ và tên</label>
      <input id="fullname" type="text" placeholder="Nguyễn Văn A">
      <div class="error-msg" id="fullnameError"></div>
    </div>

    <div class="field-group">
      <label for="email">Email</label>
      <input id="email" type="email" placeholder="example@email.com">
      <div class="error-msg" id="emailError"></div>
    </div>

    <div class="field-group">
      <label for="password">Mật khẩu</label>
      <input id="password" type="password" placeholder="Tối thiểu 8 ký tự">
      <div class="error-msg" id="passwordError"></div>
    </div>

    <div class="field-group">
      <label for="confirm">Xác nhận mật khẩu</label>
      <input id="confirm" type="password" placeholder="Nhập lại mật khẩu">
      <div class="error-msg" id="confirmError"></div>
    </div>

    <button type="submit">Đăng ký</button>
    <div id="formSuccess">Đăng ký thành công!</div>
  </form>

  <script>
    // Validators — trả về chuỗi lỗi hoặc '' nếu hợp lệ
    const validators = {
      fullname: v => {
        if (!v.trim())            return 'Họ tên không được để trống';
        if (v.trim().length < 2)  return 'Họ tên tối thiểu 2 ký tự';
        return '';
      },
      email: v => {
        if (!v.trim())            return 'Email không được để trống';
        if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v)) return 'Email không hợp lệ';
        return '';
      },
      password: v => {
        if (!v)           return 'Mật khẩu không được để trống';
        if (v.length < 8) return 'Mật khẩu tối thiểu 8 ký tự';
        return '';
      },
      confirm: v => {
        const pw = document.getElementById('password').value;
        if (!v)       return 'Vui lòng xác nhận mật khẩu';
        if (v !== pw) return 'Mật khẩu không khớp';
        return '';
      },
    };

    function validateField(id) {
      const field   = document.getElementById(id);
      const errorEl = document.getElementById(id + 'Error');
      const msg     = validators[id](field.value);

      errorEl.textContent = msg;
      field.classList.toggle('is-invalid', !!msg);
      field.classList.toggle('is-valid',   !msg && field.value.length > 0);
      return !msg;
    }

    // Real-time: validate khi blur, re-validate khi sửa
    ['fullname', 'email', 'password', 'confirm'].forEach(id => {
      const field = document.getElementById(id);
      field.addEventListener('blur',  () => validateField(id));
      field.addEventListener('input', () => {
        if (field.classList.contains('is-invalid')) validateField(id);
        if (id === 'password') validateField('confirm'); // re-check confirm
      });
    });

    // Submit
    document.getElementById('registerForm').addEventListener('submit', (e) => {
      e.preventDefault();
      const allValid = ['fullname', 'email', 'password', 'confirm'].every(id => validateField(id));
      if (!allValid) return;

      document.getElementById('formSuccess').style.display = 'block';
      // Gọi API thực tế ở đây...
    });
  </script>
</body>
</html>
```

### Ví dụ 2: Mini validator với Constraint Validation API

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8"><title>Constraint API Demo</title>
</head>
<body>
  <form id="demoForm">
    <input id="phone" type="tel"
      required
      pattern="0[0-9]{9}"
      title="Nhập số điện thoại 10 chữ số, bắt đầu bằng 0"
      placeholder="0901234567">
    <div id="phoneMsg"></div>
    <button type="submit">Gửi</button>
  </form>

  <script>
    const phoneEl = document.getElementById('phone');
    const phoneMsg = document.getElementById('phoneMsg');

    phoneEl.addEventListener('input', () => {
      if (phoneEl.validity.valueMissing) {
        phoneMsg.textContent = 'Vui lòng nhập số điện thoại';
      } else if (phoneEl.validity.patternMismatch) {
        phoneMsg.textContent = 'Số điện thoại phải gồm 10 chữ số, bắt đầu bằng 0';
      } else {
        phoneMsg.textContent = '';
      }
    });

    document.getElementById('demoForm').addEventListener('submit', (e) => {
      e.preventDefault();
      if (!phoneEl.checkValidity()) {
        phoneEl.reportValidity();
        return;
      }
      alert('Số hợp lệ: ' + phoneEl.value);
    });
  </script>
</body>
</html>
```

---

## 4. Pitfalls thường gặp

> [!warning] Pitfall 1: Quên `e.preventDefault()` khi submit
> Không có `preventDefault()`, form reload trang khi submit → mọi validation và state bị mất. Luôn gọi ngay đầu submit handler, trước khi validate.

> [!warning] Pitfall 2: Quên clear `setCustomValidity`
> Gọi `setCustomValidity('lỗi')` đánh dấu field invalid. Khi user sửa và value đúng, phải gọi `setCustomValidity('')` để clear — nếu không field vẫn invalid mãi mãi.

> [!warning] Pitfall 3: Client-side validation không thay thế server-side
> User có thể tắt JS hoặc can thiệp network request. Luôn validate lại ở server. Client validation chỉ là UX để giảm round-trip và cải thiện trải nghiệm người dùng.

> [!tip] `novalidate` trên `<form>` để tắt browser validation mặc định
> Thêm `novalidate` attribute vào `<form>` để tắt browser tooltip tự động — cho phép bạn hoàn toàn kiểm soát UI validation. Vẫn có thể dùng `validity` API bên trong JS.

---

## 5. Phỏng vấn thường gặp

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Validate phía client có đủ chưa? Khi nào HTML5, khi nào JS?"
> *"Validate phía client không đủ về mặt bảo mật, nó chỉ để cải thiện trải nghiệm, báo lỗi sớm cho người dùng. Vì client chạy trên trình duyệt nên kẻ xấu có thể bỏ qua hoặc gửi request thẳng, do đó server luôn phải validate lại. Về cách làm, validate đơn giản em ưu tiên HTML5 với các thuộc tính như required, type email, minlength, pattern vì ít code và trình duyệt tự báo. Còn logic phức tạp như mật khẩu nhập lại phải khớp, hay kiểm tra phụ thuộc nhiều ô thì em viết JS trong handler submit, gọi preventDefault để chặn gửi mặc định rồi tự kiểm tra và hiện lỗi. Thường em kết hợp cả hai, HTML5 lo cơ bản, JS lo phần khó."*

> [!note] 🧠 Mẹo nhớ
> **Client validate = UX (báo lỗi sớm), KHÔNG phải bảo mật → server LUÔN validate lại.** Đơn giản dùng **HTML5** (`required`/`pattern`), phức tạp dùng **JS** (`preventDefault` + check tay).

**Q1: Giải thích `preventDefault()` trong context form submit?**

> `e.preventDefault()` ngăn hành vi mặc định của browser khi submit form — tức là reload trang hoặc navigate đến `action` URL. Gọi nó trong `submit` listener giữ trang tại chỗ, cho phép JS validate rồi quyết định có submit hay không, và thường gọi API bằng `fetch` thay vì form native submit.

**Q2: HTML5 Constraint Validation là gì? Các attribute nào hay dùng?**

> Constraint Validation API cho phép validate input mà không cần JS validation thủ công. Attributes hay dùng: `required` (không rỗng), `type="email"/"url"/"number"` (kiểm tra format), `min`/`max` (range), `minlength`/`maxlength` (độ dài), `pattern` (regex). Kết hợp với `element.validity.valueMissing`, `element.validity.patternMismatch` trong JS để customize error messages.

**Q3: Khi nào dùng JS validation, khi nào dùng HTML5?**

> Dùng HTML5 cho validation đơn giản: field bắt buộc, format email, range số, regex pattern. Dùng JS khi cần: validation phức tạp (password match, username availability check qua API), custom error UI, validation cross-field (ngày bắt đầu < ngày kết thúc). Thực tế tốt nhất: kết hợp — HTML5 attrs + `novalidate` trên form + JS custom error display.

---

## 6. Bài tập thực hành

**Bài 1:** Tạo form đăng nhập (email + password) với real-time validation: hiện lỗi khi blur, ẩn lỗi khi input hợp lệ. Show/hide password toggle.

**Bài 2:** Xây dựng form đặt vé (tên, CMND 12 số, ngày đi, ngày về). Validate: tất cả required, CMND đúng 12 chữ số, ngày về > ngày đi. Khi submit hợp lệ, hiện bảng tóm tắt thông tin đặt vé.

---

## 7. Liên kết

- [[07-Event-Propagation-Delegation]] — preventDefault() chi tiết
- [[09-Fetch-API]] — Gọi API sau khi validate xong
- [[06-Event-Handling]] — addEventListener, event object
- [[02-Selecting-DOM-Elements]] — Lấy giá trị từ form fields
