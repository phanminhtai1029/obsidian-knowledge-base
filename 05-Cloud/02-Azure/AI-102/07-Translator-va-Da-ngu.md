---
title: "Translator & nội dung đa ngữ"
section: 05-Cloud/02-Azure/AI-102
tags: [azure, ai-102, translator, localization, fresher]
related:
  - "[[00-MOC-AI-102]]"
  - "[[06-Azure-AI-Language]]"
difficulty: ⭐⭐
estimated_time: 16m
source: ["_source/Microsoft/AI-102.docx (Lesson 13, Tim Warner)", "Microsoft Learn"]
status: ✅
---

# Translator & nội dung đa ngữ

> [!summary] TL;DR
> **Azure AI Translator** là dịch vụ **dịch máy** (neural machine translation) hỗ trợ 100+ ngôn ngữ. Năng lực: **text translation** (dịch văn bản, một nguồn → **nhiều đích cùng lúc**), **transliteration** (chuyển **chữ viết** giữa bảng ký tự, ví dụ tiếng Nhật kanji → romaji — *khác* dịch nghĩa), **language detection** (tự nhận ngôn ngữ nguồn), **dictionary lookup** (tra nghĩa + ví dụ thay thế). Để dịch **nguyên file giữ định dạng** (Word/PDF/PowerPoint) dùng **Document Translation** (batch, qua Blob Storage) thay vì cắt text thủ công. Khi thuật ngữ chuyên ngành bị dịch sai → **Custom Translator** (train trên cặp câu song ngữ của bạn để model dùng đúng từ chuyên môn). Có **profanity filter** (lọc từ tục) và **dynamic dictionary** (ép dịch cố định một số từ). Translator chỉ làm **text**; dịch **giọng nói** là **Speech Translation** (note 8).

> **Thuật ngữ:** *transliteration* = chuyển hệ chữ viết (không đổi nghĩa, chỉ đổi cách viết). *localization* = bản địa hoá nội dung cho thị trường. *batch* = xử lý hàng loạt. *profanity filter* = bộ lọc từ thô tục.

---

## 1. Translator API (text, transliteration, detect)

| Năng lực | Làm gì |
|---|---|
| **Translate** | Dịch text; **một nguồn → nhiều ngôn ngữ đích** trong một request |
| **Transliterate** | Đổi **bảng chữ** (kanji→romaji), không đổi nghĩa |
| **Detect** | Tự nhận ngôn ngữ nguồn (không cần khai báo) |
| **Dictionary lookup / examples** | Tra các bản dịch thay thế + ví dụ ngữ cảnh |

```python
# Text Translation: 1 nguồn -> nhiều đích cùng lúc
import requests
url = "https://api.cognitive.microsofttranslator.com/translate"
params = {"api-version": "3.0", "to": ["vi", "fr", "ja"]}   # 3 ngôn ngữ đích
headers = {"Ocp-Apim-Subscription-Key": "<KEY>", "Ocp-Apim-Subscription-Region": "<REGION>"}
body = [{"text": "Hello world"}]
r = requests.post(url, params=params, headers=headers, json=body)   # tự detect nguồn
print(r.json())   # trả về bản dịch cho vi/fr/ja
```

> `Ocp-Apim-Subscription-Region` thường **bắt buộc** với Translator (khác phần lớn dịch vụ AI khác) — hay bị bẫy trong đề.

---

## 2. Document Translation

| | **Text Translation** | **Document Translation** |
|---|---|---|
| Đầu vào | Chuỗi text | **Cả file** (Word/PDF/PPTX/HTML…) |
| Giữ định dạng | Không (chỉ text) | **Giữ layout/định dạng** |
| Cách chạy | Đồng bộ, từng request | **Batch async** qua Blob Storage (source → target container) |
| Dùng khi | Dịch đoạn ngắn trong app | Dịch tài liệu nguyên bản hàng loạt |

---

## 3. Custom Translator

- Khi nội dung có **thuật ngữ chuyên ngành** (y khoa, pháp lý, sản phẩm riêng) mà bản dịch chung sai → train **Custom Translator** trên **cặp câu song ngữ** của bạn.
- Kết quả: một **category ID** dùng trong API translate để model áp đúng từ vựng chuyên môn.
- Phân biệt nhanh:
  - **Dynamic dictionary** = ép dịch cố định **vài từ** ngay trong request (không train).
  - **Custom Translator** = **train cả model** cho phong cách/thuật ngữ (cần dữ liệu song ngữ).

> [!question] Phỏng vấn: "Dịch nguyên một file PDF mà giữ định dạng — dùng gì?"
> **Document Translation** (batch async qua Blob Storage: container nguồn → container đích), nó **giữ layout/định dạng** của Word/PDF/PowerPoint. Text Translation API chỉ dịch **chuỗi text** nên sẽ mất định dạng nếu tự cắt.

> [!question] Phỏng vấn: "Thuật ngữ chuyên ngành bị dịch sai, sửa thế nào?"
> Nếu chỉ vài từ → **dynamic dictionary** (ép bản dịch trong request, không train). Nếu sai có hệ thống cả phong cách/thuật ngữ → **Custom Translator**: train trên **cặp câu song ngữ** của domain rồi dùng **category ID** khi gọi translate.

---

```
★ Insight ─────────────────────────────────────
• Transliteration ≠ translation: đổi CHỮ VIẾT chứ không đổi nghĩa —
  câu hỏi bẫy kinh điển của Translator.
• Text vs Document Translation = "đoạn text" vs "nguyên file giữ
  định dạng (batch qua Blob)" — đề thi rất hay phân biệt.
• Translator chỉ làm text; dịch giọng nói là Speech Translation
  (note 8) — đừng nhầm hai dịch vụ.
─────────────────────────────────────────────────
```

---

## Tự kiểm tra

1. Translate vs transliterate khác nhau gì?
2. Một request Translator có dịch ra nhiều ngôn ngữ đích cùng lúc không?
3. Text Translation vs Document Translation — chọn theo tiêu chí nào?
4. Dynamic dictionary vs Custom Translator — khi nào dùng cái nào?
5. Dịch giọng nói thì dùng Translator hay dịch vụ nào khác?

---

## Liên quan
- [[00-MOC-AI-102]]
- [[06-Azure-AI-Language]] — NLP các tính năng khác
- [[08-Speech-to-Text-Text-to-Speech-SSML]] — speech translation (dịch giọng nói)
