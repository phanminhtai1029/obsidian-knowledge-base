---
title: "Biến & Kiểu dữ liệu"
section: 00-Foundations/06-Python
tags: [python, variables, data-types, mutable, immutable, foundations, fresher]
related:
  - "[[01-Tong-quan-Python]]"
  - "[[03-Toan-tu-va-Bieu-thuc]]"
  - "[[04-List-Tuple-Dict-Set]]"
difficulty: ⭐⭐
estimated_time: 25m
source: ["Python Official Docs", "Fluent Python — Luciano Ramalho"]
---

# Biến & Kiểu dữ liệu

> [!summary] TL;DR
> Trong Python, **biến là một cái tên trỏ tới object** trong bộ nhớ (không phải "ô chứa giá trị" như C). Gán `=` chỉ là **gắn nhãn**. Kiểu cơ bản: **int, float, complex, bool, str, bytes, NoneType**. Phân biệt sống còn: **mutable** (đổi được tại chỗ: list, dict, set) vs **immutable** (không đổi được: int, float, str, tuple, frozenset, bool). Vì biến là tham chiếu, gán `b = a` với list khiến `a` và `b` cùng trỏ 1 object → sửa cái này thấy ở cái kia.

---

## 1. Biến = cái tên trỏ tới object

```python
x = 5          # 5 là object int, x là nhãn trỏ tới nó
y = x          # y trỏ TỚI CÙNG object 5
```

Không có "khai báo kiểu". Gán lại tên có thể đổi sang kiểu khác (dynamic typing → [[01-Tong-quan-Python]]):

```python
x = 5
x = "hello"    # hợp lệ — x giờ trỏ object str
```

> [!info] `id()` và `type()`
> `id(obj)` trả địa chỉ định danh object; `type(obj)` trả kiểu; `isinstance(obj, int)` kiểm tra kiểu (ưu tiên hơn `type(x)==int` vì hiểu kế thừa).

---

## 2. Các kiểu dữ liệu cơ bản

| Kiểu | Ví dụ | Mutable? | Ghi chú |
|------|-------|----------|---------|
| `int` | `42`, `-7` | ❌ | số nguyên, **không giới hạn độ lớn** |
| `float` | `3.14` | ❌ | dấu phẩy động 64-bit |
| `complex` | `2+3j` | ❌ | số phức |
| `bool` | `True`/`False` | ❌ | **là con của `int`** (`True==1`) |
| `str` | `"hi"` | ❌ | chuỗi Unicode → [[05-String-va-Format]] |
| `bytes` | `b"hi"` | ❌ | dữ liệu nhị phân |
| `NoneType` | `None` | ❌ | "không có giá trị" (1 object duy nhất) |
| `list` | `[1,2]` | ✅ | → [[04-List-Tuple-Dict-Set]] |
| `dict` | `{"a":1}` | ✅ | → [[04-List-Tuple-Dict-Set]] |
| `set` | `{1,2}` | ✅ | → [[04-List-Tuple-Dict-Set]] |
| `tuple` | `(1,2)` | ❌ | → [[04-List-Tuple-Dict-Set]] |

---

## 3. ⭐ Mutable vs Immutable

Đây là **khái niệm cốt lõi** gây bug nhiều nhất:

| | Immutable | Mutable |
|---|-----------|---------|
| Gồm | int, float, bool, str, tuple, frozenset, bytes | list, dict, set, bytearray |
| Đổi tại chỗ? | ❌ tạo object mới | ✅ sửa ngay object cũ |
| Làm key của dict? | ✅ được (hashable) | ❌ không (unhashable) |

```python
# immutable: "sửa" thực ra tạo object mới
s = "abc"
print(id(s))
s += "d"          # tạo str MỚI "abcd", id đổi
print(id(s))      # khác id cũ

# mutable: sửa tại chỗ, id giữ nguyên
lst = [1, 2]
print(id(lst))
lst.append(3)     # sửa chính object đó
print(id(lst))    # id KHÔNG đổi
```

---

## 4. Bẫy tham chiếu (aliasing)

Vì biến là tham chiếu, gán list không **copy**:

```python
a = [1, 2, 3]
b = a             # b trỏ CÙNG list với a
b.append(4)
print(a)          # [1, 2, 3, 4]  ← a cũng đổi!

# muốn bản sao thật:
c = a.copy()      # hoặc a[:] hoặc list(a)  → shallow copy
```

> [!question] Phỏng vấn: "Gán `b = a` rồi `b.append(4)`, vì sao `a` cũng đổi?"
> Vì `a` và `b` là **hai cái tên trỏ tới CÙNG một list object**. `append` sửa tại chỗ object đó → cả hai nhãn đều thấy. Với immutable (int, str) thì không gặp vì mọi "thay đổi" tạo object mới. Copy sâu hơn dùng `copy.deepcopy` → [[18-Internals-GIL-GC]].

---

## 5. Ép kiểu (type casting)

```python
int("42")      # 42      str → int
str(42)        # "42"    int → str
float("3.14")  # 3.14
list("abc")    # ['a','b','c']
bool(0)        # False   (0, "", [], None, {} đều falsy → [[03-Toan-tu-va-Bieu-thuc]])
```

```
★ Insight ─────────────────────────────────────
• "Biến là nhãn, không phải hộp." Hiểu điều này giải thích mọi bug
  aliasing, mutable default argument, và copy vs deepcopy.
• Immutable = hashable = dùng được làm key dict / phần tử set. Đó là
  lý do tuple làm key được còn list thì không.
• bool là con của int → True+True==2. Mẹo nhưng dễ thành bug khi đếm.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. "Biến trong Python là gì" — giải thích bằng khái niệm nhãn/tham chiếu.
2. Kể 4 kiểu immutable và 3 kiểu mutable.
3. Vì sao list không làm key của dict được còn tuple thì được?
4. `a=[1,2]; b=a; b.append(3)` — `a` ra gì? Làm sao để `b` độc lập?

---

## Liên quan
- [[04-List-Tuple-Dict-Set]] — chi tiết 4 cấu trúc dữ liệu tích hợp
- [[03-Toan-tu-va-Bieu-thuc]] — `is` vs `==`, truthy/falsy
- [[18-Internals-GIL-GC]] — copy vs deepcopy, reference counting
