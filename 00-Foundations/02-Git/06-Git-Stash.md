---
title: "git stash — cất code tạm thời"
section: 00-Foundations/02-Git
tags: [git, stash, workflow, foundations, fresher]
related:
  - "[[03-Daily-Workflow]]"
  - "[[05-Branch-Merge-PR]]"
difficulty: ⭐⭐⭐
estimated_time: 25m
source: ["Git superpowers (Ronnie Sheer)", Git docs]
---

# git stash — cất code tạm thời

> [!summary] TL;DR
> `git stash` = "cất thay đổi đang dở vào túi" để có **working directory sạch** mà **không phải commit** code chưa xong. Hữu ích khi đang code dở thì sếp nhờ switch branch sửa gấp. `git stash pop` lấy lại (và xóa khỏi túi); `git stash apply` lấy lại nhưng **giữ** trong túi. Quản lý nhiều stash bằng `git stash list` + tên.

---

## 1. Vấn đề mà stash giải quyết

Bạn đang sửa dở một tính năng (code chưa chạy, chưa muốn commit). Đột nhiên cần:

- Chuyển sang branch khác để fix bug gấp, hoặc
- `git pull` mà Git báo "local changes would be overwritten".

Git **không cho** switch/pull khi working directory đang "bẩn" và sẽ bị đè. Hai lựa chọn dở: (a) commit code nửa vời (làm bẩn lịch sử), (b) copy code ra file tạm rồi dán lại (thủ công, dễ mất).

**`git stash`** là lựa chọn thứ ba, gọn gàng: cất thay đổi đi, làm sạch working dir, xong việc thì lấy lại.

```
★ Insight ─────────────────────────────────────
• Hãy hình dung stash như "cho code vào túi áo một lúc". Nó lưu cả thay đổi đã
  staged lẫn chưa staged vào một vùng riêng (thực chất là vài commit ẩn), rồi
  trả working dir về đúng trạng thái commit cuối (HEAD) — sạch sẽ để bạn switch.
• Stash là cứu cánh trực tiếp cho ràng buộc ở [[05-Branch-Merge-PR]]: "phải sạch
  mới được switch branch". Thay vì commit dở, bạn stash → switch → quay lại → pop.
─────────────────────────────────────────────────
```

---

## 2. Lệnh cơ bản — 90% trường hợp

```bash
git stash             # cất tất cả thay đổi (tracked), working dir trở nên sạch
# ... switch branch, fix bug, commit, quay lại branch cũ ...
git stash pop         # lấy lại thay đổi VÀ xóa nó khỏi danh sách stash
```

> [!tip] `git stash` cũ là dạng tắt của `git stash push`. Dùng `git stash` cho nhanh là đủ.

### `pop` vs `apply`

| Lệnh | Lấy lại code? | Còn giữ trong túi? |
|------|---------------|---------------------|
| `git stash pop` | ✅ | ❌ (xóa khỏi list) |
| `git stash apply` | ✅ | ✅ (vẫn còn, áp dụng lại được nhiều nơi) |

Dùng `apply` khi muốn áp cùng thay đổi lên **nhiều branch**, hoặc khi lo `pop` gặp conflict thì vẫn còn bản gốc.

---

## 3. Quản lý nhiều stash

```bash
git stash save "thêm smiley vào header"   # cất kèm TÊN để dễ nhớ
git stash list                            # xem danh sách
#   stash@{0}: On main: thêm smiley vào header
#   stash@{1}: WIP on feature: ...

git stash apply stash@{1}    # áp dụng stash cụ thể
git stash pop stash@{0}      # pop stash cụ thể
git stash show -p stash@{0}  # xem diff của một stash
git stash drop stash@{0}     # xóa một stash khỏi list
git stash clear              # xóa SẠCH mọi stash (cẩn thận!)
```

> [!warning] `git stash save "msg"` là cú pháp cũ; cú pháp mới là `git stash push -m "msg"`. Cả hai vẫn dùng được.

```text
Túi stash (LIFO — vào sau ra trước):
   stash@{0}  ← mới nhất (pop lấy cái này)
   stash@{1}
   stash@{2}  ← cũ nhất
```

---

## 4. Tùy chọn hay dùng

```bash
git stash -u                 # cất luôn file UNtracked (-u = --include-untracked)
git stash -a                 # cất TẤT CẢ kể cả file bị ignore (-a = --all)
git stash -p                 # chọn TỪNG ĐOẠN để stash (giống add -p, xem note 7)
git stash branch ten-moi     # tạo branch mới từ stash (hữu ích khi stash bị conflict)
```

> [!warning] **Bẫy thường gặp:** `git stash` mặc định **KHÔNG** cất file untracked (file mới chưa từng add). Nếu file mới của bạn "biến mất" sau stash — không phải mất, mà nó vẫn nằm ở working dir; còn nếu bạn tưởng đã cất hết thì nhầm. Dùng `git stash -u` để gồm cả file mới.

---

## 5. Tình huống thực tế

```bash
# Đang code dở feature/profile thì có bug production
git stash -u                          # cất hết (gồm file mới)
git switch main
git switch -c hotfix/login-crash
# ... fix, commit, push, tạo PR ...
git switch feature/profile
git stash pop                         # lấy lại code đang dở, làm tiếp
```

```
★ Insight ─────────────────────────────────────
• Stash không "thuộc" branch nào — nó là một ngăn xếp toàn cục của repo. Bạn có
  thể stash ở branch A rồi pop ở branch B (chính là cách di chuyển thay đổi chưa
  commit giữa các branch — một mẹo hay gặp trong thử thách "đổi branch cho commit").
• Đừng để stash "ngủ quên": stash càng cũ càng dễ conflict khi pop vì code nền
  đã thay đổi nhiều. Thói quen tốt: stash là việc NGẮN HẠN, xử lý trong ngày.
─────────────────────────────────────────────────
```

---

## 6. Bảng lệnh tra nhanh

| Việc | Lệnh |
|------|------|
| Cất thay đổi | `git stash` |
| Cất kèm file mới | `git stash -u` |
| Cất kèm tên | `git stash push -m "msg"` |
| Lấy lại + xóa | `git stash pop` |
| Lấy lại + giữ | `git stash apply` |
| Xem danh sách | `git stash list` |
| Xem nội dung 1 stash | `git stash show -p stash@{n}` |
| Xóa 1 stash | `git stash drop stash@{n}` |
| Xóa hết | `git stash clear` |
| Tạo branch từ stash | `git stash branch <name>` |

## 7. Tự kiểm tra

1. `pop` và `apply` khác nhau chỗ nào? *(pop xóa khỏi list, apply giữ lại)*
2. `git stash` có cất file untracked không? *(không, phải `-u`)*
3. Stash có gắn với branch không? *(không, là ngăn xếp toàn repo — pop ở branch khác được)*
4. Stack stash hoạt động kiểu gì? *(LIFO, stash@{0} là mới nhất)*

## Liên quan
- [[00-MOC-Git|⬅ MOC Git]]
- Trước: [[05-Branch-Merge-PR]] · Kế tiếp: [[07-Staging-chon-loc|Staging chọn lọc]]
