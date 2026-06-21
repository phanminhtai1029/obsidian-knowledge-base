---
title: "Push tường minh — explicit > implicit"
section: 00-Foundations/02-Git
tags: [git, push, upstream, remote, foundations, fresher]
related:
  - "[[03-Daily-Workflow]]"
  - "[[05-Branch-Merge-PR]]"
difficulty: ⭐⭐⭐
estimated_time: 20m
source: ["Git superpowers (Ronnie Sheer)", Git docs]
---

# Push tường minh — explicit > implicit

> [!summary] TL;DR
> Khi push, hãy **nói rõ remote và branch**: `git push origin feature/login`. Gõ trống `git push` (implicit) dựa vào cấu hình mặc định, dễ đẩy nhầm branch — một lỗi tốn rất nhiều thời gian sửa. Lần push đầu của một branch mới, dùng `git push -u origin <branch>` để **set upstream**; các lần sau mới được gõ gọn `git push`. Output của push còn in sẵn **link tạo Pull Request**.

---

## 1. Tại sao nên push tường minh?

Có 2 cách đẩy code:

```bash
git push                          # implicit — dựa vào config mặc định
git push origin feature/login     # explicit — nói rõ remote + branch
```

`git push` trống phụ thuộc vào `push.default` và upstream đã set. Trong cấu hình sai hoặc khi bạn đang đứng nhầm branch, nó có thể đẩy **sai nhánh lên sai nơi** — ví dụ kinh điển: vô tình đẩy `main` đè lên `main` của người khác, gây "giant headache" cho cả team.

```
★ Insight ─────────────────────────────────────
• Triết lý "explicit is better than implicit": gõ thêm 2 từ (remote + branch)
  để loại bỏ MỌI mơ hồ về "đẩy cái gì đi đâu". Vài giây gõ thêm rẻ hơn rất
  nhiều so với một buổi chiều gỡ rối vì push nhầm nhánh chính.
• Push là một trong số ít thao tác Git "đi ra thế giới" (ảnh hưởng remote &
  người khác). Với thao tác có hậu quả ngoài máy mình, mặc định nên là tường
  minh & thận trọng — khác với thao tác local (commit, stash) có thể thoải mái.
─────────────────────────────────────────────────
```

---

## 2. `--set-upstream` (`-u`) — lần push đầu của branch mới

Branch mới tạo ở local **chưa biết** nó tương ứng branch nào trên remote. Lần đầu push phải "ghép đôi":

```bash
git switch -c feature/login       # tạo branch local
# ... commit ...
git push -u origin feature/login  # -u = --set-upstream: đẩy + ghi nhớ "branch này ↔ origin/feature/login"
```

Sau khi đã set upstream, lần sau chỉ cần:

```bash
git push        # Git biết đẩy lên origin/feature/login
git pull        # và kéo về từ đó
```

| Lệnh | Khi nào |
|------|---------|
| `git push -u origin <branch>` | **Lần đầu** push một branch mới |
| `git push origin <branch>` | Push tường minh (không lưu upstream — luôn an toàn) |
| `git push` | Các lần sau, khi đã có upstream (gõ gọn) |

> [!tip] Quên `-u` ở lần đầu? Git sẽ gợi ý đúng lệnh: `git push --set-upstream origin <branch>`. Cứ copy chạy lại.

---

## 3. Mẹo: link tạo Pull Request in sẵn trong output

Sau khi push một branch mới, GitHub in ra link để mở PR ngay:

```text
remote: Create a pull request for 'feature/login' on GitHub by visiting:
remote:   https://github.com/user/repo/pull/new/feature/login
```

Copy link đó mở thẳng trang tạo PR — khỏi vào web bấm qua nhiều bước. (PR là gì: xem [[05-Branch-Merge-PR]].)

---

## 4. Vài tình huống push hay gặp

```bash
# Đẩy branch local lên remote với tên khác
git push origin local-name:remote-name

# Xóa branch trên remote
git push origin --delete feature/login

# Đẩy tag
git push origin v1.0.0
git push origin --tags            # đẩy tất cả tag

# Bị từ chối vì remote có commit mới hơn → KÉO VỀ trước rồi đẩy lại
git pull --rebase origin main
git push origin main
```

> [!warning] **Tuyệt đối tránh `git push --force` lên branch chung** (main/develop): nó **ghi đè** lịch sử remote, xóa commit của người khác. Nếu buộc phải force (vd sau rebase nhánh riêng của mình), dùng **`git push --force-with-lease`** — nó từ chối push nếu remote có commit bạn chưa thấy, an toàn hơn hẳn. Xem [[09-Hoan-tac-va-sua-lich-su]].

---

## 5. Bảng lệnh tra nhanh

| Việc | Lệnh |
|------|------|
| Push lần đầu (set upstream) | `git push -u origin <branch>` |
| Push tường minh | `git push origin <branch>` |
| Push (đã có upstream) | `git push` |
| Xóa branch remote | `git push origin --delete <branch>` |
| Push tag | `git push origin <tag>` / `--tags` |
| Force an toàn | `git push --force-with-lease` |

## 6. Tự kiểm tra

1. Vì sao nên push tường minh thay vì `git push` trống? *(tránh đẩy nhầm branch/nơi)*
2. `-u` (`--set-upstream`) làm gì? *(ghép branch local ↔ remote, lần sau push/pull gõ gọn)*
3. Push bị từ chối vì remote mới hơn — làm gì? *(`git pull --rebase` rồi push lại)*
4. Force push an toàn hơn dùng cờ nào? *(`--force-with-lease`)*

## Liên quan
- [[00-MOC-Git|⬅ MOC Git]]
- Trước: [[07-Staging-chon-loc]] · Kế tiếp: [[09-Hoan-tac-va-sua-lich-su|Hoàn tác & sửa lịch sử]]
- [[05-Branch-Merge-PR|Pull Request]]
