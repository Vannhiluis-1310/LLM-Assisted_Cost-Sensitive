---
phase: 2
plan: 1
status: complete
created: 2026-05-26
requirements_addressed: [BASE-01, BASE-02, BASE-03, BASE-04, BASE-05]
primary_artifact: notebooks/02_baselines_cost_metrics.ipynb
---

# Phase 2 Plan: ML Baselines and Cost Metrics

## Objective

Create a notebook-only Phase 2 implementation that trains baseline ML models and evaluates them with fraud-focused and cost-sensitive metrics on the fixed IEEE-CIS `TransactionDT` 70/15/15 split.

## Scope

- Create only one main code artifact: `notebooks/02_baselines_cost_metrics.ipynb`.
- Load `data/raw/train_transaction.csv` and `data/raw/train_identity.csv`.
- Left join by `TransactionID` and recreate the Phase 1 time split if processed split files are absent.
- Train and evaluate:
  - approve-all baseline,
  - Logistic Regression with imbalance handling,
  - one stronger baseline: LightGBM/XGBoost if available, otherwise Random Forest fallback.
- Tune decision thresholds on validation to minimize Total Cost.
- Evaluate final selected thresholds on test only.
- Save notebook-generated outputs under `results/` and `reports/figures/` when the notebook is run.

## Out of Scope

- No LLM embedding.
- No table-to-text serialization.
- No local/API LLM calls.
- No contextual bandit, DQN, or RL ablation.
- No `src/` script mirror.
- No large default `X_train.joblib` artifact.
- No Accuracy optimization as the main objective.

## Notebook Sections

1. **Setup and configuration**
   - Detect local/Colab project root.
   - Define `RAW_DIR`, `RESULTS_DIR`, `FIGURES_DIR`.
   - Define `SAMPLE_ROWS` for smoke testing and `RUN_FULL_BASELINES` for full run control.
   - Define cost configs:
     - `Cost-A`: `alpha=0.05`, `beta=1.0`
     - `Cost-B`: `alpha=0.10`, `beta=2.0`
     - `Cost-C`: `alpha=0.20`, `beta=5.0`

2. **Load, join, and split**
   - Read `train_transaction.csv`.
   - Read `train_identity.csv`.
   - Left join on `TransactionID`.
   - Assert joined rows equal transaction rows.
   - Sort by `TransactionDT`.
   - Split 70/15/15 into train, validation, and test.
   - Display split fraud rates.

3. **Feature and leakage policy**
   - Exclude `TransactionID` and `isFraud` from model features.
   - Keep `TransactionAmt` available both as a model feature and as the amount used for cost calculation.
   - Fit preprocessing on train only.
   - Transform validation/test without refit.

4. **Metric functions**
   - Implement PR-AUC.
   - Implement ROC-AUC.
   - Implement Recall fraud, Precision fraud, F1 fraud.
   - Implement confusion matrix fields: TN, FP, FN, TP.
   - Implement cost functions:
     - `FN Cost = sum(TransactionAmt * beta)` over fraud transactions predicted approve.
     - `FP Cost = sum(TransactionAmt * alpha)` over legitimate transactions predicted flag/block.
     - `Total Cost = FN Cost + FP Cost`.
     - `Cost Saving vs approve-all = approve_all_cost - model_total_cost`.

5. **Approve-all baseline**
   - Predict all transactions as approve.
   - Compute metrics and cost for validation/test.
   - Use this as the main cost reference.

6. **Logistic Regression baseline**
   - Use numeric median imputation.
   - Use categorical `"missing"` imputation and sparse one-hot encoding with rare-category handling when available.
   - Use `class_weight="balanced"`.
   - Use a sparse-friendly solver such as `saga` or a fallback solver that works on the local environment.
   - Predict probabilities for validation/test.

7. **Stronger ML baseline**
   - Prefer LightGBM or XGBoost only if already installable/available in the environment.
   - If unavailable, use `RandomForestClassifier` fallback with memory-aware parameters such as limited depth, `n_jobs=-1`, and class weighting.
   - Use lighter categorical preprocessing for tree models, such as ordinal encoding, to avoid huge one-hot matrices.
   - Predict probabilities for validation/test.

8. **Threshold tuning**
   - Tune thresholds on validation only.
   - For each model and each `alpha`/`beta` config, search thresholds and choose the threshold with minimum Total Cost.
   - Record PR-AUC/ROC-AUC separately from thresholded metrics.
   - Do not tune on test.

9. **Final test evaluation**
   - Apply validation-selected thresholds to test.
   - Save a main metrics table to `results/baseline_metrics.csv`.
   - Save threshold tuning table to `results/threshold_tuning.csv`.
   - Save confusion matrix table to `results/confusion_matrices.csv`.

10. **Figures**
    - PR curve for trained baselines.
    - ROC curve for trained baselines.
    - Cost-vs-threshold curve on validation.
    - Confusion matrix heatmaps for selected thresholds.
    - Total Cost bar chart by model and cost config.

11. **Notebook conclusion**
    - State which baseline has lowest validation-selected test Total Cost.
    - Report fraud Recall/F1 and PR-AUC prominently.
    - Explain why Accuracy is not a suitable primary metric for the imbalanced fraud setting.
    - Do not claim LLM/RL improvement because Phase 2 does not test LLM/RL.

## Expected Generated Outputs

These files are generated by running the notebook and are not source artifacts:

- `results/baseline_metrics.csv`
- `results/threshold_tuning.csv`
- `results/confusion_matrices.csv`
- `reports/figures/pr_curve_baselines.png`
- `reports/figures/roc_curve_baselines.png`
- `reports/figures/cost_vs_threshold.png`
- `reports/figures/total_cost_by_model.png`
- `reports/figures/confusion_matrix_*.png`

## Verification Plan

1. Notebook JSON is valid.
2. Notebook has no imports from `src`.
3. Notebook does not include LLM/table-to-text/embedding/RL implementation.
4. Smoke run with `SAMPLE_ROWS=10000` completes.
5. Full run preserves the Phase 1 join count and 70/15/15 `TransactionDT` split.
6. Preprocessing is fit on train only.
7. Validation threshold tuning does not touch test labels.
8. Test metrics include PR-AUC, ROC-AUC, Recall fraud, Precision fraud, F1 fraud, confusion matrix, FN Cost, FP Cost, Total Cost, and Cost Saving.
9. Approve-all, Logistic Regression, and one stronger ML baseline are all present.
10. Results and figures are saved when the notebook is run.

## Done Criteria

- `BASE-01..05` complete.
- `notebooks/02_baselines_cost_metrics.ipynb` exists and runs in local/Colab mode.
- Phase 2 outputs can be regenerated from raw IEEE-CIS files without relying on large Phase 1 `X_train.joblib`.
- Metrics prioritize fraud detection quality and economic cost, not Accuracy.
- LLM/table-to-text/RL remain deferred to Phase 3.

## Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Full one-hot preprocessing uses too much RAM | Use sparse matrix for Logistic Regression and ordinal/light preprocessing for tree baseline |
| Random Forest too slow on full IEEE-CIS | Use controlled depth, `n_estimators`, `max_samples` when supported, and smoke mode before full run |
| LightGBM/XGBoost not installed | Fall back to Random Forest without blocking Phase 2 |
| Threshold tuning leaks test labels | Tune on validation only and apply selected thresholds once on test |
| Accuracy looks high but model misses fraud | Report PR-AUC, fraud Recall/F1, confusion matrix, and Total Cost as primary metrics |

## Threat Model

- Do not print or commit Kaggle credentials.
- Do not include `isFraud` or `TransactionID` in model features.
- Do not tune preprocessing, thresholds, or hyperparameters on test labels.
- Do not create external API calls or upload transaction data to third-party services.
