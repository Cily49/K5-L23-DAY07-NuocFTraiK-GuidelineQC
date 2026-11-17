# Báo cáo QC — NuocFTraiK

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Thực hành đặc tả và QC nhãn bounding box 2D (người và phương tiện) trên 40 ảnh đường phố của Day 07. Tập ảnh được tuyển chọn để luyện ca biên, chưa đánh giá độ đại diện cho dữ liệu vận hành. Nhãn ở trạng thái **Hold**: không dùng để huấn luyện hay đánh giá mô hình.
- Nguồn và phép sử dụng dữ liệu: `data/ATTRIBUTION.csv` (tác giả, giấy phép CC, URL nguồn cho cả 40 ảnh); `batch_manifest.json` mang theo nguồn và SHA-256 từng ảnh. Giữ ghi công khi chia sẻ.
- Manifest và checksum nhãn cuối: `submission/qc/batch_manifest.json`; `submission/qc/labels_calibrated.json` SHA-256 `a505fd8a3921ada47fc8d36b59669b4e3b8f2f3d27dcc49cc5749865396b2dea` (40 ảnh, 558 annotation).
- Guideline và taxonomy khi gán / khi bàn giao: Gán độc lập theo `guideline_v1.0_agreed.md` + `taxonomy.json` v1.0 (10 lớp). Bàn giao kèm `guideline_v1.1_calibrated.md` (thay đổi `semantic`, xem `submission/guideline_change.json`); taxonomy giữ v1.0, mục `rules` không đổi. **Nhãn bàn giao chưa được sửa theo v1.1.**

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 120/120 (40 ảnh × 3 người, 5 thành viên 187, 234, 274, 276, 346; `submission/qc/assignment.csv`).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 992 cặp box ghép (IoU ≥ 0,5) trên 120 lượt cặp-ảnh; IoU TB **0,822** (ngưỡng 0,85); cùng lớp **94,2%** (ngưỡng 95%) → **chưa đạt IAA**. Cùng `occluded_level` 60,8%, cùng `truncated` 94,6%. Cặp có 187 thấp nhất (IoU 0,763–0,789), các cặp còn lại 0,848–0,878 (`iaa_summary.md`).
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 263/271 box chưa ghép (A/B); 0 box loại do ignore vì chỉ 276 vẽ `ignore_region` (3 vùng); 6 ca bất đồng vùng ignore (img_011, img_019). Tổng 1492 ca trên 36/40 ảnh: bỏ sót/thừa 534, IoU thấp 469, thuộc tính 423, sai lớp 58, bất đồng ignore 6, `can_review` 2.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: img_002 — nhiều người cùng vẽ `car` riêng cho ô tô trên xe chở, trái R-04 (một box `truck` cho cả khối); IAA không báo vì các box này ghép khớp nhau. Kiểm bằng cách đối chiếu nhãn với R-04 khi duyệt QC (phiếu vòng 1: 8 box `thua`). Tương tự img_028 (xe bán tải cổ gán `car`, trái R-03).

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 1492/1492 ca có quyết định trong `submission/qc/calibration_decisions.csv` (người chốt 346, phiên bản v1.1): R-09 344 ca (`lo`), R-02 27 ca (`lo`), R-01 469, 2.2 423, R-08 196, R-03/2.1/nhầm nhãn 33 (`anh`). **Chưa có bản sửa nhãn và chưa kiểm lại** (cột `phieu_sua`, `kiem_lai` ghi rõ) do hết thời gian.
- Tác động v1.1 lên nhãn và ảnh: Thay đổi `semantic` → phạm vi cả lô 40/40 ảnh. COCO "sau sửa" `labels_calibrated_90d894a0acd306f1.json` là bản đồng thuận nháp export nguyên từ CVAT, không chỉnh sửa. Không đổi taxonomy nên không cần mapping lớp.
- Tham chiếu bằng chứng khi Rework: Không áp dụng (chưa mở vòng Rework). Khi làm lại: sửa theo `calibration_decisions.csv`, chạy lại `prepare_calibration.py` → `build_final.py`, mở `--round 2`, giữ `reviews/round-01/`.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 346, 2026-11-17 (duyệt với hỗ trợ của Claude trên ảnh QC đã thu nhỏ; 346 chịu trách nhiệm xác nhận; không ai sửa nhãn nên không trùng vai người sửa)
- Đã duyệt / cả lô: 40/40 (duyệt toàn bộ)
- Ảnh lỗi / ảnh đã duyệt: 21/40 = 52,5% (ngưỡng 2%)
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 22/558 = 3,9% (thừa 11, sai lớp 11)
- Vật bỏ sót, ảnh và vùng: 9 — thiếu `ignore_region` cho cụm đông ở img_003, img_006, img_008, img_019, img_021, img_030, img_032, img_034 (R-09); xe máy độ bằng sắt tiền cảnh img_011 (R-05).
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Ca khó 21/36 ảnh lỗi; phần còn lại 0/4. Lỗi lặp: (1) `person` thay vì `rider` cho người trên xe — img_004 #51, img_005 #71, img_010 #156, img_016 #205, img_026 #377 (R-02); (2) xe bán tải gán `car` — img_012 #184, img_016 #203, img_028 #378 (R-03); (3) xe chở xe vẽ `car` riêng — img_002 (8 box), img_024 #347/#353 (R-04); (4) thiếu `ignore_region` ở 8 ảnh đông (R-09). Phiếu: `reviews/round-01/qc_review.csv`. Giới hạn: độ ôm sát box và `occluded_level`/`truncated` chưa được kiểm chi tiết; nhãn cuối còn box `can_review`.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy chặn: `validate_labels.py` cảnh báo box cao < 20 px (346.json: img_025 #112 cao 19,6 px, img_039 #219 cao 14 px) → đã xóa trước khi nộp nhãn. Cần người: img_004 #50 — box người lái tuk-tuk mang lớp `car`; hợp lệ về schema, chỉ nhìn ảnh mới thấy sai (R-02).

## 5. Quyết định bàn giao

- Quyết định: Hold
- Lý do, người nhận và phạm vi sử dụng: IAA vòng gán độc lập chưa đạt (IoU 0,822, lớp 94,2% / 992 cặp); guideline v1.1 đã chốt nhưng nhãn chưa sửa và chưa kiểm lại theo v1.1; QC vòng 1 có 21/40 ảnh lỗi (52,5% > 2%). Người nhận: coach Day 07, chỉ dùng để chấm quy trình đặc tả / IAA / calibration — không dùng nhãn cho huấn luyện.
- Nếu còn việc: (1) Sửa nhãn cả lô theo v1.1 trong CVAT (Task calibration 40 ảnh, chia 5 Job) — người xử lý: 5 thành viên; (2) kiểm lại từng Job bởi người khác người sửa, điền `kiem_lai`; (3) chạy lại `prepare_calibration.py`, `build_final.py`, QC `--round 2` và kiểm riêng độ ôm sát box + thuộc tính. Bằng chứng cần kiểm lại: phiếu vòng 2, tỉ lệ ảnh lỗi ≤ 2%.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ca C165 (`calibration_candidates.csv`: img_004, 274 `person` vs 346 `car`) → quy tắc R-02 v1.1 (`guideline_v1.1_calibrated.md`, người được xe đỡ là `rider`) → nhãn cuối `labels_calibrated.json` annotation #50 lớp `car` tại img_004 → phiếu `reviews/round-01/qc_review.csv` dòng img_004 #50 `sai_lop` R-02 → ảnh `work/review/round-01/img_004.jpg.png`. Kết quả: lần theo được đủ chuỗi; lỗi còn nguyên vì chưa sửa (khớp quyết định Hold).
