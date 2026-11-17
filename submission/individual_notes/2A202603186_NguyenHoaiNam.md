# Ghi chú cá nhân — 2A202603186 - Nguyễn Hoài Nam

## 1. Pha 1: ba ca khó tôi tìm ra trong 40 ảnh

| Ảnh | v0 không trả lời được gì | Quyết định ban đầu của tôi |
| --- | --- | --- |
| img_004 | v0 không có loại nhãn cho xe tuk-tuk, nên không biết gán vào lớp nào. | Không gán; tick `can_review` (tuk-tuk chưa có lớp, để nhóm quyết ở Pha 2). |
| img_020 | v0 không nói về xe nhìn thấy trong gương chiếu hậu, nên không rõ có gán nhãn hay không. | Không gán (phản chiếu trong gương và thân xe của mình, quy tắc 9). |
| img_033 | v0 không nói về ma-nơ-canh, nên không biết gán hay bỏ qua. | Ma-nơ-canh không gán; phần gương minivan không gán, tick `can_review`. |

## 2. Pha 2: chỗ tôi bất đồng với nhóm

- Người ngồi/lái xe máy: bản của tôi gán `person` cho từng người (quy tắc 2). Bản Thức và Thảo không gán. Bản Hiếu và Thắng gán khi người ngồi trên xe hở. Nhóm chốt (v2.0): không gán `person` cho người ngồi/lái xe. Tôi đồng ý: theo v2.0, file nhãn 24 ảnh của tôi có 41 box `person`, chỉ gồm người đi bộ hoặc đứng trên đường, đúng mục tiêu của nhóm.
- Tuk-tuk, xe buýt, xe đạp: bản của tôi không có lớp, chỉ tick `can_review`. Bản Thức và Thắng có lớp riêng; bản Hiếu để `ignore_region`. Nhóm chốt (v2.0): có lớp `tuktuk`, `bus`, `bicycle`. Tôi đồng ý: ba loại xe này có hình dạng và cách di chuyển khác ô tô và xe máy, nên tách lớp giúp mô hình học tốt hơn, và file nhãn hiện dùng đúng 9 lớp của v2.0.
- Ngưỡng `occluded_level`: bản của tôi 10/40/70%, bản Hiếu 10/40/80%. Guideline v2.0 chưa ghi số cụ thể. Tôi chưa kết luận được, vì calibration chưa chạy nên chưa có dữ liệu để so hai ngưỡng. Đề nghị đưa vào Pha 2 để nhóm chốt.
- Xe trang trí (img_019): bản của tôi để không gán. Bản Hiếu và Thắng gán `car`. Chưa chốt, đưa vào Pha 2 để nhóm quyết.

## 3. Pha 3: ảnh tôi đã gán

- Số ảnh được giao: 24 (theo `submission/qc/assignments/2A202603186.txt`). Số box đã vẽ: 142 (export COCO 1.0 ngày 16/11, 17:18). Số lần tôi tick `can_review`: 69 box.
- Ca tôi lưỡng lự nhất khi gán (ảnh, vì sao):
  - Lưỡng lự nhất: có nên gán tuk-tuk là `car` không. Tôi phân vân vì tuk-tuk có bánh và chở người như xe con, nhưng khung ba bánh khác hẳn ô tô.
  - Guideline v2.0 chốt tuk-tuk là lớp `tuktuk` riêng, không gộp vào `car`.

## 4. Pha 4–5: tôi lệch ở đâu

- Trong `calibration_candidates.csv`, các ca có tên tôi: [điền sau khi nhóm chạy `prepare_calibration.py`; hiện file chưa có].
- Lệch đã thấy khi so nhãn với guideline v2.0 (chưa phải kết quả calibration): tôi gán `person` cho người ngồi trên xe máy, trong khi v2.0 không gán. Lý do: bản v0.1 của tôi tự viết quy tắc này trước khi nhóm chốt, nên tôi làm theo bản cá nhân chứ không theo quy tắc chung. Đây là lỗi nắm quy tắc, không phải guideline mơ hồ.
- Tuk-tuk: tôi phân vân giữa `car` và lớp riêng. Guideline v2.0 chốt lớp `tuktuk`, nên đây là ca cần sửa theo nhóm.

## 5. Dùng AI

- Công cụ: Claude (trợ lý AI trong phiên làm việc này).
- Dùng cho việc gì: Hỗ trợ thao tác cvat.

## 6. Bài học

- Một điều tôi sẽ làm khác khi viết guideline lần sau: Tách rõ phần việc của mình và phần của nhóm. Tôi đã trộn lẫn quyết định cá nhân với quy tắc chung. Lần sau tôi sẽ viết hai phần riêng, để khi vào Pha 2 biết ngay đâu là điều mình đề xuất, đâu là điều cần cả nhóm quyết.
