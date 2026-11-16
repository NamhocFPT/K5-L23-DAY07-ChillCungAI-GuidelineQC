# Guideline gán nhãn — v1.0 (bản cá nhân) — 2A202603116 - Nguyễn Quang Thực

> Bản cá nhân của Nguyễn Quang Thực, được viết từ `guideline_v0_starter.md`.
> Các quyết định bổ sung cần được đối chiếu với guideline thống nhất của nhóm.

- Phiên bản: v1.0 (bản cá nhân)
- Ngày: 2026-11-16
- Người viết: 2A202603116 - Nguyễn Quang Thực
- Loại nhãn: bounding box (hộp chữ nhật) trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người đi bộ và phương tiện** trên ảnh đường phố
(ví dụ cho camera giao thông hoặc xe tự lái).

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa |
|---|---|
| `car` | Ô tô con. |
| `truck` | Xe tải, pickup hoặc xe có cabin và thùng hàng riêng. |
| `van` | Xe thùng kín cỡ nhỏ. |
| `motorcycle` | Xe máy, xe tay ga hoặc mô tô hai bánh có động cơ. |
| `bicycle` | Xe đạp hai bánh không có động cơ. |
| `tuktuk` | Xe ba bánh khung liền, gồm tuk-tuk và auto-rickshaw. |
| `person` | Người thật đang đi bộ hoặc đứng trên đường. Người đang ngồi/lái trên xe không gán `person`. |
| `ignore_region` | Vùng không cần gán từng vật. |

### Thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Mức bị che: 0 là thấy rõ, 1 là bị che ít, 2 là bị che nhiều, 3 là không rõ nhưng vẫn nhận ra được. |
| `truncated` | `no`, `yes` | Vật bị cắt ở mép ảnh. |
| `can_review` | tick / không | Tick khi bạn không chắc, để người khác xem lại. |

## 3. Quy tắc vẽ box

1. Gán box cho mọi người đi bộ và phương tiện thật trong ảnh.
2. Box ôm sát phần nhìn thấy của vật.
3. Chọn đúng lớp và thuộc tính.
4. Người đi bộ gán `person` riêng; người đang ngồi hoặc lái trên xe không gán `person`.
5. Tuk-tuk và auto-rickshaw gán lớp `tuktuk`, không gộp vào `motorcycle` hoặc `truck`.
6. Vật quá nhỏ hoặc cụm vật không thể tách riêng có thể gán `ignore_region`.
7. Tượng, ma-nơ-canh, đồ chơi, hình in và phản chiếu không gán box.

## 4. Ca khó

| Ca khó | Cách xử lý |
|---|---|
| Tuk-tuk/auto-rickshaw nhìn thấy rõ | Gán một box `tuktuk`. |
| Người lái hoặc ngồi trong tuk-tuk | Chỉ gán `tuktuk`; không gán `person` cho người đó. |
| Người đi bộ cạnh phương tiện | Gán thêm một box `person` riêng. |
| Không chắc lớp hoặc vật thật | Gán theo nhận định tốt nhất và tick `can_review`. |
| Cụm người/phương tiện quá dày | Dùng `ignore_region` nếu không thể tách từng vật. |

## 5. QC và phiên bản

- Mọi thay đổi làm đổi nhãn đều tăng phiên bản guideline.
- Ca mơ hồ: tick `can_review`.
- Các lớp và quy tắc trong bản cá nhân phải được đối chiếu với guideline thống nhất của nhóm trước khi gán chính thức.
