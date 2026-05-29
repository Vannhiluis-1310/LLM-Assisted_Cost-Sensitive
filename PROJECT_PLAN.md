# Project Plan: Five-Level Cost-Sensitive Fraud Decision Framework

## Project Title

**Vietnamese:** Khung ra quyet dinh nhay cam chi phi dong cho phat hien gian lan giao dich thuong mai dien tu mat can bang voi su tang cuong bieu dien LLM.

**English:** Dynamic Cost-Sensitive Decision Framework for Imbalanced E-commerce Transaction Fraud Detection with LLM Representation Enhancement.

## Current Decision

The project is not restarted. Existing Phase 1, Phase 2, Phase 2.1, and Phase 3 work is preserved as evidence.

Current sample results show:

- Strong supervised models such as LightGBM are better than pure Q-bandit on business cost.
- Q-bandit with and without MiniLM embedding is weak and should remain an ablation, not the main performance claim.
- The main performance path should become a cost-sensitive decision-policy framework over strong supervised risk scores.
- Phase 3.2 Level 5 v1 matched tuned Level 4 but did not improve cost, so Phase 3.3 adds Level 5 v2 as an advanced LLM-augmented hybrid attempt.

## Important Technical Correction

The original cost function is:

```text
C_FN = TransactionAmt * beta
C_FP = TransactionAmt * alpha
```

A pure expected-cost threshold cancels `TransactionAmt`:

```text
Flag if p * TransactionAmt * beta > (1-p) * TransactionAmt * alpha
=> p > alpha / (alpha + beta)
```

Therefore, the project must not claim that this closed-form threshold is dynamic by amount.

The proposed non-LLM dynamic policy uses amount-bin thresholding:

1. Split `TransactionAmt` into bins such as low, medium, high, and very high.
2. Tune one threshold per bin on validation only.
3. Apply those frozen bin thresholds to test.

## Five-Level Evaluation Framework

| Level | Model / Policy | Type | LLM | Decision Policy | Cost-Aware | Role |
|---:|---|---|---|---|---|---|
| 1 | Logistic Regression + RF/XGBoost default | Traditional ML | No | Fixed/default threshold | Class weight / scale_pos_weight | Basic baseline |
| 2 | LightGBM/XGBoost balanced + magic features | Strong supervised | No | Threshold tuning | Class weight / scale_pos_weight | Strong baseline |
| 3 | LightGBM + global cost-sensitive threshold | Cost-sensitive ML | No | One global validation-tuned threshold | Yes, global | Cost-sensitive baseline |
| 4 | LightGBM + amount-bin dynamic cost threshold | Proposed non-LLM | No | Threshold per `TransactionAmt` bin | Yes, per amount bin | Proposed baseline |
| 5 | LightGBM + MiniLM embedding + dynamic hybrid policy | Proposed LLM-assisted | Yes | Hybrid score + dynamic threshold or meta-policy | Yes, per amount bin | Main proposed method |
| Ablation | Q-Bandit without embedding | RL ablation | No | RL policy | Yes | Ablation |
| Ablation | Q-Bandit + MiniLM | RL + LLM ablation | Yes | RL policy | Yes | Ablation |

## Claim Strategy

| Claim | How to Support It |
|---|---|
| Level 3 improves over Level 1/2 | Lower Total Cost or higher Cost Saving than fixed/default and strong supervised baselines |
| Level 4 improves over Level 3 | Lower Total Cost or higher Cost Saving than global threshold on the same run mode |
| Level 5 improves over Level 4 | Lower Total Cost, higher Cost Saving, or better fraud Recall/F1 at acceptable FP Cost |
| Q-bandit is limited | RL ablation table shows weak or unstable performance honestly |

If Level 5 does not beat Level 4, the report should say so honestly. The main contribution can remain Level 4: a dynamic amount-aware cost-sensitive decision policy.

## Execution Order

1. Preserve Phase 1 data foundation.
2. Preserve Phase 2 and Phase 2.1 baseline evidence.
3. Preserve Phase 3 Q-bandit and MiniLM ablation evidence.
4. Implement Level 3 and initial Level 4 in a clean Phase 3.1 notebook.
5. Tune Level 4 in Phase 3.1b to reduce over-blocking in Cost-A/B while preserving the Cost-C gain.
6. Implement Level 5 v1 only after tuned Level 4 is stable.
7. Implement Level 5 v2 in Phase 3.3 with advanced hybrid candidates if Level 5 v1 does not show improvement.
8. Use Phase 4 for final comparison, error analysis, and report writing.

## Leakage Rules

- Do not include `TransactionID` or `isFraud` in model features.
- Fit feature transformations on train only.
- Tune thresholds on validation only.
- Derive amount-bin boundaries without using test labels or test metric optimization.
- Use test only once for final report metrics.
- Do not put labels, model scores, risk adjectives, or unsupported `ProductCD` interpretation into table-to-text input.

## Out of Scope

- Full Kaggle/FraudSquad 1st-place reproduction.
- Kaggle test-file or leaderboard post-processing.
- Fine-tuning LLMs.
- API LLM over the full dataset.
- PPO/A2C as a required path.
- Federated learning, RAG, multi-modal learning, phishing/IDS/malware/fake-review scope.
- Optimizing Accuracy as the primary objective.

## Recommended Next Artifacts

| Artifact | Purpose |
|---|---|
| `notebooks/04_dynamic_cost_policy.ipynb` | Implement Level 3 and Level 4 cleanly |
| `notebooks/04b_dynamic_policy_tuning.ipynb` | Tune Level 4 candidates after first Phase 3.1 results |
| `notebooks/05_llm_hybrid_policy.ipynb` | Implement Level 5 v1 after Level 4 is stable |
| `notebooks/06_llm_augmented_hybrid_v2.ipynb` | Implement advanced Level 5 v2 with prototype similarity, hybrid score fusion, meta-policy, and claim guard |
| `results/dynamic_policy_metrics_{RUN_MODE}.csv` | Level 3/4 metrics |
| `results/amount_bin_thresholds_{RUN_MODE}.csv` | Amount-bin thresholds and validation evidence |
| `results/five_level_comparison_level5_v2_{RUN_MODE}.csv` | Final comparison table with advanced Level 5 v2 rows |

---

Last updated: 2026-05-30 after source-implementing Phase 3.3 Level 5 v2.
