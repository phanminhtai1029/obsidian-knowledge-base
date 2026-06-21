---
title: "Comprehension & Generator expression"
section: 00-Foundations/06-Python
tags: [python, comprehension, generator, lazy, foundations, fresher]
related:
  - "[[06-Dieu-khien-luong]]"
  - "[[13-Iterator-va-Generator]]"
difficulty: ⭐⭐⭐
estimated_time: 25m
source: ["Python Official Docs"]
---

# Comprehension & Generator expression

> [!summary] TL;DR
> **Comprehension** = cú pháp gọn để tạo list/dict/set từ một iterable trong 1 dòng, thay cho vòng `for` + `append`. Có **list** `[x for x in …]`, **dict** `{k:v for …}`, **set** `{x for …}`. Đổi `[]` thành `()` được **generator expression** — không tạo sẵn toàn bộ mà **sinh từng phần tử khi cần (lazy)**, tiết kiệm bộ nhớ cho dữ liệu lớn.

---

## 1. List comprehension

```python
# thay vì:
squares = []
for x in range(5):
    squares.append(x ** 2)

# viết gọn:
squares = [x ** 2 for x in range(5)]        # [0,1,4,9,16]

# có điều kiện (filter):
evens = [x for x in range(10) if x % 2 == 0]    # [0,2,4,6,8]

# có if-else (transform):
labels = ["chẵn" if x % 2 == 0 else "lẻ" for x in range(3)]

# lồng nhau (ma trận phẳng):
flat = [v for row in matrix for v in row]
```

> Đọc theo thứ tự: `[ <biểu_thức>  for ...  if ... ]` — phần `for/if` đọc trái→phải như viết vòng lặp thường.

---

## 2. Dict & Set comprehension

```python
{x: x**2 for x in range(4)}            # {0:0, 1:1, 2:4, 3:9}
{w.lower() for w in ["A","a","B"]}     # {'a','b'} — set, tự khử trùng
{k: v for k, v in d.items() if v > 0}  # lọc dict
```

---

## 3. Generator expression — lazy

Đổi `[]` → `()`: không tạo list trong RAM, **sinh dần** từng giá trị:

```python
gen = (x ** 2 for x in range(1_000_000))   # gần như tốn 0 bộ nhớ
sum(x ** 2 for x in range(1_000_000))      # truyền thẳng vào hàm, không cần []
next(gen)                                   # 0  — lấy từng cái
```

| | List comprehension `[…]` | Generator expression `(…)` |
|---|--------------------------|----------------------------|
| Tạo | **toàn bộ** ngay trong RAM | **từng phần tử** khi cần (lazy) |
| Bộ nhớ | O(n) | **O(1)** |
| Duyệt lại | nhiều lần | **một lần** (cạn là hết) |
| Khi nào | cần list thật, dùng lại | dữ liệu lớn / stream, duyệt 1 lần |

> [!question] Phỏng vấn: "Khác nhau `[x for x in …]` và `(x for x in …)`?"
> `[]` tạo **list đầy đủ ngay** (tốn O(n) bộ nhớ). `()` tạo **generator lazy** — chỉ sinh giá trị khi lặp/`next()`, tốn O(1) bộ nhớ, **duyệt được một lần**. Với dữ liệu lớn hoặc chỉ cần đi qua một lần (vd `sum(...)`) → dùng generator. Chi tiết generator ở [[13-Iterator-va-Generator]].

```
★ Insight ─────────────────────────────────────
• Comprehension không chỉ ngắn — thường NHANH hơn for+append vì tối
  ưu ở tầng C. Nhưng đừng nhồi quá 2 tầng for/if, mất readability.
• Đổi [] thành () là chuyển từ "tốn RAM, có sẵn" sang "tiết kiệm RAM,
  sinh dần". Mẹo đáng giá khi xử lý file lớn / dòng dữ liệu.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. Viết list comprehension lấy bình phương các số chẵn 0–9.
2. Dict comprehension đảo `{1:'a'}` thành `{'a':1}`?
3. `[…]` khác `(…)` ở bộ nhứ và số lần duyệt thế nào?
4. Khi nào nên dùng generator expression thay vì list comprehension?

---

## Liên quan
- [[13-Iterator-va-Generator]] — generator, `yield`, lazy evaluation
- [[06-Dieu-khien-luong]] — vòng for mà comprehension viết gọn
- [[15-Functional-va-Context-manager]] — so với map/filter
