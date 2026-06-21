---
title: "Wireframe · Mockup · Prototype — phân biệt 3 khái niệm"
section: 00-Foundations/03-Agile-Scrum
tags: [wireframe, mockup, prototype, ux, ui, fidelity, figma, foundations, fresher]
related:
  - "[[09-Tiep-nhan-va-Phan-tich-Yeu-cau]]"
  - "[[10-Tai-lieu-Yeu-cau-va-Dac-ta]]"
difficulty: ⭐⭐⭐
estimated_time: 30m
source: [tổng hợp UX/UI practice]
---

# Wireframe · Mockup · Prototype — phân biệt 3 khái niệm

> [!summary] TL;DR
> Ba mức "bản vẽ" UI tăng dần độ chi tiết & độ thật: **Wireframe** = khung xương bố cục, **low-fidelity**, đen trắng, tập trung *cấu trúc & vị trí* (không màu, không tương tác). **Mockup** = bản **high-fidelity TĨNH**: có màu, font, hình ảnh, trông gần như thật nhưng **không bấm được**. **Prototype** = bản **TƯƠNG TÁC được**: click/chuyển màn hình, mô phỏng *luồng sử dụng* để test với người dùng. Thứ tự làm: Wireframe → Mockup → Prototype.

---

## 1. Bảng phân biệt ⭐⭐ (câu phỏng vấn kinh điển)

| | **Wireframe** | **Mockup** | **Prototype** |
|---|---|---|---|
| Fidelity (độ chi tiết) | **Low** (thấp) | **High** (cao) | High (thường) |
| Màu sắc / hình ảnh | Không (xám/đen trắng) | **Có** (màu, font, ảnh thật) | Có |
| Tương tác (bấm được)? | ❌ Không | ❌ Không (tĩnh) | ✅ **Có** |
| Tập trung vào | **Bố cục, cấu trúc, vị trí** | **Giao diện hình ảnh** (look & feel) | **Luồng & trải nghiệm** (flow) |
| Ví von | Bản phác khung nhà | Ảnh render ngôi nhà | Đi thử trong nhà mẫu |
| Công cụ | Balsamiq, giấy bút, Figma | Figma, Sketch, Adobe XD | Figma, InVision, Axure |
| Thời điểm | Sớm (ý tưởng) | Giữa (chốt giao diện) | Trước khi code (test luồng) |

```text
 ĐỘ THẬT / CHI TIẾT tăng dần ───────────────────────────────────▶
 Wireframe              Mockup                  Prototype
 ┌──────────┐          ┌──────────┐            ┌──────────┐
 │ [ logo ] │          │  🏋 GymUp │            │  🏋 GymUp │  ← bấm "Đăng nhập"
 │ ▭▭▭▭▭▭▭ │   →      │ ████ xanh│    →       │  → nhảy sang
 │ ▭▭▭  ▭▭ │          │ ảnh, font│            │     màn hình kế
 │ [ nút  ] │          │ đẹp, tĩnh│            │  (tương tác thật)
 └──────────┘          └──────────┘            └──────────┘
 khung xương           trông như thật          dùng thử được
```

---

## 2. Wireframe — khung xương

- **Low-fidelity**: hộp, đường, placeholder (chữ "Lorem ipsum", ô xám thay ảnh).
- Trả lời: **cái gì đặt ở đâu?** (header, menu, nội dung, nút) — **bố cục & phân cấp thông tin**.
- Cố tình **xấu/đơn giản** để mọi người góp ý về *cấu trúc* mà không bị phân tâm bởi màu mè.
- Nhanh, rẻ, dễ sửa.

```
★ Insight ─────────────────────────────────────
• Wireframe đen trắng là CHỦ Ý, không phải lười: khi cho khách xem bản nhiều màu,
  họ sa đà bàn "nút này nên xanh hay đỏ" thay vì "luồng này có hợp lý không".
  Low-fidelity giữ thảo luận ở đúng tầng cấu trúc — sửa cấu trúc lúc này cực rẻ,
  sửa khi đã code thì rất đắt (giống "shift left" ở [[../02-Git/12-CI-CD-la-gi]]).
─────────────────────────────────────────────────
```

---

## 3. Mockup — bản tĩnh gần thật

- **High-fidelity nhưng TĨNH**: có màu thương hiệu, typography, icon, ảnh thật, spacing chuẩn.
- Trông gần như sản phẩm cuối, nhưng **không bấm được** (chỉ là "ảnh").
- Dùng để **chốt giao diện (visual design)** với khách trước khi code; là cầu nối tới dev (đo màu, khoảng cách).

> [!tip] Wireframe + thêm "da thịt" (màu, font, ảnh) = Mockup. Mockup + thêm "tương tác" (click chuyển màn) = Prototype.

---

## 4. Prototype — bản tương tác

- **Bấm được, chuyển màn hình được**, mô phỏng **luồng sử dụng (user flow)** thật.
- Mục đích chính: **usability testing** — cho người dùng thử để phát hiện vấn đề trải nghiệm **trước khi** tốn công code.
- Fidelity có thể thấp (prototype giấy/click-through đơn giản) hoặc cao (gần như app thật), nhưng điểm phân biệt là **tính tương tác**.

| Mức prototype | Ví dụ |
|---------------|-------|
| Low-fi interactive | Click-through wireframe (bấm hình chuyển trang) |
| High-fi interactive | Figma prototype có animation, gần như app thật |

```
★ Insight ─────────────────────────────────────
• Điểm phân biệt CỐT LÕI giữa Mockup và Prototype không phải "đẹp/xấu" mà là
  TƯƠNG TÁC: mockup là danh từ tĩnh (một bức ảnh), prototype là động từ (một trải
  nghiệm bấm được). Có mockup high-fi mà vẫn không phải prototype nếu không click
  được. Trả lời được đúng chỗ này là "ăn điểm" câu phỏng vấn.
• Cả ba đều phục vụ làm rõ requirement ([[09-Tiep-nhan-va-Phan-tich-Yeu-cau]]):
  thay vì tả bằng lời (dễ hiểu lầm), cho khách THẤY → phản hồi chính xác hơn nhiều.
  "Một bản vẽ bằng ngàn dòng SRS."
─────────────────────────────────────────────────
```

---

## 5. Vài khái niệm liên quan (bonus)

| Thuật ngữ | Nghĩa |
|-----------|-------|
| **Fidelity** | Độ "giống thật"/chi tiết của bản thiết kế (low ↔ high) |
| **UX** (User Experience) | Trải nghiệm tổng thể: luồng, cảm giác, dễ dùng |
| **UI** (User Interface) | Bề mặt giao diện: màu, nút, layout |
| **Mockup ≠ Prototype** | Tĩnh vs tương tác (đừng dùng lẫn lộn) |
| **Design system** | Bộ component & quy tắc UI dùng lại (vd Material, Bootstrap) |

> Công cụ "quốc dân" hiện nay: **Figma** (làm được cả 3: wireframe, mockup, prototype + cộng tác realtime). Khác: Balsamiq (chuyên wireframe), Adobe XD, Sketch.

---

## 6. Tự kiểm tra (luyện trả lời phỏng vấn)

1. Wireframe khác Mockup khác Prototype thế nào? *(wireframe: low-fi khung xương; mockup: high-fi tĩnh; prototype: tương tác được)*
2. Điểm phân biệt cốt lõi mockup vs prototype? *(prototype TƯƠNG TÁC được; mockup tĩnh)*
3. Vì sao wireframe để đen trắng? *(giữ thảo luận ở tầng cấu trúc, không bị phân tâm bởi màu)*
4. Fidelity nghĩa là gì? *(độ giống thật/chi tiết)*
5. Thứ tự làm 3 loại? *(Wireframe → Mockup → Prototype)*
6. Một công cụ làm được cả 3? *(Figma)*

## Liên quan
- [[00-MOC-Agile-Scrum|⬅ MOC Agile-Scrum]]
- Trước: [[10-Tai-lieu-Yeu-cau-va-Dac-ta]] · Kế tiếp: [[12-Estimation|Estimation]]
- [[09-Tiep-nhan-va-Phan-tich-Yeu-cau|Làm rõ requirement]]
