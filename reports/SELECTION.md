# Vì sao chọn lô này?

Vòng 1 chọn 12 ảnh theo chiến lược uncertainty: xếp điểm giảm dần rồi chọn ảnh theo thứ tự đó, đồng thời giữ khoảng cách thời gian tối thiểu 2 giây giữa các ảnh. Năm ảnh đứng đầu CSV là frame_0182.jpg (hạng 1, 0.9591), frame_0369.jpg (hạng 2, 0.9324), frame_0380.jpg (hạng 3, 0.9170), frame_0326.jpg (hạng 4, 0.9155) và frame_0331.jpg (hạng 5, 0.9154); cả năm đều nằm trong lô được chọn.

CSV ghi `selected=True` cho đúng 12 ảnh: frame_0182.jpg, frame_0369.jpg, frame_0380.jpg, frame_0326.jpg, frame_0331.jpg, frame_0312.jpg, frame_0099.jpg, frame_0187.jpg, frame_0227.jpg, frame_0270.jpg, frame_0107.jpg và frame_0392.jpg. Ví dụ, frame_0182.jpg có điểm 0.9591, 28 box và 18 box mập mờ; frame_0099.jpg có điểm 0.9063, 29 box và 14 box mập mờ; frame_0107.jpg có điểm 0.8876, 33 box và 15 box mập mờ. Các cột `U`, `A`, `D` cùng số box mập mờ góp phần tạo điểm ưu tiên; chúng không phải số đo trực tiếp về mức cải thiện sau khi sửa.

frame_0372.jpg (hạng 6, điểm 0.9101, giây 148.8) không được chọn dù điểm cao hơn một số ảnh trong lô. frame_0369.jpg (giây 147.6) đã được chọn trước đó; hai ảnh cách nhau 1.2 giây, ngắn hơn khoảng cách tối thiểu 2 giây của bộ chọn. Vì vậy, frame_0372.jpg bị bỏ qua để giảm chọn các khung quá sát nhau.

Điểm xếp hạng phản ánh các tín hiệu bất định và đa dạng thời gian được dùng để ưu tiên ảnh cần rà. Việc chọn 12 ảnh theo các tín hiệu này không chứng minh rằng sửa chúng sẽ làm mô hình nhận diện xe tốt hơn.
