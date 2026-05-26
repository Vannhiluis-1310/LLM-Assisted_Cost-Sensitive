# Phase 2: ML Baselines and Cost Metrics - Context

**Gathered / Ngày tạo:** 2026-05-26  
**Status / Trạng thái:** Ready for planning / Sẵn sàng lập kế hoạch

<domain>

## Phase Boundary / Ranh giới Phase

**VI:** Phase 2 chỉ xây dựng baseline ML và cost-sensitive evaluation cho IEEE-CIS Fraud Detection. Artifact code chính của phase này là một notebook: `notebooks/02_baselines_cost_metrics.ipynb`.

**EN:** Phase 2 only builds ML baselines and cost-sensitive evaluation for IEEE-CIS Fraud Detection. The main code artifact for this phase is one notebook: `notebooks/02_baselines_cost_metrics.ipynb`.

**VI:** Phase này không triển khai LLM embedding, table-to-text, contextual bandit, DQN, hoặc RL ablation. Các phần đó thuộc Phase 3.

**EN:** This phase does not implement LLM embeddings, table-to-text, contextual bandits, DQN, or RL ablations. Those belong to Phase 3.

</domain>

<decisions>

## Implementation Decisions / Quyết định triển khai

### Phase 1 Carry-over / Kế thừa từ Phase 1

- **D-01:** **VI:** Phase 1 đã PASS cho EDA/data foundation; `DATA-01..05` hoàn tất. **EN:** Phase 1 passed for EDA/data foundation; `DATA-01..05` are complete.
- **D-02:** **VI:** Preprocessing cho tabular ML được xem là PASS, nhưng Phase 2 không tái sử dụng artifact nặng `X_train.joblib` nếu không cần. **EN:** Tabular ML preprocessing is considered passed, but Phase 2 should avoid reusing the heavy `X_train.joblib` artifact unless necessary.
- **D-03:** **VI:** Split chính cố định theo `TransactionDT` với tỷ lệ 70/15/15 từ Phase 1. **EN:** The main split is fixed by `TransactionDT` using the 70/15/15 policy from Phase 1.
- **D-04:** **VI:** Nếu `data/processed/` không tồn tại sau cleanup, notebook Phase 2 được phép tự load raw CSV, left join, và tái tạo split giống Phase 1. **EN:** If `data/processed/` is absent after cleanup, the Phase 2 notebook may load raw CSVs, left join, and recreate the same Phase 1 split.

### Artifact Policy / Chính sách artifact

- **D-05:** **VI:** Chỉ cần notebook `.ipynb` làm source artifact chính. Không tạo lại các file `.py` trong `src/` cho Phase 2. **EN:** Only an `.ipynb` notebook is needed as the main source artifact. Do not recreate `.py` files under `src/` for Phase 2.
- **D-06:** **VI:** Các file `results/` và `reports/figures/` chỉ là output sinh ra khi chạy notebook, không phải source code chính. **EN:** `results/` and `reports/figures/` files are generated outputs from the notebook, not primary source code.
- **D-07:** **VI:** Không lưu full feature matrix lớn dạng `X_train.joblib` trong Phase 2 mặc định. **EN:** Do not save large full feature matrices such as `X_train.joblib` by default in Phase 2.

### Baselines / Baseline ML

- **D-08:** **VI:** Bắt buộc có approve-all baseline làm cost reference. **EN:** Include an approve-all baseline as the cost reference.
- **D-09:** **VI:** Bắt buộc có Logistic Regression baseline với imbalance handling, ưu tiên `class_weight="balanced"`. **EN:** Include Logistic Regression with imbalance handling, preferably `class_weight="balanced"`.
- **D-10:** **VI:** Bắt buộc có một baseline mạnh hơn: ưu tiên LightGBM/XGBoost nếu có sẵn; nếu không có thì dùng Random Forest fallback. **EN:** Include one stronger baseline: prefer LightGBM/XGBoost if available; otherwise use Random Forest fallback.
- **D-11:** **VI:** Logistic Regression có thể dùng sparse one-hot pipeline; tree baseline nên dùng preprocessing nhẹ hơn như ordinal encoding để giảm RAM. **EN:** Logistic Regression may use a sparse one-hot pipeline; the tree baseline should use lighter preprocessing such as ordinal encoding to reduce RAM use.

### Metrics and Thresholds / Metric và ngưỡng quyết định

- **D-12:** **VI:** Metrics chính gồm PR-AUC, ROC-AUC, Recall fraud, Precision fraud, F1 fraud, confusion matrix, FN Cost, FP Cost, Total Cost, và Cost Saving. **EN:** Main metrics include PR-AUC, ROC-AUC, fraud Recall, fraud Precision, fraud F1, confusion matrix, FN Cost, FP Cost, Total Cost, and Cost Saving.
- **D-13:** **VI:** Accuracy không được dùng làm mục tiêu tối ưu chính; nếu xuất hiện thì chỉ là diagnostic phụ và phải giải thích vì sao không phù hợp. **EN:** Accuracy must not be the primary optimization target; if shown, it is only a secondary diagnostic and must be explained as unsuitable.
- **D-14:** **VI:** Threshold tuning chỉ fit/chọn trên validation, sau đó áp dụng một lần lên test. **EN:** Threshold tuning must be selected on validation only, then applied once to test.
- **D-15:** **VI:** Cần ít nhất 3 cấu hình cost sensitivity cho `alpha`/`beta`. **EN:** Include at least three cost-sensitivity configurations for `alpha`/`beta`.

### Explicitly Deferred / Tạm hoãn rõ ràng

- **D-16:** **VI:** LLM/table-to-text preprocessing chỉ là planned, chưa implement trong Phase 2. **EN:** LLM/table-to-text preprocessing is only planned and not implemented in Phase 2.
- **D-17:** **VI:** Không tạo embedding, không gọi API LLM, không tạo risk text, không làm RL trong Phase 2. **EN:** Do not create embeddings, call LLM APIs, create risk text, or implement RL in Phase 2.

</decisions>

<canonical_refs>

## Canonical References / Tài liệu tham chiếu chuẩn

- `.planning/PROJECT.md` - project scope, dataset, guardrails, and out-of-scope items.
- `.planning/REQUIREMENTS.md` - Phase 2 requirements `BASE-01..05`.
- `.planning/ROADMAP.md` - Phase 2 goal and success criteria.
- `.planning/STATE.md` - Phase 1 verification snapshot and current cleanup state.
- `notebooks/01_data_check.ipynb` - reusable Phase 1 data loading, join, EDA, split, and preprocessing logic.
- `data/raw/train_transaction.csv` and `data/raw/train_identity.csv` - local IEEE-CIS inputs.

</canonical_refs>

<deferred>

## Deferred Ideas / Ý tưởng để sau

- Table-to-text serialization belongs to Phase 3.
- MiniLM/local embedding belongs to Phase 3.
- Contextual bandit/RL ablation belongs to Phase 3.
- Final error analysis and report polish belong to Phase 4.

</deferred>

---

*Phase: 02-ml-baselines-and-cost-metrics*  
*Context gathered: 2026-05-26*
