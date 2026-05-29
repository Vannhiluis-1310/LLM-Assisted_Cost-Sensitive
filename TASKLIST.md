# Tasklist: Updated Five-Level Evaluation Framework

## Status Legend

- `[x]` Done
- `[/]` In progress / partially done
- `[ ]` Pending

## Completed Work Preserved

- [x] Phase 1 data foundation: IEEE-CIS load, left join, fraud rate, EDA, time split, preprocessing.
- [x] Phase 2 baseline notebook: approve-all, Logistic Regression, tree baseline, cost metrics, threshold tuning.
- [x] Phase 2.1 strong supervised baseline: leakage-safe magic-style features and XGBoost/LightGBM-style comparison.
- [x] Phase 3 LLM/RL ablation notebook: neutral table-to-text, MiniLM/local embedding path, Q-bandit without embedding, Q-bandit with embedding.
- [x] Sample-run evidence that pure Q-bandit is weak compared with LightGBM on business cost.

## Planning Updates

- [x] Update `.planning/ROADMAP.md` with Phase 3.1 and Phase 3.2.
- [x] Update `.planning/REQUIREMENTS.md` with policy requirements and leakage rules.
- [x] Create `PROJECT_PLAN.md` with the five-level framework.
- [x] Create `TASKLIST.md` with next implementation steps.
- [x] Update `.planning/STATE.md` to record the pivot without restarting the project.

## Phase 3.1: Dynamic Cost-Sensitive Decision Policies

- [x] Plan Phase 3.1 in `.planning/phases/03.1-dynamic-cost-sensitive-decision-policies/03.1-PLAN.md`.
- [x] Create `notebooks/04_dynamic_cost_policy.ipynb` as the clean Level 3/4 notebook.
- [x] Add Phase 2 score export code for `results/baseline_scores_{RUN_MODE}.csv`.
- [x] Implement source code to load Phase 2 scores and metrics for the same `RUN_MODE`.
- [x] Implement Level 3: LightGBM + one global cost-sensitive threshold tuned on validation.
- [x] Implement amount-bin creation for `TransactionAmt`.
- [x] Ensure amount-bin boundaries are derived from train/validation only.
- [x] Implement Level 4: one validation-tuned threshold per amount bin.
- [x] Freeze amount-bin thresholds before test evaluation in notebook source.
- [x] Implement Level 3/4 evaluation on test using PR-AUC, Recall, Precision, F1, FN Cost, FP Cost, Total Cost, and Cost Saving.
- [x] Add save code for `results/dynamic_policy_metrics_{RUN_MODE}.csv`.
- [x] Add save code for `results/amount_bin_thresholds_{RUN_MODE}.csv`.
- [x] Add save code for `results/five_level_comparison_{RUN_MODE}.csv`.
- [x] Add figure code for amount-bin thresholds and Total Cost by level.
- [ ] Run notebook 02 on Colab to generate `results/baseline_scores_{RUN_MODE}.csv`.
- [ ] Run notebook 04 on Colab to generate Phase 3.1 result CSVs and figures.
- [ ] Download Colab-generated `results/` and `reports/figures/` back to local for Phase 4 analysis.

## Phase 3.1b: Level 4 Dynamic Policy Tuning

- [x] Plan Phase 3.1b in `.planning/phases/03.1b-level4-dynamic-policy-tuning/03.1b-PLAN.md`.
- [x] Create `notebooks/04b_dynamic_policy_tuning.ipynb`.
- [x] Load `baseline_scores_{RUN_MODE}.csv`, `dynamic_policy_metrics_{RUN_MODE}.csv`, `global_cost_thresholds_{RUN_MODE}.csv`, and `amount_bin_thresholds_{RUN_MODE}.csv`.
- [x] Reconstruct Phase 3.1 Level 3/4 validation and test predictions as reference rows.
- [x] Implement `level4_shrunk_amount_bin_threshold`.
- [x] Implement `level4_precision_guard_threshold` or FP-cost cap candidate.
- [x] Implement alternate amount-bin strategy grid.
- [x] Keep modified expected-cost policy diagnostic-only if added.
- [x] Select best candidate per cost config using validation only.
- [x] Apply frozen selected candidates to test.
- [x] Save `results/phase31b_candidate_metrics_{RUN_MODE}.csv`.
- [x] Save `results/phase31b_candidate_thresholds_{RUN_MODE}.csv`.
- [x] Save `results/phase31b_selected_policies_{RUN_MODE}.csv`.
- [x] Save `results/five_level_comparison_tuned_{RUN_MODE}.csv`.
- [x] Add Phase 3.1b figures for Total Cost, FP Cost/precision trade-off, and Cost Saving.
- [x] Locally verify Phase 3.1b on `sample_100k` outputs.

## Phase 3.2: LLM-Assisted Hybrid Policy

- [x] Plan Phase 3.2 in `.planning/phases/03.2-llm-assisted-hybrid-policy/03.2-PLAN.md`.
- [x] Confirm tuned Level 4 is stable on `sample_100k` before implementing Level 5.
- [x] Create `notebooks/05_llm_hybrid_policy.ipynb`.
- [x] Load tuned Level 4 comparator from Phase 3.1b.
- [x] Reuse MiniLM embeddings if available or regenerate neutral table-to-text embeddings.
- [x] Run text leakage audit for Level 5 serializer.
- [x] Implement `level5_embedding_similarity_threshold_adjustment`.
- [x] Implement `level5_hybrid_score_threshold`.
- [ ] Stretch: optionally implement `level5_meta_policy_small`.
- [x] Tune Level 5 candidates using validation only.
- [x] Compare Level 5 directly against tuned Level 4 on the same split and run mode.
- [x] Save Level 5 rows into `results/five_level_comparison_level5_{RUN_MODE}.csv`.
- [x] If Level 5 does not beat Level 4, document it as an honest ablation outcome.
- [ ] Run notebook 05 on Colab with `embedding_backend_used = "minilm"` for final LLM claim.

## Final Evaluation and Report

- [ ] Build final result table covering Level 1, Level 2, Level 3, Level 4, Level 5, and Q-bandit ablations.
- [ ] Include Cost-A, Cost-B, and Cost-C sensitivity analysis.
- [ ] Include class imbalance and `TransactionAmt` analysis from Phase 1.
- [ ] Include PR curve, cost-vs-threshold, amount-bin threshold chart, confusion matrix, and Total Cost chart.
- [ ] Perform error analysis for high-cost false negatives.
- [ ] Perform error analysis for high-cost false positives.
- [ ] Compare Level 3 vs Level 4 disagreements.
- [ ] Compare Level 4 vs Level 5 disagreements.
- [ ] Compare supervised baseline vs Q-bandit disagreements.
- [ ] Explain why Accuracy is unsuitable.
- [ ] State clearly that FraudSquad/1st place is an external reference, not a reproduced baseline.

## Guardrails

- [ ] Do not restart Phase 1/2.
- [ ] Do not claim a closed-form expected-cost threshold is amount-dynamic.
- [ ] Do not tune on test.
- [ ] Do not fine-tune LLMs.
- [ ] Do not make PPO/A2C required.
- [ ] Do not reproduce full Kaggle SOTA unless explicitly scoped later.

---

Last updated: 2026-05-29.
