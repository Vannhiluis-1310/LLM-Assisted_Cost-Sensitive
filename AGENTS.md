# Hướng dẫn tác nhân dự án / Project Agent Guide

## Dự án / Project

Repository này dùng cho đồ án cuối kỳ.  
This repository is for the final project.

**Tiếng Việt:** Khung ra quyết định nhạy cảm chi phí động cho phát hiện gian lận giao dịch thương mại điện tử mất cân bằng với sự tăng cường biểu diễn LLM.

**English:** Dynamic Cost-Sensitive Decision Framework for Imbalanced E-commerce Transaction Fraud Detection with LLM Representation Enhancement.

## Phạm vi không được vượt / Non-negotiable Scope

- **VI:** Dùng IEEE-CIS Fraud Detection làm dataset chính.  
  **EN:** Use IEEE-CIS Fraud Detection as the main dataset.
- **VI:** Dùng `train_transaction.csv` và `train_identity.csv`, join bằng `TransactionID`.  
  **EN:** Use `train_transaction.csv` and `train_identity.csv`, joined by `TransactionID`.
- **VI:** Tối ưu fraud detection nhạy cảm chi phí, không tối ưu Accuracy.  
  **EN:** Optimize cost-sensitive fraud detection, not Accuracy.
- **VI:** Giới hạn LLM ở local embedding hoặc explanation case study nhỏ.  
  **EN:** Limit LLM usage to local embeddings or small case-study explanations.
- **VI:** Không fine-tune LLM.  
  **EN:** Do not fine-tune LLMs.
- **VI:** Không thêm phishing, IDS, malware, fake review, Federated Learning, RAG, multi-modal learning, hoặc backend/frontend phức tạp.  
  **EN:** Do not add phishing, IDS, malware, fake review, Federated Learning, RAG, multi-modal learning, or complex backend/frontend scope.

## Hướng kỹ thuật khuyến nghị / Recommended Technical Path

- **VI:** Baselines: Logistic Regression + class weight, LightGBM/XGBoost nếu có, Random Forest fallback.  
  **EN:** Baselines: Logistic Regression + class weight, LightGBM/XGBoost if available, Random Forest fallback.
- **VI:** Embedding: `sentence-transformers/all-MiniLM-L6-v2`.  
  **EN:** Embedding: `sentence-transformers/all-MiniLM-L6-v2`.
- **VI:** RL: contextual bandit one-step với actions `0 = approve`, `1 = flag/block`.  
  **EN:** RL: one-step contextual bandit with actions `0 = approve`, `1 = flag/block`.
- **VI:** Metrics chính: PR-AUC, Recall fraud, Precision fraud, F1 fraud, FN Cost, FP Cost, Total Cost, Cost Saving.  
  **EN:** Main metrics: PR-AUC, fraud recall, fraud precision, fraud F1, FN Cost, FP Cost, Total Cost, Cost Saving.

## File kế hoạch / Planning Files

- `.planning/PROJECT.md`
- `.planning/REQUIREMENTS.md`
- `.planning/ROADMAP.md`
- `.planning/STATE.md`
- `KE_HOACH_DU_AN.md`
