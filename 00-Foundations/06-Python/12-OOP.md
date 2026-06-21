---
title: "OOP trong Python"
section: 00-Foundations/06-Python
tags: [python, oop, class, inheritance, polymorphism, dunder, property, foundations, fresher]
related:
  - "[[02-Bien-va-Kieu-du-lieu]]"
  - "[[16-Type-hints-va-typing]]"
  - "[[19-Thu-vien-chuan]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 40m
source: ["Python Official Docs", "Fluent Python — Luciano Ramalho"]
---

# OOP trong Python

> [!summary] TL;DR
> Lập trình hướng đối tượng gom **dữ liệu (attribute)** + **hành vi (method)** vào **class**; tạo **object (instance)** từ class. `__init__` là **constructor**, `self` trỏ tới instance hiện tại. 4 trụ cột: **Encapsulation** (đóng gói, quy ước `_`/`__`), **Inheritance** (kế thừa), **Polymorphism** (đa hình — duck typing), **Abstraction**. **Dunder methods** (`__str__`, `__eq__`, `__len__`…) cho phép object "hòa nhập" cú pháp Python. `@property` biến method thành thuộc tính; `@classmethod`/`@staticmethod` đổi cách nhận tham số.

---

## 1. Class, instance, `self`

```python
class Dog:
    species = "Canis"                  # class attribute (dùng chung mọi instance)

    def __init__(self, name, age):     # constructor, chạy khi tạo object
        self.name = name               # instance attribute (riêng mỗi object)
        self.age = age

    def bark(self):                    # method — luôn có self đầu tiên
        return f"{self.name} sủa!"

d = Dog("Rex", 3)        # tạo instance → __init__ chạy
d.bark()                 # 'Rex sủa!'
d.name                   # 'Rex'   (instance attr)
Dog.species              # 'Canis' (class attr)
```

| | Instance attribute | Class attribute |
|---|--------------------|-----------------|
| Khai báo | `self.x = …` trong `__init__` | ngay trong thân class |
| Thuộc về | từng object riêng | dùng chung mọi instance |

> `self` **không phải từ khóa** — chỉ là quy ước tên tham số đầu (trỏ instance gọi method).

---

## 2. Encapsulation — quy ước truy cập

Python **không có `private` thật**, dùng quy ước đặt tên:

```python
class Account:
    def __init__(self):
        self.owner = "An"       # public
        self._balance = 0       # "protected" (quy ước: đừng đụng từ ngoài)
        self.__pin = "1234"     # name mangling → _Account__pin (khó đụng)
```

- `_x`: quy ước "nội bộ", không ép buộc.
- `__x`: **name mangling** → đổi tên thành `_Class__x`, tránh đụng độ khi kế thừa.

---

## 3. `@property` — getter/setter Pythonic

```python
class Circle:
    def __init__(self, r):
        self._r = r

    @property
    def area(self):              # gọi như THUỘC TÍNH, không phải method
        return 3.14159 * self._r ** 2

    @property
    def radius(self):
        return self._r

    @radius.setter
    def radius(self, value):
        if value < 0:
            raise ValueError("bán kính âm")
        self._r = value

c = Circle(5)
c.area          # 78.5… — không cần c.area()
c.radius = 10   # gọi setter (có validate)
```

---

## 4. Inheritance & super()

```python
class Animal:
    def __init__(self, name):
        self.name = name
    def speak(self):
        return "..."

class Cat(Animal):                 # Cat kế thừa Animal
    def __init__(self, name, color):
        super().__init__(name)     # gọi __init__ lớp cha
        self.color = color
    def speak(self):               # OVERRIDE — đa hình
        return "Meo"

Cat("Kit", "đen").speak()          # 'Meo'
isinstance(Cat("x","y"), Animal)   # True
```

Python hỗ trợ **đa kế thừa**; thứ tự tra method theo **MRO** (Method Resolution Order, `Cls.__mro__`).

---

## 5. Polymorphism & Duck typing

> "If it walks like a duck and quacks like a duck, it's a duck."

Python không cần cùng lớp cha — chỉ cần **có cùng method**:

```python
def make_speak(x):
    return x.speak()           # hoạt động với MỌI object có .speak()

make_speak(Cat("a","b"))       # 'Meo'
make_speak(Dog("Rex", 1))      # cũng được nếu Dog có .speak()
```

---

## 6. Dunder methods (magic methods)

Cho object tích hợp cú pháp Python:

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y
    def __repr__(self):                 # hiện khi debug/repr
        return f"Vector({self.x}, {self.y})"
    def __str__(self):                  # hiện khi print()
        return f"({self.x}, {self.y})"
    def __eq__(self, other):            # cho ==
        return self.x == other.x and self.y == other.y
    def __add__(self, other):           # cho +
        return Vector(self.x + other.x, self.y + other.y)
    def __len__(self):                  # cho len()
        return 2

v = Vector(1, 2) + Vector(3, 4)   # Vector(4, 6) — nhờ __add__
print(v)                          # (4, 6)        — nhờ __str__
```

| Dunder | Kích hoạt bởi |
|--------|---------------|
| `__init__` | tạo object |
| `__str__` / `__repr__` | `print()` / `repr()` |
| `__eq__`, `__lt__` | `==`, `<` |
| `__add__`, `__mul__` | `+`, `*` |
| `__len__`, `__getitem__` | `len()`, `obj[i]` |

> [!question] Phỏng vấn: "`@classmethod` khác `@staticmethod` khác method thường?"
> **Method thường**: nhận `self` (instance). **`@classmethod`**: nhận `cls` (chính class) — hay dùng làm **factory** (`from_string`). **`@staticmethod`**: không nhận gì — chỉ là hàm tiện ích gom trong class. Xem dưới.

```python
class Pizza:
    def __init__(self, size): self.size = size

    @classmethod
    def family(cls):                 # factory, nhận cls
        return cls(size="L")

    @staticmethod
    def is_valid(size):              # tiện ích, không cần self/cls
        return size in {"S", "M", "L"}
```

```
★ Insight ─────────────────────────────────────
• Python "private" chỉ là quy ước: _x (đừng đụng), __x (name
  mangling). Không có rào chắn cứng như Java — triết lý "we're all
  adults here".
• Dunder methods là cách object NHẬP GIA cú pháp Python: định nghĩa
  __add__ thì dùng được +, __len__ thì len() chạy. Đây là 'protocol'
  thay vì 'interface'.
• Duck typing > kiểm tra kiểu: hàm chỉ cần object 'có method đúng',
  không quan tâm lớp gì. Linh hoạt nhưng cần test kỹ.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. `self` là gì? Instance attribute khác class attribute thế nào?
2. Python có `private` thật không? `_x` vs `__x`?
3. `@property` giải quyết vấn đề gì?
4. `@classmethod` vs `@staticmethod` vs method thường — khác nhau ở tham số đầu?
5. Kể 4 dunder methods và cú pháp chúng kích hoạt.

---

## Liên quan
- [[16-Type-hints-va-typing]] — gắn kiểu cho thuộc tính (dataclass/Pydantic)
- [[19-Thu-vien-chuan]] — `@dataclass` sinh `__init__`/`__eq__` tự động
- [[14-Decorator]] — `@property`/`@classmethod` cũng là decorator
