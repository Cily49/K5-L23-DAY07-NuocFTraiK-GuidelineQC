# Thông tin nhóm

- Tên nhóm: NuocFTraiK
- Mã thành viên (ba chữ số): 346, 187, 276, 234, 274
- Người điều phối: 346
- Người review bản cuối: 346 (duyệt với hỗ trợ của Claude)
- Commit chốt bài (điền ở kênh nộp sau khi commit): 76c5f34

## Kết quả tóm tắt (điền cuối bài)

- Lượt gán hoàn tất / được giao: 120/120
- IAA trước calibration (kèm mẫu số): IoU TB 0,822, cùng lớp 94,2% trên 992 cặp box ghép — chưa đạt
- Số ảnh phải calibration: 36 / 40 (v1.1 semantic → kiểm lại cả 40)
- Vòng QC và số ảnh đã duyệt / cả lô: vòng 1, 40/40
- Tỉ lệ lỗi: 52,5 % theo ảnh (21/40), 3,9 % theo đối tượng (22/558)
- **Quyết định** (Release / Hold / Rework) và lý do một dòng: **Hold** — IAA chưa đạt, guideline v1.1 đã chốt nhưng nhãn chưa sửa theo v1.1, QC vòng 1 vượt ngưỡng 2%.
