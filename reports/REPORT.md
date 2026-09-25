# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: [Nguyễn Đại Hoàng]

Công cụ gán nhãn đã dùng: CVAT local, sửa tay trực tiếp.

## 1. Dữ liệu và cách chia tập

Tại sao tập chưa gán nhãn (pool) và tập kiểm thử (test set) được chia theo trục thời gian, có vùng đệm ở giữa, thay vì chia ngẫu nhiên?

- Việc chia theo trục thời gian và có vùng đệm giúp tránh rò rỉ dữ liệu (data leakage) giữa tập train và test. Trong video, các luồng xe chạy liên tục nên các frame đứng cạnh nhau sẽ gần như giống hệt nhau. Nếu chia ngẫu nhiên, các frame ở tập train và test bị trùng lặp, khiến kết quả đánh giá cao ảo do mô hình chỉ học thuộc vẹt (overfitting) thay vì học cách tổng quát hóa bối cảnh mới.

## 2. Mô hình khởi đầu lạnh (cold start)

Số đo vòng 0: AP50 = 0.771 | Precision = 0.925 | Recall = 0.489
Dựa vào `outputs/compare_round0.jpg` và các chỉ số recall theo kích thước (small=0.182, medium=0.547, large=0.561), mô hình khởi đầu lạnh bắt khá tốt các xe to và vừa ở gần, nhưng bỏ sót lượng lớn xe nhỏ ở xa (Recall xe nhỏ rất thấp).
Một trường hợp cần người rà lại nhãn tham chiếu trước khi kết luận mô hình sai: Những cụm xe quá nhỏ ở tít cuối đường bị mờ hoặc hòa vào bóng tối. Nhãn tham chiếu hiện tại là do AI tạo ra (không phải người tạo), do đó nó có thể sai từ đầu, cần đối chiếu với thực tế chứ không nên coi là chân lý tuyệt đối.

## 3. Chiến lược chọn mẫu

Công thức `score = W_U·U + W_A·A + W_D·D` giúp cân bằng 3 yếu tố để chọn ảnh: Bất định (Uncertainty - model bối rối), Nhập nhằng (Ambiguity - nhiều box đè nhau) và Đa dạng (Diversity - phân bố đều thời gian). `MIN_GAP_S` là thời gian tối thiểu giữa 2 ảnh để tránh chọn trùng.
Ví dụ trong `reports/SELECTION.md`: `frame_0182.jpg` được ưu tiên chọn vì điểm bất định cực cao. Trái lại, `frame_0372.jpg` dù điểm cao nhưng bị loại vì quá gần `frame_0369.jpg`. Việc gán nhãn 2 ảnh gần trùng sẽ tốn chi phí rà nhãn mà đa dạng kém. Điểm bất định cao chưa chắc cải thiện mô hình 100% vì có thể đó là những frame nhiễu quá nặng mà con người cũng không nhìn ra.

## 4. Các vòng học chủ động (active learning)

Với vòng 1, em đã thêm mới 183 box bị sót, giữ lại 142 box, sửa 15 và xóa 12 box sai của AI (từ 169 box gợi ý ban đầu tăng lên 340 box thực tế).
Tuy nhiên, chỉ số AP50 bất ngờ **giảm 0.269** (từ 0.771 xuống 0.502). Mức độ phủ (Recall) sụt giảm thê thảm ở tất cả các nhóm xe, đặc biệt xe nhỏ giảm về 0.000.
Lý do đi lùi: Việc Fine-tune trên một lô quá nhỏ (chỉ 12 ảnh) bằng YOLO dễ dẫn đến hiện tượng "Quên thảm họa" (Catastrophic Forgetting) hoặc học vẹt (Overfit) vào đúng 12 bối cảnh đó. Đồng thời, tập kiểm thử đang dùng nhãn do AI cũ tạo ra để chấm điểm, nên khi AI mới thay đổi cách bắt (giống cách người sửa) thì nó lập tức bị trừ điểm vì "không giống đáp án cũ".

## 5. Kết luận và giới hạn

So với cold start, AP50 giảm sâu, nhưng kết quả thực tế mô hình đã cố gắng học các đặc trưng mới mà em gán. Em quyết định dừng ở vòng 1 vì các giới hạn của lab:

1. Tập kiểm thử chỉ có 20 ảnh và nhãn tham chiếu là do AI cũ sinh ra chưa được rà soát bởi con người.
2. Lô 12 ảnh quá ít để train thực tế một mô hình Deep Learning phức tạp.
   Nếu AP50 giảm, điều cần làm đầu tiên trước khi train thêm là kiểm tra lại và sửa tay toàn bộ nhãn tham chiếu của 20 ảnh Test để biến nó thành "Ground Truth" thật, đồng thời phải tăng ngân sách lấy ít nhất 100-200 ảnh cho vòng Active Learning.
