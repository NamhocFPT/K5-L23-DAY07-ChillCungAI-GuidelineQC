# Guideline gán nhãn — v1.0 thống nhất

- Loại nhãn: bounding box trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0
- Phạm vi: 40 ảnh đường phố châu Á của Day 07

## 1. Mục tiêu

Tạo bộ nhãn nhất quán để phát hiện người đi bộ và phương tiện trên ảnh đường phố,
phục vụ các bài toán nhận thức môi trường cho hệ thống xe tự lái.
Mỗi vật được gán tối đa một lớp. Các ca chưa chắc phải dùng `can_review = yes`, không tự ý đổi quy tắc giữa lúc gán nhãn.

## 2. Lớp và thuộc tính

### 2.1 Lớp

| Lớp | Định nghĩa | Ví dụ / phản ví dụ |
|---|---|---|
| `car` | Ô tô con, sedan, hatchback hoặc SUV dùng chở người. | Không gán cho xe tải hoặc xe thùng kín. |
| `truck` | Xe tải hoặc xe có thùng/chở hàng riêng. | Xe bán tải có thùng hàng rõ ràng được xem là `truck`. |
| `van` | Xe thùng kín cỡ nhỏ, thân xe chủ yếu dùng để chở người/hàng. | Không gán cho ô tô con chỉ vì có màu trắng hoặc hình hộp. |
| `motorcycle` | Xe máy, mô tô, xe hai bánh có động cơ. | Không gán cho người đi bộ đẩy xe nếu không nhận diện được xe máy. |
| `tuk_tuk` | Xe ba bánh có động cơ, xe lam hoặc tuk-tuk dùng chở người/hàng. | Không gán cho xe máy kéo theo một vật nếu không phải xe ba bánh có cấu trúc riêng. |
| `bicycle` | Xe đạp không có động cơ, gồm xe đạp thường và xe đạp chở hàng. | Không gán cho xe máy điện hoặc xe có động cơ; các xe đó thuộc `motorcycle` nếu nhận diện được. |
| `person` | **Chỉ người đang đi bộ.** | Người đang ngồi/lái trên xe, tượng, ma-nơ-canh và hình người không gán `person`. |
| `ignore_region` | Vùng được nhóm xác định là không cần gán từng vật. | Không dùng thay cho một vật thể còn có thể nhận diện và gán lớp. |

Vật không thuộc các lớp trên, vật giả tưởng, tượng, ma-nơ-canh, hình in và phản chiếu không gán lớp vật thể. Người ngồi trên xe không gán `person`; chỉ gán phương tiện nếu phương tiện thuộc taxonomy.

### 2.2 Thuộc tính

| Thuộc tính | Giá trị thống nhất | Quy tắc |
|---|---|---|
| `occluded_level` | `0` | Không bị che hoặc phần bị che không đáng kể (0%). |
|  | `1` | Bị che trên 0% đến 30%, vẫn nhận diện rõ. |
|  | `2` | Bị che trên 30% đến 60%, còn thấy phần chính của vật. |
|  | `3` | Bị che trên 60% hoặc không đủ rõ để xác định chắc chắn. |
| `truncated` | `no` / `yes` | Chọn `yes` khi vật bị cắt bởi mép ảnh; không chọn chỉ vì vật bị vật khác che. |
| `can_review` | `no` / `yes` | Chọn `yes` khi còn nghi ngờ về lớp, ranh giới box hoặc thuộc tính. |

## 3. Quy tắc vẽ box

1. Box phải ôm sát phần nhìn thấy của vật, không lấy dư nền không liên quan.
2. Mỗi vật độc lập một box; không gộp nhiều người hoặc nhiều xe thành một box, trừ vùng được đánh dấu `ignore_region`.
3. Vật bị che vẫn được gán nếu còn nhận diện được. Vẽ theo phần vật có thể quan sát, đặt `occluded_level` tương ứng.
4. Vật bị cắt bởi mép ảnh vẫn được gán nếu nhận diện được; box dừng tại mép ảnh và đặt `truncated = yes`.
5. Người đi bộ đang cầm đồ vật không biến đồ vật đó thành một box riêng nếu đồ vật không thuộc taxonomy.
6. Người ngồi hoặc lái xe không gán `person`; không vẽ box người riêng.
7. Vật quá nhỏ, quá mờ hoặc không thể xác định lớp: không đoán lớp; dùng `ignore_region` nếu đó là vùng cần bỏ qua theo quyết định của nhóm, nếu không thì đặt `can_review = yes` và mang ra calibration.
8. Vật ngoài taxonomy và vật không thật không gán vào lớp gần giống nhất.

## 4. Ca khó và quyết định thống nhất

- Người đi bộ cạnh hoặc trước xe: gán một box `person` cho người và một box cho xe nếu xe nhìn thấy rõ.
- Người đang ngồi/lái trên xe: chỉ gán box phương tiện; không gán box `person`.
- Tượng, ma-nơ-canh, hình người trên biển quảng cáo: không gán `person`.
- Xe chở vật khác: gán box cho xe thuộc taxonomy; vật được chở chỉ gán nếu bản thân nó là một vật thuộc taxonomy và nhìn thấy độc lập.
- Xe ba bánh/tuk-tuk: gán `tuk_tuk` cho toàn bộ phương tiện, không tách phần cabin và thùng thành các box riêng.
- Xe đạp: gán `bicycle` khi nhận diện được khung và bánh xe; người đang đạp xe không gán `person` theo quyết định của nhóm.
- Xe giả tưởng hoặc vật có hình dáng giống xe nhưng không phải phương tiện thật: không ép vào `car`, `truck`, `van` hoặc `motorcycle`.
- Ca chưa chắc trong lúc gán: giữ nguyên guideline, đặt `can_review = yes` và ghi chú để calibration.

## 5. QC và phiên bản

- Mọi ảnh được gán độc lập bởi số người theo `assignment.csv`; không xem nhãn của nhau trước khi đo IAA.
- Sau khi mọi người hoàn tất, chạy `validate_labels.py --all`, `qc_compare.py` và xem các ảnh bị cờ.
- Các bất đồng được đưa ra calibration; quyết định phải ghi một dòng trong `calibration_decisions.csv`.
- Phiên bản v1.0 thống nhất từ v0: giới hạn `person` còn người đi bộ; bổ sung `tuk_tuk` và `bicycle`; xác định mục tiêu phục vụ nhận thức cho xe tự lái; bổ sung định nghĩa lớp, ngưỡng thuộc tính, quy tắc box và xử lý ca ngoài taxonomy.
- Quy tắc mới sau calibration sẽ ghi ở `guideline_v1.1_calibrated.md`.
