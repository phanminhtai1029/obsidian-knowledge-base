---
title: "DSA Cheatsheet — Big-O, CTDL & Glossary"
section: 00-Foundations/01-DSA
tags: [dsa, cheatsheet, big-o, glossary, reference, foundations, fresher]
module: L1_AL_NLS_DSAA
related:
  - "[[02-Do-phuc-tap-Big-O]]"
  - "[[12-Sorting]]"
  - "[[13-Searching]]"
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
| **BST (cân bằng)** | O(log n) | **O(log n)** | **O(log n)** | O(log n) | suy biến → O(n); giữ thứ tự |
| **Heap** | — (peek O(1)) | O(n) | **O(log n)** | **O(log n)** | max/min ở root, priority queue |
| **Graph** (adj list) | — | — | O(1) thêm cạnh | O(E) | duyệt BFS/DFS O(V+E) |

---

## 2. Big-O thuật toán Sort & Search

| Thuật toán | Trung bình | Worst | Best | Bộ nhớ phụ | Loại |
|------------|-----------|-------|------|------------|------|
| **Bubble sort** | O(n²) | O(n²) | O(n²) | O(1) | dạy học |
| **Selection sort** | O(n²) | O(n²) | O(n²) | O(1) | ít swap nhất |
| **Insertion sort** | O(n²) | O(n²) | **O(n)** | O(1) | tốt khi gần sort |
| **Merge sort** | **O(n log n)** | O(n log n) | O(n log n) | **O(n)** | divide & conquer, ổn định |
| **Quick sort** | **O(n log n)** | **O(n²)** | O(n log n) | O(log n) | in-place, nhanh thực tế |
| **Heap sort** | **O(n log n)** | **O(n log n)** | O(n log n) | O(1) | in-place, worst tốt |
| **Linear search** | O(n) | O(n) | O(1) | O(1) | list bất kỳ |
| **Binary search** | **O(log n)** | O(log n) | O(1) | O(1) | list **đã sort** |
| **BFS / DFS** (graph) | O(V+E) | O(V+E) | — | O(V) | duyệt đồ thị/cây |

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
| Tra cứu **có thứ tự**, range query, min/max | **BST** |
| Luôn lấy phần tử **ưu tiên nhất** (max/min) | **Heap / Priority Queue** |
| Dữ liệu **phân cấp** (thư mục, DOM, cây quyết định) | **Tree** |
| Quan hệ mạng lưới, đường đi (bản đồ, social) | **Graph** |
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

# Heap / Priority Queue (heapq là MIN heap)
import heapq
h = []; heapq.heappush(h, x); heapq.heappop(h)   # nhỏ nhất
heapq.heapify(lst)                               # build O(n)

# Binary tree node
class TreeNode:
    def __init__(self, data):
        self.data, self.left, self.right = data, None, None

# Graph adjacency list
graph = {"A": ["B", "C"], "B": ["A", "D"]}
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
| **Tree / root / leaf** | Cấu trúc phân cấp / node gốc (không cha) / node không con |
| **Parent / child / sibling** | Node cha / node con / 2 con cùng cha |
| **Binary tree** | Cây mỗi node tối đa 2 con (trái/phải) |
| **BST** | Binary tree: con trái < cha < con phải |
| **Traversal (DFS/BFS)** | Duyệt cây/đồ thị: theo chiều sâu (stack) / chiều rộng (queue) |
| **Pre/In/Post-order** | 3 kiểu DFS: thăm gốc trước/giữa/sau |
| **Heap (max/min)** | Complete binary tree: root lớn nhất / nhỏ nhất |
| **Priority queue** | Hàng đợi lấy ra theo độ ưu tiên (cài bằng heap) |
| **Graph / vertex / edge** | Đồ thị / đỉnh / cạnh |
| **Adjacency list/matrix** | 2 cách biểu diễn graph (list hàng xóm / ma trận V×V) |
| **Dijkstra** | Tìm đường ngắn nhất, trọng số không âm |

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
- [[02-Do-phuc-tap-Big-O]] · [[12-Sorting]] · [[13-Searching]]
- [[00-MOC-DSA]] — quay lại MOC
