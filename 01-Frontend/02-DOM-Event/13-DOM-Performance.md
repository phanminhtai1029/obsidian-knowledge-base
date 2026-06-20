---
title: "DOM Performance & An toàn (XSS)"
section: 02-DOM-Event
tags: [dom, performance, reflow, repaint, xss, innerhtml, textcontent, event-delegation, fresher, frontend]
related:
  - "[[02-Selecting-DOM-Elements]]"
  - "[[05-Creating-Adding-Managing-Nodes]]"
  - "[[07-Event-Propagation-Delegation]]"
difficulty: ⭐⭐⭐
estimated_time: 30m
source: [JS-DOM raw transcript]
---

# DOM Performance & An toàn (XSS)

> [!summary] TL;DR
> Thao tác DOM **đắt**, nên tối ưu bằng vài nguyên tắc: (1) **Lưu reference** phần tử sau khi select, đừng select lại trong vòng lặp (select từ trang là việc nặng); (2) **Cập nhật cục bộ** — thêm 1 dòng vào bảng thay vì render lại cả bảng; **giảm số lần chạm DOM**; (3) Dùng **event delegation** khi có nhiều phần tử con cần cùng một event (gắn 1 listener ở cha thay vì N listener); (4) Cẩn thận **`innerHTML`** — vừa dễ dính **XSS** (chèn script độc) vừa chậm (parse lại cả chuỗi HTML); ưu tiên **`createElement`** + **`textContent`** khi chỉ thêm text.

---

## 1. Lưu reference, đừng select lại

```javascript
// ❌ Chậm: select lại trong mỗi vòng lặp
for (const emp of employees) {
  document.getElementById("employeeList").appendChild(makeLi(emp));  // select N lần
}

// ✅ Nhanh: select MỘT lần, lưu reference
const list = document.getElementById("employeeList");
for (const emp of employees) {
  list.appendChild(makeLi(emp));     // dùng lại reference
}
```

> Truy vấn DOM (`getElementById`, `querySelector`) tương đối nặng. Lặp lại trong vòng lặp = lãng phí.

---

## 2. Giảm số lần chạm DOM & cập nhật cục bộ

| Nên | Tránh |
|---|---|
| Thêm 1 phần tử mới vào bảng | Render lại **cả bảng** |
| Gom nhiều thay đổi rồi chèn một lần | Chèn từng cái gây nhiều reflow |
| Dùng `DocumentFragment` để dựng off-DOM | append từng node trực tiếp lên DOM live |

```javascript
// Gom nhiều node vào fragment (ngoài DOM) rồi chèn MỘT lần → ít reflow
const frag = document.createDocumentFragment();
for (const emp of employees) frag.appendChild(makeLi(emp));
list.appendChild(frag);    // chỉ 1 lần chạm DOM live
```

```
★ Insight ─────────────────────────────────────
• Vì sao "ít chạm DOM" lại nhanh: mỗi lần đổi layout-relevant, trình duyệt phải
  REFLOW (tính lại vị trí/kích thước) rồi REPAINT (vẽ lại). Append 1000 node trực
  tiếp = có thể kích hoạt reflow nhiều lần. Dựng trong DocumentFragment (nằm ngoài
  cây hiển thị) rồi chèn một phát → một reflow. Đây là tối ưu DOM kinh điển nhất.
─────────────────────────────────────────────────
```

> **Reflow** (layout): tính lại hình học khi kích thước/vị trí đổi — đắt. **Repaint**: vẽ lại pixel khi màu/hình đổi (không đổi layout) — rẻ hơn. Mục tiêu: giảm reflow.

---

## 3. Event delegation thay vì N listener

Nhiều phần tử con cần cùng một event → gắn **một** listener ở phần tử cha, dùng `event.target` để biết ai bị click:

```javascript
// Thay vì gắn listener cho từng <li>, gắn 1 cái ở <ul>
list.addEventListener("click", (e) => {
  if (e.target.matches("li")) {
    console.log("Clicked:", e.target.textContent);
  }
});
```

Lợi: ít listener (nhẹ bộ nhớ), tự áp cho phần tử **thêm sau**, code dễ đọc. Chi tiết: [[07-Event-Propagation-Delegation]].

---

## 4. `innerHTML` — rủi ro XSS & hiệu năng

> [!warning] `innerHTML` với dữ liệu người dùng = lỗ hổng XSS
> **Cross-Site Scripting (XSS)**: kẻ xấu nhập chuỗi chứa `<script>`/`<img onerror=...>`; nếu bạn nhét thẳng vào `innerHTML`, mã độc **chạy** trong trang → đánh cắp cookie, mạo danh. Ngoài ra `innerHTML` còn **chậm** vì trình duyệt phải parse & render lại **toàn bộ** chuỗi HTML của phần tử.

| Cách | An toàn XSS | Chèn được HTML | Hiệu năng |
|---|---|---|---|
| `textContent` / `innerText` | ✅ An toàn | Không (chỉ text) | Tốt |
| `createElement` + append | ✅ An toàn | Có (tạo từng node) | Tốt |
| `innerHTML` | ❌ **Nguy hiểm** với input người dùng | Có | Kém (parse cả chuỗi) |

```javascript
// ❌ Nguy hiểm nếu userInput từ người dùng
el.innerHTML = userInput;

// ✅ An toàn: chỉ thêm text
el.textContent = userInput;

// ✅ An toàn: cần cấu trúc HTML thì tạo node
const li = document.createElement("li");
li.textContent = userInput;
el.appendChild(li);
```

> `textContent` vs `innerText`: cả hai chỉ xử lý **text thuần**. Khi cần chèn phần tử HTML con thì không thay được `innerHTML`/`createElement`.

---

## 5. Q&A phỏng vấn

> [!question] 1. Vì sao nên lưu reference phần tử thay vì select lại nhiều lần?
> Vì truy vấn DOM (`getElementById`/`querySelector`) là thao tác **nặng**. Lưu reference một lần rồi tái dùng (nhất là trong vòng lặp) giảm chi phí đáng kể.

> [!question] 2. Reflow và repaint khác nhau thế nào? Tối ưu ra sao?
> **Reflow** = tính lại layout (vị trí/kích thước) — đắt. **Repaint** = vẽ lại pixel (màu) không đổi layout — rẻ hơn. Tối ưu: giảm số lần chạm DOM, gom thay đổi, dùng **`DocumentFragment`** để dựng ngoài DOM rồi chèn một lần.

> [!question] 3. Vì sao event delegation tốt cho hiệu năng?
> Gắn **một** listener ở phần tử cha thay vì N listener ở từng con → ít bộ nhớ, tự áp cho phần tử thêm sau, code gọn. Dùng `event.target`/`matches` để xác định phần tử bị tác động.

> [!question] 4. `innerHTML` có rủi ro gì? Khi nào nên tránh?
> Dễ dính **XSS** nếu nhét dữ liệu người dùng (mã `<script>` sẽ chạy) và **chậm** (parse lại cả chuỗi). Tránh dùng với input không tin cậy; ưu tiên `textContent` hoặc `createElement`.

> [!question] 5. `textContent` và `innerHTML` chọn cái nào?
> Chỉ thêm **text** → `textContent` (an toàn, nhanh). Cần dựng **cấu trúc HTML** → `createElement` (an toàn) hoặc `innerHTML` (chỉ với dữ liệu **tin cậy**, đã sanitize).

---

## 6. Bài tập tự luyện

1. Viết 2 phiên bản render 1000 `<li>`: bản select lại trong loop và bản lưu reference + `DocumentFragment`; đo thời gian.
2. Đổi N listener trên từng `<li>` thành 1 event delegation ở `<ul>`; thêm `<li>` mới và kiểm tra vẫn hoạt động.
3. Tạo demo XSS: nhét `<img src=x onerror=alert(1)>` qua `innerHTML`, rồi sửa sang `textContent` để chặn.
4. Giải thích vì sao append vào `DocumentFragment` rồi chèn một lần ít reflow hơn append trực tiếp.

---

## 7. Liên kết
- [[02-Selecting-DOM-Elements]] — chi phí select, lưu reference
- [[05-Creating-Adding-Managing-Nodes]] — `createElement`, `DocumentFragment`
- [[07-Event-Propagation-Delegation]] — event delegation
- [[11-Accessibility-a11y]] — `textContent` an toàn
- [[00-MOC-DOM|MOC: DOM & Events]]
