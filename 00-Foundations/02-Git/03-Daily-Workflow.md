---
title: "Workflow hằng ngày — add · commit · push · pull"
section: 00-Foundations/02-Git
tags: [git, workflow, add, commit, push, pull, fetch, diff, foundations, fresher]
related:
  - "[[01-How-Git-Works]]"
  - "[[04-Gitignore-va-Khai-niem-cot-loi]]"
  - "[[05-Branch-Merge-PR]]"
difficulty: ⭐⭐
estimated_time: 40m
source: [Pro Git book, Git docs]
---

# Workflow hằng ngày — add · commit · push · pull

> [!summary] TL;DR
> Vòng lặp công việc mỗi ngày: `pull` (lấy code mới nhất) → sửa code → `status`/`diff` (kiểm tra) → `add` (stage) → `commit` (lưu local) → `push` (đẩy lên remote). Nhớ phân biệt: **commit** chỉ lưu vào repo **local**, **push** mới đưa lên remote (GitHub). `fetch` = chỉ tải về, **không** gộp; `pull` = `fetch` + `merge`.

---

## 1. Bức tranh tổng thể

```text
   remote (GitHub)
        │  ▲
   pull │  │ push
        ▼  │
   ┌─────────────┐   commit   ┌──────────┐   add   ┌─────────────────┐
   │ Local Repo  │ ◀───────── │ Staging  │ ◀────── │ Working Directory│
   │  (.git)     │            │ (Index)  │         │ (file bạn sửa)   │
   └─────────────┘            └──────────┘         └─────────────────┘
```

| Lệnh | Chuyển từ → tới | Ý nghĩa |
|------|-----------------|---------|
| `git add` | Working → Staging | Chọn thay đổi cho commit kế |
| `git commit` | Staging → Local repo | Lưu snapshot vào lịch sử local |
| `git push` | Local repo → Remote | Đẩy commit lên GitHub |
| `git pull` | Remote → Local + Working | Lấy commit mới + gộp vào |
| `git fetch` | Remote → Local (chỉ tải) | Tải về, **chưa** gộp |

---

## 2. Kiểm tra trạng thái: `status` và `diff`

```bash
git status            # file nào untracked / modified / staged
git status -s         # dạng ngắn gọn (-s = short)
```

`git status -s` đọc theo 2 cột ký hiệu:

```text
 M file1     # cột phải = Modified ở working dir (chưa add)
M  file2     # cột trái  = Modified đã Staged (đã add)
MM file3     # đã add rồi sửa tiếp → vừa staged vừa modified
A  file4     # Added (file mới đã stage)
?? file5     # Untracked (Git chưa biết)
```

Xem **chi tiết khác biệt**:

```bash
git diff               # khác biệt: Working Dir vs Staging (phần CHƯA add)
git diff --staged      # khác biệt: Staging vs commit cuối (phần ĐÃ add, sắp commit)
git diff HEAD          # khác biệt: Working Dir vs commit cuối (cả staged lẫn chưa)
```

```
★ Insight ─────────────────────────────────────
• Nhầm lẫn kinh điển của người mới: gõ `git diff` sau khi đã `git add` thì thấy
  "trống trơn" và tưởng mất code. Không hề — `git diff` chỉ soi phần CHƯA staged.
  Phần đã add phải xem bằng `git diff --staged`. Nhớ: diff không tham số = "những
  gì commit tới đây sẽ BỎ SÓT nếu tôi commit ngay bây giờ".
─────────────────────────────────────────────────
```

---

## 3. Stage: `git add`

```bash
git add index.html            # 1 file cụ thể
git add src/                  # cả thư mục
git add .                     # tất cả thay đổi từ thư mục hiện tại xuống
git add -A                    # tất cả thay đổi trong toàn repo (kể cả file bị xóa)
git add -p                    # chọn TỪNG ĐOẠN (hunk) — kiểm soát kỹ, xem [[07-Staging-chon-loc]]
```

Gỡ stage (đưa file ra khỏi Staging, **không** mất sửa đổi):

```bash
git restore --staged file     # cách mới (Git 2.23+)
git reset file                # cách cũ, tương đương
```

Bỏ luôn thay đổi ở working dir (⚠️ **mất** sửa đổi chưa commit):

```bash
git restore file              # khôi phục file về bản đã commit
```

---

## 4. Commit

```bash
git commit -m "feat: thêm form đăng nhập"        # commit nhanh với message
git commit                                        # mở editor để viết message dài
git commit -am "fix: sửa lỗi validate email"      # -a = tự add các file ĐÃ track + commit
```

> [!warning] `-am` **chỉ** stage những file Git **đã track**. File mới (untracked) sẽ KHÔNG được đưa vào — phải `git add` trước. Đây là bẫy hay gặp khi commit thiếu file mới.

**Quy ước commit message** (Conventional Commits) — chi tiết ở [[04-Gitignore-va-Khai-niem-cot-loi]]:

```text
feat: thêm tính năng mới
fix: sửa bug
docs: sửa tài liệu
refactor: tái cấu trúc, không đổi hành vi
test: thêm/sửa test
chore: việc lặt vặt (cấu hình, deps)
```

---

## 5. Đồng bộ với remote: `push`, `fetch`, `pull`

```bash
git push origin main          # đẩy branch main lên remote origin
git push                      # nếu đã set upstream thì gõ gọn (xem [[08-Push-tuong-minh]])

git fetch origin              # tải commit mới về, KHÔNG gộp — an toàn để xem trước
git pull origin main          # = fetch + merge (gộp luôn vào branch hiện tại)
git pull --rebase origin main # = fetch + rebase (lịch sử thẳng, không tạo merge commit)
```

### `fetch` vs `pull` — câu hỏi phỏng vấn

| | `fetch` | `pull` |
|---|---------|--------|
| Hành động | Tải commit mới từ remote về local | `fetch` **+** `merge` (gộp ngay) |
| An toàn | Cao — chưa đụng code của bạn | Có thể tạo conflict/merge commit ngay |
| Khi nào | Muốn xem remote có gì mới trước khi gộp | Muốn cập nhật & gộp luôn |

```
★ Insight ─────────────────────────────────────
• `pull` không phải phép màu mà là 2 bước gộp lại: fetch (lấy về) rồi merge (hòa
  vào). Khi bạn thấy "merge conflict" lúc pull, thực chất conflict sinh ra ở
  bước merge — xem cách giải quyết ở [[05-Branch-Merge-PR]].
• Thói quen tốt: trước khi bắt đầu ngày làm việc, `git pull` để code mới nhất;
  trước khi push, `git pull --rebase` để tránh merge commit thừa và push trơn tru.
─────────────────────────────────────────────────
```

---

## 6. Xem lịch sử: `git log`

```bash
git log                       # đầy đủ: hash, author, date, message
git log --oneline             # mỗi commit 1 dòng (gọn)
git log --oneline --graph     # kèm cây nhánh
git log -p file               # kèm diff của file qua từng commit
git log -3                    # 3 commit gần nhất
```

Chi tiết & mẹo dò bug: [[10-Git-Log-nang-cao]].

---

## 7. Vòng lặp một ngày làm việc điển hình

```bash
git pull origin main                 # 1. lấy code mới nhất
git switch -c feature/login          # 2. tạo branch làm tính năng (xem note 5)
# ... sửa code ...
git status                           # 3. xem mình đã đổi gì
git add -p                           # 4. stage có chọn lọc
git commit -m "feat: thêm trang login"
git push -u origin feature/login     # 5. đẩy lên, mở Pull Request
```

---

## 8. Bảng lệnh tra nhanh

| Việc | Lệnh |
|------|------|
| Xem trạng thái | `git status` / `git status -s` |
| Xem khác biệt chưa stage | `git diff` |
| Xem khác biệt đã stage | `git diff --staged` |
| Stage 1 file / tất cả | `git add file` / `git add .` |
| Gỡ stage | `git restore --staged file` |
| Bỏ sửa đổi working | `git restore file` |
| Commit | `git commit -m "msg"` |
| Đẩy lên remote | `git push origin <branch>` |
| Lấy + gộp | `git pull origin <branch>` |
| Chỉ tải về | `git fetch origin` |
| Xem lịch sử gọn | `git log --oneline --graph` |

## Liên quan
- [[00-MOC-Git|⬅ MOC Git]]
- Trước: [[02-Install-Configure-Git]] · Kế tiếp: [[04-Gitignore-va-Khai-niem-cot-loi|.gitignore & khái niệm]]
- [[07-Staging-chon-loc]] · [[08-Push-tuong-minh]] · [[14-Git-Cheatsheet|⭐ Cheatsheet]]
