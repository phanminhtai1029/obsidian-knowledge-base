---
title: "Array — Mảng"
section: 00-Foundations/01-DSA
tags: [dsa, array, data-structure, random-access, foundations, fresher]
module: L1_AL_NLS_DSAA
related:
  - "[[02-Do-phuc-tap-Big-O]]"
  - "[[04-Linked-List]]"
difficulty: ⭐⭐
estimated_time: 25m
source: ["Programming Foundations: Algorithms — Joe Marini (LinkedIn Learning)"]
---

# Array — Mảng

> [!summary] TL;DR
> **Array** = tập phần tử mà **vị trí mỗi phần tử xác định bằng một index/key**, thường lưu trong **một khối bộ nhớ liền kề (contiguous)**. Nhờ liền kề + index, có thể tính địa chỉ phần tử bằng công thức toán → **truy cập trực tiếp (random access) O(1)**. Index thường bắt đầu từ **0**. Array có thể nhiều chiều (2D = mảng của mảng). Điểm yếu: **chèn/xóa ở đầu hoặc giữa là O(n)** vì các phần tử sau phải dịch chỗ.

---

## 1. Array là gì?

Một **array** là tập phần tử tuyến tính, mỗi phần tử có **index**:

```
index:   0    1    2    3    4
       ┌────┬────┬────┬────┬────┐
       │ 15 │ 23 │ 4  │ 42 │ 16 │
       └────┴────┴────┴────┴────┘
```

Vì các phần tử nằm **liền kề** trong bộ nhớ và **cùng kích thước**, máy tính tính được địa chỉ phần tử `i` bằng công thức:

> `địa_chỉ(i) = địa_chỉ_gốc + i × kích_thước_phần_tử`

→ Đây là lý do truy cập `arr[i]` là **O(1)** (random access): không cần duyệt, tính thẳng ra.

> [!example] Truy cập theo công thức
> Lấy các phần tử **chỉ số chẵn**: `arr[n*2]` cho ra phần tử 0, 2, 4…

---

## 2. Mảng nhiều chiều (2D)

Mảng 2D = **mảng chứa các mảng**. Truy cập cần **2 index**:

```
            cột 0  cột 1
hàng 0  →  [ a ,   b  ]
hàng 1  →  [ c ,   d  ]
hàng 2  →  [ e ,   f  ]   ← arr[2][1] = f
```

`arr[2][1]`: index 2 cho chiều thứ nhất (hàng), index 1 cho chiều thứ hai (cột).

### Jagged array (mảng răng cưa)
Mảng 2D thường có mọi hàng **cùng độ dài** (hình chữ nhật). **Jagged array** (ragged array) = mảng của các mảng có **độ dài khác nhau**:

```
hàng 0  →  [ a, b, c ]
hàng 1  →  [ d ]
hàng 2  →  [ e, f ]
```

Trong Python mọi list lồng nhau đều "jagged được" tự nhiên (`[[1,2,3],[4],[5,6]]`). Một số ngôn ngữ (C#, Java) phân biệt rõ **rectangular array** (`int[,]`) vs **jagged array** (`int[][]`). Dùng jagged khi mỗi hàng có số phần tử khác nhau → tiết kiệm bộ nhớ so với mảng chữ nhật phải đệm cho đủ.

---

## 3. Big-O của Array

| Thao tác | Big-O | Vì sao |
|----------|-------|--------|
| Truy cập `arr[i]` | **O(1)** | Tính địa chỉ bằng công thức, không cần duyệt |
| Chèn/xóa ở **cuối** | **O(1)** | Không phải dịch phần tử nào (nếu còn chỗ) |
| Chèn/xóa ở **đầu** | **O(n)** | Mọi phần tử sau phải dịch sang vị trí mới |
| Chèn/xóa ở **giữa** | **O(n)** | Các phần tử còn lại phải dịch chỗ |
| Tìm giá trị (chưa sort) | **O(n)** | Phải duyệt từng phần tử → [[13-Searching]] |

> [!question] Phỏng vấn: "Tại sao truy cập array O(1) nhưng chèn đầu O(n)?"
> Truy cập chỉ là **một phép tính địa chỉ** (random access) → O(1). Nhưng chèn vào đầu buộc **tất cả phần tử phía sau dời 1 ô** để nhường chỗ → tỉ lệ với n → O(n). Đây chính là ví dụ "một cấu trúc có nhiều Big-O" ở [[02-Do-phuc-tap-Big-O]].

---

## 4. So với Linked List

| | Array | Linked List |
|---|-------|-------------|
| Bộ nhớ | Liền kề (contiguous) | Rải rác, nối bằng pointer |
| Truy cập phần tử thứ i | **O(1)** ✅ | **O(n)** (phải đi từ head) |
| Chèn/xóa ở đầu | **O(n)** | **O(1)** ✅ |
| Cấp phát | Cần khối liền kề (có thể phải resize) | Linh hoạt, cấp từng node |

→ Chi tiết ở [[04-Linked-List]].

```
★ Insight ─────────────────────────────────────
• "Random access O(1)" là siêu năng lực của array — đến từ việc dữ
  liệu LIỀN KỀ + cùng kích thước nên địa chỉ tính được. Mất tính
  liền kề (như Linked List) là mất luôn O(1).
• Index bắt đầu từ 0 không phải ngẫu nhiên: nó là "khoảng cách
  (offset) so với phần tử đầu". Phần tử đầu cách gốc 0 ô → index 0.
• Đánh đổi cốt lõi của mọi CTDL: array nhanh khi ĐỌC theo vị trí,
  chậm khi THÊM/XÓA giữa chừng. Chọn array khi bạn đọc nhiều, sửa
  cấu trúc ít.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. Vì sao truy cập `arr[i]` là O(1)? Viết công thức tính địa chỉ.
2. Chèn phần tử vào đầu array có Big-O bao nhiêu? Giải thích.
3. Truy cập phần tử `arr[1][2]` trong mảng 2D nghĩa là gì?
4. Khi nào nên chọn Array thay vì Linked List?

---

## Liên quan
- [[04-Linked-List]] — cấu trúc tuyến tính "đối thủ" của array
- [[02-Do-phuc-tap-Big-O]] — vì sao mỗi thao tác một Big-O
- [[13-Searching]] — tìm kiếm trên array
