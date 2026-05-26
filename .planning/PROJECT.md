# LLM-Assisted Cost-Sensitive RL for E-commerce Fraud Detection

## Đây là gì / What This Is

**VI:** Dự án cuối kỳ môn Bảo mật Thương mại Điện tử về phát hiện gian lận giao dịch thương mại điện tử trên IEEE-CIS Fraud Detection. Trọng tâm là xử lý dữ liệu mất cân bằng và tối ưu chi phí false negative/false positive, không tối ưu Accuracy đơn thuần.

**EN:** This is a final project for E-commerce Security on e-commerce transaction fraud detection using IEEE-CIS Fraud Detection. The focus is handling imbalanced data and optimizing false-negative/false-positive costs, not optimizing Accuracy alone.

**VI:** LLM/local embedding chỉ đóng vai trò hỗ trợ tạo biểu diễn từ table-to-text trung lập của giao dịch. RL được giới hạn ở contextual bandit hoặc DQN đơn giản để ra quyết định `approve` hoặc `flag/block`.

**EN:** The LLM/local embedding component only supports representation learning from neutral table-to-text transaction descriptions. RL is limited to a contextual bandit or simple DQN for deciding `approve` or `flag/block`.

## Giá trị cốt lõi / Core Value

**VI:** Xây dựng được pipeline thực nghiệm chạy lại được, so sánh được baseline ML với RL cost-sensitive bằng metrics fraud và chi phí kinh tế.

**EN:** Build a reproducible experimental pipeline that compares ML baselines with cost-sensitive RL using fraud-focused metrics and economic cost metrics.

## Requirements

### Đã xác thực / Validated

- **VI:** Phase 1 đã xác thực dataset chính là IEEE-CIS Fraud Detection với `train_transaction.csv` và `train_identity.csv`.  
  **EN:** Phase 1 validated IEEE-CIS Fraud Detection as the main dataset using `train_transaction.csv` and `train_identity.csv`.
- **VI:** Pipeline đã left join theo `TransactionID` và giữ nguyên `590,540` giao dịch có label.  
  **EN:** The pipeline left-joined on `TransactionID` and preserved all `590,540` labeled transactions.
- **VI:** Fraud rate đã được tính trực tiếp từ dữ liệu: `20,663 / 590,540 = 3.499%`.  
  **EN:** Fraud rate was computed directly from the data: `20,663 / 590,540 = 3.499%`.
- **VI:** Split theo `TransactionDT` 70/15/15 đã chạy xong: train `413,378`, validation `88,581`, test `88,581`.  
  **EN:** The 70/15/15 `TransactionDT` split completed: train `413,378`, validation `88,581`, test `88,581`.
- **VI:** Preprocessing fit trên train-only đã chạy xong, gồm numeric imputation, categorical `"missing"`/rare handling, loại `TransactionID`/`isFraud`, và 3 feature time-derived từ `TransactionDT`.  
  **EN:** Train-only preprocessing completed with numeric imputation, categorical `"missing"`/rare handling, exclusion of `TransactionID`/`isFraud`, and 3 time-derived `TransactionDT` features.
- **VI:** Phase 2 đã tạo notebook baseline ML `notebooks/02_baselines_cost_metrics.ipynb` với approve-all, Logistic Regression class weight, baseline mạnh LightGBM/XGBoost hoặc Random Forest fallback, threshold tuning validation-only, và cost metrics.  
  **EN:** Phase 2 created the ML baseline notebook `notebooks/02_baselines_cost_metrics.ipynb` with approve-all, Logistic Regression class weight, a stronger LightGBM/XGBoost or Random Forest fallback baseline, validation-only threshold tuning, and cost metrics.
- **VI:** Smoke run Phase 2 trên 10,000 dòng đã tạo `results/baseline_metrics.csv`, `results/threshold_tuning.csv`, và các hình PR/ROC/cost/confusion matrix.  
  **EN:** The Phase 2 10,000-row smoke run generated `results/baseline_metrics.csv`, `results/threshold_tuning.csv`, and PR/ROC/cost/confusion-matrix figures.
- **VI:** Phase 2.1 đã thêm baseline `xgboost_magic_style` với 19 feature magic-style leakage-safe, gồm count/frequency encoding và `TransactionAmt` group stats fit trên train-only.  
  **EN:** Phase 2.1 added the `xgboost_magic_style` baseline with 19 leakage-safe magic-style features, including count/frequency encoding and train-only `TransactionAmt` group stats.

### Đang làm / Active

- **VI:** Có table-to-text trung lập từ feature thật.  
  **EN:** Include neutral table-to-text serialization from real features.
- **VI:** Có local embedding bằng `sentence-transformers/all-MiniLM-L6-v2`.  
  **EN:** Include local embeddings using `sentence-transformers/all-MiniLM-L6-v2`.
- **VI:** Có contextual bandit hoặc DQN đơn giản với action `approve/block`.  
  **EN:** Include a contextual bandit or simple DQN with `approve/block` actions.
- **VI:** Có reward function dựa trên `TransactionAmt`, `alpha`, `beta`, phạt FN nặng hơn FP.  
  **EN:** Include a reward function based on `TransactionAmt`, `alpha`, and `beta`, penalizing FN more heavily than FP.
- **VI:** Có ablation RL without LLM embedding vs RL with embedding.  
  **EN:** Include ablation: RL without LLM embedding vs RL with embedding.
- **VI:** Có PR-AUC, Recall fraud, Precision fraud, F1 fraud, FN Cost, FP Cost, Total Cost, Cost Saving.  
  **EN:** Report PR-AUC, fraud recall, fraud precision, fraud F1, FN Cost, FP Cost, Total Cost, and Cost Saving.
- **VI:** Báo cáo giải thích vì sao Accuracy không phù hợp với dữ liệu mất cân bằng.  
  **EN:** Explain why Accuracy is unsuitable for imbalanced data.

### Ngoài phạm vi / Out of Scope

- **VI:** Fine-tune LLM - vượt scope và không cần thiết cho MVP.  
  **EN:** Fine-tuning LLMs - beyond scope and unnecessary for the MVP.
- **VI:** PPO/A2C - chỉ xem xét khi contextual bandit/DQN đơn giản đã hoàn thành.  
  **EN:** PPO/A2C - only consider after the contextual bandit/simple DQN is complete.
- **VI:** Multi-agent synthetic data - không nằm trong đề tài.  
  **EN:** Multi-agent synthetic data - outside the project topic.
- **VI:** Concept drift là phần chính - chỉ đưa vào Future Work.  
  **EN:** Concept drift as a main component - Future Work only.
- **VI:** Phishing email, fake review, URL phishing, IDS, malware - sai miền bài toán.  
  **EN:** Phishing email, fake review, URL phishing, IDS, malware - wrong problem domain.
- **VI:** European Credit Card Fraud làm dataset chính - feature PCA ẩn danh không phù hợp vai trò LLM.  
  **EN:** European Credit Card Fraud as the main dataset - anonymized PCA features are unsuitable for the LLM role.
- **VI:** API LLM trên toàn bộ dataset - tốn chi phí và vượt guardrail.  
  **EN:** API LLM over the whole dataset - costly and outside the guardrails.
- **VI:** RAG, multi-modal learning, Federated Learning - để Future Work nếu cần.  
  **EN:** RAG, multi-modal learning, Federated Learning - Future Work if needed.
- **VI:** Backend/frontend phức tạp - không phục vụ mục tiêu thực nghiệm 1 tháng.  
  **EN:** Complex backend/frontend - not needed for the one-month experimental goal.

## Bối cảnh / Context

**VI:** Dataset IEEE-CIS có hai bảng train cần dùng: `train_transaction.csv` chứa label, amount, time và nhiều feature giao dịch; `train_identity.csv` chứa identity/device và không phải giao dịch nào cũng có bản ghi identity. Cách join đúng là left join từ transaction sang identity theo `TransactionID`.

**EN:** IEEE-CIS provides two training tables for this project: `train_transaction.csv`, which contains labels, amount, time, and many transaction features; and `train_identity.csv`, which contains identity/device features and is not available for every transaction. The correct join is a left join from transaction to identity on `TransactionID`.

**VI:** Do dữ liệu mất cân bằng, Accuracy không thể là metric chính. Mô hình approve-all có thể đạt Accuracy cao nhưng bỏ sót fraud và tạo FN Cost lớn. Vì vậy, báo cáo phải đặt PR-AUC, Recall fraud, Total Cost và Cost Saving làm trọng tâm.

**EN:** Because the data is imbalanced, Accuracy cannot be the main metric. An approve-all model can achieve high Accuracy while missing fraud and producing large FN Cost. Therefore, the report should emphasize PR-AUC, fraud recall, Total Cost, and Cost Saving.

## Ràng buộc / Constraints

- **VI:** **Timeline:** 1 tháng - ưu tiên pipeline chạy được và giải thích được.  
  **EN:** **Timeline:** 1 month - prioritize a runnable and explainable pipeline.
- **VI:** **Dataset:** Chỉ dùng IEEE-CIS làm dataset chính.  
  **EN:** **Dataset:** Use only IEEE-CIS as the main dataset.
- **VI:** **LLM:** Không fine-tune; ưu tiên local MiniLM embedding.  
  **EN:** **LLM:** No fine-tuning; prefer local MiniLM embeddings.
- **VI:** **RL:** Contextual bandit one-step là MVP; DQN đơn giản chỉ khi còn thời gian.  
  **EN:** **RL:** One-step contextual bandit is the MVP; simple DQN only if time remains.
- **VI:** **Evaluation:** Split theo `TransactionDT`; threshold tuning chỉ dùng validation.  
  **EN:** **Evaluation:** Split by `TransactionDT`; threshold tuning uses validation only.
- **VI:** **Scope:** Không mở rộng sang backend/frontend phức tạp hoặc bài toán bảo mật khác.  
  **EN:** **Scope:** Do not expand into complex backend/frontend work or other security problems.

## Quyết định chính / Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Time-based split theo `TransactionDT` / Time-based split by `TransactionDT` | Giảm leakage và gần với bài toán giao dịch theo thời gian / Reduces leakage and better matches transaction chronology | Validated in Phase 1 |
| Left join identity vào transaction / Left join identity into transaction | Giữ tất cả giao dịch có label, identity missing là tín hiệu hợp lệ / Keeps all labeled transactions; missing identity is valid information | Validated in Phase 1 |
| MiniLM local embedding | Giảm chi phí, không phụ thuộc API, phù hợp 1 tháng / Low cost, no API dependency, feasible in 1 month | Pending |
| Contextual bandit thay vì PPO/A2C / Contextual bandit instead of PPO/A2C | Giao dịch là quyết định one-step, đơn giản và giải thích được / Transactions are one-step decisions; simpler and explainable | Pending |
| Total Cost/Cost Saving là metric chính / Total Cost and Cost Saving as main metrics | Phù hợp cost-sensitive fraud detection hơn Accuracy / Better aligned with cost-sensitive fraud detection than Accuracy | Implemented for ML baselines in Phase 2 and Phase 2.1 |

## Evolution

This document evolves at phase transitions and milestone boundaries.

After each phase transition:
1. Requirements invalidated? Move to Out of Scope with reason.
2. Requirements validated? Move to Validated with phase reference.
3. New requirements emerged? Add to Active.
4. Decisions to log? Add to Key Decisions.
5. "What This Is" still accurate? Update if drifted.

After each milestone:
1. Full review of all sections.
2. Core Value check - still the right priority?
3. Audit Out of Scope - reasons still valid?
4. Update Context with current state.

---
*Last updated: 2026-05-26 after Phase 2.1 smoke-verified implementation*
