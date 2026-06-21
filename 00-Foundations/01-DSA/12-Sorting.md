---
title: "Sorting — Thuật toán sắp xếp (Bubble, Merge, Quick)"
section: 00-Foundations/01-DSA
tags: [dsa, sorting, bubble-sort, merge-sort, quicksort, divide-conquer, foundations, fresher]
module: L1_AL_NLS_DSAA
related:
  - "[[02-Do-phuc-tap-Big-O]]"
  - "[[11-De-quy-Recursion]]"
  - "[[13-Searching]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 45m
source: ["Programming Foundations: Algorithms — Joe Marini (LinkedIn Learning)"]
---

# Sorting — Thuật toán sắp xếp

> [!summary] TL;DR
> Sắp xếp đưa tập dữ liệu về **một thứ tự** — vừa để hiển thị cho người dùng, vừa để app xử lý hiệu quả hơn (vd lọc theo khoảng). **Bubble sort**: so sánh cặp kề nhau & hoán đổi, đơn giản nhưng **O(n²)** — chỉ để dạy học. **Merge sort**: chia để trị bằng đệ quy, gộp 2 mảng đã sort, **O(n log n)** nhưng **tốn bộ nhớ phụ O(n)**. **Quick sort**: chọn **pivot**, phân hoạch quanh pivot, **O(n log n)** trung bình, sort **in-place** (ít RAM) nhưng **worst-case O(n²)** khi mảng đã gần sort. Ngôn ngữ hiện đại đã có sort sẵn — học để **hiểu & chọn đúng**.

---

## 1. Vì sao cần sort?

- **Hiển thị:** người dùng app bất động sản muốn xem nhà theo giá tăng dần.
- **Hiệu quả xử lý:** nếu dữ liệu đã sort, app lọc theo khoảng giá / **binary search** nhanh hơn nhiều ([[13-Searching]]).

3 thuật toán đại diện 3 cách tiếp cận: Bubble (ngây thơ), Merge & Quick (divide & conquer).

---

## 2. Bubble Sort — O(n²)

So sánh **2 phần tử kề nhau**, nếu cái trước lớn hơn thì **hoán đổi**. Lặp đi lặp lại; sau mỗi vòng, phần tử lớn nhất "nổi" lên cuối (như bọt khí).

```mermaid
flowchart LR
    A["[5, 3, 8, 1]"] -->|"so 5>3 → đổi"| B["[3, 5, 8, 1]"]
    B -->|"5<8 giữ"| C["[3,5,8,1]"]
    C -->|"8>1 → đổi"| D["[3,5,1,8] (8 nổi lên cuối)"]
```

```python
def bubble_sort(data):
    for i in range(len(data) - 1, 0, -1):   # vùng xét co dần
        for j in range(i):                  # so cặp kề nhau
            if data[j] > data[j + 1]:
                data[j], data[j + 1] = data[j + 1], data[j]  # swap
    return data
```

> **2 vòng `for` lồng nhau → O(n²).** Bubble sort còn không biết "mảng đã sort xong" nên vẫn chạy hết các vòng → kém. Chỉ dùng làm công cụ dạy học.

---

## 2b. Selection Sort — O(n²)

Mỗi vòng **tìm phần tử nhỏ nhất** trong phần chưa sort rồi **đưa về đầu**. Khác bubble: chỉ **1 lần swap** mỗi vòng (ít hoán đổi hơn).

```python
def selection_sort(data):
    for i in range(len(data)):
        min_idx = i
        for j in range(i + 1, len(data)):   # tìm min phần còn lại
            if data[j] < data[min_idx]:
                min_idx = j
        data[i], data[min_idx] = data[min_idx], data[i]   # 1 swap/vòng
    return data
```

## 2c. Insertion Sort — O(n²), nhưng tốt với mảng gần sort

Như xếp bài trên tay: lấy từng phần tử, **chèn vào đúng chỗ** trong phần đã sort bên trái. **Best-case O(n)** khi mảng **đã gần sort** → là sort O(n²) "tốt nhất" trong nhóm; dùng cho mảng nhỏ (Timsort của Python dùng insertion cho đoạn nhỏ).

```python
def insertion_sort(data):
    for i in range(1, len(data)):
        key = data[i]
        j = i - 1
        while j >= 0 and data[j] > key:     # dịch phần tử lớn sang phải
            data[j + 1] = data[j]
            j -= 1
        data[j + 1] = key                   # chèn key vào chỗ đúng
    return data
```

> [!tip] 3 sort O(n²) khác nhau ở đâu?
> **Bubble** so cặp kề & swap nhiều · **Selection** ít swap nhất (1/vòng) · **Insertion** nhanh nhất khi mảng **gần sort** (best O(n)). Cả 3 đều O(n²) worst-case, đơn giản, **in-place**.

---

## 3. Merge Sort — O(n log n)

**Divide & conquer:** chia mảng đôi liên tục tới khi còn **mảng 1 phần tử** (mặc nhiên đã sort), rồi **gộp (merge)** ngược lên thành mảng lớn vẫn sort.

```mermaid
flowchart TD
    A["[8,4,23,12]"] --> B["[8,4]"]
    A --> C["[23,12]"]
    B --> D["[8]"] 
    B --> E["[4]"]
    C --> F["[23]"]
    C --> G["[12]"]
    D & E --> H["merge → [4,8]"]
    F & G --> I["merge → [12,23]"]
    H & I --> J["merge → [4,8,12,23] ✅"]
```

**Chìa khóa = bước merge:** so phần tử đầu của 2 mảng đã sort, lấy cái nhỏ hơn bỏ vào kết quả, dịch index — lặp tới hết.

```python
def merge_sort(dataset):
    if len(dataset) > 1:
        mid = len(dataset) // 2
        left = dataset[:mid]
        right = dataset[mid:]
        merge_sort(left)               # đệ quy nửa trái
        merge_sort(right)              # đệ quy nửa phải

        i = j = k = 0
        while i < len(left) and j < len(right):    # bước merge
            if left[i] < right[j]:
                dataset[k] = left[i]; i += 1
            else:
                dataset[k] = right[j]; j += 1
            k += 1
        while i < len(left):           # copy phần dư bên trái
            dataset[k] = left[i]; i += 1; k += 1
        while j < len(right):          # copy phần dư bên phải
            dataset[k] = right[j]; j += 1; k += 1
    return dataset
```

> Merge sort **cần bộ nhớ phụ O(n)** (tạo `left`/`right`). Đây là cái giá cho sự ổn định.

---

## 4. Quick Sort — O(n log n) trung bình, in-place

Cũng divide & conquer + đệ quy, nhưng **mọi việc xảy ra ở bước phân hoạch (partition)**, sort **tại chỗ** không cần mảng phụ.

**Cơ chế:** chọn một **pivot** (vd phần tử đầu). Dùng 2 index chạy từ 2 phía: trái tiến tới khi gặp giá trị **> pivot**, phải lùi tới khi gặp giá trị **< pivot**, rồi **hoán đổi**. Khi 2 index cắt nhau → đổi pivot vào điểm chia. Giờ trái < pivot < phải. **Đệ quy** lên 2 nửa.

```mermaid
flowchart LR
    A["pivot=20<br/>[20,6,8,53,...,19]"] -->|partition| B["[nhỏ hơn 20] | 20 | [lớn hơn 20]"]
    B -->|đệ quy| C["quicksort(trái)"]
    B -->|đệ quy| D["quicksort(phải)"]
```

```python
def quicksort(data, first, last):
    if first < last:
        pivot_idx = partition(data, first, last)
        quicksort(data, first, pivot_idx - 1)    # đệ quy trái
        quicksort(data, pivot_idx + 1, last)     # đệ quy phải

def partition(data, first, last):
    pivot = data[first]
    lower, upper = first + 1, last
    done = False
    while not done:
        while lower <= upper and data[lower] <= pivot:
            lower += 1
        while data[upper] >= pivot and upper >= lower:
            upper -= 1
        if upper < lower:
            done = True
        else:
            data[lower], data[upper] = data[upper], data[lower]
    data[first], data[upper] = data[upper], data[first]   # pivot vào chỗ
    return upper
```

> **Worst-case O(n²)** khi mảng **đã sort/gần sort** và pivot chọn dở (vd luôn lấy phần tử đầu) → phân hoạch lệch.

---

## 5. Heap Sort — O(n log n)

Dùng **[[09-Heap-Priority-Queue|heap]]**: build heap từ mảng (O(n)), rồi liên tục **extract** phần tử lớn/nhỏ nhất ở root đưa về cuối (n × O(log n)). Sort **in-place**, worst-case vẫn **O(n log n)** (hơn quick sort ở điểm không có gót chân O(n²)), nhưng thực tế thường chậm hơn quick do hằng số lớn + truy cập bộ nhớ kém locality.

```python
import heapq
def heap_sort(data):
    heapq.heapify(data)                  # O(n) build min heap
    return [heapq.heappop(data) for _ in range(len(data))]   # n × O(log n)
```

---

## 6. Bảng so sánh

| | Bubble | Selection | Insertion | Merge | Quick | Heap |
|---|--------|-----------|-----------|-------|-------|------|
| Trung bình | O(n²) | O(n²) | O(n²) | **O(n log n)** | **O(n log n)** | **O(n log n)** |
| Worst | O(n²) | O(n²) | O(n²) | O(n log n) | **O(n²)** | **O(n log n)** |
| Best | O(n²) | O(n²) | **O(n)** | O(n log n) | O(n log n) | O(n log n) |
| Bộ nhớ phụ | O(1) | O(1) | O(1) | **O(n)** ❌ | O(log n) | O(1) |
| In-place | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ |
| Đặc điểm | dạy học | ít swap nhất | tốt khi gần sort | ổn định, đoán được | nhanh thực tế | worst tốt, no O(n²) |

> [!question] Phỏng vấn: "Merge ở đâu, Quick ở đâu?"
> Trong **Merge sort**, mảng được chẻ "ngây thơ" rồi **mọi công sức nằm ở bước GỘP**. Trong **Quick sort** ngược lại — **mọi công sức ở bước PHÂN HOẠCH** (đặt nhỏ-trái/lớn-phải quanh pivot), bước gộp chẳng làm gì vì đã sort in-place. Đây là câu hỏi "bẫy" rất hay gặp.

```
★ Insight ─────────────────────────────────────
• Bubble sort dạy ta dấu hiệu O(n²): "for lồng for". Nó còn minh họa
  điểm yếu "không biết khi nào xong" — bài học về thuật toán không
  tận dụng thông tin sẵn có.
• Merge vs Quick là đánh đổi THỜI GIAN ổn định ↔ BỘ NHỚ: Merge luôn
  O(n log n) nhưng tốn RAM; Quick tiết kiệm RAM, thường nhanh hơn,
  nhưng có gót chân Achilles O(n²). Chọn pivot ngẫu nhiên/median
  giúp Quick né worst-case.
• Thực tế bạn HIẾM KHI tự viết sort (ngôn ngữ có sẵn — Python dùng
  Timsort O(n log n)). Học 3 thuật toán này là để ĐỌC HIỂU & CHỌN,
  và vì chúng là bài thi/phỏng vấn kinh điển.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. Vì sao bubble sort là O(n²)? Dấu hiệu nào trong code báo điều đó?
2. Merge sort "chia để trị" thế nào? Bước nào làm phần lớn công việc?
3. Pivot trong quick sort là gì? Khi nào quick sort tụt xuống O(n²)?
4. Merge và Quick cùng O(n log n) — khác nhau ở space ra sao?
5. Viết lại bước merge (gộp 2 mảng đã sort) bằng lời.

---

## Liên quan
- [[11-De-quy-Recursion]] — nền tảng cho merge & quick
- [[02-Do-phuc-tap-Big-O]] — vì sao O(n²) vs O(n log n)
- [[13-Searching]] — sort xong để binary search
