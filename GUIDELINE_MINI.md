# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này trong lúc gán nhãn, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Đoàn Vĩnh Nguyên — MSSV: 2A202602201`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: `vehicle` — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | xe máy / mô tô |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm:
- Ngưỡng entry tối thiểu: xe phải hiện > 30% thân xe và nhận diện rõ ràng là xe bốn bánh mới bắt đầu gán track. Xe ở tầng xa sau cây/biển báo chỉ lộ < 10% thân xe thì không gán.
- Phản chiếu gương hoặc bóng xe không gán, kể cả khi trông giống xe thật.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che dưới 25 frame (2 giây @ 12.5 fps) | Tránh tạo track mới cho cùng một xe; IDSW = 0 là mục tiêu |
| Xe bị che lâu hơn 25 frame | tạo track mới (ID mới) | Không thể xác định chắc chắn đây là cùng một xe |
| Xe rời khung hình rồi quay lại | track mới — không reuse ID cũ | Quy tắc mặc định của lab; tránh nhầm xe khác cùng loại |
| Hai xe cắt nhau / chồng lên nhau | giữ nguyên ID mỗi xe; bbox ôm phần nhìn thấy được của từng xe, chấp nhận IoU giảm trong đoạn chồng | Mỗi xe là một entity độc lập; không merge ID |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần nhìn thấy được |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên nhìn thấy > 30% thân xe và xác định được là xe bốn bánh |
| Xe đang đỗ, không di chuyển | vẫn gán nhãn bình thường; đặt 2 keyframe (frame entry + frame exit hoặc frame cuối clip); không phải lỗi |
| Keyframe đặt dày ở đâu | đặt mỗi 5 frame ở đoạn xe to/nhỏ nhanh theo phối cảnh (tiến/lùi), hoặc đoạn bị che rồi hiện lại; đặt mỗi 10 frame ở đoạn xe di chuyển ổn định |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi frame cụ thể và ID cụ thể, không ghi chung chung.

### Ca 1 — Ghost bbox (bbox treo sau khi xe rời khung)
- Clip / frame / ID: `clip_01 / frame 149–151 / ID 4`
- Tình huống: Xe con rời khung bên trái từ frame 148, nhưng track CVAT vẫn nội suy bbox ra ngoài khung thêm 3 frame.
- Quyết định: Bấm phím `O` (outside) tại frame 149 ngay sau khi xe khuất hoàn toàn.
- Lý do: Bbox treo không ứng với vật thể thực tế → FP trong evaluation. Rule Exit: track kết thúc đúng frame xe biến mất.

### Ca 2 — Ghost bbox lần 2
- Clip / frame / ID: `clip_01 / frame 169–171 / ID 8`
- Tình huống: Xe rời khung góc trên phải từ frame 168, track dư 3 frame ghost bbox.
- Quyết định: Bấm outside tại frame 169.
- Lý do: Tương tự Ca 1 — rule Exit áp dụng nhất quán cho mọi track.

### Ca 3 — Interpolation drift (bbox trôi giữa hai keyframe)
- Clip / frame / ID: `clip_01 / frame 89–101 / ID 5`
- Tình huống: Xe di chuyển theo góc phối cảnh từ xa lại gần (bbox to dần nhanh), hai keyframe ban đầu đặt quá xa (~15 frame). Nội suy tuyến tính khiến bbox bị trôi, IoU xuống 0.52–0.58.
- Quyết định: Chèn thêm keyframe tại frame 89, 93, 95, 96, 97, 98, 99, 101; điều chỉnh bbox bám sát thân xe.
- Lý do: Xe đổi kích thước + hướng không tuyến tính → cần keyframe dày để CVAT nội suy đúng.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Ngưỡng entry: Ban đầu không nói rõ khi nào bắt đầu gán. Đã bổ sung: > 30% thân xe nhìn thấy. Xe tầng xa < 10% không gán (phát hiện sau khi so sánh với model output ID 7 dài 43 frame — model gán nhầm object không phải xe bốn bánh).
- Keyframe density: Ban đầu chỉ ghi "đặt keyframe đủ dày". Đã cụ thể hoá: mỗi 5 frame ở đoạn phối cảnh thay đổi nhanh, mỗi 10 frame ở đoạn ổn định.
- Xe đỗ tĩnh: Trước đây chưa đề cập rõ → dễ nhầm là lỗi bỏ outside. Đã thêm: xe đỗ vẫn gán bình thường với 2 keyframe.
