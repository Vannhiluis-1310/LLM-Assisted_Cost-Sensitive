---
phase: 3
status: complete
completed_smoke: 2026-05-26
upgrade_planned: 2026-05-27
run_mode: smoke
---

# Phase 3 Summary: LLM Representation and RL

## Phase 3 Upgrade — Q-value Gym-style Contextual Bandit (2026-05-27)

**Status:** In Progress — notebook upgrade pending, planning files updated.

**Reason for upgrade:** The original smoke implementation used `cost_sensitive_sgd_contextual_bandit` (SGDClassifier + sample_weight), which is supervised learning with cost-sensitive loss, not RL. To be defensible before the evaluation committee, the RL component is upgraded to a proper **Gym-style Q-value Contextual Bandit**.

**Design decisions confirmed by user (2026-05-27):**
- Keep SGD path as `cost_sensitive_supervised_sgd` — positioned as **Phase 3 strong supervised baseline** (not RL main), enabling ablation: Supervised → Q-Bandit → LLM-assisted Q-Bandit.
- Q-value implementation: **Option A** — 2 independent `sklearn.SGDRegressor` (one for `Q(s,0)`, one for `Q(s,1)`), `gamma=0`, epsilon-greedy training, deterministic evaluation.
- Add `results/phase3_upgrade_summary.csv` consolidating all 3 variants + Phase 2 baselines for Phase 4 consumption.

**New model variants:**
| Model | Type | Description |
|---|---|---|
| `cost_sensitive_supervised_sgd` | Supervised baseline | SGDClassifier + sample_weight (renamed from old rl_without/with_embedding SGD path) |
| `rl_q_bandit_without_embedding` | RL main — no embed | QBanditPolicy (2 SGDRegressor) on tabular features |
| `rl_q_bandit_with_embedding` | RL main — with LLM | QBanditPolicy on tabular + MiniLM embedding |

---

## Original Smoke Outcome (2026-05-26)

Phase 3 MVP completed in smoke mode (2026-05-26). The notebook has since been upgraded to separate `cost_sensitive_supervised_sgd` from the Q-value contextual bandit variants.

## Delivered

| Item | Status | Evidence |
|---|---|---|
| Notebook skeleton/config | Complete | `RUN_MODE`, `SAMPLE_ROWS`, `RANDOM_STATE`, cost configs, output dirs |
| IEEE-CIS load/join/split | Complete | `train_transaction.csv` + `train_identity.csv`, left join by `TransactionID`, `TransactionDT` split |
| Neutral table-to-text | Complete | Transaction amount, product code, card/address/email/device/identity fields |
| Text leakage audit | Complete | `results/phase3_text_leakage_audit.csv` |
| Embedding cache | Complete | `artifacts/embeddings/smoke_tfidf_svd_fallback_*.npy` |
| MiniLM implementation path | Complete | Notebook targets `sentence-transformers/all-MiniLM-L6-v2`; dependency added to `requirements.txt` |
| RL no embedding | Complete | `rl_q_bandit_without_embedding` in `results/rl_ablation.csv` |
| RL with embedding | Complete | `rl_q_bandit_with_embedding` in `results/rl_ablation.csv` |
| Cost-sensitive reward/eval | Complete | `Cost-A`, `Cost-B`, `Cost-C`; FN/FP/Total Cost and Cost Saving |
| Figures | Complete | `reports/figures/phase3_*.png` |
| Run metadata | Complete | `results/phase3_run_metadata.json` |

## Smoke Verification

| Check | Result |
|---|---|
| Notebook JSON | PASS, 34 cells after upgrade |
| Code syntax | PASS, no bad code cells |
| Missing cell IDs | PASS, none |
| Local execution | PASS via Jupyter nbconvert with workspace runtime |
| Models in output | `cost_sensitive_supervised_sgd`, `rl_q_bandit_without_embedding`, `rl_q_bandit_with_embedding` |
| Output rows | upgraded smoke result has 18 rows across 3 models, 2 splits, and 3 cost configs |
| Cost configs | `Cost-A`, `Cost-B`, `Cost-C` |
| Splits | `validation`, `test` |
| Local embedding backend | `tfidf_svd_fallback` because `sentence-transformers` is absent locally |

## Key Files

| File | Purpose |
|---|---|
| `notebooks/03_llm_representation_and_rl.ipynb` | Main Phase 3 notebook |
| `requirements.txt` | Adds `sentence-transformers` for MiniLM Colab/full run |
| `results/rl_ablation.csv` | Main RL ablation metrics |
| `results/rl_threshold_tuning.csv` | Validation threshold search for RL policies |
| `results/phase3_run_metadata.json` | Reproducibility and smoke/full metadata |
| `results/phase3_text_leakage_audit.csv` | Evidence that serialized text passed leakage/risk-wording audit |
| `artifacts/embeddings/*` | Generated embedding cache, ignored by Git |
| `reports/figures/phase3_*.png` | Generated Phase 3 smoke figures |

## Handoff to Colab Run

Update 2026-05-27/28: notebook 02 and notebook 03 now support matched sample modes for 12GB RAM. Start with `RUN_MODE="sample_100k"`, then increase to `sample_200k`, then `sample_300k` if the previous run completes.

Before Phase 4 sample analysis:

1. In notebook 02, set `RUN_MODE = "sample_100k"` and run it first.
2. Confirm `results/baseline_metrics_sample_100k.csv` exists.
3. In notebook 03, set `RUN_MODE = "sample_100k"` and run it second.
4. Confirm `results/phase3_upgrade_summary_sample_100k.csv` has `comparison_scope` equal to `matched_run_mode` or `matched_run_mode_canonical`.
5. If stable, repeat the same paired order for `sample_200k`, then `sample_300k`.
6. Confirm `results/phase3_run_metadata_{RUN_MODE}.json` shows `embedding_backend_used = "minilm"` if Colab can install sentence-transformers.

## Remaining Risks

- Smoke metrics are not final report metrics.
- MiniLM was not executed locally because the package is not installed; this must be verified on Colab.
- Full-data embedding may be time-consuming; use Colab runtime and cached embeddings.

## 2026-05-27 Matched 100k Update

Notebook 03 now supports `RUN_MODE = "sample_100k"` with `SAMPLE_ROWS = 100_000`, `N_JOBS = 1`, `EMBEDDING_BATCH_SIZE = 32`, and `RL_N_EPOCHS = 3`. It writes tagged outputs such as `rl_ablation_sample_100k.csv`, `phase3_upgrade_summary_sample_100k.csv`, and `phase3_run_metadata_sample_100k.json`. In `sample_100k`, it prefers `results/baseline_metrics_sample_100k.csv`; if missing, the comparison is marked `mixed_scope_canonical` and must not be used for superiority claims.

## 2026-05-28 Execute Update

Notebook 03 is now set to `RUN_MODE = "sample_100k"` by default for the next Colab run. The Q-bandit training path also includes the planned stability scaffold:

- `REWARD_SCALE_MODE = "p95_scaled"` by default in `sample_100k`;
- raw business-cost evaluation remains unchanged;
- `RL_LR = 0.005` in `sample_100k`;
- optional `Q_BANDIT_TUNING_GRID` is recorded but not run by default;
- `rl_training_log_{RUN_OUTPUT_TAG}.csv` and `rl_tuning_config_{RUN_OUTPUT_TAG}.csv` are written for Phase 4 analysis.

This implements the 100k-first path and prepares validation-only tuning evidence without turning the notebook into a heavy grid search by default.

## 2026-05-28 Gradual Sample Ladder Update

Notebook 03 now also supports `RUN_MODE = "sample_200k"` and `RUN_MODE = "sample_300k"`. These modes keep the same Phase 3 pipeline:

- neutral table-to-text from real IEEE-CIS fields;
- MiniLM/local embedding cache keyed by run mode;
- train-only tabular preprocessing and embedding reduction;
- Q-bandit without embedding;
- Q-bandit with embedding;
- raw business-cost evaluation.

All `sample_*` modes use the RAM-safe defaults: `N_JOBS = 1`, `EMBEDDING_BATCH_SIZE = 32`, `RL_N_EPOCHS = 3`, `RL_LR = 0.005`, and `REWARD_SCALE_MODE = "p95_scaled"`.

The comparison rule is strict: Phase 3 `sample_200k` should be compared only with Phase 2 `sample_200k`; Phase 3 `sample_300k` should be compared only with Phase 2 `sample_300k`.
