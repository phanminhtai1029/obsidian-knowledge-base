---
title: "GitHub Actions — workflow YAML & pipeline thực tế"
section: 00-Foundations/02-Git
tags: [github-actions, ci-cd, workflow, yaml, automation, secrets, foundations, fresher]
related:
  - "[[12-CI-CD-la-gi]]"
  - "[[05-Branch-Merge-PR]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 50m
source: [GitHub Actions docs]
---

# GitHub Actions — workflow YAML & pipeline thực tế

> [!summary] TL;DR
> **GitHub Actions** là CI/CD tích hợp sẵn trong GitHub. Bạn viết **workflow** dạng file YAML đặt ở `.github/workflows/`. Một workflow gồm: **trigger** (`on:` — push, pull_request, schedule…) → các **job** (chạy song song trên **runner**) → mỗi job có nhiều **step** (chạy lệnh shell hoặc dùng **action** có sẵn qua `uses:`). Bí mật để trong **Secrets**, dùng qua `${{ secrets.TÊN }}` — không bao giờ hardcode.

---

## 1. Cấu trúc phân cấp — nắm sơ đồ này là hiểu hết

```text
Workflow (1 file .yml)
 └─ on: <trigger>              ← khi nào chạy
 └─ jobs:
     ├─ job-1  (chạy trên 1 runner)
     │   └─ steps:
     │       ├─ step A: uses: <action có sẵn>
     │       └─ step B: run: <lệnh shell>
     └─ job-2  (mặc định chạy SONG SONG với job-1)
         └─ steps: ...
```

| Khái niệm | Là gì |
|-----------|-------|
| **Workflow** | 1 file YAML trong `.github/workflows/`, mô tả 1 quy trình tự động |
| **Event / Trigger** (`on`) | Sự kiện kích hoạt: `push`, `pull_request`, `schedule`, `workflow_dispatch` (bấm tay)… |
| **Job** | Một nhóm step chạy trên cùng 1 **runner**. Các job **song song** trừ khi khai báo `needs` |
| **Runner** | Máy ảo chạy job (`ubuntu-latest`, `windows-latest`, `macos-latest`) |
| **Step** | Một bước trong job: hoặc `run:` (lệnh shell), hoặc `uses:` (gọi action có sẵn) |
| **Action** | Đơn vị tái sử dụng đóng gói sẵn, vd `actions/checkout`, `actions/setup-node` |

```
★ Insight ─────────────────────────────────────
• Phân biệt then chốt giữa 2 loại step: `run:` là BẠN gõ lệnh shell (npm test);
  `uses:` là MƯỢN một action người khác viết sẵn (actions/checkout để lấy code).
  Hầu hết workflow là vài `uses:` (setup môi trường) + vài `run:` (chạy việc của bạn).
• Job mặc định SONG SONG và mỗi job khởi động trên một runner SẠCH, MỚI TINH —
  không chia sẻ file với nhau. Muốn job B chạy sau A và dùng kết quả của A thì
  phải khai báo `needs: A` + truyền dữ liệu qua artifact/output, chứ không tự có.
─────────────────────────────────────────────────
```

---

## 2. Workflow tối giản — "Hello CI"

File `.github/workflows/ci.yml`:

```yaml
name: CI                       # tên hiển thị ở tab Actions

on:                            # TRIGGER: chạy khi nào
  push:
    branches: [main]
  pull_request:                # chạy cả khi mở PR

jobs:
  build:                       # tên job (tự đặt)
    runs-on: ubuntu-latest     # runner
    steps:
      - name: Lấy code
        uses: actions/checkout@v4      # action lấy source về runner

      - name: In lời chào
        run: echo "Hello CI! Commit $GITHUB_SHA"
```

> [!warning] YAML **rất nhạy với thụt lề** (dùng **space**, không tab) và dấu hai chấm. Sai 1 khoảng trắng là workflow không chạy. Khi lỗi, kiểm tra indent đầu tiên.

---

## 3. Pipeline thực tế — Node.js: lint → test → build

```yaml
name: CI Node

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Cài Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'              # cache node_modules cho nhanh

      - name: Cài dependencies
        run: npm ci                 # ci = cài sạch theo package-lock (chuẩn cho CI)

      - name: Lint
        run: npm run lint

      - name: Test
        run: npm test

      - name: Build
        run: npm run build
```

### Chạy nhiều phiên bản cùng lúc — `matrix`

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:                      # tạo nhiều job từ tổ hợp
        os: [ubuntu-latest, windows-latest]
        node: [18, 20]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm ci && npm test
# → sinh 4 job: (ubuntu,18) (ubuntu,20) (win,18) (win,20) chạy song song
```

```
★ Insight ─────────────────────────────────────
• `npm ci` ≠ `npm install`: ci cài CHÍNH XÁC theo package-lock.json và xóa
  node_modules trước → tái lập 100%, nhanh hơn, hợp CI. install có thể cập nhật
  lock file (không mong muốn trên CI).
• matrix là "vòng lặp tạo job": một khai báo ngắn đẻ ra N job song song để test
  trên nhiều OS/phiên bản — bắt lỗi "chạy được trên máy tôi nhưng hỏng ở Windows"
  trước khi tới người dùng.
─────────────────────────────────────────────────
```

---

## 4. Job phụ thuộc nhau — `needs` (build → deploy)

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test

  deploy:
    needs: test                  # CHỈ chạy nếu job "test" THÀNH CÔNG
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'   # chỉ deploy khi ở nhánh main
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        run: ./deploy.sh
        env:
          API_TOKEN: ${{ secrets.DEPLOY_TOKEN }}   # lấy từ Secrets
```

`needs` biến các job song song thành chuỗi có thứ tự → đúng tinh thần pipeline ở [[12-CI-CD-la-gi]].

---

## 5. Secrets — KHÔNG BAO GIỜ hardcode khóa

Token, mật khẩu, API key để trong **GitHub → Settings → Secrets and variables → Actions**, rồi dùng qua biểu thức:

```yaml
env:
  DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
```

> [!warning] **Tuyệt đối không** viết thẳng token vào file YAML (file YAML nằm trong repo, ai cũng đọc được — nhất là repo public). Luôn dùng `${{ secrets.* }}`. GitHub tự **che (mask)** giá trị secret trong log. `GITHUB_TOKEN` là secret đặc biệt GitHub tự cấp cho mỗi run để thao tác với chính repo đó.

```
★ Insight ─────────────────────────────────────
• Đây chính là lý do bài học bảo mật xuyên suốt: secret vào repo = lộ vĩnh viễn
  ([[04-Gitignore-va-Khai-niem-cot-loi]] — .env phải ignore). Trên CI, "đường
  đúng" để code dùng secret mà không lộ là cơ chế Secrets + biến môi trường lúc
  chạy, không phải nhét vào source.
─────────────────────────────────────────────────
```

---

## 6. Biến môi trường & ngữ cảnh (context) hay dùng

```yaml
steps:
  - run: echo "Commit ${{ github.sha }} trên nhánh ${{ github.ref_name }}"
  - run: echo "Người đẩy ${{ github.actor }}"
```

| Biểu thức | Ý nghĩa |
|-----------|---------|
| `${{ github.sha }}` | Hash commit đang chạy |
| `${{ github.ref_name }}` | Tên branch/tag |
| `${{ github.actor }}` | Người kích hoạt |
| `${{ secrets.X }}` | Secret tên X |
| `${{ github.event_name }}` | Loại event (push/pull_request…) |

---

## 7. Các trigger (`on`) thường dùng

```yaml
on:
  push:
    branches: [main, develop]
    paths: ['src/**']           # chỉ chạy khi file trong src đổi
  pull_request:                 # mở/cập nhật PR
  schedule:
    - cron: '0 2 * * *'         # 2h sáng mỗi ngày (cú pháp cron)
  workflow_dispatch:            # nút "Run workflow" bấm tay trên UI
```

---

## 8. Xem kết quả & badge

- Tab **Actions** trên repo: mỗi lần trigger là một **run**, xem log từng step, log fail tô đỏ.
- Trên **PR**, kết quả CI hiện dưới dạng check ✅/❌ — gắn branch protection để chặn merge khi đỏ ([[12-CI-CD-la-gi]]).
- Gắn **badge** trạng thái vào README:
  ```markdown
  ![CI](https://github.com/user/repo/actions/workflows/ci.yml/badge.svg)
  ```

---

## 9. Bảng tra nhanh

| Khái niệm | Từ khóa YAML |
|-----------|--------------|
| Khi nào chạy | `on:` (`push`, `pull_request`, `schedule`, `workflow_dispatch`) |
| Máy chạy | `runs-on: ubuntu-latest` |
| Lấy code | `uses: actions/checkout@v4` |
| Setup môi trường | `uses: actions/setup-node@v4` (hoặc setup-python…) |
| Chạy lệnh shell | `run: <lệnh>` |
| Nhiều phiên bản | `strategy: matrix:` |
| Job chạy sau job khác | `needs: <job>` |
| Chạy có điều kiện | `if: <biểu thức>` |
| Dùng secret | `${{ secrets.TÊN }}` |
| Biến môi trường | `env:` |

## 10. Tự kiểm tra

1. Workflow đặt ở thư mục nào? *(`.github/workflows/`)*
2. `run:` vs `uses:` khác gì? *(run: lệnh shell của bạn; uses: gọi action có sẵn)*
3. Các job mặc định chạy song song hay tuần tự? *(song song; dùng `needs` để tuần tự)*
4. `npm ci` khác `npm install`? *(ci cài chính xác theo lock, sạch, hợp CI)*
5. Dùng token trong workflow thế nào cho an toàn? *(để ở Secrets, gọi qua `${{ secrets.* }}`, không hardcode)*

## Liên quan
- [[00-MOC-Git|⬅ MOC Git]]
- Trước: [[12-CI-CD-la-gi]] · Kế tiếp (Phần D): [[14-Git-Cheatsheet|⭐ Cheatsheet]]
- [[11-Git-Hooks-va-meo-GitHub|Git Hooks]] · [[05-Branch-Merge-PR|Branch protection]]
