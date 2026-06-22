---
title: "DevOps hiện đại — Platform Engineering, DevSecOps, K8s, Chaos, MLOps, AIOps"
section: 06-DevOps
tags: [devops, platform-engineering, devsecops, kubernetes, cloud-native, chaos-engineering, mlops, aiops, career, fresher]
related:
  - "[[08-IaC-Toolchain]]"
  - "[[10-SRE-Reliability]]"
  - "[[04-AI/03-LLMOps-Evaluation/00-MOC-LLMOps-Evaluation]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 40m
source: ["DevOps Foundations — Ernest Mueller & James Wickett (LinkedIn Learning)"]
---

# DevOps hiện đại

> [!summary] TL;DR
> Các hướng nâng cao của DevOps. **Platform engineering / paved road**: khi scale ra nhiều value stream, tạo **self-service automation** chuẩn (golden path) để đội tự lấy thứ cần — *quản platform như sản phẩm, theo Lean*, đừng biến nó thành "IT tập trung tồi" thời xưa. **DevSecOps**: mang CAMS vào bảo mật; **shift left** (đưa security sớm vào IDE/CI), security champions, không "blocking". **Cloud native & Kubernetes**: K8s là container orchestration gói cả provision/deploy/orchestrate; "cloud native" = "add-on cho K8s" (CNCF); cẩn thận **phức tạp & chi phí** — cân nhắc serverless cho 80% lợi ích với 20% công sức. **Chaos engineering**: cố tình **gây lỗi trong production** (Netflix Chaos Monkey, Game Days) để xây niềm tin vào resilience. **MLOps**: DevOps cho ML (quản data + model + code, HPC/GPU, drift). **AIOps**: dùng AI hỗ trợ công việc DevOps (sinh code, giải thích hệ thống) — *AI không thay kỹ sư, vẫn phải test*.

---

## 1. Platform Engineering — the Paved Road

Khi align theo nhiều value stream, mỗi đội tự dựng giải pháp → đa dạng quá → **hỗn loạn**. Giải pháp (chữ **A** Automation): tạo **golden path / paved road** — self-service framework "có ý kiến" (opinionated) làm sẵn những việc lặp (CI/CD pipeline, observability, security guidance) → đội khác chỉ việc dùng.

> [!warning] Paved road có thể thành "IT tập trung tồi" trá hình
> Ranh giới giữa *platform hiện đại tốt* và *hệ tập trung quan liêu cũ* rất mong manh. Khác biệt: platform tốt **tối ưu hệ thống tổng thể & value stream**, không tối ưu cho team trung tâm. Hai chìa khoá:
> 1. **Product management** — platform phục vụ *người dùng* (dev/ops), có product manager, **không bắt buộc dùng** (Netflix: khuyến khích chứ không ép — bị ép thì sản phẩm tệ dần).
> 2. **Lean** — *có chuyện "quá nhiều platform"*; đừng build hết trước khi biết nhu cầu. **Blaze a trail → pave the road → build the train.**

> **Platform engineering** = *kỷ luật thiết kế & xây toolchain + workflow cho phép self-service* trong tổ chức kỹ thuật.

---

## 2. DevSecOps — bảo mật theo kiểu DevOps

Bảo mật bị silo hoá → cùng vấn đề "wall of confusion": tỷ lệ điển hình **100 dev : 10 ops : 1 security** (bài toán order-of-magnitude), security nằm cuối value stream, bị "ném app qua tường" rồi ném lại "danh sách yêu cầu". Dev *có* quan tâm bảo mật nhưng thiếu thời gian & không biết security muốn gì.

Áp **CAMS** với lăng kính bảo mật:

| CAMS | DevSecOps |
|---|---|
| **Culture** | *"Nếu security gây blocking, nó sẽ bị bỏ qua, không được đón nhận"* (Etsy) — security đi *cùng* dev, không thêm cổng chặn |
| **Automation** | **Shift left** — đưa security tool **sớm** vào IDE & CI (bắt sớm = sửa rẻ). *Nhưng* shift-left sai cách = đổ việc lên dev → phải align với value stream, tối thiểu ảnh hưởng cycle time |
| **Measurement** | observability & metric bảo mật → mục tiêu chung; tránh dựa **FUD** (fear, uncertainty, doubt) |
| **Sharing** | **security champions** trong mỗi nhóm (tìm qua Capture the Flag, người từng fix bug bảo mật, volunteer) |

→ liên hệ OWASP & tư duy bảo mật: [[02-Backend/09-Authorization-RBAC]].

---

## 3. Cloud Native & Kubernetes

> [!note] Định nghĩa
> **Kubernetes (K8s)** = hệ **container orchestration** mã nguồn mở: tự động deploy, scale, quản lý app container. Nó gói cả **provisioning + deployment + orchestration** + observability/service discovery/health check/networking theo cách chuẩn hoá, **trừu tượng hoá hạ tầng** (compute/network/storage) → gần như đạt được "multi-cloud". **Cloud native** = (theo nghĩa marketing) "thứ thiết kế để chạy với K8s"; hệ sinh thái add-on nằm trong **CNCF** (Cloud Native Computing Foundation).

> [!warning] Ba thách thức của Kubernetes
> 1. **Quá nhiều lựa chọn** — >20 lựa chọn chỉ riêng network backplane; một platform K8s thường là *cả tá tool ghép lại* → nâng cấp & tương thích cực nhọc; admin thường *không nắm rõ* hệ hành xử thế nào khi deploy/scale.
> 2. **Nặng & tốn** — base cluster có thể tốn hàng trăm USD/tháng dù chưa phục vụ user nào; cần *đội chuyên trách*.
> 3. **Có thể phản tác dụng DevOps** — nếu không dùng systems thinking & CAMS, K8s dễ tạo silo & waste.
> ⇒ Cân nhắc **serverless / orchestration nhẹ**: ~80% lợi ích với ~20% công sức. Lean: bắt đầu đơn giản, thêm phức tạp khi *thật sự* cần.

---

## 4. Chaos Engineering

> [!note] Định nghĩa
> **Chaos engineering** = *kỷ luật thực nghiệm trên hệ thống để xây niềm tin vào khả năng chịu đựng điều kiện production thật* — tức **cố tình tạo nghịch cảnh**. Netflix khởi xướng với **Chaos Monkey**: một "con khỉ giận dữ" vào **production** đập ngẫu nhiên một server → buộc mọi team đảm bảo hệ không degraded khi mất component.

- Là **controlled chaos**: thiết kế experiment để validate qua **fault injection** (không phải cái cớ để ẩu — vẫn test ở dev).
- **Game Days**: tạo/giả lập sự cố lớn để test cả phần **con người** của incident response (hệ là sociotechnical).
- Giá trị: con người thường **đoán sai** cách hệ thống hành xử khi hỏng → break network/DNS/container repo để *biết thật* thay vì đoán. Như crash test xe hơi. Trực tiếp hiện thực **Way 3** (học qua feedback) → [[03-The-Three-Ways]].

---

## 5. MLOps — DevOps cho hệ ML

DevOps áp cho machine learning, với điểm khác biệt:

| Khía cạnh | MLOps khác DevOps thường |
|---|---|
| Người tạo workload chính | **data scientist** (không phải dev) — gắn chặt với phần cứng, cần cộng tác & hỗ trợ nhiều |
| Thứ phải quản | không chỉ code — còn **data set khổng lồ + model** (versioning) |
| Hạ tầng training | **HPC cluster / GPU** — intensive batch job, đắt |
| Kết quả | không phải "một đáp án đúng/test pass" — output đa dạng, **feedback loop phong phú hơn** |
| Drift | phát hiện **drift trong dự đoán AI** phức tạp hơn drift hạ tầng; **vector DB** lớn dần & tốn |

→ Ba CI/CD song song: **code · infrastructure · data/model**; ba chuyên môn: dev · ops · data scientist. Liên hệ chặt với phần AI của vault: [[04-AI/03-LLMOps-Evaluation/00-MOC-LLMOps-Evaluation]].

---

## 6. AIOps — dùng AI trong công việc DevOps

> [!warning] Nguyên tắc số 1
> AI **không thay** kỹ sư; **không** được lấy output AI commit thẳng vào repo & chạy mà không **đánh giá & test**. *"Không thể đổ lỗi cho thuật toán."*

Cách dùng & ba làn sóng:

| Ứng dụng | Ví dụ |
|---|---|
| Viết code / làm việc với API | hỏi LLM sinh script, nhớ giúp flag của `curl`… (**prompt engineering**: cho ngữ cảnh, "trả lời như DevOps engineer Linux") |
| Refactor & docs | tách script thành module, refactor Terraform, viết tài liệu pipeline |
| Vận hành | detection/alerting tốt hơn từ dữ liệu monitoring; **k8sgpt** giải thích trạng thái K8s; chuyển đổi ngôn ngữ/tool (Terraform → AWS CLI) |
| Code review & security | tìm thay đổi rủi ro trước khi vào pipeline |

**Ba wave:** (1) *code* — AI hỗ trợ code/review/test/docs trong IDE (Copilot — đã thật); (2) *systems* — AI lo alerting/monitoring/runbook (còn sơ khai); (3) *people* — AI làm platform self-service dễ dùng & tăng cộng tác.

> [!question] Phỏng vấn: "Platform engineering / paved road là gì? Rủi ro lớn nhất?"
> **Paved road / golden path** = tạo **self-service automation chuẩn hoá** (CI/CD, observability, security làm sẵn) để các đội tự lấy thứ cần thay vì chờ team khác — mở rộng (scale) văn hoá DevOps. **Platform engineering** là phiên bản toàn diện của nó. Rủi ro lớn nhất: biến thành **"IT tập trung quan liêu" trá hình** — tối ưu cho team platform thay vì cho value stream. Phòng tránh: quản platform **như sản phẩm** (có PM, không ép dùng) và theo **Lean** (đừng build quá nhiều trước khi biết nhu cầu).

> [!question] Phỏng vấn: "Khi nào KHÔNG nên dùng Kubernetes?"
> Khi độ phức tạp & chi phí của nó vượt nhu cầu thật. K8s mạnh nhưng: hàng tá tool ghép lại (khó vận hành/nâng cấp), tốn kém (base cluster hàng trăm USD/tháng dù chưa có user), cần đội chuyên trách, và dễ tạo silo nếu thiếu systems thinking. Với nhiều workload, **serverless hoặc orchestration nhẹ** cho ~80% lợi ích với ~20% công sức. Theo Lean: bắt đầu đơn giản, chỉ thêm K8s khi thật sự cần khả năng của nó.

```
★ Insight ─────────────────────────────────────
• Platform engineering, DevSecOps, K8s đều có CÙNG cái bẫy: tối ưu cho team trung
  tâm thay vì value stream → biến thành silo mới. Mọi hướng "modern" vẫn phải soi
  lại bằng CAMS + Three Ways.
• Chaos engineering đảo ngược trực giác: "test in production" nghe liều, nhưng là
  cách DUY NHẤT validate resilience ở quy mô & độ phức tạp thật — và nó test cả
  con người (Game Days).
• MLOps = DevOps + "data & model là công dân hạng nhất". Hiểu DevOps trước rồi thêm
  ba CI/CD song song (code/infra/data) là cách tư duy đúng — nối thẳng sang phần AI
  của vault.
─────────────────────────────────────────────────
```

---

## 7. DevOps & nghề nghiệp

DevOps là **mindset + thực hành**, không phải chức danh — ai cũng hưởng lợi:

| Vai trò | DevOps thêm gì |
|---|---|
| Developer | build app reliable, instrument observability, align theo value stream, on-call |
| Sysadmin/IT | (thường bị đổi tên thành "DevOps Engineer") + IaC, design-for-reliability, runbook automation |
| Chuyên biệt | **Platform engineer** (automation), **Release manager** (build/release), **SRE** (runtime support), Incident manager, APM team |
| Security | hiểu & tham gia DevOps → **DevSecOps engineer** |

---

## 8. Tự kiểm tra

1. Paved road / platform engineering là gì? Hai chìa khoá để không thành "IT tập trung tồi"?
2. DevSecOps áp CAMS thế nào? "Shift left" là gì và rủi ro khi làm sai?
3. Kubernetes gói những bài toán nào? Ba thách thức khi dùng K8s?
4. Chaos engineering là gì? Chaos Monkey & Game Days dùng để làm gì?
5. MLOps khác DevOps thường ở những điểm nào? Nguyên tắc số 1 của AIOps?

---

## 9. Liên quan
- [[08-IaC-Toolchain]] — IaC nền cho K8s/container
- [[10-SRE-Reliability]] — resilience (nền của chaos engineering)
- [[02-Backend/09-Authorization-RBAC]] — OWASP & security mindset (DevSecOps)
- [[04-AI/03-LLMOps-Evaluation/00-MOC-LLMOps-Evaluation]] — LLMOps (họ hàng MLOps)
- [[05-Cloud/00-MOC-Cloud]] — Cloud (nền cloud native)
- [[00-MOC-DevOps|MOC: DevOps]]
