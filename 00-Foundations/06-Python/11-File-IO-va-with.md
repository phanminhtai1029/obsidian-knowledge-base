---
title: "File I/O & with"
section: 00-Foundations/06-Python
tags: [python, file-io, context-manager, with, foundations, fresher]
related:
  - "[[10-Xu-ly-ngoai-le]]"
  - "[[15-Functional-va-Context-manager]]"
difficulty: ⭐⭐
estimated_time: 20m
source: ["Python Official Docs"]
---

# File I/O & with

> [!summary] TL;DR
> Mở file bằng **`open(path, mode)`**, đọc/ghi rồi **phải đóng**. Cách chuẩn là dùng **`with open(...) as f:`** — một **context manager** tự động đóng file kể cả khi có exception (thay cho `try/finally`). Mode hay dùng: `r` đọc, `w` ghi đè, `a` thêm, `b` nhị phân. Với file văn bản nên chỉ rõ `encoding="utf-8"`. Đọc dòng-một-dòng (`for line in f`) để xử lý file lớn không tốn RAM.

---

## 1. `with` — cách chuẩn mở file

```python
with open("data.txt", "r", encoding="utf-8") as f:
    content = f.read()           # đọc toàn bộ thành 1 chuỗi
# ra khỏi with → file TỰ ĐỘNG đóng (kể cả khi lỗi)
```

So sánh với cách thủ công (dễ quên `close`):

```python
f = open("data.txt")
try:
    content = f.read()
finally:
    f.close()                    # với 'with' thì khỏi cần
```

> [!question] Phỏng vấn: "Vì sao dùng `with open(...)` thay vì `open()` rồi `close()`?"
> `with` là **context manager**: tự gọi `f.close()` khi rời khối, **kể cả khi có exception**. Tránh rò rỉ tài nguyên (file handle) do quên đóng hoặc lỗi giữa chừng. Tương đương `try/finally` nhưng gọn và an toàn hơn. (Tự viết context manager → [[15-Functional-va-Context-manager]].)

---

## 2. Các mode

| Mode | Nghĩa | File chưa tồn tại |
|------|-------|-------------------|
| `r` | đọc (mặc định) | ❌ FileNotFoundError |
| `w` | ghi, **xóa sạch** nội dung cũ | ✅ tạo mới |
| `a` | ghi **nối thêm** cuối | ✅ tạo mới |
| `r+` | đọc & ghi | ❌ |
| `x` | tạo mới, lỗi nếu đã có | ✅ |
| `+b` | thêm `b` → nhị phân (ảnh, file) | |

---

## 3. Đọc & ghi

```python
# đọc cả file
with open("f.txt", encoding="utf-8") as f:
    text = f.read()           # 1 chuỗi
    # f.readlines()           # list các dòng

# đọc DÒNG MỘT DÒNG — không tốn RAM cho file lớn
with open("big.log", encoding="utf-8") as f:
    for line in f:            # lazy, từng dòng
        process(line.rstrip())

# ghi
with open("out.txt", "w", encoding="utf-8") as f:
    f.write("dòng 1\n")
    f.writelines(["a\n", "b\n"])
```

> [!tip] Đọc file lớn
> `f.read()` nạp **toàn bộ** vào RAM. File log/CSV lớn → duyệt `for line in f` (lazy, từng dòng) hoặc đọc theo khối.

---

## 4. Đường dẫn với `pathlib` (hiện đại)

```python
from pathlib import Path
p = Path("data") / "input.txt"      # ghép path đa nền tảng
p.exists(); p.suffix; p.stem
text = p.read_text(encoding="utf-8")   # đọc nhanh không cần with
```

```
★ Insight ─────────────────────────────────────
• 'with' là cú pháp đường-vào/đường-ra (RAII của Python): vào thì mở,
  ra thì dọn — dù bằng return hay exception. Không chỉ cho file mà
  cho lock, DB connection, transaction.
• Đọc file lớn: 'for line in f' lazy từng dòng, KHÔNG f.read() cả cục.
  Cùng tinh thần generator ở [[13-Iterator-va-Generator]].
• Luôn nêu encoding='utf-8' cho file văn bản — tránh lỗi tiếng Việt
  do encoding mặc định khác nhau giữa OS.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. Vì sao `with open(...)` an toàn hơn `open()` + `close()`?
2. Khác nhau mode `w` và `a`?
3. Đọc file rất lớn nên dùng cách nào để khỏi tốn RAM?
4. `with` có chỉ dùng cho file không? (gợi ý: context manager)

---

## Liên quan
- [[10-Xu-ly-ngoai-le]] — `with` thay thế try/finally
- [[15-Functional-va-Context-manager]] — tự viết context manager
- [[13-Iterator-va-Generator]] — đọc dòng-một-dòng là lazy iteration
