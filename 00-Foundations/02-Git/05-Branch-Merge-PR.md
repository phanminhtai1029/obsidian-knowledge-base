---
title: "Branch · Merge · Pull Request & giải quyết conflict"
section: 00-Foundations/02-Git
tags: [git, branch, merge, pull-request, conflict, rebase, foundations, fresher]
related:
  - "[[03-Daily-Workflow]]"
  - "[[09-Hoan-tac-va-sua-lich-su]]"
  - "[[10-Git-Log-nang-cao]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 50m
source: [Pro Git book, GitHub docs]
---

# Branch · Merge · Pull Request & giải quyết conflict

> [!summary] TL;DR
> **Branch** = một nhánh phát triển độc lập (thực chất chỉ là con trỏ tới một commit), cho phép làm tính năng mới mà không đụng code chính. **Merge** = gộp branch vào nhau: **fast-forward** (chỉ dời con trỏ, lịch sử thẳng) hoặc **3-way merge** (tạo merge commit). **Pull Request (PR)** = cơ chế của GitHub để review code trước khi merge vào nhánh chính. **Conflict** xảy ra khi 2 nhánh sửa cùng một đoạn — Git không tự quyết được, phải sửa tay.

---

## 1. Branch là gì?

Branch cho phép bạn "tách dòng" lịch sử để phát triển song song. Như đã nói ở [[01-How-Git-Works]], **branch chỉ là một con trỏ (41 byte) tới một commit** — nên tạo branch cực nhẹ và nhanh.

```text
              feature/login
                    │
                    ▼
        C1 ─── C2 ─── C3
               │
               ▼
              main
```

### Tạo & chuyển branch

```bash
git branch                    # liệt kê branch (dấu * = đang đứng)
git branch feature/login      # tạo branch mới (chưa chuyển sang)
git switch feature/login      # chuyển sang branch (Git 2.23+, khuyên dùng)
git switch -c feature/login   # tạo + chuyển luôn (-c = create)

# Cách cũ (vẫn dùng được, nhưng checkout "đa năng" dễ nhầm):
git checkout feature/login
git checkout -b feature/login
```

> [!tip] Git 2.23 tách `git checkout` (vốn làm quá nhiều việc) thành 2 lệnh rõ nghĩa: **`git switch`** (đổi branch) và **`git restore`** (khôi phục file). Nên dùng cặp mới cho đỡ nhầm.

### Quản lý branch

```bash
git branch -d feature/login   # xóa branch ĐÃ merge (an toàn)
git branch -D feature/login   # xóa cưỡng bức (kể cả chưa merge — cẩn thận!)
git branch -m ten-moi         # đổi tên branch hiện tại
git branch -a                 # liệt kê cả branch remote
git push origin --delete feature/login   # xóa branch trên remote
```

```
★ Insight ─────────────────────────────────────
• Vì branch chỉ là con trỏ, "chuyển branch" thực chất là dời HEAD + cập nhật
  working dir cho khớp snapshot của branch đó. Đó là lý do Git bắt bạn commit
  hoặc stash thay đổi đang dở trước khi switch — nếu không, working dir sẽ "lẫn"
  giữa 2 nhánh. (Cứu cánh khi đang dở: [[06-Git-Stash]].)
• Quy ước đặt tên branch: feature/x, fix/y, hotfix/z, chore/w. Tên có ý nghĩa
  giúp cả team và công cụ CI hiểu mục đích nhánh.
─────────────────────────────────────────────────
```

---

## 2. Merge — gộp branch

Sau khi làm xong tính năng trên `feature/login`, ta gộp nó vào `main`. **Luôn đứng ở branch ĐÍCH (nơi muốn nhận code) rồi merge branch nguồn vào:**

```bash
git switch main               # 1. về branch đích
git merge feature/login       # 2. gộp feature/login vào main
```

Có **2 kiểu merge** — đây là kiến thức phỏng vấn quan trọng:

### a) Fast-forward merge

Xảy ra khi `main` **không có commit mới** kể từ lúc tách nhánh. Git chỉ cần **dời con trỏ `main`** lên trùng `feature/login` — không tạo commit mới, lịch sử thẳng tắp.

```text
TRƯỚC:   C1 ─── C2(main) ─── C3 ─── C4(feature)

SAU:     C1 ─── C2 ─── C3 ─── C4(main, feature)   ← main chỉ "tiến lên"
```

### b) 3-way merge (tạo merge commit)

Xảy ra khi **cả hai branch đều có commit mới** sau điểm tách. Git tìm tổ tiên chung (common ancestor) và tạo một **merge commit** mới có **2 cha**.

```text
        C3 ─── C4 (feature)
       /              \
C1 ── C2 ────── C5 ─── M (main)   ← M là merge commit, 2 cha: C4 và C5
              (main)
```

```bash
# Buộc luôn tạo merge commit (giữ dấu vết nhánh) dù có thể fast-forward:
git merge --no-ff feature/login

# Chỉ cho phép fast-forward, nếu không thì báo lỗi (giữ lịch sử thẳng):
git merge --ff-only feature/login
```

| | Fast-forward | 3-way merge |
|---|--------------|-------------|
| Khi nào | Branch đích không có commit mới | Cả 2 branch đều tiến |
| Tạo merge commit? | Không | Có (2 cha) |
| Lịch sử | Thẳng | Phân nhánh rồi gộp |

```
★ Insight ─────────────────────────────────────
• Nhiều team bắt `--no-ff` cho merge vào main để mỗi tính năng để lại một "merge
  commit" rõ ràng → đọc lịch sử thấy ngay "đây là nơi feature X nhập vào". Đánh
  đổi: lịch sử nhiều nhánh hơn. Team thích lịch sử thẳng thì dùng rebase (mục 5).
• Merge KHÔNG sửa commit cũ — nó chỉ thêm. Nên merge an toàn cả khi đã push.
  Ngược lại rebase viết lại lịch sử nên chỉ dùng cho commit CHƯA push.
─────────────────────────────────────────────────
```

---

## 3. Pull Request (PR) — review trước khi merge

**Pull Request** không phải lệnh Git mà là **tính năng của GitHub/GitLab** (GitLab gọi là Merge Request). Đó là một "yêu cầu gộp" branch của bạn vào branch chính, kèm chỗ để:

- **Review code:** đồng đội đọc, comment từng dòng, đề nghị sửa.
- **Chạy CI tự động:** test/lint chạy trên PR trước khi cho merge (xem [[12-CI-CD-la-gi]]).
- **Thảo luận** & gắn issue.

### Luồng PR điển hình

```bash
# 1. Tạo branch & làm việc
git switch -c feature/login
# ... code, commit ...
git push -u origin feature/login    # đẩy lên, GitHub in ra link tạo PR

# 2. Lên GitHub → "Compare & pull request" → mô tả → Create
# 3. Đồng đội review, CI chạy. Sửa thêm nếu cần (commit + push tiếp, PR tự cập nhật)
# 4. Approve → "Merge pull request"
# 5. Xóa branch sau khi merge
```

### 3 chiến lược merge PR trên GitHub

| Chiến lược | Kết quả | Khi nào |
|-----------|---------|---------|
| **Create a merge commit** | Giữ mọi commit + 1 merge commit | Muốn giữ đầy đủ lịch sử nhánh |
| **Squash and merge** | Gộp **tất cả** commit của PR thành **1** commit | PR nhiều commit lặt vặt, muốn main sạch |
| **Rebase and merge** | "Dán" từng commit lên đầu main, lịch sử thẳng, **không** merge commit | Muốn lịch sử thẳng, giữ từng commit |

> [!tip] **Squash and merge** rất phổ biến: mỗi PR = 1 commit gọn trên `main`, dễ revert cả tính năng, lịch sử main sạch. Đánh đổi: mất chi tiết các commit nhỏ bên trong PR.

---

## 4. Giải quyết Merge Conflict ⭐

Conflict xảy ra khi **hai nhánh sửa CÙNG một đoạn của CÙNG một file** (hoặc 1 bên sửa, 1 bên xóa). Git không biết chọn bên nào → dừng lại nhờ bạn.

```bash
git merge feature/login
# Auto-merging index.html
# CONFLICT (content): Merge conflict in index.html
# Automatic merge failed; fix conflicts and then commit the result.
```

### Cách đọc dấu conflict trong file

Git chèn các marker vào file:

```text
<<<<<<< HEAD
<h1>Trang chủ</h1>              ← phiên bản ở branch hiện tại (HEAD, vd main)
=======
<h1>Welcome Home</h1>          ← phiên bản đến từ branch đang merge vào
>>>>>>> feature/login
```

- Phần giữa `<<<<<<< HEAD` và `=======` = code của **branch bạn đang đứng**.
- Phần giữa `=======` và `>>>>>>>` = code đến từ **branch được merge**.

### Quy trình giải quyết

```bash
# 1. Xem file nào conflict
git status

# 2. Mở từng file, sửa thủ công: GIỮ phần đúng, XÓA hết 3 dòng marker
#    (<<<<<<<, =======, >>>>>>>). Có thể giữ 1 bên, giữ cả hai, hoặc viết lại.

# 3. Đánh dấu đã giải quyết
git add index.html

# 4. Hoàn tất merge
git commit            # (message merge mặc định thường để nguyên là được)

# Nếu muốn HỦY merge giữa chừng, quay lại trạng thái trước merge:
git merge --abort
```

> [!tip] IDE (VS Code) hiển thị nút **"Accept Current Change / Accept Incoming Change / Accept Both"** ngay trên marker — bấm nhanh hơn xóa tay. Hoặc dùng `git mergetool`.

```
★ Insight ─────────────────────────────────────
• Conflict KHÔNG phải lỗi của bạn hay của Git — nó chỉ là Git nói "hai người
  cùng sửa chỗ này, tôi không tự quyết được, bạn quyết đi". Bình tĩnh đọc cả 2
  bên, hiểu ý mỗi bên định làm gì rồi mới gộp; đừng máy móc "accept current".
• Giảm conflict bằng thói quen: nhánh nhỏ & sống ngắn, pull/rebase main thường
  xuyên để không tụt hậu quá xa. Nhánh càng để lâu càng dễ "đụng" code người khác.
─────────────────────────────────────────────────
```

---

## 5. Merge vs Rebase — câu hỏi phỏng vấn kinh điển

Cả hai đều **tích hợp thay đổi từ branch này sang branch khác**, nhưng theo cách khác nhau:

| | **Merge** | **Rebase** |
|---|-----------|-----------|
| Cách làm | Tạo merge commit gộp 2 nhánh | "Nhấc" các commit của bạn đặt lên **đầu** branch kia |
| Lịch sử | Phân nhánh rồi gộp (có chỗ rẽ) | Thẳng tắp như chưa từng tách nhánh |
| Commit gốc | Giữ nguyên hash | **Viết lại** (hash mới) |
| An toàn khi đã push? | ✅ Có | ❌ Không (đừng rebase commit đã chia sẻ) |

```bash
# Đứng ở feature, đưa nó lên trên main mới nhất:
git switch feature/login
git rebase main
# Nếu có conflict: sửa → git add → git rebase --continue (lặp cho từng commit)
# Hủy: git rebase --abort
```

> [!warning] **Quy tắc vàng của rebase:** đừng bao giờ rebase một branch mà **người khác đã pull về**. Rebase đổi hash commit → lịch sử của họ và của bạn lệch nhau → loạn. Rebase chỉ dùng cho commit **local, chưa push**, hoặc nhánh chỉ mình bạn dùng. Chi tiết "sửa lịch sử an toàn" ở [[09-Hoan-tac-va-sua-lich-su]].

---

## 6. Bảng lệnh tra nhanh

| Việc | Lệnh |
|------|------|
| Liệt kê branch | `git branch` (`-a` cả remote) |
| Tạo + chuyển branch | `git switch -c <name>` |
| Chuyển branch | `git switch <name>` |
| Xóa branch (đã merge) | `git branch -d <name>` |
| Xóa branch remote | `git push origin --delete <name>` |
| Merge vào branch hiện tại | `git merge <name>` |
| Merge luôn tạo commit | `git merge --no-ff <name>` |
| Rebase lên branch khác | `git rebase <name>` |
| Hủy merge / rebase | `git merge --abort` / `git rebase --abort` |
| Đánh dấu conflict đã xong | `git add <file>` rồi `git commit` |

## 7. Tự kiểm tra

1. Fast-forward vs 3-way merge khác gì? *(FF dời con trỏ, không commit mới; 3-way tạo merge commit 2 cha)*
2. PR là lệnh Git hay tính năng GitHub? *(tính năng GitHub/GitLab)*
3. Squash and merge làm gì? *(gộp mọi commit của PR thành 1)*
4. 3 dòng marker conflict là gì, xử lý ra sao? *(<<<<<<<, =======, >>>>>>>; sửa tay, xóa marker, add, commit)*
5. Quy tắc vàng của rebase? *(không rebase commit đã được người khác pull/đã push chia sẻ)*

## Liên quan
- [[00-MOC-Git|⬅ MOC Git]]
- Trước: [[04-Gitignore-va-Khai-niem-cot-loi]] · Kế tiếp (Phần B): [[06-Git-Stash|git stash]]
- [[09-Hoan-tac-va-sua-lich-su]] · [[10-Git-Log-nang-cao]] · [[12-CI-CD-la-gi]]
