# Car Insurance Risk Classification & Automated Data Integrity Pipeline

### 📌 Tổng Quan Dự Án
Dự án  xây dựng một hệ thống đường ống tự động (Data Pipeline) khép kín, từ khâu tiền xử lý dữ liệu đến huấn luyện mô hình học máy. Mục tiêu cốt lõi là giải quyết bài toán phân loại nhị phân để hỗ trợ ra quyết định thẩm định bảo hiểm xe cơ giới tự động (`OUTCOME`: 1 - Chấp nhận, 0 - Từ chối) dựa trên hồ sơ rủi ro lái xe, nhân khẩu học và thông tin phương tiện của 105,017 khách hàng.

### ⚠️ Bài Toán Kinh Doanh & Quản Trị Rủi Ro
Trong ngành bảo hiểm, việc chuyển đổi từ thẩm định thủ công sang tự động hóa là yêu cầu tất yếu để kiểm soát rủi ro và tối đa hóa hiệu suất vận hành. Điểm nhấn sáng tạo của dự án là phát hiện và xử lý triệt để vấn đề **Rò rỉ dữ liệu (Data Leakage)** và lỗi ngữ nghĩa. Việc đưa trực tiếp các biến hành vi hiếm (như số lần vi phạm tốc độ, tai nạn do rượu bia) vào mô hình sẽ gây ra hiện tượng "học vẹt" (Overfitting), khiến mô hình đạt điểm số ảo nhưng mất khả năng dự báo thực tế. Do đó, dự án thiết lập một hệ thống khép kín: tách biệt các biến hành vi sang bộ quy tắc độc lập và dùng mô hình máy học để khai phá các tương tác phi tuyến phức tạp từ nhân khẩu học.

### 🔍 Phát Hiện Kiểm Toán Dữ Liệu (Key Data Insights)
1. **Lỗi Ngữ Nghĩa Toàn Vẹn (Semantic Errors):** 
   * Quá trình EDA phát hiện giá trị nhỏ nhất (min) của hai đặc trưng `ANNUAL_MILEAGE` (Số dặm đi hàng năm) và `PAST_ACCIDENTS` (Số vụ tai nạn) lần lượt là `-140,000` và `-5`. Đây là các giá trị âm không hợp lệ về mặt bản chất nghiệp vụ.
2. **Hiện Tượng Sự Kiện Hiếm (Rare Events Distribution):**
   * Các biến lịch sử vi phạm (`SPEEDING_VIOLATIONS`, `DUIS`, `PAST_ACCIDENTS`) có độ lệch dương cực mạnh (Positive Skewness lên tới 5.6). Phần lớn khách hàng có giá trị bằng 0, tạo thành một đuôi dài đại diện cho nhóm nhỏ khách hàng có rủi ro siêu cao.
3. **Mất Đồng Nhất Định Dạng (Categorical Inconsistency):**
   * Phát hiện lỗi không nhất quán chữ hoa/chữ thường nghiêm trọng ở biến trình độ học vấn (ví dụ: 'University' bên cạnh 'university') và biến giới tính, gây nhiễu cho thuật toán phân loại nếu không được chuẩn hóa.

### 🛠️ Kiến Trúc Hệ Thống & Công Nghệ
* **Ngôn ngữ & Thư viện:** Python (Pandas, NumPy, Scikit-Learn, FuzzyWuzzy, Joblib).
* **Mô hình thử nghiệm:** Hồi quy Logistic, SVM, KNN, Cây quyết định, Rừng ngẫu nhiên (Random Forest), XGBoost.
* **Kỹ thuật cốt lõi:** Trích xuất đặc trưng tương tác (Interaction Features), Lựa chọn đặc trưng tự động (RFE), và Kiểm thử chéo 3-Fold Cross-Validation.

### 📁 Cấu Trúc Kho Lưu Trữ (Repository Structure)
```text
car-insurance-risk-pipeline/
│
├── data/                           <- Thư mục chứa dữ liệu thô và dữ liệu sạch sau xử lý
│   ├── data.csv                    <- Tập dữ liệu thô ban đầu (105,017 bản ghi)
│   ├── final_train_data.csv        <- Dữ liệu sạch dùng để huấn luyện mô hình
│   └── final_test_data.csv         <- Dữ liệu sạch dùng để đánh giá mô hình
│
├── notebooks/
│   └── EDA.ipynb                   <- Khai phá tổng quan, chẩn đoán lỗi phân phối và ngoại lai[cite: 4]
│
├── modules/                           
│   ├── __init__.py
│   ├── utils.py                    <- Chứa Class DataLoader (nạp dữ liệu) và DataSplitter (chia tập)
│   ├── preprocessing.py            <- Pipeline tiền xử lý (Data Cleaner, Features Engineer, Transformer)
│   ├── modeling.py                 <- Class ModelTrainer và Không gian tìm kiếm tham số tối ưu
│   └── evaluation.py               <- Tính toán chỉ số hiệu suất hiệu năng
│
├── main.py                         <- Điểm khởi đầu khởi chạy (Entry Point) toàn bộ hệ thống pipeline
├── config.py                       <- File lưu trữ các tham số cấu hình hệ thống tập trung
│   └── plots/                      <- Thư mục lưu trữ biểu đồ Confusion Matrix và ROC Curve (.png)[cite: 4]
└── README.md                       <- Tài liệu hướng dẫn hệ thống
