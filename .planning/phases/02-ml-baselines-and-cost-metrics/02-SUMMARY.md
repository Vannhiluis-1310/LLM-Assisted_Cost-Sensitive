---
phase: 2
status: complete
completed: 2026-05-26
---

# Phase 2 Summary: ML Baselines and Cost Metrics

## Outcome

Phase 2 implementation is complete as a notebook-only artifact. The project now has `notebooks/02_baselines_cost_metrics.ipynb`, which stages/loads the IEEE-CIS raw files, recreates the Phase 1 `TransactionDT` 70/15/15 split, trains baseline ML models, tunes thresholds on validation, evaluates on test, and writes cost-sensitive metrics/figures.

## Delivered

| Requirement | Status | Evidence |
|---|---|---|
| BASE-01 | Complete | `approve_all` baseline predicts action `0 = approve` for every transaction and is used as the cost reference. |
| BASE-02 | Complete | `logistic_regression_balanced` uses `LogisticRegression(class_weight="balanced")`. |
| BASE-03 | Complete | Stronger baseline uses LightGBM/XGBoost if available; local smoke run used `xgboost_scale_pos_weight`. Random Forest fallback remains implemented. |
| BASE-04 | Complete | Imbalance handling exists via class weight, XGBoost `scale_pos_weight`, and validation threshold tuning. |
| BASE-05 | Complete | Thresholds are selected on validation for `Cost-A`, `Cost-B`, and `Cost-C`, then applied to test. |

## Key Files

| File | Purpose |
|---|---|
| `notebooks/02_baselines_cost_metrics.ipynb` | Main Phase 2 source artifact; no `src/` script mirror was created. |
| `results/baseline_metrics.csv` | Generated smoke-run metrics table with PR-AUC, ROC-AUC, fraud Recall/Precision/F1, confusion matrix fields, FN/FP/Total Cost, and Cost Saving. |
| `results/threshold_tuning.csv` | Generated validation-selected thresholds for each model and cost configuration. |
| `results/confusion_matrices.csv` | Generated confusion matrix counts. |
| `reports/figures/*.png` | Generated PR curve, ROC curve, cost-vs-threshold, total-cost bar chart, and confusion matrix figures. |

## Smoke Verification

The notebook was executed locally with `SAMPLE_ROWS = 10000`.

| Check | Result |
|---|---|
| Notebook JSON | PASS, final notebook has 23 cells after Phase 2.1 finalization |
| Code syntax | PASS, no bad code cells |
| `src` dependency | PASS, no `from src` or `import src` |
| Colab raw-data staging | PASS, notebook can stage CSV/ZIP files from Drive or `/content`, and can use Kaggle credentials |
| Forbidden Phase 3 implementation | PASS, no LLM/table-to-text/embedding/RL implementation |
| Join integrity | PASS, `10,000` transaction rows and `10,000` joined rows |
| Smoke split | train `7,000`, validation `1,500`, test `1,500` |
| Models | `approve_all`, `logistic_regression_balanced`, `xgboost_scale_pos_weight`; Phase 2.1 adds `xgboost_magic_style` |
| Cost configs | `Cost-A`, `Cost-B`, `Cost-C` |
| Output metrics | PASS, required cost and fraud metrics present |

## Smoke Results Snapshot

On the 10,000-row smoke run, `xgboost_scale_pos_weight` had the lowest test Total Cost for all three cost configurations. These are smoke-test numbers only; final report numbers should come from either matched `RUN_MODE="sample_100k"` runs for Phase 2/3 comparison or a full run when RAM allows.

## Notes and Risks

- The notebook now uses `RUN_MODE`; default is `"sample_100k"` for the current Colab/RAM 12GB execution plan, with `"smoke"`, `"sample_200k"`, `"sample_300k"`, and `"full"` still available.
- Logistic Regression uses `N_JOBS = 1` by default because Windows local execution blocked `n_jobs=-1`.
- Full dataset baseline training has not been run locally in this execution turn.
- Phase 2 intentionally does not implement LLM embeddings, table-to-text, contextual bandit, DQN, or RL ablations.

## 2026-05-27 Matched 100k Update

Notebook 02 now supports `RUN_MODE = "smoke" | "sample_100k" | "full"`. In `sample_100k`, it reads the first 100,000 IEEE-CIS transaction rows, preserves the `TransactionDT` 70/15/15 split, and writes tagged outputs such as `results/baseline_metrics_sample_100k.csv`, `results/threshold_tuning_sample_100k.csv`, `results/confusion_matrices_sample_100k.csv`, and `results/phase2_run_metadata_sample_100k.json`.

## 2026-05-28 Execute Update

Notebook 02 is now set to `RUN_MODE = "sample_100k"` by default so the next Colab run produces the matched Phase 2 baseline required by Phase 3. The full-data path remains available by changing the same config line back to `RUN_MODE = "full"`.

## 2026-05-28 Gradual Sample Ladder Update

Notebook 02 now also supports `RUN_MODE = "sample_200k"` and `RUN_MODE = "sample_300k"`. These modes keep the same raw IEEE-CIS source, left join, `TransactionDT` 70/15/15 split, validation-only threshold tuning, and tagged output behavior.

Recommended execution order is:

1. Run notebook 02 with `sample_100k`.
2. If it completes, run notebook 02 with `sample_200k`.
3. If it completes, run notebook 02 with `sample_300k`.
4. Attempt `full` only after the sample ladder is stable.

Each Phase 2 sample run must be paired with the same Phase 3 sample run before making baseline-vs-RL comparison claims.
