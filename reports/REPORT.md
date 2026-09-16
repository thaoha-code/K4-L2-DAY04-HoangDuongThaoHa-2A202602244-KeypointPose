# Báo cáo Ngày 4 - Keypoint & Pose

Họ tên: Hoàng Dương Thảo Hà   Nhóm: [Điền nhóm]   Ngày: 16/09/2026

## 1. Nhãn của tôi

| Chỉ số                         |      Giá trị |
| -------------------------------- | -------------: |
| Số ảnh đã gán               |             20 |
| Số skeleton                     |             29 |
| v=2 / v=1 / v=0                  | 352 / 113 / 28 |
| Thời gian trung bình mỗi ảnh |       12 phút |

Ba khớp có `%v=1` cao nhất (chép từ `reports/visibility_report.md`):

1. `left_ear` (59%)
2. `right_ear` (41%)
3. `left_wrist` (31%)

Chúng có đúng là những khớp bạn thấy khó gán nhất không? Nếu không, giải thích.
Đúng. `left_ear` và `right_ear` có tỷ lệ bị che khuất rất cao do các nhân vật thường xoay mặt góc nghiêng hoặc bị tóc/mũ che lấp, phải ước lượng vị trí. Tương tự, `left_wrist` (cổ tay trái) cũng thường xuyên bị che khuất khi nhân vật thao tác cầm nắm đồ vật hoặc bị cơ thể che khuất.

## 2. Chấm với gold

| Chỉ số                | Trước rework | Sau rework |
| ----------------------- | -------------: | ---------: |
| OKS trung bình         |          0.919 |      0.928 |
| OKS@0.50                |          0.931 |      1.000 |
| OKS@0.75                |          0.897 |      0.966 |
| Lỗi`dao_trai_phai`   |              0 |          0 |
| Lỗi`nham_nguoi`      |              1 |          0 |
| Lỗi`xoa_khop_bi_che` |              4 |          0 |

**Tôi đã sửa gì giữa hai lần chạy**:

- `train_04.jpg`, người #1, `left_wrist`: Kéo trả điểm cổ tay trái về đúng cơ thể (khắc phục lỗi nhầm người).
- `train_13.jpg`, người #1 và #2, toàn bộ khớp: Vẽ bổ sung 2 skeleton bị bỏ sót (khắc phục lỗi thiếu người).
- `train_10.jpg`, người #1, `left_hip` & `right_hip`: Đặt thêm chấm ước lượng vị trí hông bị quần áo che khuất thay vì bỏ trống.
- `train_11.jpg`, người #1, `left_hip` & `right_hip`: Đặt thêm chấm ước lượng vị trí hông bị quần áo che khuất thay vì bỏ trống.
- `train_12.jpg`, người #1, nhiều khớp: Căn chỉnh lại tọa độ các khớp cho chính xác hơn để khắc phục lỗi lệch nhẹ.

**Lỗi đảo trái/phải của tôi xảy ra ở ảnh nào?**
Không có lỗi đảo trái/phải trong toàn bộ 20 ảnh.

## 3. Kiểm chéo

Bạn cùng nhóm: ______

Khớp lệch `%v=1` nhiều nhất giữa hai bảng đếm:

| Khớp | Bạn | Họ | Lệch | Nguyên nhân (guideline hay gán sai?) |
| ----- | ---: | --: | ----: | --------------------------------------- |
|       |      |     |       |                                         |
|       |      |     |       |                                         |

Luật mới đã bổ sung vào `GUIDELINE_MINI.md` sau khi thống nhất:

## 4. Model

| Chỉ số       | yolo26n-pose gốc | Sau fine-tune |  Chênh |
| -------------- | ----------------: | ------------: | ------: |
| pose_mAP50     |            0.8450 |        0.8450 | +0.0000 |
| pose_mAP50-95  |            0.6853 |        0.6908 | +0.0055 |
| pose_precision |            0.9734 |        0.9792 | +0.0058 |
| pose_recall    |            0.8462 |        0.8462 | +0.0000 |
| box_mAP50-95   |            0.8119 |        0.8041 | -0.0078 |

### Trả lời năm câu hỏi ở cuối notebook

1. `pose_mAP50-95` thay đổi bao nhiêu?
   `pose_mAP50-95` tăng nhẹ 0.0055 (từ 0.6853 lên 0.6908). Việc chỉ dùng 20 ảnh là rất nhỏ, nhưng model cũng đã học được thêm một chút về đặc trưng tạo dáng từ nhãn để cải thiện độ chính xác tọa độ khớp, dù việc này làm giảm nhẹ khả năng bắt box tổng thể (`box_mAP` giảm 0.0078).
2. `box_mAP` và `pose_mAP` chênh nhau bao nhiêu? Model tìm *người* dễ hơn hay tìm *khớp* dễ hơn? Vì sao?
   Chênh lệch giữa `box_mAP50-95` (0.8041) và `pose_mAP50-95` (0.6908) là khoảng 0.1133. Model tìm *người* (bounding box) dễ hơn nhiều so với tìm *khớp* (pose). Vì khoanh một vùng chứa toàn bộ cơ thể người là tác vụ tổng quát và dễ nhận diện hơn so với việc định vị chính xác tọa độ x, y của 17 khớp nối bé xíu bên trong, nhất là khi tay chân hay bị vắt chéo hoặc che khuất.
3. Một ảnh test model đoán sai: Ảnh ở test_03, lỗi lệch nhẹ (chấm gần đúng khớp).
4. Ảnh nào có OKS thấp nhất giữa nhãn của bạn và model? Ảnh có OKS thấp nhất là **`train_12.jpg`**
5. Ảnh bạn gán tệ nhất có cũng là ảnh model đoán tệ nhất không? Ảnh gán tệ nhất là (`train_12.jpg`) cũng là ảnh model dự đoán tệ nhất. Điều này cho thấy bức ảnh có độ khó rất cao. Khi một dữ liệu đầu vào quá nhiễu hoặc bất thường, cả con người (người gán nhãn) và Trí tuệ nhân tạo (model) đều gặp khó khăn trong việc định vị chính xác tọa độ khớp.

<!-- Viết một rule kiểm chứng được: điều kiện nhìn thấy/căn cứ vị trí → chọn v=1 hoặc v=0.
Không chỉ ghi “cẩn thận hơn khi gán”. -->

## 5. Một rule evidence bạn đã dùng

Tại `train_01.jpg`, người người đàn ông, khớp `right_hip`. Mặc dù phần hông bị chiếc bánh pizza và cánh tay che khuất, nhưng toàn bộ thân người vẫn đang hiển thị rõ trong khung hình. Do đó, tôi quyết định chọn ước lượng đặt chấm dựa trên đường gióng từ vai phải `right_shoulder` đi thẳng xuống.
