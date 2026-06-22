---
title: "Continuous Integration / Delivery / Deployment"
section: 06-DevOps
tags: [devops, ci, cd, continuous-delivery, continuous-deployment, pipeline, testing, trunk-based, fresher]
related:
  - "[[06-Visible-Ops-Change-Control]]"
  - "[[10-SRE-Reliability]]"
  - "[[00-Foundations/02-Git/12-CI-CD-la-gi]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 35m
source: ["DevOps Foundations — Ernest Mueller & James Wickett (LinkedIn Learning)", "Jez Humble & Dave Farley — Continuous Delivery"]
---

# Continuous Integration / Delivery / Deployment

> [!summary] TL;DR
> Ba khái niệm thường bị lẫn — chúng là **pipeline** nối tiếp build → deploy → release. **CI (Continuous Integration):** tự động **build + unit test toàn bộ app** trên *mỗi commit* → app luôn ở trạng thái chạy được. **CD (Continuous Delivery):** thêm bước **deploy mỗi thay đổi lên môi trường giống production** + test tích hợp/acceptance tự động → app luôn **sẵn sàng release**. **Continuous Deployment:** **tự động release thẳng ra production** sau khi test pass (Amazon/Meta/Google dùng). Sáu thực hành CI: build nhanh (coffee test), commit nhỏ, không để build hỏng, **trunk-based + feature flags**, không có test "flaky", build trả về **status + log + artifact**. Năm thực hành CD: artifact **build một lần** dùng mọi môi trường, artifact **immutable**, pre-prod **giống hệt prod**, **dừng pipeline khi hỏng**, deploy **idempotent**.

---

## 1. Ba khái niệm — đừng nhầm

```mermaid
flowchart LR
    COMMIT[Commit] --> CI["CI<br/>build + unit test"]
    CI --> CD["CD<br/>deploy lên prod-like<br/>+ test tích hợp/acceptance"]
    CD --> CDEP["Continuous Deployment<br/>tự động release ra production"]
```

| Khái niệm | Làm gì | Trạng thái đảm bảo |
|---|---|---|
| **Continuous Integration** | tự động build + unit test toàn app trên **mỗi check-in** | app luôn **chạy được** |
| **Continuous Delivery** | + deploy mỗi change lên **prod-like env** + test tích hợp/acceptance tự động | app luôn **release-ready** |
| **Continuous Deployment** | + **tự động release** ra production khi test pass | thay đổi ra production **liên tục** |

> CI/CD thường viết gộp cho cặp CI + Continuous Delivery. **Continuous Deployment** là bước "pro" cuối cùng — *không bắt buộc*: có thể vẫn cần một bước duyệt tay / product manager sign-off / gom batch. Nếu CI & CD tốt thì làm Continuous Deployment an toàn (approval/test thành step trong pipeline; **feature flags** cho code đã deploy nhưng ẩn với user).

---

## 2. Sáu thực hành Continuous Integration

Trên mỗi commit, CI tự động: lấy toàn code → build → chạy mọi unit test & validation → đóng gói **artifact** + **status** + **log**. Bất kỳ bước nào fail → **build broken** → cả team bị chặn cho tới khi sửa.

| # | Thực hành | Vì sao |
|---|---|---|
| 1 | **Build chạy nhanh** | "coffee test" — dưới ~5 phút. Build chậm → người ta gom batch lớn → WIP cao |
| 2 | **Commit nhỏ** | dễ suy luận, dễ cô lập lỗi |
| 3 | **Không để build hỏng** | build hỏng = chặn giao hàng; văn hoá: dừng việc để sửa build |
| 4 | **Trunk-based development** | không nhánh sống lâu; thay đổi nhỏ vào trunk nhiều lần/ngày; **feature flags** để ẩn tính năng chưa xong (thay vì nhánh dài) |
| 5 | **Không có test "flaky"** | test lúc pass lúc fail vô cớ → mất niềm tin vào CI; phải sửa |
| 6 | **Build trả về status + log + artifact** | status (pass/fail) · log (troubleshoot/compliance) · artifact (bản cài được, tag theo build number → audit & immutable) |

> [!note] Trunk-based vs Branch-based
> **Branch-based:** mỗi dev giữ nhánh dài, làm cả tính năng lớn rồi merge → nhánh càng dài càng khó merge. **Trunk-based** (đặc trưng đội high-performer theo DORA): không nhánh sống lâu, commit nhỏ vào trunk nhiều lần/ngày, trunk luôn cập nhật. Tính năng lớn chưa muốn lộ → **feature flag** thay vì nhánh dài. → [[00-Foundations/02-Git/05-Branch-Merge-PR]].

---

## 3. Năm thực hành Continuous Delivery

Sau build là **deployment stage** — deploy artifact thành công lên env giống production nhất (staging/test/pre-prod), rồi test tự động tối đa.

| # | Thực hành | Chi tiết |
|---|---|---|
| 1 | **Build artifact một lần** | RPM/Deb/MSI/WAR/ZIP… build *một lần*, dùng cho **mọi** môi trường (test = production) |
| 2 | **Artifact immutable** | không đổi giữa các stage; chỉ CI ghi, deploy chỉ đọc → **checksum** chứng minh cùng bản → tin cậy & **auditability** (truy được source → artifact → hệ thống chạy) |
| 3 | **Pre-prod giống hệt prod** | gồm load balancer, network, security control, data khớp prod → acceptance/smoke/integration test đáng tin |
| 4 | **Dừng pipeline khi hỏng** | build hỏng → đừng deploy; deploy hỏng → đừng release. Tối ưu **flow tổng thể**, không tối ưu từng cá nhân |
| 5 | **Deploy idempotent** | chạy bao nhiêu lần cũng ra cùng hệ thống (qua container immutable hoặc CM tool) |

---

## 4. Vai trò QA & các loại test (test pyramid)

Tự động hoá test là **điều kiện sống còn** của CI/CD. Vẫn cần người giỏi QA — nhưng để **thiết kế & viết test** cùng dev, *"để máy làm việc click chuột"*. Manual test chỉ dùng cho acceptance cuối (có thể).

```
        /\        Acceptance / E2E (chậm, UI: Selenium, Cypress)
       /  \       Integration (component + dependency)
      /----\      Code hygiene (linter/formatter: ESLint)
     /______\     Unit test (nhanh nhất, dev viết, chạy trên máy dev)
```

- **TDD/BDD:** viết test/hành vi mong muốn **trước** khi viết code → tạo test suite toàn diện trong lúc phát triển.
- **Test chậm** (browser compat, compliance lớn): chạy **song song**, hoặc **scheduled** (nightly), hoặc **liên tục** trên test env — cân nhắc *cost of delay* vs *cost of bug*. → [[02-Backend/10-Testing-Pytest]].

---

## 5. Release stage & Production Release Patterns

Release = đánh dấu artifact đã "released" tại thời điểm, rồi deploy ra production + notify. Cần **an toàn, reversible, không downtime**:

| Pattern | Cách làm |
|---|---|
| **Rolling deployment** | nâng cấp **từng** server trong nhiều server giống nhau → chuyển traffic mượt |
| **Blue-Green** | dựng nguyên hệ mới (Green), **cắt** traffic từ Blue → Green (rollback = cắt ngược) |
| **Canary** | nâng cấp **một** server, chạy thử dưới tải production xem có lỗi không |
| **A/B deployment** | feature flag mở tính năng cho **một nhóm user** (canary ngắn hoặc public beta dài) |

> [!question] Phỏng vấn: "Phân biệt Continuous Integration, Delivery và Deployment."
> **CI** = tự động **build + unit test** toàn app trên *mỗi commit* → app luôn chạy được. **Continuous Delivery** = thêm **deploy lên môi trường giống production + test tích hợp/acceptance tự động** → app luôn **sẵn sàng release** (nhưng nút release cuối có thể là tay). **Continuous Deployment** = **tự động release thẳng ra production** sau khi test pass, không cần can thiệp tay. Câu chốt: *Delivery làm app "release-ready"; Deployment thực sự "release" tự động.*

> [!question] Phỏng vấn: "Blue-Green khác Canary thế nào?"
> **Blue-Green**: dựng **toàn bộ** môi trường mới (Green) song song với hiện tại (Blue), rồi **cắt toàn bộ traffic** sang Green; rollback nhanh bằng cắt ngược. **Canary**: chỉ nâng cấp **một phần nhỏ** (một vài server / % user) và quan sát dưới tải thật trước khi rollout phần còn lại. Blue-Green tối ưu *rollback tức thì & không downtime*; Canary tối ưu *giảm bán kính ảnh hưởng* khi phát hiện lỗi sớm. A/B là canary dựa trên feature flag theo nhóm user.

```
★ Insight ─────────────────────────────────────
• Artifact "build once, run everywhere" + immutable là nền của AUDITABILITY: truy
  ngược source commit → build artifact → hệ thống production. Rebuild giữa chừng
  phá vỡ chuỗi tin cậy này.
• "Dừng pipeline khi hỏng" gây khó chịu CÓ CHỦ ĐÍCH: nó ép cả team tối ưu flow
  tổng thể (Way 1) thay vì để mỗi người tự đi tiếp với rủi ro tích luỹ.
• CI/CD là hiện thân kỹ thuật của Way 2 (feedback) + Visible Ops (change nhỏ, test
  sớm): cùng một triết lý, ba lớp trừu tượng khác nhau.
─────────────────────────────────────────────────
```

---

## 6. CI toolchain (lớp hành tây — từ ngoài vào trong)

Thiết kế từ **end state** (deployment) ngược về version control:

| Lớp | Vai trò | Tool ví dụ |
|---|---|---|
| Deployment (ngoài cùng) | cách app chạy & rollout | feature flag: LaunchDarkly/Split; K8s/Serverless built-in |
| Artifact repository | lưu artifact | Artifactory, Nexus, container registry, S3 (MVP) |
| Testing | unit/lint/integration/e2e/security | go test, ESLint, Pytest, Selenium, Dependabot |
| Build system | thực thi build & kích pipeline | **Jenkins**, CircleCI, **GitHub Actions** |
| Version control (lõi) | commit & lịch sử | **Git** (GitHub/GitLab/Bitbucket) |

> Đo **cycle time** (1 thay đổi đi từ máy dev → production), ghi-graph-chia sẻ; cố làm giảm nó. → công cụ CI/CD chi tiết: [[00-Foundations/02-Git/13-GitHub-Actions]].

---

## 7. Tự kiểm tra

1. Phân biệt CI, Continuous Delivery, Continuous Deployment.
2. Kể 6 thực hành CI. "Coffee test" là gì?
3. Trunk-based khác branch-based ra sao? Feature flag giải quyết vấn đề gì?
4. Vì sao artifact phải build một lần & immutable?
5. Phân biệt Rolling / Blue-Green / Canary / A/B deployment.

---

## 8. Liên quan
- [[06-Visible-Ops-Change-Control]] — change nhỏ + test sớm (nền của CI)
- [[10-SRE-Reliability]] — deploy an toàn & MTTR
- [[00-Foundations/02-Git/12-CI-CD-la-gi]] — CI/CD cơ bản
- [[00-Foundations/02-Git/13-GitHub-Actions]] — pipeline thực hành
- [[02-Backend/10-Testing-Pytest]] — viết test (unit/integration)
- [[00-MOC-DevOps|MOC: DevOps]]
