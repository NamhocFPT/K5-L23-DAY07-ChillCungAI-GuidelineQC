# Guideline gán nhãn — v2.0 — Nhóm ChillCungAI

- Phiên bản: v2.0
- Ngày: 2026-11-16
- Người viết / duyệt: Nhóm ChillCungAI
- Loại nhãn: bounding box trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0

## 1. Mục tiêu

Tạo dữ liệu để huấn luyện mô hình **xe tự lái** nhận biết người đi bộ và phương tiện trên đường.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) |
|---|---|
| `car` | Ô tô con, SUV, taxi. |
| `truck` | Xe tải, pickup hoặc xe có thùng hàng riêng. |
| `van` | Xe thùng kín cỡ nhỏ, thân liền. |
| `bus` | Xe chở khách dạng buýt hoặc minibus. |
| `motorcycle` | Xe hai bánh có động cơ. |
| `bicycle` | Xe hai bánh không có động cơ. |
| `tuktuk` | Xe ba bánh khung liền chở người hoặc hàng. |
| `person` | Người thật đang đi bộ hoặc đứng trên đường. |
| `ignore_region` | Vùng có nhiều vật thật nhưng không thể tách riêng. |

Vật giả, tượng, ma-nơ-canh, đồ chơi, hình in và phản chiếu không gán nhãn.

### Thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Mức bị che: 0 không che, 1 che ít, 2 che nhiều, 3 gần như bị che hết nhưng vẫn nhận ra. |
| `truncated` | `no`, `yes` | Chọn `yes` khi vật bị cắt ở mép ảnh. |
| `can_review` | tick / không | Tick khi không chắc về lớp, box hoặc thuộc tính. |

## 3. Quy tắc vẽ box

1. Gán box cho phương tiện thật và người đi bộ nhìn thấy trong ảnh.
2. Box ôm sát phần nhìn thấy của vật.
3. Người lái hoặc ngồi trên phương tiện không gán `person`.
4. Vật chạm mép ảnh đặt `truncated = yes`.
5. Vật quá nhỏ hoặc cụm vật không tách được dùng `ignore_region`.
6. Khi không chắc, tick `can_review` để nhóm kiểm tra lại.

## 4. Ca khó

| Tình huống | Quyết định |
|---|---|
| Tuk-tuk hoặc auto-rickshaw | Gán `tuktuk`; không gán `person` cho người ngồi/lái bên trong. |
| Người đi bộ cạnh phương tiện | Gán một box `person` riêng. |
| Người ngồi/lái trên xe | Chỉ gán box phương tiện, không gán `person`. |
| Cụm xe hoặc người quá dày | Gán `ignore_region` nếu không tách được từng vật. |
| Tượng, poster, đồ chơi, phản chiếu | Không gán. |

## 5. QC và phiên bản

- Box phải ôm sát vật và chọn đúng lớp.
- Mọi thay đổi về lớp hoặc quy tắc phải tăng phiên bản guideline.
- Ca mơ hồ phải tick `can_review` và đưa vào calibration của nhóm.
- Bản này là guideline nhóm v2.0, dùng thống nhất cho dữ liệu xe tự lái.

### Lịch sử thay đổi

| Phiên bản | Thay đổi |
|---|---|
| `v1.0` | Chỉ gán nhãn phương tiện. |
| `v2.0` | Bổ sung lớp `person`; chỉ gán cho người đi bộ, không gán cho người lái/ngồi trên xe. |
