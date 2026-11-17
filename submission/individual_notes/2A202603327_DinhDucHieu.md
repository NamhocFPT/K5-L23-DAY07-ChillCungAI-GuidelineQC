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
- *Bổ sung sau khi nhóm chạy IAA và họp calibration.*

## 5. Dùng AI
- Dùng Claude Code để đọc hướng dẫn, sửa script/taxonomy và kiểm tra file export.
- Claude vẽ box trong CVAT theo guideline (không dùng mô hình tự động như YOLO); sau đó tôi tự xem lại và chỉnh trong CVAT rồi export bản cuối.

## 6. Bài học
- Phải chốt các ca "người trên xe" và "xe ngoài lớp" trước khi gán, tránh đổi guideline giữa chừng.
- Mỗi ngưỡng (mức che, vật nhỏ, đám đông) nên có ảnh ví dụ để mọi người đo giống nhau.
