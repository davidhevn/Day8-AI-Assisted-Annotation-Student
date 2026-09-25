# Vì sao chọn lô này?

Trong 50 dòng đứng đầu `outputs/selection_round1.csv`, chọn năm frame bạn sẽ ưu tiên nếu chỉ có ngân sách rà năm ảnh. Ghi tên, điểm, thời điểm, thứ tự và lý do; tối thiểu một quyết định phải xét ảnh gần trùng hoặc trường hợp model không dự đoán được box:
1. `frame_0182.jpg` (Hạng 1, Score: 0.9591, t=72.8): Điểm bất định (Uncertainty) cao nhất, AI có quá nhiều box phân vân (18 box mơ hồ).
2. `frame_0369.jpg` (Hạng 2, Score: 0.9324, t=147.6): Điểm cao thứ hai, xe đông (43 box), nhiều box bị nhập nhằng.
3. `frame_0380.jpg` (Hạng 3, Score: 0.9170, t=152.0): Bối cảnh giao thông phức tạp, AI không chắc chắn.
4. `frame_0326.jpg` (Hạng 4, Score: 0.9155, t=130.4): Độ đa dạng tốt (D=1.0).
5. `frame_0331.jpg` (Hạng 5, Score: 0.9154, t=132.4): Phủ thêm thông tin mới về bối cảnh.
(Đặc biệt loại bỏ ảnh Hạng 6 là `frame_0372.jpg` dù điểm cao vì nó quá gần thời gian với hạng 2).

Ba frame thuộc lô 12 ảnh model chọn và bằng chứng trong CSV/ảnh contact sheet:
- `frame_0182.jpg`: (Hạng 1, selected=True) Model ưu tiên tuyệt đối vì điểm bất định tổng hợp cao nhất lô.
- `frame_0369.jpg`: (Hạng 2, selected=True) Chứa rất nhiều object gây nhiễu, điểm số 0.932.
- `frame_0099.jpg`: (Hạng 8, selected=True) Dù điểm chỉ 0.906 nhưng thời gian t=39.6 (nằm ở đoạn đầu video), đảm bảo tính phân tán đa dạng (Diversity) thay vì chỉ tập trung ở cuối video.

Một frame có điểm cao nhưng không chọn hoặc một frame có điểm thấp vẫn nên xem, và lý do:
- Không chọn `frame_0372.jpg` (Hạng 6, Score 0.9101, Selected=False): Nằm ở thời điểm t=148.8s, quá gần với ảnh `frame_0369.jpg` (t=147.6s) đã được chọn trước đó. Việc gán nhãn 2 ảnh quá sát nhau (ảnh gần trùng) sẽ làm tăng chi phí rà nhãn vô ích mà model không học thêm được thông tin gì mới.

Điều phép chọn này chưa chứng minh về chất lượng mô hình:
- Phép chọn dựa vào điểm bất định (Uncertainty Sampling) chỉ giúp tìm ra những ảnh model "đang bối rối", nhưng không thể phát hiện những ảnh mà model "tự tin nhưng bị sai" (False Positives tự tin). Nó cũng không đánh giá được hiệu suất tổng thể của model trên bối cảnh chung, dễ (Easy cases).
