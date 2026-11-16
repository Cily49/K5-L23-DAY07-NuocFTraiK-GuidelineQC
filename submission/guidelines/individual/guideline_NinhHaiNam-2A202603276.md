# Guideline gán nhãn — bản cá nhân v0.1 — <mã thành viên>

- Phiên bản: v0.1 (bản cá nhân)
- Ngày: 2026-11-16
- Người viết / duyệt: <mã thành viên> / chưa duyệt

## 1. Mục tiêu

Gán nhãn người và phương tiện có thật trong ảnh đường phố để dùng huấn luyện mô hình nhận diện. Cần làm nhất quán giữa các ảnh. Không gán tượng, hình ảnh, mannequin hoặc đồ chơi thành người/phương tiện thật. Bộ ảnh có cảnh đông, xe chở hàng và nhiều vật dễ nhầm, nên gặp trường hợp chưa chắc thì ghi lại để nhóm xem xét.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa ngắn | Ví dụ | Dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở người. | Sedan, SUV, taxi. | Xe tải, xe van, ô tô đồ chơi. |
| `truck` | Xe lớn chủ yếu dùng chở hàng. | Xe tải thùng, xe chở ô tô. | Xe van, xe máy chở hàng. |
| `van` | Xe thân kín dùng chở người hoặc hàng. | Minivan, xe van giao hàng. | Ô tô con hoặc xe tải nhỏ. |
| `motorcycle` | Xe máy hoặc scooter hai bánh. | Xe máy đang chạy hoặc đỗ. | Xe đạp, xe ba bánh/tuk-tuk. |
| `three_wheeler` (đề xuất) | Phương tiện cơ giới có ba bánh, dùng chở người hoặc hàng. | Tuk-tuk, xe lam/xe ba bánh có động cơ. | Xe máy hai bánh, xe đạp ba bánh không động cơ. |
| `person` | Người thật trong ảnh, kể cả người ngồi trên xe. | Người đi bộ, người lái xe. | Tượng, mannequin, hình người trên ảnh. |
| `ignore_region` | Vùng khó tách thành từng vật riêng. | Một cụm quá đông, không thể gán rõ từng vật. | Không dùng thay cho nhãn của một vật riêng lẻ chỉ vì khó nhìn. |

Nếu vật là đồ chơi, tượng, mannequin hoặc hình ảnh thì không gán như vật thật. Lớp `three_wheeler` là đề xuất cá nhân vì bộ ảnh có tuk-tuk/xe ba bánh. Chỉ dùng lớp này nếu nhóm đồng ý và cập nhật taxonomy chung; trước khi thống nhất, đánh dấu các trường hợp đó `can_review`. Với vật không khớp lớp nào khác, không tự tạo lớp mới.

### Thuộc tính

| Thuộc tính | Giá trị | Cách chọn |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Ước lượng phần vật bị vật khác che: `0` = 0–10%; `1` = trên 10–25%; `2` = trên 25–50%; `3` = trên 50% nhưng vẫn nhận ra được. Đây là ngưỡng đề xuất để nhóm thống nhất. |
| `truncated` | `no`, `yes` | Chọn `yes` nếu vật bị khung ảnh cắt ở mép; nếu không thì `no`. Bị vật khác che không tính là bị cắt mép ảnh. |
| `can_review` | tick / không tick | Tick nếu bạn chưa chắc về lớp hoặc có nên gán vật đó hay không; ghi lý do để nhóm xem lại. |
## 3. Quy tắc vẽ box

1. Gán box cho mọi người và phương tiện trong ảnh.
2. Box ôm sát vật.
3. Chọn đúng lớp và thuộc tính.

## 4. Ca khó

(chưa có)

## 5. QC và phiên bản

- Mọi thay đổi làm đổi nhãn đều tăng phiên bản guideline.
- Ca mơ hồ: tick `can_review`.
