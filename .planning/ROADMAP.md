# Roadmap: LLM-Assisted Cost-Sensitive RL Fraud Detection

## Overview / Tổng quan

| Phase | Name / Tên | Goal / Mục tiêu | Requirements |
|---:|---|---|---|
| 1 | Data Pipeline and EDA | VI: Join, làm sạch, chia split, và hiểu IEEE-CIS mà không gây leakage. EN: Join, clean, split, and understand IEEE-CIS without leakage. | DATA-01..05, PREP-01..04 |
| 2 | ML Baselines and Cost Metrics | VI: Xây dựng baseline ML và đánh giá threshold cost-sensitive. EN: Build ML baselines and cost-sensitive threshold evaluation. | BASE-01..05 |
| 2.1 | Kaggle-inspired XGB Magic Baseline (INSERTED) | VI: Tăng sức nặng supervised baseline bằng XGBoost magic-style leakage-safe trước khi qua LLM/RL. EN: Strengthen the supervised baseline with leakage-safe XGBoost magic-style features before LLM/RL. | Strengthens BASE-03..05 |
| 3 | LLM Representation and RL | VI: Xây dựng table-to-text, MiniLM embeddings, và contextual bandit ablations. EN: Build table-to-text, MiniLM embeddings, and contextual bandit ablations. | LLM-01..04, RL-01..04 |
| 4 | Evaluation, Error Analysis, Report | VI: Tạo final metrics, charts, error analysis, và báo cáo. EN: Produce final metrics, charts, error analysis, and report. | EVAL-01..06 |

## Phase 1: Data Pipeline and EDA

**Status / Trạng thái:** Complete / Hoàn tất — verified 2026-05-25.

**Goal / Mục tiêu:**  
**VI:** Tạo nền tảng dữ liệu đúng và chống leakage.  
**EN:** Build a correct data foundation and prevent leakage.

**Success criteria / Tiêu chí thành công:**

1. **VI:** Load được `train_transaction.csv` và `train_identity.csv`. **EN:** Load `train_transaction.csv` and `train_identity.csv`.
2. **VI:** Left join đúng theo `TransactionID`, không làm mất giao dịch có label. **EN:** Correctly left join by `TransactionID` without losing labeled transactions.
3. **VI:** Fraud rate được tính trực tiếp từ dữ liệu. **EN:** Fraud rate is calculated directly from the data.
4. **VI:** Split train/validation/test theo `TransactionDT` 70/15/15. **EN:** Split train/validation/test by `TransactionDT` using 70/15/15.
5. **VI:** Preprocessing xử lý missing, categorical, rare category và loại `TransactionID`/`isFraud` khỏi feature. **EN:** Preprocessing handles missing values, categorical values, rare categories, and excludes `TransactionID`/`isFraud` from features.

**Suggested files / File gợi ý:**

- Primary / Chính: `notebooks/01_data_check.ipynb`
- Optional script mirrors / Script phụ trợ: `src/data/make_dataset.py`, `src/features/preprocess.py`

## Phase 2: ML Baselines and Cost Metrics

**Status / Trạng thái:** Complete / Hoàn tất — smoke verified 2026-05-26.

**Goal / Mục tiêu:**  
**VI:** Tạo điểm so sánh mạnh và cost-sensitive.  
**EN:** Build strong and cost-sensitive comparison points.

**Success criteria / Tiêu chí thành công:**

1. **VI:** Có approve-all baseline. **EN:** Include an approve-all baseline.
2. **VI:** Có Logistic Regression với class weight. **EN:** Include Logistic Regression with class weight.
3. **VI:** Có Random Forest hoặc LightGBM/XGBoost với imbalance handling. **EN:** Include Random Forest or LightGBM/XGBoost with imbalance handling.
4. **VI:** Có hàm tính PR-AUC, Recall/Precision/F1 fraud, FN Cost, FP Cost, Total Cost, Cost Saving. **EN:** Include functions for PR-AUC, fraud Recall/Precision/F1, FN Cost, FP Cost, Total Cost, and Cost Saving.
5. **VI:** Có threshold tuning trên validation cho từng cấu hình `alpha`/`beta`. **EN:** Include validation threshold tuning for each `alpha`/`beta` setting.

**Suggested files / File gợi ý:**

- Primary / Chính: `notebooks/02_baselines_cost_metrics.ipynb`
- Optional script mirrors / Script phụ trợ: none for current execution; Phase 2 is notebook-only.
- `results/baseline_metrics.csv`
- `results/threshold_tuning.csv`

## Phase 2.1: Kaggle-inspired XGB Magic Baseline (INSERTED)

**Status / Trạng thái:** Complete / Hoàn tất — smoke verified 2026-05-26.

**Goal / Mục tiêu:**  
**VI:** Bổ sung một supervised baseline mạnh hơn, lấy cảm hứng từ cdeotte XGB Magic và các Kaggle IEEE-CIS solution, nhưng giữ leakage-safe và đánh giá theo cost-sensitive metrics của đồ án.  
**EN:** Add a stronger supervised baseline inspired by cdeotte XGB Magic and Kaggle IEEE-CIS solutions, while keeping leakage safety and evaluating with this project's cost-sensitive metrics.

**Success criteria / Tiêu chí thành công:**

1. **VI:** Có XGBoost hoặc LightGBM magic-style baseline chạy trên cùng split `TransactionDT` 70/15/15. **EN:** Include an XGBoost or LightGBM magic-style baseline on the same `TransactionDT` 70/15/15 split.
2. **VI:** Feature engineering chỉ dùng train-fitted statistics rồi map sang validation/test. **EN:** Feature engineering uses train-fitted statistics only, then maps them to validation/test.
3. **VI:** Có count/frequency encoding và aggregation đơn giản như `card1`, `card2`, `addr1`, email domain, `card1_addr1`, và `TransactionAmt` group stats. **EN:** Include count/frequency encoding and simple aggregations for `card1`, `card2`, `addr1`, email domains, `card1_addr1`, and `TransactionAmt` group stats.
4. **VI:** Không dùng Kaggle test files, leaderboard artifacts, full 1st-place post-processing, hoặc label leakage. **EN:** Do not use Kaggle test files, leaderboard artifacts, full 1st-place post-processing, or label leakage.
5. **VI:** Đánh giá bằng PR-AUC, ROC-AUC, Recall/Precision/F1 fraud, FN/FP/Total Cost, và Cost Saving; FraudSquad 1st place chỉ là external SOTA reference nếu không reproduce đầy đủ. **EN:** Evaluate with PR-AUC, ROC-AUC, fraud Recall/Precision/F1, FN/FP/Total Cost, and Cost Saving; FraudSquad 1st place remains an external SOTA reference unless fully reproduced.

**Suggested files / File gợi ý:**

- Primary / Chính: update `notebooks/02_baselines_cost_metrics.ipynb`
- Optional generated outputs: `results/baseline_metrics.csv`, `results/threshold_tuning.csv`, `reports/figures/`

## Phase 3: LLM Representation and RL

**Goal / Mục tiêu:**  
**VI:** Kiểm tra LLM/local embedding có giúp contextual bandit hay không.  
**EN:** Test whether LLM/local embeddings help the contextual bandit.

**Success criteria / Tiêu chí thành công:**

1. **VI:** Table-to-text chỉ dùng feature thật và template trung lập. **EN:** Table-to-text uses only real features and a neutral template.
2. **VI:** MiniLM embeddings được tạo local, cache, và nối đúng split. **EN:** MiniLM embeddings are generated locally, cached, and joined to the correct split.
3. **VI:** Contextual bandit without embedding chạy được. **EN:** Contextual bandit without embedding runs successfully.
4. **VI:** Contextual bandit with embedding chạy được. **EN:** Contextual bandit with embedding runs successfully.
5. **VI:** Ablation dùng cùng split, cùng reward, cùng metric. **EN:** Ablation uses the same split, reward, and metrics.

**Suggested files / File gợi ý:**

- Primary / Chính: `notebooks/03_llm_representation_and_rl.ipynb`
- Optional script mirrors / Script phụ trợ: `src/features/text_serialize.py`, `src/features/embed.py`, `src/models/train_bandit.py`
- `results/rl_ablation.csv`

## Phase 4: Evaluation, Error Analysis, Report

**Goal / Mục tiêu:**  
**VI:** Biến pipeline thành kết quả báo cáo cuối kỳ rõ ràng và bảo vệ được.  
**EN:** Turn the pipeline into clear and defensible final-report results.

**Success criteria / Tiêu chí thành công:**

1. **VI:** Có bảng kết quả chính cho baseline, cost-sensitive ML, RL no embedding, RL with embedding. **EN:** Include main results tables for baseline, cost-sensitive ML, RL without embedding, and RL with embedding.
2. **VI:** Có sensitivity analysis với Cost-A, Cost-B, Cost-C. **EN:** Include sensitivity analysis with Cost-A, Cost-B, and Cost-C.
3. **VI:** Có biểu đồ class imbalance, amount distribution, PR curve, cost-vs-threshold, confusion matrix, cost bar chart. **EN:** Include class imbalance, amount distribution, PR curve, cost-vs-threshold, confusion matrix, and cost bar charts.
4. **VI:** Có error analysis FN cao, FP cao, baseline bắt/RL bỏ sót và baseline bỏ sót/RL bắt. **EN:** Include error analysis for high-cost FN, high-cost FP, baseline catches/RL misses, and baseline misses/RL catches.
5. **VI:** Báo cáo nêu rõ Accuracy không phù hợp và không claim LLM tốt hơn nếu kết quả không ủng hộ. **EN:** Explain why Accuracy is unsuitable and do not claim the LLM helps unless results support it.

**Suggested files / File gợi ý:**

- `notebooks/04_final_analysis.ipynb`
- `reports/final_report.md`
- `reports/figures/`
- `results/final_metrics.csv`

## Four-week Timeline / Timeline 4 tuần

| Week / Tuần | Focus / Trọng tâm | Exit condition / Điều kiện hoàn tất |
|---|---|---|
| 1 | Data pipeline, EDA, preprocessing | VI: Split và preprocessing chạy lại được. EN: Split and preprocessing are reproducible. |
| 2 | Baselines, cost metrics, threshold tuning | VI: Baseline table có đủ metric. EN: Baseline table has all required metrics. |
| 3 | Table-to-text, embedding, contextual bandit | VI: RL ablation có kết quả đầu tiên. EN: RL ablation has first results. |
| 4 | Final evaluation, charts, report | VI: Báo cáo và hình/bảng hoàn chỉnh. EN: Report, figures, and tables are complete. |

## First 7 Days / 7 ngày đầu

| Day / Ngày | Task / Việc làm | Output / Đầu ra |
|---:|---|---|
| 1 | VI: Setup repo/env, đặt dataset vào `data/raw`, đọc CSV. EN: Set up repo/env, place dataset in `data/raw`, read CSV. | Data check notebook |
| 2 | VI: Join, fraud rate, missing ratio, split. EN: Join, fraud rate, missing ratio, split. | EDA summary |
| 3 | VI: Preprocessing pipeline. EN: Preprocessing pipeline. | Feature matrices |
| 4 | VI: Logistic + RF/LightGBM baseline đầu tiên. EN: First Logistic + RF/LightGBM baseline. | Baseline scores |
| 5 | VI: Cost metrics + threshold tuning. EN: Cost metrics + threshold tuning. | Cost tables |
| 6 | VI: Table-to-text + MiniLM embedding cache. EN: Table-to-text + MiniLM embedding cache. | Embedding files |
| 7 | VI: Contextual bandit without embedding. EN: Contextual bandit without embedding. | RL no-embedding result |
