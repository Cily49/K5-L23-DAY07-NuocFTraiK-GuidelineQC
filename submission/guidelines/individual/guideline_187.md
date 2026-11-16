# Guideline gán nhãn — cá nhân (sửa từ v0) — 187

> Bản nháp cá nhân cho Pha 1. Chưa phải quyết định của nhóm; sẽ đối chiếu ở Pha 2.

- Phiên bản: v0-cá nhân
- Ngày: 2026-11-16
- Người viết / duyệt: 187

## 1. Mục tiêu
Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện** trên ảnh đường phố 2D (camera giao thông, xe tự lái).

Sai ở đâu thì nguy hiểm:
- **Gán nhầm vật không thật** (tượng, ma-nơ-canh, hình in, phản chiếu): mô hình học báo động giả, có thể phanh gấp vì một bức tượng.
- **Bỏ sót người thật** (nhất là người ngồi trên xe máy, người bị che một phần): lỗi nặng nhất vì liên quan an toàn người đi đường.

Phép thử chung: *"mô hình có phải đoán ra điều này không?"* Chỉ gán những gì nhìn thấy được trong ảnh.

## 2. Lớp và thuộc tính
### Lớp
Mỗi vật thuộc đúng một lớp. Dùng 6 lớp trong taxonomy v0 và thêm 2 lớp đề xuất `bus` và `rudimentary_vehicle`, tổng 8 lớp. Ngoài các lớp này **không tự thêm lớp mới**.

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con, SUV, taxi bốn bánh. | img_023 (sedan đen) | Xe bán tải có thùng hở → `truck`. Xe thùng kín nhỏ → `van`. Tuk-tuk → `rudimentary_vehicle`. |
| `truck` | Xe tải, bán tải có thùng hàng hở, xe đầu kéo. | img_002, img_012, img_036 | Xe thùng kín cỡ nhỏ → `van`. |
| `van` | Xe thùng kín cỡ nhỏ chở người hoặc hàng, cửa hông trượt. | img_005 (xe van trắng) | Xe buýt, xe khách lớn → `bus`. |
| `motorcycle` | Xe máy, mô tô hai bánh có động cơ, **chỉ phần thân xe**, không gồm người ngồi. | img_006, img_021 | Xe đạp, xích lô, tuk-tuk → `rudimentary_vehicle`. |
| `bus` | Xe buýt, xe khách, xe buýt nhỏ (minibus) chở nhiều hành khách, có nhiều hàng ghế và nhiều cửa sổ dọc thân. | img_009 (xe buýt nhỏ) | Xe thùng kín nhỏ cửa trượt → `van`. |
| `rudimentary_vehicle` | Xe thô sơ: xe đạp, xích lô, xe lăn, tuk-tuk (xe ba bánh). **Chỉ phần thân xe**, không gồm người ngồi. | img_004, img_016, img_023, img_035 | Xe máy hai bánh → `motorcycle`. Xe bốn bánh → `car`. |
| `person` | Người thật đi bộ hoặc ngồi trên xe (kể cả người lái và người chở sau). | img_014, img_034 | Tượng, ma-nơ-canh, hình in, ảnh trên áp phích: **không** phải `person`. |
| `ignore_region` | Vùng không gán từng vật (đám đông chen không tách được, cụm vật nhỏ xa). | phần xa trong img_021 | Không dùng để né vật rõ ràng. |

**Vật không thuộc lớp nào** trong 8 lớp trên: không gán box và không tạo lớp mới. Ghi vào `image_issues.csv` (ảnh, vùng, lý do). Người trên xe vẫn luôn gán `person` riêng (R-02).

### Thuộc tính
**`occluded_level`** (tỉ lệ phần vật bị che, tính trên phần ước lượng của cả vật):

| Mức | Che | Ghi chú |
|---|---|---|
| `0` | dưới 5% | thấy gần đủ |
| `1` | 5% đến dưới 30% | che ít |
| `2` | 30% đến 70% | che nhiều |
| `3` | trên 70% | chỉ gán nếu **vẫn nhận ra được lớp**; không nhận ra thì không gán |

**`truncated`**: `yes` khi vật bị **mép ảnh** cắt và phần mất ước tính từ 10% trở lên; còn lại `no`. Che bởi vật khác là `occluded_level`, không phải `truncated`.

**`can_review`**: tick khi tôi không chắc lớp, có gán hay không, hoặc mức che. Không tick để né quyết định rõ ràng.

## 3. Quy tắc vẽ box
- **R-01 Box ôm phần nhìn thấy.** Box sát phần nhìn thấy của vật, không đoán phần bị che hay cắt.
- **R-02 Người trên xe máy / xe thô sơ = box xe + box người.** Một `motorcycle` hoặc `rudimentary_vehicle` (xe đạp, xích lô, xe lăn, tuk-tuk) bao thân xe, một `person` cho **mỗi người** (người lái, người chở, khách). Các box được chồng nhau.
- **R-03 Hàng hóa và đồ mang theo không tính vào box.** Gánh hàng, thanh dài, bao tải thò ra ngoài thân xe hoặc thân người không được bao. Hàng nằm trong thùng xe thì đã nằm trong box xe.
- **R-04 Xe chở xe.** Gán 1 box cho xe chở (`truck`) và box `car` riêng cho mỗi ô tô trên xe còn nhìn rõ. Ô tô không nhận ra lớp thì không gán.
- **R-05 Vật không thật không gán.** Tượng, ma-nơ-canh, đồ chơi, mô hình, hình in, người trên áp phích hoặc biển quảng cáo.
- **R-06 Phản chiếu không gán.** Hình phản chiếu trên gương, kính, mặt nước, mặt đường ướt.
- **R-07 Xe thật dạng lạ vẫn gán.** Xe diễu hành, xe độ chạy được có người lái: gán theo lớp gần nhất và tick `can_review`.
- **R-08 Vật quá nhỏ.** Box cao dưới **20 px** thì không gán (theo `min_box_height_px` trong taxonomy).
- **R-09 Đám đông và cụm chen chúc.** Gán từng vật tách được và cao từ 20 px. Phần không tách được hoặc quá nhỏ vẽ một `ignore_region`.
- **R-10 Xe chủ (ego).** Phần xe chủ lọt vào ảnh (gương chiếu hậu, mũi xe) không gán.
- **R-11 Nhòe chuyển động.** Gán nếu vẫn nhận ra lớp; không nhận ra thì không gán, và vùng nhòe lớn có thể là `ignore_region`.
- **R-12 Ảnh không có vật nào cần gán.** Vẫn giữ ảnh trong Task (file COCO phải có ảnh đó).

## 4. Ca khó
| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_002 | Xe đầu kéo chở 7–8 ô tô, ô tô che nhau | R-04, R-01 | 1 `truck` + `car` cho từng ô tô nhìn rõ; ô tô che gần hết không gán | Mô hình cần đếm cả xe chở lẫn xe được chở |
| img_006 | Người ngồi trên xe máy, người chở sau bị che | R-02, R-01 | `motorcycle` + 2 `person` | Mô hình đếm người và xe riêng |
| img_027 | Mô hình xe tải bằng đá trên mộ | R-05 | Không gán | Không phải xe thật |
| img_017, img_028 | Tượng người cưỡi ngựa; tượng ngựa đặt trên xe tải cổ | R-05 | Không gán tượng; xe tải cổ thật ở img_028 gán `truck` | Tượng không phải người thật |
| img_033 | Ma-nơ-canh trong tủ kính | R-05, R-06 | Không gán | Không phải người, kính còn phản chiếu |
| img_038 | Hàng chục xe đồ chơi trong tủ kính | R-05 | Không gán | Đồ chơi |
| img_019 | Xe giả tưởng màu vàng có người lái, chuột bông, đám đông xem | R-07, R-09, R-05 | `car` + `person` lái (tick `can_review`); chuột bông không gán; người xem gán theo R-09 | Xe chạy được có người lái nhưng hình dạng lạ |
| img_020 | Gương chiếu hậu phản chiếu đường | R-06, R-10 | Không gán | Phản chiếu và xe chủ |
| img_018 | Người in trên biển quảng cáo | R-05 | Không gán | Hình in |
| img_004, img_016, img_035 | Tuk-tuk, xe ba bánh | `rudimentary_vehicle`, R-02 | Gán `rudimentary_vehicle` cho thân xe; người lái và khách gán `person` riêng | Tuk-tuk là xe thô sơ, không gộp vào `car` |
| img_023 | Xích lô có người đạp | `rudimentary_vehicle`, R-02 | Gán `rudimentary_vehicle` cho xích lô; gán `person` người đạp | Xích lô là xe thô sơ |
| img_009 | Xe buýt nhỏ | `bus` | Gán `bus` | Xe chở nhiều khách, nhiều hàng ghế |
| img_029 | Người chở thanh dài trên xe máy | R-02, R-03 | `motorcycle` + `person`; thanh dài không bao | Hàng thò ra ngoài |
| img_035 | Xe ba bánh chất hàng rất cao | R-03 | Box `rudimentary_vehicle` chỉ bao thân xe, hàng thò cao không bao | Hàng hóa |
| img_021, img_030 | Hàng chục xe máy chen nhau | R-09, R-08 | Gán xe và người tách được; phần chen và nhỏ vẽ `ignore_region` | Không tách được từng vật |
| img_032 | Nhóm người đứng xa, nhỏ | R-08, R-09 | Gán người cao từ 20 px; còn lại `ignore_region` | Quá nhỏ |
| img_014 | Người đứng rõ, xe phía sau nhòe | R-11 | Gán 2 `person`; xe nhòe không nhận ra thì không gán | Nhòe chuyển động |
| img_005, img_040 | Người đội mũ, che mặt, xe bị cắt mép | R-01, `truncated` | Gán theo phần nhìn thấy; `truncated=yes` nếu bị cắt trên 10% | Che và cắt mép |
| img_025, img_013, img_039 | Ít hoặc không có vật cần gán | R-12 | Giữ ảnh, chỉ gán vật rõ ràng | Ảnh vẫn phải có trong COCO |

Chưa chốt: tick `can_review` trên box hoặc ghi vấn đề cấp ảnh.

## 5. QC và phiên bản
Ngưỡng QC lấy từ `rules` của `taxonomy.json`, không đổi:
- Box cao tối thiểu 20 px.
- IAA: IoU 0.85 (ghép cặp ở 0.5), đồng thuận lớp 0.95.
- Tỉ lệ ảnh lỗi tối đa 2%.

Ca mơ hồ: tôi tick `can_review`; nhóm họp Pha 2 và Pha 5 để chốt, người chốt ghi vào `calibration_decisions.csv`.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0-cá nhân | Viết lại từ v0: thêm lớp rõ ràng, ngưỡng thuộc tính, quy tắc R-01 đến R-12 | v0 không trả lời được img_002, img_006, img_027 |
| v0.1-cá nhân | Tách tuk-tuk khỏi `car` thành lớp riêng `rudimentary_vehicle` | img_004, img_016, img_035 |
| v0.2-cá nhân | `rudimentary_vehicle` gồm cả xe đạp, xích lô, xe lăn; thêm lớp `bus` | img_023 (xích lô), img_009 (xe buýt nhỏ) |
