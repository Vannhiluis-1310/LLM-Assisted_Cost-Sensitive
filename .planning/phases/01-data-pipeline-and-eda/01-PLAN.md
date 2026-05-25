---
phase: 1
plan: 1
status: complete
created: 2026-05-25
---

# Phase 1 Plan: Data Pipeline and EDA

## Objective

Build a local-first, Colab-ready IEEE-CIS Fraud Detection data foundation for later ML baseline, embedding, and RL phases.

## Scope

- Use IEEE-CIS Fraud Detection as the main dataset.
- Require `train_transaction.csv` and `train_identity.csv`.
- Left join on `TransactionID`.
- Compute fraud count, legitimate count, fraud rate, and identity coverage directly from loaded data.
- Split train/validation/test by `TransactionDT` using 70/15/15.
- Create EDA tables and figures.
- Create train-only preprocessing artifacts for later baselines.
- Keep raw data, processed data, artifacts, results, figures, and `kaggle.json` out of Git.

## Tasks

1. Create local/Colab project structure and dependency files.
2. Implement Kaggle download/validation helper.
3. Implement data loading, join integrity checks, summaries, EDA plots, and time-based split.
4. Implement train-only preprocessing with numeric/categorical handling and time-derived `TransactionDT` features.
5. Run smoke test and full local verification.

## Done Criteria

- `DATA-01..05` complete.
- `PREP-01..04` complete.
- Full dataset run creates `data/processed/`, `results/`, and `reports/figures/` outputs.
- Preprocessing creates `artifacts/preprocessing/` outputs and records train-only fit policy.
- Remaining risks are documented before Phase 2.
