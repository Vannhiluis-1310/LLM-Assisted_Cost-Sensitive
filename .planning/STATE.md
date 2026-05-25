# Trạng thái dự án / Project State

Last activity: 2026-05-26 - Cleaned generated outputs and aligned state with notebook-first Phase 1

## Tham chiếu dự án / Project Reference

See: `.planning/PROJECT.md` (updated 2026-05-25)

**Giá trị cốt lõi / Core value:** Xây dựng được pipeline thực nghiệm chạy lại được, so sánh được baseline ML với RL cost-sensitive bằng metrics fraud và chi phí kinh tế.  
**Core value:** Build a reproducible experimental pipeline that compares ML baselines with cost-sensitive RL using fraud metrics and economic cost metrics.

**Trọng tâm hiện tại / Current focus:** Phase 2 - ML Baselines and Cost Metrics

## Trạng thái / Status

| Item | Status |
|---|---|
| Project initialized / Khởi tạo project | Complete |
| Requirements defined / Định nghĩa requirements | Complete |
| Roadmap created / Tạo roadmap | Complete |
| Phase 1 implementation / Triển khai Phase 1 | Complete |
| Phase 1 verification / Xác thực Phase 1 | Complete |
| Repository cleanup / Dọn cấu trúc repo | Complete |
| Phase 2 implementation / Triển khai Phase 2 | Pending |

## Phase 1 Verification Snapshot

| Check | Evidence |
|---|---|
| Dataset | IEEE-CIS Fraud Detection with `train_transaction.csv` and `train_identity.csv` in `data/raw/` |
| Join integrity | `590,540` transaction rows and `590,540` joined rows |
| Fraud rate | `20,663 / 590,540 = 3.499%`, computed directly from loaded data |
| Identity coverage | `144,233 / 590,540 = 24.42%` matched identity rows |
| Time split | train `413,378`, validation `88,581`, test `88,581`, ordered by `TransactionDT` |
| Preprocessing | train-only fit completed; metadata records `435` features before encoding, `404` numeric, `31` categorical, `3` time-derived features |

## Evidence Files

| Artifact | Purpose |
|---|---|
| `notebooks/01_data_check.ipynb` | Primary self-contained Colab-ready notebook; contains download, data pipeline, EDA, and preprocessing code |
| `data/raw/train_transaction.csv` | Local IEEE-CIS transaction input kept outside Git |
| `data/raw/train_identity.csv` | Local IEEE-CIS identity input kept outside Git |
| `requirements.txt` | Minimal local/Colab dependencies |
| `.gitignore` | Keeps raw/processed data, artifacts, results, figures, notebook checkpoints, and `kaggle.json` out of Git |

Generated outputs such as `data/processed/`, `results/`, `reports/figures/`, and `artifacts/` were intentionally removed during cleanup. They should be regenerated from `notebooks/01_data_check.ipynb` when needed.

## Risks / Concerns

- Full preprocessing artifacts were previously large: `artifacts/preprocessing/X_train.joblib` was about `4.1 GB`; validation/test matrices were about `880 MB` each. These generated files have now been removed.
- Phase 2 should avoid repeatedly regenerating full one-hot artifacts during iteration. Prefer smoke/sample mode and lighter baseline-specific preprocessing where possible.
- Raw IEEE-CIS CSV files remain local in `data/raw/`; processed data, artifacts, results, and figures are intentionally ignored by Git and should be regenerated locally/Colab-side.

## Quick Tasks Completed

| # | Description | Date | Commit | Directory |
|---|---|---|---|---|
| 260525-001 | Verify Phase 1 against REQUIREMENTS.md and ROADMAP.md before Phase 2 | 2026-05-25 | not committed - local git index is not writable in this environment | [260525-001-verify-phase-1-before-phase-2](./quick/260525-001-verify-phase-1-before-phase-2/) |
| 260525-002 | Prepare Phase 1 for Google Colab | 2026-05-25 | not committed - avoided partial initial commit with many untracked project files | [260525-002-prepare-phase-1-colab-ready](./quick/260525-002-prepare-phase-1-colab-ready/) |
| 260525-003 | Check current flow and align docs with notebook-first execution | 2026-05-25 | not committed - local git index is not writable in this environment | [260525-003-check-flow-notebook-first](./quick/260525-003-check-flow-notebook-first/) |

## Bước tiếp theo / Next Step

Start Phase 2 / Bắt đầu Phase 2:

1. **VI:** Tạo cost metric functions: PR-AUC, Recall fraud, Precision fraud, F1 fraud, FN Cost, FP Cost, Total Cost, Cost Saving.  
   **EN:** Create cost metric functions: PR-AUC, fraud recall, fraud precision, fraud F1, FN Cost, FP Cost, Total Cost, and Cost Saving.
2. **VI:** Tạo approve-all baseline để làm cost reference.  
   **EN:** Create approve-all baseline as the cost reference.
3. **VI:** Train Logistic Regression với class weight và một baseline mạnh hơn như LightGBM/XGBoost hoặc Random Forest.  
   **EN:** Train Logistic Regression with class weight and one stronger baseline such as LightGBM/XGBoost or Random Forest.
4. **VI:** Tune threshold trên validation cho ít nhất 3 cấu hình `alpha`/`beta`.  
   **EN:** Tune thresholds on validation for at least 3 `alpha`/`beta` settings.

## Session Continuity

Last session: 2026-05-25 23:45 +07:00
Latest cleanup: 2026-05-26 - removed old `src/` mirrors, generated EDA figures/results, processed splits, and heavy preprocessing artifacts; kept `.md` files and raw IEEE-CIS CSVs.
Stopped at: Phase 1 remains complete in notebook-first form; Phase 2 remains pending and ready for context discussion or planning.
Resume file: `.planning/STATE.md`
