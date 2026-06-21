---
title: "Type hints & typing"
section: 00-Foundations/06-Python
tags: [python, type-hints, typing, mypy, pydantic, foundations, fresher]
related:
  - "[[02-Bien-va-Kieu-du-lieu]]"
  - "[[12-OOP]]"
  - "[[19-Thu-vien-chuan]]"
difficulty: ⭐⭐⭐
estimated_time: 25m
source: ["Python Official Docs", "PEP 484"]
---

# Type hints & typing

> [!summary] TL;DR
> **Type hints** (PEP 484) cho phép ghi chú kiểu cho biến/tham số/giá trị trả về: `def f(x: int) -> str`. Python **không ép kiểu lúc chạy** (chỉ là gợi ý) — chúng phục vụ **đọc hiểu, IDE autocomplete, và kiểm tra tĩnh bằng `mypy`/`pyright`**. Module **`typing`** cung cấp kiểu phức: `Optional`, `Union` (`X | Y` từ 3.10), `list[int]`, `dict[str,int]`, `Callable`. **Pydantic/FastAPI** dùng type hints để **validate dữ liệu thật lúc chạy** — lý do chúng phổ biến trong backend.

---

## 1. Cú pháp cơ bản

```python
def add(a: int, b: int) -> int:        # tham số : kiểu, trả về sau ->
    return a + b

name: str = "An"                        # biến có annotation
pi: float = 3.14

def greet(name: str) -> None:           # None = không trả gì
    print(name)
```

> ⚠️ **Không ép kiểu lúc chạy:** `add("a", "b")` vẫn chạy (ra `"ab"`) — hint chỉ là gợi ý. Muốn bắt lỗi cần chạy `mypy file.py`.

---

## 2. typing — kiểu phức

```python
from typing import Optional, Union, Callable, Any

x: list[int]            = [1, 2, 3]          # list các int (3.9+)
y: dict[str, int]       = {"a": 1}
z: tuple[int, str]      = (1, "a")

a: Optional[int] = None       # = int | None  (có thể là int hoặc None)
b: Union[int, str] = 5        # = int | str   (3.10+: int | str)
f: Callable[[int], str]       # hàm nhận int trả str
g: Any                        # tắt kiểm tra kiểu
```

| Hint | Nghĩa |
|------|-------|
| `Optional[X]` | `X` hoặc `None` |
| `Union[X, Y]` / `X \| Y` | một trong các kiểu |
| `list[X]`, `dict[K,V]` | container có kiểu phần tử |
| `Callable[[Args], Ret]` | kiểu hàm |
| `Any` | bất kỳ (bỏ kiểm tra) |

---

## 3. Vì sao quan trọng (FastAPI / Pydantic)

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int
    email: str | None = None

User(name="An", age="22")    # Pydantic TỰ ép "22" → 22, validate kiểu
User(name="An", age="x")     # ❌ ValidationError lúc chạy
```

FastAPI đọc type hints để: **validate request body, sinh tài liệu OpenAPI tự động, autocomplete**. Đây là lý do type hints không chỉ "cho đẹp".

> [!question] Phỏng vấn: "Type hints có ép kiểu lúc chạy không?"
> **Không** — bản thân Python bỏ qua hint khi chạy (vẫn dynamic typing). Hint phục vụ **người đọc, IDE, và static checker (`mypy`/`pyright`)**. Tuy nhiên các thư viện như **Pydantic** *đọc* hint để **tự validate/ép kiểu lúc chạy** — đó là cơ chế riêng của Pydantic, không phải của Python.

```
★ Insight ─────────────────────────────────────
• Type hints là 'gradual typing': thêm dần, không bắt buộc, không đổi
  hành vi runtime. Lợi ích lớn nhất đến từ mypy + IDE chứ không phải
  Python tự kiểm.
• Pydantic/FastAPI 'mượn' hint để validate runtime — bắc cầu giữa
  dynamic typing linh hoạt và an toàn kiểu của backend. Nắm cái này
  giúp hiểu vì sao FastAPI dùng hint khắp nơi.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. Type hint có làm Python ép kiểu/báo lỗi lúc chạy không? Ai kiểm?
2. `Optional[int]` nghĩa là gì? Viết kiểu "int hoặc str".
3. Vì sao FastAPI/Pydantic cần type hints?
4. Khi nào dùng `Any`, và rủi ro của nó?

---

## Liên quan
- [[12-OOP]] — Pydantic `BaseModel`, dataclass dùng annotation
- [[19-Thu-vien-chuan]] — `dataclasses` sinh code từ annotation
- [[02-Bien-va-Kieu-du-lieu]] — dynamic typing nền dưới
