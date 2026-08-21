# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Nguyễn Thanh Vinh |
| MSSV | 2A202601556 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/vinz0369/Track2-Day21-2A202601556-NguyenThanhVinh |
| Ngày nộp | 21/08/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Lần chạy 3 có F1 cao nhất là 0.7149 và vượt ngưỡng triển khai 0.65. Lần chạy 1 có accuracy cao nhất nhưng F1 thấp hơn, cho thấy accuracy không phản ánh đầy đủ khả năng nhận diện lớp thu nhập cao. Cấu hình 50 cây, learning rate 0.05 và độ sâu 2 học chưa đủ nên F1 chỉ đạt 0.6051. Khi learning rate nhỏ, mô hình thường cần nhiều cây hơn để bù lại mức đóng góp nhỏ của từng cây.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Chỉ 24,8% dữ liệu thuộc lớp thu nhập trên 50K. Vì vậy, một mô hình luôn dự đoán “thu nhập thấp” vẫn đạt accuracy khoảng 75,2% nhưng F1 của lớp dương bằng 0 vì không phát hiện được người thu nhập cao nào. F1 kết hợp precision và recall, nên chỉ cao khi mô hình vừa hạn chế dự đoán dương sai, vừa tìm được phần lớn mẫu dương thật. Bài lab cần đánh giá trực tiếp lớp thu nhập cao nên dùng `f1_score(y_eval, preds)` với lớp dương mặc định. Không dùng weighted F1 vì lớp đa số sẽ chi phối kết quả; macro F1 cũng không phù hợp với quality gate này vì nó trung bình hóa hai lớp thay vì đo riêng lớp dương.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| MLflow không import được `pkg_resources` | Setuptools mới không còn module mà MLflow 2.13 sử dụng | Ghim `setuptools==80.9.0` trong requirements |
| Bộ khung mặc định dùng GCP | Bài triển khai thực tế trên AWS | Đổi sang `dvc[s3]`, `boto3`, S3 và AWS credentials action |
| Accuracy cao nhưng chưa chắc model tốt | Dữ liệu mất cân bằng 75/25 | So sánh theo F1 của lớp dương và dùng quality gate 0.65 |

---

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.8740 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.8820 |

**Nhận xét:** Sau khi tăng dữ liệu huấn luyện từ 22.361 lên 44.722 mẫu, F1 tăng 0.0205 và accuracy tăng 0.0080. Dữ liệu mới cùng phân phối nhưng cung cấp thêm ví dụ để mô hình nhận diện lớp dương tốt hơn; kết quả tăng trong lần chạy này không có nghĩa thêm dữ liệu luôn bảo đảm chỉ số tốt hơn.
