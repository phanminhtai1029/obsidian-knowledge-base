---
title: "MOC: Data Structures & Algorithms"
section: 00-Foundations/01-DSA
tags: [moc, dsa, algorithms, data-structures, foundations, fresher]
module: L1_AL_NLS_DSAA
---

# MOC: Data Structures & Algorithms

> Module `L1_AL_NLS_DSAA`. Thi: **Final Theory Test + Final Practical Test**.
> Nguồn: *Programming Foundations: Algorithms* (Joe Marini, LinkedIn) — bổ sung Tree/BST/Heap/Graph từ giáo trình Data Structures + kiến thức DSA chuẩn.
> ✅ 15 note · 6 cụm.

## Cụm A — Nền tảng lý thuyết

| # | Note | Nội dung | Trạng thái |
|---|------|----------|------------|
| 1 | [[01-Tong-quan-Thuat-toan\|Tổng quan Thuật toán]] | Thuật toán là gì, đặc điểm, phân loại, 4 nhóm, Euclid GCD | ✅ |
| 2 | [[02-Do-phuc-tap-Big-O\|Độ phức tạp & Big-O]] ⭐ | Big-O, time/space, O(1)→O(n²), nhận diện trong code | ✅ |

## Cụm B — Cấu trúc dữ liệu

| # | Note | Nội dung | Trạng thái |
|---|------|----------|------------|
| 3 | [[03-Array\|Array]] | Mảng 1D/2D, jagged, random access, Big-O | ✅ |
| 4 | [[04-Linked-List\|Linked List]] | Node/head, singly/doubly, insert/find/delete | ✅ |
| 5 | [[05-Stack-va-Queue\|Stack & Queue]] | LIFO/FIFO, deque, specialized queues | ✅ |
| 6 | [[06-Dictionary-Hash-Table\|Dictionary & Hash Table]] | Hash function, collision, Set | ✅ |
| 7 | [[07-Tree\|Tree]] | Root/leaf/sibling, binary tree, DFS/BFS traversal | ✅ |
| 8 | [[08-Binary-Search-Tree\|Binary Search Tree]] | Quy tắc BST, search/insert/delete O(log n), suy biến | ✅ |
| 9 | [[09-Heap-Priority-Queue\|Heap & Priority Queue]] | Max/min heap, priority queue, heap sort | ✅ |
| 10 | [[10-Graph\|Graph]] | Adjacency list/matrix, BFS/DFS, Dijkstra | ✅ |

## Cụm C — Đệ quy

| # | Note | Nội dung | Trạng thái |
|---|------|----------|------------|
| 11 | [[11-De-quy-Recursion\|Đệ quy]] | Base case, call stack, countdown/power/factorial | ✅ |

## Cụm D — Sắp xếp

| # | Note | Nội dung | Trạng thái |
|---|------|----------|------------|
| 12 | [[12-Sorting\|Sorting]] | Bubble/Selection/Insertion · Merge/Quick/Heap sort | ✅ |

## Cụm E — Tìm kiếm

| # | Note | Nội dung | Trạng thái |
|---|------|----------|------------|
| 13 | [[13-Searching\|Searching]] | Linear, Binary search, kiểm tra sorted | ✅ |

## Cụm F — Thực hành & tra cứu

| # | Note | Nội dung | Trạng thái |
|---|------|----------|------------|
| 14 | [[14-Thuat-toan-ung-dung\|Thuật toán ứng dụng]] ⭐ | Set lọc unique · Dict đếm · find max đệ quy · Stack cân bằng ngoặc | ✅ |
| 15 | [[15-DSA-Cheatsheet\|DSA Cheatsheet]] | Bảng Big-O tổng hợp · khi nào dùng gì · snippet · glossary | ✅ |

## Lộ trình học

```mermaid
flowchart LR
    A["A. Big-O<br/>(nền)"] --> B["B. Cấu trúc<br/>dữ liệu<br/>(Array→Graph)"]
    B --> C["C. Đệ quy"]
    C --> D["D. Sorting"]
    D --> E["E. Searching"]
    E --> F["F. Thực hành<br/>+ Cheatsheet"]
```

## Liên quan
- [[../00-MOC-Foundations|MOC: Foundations]]
