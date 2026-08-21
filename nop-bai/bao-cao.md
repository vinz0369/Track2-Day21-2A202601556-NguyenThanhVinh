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

**Lý do:** Lần 3 có F1 cao nhất (0.7149) và vượt ngưỡng 0.65. Lần 1 có accuracy cao hơn nhưng F1 thấp hơn, nên accuracy không phản ánh đầy đủ khả năng nhận diện lớp thu nhập cao.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Chỉ 24,8% dữ liệu thuộc lớp thu nhập trên 50K. Mô hình luôn đoán “thu nhập thấp” vẫn đạt accuracy khoảng 75,2% nhưng F1 lớp dương bằng 0. Vì vậy quality gate dùng `f1_score(y_eval, preds)` cho lớp thu nhập cao; F1 chỉ cao khi precision và recall cùng tốt.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| MLflow không import được `pkg_resources` | Setuptools mới không còn module mà MLflow 2.13 sử dụng | Ghim `setuptools==80.9.0` trong requirements |
| Bộ khung mặc định dùng GCP | Bài triển khai thực tế trên AWS | Đổi sang `dvc[s3]`, `boto3`, S3 và AWS credentials action |
| Push không kích hoạt workflow trong fork | Actions của fork chưa được bật đầy đủ ở cấp repository | Tắt/bật lại Actions, sau đó push tự chạy đủ 4 jobs |

---

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.8740 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.8820 |

**Nhận xét:** Khi tăng từ 22.361 lên 44.722 mẫu, F1 tăng 0.0205 và accuracy tăng 0.0080. Dữ liệu mới cùng phân phối đã giúp lần chạy này nhận diện lớp dương tốt hơn, nhưng không có nghĩa thêm dữ liệu luôn bảo đảm chỉ số tăng.
