---
phase: 1
status: complete
completed: 2026-05-25
---

# Phase 1 Summary: Data Pipeline and EDA

## Outcome

Phase 1 is complete. The project now has a local-first, Colab-ready IEEE-CIS data pipeline with raw data validation, Kaggle download support, left join integrity checks, EDA outputs, time-based splits, and train-only preprocessing. The primary runnable artifact is now the self-contained notebook `notebooks/01_data_check.ipynb`; the `.py` files under `src/` remain as script mirrors/helpers.

## Requirements Verified

| Requirement | Status | Evidence |
|---|---|---|
| DATA-01 | Complete | IEEE-CIS Fraud Detection is the only main dataset; required local files are `data/raw/train_transaction.csv` and `data/raw/train_identity.csv`. |
| DATA-02 | Complete | `notebooks/01_data_check.ipynb` loads both raw CSV files directly; `src/data/make_dataset.py` is an optional script mirror. |
| DATA-03 | Complete | Left join on `TransactionID` preserves row count: `590,540` transaction rows and `590,540` joined rows. |
| DATA-04 | Complete | `results/dataset_summary.csv` reports fraud count `20,663`, legitimate count `569,877`, fraud rate `0.0349900091`. |
| DATA-05 | Complete | `results/split_summary.csv` records 70/15/15 split ordered by `TransactionDT`. |
| PREP-01 | Complete | `notebooks/01_data_check.ipynb` includes train-only preprocessing with median numeric imputation; `src/features/preprocess.py` is an optional script mirror. |
| PREP-02 | Complete | Categorical preprocessing uses explicit `"missing"` imputation and `OneHotEncoder` rare-category handling via `min_frequency=10` when supported. |
| PREP-03 | Complete | `TransactionID` and `isFraud` are excluded from model features. |
| PREP-04 | Complete | Preprocessing metadata records 3 derived `TransactionDT` features and states no real calendar-date interpretation. |

## Full Dataset Results

| Metric | Value |
|---|---:|
| Transaction rows | `590,540` |
| Transaction columns | `394` |
| Identity rows | `144,233` |
| Identity columns | `41` |
| Joined rows | `590,540` |
| Joined columns | `434` |
| Fraud count | `20,663` |
| Legitimate count | `569,877` |
| Fraud rate | `3.499%` |
| Identity coverage | `24.42%` |

## Split Results

| Split | Rows | Fraud count | Fraud rate | TransactionDT range |
|---|---:|---:|---:|---|
| Train | `413,378` | `14,538` | `3.517%` | `86,400` to `10,437,996` |
| Validation | `88,581` | `3,042` | `3.434%` | `10,438,003` to `13,151,840` |
| Test | `88,581` | `3,083` | `3.480%` | `13,151,880` to `15,811,131` |

## Evidence Files

| File | Purpose |
|---|---|
| `requirements.txt` | Minimal local/Colab dependencies |
| `.gitignore` | Keeps raw/processed data, artifacts, results, figures, notebook checkpoints, and `kaggle.json` out of Git |
| `notebooks/01_data_check.ipynb` | Primary self-contained EDA notebook with merged download, raw loading, join, split, EDA, and preprocessing code |
| `src/data/download_dataset.py` | Optional script mirror for Kaggle download and required-file validation |
| `src/data/make_dataset.py` | Optional script mirror for raw loading, join, summaries, EDA figures, and split creation |
| `src/features/preprocess.py` | Optional script mirror for train-only preprocessing and model-ready artifact creation |
| `results/dataset_summary.csv` | Dataset and join summary |
| `results/split_summary.csv` | Split summary |
| `results/missing_summary_top30.csv` | Missing-value summary |
| `reports/figures/class_distribution.png` | Class distribution chart |
| `reports/figures/transaction_amount_distribution.png` | Amount distribution chart |
| `reports/figures/missing_values_top30.png` | Missing-value chart |
| `artifacts/preprocessing/preprocessing_metadata.csv` | Preprocessing metadata and train-only policy |

## Verification Commands

Notebook-first verification:

```text
Open notebooks/01_data_check.ipynb
Set SAMPLE_ROWS = 10000 for smoke test or None for full run
Run sections 1 through 18
Keep RUN_PREPROCESSING = False unless preprocessing artifacts must be regenerated
```

Optional script mirror verification:

```bash
python -m compileall src
python -m src.data.make_dataset --raw-dir data/raw --processed-dir data/processed --results-dir results --figures-dir reports/figures --sample-rows 10000
python -m src.data.make_dataset --raw-dir data/raw --processed-dir data/processed --results-dir results --figures-dir reports/figures --split-ratios 0.70 0.15 0.15 --seed 42
python -m src.features.preprocess --processed-dir data/processed --artifacts-dir artifacts/preprocessing
```

## Remaining Risks

- `artifacts/preprocessing/X_train.joblib` is about `4.1 GB`; validation/test matrices are about `880 MB` each. This is acceptable as a Phase 1 artifact but too heavy for repeated iteration.
- Phase 2 should prefer fast smoke runs and lighter baseline-specific preprocessing where possible.
- Data and generated artifacts are intentionally not tracked in Git. They must be regenerated locally/Colab-side when moving machines.

## Handoff to Phase 2

Phase 2 should start with cost metric functions and approve-all baseline, then Logistic Regression with class weight, then one stronger baseline such as LightGBM/XGBoost or Random Forest.
