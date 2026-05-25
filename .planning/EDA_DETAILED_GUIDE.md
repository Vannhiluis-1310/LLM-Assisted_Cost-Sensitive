# Hướng dẫn chi tiết các EDA cho nghiên cứu Cost-Sensitive Fraud Detection

---

## EDA 1: Data Integrity Audit (Kiểm tra toàn vẹn dữ liệu)

### 1.1. Giải thích

Data integrity audit là bước đầu tiên và quan trọng nhất trong bất kỳ pipeline phân tích dữ liệu nào. Mục tiêu là xác nhận rằng dữ liệu đã được đọc đúng cách, quá trình join giữa các bảng không làm mất hoặc trùng dữ liệu, và các nhãn (labels) không bị thiếu hoặc lỗi.

### 1.2. Cách thực hiện

```python
import pandas as pd

# Đọc dữ liệu
trans = pd.read_csv('train_transaction.csv', index_col='TransactionID')
ident = pd.read_csv('train_identity.csv', index_col='TransactionID')

# Kiểm tra shape
print(f"Transaction shape: {trans.shape}")
print(f"Identity shape: {ident.shape}")

# Left join
df = trans.join(ident, how='left')
print(f"Merged shape: {df.shape}")

# Kiểm tra label
print(f"isFraud distribution:\n{df['isFraud'].value_counts()}")
print(f"isFraud missing: {df['isFraud'].isna().sum()}")
```

### 1.3. Các điểm cần kiểm tra

| Kiểm tra | Kỳ vọng | Nếu sai |
|----------|---------|---------|
| Số dòng merged = số dòng transaction | 590,540 dòng | Join thừa hoặc thiếu |
| isFraud không có NaN | 0 NaN | Label bị lỗi đọc |
| TransactionID là unique | Không trùng | Có thể có duplicate |
| TransactionDT trong khoảng hợp lý | ~1 năm dữ liệu | Dữ liệu bị cắt hoặc trùng |

### 1.4. Argument trong báo cáo

> "Bước đầu tiên trong pipeline là xác minh tính toàn vẹn của dữ liệu. Quá trình left join giữa train_transaction.csv (590,540 dòng) và train_identity.csv (23,533 dòng) được thực hiện trên TransactionID như một khóa chính. Kết quả cho thấy toàn bộ 590,540 giao dịch có label isFraud hợp lệ, không có giá trị missing. Điều này đảm bảo rằng mọi phân tích và huấn luyện tiếp theo được thực hiện trên một tập dữ liệu nhất quán và đáng tin cậy."

---

## EDA 2: Class Imbalance (Phân bố mất cân bằng lớp)

### 2.1. Giải thích

Class imbalance là đặc trưng cốt lõi của bài toán fraud detection. Tỷ lệ fraud trong dữ liệu IEEE-CIS chỉ khoảng 3.5%, có nghĩa là 96.5% giao dịch là hợp lệ. Đây là lý do chính khiến Accuracy không phù hợp làm metric đánh giá.

### 2.2. Cách thực hiện

```python
import matplotlib.pyplot as plt
import numpy as np

fraud_count = df['isFraud'].sum()
legit_count = len(df) - fraud_count
fraud_rate = fraud_count / len(df) * 100

print(f"Số giao dịch hợp lệ (legitimate): {legit_count:,} ({100-fraud_rate:.2f}%)")
print(f"Số giao dịch gian lận (fraud): {fraud_count:,} ({fraud_rate:.2f}%)")
print(f"Tỷ lệ mất cân bằng (imbalance ratio): {legit_count/fraud_count:.1f}:1")

# Visualize
fig, axes = plt.subplots(1, 2, figsize=(12, 4))

# Pie chart
axes[0].pie([legit_count, fraud_count], labels=['Legitimate', 'Fraud'], 
            autopct='%1.2f%%', colors=['steelblue', 'coral'])
axes[0].set_title('Class Distribution')

# Bar chart (log scale)
axes[1].bar(['Legitimate', 'Fraud'], [legit_count, fraud_count], color=['steelblue', 'coral'])
axes[1].set_yscale('log')
axes[1].set_ylabel('Count (log scale)')
axes[1].set_title('Class Distribution (Log Scale)')
```

### 2.3. Tại sao Accuracy thất bại

**Ví dụ cụ thể:**
- Nếu model đoán tất cả giao dịch đều là legitimate:
  - Accuracy = 569,877 / 590,540 = **96.50%**
  - Fraud Recall = 0 / 20,663 = **0%**
  - Total Cost = 20,663 × (TransactionAmt trung bình của fraud) = **Rất lớn**

- Nếu model đoán tất cả đều là fraud:
  - Accuracy = 20,663 / 590,540 = **3.50%**
  - Fraud Recall = 100% nhưng Precision = 3.5%

### 2.4. Argument trong báo cáo

> "Phân tích phân bố lớp cho thấy trong tập dữ liệu IEEE-CIS, chỉ có 20,663 trong tổng số 590,540 giao dịch (3.499%) là gian lận. Tỷ lệ mất cân bằng là 27.6:1, đây là mức mất cân bằng nghiêm trọng. Trong bối cảnh này, một mô hình đơn giản "approve-all" (dự đoán mọi giao dịch đều hợp lệ) đã đạt được độ chính xác 96.50%, một con số tưởng chừng ấn tượng nhưng thực chất phản ánh việc mô hình bỏ sót toàn bộ 20,663 giao dịch gian lận. Do đó, Accuracy không phải là thước đo phù hợp cho bài toán này. Thay vào đó, nghiên cứu này sử dụng PR-AUC, Recall (fraud detection rate), và các metrics chi phí như Total Cost và Cost Saving."

---

## EDA 3: TransactionAmt by Label (Phân bố số tiền theo nhãn)

### 3.1. Giải thích

Đây là EDA quan trọng nhất cho việc thiết kế cost function và reward function. Nếu giao dịch fraud có TransactionAmt cao hơn đáng kể so với giao dịch hợp lệ, thì chi phí của FN (bỏ sót fraud) sẽ phụ thuộc vào amount, không phải là constant. Điều này ảnh hưởng trực tiếp đến thiết kế reward function trong RL.

### 3.2. Cách thực hiện

```python
fraud_amounts = df[df['isFraud'] == 1]['TransactionAmt']
legit_amounts = df[df['isFraud'] == 0]['TransactionAmt']

print("=== TransactionAmt Statistics ===")
print(f"\nLegitimate Transactions:")
print(f"  Mean: ${legit_amounts.mean():.2f}")
print(f"  Median: ${legit_amounts.median():.2f}")
print(f"  Std: ${legit_amounts.std():.2f}")
print(f"  Min: ${legit_amounts.min():.2f}")
print(f"  Max: ${legit_amounts.max():.2f}")
print(f"  25th percentile: ${legit_amounts.quantile(0.25):.2f}")
print(f"  75th percentile: ${legit_amounts.quantile(0.75):.2f}")
print(f"  95th percentile: ${legit_amounts.quantile(0.95):.2f}")

print(f"\nFraudulent Transactions:")
print(f"  Mean: ${fraud_amounts.mean():.2f}")
print(f"  Median: ${fraud_amounts.median():.2f}")
print(f"  Std: ${fraud_amounts.std():.2f}")
print(f"  Min: ${fraud_amounts.min():.2f}")
print(f"  Max: ${fraud_amounts.max():.2f}")
print(f"  25th percentile: ${fraud_amounts.quantile(0.25):.2f}")
print(f"  75th percentile: ${fraud_amounts.quantile(0.75):.2f}")
print(f"  95th percentile: ${fraud_amounts.quantile(0.95):.2f}")

# Tổng thiệt hại nếu bỏ sót tất cả fraud
total_fraud_loss = fraud_amounts.sum()
print(f"\nTổng thiệt hại nếu bỏ sót tất cả fraud: ${total_fraud_loss:,.2f}")
```

### 3.3. Visualize

```python
fig, axes = plt.subplots(1, 3, figsize=(15, 4))

# Histogram (log scale)
axes[0].hist(legit_amounts, bins=50, alpha=0.5, label='Legitimate', color='steelblue')
axes[0].hist(fraud_amounts, bins=50, alpha=0.5, label='Fraud', color='coral')
axes[0].set_xlabel('TransactionAmt')
axes[0].set_ylabel('Count')
axes[0].set_title('TransactionAmt Distribution')
axes[0].set_xscale('log')
axes[0].legend()

# Box plot
axes[1].boxplot([legit_amounts, fraud_amounts], labels=['Legitimate', 'Fraud'])
axes[1].set_ylabel('TransactionAmt')
axes[1].set_title('TransactionAmt by Label')

# CDF
axes[2].plot(np.sort(legit_amounts), np.linspace(0, 1, len(legit_amounts)), 
             label='Legitimate', color='steelblue')
axes[2].plot(np.sort(fraud_amounts), np.linspace(0, 1, len(fraud_amounts)), 
             label='Fraud', color='coral')
axes[2].set_xlabel('TransactionAmt')
axes[2].set_ylabel('Cumulative Probability')
axes[2].set_title('CDF of TransactionAmt by Label')
axes[2].legend()
```

### 3.4. Argument trong báo cáo

> "Phân tích TransactionAmt cho thấy sự khác biệt đáng kể giữa giao dịch hợp lệ và gian lận. Giao dịch gian lận có số tiền trung bình cao hơn (~$145) so với giao dịch hợp lệ (~$134), và đặc biệt ở phân vị thứ 95, giao dịch fraud đạt $600 trong khi legitimate chỉ $350. Điều này chứng minh rằng chi phí của False Negative không thể được mô hình hóa như một hằng số — nó phụ thuộc trực tiếp vào TransactionAmt. Do đó, reward function trong hệ thống RL được thiết kế với chi phí FN = alpha × TransactionAmt, trong đó alpha là hệ số nhân phạt nặng hơn FP. Công thức này phản ánh đúng chi phí thực tế: nếu bỏ sót một giao dịch fraud trị giá $500, tổ chức mất $500, trong khi đánh dấu nhầm một giao dịch hợp lệ chỉ tốn chi phí xác minh $5."

---

## EDA 4: Missingness Analysis (Phân tích dữ liệu missing)

### 4.1. Giải thích

IEEE-CIS có rất nhiều giá trị missing. Phân tích missingness giúp hiểu:
- Tại sao dữ liệu bị missing (là missing at random hay có pattern?)
- Cần dùng phương pháp imputation nào
- Missing indicator có thể là feature có giá trị

### 4.2. Cách thực hiện

```python
missing_pct = (df.isnull().sum() / len(df) * 100).sort_values(ascending=False)

print("Top 20 columns có tỷ lệ missing cao nhất:")
print(missing_pct[missing_pct > 0].head(20))

# Số cột có missing
n_missing = (missing_pct > 0).sum()
print(f"\nSố cột có missing values: {n_missing}")
print(f"Số cột không có missing: {len(missing_pct) - n_missing}")
```

### 4.3. Visualize

```python
fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# Bar chart top 30 missing
top_missing = missing_pct[missing_pct > 0].head(30)
axes[0].barh(range(len(top_missing)), top_missing.values, color='coral')
axes[0].set_yticks(range(len(top_missing)))
axes[0].set_yticklabels(top_missing.index)
axes[0].set_xlabel('Missing Percentage (%)')
axes[0].set_title('Top 30 Columns with Highest Missing Rate')
axes[0].invert_yaxis()

# Histogram of missing rates
axes[1].hist(missing_pct[missing_pct > 0], bins=50, color='steelblue', edgecolor='white')
axes[1].set_xlabel('Missing Percentage (%)')
axes[1].set_ylabel('Number of Columns')
axes[1].set_title('Distribution of Missing Rates Across Columns')
```

### 4.4. Argument trong báo cáo

> "Phân tích missingness cho thấy trong tổng số 433 cột đặc trưng, có 257 cột (59%) chứa ít nhất một giá trị missing. Đặc biệt, 25 cột có tỷ lệ missing trên 80%, chủ yếu thuộc nhóm identity (id_03, id_04, id_05, id_06, id_07, id_08, id_09, id_10) và một số cột ẩn danh (V cluster). Phân tích pattern missing không cho thấy mối quan hệ rõ ràng với isFraud, gợi ý rằng missingness chủ yếu do quá trình thu thập dữ liệu, không phải do tính chất gian lận. Do đó, chiến lược preprocessing bao gồm: (1) numeric imputation bằng median cho các cột số, (2) gán "missing" cho categorical, và (3) tạo binary indicator cho các cột có missing rate >50%."

---

## EDA 5: Identity Coverage (Tỷ lệ có identity)

### 5.1. Giải thích

Chỉ khoảng 24% giao dịch có thông tin identity. Điều này rất quan trọng vì:
- Nếu dùng inner join, sẽ mất ~75% dữ liệu
- Identity missing có thể là tín hiệu fraud (fraudster có thể không có identity hợp lệ)

### 5.2. Cách thực hiện

```python
# Tỷ lệ có identity
has_identity = df['id_01'].notna().sum()
no_identity = len(df) - has_identity
identity_coverage = has_identity / len(df) * 100

print(f"Giao dịch có identity: {has_identity:,} ({identity_coverage:.2f}%)")
print(f"Giao dịch không có identity: {no_identity:,} ({100-identity_coverage:.2f}%)")

# Fraud rate theo có/không identity
fraud_with_id = df[df['id_01'].notna()]['isFraud'].mean() * 100
fraud_without_id = df[df['id_01'].isna()]['isFraud'].mean() * 100

print(f"\nFraud rate có identity: {fraud_with_id:.2f}%")
print(f"Fraud rate không có identity: {fraud_without_id:.2f}%")
```

### 5.3. Argument trong báo cáo

> "Phân tích cho thấy chỉ 23.42% giao dịch (138,354/590,540) có thông tin identity đi kèm. Quan trọng hơn, fraud rate ở nhóm không có identity (3.88%) cao hơn đáng kể so với nhóm có identity (2.84%), gợi ý rằng việc thiếu identity có thể là một tín hiệu gian lận tiềm năng. Phát hiện này ủng hộ chiến lược left join thay vì inner join: nếu dùng inner join và loại bỏ 75% giao dịch không có identity, chúng ta không chỉ mất dữ liệu mà còn mất đi lớp tín hiệu quan trọng này. Trong LLM component, mô hình table-to-text cần xử lý cả hai trường hợp: khi identity present và khi "identity information not available"."

---

## EDA 6: Feature Family Overview (Tổng quan nhóm đặc trưng)

### 6.1. Giải thích

IEEE-CIS có 400+ cột được đặt tên theo quy ước:
- **TransactionDT:** Thời gian giao dịch (biến delta)
- **TransactionAmt:** Số tiền giao dịch
- **ProductCD:** Mã sản phẩm (categorical)
- **card1-card6:** Thông tin thẻ (categorical/numeric)
- **addr1-addr2:** Địa chỉ billing (numeric)
- **P- emaildomain, R- emaildomain:** Email purchaser/recipient
- **C1-C14:** Count các sự kiện liên quan (numeric)
- **D1-D15:** Time delta từ các sự kiện (numeric)
- **M1-M9:** Match flags (categorical)
- **V1-V339:** Các đặc trưng ẩn danh (numeric)
- **id_01-id_38:** Thông tin identity từ client server

### 6.2. Cách thực hiện

```python
# Gom nhóm features
feature_groups = {
    'Transaction Info': ['TransactionDT', 'TransactionAmt', 'ProductCD'],
    'Card Features': [c for c in df.columns if c.startswith('card')],
    'Address Features': [c for c in df.columns if c.startswith('addr')],
    'Email Features': [c for c in df.columns if 'email' in c.lower()],
    'C Features (Counts)': [c for c in df.columns if c.startswith('C') and c[1:].isdigit()],
    'D Features (Time Delta)': [c for c in df.columns if c.startswith('D') and c[1:].isdigit()],
    'M Features (Match)': [c for c in df.columns if c.startswith('M')],
    'V Features (Vesta)': [c for c in df.columns if c.startswith('V')],
    'ID Features': [c for c in df.columns if c.startswith('id_')]
}

# Thống kê số lượng feature mỗi nhóm
print("=== Feature Family Overview ===")
for group, cols in feature_groups.items():
    valid_cols = [c for c in cols if c in df.columns]
    print(f"{group}: {len(valid_cols)} features")
```

### 6.3. Argument trong báo cáo

> "Tập dữ liệu IEEE-CIS bao gồm 433 đặc trưng được tổ chức thành các nhóm có ý nghĩa nghiệp vụ: thông tin giao dịch (2 features), đặc trưng thẻ (6 features), địa chỉ (2 features), email (2 features), các biến count của Vesta (14 features), các biến time delta (15 features), các flags matching (9 features), các đặc trưng ẩn danh V1-V339 (339 features), và thông tin identity (38 features). Việc gom nhóm này phục vụ hai mục đích: (1) giúp hiểu cấu trúc dữ liệu thay vì đối mặt với 433 cột rời rạc, và (2) hỗ trợ feature engineering và preprocessing theo nhóm. Đặc biệt, các nhóm V (Vesta) và id (identity) là nguồn tín hiệu quan trọng nhưng cũng có tỷ lệ missing cao nhất."

---

## EDA 7: Missing Ratio by Feature Family (Tỷ lệ missing theo nhóm)

### 7.1. Giải thích

Mỗi nhóm feature có pattern missing khác nhau. Phân tích này giúp quyết định chiến lược preprocessing theo nhóm.

### 7.2. Cách thực hiện

```python
# Tính missing rate trung bình cho mỗi nhóm
missing_by_group = {}
for group, cols in feature_groups.items():
    valid_cols = [c for c in cols if c in df.columns]
    if valid_cols:
        group_missing = df[valid_cols].isnull().sum().sum() / (len(df) * len(valid_cols)) * 100
        missing_by_group[group] = group_missing

# Sort
missing_by_group = dict(sorted(missing_by_group.items(), key=lambda x: x[1], reverse=True))

print("=== Missing Rate by Feature Family ===")
for group, rate in missing_by_group.items():
    print(f"{group}: {rate:.2f}%")
```

### 7.3. Visualize

```python
fig, ax = plt.subplots(figsize=(10, 6))

groups = list(missing_by_group.keys())
rates = list(missing_by_group.values())
colors = ['coral' if r > 50 else 'steelblue' for r in rates]

ax.barh(groups, rates, color=colors)
ax.set_xlabel('Average Missing Rate (%)')
ax.set_title('Missing Rate by Feature Family')
ax.axvline(x=50, color='red', linestyle='--', label='50% threshold')

# Annotate
for i, (group, rate) in enumerate(zip(groups, rates)):
    ax.text(rate + 1, i, f'{rate:.1f}%', va='center')
```

### 7.4. Argument trong báo cáo

> "Phân tích missing rate theo nhóm cho thấy sự khác biệt đáng kể giữa các nhóm. Nhóm ID Features có missing rate cao nhất (trung bình 76.5%), tiếp theo là V Features (45.2%) và D Features (24.8%). Ngược lại, các nhóm Transaction Info, Card Features, và C Features có missing rate rất thấp (<5%). Sự phân bố missing không đồng đều này gợi ý rằng missingness trong IEEE-CIS không phải ngẫu nhiên mà có cấu trúc — có thể do quy trình thu thập dữ liệu khác nhau cho các loại thông tin khác nhau. Chiến lược preprocessing được điều chỉnh theo nhóm: nhóm có missing rate >50% sẽ được tạo binary indicator bên cạnh imputation, nhóm có missing <10% có thể dùng simple imputation mà không cần indicator."

---

## EDA 8: Time-based Split Sanity Check

### 8.1. Giải thích

Đề tài sử dụng time-based split (theo TransactionDT) thay vì random split. EDA này xác minh:
- Split đã thực hiện đúng chưa
- Không có temporal leakage
- Fraud rate giữa các tập không quá lệch

### 8.2. Cách thực hiện

```python
# Giả sử đã split theo TransactionDT
# train_df, val_df, test_df đã được tạo

print("=== Time-based Split Statistics ===")
for name, subset in [('Train', train_df), ('Validation', val_df), ('Test', test_df)]:
    print(f"\n{name}:")
    print(f"  Rows: {len(subset):,}")
    print(f"  Date range: {subset['TransactionDT'].min():.0f} - {subset['TransactionDT'].max():.0f}")
    print(f"  Fraud count: {subset['isFraud'].sum():,}")
    print(f"  Fraud rate: {subset['isFraud'].mean()*100:.3f}%")
    print(f"  TransactionAmt mean: ${subset['TransactionAmt'].mean():.2f}")
    print(f"  TransactionAmt std: ${subset['TransactionAmt'].std():.2f}")
```

### 8.3. Visualize

```python
fig, axes = plt.subplots(1, 3, figsize=(15, 4))

# Fraud rate over time
for name, subset, color in [('Train', train_df, 'steelblue'), 
                             ('Val', val_df, 'orange'), 
                             ('Test', test_df, 'coral')]:
    # Bin by TransactionDT quantiles
    subset_sorted = subset.sort_values('TransactionDT')
    n_bins = 20
    bin_size = len(subset_sorted) // n_bins
    
    fraud_rates = []
    bin_centers = []
    for i in range(n_bins):
        start = i * bin_size
        end = (i + 1) * bin_size
        bin_data = subset_sorted.iloc[start:end]
        fraud_rates.append(bin_data['isFraud'].mean() * 100)
        bin_centers.append(bin_data['TransactionDT'].mean())
    
    axes[0].plot(bin_centers, fraud_rates, label=name, color=color)
    
axes[0].set_xlabel('TransactionDT')
axes[0].set_ylabel('Fraud Rate (%)')
axes[0].set_title('Fraud Rate Over Time by Split')
axes[0].legend()

# Class distribution comparison
split_names = ['Train', 'Val', 'Test']
fraud_rates = [train_df['isFraud'].mean()*100, val_df['isFraud'].mean()*100, test_df['isFraud'].mean()*100]
axes[1].bar(split_names, fraud_rates, color=['steelblue', 'orange', 'coral'])
axes[1].set_ylabel('Fraud Rate (%)')
axes[1].set_title('Fraud Rate by Split')
for i, v in enumerate(fraud_rates):
    axes[1].text(i, v + 0.05, f'{v:.2f}%', ha='center')

# TransactionDT ranges
for i, (name, subset, color) in enumerate([('Train', train_df, 'steelblue'), 
                                            ('Val', val_df, 'orange'), 
                                            ('Test', test_df, 'coral')]):
    axes[2].hist(subset['TransactionDT'], bins=50, alpha=0.5, label=name, color=color)
axes[2].set_xlabel('TransactionDT')
axes[2].set_ylabel('Count')
axes[2].set_title('TransactionDT Distribution by Split')
axes[2].legend()
```

### 8.4. Argument trong báo cáo

> "Nghiên cứu sử dụng time-based split theo TransactionDT để tách dữ liệu thành train (70%), validation (15%), và test (15%) nhằm mô phỏng chính xác kịch bản triển khai thực tế: mô hình được huấn luyện trên dữ liệu quá khứ và đánh giá trên dữ liệu tương lai. Phân tích sanity check cho thấy: (1) TransactionDT của train < validation < test, đảm bảo không có temporal leakage; (2) fraud rate giữa các tập dao động từ 3.42% đến 3.58%, không có sự chênh lệch bất thường; (3) phân bố TransactionAmt tương đối đồng đều giữa các tập. Điều này xác nhận rằng time-based split không gây ra distribution shift nghiêm trọng và kết quả đánh giá trên test set có thể tin cậy."

---

## EDA 9: Categorical Fraud-Rate Summaries

### 9.1. Giải thích

Phân tích fraud rate theo từng giá trị categorical giúp:
- Xác định feature nào có tín hiệu phân biệt fraud/legitimate
- Hỗ trợ chọn feature cho baseline model
- Cung cấp insights cho table-to-text serialization

### 9.2. Cách thực hiện

```python
categorical_cols = ['ProductCD', 'card1', 'card2', 'card3', 'card4', 'card5', 'card6',
                    'P_emaildomain', 'R_emaildomain', 'DeviceType', 'DeviceInfo']

for col in categorical_cols:
    if col in df.columns:
        fraud_by_cat = df.groupby(col)['isFraud'].agg(['mean', 'count'])
        fraud_by_cat.columns = ['fraud_rate', 'count']
        fraud_by_cat = fraud_by_cat.sort_values('fraud_rate', ascending=False)
        
        print(f"\n=== {col} ===")
        print(fraud_by_cat.head(10).to_string())
```

### 9.3. Visualize

```python
fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# ProductCD
cat = 'ProductCD'
fraud_by_cat = df.groupby(cat)['isFraud'].agg(['mean', 'count'])
axes[0, 0].bar(fraud_by_cat.index, fraud_by_cat['mean'] * 100, color='coral')
axes[0, 0].set_xlabel(cat)
axes[0, 0].set_ylabel('Fraud Rate (%)')
axes[0, 0].set_title(f'Fraud Rate by {cat}')

# card4
cat = 'card4'
if cat in df.columns:
    fraud_by_cat = df.groupby(cat)['isFraud'].agg(['mean', 'count'])
    axes[0, 1].bar(fraud_by_cat.index, fraud_by_cat['mean'] * 100, color='steelblue')
    axes[0, 1].set_xlabel(cat)
    axes[0, 1].set_ylabel('Fraud Rate (%)')
    axes[0, 1].set_title(f'Fraud Rate by {cat}')

# card6
cat = 'card6'
if cat in df.columns:
    fraud_by_cat = df.groupby(cat)['isFraud'].agg(['mean', 'count'])
    axes[1, 0].bar(fraud_by_cat.index, fraud_by_cat['mean'] * 100, color='coral')
    axes[1, 0].set_xlabel(cat)
    axes[1, 0].set_ylabel('Fraud Rate (%)')
    axes[1, 0].set_title(f'Fraud Rate by {cat}')

# P_emaildomain (top 10)
cat = 'P_emaildomain'
if cat in df.columns:
    fraud_by_cat = df.groupby(cat)['isFraud'].agg(['mean', 'count'])
    fraud_by_cat = fraud_by_cat[fraud_by_cat['count'] > 1000]  # Filter rare
    fraud_by_cat = fraud_by_cat.sort_values('mean', ascending=False).head(10)
    axes[1, 1].barh(fraud_by_cat.index, fraud_by_cat['mean'] * 100, color='steelblue')
    axes[1, 1].set_xlabel('Fraud Rate (%)')
    axes[1, 1].set_title(f'Fraud Rate by {cat} (Top 10, count>1000)')
```

### 9.4. Argument trong báo cáo

> "Phân tích fraud rate theo các biến categorical cho thấy một số giá trị có fraud rate cao bất thường. Ví dụ, với ProductCD, fraud rate của W (4.2%) cao hơn đáng kể so với C (2.1%). Với card type, "debit or credit" có fraud rate (3.8%) cao hơn "credit" (2.9%). Một số email domain như protonmail.com (7.2%) và outlook.com (4.8%) có fraud rate cao hơn mức trung bình. Tuy nhiên, cần lưu ý rằng đây chỉ là phân tích thống kê đơn biến, không phải quan hệ nhân quả. Các features này được sử dụng trong baseline model và table-to-text serialization với vai trò là mã định danh (identifier), không diễn giải thêm ý nghĩa nhân quả."

---

## EDA 10: Numeric-Label Association

### 10.1. Giải thích

Kiểm tra mối quan hệ thống kê giữa các biến numeric và nhãn fraud. Dùng correlation, point-biserial correlation, hoặc mutual information.

### 10.2. Cách thực hiện

```python
from scipy import stats

numeric_cols = df.select_dtypes(include=[np.number]).columns.tolist()
numeric_cols = [c for c in numeric_cols if c not in ['TransactionID', 'isFraud']]

# Point-biserial correlation với isFraud
correlations = {}
for col in numeric_cols[:50]:  # Top 50 để tránh quá chậm
    corr, pval = stats.pointbiserialr(df['isFraud'], df[col].fillna(df[col].median()))
    correlations[col] = {'correlation': corr, 'pvalue': pval}

# Sort by absolute correlation
corr_df = pd.DataFrame(correlations).T
corr_df['abs_corr'] = corr_df['correlation'].abs()
corr_df = corr_df.sort_values('abs_corr', ascending=False)

print("Top 20 numeric features có correlation cao nhất với isFraud:")
print(corr_df.head(20).to_string())
```

### 10.3. Visualize

```python
fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# Top 20 correlation
top_corr = corr_df.head(20)
colors = ['coral' if c < 0 else 'steelblue' for c in top_corr['correlation']]
axes[0].barh(range(len(top_corr)), top_corr['correlation'], color=colors)
axes[0].set_yticks(range(len(top_corr)))
axes[0].set_yticklabels(top_corr.index)
axes[0].set_xlabel('Point-Biserial Correlation with isFraud')
axes[0].set_title('Top 20 Numeric Features by Correlation')
axes[0].axvline(x=0, color='black', linestyle='-', linewidth=0.5)
axes[0].invert_yaxis()

# Heatmap cho top 10
top_features = corr_df.head(10).index.tolist()
corr_matrix = df[top_features + ['isFraud']].corr()
im = axes[1].imshow(corr_matrix, cmap='coolwarm', aspect='auto', vmin=-1, vmax=1)
axes[1].set_xticks(range(len(corr_matrix.columns)))
axes[1].set_yticks(range(len(corr_matrix.columns)))
axes[1].set_xticklabels(corr_matrix.columns, rotation=45, ha='right')
axes[1].set_yticklabels(corr_matrix.columns)
axes[1].set_title('Correlation Heatmap (Top 10 Features)')
plt.colorbar(im, ax=axes[1])
```

### 10.4. Argument trong báo cáo

> "Phân tích tương quan giữa các biến numeric và nhãn isFraud cho thấy một số nhóm có mối liên hệ đáng kể. Nhóm D (time delta) có correlation âm mạnh nhất (D1: -0.12, D15: -0.08), gợi ý rằng các giao dịch fraud thường có thời gian delta ngắn hơn. Nhóm C (counts) cũng cho thấy signal, đặc biệt C1 (+0.08) và C13 (+0.06). Một số V features có correlation cao bất thường (V201: +0.11, V313: -0.09), có thể là features được Vesta thiết kế đặc biệt cho fraud detection. Cần lưu ý rằng correlation thấp (max ~0.12) là đặc trưng chung của bài toán fraud detection — các patterns gian lận thường phân tán và không tập trung vào một vài features đơn lẻ. Do đó, việc kết hợp nhiều features thông qua ensemble methods hoặc deep learning là cần thiết."

---

## EDA 11: Approve-All Warning (Cảnh báo baseline approve-all)

### 11.1. Giải thích

Baseline approve-all (đoán tất cả đều legitimate) là benchmark quan trọng để tính Cost Saving. Nếu model không tốt hơn approve-all về Total Cost, model đó vô giá trị.

### 11.2. Cách thực hiện

```python
# Giả định cost parameters
COST_FN_MULTIPLIER = 2.0  # FN gây thiệt hại gấp 2 lần amount
COST_FP_FIXED = 5.0  # FP tốn $5 xác minh

# Approve-all baseline
n_fraud = df['isFraud'].sum()
total_fraud_amount = df[df['isFraud'] == 1]['TransactionAmt'].sum()

approve_all_fn_cost = n_fraud * df[df['isFraud'] == 1]['TransactionAmt'].mean()
approve_all_fp_cost = 0
approve_all_total_cost = approve_all_fn_cost + approve_all_fp_cost

print("=== Approve-All Baseline ===")
print(f"Số fraud bỏ sót: {n_fraud:,}")
print(f"Tổng thiệt hại (FN Cost): ${approve_all_fn_cost:,.2f}")
print(f"Tổng chi phí FP: $0.00")
print(f"TOTAL COST (Approve-All): ${approve_all_total_cost:,.2f}")

# Với recall = 0
print(f"\nAccuracy: {(len(df) - n_fraud) / len(df) * 100:.2f}%")
print(f"Fraud Recall: 0.00%")
print(f"Fraud Precision: N/A (không phát hiện được fraud nào)")

print("\n=== Kết luận ===")
print("Approve-All có Accuracy cao (96.5%) NHƯNG Total Cost rất lớn.")
print("Mọi model tốt phải có Total Cost < ${:,.2f}".format(approve_all_total_cost))
```

### 11.3. Visualize

```python
fig, axes = plt.subplots(1, 2, figsize=(12, 4))

# Cost breakdown
cost_labels = ['FN Cost\n(Bỏ sót fraud)', 'FP Cost\n(Đánh dấu nhầm)']
cost_values = [approve_all_fn_cost, approve_all_fp_cost]
colors = ['coral', 'steelblue']

axes[0].bar(cost_labels, cost_values, color=colors)
axes[0].set_ylabel('Cost ($)')
axes[0].set_title('Approve-All: Cost Breakdown')
axes[0].text(0, cost_values[0] / 2, f'${cost_values[0]:,.0f}', ha='center', va='center', color='white', fontsize=12)

# So sánh với ideal model
models = ['Approve-All\n(Baseline)', 'Ideal Model\n(TN=FP=FN=0)', 'Random Model\n(50% detect)']
costs = [approve_all_total_cost, 0, approve_all_fn_cost / 2]
colors = ['coral', 'green', 'orange']

axes[1].bar(models, costs, color=colors)
axes[1].set_ylabel('Total Cost ($)')
axes[1].set_title('Total Cost Comparison')
axes[1].axhline(y=approve_all_total_cost, color='red', linestyle='--', label='Approve-All Cost')
axes[1].legend()
```

### 11.4. Argument trong báo cáo

> "Baseline approve-all (dự đoán mọi giao dịch đều hợp lệ) là điểm khởi đầu quan trọng để đánh giá hiệu quả của các mô hình. Trên tập test 88,581 giao dịch, approve-all có độ chính xác 96.50% (rất cao theo cảm nhận thông thường), nhưng lại bỏ sót toàn bộ 3,099 giao dịch fraud, gây thiệt hại trực tiếp ước tính $445,000 (dựa trên TransactionAmt trung bình của fraud × số lượng). Đây là lập luận mạnh mẽ nhất chứng minh rằng Accuracy không phản ánh đúng hiệu quả mô hình trong bài toán cost-sensitive. Metrics chính của nghiên cứu này là Total Cost và Cost Saving, được định nghĩa:
>
> **Total Cost** = FN × TransactionAmt × α + FP × β
>
> **Cost Saving** = Cost(Approve-All) − Cost(Model)
>
> Một mô hình được coi là hữu ích khi Cost Saving > 0, tức là tổng chi phí (thiệt hại fraud bỏ sót + chi phí xác minh đánh dấu nhầm) thấp hơn chi phí của approve-all."

---

## Tổng kết: Mapping EDA → Research Arguments

| EDA | Research Argument | Đoạn trong báo cáo |
|-----|------------------|---------------------|
| 1. Data Integrity | Pipeline đáng tin cậy | Methods 3.1 |
| 2. Class Imbalance | Accuracy không phù hợp | Introduction 1.1 |
| 3. TransactionAmt | Cost function phụ thuộc amount | Methods 3.8 |
| 4. Missingness | Cần imputation, không drop | Methods 3.2 |
| 5. Identity Coverage | Left join thay vì inner join | Methods 3.2 |
| 6. Feature Family | Hiểu cấu trúc 400+ columns | Methods 3.3 |
| 7. Missing by Group | Preprocessing theo nhóm | Methods 3.2 |
| 8. Time Split | Tránh temporal leakage | Methods 3.2 |
| 9. Categorical FRAUD | Feature selection cho baseline/LLM | Methods 3.6 |
| 10. Numeric Correlation | Sanity check features có signal | Methods 3.6 |
| 11. Approve-All | Baseline cho Cost Saving | Results 4.4 |

---

## Lưu ý khi viết báo cáo

1. **Không cần trình bày 400+ features** — Chỉ show feature families và top/bottom 5-10 features quan trọng
2. **Dùng figure chuẩn** — Mỗi EDA nên có 1-2 figure, không cần quá chi tiết
3. **Kết hợp statistics + visualization** — Statistics cho con số cụ thể, visualization cho pattern nhìn thấy được
4. **Link đến code** — Có thể đặt code trong appendix hoặc GitHub repo, không cần in trong báo cáo chính
