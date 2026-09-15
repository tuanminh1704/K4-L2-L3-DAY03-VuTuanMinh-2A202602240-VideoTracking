# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Vũ Tuấn Minh`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `40` phút |
| Thời gian gán `clip_01` | `45` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `3` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe bị che khuất một phần bởi các phương tiện khác.
   → Tôi theo dõi vị trí và hình dạng của xe qua các frame liên tiếp, đồng thời giữ cùng track khi xe xuất hiện trở lại.

2. Các xe di chuyển gần nhau hoặc bị chồng lấn bounding box.
   → Tôi kiểm tra vị trí và quỹ đạo di chuyển của từng xe qua nhiều frame để tránh nhầm giữa các track.

3. Xexuất hiện không rõ ràng trong một số frame.
   → Tôi dựa vào các frame trước và sau để xác định sự liên tục của đối tượng, chỉ tạo track mới khi xác định đó là một xe khác.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `...`
- Lượt 2: `...`
- Lượt 3: `...`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `...` |
| Thời điểm khóa | `...` |
| Số row / frame / track trước khi mở reference | `...` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.762 | 0.707 | 0.824 | 0.882 | 0.891 | 0.776 | 0.871 | 89 | 41 | 2 |
| Sau rework | chưa có | chưa có | chưa có | chưa có | chưa có | chưa có | chưa có | chưa có | chưa có | chưa có |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Lệch ID | 87 | 17 → 18 | Kiểm tra và thống nhất ID của cùng một xe, tránh đổi ID giữa các frame |
| Lệch ID | 113 | 28 → 31 | Kiểm tra và thống nhất ID của cùng một xe, tránh đổi ID giữa các frame |
| BBOX lệch nhau | 140 | Track A 3 | Thống nhất cách khoanh bbox theo phần xe nhìn thấy và sửa bbox cho khớp |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
|---|---|
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / ByteTrack (bytetrack.yaml) / BoT-SORT + ReID (botsort-reid.yaml)` |
| conf / IoU / imgsz / classes | `0.25 / 0.7 / 960 / [2, 5, 7]` |
| device | `cpu` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Bạn vs gold | 0.822 | 0.808 | 0.838 | 0.873 | 0.977 | 0.953 | 0.857 | 22 | 5 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.762 | 0.707 | 0.824 | 0.882 | 0.891 | 0.776 | 0.871 | 89 | 41 | 2 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`MOTA của bạn cao hơn IDF1: MOTA = 0.953, trong khi IDF1 = 0.977, tức IDF1 vẫn cao hơn MOTA. MOTA chủ yếu tổng hợp các lỗi FP, FN và ID switch, trong khi IDF1 tập trung vào mức độ đúng của việc duy trì và ghép ID. Vì vậy, MOTA có thể vẫn cao khi có một số lỗi ID nếu số lượng FP/FN và ID switch không quá lớn. Trong kết quả của bạn, IDSW = 0 nên cả MOTA và IDF1 đều rất cao.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`ByteTrack control có IDF1 = 0.875, AssA = 0.776 và IDSW = 2. BoT-SORT + ReID có IDF1 = 0.900, AssA = 0.820 và IDSW = 2. Như vậy BoT-SORT + ReID tốt hơn ByteTrack về IDF1 và AssA, trong khi số IDSW không đổi. Một sequence đáng chú ý là frame 87 và frame 113 trong so sánh ReID với nhãn của tôi: tại frame 87 track bản A 5 đổi giữa ID 17 và 18, còn tại frame 113 track bản A 6 đổi giữa ID 28 và 31. Tuy nhiên, các kết quả này không cô lập causal effect của ReID vì ByteTrack và BoT-SORT là hai tracker implementation khác nhau, nên không thể kết luận toàn bộ mức cải thiện là do ReID.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`So với ByteTrack control, BoT-SORT + ReID có DetA tăng từ 0.649 lên 0.711, FN giảm từ 54 xuống 26 nhưng FP tăng nhẹ từ 88 lên 91. Điều này cho thấy khả năng phát hiện/ghép detection nhìn chung tốt hơn ở BoT-SORT + ReID, đặc biệt là giảm bỏ sót. Tuy nhiên vẫn còn lỗi detection vì vẫn có FP và FN. Đồng thời vẫn còn lỗi association vì IDSW = 2 và có các trường hợp lệch ID, tách track trong danh sách lỗi. Vì vậy lỗi còn lại gồm cả detector và association, không chỉ một phía.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Frame 87, track bản A 5: bản A đang sử dụng ID 17 nhưng sau đó nhảy sang ID 18. Đây là trường hợp hai bản lệch ID, cùng một xe nhưng bị gán hai ID khác nhau. Nếu annotation của tôi giữ ID liên tục cho cùng một xe thì tại sequence này ReID đã không duy trì đúng ID của đối tượng.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Frame 140, track bản A 3: IoU chỉ đạt 0.51. Đây là trường hợp cùng một xe nhưng hai bên khoanh bbox khác nhau. Kết quả này khiến tôi cần xem lại annotation và thống nhất luật khoanh bbox, đặc biệt là bbox phải ôm phần vehicle nhìn thấy được. Tuy nhiên chỉ từ IoU = 0.51 chưa đủ bằng chứng để kết luận ReID sai; cần kiểm tra frame thực tế và annotation tương ứng trước khi sửa.`

## 6. Nếu phải gán thêm 10 clip nữa

`Tôi sẽ làm rõ hơn trong GUIDELINE_MINI.md về ba điểm: (1) quy tắc giữ ID khi vehicle bị che khuất, với ngưỡng 25 frame; (2) quy tắc tạo track mới khi vehicle rời khung hình rồi quay lại; (3) quy tắc khoanh bbox khi vehicle bị che, bị cắt bởi rìa ảnh hoặc hai vehicle chồng lấn. Trong quy trình làm việc, tôi sẽ kiểm tra kỹ các đoạn có xe giao nhau, bị che khuất và xuất hiện/biến mất; đồng thời kiểm tra các frame có IoU thấp và các trường hợp ID switch trước khi khóa annotation. Sau mỗi clip, tôi sẽ ghi lại frame và ID của các ca mơ hồ để guideline được cập nhật từ các lỗi thực tế thay vì chỉ dựa trên quy tắc ban đầu.`

## 7. Tệp đã nộp

- [X] `annotations/clip_01/gt.txt`
- [X] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [X] `GUIDELINE_MINI.md` đã điền
- [X] `outputs/eval_vs_gold.json`
- [X] `outputs/model_bytetrack_clip_01.txt`
- [X] `outputs/model_reid_clip_01.txt`
- [X] `outputs/model_run_config.json`
- [X] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [X] `reports/REPORT.md` (file này)
