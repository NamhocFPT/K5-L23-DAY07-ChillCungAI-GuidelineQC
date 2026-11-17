## 1. Mục tiêu

Xây dựng bộ dữ liệu gán nhãn phục vụ huấn luyện mô hình phát hiện người và phương tiện giao thông trên ảnh đường phố, ứng dụng trong hệ thống xe tự lái.

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa |
|---|---|
| `car` | Ô tô con. |
| `truck` | Xe tải. |
| `van` | Xe thùng kín cỡ nhỏ. |
| `motorcycle` | Xe máy, mô tô. |
| `bicycle` | Xe đạp. |
| `other vehicle` | Phương tiện khác. |
| `person` | Người. |

### Thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Mức bị che: 0 là thấy rõ, 1 là bị che ít, 2 là bị che nhiều, 3 là không rõ. |
| `truncated` | `no`, `yes` | Vật bị cắt ở mép ảnh. |
| `can_review` | tick / không | Tick khi bạn không chắc, để người khác xem lại. |

## 3. Quy tắc vẽ box

Gán nhãn cho tất cả người và phương tiện có thể nhận diện trong ảnh.

Bounding Box phải ôm sát đối tượng, hạn chế lấy thừa nền hoặc bỏ sót phần đối tượng nhìn thấy.

Chọn đúng class theo đặc điểm nhận diện của phương tiện hoặc người.

Gán nhãn phương tiện đồng thời phải gán nhãn người điều khiển hoặc người ngồi trên phương tiện nếu có thể quan sát và nhận diện được.

Không bỏ qua đối tượng chỉ vì bị che khuất một phần hoặc nằm sát mép ảnh.

Sử dụng thuộc tính occluded_level và truncated độc lập với nhau.

Nếu không chắc chắn về class hoặc thuộc tính, đánh dấu can_review để kiểm tra lại.

## 4. Ca khó

1. Người ngồi trên xe máy: Gán cả person và motorcycle nếu nhận diện được cả hai.

2. Phương tiện bị che khuất: Vẫn gán nhãn nếu có đủ đặc điểm để nhận diện; chọn occluded_level phù hợp.

3. Phương tiện bị cắt ở mép ảnh: Gán nhãn phần đối tượng nhìn thấy và đặt truncated = yes.

4. Người bị che bởi phương tiện khác: Gán nhãn nếu vẫn nhận diện được người; đánh giá mức độ occlusion.

5. Xe có hình dạng khó phân biệt giữa car, van, truck: Đối chiếu đặc điểm hình dạng và mục đích sử dụng; nếu chưa chắc chắn, đánh dấu can_review.

6. Phương tiện không thuộc các lớp đã định nghĩa: Gán other vehicle nếu xác định được đó là phương tiện.

7. Đối tượng rất nhỏ hoặc ở xa: Gán nếu vẫn nhận diện được; đánh dấu can_review nếu không đủ căn cứ.

8. Nhiều đối tượng chồng lấn: Tạo box riêng cho từng đối tượng có thể nhận diện, không gộp nhiều đối tượng thành một box.

## 5. QC và phiên bản

Mỗi nhãn cần được kiểm tra về class, Bounding Box và các thuộc tính.

Các trường hợp bất đồng giữa thành viên cần được đối chiếu với ảnh gốc và thống nhất cách xử lý.

Các trường hợp chưa chắc chắn được đánh dấu can_review.

Mọi thay đổi làm ảnh hưởng đến quy tắc gán nhãn phải cập nhật phiên bản guideline.

Sau calibration, cập nhật guideline từ v1.0 lên v1.1 và ghi rõ những thay đổi.

Thực hiện QC lại để đánh giá mức độ đồng thuận sau khi cập nhật guideline.
