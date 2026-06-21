---
title: "Điều khiển luồng"
section: 00-Foundations/06-Python
tags: [python, control-flow, loop, if, for, while, foundations, fresher]
related:
  - "[[03-Toan-tu-va-Bieu-thuc]]"
  - "[[07-Ham]]"
difficulty: ⭐⭐
estimated_time: 20m
source: ["Python Official Docs"]
---

# Điều khiển luồng

> [!summary] TL;DR
> Rẽ nhánh bằng **`if / elif / else`** (dựa trên truthy/falsy → [[03-Toan-tu-va-Bieu-thuc]]). Lặp bằng **`for`** (duyệt iterable: list, str, range, dict…) và **`while`** (lặp theo điều kiện). Điều khiển vòng lặp: **`break`** (thoát), **`continue`** (bỏ qua vòng này). Python có nét riêng **`for/while ... else`** (chạy `else` khi vòng lặp kết thúc *không* gặp `break`). Đếm số dùng **`range`**, duyệt kèm chỉ số dùng **`enumerate`**.

---

## 1. if / elif / else

```python
if score >= 90:
    grade = "A"
elif score >= 70:        # elif = else if, kiểm lần lượt
    grade = "B"
else:
    grade = "C"

# toán tử 3 ngôi (ternary)
status = "đỗ" if score >= 50 else "trượt"
```

Điều kiện dựa trên **truthy/falsy**: `if items:` thay vì `if len(items) > 0:`.

---

## 2. Vòng `for` — duyệt iterable

```python
for x in [10, 20, 30]:      # duyệt list
    print(x)

for ch in "abc":            # duyệt chuỗi
    print(ch)

for i in range(5):          # 0,1,2,3,4
    print(i)

for i, v in enumerate(["a","b"]):   # (0,'a'), (1,'b') — kèm chỉ số
    print(i, v)

for k, v in {"a":1}.items():        # duyệt dict
    print(k, v)

for a, b in zip([1,2], ["x","y"]):  # ghép song song → (1,'x'),(2,'y')
    print(a, b)
```

> [!tip] Đừng `for i in range(len(lst))` rồi `lst[i]`
> Pythonic là duyệt thẳng `for x in lst`, cần chỉ số thì `enumerate(lst)`.

### `range`
```python
range(5)          # 0..4
range(2, 8)       # 2..7
range(0, 10, 2)   # 0,2,4,6,8  (bước nhảy)
range(10, 0, -1)  # đếm ngược
```

---

## 3. Vòng `while`

```python
n = 5
while n > 0:        # lặp tới khi điều kiện sai
    print(n)
    n -= 1
```

---

## 4. break / continue / for-else

```python
for x in data:
    if x < 0:
        break       # thoát hẳn vòng lặp
    if x == 0:
        continue    # bỏ qua, sang vòng tiếp
    print(x)
```

**`for ... else`** (đặc trưng Python): khối `else` chạy khi vòng lặp kết thúc **bình thường** (không `break`):

```python
for x in data:
    if x == target:
        print("tìm thấy")
        break
else:                      # chỉ chạy nếu KHÔNG break
    print("không thấy")
```

> [!question] Phỏng vấn: "`for...else` chạy `else` khi nào?"
> Khi vòng lặp **chạy hết mà không gặp `break`**. Dùng tiện cho mẫu "tìm kiếm — nếu không thấy thì…", khỏi cần biến cờ `found`.

```
★ Insight ─────────────────────────────────────
• Python lặp trên iterable trực tiếp (for x in lst), không lặp theo
  chỉ số như C. Cần chỉ số → enumerate; ghép 2 list → zip. Đây là
  dấu hiệu code "Pythonic".
• for-else là nét lạ ít người biết: else = "vòng lặp xong êm, không
  break" — gọn cho bài toán tìm-không-thấy.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. Khác nhau `break` vs `continue`?
2. `for...else` chạy khối `else` khi nào?
3. Duyệt list kèm chỉ số nên dùng gì? Ghép 2 list song song dùng gì?
4. `range(10, 0, -1)` sinh ra dãy nào?

---

## Liên quan
- [[03-Toan-tu-va-Bieu-thuc]] — truthy/falsy quyết định rẽ nhánh
- [[07-Ham]] — gói logic lặp/nhánh thành hàm
- [[08-Comprehension-va-Generator-expression]] — viết gọn vòng for
