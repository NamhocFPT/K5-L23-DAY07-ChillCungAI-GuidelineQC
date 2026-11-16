# Guideline gán nhãn — v0.1 (bản cá nhân) — 2A202603158 - Nguyen Quoc Thang

- Phiên bản: v0.1-cá nhân (sửa từ v0 của giảng viên)
- Ngày: 16/11/2026
- Người viết: 2A202603158 - Nguyen Quoc Thang · Người duyệt: cả nhóm (Pha 2)
- Loại nhãn: bounding box trên ảnh đường phố · Công cụ: CVAT, xuất COCO 1.0

> Đây là bản đầu tiên, viết độc lập sau khi xem hết 40 ảnh. Những chỗ ghi **[chưa chốt, cần họp]**
> là chỗ tôi đã chọn một cách nhưng chưa chắc; nhóm cần quyết ở Pha 2.

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **phát hiện người và phương tiện đang có mặt thật trên đường**, cho camera
giao thông và xe tự lái ở đô thị châu Á (nhiều xe máy, xe ba bánh, giao thông dày đặc).

Câu hỏi gốc khi phân vân: **"Vật này có thể di chuyển ra đường, hoặc chiếm chỗ trên đường, mà xe phải tránh không?"**
Có → gán. Không → không gán.

Sai ở đâu thì nguy hiểm:

| Loại sai | Hậu quả | Mức nghiêm trọng |
|---|---|---|
| Bỏ sót người / xe máy có thật | Xe không thấy, có thể va chạm | **Cao nhất** |
| Gán tượng, ma-nơ-canh, hình in, phản chiếu là người/xe | Mô hình phanh gấp vô cớ, học sai | Cao |
| Box sai kích thước (thiếu đuôi xe, thiếu chân người) | Ước lượng khoảng cách sai | Trung bình |
| Sai lớp giữa các loại xe (car ↔ truck ↔ van) | Dự đoán chuyển động sai | Trung bình |
| Sai thuộc tính | Lọc / thống kê sai, không ảnh hưởng trực tiếp mô hình | Thấp |

---

## 2. Lớp và thuộc tính

### 2.1. Lớp

Mỗi vật thuộc **đúng một** lớp. Ba lớp `bus`, `bicycle`, `three_wheeler` là **lớp tôi đề xuất thêm** vì 40 ảnh có rất
nhiều vật này mà v0 không có chỗ để xếp **[chưa chốt, cần họp]**.

Phép thử lớp/thuộc tính: *"Mô hình có phải đoán ra điều này không?"* Xe buýt, xe đạp, tuk-tuk có hình dạng và cách
di chuyển khác hẳn → mô hình phải phân biệt → là **lớp**, không phải thuộc tính.

| Lớp | Định nghĩa (một câu) | Ví dụ (ảnh) | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Xe 4 bánh chở người, ≤ 9 chỗ, không có thùng hàng tách rời: sedan, hatchback, SUV, taxi, xe cổ, xe diễu hành chạy thật. | img_015 (xe đỏ, xe bạc), img_012 (Mazda SUV), img_023 (sedan đen), img_040 (taxi xanh), img_019 (xe Dangermouse có biển số, người lái) | Pickup → `truck`. Xe 7–16 chỗ thân hộp → `van`. Ô tô đồ chơi, mô hình, tượng → **không gán** (img_038, img_024 xe vàng trên nóc, img_027). |
| `truck` | Xe có **khoang lái tách riêng với phần chở hàng**: xe tải, xe tải nhẹ, pickup (kể cả có nắp thùng), xe chở ô tô (cả đầu kéo + rơ-moóc). | img_001 (Suzuki thùng lửng), img_002 (xe chở ô tô), img_007 (Isuzu ELF, Hilux có thùng kín), img_012, img_028, img_036 (CMC Veryca thùng bạt), img_037 (pickup có nắp thùng) | Van thân liền một khối → `van`. Pickup chở khách kiểu songthaew vẫn là `truck` (theo hình dạng) **[chưa chốt, cần họp]**. |
| `van` | Xe 4 bánh **thân liền một khối hộp** (khoang lái và khoang sau không tách), đầu ngắn, cửa trượt hông, thường 7–16 chỗ. | img_005 (van bạc phía sau), img_016 (Toyota HiAce trắng), img_011 (van trắng sọc xanh sau tượng) | Minibus có cửa gập kiểu buýt, ≥ 5 ô cửa sổ hông → `bus` (img_009). Pickup → `truck`. |
| `bus` *(đề xuất)* | Xe chở khách thân dài, cửa lên xuống kiểu buýt (gập/đôi), **≥ 5 ô cửa sổ hông** hoặc chiều dài ước tính ≥ 7 m; gồm cả minibus 25–30 chỗ. | img_009 (minibus Hyundai County), img_031 (buýt đèn LED số 62), img_037 (buýt GO xanh), img_040 (buýt phía sau) | HiAce / Transit → `van`. Nhà chờ xe buýt (img_032) không phải xe. |
| `motorcycle` | Xe 2 bánh có động cơ: xe máy, xe tay ga, mô tô; **kể cả xe có thùng/sidecar gắn bên hông**. | img_006, img_008, img_021, img_026, img_022 (mô tô sidecar), img_025 (xe máy kéo thùng bán hàng) | Xe đạp → `bicycle`. Xe ba bánh khung liền (tuk-tuk) → `three_wheeler`. Tác phẩm điêu khắc hình xe máy → không gán (img_011). |
| `bicycle` *(đề xuất)* | Xe 2 bánh **không động cơ**, kể cả xe đạp chở hàng rong. | img_003 (xe đạp dựng), img_006 (người đạp xe bên phải), img_010 (xe đạp chở hoa quả), img_029 (xe đạp chở ống) | Xe đạp điện có hình dáng xe tay ga → `motorcycle`. Xích lô (3 bánh) → `three_wheeler`. |
| `three_wheeler` *(đề xuất)* | Xe **3 bánh khung liền** chở người hoặc hàng, có hoặc không có động cơ: tuk-tuk, auto-rickshaw, xích lô, xe ba gác. | img_004 (2 tuk-tuk), img_016 (tuk-tuk xanh dương, xanh lá), img_035 (auto-rickshaw vàng chở đầy hàng), img_023 (xích lô), img_040 (tuk-tuk vàng) | Xe máy gắn sidecar tháo rời được → `motorcycle` (img_022, img_025). |
| `person` | Người thật, đang đi bộ, đứng, ngồi, hoặc ngồi trên xe **hở** (xe máy, xe đạp, tuk-tuk, xích lô, xe mui trần). | img_014 (2 người qua đường), img_005 (người trên xe máy), img_004 (tài xế tuk-tuk), img_019 (người lái xe hở mui), img_022 (người trong sidecar) | Tượng (img_017, img_011 Predator), ma-nơ-canh (img_033), người trên biển quảng cáo / poster (img_018, img_030, img_033 bên trái), linh vật / thú nhồi (con chuột img_019) → **không gán**. Người ngồi trong xe kín sau kính → không gán (xem 3.6). |
| `ignore_region` | Vùng **không gán từng vật**: vật cao < 20 px, đám đông / cụm xe dày đặc không tách được, vật mờ không xác định được ranh giới. Bỏ sót trong vùng này không tính lỗi. | img_019 (đám đông sau rào), img_021 (khối xe máy phía xa), img_030 (bãi xe máy phía sau), img_025 (người tắm nắng xa) | **Không** dùng `ignore_region` cho tượng / hình in / phản chiếu: những vật đó để **trống** (không box) để mô hình học rằng đó *không phải* người/xe. |

**Nếu nhóm không thêm 3 lớp đề xuất**, phương án dự phòng: `bus` → `truck`; `three_wheeler` có động cơ → `motorcycle`;
`bicycle` và xích lô → `ignore_region` (vẫn gán `person` cho người đạp). Phương án này làm mất thông tin, tôi không khuyến khích.

### 2.2. Vật không thuộc lớp nào

| Loại vật | Xử lý | Lý do |
|---|---|---|
| Vật **không thật**: tượng, ma-nơ-canh, đồ chơi, mô hình xe, xe điêu khắc, hình in, poster, tranh tường, biển quảng cáo | **Không gán** (không box, không ignore) | Đây là negative mẫu tốt: dạy mô hình đừng coi chúng là người/xe. |
| **Phản chiếu**: trong gương, kính cửa hàng, mặt đường ướt, vũng nước | **Không gán** | Không phải vật thật ở vị trí đó. |
| **Xe chủ (ego)**: thân xe, gương, nắp capo của xe người chụp | **Không gán** | Không phải vật cần phát hiện. |
| **Hàng hoá chở trên xe**: ô tô trên xe chở ô tô, tượng ngựa trên pickup, xe đồ chơi trên giá nóc | **Không gán riêng**; thuộc box của xe chở (xem 3.5) | Chúng di chuyển cùng xe, không phải phương tiện độc lập. |
| Động vật, chim, cọc tiêu, xe đẩy tay, ghế, ô dù | **Không gán** | Ngoài phạm vi taxonomy. Nếu nhóm muốn thêm `animal`, cần họp. |

Quy tắc kiểm tra "thật hay không": vật là thật nếu **có kích thước thật + có thể tự di chuyển / được lái**
(có bánh lốp chịu tải, có người lái hoặc biển số). Xe diễu hành có biển số và người lái → thật (img_019).

### 2.3. Thuộc tính

**`occluded_level`** — phần trăm diện tích của vật bị **vật khác** che (ước lượng theo hình dạng đầy đủ của vật).

| Mức | Nghĩa | Ngưỡng | Ví dụ |
|---|---|---|---|
| `0` | Không bị che | ≤ 10% | 2 người qua đường img_014 |
| `1` | Che ít | > 10% – 40% | Xe sau hàng rào song thưa img_018 |
| `2` | Che nhiều | > 40% – 70% | Van bị tượng Predator che img_011 |
| `3` | Che gần hết | > 70% (nhưng vẫn nhận ra lớp) | Xe máy giữa bãi xe img_030 |

- Bị **mép ảnh** cắt **không** tính vào `occluded_level` (đó là `truncated`).
- **Người ngồi trên xe không tính là che xe** của chính họ (người + xe là một cụm). Người/xe *khác* đứng trước thì tính.
- Hàng rào / lưới thưa mà vẫn thấy rõ đường viền vật phía sau → tối đa mức `1`.
- Nhìn thấy < 10% vật **và** không nhận ra lớp → không gán box riêng (gộp vào `ignore_region` nếu nằm trong cụm).
- Phân vân giữa hai mức → chọn **mức cao hơn**.

**`truncated`** — vật bị mép ảnh cắt.

| Giá trị | Ngưỡng |
|---|---|
| `no` | Thấy trọn vật trong khung, hoặc phần ra ngoài khung ước tính < 5% |
| `yes` | Ước tính ≥ 5% vật nằm ngoài khung ảnh (box chạm mép ảnh, vật rõ ràng còn tiếp tục) |

**`can_review`** (checkbox) — tick khi **quy tắc trong guideline không quyết định được** (không phải khi lười đo).
Mỗi lần tick ghi lại mã ảnh + lý do vào danh sách ca khó để mang đi họp. Không tự đoán quy tắc mới.

---

## 3. Quy tắc vẽ box

### 3.1. Nguyên tắc chung
1. Box **ôm sát phần nhìn thấy** của vật (không đoán phần bị che). Mép box chạm pixel ngoài cùng của vật,
   sai số ≤ 2 px với vật cao ≥ 50 px, ≤ 1 px với vật cao 20–50 px (để đạt IoU 0.85 / 0.7 khi so hai người).
2. Gán **cả xe đang chạy và xe đang đỗ** (xe đỗ vẫn là chướng ngại vật).
3. Ảnh đen trắng, ảnh chỉnh màu, ảnh qua kính chắn gió (img_015, img_018, img_022, img_029) vẫn gán bình thường.
4. Ảnh không có vật hợp lệ vẫn phải nằm trong file export với 0 box.

### 3.2. Thành phần nằm trong box
| Vật | Gồm | Không gồm |
|---|---|---|
| Ô tô / van / truck / bus | Thân, bánh, gương chiếu hậu, cản, đèn, thùng hàng, rơ-moóc đang nối | Ăng-ten, cờ, dây buộc lỏng, bóng đổ |
| Xe máy / xe đạp | Bánh trước – bánh sau, tay lái, gương, giỏ, sidecar, thùng chở gắn cứng | Chân chống không có tác dụng hình dạng (không cần cố tình kéo box tới) |
| Người | Đầu (gồm mũ, nón lá, mũ bảo hiểm) tới chân, ba lô / túi đeo trên người | Đồ cầm xa người: gánh hàng (img_003), ô, ống dài (img_029), hàng hoá |

### 3.3. Vật bị che
- Che tách vật thành 2 phần nhìn thấy (ví dụ xe sau cột điện) → **một** box bao cả hai phần, không vẽ 2 box.
- Ghi `occluded_level` theo bảng 2.3.

### 3.4. Vật bị cắt mép
- Box kéo tới đúng mép ảnh, `truncated = yes`.
- Chỉ thấy một mảnh nhỏ không nhận ra lớp (ví dụ chỉ thấy gương xe máy ở đáy img_040) → **không gán**.

### 3.5. Người trên xe, xe chở vật khác
- **Người lái / người ngồi trên xe hở** → 1 box `person` cho mỗi người + 1 box cho xe. Box được phép chồng nhau.
  img_005: mỗi người một box, mỗi xe máy nhận ra được một box. img_026: 2 `person` + 1 `motorcycle`.
- **Người trong xe kín** (sau kính ô tô, xe tải, buýt) → không gán (mô hình chỉ cần phát hiện xe).
- **Hàng chở trên xe** nằm trong box của xe nếu phần nhô ra ≤ 50% chiều dài thân xe (img_035: box auto-rickshaw gồm
  đống bao hàng trên nóc). Nhô ra > 50% → box chỉ ôm thân xe, tick `can_review` (img_029: ống kim loại dài).
- **Xe chở xe** → một box `truck` cho cả đầu kéo + rơ-moóc + hàng; ô tô trên xe chở **không** gán riêng (img_002)
  **[chưa chốt, cần họp]** — có người sẽ cho rằng xe con trên đó vẫn là `car`.

### 3.6. Vật nhỏ, đám đông, mờ, tối
| Tình huống | Ngưỡng | Xử lý |
|---|---|---|
| Vật quá nhỏ | Box cao < 20 px | Không gán từng vật; gộp các vật nhỏ gần nhau vào **một** `ignore_region` |
| Đám đông / cụm xe dày đặc | ≥ 5 vật cùng lớp mà hơn một nửa bị che > 50% lẫn nhau, không xác định được ranh giới từng vật | **Một** `ignore_region` bao cụm. Vật ở **rìa** cụm, cao ≥ 20 px, tách được ranh giới → vẫn gán riêng |
| Nhoè chuyển động | Nhận ra lớp **và** xác định được mép vật trong khoảng ±10% | Gán, box ôm phần đậm nhất (bỏ vệt mờ mỏng phía sau), tick `can_review` |
| Nhoè không nhận ra | Không nhận ra lớp hoặc không xác định được mép | `ignore_region` |
| Quá tối | Không nhìn ra đường viền vật dù đã tăng sáng trong CVAT | `ignore_region` nếu chắc có vật; không có gì để thấy thì bỏ qua |

---

## 4. Ca khó

| Ảnh | Tình huống | Quyết định | Lý do |
|---|---|---|---|
| img_001 | Người ngồi quanh bàn bị bàn ghế che; poster vẽ hot dog | Mỗi người cao ≥ 20 px một box, `occluded_level` theo % che; poster không gán. Suzuki thùng lửng → `truck`, `truncated = yes` | Người thật dù ngồi; thùng lửng tách khỏi cabin |
| img_002 | Xe chở 8 ô tô con | 1 box `truck` (đầu kéo + rơ-moóc); ô tô trên xe không gán riêng **[chưa chốt, cần họp]** | Ô tô là hàng hoá, chuyển động cùng xe tải |
| img_003 | Người gánh hàng (2 thúng), xe đạp dựng, bãi xe máy dày | Box người chỉ ôm thân người, không ôm thúng; xe đạp → `bicycle`; bãi xe máy phía xa → `ignore_region` | Gánh hàng không phải phần cơ thể; cụm xe không tách được |
| img_004 | 2 tuk-tuk có tài xế ngồi trong | 2 `three_wheeler` + 2 `person` (tài xế thấy rõ, xe hở hông) | Tuk-tuk không có cửa kính kín |
| img_005 | Người trên xe máy cận cảnh, xe máy gần như bị người che hết | Mỗi người một box; xe máy chỉ gán nếu nhận ra (thấy bánh/đầu xe), người trên xe không tính che xe | Tránh vẽ box xe máy đoán mò |
| img_007 | Nhà kho tối, Hilux gắn thùng kín | Hilux có thùng kín → `truck`; Isuzu ELF → `truck` | Cabin tách khỏi thùng |
| img_008 | Chợ đêm: xe máy đỗ sát nhau, xe ôm mặc áo cam, songthaew ở xa | Xe máy tiền cảnh gán riêng; cụm xa → `ignore_region`; songthaew → `truck` **[chưa chốt, cần họp]** | Theo hình dạng pickup |
| img_009 | Minibus 25 chỗ | `bus` | Cửa gập kiểu buýt, ≥ 5 cửa sổ hông |
| img_011 | Tượng Predator khổng lồ, xe máy làm bằng sắt vụn, van phía sau | Tượng và xe máy điêu khắc: không gán; van → `van`, `occluded_level = 2`; người xa cao ≥ 20 px gán riêng, đám đông xa → `ignore_region` | Tượng lớn hơn người thật; xe điêu khắc không lái được. Xe sắt vụn tick `can_review` nếu ai thấy nó chạy được |
| img_013 | Đêm mưa, tượng rồng vàng cạnh 2 người đi bộ, mặt đường phản chiếu | 2 `person`; tượng không gán; phản chiếu không gán; xe xa cao < 20 px → `ignore_region` | Phản chiếu không phải vật thật |
| img_014 | Người đi xe máy nhoè chuyển động | Gán nếu nhận ra người/xe và mép rõ ±10%, tick `can_review`; vệt mờ không nhận ra → `ignore_region` | Vẫn là vật thật nguy hiểm, nhưng mép box không chắc |
| img_015 | Xe đen nhoè thành vệt (phơi sáng lâu), mặt đường ướt phản chiếu | Xe đen: gán `car` ôm phần đậm nhất, tick `can_review`; phản chiếu không gán; xe đỏ + xe bạc `truncated = yes` | Như img_014 |
| img_016 | Tuk-tuk, van HiAce, xe tay ga, pickup, người cận cảnh bị cắt | Tuk-tuk → `three_wheeler`; HiAce → `van`; pickup → `truck`; người phụ nữ → `person`, `truncated = yes`; khách trong tuk-tuk → `person` | Phân biệt van/pickup theo thân liền hay thùng tách |
| img_017 | Tượng người cưỡi ngựa đội cọc tiêu | Không gán gì (ảnh 0 box) | Tượng |
| img_018 | Người trên biển quảng cáo; xe sau hàng rào song | Người trên biển: không gán; xe sau rào → `car`, `occluded_level = 1` | Hình in; rào thưa vẫn thấy rõ viền xe |
| img_019 | Xe Dangermouse diễu hành, linh vật chuột, đám đông sau rào | Xe → `car` (có biển số, người lái), người lái → `person`; chuột: không gán; đám đông → một `ignore_region` | Xe chạy thật trên đường; linh vật là đồ vật |
| img_020 | Gương chiếu hậu phản chiếu xe phía sau | Không gán gì (ảnh 0 box); thân xe chủ không gán | Phản chiếu + xe chủ |
| img_021 | Hàng trăm xe máy chờ đèn | Tiền cảnh tách được: gán từng người + xe; khối xe phía sau → `ignore_region` | Theo ngưỡng đám đông 3.6 |
| img_022 | Ảnh đen trắng cũ, mô tô sidecar có người ngồi | 1 `motorcycle` gồm cả sidecar + `person` cho người ngồi | Sidecar là phụ kiện gắn vào xe máy |
| img_023 | Xích lô có người đạp | Xích lô → `three_wheeler`; người đạp → `person` | 3 bánh khung liền |
| img_024 | Bảo tàng ô tô, mô hình xe vàng trên giá nóc | Xe thật trong bảo tàng → `car` (đủ kích thước thật); xe vàng trên nóc: không gán | Xe trưng bày vẫn là xe thật; xe trên nóc là hàng / mô hình **[chưa chốt, cần họp]** |
| img_025 | Người đi bộ xa, người nằm ghế bãi biển, xe máy kéo thùng | Người cao ≥ 20 px gán riêng, nhóm người xa ở bãi biển → `ignore_region`; xe máy kéo thùng → `motorcycle` | Ngưỡng 20 px |
| img_026 | Đôi nam nữ trên xe máy, tranh in cảng biển phía sau | 2 `person` + 1 `motorcycle`; tàu trong tranh: không gán | Hình in |
| img_027 | Tượng pickup bằng đá ở nghĩa trang | Không gán (ảnh 0 box) | Tượng |
| img_028 | Pickup thật chở tượng ngựa | Pickup → `truck`, box gồm tượng ngựa (hàng trên xe, không nhô quá 50%); SUV đỏ phía sau → `car` | Tượng là hàng hoá |
| img_029 | Xe máy và xe đạp cùng khiêng một ống kim loại dài | `motorcycle` + `bicycle` + 2 `person`, box xe không ôm ống; tick `can_review` | Ống nhô ra > 50% chiều dài xe |
| img_030 | Bãi xe máy dày đặc, poster người | Xe máy tiền cảnh tách được gán riêng (`occluded_level` 2–3); khối phía sau → `ignore_region`; poster không gán | Theo ngưỡng đám đông |
| img_031 | Đêm mưa, tranh tường hoạt hình, phản chiếu đường ướt | Gán xe/người thật; tranh tường và phản chiếu không gán | Hình in + phản chiếu |
| img_032 | Đám người đứng chờ xe buýt chồng lên nhau | Người ở rìa tách được → `person`; phần giữa dày đặc → `ignore_region`; xe xanh bên trái → `car`, `truncated = yes` | Ngưỡng đám đông |
| img_033 | Ma-nơ-canh, ảnh in người mẫu, phản chiếu chiếc xe trên kính | Không gán gì (ảnh 0 box) | Ma-nơ-canh + hình in + phản chiếu |
| img_035 | Auto-rickshaw chở hàng cao gấp đôi thân xe | `three_wheeler`, box gồm cả đống hàng (hàng chồng lên trên, không nhô dài ra trước/sau) | Hàng gắn cứng, ảnh hưởng kích thước thật của xe |
| img_036 | Xe tải nhẹ thùng bạt, bãi xe tay ga phía sau | `truck`; xe tay ga tách được gán riêng, cụm → `ignore_region` | Cabin tách khỏi thùng |
| img_037 | Pickup gắn nắp thùng cao, buýt phía sau | Pickup → `truck`; buýt → `bus`; người đi bộ → `person` | Nắp thùng không biến pickup thành van |
| img_038 | Tủ kính bày ô tô mô hình, đồ chơi | Không gán gì (ảnh 0 box) | Đồ chơi |
| img_040 | Người đội mũ bảo hiểm cận cảnh, chỉ thấy gương xe máy | `person`, `truncated = yes`; xe máy của người đó: không gán (chỉ thấy gương); taxi → `car`; tuk-tuk → `three_wheeler`; buýt → `bus` | Mảnh xe quá nhỏ để nhận ra lớp |

**Ảnh dự kiến 0 box:** img_017, img_020, img_027, img_033, img_038.

**Câu hỏi chưa chốt, cần họp:**
1. Có thêm 3 lớp `bus`, `bicycle`, `three_wheeler` không? (ảnh hưởng ≥ 15/40 ảnh)
2. Ô tô trên xe chở ô tô (img_002): hàng hoá hay `car`?
3. Songthaew / pickup chở khách (img_008): `truck` hay `bus`?
4. Vật nhoè chuyển động (img_014, img_015): gán + `can_review` hay luôn `ignore_region`?
5. Xe máy sắt vụn (img_011): là tác phẩm điêu khắc hay xe chạy được?

---

## 5. QC và phiên bản

**Ngưỡng áp dụng** (giữ nguyên mục `rules` trong `taxonomy.json`, không đổi):

| Số đo | Ngưỡng |
|---|---|
| Box cao tối thiểu để gán riêng | 20 px |
| IoU giữa hai người gán | ≥ 0.85 (box cao ≥ 50 px), ≥ 0.7 (box cao 20–50 px) |
| IAA | IoU trung bình ≥ 0.85 **và** cùng lớp ≥ 95% (luôn ghi mẫu số: bao nhiêu cặp box) |
| Tỉ lệ ảnh có lỗi để Release | ≤ 2% (ghi mẫu số, ví dụ "1/40 ảnh") |

**Ai quyết ca mơ hồ:** người gán tick `can_review` + ghi mã ảnh và lý do → cả nhóm chốt trong buổi calibration →
mỗi quyết định ghi một dòng vào `calibration_decisions.csv`; quyết định nào là quy tắc mới thì vào guideline bản sau.
Không tự đổi quy tắc giữa lúc gán (Pha 3).

**Quyết định lô nhãn:** Hold (guideline chưa rõ / dữ liệu hỏng) → Rework (> 2% ảnh lỗi, lỗi lặp cùng kiểu,
hoặc IAA dưới ngưỡng) → Release.

**Lịch sử thay đổi:**

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v0.1-cá nhân | Đề xuất thêm `bus`, `bicycle`, `three_wheeler` | img_004, 009, 016, 023, 035, 037, 040 (tuk-tuk, xích lô, buýt); img_003, 006, 010, 029 (xe đạp) |
| v0.1-cá nhân | Định nghĩa lại `car` / `truck` / `van` theo "cabin tách thùng" và "thân liền một khối"; pickup → `truck` | img_001, 007, 012, 016, 028, 036, 037 |
| v0.1-cá nhân | `occluded_level` có ngưỡng % (10 / 40 / 70); tách rõ che ≠ cắt mép; người trên xe không tính che xe | img_005, 011, 018, 030 |
| v0.1-cá nhân | `truncated = yes` khi ≥ 5% vật ra ngoài khung | img_015, 016, 032, 040 |
| v0.1-cá nhân | Quy tắc vật không thật: tượng, ma-nơ-canh, đồ chơi, hình in, phản chiếu → không gán (không dùng `ignore_region`) | img_011, 013, 017, 018, 019, 020, 024, 027, 033, 038 |
| v0.1-cá nhân | Quy tắc người trên xe (hở / kín), hàng chở trên xe (ngưỡng 50%), xe chở xe | img_002, 004, 005, 026, 028, 029, 035 |
| v0.1-cá nhân | Quy tắc vật nhỏ (< 20 px), đám đông (≥ 5 vật che > 50%), nhoè, tối | img_014, 015, 019, 021, 025, 030, 032 |
