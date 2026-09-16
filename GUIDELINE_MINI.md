# Mini guideline - nhóm: K4-L2  |  người gán: Trần Minh Hiếu  |  ngày: 16/09/2026

> Điền file này **trong lúc** gán nhãn, không phải sau khi xong. Mỗi lần bạn dừng lại
> hơn 10 giây để phân vân, đó là một dòng phải ghi vào đây.

## 1. Luật bắt buộc (đã thống nhất cả lớp - không sửa)

- Bộ 17 điểm COCO, đúng tên, đúng thứ tự. Lấy từ file `.SVG` chung.
- Mọi người trong ảnh đều có **đủ 17 điểm**. Điểm không dùng được thì gắn cờ, không xoá.
- Trái/phải tính theo **cơ thể người**, không theo bức ảnh.
- Bị che, còn trong khung -> `v = 1`, **vẫn đặt chấm** ở vị trí ước lượng.
- Ra ngoài mép ảnh -> `v = 0`, **không** đặt chấm.
- Không dùng `Hidden` (`h`) - nó không được lưu vào file.

## 2. Luật của nhóm bạn <Không có nhóm>

## 3. Ba ca mơ hồ đã gặp (bắt buộc, ghi ít nhất 3)

### Ca 1 - ảnh `train_04.jpg`, người thứ `1`, khớp `left_wrist`

- **Mơ hồ ở chỗ nào:** Bàn tay và cổ tay trái của nhân vật đang gập và đút hoàn toàn vào trong túi áo khoác, không thấy bất kỳ phần da hay bề mặt khớp nào.
- **Bạn quyết thế nào:** Chọn `v = 1`, chấm điểm ước lượng tại vị trí nếp gấp mép túi áo khoác (nơi cẳng tay đi vào túi).
- **Vì sao:** Cánh tay và vạt túi áo vẫn nằm gọn trong khung hình (cách biên ảnh khoảng 15%). Khớp chỉ bị che bởi vải áo chứ chưa ra khỏi mép ảnh.
- **Nếu người khác quyết ngược lại (chọn `v = 0`):** Model sẽ học sai quy tắc visibility: hiểu nhầm rằng cứ không thấy bề mặt khớp là đặt `v=0`, dẫn đến bỏ sót thông tin không gian keypoint khi huấn luyện các ca bị che khuất (occlusion).

### Ca 2 - ảnh `train_01.jpg`, người thứ `1`, khớp `left_ankle`

- **Mơ hồ ở chỗ nào:** Nhân vật đứng ở viền dưới bức ảnh, phần bàn chân và cổ chân chạm sát mép dưới khung hình.
- **Bạn quyết thế nào:** Chọn `v = 0`, không đặt chấm tọa độ.
- **Vì sao:** Khi phóng to trên CVAT, tâm điểm giải phẫu của cổ chân nằm chớm phía dưới đường viền cắt của bức ảnh ($y > 1.0$).
- **Nếu người khác quyết ngược lại (chọn `v = 1`):** Model sẽ bị nhiễu do học một tọa độ ảo nằm ngoài biên ảnh hoặc ép tọa độ về mép $(y=1.0)$, gây lỗi gán tọa độ trượt biên.

### Ca 3 - ảnh `train_10.jpg`, người thứ `1`, khớp `left_hip` / `right_hip`

- **Mơ hồ ở chỗ nào:** Nhân vật mặc áo phông dáng rộng trùm qua đùi, toàn bộ vùng chậu và khớp hông không có ranh giới rõ ràng.
- **Bạn quyết thế nào:** Chọn `v = 1`, chấm điểm ước lượng dựa vào trục đường vai gáy kéo thẳng xuống và độ gập của đùi.
- **Vì sao:** Vùng hông nằm chính giữa trung tâm bức ảnh, hoàn toàn nằm trong khung hình nhưng bị trang phục phồng che khuất.
- **Nếu người khác quyết ngược lại (xóa khớp/chọn `v = 0`):** Tạo ra lỗi `xoa_khop_bi_che`. Model sẽ bị mất tín hiệu giám sát OKS ở vùng trung tâm cơ thể, khiến khả năng dự đoán gốc xương chậu khi fine-tune bị suy giảm.

## 4. Sau khi so visibility report với bạn cùng nhóm

- Khớp lệch `%v=1` nhiều nhất: `right_wrist` (bạn `32%` / họ `18%`)
- Nguyên nhân là **guideline chưa rõ** hay **một trong hai bên gán sai**: Guideline chưa làm rõ ràng ranh giới giữa khớp bị khuất sau lưng/vật thể (`v=1`) với khớp bị đẩy hẳn ra ngoài biên ảnh (`v=0`).
- Luật mới bổ sung vào mục 2 sau khi thống nhất:
  - **Rule phân định v=1 và v=0:** Nếu khớp bị che bởi bộ phận khác của cơ thể, quần áo hoặc vật thể nhưng vị trí giải phẫu vẫn nằm bên trong biên bức ảnh, bắt buộc phải ước lượng vị trí và đặt cờ `v=1`. Chỉ đặt `v=0` khi khớp thực sự bị cắt ra khỏi rìa khung hình ($x < 0, x > 1, y < 0, y > 1$).