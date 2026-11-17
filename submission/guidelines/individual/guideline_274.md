# Guideline gán nhãn — 274

> Đây là bản guideline gán nhãn của 274

- Phiên bản: v1.0
- Loại nhãn: bounding box (hộp chữ nhật) trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện** trên ảnh đường phố
(ví dụ cho camera giao thông hoặc xe tự lái).

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa |
|---|---|
| `car` | Ô tô con. |  
| `truck` | Xe tải. |  
| `bus` | Xe buýt |
| `van` | Xe thùng kín cỡ nhỏ. |
| `motorcycle` | Xe máy, mô tô. |
| `bicycle` | Xe đạp |
| `fence_trailer`| Xe lồng, rơ mooc lồng |
| `tuk tuk`| Xe lam, xe kéo tự động. |
| `cyclo`| Xe xích lô |
| `rider` | Người tham gia giao thông phương tiện hở |
| `person` | Người. |
| `ignore_region` | Vùng không cần gán từng vật. |

### Thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Mức bị che: 0 là thấy rõ, 1 là bị che dưới 50%, 2 là bị che trên 50%, 3 là không xác định. |
| `truncated` | `no`, `yes` | Vật bị cắt ở mép ảnh. |
| `can_review` | tick / không | Tick khi bạn không chắc, để người khác xem lại. |

## 3. Quy tắc vẽ box

1. Khung ôm sát phần nhìn thấy của vật:
- Mép khung chạm điểm ngoài cùng của vật: trên cùng, dưới cùng, trái cùng, phải cùng.
- Không chừa lề, không chừa khoảng trống.
- Vật bị che: chỉ bao phần nhìn thấy, không đoán phần bị khuất.
- Vật bị mép ảnh cắt: bao phần nằm trong ảnh.
- Chọn đúng class của vật.

2. Bao nhiêu khung:
- Mỗi vật vẽ một khung. Không vẽ một khung cho cả sự vật đôi.
- Vẽ cho mọi vật đủ rõ.
- Hai vật chồng lên nhau, cả hai thấy rõ → 2 khung, được phép chồng nhau.
- Vật có thứ khác nằm trên → 1 khung cho vật. Mô hình chỉ cần biết vật đó, không khoanh những thứ khác.
- Không khoanh riêng từng thành phần của vật. Khung luôn bao cả vật.
- Không vẽ vật quá nhỏ (cạnh ngắn dưới khoảng 32 điểm ảnh), quá mờ, hoặc bị che quá nửa.

3. Không gán:
- Vật phản chiếu trong vũng nước, tấm nhựa hoặc kính nhà lưới.
- Hình vật in trên bao bì, poster, tài liệu hoặc hiện trên màn hình điện thoại.
- Tay người chụp, dụng cụ, thước đo, nóc capo hướng ra.  
- Các vật cần ignore.

## 4. Ca khó

- img_025: có người đi xe máy lôi theo 1 xe hàng, có gán xe hàng không?
- img_022: xe bên cạnh xe máy là xe gì?

## 5. QC và phiên bản

- Mọi thay đổi làm đổi nhãn đều tăng phiên bản guideline.
- Ca mơ hồ: tick `can_review`.
