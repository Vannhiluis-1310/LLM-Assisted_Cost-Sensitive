# Requirements: LLM-Assisted Cost-Sensitive Fraud Decision Policies

**Defined:** 2026-05-24  
**Last Updated:** 2026-05-29  
**Core Value:** Build a reproducible experimental pipeline that compares fraud detection decision policies using fraud-focused metrics and economic cost metrics, not Accuracy.

## Scope Update

The project is not restarted. Completed Phase 1/2/2.1/3 work remains valid evidence. The updated plan changes the final comparison framework because current sample results show that pure Q-bandit is not strong enough to be the main performance claim.

The new main framework has five levels:

1. Basic ML baseline.
2. Strong supervised baseline.
3. Global cost-sensitive threshold.
4. Amount-bin dynamic cost threshold.
5. LLM-assisted hybrid dynamic policy.

Q-bandit without/with MiniLM remains as ablation evidence.

## Data Requirements

- [x] **DATA-01:** Use IEEE-CIS Fraud Detection as the main dataset.
- [x] **DATA-02:** Read `train_transaction.csv` and `train_identity.csv`.
- [x] **DATA-03:** Left join the two tables by `TransactionID` and preserve all labeled transactions.
- [x] **DATA-04:** Calculate fraud rate directly from the loaded data.
- [x] **DATA-05:** Create train/validation/test splits ordered by `TransactionDT`.

## Preprocessing Requirements

- [x] **PREP-01:** Handle missing numeric values using imputation fit on train only.
- [x] **PREP-02:** Handle categorical values using `"missing"` and rare-category handling.
- [x] **PREP-03:** Exclude `TransactionID` and `isFraud` from model features.
- [x] **PREP-04:** Create time-derived features from `TransactionDT` without interpreting it as a real calendar date.

## Baseline Requirements

- [x] **BASE-01:** Include approve-all baseline as the cost reference.
- [x] **BASE-02:** Include Logistic Regression baseline.
- [x] **BASE-03:** Include Random Forest, LightGBM, or XGBoost baseline.
- [x] **BASE-04:** Include imbalance handling via class weight, threshold tuning, or undersampling.
- [x] **BASE-05:** Include validation-based threshold tuning to minimize Total Cost.
- [x] **BASE-06:** Include a stronger supervised baseline with XGBoost/LightGBM when available.
- [x] **BASE-07:** Include leakage-safe magic-style feature engineering as a strong supervised reference when available.

## LLM Representation Requirements

- [x] **LLM-01:** Include neutral table-to-text serialization from real IEEE-CIS features.
- [x] **LLM-02:** Text input must not contain labels, model scores, risk wording, or interpretation of `ProductCD`.
- [x] **LLM-03:** Include local embeddings using `sentence-transformers/all-MiniLM-L6-v2`; fallback must be clearly labeled if MiniLM is unavailable.
- [x] **LLM-04:** Embeddings are cached and joined to the correct split; dimensionality reduction is fit on train only.
- [x] **LLM-05:** Level 5 hybrid policy must use MiniLM embedding only as an additional representation, not as a source of labels, model scores, or risk-wording leakage.

## RL / Bandit Ablation Requirements

- [x] **RL-01:** Include a contextual bandit or simple Q-bandit with actions `0 = approve`, `1 = flag/block`.
- [x] **RL-02:** Reward uses `TransactionAmt`, `alpha`, and `beta`, with FN penalized more heavily than FP.
- [x] **RL-03:** Include separate training and evaluation loops; evaluation uses no exploration.
- [x] **RL-04:** Include RL without embedding and RL with embedding for ablation.
- [x] **RL-05:** Include a Gym-style transaction environment with `reset()`, `step(action)`, state, action, reward, done, and info, without requiring a `gymnasium` dependency.
- [x] **RL-06:** Include Q-value contextual bandit policy with `Q(s, approve)`, `Q(s, flag/block)`, `gamma = 0`, epsilon-greedy training, and deterministic evaluation.
- [x] **RL-07:** Keep weighted cost-sensitive SGD as `cost_sensitive_supervised_sgd`, positioned as supervised baseline/ablation evidence, not the main RL claim.

## Policy Requirements

- [x] **POL-01:** Final comparison must include five levels: basic ML, strong supervised, global cost-sensitive threshold, amount-bin dynamic threshold, and LLM-assisted hybrid policy.
- [x] **POL-02:** Level 3 must tune one global threshold on validation only for each cost configuration.
- [x] **POL-03:** Level 4 must split `TransactionAmt` into bins such as low, medium, high, and very high.
- [x] **POL-04:** Level 4 amount bins must be derived from train/validation only, never from test labels or test optimization.
- [x] **POL-05:** Level 4 must tune one threshold per amount bin on validation only and apply those frozen bin thresholds to test.
- [x] **POL-06:** Level 4 must not claim that the closed-form expected-cost threshold is dynamic by amount, because `TransactionAmt` cancels under the original cost formula.
- [x] **POL-07:** Level 5 must combine LightGBM/XGBoost risk score with MiniLM embedding through a simple hybrid score or meta-policy.
- [x] **POL-08:** Level 5 must be compared directly against Level 4 on the same run mode, split, cost configuration, and metric set.
- [x] **POL-09:** If Level 5 does not beat Level 4, the report must state this honestly and keep the main contribution on dynamic cost-sensitive decision policy.
- [x] **POL-10:** Phase 3.1b must tune Level 4 candidate policies to reduce over-blocking in Cost-A/B while preserving or explicitly reporting trade-offs against the Cost-C gain.
- [x] **POL-11:** Phase 3.1b must include at least one guard against false-positive explosion, such as a validation precision floor, FP-cost cap, or threshold shrinkage toward the global threshold.
- [x] **POL-12:** Any modified expected-cost policy that adds fixed review/lost-sale cost must be labeled as a diagnostic alternative cost model and must not replace the original `TransactionAmt * alpha/beta` formulation.
- [x] **POL-13:** Phase 3.1b candidate selection must be based on validation metrics only, with selected hyperparameters and fallbacks saved to metadata before test evaluation.

## Evaluation Requirements

- [ ] **EVAL-01:** Report PR-AUC, fraud Recall, fraud Precision, and fraud F1.
- [ ] **EVAL-02:** Report FN Cost, FP Cost, and Total Cost.
- [ ] **EVAL-03:** Report Cost Saving against approve-all and relevant ML baselines.
- [ ] **EVAL-04:** Include sensitivity analysis with at least Cost-A, Cost-B, and Cost-C.
- [ ] **EVAL-05:** Include error analysis for high-cost FN, high-cost FP, Level 3 vs Level 4 disagreements, Level 4 vs Level 5 disagreements, and baseline-vs-RL disagreements.
- [ ] **EVAL-06:** Explain why Accuracy is unsuitable for imbalanced fraud detection.
- [ ] **EVAL-07:** Final claims must prioritize Total Cost, Cost Saving, fraud Recall/Precision/F1, not leaderboard ROC-AUC.

## Leakage and Integrity Rules

- [x] **LEAK-01:** Do not include `TransactionID` or `isFraud` in model features.
- [x] **LEAK-02:** Fit imputation, encoders, scalers, dimensionality reduction, and feature-engineering statistics on train only unless a step is explicitly validation tuning.
- [x] **LEAK-03:** Tune thresholds on validation only; never tune on test.
- [x] **LEAK-04:** Derive amount-bin boundaries without using test labels or test metric optimization.
- [x] **LEAK-05:** Test is report-only for final metrics.
- [x] **LEAK-06:** LLM/table-to-text must not include labels, model scores, risk adjectives, or unsupported interpretation of `ProductCD`.

## Out of Scope

| Feature | Reason |
|---|---|
| Full Kaggle 1st-place/FraudSquad reproduction | Too large; keep only as external reference unless fully reproduced |
| Kaggle test-file or leaderboard post-processing | Not needed and risks leakage/confusing evaluation scope |
| Fine-tune LLM | Beyond scope and unnecessary for MVP |
| PPO/A2C as required path | Too complex for current timeline and not needed before bandit/DQN stability |
| API LLM over whole dataset | Costly and outside guardrails |
| Synthetic data multi-agent | Outside project scope |
| Concept drift as main contribution | Future Work only |
| Phishing/IDS/malware/fake review | Wrong problem domain |
| European Credit Card Fraud as main dataset | Anonymized PCA features weaken LLM representation role |
| Backend/frontend complex app | Not needed for the one-month experimental goal |
| Federated learning, RAG, multi-modal learning | Out-of-scope extensions |
| Accuracy optimization as main objective | Misaligned with imbalanced fraud detection |

## Traceability

| Requirement | Phase | Status |
|---|---|---|
| DATA-01..05 | Phase 1 | Complete |
| PREP-01..04 | Phase 1 | Complete |
| BASE-01..05 | Phase 2 | Complete |
| BASE-06..07 | Phase 2.1 | Complete |
| LLM-01..04 | Phase 3 | Complete |
| LLM-05 | Phase 3.2 | Source complete; local fallback verified; final MiniLM run pending for claim |
| RL-01..07 | Phase 3 | Complete as ablation evidence |
| POL-01..06 | Phase 3.1 | Complete |
| POL-10..13 | Phase 3.1b | Complete |
| POL-07..09 | Phase 3.2 | Complete; local fallback result was honestly reported |
| EVAL-01..07 | Phase 4 | Pending |
| LEAK-01..02 | Phase 1/2/3 | Complete |
| LEAK-03..05 | Phase 3.1/3.1b | Complete |
| LEAK-06 | Phase 3/3.2 | Complete for Level 5 local run; rerun audit when Colab MiniLM output is generated |

## Coverage

- Completed foundation: data, preprocessing, baseline ML, strong supervised baseline, LLM/RL ablation.
- New pending work: amount-bin dynamic policy and LLM-assisted hybrid policy.
- No requirement asks to restart Phase 1/2.

---

Last updated: 2026-05-29 after executing Phase 3.2 Level 5 hybrid policy source/local fallback verification.
