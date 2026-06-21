---
title: "Install & Configure Git — Cài đặt & cấu hình"
section: 00-Foundations/02-Git
tags: [git, install, config, ssh, github, foundations, fresher]
related:
  - "[[01-How-Git-Works]]"
  - "[[03-Daily-Workflow]]"
difficulty: ⭐⭐
estimated_time: 30m
source: [Git docs, GitHub docs]
---

# Install & Configure Git — Cài đặt & cấu hình

> [!summary] TL;DR
> Cài Git → cấu hình **danh tính** (`user.name`, `user.email`) vì mỗi commit gắn tên/email người tạo. Config có **3 cấp**: `--system` (cả máy) → `--global` (user hiện tại, hay dùng nhất) → `--local` (riêng 1 repo). Để push lên GitHub không phải gõ mật khẩu liên tục, dùng **SSH key** (hoặc Personal Access Token cho HTTPS). GitHub đã **bỏ xác thực bằng mật khẩu** từ 2021.

---

## 1. Cài đặt Git

| Hệ điều hành | Cách cài |
|--------------|----------|
| **Windows** | Tải [git-scm.com](https://git-scm.com) → cài "Git for Windows" (kèm Git Bash) |
| **macOS** | `brew install git` (Homebrew) hoặc `xcode-select --install` |
| **Ubuntu/Debian** | `sudo apt update && sudo apt install git` |
| **Fedora** | `sudo dnf install git` |

Kiểm tra cài thành công:

```bash
git --version      # vd: git version 2.43.0
```

---

## 2. Cấu hình danh tính (bắt buộc)

Mỗi commit ghi lại **ai** tạo. Phải set trước khi commit đầu tiên, nếu không Git sẽ cảnh báo.

```bash
git config --global user.name  "Phan Minh Tai"
git config --global user.email "phanminhtai1029@gmail.com"
```

> [!warning] Email nên trùng với email tài khoản GitHub thì commit mới được "gắn" đúng avatar và tính vào contribution graph. Nếu muốn giấu email thật, GitHub cấp email dạng `ID+username@users.noreply.github.com`.

---

## 3. Ba cấp config — quan trọng cho phỏng vấn

Git đọc config theo thứ tự, cấp **càng cụ thể càng thắng** (local đè global đè system):

| Cấp | Lệnh | File | Phạm vi |
|-----|------|------|---------|
| **System** | `git config --system` | `/etc/gitconfig` | Mọi user trên máy |
| **Global** | `git config --global` | `~/.gitconfig` | User hiện tại (mọi repo) — **hay dùng nhất** |
| **Local** | `git config --local` | `<repo>/.git/config` | Chỉ repo đang đứng |

Ví dụ dùng local để override: trong repo công ty dùng email công ty, còn global là email cá nhân.

```bash
cd ~/work/company-project
git config --local user.email "tai.phan@company.com"   # chỉ repo này
```

Xem config hiện hành (kèm nguồn từ file nào):

```bash
git config --list --show-origin
git config user.email          # xem 1 giá trị cụ thể
```

```
★ Insight ─────────────────────────────────────
• Cơ chế "cấp cụ thể đè cấp chung" giống hệt CSS specificity hay cách OS đọc
  biến môi trường: định nghĩa gần nhất thắng. Khi commit ra "sai tên/email",
  90% là do quên set --local cho repo công ty → commit dính email cá nhân.
• `--show-origin` là cứu cánh khi debug "tại sao config này lại thế?" — nó chỉ
  ra ĐÚNG file đang quyết định giá trị, khỏi đoán mò.
─────────────────────────────────────────────────
```

---

## 4. Vài config nên đặt ngay

```bash
# Trình soạn thảo mặc định (cho commit message, rebase...)
git config --global core.editor "code --wait"     # VS Code; hoặc "vim", "nano"

# Branch mặc định khi git init là "main" (thay vì "master")
git config --global init.defaultBranch main

# Màu cho output dễ đọc
git config --global color.ui auto

# Xử lý ký tự xuống dòng (CRLF) — tránh "cả file thay đổi" giữa Win/Mac/Linux
git config --global core.autocrlf input    # macOS/Linux
# git config --global core.autocrlf true    # Windows

# Bí danh (alias) gõ nhanh
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.lg "log --oneline --graph --all"
```

> [!tip] Sau khi đặt alias `lg`, gõ `git lg` là ra cây lịch sử đẹp — xem [[10-Git-Log-nang-cao]].

---

## 5. Kết nối GitHub: SSH vs HTTPS

GitHub **không còn cho xác thực bằng mật khẩu** (từ 8/2021). Hai cách phổ biến:

| | SSH | HTTPS + Token |
|---|-----|----------------|
| URL remote | `git@github.com:user/repo.git` | `https://github.com/user/repo.git` |
| Xác thực | Cặp khóa SSH (public/private) | **Personal Access Token (PAT)** thay mật khẩu |
| Trải nghiệm | Cài 1 lần, push/pull khỏi nhập gì | Dán token (nên dùng credential helper để nhớ) |
| Hợp với | Máy cá nhân dùng lâu dài | CI, máy tạm, hoặc qua tường lửa chặn SSH |

### Tạo SSH key (khuyến nghị cho máy cá nhân)

```bash
# 1. Sinh khóa (ed25519 hiện đại hơn rsa)
ssh-keygen -t ed25519 -C "phanminhtai1029@gmail.com"
#   Enter để chọn đường dẫn mặc định ~/.ssh/id_ed25519
#   Đặt passphrase (tùy chọn nhưng nên có)

# 2. Khởi động ssh-agent và nạp khóa
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 3. Copy KHÓA CÔNG KHAI (.pub) để dán lên GitHub
cat ~/.ssh/id_ed25519.pub
```

Dán nội dung `.pub` vào **GitHub → Settings → SSH and GPG keys → New SSH key**. Kiểm tra:

```bash
ssh -T git@github.com      # "Hi <username>! You've successfully authenticated..."
```

> [!warning] **Chỉ dán file `.pub` (public key)**. File `id_ed25519` (KHÔNG có `.pub`) là **private key** — tuyệt đối không chia sẻ, không commit lên repo. Lộ private key = người khác mạo danh bạn.

### Dùng HTTPS + PAT

Tạo token tại **GitHub → Settings → Developer settings → Personal access tokens**. Khi `git push`, nhập username + dán **token** vào ô password. Để Git nhớ token:

```bash
git config --global credential.helper store      # lưu plaintext (đơn giản, kém an toàn)
# hoặc trên macOS: credential.helper osxkeychain
# Windows: credential.helper manager
```

```
★ Insight ─────────────────────────────────────
• SSH key giải quyết bài toán "xác thực không mật khẩu" bằng mã hóa bất đối xứng:
  bạn giữ private key, GitHub giữ public key; GitHub thách thức, chỉ ai có
  private key mới trả lời đúng. Không có mật khẩu nào truyền qua mạng.
• PAT là "mật khẩu phạm vi hẹp, hết hạn được": bạn cấp đúng quyền (repo, workflow)
  và đặt ngày hết hạn → lộ token cũng giới hạn thiệt hại. CI/CD luôn dùng token,
  không dùng SSH cá nhân — xem [[13-GitHub-Actions]].
─────────────────────────────────────────────────
```

---

## 6. Bắt đầu một repo

```bash
# Cách 1: tạo repo mới từ thư mục có sẵn
cd my-project
git init                       # tạo .git/
git add .
git commit -m "init: khởi tạo project"
git remote add origin git@github.com:user/my-project.git
git push -u origin main

# Cách 2: clone repo đã có trên GitHub về
git clone git@github.com:user/repo.git
```

> [!tip] `git remote -v` để xem các remote đang trỏ tới đâu. `origin` là tên quy ước cho remote chính.

---

## 7. Tự kiểm tra

1. 3 cấp config là gì, cấp nào đè cấp nào? *(local > global > system)*
2. Vì sao phải set `user.email`? *(gắn danh tính vào commit; trùng email GitHub để tính contribution)*
3. File nào KHÔNG bao giờ được chia sẻ trong cặp SSH key? *(private key — file không có `.pub`)*
4. GitHub còn cho login bằng mật khẩu khi push không? *(không, dùng SSH hoặc PAT)*
5. `git remote add origin <url>` để làm gì? *(gắn repo local với remote tên origin)*

## Liên quan
- [[00-MOC-Git|⬅ MOC Git]]
- Trước: [[01-How-Git-Works]] · Kế tiếp: [[03-Daily-Workflow|Workflow hằng ngày]]
- [[11-Git-Hooks-va-meo-GitHub|Mẹo GitHub]] · [[13-GitHub-Actions|GitHub Actions]]
