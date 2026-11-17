# Guideline gán nhãn — v0.1 (cá nhân) — 2A202603327 - Đinh Đức Hiếu

> Bản cá nhân, viết độc lập ở Pha 1, dựa trên `guideline/guideline_v0_starter.md`.
> Không sửa lại sau khi họp nhóm; bản thống nhất viết ở `guideline_v1.0_agreed.md`.
>
> **Cách dùng khung này:** mỗi chỗ `TODO` là một câu v0 chưa trả lời. Xem 40 ảnh trong `data/images/`,
> rồi thay `TODO` bằng quyết định của bạn. Câu nào chưa chắc: ghi **chưa chốt, cần họp**.
> Xoá các dòng hướng dẫn (bắt đầu bằng `>`) trước khi nộp.

- Phiên bản: v0.1 (cá nhân)
- Ngày: 2026-11-16
- Người viết: 2A202603327 - Đinh Đức Hiếu
- Loại nhãn: bounding box trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện tham gia giao thông** (camera giao thông, xe tự lái).

- Sai nguy hiểm nhất: **bỏ sót người thật**. Gán tượng/ma-nơ-canh thành người cũng có hại (xe phanh vô cớ).
- Phân vân một vật **thật** → **thà gán thừa** + tick `can_review`. Vật **không thật** → không gán.
- "Phương tiện" = vật **thật, tự di chuyển được trên đường** (chạy, đỗ, hỏng đều tính). Đồ chơi, mô hình, tượng, hình in thì không tính.

## 2. Lớp và thuộc tính

### 2.1 Lớp

| Lớp | Định nghĩa | Ví dụ | Dễ nhầm với |
|---|---|---|---|
| `car` | Xe chở người ≤ 9 chỗ, thân liền khối: sedan, hatchback, wagon, SUV, MPV, taxi | img_012, img_023, img_024, img_040 | Pickup → `truck`; xe hộp mũi ngắn → `van` |
| `truck` | Xe có **cabin và thùng hàng tách riêng**: pickup (kể cả có canopy), xe tải, đầu kéo + rơ-moóc | img_001, img_007, img_036, img_037, img_002 | Pickup luôn là `truck`, phân lớp theo kết cấu, không theo công dụng |
| `van` | Xe **thân hộp một khối**, mũi ngắn, cỡ nhỏ hơn xe tải (Hiace, Transit) | img_005, img_016 | Minibus (img_009) không phải `van` |
| `motorcycle` | Xe hai bánh **có động cơ** (xe máy, tay ga, mô tô, xe điện); có sidecar thì box ôm cả sidecar | img_021, img_026, img_022 | Xe đạp, xích lô, tuk-tuk, xe ba bánh không phải `motorcycle` |
| `person` | **Người thật** thấy rõ đầu hoặc thân: đi bộ, đứng, ngồi, lái, ngồi sau xe, trong xe nhìn qua kính | img_014, img_034, img_026, img_004 | Tượng (img_017), ma-nơ-canh (img_033), billboard (img_018), linh vật (img_019) → không gán |
| `ignore_region` | Vùng chứa vật **thật** không gán được từng box: đám đông dày, vật ngoài lớp, vật nhòe không rõ biên | img_021, img_030, img_019 | Không dùng cho vật giả; không dùng thay cho vật rõ ràng |

**Vật ngoài lớp** (bus, minibus, tuk-tuk, xích lô, xe ba bánh, xe đạp): vẽ **`ignore_region`** (b). Đó là xe thật, nên không gán sẽ dạy sai mô hình, còn gán vào lớp gần nhất làm bẩn lớp. **Người** trên các xe này vẫn gán `person`. Vật không thật thì không gán (a).

### 2.2 Thuộc tính

**`occluded_level`**: % diện tích bị vật khác che, tính trên phần vật nằm trong ảnh:

| Mức | Nghĩa | Ngưỡng |
|---|---|---|
| `0` | Thấy rõ | < 10% |
| `1` | Che ít | 10 – < 40% |
| `2` | Che nhiều | 40 – < 80% |
| `3` | Che gần hết nhưng **vẫn nhận ra chắc chắn** | ≥ 80% |

- Cắt mép ảnh **không** tính vào % che (đã có `truncated`).
- Người trên xe bị xe che chân: **có tính** (thường mức 1). Hàng rào song thưa: thường mức 1. Kính trong suốt không tính.
- Không nhận ra lớp thì không phải mức 3: vẽ `ignore_region`, hoặc gán + `can_review`.

**`truncated`**:

| Giá trị | Khi nào |
|---|---|
| `no` | Nằm trọn trong ảnh, hoặc phần bị cắt < 10% |
| `yes` | Box chạm mép ảnh (≤ 2 px) **và** phần nằm ngoài ảnh ≥ 10% |

**`can_review`**: tick khi (1) phân vân `car`/`van`/`truck`; (2) không chắc vật thật hay giả (xe diễu hành, người hóa trang, phản chiếu); (3) nhòe/tối khiến biên có thể lệch > 10%; (4) occlusion/truncation sát ngưỡng; (5) vật cao 18–25 px; (6) phân vân box riêng hay `ignore_region`; (7) ca nằm trong "Chưa chốt".

Không cần thuộc tính mới: vật giả xử lý bằng cách không gán.

## 3. Quy tắc vẽ box

1. **Ôm sát:** chỉ ôm phần **nhìn thấy**, sai số ≤ 2 px. Xe: tính gương, cản, giá nóc, hàng nằm trong thùng; **không** tính vật dài nhô xa khỏi xe (ống sắt img_029).
2. **Vật bị che:** chỉ vẽ phần thấy được; bị che tách làm hai mảnh thì vẫn vẽ **một** box ôm cả hai.
3. **Cắt mép:** box dừng đúng mép ảnh, đặt `truncated` theo 2.2.
4. **Vật không thật** (phản chiếu, hình in, tượng, ma-nơ-canh, đồ chơi, mô hình): **không gán, không vẽ ignore**. Không chắc → gán + `can_review`.
5. **Người trên xe:** 1 box xe (chỉ ôm xe) + 1 box cho **mỗi** người. Box chồng nhau là bình thường.
6. **Xe chở xe/vật:** xe chở = `truck` (đầu kéo + rơ-moóc là 1 box); xe thật được chở = box `car` riêng; vật được chở ngoài lớp thì không gán (img_002, img_028).
7. **Xe chủ:** mui, táp-lô, gương của xe người chụp không gán; mọi thứ trong gương cũng không gán (img_020).
8. **Vật < 20 px:** không vẽ box riêng; cụm ≥ 3 vật nhỏ sát nhau thì vẽ 1 `ignore_region`.
9. **Đám đông:** gán riêng mọi vật tách được biên; **≥ 5 vật cùng lớp chồng nhau** không phân được biên thì vẽ `ignore_region`. Vật phía trước rõ ràng vẫn phải có box riêng.
10. **Nhòe, tối, ngược sáng:** nhận ra được lớp thì vẫn gán, box ôm phần thân rõ nhất, không ôm vệt nhòe; nhòe thành vệt không rõ → `ignore_region` (img_014).
11. **Xe đỗ, hỏng, không người, xe trưng bày thật:** gán như xe đang chạy.

## 4. Ca khó

| Ảnh | Tình huống | Quyết định | Lý do |
|---|---|---|---|
| img_002 | Đầu kéo chở ~8 ô tô | 1 box `truck`; mỗi ô tô = `car`, occluded theo khung che | Xe được chở vẫn là xe thật |
| img_037 | Pickup có canopy kín | `truck` | Kết cấu cabin + thùng tách |
| img_007 | Hilux + Isuzu trong kho tối | Cả hai `truck`, `can_review` nếu khó thấy biên | Pickup luôn là `truck`; xe đỗ vẫn gán |
| img_005 | Van bị đám xe máy che phần lớn | `van`, occluded `2` | Thân hộp một khối, che ~50–70% |
| img_009 | Minibus | `ignore_region` | Ngoài lớp, không xếp vào `van` |
| img_004, img_016, img_040 | Tuk-tuk có người bên trong | Xe → `ignore_region`; người → `person` | Xe ngoài lớp, người vẫn thật |
| img_023 | Xích lô | Xe → `ignore_region`; người đạp → `person` | Không có động cơ |
| img_029 | Xe máy kéo ống dài + xe đạp | `motorcycle` không ôm ống; xe đạp → `ignore_region` | Ống làm box phình, IoU lệch |
| img_022 | Mô tô có sidecar | 1 box `motorcycle` ôm cả sidecar | Sidecar gắn liền xe |
| img_017 | Tượng người cưỡi ngựa | Không gán | Không phải người thật |
| img_027 | Tượng đá hình pickup | Không gán | Không di chuyển được |
| img_028 | Pickup chở tượng ngựa | `truck`; ngựa không gán | Xe thật, vật chở là đồ giả |
| img_038 | Tủ kính xe đồ chơi | Không gán (0 box) | Mẫu negative |
| img_018 | Người trên billboard; xe sau rào | Billboard không gán; xe → `car`, occluded `1` | Hình in; rào thưa |
| img_020 | Gương chiếu hậu xe chủ | Không gán | Xe chủ + phản chiếu |
| img_033 | Ma-nơ-canh, phản chiếu tủ kính | Không gán; phần xe bên phải: gán + `can_review` nếu là xe thật | Dễ nhầm người nhất |
| img_019 | Xe diễu hành Dangermouse, đám đông sau rào | Xe → `car` + `can_review`; người lái → `person`; linh vật không gán; đám đông sau → `ignore_region` | Xe thật chạy trên đường |
| img_021, img_030 | Xe máy cực dày | Box riêng cho xe phía trước; mảng sau → `ignore_region` | Quy tắc 3.9 |
| img_014 | Xe máy nhòe chuyển động | Người rõ → `person`; vệt nhòe → `ignore_region` | Không đo được biên |
| img_015 | Đêm mưa, xe bị cắt mép | `car`, `truncated = yes`; xe xa nhòe → `can_review` | Cắt mép không tính vào che |
| img_040 | Người cận cảnh, chỉ thấy gương xe máy | `person` truncated; xe máy không gán | Không đủ thân xe |
| img_032 | Hàng người dài ở trạm bus | Người tách được ≥ 20 px → `person`; đoạn chồng ≥ 5 → `ignore_region`; bus → `ignore_region` | Đám đông + ngoài lớp |
| img_010 | Người trên xe tay ga, xe đẩy hàng | `person` + `motorcycle`; xe đẩy không gán | Xe đẩy không phải phương tiện |

### Chưa chốt, cần họp

| Ảnh | Câu hỏi | Phương án |
|---|---|---|
| img_009, img_016, img_029 | Thêm lớp cho bus, xe đạp, tuk-tuk? | Giữ `ignore_region` / thêm `bus`, `bicycle` / gộp `other_vehicle` |
| img_002 | Xe được chở gán `car` hay không? | `car` / `ignore_region` / không gán |
| img_008 | Songthaew (pickup chở khách có mái) | `truck` / `ignore_region` |
| img_011 | Predator là người hóa trang hay tượng? Mô tô độ có chạy được? | `person` hoặc không gán; `motorcycle` + `can_review` hoặc không gán |
| img_019 | Xe giả tưởng: `car` hay ngoài lớp? | `car` + `can_review` / `ignore_region` |
| chung | Người trong ô tô nhìn qua kính | Gán khi thấy rõ đầu, ≥ 20 px / không gán |

## 5. QC và phiên bản

- Mọi thay đổi làm đổi nhãn đều tăng phiên bản guideline.
- Ca mơ hồ: tick `can_review`, ghi chú, đem ra calibration; không tự đoán.
- Ngưỡng QC (theo `taxonomy.json`): IoU box ≥ 0.85 (cao ≥ 50 px), ≥ 0.7 (20–50 px); IAA: IoU trung bình ≥ 0.85, cùng lớp ≥ 95%; ảnh lỗi ≤ 2% mới Release.
- Ai quyết ca mơ hồ: nhóm **biểu quyết đa số** trong calibration; hòa thì **người phụ trách QC** quyết; ghi vào mục 4 và tăng phiên bản.

### Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v0.1 | Định nghĩa 5 lớp đo được (pickup → `truck`, sidecar → `motorcycle`); vật ngoài lớp → `ignore_region`, vật giả → không gán; ngưỡng occlusion 10/40/80%, truncated ≥ 10%; 11 quy tắc vẽ box; danh sách tình huống `can_review` | img_002, img_009, img_014, img_017, img_019, img_020, img_021, img_029, img_033, img_037, img_038 |
