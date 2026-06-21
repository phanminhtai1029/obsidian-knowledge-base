---
title: "Module · Package · pip · venv"
section: 00-Foundations/06-Python
tags: [python, module, package, import, pip, virtualenv, foundations, fresher]
related:
  - "[[01-Tong-quan-Python]]"
  - "[[19-Thu-vien-chuan]]"
difficulty: ⭐⭐
estimated_time: 25m
source: ["Python Official Docs"]
---

# Module · Package · pip · venv

> [!summary] TL;DR
> **Module** = một file `.py`. **Package** = thư mục chứa nhiều module (thường có `__init__.py`). Dùng lại code bằng **`import`**. Biến đặc biệt **`__name__`** = `"__main__"` khi file chạy trực tiếp → mẫu `if __name__ == "__main__":`. Cài thư viện ngoài bằng **`pip`** (từ PyPI). Mỗi dự án nên có **virtual environment (venv)** riêng để cô lập dependency, kèm **`requirements.txt`** ghi danh sách gói.

---

## 1. Module & import

```python
# file mymath.py
def add(a, b): return a + b
PI = 3.14159
```

```python
import mymath                  # dùng: mymath.add(1,2)
from mymath import add, PI     # dùng thẳng: add(1,2)
from mymath import add as cong # đổi tên
import numpy as np             # alias quy ước
# from mymath import *         # ⚠️ tránh — ô nhiễm namespace
```

Python tìm module theo `sys.path` (thư mục hiện tại → site-packages → built-in).

---

## 2. `if __name__ == "__main__"`

```python
def main():
    print("chạy chương trình")

if __name__ == "__main__":     # chỉ chạy khi file được CHẠY TRỰC TIẾP
    main()                     # KHÔNG chạy khi file bị import
```

| Tình huống | `__name__` bằng |
|------------|-----------------|
| Chạy trực tiếp `python file.py` | `"__main__"` |
| Bị file khác `import` | tên module (`"file"`) |

> [!question] Phỏng vấn: "`if __name__ == '__main__'` để làm gì?"
> Để phần code (test/demo/entry point) **chỉ chạy khi file được thực thi trực tiếp**, chứ **không chạy khi file bị import** làm module. Nhờ đó một file vừa dùng như thư viện vừa chạy độc lập được.

---

## 3. Package

```
mypackage/
├── __init__.py        # đánh dấu package (có thể rỗng)
├── core.py
└── utils.py
```

```python
from mypackage import core
from mypackage.utils import helper
```

`__init__.py` chạy khi package được import — nơi gom export công khai.

---

## 4. pip — cài thư viện ngoài

```bash
pip install requests            # cài gói từ PyPI
pip install "fastapi==0.110.0"  # ghim phiên bản
pip uninstall requests
pip list                        # liệt kê đã cài
pip freeze > requirements.txt   # xuất danh sách gói + version
pip install -r requirements.txt # cài lại từ file
```

---

## 5. ⭐ Virtual environment (venv)

Mỗi dự án **cô lập gói riêng**, tránh xung đột version giữa các dự án:

```bash
python -m venv .venv            # tạo môi trường ảo trong .venv/
source .venv/bin/activate       # kích hoạt (Linux/macOS)
# .venv\Scripts\activate        # (Windows)
pip install fastapi             # cài VÀO môi trường này
deactivate                      # thoát
```

> [!question] Phỏng vấn: "Vì sao cần virtual environment?"
> Để **cô lập dependency theo từng dự án**. Dự án A cần Django 3, dự án B cần Django 5 — nếu cài chung toàn cục sẽ xung đột. venv cho mỗi dự án bộ gói riêng, tái lập được qua `requirements.txt`. (Công cụ mới hơn: `poetry`, `pipenv`, `uv`.)

```
★ Insight ─────────────────────────────────────
• Module = 1 file, Package = thư mục module. import là cách Python
  tái sử dụng & tổ chức code thành namespace.
• `if __name__=='__main__'` là "công tắc" tách phần chạy-trực-tiếp
  khỏi phần được-import — gặp trong gần như mọi script Python.
• venv + requirements.txt = tái lập môi trường. Không có nó, "máy tôi
  chạy được mà" thành ác mộng khi deploy.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. Module khác Package thế nào? `__init__.py` để làm gì?
2. `if __name__ == "__main__":` giải quyết vấn đề gì?
3. Lệnh xuất & cài lại danh sách gói là gì?
4. Vì sao mỗi dự án nên có virtual environment riêng?

---

## Liên quan
- [[19-Thu-vien-chuan]] — module chuẩn có sẵn (không cần pip)
- [[01-Tong-quan-Python]] — pip/PyPI trong hệ sinh thái Python
