---
title: "Staging chọn lọc — 3 cách add & git add -p"
section: 00-Foundations/02-Git
tags: [git, add, staging, patch, hunk, foundations, fresher]
related:
  - "[[03-Daily-Workflow]]"
  - "[[01-How-Git-Works]]"
difficulty: ⭐⭐⭐
estimated_time: 25m
source: ["Git superpowers (Ronnie Sheer)", Git docs]
---

# Staging chọn lọc — 3 cách add & `git add -p`

> [!summary] TL;DR
> Có 3 cách đưa code vào commit, từ "nhanh & ẩu" đến "chậm & an toàn": (1) `git commit -am` (add + commit gộp), (2) `git add .` (stage tất cả), (3) **`git add -p`** (duyệt **từng đoạn**, gõ y/n cho mỗi thay đổi). Cách 3 tốn thêm vài giây nhưng chặn được những thứ "lọt" vào commit ngoài ý muốn: `console.log` debug, comment thừa, file rác.

---

## 1. Ba cách add — từ ẩu đến cẩn thận

| Cách | Lệnh | Mức kiểm soát | Rủi ro |
|------|------|---------------|--------|
| 1. Gộp luôn | `git commit -am "msg"` | Thấp nhất | Dễ commit nhầm file đã track; bỏ sót file mới |
| 2. Stage tất cả | `git add .` rồi `git commit` | Trung bình | Lỡ stage cả debug code, file tạm |
| 3. Stage từng đoạn | `git add -p` rồi `git commit` | Cao nhất | Chậm hơn vài giây |

```bash
# Cách 1 — nhanh nhất, ít kiểm soát nhất
git commit -am "fix: sửa validate"   # tự add file ĐÃ track + commit (bỏ qua file mới!)

# Cách 2 — phổ biến
git add .
git commit -m "feat: thêm trang chủ"

# Cách 3 — khuyến nghị khi cần kỹ
git add -p
git commit -m "feat: thêm trang chủ"
```

> [!warning] `git commit -am` **không** add file untracked (file mới). Nếu commit "thiếu file mới", thủ phạm thường là `-am`. Với file mới phải `git add` rõ ràng trước.

```
★ Insight ─────────────────────────────────────
• "Explicit is better than implicit" (triết lý Python) áp dụng rất đúng cho
  staging. Tiết kiệm vài giây bằng `commit -am` hôm nay có thể đẻ ra bug khó
  tìm ngày mai vì một dòng debug lọt vào production. Đội ngũ giàu kinh nghiệm
  thường mặc định dùng `add -p` để "soi" lại đúng những gì mình đưa vào lịch sử.
─────────────────────────────────────────────────
```

---

## 2. `git add -p` — stage từng đoạn (hunk)

`-p` = `--patch`. Git chia thay đổi thành các **hunk** (đoạn) và hỏi bạn xử lý từng cái:

```bash
git add -p
```

```text
@@ -10,6 +10,8 @@ function login() {
   const user = getUser();
+  console.log(user);          ← bạn KHÔNG muốn dòng debug này vào commit
   return validate(user);
+  // TODO: thêm rate limit
 }
Stage this hunk [y,n,q,a,d,s,e,?]?
```

### Các phím trả lời quan trọng

| Phím | Ý nghĩa |
|------|---------|
| `y` | **yes** — stage hunk này |
| `n` | **no** — bỏ qua hunk này (không stage) |
| `s` | **split** — tách hunk lớn thành các hunk nhỏ hơn để chọn tinh hơn |
| `e` | **edit** — sửa tay đúng những dòng muốn stage |
| `q` | **quit** — thoát, không xét tiếp |
| `a` | stage hunk này và **tất cả** hunk còn lại của file |
| `d` | bỏ qua hunk này và tất cả hunk còn lại của file |
| `?` | hiện trợ giúp |

> [!tip] Khi một hunk gộp cả code tốt lẫn code muốn loại, bấm **`s`** để Git tách nhỏ ra; nếu vẫn dính nhau thì bấm **`e`** để sửa tay (xóa các dòng `+`/`-` không muốn stage).

---

## 3. Kiểm tra & gỡ stage

```bash
git status              # file nào đã staged, file nào còn modified
git diff                # phần CHƯA staged (cái add -p đã bỏ lại)
git diff --staged       # phần SẼ vào commit — nên xem lại lần cuối trước khi commit

git restore --staged file   # gỡ 1 file khỏi stage (giữ sửa đổi)
git reset file              # cách cũ, tương đương
```

```
★ Insight ─────────────────────────────────────
• Sau `git add -p`, một file thường ở trạng thái VỪA staged VỪA modified: đoạn
  bạn chọn (y) đã vào Staging, đoạn bỏ (n) còn ở Working Dir. Đây chính là minh
  họa sống động cho mô hình 3 vùng ở [[01-How-Git-Works]] — staging cho phép
  một file "đứng hai chân" ở hai vùng.
• `git diff --staged` là bước "đọc lại bản nháp trước khi gửi": luôn liếc nó
  trước khi commit để chắc đúng những gì mình định đưa vào lịch sử.
─────────────────────────────────────────────────
```

---

## 4. Thử thách thực hành (từ khoá gốc)

> Sửa **2 file**, nhưng chỉ commit thay đổi của **1 file**, để file kia lại.

```bash
# Đã sửa index.html và style.css
git add -p
#   hunk của style.css  → n  (bỏ qua)
#   hunk của index.html → y  (stage)
git status
#   modified: index.html   (staged, xanh — sẽ commit)
#   modified: style.css    (đỏ — KHÔNG commit)
git commit -m "feat: cập nhật trang chủ"
# style.css vẫn còn modified & unstaged, để dành commit sau
```

---

## 5. Bảng lệnh tra nhanh

| Việc | Lệnh |
|------|------|
| Add + commit file đã track | `git commit -am "msg"` |
| Stage tất cả | `git add .` / `git add -A` |
| Stage từng đoạn | `git add -p` |
| Stage 1 file | `git add <file>` |
| Gỡ stage | `git restore --staged <file>` |
| Xem phần sắp commit | `git diff --staged` |

## 6. Tự kiểm tra

1. `git commit -am` bỏ sót loại file nào? *(file untracked/mới)*
2. Trong `add -p`, phím `s` và `e` để làm gì? *(s: tách hunk; e: sửa tay đoạn stage)*
3. Sau `add -p` chọn vài hunk, file ở trạng thái gì? *(vừa staged vừa modified)*
4. Lệnh nào nên xem trước khi commit để chắc nội dung? *(`git diff --staged`)*

## Liên quan
- [[00-MOC-Git|⬅ MOC Git]]
- Trước: [[06-Git-Stash]] · Kế tiếp: [[08-Push-tuong-minh|Push tường minh]]
- [[01-How-Git-Works|Mô hình 3 vùng]]
