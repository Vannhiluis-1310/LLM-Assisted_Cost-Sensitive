---
phase: 2
status: passed
verified: 2026-05-26
---

# Phase 2 Verification: ML Baselines and Cost Metrics

## Verdict

PASS for Phase 2 implementation scope.

The notebook-only deliverable exists, runs in local smoke mode, and covers `BASE-01..05`. It does not implement Phase 3 LLM/RL scope.

## Automated Checks

| Check | Status | Evidence |
|---|---|---|
| Notebook exists | PASS | `notebooks/02_baselines_cost_metrics.ipynb` |
| Notebook JSON valid | PASS | `notebook_json_ok 23` after Phase 2.1 finalization |
| Code cells parse | PASS | `bad_code_cells []` |
| No `src` imports | PASS | `from_src False` |
| Colab raw-data staging | PASS | Notebook stages CSV/ZIP files from Drive or `/content`, or uses Kaggle credentials, before loading `data/raw` |
| Metrics implemented | PASS | PR-AUC, ROC-AUC, fraud Recall/Precision/F1, confusion matrix, FN Cost, FP Cost, Total Cost, Cost Saving |
| Baselines implemented | PASS | approve-all, Logistic Regression class weight, stronger baseline with XGBoost/LightGBM/Random Forest fallback |
| Threshold tuning | PASS | validation-only threshold tuning for `Cost-A`, `Cost-B`, `Cost-C` |
| Phase 3 guardrail | PASS | no LLM/table-to-text/embedding/RL implementation |

## Generated Smoke Evidence

| Output | Status |
|---|---|
| `results/baseline_metrics.csv` | Created |
| `results/threshold_tuning.csv` | Created |
| `results/threshold_curve_validation.csv` | Created |
| `results/confusion_matrices.csv` | Created |
| `reports/figures/pr_curve_baselines.png` | Created |
| `reports/figures/roc_curve_baselines.png` | Created |
| `reports/figures/cost_vs_threshold.png` | Created |
| `reports/figures/total_cost_by_model.png` | Created |
| `reports/figures/confusion_matrix_*.png` | Created |

## Remaining Risk

Full dataset and 100k model training were not executed locally. This is intentional: notebook 02 is now configured to regenerate outputs via `RUN_MODE="full"` or `RUN_MODE="sample_100k"`, while local verification only checked notebook JSON/code syntax and the previously verified smoke flow.


## 2026-05-27 Static Verification Addendum

| Check | Status | Evidence |
|---|---|---|
| `sample_100k` mode exists | PASS | `RUN_MODE_SAMPLE_ROWS["sample_100k"] = 100_000` in notebook 02 |
| Tagged Phase 2 outputs | PASS | `baseline_metrics_{RUN_OUTPUT_TAG}.csv`, `threshold_tuning_{RUN_OUTPUT_TAG}.csv`, `confusion_matrices_{RUN_OUTPUT_TAG}.csv` |
| Run metadata | PASS | `phase2_run_metadata_{RUN_OUTPUT_TAG}.json` |
| Notebook JSON/code syntax | PASS | `ast.parse` over all code cells completed successfully on 2026-05-27 |

Execution of `RUN_MODE="sample_100k"` is pending user-run Colab execution.

## 2026-05-28 Execute Verification Addendum

| Check | Status | Evidence |
|---|---|---|
| Default run mode for Colab plan | PASS | Notebook 02 now defaults to `RUN_MODE = "sample_100k"` |
| Existing modes preserved | PASS | `smoke`, `sample_100k`, `sample_200k`, `sample_300k`, and `full` remain in `RUN_MODE_SAMPLE_ROWS` |
| Notebook syntax | PASS | `ast.parse` over all code cells completed successfully on 2026-05-28 |
| Stale outputs cleared | PASS | Notebook 02 has zero code cells with saved outputs after the execute update |

No 100k/full execution was run locally; this remains a Colab execution step.

## 2026-05-28 Gradual Sample Ladder Verification Addendum

| Check | Status | Evidence |
|---|---|---|
| `sample_200k` mode exists | PASS | `RUN_MODE_SAMPLE_ROWS["sample_200k"] = 200_000` in notebook 02 |
| `sample_300k` mode exists | PASS | `RUN_MODE_SAMPLE_ROWS["sample_300k"] = 300_000` in notebook 02 |
| Sample-mode RAM guard | PASS | `N_JOBS = 1 if RUN_MODE.startswith("sample_") else 2` |
| Tagged outputs remain mode-specific | PASS | Existing `baseline_metrics_{RUN_OUTPUT_TAG}.csv`, `threshold_tuning_{RUN_OUTPUT_TAG}.csv`, and metadata paths cover all sample modes |
| Static verification | PASS | Notebook JSON/code syntax checked after adding the ladder |

No 200k/300k/full execution was run locally; these are intended for user-run Colab execution.
