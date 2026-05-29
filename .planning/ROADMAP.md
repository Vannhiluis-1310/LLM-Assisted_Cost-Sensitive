# Roadmap: LLM-Assisted Cost-Sensitive Fraud Decision Policies

## Overview

The project is not restarted. Phase 1, Phase 2, Phase 2.1, and Phase 3 remain completed evidence:

- Phase 1 proves the IEEE-CIS data foundation, join integrity, EDA, preprocessing, and time split.
- Phase 2 proves basic ML baselines and cost-sensitive metrics.
- Phase 2.1 proves a stronger Kaggle-inspired supervised baseline.
- Phase 3 proves table-to-text, MiniLM/local embedding, and Q-bandit ablations. Current sample results show that pure Q-bandit is weak, so it becomes ablation evidence rather than the main claim.

The updated direction is a five-level evaluation framework centered on business cost, not Accuracy or leaderboard AUC.

| Phase | Name | Goal | Requirements |
|---:|---|---|---|
| 1 | Data Pipeline and EDA | Join, clean, split, and understand IEEE-CIS without leakage. | DATA-01..05, PREP-01..04 |
| 2 | ML Baselines and Cost Metrics | Build basic ML baselines, cost metrics, and validation threshold tuning. | BASE-01..05 |
| 2.1 | Kaggle-Inspired Strong Supervised Baseline | Add leakage-safe XGBoost/LightGBM magic-style features as a strong supervised reference. | BASE-03..05, SOTA-01 |
| 3 | LLM Representation and RL Ablations | Build neutral table-to-text, MiniLM embeddings, and Q-bandit ablations; keep weak RL results as evidence. | LLM-01..04, RL-01..07 |
| 3.1 | Dynamic Cost-Sensitive Decision Policies | Implement the five-level framework, especially Level 3 global threshold and Level 4 amount-bin dynamic threshold. | POL-01..06, EVAL-01..04 |
| 3.1b | Level 4 Dynamic Policy Tuning | Tune Level 4 candidates to reduce over-blocking in Cost-A/B while preserving the Cost-C gain. | POL-10..13, EVAL-01..04, EVAL-07 |
| 3.2 | LLM-Assisted Hybrid Policy | After Level 4 is stable, test Level 5: LightGBM score + MiniLM embedding + dynamic hybrid policy. | POL-07..09, LLM-01..04 |
| 4 | Final Evaluation, Error Analysis, Report | Produce final result tables, figures, error analysis, and report using the five-level framework. | EVAL-01..06 |

## Updated Evaluation Framework

| Level | Model / Policy | Type | LLM | Decision Policy | Cost-Aware | Role |
|---:|---|---|---|---|---|---|
| 1 | Logistic Regression + RF/XGBoost default | Traditional ML | No | Fixed/default threshold | Class weight / scale_pos_weight | Basic baseline |
| 2 | LightGBM/XGBoost balanced + magic features | Strong supervised | No | Threshold tuning | Class weight / scale_pos_weight | Strong baseline |
| 3 | LightGBM + global cost-sensitive threshold | Cost-sensitive ML | No | One global validation-tuned threshold | Yes, global | Cost-sensitive baseline |
| 4 | LightGBM + amount-bin dynamic cost threshold | Proposed non-LLM | No | Threshold per `TransactionAmt` bin | Yes, per amount bin | Proposed baseline |
| 5 | LightGBM + MiniLM embedding + dynamic hybrid policy | Proposed LLM-assisted | Yes | Hybrid score + dynamic threshold or meta-policy | Yes, per amount bin | Main proposed method |
| Ablation | Q-Bandit without embedding | RL ablation | No | RL policy | Yes | Ablation |
| Ablation | Q-Bandit + MiniLM | RL + LLM ablation | Yes | RL policy | Yes | Ablation |

## Important Correction

The original cost formula cancels `TransactionAmt` when deriving a pure closed-form expected-cost threshold:

```text
Flag if p * TransactionAmt * beta > (1-p) * TransactionAmt * alpha
=> p > alpha / (alpha + beta)
```

Therefore, the project must not claim that this closed-form threshold is dynamic by amount. The new Level 4 policy is dynamic because it tunes separate validation thresholds for amount bins such as low, medium, high, and very high.

## Phase 1: Data Pipeline and EDA

**Status:** Complete, verified 2026-05-25.

**Goal:** Build a correct data foundation and prevent leakage.

**Success Criteria:**

1. Load `train_transaction.csv` and `train_identity.csv`.
2. Left join by `TransactionID` without losing labeled transactions.
3. Calculate fraud rate directly from the loaded data.
4. Split train/validation/test by `TransactionDT` using 70/15/15.
5. Handle missing values, categorical values, rare categories, and exclude `TransactionID`/`isFraud` from features.

**Primary Artifact:** `notebooks/01_data_check.ipynb`

## Phase 2: ML Baselines and Cost Metrics

**Status:** Complete, smoke/sample verified.

**Goal:** Build Level 1 and part of Level 2 baseline evidence.

**Success Criteria:**

1. Include approve-all baseline.
2. Include Logistic Regression with class weight.
3. Include Random Forest or LightGBM/XGBoost with imbalance handling.
4. Include PR-AUC, ROC-AUC, fraud Recall/Precision/F1, FN Cost, FP Cost, Total Cost, and Cost Saving.
5. Tune thresholds on validation only for each `alpha`/`beta`.

**Primary Artifact:** `notebooks/02_baselines_cost_metrics.ipynb`

## Phase 2.1: Kaggle-Inspired Strong Supervised Baseline

**Status:** Complete, smoke/sample verified.

**Goal:** Strengthen Level 2 with a leakage-safe magic-style supervised baseline.

**Success Criteria:**

1. Add XGBoost/LightGBM magic-style features on the same `TransactionDT` split.
2. Fit feature engineering statistics on train only, then map validation/test.
3. Include count/frequency encoding and simple `TransactionAmt` group statistics.
4. Do not use Kaggle test files, leaderboard artifacts, full 1st-place post-processing, or label leakage.
5. Keep FraudSquad/1st-place solution only as an external reference unless fully reproduced.

**Primary Artifact:** `notebooks/02_baselines_cost_metrics.ipynb`

## Phase 3: LLM Representation and RL Ablations

**Status:** Complete as ablation evidence.

**Goal:** Test whether LLM/local embeddings help a Q-bandit policy. Current results show Q-bandit is weak, so it is not the main performance claim.

**Success Criteria:**

1. Table-to-text uses only real IEEE-CIS features and a neutral template.
2. MiniLM embeddings are generated locally, cached, and joined to the correct split.
3. Q-bandit without embedding runs.
4. Q-bandit with embedding runs.
5. Ablations use the same split, reward, and metrics.

**Primary Artifact:** `notebooks/03_llm_representation_and_rl.ipynb`

## Phase 3.1: Dynamic Cost-Sensitive Decision Policies

**Status:** Implemented source; Colab result review indicates Cost-C success and Cost-A/B over-blocking.

**Goal:** Implement Levels 3 and 4 cleanly so the project can make a stronger cost-sensitive claim without pretending pure RL beats LightGBM.

**Success Criteria:**

1. Level 3 tunes one global threshold on validation only for each `alpha`/`beta`.
2. Level 4 derives `TransactionAmt` bins from train/validation only.
3. Level 4 tunes one threshold per amount bin on validation only.
4. Test is used only for final reporting, never for bin fitting or threshold tuning.
5. Outputs include `dynamic_policy_metrics_{RUN_MODE}.csv`, `amount_bin_thresholds_{RUN_MODE}.csv`, and `five_level_comparison_{RUN_MODE}.csv`.
6. Claims are based on Total Cost, Cost Saving, fraud Recall/Precision/F1, not Accuracy.

**Suggested Artifact:** `notebooks/04_dynamic_cost_policy.ipynb` or a clean new section in `notebooks/02_baselines_cost_metrics.ipynb`

## Phase 3.1b: Level 4 Dynamic Policy Tuning

**Status:** Implemented and sample-verified on 2026-05-29.

**Goal:** Improve the Level 4 amount-bin dynamic policy so it reduces over-blocking in Cost-A/B while preserving the high-FN-cost Cost-C advantage.

**Success Criteria:**

1. Create a separate tuning notebook so Phase 3.1 remains the clean Level 3/4 implementation.
2. Load Phase 3.1 outputs and Phase 2 scores for the same `RUN_MODE`.
3. Implement validation-only candidate policies:
   - shrinkage from per-bin thresholds toward global threshold;
   - precision guard or FP-cost cap;
   - alternate amount-bin strategies.
4. Keep any modified expected-cost policy as a clearly labeled diagnostic only.
5. Select candidates on validation only and evaluate frozen choices on test.
6. Report whether Cost-A/B precision or FP Cost improves and whether Cost-C advantage is preserved.

**Suggested Artifact:** `notebooks/04b_dynamic_policy_tuning.ipynb`

## Phase 3.2: LLM-Assisted Hybrid Policy

**Status:** Complete for source/local fallback verification; Colab MiniLM rerun pending for final LLM claim.

**Goal:** Test Level 5 by combining LightGBM risk score, MiniLM embedding, and a dynamic threshold/meta-policy.

**Success Criteria:**

1. MiniLM embeddings come from neutral table-to-text with no label, risk wording, or model-score leakage in the text input.
2. Hybrid policy is fit only on train/validation; test is report-only.
3. Level 5 is compared directly with Level 4 using the same run mode and split.
4. If Level 5 does not beat Level 4, report honestly and keep the main claim on dynamic cost-sensitive policy.

**Artifact:** `notebooks/05_llm_hybrid_policy.ipynb`

**Execution Evidence:** Local `sample_100k` run completed with `tfidf_svd_fallback`, producing `results/phase32_*_sample_100k.csv`, `results/five_level_comparison_level5_sample_100k.csv`, and `reports/figures/phase32_*_sample_100k.png`. This verifies the flow but does not support a final LLM claim until Colab MiniLM output is generated.

## Phase 4: Final Evaluation, Error Analysis, Report

**Status:** Pending.

**Goal:** Turn the pipeline into defensible final-report results.

**Success Criteria:**

1. Main table covers Level 1..5 plus Q-bandit ablations.
2. Sensitivity analysis covers at least Cost-A, Cost-B, and Cost-C.
3. Figures include class imbalance, amount distribution, PR curve, cost-vs-threshold, confusion matrix, amount-bin thresholds, and total-cost bars.
4. Error analysis covers high-cost FN, high-cost FP, Level 3 vs Level 4 disagreements, and Level 4 vs Level 5 disagreements.
5. Report explains why Accuracy is unsuitable.
6. Report does not claim LLM helps unless Level 5 results support it.

## Claim Strategy

| Claim | Required Evidence |
|---|---|
| Level 3 improves over Level 1/2 | Lower Total Cost or higher Cost Saving than fixed/default and strong supervised baselines |
| Level 4 improves over Level 3 | Lower Total Cost or higher Cost Saving than global threshold on the same run mode |
| Level 5 improves over Level 4 | Lower Total Cost, higher Cost Saving, or better fraud Recall/F1 at acceptable FP Cost |
| Q-bandit is limited | RL ablation table shows weak or unstable performance honestly |

## Out of Scope

- Full Kaggle 1st-place/FraudSquad reproduction.
- Kaggle test-file or leaderboard post-processing.
- Fine-tuning LLMs.
- API LLM over the full dataset.
- PPO/A2C as a required path.
- Federated learning, RAG, multi-modal learning, phishing/IDS/malware/fake-review scope.
- Accuracy optimization as the primary objective.

## Four-Week Timeline

| Week | Focus | Exit Condition |
|---|---|---|
| 1 | Data pipeline, EDA, preprocessing | Split and preprocessing are reproducible |
| 2 | Baselines, cost metrics, strong supervised baseline | Level 1/2 baseline table exists |
| 3 | LLM/RL ablations and dynamic cost policies | Q-bandit ablation exists; Level 3/4 dynamic policy runs |
| 4 | LLM-assisted hybrid policy and final report | Level 5 comparison, figures, error analysis, and report are complete |

---

Last updated: 2026-05-29 after executing Phase 3.2 LLM-assisted hybrid policy source/local fallback verification.
