---
title: "⭐ Cheatsheet lệnh Git — tra nhanh theo tình huống"
section: 00-Foundations/02-Git
tags: [git, cheatsheet, reference, foundations, fresher]
related:
  - "[[00-MOC-Git]]"
difficulty: ⭐
estimated_time: 15m
source: [tổng hợp note 1-13]
---

# ⭐ Cheatsheet lệnh Git — tra nhanh theo tình huống

> [!summary] TL;DR
> Trang tra nhanh gom toàn bộ lệnh từ note 1–13, sắp theo **tình huống thực tế** ("tôi muốn làm X → gõ gì"). Mỗi nhóm có link về note giải thích chi tiết. In/ghim trang này khi ôn gấp.

---

## 1. Cấu hình & khởi tạo → [[02-Install-Configure-Git]]

```bash
git config --global user.name "Tên"
git config --global user.email "email@x.com"
git config --list --show-origin       # xem config & nguồn
git config --global alias.lg "log --oneline --graph --all"

git init                              # tạo repo mới
git clone <url>                       # clone repo về
git remote -v                         # xem remote
git remote add origin <url>           # gắn remote
```

## 2. Làm việc hằng ngày → [[03-Daily-Workflow]]

```bash
git status            git status -s    # trạng thái (ngắn)
git diff                               # khác biệt CHƯA stage
git diff --staged                      # khác biệt ĐÃ stage (sắp commit)
git add <file>        git add .        # stage
git add -p                             # stage từng đoạn → [[07-Staging-chon-loc]]
git restore <file>                     # bỏ sửa đổi working
git restore --staged <file>            # gỡ stage
git commit -m "msg"                    # commit
git commit -am "msg"                   # add(file đã track) + commit
git push origin <branch>               # đẩy lên → [[08-Push-tuong-minh]]
git pull origin <branch>               # fetch + merge
git fetch origin                       # chỉ tải, không gộp
```

## 3. Branch & merge → [[05-Branch-Merge-PR]]

```bash
git branch                git branch -a        # liệt kê (cả remote)
git switch <branch>       git switch -c <new>  # chuyển / tạo+chuyển
git branch -d <branch>                          # xóa (đã merge)
git branch -D <branch>                          # xóa cưỡng bức
git push origin --delete <branch>               # xóa branch remote

git merge <branch>                              # gộp vào branch hiện tại
git merge --no-ff <branch>                      # luôn tạo merge commit
git rebase <branch>                             # rebase (chỉ commit chưa push!)
git merge --abort     git rebase --abort        # hủy

# Conflict: sửa tay, xóa marker <<<<<<< ======= >>>>>>>, rồi:
git add <file>    &&    git commit
```

## 4. Cất tạm (stash) → [[06-Git-Stash]]

```bash
git stash             git stash -u             # cất (kèm file mới)
git stash push -m "tên"                         # cất kèm tên
git stash list                                  # danh sách
git stash pop                                   # lấy lại + xóa
git stash apply                                 # lấy lại + giữ
git stash drop stash@{n}   git stash clear      # xóa
```

## 5. Hoàn tác & sửa lịch sử → [[09-Hoan-tac-va-sua-lich-su]]

> **Hỏi trước: đã PUSH chưa?** Chưa → amend/reset OK. Đã push → chỉ `revert`.

```bash
git commit --amend -m "msg đúng"                # sửa commit cuối (chưa push)
git commit --amend --no-edit                    # thêm file vào commit cuối

git reset --soft HEAD~1                          # bỏ commit, GIỮ staged
git reset HEAD~1                                 # bỏ commit, giữ working (mixed)
git reset --hard HEAD~1                          # ⚠️ XÓA sạch

git revert <hash>                               # đảo ngược commit ĐÃ push (an toàn)

git reflog                                       # cứu commit lỡ mất
```

## 6. Xem lịch sử & dò bug → [[10-Git-Log-nang-cao]]

```bash
git log --oneline --graph --all                 # cây lịch sử
git log --grep="x"     git log -S "x"           # tìm theo message / theo code
git log A..B                                     # B có gì hơn A
git log -- <file>                               # lịch sử 1 file
git show <hash>                                  # soi 1 commit
git blame <file>                                # ai sửa dòng nào
git bisect start / good <ok> / bad              # dò commit gây lỗi
git diff <a> <b>                                # so 2 điểm bất kỳ
```

## 7. Hooks & GitHub → [[11-Git-Hooks-va-meo-GitHub]]

```bash
# pre-commit hook: .git/hooks/pre-commit (exit ≠ 0 = chặn)
git commit --no-verify                          # bỏ qua hook 1 lần
npx husky init                                  # hook chia sẻ được (Node)
pre-commit install                              # hook chia sẻ được (Python)
# Phím "." trên repo GitHub → mở github.dev
# Fork → clone → PR để đóng góp repo người khác
```

## 8. CI/CD & GitHub Actions → [[12-CI-CD-la-gi]] · [[13-GitHub-Actions]]

```yaml
# .github/workflows/ci.yml
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npm test
        env:
          TOKEN: ${{ secrets.TOKEN }}     # KHÔNG hardcode secret
```

---

## 9. "Tôi lỡ tay…" — cứu hộ nhanh

| Tình huống | Cách xử lý |
|------------|------------|
| Commit nhầm message (chưa push) | `git commit --amend -m "đúng"` |
| Quên 1 file trong commit cuối (chưa push) | `git add f && git commit --amend --no-edit` |
| Muốn bỏ commit cuối, giữ code | `git reset --soft HEAD~1` |
| Lỡ `reset --hard`, mất commit | `git reflog` → `git reset --hard <hash>` |
| Commit lỗi đã push lên remote | `git revert <hash>` (KHÔNG reset) |
| Bỏ sửa đổi 1 file chưa commit | `git restore <file>` |
| Đang code dở phải đổi branch | `git stash -u` → switch → `git stash pop` |
| Push bị từ chối (remote mới hơn) | `git pull --rebase` → `git push` |
| Đẩy nhầm branch | luôn push tường minh: `git push origin <branch>` |
| Đã commit `.env` chứa secret | `git rm --cached .env` + ignore + **đổi key ngay** |

---

## 10. Quy ước commit message → [[04-Gitignore-va-Khai-niem-cot-loi]]

```text
feat:     tính năng mới        fix:      sửa bug
docs:     tài liệu             refactor: tái cấu trúc
test:     test                 chore:    việc lặt vặt
perf:     hiệu năng            ci:       sửa pipeline
style:    định dạng
```

## Liên quan
- [[00-MOC-Git|⬅ MOC Git]] — quay lại mục lục
- Toàn bộ note nền tảng: [[01-How-Git-Works]] → [[13-GitHub-Actions]]
