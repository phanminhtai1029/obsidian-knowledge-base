---
title: "CI/CD là gì — Continuous Integration / Delivery / Deployment"
section: 00-Foundations/02-Git
tags: [ci-cd, devops, pipeline, automation, github-actions, foundations, fresher]
related:
  - "[[13-GitHub-Actions]]"
  - "[[11-Git-Hooks-va-meo-GitHub]]"
  - "[[05-Branch-Merge-PR]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 40m
source: [Martin Fowler - Continuous Integration, GitHub docs]
---

# CI/CD là gì — Continuous Integration / Delivery / Deployment

> [!summary] TL;DR
> **CI (Continuous Integration):** tự động **build + test** mỗi khi có code mới (push/PR), để phát hiện lỗi tích hợp **sớm**. **CD** có 2 nghĩa: **Continuous Delivery** = tự động chuẩn bị bản release "sẵn sàng deploy", nhưng bước bấm nút lên production là **thủ công**; **Continuous Deployment** = tự động **deploy thẳng** lên production khi qua hết test, **không cần** ai bấm. **Pipeline** = chuỗi các bước (lint → test → build → deploy) chạy tự động.

---

## 1. Vấn đề CI/CD giải quyết

Ngày xưa, các lập trình viên code riêng nhiều tuần rồi mới gộp lại ("integration hell"): hàng loạt conflict, lỗi tích hợp, bug chỉ lộ ra phút chót. Deploy thì làm tay, dễ sai, đêm hôm căng thẳng.

**CI/CD** = tự động hóa toàn bộ chặng từ "code vừa viết" → "chạy được trên production", với hai mục tiêu: **phát hiện lỗi sớm** và **release nhanh, đều, an toàn**.

```
★ Insight ─────────────────────────────────────
• "Continuous" (liên tục) là tinh thần cốt lõi: gộp & kiểm tra THƯỜNG XUYÊN
  (nhiều lần/ngày) thay vì dồn cục. Lỗi tìm thấy sau 5 phút (CI báo PR đỏ) rẻ
  hơn nhiều lần lỗi tìm thấy sau 3 tuần. Đây cũng là lý do nhánh nên nhỏ & sống
  ngắn ([[05-Branch-Merge-PR]]) — gộp sớm thì CI bắt lỗi sớm.
─────────────────────────────────────────────────
```

---

## 2. Ba khái niệm — phân biệt cho rõ (câu hỏi phỏng vấn)

```text
 Code  →  CI  ─────────────────────────────────────────►  Production
          │                    │                    │
   build + test         tạo artifact          deploy
       (CI)            sẵn-sàng-deploy        lên prod
                       (Cont. Delivery)    (Cont. Deployment)
                              │                    │
                       bấm nút THỦ CÔNG      TỰ ĐỘNG hoàn toàn
                       để lên production
```

| | **Continuous Integration** | **Continuous Delivery** | **Continuous Deployment** |
|---|---|---|---|
| Làm gì | Build + test tự động mỗi commit/PR | Luôn có bản **sẵn sàng** deploy | Deploy thẳng lên prod tự động |
| Bước lên production | (chưa tới) | **Thủ công** (người bấm) | **Tự động** (không ai bấm) |
| Mục tiêu | Bắt lỗi tích hợp sớm | Release lúc nào cũng được, an toàn | Mỗi commit qua test = lên prod ngay |

> [!tip] Mẹo nhớ: **cả hai "CD" giống nhau đến phút chót**. Khác duy nhất ở bước cuối lên production: **Delivery = người bấm nút**, **Deployment = máy tự bấm**. Continuous Deployment là Continuous Delivery + bỏ luôn cái nút thủ công.

---

## 3. Pipeline — chuỗi các bước tự động

**Pipeline** là quy trình gồm nhiều **stage** chạy nối tiếp; một stage fail thì dừng (không deploy code lỗi). Pipeline điển hình:

```text
 ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐  ┌──────────┐
 │ Checkout│→ │  Lint   │→ │  Test   │→ │  Build  │→ │  Deploy   │
 │  code   │  │ (style) │  │(unit/IT)│  │artifact │  │ (staging/ │
 └────────┘  └────────┘  └────────┘  └────────┘  │   prod)   │
                                                  └──────────┘
   ↑ trigger: push / pull request / theo lịch / bấm tay
```

| Stage | Làm gì |
|-------|--------|
| **Checkout** | Lấy code từ repo |
| **Lint / Format** | Kiểm style, bắt lỗi cú pháp sớm |
| **Test** | Chạy unit test, integration test; có thể đo coverage |
| **Build** | Biên dịch / đóng gói (artifact, Docker image) |
| **Deploy** | Đưa lên môi trường (dev → staging → production) |

---

## 4. Hook (local) vs CI (server) — bổ sung cho nhau

Ở [[11-Git-Hooks-va-meo-GitHub]] ta thấy pre-commit hook chặn lỗi **trên máy lập trình viên**. Nhưng hook có thể bị bỏ qua (`--no-verify`) và không phải ai cũng cài. CI là **hàng rào trên server, bắt buộc, không vượt được**.

| | Git Hook (pre-commit) | CI (server) |
|---|----------------------|-------------|
| Chạy ở đâu | Máy cá nhân | Máy chủ CI (GitHub Actions runner) |
| Có thể bỏ qua? | Có (`--no-verify`) | Không |
| Tốc độ phản hồi | Tức thì, trước commit | Sau khi push (phút) |
| Vai trò | Hàng rào sớm, tiện | Hàng rào cuối, đáng tin |

```
★ Insight ─────────────────────────────────────
• Chiến lược "shift left": đẩy việc kiểm tra càng SỚM (về bên trái timeline)
  càng tốt. Hook = sớm nhất (trước commit); CI trên PR = sớm nhì (trước merge).
  Cả hai cùng mục tiêu: đừng để lỗi trôi tới production. Nhưng chỉ CI mới là
  "cổng" thật sự gác việc merge — vì nó không ai tắt được.
─────────────────────────────────────────────────
```

---

## 5. Branch protection — CI gắn với Pull Request

Sức mạnh thật của CI lộ ra khi gắn vào PR ([[05-Branch-Merge-PR]]): GitHub có thể đặt **branch protection rule** cho `main`:

- ❌ Không cho merge nếu **CI fail** (test/lint đỏ).
- ✅ Bắt buộc ít nhất N người **review approve**.
- ✅ Bắt buộc branch **cập nhật** với main trước khi merge.

→ Code lên `main` luôn xanh (qua test) và đã được review. Đây là "luật chơi" giữ chất lượng nhánh chính.

---

## 6. Lợi ích & môi trường

**Lợi ích CI/CD:**
- Phát hiện lỗi sớm, rẻ.
- Release nhanh & thường xuyên, ít rủi ro mỗi lần.
- Giảm thao tác tay → bớt sai sót con người.
- Quy trình lặp lại được, có dấu vết (mỗi run có log).

**Môi trường (environment) thường gặp:**

```text
 dev  →  staging (giống prod, để QA/UAT)  →  production (người dùng thật)
```

> [!tip] Một số công cụ CI/CD phổ biến: **GitHub Actions** (tích hợp sẵn GitHub — sẽ học ở [[13-GitHub-Actions]]), GitLab CI, Jenkins, CircleCI, Travis CI. Khái niệm pipeline/stage giống nhau, chỉ khác cú pháp cấu hình.

---

## 7. Tự kiểm tra

1. CI là viết tắt của gì, làm gì? *(Continuous Integration — build + test tự động mỗi commit/PR)*
2. Continuous Delivery vs Continuous Deployment khác nhau chỗ nào? *(Delivery: bước lên prod thủ công; Deployment: tự động hoàn toàn)*
3. Pipeline gồm các stage điển hình nào? *(checkout → lint → test → build → deploy)*
4. Vì sao CI đáng tin hơn Git hook? *(chạy trên server, không bị --no-verify bỏ qua)*
5. Branch protection dùng CI để làm gì? *(chặn merge khi test đỏ / chưa review)*

## Liên quan
- [[00-MOC-Git|⬅ MOC Git]]
- Trước: [[11-Git-Hooks-va-meo-GitHub]] · Kế tiếp: [[13-GitHub-Actions|GitHub Actions]]
- [[05-Branch-Merge-PR|Pull Request & branch protection]]
