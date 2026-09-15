# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Đoàn Vĩnh Nguyên — MSSV: 2A202602201`
Ngày: `2026-09-15`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | ~50 phút |
| Thời gian gán `clip_01` | ~90 phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | ~12 keyframe/track |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe bị che bởi biển báo giao thông (track 5, frame 85–101): Xe di chuyển từ xa lại gần dưới góc phối cảnh khiến bbox thay đổi kích thước nhanh. Xử lý: đặt keyframe dày mỗi 5–7 frame trong đoạn này để tránh interpolation drift, IoU đã cải thiện từ 0.52 lên > 0.85.
2. Xe rời khung — bấm outside đúng frame (track 4 frame 148, track 8 frame 168): Xe biến mất dần ở mép ảnh, dễ dư 2–3 frame ghost bbox. Xử lý: tua chậm, bấm `O` (outside) đúng frame cuối cùng còn thấy xe.
3. Xe đỗ tĩnh nhiều frame (track 3, đỗ lề đường): Dễ nhầm là lỗi nhưng thực tế là xe thật. Xử lý: xác nhận bằng cách zoom in, đặt 2 keyframe đầu–cuối và giữ bbox cố định; đóng là `not-a-defect` khi review.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1 (identity/timeline): 8 track xuyên suốt, không nhấp nháy ID, IDSW = 0. Mỗi track giữ một ID độc lập từ frame entry đến exit.
- Lượt 2 (endpoint/scope): Phát hiện track 4 và track 8 có ghost bbox dư 3 frame sau khi xe khuất khung (frame 149–151 và 169–171). Đã sửa bằng cách bấm outside.
- Lượt 3 (geometry/interpolation): Phát hiện bbox trôi tại track 5 khoảng frame 89–101 do hai keyframe đặt xa nhau. Đã chèn thêm keyframe, IoU tăng lên > 0.85.

Kiểm chéo với: N/A — bài làm cá nhân.

Luật nào còn thiếu trong `GUIDELINE_MINI.md` (phát hiện qua self-QC ba lượt)?

Luật còn thiếu trước đây: chưa nói rõ ngưỡng % xe nhìn thấy tối thiểu để bắt đầu gán (xe tầng xa sau cây/biển, < 10% thân xe), và chưa đề cập rõ xe đỗ tĩnh không phải lỗi outside. Đã bổ sung cả hai vào GUIDELINE_MINI.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | xem `evidence/pre-gold/clip_01/manifest.json` |
| Thời điểm khóa | 2026-09-15 (trước khi mở gold reference) |
| Số row / frame / track trước khi mở reference | 561 bbox · 190 frame · 8 track |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold (sau rework) | 0.836 | 0.819 | 0.855 | 0.893 | 0.965 | 0.930 | 0.882 | 14 | 26 | 0 |
| Sau rework | 0.836 | 0.819 | 0.855 | 0.893 | 0.965 | 0.930 | 0.882 | 14 | 26 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): ĐẠT — IDF1 0.965 ✓ · MOTA 0.930 ✓ · MOTP 0.882 ✓

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox treo / Ghost bbox | 149–151 | 4 | Bấm outside tại frame 149, xóa 3 frame ghost bbox sau khi xe rời khung trái |
| Bbox treo / Ghost bbox | 169–171 | 8 | Bấm outside tại frame 169, xóa 3 frame ghost bbox sau khi xe rời khung |
| Interpolation drift | 89–101 | 5 | Chèn keyframe bổ sung tại frame 89, 93, 95, 96, 97, 98, 99, 101; IoU tăng lên > 0.85 |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ notebook (model_run_config.json):

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13 |
| weights / hai tracker | yolo26n.pt / ByteTrack (control) · BoT-SORT+ReID (treatment) |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / [2, 5, 7] (car, bus, truck) |
| device | CPU (Google Colab) |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.836 | 0.819 | 0.855 | 0.893 | 0.965 | 0.930 | 0.882 | 14 | 26 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.818 | 0.760 | 0.881 | 0.914 | 0.912 | 0.813 | 0.905 | 91 | 14 | 0 |

## 5. Phân tích — năm câu hỏi

1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?

Bản annotation của tôi có MOTA = 0.930 thấp hơn IDF1 = 0.965 — nghĩa là annotation chặt về mặt identity (ít sai ID) nhưng còn một số FN (bỏ sót 26 bbox so với gold). Nếu MOTA cao mà IDF1 thấp thì có nghĩa là tracker tìm đúng nhiều vật thể (ít FP/FN) nhưng hay đổi ID — phát hiện đúng vật nhưng không nhớ được "đây là xe nào". MOTA không phạt nặng IDSW vì trong công thức MOTA = 1 − (FP+FN+IDSW)/GT, mỗi ID switch chỉ bị trừ 1 điểm, bằng một FP hay FN thông thường, trong khi xe có thể xuất hiện hàng trăm frame. Ngược lại IDF1 đo proportion bằng xâu chuỗi match qua toàn bộ lifetime của track nên nhạy cảm hơn nhiều với việc đổi ID.

2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.

ReID treatment cải thiện rõ ở IDF1 (0.900 vs 0.875) và AssA (0.820 vs 0.776), nhưng IDSW vẫn bằng nhau (đều là 2). Cả hai đều có 2 ID switch, nhưng xảy ra ở frame khác nhau: ByteTrack bị switch tại frame 59 (track gold 4) và frame 94 (track gold 5); ReID bị switch tại frame 87 (track gold 5) và frame 113 (track gold 6). Ví dụ cụ thể: đoạn frame 85–115, track gold 5 (xe đi từ xa lại gần, bị che một phần bởi xe khác): ByteTrack bị switch ID ở frame 94, trong khi ReID giữ được association đến frame 87 (muộn hơn 7 frame). Điều này nhất quán với việc ReID dùng appearance feature để re-link sau occlusion, giúp AssA cao hơn. Lưu ý: không thể kết luận ReID là nguyên nhân trực tiếp vì ByteTrack và BoT-SORT là hai tracker implementation hoàn toàn khác nhau (Kalman filter state, matching cost, hyperparameter).

3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?

So sánh ByteTrack vs ReID treatment:
- DetA tăng từ 0.649 → 0.711 (ReID cao hơn 9.6%)
- FP tương đương: ByteTrack 88 vs ReID 91 (ReID thậm chí thêm 3 FP)
- FN giảm mạnh: ByteTrack 54 vs ReID 26 (ReID bỏ sót ít hơn 52%)

Như vậy lỗi cải thiện chủ yếu ở FN — detector/tracker của BoT-SORT bắt được xe trong nhiều frame hơn (số bbox = 638 vs 607). Tuy nhiên FP tăng nhẹ vì BoT-SORT cũng sinh thêm bbox giả (ID thừa). Nhìn vào 5 ID thừa của mỗi tracker (đều có 5 BBOX_THỪA), lỗi còn lại có cả detector (bbox lệch) lẫn association (tách track), nhưng detector là bottleneck lớn hơn: LocA của cả hai đều ≈ 0.85–0.87, thấp hơn hẳn bản tay (0.893), cho thấy bbox của model chưa khít vật thể bằng người gán.

4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):

Frame 85–88, track gold 5: model ReID sinh ID giả tại frame 85 (ID 17 xuất hiện 1 frame rồi biến mất, FP). Bản annotation tay của tôi không có bbox ở vị trí đó vì xe chưa đủ nhìn rõ (< 30% thân xe hiện ra). Model ReID sinh FP do conf threshold 0.25 thấp, bắt cả noise/partial detection. Bản tay đúng hơn vì tôi đặt entry keyframe sau khi xe xuất hiện đủ rõ và xác định được là xe bốn bánh.

5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:

Frame 16–116, model ID 7 (43 frame liên tục không khớp track gold nào): ReID sinh một track rất dài (43 frame) hoàn toàn không tương ứng với xe nào trong gold. Điều này khiến tôi xem lại toàn bộ đoạn frame 16–116 để kiểm tra xem có xe nào tôi bỏ sót không. Kết quả xác nhận đây là model sai: object bị detect có thể là xe máy hoặc phản chiếu gương xe bị phân loại nhầm thành car/truck (class 2/7) do conf threshold quá thấp. Bản annotation tay chính xác hơn vì tôi nhìn trực tiếp và xác định đây không phải xe bốn bánh.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- Bổ sung ngưỡng entry rõ ràng: Xe phải hiện > 30% thân xe và nhận diện được là xe bốn bánh mới bắt đầu track; xe tầng xa sau cây/biển < 10% không gán.
- Tăng dày keyframe đoạn phối cảnh: Với xe đang tiến/lùi theo chiều sâu (to dần hoặc nhỏ dần nhanh), đặt keyframe mỗi 5 frame thay vì 10 frame để tránh interpolation drift (bài học từ track 5, frame 89–101).
- Tua ngược khi kiểm tra endpoint: Sau khi gán xong, tua từ frame cuối về đầu để dễ phát hiện ghost bbox hơn là tua xuôi.
- Dùng model output làm tham chiếu sơ bộ: Chạy BoT-SORT+ReID trước để biết sơ bộ có bao nhiêu xe, sau đó gán nhãn tay và so sánh — giúp phát hiện xe bị bỏ sót sớm hơn.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
