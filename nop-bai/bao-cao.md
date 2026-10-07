# Báo Cáo Lab Day 21 - CI/CD cho AI Systems
|              |                                                                                                                                                                                                                   |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Họ và tên | Nguyễn Thị Minh Tiến                                                                                                                                                                                           |
| MSSV         | 2A202602997                                                                                                                                                                                                       |
| Lớp / Khóa | K4                                                                                                                                                                                                                |
| Repo GitHub  | [github.com/MinhTienNguyen05/K4-L3L4-Track2-Day21-NguyenThiMinhTien-2A202602997-CI-CD-for-AI-Systems](https://github.com/MinhTienNguyen05/K4-L3L4-Track2-Day21-NguyenThiMinhTien-2A202602997-CI-CD-for-AI-Systems) |
| Ngày nộp   | 7/10/2026                                                                                                                                                                                                         |

---

# Báo Cáo CI/CD Cho Hệ Thống AI

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
| :---: | :---: | :---: | :---: | :---: | :---: |
| 1 | 200 | 0.1 | 5 | 0.7149 | 0.874 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.846 |
| 3 | 100 | 0.1 | 3 | 0.7109 | 0.878 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`[cite: 3].

**Lý do:** Chọn Lần 1 vì đạt `f1_score` cao nhất (0.7149)[cite: 3], thước đo cốt lõi để đánh giá lớp thiểu số trên tập dữ liệu mất cân bằng thay vì `accuracy` dễ bị điểm ảo. Lần có accuracy cao nhất (Lần 3 đạt 0.878) không trùng F1 cao nhất do thiên vị đoán lớp đa số. Về đánh đổi, mô hình bị underfitting ở Lần 2 khi ít cây và học chậm (F1 chỉ 0.605)[cite: 4]; cấu hình 200 cây, `learning_rate=0.1`, `max_depth=5` giúp trích xuất đặc trưng sâu và hội tụ tối ưu mà không quá khớp[cite: 3, 4].

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

* **Mất cân bằng nhãn:** Nhóm thu nhập cao (>50K) chiếm 24.8%, nhóm <=50K chiếm 75.2%. Mô hình luôn đoán "thu nhập thấp" vẫn đạt Accuracy 75.2% dù Recall lớp dương bằng 0, gây đánh giá sai lệch.
* **F1 đo đúng trọng tâm:** F1 cân bằng giữa Precision và Recall riêng cho lớp thiểu số, buộc mô hình nhận diện chính xác nhóm thu nhập cao thay vì đoán dồn vào lớp đa số.
* **Không dùng macro hay weighted:** Chỉ số `weighted` bị chi phối hơn 75% bởi lớp đa số; `macro` bị kéo cao do F1 lớp đa số vốn rất lớn. Chỉ có F1 nhị phân (`binary`) trên lớp dương mới đủ khắt khe cho Quality Gate.

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
| :--- | :--- | :--- |
| Job Unit Test bị lỗi | Commit nhầm `mlruns/` và thiếu URI tracking. | Xóa `mlruns` khỏi Git, thêm vào `.gitignore` và đặt `MLFLOW_TRACKING_URI` về SQLite. |
| Job Release lỗi SSH `no key found` | Secret `SERVER_SSH_KEY` sai định dạng Private Key. | Cập nhật lại nội dung Private Key hợp lệ vào GitHub Secrets. |
| Service EC2 crash, health check thất bại | Thiếu biến môi trường `ARTIFACT_BUCKET` trong file systemd. | Thêm biến vào file service, chạy `daemon-reload` và restart lại dịch vụ. |

## 4. So Sánh Bước 2 và Bước 3

| Tập dữ liệu | f1_score | accuracy |
| :--- | :---: | :---: |
| Bước 2 (chỉ `train_batch1` - 22.361 mẫu) | 0.7149[cite: 2] | 0.874[cite: 2] |
| Bước 3 (thêm `train_batch2` - 44.722 mẫu) | 0.7354 | 0.882 |

**Nhận xét:** Khi bổ sung thêm 22.361 mẫu ở Bước 3, `f1_score` tăng từ 0.7149 lên 0.7354 và `accuracy` tăng từ 0.874 lên 0.882 nhờ mô hình tiếp cận thêm các mẫu biên của lớp thiểu số[cite: 2]. Mức tăng nhẹ do dữ liệu mới có cùng phân phối với tập ban đầu. Kết quả khẳng định đường ống CI/CD đã tự nhận diện thay đổi file DVC, kích hoạt huấn luyện lại và cập nhật thành công mô hình mới lên VM.