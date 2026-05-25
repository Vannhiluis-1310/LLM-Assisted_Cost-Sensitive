# Kế hoạch dự án cuối kỳ / Final Project Plan

## 1. Tên đề tài / Topic Title

| Ngôn ngữ / Language | Tên đề tài / Title |
|---|---|
| Tiếng Việt / Vietnamese | Ứng dụng LLM hỗ trợ học tăng cường nhạy cảm chi phí trong phát hiện gian lận giao dịch thương mại điện tử trên dữ liệu mất cân bằng |
| Tiếng Anh / English | LLM-Assisted Cost-Sensitive Reinforcement Learning for E-commerce Transaction Fraud Detection on Imbalanced Data |

## 2. Problem Statement / Phát biểu bài toán

**VI:** Phát hiện gian lận giao dịch thương mại điện tử là bài toán binary classification trên dữ liệu mất cân bằng, trong đó số giao dịch gian lận thường rất ít so với giao dịch hợp lệ. Nếu tối ưu Accuracy, mô hình có thể đạt điểm cao bằng cách dự đoán hầu hết giao dịch là hợp lệ, nhưng bỏ sót fraud có chi phí kinh tế lớn.

**EN:** E-commerce transaction fraud detection is a binary classification problem on imbalanced data, where fraudulent transactions are usually much fewer than legitimate ones. If Accuracy is optimized, a model can score highly by predicting most transactions as legitimate, while still missing costly fraud cases.

**VI:** Dự án xây dựng pipeline thực nghiệm trên IEEE-CIS Fraud Detection, tập trung vào xử lý mất cân bằng và tối ưu quyết định approve/block theo chi phí. False negative bị phạt nặng hơn false positive, và chi phí phụ thuộc `TransactionAmt`.

**EN:** The project builds an experimental pipeline on IEEE-CIS Fraud Detection, focusing on imbalance handling and cost-sensitive approve/block decisions. False negatives are penalized more heavily than false positives, and the cost depends on `TransactionAmt`.

**VI:** LLM không được fine-tune. Vai trò của LLM/local embedding là tạo biểu diễn bổ sung từ mô tả table-to-text trung lập của giao dịch. RL được giới hạn ở contextual bandit hoặc DQN đơn giản với action space `0 = approve`, `1 = flag/block`.

**EN:** The LLM is not fine-tuned. The role of LLM/local embeddings is to create additional representations from neutral table-to-text transaction descriptions. RL is limited to a contextual bandit or simple DQN with action space `0 = approve`, `1 = flag/block`.

## 3. Research Questions / Câu hỏi nghiên cứu

| ID | VI | EN |
|---|---|---|
| RQ1 | Baseline ML có xử lý imbalance có giảm Total Cost tốt hơn approve-all không? | Does an imbalance-aware ML baseline reduce Total Cost compared with approve-all? |
| RQ2 | Cost-sensitive decision threshold có phù hợp hơn threshold mặc định 0.5 không? | Is a cost-sensitive decision threshold better than the default 0.5 threshold? |
| RQ3 | Contextual bandit có tối ưu chi phí tốt hơn baseline ML không? | Can a contextual bandit optimize cost better than ML baselines? |
| RQ4 | LLM/local embedding từ table-to-text có cải thiện RL so với RL không embedding không? | Do LLM/local embeddings from table-to-text improve RL compared with RL without embeddings? |
| RQ5 | Kết quả có nhạy với `alpha`, `beta` không? | Are results sensitive to `alpha` and `beta`? |

## 4. Contribution dự kiến / Expected Contributions

### MVP

| VI | EN |
|---|---|
| Pipeline join, preprocess, split theo thời gian và đánh giá trên IEEE-CIS. | Join, preprocessing, time-based split, and evaluation pipeline on IEEE-CIS. |
| Bộ metric cost-sensitive: PR-AUC, Recall/Precision/F1 fraud, FN Cost, FP Cost, Total Cost, Cost Saving. | Cost-sensitive metric set: PR-AUC, fraud Recall/Precision/F1, FN Cost, FP Cost, Total Cost, Cost Saving. |
| Baseline ML: Logistic Regression, Random Forest hoặc LightGBM/XGBoost, và ít nhất một imbalance baseline. | ML baselines: Logistic Regression, Random Forest or LightGBM/XGBoost, and at least one imbalance baseline. |
| Table-to-text trung lập cho một phần transaction records. | Neutral table-to-text serialization for a subset of transaction records. |
| Local embedding bằng `sentence-transformers/all-MiniLM-L6-v2`. | Local embeddings using `sentence-transformers/all-MiniLM-L6-v2`. |
| Contextual bandit cost-sensitive với action approve/block. | Cost-sensitive contextual bandit with approve/block actions. |
| Ablation RL without embedding vs RL with embedding. | Ablation: RL without embedding vs RL with embedding. |

### Stretch Goal

| VI | EN |
|---|---|
| Thêm LightGBM/XGBoost nếu môi trường cài đặt ổn định. | Add LightGBM/XGBoost if the environment supports it. |
| Giảm chiều embedding bằng PCA 32/64/128 và so sánh. | Compare PCA-reduced embeddings at 32/64/128 dimensions. |
| Case study nhỏ 100-300 giao dịch có risk explanation trung lập, không dùng API LLM trên toàn bộ dataset. | Small 100-300 transaction case study with neutral explanations; no API LLM over the full dataset. |
| Thêm SHAP/feature importance cho baseline ML nếu còn thời gian. | Add SHAP/feature importance for ML baselines if time remains. |

## 5. Dataset Description / Mô tả dataset

| Hạng mục / Item | VI | EN |
|---|---|---|
| Dataset chính / Main dataset | IEEE-CIS Fraud Detection | IEEE-CIS Fraud Detection |
| File dùng / Files used | `train_transaction.csv`, `train_identity.csv` | `train_transaction.csv`, `train_identity.csv` |
| File không dùng làm test chính / Files not used as main test | `test_transaction.csv`, `test_identity.csv` vì không có public label `isFraud`. | `test_transaction.csv`, `test_identity.csv` because public `isFraud` labels are unavailable. |
| Key join | `TransactionID` | `TransactionID` |
| Cách join / Join method | Left join từ transaction sang identity để giữ tất cả giao dịch có label. | Left join from transaction to identity to preserve all labeled transactions. |
| Label | `isFraud`: `1 = fraud`, `0 = legitimate` | `isFraud`: `1 = fraud`, `0 = legitimate` |
| ID không đưa vào model / ID excluded from model | `TransactionID` | `TransactionID` |

### Cột quan trọng / Important Columns

| Nhóm / Group | Columns |
|---|---|
| Label/amount/time | `isFraud`, `TransactionAmt`, `TransactionDT` |
| Sản phẩm/thanh toán / Product/payment | `ProductCD`, `card1`-`card6`, `addr1`, `addr2`, `dist1`, `dist2` |
| Email domain | `P_emaildomain`, `R_emaildomain` |
| Count/timedelta/match | `C1`-`C14`, `D1`-`D15`, `M1`-`M9` |
| Biến ẩn danh / Anonymous variables | `V1`-`V339` |
| Identity | `id_01`-`id_38`, `DeviceType`, `DeviceInfo` |
| Browser/OS tham khảo / Browser/OS references | `id_31`, `id_30`; use as raw text values only, without over-interpreting them. |

### Missing Values / Giá trị thiếu

| Loại feature / Feature type | VI | EN |
|---|---|---|
| Numeric | Median imputation fit trên train, thêm missing indicator cho cột missing nhiều. | Median imputation fit on train; add missing indicators for high-missing columns. |
| Categorical | Điền `"missing"`, gom rare categories vào `"rare"` nếu tần suất thấp. | Fill with `"missing"`; group rare categories as `"rare"` when frequency is low. |
| Identity missing | Giữ là `"missing"`/indicator riêng, không xóa dòng. | Keep as `"missing"`/separate indicator; do not drop rows. |
| Amount | Nếu missing thì median, nhưng cần kiểm tra vì `TransactionAmt` là cost column quan trọng. | If missing, use median; verify carefully because `TransactionAmt` is cost-critical. |
| Text serialization | Missing hiển thị là `"unknown"`. | Missing values appear as `"unknown"`. |

### Train/Validation/Test Split

**VI:** Main split phải theo thời gian để giảm leakage:

**EN:** The main split must be time-based to reduce leakage:

1. Sort theo / Sort by `TransactionDT`.
2. Train = 70% giao dịch đầu / first 70% of transactions.
3. Validation = 15% tiếp theo / next 15%.
4. Test = 15% cuối / final 15%.
5. **VI:** Imputer, scaler, encoder, PCA, threshold tuning chỉ fit trên đúng split cho phép, không fit trên test.  
   **EN:** Imputer, scaler, encoder, PCA, and threshold tuning are fit only on allowed splits, never on test.

**VI:** Stratified split chỉ dùng như sanity check phụ.  
**EN:** Stratified split is only a secondary sanity check.

## 6. Pipeline chi tiết / Detailed Pipeline

| Step | Input | VI xử lý | EN processing | Output |
|---|---|---|---|---|
| 1. Load data | `train_transaction`, `train_identity` | Đọc CSV, giảm memory, kiểm tra shape, fraud rate. | Read CSV, reduce memory, check shape, fraud rate. | Raw dataframe |
| 2. Join | Raw tables | Left join bằng `TransactionID`. | Left join by `TransactionID`. | Labeled transaction table |
| 3. EDA | Joined table | Label distribution, amount distribution, missing ratio, fraud rate. | Label distribution, amount distribution, missing ratio, fraud rate. | EDA tables/plots |
| 4. Split | Joined table | Time-based 70/15/15. | Time-based 70/15/15. | Train/val/test |
| 5. Preprocess ML | Split data | Impute, encode, scale, rare-category handling. | Impute, encode, scale, rare-category handling. | `X_train`, `X_val`, `X_test` |
| 6. Baseline ML | Preprocessed features | Logistic, RF/LightGBM, class weight. | Logistic, RF/LightGBM, class weight. | Scores/probabilities |
| 7. Threshold tuning | Validation scores | Chọn threshold minimize Total Cost theo alpha/beta. | Choose threshold minimizing Total Cost per alpha/beta. | Optimal threshold |
| 8. Table-to-text | Selected records | Template trung lập từ feature thật. | Neutral template from real features. | Text records |
| 9. Embedding | Text records | MiniLM encode, cache `.npy`, optional PCA. | MiniLM encode, cache `.npy`, optional PCA. | Embedding vectors |
| 10. RL no embedding | Tabular state | Train contextual bandit. | Train contextual bandit. | Policy no embedding |
| 11. RL with embedding | Tabular + embedding | Train contextual bandit. | Train contextual bandit. | Policy with embedding |
| 12. Evaluation | Test split | Metrics và cost. | Metrics and cost. | Results, charts, error analysis |

## 7. Table-to-text

Template MVP:

```text
A transaction with amount {TransactionAmt}, product code {ProductCD}, card network {card4}, card type {card6}, purchaser email domain {P_emaildomain}, recipient email domain {R_emaildomain}, billing address code {addr1}, device type {DeviceType}, device info {DeviceInfo}, browser {id_31}, operating system {id_30}.
```

| Quy tắc / Rule | VI | EN |
|---|---|---|
| No leakage | Không đưa `isFraud`, `TransactionID`, hoặc kết quả model vào text. | Do not include `isFraud`, `TransactionID`, or model outputs in text. |
| Neutral wording | Không viết "high risk", "suspicious", "fraud-like" trong input. | Do not write "high risk", "suspicious", or "fraud-like" in input. |
| ProductCD | Không diễn giải `ProductCD` thành loại hàng cụ thể. | Do not interpret `ProductCD` as a concrete product category. |
| Missing | Missing value ghi là `"unknown"`. | Missing values are written as `"unknown"`. |
| Sample size | MVP encode 30,000-80,000 records tùy tài nguyên; báo cáo rõ tập dùng. | MVP encodes 30,000-80,000 records depending on resources; report the exact subset used. |

## 8. Embedding

| Hạng mục / Item | Quyết định / Decision |
|---|---|
| Model | `sentence-transformers/all-MiniLM-L6-v2` |
| Vector size / Kích thước vector | 384 |
| Fine-tune | VI: Không. EN: No. |
| Cache | VI: Lưu embedding theo `TransactionID` và split. EN: Cache embeddings by `TransactionID` and split. |
| Dimensionality reduction / Giảm chiều | VI: PCA 64 chiều fit trên train nếu RL chậm. EN: PCA to 64 dimensions fit on train if RL is slow. |
| API LLM | VI: Không dùng trong MVP. EN: Not used in the MVP. |

## 9. Reward Function / Hàm reward

```text
R = - [ y * (1 - a) * C_FN + (1 - y) * a * C_FP ]
C_FN = TransactionAmt * beta
C_FP = TransactionAmt * alpha
```

| Ký hiệu / Symbol | VI | EN |
|---|---|---|
| `y` | Label thật: `1 = fraud`, `0 = legitimate`. | True label: `1 = fraud`, `0 = legitimate`. |
| `a` | Action: `0 = approve`, `1 = flag/block`. | Action: `0 = approve`, `1 = flag/block`. |
| `C_FN` | Chi phí khi approve nhầm giao dịch fraud. | Cost of wrongly approving a fraudulent transaction. |
| `C_FP` | Chi phí khi block nhầm giao dịch hợp lệ. | Cost of wrongly blocking a legitimate transaction. |
| `beta` | Hệ số thiệt hại fraud, phải lớn hơn `alpha`. | Fraud-loss coefficient, must be larger than `alpha`. |
| `alpha` | Hệ số friction/operation cost khi block nhầm legitimate. | Friction/operation cost coefficient for wrongly blocked legitimate transactions. |

### Sensitivity Analysis

| Setting | `alpha` | `beta` | VI | EN |
|---|---:|---:|---|---|
| Cost-A | 0.02 | 1.0 | FP cost nhỏ, FN mất gần bằng amount. | Small FP cost; FN loss roughly equals amount. |
| Cost-B | 0.05 | 2.0 | Phạt FN mạnh hơn, vẫn kiểm soát FP. | Stronger FN penalty while controlling FP. |
| Cost-C | 0.10 | 5.0 | Rất ưu tiên bắt fraud, chấp nhận block nhầm nhiều hơn. | Strong fraud-catching priority, accepting more false blocks. |

## 10. Thiết kế RL/contextual bandit / RL Design

| Thành phần / Component | MVP |
|---|---|
| Dạng bài toán / Problem form | Offline contextual bandit, one-step decision, no state transition. |
| State without LLM | Preprocessed tabular vector: amount, time-derived, card, email, C/D/M/V/id/device features. |
| State with LLM | Concatenate tabular vector with MiniLM/PCA embedding. |
| Action | `0 = approve`, `1 = flag/block` |
| Reward | Cost-sensitive reward above / Hàm reward cost-sensitive ở trên |
| Policy/model | PyTorch MLP with 2 outputs: `Q(s, approve)`, `Q(s, block)`; sklearn MLP/SGD fallback. |
| Discount | `gamma = 0` because this is a one-step bandit. |
| Exploration | Epsilon-greedy during training, decay from 0.2 to 0.02. |
| Loss | MSE between predicted Q for the action and reward; stable version may train full-information targets for both actions. |

### Training loop MVP

1. **VI:** Lặp qua train samples theo random order. **EN:** Iterate through train samples in random order.
2. **VI:** Lấy state `s`, label `y`, amount `TransactionAmt`. **EN:** Read state `s`, label `y`, and amount `TransactionAmt`.
3. **VI:** Chọn action `a` bằng epsilon-greedy. **EN:** Choose action `a` via epsilon-greedy.
4. **VI:** Tính reward theo `alpha`, `beta`. **EN:** Compute reward using `alpha`, `beta`.
5. **VI:** Cập nhật model để tăng Q của action có reward tốt hơn. **EN:** Update the model toward better-reward actions.
6. **VI:** Theo dõi validation Total Cost mỗi epoch. **EN:** Track validation Total Cost each epoch.
7. **VI:** Early stop theo validation Total Cost, không theo Accuracy. **EN:** Early stop by validation Total Cost, not Accuracy.

### Evaluation loop

1. **VI:** Tắt exploration, chọn `a = argmax_a Q(s,a)`. **EN:** Disable exploration and choose `a = argmax_a Q(s,a)`.
2. **VI:** Score PR-AUC có thể dùng `Q(s, block) - Q(s, approve)` hoặc probability block. **EN:** PR-AUC score can use `Q(s, block) - Q(s, approve)` or block probability.
3. **VI:** Tính TP, FP, FN, TN trên test. **EN:** Compute TP, FP, FN, TN on test.
4. **VI:** Tính fraud metrics và cost theo từng alpha/beta. **EN:** Compute fraud metrics and cost for each alpha/beta setting.

## 11. Baseline và ablation / Baselines and Ablations

| Nhóm / Group | Model | Bắt buộc/MVP |
|---|---|---|
| Naive cost baseline | Approve-all | Required / Bắt buộc |
| Baseline ML 1 | Logistic Regression + class_weight balanced | Required / Bắt buộc |
| Baseline ML 2 | Random Forest class_weight hoặc LightGBM/XGBoost scale_pos_weight | Required / Bắt buộc |
| Cost-sensitive ML | Threshold tuning trên validation để minimize Total Cost | Required / Bắt buộc |
| RL ablation 1 | Contextual bandit without LLM embedding | Required / Bắt buộc |
| RL ablation 2 | Contextual bandit with MiniLM embedding | Required / Bắt buộc |
| Stretch | LightGBM/XGBoost + threshold tuning nếu MVP dùng Random Forest | Stretch |

## 12. Metrics và công thức / Metrics and Formulas

| Metric | Formula / Công thức |
|---|---|
| PR-AUC | Average Precision on fraud score |
| Recall fraud | `TP / (TP + FN)` |
| Precision fraud | `TP / (TP + FP)` |
| F1 fraud | `2 * Precision * Recall / (Precision + Recall)` |
| FN Cost | `sum(TransactionAmt * beta)` for `y=1, a=0` |
| FP Cost | `sum(TransactionAmt * alpha)` for `y=0, a=1` |
| Total Cost | `FN Cost + FP Cost` |
| Cost Saving vs approve-all | `(Cost_approve_all - Cost_model) / Cost_approve_all * 100%` |
| Cost Saving vs ML baseline | `(Cost_ML - Cost_RL) / Cost_ML * 100%` |

**VI:** Accuracy chỉ báo cáo phụ nếu cần, và phải giải thích rằng nó không phù hợp vì dữ liệu mất cân bằng.  
**EN:** Accuracy is only a secondary metric if reported, and the report must explain why it is unsuitable for imbalanced data.

## 13. Bảng kết quả cần có / Required Result Tables

| Table | VI | EN |
|---|---|---|
| Table 1 | Dataset summary: số dòng, số cột, fraud count, legitimate count, fraud rate tính từ train. | Dataset summary: row count, column count, fraud count, legitimate count, fraud rate computed from train. |
| Table 2 | Missing value top columns sau join. | Top missing-value columns after join. |
| Table 3 | Baseline ML metrics trên test. | Baseline ML metrics on test. |
| Table 4 | Threshold tuning: threshold, Recall, Precision, Total Cost. | Threshold tuning: threshold, Recall, Precision, Total Cost. |
| Table 5 | Sensitivity analysis theo alpha/beta. | Sensitivity analysis by alpha/beta. |
| Table 6 | Ablation: RL no embedding vs RL with embedding vs ML baseline mạnh. | Ablation: RL no embedding vs RL with embedding vs strong ML baseline. |
| Table 7 | Error analysis: top false negatives theo cost. | Error analysis: top false negatives by cost. |
| Table 8 | Error analysis: top false positives theo cost. | Error analysis: top false positives by cost. |

## 14. Biểu đồ cần vẽ / Required Charts

1. Class distribution bar chart by `isFraud` / Bar chart phân phối class theo `isFraud`.
2. `TransactionAmt` histogram/log-histogram by fraud vs legitimate / Histogram hoặc log-histogram `TransactionAmt` theo fraud vs legitimate.
3. Top 30 missing-value ratio columns / Missing value ratio top 30 columns.
4. Precision-Recall curve for Logistic, RF/LightGBM, RL / PR curve cho Logistic, RF/LightGBM, RL.
5. Total Cost vs threshold on validation / Total Cost vs threshold trên validation.
6. Confusion matrix for strong baseline and RL / Confusion matrix của baseline mạnh và RL.
7. FN Cost, FP Cost, Total Cost bar chart by model / Bar chart cost theo model.
8. Ablation chart: Cost Saving of RL without embedding and RL with embedding / Chart ablation Cost Saving.
9. Sensitivity chart by alpha/beta / Sensitivity chart theo alpha/beta.

## 15. Error Analysis / Phân tích lỗi

| Nhóm lỗi / Error group | VI | EN |
|---|---|---|
| False negatives cost cao | Lấy top 20 giao dịch `y=1, a=0` theo `TransactionAmt * beta`; xem amount, ProductCD, card4/card6, email domain, DeviceType, score. | Take top 20 transactions with `y=1, a=0` by `TransactionAmt * beta`; inspect amount, ProductCD, card4/card6, email domain, DeviceType, score. |
| False positives cost cao | Lấy top 20 giao dịch `y=0, a=1` theo `TransactionAmt * alpha`; xem có threshold quá gay gắt không. | Take top 20 transactions with `y=0, a=1` by `TransactionAmt * alpha`; check whether threshold is too aggressive. |
| Baseline bỏ sót nhưng RL bắt được | Lọc `y=1`, baseline approve, RL block; phân tích pattern. | Filter `y=1`, baseline approve, RL block; analyze patterns. |
| Baseline bắt được nhưng RL bỏ sót | Lọc `y=1`, baseline block, RL approve; phân tích vì sao RL thua. | Filter `y=1`, baseline block, RL approve; analyze why RL failed. |
| Embedding có ích/hại | So sánh cases RL with embedding đổi action so với RL no embedding. | Compare cases where RL with embedding changes action vs RL no embedding. |

## 16. Timeline 4 tuần / Four-week Timeline

| Tuần / Week | Mục tiêu / Goal | Deliverables |
|---|---|---|
| Tuần 1 / Week 1 | Data pipeline và EDA / Data pipeline and EDA | Joined dataset, split, preprocessing, dataset summary, class imbalance explanation |
| Tuần 2 / Week 2 | Baseline ML và cost metrics / ML baselines and cost metrics | Logistic, RF/LightGBM, class weight, threshold tuning, PR-AUC and cost tables |
| Tuần 3 / Week 3 | Table-to-text, embedding, RL | MiniLM embedding cache, contextual bandit no embedding/with embedding, ablation |
| Tuần 4 / Week 4 | Error analysis và báo cáo / Error analysis and report | Final tables, charts, report, slides/demo notebook, reproducibility checklist |

## 17. Definition of Done

1. **VI:** Có notebook/script tải được IEEE-CIS và join đúng `TransactionID`. **EN:** Notebook/script can load IEEE-CIS and join correctly by `TransactionID`.
2. **VI:** Có fraud rate tính trực tiếp từ dataset, không claim số liệu không kiểm chứng. **EN:** Fraud rate is calculated directly from the dataset; no unverified claims.
3. **VI:** Có split theo `TransactionDT` và không leakage test. **EN:** Split by `TransactionDT` with no test leakage.
4. **VI:** Có Logistic Regression và một baseline mạnh RF/LightGBM/XGBoost. **EN:** Logistic Regression and one strong RF/LightGBM/XGBoost baseline exist.
5. **VI:** Có imbalance handling bằng class weight và/hoặc threshold tuning. **EN:** Imbalance handling via class weight and/or threshold tuning exists.
6. **VI:** Có reward function dùng `TransactionAmt`, `alpha`, `beta`. **EN:** Reward function uses `TransactionAmt`, `alpha`, and `beta`.
7. **VI:** Có contextual bandit hoặc DQN đơn giản với action approve/block. **EN:** Contextual bandit or simple DQN exists with approve/block actions.
8. **VI:** Có table-to-text trung lập, không label leakage. **EN:** Neutral table-to-text exists with no label leakage.
9. **VI:** Có MiniLM/local embedding, không fine-tune LLM. **EN:** MiniLM/local embedding exists; no LLM fine-tuning.
10. **VI:** Có ablation RL without embedding vs RL with embedding. **EN:** Ablation RL without embedding vs RL with embedding exists.
11. **VI:** Có tất cả metrics bắt buộc. **EN:** All required metrics are present.
12. **VI:** Có bảng kết quả và biểu đồ cho báo cáo. **EN:** Result tables and report figures are present.
13. **VI:** Có error analysis 4 nhóm. **EN:** Four-group error analysis is present.
14. **VI:** Báo cáo giải thích vì sao Accuracy không phù hợp. **EN:** Report explains why Accuracy is unsuitable.
15. **VI:** Code chạy lại được bằng README/notebook hướng dẫn. **EN:** Code is reproducible using README/notebook instructions.

## 18. Rủi ro mất điểm và cách xử lý / Grading Risks and Mitigations

| Rủi ro / Risk | Tác động / Impact | Cách xử lý / Mitigation |
|---|---|---|
| Đi lạc sang phishing/IDS/malware | Sai scope đề tài / Wrong scope | Giữ toàn bộ nội dung là ecommerce transaction fraud / Keep everything within e-commerce transaction fraud |
| Tối ưu Accuracy | Mất trọng tâm cost-sensitive / Loses cost-sensitive focus | Chọn Total Cost/Cost Saving và Recall fraud làm metric chính / Use Total Cost, Cost Saving, and fraud Recall as main metrics |
| Dùng API LLM trên toàn dataset | Tốn chi phí, sai guardrail / Costly and violates guardrails | Dùng local MiniLM; API chỉ là case study nhỏ nếu có / Use local MiniLM; API only for small case study if any |
| Table-to-text có label leakage | Kết quả không hợp lệ / Invalid results | Template không chứa label, score, risk word / Template contains no labels, scores, or risk words |
| Diễn giải `ProductCD` quá mức | Claim sai nguồn / Unsupported claim | Giữ nguyên "product code Y" / Keep "product code Y" |
| RL phức tạp quá mức | Không kịp 1 tháng / Not feasible in 1 month | Dùng contextual bandit gamma=0 trước / Use contextual bandit with gamma=0 first |
| Baseline ML quá yếu | Không thuyết phục / Not convincing | Có threshold tuning và thêm LightGBM/XGBoost nếu cài được / Add threshold tuning and LightGBM/XGBoost if available |
| Embedding làm kết quả xấu | Không được claim LLM tốt hơn / Cannot claim LLM improves results | Báo cáo trung thực theo ablation / Report ablation honestly |
| Join identity làm mất dòng | Sai dataset / Incorrect dataset | Left join từ transaction sang identity / Left join transaction to identity |
| Claim fraud rate sai | Mất điểm tin cậy / Loses credibility | Tính fraud rate trực tiếp sau load data / Compute fraud rate directly after loading |

## 19. Future Work chỉ đưa vào báo cáo / Future Work Only

1. Concept drift theo thời gian / Temporal concept drift.
2. Federated learning.
3. PPO/A2C hoặc RL phức tạp hơn / PPO/A2C or more complex RL.
4. RAG.
5. Multi-modal learning.
6. Multi-agent synthetic data.
7. API LLM scale lớn / Large-scale API LLM.
8. Fine-tune LLM.
9. Production backend/frontend.

## 20. Kế hoạch 7 ngày đầu / First 7 Days

| Ngày / Day | Việc phải xong / Task | File/notebook đầu ra / Output |
|---|---|---|
| Ngày 1 / Day 1 | Tạo env, cấu trúc repo, đặt dataset vào `data/raw`, đọc 2 CSV train, in shape/cột / Create env, repo structure, place dataset in `data/raw`, read two train CSVs, print shapes/columns | `notebooks/01_data_check.ipynb` |
| Ngày 2 / Day 2 | Left join, tính fraud rate, missing ratio, split theo `TransactionDT` / Left join, fraud rate, missing ratio, split by `TransactionDT` | `src/data/make_dataset.py`, EDA figures |
| Ngày 3 / Day 3 | Xây preprocessing pipeline numeric/categorical, lưu split và transformer / Build numeric/categorical preprocessing, save splits and transformer | `src/features/preprocess.py` |
| Ngày 4 / Day 4 | Train Logistic Regression và RF/LightGBM baseline đầu tiên / Train first Logistic Regression and RF/LightGBM baseline | `src/models/train_baselines.py` |
| Ngày 5 / Day 5 | Viết cost metrics, threshold tuning, approve-all baseline / Write cost metrics, threshold tuning, approve-all baseline | `src/evaluation/metrics.py`, `results/baseline_cost.csv` |
| Ngày 6 / Day 6 | Viết table-to-text và encode MiniLM sample, cache embedding / Write table-to-text and encode MiniLM sample, cache embeddings | `src/features/text_serialize.py`, `src/features/embed.py` |
| Ngày 7 / Day 7 | Implement contextual bandit no embedding, evaluation loop đầu tiên / Implement contextual bandit without embedding and first evaluation loop | `src/models/train_bandit.py`, `results/rl_no_embedding.csv` |

## Quyết định kỹ thuật khuyến nghị cuối cùng / Final Recommended Technical Decisions

| Hạng mục / Item | Quyết định / Decision |
|---|---|
| Dataset | IEEE-CIS Fraud Detection; use `train_transaction.csv` + `train_identity.csv`, left join by `TransactionID`. |
| Baseline | Logistic Regression + class_weight; LightGBM/XGBoost + scale_pos_weight if installable; Random Forest class_weight fallback. |
| Embedding model | `sentence-transformers/all-MiniLM-L6-v2`, local, no fine-tuning. |
| RL algorithm | One-step contextual bandit, PyTorch MLP with 2 Q-values, `gamma=0`; DQN only as stretch. |
| Main metrics / Metrics chính | PR-AUC, fraud Recall, fraud Precision, fraud F1, FN Cost, FP Cost, Total Cost, Cost Saving. |
