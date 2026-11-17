# Guideline gán nhãn — v1.1 — Nhóm ChillCungAI

- Phiên bản: v1.1
- Ngày: 2026-11-17
- Người viết / duyệt: Nhóm ChillCungAI
- Loại nhãn: bounding box trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0

## 1. Mục tiêu

Tạo dữ liệu nhất quán để huấn luyện mô hình nhận biết người đi bộ và phương tiện trong ảnh đường phố.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa và ranh giới |
|---|---|
| `car` | Ô tô con, SUV hoặc taxi; không dùng cho xe có thùng hàng riêng. |
| `truck` | Xe tải, pickup hoặc xe có thùng hàng riêng. Pickup vẫn là `truck`. |
| `van` | Xe thùng kín cỡ nhỏ, thân liền; không có thùng hàng tách biệt. |
| `bus` | Xe buýt hoặc minibus dùng để chở nhiều hành khách. |
| `motorcycle` | Xe hai bánh có động cơ. |
| `bicycle` | Xe hai bánh không có động cơ. |
| `tuktuk` | Xe ba bánh khung liền chở người hoặc hàng, gồm auto-rickshaw. |
| `person` | Người thật đang đi bộ hoặc đứng trên đường. Người lái hoặc ngồi trên phương tiện không gán `person`. |
| `ignore_region` | Vùng có nhiều vật thật quá nhỏ hoặc quá dày để tách thành từng box; không phải một lớp vật thể. |

Không gán vật giả, tượng, ma-nơ-canh, đồ chơi, hình in, tranh ảnh hoặc phản chiếu. Nếu chúng nằm trong vùng đông vật thật và không thể tách riêng, chỉ dùng `ignore_region` theo quy tắc bên dưới.

### Thuộc tính

| Thuộc tính | Giá trị | Quy tắc |
|---|---|---|
| `occluded_level` | `0` | Không bị vật khác che. |
| `occluded_level` | `1` | Bị che trên 0% đến dưới 25% diện tích vật dự kiến. |
| `occluded_level` | `2` | Bị che từ 25% đến 75% diện tích vật dự kiến. |
| `occluded_level` | `3` | Bị che trên 75% nhưng phần còn lại vẫn đủ nhận ra lớp. |
| `truncated` | `no` / `yes` | Chọn `yes` khi vật bị cắt bởi mép ảnh; không dùng thuộc tính này cho phần bị vật khác che. |
| `can_review` | `false` / `true` | Chọn `true` khi sau khi áp dụng guideline vẫn còn nghi ngờ về lớp, box hoặc thuộc tính. |

Nếu vừa bị che vừa bị cắt mép, đặt cả `occluded_level` và `truncated` theo hai nguyên nhân riêng biệt.

## 3. Quy tắc vẽ box

1. Mỗi vật thật đủ lớn và tách được phải có một box riêng.
2. Box ôm sát phần vật nhìn thấy trong ảnh; không kéo box qua vùng bị che để đoán phần khuất.
3. Khi vật chạm mép ảnh, box chỉ bao phần nhìn thấy và đặt `truncated = yes`.
4. Vật có chiều cao dưới 20 px không gán box riêng; nếu có nhiều vật nhỏ hoặc không thể tách riêng, vẽ một `ignore_region` bao quanh vùng đó.
5. `ignore_region` không dùng để thay thế box của một vật đơn lẻ vẫn có thể nhận diện và tách được.
6. Người đi bộ đứng hoặc đi cạnh phương tiện được gán `person` riêng. Người lái/ngồi trên xe chỉ gán box phương tiện.
7. Với xe bị che, vẫn gán nếu nhận ra lớp; dùng `occluded_level` để ghi mức che. Không gán nếu chỉ còn dấu hiệu không đủ xác định lớp.
8. Khi không chắc giữa hai lớp, chọn lớp theo định nghĩa ở mục 2 và đặt `can_review = true`; không tự tạo lớp mới.

## 4. Các ca khó thống nhất

| Tình huống | Quyết định |
|---|---|
| Pickup hoặc xe có thùng hàng riêng | `truck`, không phải `car`. |
| Xe ba bánh khung liền/auto-rickshaw | `tuktuk`. |
| Người đi bộ cạnh xe | Gán thêm một box `person`. |
| Người lái hoặc ngồi trên xe | Không gán `person`; chỉ gán phương tiện. |
| Tượng, poster, ma-nơ-canh, đồ chơi, phản chiếu | Không gán. |
| Vật dưới 20 px chiều cao | Không gán riêng; dùng `ignore_region` nếu cần biểu diễn vùng đông vật. |
| Nhiều vật chồng lấn nhưng vẫn tách được | Gán từng box; không dùng `ignore_region`. |
| Vật bị cắt ở mép ảnh | Box phần nhìn thấy, `truncated = yes`. |
| Vật bị vật khác che | Box phần nhìn thấy, đặt `occluded_level` theo tỷ lệ che. |
| Không chắc sau khi áp dụng quy tắc | Gán theo quyết định tốt nhất và đặt `can_review = true`. |

## 5. QC và lịch sử phiên bản

- Kiểm tra máy: tên lớp, thuộc tính hợp lệ, box trong ảnh, chiều cao tối thiểu và đủ ảnh được giao.
- Kiểm tra bằng mắt: box có ôm sát vật không, lớp có đúng thực tế không, có bỏ sót vật mà cả nhóm cùng bỏ sót không.
- Các ca `can_review` và mọi bất đồng IAA phải được đưa vào calibration; không tự đổi quy tắc giữa lúc gán độc lập.
- Ngưỡng QC giữ theo `submission/taxonomy.json`: IoU IAA tối thiểu `0.85`, đồng thuận lớp tối thiểu `95%`, chiều cao box tối thiểu `20 px`.

### Lịch sử thay đổi

| Phiên bản | Thay đổi |
|---|---|
| `v1.0` | Guideline phương tiện ban đầu. |
| `v2.0` | Bổ sung lớp `person`. |
| `v1.1` | Làm rõ ranh giới lớp, box nhỏ, che khuất, cắt mép, `ignore_region` và `can_review` để dùng sau calibration. |
