# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: Đỗ Trung Kiên - MSSV: 2A202602283 (Khóa 4) - Làm cá nhân
Ngày: 15/09/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | 30 phút |
| Thời gian gán `clip_01` | 80 phút |
| Số track đã vẽ trong `clip_01` | 8 track (sau rework; 10 track ở pre-gold) |
| Số keyframe trung bình mỗi track | 6 keyframe / track |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe lướt nhanh ra khỏi khung hình (Track 1 ở góc dưới trái):** Xe chỉ xuất hiện vỏn vẹn 11 frame rồi đi ra mép ảnh, cần bấm chính xác phím Outside (`O`) ngay tại frame xe rời khung để không để lại bbox treo rác ở các frame sau.
2. **Xe con bám sát sau đuôi xe lớn bị che khuất (Track 5):** Xe con màu xám bám sau xe buýt, ở các frame 79–85 chỉ lộ diện một phần đầu xe. Xử lý bằng cách vẽ bbox ôm sát phần đầu nhìn thấy được và đặt keyframe dày hơn (cách 3–5 frame) khi xe dần lộ rõ.
3. **Xe SUV đỗ bất động suốt 190 frame (Track 3):** Xe đứng yên ở làn giữa, dễ bị nhầm là không cần track hoặc bấm nhầm Shape. Xử lý bằng cách tạo Rectangle Track duy nhất và giữ nguyên ID từ frame 1 đến 190.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Quan sát tập trung vào con số ID trên các xe; xác nhận không có xe nào bị đổi số giữa chừng (0 ID switch), ID của các xe giữ ổn định 100%.
- Lượt 2: Soi frame đầu và frame cuối của từng track; phát hiện xe Track 5 (bám sau xe buýt) bị vẽ bắt đầu muộn ở frame 101 và đã chỉnh kéo về frame 80; rà soát toàn bộ các điểm kết thúc đảm bảo đã bấm Outside.
- Lượt 3: Tua vào giữa các khoảng cách keyframe; phát hiện hai xe ở làn xa tít đằng sau rặng cây (Track 4 và 5 cũ) là nguyên nhân gây ra 111 FP thừa, đã quyết định loại bỏ để khớp với schema của Gold reference.

Kiểm chéo / Tự kiểm: Làm cá nhân (Solo), áp dụng quy trình **Self-QC 3 lượt tua** độc lập. Chi tiết tại `reports/review_partner.md`.
Số lỗi tự phát hiện qua 3 lượt tua: 3 lỗi chính (2 track ngoài schema và 1 track thiếu đoạn).
Các lỗi phát hiện được xử lý theo closure: `fixed` trong `reports/review_partner.md`.

Ca mơ hồ nào tự phát hiện trong quá trình tua kiểm tra, và luật nào đã được bổ sung / làm rõ trong `GUIDELINE_MINI.md`?

Quy chuẩn ngưỡng kích thước tối thiểu cho các xe ở làn đường phụ phía xa: Xe ở quá xa (<35px) và bị che khuất bởi rặng cây ở làn đường đối diện không nên gán để tránh gây nhiễu và dính False Positive. Luật này đã được bổ sung làm rõ vào Mục 1 và Mục 3 của `GUIDELINE_MINI.md`.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `bc6674a5c03871493c5acb2c2ea0566f505a9d9b492c543f44a7188a123a2f40` |
| Thời điểm khóa | `2026-09-15T09:48:49.181356+00:00 (16:48:49 GMT+7)` |
| Số row / frame / track trước khi mở reference | 698 rows / 190 frames / 10 tracks |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.756 | 0.687 | 0.835 | 0.881 | 0.883 | 0.740 | 0.870 | 137 | 12 | 0 |
| Sau rework | 0.834 | 0.824 | 0.844 | 0.882 | 0.974 | 0.949 | 0.872 | 6 | 23 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có (ĐẠT XUẤT SẮC cả 3 tiêu chí)**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox thừa (FP - ngoài schema) | 1–80 | Track 4 | Xóa track xe con ở làn đường xa phía sau cây do nằm dưới ngưỡng kích thước của Gold |
| Bbox thừa (FP - ngoài schema) | 3–33 | Track 5 (cũ) | Xóa track xe nhỏ ở góc xa bên trái do Gold không gán xe ngoài lề |
| Bbox thừa (FP - bắt đầu sớm) | 81–100 | Track 8 (cũ) | Cắt bỏ 20 frame đầu, chuyển điểm bắt đầu track về frame 101 khi xe vào rõ nét |
| Thiếu đoạn (FN) | 80–100 | Track 5 (mới) | Kéo điểm bắt đầu về frame 80 bám sát sau đuôi xe buýt để phủ đủ 100% quãng đời xe |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 |
| weights / hai tracker | yolo26n.pt / bytetrack.yaml (control) & botsort-reid.yaml (treatment) |
| conf / IoU / imgsz / classes | conf: 0.25 / IoU: 0.7 / imgsz: 960 / classes: [2, 5, 7] |
| device | 0 (Tesla T4 GPU) |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bạn vs gold | 0.834 | 0.824 | 0.844 | 0.882 | 0.974 | 0.949 | 0.872 | 6 | 23 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.764 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.859 | 91 | 26 | 2 |
| ReID vs bạn | 0.771 | 0.707 | 0.846 | 0.873 | 0.910 | 0.808 | 0.856 | 94 | 12 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

- Trong kết quả của tôi (sau rework): IDF1 đạt 0.974 và MOTA đạt 0.949 (cả hai đều đạt mức xuất sắc $> 0.90$).
- Về bản chất lý thuyết: Nếu một hệ thống có **MOTA cao mà IDF1 thấp**, điều đó phản ánh rằng detector phát hiện và định vị các box rất tốt (rất ít FP và FN), nhưng thuật toán theo dõi bị lỗi **phân mảnh danh tính hoặc tráo đổi ID liên tục (ID switch / fragmentation)**.
- Lý do MOTA không phạt nặng lỗi ID: Theo công thức $\text{MOTA} = 1 - \frac{\text{FP} + \text{FN} + \text{IDSW}}{\text{GT}}$, mỗi lần xảy ra ID switch chỉ bị tính là **1 lỗi đơn lẻ** (cộng 1 vào tử số), bất kể sau đó xe tiếp tục chạy thêm 50 hay 100 frame dưới ID sai. Ngược lại, $\text{IDF1}$ đánh giá sự nhất quán danh tính trên **toàn bộ quãng đời (trajectory)** của đối tượng — nếu một track bị chia đôi ở giữa, một nửa số frame đó sẽ bị coi là IDFP/IDFN, khiến IDF1 bị tụt dốc thê thảm trong khi MOTA vẫn giữ ở mức cao.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- So sánh hai mô hình khi đối chiếu với Gold:
  - **IDF1:** BoT-SORT + ReID đạt **0.900**, vượt trội hơn ByteTrack (**0.875**).
  - **AssA:** BoT-SORT + ReID đạt **0.820**, cao hơn ByteTrack (**0.776**).
  - **IDSW:** Cả hai đều có 2 lần ID switch.
  - **HOTA tổng thể:** BoT-SORT + ReID đạt **0.764** so với **0.709** của ByteTrack.
- Minh chứng qua chuỗi frame (frame sequence):
  - Ở đoạn frame 80–120 (khi chiếc xe con bám sau xe buýt bị che khuất một phần và đi ngang qua dải phân cách): ByteTrack thuần dựa trên chuyển động (Kalman Filter + IoU overlap) gặp khó khăn khi IoU giữa các frame bị sụt giảm hoặc chuyển động phi tuyến tính. Ngược lại, BoT-SORT tích hợp vector đặc trưng ngoại hình (ReID appearance embedding) và cơ chế bù trừ chuyển động camera (CMC) giúp duy trì liên kết track ổn định hơn, giảm thiểu tình trạng đứt đoạn track (FN giảm từ 54 xuống 26).
- **Lưu ý phương pháp luận:** Đây là so sánh ở cấp độ toàn bộ hệ thống (system-level comparison). Ta **không thể cô lập causal effect duy nhất của ReID**, bởi vì ByteTrack và BoT-SORT là hai implementation khác nhau (BoT-SORT còn có module Camera Motion Compensation, ma trận khoảng cách kết hợp và chiến lược phân bổ track khác với ByteTrack).

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- **DetA:** Tăng từ 0.649 (ByteTrack) lên 0.711 (BoT-SORT + ReID).
- **FN:** Giảm mạnh hơn một nửa, từ 54 frame xuống còn 26 frame (ReID giữ được track lâu hơn nên ít bị mất dấu đối tượng).
- **FP:** Tăng nhẹ từ 88 lên 91.
- **Đánh giá nguồn lỗi:** Cả hai tracker đều sử dụng chung detector `yolo26n.pt` (zero-shot trên COCO). Lỗi còn lại chủ yếu nằm ở **Detector**: 
  - Detector sinh ra lượng FP khá lớn (~88–91 box) do nhận diện nhầm các vật thể tĩnh và chi tiết ở làn đường xa thành phương tiện.
  - Các FN còn lại cũng do detector không bắt được xe khi bị che khuất quá nặng.
  - Ngược lại, lỗi từ **Association** rất nhỏ (chỉ có 2 IDSW trên toàn bộ 190 frame).

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

- **Tại Track 1 (Gold Track 2) ở frame 1–11:** Chiếc xe con ở góc dưới bên trái lướt nhanh ra khỏi khung hình ở frame 11. Nhãn tay của tôi theo sát chuyển động và bấm Outside (`O`) kết thúc track chính xác tại frame 11 (khớp hoàn hảo với Gold, 0 lỗi FP).
- Trong khi đó, ReID model bị FP ở các vùng rìa (ví dụ track ma `pred_track 10` xuất hiện ở các frame đầu và kéo dài tới frame 116 mà không khớp vật thể nào trong Gold). Nhãn con người hiểu được ngữ cảnh biên ảnh tốt hơn model.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

- **Tại frame 101–110 (Track 6 trong Gold):** Chiếc xe con đi từ góc phải vào. Ban đầu ở bản Pre-gold, tôi đã bắt đầu vẽ xe này từ frame 81 lúc nó mới nhú một đốm nhỏ ở mép xa.
- Khi đối chiếu với model ReID và Gold, model cũng chỉ bắt đầu track xe này từ frame 101 khi xe đã tiến sâu vào làn đường và có hình dáng 4 bánh rõ ràng. Điều này làm tôi xem lại quy chuẩn nhận diện kích thước tối thiểu và thực hiện rework cắt bỏ 20 frame xuất hiện sớm, giúp MOTA nhảy vọt lên 0.949.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- **Trong `GUIDELINE_MINI.md`:** 
  1. Bổ sung quy định định lượng rõ ràng về ngưỡng kích thước tối thiểu (chỉ gán xe có diện tích $> 35 \times 25$ px) và phạm vi không gian (chỉ gán xe ở các làn đường chính phía trước, không gán làn đường phụ phía sau rặng cây).
  2. Nêu rõ quy tắc cho xe bám đuôi: bắt đầu gán ngay khi diện tích xe lộ ra $> 20\%$.
- **Trong quy trình làm việc:**
  1. Xem lướt toàn bộ clip 1 lần trước khi gán để đếm số lượng xe và nhận diện các xe đỗ tĩnh.
  2. Sử dụng chiến thuật Object-centric triệt để: Gán xong trọn vẹn 1 xe từ lúc xuất hiện đến lúc bấm Outside (`O`) rồi mới chuyển sang xe tiếp theo.
  3. Áp dụng quy trình Self-QC 3 lượt ngay sau khi export để phát hiện sớm các lỗi trôi box hoặc quên bấm outside.

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
