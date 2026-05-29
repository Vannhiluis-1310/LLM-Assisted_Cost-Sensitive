# Trạng thái dự án / Project State

Last activity: 2026-05-30 - Source-implemented Phase 3.3 advanced Level 5 v2 notebook

## Tham chiếu dự án / Project Reference

See: `.planning/PROJECT.md` (updated 2026-05-25)

**Giá trị cốt lõi / Core value:** Xây dựng được pipeline thực nghiệm chạy lại được, so sánh được baseline ML với RL cost-sensitive bằng metrics fraud và chi phí kinh tế.  
**Core value:** Build a reproducible experimental pipeline that compares ML baselines with cost-sensitive RL using fraud metrics and economic cost metrics.

**Trọng tâm hiện tại / Current focus:** Run `notebooks/06_llm_augmented_hybrid_v2.ipynb` on Colab with MiniLM and download `phase33_*` outputs

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
| Phase 3 planning / Lập kế hoạch Phase 3 | Complete |
| Phase 3 implementation (SGD smoke) / Triển khai Phase 3 lần 1 | Complete (smoke only) |
| Phase 3 verification (SGD smoke) / Xác thực Phase 3 lần 1 | Complete (smoke only) |
| Phase 3 upgrade planning / Lập kế hoạch nâng cấp Phase 3 | Complete |
| Phase 3 upgrade implementation / Tri?n khai n?ng c?p Phase 3 | Complete (smoke verified) |
| Phase 3 upgrade verification / X?c th?c n?ng c?p Phase 3 | Complete (smoke/static verified) |
| Phase 2/3 matched 100k mode planning / Lập kế hoạch mode 100k đồng bộ Phase 2/3 | Complete |
| Phase 2/3 matched 100k mode implementation / Tri?n khai mode 100k ??ng b? Phase 2/3 | Complete (code/static verified; Colab run pending) |
| Phase 2/3 gradual sample ladder / 100k -> 200k -> 300k | Complete (code/static verified; Colab run pending) |
| Five-level evaluation framework update / Cập nhật khung đánh giá 5 mức | Complete |
| Phase 3.1 planning / Lập kế hoạch Phase 3.1 | Complete |
| Phase 3.1 dynamic cost-sensitive policy / Chính sách dynamic cost-sensitive | Complete (source/static verified; Colab run pending) |
| Phase 3.1b planning / Lập kế hoạch Phase 3.1b | Complete |
| Phase 3.1b Level 4 tuning / Tinh chỉnh Level 4 | Complete (sample_100k verified) |
| Phase 3.2 planning / Lập kế hoạch Phase 3.2 | Complete |
| Phase 3.2 LLM-assisted hybrid policy / Chính sách hybrid có LLM | Complete (Colab MiniLM sample_100k verified; Level 5 v1 matched but did not beat Level 4) |
| Phase 3.3 planning / Advanced LLM-augmented hybrid decision framework | Complete |
| Phase 3.3 implementation / Level 5 v2 advanced hybrid policy | Complete (source/static verified; Colab MiniLM numeric run pending) |
| Phase 4 implementation / Triển khai Phase 4 | Pending |

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
| `PROJECT_PLAN.md` | Root project plan for the five-level cost-sensitive decision framework |
| `TASKLIST.md` | Root task checklist for Phase 3.1, Phase 3.2, and final evaluation |
| `.planning/phases/02-ml-baselines-and-cost-metrics/02-CONTEXT.md` | Phase 2 notebook-only scope and carry-over decisions |
| `.planning/phases/02-ml-baselines-and-cost-metrics/02-PLAN.md` | Phase 2 execution plan for ML baselines and cost-sensitive evaluation |
| `notebooks/02_baselines_cost_metrics.ipynb` | Main Phase 2/2.1 notebook for Colab-ready raw-data staging, approve-all, ML baselines, magic-style baseline, threshold tuning, metrics, and figures |
| `.planning/phases/02-ml-baselines-and-cost-metrics/02-SUMMARY.md` | Phase 2 execution summary |
| `.planning/phases/02-ml-baselines-and-cost-metrics/02-VERIFICATION.md` | Phase 2 verification report |
| `.planning/phases/02.1-kaggle-inspired-xgb-magic-baseline/02.1-CONTEXT.md` | Phase 2.1 scope and leakage-safety decisions |
| `.planning/phases/02.1-kaggle-inspired-xgb-magic-baseline/02.1-PLAN.md` | Phase 2.1 execution plan for Kaggle-inspired supervised baseline |
| `.planning/phases/02.1-kaggle-inspired-xgb-magic-baseline/02.1-SUMMARY.md` | Phase 2.1 execution summary |
| `.planning/phases/02.1-kaggle-inspired-xgb-magic-baseline/02.1-VERIFICATION.md` | Phase 2.1 verification report |
| `.planning/phases/03-llm-representation-and-rl/03-CONTEXT.md` | Phase 3 notebook-only smoke-local/full-Colab scope and guardrails |
| `.planning/phases/03-llm-representation-and-rl/03-PLAN.md` | Phase 3 execution plan for table-to-text, MiniLM embeddings, and contextual bandit ablations |
| `notebooks/03_llm_representation_and_rl.ipynb` | Main Phase 3 notebook for table-to-text, embedding cache, and contextual bandit ablations |
| `.planning/phases/03-llm-representation-and-rl/03-SUMMARY.md` | Phase 3 execution summary |
| `.planning/phases/03-llm-representation-and-rl/03-VERIFICATION.md` | Phase 3 smoke execution verification report |
| `.planning/phases/03.1-dynamic-cost-sensitive-decision-policies/03.1-CONTEXT.md` | Locked Phase 3.1 decisions: new notebook 04, score input contract, no notebook 03 changes |
| `.planning/phases/03.1-dynamic-cost-sensitive-decision-policies/03.1-PLAN.md` | Phase 3.1 execution plan for Level 3 global threshold and Level 4 amount-bin dynamic threshold |
| `notebooks/04_dynamic_cost_policy.ipynb` | Phase 3.1 notebook for Level 3 global cost threshold and Level 4 amount-bin dynamic threshold |
| `.planning/phases/03.1-dynamic-cost-sensitive-decision-policies/03.1-SUMMARY.md` | Phase 3.1 source implementation summary |
| `.planning/phases/03.1-dynamic-cost-sensitive-decision-policies/03.1-VERIFICATION.md` | Phase 3.1 source/static verification |
| `.planning/phases/03.1b-level4-dynamic-policy-tuning/03.1b-CONTEXT.md` | Phase 3.1b context from first dynamic-policy result review |
| `.planning/phases/03.1b-level4-dynamic-policy-tuning/03.1b-PLAN.md` | Phase 3.1b plan for tuned Level 4 candidate policies |
| `notebooks/04b_dynamic_policy_tuning.ipynb` | Phase 3.1b notebook for tuned Level 4 candidate policies |
| `.planning/phases/03.1b-level4-dynamic-policy-tuning/03.1b-SUMMARY.md` | Phase 3.1b implementation summary |
| `.planning/phases/03.1b-level4-dynamic-policy-tuning/03.1b-VERIFICATION.md` | Phase 3.1b implementation verification |
| `results/phase31b_candidate_metrics_sample_100k.csv` | Phase 3.1b candidate and selected-policy metrics |
| `results/five_level_comparison_tuned_sample_100k.csv` | Five-level comparison with tuned Level 4 rows |
| `.planning/phases/03.2-llm-assisted-hybrid-policy/03.2-CONTEXT.md` | Phase 3.2 context for Level 5 LLM-assisted hybrid policy |
| `.planning/phases/03.2-llm-assisted-hybrid-policy/03.2-PLAN.md` | Phase 3.2 plan for `notebooks/05_llm_hybrid_policy.ipynb` |
| `.planning/phases/03.2-llm-assisted-hybrid-policy/03.2-SUMMARY.md` | Phase 3.2 implementation summary |
| `.planning/phases/03.2-llm-assisted-hybrid-policy/03.2-VERIFICATION.md` | Phase 3.2 execution verification |
| `notebooks/05_llm_hybrid_policy.ipynb` | Phase 3.2 Level 5 LLM-assisted hybrid policy notebook |
| `results/phase32_hybrid_candidate_metrics_sample_100k.csv` | Phase 3.2 Level 5 candidate metrics from Colab MiniLM sample_100k run |
| `results/phase32_selected_policy_sample_100k.csv` | Selected Level 5 policy per cost config |
| `results/five_level_comparison_level5_sample_100k.csv` | Five-level comparison with Level 5 rows |
| `results/phase32_run_metadata_sample_100k.json` | Phase 3.2 metadata and claim guard |
| `.planning/phases/03.3-advanced-llm-augmented-hybrid-decision-framework/03.3-CONTEXT.md` | Phase 3.3 context and leakage-safe decisions |
| `.planning/phases/03.3-advanced-llm-augmented-hybrid-decision-framework/03.3-PLAN.md` | Phase 3.3 plan for `notebooks/06_llm_augmented_hybrid_v2.ipynb` |
| `.planning/phases/03.3-advanced-llm-augmented-hybrid-decision-framework/03.3-SUMMARY.md` | Phase 3.3 source implementation summary |
| `.planning/phases/03.3-advanced-llm-augmented-hybrid-decision-framework/03.3-VERIFICATION.md` | Phase 3.3 source/static verification |
| `notebooks/06_llm_augmented_hybrid_v2.ipynb` | Phase 3.3 advanced Level 5 v2 notebook |

Generated outputs such as `data/processed/`, `results/`, `reports/figures/`, and `artifacts/` were intentionally removed during cleanup. They should be regenerated from `notebooks/01_data_check.ipynb` when needed.

Phase 2 regenerated `results/` and `reports/figures/` during smoke verification. These generated outputs remain ignored by Git.

## Risks / Concerns

- Full preprocessing artifacts were previously large: `artifacts/preprocessing/X_train.joblib` was about `4.1 GB`; validation/test matrices were about `880 MB` each. These generated files have now been removed.
- Phase 2 should avoid repeatedly regenerating full one-hot artifacts during iteration. Prefer smoke/sample mode and lighter baseline-specific preprocessing where possible.
- Raw IEEE-CIS CSV files remain local in `data/raw/`; processed data, artifacts, results, and figures are intentionally ignored by Git and should be regenerated locally/Colab-side.
- Phase 2 full dataset model training was not executed locally; `notebooks/02_baselines_cost_metrics.ipynb` defaults to `SAMPLE_ROWS = 10000` and can be switched to `None` for full run.
- Kaggle-inspired XGBoost Magic-style baseline has been implemented and smoke-verified, but full dataset training is still pending for final report numbers.
- Phase 3 smoke execution is complete. Full-data Phase 2/2.1 and Phase 3 results should still be produced on Colab and downloaded for Phase 4 analysis.
- Local Phase 3 smoke used `tfidf_svd_fallback` because `sentence-transformers` is not installed locally. Colab/full run should verify `embedding_backend_used = "minilm"` in `results/phase3_run_metadata.json`.
- Notebook 02 and notebook 03 now default to `RUN_MODE="sample_100k"` for the current Colab/RAM 12GB execution plan. `smoke`, `sample_200k`, `sample_300k`, and `full` modes remain available. The sample ladder has been statically verified but not executed locally.
- Current sample results indicate Q-bandit is not strong enough to be the main performance claim. It remains RL ablation evidence.
- The closed-form expected-cost threshold is not amount-dynamic because `TransactionAmt` cancels under `C_FN = TransactionAmt * beta` and `C_FP = TransactionAmt * alpha`. Level 4 must use validation-tuned amount-bin thresholds instead.
- Phase 3.1 source is implemented but numeric evidence depends on rerunning notebook 02 first; otherwise notebook 04 will fail because `results/baseline_scores_{RUN_MODE}.csv` will be missing.
- First Phase 3.1 results suggest Level 4 is useful in Cost-C but can over-block in Cost-A/B. Phase 3.1b must tune this trade-off before Phase 3.2 claims.
- Phase 3.1b sample results improve the over-blocking story, but they are still `sample_100k`; final report should label the run mode clearly unless larger/full outputs are regenerated.
- The Phase 3.1b local execution wrapper timed out after printing completion; post-run validation confirmed all expected outputs were written.
- Phase 3.2 Colab run used `minilm`, selected gamma `0.00`, and matched tuned Level 4 exactly on Cost-A/B/C. This is valid Level 5 v1 evidence, but it does not support an LLM-improves-cost claim.
- Phase 3.3 targets a stronger Level 5 v2, but the 2/3 cost-setting improvement is an aspirational target, not a guaranteed claim. Train-only prototypes and validation-only tuning are required to avoid leakage.
- Phase 3.3 source is implemented and syntax-verified, but final numeric evidence still requires a Colab MiniLM run. If `phase33_run_metadata_sample_100k.json` reports `tfidf_svd_fallback`, treat it as smoke-only.
- Full Kaggle/FraudSquad SOTA reproduction is out of scope. It remains an external reference unless explicitly scoped and reproduced.

## Quick Tasks Completed

| # | Description | Date | Commit | Directory |
|---|---|---|---|---|
| 260525-001 | Verify Phase 1 against REQUIREMENTS.md and ROADMAP.md before Phase 2 | 2026-05-25 | not committed - local git index is not writable in this environment | [260525-001-verify-phase-1-before-phase-2](./quick/260525-001-verify-phase-1-before-phase-2/) |
| 260525-002 | Prepare Phase 1 for Google Colab | 2026-05-25 | not committed - avoided partial initial commit with many untracked project files | [260525-002-prepare-phase-1-colab-ready](./quick/260525-002-prepare-phase-1-colab-ready/) |
| 260525-003 | Check current flow and align docs with notebook-first execution | 2026-05-25 | not committed - local git index is not writable in this environment | [260525-003-check-flow-notebook-first](./quick/260525-003-check-flow-notebook-first/) |
| 260526-001 | Finalize Phase 2 supervised baseline handoff | 2026-05-26 | not committed - local git index is not writable in this environment | [260526-001-finalize-phase-2](./quick/260526-001-finalize-phase-2/) |
| 260528-001 | Update evaluation framework after poor Q-bandit results | 2026-05-28 | not committed | [260528-001-update-evaluation-framework](./quick/260528-001-update-evaluation-framework/) |

## Bước tiếp theo / Next Step

Run Phase 3.3 on Colab:

1. Open `notebooks/06_llm_augmented_hybrid_v2.ipynb`.
2. Keep `RUN_MODE = "sample_100k"` for the first run.
3. Run all cells on Colab/GPU.
4. Confirm `results/phase33_run_metadata_sample_100k.json` records `embedding_backend_used = "minilm"`.
5. Download `results/phase33_*_sample_100k.*`, `results/five_level_comparison_level5_v2_sample_100k.csv`, and `reports/figures/phase33_*_sample_100k.png`.
6. Use the win count in `phase33_level5_v2_vs_level4_sample_100k.csv` to decide whether Level 5 v2 supports the LLM claim.

## Accumulated Context

### Roadmap Evolution

- Phase 2.1 inserted after Phase 2: Kaggle-inspired XGB Magic Baseline (URGENT)
- Phase 3 upgrade inserted 2026-05-27: SGDClassifier → Q-value Gym-style Contextual Bandit (RL defensibility fix)
- Phase 2/3 matched mode planned 2026-05-27: add `sample_100k` run mode for RAM 12GB and fair baseline-vs-RL comparison
- Phase 2/3 matched mode implemented 2026-05-27: notebook 02 writes tagged Phase 2 outputs; notebook 03 supports `sample_100k`, uses RAM-safe embedding/training settings, and prefers `baseline_metrics_sample_100k.csv`.
- Phase 3 runtime fix 2026-05-28: notebook 03 Section 5 now exposes `load_or_create_raw_embeddings()`, Section 6 reloads raw embeddings when missing, and local project-root override is Colab-only.
- Phase 2/3 tuning plan 2026-05-28: execute matched `sample_100k` first, then tune Phase 3 reward scaling, epsilon/learning-rate schedule, and optional leakage-safe magic-lite state features only if 100k results show Q-bandit remains weak.
- Phase 2/3 execute update 2026-05-28: notebook 02 and 03 now default to `sample_100k`; notebook 03 uses `p95_scaled` Q-bandit training reward by default in sample mode and writes RL training/tuning config logs.
- Phase 2/3 gradual sample ladder 2026-05-28: notebook 02 and 03 now support `sample_100k`, `sample_200k`, `sample_300k`, and `full`; all sample modes write tagged outputs and must be compared only against matching sample-mode outputs.
- Five-level evaluation framework adopted 2026-05-28: keep Phase 1/2/2.1/3 as evidence, demote weak pure Q-bandit to ablation, add Phase 3.1 for Level 3/4 dynamic cost-sensitive policies, and add Phase 3.2 for Level 5 LLM-assisted hybrid policy.
- Amount-threshold correction 2026-05-28: do not claim the closed-form expected-cost threshold is amount-dynamic because `TransactionAmt` cancels; use amount-bin validation thresholds for Level 4.
- Phase 3.1 execution 2026-05-29: notebook 02 now exports per-row baseline scores, and notebook 04 implements Level 3 global threshold plus Level 4 amount-bin dynamic threshold; static notebook verification passed, Colab numeric run pending.
- Phase 3.1 Colab path fix 2026-05-29: notebook 04 now supports `PROJECT_ROOT_OVERRIDE`, common Colab project folders, and clearer diagnostics when `baseline_scores_{RUN_MODE}.csv` is missing.
- Phase 3.1 Drive alignment 2026-05-29: notebook 04 now mounts Google Drive and defaults to the same Colab project path used by notebook 03.
- Phase 3.1b planned 2026-05-29: add a tuning phase for Level 4 after first results showed Cost-C success but Cost-A/B over-blocking.
- Phase 3.1b executed 2026-05-29: created `notebooks/04b_dynamic_policy_tuning.ipynb`, generated `phase31b_*_sample_100k` outputs, and improved guarded selector precision/FP-cost trade-off versus Phase 3.1 Level 4.
- Phase 3.2 planned 2026-05-29: Level 5 will use LightGBM score + MiniLM embedding in `notebooks/05_llm_hybrid_policy.ipynb`, compared directly against tuned Level 4.
- Phase 3.2 executed 2026-05-29: created `notebooks/05_llm_hybrid_policy.ipynb`, locally ran `sample_100k` with `tfidf_svd_fallback`, produced `phase32_*` outputs, and verified the claim guard requiring MiniLM for final LLM claims.
- Phase 3.2 Colab MiniLM output verified 2026-05-30: downloaded `phase32_*_sample_100k` files record `embedding_backend_used = minilm`; selected Level 5 v1 matched tuned Level 4 on all three cost settings and did not improve cost.
- Phase 3.3 planned 2026-05-30: add advanced Level 5 v2 in `notebooks/06_llm_augmented_hybrid_v2.ipynb` with train-only prototype similarity, hybrid score fusion, and compact meta-policy candidates.
- Phase 3.3 executed 2026-05-30: created `notebooks/06_llm_augmented_hybrid_v2.ipynb`, added advanced Level 5 v2 candidate families, `phase33_*` output code, disagreement analysis, and claim guard; static notebook verification passed.

## Session Continuity

Last session: 2026-05-28 00:16 +07:00
Latest cleanup: 2026-05-26 - removed old `src/` mirrors, generated EDA figures/results, processed splits, and heavy preprocessing artifacts; kept `.md` files and raw IEEE-CIS CSVs.
Latest planning: 2026-05-26 - Phase 2 planned as notebook-only baseline ML and cost-sensitive evaluation; LLM/table-to-text/RL deferred to Phase 3.
Latest execution: 2026-05-26 - created and smoke-verified `notebooks/02_baselines_cost_metrics.ipynb`; generated baseline metrics and figures on 10,000-row smoke run.
Latest insertion: 2026-05-26 - inserted Phase 2.1 before Phase 3 to add a Kaggle-inspired XGB Magic-style supervised baseline.
Latest Phase 2.1 planning: 2026-05-26 - created context and plan for leakage-safe magic-style supervised baseline.
Latest Phase 2.1 execution: 2026-05-26 - updated `notebooks/02_baselines_cost_metrics.ipynb` with `xgboost_magic_style`, generated `magic_feature_policy.csv`, and smoke-verified the full flow.
Latest Phase 2 finalization: 2026-05-26 - confirmed baseline metrics/threshold outputs, restored Colab raw-data staging in notebook 02, and marked Phase 2/2.1 ready for Phase 3 comparison.
Latest Phase 3 planning: 2026-05-26 - created context, plan, and planning verification for notebook-only table-to-text, MiniLM embeddings, and contextual bandit ablations with smoke-local/full-Colab run strategy.
Latest Phase 3 execution: 2026-05-26 - created and smoke-verified `notebooks/03_llm_representation_and_rl.ipynb`; generated `results/rl_ablation.csv`, Phase 3 metadata, embedding cache, and Phase 3 figures.
Latest Phase 3 upgrade planning: 2026-05-27 - identified SGD implementation as supervised learning, not RL; planned upgrade to Q-value Gym-style Contextual Bandit (TransactionFraudBanditEnv + QBanditPolicy with 2 SGDRegressors + epsilon-greedy). All 3 design decisions confirmed by user: keep SGD as supervised baseline, use Option A (2 SGDRegressor), add phase3_upgrade_summary.csv.
Latest Phase 2/3 matched-mode implementation: 2026-05-27 - implemented non-breaking `sample_100k` option for notebook 02 and notebook 03; existing `smoke` and `full` modes remain.
Latest Phase 3 runtime fix: 2026-05-28 - fixed `NameError: emb_train_raw is not defined` by adding an embedding load/create guard before reduction; verified by local partial smoke execution through the previously failing cell.
Latest Phase 2/3 tuning plan: 2026-05-28 - added plan addenda that frame the contribution as a compute-constrained cost-sensitive RL/LLM ablation pipeline; do not claim superiority over XGB Magic unless matched test results support it.
Latest Phase 2/3 execute update: 2026-05-28 - set notebooks 02 and 03 to `sample_100k` by default, added Phase 3 Q-bandit reward scaling/training logs, cleared stale notebook outputs, and verified syntax plus Q-bandit sanity locally.
Latest gradual sample update: 2026-05-28 - added `sample_200k` and `sample_300k` modes to notebooks 02 and 03; updated planning state and verification docs.
Latest framework update: 2026-05-28 - adopted five-level framework after weak Q-bandit results; added Phase 3.1 amount-bin dynamic cost threshold and Phase 3.2 LLM-assisted hybrid policy; created `PROJECT_PLAN.md` and `TASKLIST.md`.
Latest Phase 3.1 planning: 2026-05-29 - created context, plan, and planning verification for `notebooks/04_dynamic_cost_policy.ipynb`; plan includes Phase 2 per-row score export and Level 3/4 dynamic policy outputs.
Latest Phase 3.1 execution: 2026-05-29 - updated notebook 02 score export and created `notebooks/04_dynamic_cost_policy.ipynb`; static JSON/AST checks passed.
Latest Phase 3.1 fix: 2026-05-29 - hardened notebook 04 Colab project-root and score-file discovery after `/content/results/...` path issue.
Latest Phase 3.1 Drive alignment: 2026-05-29 - updated notebook 04 to mount Google Drive and default to `/content/drive/MyDrive/BaoMatCuoiKy/LLM-Assisted_Cost-Sensitive`.
Latest Phase 3.1b planning: 2026-05-29 - created context, plan, and verification for Level 4 tuning via `notebooks/04b_dynamic_policy_tuning.ipynb`.
Latest Phase 3.1b execution: 2026-05-29 - created and locally verified `notebooks/04b_dynamic_policy_tuning.ipynb`; generated tuned Level 4 outputs for `sample_100k`.
Latest Phase 3.2 execution: 2026-05-30 - verified downloaded Colab MiniLM `sample_100k` outputs for `notebooks/05_llm_hybrid_policy.ipynb`; Level 5 v1 matched tuned Level 4, so Phase 3.3 remains the next improvement path.
Latest Phase 3.3 planning: 2026-05-30 - created context, plan, and verification for `notebooks/06_llm_augmented_hybrid_v2.ipynb`.
Latest Phase 3.3 execution: 2026-05-30 - created and static-verified `notebooks/06_llm_augmented_hybrid_v2.ipynb`; Colab MiniLM numeric run is pending.
Stopped at: Phase 3.3 source complete; next step is run notebook 06 on Colab with MiniLM and download `phase33_*` outputs.
Resume file: `.planning/STATE.md`
