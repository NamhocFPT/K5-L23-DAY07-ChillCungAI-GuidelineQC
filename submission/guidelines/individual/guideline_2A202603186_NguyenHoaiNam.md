# Guideline gán nhãn — v0.1 (cá nhân) — 2A202603186 - Nguyễn Hoài Nam | Nhóm: (chưa điền)

> Bản cá nhân, viết từ `guideline_v0_starter.md` sau khi xem đủ 40 ảnh trong `data/images/`.
> Các mục đánh dấu **Chưa chốt** cần thảo luận trong nhóm ở Pha 2.

- Phiên bản: v0.1 (cá nhân)
- Ngày: 2026-11-16
- Người viết / duyệt: Nguyễn Hoài Nam

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện** trên ảnh đường phố châu Á (xe máy, xe tải, xe con, xe van).
Sai nguy hiểm nhất là **bỏ sót người và xe máy**, vì đây là đối tượng chính của camera giao thông và xe tự lái.
Sai thứ hai là **gán nhầm tượng, ma-nơ-canh, đồ chơi, hình in, phản chiếu thành xe hoặc người** (false positive), vì mô hình sẽ học sai.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con, sedan, SUV, taxi, station wagon, có bánh xe thật | sedan đen (023), taxi xanh (040) | pickup (→ `truck`); xe đồ chơi, mô hình (038) |
| `truck` | Xe có thùng hở, thùng bạt, thùng kín lớn, pickup, xe chở xe | xe chở xe (002), Isuzu thùng kín (007), pickup chở hàng (012) | bán tải làm mộ (027); tượng ngựa trên thùng xe (028) |
| `van` | Xe thùng kín cỡ nhỏ, minivan chở khách | minivan bạc (003, 005) | xe tải nhỏ thùng bạt (036 → `truck`) |
| `motorcycle` | Xe máy, xe tay ga, mô tô, kể cả xe có thùng bên | xe máy chở 3 người (005) | xe máy làm bằng kim loại trong tượng (011) |
| `person` | Người thật, thấy được ít nhất đầu và thân | người đi bộ (014), người lái taxi (040) | ma-nơ-canh (033); tượng người trên ngựa (017); người in trên billboard (018) |
| `ignore_region` | Vùng đông vật cùng lớp mà không tách được từng vật | cụm xe máy đậu (030), đám người xem (019) | vật riêng lẻ nhìn được thì gán box thường |

**Vật không thuộc lớp nào (không gán box, không ignore):**
- Xe không có lớp trong v0: xe buýt, xe đạp, xe ba bánh (tuk-tuk, auto-rickshaw), xích lô, xe đẩy tay. Không gán box, tick `can_review`. Người trên các xe này vẫn gán `person`.
- Vật không thật: tượng, mô hình, đồ chơi, xe trang trí, tượng đặt trên xe (011, 017, 027, 028, 038).
- Hình in, billboard, tranh tường, phản chiếu trong gương (018, 020, 026).
- Xe của chính người quay (xe chủ): phần thân xe thấy trong gương (020).
- Hàng hóa, thùng và bao đồ chở trên xe: không gán riêng.

### Thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa và ngưỡng |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | `0`: bị che dưới 10%. `1`: che 10–40%. `2`: che 40–70%. `3`: che trên 70% nhưng vẫn nhận ra lớp. Che trên 70% mà không nhận ra lớp thì không gán box. |
| `truncated` | `no`, `yes` | `yes` khi vật chạm mép ảnh và có phần nằm ngoài ảnh (cạnh box cách mép ảnh dưới 2 px). Box vẽ theo phần nhìn thấy. |
| `can_review` | tick / không | Tick khi: lớp không chắc; mức che nằm sát ngưỡng 10%, 40% hoặc 70%; vật cao dưới 20 px; ảnh mờ hoặc chuyển động; thuộc mục **Chưa chốt**. |

## 3. Quy tắc vẽ box

1. Ô tô, xe máy, xe tải, van, người: vẽ box ôm sát phần nhìn thấy. Vật bị che thì box chỉ phần thấy, không đoán phần khuất.
2. Người ngồi trên xe máy: mỗi người một box `person`, xe máy một box `motorcycle` riêng.
3. Người đứng cạnh xe mà không ngồi trên xe: gán `person` riêng, không gộp với xe.
4. Xe nằm trên xe khác (xe chở xe, xe trên mui): gán cả xe trên và xe dưới nếu nhận ra đều là xe thật. Xe trên bị che từ 40% trở lên thì `occluded_level` là 2 hoặc 3.
5. Xe có hàng chở: box bao xe, không bao hàng.
6. Vật cắt mép ảnh: vẽ box đến mép ảnh, `truncated = yes`.
7. Vật xa, nhỏ, cao dưới 20 px: gán nếu nhận ra lớp, tick `can_review`. Không nhận ra lớp thì bỏ.
8. Cụm đông: nếu có từ 5 vật cùng lớp trở lên chồng nhau từ 60% trở lên và không tách được, khoanh cả cụm bằng `ignore_region`. Vật riêng lẻ ở mép cụm vẫn gán box thường.
9. Vật không thật, phản chiếu, hình in: không gán và không ignore.
10. Phân biệt van và truck: thùng kín cỡ nhỏ hoặc minivan chở khách là `van`. Thùng kín lớn, thùng bạt, pickup có mui là `truck`. Không chắc thì tick `can_review`.
11. Pickup (bán tải) là `truck`.

## 4. Ca khó

Ghi theo 40 ảnh. Ảnh không có trong bảng chỉ có các lớp rõ ràng (`car`, `motorcycle`, `person`), gán theo mục 2 và 3.

| Ảnh | Tình huống | Quyết định | Lý do |
|---|---|---|---|
| img_001 | Pickup cũ chở đồ; người ngồi bàn trước xe | Pickup → `truck`; người → `person` (occluded 1–2) | Quy tắc 11 |
| img_002 | Xe chở xe nhiều xe con | Cabin và thùng → `truck`; xe trên thùng → `car` (occluded 1–2) | Quy tắc 4 |
| img_003 | Minivan bạc phía sau; xe máy đông; người gánh đồ | `van` (occluded 1); xe máy và người gán (occluded 1–2); gánh không gán | Minivan là `van` |
| img_004 | Hai tuk-tuk xanh | Không gán; tick `can_review` | Tuk-tuk không có lớp (Chưa chốt A) |
| img_005 | Minivan phía sau; xe máy chở 3 người | `van`; mỗi người `person`; xe máy `motorcycle` (occluded 2) | Quy tắc 2 |
| img_006 | Taxi bên trái; xe đạp; xe máy đông | Taxi → `car`; xe đạp không gán; người → `person` | Xe đạp không có lớp |
| img_007 | Isuzu thùng kín lớn; pickup trắng | Isuzu → `truck`; pickup → `truck`; tick `can_review` | Quy tắc 10 |
| img_008 | Đêm; xe máy đậu dày; người đi bộ | `motorcycle` occluded 2–3; cụm xa → `ignore_region` | Quy tắc 8 |
| img_009 | Xe khách nhỏ; xe máy xa | Xe khách không gán; xe máy xa gán nếu nhận được, tick `can_review` | Xe khách không có lớp |
| img_010 | Xe đẩy bán chuối; người mua; xe máy | Xe đẩy không gán; người → `person`; xe máy occluded 1–2 | Xe đẩy không có lớp |
| img_011 | Tượng quái vật; tượng xe máy kim loại; người xa | Tượng không gán; người xa → `person`, tick `can_review` nếu dưới 20 px | Vật không thật |
| img_012 | Pickup chở hàng; sedan bạc bên trái | Pickup → `truck`; sedan → `car`; hàng không gán | Quy tắc 5 |
| img_013 | Đường đêm; nhiều xe xa nhỏ | Xe và người nhận được thì gán; còn lại tick `can_review` | Vật cao dưới 20 px |
| img_014 | Hai người băng đường; xe máy mờ phía sau | `person`; xe máy mờ `occluded_level = 3`, tick `can_review` | Chuyển động mờ |
| img_015 | Xe đậu; xe đỏ và xe bạc bị cắt mép | `car`; `truncated = yes` cho hai xe cắt mép | Quy tắc 6 |
| img_016 | Hai tuk-tuk; xe máy; minivan trắng; pickup phía sau | Tuk-tuk không gán; minivan → `van`; pickup → `truck` (occluded 2); xe máy và người gán; người áo đỏ bị cắt mép phải → `truncated = yes` | Nhiều lớp chồng nhau |
| img_017 | Tượng kỵ sĩ đội nón chóp | Không gán | Vật không thật |
| img_018 | Xe đậu sau hàng rào; người trên billboard | Xe thật gán (occluded 1–2 do hàng rào); người trên billboard không gán | Hình in |
| img_019 | Xe trang trí có tài xế; đám đông người xem | Xe trang trí không gán; tài xế → `person` (occluded 2); đám đông dày → `ignore_region`; người xem rõ → `person` | Xe không thật; quy tắc 8 |
| img_020 | Gương chiếu xe phía sau; thân xe của mình | Không gán | Phản chiếu và xe chủ (quy tắc 9) |
| img_021 | Kẹt xe giờ cao điểm; hàng trăm xe máy | Người và xe máy phía trước gán (occluded 1–2); phần sau → `ignore_region` | Quy tắc 8 |
| img_022 | Xe máy có thùng bên, ảnh đen trắng | `motorcycle` bao cả thùng bên, tick `can_review`; người lái → `person` (occluded 2) | Ảnh cũ, lớp có thể lẫn |
| img_023 | Sedan đen; xe đạp kéo chở khách | Sedan → `car`; xe đạp kéo không gán, tick `can_review`; người lái → `person` | Xe đạp kéo không có lớp |
| img_024 | Station wagon đen có xe vàng trên mui | Station wagon → `car`; xe vàng trên mui → `car` (occluded 1), tick `can_review` | Chưa rõ xe thật hay mô hình (Chưa chốt E) |
| img_025 | Taxi vàng xanh; xe máy kéo hàng; người đi bộ | Taxi → `car`; người → `person`; xe kéo không gán | Quy tắc chung |
| img_026 | Hai người trên xe máy đỏ; tranh tường | Người → `person` (occluded 1); xe máy → `motorcycle`, `truncated = yes` (bánh cắt mép dưới); tranh tường không gán | Quy tắc 6, 9 |
| img_027 | Bán tải đen làm mộ | Không gán | Vật không thật |
| img_028 | Bán tải cũ chở tượng ngựa; SUV đỏ phía sau | Bán tải → `truck`; tượng ngựa không gán; SUV → `car` (occluded 2) | Tượng không phải người |
| img_029 | Người chở ống dài trên xe máy; xe đạp | Hai người → `person`; xe máy → `motorcycle`; xe đạp không gán; ống không gán | Quy tắc 2, 5 |
| img_030 | Hàng xe máy đậu dày đặc | Xe máy đỏ tiền cảnh gán (occluded 1); phần còn lại → `ignore_region` | Quy tắc 8 |
| img_031 | Đêm mưa; SUV trắng; sedan; xe buýt | SUV và sedan → `car`; xe buýt không gán, tick `can_review`; người → `person` | Xe buýt không có lớp (Chưa chốt B) |
| img_032 | Đám sinh viên chờ xe buýt; xe xanh bị cắt mép trái | Người rõ → `person` theo mức che; đám xa → `ignore_region`; xe xanh `truncated = yes`; nón giao thông không gán | Cụm người đông |
| img_033 | Ma-nơ-canh trong kính; gương của minivan | Ma-nơ-canh không gán; chỉ thấy gương minivan thì không gán, tick `can_review` | Quy tắc 9 |
| img_034 | Người băng đường; sedan đen; xe máy cắt mép phải | Người → `person`; sedan → `car` (occluded 1); xe máy → `motorcycle`, `truncated = yes` | Quy tắc 6 |
| img_035 | Xe ba bánh chở đồ cao; xe máy nhỏ; sedan trắng | Ba bánh không gán; xe máy và người gán; sedan → `car` | Xe ba bánh không có lớp (Chưa chốt A) |
| img_036 | Xe tải nhỏ thùng bạt xanh | `truck`, tick `can_review` | Thùng bạt không phải van (quy tắc 10) |
| img_037 | Pickup có mui kín (xe dịch vụ); xe buýt xanh; người mang ba lô | Pickup mui kín → `truck`, tick `can_review`; xe buýt không gán; người → `person` | Quy tắc 10; Chưa chốt B |
| img_038 | Kệ đồ chơi, mô hình xe, vịt cao su | Không gán | Đồ chơi |
| img_039 | Đường nhiều xe máy; người đi bộ; xe đạp | Xe máy và người gán; xe đạp không gán; xe xa tick `can_review` | Quy tắc chung |
| img_040 | Taxi xanh; người đội mũ đen che taxi; tuk-tuk; xe buýt vàng | Taxi → `car` (occluded 1); người → `person` (occluded 1); tuk-tuk và xe buýt không gán, tick `can_review` | Chưa chốt A và B |

**Chưa chốt (cần thảo luận trong nhóm):**
- **A.** Tuk-tuk, auto-rickshaw và xích lô: thêm lớp mới (ví dụ `tuktuk`) hay gộp vào `car`/`motorcycle`? (004, 016, 023, 035, 040)
- **B.** Xe buýt và xe khách nhỏ: thêm lớp `bus` hay gán `truck`? (009, 031, 037, 040)
- **C.** Xe đạp và xe đạp kéo: có cần lớp `bicycle` không? (003, 006, 023, 029, 039)
- **D.** Ranh giới `van` và `truck` với xe tải thùng bạt và pickup mui kín (007, 036, 037).
- **E.** Xe trên mui (024) và xe chở xe (002): gán cả hai hay chỉ xe trên cùng?
- **F.** Ngưỡng cụm đông cho `ignore_region`: 5 vật, chồng từ 60% (021, 030, 032).

## 5. QC và phiên bản

- Ca mơ hồ: tick `can_review` và ghi vào bảng ca khó, không tự quyết ở bản cá nhân.
- Quyết định các mục **Chưa chốt** thuộc về nhóm, ở Pha 2 hoặc Pha 5.
- Ngưỡng QC áp dụng khi kiểm: occluded theo mục 2; truncated theo mục 2; vật thiếu hoặc thừa theo quy tắc 7 và 9.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 → v0.1 (cá nhân) | Viết lại cả 5 phần. Thêm ngưỡng `occluded_level` (10/40/70%). Định nghĩa `truncated`. Thêm quy tắc người trên xe, cụm đông, vật không thật, xe không có lớp, phân biệt van và truck. | Ca khó trong 40 ảnh, đặc biệt 004, 019, 021, 030, 033 |
