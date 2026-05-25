# Phase 1: Data Pipeline and EDA - Discussion Log

> **Audit trail only / Chỉ để audit.** Do not use as input to planning, research, or execution agents.  
> Không dùng file này làm input cho planning, research, hoặc execution agents.
>
> Decisions are captured in `01-CONTEXT.md`. / Các quyết định chính nằm trong `01-CONTEXT.md`.

**Date / Ngày:** 2026-05-24  
**Phase:** 1 - Data Pipeline and EDA  
**Areas discussed / Phạm vi đã thảo luận:** Dataset placement, join integrity, split strategy, preprocessing scope, EDA outputs, output organization.

---

## Interaction Note / Ghi chú tương tác

**VI:** Interactive `request_user_input` không khả dụng trong Default mode hiện tại. Workflow fallback được áp dụng: chọn recommended core areas và capture conservative defaults nhất quán với tài liệu dự án.

**EN:** Interactive `request_user_input` was unavailable in the current Default mode. The workflow fallback was applied: select the recommended core areas and capture conservative defaults consistent with existing project documents.

## User Follow-up / Phản hồi bổ sung từ người dùng

**VI:** Người dùng muốn tạo các file ở local nhưng upload lên Google Colab vẫn chạy bình thường. Người dùng cũng phản hồi rằng dataset chưa được nêu đủ rõ trong context Phase 1.

**EN:** The user wants files to be created locally while still running correctly after upload to Google Colab. The user also noted that the dataset was not stated clearly enough in the Phase 1 context.

---

## Dataset Placement / Vị trí dataset

| Option | Description / Mô tả | Selected |
|--------|----------------------|----------|
| Explicit IEEE-CIS dataset | VI: Ghi rõ dataset chính là IEEE-CIS Fraud Detection, dùng `train_transaction.csv` và `train_identity.csv`. EN: Explicitly name IEEE-CIS Fraud Detection as the main dataset, using `train_transaction.csv` and `train_identity.csv`. | yes |
| `data/raw/` | VI: Folder raw-data thông dụng, dễ ignore trong Git và dễ document. EN: Conventional raw-data folder, easy to ignore in Git and document. | yes |
| Repository root | VI: Đơn giản nhưng dễ lộn xộn và rủi ro với CSV lớn. EN: Simple but messy and risky for large CSV files. | |
| External absolute path only | VI: Tránh lưu trong repo nhưng giảm reproducibility. EN: Avoids repo storage but hurts reproducibility. | |

**User's choice / Lựa chọn:** Fallback initially selected `data/raw/`; user follow-up locked explicit dataset naming.  
**Notes / Ghi chú:** Raw data should be local and ignored by Git. Dataset must be stated as IEEE-CIS Fraud Detection with required files `train_transaction.csv` and `train_identity.csv`.

---

## Join Integrity / Toàn vẹn join

| Option | Description / Mô tả | Selected |
|--------|----------------------|----------|
| Left join transaction to identity | VI: Giữ tất cả labeled transactions và coi identity missing là missing data. EN: Preserves all labeled transactions and treats missing identity as missing data. | yes |
| Inner join | VI: Drop transactions không có identity và có thể gây bias. EN: Drops transactions without identity rows and can bias the dataset. | |
| Ignore identity table | VI: Đơn giản nhưng vi phạm requirement. EN: Simpler but violates the project requirement. | |

**User's choice / Lựa chọn:** Fallback selected recommended option: left join.  
**Notes / Ghi chú:** Must assert joined row count equals transaction row count. / Cần assert joined row count bằng transaction row count.

---

## Split Strategy / Chiến lược split

| Option | Description / Mô tả | Selected |
|--------|----------------------|----------|
| Time-based 70/15/15 by `TransactionDT` | VI: Phù hợp chronology và tránh optimistic random leakage. EN: Best aligned with chronology and avoids optimistic random leakage. | yes |
| Stratified random split | VI: Dùng được để sanity check nhưng không phải main report. EN: Useful sanity check but not for the main report. | |
| Kaggle test files as test | VI: Không phù hợp vì public labels không có sẵn. EN: Not suitable because public labels are unavailable. | |

**User's choice / Lựa chọn:** Fallback selected recommended option: time-based split.  
**Notes / Ghi chú:** Validation/test must not refit preprocessing artifacts. / Validation/test không được refit preprocessing artifacts.

---

## Preprocessing Scope / Phạm vi preprocessing

| Option | Description / Mô tả | Selected |
|--------|----------------------|----------|
| First reproducible model-ready preprocessing | VI: Đủ cho baseline sau nhưng vẫn giữ Phase 1 gọn. EN: Enough for later baselines while keeping Phase 1 focused. | yes |
| Heavy feature engineering | VI: Để sau nếu baseline cần. EN: Belongs later if baselines require it. | |
| EDA only, no preprocessing module | VI: Quá yếu để handoff sang Phase 2. EN: Too weak for Phase 2 handoff. | |

**User's choice / Lựa chọn:** Fallback selected recommended option: first reproducible model-ready preprocessing.  
**Notes / Ghi chú:** Preserve `TransactionAmt` and `TransactionDT`; exclude `TransactionID` and `isFraud` from features.

---

## EDA Outputs / Đầu ra EDA

| Option | Description / Mô tả | Selected |
|--------|----------------------|----------|
| Dataset summary + imbalance + missing + amount plots | VI: Hỗ trợ trực tiếp report và metric choices sau. EN: Directly supports report requirements and later metric choices. | yes |
| Minimal shape/count printout only | VI: Quá mỏng cho final report. EN: Too thin for the final report. | |
| Full exploratory report with many feature plots | VI: Quá rộng cho Phase 1. EN: Too broad for Phase 1. | |

**User's choice / Lựa chọn:** Fallback selected recommended option: focused EDA outputs.  
**Notes / Ghi chú:** Fraud rate must be computed directly from data. / Fraud rate phải tính trực tiếp từ data.

---

## Output Organization / Tổ chức đầu ra

| Option | Description / Mô tả | Selected |
|--------|----------------------|----------|
| Scripts for reproducibility, notebooks for inspection | VI: Cân bằng giữa grading/report và repeatable pipeline. EN: Balances grading/report needs with repeatable execution. | yes |
| Notebook-only workflow | VI: Nhanh để sketch nhưng khó reproduce. EN: Faster to sketch but harder to reproduce. | |
| Script-only workflow | VI: Reproducible nhưng kém tiện cho EDA/report iteration. EN: Reproducible but less convenient for EDA/report iteration. | |

**User's choice / Lựa chọn:** Fallback selected recommended option: scripts plus notebooks.  
**Notes / Ghi chú:** Prefer `data/processed/`, `results/`, and `reports/figures/`.

---

## Local and Google Colab Compatibility / Tương thích local và Google Colab

| Option | Description / Mô tả | Selected |
|--------|----------------------|----------|
| Local-first, Colab-ready | VI: Tạo file local, nhưng mọi script/notebook dùng relative paths hoặc CLI args để upload lên Colab vẫn chạy. EN: Create files locally, but all scripts/notebooks use relative paths or CLI args so they still run after upload to Colab. | yes |
| Local-only | VI: Chỉ chạy trên máy Windows hiện tại. EN: Runs only on the current Windows machine. | |
| Colab-only | VI: Chỉ tối ưu cho Colab, bất tiện khi chạy local. EN: Optimized only for Colab, inconvenient locally. | |

**User's choice / Lựa chọn:** User explicitly requested local files that run normally after upload to Google Colab.  
**Notes / Ghi chú:** Planning should include `requirements.txt`, no hardcoded Windows paths, clear `data/raw/` dataset check, and a Colab-friendly notebook.

---

## the agent's Discretion / Phần agent được tự quyết

- Exact processed file format: Parquet preferred, CSV fallback acceptable.
- Exact plotting style.
- Exact rare-category threshold.
- Exact memory optimization strategy.
- Exact Colab notebook name, as long as setup instructions are clear.

## Deferred Ideas / Ý tưởng để sau

None. / Không có.
