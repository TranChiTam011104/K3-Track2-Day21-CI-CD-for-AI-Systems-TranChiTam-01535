# Báo Cáo MLOps Pipeline - CI/CD for AI Systems

## 1. Bộ Siêu Tham Số Đã Chọn

| Tham số         | Giá trị | Lý do chọn                           |
|-----------------|---------|--------------------------------------|
| n_estimators    | 500     | Tăng số cây để giảm overfitting     |
| max_depth       | 30      | Hạn chế độ sâu tránh overfitting    |
| min_samples_split | 2     | Cho phép chia sớm, phù hợp dữ liệu nhỏ |

## 2. So Sánh Kết Quả Giữa 2 Lần Chạy

| Chỉ số      | Lần 1 (chỉ data1) | Lần 2 (data1 + data2) |
|-------------|-------------------|------------------------|
| Accuracy    | 0.6936            | **0.7460**             |
| F1-Score    | 0.6804            | **~0.74**             |
| Mẫu train   | 2,998             | 5,996                  |
| Trạng thái  | ❌ FAIL (< 0.70)  | ✅ PASS (≥ 0.70)       |

**Nhận xét:** Kết hợp 2 dataset (train_phase1 + train_phase2) tăng accuracy từ 0.6936 → 0.7460 (+5.24%). Điều này chứng minh model được cải thiện đáng kể khi có thêm dữ liệu huấn luyện.

## 3. Khó Khăn và Cách Giải Quyết

### a) Systemd Service File Lỗi
**Vấn đề:** Service `mlops-serve.service` báo lỗi "Invalid syntax" do dấu `\` ở cuối dòng Environment:
```
Environment=\GCS_BUCKET=mlops-lab-tranchitam-01535-bucket\
```

**Giải quyết:** Xóa file cũ và tạo lại với định dạng đúng:
```ini
Environment="GCS_BUCKET=mlops-lab-tranchitam-01535-bucket"
```

### b) YAML Parsing Trong GitHub Actions
**Vấn đề:** Dùng heredoc `<<EOF` trong `script:` của appleboy/ssh-action gây lỗi YAML parsing vì `[Unit]`, `[Service]` bị interpret như map keys.

**Giải quyết:** Thay heredoc bằng `printf`:
```yaml
script: |
  printf '[Unit]\nDescription=MLOps...\n...' | sudo tee /etc/systemd/system/mlops-serve.service
```

## 4. Kết Luận

Pipeline CI/CD hoạt động hoàn chỉnh: Test → Train → Eval (với accuracy gate ≥ 0.70) → Deploy. Việc tăng dataset từ ~3K lên ~6K mẫu giúp model đạt accuracy 0.75, vượt ngưỡng và deploy thành công lên VM.
