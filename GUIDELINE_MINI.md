# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: Đỗ Trung Kiên - MSSV: 2A202602283 (Khóa 4)
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): Chỉ gán các xe di chuyển trên mặt đường chính hoặc đỗ trong khung nhìn rõ; loại bỏ các xe ở quá xa/mờ (chiều dài hoặc rộng < 35px) ở làn đường đối diện phía sau cây cối.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che dưới 25 frame (2 giây @ 12.5 fps) | Tránh ID switch khi xe tạm thời bị che bởi cột biển báo hoặc xe khác |
| Xe bị che lâu hơn ngưỡng trên | Mở track mới | Sau thời gian dài, chuyển động và vị trí không còn đảm bảo cùng một đối tượng |
| Xe rời khung hình rồi quay lại | Mặc định: **track mới** | Đã ra khỏi khung hình là kết thúc vòng đời của track đó |
| Hai xe cắt nhau / chồng lên nhau | Giữ nguyên ID từng xe, bbox chỉ ôm phần nhìn thấy của xe ở trước và phần hở của xe ở sau | Bảo toàn tính liên tục của trajectory, không tráo đổi ID giữa 2 xe |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa ảnh, không tự đoán phần thân xe nằm ngoài ảnh |
| Xe bị xe khác che một phần | Bbox chỉ ôm sát phần **nhìn thấy được**, bật cờ Occluded nếu bị che > 30% |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định rõ là xe 4 bánh (ngưỡng diện tích > 35x25 px) |
| Xe đang đỗ, không di chuyển | Vẫn giữ nguyên 1 track và cùng 1 ID từ frame đầu đến frame cuối của video |
| Keyframe đặt dày ở đâu | Đặt dày (cách 3–5 frame) ở khúc rẽ, phanh, đổi hướng, hoặc khi bắt đầu/kết thúc bị che khuất |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01` / Frame 1–190 / Track 3 (SUV trắng đỗ yên)
- Tình huống: Chiếc SUV màu trắng đỗ ở làn giữa đứng yên suốt 190 frame, không di chuyển.
- Quyết định: Vẫn tạo một Rectangle Track và gán từ frame 1 đến 190 với cùng 1 ID.
- Lý do: Theo schema bài lab, xe đang đỗ vẫn là `vehicle` hợp lệ và cần giữ nguyên ID suốt thời gian nó xuất hiện trong khung hình.

### Ca 2
- Clip / frame / ID: `clip_01` / Frame 1–80 / Track 4 cũ (xe ở làn xa)
- Tình huống: Một chiếc xe con nhỏ di chuyển ở làn đường phía xa tít đằng sau rặng cây và cột biển báo ($x \approx 557$ về $x \approx 58$).
- Quyết định: Ban đầu gán (Pre-gold), sau khi đối chiếu với Gold reference thì loại bỏ (Rework xóa track này).
- Lý do: Xe ở quá xa, kích thước nhỏ dưới ngưỡng phân giải mục tiêu của dataset ($< 35$ px) và bị che khuất nhiều bởi cây cối nên Gold reference không gán. Việc gán tạo ra 80 FP.

### Ca 3
- Clip / frame / ID: `clip_01` / Frame 79–100 / Track 5 (Xe con bám đuôi xe buýt)
- Tình huống: Chiếc xe con màu xám bám sát ngay sau đuôi xe buýt lớn khi xe buýt đi qua ở frame 79–80.
- Quyết định: Bắt đầu vẽ keyframe cho xe này từ frame 80 khi đuôi xe buýt vừa để lộ đầu xe con, theo dõi liên tục đến frame 138 khi xe rời khung.
- Lý do: Xe di chuyển liên tục cùng luồng giao thông chính; nếu bỏ lỡ 20 frame đầu sẽ phạm lỗi "Thiếu đoạn" (chỉ phủ được 63% vòng đời).

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Làm rõ quy chuẩn về **kích thước tối thiểu và khoảng cách vật thể**: Chỉ gán các xe tham gia giao thông ở các làn đường chính phía trước; không gán các xe ở làn đường phụ phía xa tít đằng sau rặng cây để tránh gây nhiễu và dính False Positive.
- Chuẩn hóa mốc bắt đầu của xe bám đuôi: Khi xe đi sau xe lớn bị che khuất, ngay khi lộ diện $\ge 20\%$ diện tích đầu xe là phải bắt đầu track ngay lập tức, không chờ đến khi xe ra giữa đường mới bắt đầu vẽ.
