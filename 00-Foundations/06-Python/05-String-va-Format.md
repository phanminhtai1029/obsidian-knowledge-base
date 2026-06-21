---
title: "String & f-string"
section: 00-Foundations/06-Python
tags: [python, string, f-string, slicing, immutable, foundations, fresher]
related:
  - "[[02-Bien-va-Kieu-du-lieu]]"
  - "[[04-List-Tuple-Dict-Set]]"
difficulty: ⭐⭐
estimated_time: 20m
source: ["Python Official Docs"]
---

# String & f-string

> [!summary] TL;DR
> `str` là **dãy ký tự Unicode, immutable** — mọi "thay đổi" tạo chuỗi mới. Hỗ trợ **indexing & slicing** như list (`s[0]`, `s[1:4]`, `s[::-1]` đảo ngược). Nhiều method hữu ích (`.strip() .split() .join() .replace() .lower()`). Cách format hiện đại & nhanh nhất là **f-string** (`f"{name=}"`), thay cho `%` cũ và `.format()`.

---

## 1. String là dãy immutable

```python
s = "Python"
s[0]        # 'P'   indexing
s[1:4]      # 'yth' slicing [bắt đầu:kết thúc)
s[::-1]     # 'nohtyP' đảo ngược
len(s)      # 6

s[0] = "J"  # ❌ TypeError — immutable!
s = "J" + s[1:]   # tạo chuỗi MỚI "Jython"
```

Vì immutable → **hashable** → làm key dict được ([[04-List-Tuple-Dict-Set]]).

---

## 2. f-string (Python 3.6+) — cách format chuẩn

```python
name, age = "An", 22
f"Tôi là {name}, {age} tuổi"        # nhúng biểu thức trong {}
f"{age + 1}"                         # tính ngay: '23'
f"{3.14159:.2f}"                     # định dạng: '3.14'
f"{1000000:,}"                       # phân tách nghìn: '1,000,000'
f"{name=}"                           # debug: "name='An'" (3.8+)
f"{'x':>10}"                         # căn phải rộng 10
```

| Cách | Ví dụ | Đánh giá |
|------|-------|----------|
| `%` (cũ) | `"%s %d" % (name, age)` | kiểu C, tránh dùng |
| `.format()` | `"{} {}".format(name, age)` | dài dòng |
| **f-string** | `f"{name} {age}"` | ✅ ngắn, nhanh nhất, dễ đọc |

---

## 3. Method hay dùng

```python
"  hi  ".strip()           # 'hi'        bỏ khoảng trắng 2 đầu
"a,b,c".split(",")          # ['a','b','c']
"-".join(["a","b","c"])     # 'a-b-c'     ngược của split
"Hello".lower() / .upper()  # 'hello' / 'HELLO'
"hello".replace("l", "L")   # 'heLLo'
"hello".startswith("he")    # True
"abc123".isdigit()          # False
"text".find("x")            # 2  (vị trí, -1 nếu không có)
```

> [!tip] `join` để nối list chuỗi, đừng `+=` trong vòng lặp
> Nối chuỗi bằng `+=` trong loop là **O(n²)** (mỗi lần tạo chuỗi mới). Dùng `"".join(list_chuỗi)` → **O(n)**.

> [!question] Phỏng vấn: "String mutable hay immutable? Hệ quả?"
> **Immutable.** Hệ quả: (1) sửa = tạo chuỗi mới → nối nhiều nên dùng `join` thay `+=`; (2) string **hashable** nên làm key dict/phần tử set; (3) an toàn khi truyền đi (không ai sửa trộm).

```
★ Insight ─────────────────────────────────────
• String là "list ký tự nhưng khóa lại" — slicing y hệt list nhưng
  không gán phần tử được. s[::-1] đảo chuỗi là mẹo kinh điển.
• f-string vừa nhanh vừa rõ; `f"{var=}"` là vũ khí debug nhanh.
• Nối chuỗi trong loop bằng += là bẫy O(n²) — join là câu trả lời.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. String mutable hay immutable? Nêu 2 hệ quả.
2. `s[::-1]` làm gì? `s[1:4]` lấy ký tự nào?
3. Vì sao nên dùng `"".join(...)` thay vì `+=` trong vòng lặp?
4. Viết f-string in số `3.14159` với 2 chữ số thập phân.

---

## Liên quan
- [[02-Bien-va-Kieu-du-lieu]] — immutable & hashable
- [[04-List-Tuple-Dict-Set]] — string cũng là một sequence
