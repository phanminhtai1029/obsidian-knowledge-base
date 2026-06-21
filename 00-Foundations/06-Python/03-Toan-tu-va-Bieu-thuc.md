---
title: "Toán tử & Biểu thức"
section: 00-Foundations/06-Python
tags: [python, operators, is-vs-equals, truthy, membership, foundations, fresher]
related:
  - "[[02-Bien-va-Kieu-du-lieu]]"
  - "[[04-List-Tuple-Dict-Set]]"
difficulty: ⭐⭐
estimated_time: 20m
source: ["Python Official Docs"]
---

# Toán tử & Biểu thức

> [!summary] TL;DR
> Python có toán tử **số học** (`+ - * / // % **`), **so sánh** (`== != < > <= >=`), **logic** (`and or not`), **gán** (`= += …`), **membership** (`in`, `not in`), và **identity** (`is`, `is not`). Hai cặp dễ bẫy: **`==` so sánh GIÁ TRỊ**, **`is` so sánh ĐỊNH DANH object (id)**; và mọi object đều có giá trị **truthy/falsy** dùng trong `if`. `and`/`or` **short-circuit** và trả về **toán hạng** (không phải bool).

---

## 1. Toán tử số học

| Toán tử | Nghĩa | Ví dụ | Kết quả |
|---------|-------|-------|---------|
| `/` | chia **thực** | `7 / 2` | `3.5` |
| `//` | chia **lấy nguyên** (floor) | `7 // 2` | `3` |
| `%` | chia lấy **dư** (modulo) | `7 % 2` | `1` |
| `**` | lũy thừa | `2 ** 10` | `1024` |

> `//` làm tròn xuống (floor): `-7 // 2 == -4` (không phải -3).

---

## 2. ⭐ `==` vs `is`

| | `==` | `is` |
|---|------|------|
| So sánh | **giá trị** (gọi `__eq__`) | **định danh** (cùng object? cùng `id()`?) |
| Dùng khi | so nội dung | so với **None**, singleton |

```python
a = [1, 2, 3]
b = [1, 2, 3]
a == b     # True  — cùng giá trị
a is b     # False — hai object khác nhau trong bộ nhớ

x = None
x is None  # ✅ đúng cách kiểm tra None
```

> [!warning] Bẫy: `is` với số nhỏ
> `256 is 256` → `True` nhưng `1000 is 1000` có thể `False`. CPython **cache số nguyên nhỏ (-5..256)** nên chúng dùng chung object. ĐỪNG dùng `is` để so số/chuỗi — chỉ dùng `==`. Chỉ dùng `is` cho `None`/`True`/`False`.

> [!question] Phỏng vấn: "Khi nào dùng `is`, khi nào `==`?"
> `==` so **giá trị** (hầu hết trường hợp). `is` so **có phải cùng một object không** — dùng cho `x is None`, hoặc kiểm tra singleton. Lý do `x is None` được khuyên: `None` là object duy nhất, `is` nhanh và không bị `__eq__` ghi đè đánh lừa.

---

## 3. Toán tử logic & short-circuit

`and`, `or`, `not` — **short-circuit** (dừng sớm) và trả về **toán hạng**, không phải bool:

```python
True and "hi"     # "hi"   (and trả vế cuối nếu vế trái truthy)
0 or "default"    # "default"  (or trả vế đầu truthy)
name = user_input or "Khách"   # mẹo gán mặc định
```

- `a and b`: nếu `a` falsy → trả `a` (không xét `b`); ngược lại trả `b`.
- `a or b`: nếu `a` truthy → trả `a`; ngược lại trả `b`.

---

## 4. Truthy & Falsy

Trong `if`/`while`, mọi object tự quy về bool. **Falsy**:

```python
False, None, 0, 0.0, 0j, "", [], {}, set(), ()   # tất cả là falsy
# mọi thứ khác là truthy (kể cả "0", [0], " ")
```

```python
items = []
if items:                  # rỗng → falsy → không vào
    print("có hàng")
else:
    print("rỗng")          # in cái này
```

> [!tip] Pythonic: đừng viết `if len(items) > 0:`
> Viết thẳng `if items:` — ngắn, rõ, đúng tinh thần Zen of Python.

---

## 5. Membership & độ ưu tiên

```python
3 in [1, 2, 3]       # True
"a" in {"a": 1}      # True  — kiểm tra KEY của dict
"x" not in "text"    # False

# chuỗi so sánh (chaining) — rất Pythonic
if 0 <= age < 18:    # tương đương (0<=age) and (age<18)
    print("vị thành niên")
```

```
★ Insight ─────────────────────────────────────
• "==" hỏi 'giá trị bằng nhau?', "is" hỏi 'cùng một vật?'. Nhầm 2 cái
  này gây bug khó tìm vì với số nhỏ/chuỗi ngắn 'is' may mắn đúng.
• and/or trả TOÁN HẠNG chứ không phải True/False → tận dụng làm giá
  trị mặc định: `x = a or default`.
• "if items:" thay cho "if len(items)>0:" — nhỏ nhưng là dấu hiệu
  người viết hiểu truthy/falsy, ghi điểm khi review code.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. `7 // 2`, `7 % 2`, `2 ** 5` ra gì?
2. `==` khác `is` thế nào? Khi nào bắt buộc dùng `is`?
3. Vì sao `1000 is 1000` có thể là `False`?
4. Liệt kê các giá trị falsy. `if items:` nghĩa là gì khi `items=[]`?
5. `0 or "x"` và `5 and "y"` trả gì?

---

## Liên quan
- [[02-Bien-va-Kieu-du-lieu]] — vì sao `is` liên quan tới `id()`/tham chiếu
- [[04-List-Tuple-Dict-Set]] — `in` trên list/dict/set
- [[06-Dieu-khien-luong]] — truthy/falsy trong if/while
