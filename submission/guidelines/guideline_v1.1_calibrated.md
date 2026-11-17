# Label contract — Phố châu Á — Bổ sung v1.1

Bản bổ sung dựa trên bốn ca trong [bảng calibration](../qc/calibration_decisions.csv). Các quy tắc dưới đây là đề xuất để người gán nhãn xác nhận theo ảnh.

## 1. Câu hỏi mở trong v1.0

| Câu hỏi | Quy tắc đề xuất | Ảnh ví dụ | Ca |
|---|---|---|---|
| 7. Có dùng ngưỡng conf để tự chấp nhận box? | Mục 4.3. Dùng conf để xếp thứ tự kiểm. Chỉ chấp nhận box sau khi đối chiếu lớp và đường biên trên ảnh. | `img_011.jpg`: YOLO26 chấm 0.77 cho box `truck` đang bao tuk-tuk ở mép trái. | CAL-01 |
| 6. Xử lý box chồng nhau thế nào? | Mục 4.1. Dùng phần xe nhìn thấy để xác định từng vật. Hai box cùng bao một xe thì giữ một box sát vật. Hai xe khác nhau vẫn giữ box riêng dù chồng nhau. Với ảnh tối khó phân biệt, đánh dấu `can_review`. | `img_005.jpg`: box 58 bao đuôi và biển số xe đã có box 47. Đây là ví dụ cảnh đông ban ngày cho quy tắc đối chiếu cùng vật. | CAL-02 |

Câu 1–5 giữ nguyên trạng thái của v1.0 trong đợt này vì chưa đủ ca đối chiếu để chốt thêm.

## 2. Ca khó từ lỗi mask

| Ảnh | Quan sát | Quy tắc đề xuất | Ca |
|---|---|---|---|
| `img_013.jpg` | SAM 2.1 vẫn phủ một phần vai và chân người lái ở vùng giáp ghế và sàn. | Mục 4.2. Rà riêng vai và chân tại vùng tiếp giáp. Đặt điểm âm vào phần người còn dính rồi kiểm lại mép ghế, sàn và bánh nhìn thấy. | CAL-03 |
| `img_018.jpg` | Lượt đầu lọt bẫy `loose` ở box xe máy nhỏ trong vùng tối. | Mục 4.1. Phóng to vùng xe nhỏ và rà từng cạnh box theo phần xe nhìn thấy. Kiểm riêng cạnh trên và cạnh bên nếu box bị lệch. | CAL-04 |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi |
|---|---|
| v1.0 | Bản giảng viên. |
| v1.1 | Bổ sung cách xác minh box điểm cao, rà box trùng và kiểm ranh giới người lái với xe và biên box xe nhỏ (CAL-01 đến CAL-04). |
