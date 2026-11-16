# Guideline gán nhãn — bản cá nhân v0.1 — 2A202603276

- Phiên bản: v0.1 (bản cá nhân)
- Ngày: 2026-11-16
- Người viết / duyệt: Ninh Hai nam / Le Tien Binh

## 1. Mục tiêu

Gán nhãn người và phương tiện có thật trong ảnh đường phố để dùng huấn luyện mô hình nhận diện. Cần làm nhất quán giữa các ảnh. Không gán tượng, hình ảnh, mannequin hoặc đồ chơi thành người/phương tiện thật. Bộ ảnh có cảnh đông, xe chở hàng và nhiều vật dễ nhầm, nên gặp trường hợp chưa chắc thì ghi lại để nhóm xem xét.

## 2. Lớp và thuộc tính

### Lớp

| Lớp                       | Định nghĩa ngắn                                           | Ví dụ                                        | Dễ nhầm với                                                    |
| ------------------------- | --------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------- |
| `car`                     | Ô tô chở người.                                           | Sedan, SUV, taxi.                            | Xe tải, xe van, ô tô đồ chơi.                                  |
| `truck`                   | Xe lớn chủ yếu dùng chở hàng.                             | Xe tải thùng, xe chở ô tô.                   | Xe van, xe máy chở hàng.                                       |
| `van`                     | Xe thân kín dùng chở người hoặc hàng.                     | Minivan, xe van giao hàng.                   | Ô tô con hoặc xe tải nhỏ.                                      |
| `motorcycle`              | Xe máy hoặc scooter hai bánh.                             | Xe máy đang chạy hoặc đỗ.                    | Xe đạp, xe ba bánh/tuk-tuk.                                    |
| `three_wheeler` (đề xuất) | Phương tiện cơ giới có ba bánh, dùng chở người hoặc hàng. | Tuk-tuk, xe lam/xe ba bánh có động cơ.       | Xe máy hai bánh, xe đạp ba bánh không động cơ.                 |
| `person`                  | Người thật trong ảnh, kể cả người ngồi trên xe.           | Người đi bộ, người lái xe.                   | Tượng, mannequin, hình người trên ảnh.                         |
| `ignore_region`           | Vùng khó tách thành từng vật riêng.                       | Một cụm quá đông, không thể gán rõ từng vật. | Không dùng thay cho nhãn của một vật riêng lẻ chỉ vì khó nhìn. |

Nếu vật là đồ chơi, tượng, mannequin hoặc hình ảnh thì không gán như vật thật. Lớp `three_wheeler` là đề xuất cá nhân vì bộ ảnh có tuk-tuk/xe ba bánh. Chỉ dùng lớp này nếu nhóm đồng ý và cập nhật taxonomy chung; trước khi thống nhất, đánh dấu các trường hợp đó `can_review`. Với vật không khớp lớp nào khác, không tự tạo lớp mới.

### Thuộc tính

| Thuộc tính       | Giá trị            | Cách chọn                                                                                                                                                               |
| ---------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `occluded_level` | `0`, `1`, `2`, `3` | Ước lượng phần vật bị vật khác che: `0` = 0–10%; `1` = trên 10–25%; `2` = trên 25–50%; `3` = trên 50% nhưng vẫn nhận ra được. Đây là ngưỡng đề xuất để nhóm thống nhất. |
| `truncated`      | `no`, `yes`        | Chọn `yes` nếu vật bị khung ảnh cắt ở mép; nếu không thì `no`. Bị vật khác che không tính là bị cắt mép ảnh.                                                            |
| `can_review`     | tick / không tick  | Tick nếu bạn chưa chắc về lớp hoặc có nên gán vật đó hay không; ghi lý do để nhóm xem lại.                                                                              |

## 3. Quy tắc vẽ box

1. Vẽ một box riêng cho mỗi người và phương tiện thật thuộc lớp đã thống nhất. Không gộp nhiều vật vào một box.
2. Box ôm sát phần nhìn thấy của vật. Nếu vật bị che, không đoán phần khuất; chọn `occluded_level` theo phần 2.
3. Nếu vật bị khung ảnh cắt, vẽ box đến mép ảnh, không vượt ra ngoài ảnh, rồi chọn `truncated=yes`.
4. Gán người và xe riêng: người ngồi trên xe vẫn có box `person`, xe có box riêng.
5. Xe đang được chở cũng là vật riêng nếu nhìn rõ: gán box xe chở và từng xe được chở.
6. Không gán tượng, mannequin, hình in hoặc đồ chơi như người/phương tiện thật. Với ảnh phản chiếu hoặc xe ba bánh chưa có lớp được nhóm duyệt, đánh dấu `can_review` và ghi lại để hỏi nhóm.
7. Chỉ dùng `ignore_region` cho cụm quá đông/khó tách theo quy tắc nhóm; không dùng thay cho nhãn của một vật riêng lẻ. Không tự thêm lớp mới.
8. Theo ngưỡng bài lab, bỏ qua vật cao dưới 20 pixel. Nếu việc này gây nghi ngờ, ghi lại để nhóm xem.
## 4. Ca khó

Các quyết định dưới đây là đề xuất cá nhân để nhóm thảo luận; những ca có `can_review` cần được thống nhất trước khi gán nhãn chính thức.

| Ảnh                             | Tình huống                                      | Rule ID | Quyết định ban đầu                                                                                                           | Lý do                                                                                    |
| ------------------------------- | ----------------------------------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `img_002`                       | Xe tải chở nhiều ô tô                           | R-01    | Gán `truck` cho xe chở và `car` riêng cho từng ô tô nhìn thấy rõ.                                                            | Mỗi phương tiện là một vật riêng, dù ô tô đang được chở.                                 |
| `img_004`, `img_016`, `img_035` | Tuk-tuk/xe ba bánh có động cơ                   | R-02    | Đề xuất gán `three_wheeler`; đánh dấu `can_review` cho tới khi nhóm duyệt lớp này.                                           | Taxonomy hiện tại chưa có lớp phù hợp; không nên tự gán thành `motorcycle` hay `car`.    |
| `img_006`, `img_021`            | Cảnh đông, người và xe máy che lẫn nhau         | R-03    | Gán riêng từng người/xe nhận diện được; dùng mức che khuất theo phần 2. Đánh dấu `can_review` nếu không chắc vật nào bị che. | Tránh gộp nhiều người hoặc nhiều xe vào một nhãn.                                        |
| `img_020`                       | Xe và đường chỉ xuất hiện trong gương chiếu hậu | R-04    | Đánh dấu `can_review`; đề xuất không gán vật chỉ nhìn thấy qua ảnh phản chiếu cho tới khi nhóm thống nhất.                   | Quy tắc v0 chưa nói rõ ảnh phản chiếu có thuộc đối tượng cần gán hay không.              |
| `img_027`                       | Xe tải dạng tượng/trưng bày                     | R-05    | Không gán tượng xe là `truck`; gán các đối tượng thật khác nếu có.                                                           | Hình dáng giống xe nhưng đây không phải phương tiện thật đang được nhận diện trong cảnh. |
| `img_038`                       | Xe đồ chơi và mannequin trong cửa kính          | R-06    | Không gán xe đồ chơi là `car`, không gán mannequin là `person`.                                                              | Đây là đồ trưng bày, không phải xe/người thật.                                           |
| `img_029`                       | Người đi xe đạp và người đi xe máy              | R-07    | Gán người và xe máy nhìn thấy; không gán xe đạp vì taxonomy hiện không có lớp xe đạp.                                        | Không ép xe đạp vào lớp `motorcycle`.                                                    |
| `img_017`, `img_022`            | Tượng người cưỡi ngựa/tượng người               | R-08    | Không gán tượng là `person`.                                                                                                 | Người trong ảnh là tượng, không phải người thật.                                         |

## 5. QC và phiên bản

### Kiểm tra chất lượng (đề xuất cá nhân)

- Chạy validator của repo để kiểm tra ảnh, lớp, thuộc tính và box có hợp lệ không.
- So sánh nhãn giữa các thành viên và xem ảnh overlay để tìm vật bị bỏ sót, box thừa, sai lớp hoặc lệch vị trí. Script hỗ trợ kiểm tra nhưng không thay thế việc xem ảnh gốc.
- Theo ngưỡng hiện có trong `taxonomy.json`: IoU là `0.85` với vật lớn và `0.70` với vật nhỏ; IAA yêu cầu IoU `0.85` và đồng thuận lớp `0.95`.
- Tỷ lệ ảnh lỗi tối đa là `2%`; báo cáo cần ghi cả số lỗi và tổng số ảnh đã kiểm. Không tự thay đổi các ngưỡng trong `taxonomy.json`.

### Ca chưa chắc và lịch sử phiên bản

Khi chưa chắc, đánh dấu `can_review`, ghi mã ảnh và lý do. Nhóm sẽ thống nhất cách xử lý; không xem đề xuất cá nhân là quy tắc chung đã được duyệt.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| `v0.1` | Làm rõ lớp, thuộc tính, quy tắc box và ca khó; đề xuất lớp `three_wheeler`. | Các tình huống trong bộ ảnh cần được nhóm xem xét; lớp mới chỉ dùng sau khi nhóm duyệt. |
