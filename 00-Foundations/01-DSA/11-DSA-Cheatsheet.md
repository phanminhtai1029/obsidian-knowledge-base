---
title: "DSA Cheatsheet — Big-O, CTDL & Glossary"
section: 00-Foundations/01-DSA
tags: [dsa, cheatsheet, big-o, glossary, reference, foundations, fresher]
module: L1_AL_NLS_DSAA
related:
  - "[[02-Do-phuc-tap-Big-O]]"
  - "[[08-Sorting]]"
  - "[[09-Searching]]"
difficulty: ⭐⭐
estimated_time: 20m
source: ["Programming Foundations: Algorithms — Joe Marini (LinkedIn Learning)"]
---

# DSA Cheatsheet — Tra cứu nhanh

> [!summary] TL;DR
> Trang tổng hợp: bảng **Big-O của mọi CTDL × thao tác**, **Big-O các thuật toán sort/search**, **"khi nào dùng cấu trúc nào"**, snippet Python, và **glossary** thuật ngữ. Dùng để ôn nhanh trước kỳ thi `L1_AL_NLS_DSAA`.

---

## 1. Big-O cấu trúc dữ liệu (thao tác chính)

| Cấu trúc | Truy cập | Tìm kiếm | Chèn | Xóa | Ghi chú |
|----------|----------|----------|------|-----|---------|
| **Array** | **O(1)** | O(n) | O(n) (đầu/giữa), O(1) (cuối) | O(n) | Random access nhờ liền kề |
| **Linked List** | O(n) | O(n) | **O(1)** (head) | **O(1)** (head) | Đổi pointer, không dịch RAM |
| **Stack** | O(n) | O(n) | **O(1)** (push) | **O(1)** (pop) | LIFO |
| **Queue (deque)** | O(n) | O(n) | **O(1)** (enqueue) | **O(1)** (dequeue) | FIFO |
| **Dictionary / Hash** | — | **O(1)**\* | **O(1)**\* | **O(1)**\* | \*trung bình; worst O(n) khi collision |
| **Set** | — | **O(1)**\* | **O(1)**\* | **O(1)**\* | giá trị duy nhất |

---

## 2. Big-O thuật toán Sort & Search

| Thuật toán | Trung bình | Worst | Bộ nhớ phụ | Loại |
|------------|-----------|-------|------------|------|
| **Bubble sort** | O(n²) | O(n²) | O(1) | dạy học |
| **Merge sort** | **O(n log n)** | O(n log n) | **O(n)** | divide & conquer |
| **Quick sort** | **O(n log n)** | O(n²) | O(log n) in-place | divide & conquer |
| **Linear search** | O(n) | O(n) | O(1) | list bất kỳ |
| **Binary search** | **O(log n)** | O(log n) | O(1) | list **đã sort** |

---

## 3. Thang Big-O (tốt → xấu)

| Big-O | Tên | Ví dụ |
|-------|-----|-------|
| **O(1)** | Constant | `arr[i]`, dict lookup |
| **O(log n)** | Logarithmic | Binary search |
| **O(n)** | Linear | Linear search, đếm, lọc |
| **O(n log n)** | Log-linear | Merge/Quick/Heap sort |
| **O(n²)** | Quadratic | Bubble sort, nested loop |

---

## 4. Khi nào dùng cấu trúc nào?

| Cần… | Chọn |
|------|------|
| Đọc phần tử theo index thật nhanh | **Array** |
| Chèn/xóa nhiều ở đầu, không cần index | **Linked List** |
| Vào sau ra trước (back, undo, biểu thức) | **Stack** |
| Vào trước ra trước (hàng đợi, đơn hàng) | **Queue / deque** |
| Tra cứu theo khóa, đếm tần suất | **Dictionary** |
| Khử trùng, kiểm tra "đã có chưa" | **Set** |
| Bài toán phân rã (cây, chia để trị) | **Recursion** |

---

## 5. Snippet Python nhanh

```python
# Stack
st = []; st.append(x); st.pop()

# Queue
from collections import deque
q = deque(); q.append(x); q.popleft()

# Set khử trùng
unique = set(items)

# Dict counter
from collections import Counter
freq = Counter(items)

# Binary search có sẵn
import bisect
i = bisect.bisect_left(sorted_list, x)

# Sort có sẵn (Timsort, O(n log n))
data.sort()                 # in-place
new = sorted(data, reverse=True)
```

---

## 6. Glossary — thuật ngữ

| Thuật ngữ | Nghĩa |
|-----------|-------|
| **Algorithm** | Tập bước giải một bài toán cụ thể |
| **Big-O** | Mô tả hiệu năng theo độ lớn input (order of operation) |
| **Time / Space complexity** | Độ phức tạp về thời gian / bộ nhớ |
| **Constant time** O(1) | Không phụ thuộc số phần tử |
| **Linear time** O(n) | Tỉ lệ thuận với số phần tử |
| **Logarithmic** O(log n) | Tỉ lệ với log số phần tử |
| **Log-linear** O(n log n) | n nhân log n |
| **Quadratic** O(n²) | Tỉ lệ bình phương số phần tử |
| **Random access** | Truy cập trực tiếp phần tử qua index (O(1)) |
| **Node** | Phần tử trong linked list (data + pointer) |
| **Head** | Node đầu của linked list |
| **LIFO / FIFO** | Last-In-First-Out (stack) / First-In-First-Out (queue) |
| **Deque** | Double-ended queue, thêm/lấy 2 đầu O(1) |
| **Hash function** | Hàm biến key → index trong hash table |
| **Collision** | 2 key băm ra cùng slot |
| **Associative array** | Tên khác của dictionary/map |
| **Recursion** | Hàm tự gọi chính nó |
| **Base/breaking condition** | Điều kiện dừng đệ quy |
| **Call stack** | Stack lưu các lời gọi hàm |
| **Divide & conquer** | Chia bài toán nhỏ, giải rồi gộp |
| **Pivot** | Mốc phân hoạch trong quick sort |
| **Deterministic** | Mỗi bước có quyết định chính xác |
| **Exact / Approximate** | Kết quả đúng tuyệt đối / có thể xấp xỉ |
| **Factorial** | n! = n×(n-1)×…×1 |
| **Filtering** | Giữ phần tử thỏa điều kiện, bỏ phần còn lại |

```
★ Insight ─────────────────────────────────────
• Học thuộc 1 con số quan trọng: "for lồng for = O(n²)". Nhận ra nó
  trong code là kỹ năng tối ưu thực dụng nhất.
• Bảng "khi nào dùng cấu trúc nào" quan trọng hơn việc nhớ cài đặt
  — đi làm bạn dùng thư viện, nhưng CHỌN SAI cấu trúc thì không
  thư viện nào cứu được.
─────────────────────────────────────────────────
```

---

## Liên quan
- [[02-Do-phuc-tap-Big-O]] · [[08-Sorting]] · [[09-Searching]]
- [[00-MOC-DSA]] — quay lại MOC
