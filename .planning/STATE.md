# Trạng thái dự án / Project State

Last activity: 2026-05-26 - Finalized Phase 2 and Phase 2.1 supervised baseline handoff

## Tham chiếu dự án / Project Reference

See: `.planning/PROJECT.md` (updated 2026-05-25)

**Giá trị cốt lõi / Core value:** Xây dựng được pipeline thực nghiệm chạy lại được, so sánh được baseline ML với RL cost-sensitive bằng metrics fraud và chi phí kinh tế.  
**Core value:** Build a reproducible experimental pipeline that compares ML baselines with cost-sensitive RL using fraud metrics and economic cost metrics.

**Trọng tâm hiện tại / Current focus:** Phase 3 - LLM Representation and RL

## Trạng thái / Status

| Item | Status |
|---|---|
| Project initialized / Khởi tạo project | Complete |
| Requirements defined / Định nghĩa requirements | Complete |
| Roadmap created / Tạo roadmap | Complete |
| Phase 1 implementation / Triển khai Phase 1 | Complete |
| Phase 1 verification / Xác thực Phase 1 | Complete |
| Repository cleanup / Dọn cấu trúc repo | Complete |
| Phase 2 planning / Lập kế hoạch Phase 2 | Complete |
| Phase 2 implementation / Triển khai Phase 2 | Complete |
| Phase 2 verification / Xác thực Phase 2 | Complete |
| Phase 2.1 insertion / Chèn Phase 2.1 | Complete |
| Phase 2.1 planning / Lập kế hoạch Phase 2.1 | Complete |
| Phase 2.1 implementation / Triển khai Phase 2.1 | Complete |
| Phase 2.1 verification / Xác thực Phase 2.1 | Complete |
| Phase 3 implementation / Triển khai Phase 3 | Pending |

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
| `.planning/phases/02-ml-baselines-and-cost-metrics/02-CONTEXT.md` | Phase 2 notebook-only scope and carry-over decisions |
| `.planning/phases/02-ml-baselines-and-cost-metrics/02-PLAN.md` | Phase 2 execution plan for ML baselines and cost-sensitive evaluation |
| `notebooks/02_baselines_cost_metrics.ipynb` | Main Phase 2/2.1 notebook for Colab-ready raw-data staging, approve-all, ML baselines, magic-style baseline, threshold tuning, metrics, and figures |
| `.planning/phases/02-ml-baselines-and-cost-metrics/02-SUMMARY.md` | Phase 2 execution summary |
| `.planning/phases/02-ml-baselines-and-cost-metrics/02-VERIFICATION.md` | Phase 2 verification report |
| `.planning/phases/02.1-kaggle-inspired-xgb-magic-baseline/02.1-CONTEXT.md` | Phase 2.1 scope and leakage-safety decisions |
| `.planning/phases/02.1-kaggle-inspired-xgb-magic-baseline/02.1-PLAN.md` | Phase 2.1 execution plan for Kaggle-inspired supervised baseline |
| `.planning/phases/02.1-kaggle-inspired-xgb-magic-baseline/02.1-SUMMARY.md` | Phase 2.1 execution summary |
| `.planning/phases/02.1-kaggle-inspired-xgb-magic-baseline/02.1-VERIFICATION.md` | Phase 2.1 verification report |

Generated outputs such as `data/processed/`, `results/`, `reports/figures/`, and `artifacts/` were intentionally removed during cleanup. They should be regenerated from `notebooks/01_data_check.ipynb` when needed.

Phase 2 regenerated `results/` and `reports/figures/` during smoke verification. These generated outputs remain ignored by Git.

## Risks / Concerns

- Full preprocessing artifacts were previously large: `artifacts/preprocessing/X_train.joblib` was about `4.1 GB`; validation/test matrices were about `880 MB` each. These generated files have now been removed.
- Phase 2 should avoid repeatedly regenerating full one-hot artifacts during iteration. Prefer smoke/sample mode and lighter baseline-specific preprocessing where possible.
- Raw IEEE-CIS CSV files remain local in `data/raw/`; processed data, artifacts, results, and figures are intentionally ignored by Git and should be regenerated locally/Colab-side.
- Phase 2 full dataset model training was not executed locally; `notebooks/02_baselines_cost_metrics.ipynb` defaults to `SAMPLE_ROWS = 10000` and can be switched to `None` for full run.
- Kaggle-inspired XGBoost Magic-style baseline has been implemented and smoke-verified, but full dataset training is still pending for final report numbers.

## Quick Tasks Completed

| # | Description | Date | Commit | Directory |
|---|---|---|---|---|
| 260525-001 | Verify Phase 1 against REQUIREMENTS.md and ROADMAP.md before Phase 2 | 2026-05-25 | not committed - local git index is not writable in this environment | [260525-001-verify-phase-1-before-phase-2](./quick/260525-001-verify-phase-1-before-phase-2/) |
| 260525-002 | Prepare Phase 1 for Google Colab | 2026-05-25 | not committed - avoided partial initial commit with many untracked project files | [260525-002-prepare-phase-1-colab-ready](./quick/260525-002-prepare-phase-1-colab-ready/) |
| 260525-003 | Check current flow and align docs with notebook-first execution | 2026-05-25 | not committed - local git index is not writable in this environment | [260525-003-check-flow-notebook-first](./quick/260525-003-check-flow-notebook-first/) |
| 260526-001 | Finalize Phase 2 supervised baseline handoff | 2026-05-26 | pending local commit | [260526-001-finalize-phase-2](./quick/260526-001-finalize-phase-2/) |

## Bước tiếp theo / Next Step

Start Phase 3 / Bắt đầu Phase 3:

1. **VI:** Lập context/plan cho table-to-text serialization trung lập từ feature thật.  
   **EN:** Plan neutral table-to-text serialization from real IEEE-CIS features.
2. **VI:** Tạo local MiniLM embedding cache, nối đúng split, không fit PCA trên test.  
   **EN:** Create local MiniLM embedding cache, join to the correct split, and avoid fitting PCA on test.
3. **VI:** Implement contextual bandit without embedding và with embedding để ablation.  
   **EN:** Implement contextual bandit without embedding and with embedding for ablation.
4. **VI:** Dùng cùng reward, split, và metrics cost-sensitive với Phase 2/2.1 baselines.  
   **EN:** Use the same reward, split, and cost-sensitive metrics as Phase 2/2.1 baselines.

## Accumulated Context

### Roadmap Evolution

- Phase 2.1 inserted after Phase 2: Kaggle-inspired XGB Magic Baseline (URGENT)

## Session Continuity

Last session: 2026-05-25 23:45 +07:00
Latest cleanup: 2026-05-26 - removed old `src/` mirrors, generated EDA figures/results, processed splits, and heavy preprocessing artifacts; kept `.md` files and raw IEEE-CIS CSVs.
Latest planning: 2026-05-26 - Phase 2 planned as notebook-only baseline ML and cost-sensitive evaluation; LLM/table-to-text/RL deferred to Phase 3.
Latest execution: 2026-05-26 - created and smoke-verified `notebooks/02_baselines_cost_metrics.ipynb`; generated baseline metrics and figures on 10,000-row smoke run.
Latest insertion: 2026-05-26 - inserted Phase 2.1 before Phase 3 to add a Kaggle-inspired XGB Magic-style supervised baseline.
Latest Phase 2.1 planning: 2026-05-26 - created context and plan for leakage-safe magic-style supervised baseline.
Latest Phase 2.1 execution: 2026-05-26 - updated `notebooks/02_baselines_cost_metrics.ipynb` with `xgboost_magic_style`, generated `magic_feature_policy.csv`, and smoke-verified the full flow.
Latest Phase 2 finalization: 2026-05-26 - confirmed baseline metrics/threshold outputs, restored Colab raw-data staging in notebook 02, and marked Phase 2/2.1 ready for Phase 3 comparison.
Stopped at: Phase 2/2.1 finalized; Phase 3 remains pending and should start with context/planning before implementation.
Resume file: `.planning/STATE.md`
