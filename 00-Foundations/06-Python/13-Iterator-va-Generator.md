---
title: "Iterator & Generator"
section: 00-Foundations/06-Python
tags: [python, iterator, generator, yield, lazy, foundations, fresher]
related:
  - "[[08-Comprehension-va-Generator-expression]]"
  - "[[06-Dieu-khien-luong]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 30m
source: ["Python Official Docs", "Fluent Python — Luciano Ramalho"]
---

# Iterator & Generator

> [!summary] TL;DR
> **Iterable** = object lặp được bằng `for` (list, str, dict, file…) — có `__iter__`. **Iterator** = object sinh từng giá trị qua `__next__`, cạn thì ném `StopIteration`. **Generator** = cách dễ nhất tạo iterator: viết hàm có **`yield`** thay vì `return`. Generator **lazy** — sinh giá trị **khi cần**, giữ nguyên trạng thái giữa các lần gọi, tốn **O(1) bộ nhớ** thay vì tạo cả list. Cốt lõi cho xử lý dữ liệu lớn / vô hạn / streaming.

---

## 1. Iterable vs Iterator

```python
nums = [1, 2, 3]          # iterable (có __iter__)
it = iter(nums)           # lấy iterator
next(it)                  # 1
next(it)                  # 2
next(it)                  # 3
next(it)                  # StopIteration!  (cạn)
```

Vòng `for` thực chất: gọi `iter()` lấy iterator, lặp `next()` tới khi `StopIteration`.

| | Iterable | Iterator |
|---|----------|----------|
| Là gì | "lặp được" | "đang lặp, biết giá trị kế tiếp" |
| Method | `__iter__` | `__iter__` **và** `__next__` |
| Ví dụ | list, str, dict | kết quả của `iter()`, generator |

---

## 2. ⭐ Generator với `yield`

Hàm có `yield` → gọi nó **không chạy ngay**, trả về một **generator object**:

```python
def countdown(n):
    while n > 0:
        yield n          # 'tạm dừng' tại đây, trả 1 giá trị, NHỚ trạng thái
        n -= 1

gen = countdown(3)       # chưa chạy gì
next(gen)                # 3  (chạy tới yield đầu)
next(gen)                # 2  (chạy TIẾP từ chỗ dừng)
for x in countdown(3):   # 3, 2, 1
    print(x)
```

`yield` khác `return`: `return` kết thúc hàm; `yield` **tạm dừng và lưu trạng thái**, lần `next` sau chạy tiếp từ đó.

---

## 3. Vì sao generator quan trọng — lazy & bộ nhớ

```python
# list: tạo 10 triệu phần tử trong RAM ngay
nums = [x ** 2 for x in range(10_000_000)]     # tốn ~hàng trăm MB

# generator: gần như 0 bộ nhớ, sinh dần
def squares(n):
    for x in range(n):
        yield x ** 2

total = sum(squares(10_000_000))               # O(1) bộ nhớ

# generator còn biểu diễn dãy VÔ HẠN:
def naturals():
    n = 1
    while True:           # không bao giờ hết
        yield n
        n += 1
```

| | List | Generator |
|---|------|-----------|
| Sinh giá trị | tất cả ngay | từng cái khi cần (lazy) |
| Bộ nhớ | O(n) | **O(1)** |
| Duyệt lại | nhiều lần | **một lần** (cạn là hết) |
| Dãy vô hạn | ❌ | ✅ |

> [!question] Phỏng vấn: "`yield` khác `return` thế nào? Khi nào dùng generator?"
> `return` trả giá trị **rồi kết thúc** hàm. `yield` trả giá trị nhưng **tạm dừng, giữ nguyên trạng thái** — lần sau chạy tiếp. Hàm có `yield` là **generator**, sinh giá trị **lazy** (khi cần). Dùng khi: dữ liệu **lớn** (khỏi nạp hết vào RAM), **dòng/stream**, hoặc dãy **vô hạn**. So với comprehension lazy `( … )` ở [[08-Comprehension-va-Generator-expression]].

---

## 4. `yield from` — ủy quyền

```python
def chain(*iterables):
    for it in iterables:
        yield from it          # nối các iterable, gọn hơn for...yield
list(chain([1,2], [3,4]))      # [1,2,3,4]
```

```
★ Insight ─────────────────────────────────────
• for là 'iter() + next() tới StopIteration' — hiểu điều này giải
  thích vì sao file, dict, range đều for được: chúng là iterable.
• yield biến hàm thành 'cỗ máy tạm dừng được'. Trạng thái local được
  giữ nguyên giữa các next — đây là điều return không làm được.
• Generator = bộ lọc bộ nhớ: đổi 'có sẵn tất cả, tốn RAM' lấy 'sinh
  dần, gần như 0 RAM, đi 1 lần'. Nền cho pipeline xử lý dữ liệu lớn.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. Iterable khác Iterator thế nào (theo `__iter__`/`__next__`)?
2. `for` lặp một list, dưới nắp ca-pô gọi những gì?
3. `yield` khác `return` ra sao? Hàm có `yield` gọi ra cái gì?
4. Khi nào generator thắng list? Generator biểu diễn dãy vô hạn được không?

---

## Liên quan
- [[08-Comprehension-va-Generator-expression]] — generator expression `(…)`
- [[06-Dieu-khien-luong]] — `for` chạy trên iterator
- [[11-File-IO-va-with]] — đọc file `for line in f` là lazy iteration
