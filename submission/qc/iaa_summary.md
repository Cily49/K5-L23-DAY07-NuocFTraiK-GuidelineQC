# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 5
- Lượt gán đủ **120/120**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 992 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 120 | 992 | 0.822 | 94.2% | 60.8% | 94.6% | 263/271 | 0 | CHƯA ĐẠT |
| 187|234 | 13 | 89 | 0.766 | 92.1% | 56.2% | 97.8% | 40/6 | 0 | CHƯA ĐẠT |
| 187|274 | 12 | 134 | 0.772 | 97.0% | 43.3% | 95.5% | 53/32 | 0 | CHƯA ĐẠT |
| 187|276 | 12 | 118 | 0.763 | 97.5% | 61.9% | 90.7% | 32/30 | 0 | CHƯA ĐẠT |
| 187|346 | 11 | 85 | 0.789 | 94.1% | 52.9% | 95.3% | 27/10 | 0 | CHƯA ĐẠT |
| 234|274 | 13 | 99 | 0.855 | 94.9% | 67.7% | 93.9% | 7/32 | 0 | CHƯA ĐẠT |
| 234|276 | 11 | 87 | 0.858 | 94.3% | 63.2% | 90.8% | 8/23 | 0 | CHƯA ĐẠT |
| 234|346 | 11 | 65 | 0.848 | 93.8% | 67.7% | 98.5% | 1/23 | 0 | CHƯA ĐẠT |
| 274|276 | 11 | 90 | 0.853 | 93.3% | 57.8% | 92.2% | 28/57 | 0 | CHƯA ĐẠT |
| 274|346 | 12 | 95 | 0.855 | 91.6% | 60.0% | 94.7% | 18/33 | 0 | CHƯA ĐẠT |
| 276|346 | 14 | 130 | 0.878 | 91.5% | 78.5% | 96.9% | 49/25 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 1492 ({'bo_sot_thua': 534, 'iou_thap': 469, 'sai_thuoc_tinh': 423, 'sai_lop': 58, 'co_can_review': 2, 'bat_dong_ignore': 6}), trên 36 ảnh cần calibration.
