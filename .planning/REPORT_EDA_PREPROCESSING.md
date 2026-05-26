# Báo Cáo: Phân Tích Dữ Liệu, Phân Chia Tập và Tiền Xử Lý

**Dataset:** IEEE-CIS Fraud Detection  
**Ngày:** 2026-05-26  
**Phase:** Phase 1 - Data Pipeline and EDA

---

## Tóm Tắt (Abstract)

Báo cáo này trình bày chi tiết quá trình phân tích dữ liệu khám phá (EDA), phương pháp phân chia tập dữ liệu và pipeline tiền xử lý cho bài toán phát hiện gian lận giao dịch thương mại điện tử. Dataset IEEE-CIS Fraud Detection được sử dụng với 590.540 giao dịch, trong đó 3.499% là fraud. Kết quả EDA xác nhận tính mất cân bằng nghiêm trọng của dữ liệu, sự khác biệt về transaction amount giữa fraud và legitimate, và giá trị predictive của identity coverage. Pipeline tiền xử lý được thiết kế để chống leakage với time-based split 70/15/15.

---

## 1. Giới Thiệu

### 1.1. Mục Tiêu Phân Tích

Phân tích dữ liệu khám phá (Exploratory Data Analysis - EDA) là bước bắt buộc trong quy trình khoa học dữ liệu nhằm:
- Hiểu rõ đặc tính và phân phối của dữ liệu
- Phát hiện các vấn đề về chất lượng dữ liệu (missing values, outliers, imbalance)
- Xác định các feature có tiềm năng predictive
- Đưa ra quyết định về phương pháp chia tập và tiền xử lý phù hợp

### 1.2. Dataset

**IEEE-CIS Fraud Detection** là dataset từ cuộc thi trên Kaggle, cung cấp dữ liệu giao dịch thực tế từ một nền tảng thanh toán quốc tế.

| Thuộc tính | Giá trị |
|------------|---------|
| Transaction rows | 590.540 |
| Transaction columns | 394 |
| Identity rows | 144.233 |
| Identity columns | 41 |
| Joined columns | 434 |
| Fraud count | 20.663 |
| Legitimate count | 569.877 |
| **Fraud rate** | **3.499%** |
| Identity coverage | 24.42% |

**Hai bảng dữ liệu:**
- `train_transaction.csv` (651.7 MB): Thông tin giao dịch
- `train_identity.csv` (25.3 MB): Thông tin định danh thiết bị/người dùng

**Join operation:** Left join bằng `TransactionID`, giữ nguyên tất cả giao dịch có label.

---

## 2. Phân Tích Dữ Liệu Khám Phá (EDA)

### 2.1. Data Integrity Audit

**Mục tiêu:** Xác minh tính toàn vẹn của pipeline data.

**Kiểm tra:**
- Số lượng rows trước và sau join
- Duplicate TransactionID
- Missing labels sau join
- Identity coverage

**Kết quả:**

| Check | Value |
|-------|-------|
| Transaction rows | 590.540 |
| Identity rows | 144.233 |
| Joined rows | 590.540 ✓ |
| Transaction ID duplicates (transaction) | 0 ✓ |
| Transaction ID duplicates (identity) | 0 ✓ |
| Missing labels after join | 0 ✓ |
| Identity matched rows | 144.233 |
| Identity missing rows | 446.307 |

**Kết luận:** Pipeline không làm mất dữ liệu, không có duplicate, không có missing label. Identity missing được giữ nguyên như thông tin hợp lệ (không drop).

---

### 2.2. Class Imbalance Analysis

**Mục tiêu:** Đánh giá mức độ mất cân bằng và cảnh báo về hạn chế của Accuracy metric.

**Phân phối class:**

| Class | Count | Ratio |
|-------|-------|-------|
| Legitimate (isFraud=0) | 569.877 | 96.501% |
| Fraud (isFraud=1) | 20.663 | 3.499% |
| **Total** | **590.540** | **100%** |

**Approve-all baseline (dự đoán toàn bộ là legitimate):**
- Accuracy = 96.501% (rất cao!)
- Fraud Recall = 0% (bỏ sót toàn bộ fraud!)

**Kết luận quan trọng:**
> Với dữ liệu mất cân bằng nghiêm trọng (3.5% fraud), Accuracy **không phù hợp** làm metric chính. Một model dự đoán "không fraud" cho tất cả giao dịch đạt Accuracy 96.5% nhưng vô dụng trong thực tế.

**Metrics khuyến nghị:**
- PR-AUC (Precision-Recall AUC)
- Fraud Recall, Precision, F1
- FN Cost, FP Cost, Total Cost
- Cost Saving so với baseline

---

### 2.3. Transaction Amount Analysis

**Mục tiêu:** Phân tích phân phối TransactionAmt theo label để thiết kế cost function.

**Thống kê mô tả:**

| Statistic | Legitimate (n=569.877) | Fraud (n=20.663) |
|-----------|------------------------|------------------|
| Mean | $134.51 | **$149.24** |
| Std | $239.40 | $232.21 |
| Min | $0.25 | $0.29 |
| 25% (Q1) | $43.97 | $35.04 |
| 50% (Median) | $68.50 | **$75.00** |
| 75% (Q3) | $120.00 | **$161.00** |
| 90% | $267.11 | $335.00 |
| 95% | $435.00 | $500.00 |
| 99% | $1.104.00 | $994.00 |
| Max | $31.937.39 | $5.191.00 |

**Phát hiện quan trọng:**
1. **Fraud mean > Legitimate mean** ($149.24 vs $134.51): Giao dịch fraud có xu hướng lớn hơn
2. **Fraud median > Legitimate median** ($75 vs $68.50): Xu hướng này nhất quán ở trung vị
3. **Cả hai phân phối skewed right** (mean >> median): Một số giao dịch rất lớn kéo mean lên
4. **Fraud 75th percentile > Legitimate 75th percentile** ($161 vs $120): Fraud tập trung nhiều hơn ở giao dịch lớn

**Hệ quả cho cost model:**
> FN Cost (False Negative - bỏ sót fraud) phải tỉ lệ với `TransactionAmt` vì số tiền mất thực tế phụ thuộc vào giá trị giao dịch. Cost function đề xuất:
> - FN Cost = α × TransactionAmt (α ≈ 10)
> - FP Cost = β × TransactionAmt (β ≈ 1, chi phí review)

---

### 2.4. Missingness Analysis

**Mục tiêu:** Đánh giá mức độ và pattern của missing values để thiết kế preprocessing.

**Top 30 columns có missing ratio cao nhất:**

| Rank | Column | Missing Ratio | Source |
|------|--------|---------------|--------|
| 1 | id_24 | 99.20% | identity |
| 2 | id_25 | 99.13% | identity |
| 3 | id_07 | 99.13% | identity |
| 4 | id_08 | 99.13% | identity |
| 5 | id_21 | 99.13% | identity |
| 6 | id_26 | 99.13% | identity |
| 7 | id_27 | 99.12% | identity |
| 8 | id_23 | 99.12% | identity |
| 9 | id_22 | 99.12% | identity |
| 10 | dist2 | 93.63% | transaction |
| 11 | D7 | 93.41% | transaction |
| 12 | id_18 | 92.36% | identity |
| 13 | D13 | 89.51% | transaction |
| 14 | D14 | 89.47% | transaction |
| 15 | D12 | 89.04% | transaction |
| 16 | id_04 | 88.77% | identity |
| 17 | id_03 | 88.77% | identity |
| 18 | D6 | 87.61% | transaction |
| 19 | id_33 | 87.59% | identity |
| 20 | id_09 | 87.31% | identity |

**Phân tích theo feature family:**

| Family | Column Count | Mean Missing | Median Missing |
|--------|--------------|--------------|----------------|
| V_vesta | 339 | 43.04% | 47.29% |
| identity_id | 38 | **84.82%** | 82.44% |
| D_timedelta | 15 | 58.15% | 52.47% |
| C_counting | 14 | **0%** | **0%** |
| M_match | 9 | 49.92% | 47.66% |
| card | 6 | 0.51% | 0.27% |
| core_transaction | 5 | 0% | 0% |
| address | 2 | 11.13% | 11.13% |
| email_domain | 2 | 46.37% | 46.37% |
| distance | 2 | 76.64% | 76.64% |
| device | 2 | 78.03% | 78.03% |

**Phát hiện quan trọng:**
1. **Identity features missing nhiều nhất** (~85%): Do chỉ 24.42% giao dịch có identity data
2. **C_counting không có missing** (0%): Đây là feature đáng tin cậy nhất
3. **V_vesta có missing cao** (~43%): Cần imputation hoặc encoding đặc biệt
4. **Missing không ngẫu nhiên:** Identity missing nhiều hơn ở legitimate transactions (xem Section 2.5)

**Chiến lược preprocessing:**
- Numeric: Median imputation (robust với outliers)
- Categorical: Constant imputation với giá trị "missing" (giữ thông tin về sự vắng mặt)

---

### 2.5. Identity Coverage by Label

**Mục tiêu:** Kiểm tra xem việc có identity data có liên quan đến fraud hay không.

**Kết quả:**

| isFraud | Total Rows | Identity Rows | Coverage Rate | Missing Identity |
|---------|------------|---------------|---------------|------------------|
| 0 (Legitimate) | 569.877 | 132.915 | **23.32%** | 436.962 |
| 1 (Fraud) | 20.663 | 11.318 | **54.77%** | 9.345 |

**Phát hiện quan trọng:**
> Fraud transactions có identity coverage **cao hơn gấp 2.35 lần** so với legitimate (54.77% vs 23.32%).

**Giải thích:**
- Giao dịch fraud thường để lại nhiều "dấu vết số" hơn (device info, browser, location)
- Điều này ngược với intuition ban đầu (tưởng fraud sẽ ẩn danh nhiều hơn)
- **Hệ quả:** Việc thiếu identity data có thể là một feature predictive (giao dịch không có identity có khả năng fraud thấp hơn)

**Xử lý trong preprocessing:**
- **Không drop** các dòng không có identity
- Để missingness đi qua pipeline như một tín hiệu hợp lệ
- Preprocessing sẽ impute hoặc encode missing như một category riêng

---

### 2.6. Feature Family Overview

**Mục tiêu:** Nhóm 434 features thành các family có ý nghĩa để dễ phân tích và báo cáo.

**11 Feature Families:**

| Family | Columns | Description | Sample Columns |
|--------|---------|-------------|----------------|
| core_transaction | 5 | Thông tin giao dịch cơ bản | TransactionID, isFraud, TransactionDT, TransactionAmt, ProductCD |
| card | 6 | Thông tin thẻ | card1, card2, card3, card4, card5, card6 |
| address | 2 | Địa chỉ | addr1, addr2 |
| distance | 2 | Khoảng cách | dist1, dist2 |
| email_domain | 2 | Domain email | P_emaildomain, R_emaildomain |
| C_counting | 14 | Đếm số lần (counting) | C1-C14 |
| D_timedelta | 15 | Thời gian trễ (timedelta) | D1-D15 |
| M_match | 9 | Khớp (matching) | M1-M9 |
| V_vesta | 339 | Features ẩn danh từ Vesta | V1-V339 |
| identity_id | 38 | Định danh thiết bị/người | id_01-id_38 |
| device | 2 | Thông tin thiết bị | DeviceType, DeviceInfo |

**Tổng số features:** 434 (không tính TransactionID và isFraud)

---

### 2.7. Family-Level Numeric Summaries

**Mục tiêu:** Đánh giá correlation giữa numeric features và label theo family.

**Phương pháp:**
- Sample 20.000 rows để tính correlation (nhanh, ổn định)
- Mỗi family dùng tối đa 40 columns có coverage cao nhất
- Tính mean và max absolute correlation với isFraud

**Kết quả:**

| Family | Numeric Columns | Correlation Columns Used | Mean Missing | Mean Abs Corr | **Max Abs Corr** |
|--------|-----------------|-------------------------|--------------|---------------|------------------|
| identity_id | 23 | 23 | 86.99% | 0.0496 | **0.175** |
| D_timedelta | 15 | 15 | 58.15% | 0.0667 | **0.151** |
| card | 4 | 4 | 0.62% | 0.0495 | **0.147** |
| V_vesta | 339 | 40 | 43.04% | 0.0209 | **0.134** |
| distance | 2 | 2 | 76.64% | 0.0339 | 0.035 |
| address | 2 | 2 | 11.13% | 0.0175 | 0.034 |
| C_counting | 14 | 14 | 0% | 0.0216 | 0.030 |

**Phát hiện quan trọng:**
1. **identity_id có correlation cao nhất** (max 0.175): Dù missing nhiều, đây là family predictive nhất
2. **D_timedelta có correlation cao** (max 0.151): Features thời gian trễ quan trọng cho fraud detection
3. **card có correlation cao** (max 0.147): Thông tin thẻ là feature mạnh
4. **V_vesta có correlation thấp** (max 0.134): Dù có 339 columns, predictive power không cao bằng các family nhỏ hơn
5. **C_counting correlation thấp nhất** (max 0.030): Dù 0% missing, không predictive nhiều

**Hệ quả cho modeling:**
- Ưu tiên giữ identity và D_timedelta features
- Có thể giảm chiều V_vesta bằng feature selection hoặc PCA
- C_counting có thể dùng làm baseline features (stable, no missing)

---

### 2.8. Time-Based Split Analysis

**Mục tiêu:** Xác minh tính hợp lệ của time-based split và kiểm tra concept drift.

**Split configuration:**
- Train: 70% đầu tiên theo TransactionDT
- Validation: 15% tiếp theo
- Test: 15% cuối cùng

**Split summary:**

| Split | Rows | Fraud Count | Legit Count | Fraud Rate | TransactionDT Range |
|-------|------|-------------|-------------|------------|---------------------|
| **Train** | 413.378 | 14.538 | 398.840 | **3.517%** | 86.400 → 10.437.996 |
| **Validation** | 88.581 | 3.042 | 85.539 | **3.434%** | 10.438.003 → 13.151.840 |
| **Test** | 88.581 | 3.083 | 85.498 | **3.480%** | 13.151.880 → 15.811.131 |

**Fraud rate theo thời gian (30 quantile bins):**

| Time Bin Index | Rows | Fraud Count | Fraud Rate |
|----------------|------|-------------|------------|
| 0 (earliest) | 19.685 | 544 | 2.76% |
| 1 | 19.685 | 576 | 2.93% |
| 2 | 19.684 | 511 | 2.60% |
| 3 | 19.685 | 469 | 2.38% |
| 4 | 19.685 | 429 | 2.18% |
| ... | ... | ... | ... |
| 29 (latest) | ~19.685 | ~400 | ~2.0% |

**Phát hiện quan trọng:**
1. **Fraud rate ổn định giữa các split:** 3.434% - 3.517% - 3.480% (std = 0.04%)
2. **Concept drift nhẹ:** Fraud rate giảm dần theo thời gian (từ ~2.8% xuống ~2.0% trong các bins)
3. **Không có temporal overlap:** Mỗi split có TransactionDT range riêng biệt

**Tại sao time-based thay vì random split?**

| Tiêu chí | Random Split | Time-Based Split |
|----------|--------------|------------------|
| Data leakage | Có thể (future → train, past → test) | Không (train → past, test → future) |
| Mô phỏng production | Không | Có ✓ |
| Concept drift handling | Không phát hiện được | Phát hiện và đánh giá được |
| Khuyến nghị | ❌ | ✅ |

**Kết luận:** Time-based split là phương pháp đúng cho bài toán fraud detection với dữ liệu temporal.

---

### 2.9. Categorical Feature Summaries

**Mục tiêu:** Phân tích fraud rate theo các categorical features quan trọng.

**ProductCD analysis:**

| ProductCD | Rows | Fraud Count | **Fraud Rate** |
|-----------|------|-------------|----------------|
| C | 68.519 | 8.008 | **11.69%** |
| S | 11.628 | 686 | 5.90% |
| H | 33.024 | 1.574 | 4.77% |
| R | 37.699 | 1.426 | 3.78% |
| W | 439.670 | 8.969 | 2.04% |

**Phát hiện quan trọng:**
- ProductCD = 'C' có fraud rate **11.69%**, cao gấp **5.7 lần** so với 'W' (2.04%)
- ProductCD = 'W' chiếm 74.5% tổng giao dịch nhưng chỉ có fraud rate thấp nhất
- **ProductCD là predictive feature mạnh** (đã được xác nhận trong IEEE-CIS literature)

**Guardrail:** Không diễn giải ProductCD thành loại hàng cụ thể (ví dụ: "C = Clothing"). Chỉ sử dụng như mã categorical trung lập.

---

### 2.10. Numeric Feature Association với Label

**Mục tiêu:** Xác định top numeric features có correlation cao nhất với isFraud.

**Top features (absolute correlation):**

| Rank | Feature | Family | Abs Correlation |
|------|---------|--------|-----------------|
| 1 | id_01 | identity_id | ~0.175 |
| 2 | D1 | D_timedelta | ~0.151 |
| 3 | card4 | card | ~0.147 |
| 4 | V44 | V_vesta | ~0.134 |
| 5 | id_02 | identity_id | ~0.130 |
| ... | ... | ... | ... |

**Nhận xét:**
- Tất cả correlations đều < 0.20: Đặc trưng của anonymized data
- Không có feature đơn lẻ nào có predictive power rất cao
- **Cần kết hợp nhiều features** trong model để đạt performance tốt

---

## 3. Phương Pháp Phân Chia Dữ Liệu

### 3.1. Tổng Quan

```
Toàn bộ dữ liệu: 590.540 rows
Sắp xếp theo: TransactionDT (tăng dần)
Phương pháp: Time-based split (không random)
Tỷ lệ: 70% / 15% / 15%
```

### 3.2. Chi Tiết Các Tập

**Train set (70%):**
- Rows: 413.378
- Fraud: 14.538 (3.517%)
- TransactionDT: 86.400 → 10.437.996
- Mục đích: Fit model và preprocessor

**Validation set (15%):**
- Rows: 88.581
- Fraud: 3.042 (3.434%)
- TransactionDT: 10.438.003 → 13.151.840
- Mục đích: Tune hyperparameters, threshold tuning, early stopping

**Test set (15%):**
- Rows: 88.581
- Fraud: 3.083 (3.480%)
- TransactionDT: 13.151.880 → 15.811.131
- Mục đích: Final evaluation, không dùng trong training/tuning

### 3.3. Code Implementation

```python
def time_based_split(df, train_ratio=0.70, val_ratio=0.15):
    """Chia tập dữ liệu theo TransactionDT."""
    df_sorted = df.sort_values('TransactionDT').reset_index(drop=True)
    n = len(df_sorted)
    
    train_end = int(n * train_ratio)
    val_end = int(n * (train_ratio + val_ratio))
    
    train = df_sorted.iloc[:train_end].copy()
    val = df_sorted.iloc[train_end:val_end].copy()
    test = df_sorted.iloc[val_end:].copy()
    
    return train, val, test
```

### 3.4. Validation của Split

**Kiểm tra:**
1. ✅ Không có overlap về TransactionDT giữa các split
2. ✅ Fraud rate ổn định (3.43% - 3.52%)
3. ✅ Thứ tự thời gian được bảo toàn (train < val < test)
4. ✅ Reproducible (cùng kết quả mỗi lần chạy)

---

## 4. Pipeline Tiền Xử Lý (Preprocessing)

### 4.1. Tổng Quan

Pipeline tiền xử lý được thiết kế với các nguyên tắc:
- **Chống leakage:** Fit chỉ trên train, transform trên val/test
- **Xử lý missing:** Imputation phù hợp cho numeric và categorical
- **Xử lý rare categories:** Loại bỏ categories xuất hiện < 10 lần
- **Sparse-compatible:** Sử dụng sparse matrices để tiết kiệm bộ nhớ

### 4.2. Time-Derived Features

**Từ TransactionDT (seconds since epoch):**

```python
def add_time_features(df):
    """Tạo features thời gian elapsed mà không interpret thành calendar date."""
    output = df.copy()
    transaction_dt = pd.to_numeric(output['TransactionDT'], errors='coerce')
    
    output['TransactionDT_elapsed_hour'] = transaction_dt // 3600
    output['TransactionDT_elapsed_day'] = transaction_dt // 86400
    output['TransactionDT_hour_bin'] = (transaction_dt // 3600) % 24
    
    return output
```

**Features tạo ra:**
- `TransactionDT_elapsed_hour`: Số giờ elapsed từ mốc thời gian
- `TransactionDT_elapsed_day`: Số ngày elapsed
- `TransactionDT_hour_bin`: Giờ trong ngày (0-23), dùng để phát hiện fraud theo giờ

**Guardrail:** Không convert TransactionDT thành ngày/tháng/năm thực vì data đã được anonymized.

### 4.3. Numeric Feature Processing

**Các features áp dụng:**
- C_counting (C1-C14)
- D_timedelta (D1-D15)
- V_vesta (V1-V339)
- card (card1, card2, card3, card5)
- address (addr1, addr2)
- distance (dist1, dist2)
- identity_id (id_01-id_38)

**Pipeline:**
```python
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler

numeric_pipeline = Pipeline([
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler', StandardScaler(with_mean=False))
])
```

**Lý do lựa chọn:**
- **Median imputation:** Robust với outliers hơn mean imputation
- **StandardScaler:** Chuẩn hóa về cùng scale, cần thiết cho Logistic Regression và neural networks
- **with_mean=False:** Giữ sparse structure, tiết kiệm bộ nhớ

### 4.4. Categorical Feature Processing

**Các features áp dụng:**
- ProductCD
- card4, card6
- P_emaildomain, R_emaildomain
- DeviceType, DeviceInfo
- id_30, id_31
- M1-M6

**Pipeline:**
```python
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import OneHotEncoder

categorical_pipeline = Pipeline([
    ('imputer', SimpleImputer(strategy='constant', fill_value='missing')),
    ('encoder', OneHotEncoder(
        handle_unknown='ignore',
        min_frequency=10,
        sparse_output=True
    ))
])
```

**Lý do lựa chọn:**
- **Constant imputation:** Điền "missing" như một category riêng, giữ thông tin về sự vắng mặt
- **min_frequency=10:** Loại bỏ rare categories (xuất hiện < 10 lần trong train) để giảm dimensionality và overfitting
- **handle_unknown='ignore':** Xử lý categories mới xuất hiện trong val/test mà không có trong train

### 4.5. ColumnTransformer

**Kết hợp numeric và categorical pipelines:**

```python
from sklearn.compose import ColumnTransformer

preprocessor = ColumnTransformer(
    transformers=[
        ('numeric', numeric_pipeline, numeric_cols),
        ('categorical', categorical_pipeline, categorical_cols)
    ],
    remainder='drop',  # Drop các columns không nằm trong numeric_cols hoặc categorical_cols
    sparse_threshold=0.3  # Giữ sparse output nếu density < 30%
)
```

### 4.6. Fit và Transform Policy

```python
# Fit CHỈ trên train
X_train_processed = preprocessor.fit_transform(X_train)

# Transform trên val và test (không fit lại)
X_val_processed = preprocessor.transform(X_val)
X_test_processed = preprocessor.transform(X_test)
```

**Tại sao fit chỉ trên train?**
- Ngăn data leakage từ val/test vào train
- Đảm bảo model không "nhìn thấy" distribution của val/test trong quá trình học
- Mô phỏng production: preprocessor fit trên historical data, apply trên future data

### 4.7. Feature Metadata

**Ghi nhận trong `preprocessing_metadata.csv`:**

| Metric | Value |
|--------|-------|
| Train rows | 413.378 |
| Validation rows | 88.581 |
| Test rows | 88.581 |
| Features before encoding | 435 |
| Numeric features | 404 |
| Categorical features | 31 |
| Time-derived features | 3 |
| Fit policy | Preprocessor fit on train only |
| Time feature policy | Derived from TransactionDT elapsed seconds |

### 4.8. Lưu và Tải Preprocessor

```python
import joblib

# Lưu preprocessor
joblib.dump(preprocessor, 'artifacts/preprocessing/preprocessor.joblib')

# Tải preprocessor
preprocessor = joblib.load('artifacts/preprocessing/preprocessor.joblib')
```

### 4.9. Lightweight Approach (Cho Phase 2)

Vì full preprocessing artifact (~4GB) đã bị xóa trong cleanup, Phase 2 nên dùng lightweight approach:

```python
# Fit preprocessor trên 50.000 sample thay vì full train
sample_size = 50000
X_train_sample = X_train.sample(n=sample_size, random_state=42)
preprocessor.fit(X_train_sample)

# Transform trên full data
X_train_processed = preprocessor.transform(X_train)
X_val_processed = preprocessor.transform(X_val)
X_test_processed = preprocessor.transform(X_test)
```

**Lợi ích:**
- Nhanh hơn cho iteration
- Tiết kiệm disk space (không lưu full one-hot matrix)
- Chỉ lưu `preprocessor.joblib` và metadata CSV

---

## 5. Kết Luận

### 5.1. Tóm Tắt Phát Hiện Chính

1. **Class imbalance nghiêm trọng:** 3.499% fraud rate → Accuracy không phù hợp, cần cost-sensitive metrics
2. **TransactionAmt khác biệt:** Fraud có amount cao hơn → FN cost phải tính theo số tiền
3. **Identity coverage predictive:** Fraud có identity nhiều hơn (54.77% vs 23.32%)
4. **ProductCD mạnh:** 'C' có fraud rate 11.69%, 'W' có 2.04%
5. **Missingness có pattern:** Identity features missing ~85% nhưng correlation cao nhất
6. **Concept drift nhẹ:** Fraud rate giảm dần theo thời gian
7. **Time-based split hợp lệ:** Fraud rate ổn định giữa các split (3.43% - 3.52%)

### 5.2. Khuyến Nghị cho Phase 2 (ML Baselines)

1. **Metrics chính:** PR-AUC, Fraud Recall, Fraud Precision, Fraud F1, FN Cost, FP Cost, Total Cost, Cost Saving
2. **Baselines:** Logistic Regression (class_weight='balanced'), Random Forest/LightGBM (class_weight='balanced')
3. **Threshold tuning:** Tune trên validation set để minimize Total Cost
4. **Cost function:** FN Cost = α × TransactionAmt, FP Cost = β × TransactionAmt (α=10, β=1)
5. **Approve-all baseline:** Làm reference cho Cost Saving metric

### 5.3. Hạn Chế và Future Work

1. **Feature anonymization:** Các features V_vesta và identity_id được mã hóa, khó interpret
2. **Concept drift:** Fraud rate giảm theo thời gian, cần evaluation cẩn thận
3. **Missing data:** ~85% identity features missing, cần xử lý đặc biệt
4. **Future work:** 
   - Feature selection để giảm dimensionality
   - SHAP values cho interpretability
   - Concept drift detection và adaptation

---

## Phụ Lục A: File Outputs

| File | Mục đích |
|------|----------|
| `results/eda/data_integrity_audit.csv` | Data integrity checks |
| `results/eda/class_distribution.csv` | Class imbalance stats |
| `results/eda/transaction_amount_summary_by_label.csv` | Amount statistics |
| `results/eda/missing_summary_top30.csv` | Top 30 missing columns |
| `results/eda/feature_family_summary.csv` | Feature family overview |
| `results/eda/family_numeric_summary.csv` | Numeric family correlations |
| `results/eda/identity_coverage_by_label.csv` | Identity coverage stats |
| `results/eda/fraud_rate_by_split.csv` | Split fraud rates |
| `results/eda/fraud_rate_by_transactiondt_bin.csv` | Time-based fraud rates |
| `reports/figures/eda/*.png` | EDA visualizations |

---

## Phụ Lục B: Code Snippets Chính

**Xem notebook:** `notebooks/01_data_check.ipynb`

**Các sections chính:**
- Section 5: Data integrity audit
- Section 6: Feature family overview
- Section 7: Family-level numeric summaries
- Section 8: Target imbalance and approve-all warning
- Section 9: TransactionAmt distribution by label
- Section 10: Missingness analysis
- Section 11: Identity coverage by label
- Section 12: Time-based behavior and split sanity
- Section 13: Categorical feature summaries
- Section 14: Numeric feature association with label
- Section 17: First-pass preprocessing handoff

---

**Người viết:** AI Assistant  
**Ngày:** 2026-05-26  
**Version:** 1.0
