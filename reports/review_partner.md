# Peer review — Day 3

| Trường | Giá trị |
| --- | --- |
| Author | Đỗ Trung Kiên (MSSV: 2A202602283) |
| Reviewer | Đỗ Trung Kiên (Tự kiểm 3 lượt cá nhân - Self-QC) |
| Pair ID | Solo-2A202602283 |
| CVAT version | CVAT Community / Online |
| Thời điểm review | 15/09/2026 |

## Danh sách finding

Mỗi finding bắt buộc có `frame + ID + lỗi + sửa thế nào`. Nếu chưa thống nhất,
dùng `needs-review`; không ép tác giả sửa theo cảm tính.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 0–79 | 1–80 | 4 | Bbox ngoài schema (FP) | Xe con ở làn đường xa phía sau cây, kích thước quá nhỏ (<35px) gây ra 80 FP so với Gold | Xóa track 4 khỏi CVAT để tuân thủ ngưỡng phân giải của bộ nhãn chuẩn | fixed |
| 2 | 2–32 | 3–33 | 5 | Bbox ngoài schema (FP) | Xe nhỏ ở mép xa bên trái, gây 31 FP | Xóa track 5 khỏi CVAT | fixed |
| 3 | 79–100 | 80–101 | 5 (sau đổi) | Thiếu đoạn (FN) | Xe bám sau xe buýt bị cắt mất 21 frame đầu, chỉ bắt đầu từ frame 101 | Kéo keyframe bắt đầu về frame 80 khi xe vừa lộ diện sau xe buýt | fixed |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | clip_01 có 8 track hợp lệ, clip_02 có 6 track hợp lệ |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | ID switch = 0 trên cả 2 clip, IDF1 đạt > 0.95 |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | Xe buýt và xe con đi ngang qua giữ nguyên ID ổn định |
| Entry/exit đúng; không box treo sau khi xe rời khung | PASS | Đã bấm Outside (phím O) đúng frame xe biến mất |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | PASS | Bbox chạm mép ảnh, LocA đạt 0.88+ |
| Frame giữa hai keyframe không bị interpolation drift | PASS | Đã tua rà soát frame 81, 82 và chỉnh khít IoU |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | Đã chạy check_mot_labels.py đạt 0 lỗi |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | Cả 3 finding đều có closure fixed |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | ID xuyên suốt không đổi, 0 ID switch |
| 2 — endpoint/scope | ĐÃ SỬA | Đã chỉnh mốc xuất hiện của xe sau xe buýt về frame 80 |
| 3 — geometry/interpolation | ĐÃ SỬA | Đã xóa 2 track xe ở làn xa ngoài schema và nắn lại bbox mép ảnh |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: Cần xác định đúng ngưỡng kích thước xe tối thiểu (resolution/distance threshold). Xe quá xa và mờ ở làn sau cây cối không nên gán để tránh FP.
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do (nếu có): Track 3 (SUV trắng đỗ yên) ban đầu có cảnh báo đứng im, nhưng được đóng là not-a-defect vì xe đỗ hợp lệ vẫn là `vehicle` theo task spec.
3. Một rule cần Lab Coach làm rõ (nếu có): Ngưỡng diện tích pixel cụ thể cho các xe xuất hiện ở làn đường phụ phía xa trong các sequence phức tạp.
