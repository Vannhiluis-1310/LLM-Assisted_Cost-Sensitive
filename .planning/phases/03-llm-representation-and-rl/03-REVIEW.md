---
phase: 3
status: action-required
reviewed: 2026-05-26
upgraded: 2026-05-27
---

# Phase 3 Code Review

## Findings

No blocking source issues found in `notebooks/03_llm_representation_and_rl.ipynb` after smoke execution.

**Action required (2026-05-27):** RL implementation identified as supervised learning, not RL. See Blocking Issue below.

## Checks

| Check | Result |
|---|---|
| Notebook JSON parses | PASS |
| Code cells parse with `ast` | PASS |
| Notebook executed in smoke mode | PASS |
| Required RL ablation models present | PASS |
| Required cost configs present | PASS |
| Text leakage audit implemented | PASS |
| Scope guardrails | PASS |

## Non-Blocking Notes

- Local smoke used `tfidf_svd_fallback` because `sentence-transformers` is not installed locally.
- Full Colab execution should confirm `embedding_backend_used = "minilm"` before Phase 4 uses the results as final evidence.
- Smoke metrics are intentionally not final report numbers.

## Blocking Issue (resolved by upgrade plan 2026-05-27)

**Issue:** `cost_sensitive_sgd_contextual_bandit` = `SGDClassifier + sample_weight + threshold tuning`. This is supervised learning with cost-sensitive loss, not Reinforcement Learning. If presented as RL to the evaluation committee, it would not be defensible.

**Resolution:** Upgrade notebook to Q-value Gym-style Contextual Bandit:
- `TransactionFraudBanditEnv` with `reset()`, `step(action)`, reward, done, info.
- `QBanditPolicy` with 2 `sklearn.SGDRegressor` (`Q(s,0)`, `Q(s,1)`), `gamma=0`, epsilon-greedy training, deterministic evaluation.
- Rename old SGD path to `cost_sensitive_supervised_sgd` and position as Phase 3 supervised baseline.
- New RL variants: `rl_q_bandit_without_embedding`, `rl_q_bandit_with_embedding`.

**Status:** Upgrade planned and confirmed. Notebook edit pending.
