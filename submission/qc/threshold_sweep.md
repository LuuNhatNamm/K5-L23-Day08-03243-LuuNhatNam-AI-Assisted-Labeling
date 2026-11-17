# Quét ngưỡng theo lớp

Mục tiêu recall: 90%. Ảnh dùng để quét: 4.

| Lớp | Ngưỡng gợi ý | Lý do |
|---|---|---|
| person | (mặc định) | không ngưỡng nào <= 0.5 đạt recall 90% (tối đa 85%, 20 vật): dùng ngưỡng mặc định, nói rõ trong báo cáo |
| bicycle | (mặc định) | chỉ 4 vật trong ảnh gold (< 5): mẫu quá ít, KHÔNG chọn ngưỡng theo lớp này, dùng ngưỡng mặc định |
| motorbike | 0.1 | recall 92% (>= 90%), precision 44% (13 vật gold) |
| car | 0.5 | recall 90% (>= 90%), precision 69% (10 vật gold) |
| bus | (mặc định) | chỉ 0 vật trong ảnh gold (< 5): mẫu quá ít, KHÔNG chọn ngưỡng theo lớp này, dùng ngưỡng mặc định |
| truck | (mặc định) | chỉ 0 vật trong ảnh gold (< 5): mẫu quá ít, KHÔNG chọn ngưỡng theo lớp này, dùng ngưỡng mặc định |
| three_wheeler | 0.15 | recall 100% (>= 90%), precision 67% (6 vật gold) |

Dán vào lệnh merge_prelabels (lớp không có trong chuỗi dùng `--conf` mặc định 0.3): `--thr "motorbike=0.1,car=0.5,three_wheeler=0.15"`
