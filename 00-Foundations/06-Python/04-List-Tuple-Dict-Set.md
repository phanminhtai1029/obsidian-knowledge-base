---
title: "List · Tuple · Dict · Set — 4 cấu trúc dữ liệu tích hợp"
section: 00-Foundations/06-Python
tags: [python, list, tuple, dict, set, array, data-structure, interview, foundations, fresher]
related:
  - "[[02-Bien-va-Kieu-du-lieu]]"
  - "[[../01-DSA/03-Array]]"
  - "[[../01-DSA/06-Dictionary-Hash-Table]]"
difficulty: ⭐⭐⭐
estimated_time: 35m
source: ["Python Official Docs", "Fluent Python — Luciano Ramalho"]
---

# List · Tuple · Dict · Set

> [!summary] TL;DR
> 4 cấu trúc dữ liệu tích hợp của Python: **list** `[]` (dãy có thứ tự, **mutable**, cho trùng), **tuple** `()` (dãy có thứ tự, **immutable**), **dict** `{k:v}` (cặp **key→value**, key duy nhất & hashable, tra cứu **O(1)**), **set** `{}` (tập **không trùng, không thứ tự**, kiểm tra thành viên O(1)). Câu phỏng vấn kinh điển **"array vs list vs dict"**: Python **không có "array" gốc** như C — `list` là dynamic array, `dict` là hash table; muốn array số học thuần thì dùng module `array` hoặc **numpy**.

---

## 1. Bảng so sánh tổng quát

| | **list** | **tuple** | **dict** | **set** |
|---|----------|-----------|----------|---------|
| Cú pháp | `[1, 2, 3]` | `(1, 2, 3)` | `{"a": 1}` | `{1, 2, 3}` |
| Thứ tự | ✅ có | ✅ có | ✅ (giữ thứ tự chèn, từ 3.7) | ❌ không |
| Mutable | ✅ | ❌ | ✅ | ✅ |
| Cho trùng | ✅ | ✅ | key ❌, value ✅ | ❌ |
| Truy cập | index `lst[0]` | index `t[0]` | key `d["a"]` | không index |
| Tra cứu phần tử | O(n) | O(n) | **O(1)** key | **O(1)** |
| Hashable (làm key/phần tử set)? | ❌ | ✅ (nếu phần tử immutable) | ❌ | ❌ |
| Dùng khi | dãy thay đổi | dãy cố định, trả nhiều giá trị | ánh xạ key→value | khử trùng, phép tập hợp |

---

## 2. ⭐ Câu phỏng vấn: "Array vs List vs Dict khác gì?"

Đây là câu bạn **đã bị hỏi**. Trả lời theo 3 ý:

**(a) Python không có "array" như C.** Trong C/Java, *array* là khối bộ nhớ liền kề, **cố định kích thước, cùng kiểu**. Python `list` là **dynamic array**: tự nới rộng, chứa **kiểu bất kỳ** lẫn lộn. Muốn "array" thật (cùng kiểu, gọn bộ nhớ) → module `array` hoặc **numpy ndarray** (cho tính toán số).

**(b) List vs Dict — khác bản chất truy cập:**

| | list | dict |
|---|------|------|
| Truy cập theo | **vị trí (index 0,1,2…)** | **khóa (key bất kỳ hashable)** |
| Cấu trúc bên dưới | dynamic array | **hash table** |
| Tìm 1 giá trị | O(n) duyệt | **O(1)** trung bình theo key |
| Khi nào dùng | dữ liệu **có thứ tự**, lặp tuần tự | **tra cứu nhanh theo nhãn** (vd `user["email"]`) |

**(c) Câu chốt ghi điểm:** "List trả lời *phần tử thứ i là gì*; dict trả lời *giá trị ứng với khóa k là gì*. List nhanh khi đọc theo index, dict nhanh khi tra theo key nhờ hash table."

> [!info] Liên hệ DSA
> `list` ↔ [[../01-DSA/03-Array|Array]] (dynamic array). `dict`/`set` ↔ [[../01-DSA/06-Dictionary-Hash-Table|Hash Table]] (hash function + collision). `tuple` ↔ array immutable.

```python
import array
arr = array.array("i", [1, 2, 3])   # array số nguyên thuần (cùng kiểu)
# numpy cho tính toán vector hóa:
# import numpy as np; np.array([1,2,3]) * 2  →  [2,4,6]
```

---

## 3. List — thao tác hay dùng

```python
lst = [3, 1, 2]
lst.append(4)        # thêm cuối           → [3,1,2,4]
lst.insert(0, 9)     # chèn vị trí 0 (O(n))→ [9,3,1,2,4]
lst.pop()            # lấy & xóa cuối O(1)
lst.remove(1)        # xóa giá trị 1 đầu tiên
lst.sort()           # sắp xếp tại chỗ
lst[1:3]             # slicing → list con
[x*2 for x in lst]   # comprehension → [18-Internals...] xem [[08-Comprehension-va-Generator-expression]]
```

---

## 4. Tuple — bất biến & unpacking

```python
point = (3, 4)
x, y = point          # unpacking
def minmax(data): return min(data), max(data)   # trả nhiều giá trị = tuple
lo, hi = minmax([5,1,9])
```

Tuple **immutable** → làm **key của dict** được, an toàn khi cần "không ai sửa".

---

## 5. Dict — hash table

```python
user = {"name": "An", "age": 22}
user["email"] = "a@x.com"        # thêm/cập nhật O(1)
user.get("phone", "N/A")         # tra an toàn, có default
for k, v in user.items(): ...    # duyệt cặp
"name" in user                   # kiểm tra KEY — O(1)
```

Key phải **hashable** (immutable). Từ Python 3.7 dict **giữ thứ tự chèn**.

---

## 6. Set — khử trùng & phép tập hợp

```python
s = {1, 2, 2, 3}        # {1, 2, 3} — tự khử trùng
s.add(4); s.discard(1)
a, b = {1,2,3}, {2,3,4}
a & b                   # giao  {2,3}
a | b                   # hợp   {1,2,3,4}
a - b                   # hiệu  {1}
unique = list(set(my_list))   # khử trùng list nhanh
```

```
★ Insight ─────────────────────────────────────
• "Array vs list vs dict": chìa khóa là TRUY CẬP THEO GÌ — list theo
  index (vị trí), dict theo key (nhãn). dict/set nhanh O(1) vì là
  hash table, đổi lại không có thứ tự index.
• Python list ≠ array C: list là dynamic array chứa kiểu hỗn hợp.
  Cần "array thật" (số, gọn, vector hóa) thì dùng numpy.
• tuple immutable nên hashable → dùng làm key dict / phần tử set khi
  cần khóa ghép (vd toạ độ (x,y)). list không làm được.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. **(câu PV thật)** Array, list, dict khác nhau thế nào? Trả lời theo "truy cập theo gì" + "cấu trúc bên dưới".
2. Vì sao tra cứu dict là O(1) còn tìm trong list là O(n)?
3. Khi nào chọn tuple thay vì list?
4. Python "có array không"? Muốn array số học thật thì dùng gì?
5. Viết 1 dòng khử trùng phần tử trong list.

---

## Liên quan
- [[../01-DSA/03-Array]] — list = dynamic array, vì sao index O(1)
- [[../01-DSA/06-Dictionary-Hash-Table]] — dict/set = hash table, collision
- [[02-Bien-va-Kieu-du-lieu]] — mutable/immutable, hashable
- [[05-String-va-Format]] — str cũng là dãy (immutable)
