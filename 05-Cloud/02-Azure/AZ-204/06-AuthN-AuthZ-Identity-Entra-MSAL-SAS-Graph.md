---
title: "AuthN/AuthZ: Identity platform, Entra ID, MSAL, SAS, Graph"
section: 05-Cloud/02-Azure/AZ-204
tags: [azure, az-204, identity, entra-id, msal, sas, microsoft-graph, oauth2, fresher]
related:
  - "[[00-MOC-AZ-204]]"
  - "[[07-Key-Vault-App-Configuration-Managed-Identity]]"
difficulty: ⭐⭐⭐⭐
estimated_time: 28m
source: ["_source/Microsoft/Az-204.docx (Lesson 6, Module 3)", "Microsoft Learn"]
status: ✅
---

# AuthN/AuthZ: Identity platform, Entra ID, MSAL, SAS, Graph

> [!summary] TL;DR
> **AuthN** (authentication — *bạn là ai*) khác **AuthZ** (authorization — *bạn được làm gì*). **Microsoft Identity platform** là nhà cung cấp danh tính dựa trên **OAuth 2.0 / OpenID Connect (OIDC)**; muốn app dùng nó phải **app registration** (đăng ký app → có **client ID**, **tenant ID**, **redirect URI**, và **secret/cert**). App xin quyền qua **scopes/permissions** kiểu **delegated** (thay mặt user) hoặc **application** (tự app, không có user). **MSAL** (Microsoft Authentication Library) là thư viện lo việc **lấy & cache token**; các luồng chính: **Authorization Code** (app có user đăng nhập), **Client Credentials** (app-to-app, không user), **On-Behalf-Of** (API gọi API hộ user). Token gồm **access token** (gọi API), **ID token** (thông tin user), **refresh token** (lấy access token mới). **Entra ID** (tên cũ Azure AD) chứa tenant/user/group/service principal. Với Storage, **SAS** cấp quyền **giới hạn & tạm thời** an toàn hơn account key. **Microsoft Graph** là API thống nhất truy cập dữ liệu Microsoft 365.

---

## 1. Microsoft Identity platform & app registration

- **Microsoft Identity platform** = identity provider chuẩn **OAuth 2.0 / OIDC** cho phép app đăng nhập user và gọi API được bảo vệ.
- **App registration** (đăng ký ứng dụng) tạo một danh tính cho app trong Entra ID:

| Thành phần | Ý nghĩa |
|---|---|
| **Application (client) ID** | Định danh duy nhất của app |
| **Directory (tenant) ID** | Định danh tổ chức (tenant) chứa app |
| **Redirect URI** | Nơi Entra trả mã/token về sau đăng nhập |
| **Client secret / certificate** | "Mật khẩu" để app tự xác thực (nên ưu tiên **cert** hoặc **Managed Identity**) |
| **API permissions / scopes** | Quyền app xin để gọi API (vd `User.Read`) |

- **Delegated permission** = app hành động **thay mặt user đã đăng nhập** (quyền hiệu lực = giao của quyền app ∩ quyền user). **Application permission** = app **tự hành động, không có user** (daemon/background) — cần admin consent.

---

## 2. AuthN vs AuthZ (OAuth2 / OIDC)

| | **Authentication (AuthN)** | **Authorization (AuthZ)** |
|---|---|---|
| Câu hỏi | *Bạn là ai?* | *Bạn được làm gì?* |
| Chuẩn | **OpenID Connect** (trên OAuth2) → **ID token** | **OAuth 2.0** → **access token** + scopes/roles |
| Ví dụ | Đăng nhập bằng tài khoản tổ chức | Token cho phép gọi `GET /me/mail` |

- **Luồng OAuth2 cơ bản:** app → chuyển user tới Entra đăng nhập → Entra trả **authorization code** → app đổi code lấy **token** → dùng access token gọi API.

---

## 3. MSAL & các token flow

**MSAL** lo: chuyển hướng đăng nhập, đổi code lấy token, **cache** token, **tự refresh** khi hết hạn.

| Flow | Dùng khi | Có user? |
|---|---|---|
| **Authorization Code** | Web/SPA/mobile có user đăng nhập | ✅ |
| **Client Credentials** | App-to-app / daemon / job nền | ❌ |
| **On-Behalf-Of (OBO)** | API A nhận token user rồi gọi API B **hộ user** | ✅ (gián tiếp) |
| **Device Code** | Thiết bị không có trình duyệt (CLI, IoT) | ✅ |

**3 loại token:**
- **Access token** — "vé" (JWT) để **gọi API** được bảo vệ; có hạn ngắn (~1h), chứa scopes/roles.
- **ID token** — chứng minh **danh tính user** (OIDC), thông tin profile.
- **Refresh token** — đổi lấy access token mới khi hết hạn, **không phải đăng nhập lại**.

```python
import msal
app = msal.ConfidentialClientApplication(
    client_id, authority=f"https://login.microsoftonline.com/{tenant_id}",
    client_credential=client_secret)            # hoặc cert / Managed Identity
result = app.acquire_token_for_client(          # Client Credentials flow (app-to-app)
    scopes=["https://graph.microsoft.com/.default"])
token = result["access_token"]
```

---

## 4. Shared Access Signature (SAS) cho Storage

- **SAS** = chuỗi token cấp quyền **giới hạn & có thời hạn** tới tài nguyên Storage, **không lộ account key**.

| Loại SAS | Ký bằng | Đặc điểm |
|---|---|---|
| **User delegation SAS** | **Entra ID** (Managed Identity) | **An toàn nhất** — không dùng account key; thu hồi được |
| **Service SAS** | Account key | Cấp quyền tới 1 dịch vụ (vd Blob) |
| **Account SAS** | Account key | Cấp quyền rộng nhiều dịch vụ |

- SAS giới hạn được: **quyền** (read/write/list/delete), **thời hạn** (start/expiry), **IP**, **protocol (HTTPS)**.
- **Vì sao an toàn hơn account key:** account key = "chìa khóa vạn năng" toàn quyền vĩnh viễn — lộ là mất tất. SAS = chìa hẹp, có hạn; **user delegation SAS** còn không đụng tới key và thu hồi được.

---

## 5. Microsoft Graph

- **Microsoft Graph** = **một API thống nhất** (`https://graph.microsoft.com`) truy cập dữ liệu Microsoft 365: user, group, mail, calendar, files (OneDrive/SharePoint), Teams…
- Gọi qua **REST** hoặc **Graph SDK**, kèm **access token** (xin đúng scope, vd `Mail.Read`).

```python
import httpx
r = httpx.get("https://graph.microsoft.com/v1.0/me",
              headers={"Authorization": f"Bearer {token}"})
```

> [!question] Phỏng vấn: "Phân biệt AuthN và AuthZ. Token nào dùng cho cái nào?"
> **AuthN** trả lời *bạn là ai* — qua **OIDC**, cho ra **ID token**. **AuthZ** trả lời *bạn được làm gì* — qua **OAuth2**, cho ra **access token** mang scopes/roles để gọi API. Một bên định danh, một bên trao quyền.

> [!question] Phỏng vấn: "Service A (API) cần gọi service B hộ người dùng đang đăng nhập — dùng flow nào?"
> **On-Behalf-Of (OBO)**: A nhận access token của user, đổi lấy token mới để gọi B **thay mặt user** (giữ nguyên danh tính & quyền của user), thay vì Client Credentials (mất ngữ cảnh user).

> [!question] Phỏng vấn: "Cho đối tác tải file từ Blob trong 1 giờ, chỉ quyền đọc, không lộ key?"
> Cấp **SAS** quyền read, expiry 1 giờ, ràng IP/HTTPS; tốt nhất là **user delegation SAS** (ký bằng Entra/Managed Identity, không dùng account key, thu hồi được).

---

```
★ Insight ─────────────────────────────────────
• Nhớ cặp: OIDC↔ID token↔AuthN (ai), OAuth2↔access token↔AuthZ (làm gì).
  Đề thi rất hay trộn hai khái niệm này.
• Chọn flow theo "có user hay không": có user đăng nhập → Auth Code;
  app tự chạy nền → Client Credentials; API gọi API hộ user → OBO.
• SAS và Managed Identity cùng một triết lý "đừng dùng chìa vạn năng":
  account key là rủi ro lớn nhất; user delegation SAS + Managed
  Identity ([[07-Key-Vault-App-Configuration-Managed-Identity]]) loại
  bỏ key khỏi vòng đời.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. Phân biệt **AuthN** và **AuthZ**; mỗi cái gắn với chuẩn & loại token nào?
2. **App registration** sinh ra những gì? **Delegated** vs **application permission** khác nhau ra sao?
3. Khi nào dùng **Authorization Code** vs **Client Credentials** vs **On-Behalf-Of**?
4. Phân biệt **access / ID / refresh token**.
5. 3 loại **SAS**? Vì sao **user delegation SAS** an toàn hơn account key?

---

## Liên quan
- [[00-MOC-AZ-204]]
- [[07-Key-Vault-App-Configuration-Managed-Identity]] — Managed Identity (bỏ secret hẳn)
- [[05-Blob-Storage-SDK-Lifecycle]] — SAS dùng cho upload Blob
- [[../AZ-900/10-Identity-Security-AzureAD-RBAC]] — Entra ID/RBAC góc AZ-900
