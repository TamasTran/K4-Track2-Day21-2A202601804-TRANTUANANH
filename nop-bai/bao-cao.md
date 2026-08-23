# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Trần Tuấn Anh |
| MSSV | 2A202601804 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/TamasTran/K4-Track2-Day21-2A202601804-TRANTUANANH |
| Ngày nộp | 23/08/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.20 | 3 | 0.7290 | 0.8840 |
| 2 | 200 | 0.10 | 5 | 0.7149 | 0.8740 |
| 3 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |

**Bộ siêu tham số đã chọn:** `n_estimators=100`, `learning_rate=0.2`, `max_depth=3`.

**Lý do:** Em thấy bộ siêu tham số này mang lại chỉ số F1-score cao nhất (0.7290) và đồng thời đạt accuracy cao nhất (0.8840) trong tất cả các lần chạy thực nghiệm trên MLflow. Em thấy lần chạy có accuracy cao nhất hoàn toàn trùng khớp với lần có F1-score cao nhất, chứng minh mô hình không chỉ phân loại đúng tổng thể mà còn nhận diện rất tốt lớp thiểu số thu nhập cao. Em thấy giữa `n_estimators` và `learning_rate` luôn có sự đánh đổi rõ rệt: khi đặt `learning_rate=0.05` quá nhỏ với 50 cây, mô hình bị underfitting nặng và F1-score tụt xuống 0.6051; ngược lại khi tăng lên 200 cây với `max_depth=5`, mô hình bắt đầu có dấu hiệu quá khớp cục bộ khiến F1-score giảm còn 0.7149. Cấu hình 100 cây cùng độ sâu 3 giúp cân bằng hoàn hảo giữa khả năng biểu diễn và tính tổng quát hóa.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Em thấy tập dữ liệu Adult có sự mất cân bằng lớp rất rõ rệt khi lớp thu nhập cao (>50K) chỉ chiếm khoảng 24.8% tổng số mẫu, còn lớp thu nhập thấp chiếm tới 75.2%. Em thấy nếu một mô hình ngây thơ luôn luôn dự đoán mọi người đều có "thu nhập thấp", nó vẫn dễ dàng đạt được accuracy lên tới 75.2% dù hoàn toàn vô dụng và không phân loại được bất kỳ khách hàng thu nhập cao nào. Em thấy F1-score trên lớp dương chính là thước đo trung hòa hài hòa giữa Precision và Recall, phản ánh chính xác năng lực phát hiện lớp thiểu số quan trọng mà Accuracy hoàn toàn bỏ sót. Em thấy chúng ta không dùng `average="weighted"` hay `average="macro"` vì trọng số của lớp đa số 75.2% sẽ kéo điểm số chung lên cao giả tạo, làm mất đi ý nghĩa bảo vệ chất lượng của Quality Gate trong pipeline CI/CD.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Xung đột phiên bản thư viện khi chạy CI trên GitHub Actions | Gói `dvc-s3` và `boto3` bị xung đột phụ thuộc chéo làm pip chạy backtracking kéo dài | Em thấy đã ghim phiên bản tương thích trong `requirements.txt` và nâng cấp pip trước khi cài |
| Thư mục `mlruns` bị Git theo dõi gây nặng kho chứa mã nguồn | Chưa bổ sung thư mục thực nghiệm `mlruns/` vào `.gitignore` trước khi chạy các vòng lặp | Em thấy đã dùng `git rm -r --cached mlruns` để bỏ theo dõi và cập nhật lại file `.gitignore` |
| Runner trên CI không tìm thấy file dữ liệu khi chạy huấn luyện | File dữ liệu lớn chưa được kéo về môi trường CI trước khi script `train.py` thực thi | Em thấy đã cấu hình chuẩn xác bước xác thực AWS S3 và thêm lệnh `dvc pull` trước bước train |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7290 | 0.8840 |
| Bước 3 (thêm `train_batch2`) | 0.7330 | 0.8820 |


**Nhận xét:** Em thấy khi bổ sung `train_batch2` để nâng quy mô dữ liệu lên 44.722 mẫu, chỉ số F1-score tăng nhẹ từ 0.7290 lên 0.7330 trong khi accuracy giữ mức ổn định 88.2%. Em thấy sự cải thiện này diễn ra với biên độ nhỏ vì dữ liệu mới cùng phân phối với batch 1, chứng tỏ mô hình đã học được gần như toàn bộ đặc trưng cốt lõi ngay từ giai đoạn đầu. Em thấy giá trị cốt lõi nhất ở Bước 3 là kiểm chứng thành công pipeline CI/CD tự động kích hoạt huấn luyện lại và tái triển khai lên VM chỉ sau một lần push dữ liệu DVC.

---

## 5. Phần Bonus Đã Thực Hiện (nếu có)

<!-- Xóa cả mục 5 nếu không làm bonus. Mỗi bonus tối đa 1 dòng. -->

- [ ] Bonus 1 - Tracking MLflow từ xa với DagsHub: ___
- [ ] Bonus 2 - Điều chỉnh ngưỡng quyết định: ___
- [ ] Bonus 3 - Báo cáo precision / recall tự động: ___
- [ ] Bonus 4 - Hoàn trả về phiên bản trước: ___
- [ ] Bonus 5 - Cảnh báo lệch lạc dữ liệu: ___
