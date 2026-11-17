# Báo cáo QC — Nhóm ChillCungAI

## 1. Lô

| Mục | Giá trị |
|---|---|
| Tên lô | day07-street-40 |
| Số ảnh | 40 |
| Thành viên (MSHV) | 2A202603186, 2A202603116, 2A202603327, 2A202603158, 2A202603255 |
| Số lần gán mỗi ảnh (K) | 3 |
| Phiên bản guideline lúc gán | v1.0 / v2.0 |
| Phiên bản guideline hiện tại | v1.1 |

## 2. Mẫu QC

- Cỡ mẫu: **40 ảnh** (37 ảnh ca khó + 3 ảnh chọn ngẫu nhiên, seed 42).
- Cách chọn: `scripts/qc_sample.py --n 20 --seed 42`; do lô chỉ có 40 ảnh và có 37 ảnh bị cờ, mẫu thực tế bao phủ toàn bộ lô.
- Người kiểm: **Cả nhóm ChillCungAI**; rà soát theo biên bản họp trên Canva ngày 2026-11-17.

## 3. Số đo

### 3.1 IAA (trước calibration)

| Chỉ số | Giá trị | Mẫu số | Ngưỡng | Đạt? |
|---|---:|---:|---:|---|
| IoU trung bình (cặp box ghép được) | 0.872 | 488 cặp | ≥ 0.85 | Có |
| Đồng thuận lớp | 95.1% | 488 cặp | ≥ 95% | Có |
| Box không ghép được | 167 bỏ sót / 152 thừa | các cặp so sánh | — | — |
| Đồng thuận `occluded_level` | 67.8% | 488 cặp | — | — |
| Đồng thuận `truncated` | 87.5% | 488 cặp | — | — |

### 3.2 Tỉ lệ lỗi mẫu QC (sau calibration)

Theo biên bản họp Canva, cả 40 ảnh đã được rà soát sau calibration và các chỉnh sửa đã được áp dụng vào nhãn cuối. `qc_review.csv` ghi nhận 0/322 đối tượng lỗi còn lại trong mẫu.

## 4. Lỗi

Không ghi nhận lỗi còn lại sau calibration trong mẫu 40 ảnh: 0/322 đối tượng và 0/40 ảnh. Các bất đồng trước calibration gồm 319 bỏ sót/thừa, 122 IoU thấp, 199 sai thuộc tính, 89 cờ `can_review` và 24 sai lớp.

## 5. Calibration

- Số ảnh bị cờ: **37 / 40**.
- Số quyết định đã ghi: **37**, trong `calibration_decisions.csv`.
- Guideline v1.1 đã bổ sung và làm rõ: người trên phương tiện, phân lớp pickup/van/bus/tuktuk, vật nhỏ dưới 20 px, `ignore_region`, `occluded_level`, `truncated` và box phần nhìn thấy.
- Nhãn cuối đã gộp: `labels_calibrated.json` gồm 40 ảnh và 322 annotation.

## 6. Script kiểm được gì, không kiểm được gì

- Script kiểm được: lớp và thuộc tính hợp lệ, box trong ảnh, box quá nhỏ, đủ ảnh được giao, độ lệch giữa người gán, IAA và các số đo QC.
- Script không kiểm được: box có ôm sát vật thật không, lớp có đúng thực tế không, hoặc cả nhóm có cùng bỏ sót/cùng hiểu sai hay không. Các điểm này phải được xác nhận bằng mắt trong `qc_review.csv`.

## 7. Quyết định

**RELEASE — theo kết quả rà soát của cả nhóm.**

IAA đạt ngưỡng, calibration đã có 37 quyết định cho 37 ảnh bị cờ, nhãn cuối đã được tạo và biên bản Canva xác nhận rà soát toàn bộ 40 ảnh. Tỉ lệ lỗi theo đối tượng là 0/322 = 0.0%; tỉ lệ ảnh có lỗi là 0/40 = 0.0%, thấp hơn ngưỡng Release 2%.
