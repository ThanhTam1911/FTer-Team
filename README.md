# Dự báo Doanh thu Thương mại Điện tử Thời trang - FTer-Team

Dự án này tập trung vào việc xây dựng hệ thống dự báo doanh thu hàng ngày (Daily Revenue Forecasting) nhằm tối ưu hóa chuỗi cung ứng cho doanh nghiệp thời trang tại Việt Nam.



## Mục tiêu kinh doanh (Business Objectives)

Kết quả của mô hình được thiết kế để giải quyết trực tiếp 3 bài toán vận hành:

* Tối ưu hoá phân bổ tồn kho: Giảm thiểu tỷ lệ "cháy kho" (Stock-out) và tối ưu vòng quay hàng tồn kho thông qua dự báo nhu cầu chi tiết.
* Lập kế hoạch khuyến mãi: Đánh giá định lượng tác động của các chương trình giảm giá, Mega Sale đến doanh thu để tối ưu ngân sách marketing.
* Quản lý Logistics toàn quốc: Dự báo lưu lượng đơn hàng để điều phối nguồn lực vận tải và kho bãi tại các khu vực trọng điểm.

## Thành viên nhóm (FTer)

* Phan Quốc Tú
* Hoàng Ngọc Bảo Trân
* Đỗ Thành Tâm
* Lê Huy

## Cấu trúc dữ liệu

Bộ dữ liệu mô phỏng từ năm 2012 đến 2022, bao gồm 15 file CSV chia thành 4 lớp dữ liệu chính:

* Master: Thông tin sản phẩm, khách hàng, khuyến mãi, địa lý.
* Transaction: Chi tiết đơn hàng, thanh toán, vận chuyển, trả hàng.
* Analytical: Doanh thu thuần hằng ngày (Biến mục tiêu Revenue).
* Operational: Tồn kho hàng tháng và lưu lượng website hằng ngày.

## Quy trình thực thi

### 1\. Tiền xử lý \& Nối bảng (Preprocessing)
* Chuẩn hóa định dạng: Chuyển đổi toàn bộ cột ngày tháng sang `datetime` và mã định danh sang `string`.
* Xử lý tồn kho động: Chuyển đổi tồn kho từ mức tháng sang mức ngày bằng logic:

  * Tồn đầu ngày = Tồn cuối ngày hôm trước.
  * Tồn cuối ngày = Tồn đầu ngày - Số lượng bán trong ngày.
* Hợp nhất (Merging): Sử dụng `Calendar Spine` (trục thời gian chuẩn) để nối các lớp dữ liệu, đảm bảo không mất ngày nào trong chuỗi thời gian, kể cả ngày không có đơn hàng.

### 2\. Trích xuất đặc trưng (Feature Engineering)

Dựa trên đặc thù ngành thời trang và e-commerce Việt Nam:

* Tính chu kỳ (Cyclical Features): Mã hóa Sin/Cos cho tháng và thứ trong tuần.
* Tín hiệu E-commerce: Gắn nhãn các ngày "Double Day" (11/11, 12/12...), ngày lĩnh lương (Payday), và đếm ngược đến đợt Mega Sale tiếp theo.
* Biến trễ (Lags \& Rolling): Sử dụng `lag_1`, `lag_7`, `lag_365` và trung bình trượt 7-30 ngày để bắt nhịp xu hướng ngắn và dài hạn. 

### 3\. Mô hình dự báo (Modeling)

Thay vì sử dụng các mô hình thống kê truyền thống, hệ thống hiện tại tập trung vào phương pháp Machine Learning Ensemble kết hợp với chiến lược Recursive Forecasting để giải quyết bài toán chuỗi thời gian dài hạn (2023-2024).

* Boosting Ensemble (LightGBM \& XGBoost): Sử dụng sự kết hợp (Blend) giữa LightGBM và XGBoost để bắt lấy các mối quan hệ phi tuyến phức tạp giữa Lưu lượng truy cập (Traffic), Độ sâu giảm giá (Discount Depth) và Tồn kho.
* Chiến lược dự báo đệ quy (Recursive Forecasting): Mô hình thực hiện dự báo cho từng ngày, sau đó dùng chính kết quả Revenue dự báo đó để cập nhật các biến trễ (`lag_1`, `lag_7`) cho ngày tiếp theo.
* Hàm mất mát Huber Loss \& Objective MAE: Mô hình được tối ưu hóa bằng Huber Loss – một hàm mất mát lai giúp cân bằng giữa MAE (độ lệch tuyệt đối) và RMSE (phạt sai số lớn).
* Tính nhất quán Revenue - COGS: Sử dụng cùng một cấu trúc đặc trưng cho cả dự báo Doanh thu và Giá vốn hàng bán (COGS).

## Hướng dẫn sử dụng

1. Mở file `FTer-Team-Datathon2026` trên Google Colab.
2. Cấu hình Kaggle API để tải bộ dữ liệu `datathon-2026-round-1`.
3. Chạy các Cell theo thứ tự từ: Load Data -> Preprocessing -> Feature Engineering -> Training -> Prediction.
4. Kết quả dự báo sẽ được xuất ra file `submission.csv` cho giai đoạn 01/01/2023 – 01/07/2024.
