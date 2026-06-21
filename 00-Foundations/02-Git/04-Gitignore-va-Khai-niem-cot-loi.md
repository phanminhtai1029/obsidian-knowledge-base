---
title: ".gitignore & khái niệm cốt lõi"
section: 00-Foundations/02-Git
tags: [git, gitignore, commit-message, conventional-commits, foundations, fresher]
related:
  - "[[01-How-Git-Works]]"
  - "[[03-Daily-Workflow]]"
difficulty: ⭐⭐
estimated_time: 35m
source: [Git docs, conventionalcommits.org]
---

# .gitignore & khái niệm cốt lõi

> [!summary] TL;DR
> **`.gitignore`** liệt kê các file/thư mục Git **bỏ qua** (không track): `node_modules/`, `.env`, file build, log… Lưu ý: `.gitignore` **chỉ chặn file chưa được track** — file đã commit rồi thì thêm vào ignore vô tác dụng, phải `git rm --cached`. **Commit message tốt** theo Conventional Commits giúp lịch sử đọc được & tự sinh changelog. Thư mục `.git/` là "bộ não" chứa toàn bộ lịch sử.

---

## 1. `.gitignore` — bỏ qua file không cần track

File `.gitignore` (đặt ở gốc repo, hoặc trong thư mục con) khai báo những gì Git **không** theo dõi.

### Tại sao cần?

- **Không commit file sinh tự động:** `node_modules/`, `dist/`, `build/`, `__pycache__/`, `.venv/` — tải/được tạo lại được, commit chỉ làm phình repo.
- **Không commit bí mật:** `.env`, file chứa API key, mật khẩu, token.
- **Không commit rác cá nhân:** `.DS_Store` (macOS), `*.log`, file của IDE (`.idea/`, `.vscode/`).

### Cú pháp

```gitignore
# Comment bắt đầu bằng #

node_modules/        # bỏ qua cả thư mục (dấu / cuối = thư mục)
*.log                # mọi file .log (* = wildcard)
.env                 # file cụ thể
.env.*               # .env.local, .env.production...
!.env.example        # ! = NGOẠI LỆ, vẫn track file này dù khớp luật trên
build/               # thư mục build
/secret.txt          # / đầu = chỉ ở gốc repo, không áp dụng thư mục con
temp/**/cache        # ** = khớp nhiều cấp thư mục
```

| Mẫu | Khớp |
|-----|------|
| `*.log` | mọi file kết thúc `.log`, mọi cấp |
| `build/` | thư mục tên `build` ở bất kỳ đâu |
| `/build/` | chỉ thư mục `build` ở **gốc** repo |
| `!keep.log` | **không** bỏ qua `keep.log` (đảo luật trước đó) |
| `doc/*.txt` | file `.txt` ngay trong `doc/` (không gồm thư mục con) |
| `doc/**/*.txt` | file `.txt` trong `doc/` và mọi thư mục con |

> [!warning] **Bẫy kinh điển:** `.gitignore` **chỉ áp dụng cho file CHƯA được track**. Nếu bạn đã `git add .env` và commit, thì sau đó thêm `.env` vào `.gitignore` cũng **vô tác dụng** — Git vẫn theo dõi nó. Phải gỡ khỏi tracking:
> ```bash
> git rm --cached .env          # gỡ khỏi Git nhưng GIỮ file trên ổ đĩa
> echo ".env" >> .gitignore
> git commit -m "chore: stop tracking .env"
> ```
> ⚠️ Nếu `.env` đã từng được push lên remote, secret **vẫn nằm trong lịch sử** — phải coi như đã lộ và **đổi key ngay**.

```
★ Insight ─────────────────────────────────────
• .gitignore là "luật phòng ngừa", không phải "luật dọn dẹp": nó ngăn file MỚI
  lọt vào, chứ không gỡ thứ đã lỡ track. Quy tắc vàng: tạo .gitignore NGAY khi
  init repo, trước commit đầu tiên, để không bao giờ phải `git rm --cached`.
• Bí mật (secret) một khi đã vào lịch sử commit thì coi như công khai vĩnh viễn
  (ai clone cũng thấy bằng `git log`). Xóa file ở commit mới KHÔNG xóa nó khỏi
  commit cũ. Cách duy nhất thật sự: đổi secret + rewrite history (git filter-repo).
─────────────────────────────────────────────────
```

### Lấy mẫu .gitignore sẵn

[github.com/github/gitignore](https://github.com/github/gitignore) có mẫu cho mọi ngôn ngữ (Node, Python, Java…). Trên GitHub khi tạo repo cũng chọn được template.

---

## 2. Thư mục `.git/` — "bộ não" của repo

Khi `git init`, Git tạo `.git/` chứa **toàn bộ** dữ liệu version control:

| Thành phần | Vai trò |
|-----------|---------|
| `.git/objects/` | Mọi snapshot (blob/tree/commit) — xem [[01-How-Git-Works]] |
| `.git/refs/` | Các con trỏ branch & tag |
| `.git/HEAD` | Con trỏ tới branch/commit hiện tại |
| `.git/config` | Config cấp **local** của repo |
| `.git/index` | Chính là **Staging Area** |

> [!warning] Đây là thư mục ẩn (bắt đầu bằng dấu chấm). Xóa `.git/` = biến repo thành thư mục thường, **mất sạch lịch sử**.

---

## 3. Commit message tốt — Conventional Commits

Commit message là "nhật ký dự án". Message tốt giúp đồng đội (và chính bạn sau 3 tháng) hiểu **tại sao** một thay đổi xảy ra.

### Cấu trúc Conventional Commits

```text
<type>(<scope tùy chọn>): <mô tả ngắn>

<thân: giải thích TẠI SAO, tùy chọn>

<footer: BREAKING CHANGE, link issue... tùy chọn>
```

Ví dụ:

```text
feat(auth): thêm đăng nhập bằng Google OAuth

Người dùng yêu cầu đăng nhập nhanh. Dùng passport-google-oauth20.

Closes #42
```

### Các `type` thông dụng

| Type | Dùng khi |
|------|----------|
| `feat` | Thêm tính năng mới |
| `fix` | Sửa bug |
| `docs` | Chỉ sửa tài liệu |
| `style` | Định dạng (khoảng trắng, dấu `;`), không đổi logic |
| `refactor` | Tái cấu trúc code, không thêm feature/sửa bug |
| `perf` | Cải thiện hiệu năng |
| `test` | Thêm/sửa test |
| `chore` | Việc lặt vặt (build, deps, cấu hình) |
| `ci` | Sửa pipeline CI/CD |

### Nguyên tắc viết

- **Dòng tiêu đề ≤ ~50 ký tự**, viết ở thể mệnh lệnh ("thêm", "sửa" — không phải "đã thêm").
- Tách **tiêu đề** và **thân** bằng 1 dòng trống.
- Thân giải thích **tại sao**, không phải **cái gì** (cái gì đã có trong diff).
- 1 commit = 1 mục đích logic (atomic). Đừng gộp "sửa bug + đổi tên biến + thêm feature" vào 1 commit.

```
★ Insight ─────────────────────────────────────
• Conventional Commits không chỉ để đẹp: vì type được chuẩn hóa, công cụ
  (semantic-release, changelog generator) tự đọc được để TỰ ĐỘNG tăng version
  (feat → minor, fix → patch, BREAKING CHANGE → major) và sinh changelog. Đây là
  cầu nối tới tự động hóa release trong [[12-CI-CD-la-gi]].
• Commit atomic (1 việc/commit) làm `git revert` và `git bisect` (dò bug) hiệu
  quả: revert đúng 1 thay đổi mà không kéo theo thứ khác — xem [[09-Hoan-tac-va-sua-lich-su]].
─────────────────────────────────────────────────
```

---

## 4. Vài khái niệm hay nhầm

| Thuật ngữ | Nghĩa |
|-----------|-------|
| **Tracked** | File Git đang theo dõi (đã từng được add/commit) |
| **Untracked** | File Git chưa biết tới |
| **Staged** | Đã `add`, chờ vào commit kế |
| **origin** | Tên quy ước của remote chính (không phải từ khóa bắt buộc) |
| **main / master** | Tên branch mặc định (`main` là tên mới chuẩn) |
| **HEAD** | Con trỏ tới commit/branch hiện tại |

---

## 5. Tự kiểm tra

1. Đã commit `.env` rồi mới thêm vào `.gitignore` — có hết bị track không? *(không; phải `git rm --cached`)*
2. `!file.txt` trong `.gitignore` nghĩa gì? *(ngoại lệ — vẫn track file đó)*
3. Type commit cho "sửa bug" và "thêm tính năng"? *(`fix`, `feat`)*
4. Vì sao thân commit nên nói "tại sao" hơn "cái gì"? *(cái gì đã nằm trong diff; tại sao mới là thông tin mất đi)*
5. Xóa `.git/` mất gì? *(toàn bộ lịch sử)*

## Liên quan
- [[00-MOC-Git|⬅ MOC Git]]
- Trước: [[03-Daily-Workflow]] · Kế tiếp: [[05-Branch-Merge-PR|Branch · Merge · PR]]
- [[09-Hoan-tac-va-sua-lich-su]] · [[12-CI-CD-la-gi]]
