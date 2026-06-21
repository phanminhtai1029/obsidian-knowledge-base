---
title: "Thư viện chuẩn hay dùng"
section: 00-Foundations/06-Python
tags: [python, stdlib, collections, itertools, functools, dataclasses, enum, foundations, fresher]
related:
  - "[[04-List-Tuple-Dict-Set]]"
  - "[[12-OOP]]"
  - "[[14-Decorator]]"
difficulty: ⭐⭐⭐
estimated_time: 25m
source: ["Python Official Docs"]
---

# Thư viện chuẩn hay dùng

> [!summary] TL;DR
> Python "pin sẵn" (batteries included): nhiều module chuẩn giải quyết việc thường gặp khỏi cài thêm. Đáng nhớ: **`collections`** (`Counter`, `defaultdict`, `deque`, `namedtuple`), **`itertools`** (tổ hợp lười: `chain`, `product`, `combinations`, `groupby`), **`functools`** (`reduce`, `lru_cache`, `wraps`, `partial`), **`dataclasses`** (sinh `__init__`/`__eq__` tự động), **`enum`** (hằng có tên). Biết chúng giúp viết ít code, đúng & nhanh.

---

## 1. collections

```python
from collections import Counter, defaultdict, deque, namedtuple

Counter("banana")              # {'a':3,'n':2,'b':1} — đếm tần suất
Counter(words).most_common(3)  # 3 phần tử nhiều nhất

d = defaultdict(list)          # key thiếu tự tạo [] (khỏi KeyError)
d["x"].append(1)               # không cần khởi tạo trước

q = deque([1,2,3])             # hàng đợi 2 đầu O(1)
q.appendleft(0); q.popleft()   # nhanh ở ĐẦU (list thì O(n))

Point = namedtuple("Point", "x y")
p = Point(1, 2); p.x           # tuple có tên trường
```

| Công cụ | Dùng để |
|---------|---------|
| `Counter` | đếm tần suất nhanh |
| `defaultdict` | dict có giá trị mặc định, tránh KeyError |
| `deque` | queue/stack 2 đầu O(1) |
| `namedtuple` | tuple có tên trường (nhẹ hơn class) |

---

## 2. itertools — lặp hiệu quả (lazy)

```python
import itertools as it
list(it.chain([1,2], [3,4]))            # [1,2,3,4] nối iterable
list(it.product([1,2], ["a","b"]))      # tích Descartes
list(it.combinations([1,2,3], 2))       # [(1,2),(1,3),(2,3)]
list(it.permutations([1,2], 2))         # hoán vị
list(it.islice(it.count(), 5))          # 0..4 từ bộ đếm vô hạn
for k, grp in it.groupby(sorted(data)): ...   # gom nhóm
```

---

## 3. functools

```python
from functools import reduce, lru_cache, partial, wraps

reduce(lambda a,b: a*b, [1,2,3,4])      # 24 — gộp
@lru_cache(maxsize=None)                # memoization (→ [[14-Decorator]])
def fib(n): ...

add = lambda a, b: a + b
inc = partial(add, 1)                   # cố định 1 tham số → inc(5)=6
```

---

## 4. dataclasses — class dữ liệu khỏi viết boilerplate

```python
from dataclasses import dataclass, field

@dataclass
class Point:
    x: int
    y: int = 0                          # default
    tags: list = field(default_factory=list)   # tránh mutable default!

p = Point(1, 2)
p                                       # Point(x=1, y=2)  — __repr__ tự sinh
Point(1,2) == Point(1,2)               # True             — __eq__ tự sinh
```

`@dataclass` tự sinh `__init__`, `__repr__`, `__eq__` từ annotation → ít code, ít lỗi. (Pydantic `BaseModel` là họ hàng có validate → [[16-Type-hints-va-typing]].)

---

## 5. enum — hằng số có tên

```python
from enum import Enum

class Status(Enum):
    PENDING = "pending"
    ACTIVE  = "active"
    DONE    = "done"

Status.ACTIVE          # <Status.ACTIVE: 'active'>
Status.ACTIVE.value    # 'active'
Status("active")       # <Status.ACTIVE>  — tra ngược
```

> [!question] Phỏng vấn: "Đếm tần suất phần tử trong list, cách Pythonic?"
> `from collections import Counter; Counter(my_list)` — trả dict {phần tử: số lần}, kèm `.most_common(n)`. Gọn và nhanh hơn tự viết vòng lặp `defaultdict(int)`.

```
★ Insight ─────────────────────────────────────
• "Batteries included": trước khi pip install hay tự viết, hỏi 'stdlib
  có sẵn chưa?'. Counter/defaultdict/deque giải quyết 80% bài CTDL.
• itertools là 'đại số của vòng lặp' — lazy, tổ hợp được, tiết kiệm
  bộ nhớ. combinations/product hay gặp trong bài thuật toán.
• @dataclass dùng field(default_factory=list) để né đúng bẫy mutable
  default — minh hoạ stdlib học từ chính cạm bẫy của ngôn ngữ.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. Đếm tần suất phần tử Pythonic nhất dùng gì?
2. `defaultdict(list)` khác `dict` thường thế nào?
3. `deque` hơn `list` ở thao tác nào?
4. `@dataclass` tự sinh những method nào? Vì sao dùng `default_factory`?

---

## Liên quan
- [[04-List-Tuple-Dict-Set]] — Counter/defaultdict/deque mở rộng dict/list
- [[14-Decorator]] — `functools.lru_cache`, `wraps`
- [[16-Type-hints-va-typing]] — dataclass & Pydantic dùng annotation
