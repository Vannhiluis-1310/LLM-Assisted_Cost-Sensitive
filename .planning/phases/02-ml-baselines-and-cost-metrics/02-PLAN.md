---
phase: 2
plan: 1
status: complete
created: 2026-05-26
updated: 2026-05-28
update_scope: add optional matched sample ladder 100k/200k/300k
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
- Add optional matched sample modes for fair Phase 2 vs Phase 3 comparison:
  - keep current full run behavior;
  - keep current smoke/local behavior;
  - add `RUN_MODE = "sample_100k"` so Phase 2 can run on the same first `100_000` IEEE-CIS rows used by Phase 3 sample mode;
  - add `RUN_MODE = "sample_200k"` and `RUN_MODE = "sample_300k"` as gradual Colab RAM ladder modes before attempting `full`.

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
   - Define `RUN_MODE` and derive `SAMPLE_ROWS` from it.
   - Supported modes:
     - `RUN_MODE = "smoke"` -> `SAMPLE_ROWS = 10_000`, local debug only.
     - `RUN_MODE = "sample_100k"` -> `SAMPLE_ROWS = 100_000`, RAM-safe matched comparison mode for Phase 3.
     - `RUN_MODE = "sample_200k"` -> `SAMPLE_ROWS = 200_000`, medium matched comparison mode after 100k passes.
     - `RUN_MODE = "sample_300k"` -> `SAMPLE_ROWS = 300_000`, larger matched comparison mode before full-data run.
     - `RUN_MODE = "full"` -> `SAMPLE_ROWS = None`, full IEEE-CIS run.
   - Preserve backward-compatible behavior by allowing direct manual `SAMPLE_ROWS` override if needed.
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
   - Also save run-mode-tagged copies so smoke/full/sample results do not overwrite each other:
     - `results/baseline_metrics_{RUN_MODE}.csv`
     - `results/threshold_tuning_{RUN_MODE}.csv`
     - `results/confusion_matrices_{RUN_MODE}.csv`
     - `results/phase2_run_metadata_{RUN_MODE}.json`
   - Every row in `baseline_metrics` must include `run_mode` and `sample_rows`.

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
5. `RUN_MODE="sample_100k"` produces exactly 100,000 joined rows split 70/15/15 as 70,000/15,000/15,000.
6. `RUN_MODE="sample_200k"` and `RUN_MODE="sample_300k"` produce matched tagged outputs and preserve the same split policy.
7. Full run preserves the Phase 1 join count and 70/15/15 `TransactionDT` split.
8. Preprocessing is fit on train only.
9. Validation threshold tuning does not touch test labels.
10. Test metrics include PR-AUC, ROC-AUC, Recall fraud, Precision fraud, F1 fraud, confusion matrix, FN Cost, FP Cost, Total Cost, and Cost Saving.
11. Approve-all, Logistic Regression, and one stronger ML baseline are all present.
12. Results and figures are saved when the notebook is run.
13. Tagged result files exist for each sample run mode so Phase 3 can consume matched baseline results without mixing sample sizes or full-data outputs.

## Done Criteria

- `BASE-01..05` complete.
- `notebooks/02_baselines_cost_metrics.ipynb` exists and runs in local/Colab mode.
- Phase 2 outputs can be regenerated from raw IEEE-CIS files without relying on large Phase 1 `X_train.joblib`.
- Metrics prioritize fraud detection quality and economic cost, not Accuracy.
- LLM/table-to-text/RL remain deferred to Phase 3.
- `RUN_MODE="sample_100k"`, `RUN_MODE="sample_200k"`, and `RUN_MODE="sample_300k"` are available for gradual matched Phase 2 vs Phase 3 comparison under 12GB RAM constraints.
- Existing `smoke` and `full` paths remain available.

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

## 2026-05-28 Addendum: Matched 100k Baseline Gate

### Rationale

Phase 3 smoke results are not enough to judge RL or LLM embeddings because 10,000 rows produce high variance and very few fraud examples. Before tuning Phase 3, Phase 2 must produce a matched supervised baseline on exactly the same first 100,000 transaction rows.

This keeps the comparison defensible:

- same IEEE-CIS raw files;
- same `TransactionID` left join;
- same `TransactionDT` 70/15/15 split;
- same `Cost-A`, `Cost-B`, `Cost-C`;
- same validation-only threshold tuning;
- no mixing full-data ML results with sample-based RL results.

### Execution Plan

| Step | Action | Output | Required Before Phase 3 Claim? |
|---:|---|---|---:|
| 1 | Set `RUN_MODE = "sample_100k"` in notebook 02 | notebook config print shows `SAMPLE_ROWS=100000` | Yes |
| 2 | Run notebook 02 on Colab | `results/baseline_metrics_sample_100k.csv` | Yes |
| 3 | Verify split rows | train `70000`, validation `15000`, test `15000` | Yes |
| 4 | Confirm models present | `approve_all`, Logistic Regression, strong tree baseline, magic-style baseline if available | Yes |
| 5 | Use only tagged 100k outputs for Phase 3 comparison | `baseline_metrics_sample_100k.csv` | Yes |

### Acceptance Criteria

1. `results/baseline_metrics_sample_100k.csv` exists.
2. `results/threshold_tuning_sample_100k.csv` exists.
3. `results/confusion_matrices_sample_100k.csv` exists.
4. `results/phase2_run_metadata_sample_100k.json` records `run_mode = "sample_100k"`.
5. The test table includes PR-AUC, ROC-AUC, fraud Recall/Precision/F1, FN Cost, FP Cost, Total Cost, and Cost Saving.
6. Phase 3 comparison must reject or label the run as mixed-scope if this file is missing.

### Non-Goals

- Do not tune Phase 2 models to help Phase 3.
- Do not add LLM/RL code to Phase 2.
- Do not report full-data Phase 2 numbers as directly comparable to Phase 3 `sample_100k`.

### Execute Update 2026-05-28

Notebook 02 now defaults to `RUN_MODE = "sample_100k"` for the current Colab execution plan. The `smoke` and `full` modes remain available through the same config variable.

### Gradual Sample Ladder Update 2026-05-28

Notebook 02 now supports a RAM-aware run ladder:

| Order | `RUN_MODE` | Rows | Purpose |
|---:|---|---:|---|
| 1 | `sample_100k` | 100,000 | Default Colab 12GB starting point |
| 2 | `sample_200k` | 200,000 | Increase sample only after 100k completes |
| 3 | `sample_300k` | 300,000 | Larger matched sample before attempting full |
| 4 | `full` | all rows | Final run only if RAM/runtime allow |

Each sample mode writes mode-tagged outputs such as `baseline_metrics_sample_200k.csv` or `baseline_metrics_sample_300k.csv`. Phase 3 must use the same run mode for a fair comparison.
