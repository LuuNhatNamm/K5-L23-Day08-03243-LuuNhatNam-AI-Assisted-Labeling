# Báo cáo Day 08 — 202603243 - Lưu Nhật Nam

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | motorbike 0.10, car 0.50, các lớp còn lại mặc định 0.30 | Theo `qc/threshold_sweep.md`, motorbike đạt recall 12/13 và car đạt 9/10. Recall cao nhất của person là 17/20. Gold có 4 bicycle, 0 bus và 0 truck. |
| three_wheeler | YOLOE-26s, prompt tuk-tuk / cyclo / cargo tricycle | 0.15 | Theo `qc/threshold_sweep.md`, recall 6/6 và precision 6/9. Xem thêm lượt r03 trong `model_log.csv`. |

- Lệnh gộp đã chạy (lịch sử PowerShell):

  ```powershell
  python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,motorbike=0.5,car=0.35" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
  ```

  Lượt gộp CP4 dùng chuỗi ngưỡng ví dụ trong hướng dẫn. Chuỗi script quét trên gold của bài này là `motorbike=0.1,car=0.5,three_wheeler=0.15`. Số liệu review và QC bên dưới thuộc bản thực tế đã nạp vào CVAT.
- **SAM đã thử:** SAM 2.1 Base (`sam2.1_b.pt`), lượt `r04` trong `model_log.csv`. Kết quả có 16 mask từ 16 box trên 2 ảnh, với thời gian pipeline 5,12 giây/ảnh trên CPU. Mask người đứng bên phải ở `img_004.jpg` bám khá sát thân và chân. Mask tuk-tuk ở `img_013.jpg` bỏ phần đuôi bên trái.
- **Lỗi không sửa được chỉ bằng `--conf`:** YOLO26 gán tuk-tuk sát mép trái `img_011.jpg` thành `truck` ở cả hai ngưỡng 0.25 và 0.50 (`r01`, `r02`). YOLO26 thiếu lớp `three_wheeler`. Dùng YOLOE để đề xuất lớp này và rà nhãn thủ công khi duyệt.
- **Điểm cao mà sai:** Box tuk-tuk nói trên mang nhãn `truck` với điểm 0.77 trong ảnh trực quan của `r01` và `r02`. Điểm tin cậy của model cần được đối chiếu với ảnh.
- Giới hạn: ngưỡng được chọn và đo trên cùng 4 ảnh gold. Precision và recall là số đo in-sample, có thể lạc quan khi áp dụng cho lô 24 ảnh. Script dùng ngưỡng mặc định cho lớp có dưới 5 đối tượng gold.

## 2. AI error profile

Kết quả bản cuối từ `qc/review_stats.md`, tính trên 24 ảnh batch. Script loại 25 đối tượng quanh các bẫy khỏi bảng này.

| Trạng thái | Số lượng | Tỉ lệ và mẫu số |
|---|---:|---|
| accepted | 388 | 388/432 box máy (89,8%) |
| edited | 8 | 8/432 box máy (1,9%) |
| deleted | 32 | 32/432 box máy (7,4%) |
| added | 44 | 44/440 đối tượng sau duyệt (10,0%) |

`three_wheeler` có 7 box sửa, 13 box xóa và 8 box thêm. Ví dụ, YOLO26 gán tuk-tuk xanh ở `img_011.jpg` thành `truck` trong các lượt pilot `r01` và `r02`. `motorbike` có nhiều box thêm nhất (19).

Trên 4 ảnh gold, máy đạt **precision 42/62 (67,7%)** và **recall 42/53 (79,2%)**. Script đếm 42 TP, 20 FP và 11 FN.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Lượt đầu bắt **9/12 bẫy (75,0%)**. Lọt `missing` ở `img_013.jpg` và hai bẫy `loose` ở `img_018.jpg` và `img_029.jpg`. Hai dạng lọt tương ứng với bước tìm vật thiếu và kiểm lại biên box.
- Bản cuối bắt **12/12 (100%)**, gồm 3/3 mỗi loại `missing`, `loose`, `wrong_class`, `extra`. Khoảng tin cậy 95% là 76–100%.
- Tỉ lệ giữ nguyên **388/432 box (89,8%)** và tỉ lệ bắt bẫy có mẫu số khác nhau. Lượt cuối cho thấy phần lớn box máy được giữ sau duyệt, đồng thời 12 bẫy đã được xử lý. Kết quả áp dụng cho các bẫy đã cài trong bài này.

## 4. Máy có đáng dùng không?

- **Gán tay:** 10 phút cho 4 ảnh gold, trung bình **2,5 phút/ảnh** (nguồn `time_log.csv`).
- **Duyệt nhãn máy:** 50 phút cho 24 ảnh batch, trung bình **2,08 phút/ảnh** (nguồn `time_log.csv`). Nếu lấy tốc độ gán tay trên gold là 2,5 phút/ảnh làm mốc, gán tay 24 ảnh sẽ mất khoảng 60 phút. Lượt duyệt ngắn hơn mốc ước tính 10 phút (16,7%). Hai tập có thể khác độ khó, và việc gán gold trước có thể giúp quen guideline khi duyệt batch.
- **Số liệu để đánh giá pre-label:** bản review cuối giữ nguyên 388/432 box máy (89,8%), vẽ thêm 44/440 đối tượng sau duyệt (10,0%) và bắt 12/12 bẫy. Số liệu cho thấy nhiều box máy được giữ qua review và vẫn có đối tượng cần bổ sung. Trong bài này, lượt duyệt batch có số phút/ảnh thấp hơn lượt gán tay gold.
- **Ba bước kiểm soát thiên kiến để đối chiếu với lượt duyệt** (nguồn `docs/REVIEW_CHECKLIST.md`):
  1. Ẩn box máy và quét từng ảnh trước khi bật lại, chú ý vùng xa và vùng bị che.
  2. Rà từng ảnh theo bốn lỗi: thiếu box, thừa box, sai lớp và lệch khung.
  3. Xem kỹ cụm xe nhỏ và các ca khó phân lớp, dùng `can_review` khi còn phân vân.


## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | Model | Điểm xanh / đỏ | Người lái hoặc hàng hóa trong mask | Lỗi quan sát | Ảnh kết quả |
|---|---|---|---|---|---|
| `img_004.jpg` | SAM 2.1 Base | 2 / 2 | Người đứng cạnh xe nằm ngoài mask | Đưa điểm xanh lên bánh trước và đặt điểm đỏ dưới gầm giúp loại phần bóng bị phủ ở bản đầu. | [Ảnh mask](masks/tuktuk_img004_sam2_v2.jpg) |
| `img_013.jpg` | MobileSAM | 4 / 3 | Mặt, áo và chân người lái phần lớn nằm ngoài mask | Giữ được mái, thân và bánh sau. Còn thiếu một phần sàn và viền trước. | [Ảnh mask](masks/tuktuk_img013_sam1_v3.jpg) |
| `img_013.jpg` | SAM 2.1 Base | 4 / 3 | Mask còn phủ vào một phần vai và chân người lái | Giữ thân, bánh sau và sàn trước tốt hơn trên ảnh này. Mask còn lan vào vùng trống phía trước và xe máy phía sau. | [Ảnh so sánh](masks/tuktuk_img013_sam2_v3.jpg) |
| `img_017.jpg` | SAM 2.1 Base | 3 / 1 | Hai người đi bộ nằm ngoài mask | Dời điểm đỏ khỏi mái vào ô hở phía sau ghế giữ được mái và loại phần nền qua ô đó. Còn vùng hồng ở một phần khoảng hở phía trước. | [Ảnh mask](masks/tuktuk_img017_sam2_v2.jpg) |

Hai model dùng chung 7 điểm trên `img_013.jpg`. Điểm xanh đặt ở `(400,480)`, `(600,110)`, `(460,635)`, `(154,75)`. Điểm đỏ đặt ở `(720,300)`, `(750,245)`, `(790,436)`. MobileSAM phủ 31,9% ảnh và SAM 2.1 phủ 41,4%. So sánh đường biên và vùng người lái dựa trên ảnh trực quan của một xe.

Đối chiếu mục 4.2, mask bám phần xe nhìn thấy, giữ mái, kính, khung và bánh, loại bóng đổ và khoảng hở lớn. Với `img_013.jpg`, cần rà riêng ranh giới giữa người lái, tay lái và sàn vì chúng tiếp xúc trực tiếp.


## 6. Calibration (Part 6)

- Có 4 ca đề xuất trong `qc/calibration_decisions.csv`. CAL-01 ghi lại trường hợp YOLO26 gán tuk-tuk thành `truck` với điểm 0.77 ở `img_011.jpg`.
- Bản bổ sung `guidelines/guideline_v1.1_calibrated.md` có ví dụ cho ba nội dung: kiểm box điểm cao, xử lý box trùng, rà ranh giới mask và rà biên box xe nhỏ trong vùng tối.
- Hai nhóm vấn đề quan sát được là giới hạn lớp của YOLO26 và lỗi đường biên còn sót trong các đầu ra dùng để duyệt.

## 7. QC và quyết định (Part 6)

QC dùng 6 ảnh (1 ca khó và 5 ảnh ngẫu nhiên, seed 42). Codex rà ảnh có overlay và ghi phiếu `qc/qc_review.csv`. Người gán nhãn sửa trong CVAT và xuất bản mới.

| Vòng QC | Đối tượng lỗi | Ảnh có lỗi | Gợi ý script |
|---|---:|---:|---|
| 1 | 2/101 (2,0%) | 2/6 (33,3%) | REWORK |
| 2 | 2/100 (2,0%) | 2/6 (33,3%) | REWORK |
| 3 | 0/99 (0,0%) | 0/6 (0,0%) | RELEASE |

Hai lỗi được xử lý trong bản cuối là box xe máy trùng ở `img_005.jpg` và cạnh trái box tuk-tuk lấn sang xe bên cạnh ở `img_017.jpg`. Bản cuối còn đạt 12/12 bẫy.

**Gợi ý cho bản hiện tại ngày 17/11/2026: RELEASE theo tiêu chí lab.** Tỉ lệ lỗi quan sát 0/6 ảnh nằm trong ngưỡng ≤ 2%. Cận trên Wilson 95% là 39%, nên mẫu 6 ảnh chưa đủ chứng minh toàn lô đạt ≤ 2%. Script tổng hợp lỗi đã ghi trong phiếu. Việc xác định box lỏng và đối tượng bị bỏ sót dựa vào rà ảnh. Quyết định nộp bản release do người gán nhãn xác nhận.

## 8. Ghi chú cá nhân

- **Dùng AI:** Codex hỗ trợ kiểm đầu ra dự đoán, tổng hợp kết quả script vào báo cáo. Kiểm chứng dựa trên ảnh xuất và các file CSV, COCO.
- **Tính độc lập của gold:** 4 ảnh có file export gán tay `labels/gold_manual.json`. Việc có xem pre-label trước khi hoàn tất gold cần người gán nhãn xác nhận.
- **Môi trường:** Windows, các lượt pre-label được ghi là CPU, CVAT Docker local 2.82.0. Khi khởi động, CVAT từng trả Bad Gateway trong lúc dịch vụ lên, sau đó trang chủ trả HTTP 200.
- **Cache:** hai lượt inference toàn bộ 36 ảnh được chạy local và sinh kết quả trong `work/prelabel/`, theo `work/runs.csv`.
- **Lỗi script:** không ghi nhận sửa mã pipeline trong các bước đã thực hiện. Validator có cảnh báo một số box dưới 20 px.
- **Điều có thể làm khác ở lượt sau:** sau khi sửa box trong CVAT, mở lại ảnh overlay từ đúng bản export mới để kiểm vị trí sửa trước khi chạy QC.
