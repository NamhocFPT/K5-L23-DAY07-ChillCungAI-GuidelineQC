# Ghi chú cá nhân — 2A202603116 - Nguyễn Quang Thực

## 1. Pha 1: ba ca khó tôi tìm ra trong 40 ảnh

| Ảnh | v0 không trả lời được gì | Quyết định ban đầu của tôi |
|---|---|---|
| img_0__ | Bổ sung ảnh và tình huống thực tế sau khi tự xem đủ 40 ảnh. | Chưa chốt |
| img_0__ | Bổ sung ảnh và tình huống thực tế sau khi tự xem đủ 40 ảnh. | Chưa chốt |
| img_0__ | Bổ sung ảnh và tình huống thực tế sau khi tự xem đủ 40 ảnh. | Chưa chốt |

## 2. Pha 2: chỗ tôi bất đồng với nhóm

- Quyết định thống nhất quan trọng: `person` chỉ dùng cho người đi bộ; người ngồi hoặc lái trên xe không gán `person`.
- Các bất đồng khác và lý do nhóm chốt: bổ sung sau buổi họp nhóm.

## 3. Pha 3: ảnh tôi đã gán

- Số ảnh được giao: **24**.
- Số box đã vẽ: bổ sung sau khi hoàn thành gán nhãn.
- Số lần tôi chọn `can_review = yes`: bổ sung sau khi hoàn thành gán nhãn.
- Ca tôi lưỡng lự nhất khi gán: bổ sung sau khi xem và gán các ảnh được phân công.

## 4. Pha 4–5: tôi lệch ở đâu

- Các ca có tên tôi trong `calibration_candidates.csv`: bổ sung sau khi chạy IAA.
- Phân loại nguyên nhân: guideline mơ hồ hay chưa nắm quy tắc — bổ sung sau calibration.
- Quyết định calibration quan trọng nhất: bổ sung sau buổi calibration.

## 5. Dùng AI

- Dùng AI để đọc và tóm tắt quy trình bài, kiểm tra cấu trúc file, hỗ trợ sửa lỗi script/định dạng CVAT và kiểm tra tính hợp lệ của JSON.
- Không dùng AI hoặc mô hình để xem ảnh và tự động gán box thay cho người gán nhãn.
- Các nhãn cuối cùng phải do thành viên tự quan sát ảnh và thực hiện trên CVAT.

## 6. Bài học

- Cần chốt định nghĩa lớp và ngưỡng thuộc tính bằng ví dụ cụ thể trước khi bắt đầu gán nhãn.
- Cần giữ nguyên guideline sau khi thống nhất trong Pha 2; các ca chưa chắc phải đánh dấu để calibration thay vì tự đổi quy tắc.
