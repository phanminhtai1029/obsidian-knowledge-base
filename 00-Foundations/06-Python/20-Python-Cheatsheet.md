---
title: "Python Cheatsheet — Cú pháp, CTDL & Glossary"
section: 00-Foundations/06-Python
tags: [python, cheatsheet, reference, glossary, interview, foundations, fresher]
related:
  - "[[04-List-Tuple-Dict-Set]]"
  - "[[12-OOP]]"
  - "[[00-MOC-Python]]"
difficulty: ⭐⭐
estimated_time: 20m
source: ["Python Official Docs"]
---

# Python Cheatsheet — Tra cứu nhanh

> [!summary] TL;DR
> Trang tổng hợp ôn nhanh trước thi/phỏng vấn: **bảng so sánh 4 CTDL**, **mutable vs immutable**, **cú pháp nhanh**, **bẫy phỏng vấn**, và **glossary** thuật ngữ. Câu hỏi vàng: *"array vs list vs dict"* → [[04-List-Tuple-Dict-Set]].

---

## 1. List · Tuple · Dict · Set

| | list `[]` | tuple `()` | dict `{k:v}` | set `{}` |
|---|-----------|-----------|--------------|----------|
| Thứ tự | ✅ | ✅ | ✅ (chèn) | ❌ |
| Mutable | ✅ | ❌ | ✅ | ✅ |
| Trùng | ✅ | ✅ | key ❌ | ❌ |
| Truy cập | index | index | key | — |
| Tìm phần tử | O(n) | O(n) | **O(1)** | **O(1)** |
| Hashable | ❌ | ✅ | ❌ | ❌ |

---

## 2. Mutable vs Immutable

| Immutable (hashable) | Mutable |
|----------------------|---------|
| int, float, bool, str, tuple, frozenset, bytes | list, dict, set, bytearray |
| "sửa" = tạo object mới | sửa tại chỗ, `id` giữ nguyên |
| làm key dict / phần tử set ✅ | không ✅ |

---

## 3. Cú pháp nhanh

```python
# Slicing
s[a:b:c]; s[::-1]                  # đảo ngược

# Comprehension
[x*2 for x in xs if x>0]           # list
{k: v for k, v in pairs}           # dict
(x*2 for x in xs)                  # generator (lazy)

# Hàm
def f(a, b=0, *args, **kwargs): ...
f(*lst); f(**d)                    # unpacking

# Unpacking
a, *rest = [1,2,3,4]               # a=1, rest=[2,3,4]
a, b = b, a                        # hoán đổi

# Ternary / mặc định
x = a if cond else b
name = user or "Khách"

# Xử lý lỗi
try: ...
except ValueError as e: ...
finally: ...

# File
with open("f", encoding="utf-8") as fp: data = fp.read()

# Class
class C(Base):
    def __init__(self, x): super().__init__(); self.x = x

# Async
async def g(): await coro()
asyncio.run(g())
```

---

## 4. Thư viện chuẩn đáng nhớ

```python
from collections import Counter, defaultdict, deque
Counter(xs).most_common(3)
defaultdict(list); deque(maxlen=10)

import itertools as it
it.chain, it.product, it.combinations, it.groupby

from functools import lru_cache, reduce, partial, wraps
from dataclasses import dataclass, field
from enum import Enum
import bisect, heapq          # binary search / heap (→ DSA)
```

---

## 5. ⚠️ Bẫy phỏng vấn

| Bẫy | Đúng |
|-----|------|
| `def f(x, lst=[])` | dùng `lst=None` rồi gán `[]` trong hàm |
| `a = b` (list) tưởng là copy | `b = a.copy()` / `deepcopy` nếu lồng |
| `is` để so giá trị | dùng `==`; `is` chỉ cho `None`/singleton |
| `1000 is 1000` → False | số nhỏ (-5..256) mới được cache |
| `+=` nối chuỗi trong loop | `"".join(list)` (O(n) thay O(n²)) |
| async cho CPU-bound | async cho **I/O**; CPU → multiprocessing |
| `except: pass` | bắt **cụ thể** + log, đừng nuốt lỗi |
| quên `functools.wraps` | thêm vào decorator để giữ metadata |

---

## 6. Glossary

| Thuật ngữ | Nghĩa |
|-----------|-------|
| **Interpreter / bytecode** | Trình thông dịch; mã trung gian `.pyc` chạy trên PVM |
| **CPython** | Bản hiện thực Python chuẩn (viết bằng C) |
| **Dynamic / strong typing** | Kiểu xác định lúc chạy / không tự ép kiểu bừa |
| **Mutable / immutable** | Đổi được tại chỗ / không đổi (tạo mới) |
| **Hashable** | Có `__hash__` → làm key dict / phần tử set (object immutable) |
| **`is` vs `==`** | Cùng object (id) / cùng giá trị |
| **Truthy / falsy** | Giá trị quy về True/False trong `if` |
| **LEGB** | Thứ tự tìm tên: Local→Enclosing→Global→Built-in |
| **`*args` / `**kwargs`** | Gom positional (tuple) / keyword (dict) thừa |
| **Closure** | Hàm trong nhớ biến của hàm bao |
| **Decorator** | Hàm bọc hàm khác, `@d` = `f=d(f)` |
| **Comprehension** | Cú pháp gọn tạo list/dict/set |
| **Iterable / iterator** | Lặp được (`__iter__`) / sinh giá trị (`__next__`) |
| **Generator / `yield`** | Iterator lazy giữ trạng thái, sinh giá trị dần |
| **Coroutine / `await`** | Hàm async tạm dừng được, chạy bởi event loop |
| **Context manager** | Object dùng với `with` (`__enter__`/`__exit__`) |
| **Dunder method** | Method `__x__` móc vào cú pháp Python |
| **Duck typing** | "Có method đúng là dùng được", không xét lớp |
| **GIL** | Khóa toàn cục: 1 luồng chạy bytecode 1 lúc (CPython) |
| **Reference counting / GC** | Đếm tham chiếu giải phóng; GC dọn vòng tham chiếu |
| **Shallow / deep copy** | Chép vỏ ngoài / chép sâu đệ quy |
| **Type hint** | Ghi chú kiểu (PEP 484), không ép lúc chạy |
| **PEP 8 / Zen** | Style guide / triết lý thiết kế Python |

```
★ Insight ─────────────────────────────────────
• 3 câu chốt cửa phỏng vấn fresher: (1) array/list/dict khác gì,
  (2) mutable vs immutable & is/==, (3) GIL → async vs multiprocessing.
• "Pythonic" = dùng đúng công cụ ngôn ngữ cho sẵn: comprehension,
  with, enumerate/zip, Counter, f-string. Người review nhận ra ngay.
─────────────────────────────────────────────────
```

---

## Liên quan
- [[04-List-Tuple-Dict-Set]] ⭐ — câu PV "array vs list vs dict"
- [[12-OOP]] · [[14-Decorator]] · [[17-Async-asyncio]] — chủ đề nâng cao hay hỏi
- [[00-MOC-Python]] — quay lại MOC
