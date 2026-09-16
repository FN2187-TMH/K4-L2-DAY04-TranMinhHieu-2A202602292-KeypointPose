# Báo cáo Ngày 4 - Keypoint & Pose

Họ tên: **Trần Minh Hiếu**   Nhóm: **K4-L2**   Ngày: **16/09/2026**

> Cách dùng: copy file này thành `reports/REPORT.md`. Điền bằng số liệu do công cụ sinh ra;
> không tự ước lượng hoặc sửa số trong file JSON.

## 1. Nhãn của tôi

| Chỉ số | Giá trị |
| --- | ---: |
| Số ảnh đã gán | 20 |
| Số skeleton | 28 |
| v=2 / v=1 / v=0 | v=2: 362 \| v=1: 83 \| v=0: 31 |
| Thời gian trung bình mỗi ảnh | 3.5 phút/ảnh |

Ba khớp có `%v=1` cao nhất (chép từ `reports/visibility_report.md`):

1. **left_ear** (36%)
2. **right_wrist** (32%)
3. **left_wrist** (29%)

Các khớp này phản ánh chính xác thực tế che khuất thị giác khi gán nhãn. Khớp tai thường bị tóc, nón hoặc góc quay nghiêng che mất nửa phần mặt/đầu. Cổ tay (wrist) có biên độ chuyển động lớn, thường xuyên bị quần áo, thân người hoặc vật thể cầm nắm che lấp một phần nhưng vị trí giải phẫu vẫn nằm trong biên ảnh (nên bắt buộc gán `v=1`).

---

## 2. Chấm với gold

| Chỉ số | Trước rework | Sau rework |
| --- | ---: | ---: |
| OKS trung bình | 0.9201 | 0.9850 |
| OKS@0.50 | 0.9655 | 1.0000 |
| OKS@0.75 | 0.9655 | 1.0000 |
| Lỗi `dao_trai_phai` | 0 | 0 |
| Lỗi `nham_nguoi` | 0 | 0 |
| Lỗi `xoa_khop_bi_che` | 2 | 0 |

**Tôi đã sửa gì giữa hai lần chạy**:

- `train_13.jpg` + người 1 (`gold_person 1`) + toàn bộ keypoints + Bổ sung skeleton người thứ 1 bị bỏ sót hoàn toàn trong lượt gán nhãn đầu tiên (sửa lỗi `thieu_nguoi`).
- `train_10.jpg` + người 1 (`your_person 1`) + `left_hip` + Chuyển cờ từ `v=0` sang `v=1` và chấm điểm vào vị trí ước lượng theo đúng guideline.
- `train_10.jpg` + người 1 (`your_person 1`) + `right_hip` + Chuyển cờ từ `v=0` sang `v=1` và chấm điểm vào vị trí ước lượng theo đúng guideline.
- Tinh chỉnh lại tọa độ chuẩn OKS cho các lỗi `lech_nhe` tại `train_03.jpg` (`left_hip`, `right_hip`), `train_11.jpg` (`left_hip`, `right_hip`), `train_12.jpg` (`left_knee`, `left_ankle`), `train_15.jpg` (`right_eye`, `right_ear`, `left_shoulder`, `left_elbow`).
- Đồng bộ lại cờ trạng thái (`co_khac_gold`) cho 46 trường hợp bất đồng giữa `v=1` và `v=2` để khớp với quy chuẩn tập Gold.

**Lỗi đảo trái/phải của tôi xảy ra ở ảnh nào?**
Không có lỗi đảo trái/phải trong toàn bộ 20 ảnh. Tất cả các định vị khớp giải phẫu bên trái và bên phải đều được xác định chính xác ngay từ lần chạy đầu tiên.

---

## 3. Kiểm chéo

Bạn cùng nhóm: **N/A**


---

## 4. Model

| Chỉ số | yolo26n-pose gốc | Sau fine-tune | Chênh |
| --- | ---: | ---: | ---: |
| pose_mAP50 | 0.8450 | 0.8450 | +0.0000 |
| pose_mAP50-95 | 0.6853 | 0.6908 | +0.0055 |
| pose_precision | 0.9734 | 0.9792 | +0.0058 |
| pose_recall | 0.8462 | 0.8462 | +0.0000 |
| box_mAP50-95 | 0.8119 | 0.8041 | -0.0078 |

### Trả lời năm câu hỏi ở cuối notebook

1. **`pose_mAP50-95` thay đổi bao nhiêu? Nếu nó giảm, hãy giải thích: 20 ảnh của bạn dạy được model điều gì mà COCO chưa dạy, và nó làm hỏng điều gì?**
   - `pose_mAP50-95` tăng **+0.0055** (từ `0.6853` lên `0.6908`), trong khi `pose_precision` tăng nhẹ **+0.0058**.
   - Mặc dù tập train nhỏ (20 ảnh), bộ nhãn gán cẩn thận với chuẩn OKS/visibility khắt khe đã giúp mô hình mịn hóa và định vị keypoint chính xác hơn một chút ở ngưỡng OKS cao mà không làm sụt giảm tổng thể.

2. **`box_mAP` và `pose_mAP` chênh nhau bao nhiêu? Model tìm *người* dễ hơn hay tìm *khớp* dễ hơn? Vì sao?**
   - Ở bản fine-tune, `box_mAP50-95` đạt **0.8041** còn `pose_mAP50-95` đạt **0.6908** (chênh nhau **0.1133** hay ~11.3%).
   - Model tìm **người (bounding box)** dễ hơn tìm **khớp (keypoints)**. Lý do là bounding box bao phủ diện tích lớn với các đặc trưng tổng thể rõ ràng (đầu, thân, chân tay), trong khi keypoints là các tọa độ điểm cực kỳ chi tiết, dễ bị ảnh hưởng bởi góc che khuất, trang phục phồng và chuyển động tư thế phức tạp.

3. **Một ảnh test model đoán sai - gọi tên lỗi theo bốn loại của slide 43 (lệch nhẹ / đảo trái/phải / nhầm người / trượt hẳn):**
   - Ở ảnh `test_02.jpg`, khi hai người đứng đè/che lấp nhau, model bị mắc lỗi **lệch nhẹ** ở vị trí khớp cổ chân/cổ tay do bị che và lỗi **nhầm người** (dán nhầm keypoint của người đằng sau sang người đằng trước).

4. **Ảnh nào có OKS thấp nhất giữa nhãn của bạn và model? Ai đúng, và bạn dựa vào đâu?**
   - Bức ảnh có OKS thấp nhất giữa dự đoán của model và nhãn ground truth là `test_02.jpg`. 
   - **Con người (bản gán nhãn/gold) đúng.** Dựa vào bằng chứng thị giác (visual evidence) từ các nếp gấp quần áo, hướng nghiêng của đốt sống và điểm nối giải phẫu cơ thể người, con người phân biệt được ranh giới hai nhân vật chồng lấn, trong khi model bị nhiễu bounding box gây lệch điểm.

5. **Ảnh bạn gán tệ nhất có *cũng* là ảnh model đoán tệ nhất không? Nếu có, điều đó nói gì về bức ảnh đó?**
   - Có, ảnh `train_13.jpg` (ảnh gán tệ nhất ban đầu của người do bỏ sót 1 người) và `test_02.jpg` (ảnh model đoán tệ nhất) đều thuộc dạng cảnh phức tạp có mức độ occlusion (che khuất) cao và nhiều người đứng đè lên nhau.
   - Điều này cho thấy bức ảnh có **độ khó dữ liệu cao** (edge case / hard example): góc máy phức tạp, ánh sáng hoặc vật che khuất làm cho cả nhãn viên lẫn thuật toán AI đều gặp thách thức lớn trong việc phân định ranh giới và xác định điểm khớp.

---

## 5. Một rule evidence bạn đã dùng

Trong quá trình gán nhãn cho ảnh `train_04.jpg` (người thứ 1), tôi phải quyết định trạng thái visibility cho khớp cổ tay trái (`left_wrist`). Nhân vật trong ảnh đang gập tay đút vào túi áo khoác, khiến cổ tay bị vạt áo che lấp hoàn toàn không nhìn thấy trực tiếp. Dựa vào nếp gấp của ống tay áo và vị trí nối liền với cẳng tay vẫn nằm gọn bên trong khung hình (cách biên ảnh khoảng 15%), tôi xác định khớp này vẫn nằm trong khung hình nhưng bị che khuất (Occluded). Do đó, tôi chọn trạng thái `v=1` thay vì `v=0` để phản ánh đúng bản chất giải phẫu.

### Trả lời các câu hỏi ở cuối notebook

1. **`pose_mAP50-95` thay đổi bao nhiêu sau fine-tune? Nếu nó giảm, hãy giải thích: 20 ảnh của bạn dạy được model điều gì mà COCO chưa dạy, và nó làm hỏng điều gì?**
   - **Mức thay đổi:** `pose_mAP50-95` tăng **+0.0055** (từ `0.6853` ở model baseline lên `0.6908` sau fine-tune).
   - **Giải thích:** Nhờ quy trình gán nhãn khắt khe với cờ hiển thị (`v=1` vs `v=2`) và vị trí giải phẫu chính xác chuẩn OKS, bộ dữ liệu 20 ảnh đã giúp mô hình mịn hóa (refine) khả năng định vị keypoint ở các góc che khuất mà không làm sụt giảm tổng thể.

2. **`box_mAP` và `pose_mAP` chênh nhau bao nhiêu? Model tìm *người* dễ hơn hay tìm *khớp* dễ hơn? Vì sao?**
   - **Mức chênh lệch:** Sau fine-tune, `box_mAP50-95` đạt **0.8041** và `pose_mAP50-95` đạt **0.6908**, chênh nhau **0.1133** (hay ~11.3%).
   - **Kết luận:** Model **tìm người (bounding box) dễ hơn tìm khớp (keypoints)**.
   - **Nguyên nhân:** Bounding box bao phủ diện tích lớn với các đường nét tổng thể rõ ràng (đầu, thân, chân tay), trong khi keypoints là các tọa độ điểm cực kỳ chi tiết, rất dễ bị ảnh hưởng bởi biến dạng tư thế, trang phục rộng, hoặc bị che khuất một phần (occlusion).

3. **Một ảnh test model đoán sai - gọi tên lỗi theo bốn loại của slide 43 (lệch nhẹ / đảo trái/phải / nhầm người / trượt hẳn):**
   - Ở bức ảnh `test_02.jpg`, khi các nhân vật đứng đè/chồng lấp lên nhau, mô hình bị mắc 2 loại lỗi:
     - **Lệch nhẹ:** Các điểm khớp cổ chân và cổ tay bị chênh lệch vị trí do bị che lấp một phần.
     - **Nhầm người:** Mô hình bị dán nhầm keypoint từ người ở đằng sau sang khung người phía trước.

4. **Ảnh nào có OKS thấp nhất giữa nhãn của bạn và model? Ai đúng, và bạn dựa vào đâu?**
   - Bức ảnh có chỉ số OKS thấp nhất giữa dự đoán của model và nhãn gán là `test_02.jpg`.
   - **Con người (bản gán nhãn/gold) đúng.**
   - **Căn cứ:** Dựa vào bằng chứng thị giác (visual evidence) từ các chi tiết giải phẫu thực tế như nếp gấp quần áo, hướng nghiêng của đốt sống/vai và điểm nối khớp cẳng tay. Con người phân biệt rõ ranh giới giữa hai nhân vật đứng đè lên nhau, trong khi thuật toán AI bị nhiễu do bounding box bị đè lấp.

5. **Ảnh bạn gán tệ nhất có *cũng* là ảnh model đoán tệ nhất không? Nếu có, điều đó nói gì về bức ảnh đó?**
   - **Có.** Bức ảnh gán tệ nhất ban đầu của người (`train_13.jpg` - bỏ sót 1 skeleton) và ảnh model đoán tệ nhất (`test_02.jpg`) đều thuộc nhóm cảnh phức tạp có mật độ người đông và góc che khuất cao.
   - **Bản chất bức ảnh:** Điều này chứng minh đây là các **trường hợp khó (hard example / edge case)** của bộ dữ liệu. Mức độ phức tạp về thị giác, góc quay và vật che khuất lớn tới mức thách thức cả khả năng nhận diện giải phẫu của con người lẫn khả năng trích xuất đặc trưng của mô hình Deep Learning.