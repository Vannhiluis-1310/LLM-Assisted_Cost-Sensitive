---
created: 2026-05-26T00:00:00+07:00
completed: 2026-05-26T00:00:00+07:00
title: Add Kaggle-inspired XGB Magic baseline
area: general
files:
  - notebooks/02_baselines_cost_metrics.ipynb
  - .planning/phases/02.1-kaggle-inspired-xgb-magic-baseline/02.1-SUMMARY.md
  - .planning/phases/02.1-kaggle-inspired-xgb-magic-baseline/02.1-VERIFICATION.md
---

## Problem

Phase 2 baseline needed a stronger supervised reference because IEEE-CIS has well-known Kaggle baselines such as cdeotte XGB Magic and FraudSquad 1st place.

## Solution

Completed in Phase 2.1. The notebook now includes a leakage-safe `xgboost_magic_style` baseline with train-only count/frequency encodings and `TransactionAmt` group statistics. FraudSquad 1st place remains an external SOTA reference unless fully reproduced under the same split and cost-sensitive protocol.

## Completion Evidence

- `notebooks/02_baselines_cost_metrics.ipynb` includes `xgboost_magic_style`.
- Smoke run passed with `phase2_1_notebook_smoke_ok`.
- `results/magic_feature_policy.csv` generated 19 train-only magic features.
- `results/baseline_metrics.csv` includes `xgboost_magic_style`.

