---
title: "Decorator"
section: 00-Foundations/06-Python
tags: [python, decorator, closure, functools, foundations, fresher]
related:
  - "[[07-Ham]]"
  - "[[12-OOP]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 30m
source: ["Python Official Docs", "Fluent Python — Luciano Ramalho"]
---

# Decorator

> [!summary] TL;DR
> **Decorator** = một hàm **nhận một hàm và trả về một hàm mới** đã được "bọc" thêm hành vi, mà **không sửa code gốc**. Cú pháp `@decorator` đặt trên `def` chỉ là đường tắt của `func = decorator(func)`. Nền tảng là **closure** (hàm trong nhớ được biến của hàm bao). Dùng để thêm log, đo thời gian, cache, kiểm tra quyền… (FastAPI, Flask route đều là decorator). Nhớ dùng **`functools.wraps`** để giữ tên/docstring hàm gốc.

---

## 1. Closure — nền của decorator

Hàm trong nhớ được biến của hàm bao (tận dụng tầng **E** trong LEGB → [[07-Ham]]):

```python
def multiplier(factor):           # hàm bao
    def mul(x):                   # closure — nhớ 'factor'
        return x * factor
    return mul                    # trả về HÀM

double = multiplier(2)
double(5)                         # 10  ('factor=2' vẫn được nhớ)
```

---

## 2. Decorator cơ bản

```python
import functools

def logged(func):
    @functools.wraps(func)             # giữ __name__, __doc__ của func gốc
    def wrapper(*args, **kwargs):      # bọc: nhận MỌI tham số
        print(f"Gọi {func.__name__}")
        result = func(*args, **kwargs) # gọi hàm gốc
        print(f"Xong {func.__name__}")
        return result
    return wrapper

@logged                                # = greet = logged(greet)
def greet(name):
    return f"Hi {name}"

greet("An")
# Gọi greet
# Xong greet
```

> `@logged` phía trên `def greet` **chính xác bằng** `greet = logged(greet)`.

---

## 3. Vì sao cần `functools.wraps`

Không có nó, hàm bị thay bằng `wrapper` → mất danh tính:

```python
greet.__name__     # không có wraps → 'wrapper' (sai!)
                   # có wraps        → 'greet' (đúng)
```

---

## 4. Ví dụ thực tế: đo thời gian & cache

```python
import time, functools

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        t0 = time.perf_counter()
        r = func(*args, **kwargs)
        print(f"{func.__name__}: {time.perf_counter()-t0:.4f}s")
        return r
    return wrapper

# cache có sẵn trong thư viện chuẩn:
@functools.lru_cache(maxsize=None)
def fib(n):
    return n if n < 2 else fib(n-1) + fib(n-2)   # nhanh nhờ nhớ kết quả
```

## 5. Decorator có tham số

```python
def repeat(times):                    # decorator factory
    def deco(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                r = func(*args, **kwargs)
            return r
        return wrapper
    return deco

@repeat(times=3)                      # 3 tầng: repeat → deco → wrapper
def hi(): print("hi")
```

> [!question] Phỏng vấn: "Decorator là gì? `@x` thực chất làm gì?"
> Decorator là hàm **nhận hàm, trả về hàm mới** bọc thêm hành vi mà không sửa hàm gốc — ứng dụng nguyên lý Open/Closed. `@x` trên `def f` chỉ là cú pháp đường tắt cho `f = x(f)`. Cơ chế dựa trên **closure**. Luôn thêm `@functools.wraps(func)` để hàm bọc giữ tên & docstring gốc.

```
★ Insight ─────────────────────────────────────
• @deco là 'syntactic sugar' của f = deco(f). Nắm điều này thì mọi
  decorator (kể cả @property, @staticmethod) đều hết bí ẩn.
• functools.wraps không 'chạy' gì cho logic — nó chỉ chép metadata
  (__name__, __doc__) để debug/introspection không bị lừa. Dễ quên
  nhưng review code sẽ bắt.
• lru_cache là decorator 'miễn phí' biến hàm đệ quy chậm O(2^n) thành
  O(n) chỉ bằng 1 dòng — minh hoạ sức mạnh memoization.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. Decorator nhận gì và trả về gì? `@deco` tương đương biểu thức nào?
2. Closure là gì? Vì sao decorator cần closure?
3. `functools.wraps` để làm gì? Bỏ nó thì mất gì?
4. Viết phác 1 decorator `timer` đo thời gian chạy hàm.

---

## Liên quan
- [[07-Ham]] — closure, LEGB (tầng Enclosing), hàm first-class
- [[12-OOP]] — `@property`/`@classmethod` cũng là decorator
- [[19-Thu-vien-chuan]] — `functools` (wraps, lru_cache, reduce)
