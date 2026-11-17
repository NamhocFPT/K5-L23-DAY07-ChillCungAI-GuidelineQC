# Ghi chú cá nhân — 2A202603158 - NguyenQuocThang

## 1. Pha 1: ba ca khó tôi tìm ra trong 40 ảnh
| Ảnh | v0 không trả lời được gì | Quyết định ban đầu của tôi |
|---|---|---|
| img_004, img_016, img_023 | v0 không có lớp cho tuk-tuk, auto-rickshaw, xích lô; cũng không nói người lái/ngồi trong xe có gán `person` không. | Đề xuất lớp mới `three_wheeler`; tài xế và khách nhìn rõ thì gán `person`. |
| img_009, img_037, img_003 | v0 không có `bus` và `bicycle`, nên minibus và xe đạp không có chỗ xếp. | Đề xuất thêm `bus` (cửa gập, ≥ 5 cửa sổ hông) và `bicycle`. |
| img_017, img_027, img_033 | v0 không nói tượng, ma-nơ-canh, hình in, phản chiếu có gán không, và có dùng `ignore_region` cho chúng không. | Không gán gì, không dùng `ignore_region`; các ảnh này là ảnh 0 box. |

## 2. Pha 2: chỗ tôi bất đồng với nhóm
Nhóm chốt theo `guideline_v2.0_agreed.md`.
- **Lớp mới:** nhóm nhận `bus`, `bicycle` và lớp xe ba bánh, đặt tên `tuktuk` thay cho `three_wheeler` của tôi. Tôi đồng ý.
- **Người trên xe:** bản của tôi gán `person` cho người lái/ngồi trên xe. v2.0 chỉ gán `person` cho người đi bộ hoặc đứng trên đường, người trên xe chỉ gán box phương tiện. Tôi đồng ý vì mục tiêu là xe tự lái: người trên xe di chuyển cùng xe, box phương tiện đã đủ.
- **Vật giả:** v2.0 giống bản của tôi: tượng, poster, đồ chơi, phản chiếu không gán.
- **Ngưỡng đo được:** tôi chưa đồng ý hẳn. Bản của tôi có ngưỡng số: `occluded_level` 0/1/2/3 là ≤ 10% / ≤ 40% / ≤ 70% / > 70%; `truncated = yes` khi ≥ 5% vật ra ngoài khung; vật cao < 20 px hoặc ≥ 5 vật che nhau > 50% thì dùng `ignore_region`. v2.0 chỉ ghi "che ít / che nhiều", "quá nhỏ", nên hai người dễ chọn khác mức.
- **Chưa chốt:** v2.0 chưa nói rõ ô tô trên xe chở ô tô (img_002), songthaew (img_008), vật nhoè chuyển động (img_014, img_015).

## 3. Pha 3: ảnh tôi đã gán
- Số ảnh được giao: 24. Số box đã vẽ: 178 (car 45, motorcycle 41, person 84, truck 8). Số lần tôi tick `can_review`: 0.
- Box do YOLO11m gán nháp (xem mục 5). File tôi nộp là kết quả đó, chưa sửa tay, và Task CVAT lúc gán dùng bộ lớp v0. Vì vậy file chưa có `van`, `bus`, `bicycle`, `tuktuk`, và nhiều box `person` là người đang ngồi trên xe máy, trái với v2.0.
- Ca tôi lưỡng lự nhất: các ảnh có tuk-tuk và người trên xe máy (img_004, img_016, img_023, img_040). YOLO gán tuk-tuk thành `car` hoặc `motorcycle`, và gán `person` cho cả tài xế, nên nhãn theo v2.0 ở các ảnh này khác nhiều so với nhãn nháp.

## 4. Pha 4–5: tôi lệch ở đâu
- Chưa có `calibration_candidates.csv` (nhóm chưa chạy `qc_compare.py`). Điền sau Pha 4.
- Dự đoán: tôi sẽ lệch nhiều ở `person` (người trên xe) và các lớp mới. Nguyên nhân là tôi không sửa nhãn nháp theo v2.0, không phải do guideline mơ hồ.

## 5. Dùng AI
- **Claude (Claude Code):** soạn nháp guideline cá nhân sau khi tôi xem 40 ảnh; sửa JSON nhãn CVAT (`type: rectangle`, `values` của checkbox); sửa regex trong `validate_labels.py` để nhận MSHV có chữ; hướng dẫn dùng CVAT và `cvat-cli`; soạn nháp ghi chú này. Tôi kiểm tra bằng cách đọc lại và chạy `validate_labels.py --all`.
- **YOLO11m (Ultralytics):** gán nháp 24 ảnh bằng function `detector/yolo` của repo `cvat-models` qua `cvat-cli task auto-annotate`, `conf-threshold` 0.35. Ra 178 box; chỉ giữ các lớp trùng tên trong Project (`person`, `car`, `truck`, `motorcycle`).
- **Cách kiểm tra:** `validate_labels.py --all` báo 0 lỗi, nhưng script chỉ kiểm định dạng, không biết box đúng lớp hay ôm sát. Tôi chưa sửa tay nhãn nháp, nên nhãn chưa phản ánh đúng v2.0. RULES.md dòng 18 không cho dùng mô hình gán box ở Day 07; tôi khai rõ ở đây.

## 6. Bài học
- Guideline phải có ngưỡng đo được (% che, % ra ngoài khung, số px) ngay trong bản thống nhất. Chỉ ghi "che ít / che nhiều" thì mỗi người chọn một mức.
- Chốt bộ lớp trước khi gán, và cập nhật `taxonomy.json` + bộ nhãn CVAT cùng lúc. Nhóm đổi v1.0 → v2.0 giữa Pha 3 nên Task của tôi còn dùng bộ lớp cũ.
- Pre-label giúp nhanh, nhưng nhãn nháp phải được sửa theo guideline. Nếu không, nhãn chỉ là ý kiến của mô hình.
