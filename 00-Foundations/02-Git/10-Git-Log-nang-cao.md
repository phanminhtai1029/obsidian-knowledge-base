---
title: "git log nâng cao — đọc lịch sử & dò bug"
section: 00-Foundations/02-Git
tags: [git, log, graph, blame, bisect, diff, foundations, fresher]
related:
  - "[[03-Daily-Workflow]]"
  - "[[09-Hoan-tac-va-sua-lich-su]]"
difficulty: ⭐⭐⭐
estimated_time: 30m
source: ["Git superpowers (Ronnie Sheer)", Git docs]
---

# git log nâng cao — đọc lịch sử & dò bug

> [!summary] TL;DR
> "Trong source control, kiến thức là sức mạnh." `git log --oneline` xem lịch sử gọn; thêm `--graph --all` để thấy **cây nhánh** (branch nào merge vào đâu). Dùng log để định vị mình đang ở đâu trước khi push/PR, và để **dò bug**: `git blame` (ai sửa dòng này), `git bisect` (tìm commit gây lỗi bằng nhị phân), `git log -S` (commit nào thêm/xóa một đoạn code).

---

## 1. Các dạng `git log` thường dùng

```bash
git log                    # đầy đủ: hash, author, date, message (dài)
git log --oneline          # mỗi commit 1 dòng: hash ngắn + message
git log --oneline --graph  # kèm cây ASCII của nhánh
git log --oneline --graph --all   # tất cả branch, không chỉ branch hiện tại
git log -5                 # 5 commit gần nhất
git log --stat             # kèm thống kê file nào đổi, +/- bao nhiêu dòng
git log -p                 # kèm full diff mỗi commit
```

Ví dụ `--oneline --graph --all` — thấy ngay nhánh nào nhập vào đâu:

```text
*   3f2a1b (HEAD -> main) Merge branch 'feature/login'
|\
| * a1b2c3 (feature/login) feat: thêm form login
| * d4e5f6 feat: validate email
* | 9c8d7e fix: sửa header
|/
* 7a6b5c init: khởi tạo project
```

> [!tip] Đặt alias cho gọn (xem [[02-Install-Configure-Git]]):
> ```bash
> git config --global alias.lg "log --oneline --graph --all"
> ```
> Từ đó chỉ gõ `git lg`.

```
★ Insight ─────────────────────────────────────
• `--graph` biến lịch sử trừu tượng thành hình ảnh: bạn THẤY được "vài commit
  trước tôi tách nhánh, làm vài commit, rồi merge lại". Khi rối không biết đang
  ở đâu, đây là tấm bản đồ. Thói quen tốt: chạy `git lg` trước khi push/tạo PR
  để xác nhận "mình đúng là đang ở chỗ mình nghĩ".
─────────────────────────────────────────────────
```

---

## 2. Lọc & tìm trong lịch sử

```bash
git log --author="Tai"            # commit của một người
git log --since="2 weeks ago"     # theo thời gian
git log --until="2026-06-01"
git log --grep="login"            # message chứa "login"
git log -S "functionName"         # commit nào THÊM/XÓA chuỗi "functionName" (pickaxe)
git log -G "regex"                # tương tự nhưng theo regex
git log <branch1>..<branch2>      # commit có ở branch2 mà chưa có ở branch1
git log -- path/to/file           # lịch sử của riêng 1 file
git log --oneline --follow file   # theo dõi cả khi file bị đổi tên
```

> [!tip] `git log -S "console.log"` cực mạnh khi truy "dòng debug này lọt vào từ commit nào?". `..` rất hữu ích trước khi merge: `git log main..feature` = "feature có gì mới so với main".

---

## 3. `git show` — soi 1 commit

```bash
git show                # commit hiện tại (HEAD): metadata + diff
git show a1b2c3d        # commit cụ thể
git show a1b2c3d:path/file   # nội dung 1 file TẠI commit đó
git show HEAD~2         # commit lùi 2 bước
```

---

## 4. Dò bug — bộ ba mạnh: blame, bisect, diff

### `git blame` — ai sửa dòng này, ở commit nào?

```bash
git blame file.js                 # mỗi dòng: hash + author + ngày + nội dung
git blame -L 10,20 file.js        # chỉ dòng 10–20
```

Dùng để hiểu **vì sao** một dòng tồn tại: lấy hash rồi `git show <hash>` đọc commit đó.

### `git bisect` — tìm commit gây lỗi bằng tìm nhị phân

Khi bug xuất hiện đâu đó giữa "bản tốt" và "bản hỏng" qua hàng trăm commit, bisect tìm thủ phạm trong ~log₂(n) bước:

```bash
git bisect start
git bisect bad                    # commit hiện tại đang LỖI
git bisect good v1.0              # commit/tag mà bạn biết còn TỐT
# Git nhảy tới commit giữa → bạn test → trả lời:
git bisect good                   # commit này ổn
# hoặc
git bisect bad                    # commit này đã lỗi
# ... lặp lại, Git thu hẹp dần ...
# → Git in ra "<hash> is the first bad commit"
git bisect reset                  # kết thúc, quay về branch cũ
```

```
★ Insight ─────────────────────────────────────
• bisect là thuật toán tìm kiếm nhị phân áp vào lịch sử git: 1000 commit chỉ cần
  ~10 lần test thay vì dò tuần tự. Điều kiện để nó hiệu quả: commit ATOMIC, mỗi
  commit build/chạy được — đúng lý do nên commit nhỏ & sạch ([[04-Gitignore-va-Khai-niem-cot-loi]]).
• blame + show + bisect là quy trình điều tra: blame chỉ "nghi phạm" (commit nào
  chạm dòng lỗi) → show đọc chi tiết → bisect xác nhận chính xác commit làm vỡ.
─────────────────────────────────────────────────
```

### `git diff` giữa các điểm

```bash
git diff HEAD~1 HEAD              # so 2 commit liền kề
git diff main feature            # so 2 branch
git diff a1b2c3d e4f5g6h         # so 2 commit bất kỳ
git diff --stat main feature     # chỉ tóm tắt file & số dòng
```

---

## 5. Bảng lệnh tra nhanh

| Việc | Lệnh |
|------|------|
| Lịch sử gọn | `git log --oneline` |
| Cây nhánh | `git log --oneline --graph --all` |
| Lịch sử 1 file | `git log -- <file>` |
| Tìm theo message | `git log --grep="..."` |
| Tìm commit thêm/xóa chuỗi | `git log -S "..."` |
| Branch B có gì hơn A | `git log A..B` |
| Soi 1 commit | `git show <hash>` |
| Ai sửa dòng nào | `git blame <file>` |
| Dò commit gây lỗi | `git bisect start/good/bad` |
| So sánh 2 điểm | `git diff <a> <b>` |

## 6. Tự kiểm tra

1. Cờ nào để thấy cây nhánh trong log? *(`--graph`, thường kèm `--all`)*
2. `git log main..feature` cho ra gì? *(commit feature có mà main chưa có)*
3. `git log -S "x"` tìm gì? *(commit thêm/xóa chuỗi "x")*
4. `git bisect` dùng thuật toán gì, lợi ích? *(tìm nhị phân — định vị commit lỗi cực nhanh)*
5. Muốn biết ai viết một dòng code? *(`git blame`)*

## Liên quan
- [[00-MOC-Git|⬅ MOC Git]]
- Trước: [[09-Hoan-tac-va-sua-lich-su]] · Kế tiếp: [[11-Git-Hooks-va-meo-GitHub|Git Hooks & mẹo GitHub]]
