# Guideline gán nhãn — <phiên bản> — <mã hoặc nhóm>

> Copy file này (hoặc `guideline_v0_starter.md`) rồi viết. Một guideline đủ gồm **5 phần**. Câu nào của v0
> làm hai người đọc ra hai cách khác nhau thì phải viết lại cho một cách hiểu duy nhất.

- Phiên bản: v2.0 - cá nhân
- Ngày: 16/11/2026
- Người viết / duyệt: 346 / (chưa duyệt)

## 1. Mục tiêu
Nhãn dùng để huấn luyện mô hình phát hiện vật 2D (bounding box) cho phương tiện và người trên ảnh đường phố.
Mô hình cần phát hiện vật thật đang tham gia giao thông, phục vụ hệ thống hỗ trợ lái xe / giám sát giao thông.
Sai nguy hiểm nhất: (1) bỏ sót người hoặc xe thật → mô hình không thấy vật cản; (2) gán nhầm vật giả (tượng, ma-nơ-canh, ảnh in, phản chiếu) thành vật thật → mô hình báo động sai.

## 2. Lớp và thuộc tính
### Lớp
| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| car | Ô tô con chở người (sedan, SUV, hatchback, taxi), tối đa ~9 chỗ. | img_015 | Xe đồ chơi/mô hình (img_038), tượng xe (img_027) |
| truck | Xe chở hàng có cabin tách rời thùng hàng, kể cả xe bán tải (có hoặc không có nắp thùng), xe đầu kéo và xe chở ô tô. | img_002, img_037 | Xe thùng kín liền khối cabin → `van` |
| van | Xe thùng kín liền khối cabin, chở hàng hoặc chở khách, tối đa 16 chỗ (minivan, xe 16 chỗ). | img_005 | Xe chở khách trên 16 chỗ → `bus` |
| bus | Xe chở khách cỡ lớn trên 16 chỗ (xe buýt thành phố, xe khách, xe du lịch). | img_032, img_037 (xe buýt phía sau) | Xe 16 chỗ trở xuống → `van`; xe tải thùng kín có cabin tách rời → `truck` |
| motorcycle | Xe hai bánh có động cơ (xe máy, xe tay ga, mô tô), kể cả mô tô gắn thùng sidecar. | img_026, img_022 | Xe đạp, xích lô không động cơ (img_006, img_029, img_039); xe 3 bánh có mui → `three_wheeler` |
| three_wheeler | Xe ba bánh có động cơ, có mui, chở khách hoặc hàng (tuk-tuk, auto rickshaw, xe lam). | img_004, img_035 | Xích lô đạp không động cơ (img_006) → không gán; mô tô sidecar (img_022) → `motorcycle` |
| person | Người thật, đứng, đi bộ hoặc đang ngồi trên xe. | img_001 | Tượng (img_011, img_017), ma-nơ-canh (img_033), ảnh người trên biển quảng cáo (img_018) → không gán |
| ignore_region | Vùng bỏ qua (không phải vật): cụm vật quá nhỏ (cao < 20 px) hoặc đám đông dày không tách được từng cá thể; bỏ sót vật bên trong vùng này không tính là lỗi. | img_019 (đám đông sau rào chắn) | Người/xe tách được và cao ≥ 20 px → vẽ box riêng theo lớp; vật giả → không gán, không dùng `ignore_region` |

Vật **không thuộc lớp nào**:
- Vật giả (tượng, đồ chơi, ma-nơ-canh, hình in, phản chiếu trong gương/kính) → **không gán**.
- Xe không động cơ (xe đạp, xích lô) → **không gán**.
- Phương tiện thật không khớp lớp nào ở trên → tick `can_review` và ghi vào bảng Ca khó.

### Thuộc tính
| Thuộc tính | Các mức | Ngưỡng cụ thể |
|---|---|---|
| occluded_level | 0, 1, 2, 3 | 0 = không bị che; 1 = che 1–30% diện tích vật; 2 = che 31–70%; 3 = che > 70% nhưng vẫn nhận dạng chắc chắn lớp. Nếu che quá nhiều không nhận ra lớp → không gán. |
| truncated | true / false | true khi một phần vật nằm ngoài mép ảnh (box chạm mép ảnh và vật còn tiếp tục ra ngoài). |
| can_review | true / false | true khi người gán không chắc lớp hoặc phạm vi box theo guideline hiện tại; phải ghi ca đó vào bảng Ca khó. |

## 3. Quy tắc vẽ box
- **R-01 Ôm sát phần nhìn thấy:** box ôm sát mép ngoài cùng của phần vật nhìn thấy được, không đoán phần bị che.
- **R-02 Vật bị che:** vẫn gán nếu nhận dạng chắc chắn lớp; chọn `occluded_level` theo % bị che.
- **R-03 Cắt mép ảnh:** box dừng ở mép ảnh, đặt `truncated = true`.
- **R-04 Vật giả:** không gán tượng, đồ chơi, ma-nơ-canh, hình in trên biển quảng cáo, phản chiếu qua gương/kính.
- **R-05 Xe chủ (ego):** không gán phần đầu/capo xe của chính xe đang quay.
- **R-06 Vật nhỏ:** vật cao < 20 px không vẽ box riêng; nhiều vật nhỏ gần nhau → một `ignore_region`.
- **R-07 Người đi xe máy:** 1 box `motorcycle` ôm cả người và xe.
- **R-08 Xe chở xe (img_002):** 1 box `truck` + từng box `car` cho ô tô trên sàn.
- **R-09 Hàng hóa chất cao (tuk-tuk img_035):** box `three_wheeler` chỉ ôm thân xe, không ôm phần hàng chất nhô lên trên.

## 4. Ca khó
| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_002 | Xe tải chở 8 ô tô trên 2 sàn | R-08 | Gán box car cho xe car, box truck cho xe chở car | Car và truck đều có trong ảnh nên phải gán label cho cả 2 |
| img_004 | Hai xe tuk-tuk 3 bánh đỗ cạnh nhau | Định nghĩa `three_wheeler`, R-02 | Gán mỗi xe 1 box `three_wheeler`; xe phía sau bị che chọn `occluded_level` theo % bị che | Tuk-tuk (3 bánh, có mui, chở khách) không khớp `car` hay `motorcycle`; ép vào lớp gần nhất sẽ làm bẩn định nghĩa hai lớp đó, nên tách thành lớp riêng |
| img_017 | Tượng người | R-04 | Không gán | Không phải người thật |
| img_020 | Phản chiếu qua gương chiếu hậu | R-04 | Không gán | Vật phản chiếu |
| img_037 | Bán tải gắn nắp thùng cao (truck cap) | Định nghĩa `truck` | Gán `truck` | Cabin vẫn tách rời thùng hàng (thấy rõ khe giữa cabin và nắp thùng); nắp thùng chỉ là phụ kiện gắn thêm, không biến xe thành `van` liền khối. Mọi xe bán tải (img_001, img_012, img_028) đều gán `truck` để nhất quán |
| img_028 | Bán tải cổ chở tượng ngựa; ô tô đỏ phía sau bị che | R-04, R-02 | Gán `truck` cho bán tải; tượng ngựa không gán; ô tô đỏ gán `car`, `occluded_level` = 2 | Tượng là vật giả; ô tô đỏ vẫn nhận ra chắc chắn là `car` dù bị che khoảng 40–60% |
| img_019 | Xe diễu hành hình hoạt hình, có người lái thật và mascot chuột | R-04, Mục tiêu | Gán `car` cho xe; mascot không gán; đám đông dày sau rào chắn → 1 `ignore_region`; tick `can_review` | Xe có biển số, đang chạy trên đường nên là phương tiện thật dù hình dáng lạ; mascot là vật giả; guideline chưa nói người ngồi trong ô tô có vẽ `person` riêng hay không |
| img_022 | Mô tô kèm thùng sidecar có người ngồi | R-07 | Gán 1 box `motorcycle` ôm cả mô tô, thùng sidecar và người ngồi | Mở rộng R-07: sidecar là một phần của phương tiện, di chuyển cùng mô tô như một khối; tách riêng sẽ ra box không thuộc lớp nào |

Nếu chưa chốt, đánh dấu `can_review` trên box hoặc ghi vấn đề cấp ảnh vào `submission/qc/image_issues.csv`. Dùng mã quy tắc trong nhật ký QC.


## 5. QC và phiên bản
### Ngưỡng QC áp dụng
| Đại lượng | Ngưỡng | Ghi chú |
|---|---|---|
| IoU khớp box | box cao ≥ 50 px: ≥ 0.85; box cao < 50 px: ≥ 0.7 | Dưới ngưỡng → lỗi `sai_vi_tri_kich_thuoc` |
| IAA | IoU trung bình ≥ 0.85 và cùng lớp ≥ 95% | Chỉ tính trên cặp box ghép 1–1 (IoU ghép ≥ 0.5); chỉ đo khi độ phủ `assignment.csv` đạt 100% |
| Vật nhỏ | cao < 20 px | Không vẽ box riêng, dùng `ignore_region` (R-06) |
| Tỉ lệ ảnh lỗi | ≤ 2% số ảnh đã duyệt | Lô 40 ảnh: chỉ cần 1 ảnh lỗi (2,5%) là vượt ngưỡng |

- Mọi số đo ghi dạng `tử số/mẫu số`; không có cặp box thì ghi N/A. Giữ số IAA **trước** calibration trong hồ sơ.
- Lỗi ghi vào `qc_review.csv` theo 5 loại: `bo_sot`, `thua`, `sai_lop`, `sai_vi_tri_kich_thuoc`, `sai_thuoc_tinh` (ảnh sạch ghi `khong`), kèm mã quy tắc (R-xx) bị vi phạm.
- Quyết định lô: **Hold** khi thiếu bằng chứng, guideline còn ca chưa chốt, review lệch phiên bản hoặc mới chỉ lấy mẫu; **Rework** khi > 2% ảnh lỗi hoặc có lỗi lặp cùng một kiểu (lỗi hệ thống); **Release** khi đã duyệt toàn bộ lô và reviewer xác nhận đạt.

### Ai quyết ca mơ hồ
1. Người gán gặp ca guideline chưa trả lời được → tick `can_review` trên box (hoặc ghi vào `submission/qc/image_issues.csv` nếu là vấn đề cấp ảnh), **không tự đặt quy tắc mới**.
2. Cuối mỗi checkpoint, leader nhóm gom các ca `can_review` và ca lệch trong `calibration_candidates.csv`, cả nhóm thảo luận; leader chốt quyết định cuối cùng.
3. Phân loại nguyên nhân lệch: cả nhóm lệch cùng kiểu ca → quy tắc mơ hồ, **sửa guideline**; chỉ một người lệch → hướng dẫn lại người đó; mọi người giống nhau nhưng cùng sai → đối chiếu ca tham chiếu.
4. Quyết định đã chốt được ghi vào bảng Ca khó (mục 4) kèm mã quy tắc. Thay đổi nào làm đổi nhãn đã gán thì **tăng phiên bản** và gán lại các ảnh bị ảnh hưởng.

### Lịch sử thay đổi
| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của lab: 5 lớp, 3 thuộc tính, quy tắc chung | — |
| v1.0 | Viết lại định nghĩa từng lớp kèm ví dụ / phản ví dụ; định lượng `occluded_level` theo % bị che; thêm R-01 → R-09 | v0 làm hai người đọc ra hai cách: vật giả (img_017, img_020, img_033), xe chở xe (img_002), người đi xe máy (img_026) |
| v1.0 | Bán tải (kể cả có nắp thùng) gán `truck`; mô tô sidecar gộp 1 box `motorcycle` | img_037, img_028, img_022 |
| v1.0 | Thêm lớp `bus` (> 16 chỗ, tách khỏi `van`) và `three_wheeler` (xe 3 bánh có động cơ, có mui); đưa `ignore_region` vào bảng lớp với định nghĩa và phản ví dụ | img_004, img_035 (tuk-tuk), img_032, img_037 (xe buýt), img_019 (đám đông) |
| (chờ chốt) | Người ngồi trong ô tô có vẽ `person` riêng hay không | img_019 |
