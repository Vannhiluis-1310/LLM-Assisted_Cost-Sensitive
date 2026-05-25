# Hướng Dẫn Chạy Phase 1 Trên Google Colab

Tài liệu này mô tả flow hiện tại của Phase 1 sau khi code chính đã được chuyển sang notebook. Artifact chính để chạy/nộp là:

`notebooks/01_data_check.ipynb`

Notebook này là **self-contained**: đã merge code download/validate dataset, load CSV, left join, EDA, split theo `TransactionDT`, và preprocessing train-only. Các file `.py` trong `src/` chỉ giữ vai trò script mirror/helper.

## 1. Chuẩn Bị Project

Upload toàn bộ thư mục project lên Google Drive hoặc Colab runtime. Không chỉ upload riêng notebook, vì notebook vẫn cần `requirements.txt` và cấu trúc thư mục project.

Ví dụ:

- Google Drive: `MyDrive/LLM-Assisted_Cost-Sensitive/`
- Colab runtime: `/content/LLM-Assisted_Cost-Sensitive/`

Mở notebook:

`notebooks/01_data_check.ipynb`

Nếu project nằm trong Google Drive, ở cell setup đặt:

```python
MOUNT_GOOGLE_DRIVE = True
```

Nếu notebook không tự tìm được project root, đặt:

```python
PROJECT_ROOT_OVERRIDE = "/content/drive/MyDrive/LLM-Assisted_Cost-Sensitive"
```

## 2. Chuẩn Bị Dataset IEEE-CIS

Notebook cần đúng hai file:

- `train_transaction.csv`
- `train_identity.csv`

Có 3 cách đưa dataset vào Colab.

### Cách A: Google Drive

Tạo thư mục:

`MyDrive/ieee-fraud-detection/`

Đặt vào đó hai file CSV hoặc file ZIP Kaggle. Notebook sẽ copy/unzip vào:

`data/raw/`

### Cách B: Upload Trực Tiếp

Trong Colab Files panel, tạo:

`/content/ieee-fraud-detection/`

Upload hai CSV hoặc ZIP Kaggle vào đó. Notebook sẽ tự stage sang `data/raw/`.

### Cách C: Kaggle API

Tải `kaggle.json` từ Kaggle Account -> API -> Create New Token.

Đặt `kaggle.json` ở một trong các vị trí:

- `/content/kaggle.json`
- `PROJECT_ROOT/kaggle.json`
- `MyDrive/kaggle.json`

Trước khi tải bằng API, phải vào Kaggle competition `ieee-fraud-detection` và accept rules.

## 3. Flow Chạy Notebook

Chạy lần lượt các section trong `notebooks/01_data_check.ipynb`:

1. Setup local/Colab project root
2. Install/check dependencies
3. Prepare IEEE-CIS data
4. Run reproducible pipeline
5. Data integrity audit
6. Feature family overview
7. Family-level numeric summaries
8. Target imbalance and approve-all warning
9. `TransactionAmt` distribution by label
10. Missingness analysis
11. Identity coverage by label
12. Time-based behavior and split sanity
13. Categorical feature summaries
14. Numeric feature association with label
15. Display saved report figures
16. Why Accuracy is not the main metric
17. First-pass preprocessing handoff
18. End-to-end output checklist

Để smoke test nhanh:

```python
SAMPLE_ROWS = 10000
```

Để chạy kết quả chính thức:

```python
SAMPLE_ROWS = None
```

Mặc định:

```python
RUN_PREPROCESSING = False
```

Chỉ đổi thành `True` khi cần regenerate artifact preprocessing. Full preprocessing có thể tạo `X_train.joblib` khoảng vài GB.

## 4. Output Mong Đợi

Sau khi chạy notebook, các output chính gồm:

- `results/dataset_summary.csv`
- `results/split_summary.csv`
- `results/missing_summary_top30.csv`
- `results/eda/data_integrity_audit.csv`
- `results/eda/feature_family_summary.csv`
- `results/eda/family_numeric_summary.csv`
- `results/eda/class_distribution.csv`
- `results/eda/transaction_amount_summary_by_label.csv`
- `results/eda/identity_coverage_by_label.csv`
- `results/eda/fraud_rate_by_transactiondt_bin.csv`
- `results/eda/numeric_label_correlation_top30.csv`
- `data/processed/train.csv`
- `data/processed/validation.csv`
- `data/processed/test.csv`
- `reports/figures/class_distribution.png`
- `reports/figures/transaction_amount_distribution.png`
- `reports/figures/missing_values_top30.png`
- `reports/figures/eda/*.png`

Nếu bật `RUN_PREPROCESSING = True`, notebook còn tạo:

- `artifacts/preprocessing/preprocessing_metadata.csv`
- `artifacts/preprocessing/preprocessor.joblib`
- `artifacts/preprocessing/X_train.joblib`
- `artifacts/preprocessing/X_validation.joblib`
- `artifacts/preprocessing/X_test.joblib`
- `artifacts/preprocessing/y_*.csv`
- `artifacts/preprocessing/meta_*.csv`

## 5. Lưu Ý Khi Chạy Full Dataset

- Full EDA đọc `train_transaction.csv` khoảng 650 MB, nên cần RAM tương đối ổn.
- Một số biểu đồ dùng sample cố định `50,000` dòng để tránh đơ notebook; các bảng summary chính vẫn tính trên dữ liệu đang load.
- Full preprocessing nặng hơn EDA nhiều. Chỉ bật `RUN_PREPROCESSING = True` khi thật sự cần.
- Không dùng Accuracy làm metric chính cho các phase sau. Phase 1 chỉ dùng Accuracy trong approve-all warning để chứng minh Accuracy gây hiểu nhầm trên dữ liệu mất cân bằng.

## 6. Lỗi Thường Gặp

**Không tìm thấy project root**

Upload/clone toàn bộ project. Nếu project nằm trong Drive, bật `MOUNT_GOOGLE_DRIVE = True`. Nếu tên thư mục khác mặc định, đặt `PROJECT_ROOT_OVERRIDE`.

**Thiếu dataset**

Đảm bảo có `train_transaction.csv` và `train_identity.csv`, hoặc ZIP Kaggle, trong `MyDrive/ieee-fraud-detection/` hoặc `/content/ieee-fraud-detection/`.

**Kaggle download failed**

Kiểm tra đã accept rules trên Kaggle chưa, và `kaggle.json` có đúng token mới nhất không.

**Preprocessing quá nặng**

Chạy smoke test trước bằng `SAMPLE_ROWS = 10000`. Khi chạy full, nên dùng Colab high-RAM hoặc máy local đủ RAM/dung lượng.
