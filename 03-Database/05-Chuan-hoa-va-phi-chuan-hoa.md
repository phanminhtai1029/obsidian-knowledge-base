---
title: "Chuẩn hóa & phi chuẩn hóa"
section: 03-Database
tags: [database, normalization, 1nf, 2nf, 3nf, denormalization, fresher]
related:
  - "[[02-Keys-va-rang-buoc]]"
  - "[[03-Quan-he-va-mo-hinh-ER]]"
  - "[[06-ACID-va-Transaction]]"
difficulty: ⭐⭐⭐
estimated_time: 30m
source: [LinkedIn "Database Foundations" - Scott Simpson, Edgar Codd]
---

# Chuẩn hóa & phi chuẩn hóa

> [!summary] TL;DR
> **Normalization** (Edgar Codd, đầu 1970s) = bộ quy tắc tổ chức dữ liệu để **giảm dư thừa + tăng toàn vẹn**. **1NF**: mỗi ô **atomic** (1 giá trị), không có nhóm lặp/cột lặp. **2NF**: (đã 1NF) không cột non-key nào phụ thuộc vào **một phần** của khóa (vấn đề của composite key). **3NF**: (đã 2NF) không cột non-key nào phụ thuộc vào **cột non-key khác** (không lưu giá trị suy ra được). Chuẩn tới 3NF là mức "đạt chuẩn" cho DB nghiệp vụ. **Denormalization** = cố ý nhân bản dữ liệu (vi phạm chuẩn) để **đổi nhất quán lấy tốc độ** — làm *sau* khi đã chuẩn hóa.

---

## 1. Ba normal form ⭐ (xây chồng lên nhau)

| Form | Yêu cầu | Vi phạm điển hình | Cách sửa |
|------|---------|-------------------|----------|
| **1NF** | Mỗi ô **atomic** (1 giá trị); không nhóm lặp / cột lặp; không hàng trùng | `FavoriteDish1, FavoriteDish2, FavoriteDish3`; hoặc nhét list `"3,7,15"` vào 1 ô | Tách nhóm lặp ra **bảng riêng** + nối bằng khóa |
| **2NF** | Đã 1NF; cột non-key phụ thuộc **toàn bộ** khóa (không chỉ một phần) | Với composite key (EventName+EventDate), cột `Location` chỉ phụ thuộc `EventName` | Tách thành bảng mới (`EventsLocations`) |
| **3NF** | Đã 2NF; cột non-key **không** phụ thuộc cột non-key khác | `LunchPrice = Price × 50%` (suy ra từ Price) | Bỏ cột suy ra được, hoặc tách bảng |

### 1NF — chi tiết
- "Atomic" = một ô chỉ chứa **một** giá trị.
- Không **repeating group**: không tạo cột kiểu `Dish1, Dish2…` và không nhét nhiều giá trị vào một ô.
- Mở rộng: không có **hàng trùng lặp hoàn toàn** → thêm khóa duy nhất cho bảng nối nếu cần.
- Thứ tự hàng/cột **không được** mang ý nghĩa (cần thứ tự → dùng auto-increment / timestamp).

### 2NF — chi tiết
Chỉ phát sinh khi có **composite key**. Nếu một cột non-key chỉ phụ thuộc **một phần** khóa ghép → tách ra. (Vd: địa điểm event suy ra được chỉ từ tên event, không cần ngày → tách `EventsLocations`.)

### 3NF — chi tiết
Cấm **phụ thuộc bắc cầu (transitive)**: nếu giá trị một cột suy ra được từ một cột **non-key** khác theo một quy tắc → vi phạm. (`LunchPrice` luôn = 50% `Price` → bỏ, tính khi cần.) Nhưng nếu giá trị **không** suy ra được (giá lunch đặt riêng từng món) thì nó là cột non-key độc lập → hợp lệ.

```
★ Insight ─────────────────────────────────────
• Mẹo nhớ 3NF: "The key, the whole key, and nothing but the key."
  - 1NF: dữ liệu phụ thuộc THE KEY (có khóa, ô atomic).
  - 2NF: phụ thuộc THE WHOLE KEY (toàn bộ khóa, không phần).
  - 3NF: NOTHING BUT THE KEY (chỉ khóa, không cột non-key khác).
• Lưu giá trị suy ra được (LunchPrice) gây rủi ro: ai đó sửa Price mà quên
  sửa LunchPrice → dữ liệu mâu thuẫn. Chuẩn hóa loại bỏ nguồn mâu thuẫn này.
─────────────────────────────────────────────────
```

## 2. Denormalization — đổi nhất quán lấy tốc độ

**Denormalization** = cố ý **nhân bản/lưu dữ liệu suy ra được**, vi phạm chuẩn — *sau khi* đã chuẩn hóa (không phải bỏ qua chuẩn hóa).

- Ví dụ: lưu sẵn `ItemCount` và `Total` trên bảng Orders (thay vì tính lại từ OrdersDishes mỗi lần).
- **Lợi**: truy vấn nhanh hơn (đỡ JOIN + tính toán) — đáng giá khi DB rất lớn, server chậm, hoặc lượng request khổng lồ.
- **Hại**: rủi ro **mất nhất quán** (sửa số lượng mà total không cập nhật).
- → Là **trade-off** có chủ đích, quyết định theo yêu cầu nghiệp vụ.

| | Normalization | Denormalization |
|--|---------------|-----------------|
| Mục tiêu | giảm dư thừa, tăng toàn vẹn | tăng tốc đọc |
| Dư thừa | thấp | có chủ đích |
| Rủi ro | join nhiều → chậm | mất nhất quán |
| Khi nào | mặc định (chuẩn tới 3NF) | tối ưu hiệu năng có chọn lọc |

## Tự kiểm tra
1. 1NF yêu cầu gì? *(ô atomic, không nhóm lặp/cột lặp, không hàng trùng)*
2. 2NF chỉ liên quan tới loại khóa nào? *(composite key — cấm phụ thuộc một phần khóa)*
3. 3NF cấm điều gì? *(phụ thuộc bắc cầu: non-key suy ra từ non-key khác)*
4. Câu thần chú 3NF? *("the key, the whole key, and nothing but the key")*
5. Denormalization đánh đổi gì lấy gì, và làm khi nào? *(đổi nhất quán lấy tốc độ; làm sau khi đã chuẩn hóa)*

## Liên quan
- [[00-MOC-Database|⬅ MOC Database]]
- Trước: [[04-Kieu-du-lieu-va-thiet-ke-bang]] · Kế tiếp: [[06-ACID-va-Transaction]]
- [[03-Quan-he-va-mo-hinh-ER|Linking table & 1NF]] · [[02-Keys-va-rang-buoc|Composite key & 2NF]]
