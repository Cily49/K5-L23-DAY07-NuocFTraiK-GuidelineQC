# Guideline gán nhãn — v1.1 (sau calibration) — NuocFTraiK

- Phiên bản: v1.1 (thay đổi `semantic` so với v1.0 — xem lịch sử phiên bản ở mục 6 và `submission/guideline_change.json`)
- Ngày: 2026-11-17
- Tổng hợp từ 5 bản cá nhân: 187, 234, 274, 276, 346; sửa theo IAA vòng 1 (`submission/qc/iaa_summary.md`)
- Người chốt: 346 (điều phối nhóm)
- Taxonomy đi kèm: `submission/taxonomy.json` (`version: v1.0`, **không đổi** lớp / thuộc tính; mục `rules` giữ nguyên)

> **Thay đổi chính v1.1:** R-09 (bắt buộc `ignore_region` trong cụm đông, tiêu chí đo được), R-02 (tiêu chí `rider` / `person`),
> R-01 (ôm sát), 2.2 (luật phân vân `occluded_level` / `truncated`), R-03 (ranh giới car / van). Các chỗ đổi đánh dấu **[v1.1]**.

> Quy tắc đọc: mỗi câu trong file này chỉ có **một** cách hiểu. Nếu khi gán vẫn thấy hai cách hiểu,
> vẽ theo cách hiểu tốt nhất, tick `can_review` và ghi ảnh + lý do (xem mục 5).

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện thật đang tham gia hoặc có thể cản trở giao thông** trên ảnh đường phố 2D (camera giao thông, hỗ trợ lái xe / xe tự lái).

Sai ở đâu thì nguy hiểm (xếp theo mức nghiêm trọng):

1. **Bỏ sót người hoặc xe thật** → mô hình "không thấy" vật cản. Lỗi nặng nhất, nhất là người trên xe máy và người bị che một phần.
2. **Gán nhầm vật không thật** (tượng, ma-nơ-canh, hình in, phản chiếu, đồ chơi) → mô hình báo động giả, phanh gấp vô cớ.
3. **Box lệch / sai kích thước** → sai ước lượng khoảng cách và vùng trống.
4. **Mỗi người gán một kiểu** → mô hình học lẫn lộn, IAA thấp.

Phép thử chung: *vật này có phải là một người / một phương tiện thật, có thể di chuyển trên đường không?* Có → gán theo lớp. Không → không gán.

---

## 2. Lớp và thuộc tính

### 2.1. Lớp (mỗi vật thuộc đúng một lớp — 10 lớp)

Phân loại theo **hình dạng nhìn thấy được**, không theo tên gọi địa phương hay tải trọng đoán.

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con 4 bánh chở người, khoang khách và khoang hành lý liền một khối (sedan, hatchback, SUV, crossover, taxi, MPV ≤ 7 chỗ). | img_015, img_023, img_019 (xe diễu hành) | Có thùng hàng hở tách cabin → `truck`; thân hộp, đầu ngắn → `van`; xe đồ chơi img_038, tượng xe img_027 → không gán |
| `van` | Xe thân hộp nguyên khối, đầu xe ngắn hoặc mũi bẹt, khoang sau liền cabin, dài cỡ ô tô con (minivan, xe van giao hàng, xe ≤ 16 chỗ). **[v1.1]** Phải có **đủ 2 dấu hiệu**: (a) nóc xe phẳng, kéo thẳng tới đuôi; (b) đuôi xe gần thẳng đứng. Thiếu một → `car`. | img_005 | MPV / SUV đuôi vát → `car`; dài ≥ ~1,5 lần ô tô con, nhiều hàng cửa sổ → `bus` |
| `truck` | Xe có thùng hàng **tách khỏi cabin**: xe tải thùng kín / bạt / ben, xe bồn, đầu kéo, **xe chở xe (một box bao cả xe chở lẫn các xe được chở)**, **xe bán tải (pickup) kể cả có nắp thùng**. | img_002, img_001, img_012, img_028, img_036, img_037 | SUV cốp liền khoang khách → `car`; thân hộp liền cabin → `van`; ô tô đang nằm trên xe chở → **không** gán `car` riêng |
| `bus` | Xe chở khách nhiều hàng ghế, dài ≥ ~1,5 lần ô tô con, nhiều cửa sổ dọc thân (minibus, xe buýt, xe khách). | img_009, img_032 | Dài cỡ ô tô con → `van`; có thùng hàng → `truck` |
| `motorcycle` | Xe **hai bánh có động cơ** (xe máy, xe tay ga, mô tô), gồm cả phần gắn liền di chuyển cùng xe: thùng bên (sidecar), thùng / xe hàng kéo theo không có mui. | img_006, img_021, img_022 (sidecar), img_025 (kéo xe hàng) | Không động cơ → `bicycle`; 3 bánh có mui / khoang sau → `three_wheeler` |
| `three_wheeler` | **Lớp mới.** Xe cơ giới 3 bánh có mui hoặc khoang chở khách / hàng phía sau người lái (tuk-tuk, xe lam, auto rickshaw). | img_004, img_016, img_035 | Xe máy kéo thùng không mui → `motorcycle`; xích lô (đạp) → `bicycle` |
| `bicycle` | **Lớp mới.** Xe dùng sức người đạp, không động cơ: xe đạp, xích lô. | img_003, img_006 (bên phải), img_029, img_023, img_039 | Có động cơ / ống xả / biển số → `motorcycle` |
| `rider` | **Lớp mới.** Người thật đang tham gia giao thông **bằng phương tiện hở**: người lái, người ngồi sau, trẻ em ngồi trước, người trong sidecar, khách xích lô / tuk-tuk, người lái xe mui trần. | img_006, img_022, img_026, img_023 | Người đứng / đi bộ cạnh xe, dắt xe → `person`; người ngồi trong cabin kín nhìn qua kính → không gán |
| `person` | Người thật **không** ngồi trên phương tiện: đi bộ, đứng, ngồi trên vỉa hè, dắt xe, người hóa trang. | img_001, img_014, img_034 | Người đang ngồi trên xe → `rider`; tượng (img_011, img_017), ma-nơ-canh (img_033), người trên biển quảng cáo (img_018), mascot thú bông (img_019) → không gán |
| `ignore_region` | **Vùng** (không phải vật): cụm đông / cụm vật nhỏ không tách được từng cá thể; bỏ sót vật bên trong không tính lỗi. | đám đông sau rào img_019, cụm xe xa img_021 | Vật tách được và cao ≥ 20 px → vẽ box riêng; vật không thật → không gán, **không** dùng `ignore_region` |

**Vật không thuộc lớp nào:**

- Vật không thật (tượng, ma-nơ-canh, đồ chơi, mô hình, hình in 2D, phản chiếu) → **không gán, không vẽ `ignore_region`** (R-05, R-06).
- Phương tiện thật không khớp lớp nào (ví dụ xe lăn, xe ba gác đẩy tay) → không gán box, ghi vào `submission/qc/image_issues.csv` (ảnh, vùng, lý do). **Không tự thêm lớp.**

### 2.2. Thuộc tính

**`occluded_level`** — tỉ lệ diện tích vật bị **vật khác** che, so với diện tích ước lượng của toàn bộ vật:

| Mức | Ngưỡng che | Nhận biết |
|---|---|---|
| `0` | ≤ 10% | Thấy trọn vật, hoặc chỉ bị che bởi chi tiết mảnh (cành cây, dây điện). |
| `1` | > 10% đến 30% | Mất một phần đầu / đuôi / bánh, vẫn thấy rõ phần lớn thân. |
| `2` | > 30% đến 70% | Mất khoảng nửa thân; phải suy luận biên từ phần còn lại. |
| `3` | > 70% | Chỉ lộ một phần nhỏ **nhưng vẫn chắc chắn lớp**. Không nhận ra lớp → không gán. |

**[v1.1] Cách ước lượng** (IAA vòng 1 chỉ khớp 60,8% thuộc tính này):
1. Hình dung box của **toàn bộ** vật nếu không bị che, chia làm 4 phần bằng nhau theo chiều ngang hoặc dọc.
2. Đếm số phần bị che: < ½ phần → `0`; khoảng 1 phần → `1`; 2 phần → `2`; 3 phần trở lên → `3`.
3. **Phân vân giữa hai mức → chọn mức thấp hơn.**
4. Chỉ tính vật che thật (người, xe, cột, rào, cây). Người ngồi trên xe **không** tính là che xe và ngược lại (box `rider` và box xe luôn chồng nhau theo R-02).

**`truncated`** — vật bị **mép ảnh** cắt (bị vật khác che thì dùng `occluded_level`, không dùng `truncated`):

- `yes`: box chạm mép ảnh **và** phần vật nằm ngoài khung ước tính ≥ 10%.
- `no`: không chạm mép, hoặc chỉ mất < 10% (một mẩu cản xe, gương).
- **[v1.1]** Phân vân (vật chạm mép nhưng không chắc đã mất ≥ 10%) → `no`. Người: mất cả bàn chân + cẳng chân, hoặc nửa đầu → `yes`.

**`can_review`** — tick khi không chắc lớp, có nên gán hay không, hoặc phạm vi box theo guideline này. Mỗi box tick phải ghi ảnh + lý do vào nhật ký. Không dùng để né quyết định mà guideline đã trả lời.

`ignore_region` không mang thuộc tính.

---

## 3. Quy tắc vẽ box

### R-01 — Box ôm sát phần nhìn thấy
Mép box chạm điểm ngoài cùng của **phần nhìn thấy** (trên, dưới, trái, phải), không chừa lề, không đoán phần bị che. Vật bị cắt mép thì box dừng ở mép ảnh.

**[v1.1]** Điểm ngoài cùng cụ thể (IAA vòng 1: 469 ca IoU thấp):
- Người / `rider`: đỉnh tóc hoặc mũ bảo hiểm → đế giày; tay dang ra và túi xách đang cầm **nằm trong** box.
- Xe: gồm gương chiếu hậu, bánh xe chạm đất, cản trước / sau; **không** gồm ăng-ten, bóng đổ.
- Khe hở giữa box và vật ≤ 2 px ở cả 4 cạnh. Sau khi vẽ, **zoom ≥ 200%** kiểm lại 4 cạnh trước khi chuyển vật khác.

### R-02 — Người trên xe: box xe + box `rider`, tách riêng
- Một box xe (`motorcycle`, `bicycle`, `three_wheeler`) + **một box `rider` cho mỗi người** ngồi trên xe (người lái, người ngồi sau, trẻ em ngồi trước, người trong sidecar, khách xích lô). Các box được chồng nhau.
- Box `rider` bao toàn bộ phần nhìn thấy của người, kể cả phần chồng lên xe (chân trên gác chân, tay trên tay lái).
- Box xe chỉ bao các bộ phận của xe (bánh, tay lái, gương, thân, đuôi, thùng bên). Không nới box xe để bao đầu, lưng, chân người thò ra ngoài khung xe.
- Người ngồi trong **cabin kín** (ô tô, van, bus, cabin tải) nhìn qua kính → không gán. Người ngồi trên xe **hở** (xe mui trần, thùng xe tải hở, tuk-tuk mà thân người lộ rõ) → `rider` nếu thỏa R-08.
- **[v1.1] Phân biệt `rider` / `person` — chỉ một câu hỏi: người đó có đang được phương tiện đỡ (ngồi trên yên / ghế / sàn xe, hoặc đứng trên bàn đạp / sàn xe) không?**
  - Có → `rider`. Gồm cả: người ngồi trên xe máy đang dừng đèn đỏ dù một chân chống đất; người ngồi trên xe đỗ bên đường; **người lái và khách trong tuk-tuk / xe lam** (img_004, img_016, img_035) khi thấy được thân người; khách xích lô.
  - Không → `person`. Gồm: đứng cạnh xe, dắt xe, dựa vào xe, đang leo lên / bước xuống mà mông chưa chạm yên.
  - Không thấy được người đó có ngồi trên xe hay không (bị che phần dưới) nhưng đầu / thân nằm ngay trên khung xe → `rider`.
  - Không tick `can_review` cho câu hỏi này nữa — luật trên đã trả lời.
- **[v1.1]** Không gán nhầm lớp xe cho người hoặc ngược lại: trước khi chuyển sang vật khác, kiểm tên lớp hiện trên box (IAA vòng 1 có box người bị gán `car` ở img_004, img_005 và box xe gán `person` ở img_039 — lỗi chọn nhãn).

### R-03 — Phân biệt car / van / truck / bus
Theo cấu trúc khung nhìn thấy (định nghĩa 2.1). Pickup → `truck`; pickup có nắp thùng → vẫn `truck` nếu thấy khe chia giữa cabin và thùng. Không chắc giữa hai lớp → chọn lớp gần nhất theo định nghĩa và tick `can_review`.

**[v1.1]** Nhắc lại vì IAA vòng 1 có 8 ca `car` ↔ `truck` ở img_007, img_012, img_028, img_037: **mọi xe bán tải là `truck`**, kể cả xe bán tải cổ, xe bán tải có nắp thùng, xe bán tải đang chở người / tượng. Ranh giới `car` / `van`: dùng 2 dấu hiệu ở định nghĩa `van` (img_011, img_016).

### R-04 — Xe chở xe: một box `truck` cho cả xe
- Phương tiện đang chở phương tiện khác (xe chở ô tô, xe tải chở xe máy, ô tô chở ô tô trên nóc) → **một** box `truck` bao cả xe chở **và** toàn bộ các xe được chở.
- **Không** vẽ box `car` / `motorcycle` riêng cho xe đang nằm trên xe chở.
- `occluded_level` / `truncated` tính cho cả khối (xe chở + hàng xe).

### R-05 — Vật không thật: không gán
Tượng, ma-nơ-canh, mô hình, đồ chơi, mascot thú bông, hình in trên biển quảng cáo / tranh tường / màn hình → **không vẽ box, không vẽ `ignore_region`**.
- Tượng đặt trên xe thật (img_028): xe gán bình thường; tượng không gán và không tính vào `occluded_level` của xe nếu không che thân xe.
- Xe diễu hành, xe độ có bánh và chạy được trên đường (img_019, img_011): **là xe thật** → gán theo hình dạng gần nhất.

### R-06 — Phản chiếu và xe chủ (ego): không gán
- Phản chiếu trên gương chiếu hậu, kính cửa hàng, vũng nước, mặt đường ướt (img_013, img_020, img_031, img_033) → không gán.
- Phần xe mang camera (ca-pô, đầu xe, gương, tay lái) ở mép ảnh (img_020, img_040) → không gán.

### R-07 — Hàng hóa không tính vào box
Box xe chỉ bao **thân xe**. Hàng chất cao, bao tải, thanh dài thò ra ngoài thân xe / thân người → để ngoài box (img_029, img_035). Hàng nằm gọn trong thùng xe thì tự nhiên đã nằm trong box xe.

### R-08 — Vật nhỏ
- Ngưỡng `min_box_height_px = 20`, đo chiều cao box trên **độ phân giải gốc**.
- Vật đơn lẻ cao < 20 px → bỏ qua.
- Cụm nhiều vật < 20 px nằm sát nhau (cuối đường, chân trời) → một box `ignore_region` bao cụm.

### R-09 — Đám đông và bãi xe **[v1.1 — viết lại]**

IAA vòng 1: 4/5 người không vẽ `ignore_region` nào, mỗi người tự chọn vẽ bao nhiêu vật trong đám đông → 534 ca bỏ sót / thừa, tập trung ở img_021, img_019, img_003, img_008, img_032, img_006. v1.1 thay bằng tiêu chí đo được:

1. **Cụm đông** = từ **5 vật trở lên** (người / xe) mà box của chúng chồng lên nhau liên tiếp.
2. Trong cụm đông, chỉ vẽ box riêng cho vật thỏa **đủ 3 điều kiện**: (a) cao ≥ 20 px; (b) `occluded_level` 0, 1 hoặc 2 (che ≤ 70%); (c) nhìn được **cả 4 cạnh** để vẽ box (không phải đoán cạnh nào).
3. Vật trong cụm thiếu một điều kiện → **không** vẽ box riêng; phần đó bắt buộc bao bằng `ignore_region`.
4. **Ảnh có cụm đông thì bắt buộc có ít nhất 1 `ignore_region`.** Ảnh trong lô thuộc diện này: img_003, img_006, img_008, img_019, img_021, img_030, img_032, img_034.
5. Cách vẽ `ignore_region`: một hình chữ nhật bao phần cụm còn lại, mép ôm theo **đỉnh đầu cao nhất và chân / bánh thấp nhất** của phần đó. Cụm bị cột / cây chia đôi → tối đa 3 vùng. Không vẽ `ignore_region` cho từng người.
6. `ignore_region` **được phép** chồng một phần lên box riêng (v1.0 cấm — không vẽ được trong thực tế), nhưng **không** được bao trọn một box riêng nào.

### R-10 — Nhòe chuyển động
Gán bình thường nếu vẫn nhận ra lớp và ước lượng được biên (img_014). Không nhận ra → không gán; vệt nhòe lớn chứa nhiều vật → `ignore_region`.

### R-11 — Ảnh không có vật cần gán
Vẫn giữ ảnh trong Task; file COCO phải có ảnh đó (0 annotation).

---

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_022 | Mô tô kèm thùng bên (sidecar), có người lái | R-02, 2.1 | 1 `motorcycle` bao cả mô tô + sidecar; người lái → `rider` riêng | Sidecar gắn liền, di chuyển như một khối |
| img_029 | Người đi xe đạp chở thanh dài | 2.1, R-02, R-07 | `bicycle` + `rider`; thanh dài để ngoài box | Xe thật nhưng khác xe máy về tốc độ / quỹ đạo; hàng thò ra không thuộc thân xe |
| img_037 | Pickup có nắp thùng | R-03 | `truck` | Vẫn thấy khe chia cabin / thùng |
| img_002 | Xe đầu kéo chở 7–8 ô tô trên 2 sàn | R-04 | **Một** box `truck` bao cả đầu kéo, rơ-moóc và toàn bộ ô tô được chở; không gán `car` riêng | Cả khối di chuyển như một vật; ô tô trên giàn bị thanh chắn che nhiều, vẽ riêng gây lệch IoU |
| img_019 | Xe diễu hành mui trần có người lái thật; mascot chuột; đám đông sau rào | R-05, R-02, R-09 | Xe → `car`; người lái → `rider` (xe hở); mascot không gán; người đứng hàng đầu tách được → `person`, phần còn lại → `ignore_region` | Xe có biển số, đang chạy; mascot là thú bông; người xem đứng trên đất |
| img_021, img_030 | Hàng chục xe máy chen nhau | R-09, R-02 | Mỗi xe đủ 3 điều kiện R-09 → `motorcycle`, người trên xe → `rider`; phần còn lại **bắt buộc** `ignore_region` | Không tách được biên từng vật |
| img_004, img_016, img_035 **[v1.1]** | Người lái và khách trong tuk-tuk | R-02 | `rider` (không phải `person`) | Được sàn / ghế xe đỡ; IAA vòng 1 lệch `person` ↔ `rider` ở cả 3 ảnh |
| img_005, img_026, img_010 **[v1.1]** | Người ngồi trên xe máy đang dừng / đỗ, chân chống đất | R-02 | `rider` | Mông trên yên → được xe đỡ |
| img_007, img_012, img_028, img_037 **[v1.1]** | Xe bán tải | R-03 | `truck` | IAA vòng 1 có người gán `car` |
| img_023 **[v1.1]** | Xích lô | 2.1 | `bicycle` (không phải `three_wheeler`) | Không có động cơ |
| img_008, img_012 **[v1.1]** | Hai ca 274 tick `can_review` (xe tải img_008, ô tô img_012) | R-03 | img_008 → `truck`; img_012 → theo ảnh gốc: xe bán tải → `truck`, còn lại → `car` | Đã có luật trả lời, bỏ cờ `can_review` |

---

## 5. Giải quyết bất đồng giữa các bản cá nhân

| Vấn đề | Các đề xuất | Quyết định v1.0 | Lý do |
|---|---|---|---|
| Người đi xe máy | 346: 1 box `motorcycle` bao cả người; 187, 234, 274, 276: tách box người + box xe | **Tách**: box xe + box `rider` (R-02) | 4/5 đồng ý; mục tiêu là không bỏ sót người — gộp làm mô hình không học được người trên xe |
| Lớp cho người trên xe | Cả 5 bản: dùng chung `person` | Lớp mới **`rider`** (quyết định họp nhóm) | Người đi bộ và người trên xe có tốc độ, quỹ đạo, vùng nguy hiểm khác nhau; mô hình cần tách để dự đoán hành vi |
| Xe chở xe | 187, 234, 276, 346: 1 `truck` + `car` riêng cho từng xe được chở | **Một box `truck` cho cả xe** (R-04) (quyết định họp nhóm) | Cả khối di chuyển như một vật cản; xe trên giàn bị che nhiều, vẽ riêng làm lệch IoU và đếm thừa phương tiện đang lưu thông |
| Xe 3 bánh có động cơ | 234 `auto_rickshaw`; 276, 346 `three_wheeler`; 187 `rudimentary_vehicle`; 274 `tuk tuk` | Lớp mới **`three_wheeler`** | 5/5 muốn tách khỏi `car`/`motorcycle`; tên mô tả hình dạng, không theo địa phương; `tuk tuk` sai pattern tên lớp |
| Xe đạp / xích lô | 234 `bicycle` (gồm xích lô); 274 `bicycle` + `Cyclo`; 187 gộp vào `rudimentary_vehicle`; 276, 346 không gán | Lớp mới **`bicycle`**, gồm xích lô | Xe thật trên đường; không gán thì bỏ sót vật cản (lỗi nguy hiểm nhất). Xích lô ít mẫu, cùng bản chất "đạp" nên gộp. Không gộp với tuk-tuk vì tốc độ / kích thước khác hẳn |
| Xe buýt | 187, 234, 274, 346 thêm `bus`; 276 không | Lớp mới **`bus`** | 4/5; img_009, img_032 ép vào `van`/`truck` làm bẩn định nghĩa |
| Ranh giới van / bus | 346: theo số chỗ (16); 234: theo chiều dài (≥ 1,5 ô tô con) | Theo **chiều dài + số hàng cửa sổ**, 16 chỗ chỉ là gợi ý | Số chỗ không nhìn thấy được trong ảnh |
| Xe lồng / rơ-moóc (274 `fence_trailer`) | Chỉ 274 đề xuất | **Không thêm lớp** | Ít mẫu; xe hàng kéo bởi xe máy gộp vào `motorcycle` (img_025) |
| Tượng, ma-nơ-canh, gương, xe chủ | 234: `ignore_region`; 187, 274, 276, 346: không gán | **Không gán** (R-05, R-06) | 4/5; `ignore_region` vẽ quanh tượng tạo thêm box để lệch IoU mà không giúp mô hình |
| Hàng hóa chất cao | 234: box bao cả hàng; 187, 346: chỉ thân xe | **Chỉ thân xe** (R-07) | Biên hàng hóa khó thống nhất; lớp định nghĩa theo hình dạng xe |
| Thang `occluded_level` | 234: 10/40/70/90; 187: 5/30/70; 346: 0/30/70; 276: 10/25/50; 274: 50 | **≤10 / ≤30 / ≤70 / >70** | 30% và 70% là mốc chung của 187, 234, 346; mốc 10% cho mức 0 (234, 276) chịu được che bởi chi tiết mảnh |
| `truncated` | 346, 276: bất kỳ phần nào ra ngoài; 187, 234: ≥ 10% | **≥ 10%** | Tránh lệch vì mẩu gương / cản xe chạm mép |
| Ngưỡng vật nhỏ | 274: cạnh ngắn < 32 px; còn lại: cao < 20 px | **Cao < 20 px** | `rules.min_box_height_px` của taxonomy cố định, không được đổi |
| Người trong ô tô (346 để chờ chốt) | Chưa ai quyết | Cabin kín → không gán; xe hở → `rider` (R-02) | Người trong cabin kín không phải vật cản riêng; người lộ thân trên xe hở thì có |
| Nhận diện ảnh | 276 ghi img_022 là tượng; 346 ghi img_006 là xích lô | Sửa: img_022 là mô tô sidecar; img_006 là xe máy + xe đạp | Đối chiếu lại ảnh gốc |

---

## 6. QC và phiên bản

### Ngưỡng QC (từ `rules` của `taxonomy.json`, không đổi)

| Kiểm tra | Ngưỡng đạt |
|---|---|
| IoU khớp box | Box cao ≥ 50 px: ≥ 0,85 · box cao < 50 px: ≥ 0,7 (ghép cặp ở IoU ≥ 0,5) |
| IAA | IoU trung bình ≥ 0,85 **và** cùng lớp ≥ 95% |
| Vật nhỏ | Cao < 20 px không gán riêng (R-08) |
| Tỉ lệ ảnh lỗi | ≤ 2% ảnh đã duyệt — lô 40 ảnh: 1 ảnh lỗi (2,5%) là vượt |

- Mọi số ghi dạng `tử/mẫu` (ví dụ `1/40`); không có cặp box thì ghi N/A. Giữ số IAA **trước** calibration.
- Lỗi ghi vào `qc_review.csv` theo 5 loại: `bo_sot`, `thua`, `sai_lop`, `sai_vi_tri_kich_thuoc`, `sai_thuoc_tinh` (ảnh sạch: `khong`), kèm mã quy tắc R-xx.

### Ai quyết ca mơ hồ

1. **Lúc gán:** không hỏi nhau, không tự đặt quy tắc mới. Vẽ theo cách hiểu tốt nhất, tick `can_review`; vấn đề cấp ảnh ghi vào `submission/qc/image_issues.csv`.
2. **Lúc calibration:** cả nhóm xem ca trong `calibration_candidates.csv`; **346 chốt**. Mỗi quyết định một dòng trong `calibration_decisions.csv` (ca, quyết định, R-xx, người chốt).
3. **Phân loại nguyên nhân lệch:** cả nhóm lệch cùng kiểu → guideline mơ hồ, sửa guideline; một người lệch → hướng dẫn lại; mọi người giống nhau nhưng cùng sai → đối chiếu ca tham chiếu.

### Khi nào tăng phiên bản

- Sửa câu chữ, không đổi cách gán → `editorial`, tăng nhỏ, không gán lại.
- Đổi lớp / cách vẽ box / thuộc tính → tăng phiên bản, cập nhật `taxonomy.json` nếu đổi schema, **kiểm lại các ảnh bị ảnh hưởng**.

### Duyệt và bàn giao

Duyệt toàn bộ 40 ảnh; người duyệt khác người sửa nhãn. Thứ tự quyết định: **Hold** (thiếu bằng chứng, guideline chưa chốt) → **Rework** (> 2% ảnh lỗi hoặc lỗi lặp) → **Release** (đạt hết, người duyệt xác nhận).

### Lịch sử phiên bản

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu: 6 lớp, 3 thuộc tính, 3 quy tắc chung | — |
| v1.0 | Hợp nhất 5 bản cá nhân: thêm `bus`, `three_wheeler`, `bicycle`, `rider`; xe chở xe gán một box `truck`; định lượng `occluded_level` và `truncated`; R-01…R-11; bảng ca khó và bảng giải quyết bất đồng | img_002, img_019, img_021, img_022, img_029, img_030, img_037 |
| v1.1 (`semantic`) | R-09 viết lại: định nghĩa cụm đông (≥ 5 vật), 3 điều kiện vẽ box riêng, **bắt buộc** `ignore_region`, cho phép chồng một phần; R-02: tiêu chí duy nhất "được xe đỡ" cho `rider`, khách tuk-tuk là `rider`; R-01: điểm ngoài cùng và kiểm zoom; 2.2: cách ước lượng 4 phần, phân vân chọn mức thấp, phân vân `truncated` → `no`; `van` cần đủ 2 dấu hiệu; nhắc pickup → `truck`. Taxonomy không đổi. | IAA vòng 1: IoU 0,822, lớp 94,2% (992 cặp) — chưa đạt. 534 ca bỏ sót / thừa (img_021, img_019, img_003, img_008, img_032, img_006); 27 ca `person` ↔ `rider` (img_016, img_026, img_035…); 8 ca `car` ↔ `truck` (img_007, img_012, img_028, img_037); 423 ca thuộc tính |
