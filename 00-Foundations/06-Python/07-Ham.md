---
title: "Hàm (Functions)"
section: 00-Foundations/06-Python
tags: [python, function, args, kwargs, scope, legb, lambda, foundations, fresher]
related:
  - "[[06-Dieu-khien-luong]]"
  - "[[14-Decorator]]"
  - "[[18-Internals-GIL-GC]]"
difficulty: ⭐⭐⭐
estimated_time: 30m
source: ["Python Official Docs", "Fluent Python — Luciano Ramalho"]
---

# Hàm (Functions)

> [!summary] TL;DR
> Định nghĩa bằng `def`, gọi lại bằng tên. Tham số linh hoạt: **positional**, **keyword**, **default**, **`*args`** (gom positional thừa thành tuple), **`**kwargs`** (gom keyword thừa thành dict). Hàm là **first-class object** (gán biến, truyền đi, trả về). Phạm vi biến theo quy tắc **LEGB** (Local → Enclosing → Global → Built-in). Hàm ẩn danh 1 dòng = **lambda**. ⚠️ Bẫy kinh điển: **mutable default argument**.

---

## 1. Định nghĩa & tham số

```python
def greet(name, greeting="Xin chào"):   # greeting có default
    return f"{greeting}, {name}!"

greet("An")                       # 'Xin chào, An!'  (dùng default)
greet("An", "Hi")                 # positional
greet(name="An", greeting="Hi")   # keyword (rõ ràng hơn)
```

- **Positional**: theo thứ tự. **Keyword**: gọi tên `key=value`.
- **Default**: tham số có giá trị mặc định phải đặt **sau** tham số không default.

---

## 2. `*args` và `**kwargs`

```python
def demo(*args, **kwargs):
    print(args)      # tuple các positional thừa
    print(kwargs)    # dict các keyword thừa

demo(1, 2, 3, x=10, y=20)
# args   = (1, 2, 3)
# kwargs = {'x': 10, 'y': 20}

def total(*nums):           # nhận số lượng tham số bất kỳ
    return sum(nums)
total(1, 2, 3, 4)           # 10

# unpacking khi gọi:
nums = [1, 2, 3]
total(*nums)                # trải list thành positional
info = {"name": "An"}
greet(**info)               # trải dict thành keyword
```

> [!info] `/` và `*` trong chữ ký hàm
> `def f(a, b, /, c, *, d)`: trước `/` = **chỉ positional**, sau `*` = **chỉ keyword**. Hay gặp trong thư viện chuẩn.

---

## 3. ⭐ Phạm vi biến — quy tắc LEGB

Python tìm tên theo thứ tự **L → E → G → B**:

| | Phạm vi | Ví dụ |
|---|---------|-------|
| **L** | Local | biến trong hàm hiện tại |
| **E** | Enclosing | hàm bao ngoài (closure → [[14-Decorator]]) |
| **G** | Global | cấp module |
| **B** | Built-in | `len`, `print`, `range`… |

```python
x = "global"
def outer():
    x = "enclosing"
    def inner():
        x = "local"
        print(x)        # 'local' (L trước tiên)
    inner()

# sửa biến ngoài phạm vi:
count = 0
def inc():
    global count        # cho phép gán biến global
    count += 1

def make():
    n = 0
    def step():
        nonlocal n      # gán biến của hàm bao (enclosing)
        n += 1
        return n
    return step
```

> [!question] Phỏng vấn: "`global` và `nonlocal` khác gì?"
> `global x` cho phép **gán** biến ở cấp **module** từ trong hàm. `nonlocal x` cho phép gán biến ở **hàm bao gần nhất** (closure). Chỉ ĐỌC thì không cần khai báo gì (LEGB tự tìm); chỉ khi **gán** mới cần.

---

## 4. ⚠️ Bẫy: mutable default argument

```python
def add(item, bucket=[]):     # ❌ NGUY HIỂM
    bucket.append(item)
    return bucket

add(1)    # [1]
add(2)    # [1, 2]  ← KHÔNG phải [2]! default dùng chung 1 list
```

Default được tạo **một lần** lúc định nghĩa hàm, không phải mỗi lần gọi. Sửa:

```python
def add(item, bucket=None):   # ✅
    if bucket is None:
        bucket = []
    bucket.append(item)
    return bucket
```

→ Xem thêm cơ chế ở [[18-Internals-GIL-GC]].

---

## 5. lambda & hàm là first-class

```python
square = lambda x: x ** 2     # hàm ẩn danh 1 biểu thức
square(5)                     # 25

# dùng làm key sắp xếp:
people = [("An", 22), ("Bình", 19)]
people.sort(key=lambda p: p[1])     # sắp theo tuổi

# hàm truyền như tham số (first-class):
def apply(fn, value): return fn(value)
apply(square, 4)              # 16
```

```
★ Insight ─────────────────────────────────────
• LEGB giải thích "vì sao đọc được biến ngoài nhưng gán lại báo lỗi/
  tạo biến mới". Gán biến ngoài cần global/nonlocal.
• Mutable default = bẫy fresher kinh điển: default tạo MỘT LẦN, dùng
  chung mọi lời gọi. Luôn dùng None rồi khởi tạo bên trong.
• Hàm là first-class object → nền tảng cho decorator, callback, và
  lập trình hàm (map/filter) ở [[15-Functional-va-Context-manager]].
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. `*args` và `**kwargs` gom gì? Kiểu dữ liệu mỗi cái?
2. Liệt kê LEGB. Khi nào cần `global`/`nonlocal`?
3. Vì sao `def f(x, lst=[])` nguy hiểm? Sửa thế nào?
4. "Hàm là first-class object" nghĩa là gì? Cho 1 ví dụ.

---

## Liên quan
- [[14-Decorator]] — closure & hàm trả về hàm (tận dụng E trong LEGB)
- [[15-Functional-va-Context-manager]] — lambda với map/filter/reduce
- [[18-Internals-GIL-GC]] — vì sao mutable default dùng chung object
