# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Đỗ Nguyễn Việt Linh

Công cụ gán nhãn đã dùng: CVAT

## 1. Dữ liệu và cách chia tập

Camera cố định nên một chiếc xe có thể xuất hiện trong nhiều khung hình liên tiếp. Pool dùng để chọn ảnh học và tập kiểm thử được chia theo thời gian, có vùng đệm ở giữa, để các khung có cùng xe hoặc gần như cùng cảnh không lọt sang cả hai tập. Nếu chia ngẫu nhiên, những khung gần như trùng nhau có thể xuất hiện ở cả tập học và tập kiểm thử; mô hình đã gặp chiếc xe đó khi học nên điểm kiểm thử dễ cao hơn khả năng nhận diện trên cảnh mới.

## 2. Mô hình khởi đầu lạnh (cold start)

Ở vòng 0, YOLOv8n cold start (COCO car+bus+truck) chưa dùng ảnh train (0 ảnh, 0 box), đạt AP50 0.771; tại ngưỡng conf 0.25, P=0.925, R=0.489, F1=0.640. Recall theo kích thước là 0.182 với xe nhỏ, 0.547 với xe vừa và 0.561 với xe lớn, cho thấy xe nhỏ bị bỏ sót nhiều hơn rõ rệt. Trong `compare_round0.jpg`, ở frame_050.jpg có một xe xa gần phía trên giữa ảnh mà dự đoán của cold start lệch và chồng khung so với khung tham chiếu. Tuy vậy, các nhãn tham chiếu dùng để chấm cũng do máy tạo và chưa được người rà từng khung, nên trường hợp này cần kiểm tra nhãn tham chiếu trước khi kết luận mô hình sai.

## 3. Chiến lược chọn mẫu

Bộ chọn tính điểm mỗi ảnh bằng 0.5×U + 0.3×A + 0.2×D: một nửa điểm đến từ độ bất định của dự đoán, ba phần mười từ số box mập mờ, và hai phần mười từ độ cách biệt thời gian với ảnh đã gán nhãn. Trong lô hiện tại chưa có ảnh đã gán trước đó nên D bằng nhau cho mọi ảnh; giữa các ảnh được chọn trong cùng lô, `MIN_GAP_S` yêu cầu cách nhau ít nhất 2 giây để tránh lấy nhiều khung gần như cùng cảnh. Ba ảnh được chọn là frame_0182.jpg (điểm 0.9591, 18 box mập mờ), frame_0099.jpg (0.9063, 14) và frame_0107.jpg (0.8876, 15). frame_0372.jpg có điểm 0.9101 nhưng không được chọn vì cách frame_0369.jpg đã chọn chỉ 1.2 giây, thấp hơn khoảng cách tối thiểu. Điểm cao giúp ưu tiên ảnh để rà, nhưng không chứng minh sửa ảnh đó sẽ làm AI giỏi hơn.

## 4. Các vòng học chủ động (active learning)

| Vòng | Model | Ảnh train | Box train | AP50 | Δ AP50 so cold start | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | yolov8n cold start (COCO car+bus+truck) | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |
| 1 | yolov8n fine-tune vòng 1 | 12 | 327 | 0.749 | -0.023 | 1.000 | 0.169 | 0.289 | 0.000 | 0.142 | 0.634 |

Trong 12 ảnh, tôi giữ nguyên 128 box, chỉnh 21, xóa 20 box dự đoán sai và thêm 178 box bị bỏ sót; tổng cộng nhãn sau sửa có 327 box. So với vòng 0, AP50 giảm 0.022 theo phép trừ trực tiếp các số đã làm tròn (bảng ghi Δ=-0.023 từ số chưa làm tròn); recall xe nhỏ giảm từ 0.182 xuống 0.000, xe vừa từ 0.547 xuống 0.142, còn xe lớn tăng từ 0.561 lên 0.634. Ở frame_050.jpg, hình so sánh cho thấy cold start có TP=11, FP=2, FN=7, còn vòng 1 có TP=4, FP=0, FN=14; đây là một ca kết quả xấu đi trên bộ tham chiếu. Trước khi xem pre-label, tôi ghi nhận ở frame_0182.jpg nhìn thấy 25 xe, trong đó xe ngoài cùng bên trái nhỏ và xe sát xe trắng bên phải dễ bị bỏ sót hoặc gán nhầm (`BLIND_SCAN.md`). Log sửa nhãn ghi tôi đã thêm xe ở góc dưới bên phải frame_0099.jpg, xe mờ góc dưới bên trái frame_0107.jpg và xe bị khuất sau van ở frame_0270.jpg (`REVIEW_LOG.csv`). Đây là quan sát độc lập, các box pre-label đã sửa, và kết quả mô hình sau huấn luyện — ba loại bằng chứng khác nhau. Với xe chỉ còn hai chấm đèn, guideline cho phép cân nhắc bỏ qua nếu box cao dưới khoảng 16 pixel; nếu gán thì cần ước lượng thân xe, không chỉ khoanh hai chấm đèn.

## 5. Kết luận và giới hạn

Vòng 1 có AP50 0.749, thấp hơn vòng 0 là 0.771, đồng thời recall xe nhỏ và vừa giảm mạnh. Tôi sẽ tạm dừng việc cho mô hình học thêm và rà lại nhãn đã sửa cùng các nhãn tham chiếu trước khi quyết định vòng tiếp theo; thêm dữ liệu ngay khi điểm đang giảm có thể khiến mô hình học từ nhãn chưa nhất quán. Hai tình huống còn khó là xe rất xa chỉ thấy hai chấm đèn và các xe sát hoặc che khuất nhau; chúng cần rà từng xe cẩn thận, còn các ảnh sát nhau cần tránh chọn lặp vì tốn công mà ít thêm cảnh mới. Phép chấm chỉ dùng 20 ảnh, bỏ qua 14 box tham chiếu cao dưới 16 pixel, và các nhãn tham chiếu chưa được người kiểm từng box, nên số đo chưa đủ để kết luận chắc chắn về chất lượng thực tế. Vì AP50 giảm, trước khi train thêm tôi sẽ kiểm tra lại box đã sửa, đặc biệt các box thêm hoặc kéo chỉnh, và đối chiếu các nhãn tham chiếu trong những khung test bị giảm recall.
