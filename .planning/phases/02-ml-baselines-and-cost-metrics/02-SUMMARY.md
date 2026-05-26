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

On the 10,000-row smoke run, `xgboost_scale_pos_weight` had the lowest test Total Cost for all three cost configurations. These are smoke-test numbers only; final report numbers should come from a full run by setting `SAMPLE_ROWS = None`.

## Notes and Risks

- The notebook defaults to smoke mode to avoid local CPU/RAM issues.
- Logistic Regression uses `N_JOBS = 1` by default because Windows local execution blocked `n_jobs=-1`.
- Full dataset baseline training has not been run locally in this execution turn.
- Phase 2 intentionally does not implement LLM embeddings, table-to-text, contextual bandit, DQN, or RL ablations.
