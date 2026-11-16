# Guideline gán nhãn — bản cá nhân (trước v1.0) — 234

- Phiên bản: cá nhân, dựa trên v0
- Ngày: 2026-11-16
- Người viết: 234

## 1. Mục tiêu

**Nhãn dùng để làm gì?** Dạy máy tính nhìn ảnh đường phố và tự khoanh ra **người** và **xe** (ô tô, xe van, xe tải, xe buýt, xe máy, xe đạp, xe lam / tuk-tuk). Ứng dụng: camera giao thông, xe tự lái.

**Sai ở đâu thì nguy hiểm?**
- **Bỏ sót** người hoặc xe thật → xe tự lái "không thấy" để tránh. Đây là lỗi nguy hiểm nhất.
- **Gán nhầm** vật không thật (tượng, biển quảng cáo, hình trong gương) → hệ thống phanh gấp vô cớ.
- **Box lệch hoặc sai kích thước** → máy đoán sai khoảng cách, sai chỗ trống để đi.
- **Mỗi người gán một kiểu** → máy học lẫn lộn. Vì vậy mọi quy tắc dưới đây phải hiểu được **một cách duy nhất**.

Nguyên tắc chung đã dùng trong bản này:
- Mô hình chỉ học những vật thể thật có thể tham gia hoặc cản trở giao thông.
- Thêm 3 lớp vào 6 lớp của `taxonomy.json` v0: `bus`, `bicycle`, `auto_rickshaw` (xem bảng lớp). Lý do: các loại xe này có nhiều trong 40 ảnh; ép chúng vào `car` / `motorcycle` / `truck` hoặc bỏ qua thì mô hình học sai hoặc bỏ sót xe thật.
- Phân loại theo **hình dạng nhìn thấy**, không theo quốc gia hay tên gọi địa phương: cùng một loại xe thì cùng lớp, dù ở Việt Nam, Thái Lan hay Ấn Độ.
- Không thêm lớp khi trong lô gần như không có mẫu (ví dụ xe ba gác chở hàng: không có ảnh rõ).

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con: khoang hành khách và khoang hành lý liền một khối (sedan, hatchback, SUV, crossover, MPV 5–7 chỗ). | 023, 015 | `van`: thân hộp, đầu xe ngắn |
| `van` | Xe van chở người hoặc hàng, dài cỡ ô tô con: thân hình hộp nguyên khối, đầu xe ngắn (mũi bẹt hoặc nhô rất ngắn), khoang sau liền cabin, có cửa trượt hông hoặc cửa đôi phía sau. | 005 | `car` (MPV); minibus, xe buýt (→ `bus`) |
| `truck` | Xe có thùng hàng riêng, tách khỏi cabin: (a) xe tải — cabin đầu vuông hoặc đầu dài chuyên dụng, thùng kín / bạt / ben, xe bồn, đầu kéo; (b) xe bán tải (pickup) — thùng hở phía sau tách khỏi cabin bằng vách ngăn và khe hở cấu trúc. | 007, 002, 001, 012, 028, 036, 037 | `car` (SUV: cốp liền khoang khách), `van`, `bus` |
| `bus` | **Lớp mới.** Xe chở khách nhiều hàng ghế, dài từ khoảng 1,5 lần ô tô con trở lên, nhiều cửa sổ dọc thân (minibus, xe buýt, xe khách). | 009, 032 | `van` (dài cỡ ô tô con), `truck` (có thùng hàng) |
| `motorcycle` | Xe máy, mô tô **hai bánh**; gồm cả xe có thùng bên (sidecar). | 006, 022 | `auto_rickshaw` (3 bánh, có mui / khoang khách), `bicycle` (đạp, không động cơ) |
| `auto_rickshaw` | **Lớp mới.** Xe cơ giới 3 bánh có mui hoặc khoang chở khách / hàng phía sau người lái: xe lam, tuk-tuk, auto rickshaw. | 004, 016, 035 | `motorcycle` (2 bánh, không mui), `bicycle` (xích lô: đạp bằng chân) |
| `bicycle` | **Lớp mới.** Xe dùng sức người đạp, không động cơ: xe đạp, xích lô. | 003, 029, 023, 039 | `motorcycle` (có động cơ, ống xả, biển số) |
| `person` | Người thật, kể cả người hóa trang. | 014, 034, 011 | tượng, ma-nơ-canh, hình in |
| `ignore_region` | Vùng không gán từng vật; bỏ sót trong vùng không tính lỗi. Dùng cho tượng, gương, xe chủ, cụm đông / nhỏ. | 021 (cụm xe phía sau), 017 (tượng) | — |

### Vật không thuộc lớp nào

Xem R-05 (vật không thật), R-06 (phản chiếu, xe chủ), R-07 (đám đông), R-08 (vật nhỏ).

### Thuộc tính

**`occluded_level`** (R-10): bỏ mức "3 = không rõ" của v0. Tính theo tỉ lệ diện tích bị che so với diện tích ước lượng của toàn bộ vật.

| Giá trị | Tên | Ngưỡng | Cách nhận biết |
|---|---|---|---|
| `0` | Không che | ≤ 10% | Thấy trọn vật, hoặc chỉ bị che bởi chi tiết rất nhỏ (cành mảnh, dây điện). |
| `1` | Che nhẹ | > 10% đến 40% | Mất một phần đầu, đuôi hoặc bánh, vẫn thấy rõ phần lớn thân. |
| `2` | Che trung bình | > 40% đến 70% | Mất khoảng nửa thân đến phần lớn thân; phải suy luận biên box từ phần còn lại. |
| `3` | Che nặng | > 70% đến 90% | Chỉ lộ một góc nhỏ (đầu xe, nóc xe, một bên vai) nhưng vẫn chắc chắn là vật gì. |
| (không gán) | Che toàn phần | > 90% | Không gán; nếu cần thì gộp vào `ignore_region`. |

**`truncated`** (R-09):
- `yes`: vật chạm ít nhất một mép ảnh **và** mất từ 10% kích thước thực tế trở lên ra ngoài khung.
- `no`: không chạm mép, hoặc chỉ mất dưới 10% (ví dụ một mẩu cản xe hay gương).

**`can_review`**: tick khi không chắc, để người khác xem lại (giữ như v0).

## 3. Quy tắc vẽ box

### R-01 — Người trên xe máy, xe đạp, xích lô
- Vẽ tách 2 loại box: `person` cho người và box xe (`motorcycle`, `bicycle`, `auto_rickshaw`). Không gộp làm một box.
- Mỗi người một box `person`: người lái, người ngồi sau, trẻ em ngồi trước hoặc giữa.
- Box `motorcycle` chỉ ôm các bộ phận của xe (bánh trước, bánh sau, tay lái, gương, đuôi xe). Không nới box để bao đầu, lưng hay chân người thò ra ngoài khung xe. Ngoại lệ: bàn chân đặt trên gác chân nằm gọn trong biên của xe.

### R-02 — Xe đặc thù (xe buýt, xe 3 bánh, xe đạp, xích lô)

| Loại xe | Ảnh | Quy tắc |
|---|---|---|
| Minibus, xe buýt | 009, 032 | `bus` nếu dài từ khoảng 1,5 lần ô tô con trở lên. Ngắn hơn → `van`. |
| Tuk-tuk, xe lam, auto rickshaw | 004, 016, 035 | `auto_rickshaw` khi có **đủ**: 3 bánh, động cơ, mui hoặc khoang sau người lái. Thiếu mui / khoang (xe máy kéo thùng) → `motorcycle`. |
| Xe có thùng bên (sidecar) | 022 | Một box `motorcycle` bao cả thân xe máy và thùng bên. |
| Xe đạp | 003, 029 | `bicycle` (tốc độ và quỹ đạo khác xe máy nên không gộp vào `motorcycle`). Người đạp xe gán `person` riêng. |
| Xích lô | 023, 039 | `bicycle` (dùng sức người đạp). Không bao giờ gán `motorcycle`, `auto_rickshaw` hay `car`. Người đạp và khách gán `person` riêng. |

### R-03 — Phân biệt car / van / truck / bus
Dựa vào cấu trúc khung và khoang chở hàng nhìn thấy được, không đoán theo tải trọng (định nghĩa ở mục 2).
- Xe bán tải (pickup) có thùng hở tách khỏi cabin → `truck` (taxonomy: "xe có thùng hàng riêng").
- Pickup có nắp thùng (037): vẫn là `truck` nếu thấy đường chia (khe dọc) giữa cabin và thùng sau, hoặc nắp thùng có đường viền / chất liệu tách rõ khỏi nóc cabin.

### R-04 — Xe chở xe
- Xe chở nhiều ô tô (002): một box `truck` cho đầu kéo / xe chở. Gán `car` cho từng ô tô được chở nếu cao ≥ 20 px và nhận rõ hình dáng xe.
- Ô tô chở ô tô trên nóc hoặc trong thùng (024): một box cho xe dưới, một box cho xe trên.
- `occluded_level` của xe được chở: tính theo phần diện tích của chính xe đó bị thanh chắn rơ-moóc, thành thùng hoặc xe khác che, so với diện tích ước lượng của nó.

### R-05 — Vật không thật

| Nhóm | Ảnh | Quy tắc |
|---|---|---|
| Hình 2D trên mặt phẳng (người trên biển quảng cáo, tranh tường) | 018, 026 | Bỏ qua hoàn toàn: không box, không `ignore_region`. |
| Tượng, ma-nơ-canh (tượng người, tượng pickup, tượng ngựa, ma-nơ-canh) | 017, 027, 028, 033 | `ignore_region` (hình 3D giống vật thật nhưng không di chuyển; tránh phạt nhầm false positive). |
| Tượng đặt trên xe thật | 028 | Xe thật gán bình thường (028: `truck`). Tượng phủ `ignore_region` riêng. Tượng **không** tính vào `occluded_level` của xe nếu nằm trên xe, không che thân xe. |
| Mô hình nhỏ / đồ chơi | 038 | Bỏ qua nếu nhỏ hơn vật thật rõ rệt và nằm trên vỉa hè / bàn / kệ; `ignore_region` nếu nằm dưới lòng đường gây nhiễu. |
| Người hóa trang | 011 | `person` bình thường (người thật đang di chuyển). |
| Xe diễu hành, xe độ / trang trí | 011, 019 | Có bánh và chạy được trên đường → gán theo hình dạng như xe thường (019 → `car`; 011 các xe máy độ → `motorcycle`). Không phải phương tiện (khung, mô hình trang trí không có bánh chạy được) → `ignore_region`. |

### R-06 — Phản chiếu và xe chủ (ego)
- Phản chiếu trên mặt đường ướt, vũng nước (013, 031): bỏ qua hoàn toàn.
- Phản chiếu trong gương chiếu hậu (020) hoặc kính cửa hàng: `ignore_region` phủ mặt gương / kính nếu bên trong thấy rõ xe hoặc người (ảnh bị đảo chiều, méo phối cảnh, sẽ làm sai hướng di chuyển).
- Xe chủ (040): phần đầu xe, ca-pô, gương hoặc thân của chính xe mang camera (hoặc xe đang chở người chụp) nằm cố định ở mép dưới / mép bên: không gán lớp phương tiện, phủ `ignore_region`.

### R-07 — Đám đông và bãi đỗ xe (008, 019, 021, 030, 032)
- Gán box riêng cho **mọi** vật thỏa **cả hai** điều kiện: (1) nhìn được biên để vẽ box riêng, **và** (2) bị che ≤ 70% (`occluded_level` 0–2).
- Không giới hạn số lượng: hàng đầu có bao nhiêu vật thỏa điều kiện thì gán bấy nhiêu.
- Phần còn lại của cụm (thường ở phía sau: che > 70% **hoặc** không tách được biên): một hoặc vài box `ignore_region` bao cả cụm.
- Không vẽ `ignore_region` đè lên các vật đã gán box riêng.

### R-08 — Vật nhỏ, ở xa (009, 025, 031, 034, 039)
- Ngưỡng `min_box_height_px = 20`, đo theo chiều cao box trên độ phân giải gốc.
- Vật đơn lẻ cao < 20 px: bỏ qua.
- Cụm nhiều vật < 20 px nằm sát nhau ở cuối đường / đường chân trời: một box `ignore_region` bao cụm.

### R-09 — Ảnh mờ, cắt mép, hàng hóa
- Mờ chuyển động (014): gán bình thường nếu vẫn chắc chắn loại xe và biên tương đối. Chỉ bỏ qua hoặc dùng `ignore_region` khi vệt nhòe không còn nhận ra là vật gì.
- Cắt mép (015, 040): ngưỡng `truncated` ở mục 2.
- Hàng chất cao trên xe (035): box xe (`motorcycle`, `auto_rickshaw`, `truck`) bao cả hàng cố định trên xe nếu hàng làm thay đổi biên dạng (cao, rộng) của xe. Ngoại lệ: vật quá mảnh như ống nước, thanh sắt thò ra quá 1 m phía sau thì để ngoài box, không vẽ thêm gì.

### R-10 — Mức che khuất
Thang ở mục 2.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| 005, 006, 010, 026 | Người lái và người ngồi sau trên xe máy | R-01 | 1 `motorcycle` + 1 `person` mỗi người | Người và xe là hai vật khác nhau |
| 009, 032 | Minibus (009), xe buýt (032) | R-02 | `bus` | Xe chở khách dài ≥ 1,5 lần ô tô con |
| 004, 016, 035 | Tuk-tuk, auto rickshaw (035 chở hàng chất cao) | R-02 | `auto_rickshaw` | 3 bánh, động cơ, có mui / khoang sau |
| 022 | Xe có thùng bên | R-02 | Một box `motorcycle` | Thùng gắn liền xe |
| 003, 029 | Xe đạp | R-02 | `bicycle` cho xe, `person` cho người | Xe thật, khác xe máy |
| 023, 039 | Xích lô | R-02 | `bicycle` cho xe, `person` cho người | Dùng sức người đạp |
| 001, 012, 028, 036 | Xe bán tải | R-03 | `truck` | Có thùng hàng riêng tách cabin |
| 028 | Tượng ngựa trên thùng xe bán tải | R-05 | Xe `truck`; tượng `ignore_region`; tượng không tính vào che khuất của xe | Xe thật, tượng không thật |
| 037 | Pickup có nắp thùng | R-03 | `truck` | Thấy khe chia cabin / thùng |
| 002 | Xe chở ô tô | R-04 | `truck` + `car` cho từng xe ≥ 20 px | Mỗi xe là một vật thật |
| 024 | Ô tô trên nóc ô tô | R-04 | Hai box riêng | Hai vật thật |
| 018, 026 | Người trên biển quảng cáo, tranh tường | R-05 | Bỏ qua | Hình 2D |
| 017, 027, 028, 033 | Tượng, ma-nơ-canh | R-05 | `ignore_region` | 3D giống thật, không di chuyển |
| 038 | Xe đồ chơi trong tủ kính | R-05 | Bỏ qua | Nhỏ hơn vật thật rõ rệt, trên kệ |
| 011 | Người hóa trang, xe máy độ | R-05 | `person`; xe máy độ: `motorcycle` | Người thật; xe có bánh, chạy được |
| 019 | Xe diễu hành dạng ô tô | R-05 | `car` | Có bánh, chạy được, hình dạng ô tô |
| 013, 031 | Phản chiếu mặt đường ướt | R-06 | Bỏ qua | Không phải vật |
| 020 | Gương chiếu hậu | R-06 | `ignore_region` phủ mặt gương | Ảnh đảo chiều, méo |
| 040 | Xe chủ ở tiền cảnh | R-06 | `ignore_region` | Vật tĩnh luôn bám camera |
| 008, 019, 021, 030, 032 | Đám đông, bãi xe | R-07 | Box riêng cho vật tách được biên và che ≤ 70%; phần còn lại `ignore_region` | Không tách được biên |
| 009, 025, 031, 034, 039 | Vật nhỏ ở xa | R-08 | < 20 px bỏ qua; cụm thì `ignore_region` | Ngưỡng taxonomy |
| 014 | Xe máy nhòe chuyển động | R-09 | Gán nếu nhận ra | Vẫn là vật thật |
| 015, 040 | Xe bị cắt mép | R-09 | `truncated` theo ngưỡng 10% | Đo được |
| 035 | Hàng chất cao trên xe | R-09 | Box xe bao cả hàng | Kích thước vật lý thật |

## 5. QC và phiên bản

### Thế nào là "đạt"? (ngưỡng lấy từ `taxonomy.json`, không đổi)

| Kiểm tra | Hiểu đơn giản | Ngưỡng đạt |
|---|---|---|
| Box khớp nhau (IoU) | Hai người vẽ cùng một vật, hai box trùng nhau bao nhiêu | Vật cao ≥ 50 px: ≥ 0,85 · vật nhỏ hơn: ≥ 0,7 |
| Đồng thuận giữa người gán (IAA) | Mọi người gán có giống nhau không | IoU trung bình ≥ 0,85 và cùng lớp ≥ 95% |
| Vật quá nhỏ | Cao dưới 20 px thì không gán từng vật | theo R-08 |
| Tỉ lệ ảnh lỗi | Số ảnh có ít nhất 1 lỗi / số ảnh đã duyệt | ≤ 2% (lô 40 ảnh: **0 ảnh lỗi**) |

Mọi con số luôn ghi kèm mẫu số, ví dụ `1/40`, để người đọc biết đã kiểm bao nhiêu.

### Ca mơ hồ thì ai quyết?
1. **Lúc gán:** không hỏi bạn cùng nhóm. Vẫn vẽ box theo cách hiểu tốt nhất, tick `can_review`. Không vẽ được thì ghi ảnh, vùng, lý do.
2. **Lúc họp calibration:** cả nhóm xem ca đó, người điều phối (346) chốt. Mỗi quyết định ghi một dòng: ca nào, quyết định gì, quy tắc nào (R-xx), ai chốt.
3. **Quyết định thành quy tắc mới:** ghi vào guideline và tăng phiên bản.

### Khi nào tăng phiên bản?
- **Sửa câu chữ**, không đổi cách gán → tăng nhỏ, không phải gán lại.
- **Đổi cách gán** (lớp, box, thuộc tính) → tăng phiên bản và **kiểm lại cả lô** theo quy tắc mới.

### Duyệt và bàn giao
- Duyệt **toàn bộ 40 ảnh**. Người duyệt khác người sửa nhãn.
- Thứ tự quyết định: **Hold** (thiếu bằng chứng, guideline chưa chốt) → **Rework** (> 2% ảnh lỗi, hoặc một lỗi lặp lại nhiều lần) → **Release** (đạt hết, người duyệt xác nhận).

### Lịch sử phiên bản

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| cá nhân | Viết lại từ v0: thêm R-01…R-10, đổi thang `occluded_level`; pickup → `truck` | Các ca ở mục 4 |
| cá nhân (sửa) | Thêm 3 lớp `bus`, `bicycle`, `auto_rickshaw`; không thêm lớp xe ba gác chở hàng (không có mẫu) | 009, 032; 003, 029, 023, 039; 004, 016, 035 |
