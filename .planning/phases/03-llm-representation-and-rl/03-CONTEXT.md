# Phase 3: LLM Representation and RL - Context

**Gathered:** 2026-05-26
**Status:** Ready for planning
**Source:** User discussion after Phase 2/2.1 progress review

<domain>
## Phase Boundary

Phase 3 builds the proposed model path for the project:

- neutral table-to-text serialization from real IEEE-CIS transaction features;
- local MiniLM sentence embeddings for a controlled subset/full run depending on mode;
- one-step cost-sensitive contextual bandit with actions `0 = approve` and `1 = flag/block`;
- ablation between RL without embedding and RL with local embedding.

This phase does not produce the final report. Phase 4 will consume the saved Phase 2/2.1 and Phase 3 result files for final tables, plots, and error analysis.
</domain>

<decisions>
## Implementation Decisions

### Notebook-first execution
- Phase 3 must be implemented as a primary notebook: `notebooks/03_llm_representation_and_rl.ipynb`.
- Do not recreate `src/` script mirrors unless the user explicitly asks later.
- The notebook must be local-first and Colab-ready, following the style of `01_data_check.ipynb` and `02_baselines_cost_metrics.ipynb`.

### Smoke-first, Colab-full strategy
- Local execution should default to smoke mode to avoid CPU/RAM lag.
- Use a clear config cell:
  - `RUN_MODE = "smoke"` or `"full"`;
  - `SAMPLE_ROWS = 10000` when smoke;
  - `SAMPLE_ROWS = None` when full.
- Smoke results are engineering evidence only and must not be used as final scientific conclusions.
- Full dataset results should be generated on Colab, downloaded, then analyzed in Phase 4.

### Data and split policy
- Reuse IEEE-CIS raw files: `data/raw/train_transaction.csv` and `data/raw/train_identity.csv`.
- Left join identity into transaction using `TransactionID`.
- Use the same `TransactionDT` 70/15/15 split policy as Phase 1 and Phase 2.
- Do not use Kaggle test files.
- Do not fit any preprocessing, PCA, scaler, reducer, or threshold on the test split.

### Table-to-text policy
- Use only real columns from IEEE-CIS.
- Do not include `isFraud`, model scores, baseline predictions, cost labels, or any target-derived information in the text.
- Do not describe `ProductCD` as a real product category; keep it as raw product code.
- Do not include words such as "high risk", "suspicious", or "fraud-like" in the input text.
- Recommended neutral template:
  `A transaction with amount X, product code Y, card fields ..., purchaser email domain D, recipient email domain R, device type T, browser B, and operating system O.`

### Embedding policy
- Prefer `sentence-transformers/all-MiniLM-L6-v2`.
- Cache embeddings by split and run mode.
- Save metadata that proves row counts and split names match the source split.
- If full embedding is too heavy, keep a documented smoke run and prepare the notebook so Colab full run is one config change.

### RL policy
- **Main RL approach: Gym-style Q-value Contextual Bandit (upgraded from SGDClassifier 2026-05-27).**
- `TransactionFraudBanditEnv` (no external gymnasium dependency required):
  - `state s`: preprocessed transaction feature vector (tabular, optionally + embedding).
  - `action a`: `0 = approve`, `1 = flag/block`.
  - `reward`: `R = -[y*(1-a)*C_FN + (1-y)*a*C_FP]` where `C_FN = TransactionAmt*beta`, `C_FP = TransactionAmt*alpha`.
  - Episode: transactions ordered by `TransactionDT`; each call to `step(action)` advances one transaction.
  - Methods: `reset() -> state`, `step(action) -> (next_state, reward, done, info)`.
- `QBanditPolicy` (Linear Q-learning, Option A):
  - Two independent `sklearn.SGDRegressor`: one for `Q(s, approve)`, one for `Q(s, block)`.
  - `gamma = 0` (one-step bandit, no discounting).
  - Training: epsilon-greedy (epsilon decays over epochs).
  - Evaluation: deterministic `argmax_a Q(s, a)`, no exploration.
- Three model variants for ablation:
  - `cost_sensitive_supervised_sgd`: SGDClassifier + sample_weight (kept as **Phase 3 strong supervised baseline**, not RL).
  - `rl_q_bandit_without_embedding`: QBanditPolicy on tabular features only.
  - `rl_q_bandit_with_embedding`: QBanditPolicy on tabular + MiniLM embedding.
- DQN is a stretch goal only; not required for Phase 3 MVP.
- PPO/A2C: out of scope.

### Evaluation policy
- Use the same cost configurations as Phase 2/2.1: `Cost-A`, `Cost-B`, `Cost-C`.
- Report PR-AUC, ROC-AUC if available, fraud recall, fraud precision, fraud F1, FN cost, FP cost, total cost, and cost saving.
- Compare RL variants against approve-all and Phase 2/2.1 ML baselines by saved CSVs, but final synthesis belongs to Phase 4.
</decisions>

<canonical_refs>
## Canonical References

Downstream agents must read these before implementation:

- `.planning/REQUIREMENTS.md` - requirement IDs for LLM and RL scope.
- `.planning/ROADMAP.md` - Phase 3 boundary and Phase 4 handoff.
- `.planning/STATE.md` - current project state and carry-over risks.
- `.planning/phases/01-data-pipeline-and-eda/01-SUMMARY.md` - verified row counts, split policy, and preprocessing lessons.
- `.planning/phases/02-ml-baselines-and-cost-metrics/02-SUMMARY.md` - baseline metric protocol and smoke/full caveats.
- `.planning/phases/02.1-kaggle-inspired-xgb-magic-baseline/02.1-SUMMARY.md` - strong supervised baseline status and wording constraints.
- `notebooks/01_data_check.ipynb` - notebook-first data pipeline pattern.
- `notebooks/02_baselines_cost_metrics.ipynb` - cost metrics, threshold tuning, baseline result format, and figure/output conventions.
- `AGENTS.md` - non-negotiable project guardrails.
</canonical_refs>

<specifics>
## Specific Ideas

- Phase 3 should write `results/rl_ablation.csv`.
- Phase 3 should write `results/phase3_run_metadata.json` or CSV equivalent.
- Phase 3 should write embedding cache files under an ignored directory such as `artifacts/embeddings/`.
- The notebook should include guard assertions for:
  - no label columns in text input;
  - no risk wording in serialized text;
  - train/validation/test `TransactionID` disjointness;
  - embedding rows equal split rows;
  - evaluation loop uses no exploration.
</specifics>

<deferred>
## Deferred Ideas

- Full final error analysis is Phase 4.
- Full report writing is Phase 4.
- DQN is optional only after contextual bandit MVP works.
- API LLM explanations are optional case-study material only, not part of Phase 3 MVP.
- Concept drift, RAG, federated learning, multi-modal learning, phishing/IDS/malware/fake-review tasks remain out of scope.
</deferred>

---

*Phase: 03-llm-representation-and-rl*
*Context gathered: 2026-05-26*
*RL policy upgraded: 2026-05-27 — SGDClassifier → Q-value Gym-style Contextual Bandit (user confirmed)*
