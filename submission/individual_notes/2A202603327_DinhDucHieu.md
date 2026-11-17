# Ghi chú cá nhân — 2A202603327 - Đinh Đức Hiếu

## 1. Pha 1: ba ca khó tôi tìm ra
| Ảnh | v0 không trả lời được gì | Quyết định ban đầu |
|---|---|---|
| img_002 | Xe tải chở ô tô: 1 box hay từng xe? | 1 `truck` + mỗi ô tô 1 `car` |
| img_017, img_027, img_033, img_038 | Tượng, ma-nơ-canh, đồ chơi có gán không? | Không gán (vật giả) |
| img_004, img_016, img_023 | Tuk-tuk, xích lô không thuộc lớp nào | Xe → `ignore_region`, người trong xe → `person` |

## 2. Pha 2: chỗ tôi khác nhóm
- **Xe ngoài lớp:** tôi để `ignore_region`; nhóm thêm lớp `bus`, `bicycle`, `tuktuk`. Tôi đồng ý vì các xe này xuất hiện nhiều.
- **Người trên xe:** tôi gán `person`; nhóm (v2.0) chỉ gán người đi bộ. Tôi đồng ý vì dễ thống nhất, ít box chồng nhau.
- **Xích lô (img_023):** chưa có quy tắc, tôi gán `tuktuk` + `can_review`, cần chốt ở calibration.

## 3. Pha 3: ảnh tôi đã gán
- 24 ảnh, **137 box** (motorcycle 48, car 36, person 21, ignore_region 10, truck 8, tuktuk 5, bus 4, van 3, bicycle 2). 4 ảnh để trống vì là vật giả.
- Tick `can_review`: **13** lần (xe nhoè, xe bị che nhiều, xích lô, xe diễu hành, mô tô sidecar).
- Lưỡng lự nhất: **img_019** (xe diễu hành trông như đồ chơi nhưng có biển số, người lái thật → `car` + `can_review`); **img_008, img_030** (bãi xe quá dày, khó chọn box riêng hay `ignore_region`).
- `validate_labels.py`: 0 lỗi, 1 cảnh báo (1 box cao 15 px ở img_025).

## 4. Pha 4–5: tôi lệch ở đâu
- **IAA của tôi** (theo `iaa_summary.md`): IoU TB với 4 bạn từ 0.869–0.895 (đạt ≥ 0.85), nhưng cùng lớp chỉ 83.8–97.9%. Chỉ cặp với Nam (2A202603186) đạt cả hai ngưỡng (47 cặp box, IoU 0.869, cùng lớp 97.9%).
- Có **266/942** ca trong `calibration_candidates.csv` có tên tôi: bỏ sót/thừa 113, sai thuộc tính 86, IoU thấp 37, sai lớp 17, `can_review` 13. Nhiều nhất ở img_008, img_039, img_005, img_023 (ảnh đông xe).
- **Lệch do guideline mơ hồ (cả nhóm cùng lệch):**
  - *Vật nhỏ/xa:* tôi vẽ thêm nhiều xe máy, ô tô nhỏ ở xa mà người khác bỏ qua. Guideline chưa nói rõ bao nhiêu px thì vẽ box riêng hay gộp `ignore_region`.
  - *occluded_level:* tôi thường chấm 1 khi các bạn chấm 0. v2.0 chỉ ghi "che ít / che nhiều", không có ngưỡng % nên mỗi người hiểu một kiểu.
  - *Xe lạ ở img_008:* tôi gán `bus`, bạn khác gán `truck` hoặc `tuktuk` (songthaew chưa có quy tắc).
- **Lệch do người gán chưa theo quy tắc:**
  - *Pickup:* tôi gán `truck` đúng định nghĩa v2.0; một bạn gán `car` (img_007, img_012, img_016, img_028).
  - *Người trên xe:* một số bạn gán `person` cho người lái (img_023, img_005), trái v2.0; tôi chỉ gán phương tiện.
  - *Phần của tôi cần xem lại:* box `car` cao 15 px ở img_025 (nên gộp `ignore_region`) và một số xe nhỏ tôi vẽ dư.
- **Quyết định calibration quan trọng nhất:** *bổ sung sau buổi họp* (đề xuất: chốt ngưỡng % cho `occluded_level` và ngưỡng vật nhỏ → `ignore_region`).

## 5. Dùng AI
- Dùng Claude Code để: đọc hướng dẫn, sửa `validate_labels.py` (MSHV có chữ), cập nhật `taxonomy.json` lên v2.0, kiểm tra file export COCO và đọc kết quả IAA.
- **Gán nhãn:** bản đầu do Claude nhìn ảnh và vẽ box trong CVAT theo guideline v2.0 (không dùng mô hình phát hiện tự động như YOLO). Sau đó tôi tự xem lại từng ảnh và sửa trong CVAT (img_005, img_008, img_009, img_016, img_019, img_023, img_025, img_030, img_034) rồi export bản nộp.
- Tôi ghi rõ điều này vì quy tắc lab không cho dùng AI để gán box; kết quả IAA của tôi nên được nhìn với lưu ý đó.
- Cách kiểm tra: xem lại ảnh trong CVAT, chạy `validate_labels.py`, đối chiếu số box theo lớp.

## 6. Bài học
- Phải chốt các ca "người trên xe" và "xe ngoài lớp" trước khi gán, tránh đổi guideline giữa chừng.
- Mỗi ngưỡng (mức che, vật nhỏ, đám đông) nên có ảnh ví dụ để mọi người đo giống nhau.
