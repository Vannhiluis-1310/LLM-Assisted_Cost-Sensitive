---
phase: 3
name: LLM Representation and RL (Upgraded)
status: in-progress
created: 2026-05-26
updated: 2026-05-28
update_scope: add optional matched sample ladder 100k/200k/300k
requirements_addressed:
  - LLM-01
  - LLM-02
  - LLM-03
  - LLM-04
  - RL-01
  - RL-02
  - RL-03
  - RL-04
  - RL-05
  - RL-06
  - RL-07
primary_artifact: notebooks/03_llm_representation_and_rl.ipynb
execution_mode: notebook-only
waves: 1
---

# Phase 3 Plan: LLM Representation and RL

<objective>
Build a Colab-ready notebook implementing the full LLM-assisted cost-sensitive RL path: neutral table-to-text, local MiniLM embeddings, and a Gym-style Q-value Contextual Bandit (upgraded from SGDClassifier). Three ablation variants: cost_sensitive_supervised_sgd (Phase 3 supervised baseline), rl_q_bandit_without_embedding, rl_q_bandit_with_embedding.
</objective>

## Scope

### MVP

- Create `notebooks/03_llm_representation_and_rl.ipynb`.
- Default to `RUN_MODE = "smoke"` and `SAMPLE_ROWS = 10000`.
- Add `RUN_MODE = "sample_100k"` and `SAMPLE_ROWS = 100000` as the recommended 12GB-RAM Colab starting mode.
- Add `RUN_MODE = "sample_200k"` and `RUN_MODE = "sample_300k"` as gradual RAM ladder modes after 100k completes.
- Keep `RUN_MODE = "full"` available for high-RAM runs; do not remove the current smoke/full code path.
- Load and join IEEE-CIS `train_transaction.csv` and `train_identity.csv` by `TransactionID`.
- Recreate the same `TransactionDT` 70/15/15 split used by Phase 1/2.
- Implement neutral table-to-text serialization from real columns only.
- Generate local embeddings with `sentence-transformers/all-MiniLM-L6-v2`.
- Cache embeddings by split and run mode.
- Implement contextual bandit without embedding.
- Implement contextual bandit with embedding.
- Evaluate both RL variants with the same cost metrics and cost configurations as Phase 2.
- Save machine-readable results for Phase 4.

### Stretch Goal

- Add optional embedding dimensionality reduction fit only on train.
- Add a small neutral case-study table for 20-50 transactions.
- Add DQN only if contextual bandit is already stable and time remains. This is not required for Phase 3 completion.

### Out of Scope

- Fine-tuning any LLM.
- PPO/A2C.
- RAG, federated learning, multi-modal learning.
- API LLM over the full dataset.
- Full FraudSquad/leaderboard reproduction.
- Final report/error-analysis synthesis; that belongs to Phase 4.

## Implementation Tasks

| ID | Task | Details | Output |
|---|---|---|---|
| 3.1 | Notebook skeleton and config | Add setup, local/Colab path handling, `RUN_MODE`, `SAMPLE_ROWS`, `RANDOM_STATE`, `N_JOBS`, output dirs. | `notebooks/03_llm_representation_and_rl.ipynb` |
| 3.1a | Matched sample run modes | Add `RUN_MODE = "sample_100k"`, `RUN_MODE = "sample_200k"`, and `RUN_MODE = "sample_300k"` with matching `SAMPLE_ROWS`, `N_JOBS = 1`, `EMBEDDING_BATCH_SIZE = 32`, `RL_N_EPOCHS = 3`, and `RUN_OUTPUT_TAG = RUN_MODE`. Keep existing `smoke` and `full` modes unchanged. | sample ladder config |
| 3.2 | Data load and split | Load raw CSVs, left join by `TransactionID`, assert joined rows equal transaction rows, sort/split by `TransactionDT`. | split dataframes and split summary |
| 3.3 | Feature policy | Exclude `TransactionID`, `isFraud`, raw `TransactionDT` from model features; keep `TransactionAmt` separately for reward/cost. | feature policy table |
| 3.4 | Table-to-text serialization | Serialize a curated set of real fields: amount, product code, card fields, addresses, email domains, selected `id_*`, `DeviceType`, `DeviceInfo`. | serialized text columns |
| 3.5 | Text leakage audit | Assert serialized text does not include label names, model scores, risk wording, or interpreted `ProductCD`. | leakage audit table |
| 3.6 | MiniLM embedding | Install/import `sentence-transformers` when needed, encode texts in batches, cache per split/run mode. | `artifacts/embeddings/*` |
| 3.7 | Tabular state preprocessing | Build light tabular features for RL no-embedding using train-only imputation/encoding/scaling. Avoid huge full one-hot artifacts. | train/validation/test state matrices |
| 3.8 | Gym-style environment and Q-value policy | Implement `TransactionFraudBanditEnv`: `reset() -> state`, `step(action) -> (next_state, reward, done, info)`. Implement `QBanditPolicy` with two `sklearn.SGDRegressor` for `Q(s,0)` and `Q(s,1)`, `gamma=0`, epsilon-greedy training, deterministic eval. Keep `cost_sensitive_supervised_sgd` (SGDClassifier + sample_weight) as Phase 3 supervised baseline; rename from old `rl_without_embedding` variant. | `TransactionFraudBanditEnv`, `QBanditPolicy` classes |
| 3.9 | Reward and cost configs | Implement `R = -[y*(1-a)*C_FN + (1-y)*a*C_FP]`, `C_FN = TransactionAmt*beta`, `C_FP = TransactionAmt*alpha`. Ensure reward is computed inside `env.step()` and logged in `info`. | reward functions inside env |
| 3.10 | Training loops | For each variant, run epsilon-greedy training loop: `env.reset()`, iterate transactions by `TransactionDT` order, `policy.act(state, epsilon)`, `env.step(action)`, `policy.update(state, action, reward)`. Log cumulative reward and epsilon per epoch. For supervised SGD baseline, fit as before with sample_weight. Validation threshold tuning per cost config. | trained policy checkpoints |
| 3.11 | Evaluation loops | Evaluate on validation/test with no exploration. Compute PR-AUC, ROC-AUC, fraud recall/precision/F1, FN/FP/Total Cost, Cost Saving. | `results/rl_ablation.csv` |
| 3.12 | Baseline comparison hook | Load Phase 2/2.1 `results/baseline_metrics.csv` when present and prepare comparison tables without rewriting Phase 2. | comparison preview |
| 3.13 | Figures | Generate lightweight PR curve, cost comparison bar chart, confusion matrices for RL variants. | `reports/figures/phase3_*` |
| 3.14 | Metadata and handoff | Save run mode, sample rows, split counts, package availability, embedding model, cost configs, and runtime notes. | `results/phase3_run_metadata.*` |

## Technical Design

### Run Configuration

```python
RUN_MODE = "smoke"  # "smoke", "sample_100k", "sample_200k", "sample_300k", or "full"
RUN_MODE_SAMPLE_ROWS = {
    "smoke": 10_000,
    "sample_100k": 100_000,
    "sample_200k": 200_000,
    "sample_300k": 300_000,
    "full": None,
}
SAMPLE_ROWS = RUN_MODE_SAMPLE_ROWS[RUN_MODE]
RANDOM_STATE = 42
EMBEDDING_MODEL_NAME = "sentence-transformers/all-MiniLM-L6-v2"
```

The notebook must clearly print whether a run is `smoke`, one of the sample modes, or `full`. Every exported result must include `run_mode` and `sample_rows`.

Recommended 12GB RAM ladder:

```python
RUN_MODE = "sample_100k"
N_JOBS = 1
EMBEDDING_BATCH_SIZE = 32
RL_N_EPOCHS = 3
EMBEDDING_REDUCTION_COMPONENTS = 64
```

Start with `sample_100k`. If it completes, rerun both Phase 2 and Phase 3 with `sample_200k`, then `sample_300k`. The sample modes are not replacements for full mode; they are explicitly labeled compute-constrained run paths.

### Table-to-Text Template

Use neutral language only. Example:

```text
A transaction with amount {TransactionAmt}, product code {ProductCD}, card fields {card1}, {card2}, {card3}, {card4}, {card5}, {card6}, purchaser email domain {P_emaildomain}, recipient email domain {R_emaildomain}, device type {DeviceType}, and device info {DeviceInfo}.
```

Rules:

- Missing values become `unknown`.
- Do not include `isFraud`.
- Do not include baseline predictions or model scores.
- Do not include risk adjectives.
- Do not translate `ProductCD` into a product category.

### Embedding Cache

Suggested files:

| Split | File |
|---|---|
| Train | `artifacts/embeddings/{run_mode}_train_minilm.npy` |
| Validation | `artifacts/embeddings/{run_mode}_validation_minilm.npy` |
| Test | `artifacts/embeddings/{run_mode}_test_minilm.npy` |
| Metadata | `artifacts/embeddings/{run_mode}_embedding_metadata.csv` |

Metadata must include split, row count, embedding dimension, source row hash/checksum or `TransactionID` count, and model name.

For `RUN_MODE = "sample_100k"`, expected split rows are:

| Split | Rows |
|---|---:|
| Train | 70,000 |
| Validation | 15,000 |
| Test | 15,000 |

This mode must use the same first `100_000` raw `train_transaction.csv` rows and the same `TransactionDT` 70/15/15 split policy as Phase 2 `sample_100k`, so Phase 2 and Phase 3 comparisons are sample-matched.

### Contextual Bandit

Use one-step contextual bandit framing:

- State `s`: tabular features, optionally concatenated with MiniLM embedding.
- Action `a`:
  - `0 = approve`
  - `1 = flag/block`
- Label `y`:
  - `1 = fraud`
  - `0 = legitimate`
- Reward:

```text
R = - [ y * (1 - a) * C_FN + (1 - y) * a * C_FP ]
C_FN = TransactionAmt * beta
C_FP = TransactionAmt * alpha
```

Recommended cost configurations:

| Config | alpha | beta | Meaning |
|---|---:|---:|---|
| Cost-A | 0.05 | 1.00 | Moderate false-positive cost, base false-negative cost |
| Cost-B | 0.10 | 2.00 | Higher fraud loss pressure |
| Cost-C | 0.20 | 5.00 | Very high false-negative penalty |

Policy implementation should stay simple:

- MVP option: train a linear policy/classifier using sample weights derived from `TransactionAmt` and cost config, then treat score threshold as the bandit policy.
- Evaluation must be deterministic: no epsilon exploration on validation/test.
- Save chosen thresholds/policy parameters per cost config.

## Verification Plan

| Check | Expected Result |
|---|---|
| Notebook JSON valid | `03_llm_representation_and_rl.ipynb` opens and parses |
| Smoke run | `RUN_MODE="smoke"` completes on local sample |
| Join integrity | joined row count equals transaction row count |
| Split policy | train/validation/test are sorted by `TransactionDT` and disjoint by `TransactionID` |
| Text leakage audit | no `isFraud`, label, risk wording, model score, or ProductCD interpretation in serialized text |
| Embedding cache integrity | cached rows equal split rows and metadata records model name/dim |
| Env reset | `env.reset()` returns a valid state vector without error |
| Env step | `env.step(0)` and `env.step(1)` return `(next_state, reward, done, info)` with correct reward sign |
| RL action space | only actions 0 and 1 appear |
| Q-value divergence | `Q(s,0) != Q(s,1)` for at least some states after training (policy learned something) |
| Epsilon-greedy flag | `exploration=True` during train, `exploration=False` (or `none_in_evaluation`) during eval |
| Reward function | FN cost uses beta and is heavier than FP in selected configs |
| Model names | `cost_sensitive_supervised_sgd`, `rl_q_bandit_without_embedding`, `rl_q_bandit_with_embedding` in CSV |
| Evaluation loop | no exploration during validation/test |
| Ablation outputs | all 3 variants present in result CSV |
| Metrics | PR-AUC, recall, precision, F1, FN cost, FP cost, total cost, cost saving present |
| Upgrade summary | `results/phase3_upgrade_summary.csv` present with 3 variants + Phase 2 baselines |
| 100k mode | `RUN_MODE="sample_100k"` produces 100,000 joined rows split 70,000/15,000/15,000 |
| 200k/300k modes | `RUN_MODE="sample_200k"` and `RUN_MODE="sample_300k"` preserve the same split policy and write tagged outputs |
| Matched baseline lookup | In each sample mode, Phase 3 prefers `results/baseline_metrics_{RUN_MODE}.csv` over `results/baseline_metrics.csv`; if matched baseline is missing, comparison output must clearly label the result as mixed-scope |
| Scope guard | no fine-tuning, PPO/A2C, RAG, multimodal, federated learning, API LLM full-dataset |

## Deliverables

| Artifact | Required | Purpose |
|---|---:|---|
| `notebooks/03_llm_representation_and_rl.ipynb` | Yes | Main Phase 3 implementation (upgraded) |
| `results/rl_ablation.csv` | Yes | All 3 variant metrics (supervised SGD, Q-bandit no-embed, Q-bandit embed) |
| `results/phase3_upgrade_summary.csv` | Yes | Consolidated 3-variant + Phase 2 baseline comparison for Phase 4 |
| `results/phase3_run_metadata.csv` or `.json` | Yes | Reproducibility/run-mode evidence |
| `artifacts/embeddings/*` | Yes, generated ignored artifact | Cached local embeddings |
| `reports/figures/phase3_*.png` | Yes | Comparison figures (updated for 3 variants) |
| `.planning/phases/03-llm-representation-and-rl/03-SUMMARY.md` | During execute | Phase execution summary (upgrade edition) |
| `.planning/phases/03-llm-representation-and-rl/03-VERIFICATION.md` | During execute or verify | Verification evidence (upgrade edition) |

## Done Criteria

- `notebooks/03_llm_representation_and_rl.ipynb` exists and runs in smoke mode.
- `results/rl_ablation.csv` contains all three variants:
  - `cost_sensitive_supervised_sgd` (supervised baseline)
  - `rl_q_bandit_without_embedding` (Q-value bandit, tabular only)
  - `rl_q_bandit_with_embedding` (Q-value bandit, tabular + embedding)
- All three variants evaluated on `Cost-A`, `Cost-B`, `Cost-C`.
- `TransactionFraudBanditEnv` implements `reset()` and `step(action)` correctly.
- `QBanditPolicy` uses two `SGDRegressor` with `gamma=0`; epsilon-greedy in train, deterministic in eval.
- Embedding and RL outputs tied to same `TransactionDT` split as Phase 2.
- Text serialization passes leakage/risk-wording checks.
- The notebook clearly labels smoke results as smoke-only.
- The notebook clearly labels sample-mode results as sample results, not full-data results.
- `results/phase3_upgrade_summary.csv` present with all 3 variants + Phase 2 baselines for Phase 4 consumption.
- A sample-mode output is comparable only with the same Phase 2 sample mode unless the summary explicitly marks it as mixed-scope.
- No Phase 3 code uses forbidden out-of-scope methods.

<threat_model>
## Threat Model

| Threat | Severity | Mitigation |
|---|---|---|
| Label leakage into text or state | High | Explicit forbidden columns, text audit, no model scores in serialization |
| Split mismatch between baseline, embedding, and RL | High | Recreate split by `TransactionDT`, assert disjoint `TransactionID`, save split metadata |
| Test leakage through threshold/PCA/scaler fitting | High | Fit only on train or validation as appropriate; test is report-only |
| Smoke metrics mistaken for final metrics | Medium | Add `run_mode` to all outputs and print smoke warnings |
| 100k sample mistaken for full-data result | Medium | Add `run_mode=sample_100k`, `sample_rows=100000`, and report wording that this is compute-constrained sample evaluation |
| Phase 2 full results mixed with Phase 3 sample results | High | Prefer `baseline_metrics_sample_100k.csv`; if absent, mark comparison as mixed-scope and do not use it for final superiority claims |
| Colab/local environment drift | Medium | Save package/model metadata and deterministic seed |
| Oversized artifacts causing local lag | Medium | Smoke default, batch embedding, ignored generated artifacts |
</threat_model>

## Handoff to Phase 4

Phase 4 should consume:

- full-data Phase 2/2.1 baseline metrics from Colab;
- Phase 3 `sample_100k` metrics if 12GB RAM cannot support full Phase 3;
- Phase 2 `sample_100k` metrics for fair comparison with Phase 3 `sample_100k`;
- full-data Phase 3 RL ablation metrics from Colab only if high-RAM runtime is available;
- smoke results only as engineering verification evidence;
- saved metadata to prove all comparisons use the same split and cost configurations.

## 2026-05-28 Addendum: 100k-First RL Tuning Plan

### Assessment

The proposed contribution is valid if it is framed precisely:

> A compute-constrained, cost-sensitive experimental pipeline that evaluates whether local LLM embeddings improve a simple RL decision policy under imbalanced e-commerce fraud data.

The current weak point is not the dataset choice. It is that the linear Q-bandit can have too little model capacity and unstable reward targets. This shows up as very low fraud precision, many false positives, high FP cost, and weak or negative cost saving.

Therefore, the next work should not claim that LLM embeddings improve fraud detection yet. It should execute a controlled 100k experiment first, then tune reward/training only on validation.

### Priority Order

| Priority | Action | Why | Expected Output |
|---:|---|---|---|
| 1 | Run Phase 3 with `RUN_MODE = "sample_100k"` | Reduces smoke variance and gives embeddings enough examples to matter | `rl_ablation_sample_100k.csv` |
| 2 | Use Phase 2 `baseline_metrics_sample_100k.csv` | Makes baseline-vs-RL comparison fair | `phase3_upgrade_summary_sample_100k.csv` with matched scope |
| 3 | Tune reward target scaling on validation | Linear Q targets can be dominated by large `TransactionAmt` outliers | reward sensitivity table |
| 4 | Tune epsilon decay, learning rate, and epochs | Current exploration/training may not discover a useful flag policy | hyperparameter sensitivity table |
| 5 | Add leakage-safe magic-lite state features if needed | Improves state representation without changing the LLM/RL scope | RL no-embed and with-embed rerun on same state |
| 6 | Try full dataset only after 100k is stable | Full run is expensive and should not be the debugging surface | full-run metadata, if RAM allows |

### Reward Sensitivity Plan

Keep the business cost metric unchanged:

```text
FN Cost = sum(TransactionAmt * beta) for fraud approved
FP Cost = sum(TransactionAmt * alpha) for legitimate flagged
Total Cost = FN Cost + FP Cost
```

Only the training target for Q-value regression may be scaled for stability. Evaluation must always use raw business cost.

| Variant | Training target | Purpose |
|---|---|---|
| `reward_raw` | `reward = -raw_cost` | Current baseline; fully interpretable |
| `reward_p95_scaled` | `reward = -min(raw_cost, p95_train_cost) / p95_train_cost` | Reduces domination by extreme transaction amounts |
| `reward_log1p` | `reward = -log1p(raw_cost)` for nonzero costs | Compresses large costs while preserving order |

Required guardrail: all scaling constants must be fit on train only.

### Alpha/Beta Sensitivity

Keep existing required configs:

| Config | alpha | beta |
|---|---:|---:|
| Cost-A | 0.05 | 1.0 |
| Cost-B | 0.10 | 2.0 |
| Cost-C | 0.20 | 5.0 |

Optional tuning configs may be added only as sensitivity analysis, not as replacement:

| Config | alpha | beta | Use |
|---|---:|---:|---|
| Cost-D | 0.05 | 5.0 | Stronger FN pressure with low FP penalty |
| Cost-E | 0.10 | 10.0 | Very high FN pressure stress test |

Do not select the final model based on test-set Total Cost. Use validation for selection, then report test once.

### Hyperparameter Grid

Use a small grid to avoid turning the project into hyperparameter search:

| Parameter | Values |
|---|---|
| `RL_LR` | `0.001`, `0.005`, `0.01` |
| `RL_N_EPOCHS` | `3`, `5` |
| `RL_EPSILON_START` | `0.20`, `0.30`, `0.50` |
| `RL_EPSILON_END` | `0.02`, `0.05` |
| `SHUFFLE_TRAIN_EACH_EPOCH` | `False`, `True` |

Evaluation remains deterministic with no exploration.

### State Representation Tuning

If 100k Q-bandit precision remains near random, add a controlled `magic_lite` state option inspired by Phase 2.1:

- train-only count/frequency encoding for `card1`, `card2`, `addr1`, `P_emaildomain`, `R_emaildomain`, `card1_addr1`;
- train-only `TransactionAmt` group stats for `card1` and `card1_addr1`;
- no label statistics;
- no model scores;
- no Kaggle test files or UID post-processing.

Run both RL variants on the same state representation:

| Variant | State |
|---|---|
| RL without embedding | tabular basic or magic-lite |
| RL with embedding | same tabular state + MiniLM/local embedding |

This preserves the LLM ablation: the only difference between the two RL variants remains the embedding.

### Decision Rules

| Outcome | Interpretation | Next Action |
|---|---|---|
| RL no-embed and RL with-embed both bad | RL policy/training is weak; tune reward/hyperparams before discussing LLM |
| RL with-embed improves PR-AUC/Recall but FP cost explodes | Embedding may increase sensitivity but needs threshold/reward tuning |
| RL with-embed reduces Total Cost vs RL no-embed but not vs XGB Magic | Claim embedding helps RL ablation, not that it beats supervised SOTA |
| XGB Magic remains best | Present it honestly as strongest supervised baseline; position RL as cost-sensitive decision framework |
| `comparison_scope` is mixed | Do not make comparison claims; rerun Phase 2 sample_100k |

### Acceptance Criteria

1. Phase 3 `sample_100k` completes and writes tagged result files.
2. Phase 3 summary uses Phase 2 `baseline_metrics_sample_100k.csv`; otherwise it labels mixed scope.
3. Reward/hyperparameter tuning, if implemented, writes a validation-selection table.
4. Final test metrics are reported once per selected configuration.
5. The report does not claim LLM/RL beats XGB Magic unless the matched test results support it.
6. The report can still claim a valid contribution if LLM/RL does not beat XGB Magic: the contribution is a reproducible cost-sensitive RL/LLM ablation pipeline under imbalanced fraud detection.

### Execute Update 2026-05-28

Notebook 03 now defaults to `RUN_MODE = "sample_100k"` for the current Colab execution plan. The Q-bandit training path implements the planned reward-stability scaffold:

- `REWARD_SCALE_MODE = "p95_scaled"` in `sample_100k`;
- business-cost evaluation still uses raw `TransactionAmt * alpha/beta`;
- `RL_LR = 0.005` in `sample_100k`;
- `Q_BANDIT_TUNING_GRID` is recorded but not executed by default;
- tagged `rl_training_log_*` and `rl_tuning_config_*` outputs are generated.

### Gradual Sample Ladder Update 2026-05-28

Notebook 03 now supports the same RAM-aware run ladder as notebook 02:

| Order | `RUN_MODE` | Rows | Phase 3 behavior |
|---:|---|---:|---|
| 1 | `sample_100k` | 100,000 | Default 12GB Colab starting point |
| 2 | `sample_200k` | 200,000 | Larger matched sample after 100k succeeds |
| 3 | `sample_300k` | 300,000 | Largest sample-mode run before full |
| 4 | `full` | all rows | Use only if RAM/runtime allow |

All `sample_*` modes use the RAM-safe settings `N_JOBS = 1`, `EMBEDDING_BATCH_SIZE = 32`, `RL_N_EPOCHS = 3`, `RL_LR = 0.005`, and `REWARD_SCALE_MODE = "p95_scaled"`. Business-cost evaluation remains raw and comparable with Phase 2.
