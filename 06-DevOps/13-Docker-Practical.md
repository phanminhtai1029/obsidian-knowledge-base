---
title: "Docker thực hành: Image, Layer, Volume, Network, Compose"
section: 06-DevOps
tags: [devops, docker, container, image, alpine, dockerignore, volume, bind-mount, port-mapping, docker-compose, fresher]
related:
  - "[[12-Modern-DevOps]]"
  - "[[07-IaC-Concepts]]"
  - "[[08-IaC-Toolchain]]"
difficulty: ⭐⭐⭐
estimated_time: 40m
source: [DevOps Foundations, Docker docs, đề phỏng vấn FresherAI]
---

# Docker thực hành: Image, Layer, Volume, Network, Compose

> [!summary] TL;DR
> **Container** = cách đóng gói app cùng *toàn bộ thứ nó cần để chạy* (code, thư viện, runtime) vào một "hộp" cô lập, để **"chạy ở máy nào cũng giống nhau"** — hết cảnh "trên máy em chạy ngon mà". **Image** = bản *khuôn đúc* (read-only, bất biến) để tạo container; **Container** = một *instance đang chạy* dựng từ image (image là class, container là object). Image được xây từ nhiều **layer** (mỗi dòng lệnh trong `Dockerfile` tạo một lớp; Docker **cache** lại từng lớp → đặt thứ ít đổi lên trên để build nhanh). **Alpine** = bản image Linux *siêu nhẹ* (~5MB, dựa trên `musl libc` thay vì `glibc`) → image gọn, ít lỗ hổng bảo mật, nhưng đôi khi vênh thư viện C. **`.dockerignore`** = liệt kê file *không* gửi vào quá trình build (như `.gitignore`) → build nhanh, image sạch, tránh lọt secret. **Port mapping `-p host:container`** = nối cổng máy thật vào cổng trong container. **Volume** giữ dữ liệu *sống lâu hơn* vòng đời container: **bind mount** (gắn 1 thư mục máy thật vào container — hợp lúc dev) vs **named volume** (Docker tự quản, hợp lúc production cho DB). **docker-compose** = mô tả nhiều container trong 1 file YAML; các service gọi nhau qua **tên service** (Docker tự lo DNS nội bộ).

> [!tip] 🎯 Hiểu trong 30 giây
> Hãy hình dung **đóng thùng container chở hàng**: bất kể bên trong là tủ lạnh hay chuối, cái thùng có kích thước chuẩn nên *cẩu nào, tàu nào, cảng nào* cũng bốc dỡ được như nhau. Docker làm đúng vậy với phần mềm: nhét app + mọi thư viện vào một "thùng" chuẩn, rồi thùng đó chạy y hệt trên laptop của bạn, trên server, hay trên cloud. **Vấn đề nó giải quyết:** dẹp bỏ câu kinh điển *"trên máy em chạy được mà!"* — vì môi trường giờ đi kèm trong thùng, không phụ thuộc máy.
>
> - **Image** = công thức + nguyên liệu đóng sẵn (bất biến). **Container** = món ăn đang nấu từ công thức đó.
> - **Layer** = các lớp xếp chồng trong image; sửa lớp dưới thì mọi lớp trên phải làm lại (nên build chậm) → xếp thứ tự khéo để tận dụng **cache**.
> - **Volume** = ổ cứng ngoài cắm vào thùng, để khi vứt thùng đi dữ liệu vẫn còn.

---

## 1. Image vs Container — phân biệt gốc rễ

| | **Image** | **Container** |
|---|---|---|
| Bản chất | *Khuôn đúc* read-only (template bất biến) | *Instance đang chạy* dựng từ image |
| Ví von OOP | **Class** | **Object** (`new` từ class) |
| Số lượng | 1 image → tạo được **nhiều** container | Mỗi container là một tiến trình riêng |
| Lưu ở đâu | Registry (Docker Hub, GHCR...) + cache máy | Chạy trên Docker Engine |
| Thay đổi | Không đổi (muốn đổi → build image mới) | Có lớp ghi tạm; xoá container là mất |

```sh
docker build -t myapp:1.0 .     # tạo IMAGE từ Dockerfile ở thư mục hiện tại
docker run myapp:1.0            # tạo & chạy 1 CONTAINER từ image đó
docker run myapp:1.0            # chạy thêm container thứ 2 từ cùng image
```

> [!note] "Container nhẹ hơn VM" nghĩa là gì?
> **Máy ảo (VM)** ảo hoá *cả phần cứng* và chạy nguyên một hệ điều hành khách (kernel riêng) → nặng GB, khởi động hàng chục giây. **Container** *dùng chung kernel* của máy host, chỉ cô lập ở mức tiến trình (namespace + cgroup) → nhẹ MB, khởi động trong mili-giây. Đổi lại: container kém cô lập hơn VM về bảo mật.

---

## 2. Layer & Build Cache — vì sao thứ tự `Dockerfile` quan trọng

Mỗi **chỉ thị** trong `Dockerfile` (`FROM`, `COPY`, `RUN`...) tạo ra **một layer** (lớp) chỉ-đọc xếp chồng lên nhau. Docker **cache** từng layer; khi build lại, nếu một layer *và mọi layer phía trên nó* không đổi, Docker dùng lại cache → build cực nhanh. Nhưng **chỉ cần một layer đổi, mọi layer bên dưới (sau nó) phải build lại**.

```dockerfile
FROM python:3.12-slim            # layer nền

WORKDIR /app

COPY requirements.txt .          # ① copy RIÊNG file deps trước
RUN pip install -r requirements.txt   # ② cài deps — layer NẶNG, ít khi đổi

COPY . .                         # ③ copy code (đổi liên tục) — để CUỐI

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

> [!warning] Bẫy kinh điển: `COPY . .` trước khi cài deps
> Nếu bạn `COPY . .` rồi mới `pip install`, thì **mỗi lần sửa một dòng code**, layer copy đổi → layer `pip install` (nằm sau) **mất cache → cài lại toàn bộ thư viện** mỗi lần build (chậm khủng khiếp). Mẹo: copy `requirements.txt` và cài deps *trước*, copy code *sau cùng* — vì code đổi thường xuyên còn deps thì hiếm.

```
★ Insight ─────────────────────────────────────
• Quy tắc vàng xếp Dockerfile: "thứ ít thay đổi để LÊN TRÊN, thứ hay thay đổi
  để XUỐNG DƯỚI". Vì cache bị phá từ layer đổi trở xuống → giữ phần đắt đỏ
  (cài thư viện) ở phía trên, an toàn trong cache.
• Image to hay nhỏ phụ thuộc TỔNG các layer. Một `RUN apt-get install ...` rồi
  một `RUN rm ...` ở layer sau KHÔNG làm image nhỏ lại — file đã nằm trong layer
  trước rồi (layer chỉ chồng thêm, không xoá ngược). Muốn nhỏ: gộp vào 1 RUN,
  hoặc dùng multi-stage build.
─────────────────────────────────────────────────
```

> [!tip] Multi-stage build — tách "máy xây" khỏi "máy chạy"
> Dùng nhiều `FROM`: stage đầu cài đủ công cụ để *build* (compiler, dev deps), stage cuối chỉ `COPY` sản phẩm đã build sang một image nền gọn → image cuối **không mang theo** rác build. Đây là cách phổ biến để image production nhỏ và an toàn.

---

## 3. Alpine image — siêu nhẹ, đánh đổi gì?

`alpine` là một bản phân phối Linux tối giản (~5MB). Các image kiểu `python:3.12-alpine`, `node:20-alpine` dựa trên nó.

| Tiêu chí | `python:3.12` (đầy đủ) | `python:3.12-slim` | `python:3.12-alpine` |
|---|---|---|---|
| Kích thước | ~1GB | ~150MB | ~50MB |
| Thư viện C | `glibc` (chuẩn) | `glibc` | **`musl libc`** (khác) |
| Build wheel C-extension | Dễ (có sẵn) | Cần thêm | **Hay phải compile lại** (chậm/lỗi) |
| Bề mặt tấn công | Lớn | Vừa | **Nhỏ nhất** |

> [!warning] Mặt trái của Alpine (recruiter hay hỏi)
> Alpine dùng **`musl libc`** thay cho **`glibc`** (thư viện C chuẩn mà hầu hết Linux dùng). Nhiều package Python có sẵn bản biên dịch (`wheel`) cho glibc nhưng **không có** cho musl → pip phải *biên dịch lại từ mã nguồn* (cần thêm `gcc`, build lâu, đôi khi lỗi). Vì vậy nhiều team Python lại chọn `-slim` (glibc, vẫn nhỏ) thay vì `-alpine`. **Câu chốt:** Alpine nhỏ & ít lỗ hổng, nhưng cân nhắc rủi ro tương thích `musl` cho hệ Python/ML.

---

## 4. `.dockerignore` — đừng gửi cả kho rác vào build

Khi `docker build`, Docker đóng gói **toàn bộ thư mục hiện tại** (build context) gửi cho Docker Engine. `.dockerignore` (cú pháp giống `.gitignore`) loại các file **không cần**:

```dockerignore
.git
node_modules
__pycache__/
*.pyc
.env                  # ⚠️ tránh nướng SECRET vào image
.venv
Dockerfile
docker-compose.yml
*.md
```

> [!note] Ba lý do bắt buộc có `.dockerignore`
> 1. **Nhanh hơn:** không gửi `node_modules`/`.git` nặng hàng trăm MB vào build context.
> 2. **Image sạch & nhỏ:** `COPY . .` sẽ không lôi rác vào image.
> 3. **Bảo mật:** chặn `.env`, key, credential **lọt vào image** — image hay bị push lên registry, lộ là toang. Đây là điểm an toàn quan trọng nhất.

---

## 5. Port mapping — `-p host:container`

Container có mạng *riêng*; cổng bên trong **không tự** lộ ra máy thật. Phải **ánh xạ cổng**:

```sh
docker run -p 8080:8000 myapp
#            │    └── cổng BÊN TRONG container (app uvicorn lắng nghe :8000)
#            └── cổng trên MÁY THẬT (host) → mở localhost:8080
```

Truy cập `http://localhost:8080` ở máy thật → Docker chuyển vào `:8000` trong container.

> [!warning] Hai bẫy port hay gặp
> - **Bind `127.0.0.1` bên trong container:** app phải lắng nghe `0.0.0.0` (mọi interface), **không** phải `127.0.0.1` — nếu chỉ bind localhost *trong* container thì host map vào cũng không thấy. (Vì vậy lệnh chạy uvicorn trong Docker luôn `--host 0.0.0.0`.)
> - **Nhầm thứ tự:** `-p` luôn là **`host:container`**. Viết ngược là mở nhầm cổng.

---

## 6. Volume — giữ dữ liệu sống lâu hơn container

Container **vô thường (ephemeral)**: xoá container là **mất sạch** dữ liệu ghi bên trong (lớp ghi tạm biến mất). Muốn giữ DB, file upload... phải dùng **volume** (lưu dữ liệu *ngoài* lớp container).

| | **Bind mount** | **Named volume** |
|---|---|---|
| Là gì | Gắn **một thư mục cụ thể trên máy host** vào container | Vùng lưu **do Docker quản lý** (trong `/var/lib/docker/volumes`) |
| Cú pháp | `-v /home/me/code:/app` (đường dẫn host) | `-v mydata:/var/lib/postgresql/data` (tên) |
| Ai kiểm soát path | **Bạn** (đường dẫn rõ trên host) | **Docker** (bạn không cần biết path thật) |
| Hợp cảnh nào | **Dev**: sửa code máy thật → container thấy ngay (hot-reload) | **Production**: dữ liệu DB, bền vững, dễ backup/di chuyển |
| Nhược | Lệ thuộc cấu trúc thư mục host, lệch quyền (permission) giữa OS | Khó "ngó" file trực tiếp hơn |

```sh
# Bind mount — dev: code trên máy đổi thì trong container đổi theo
docker run -v $(pwd):/app -p 8000:8000 myapp

# Named volume — production: dữ liệu Postgres sống qua mọi lần tạo lại container
docker run -v pgdata:/var/lib/postgresql/data postgres:16
```

```
★ Insight ─────────────────────────────────────
• Nhớ nhanh: BIND mount = "tao chỉ đích danh thư mục máy tao" (dev, trực quan).
  NAMED volume = "Docker, mày lo chỗ cất, miễn dữ liệu bền" (prod, gọn gàng).
• Có loại thứ 3 là tmpfs (chỉ nằm RAM, mất khi tắt) — dùng cho dữ liệu nhạy cảm
  tạm thời. Ít gặp ở fresher nhưng biết để không bị bất ngờ.
─────────────────────────────────────────────────
```

---

## 7. docker-compose & giao tiếp qua **tên service**

Một app thật thường nhiều container (web + db + redis). **`docker-compose.yml`** mô tả tất cả trong 1 file, bật bằng `docker compose up`:

```yaml
services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: "postgresql://user:pass@db:5432/app"   # ← "db" là TÊN SERVICE
    depends_on:
      - db
  db:
    image: postgres:16
    volumes:
      - pgdata:/var/lib/postgresql/data     # named volume cho DB
    environment:
      POSTGRES_PASSWORD: pass

volumes:
  pgdata:
```

> [!note] Vì sao connect tới `db` mà không phải `localhost`?
> Compose tạo một **mạng ảo riêng** cho các service và chạy một **DNS nội bộ**: trong mạng đó, mỗi service được phân giải bằng **tên service**. Nên container `web` gọi DB qua host **`db`** (không phải `localhost`). **Bẫy fresher:** dùng `localhost:5432` trong container `web` → `localhost` ở đây trỏ về *chính container web*, không phải container db → "connection refused". Phải dùng **tên service** làm hostname.

> [!tip] `depends_on` ≠ "đợi DB sẵn sàng"
> `depends_on` chỉ đảm bảo **thứ tự khởi động** (db start trước web), **không** đợi DB *thật sự nhận kết nối*. App vẫn nên có **retry** khi connect DB lúc khởi động, hoặc dùng healthcheck.

---

## 8. Q&A phỏng vấn

> [!example] 🗣️ Trả lời mẫu (nói thành lời) — "Image và container khác gì nhau?"
> *"Image là một bản đóng gói chỉ-đọc, bất biến, chứa app cùng mọi thứ nó cần để chạy — giống như một cái khuôn hay một class. Container là một instance đang chạy được dựng lên từ image đó, giống như object new ra từ class. Từ một image em chạy được nhiều container độc lập. Image được xây từ nhiều layer xếp chồng, mỗi chỉ thị trong Dockerfile là một layer và Docker cache lại để build sau nhanh hơn. Điểm hay bị hỏi tiếp là container thì vô thường: xoá đi là mất dữ liệu bên trong, nên nếu cần giữ dữ liệu như database thì em phải gắn volume."*

> [!example] 🗣️ Trả lời mẫu — "Bind mount khác named volume thế nào?"
> *"Cả hai đều để giữ dữ liệu sống lâu hơn container. Bind mount là em gắn thẳng một thư mục cụ thể trên máy host vào trong container, nên em thấy và sửa file trực tiếp — rất hợp lúc dev vì sửa code ngoài máy thì trong container nhận ngay. Named volume thì để Docker tự quản lý chỗ lưu, em chỉ đặt tên; nó hợp với production, ví dụ dữ liệu Postgres, vì bền vững, dễ backup và không lệ thuộc đường dẫn máy host. Quy tắc của em: dev thì bind mount cho tiện, production và database thì named volume."*

> [!note] 🧠 Mẹo nhớ
> - **Image = class (khuôn), Container = object (đang chạy).** Container vô thường → cần **volume** mới giữ được dữ liệu.
> - **Layer:** ít đổi để TRÊN, hay đổi (code) để DƯỚI → tận dụng cache. `COPY requirements` trước `pip install`.
> - **Alpine** nhỏ nhưng `musl` ≠ `glibc` → Python hay phải compile lại; thường chọn `-slim`.
> - **`-p host:container`** (đừng viết ngược) + app bind `0.0.0.0`.
> - **bind mount = dev** (chỉ đích danh thư mục), **named volume = prod** (Docker tự quản).
> - Trong compose, gọi nhau bằng **tên service**, KHÔNG phải `localhost`.

> [!question] 1. Tại sao nên dùng image Alpine? Có rủi ro gì?
> Alpine ~5MB → image **nhỏ, ít lỗ hổng, kéo/đẩy nhanh**. Rủi ro: dùng **`musl libc`** thay `glibc` → nhiều package Python không có wheel sẵn, phải **compile lại** (chậm, dễ lỗi). Với hệ Python/ML nhiều team chọn `-slim` (vẫn nhỏ mà vẫn `glibc`).

> [!question] 2. `.dockerignore` để làm gì?
> Loại file **không cần** khỏi build context (giống `.gitignore`): build **nhanh hơn** (bỏ `node_modules`, `.git`), image **sạch/nhỏ**, và **chặn secret** (`.env`, key) lọt vào image — quan trọng vì image hay được push lên registry.

> [!question] 3. `docker run -p 8080:8000` nghĩa là gì?
> Ánh xạ cổng **8080 trên máy host** → **8000 trong container** (`host:container`). Truy cập `localhost:8080` sẽ vào app lắng nghe `:8000` bên trong. Lưu ý app trong container phải bind `0.0.0.0` chứ không phải `127.0.0.1`.

> [!question] 4. Bind mount khác named volume ở điểm nào? Khi nào dùng cái nào?
> **Bind mount** gắn thư mục cụ thể trên host → trực quan, hợp **dev** (sửa code thấy ngay). **Named volume** do Docker quản lý → bền vững, dễ backup, hợp **production/DB**. Cả hai đều giúp dữ liệu **sống lâu hơn** vòng đời container.

> [!question] 5. Trong docker-compose, service `web` connect tới database bằng host gì?
> Bằng **tên service** (vd `db`), không phải `localhost`. Compose tạo mạng ảo + DNS nội bộ phân giải theo tên service. Dùng `localhost` trong container `web` sẽ trỏ về chính nó → lỗi kết nối.

> [!question] 6. Vì sao thứ tự lệnh trong Dockerfile ảnh hưởng tốc độ build?
> Mỗi lệnh là một **layer** được **cache**; một layer đổi thì mọi layer **sau** nó mất cache. Đặt phần ít đổi (cài deps) **lên trên**, phần hay đổi (code) **xuống dưới** → tránh cài lại thư viện mỗi lần sửa code. Vì vậy `COPY requirements.txt` + `pip install` **trước** `COPY . .`.

---

## 9. Bài tập tự luyện

1. **Đo cache:** viết `Dockerfile` cho app FastAPI theo 2 cách (copy code trước vs copy `requirements.txt` trước), sửa 1 dòng code rồi build lại, so sánh thời gian.
2. **So sánh image:** build cùng app với `python:3.12`, `:3.12-slim`, `:3.12-alpine`; chạy `docker images` so kích thước và ghi nhận có package nào phải compile khi dùng alpine.
3. **Volume thực chiến:** chạy `postgres` với named volume, tạo bảng + chèn dữ liệu, `docker rm` container rồi tạo lại từ cùng volume, kiểm tra dữ liệu còn không.
4. **Compose:** viết `docker-compose.yml` gồm `web` (FastAPI) + `db` (Postgres), cho `web` connect qua tên service `db`, chạy `docker compose up` và gọi API.
5. **Bảo mật:** cố tình bỏ `.dockerignore`, build rồi `docker history`/giải nén image xem `.env` có lọt vào không; thêm `.dockerignore` và kiểm tra lại.

---

## 10. Liên quan
- [[12-Modern-DevOps]] — Cloud native, Kubernetes (orchestration cho container ở quy mô lớn)
- [[07-IaC-Concepts]] — immutable infrastructure, cattle-not-pets (triết lý container)
- [[08-IaC-Toolchain]] — Docker "baking" image, Packer
- [[02-Backend/12-Deployment-Uvicorn|Backend: Deployment Uvicorn]] — đóng gói FastAPI để chạy trong container
- [[00-MOC-DevOps|MOC: DevOps]]
