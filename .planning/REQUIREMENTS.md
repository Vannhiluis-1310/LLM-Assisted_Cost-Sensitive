# Requirements: LLM-Assisted Cost-Sensitive RL Fraud Detection

**Defined:** 2026-05-24  
**Core Value / Giá trị cốt lõi:** Build a reproducible experimental pipeline that compares ML baselines with cost-sensitive RL using fraud metrics and economic cost metrics. / Xây dựng được pipeline thực nghiệm chạy lại được, so sánh được baseline ML với RL cost-sensitive bằng metrics fraud và chi phí kinh tế.

## v1 Requirements

### Data / Dữ liệu

- [x] **DATA-01**: **VI:** Dự án sử dụng IEEE-CIS Fraud Detection làm dataset chính. **EN:** The project uses IEEE-CIS Fraud Detection as the main dataset.
- [x] **DATA-02**: **VI:** Pipeline đọc `train_transaction.csv` và `train_identity.csv`. **EN:** The pipeline reads `train_transaction.csv` and `train_identity.csv`.
- [x] **DATA-03**: **VI:** Pipeline left join hai bảng bằng `TransactionID` và giữ tất cả giao dịch có label. **EN:** The pipeline left joins the two tables by `TransactionID` and preserves all labeled transactions.
- [x] **DATA-04**: **VI:** Pipeline tính fraud rate trực tiếp từ dataset sau khi load. **EN:** The pipeline calculates the fraud rate directly from the loaded dataset.
- [x] **DATA-05**: **VI:** Pipeline chia train/validation/test theo thứ tự `TransactionDT`. **EN:** The pipeline creates train/validation/test splits ordered by `TransactionDT`.

### Preprocessing / Tiền xử lý

- [x] **PREP-01**: **VI:** Pipeline xử lý missing numeric bằng imputation fit trên train. **EN:** The pipeline handles missing numeric values using imputation fit on train only.
- [x] **PREP-02**: **VI:** Pipeline xử lý categorical bằng `"missing"` và rare-category handling. **EN:** The pipeline handles categorical values using `"missing"` and rare-category handling.
- [x] **PREP-03**: **VI:** Pipeline không đưa `TransactionID` và `isFraud` vào feature model. **EN:** The pipeline excludes `TransactionID` and `isFraud` from model features.
- [x] **PREP-04**: **VI:** Pipeline tạo feature time-derived từ `TransactionDT` mà không diễn giải thành ngày thật. **EN:** The pipeline creates time-derived features from `TransactionDT` without interpreting it as a real calendar date.

### Baselines / Mô hình baseline

- [ ] **BASE-01**: **VI:** Có approve-all baseline để tính cost reference. **EN:** Include an approve-all baseline as the cost reference.
- [ ] **BASE-02**: **VI:** Có Logistic Regression baseline. **EN:** Include a Logistic Regression baseline.
- [ ] **BASE-03**: **VI:** Có Random Forest hoặc LightGBM/XGBoost baseline. **EN:** Include a Random Forest or LightGBM/XGBoost baseline.
- [ ] **BASE-04**: **VI:** Có imbalance handling bằng class weight, threshold tuning, hoặc undersampling. **EN:** Include imbalance handling via class weight, threshold tuning, or undersampling.
- [ ] **BASE-05**: **VI:** Có threshold tuning trên validation để minimize Total Cost. **EN:** Include validation-based threshold tuning to minimize Total Cost.

### LLM Representation / Biểu diễn LLM

- [ ] **LLM-01**: **VI:** Có table-to-text serialization trung lập từ feature thật của IEEE-CIS. **EN:** Include neutral table-to-text serialization from real IEEE-CIS features.
- [ ] **LLM-02**: **VI:** Text input không chứa label, model score, risk wording, hoặc diễn giải `ProductCD`. **EN:** Text input must not contain labels, model scores, risk wording, or interpretation of `ProductCD`.
- [ ] **LLM-03**: **VI:** Có local embedding bằng `sentence-transformers/all-MiniLM-L6-v2`. **EN:** Include local embeddings using `sentence-transformers/all-MiniLM-L6-v2`.
- [ ] **LLM-04**: **VI:** Embedding được cache và nối với đúng split, không fit PCA trên test. **EN:** Embeddings are cached and joined to the correct split; PCA is not fit on test.

### Reinforcement Learning / Học tăng cường

- [ ] **RL-01**: **VI:** Có contextual bandit hoặc DQN đơn giản với action `0 = approve`, `1 = flag/block`. **EN:** Include a contextual bandit or simple DQN with actions `0 = approve`, `1 = flag/block`.
- [ ] **RL-02**: **VI:** Reward dùng `TransactionAmt`, `alpha`, `beta`, trong đó FN bị phạt nặng hơn FP. **EN:** Reward uses `TransactionAmt`, `alpha`, and `beta`, with FN penalized more heavily than FP.
- [ ] **RL-03**: **VI:** Có training loop và evaluation loop riêng, evaluation không exploration. **EN:** Include separate training and evaluation loops; evaluation uses no exploration.
- [ ] **RL-04**: **VI:** Có RL without embedding và RL with embedding để ablation. **EN:** Include RL without embedding and RL with embedding for ablation.

### Evaluation / Đánh giá

- [ ] **EVAL-01**: **VI:** Báo cáo PR-AUC, Recall fraud, Precision fraud, F1 fraud. **EN:** Report PR-AUC, fraud recall, fraud precision, and fraud F1.
- [ ] **EVAL-02**: **VI:** Báo cáo FN Cost, FP Cost, Total Cost. **EN:** Report FN Cost, FP Cost, and Total Cost.
- [ ] **EVAL-03**: **VI:** Báo cáo Cost Saving so với approve-all và baseline ML. **EN:** Report Cost Saving against approve-all and ML baselines.
- [ ] **EVAL-04**: **VI:** Có sensitivity analysis ít nhất 3 cấu hình `alpha`/`beta`. **EN:** Include sensitivity analysis with at least three `alpha`/`beta` settings.
- [ ] **EVAL-05**: **VI:** Có error analysis cho FN cost cao, FP cost cao, baseline-vs-RL disagreement. **EN:** Include error analysis for high-cost FN, high-cost FP, and baseline-vs-RL disagreements.
- [ ] **EVAL-06**: **VI:** Báo cáo giải thích vì sao Accuracy không phù hợp. **EN:** Explain why Accuracy is unsuitable.

## v2 Requirements / Yêu cầu v2

### Optional Enhancements / Nâng cấp tùy chọn

- **OPT-01**: **VI:** Thêm SHAP/feature importance nếu còn thời gian. **EN:** Add SHAP/feature importance if time remains.
- **OPT-02**: **VI:** Thêm DQN đơn giản nếu contextual bandit đã hoàn thành. **EN:** Add a simple DQN if the contextual bandit is complete.
- **OPT-03**: **VI:** Thêm case study 100-300 giao dịch có explanation trung lập. **EN:** Add a 100-300 transaction case study with neutral explanations.
- **OPT-04**: **VI:** Thêm LightGBM/XGBoost nếu MVP ban đầu dùng Random Forest. **EN:** Add LightGBM/XGBoost if the initial MVP uses Random Forest.

## Ngoài phạm vi / Out of Scope

| Feature | Reason / Lý do |
|---------|----------------|
| Fine-tune LLM | VI: Vượt scope, không cần cho MVP. EN: Beyond scope and unnecessary for the MVP. |
| PPO/A2C | VI: Quá phức tạp nếu contextual bandit chưa xong. EN: Too complex before the contextual bandit is complete. |
| API LLM trên toàn bộ dataset | VI: Tốn chi phí và sai guardrail. EN: Costly and outside the guardrails. |
| Synthetic data multi-agent | VI: Không thuộc phạm vi. EN: Outside the project scope. |
| Concept drift là phần chính | VI: Chỉ để Future Work. EN: Future Work only. |
| Phishing/IDS/malware/fake review | VI: Sai miền bài toán. EN: Wrong problem domain. |
| European Credit Card Fraud là dataset chính | VI: Feature PCA ẩn danh làm LLM khó phát huy. EN: Anonymized PCA features weaken the LLM role. |
| Backend/frontend phức tạp | VI: Không cần cho thực nghiệm 1 tháng. EN: Not needed for a one-month experiment. |
| Federated Learning, RAG, multi-modal | VI: Mở rộng ngoài scope. EN: Out-of-scope extensions. |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| DATA-01 | Phase 1 | Complete |
| DATA-02 | Phase 1 | Complete |
| DATA-03 | Phase 1 | Complete |
| DATA-04 | Phase 1 | Complete |
| DATA-05 | Phase 1 | Complete |
| PREP-01 | Phase 1 | Complete |
| PREP-02 | Phase 1 | Complete |
| PREP-03 | Phase 1 | Complete |
| PREP-04 | Phase 1 | Complete |
| BASE-01 | Phase 2 | Pending |
| BASE-02 | Phase 2 | Pending |
| BASE-03 | Phase 2 | Pending |
| BASE-04 | Phase 2 | Pending |
| BASE-05 | Phase 2 | Pending |
| LLM-01 | Phase 3 | Pending |
| LLM-02 | Phase 3 | Pending |
| LLM-03 | Phase 3 | Pending |
| LLM-04 | Phase 3 | Pending |
| RL-01 | Phase 3 | Pending |
| RL-02 | Phase 3 | Pending |
| RL-03 | Phase 3 | Pending |
| RL-04 | Phase 3 | Pending |
| EVAL-01 | Phase 4 | Pending |
| EVAL-02 | Phase 4 | Pending |
| EVAL-03 | Phase 4 | Pending |
| EVAL-04 | Phase 4 | Pending |
| EVAL-05 | Phase 4 | Pending |
| EVAL-06 | Phase 4 | Pending |

**Coverage / Độ phủ:**
- v1 requirements: 28 total
- Mapped to phases: 28
- Unmapped: 0

---
*Requirements defined: 2026-05-24*
*Last updated: 2026-05-25 after Phase 1 verification*
