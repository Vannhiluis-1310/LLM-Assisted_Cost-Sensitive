# Phase 1: Data Pipeline and EDA - Context

**Gathered / Ngày tạo:** 2026-05-24  
**Status / Trạng thái:** Ready for planning / Sẵn sàng lập kế hoạch

<domain>

## Phase Boundary / Ranh giới Phase

**VI:** Phase 1 chỉ tạo nền tảng dữ liệu cho dự án: cấu trúc repository cho data work, load train data của IEEE-CIS, left join giữa `train_transaction.csv` và `train_identity.csv`, tính fraud rate trực tiếp, split train/validation/test theo thời gian, chốt preprocessing ban đầu, và tạo EDA artifacts phục vụ các phase baseline/RL sau.

**EN:** Phase 1 delivers only the project data foundation: repository structure for data work, IEEE-CIS train data loading, left join between `train_transaction.csv` and `train_identity.csv`, direct fraud-rate calculation, time-based train/validation/test split, initial preprocessing decisions, and EDA artifacts needed for later baseline/RL phases.

**VI:** Phase này không train ML baselines, không tạo embeddings, và không implement RL. Các phần đó thuộc phase sau.

**EN:** This phase does not train ML baselines, create embeddings, or implement RL. Those belong to later phases.

</domain>

<decisions>

## Implementation Decisions / Quyết định triển khai

### Dataset Placement and Source Files / Vị trí dataset và file nguồn

- **D-00:** **VI:** Dataset chính bắt buộc là IEEE-CIS Fraud Detection trên Kaggle; Phase 1 chỉ yêu cầu hai file có label: `train_transaction.csv` và `train_identity.csv`. **EN:** The required main dataset is IEEE-CIS Fraud Detection from Kaggle; Phase 1 only requires the two labeled files: `train_transaction.csv` and `train_identity.csv`.
- **D-01:** **VI:** Raw IEEE-CIS files đặt dưới `data/raw/`. **EN:** Raw IEEE-CIS files should be placed under `data/raw/`.
- **D-02:** **VI:** Phase 1 dùng `data/raw/train_transaction.csv` và `data/raw/train_identity.csv` làm input bắt buộc. **EN:** Phase 1 uses `data/raw/train_transaction.csv` and `data/raw/train_identity.csv` as required inputs.
- **D-03:** **VI:** Kaggle test files không thuộc evaluation path chính vì không có public `isFraud` labels cho workflow này. **EN:** Kaggle test files are not part of the main evaluation path because public `isFraud` labels are unavailable for this workflow.
- **D-04:** **VI:** Raw dataset files không commit vào Git; plan/implementation cần thêm ignore rules cho dữ liệu lớn. **EN:** Raw dataset files should not be committed to Git; planning/implementation should add ignore rules for large local data artifacts.

### Join and Data Integrity Checks / Join và kiểm tra toàn vẹn dữ liệu

- **D-05:** **VI:** Left join từ `train_transaction.csv` sang `train_identity.csv` theo `TransactionID`. **EN:** Use a left join from `train_transaction.csv` to `train_identity.csv` on `TransactionID`.
- **D-06:** **VI:** Sau join, assert số dòng bằng số dòng của `train_transaction.csv`. **EN:** After the join, assert that the row count equals the row count of `train_transaction.csv`.
- **D-07:** **VI:** Identity missing là missing information hợp lệ, không phải lý do drop row. **EN:** Missing identity rows are valid missing information, not a reason to drop rows.
- **D-08:** **VI:** Báo cáo identity coverage: total transaction rows, identity-matched rows, missing-identity rows. **EN:** Report identity coverage: total transaction rows, identity-matched rows, and missing-identity rows.

### Split Strategy / Chiến lược chia dữ liệu

- **D-09:** **VI:** Main split theo thời gian: sort `TransactionDT`, rồi 70% train, 15% validation, 15% test. **EN:** Main split is time-based: sort by `TransactionDT`, then use 70% train, 15% validation, and 15% test.
- **D-10:** **VI:** Stratified random split chỉ dùng làm sanity check, không dùng làm kết quả chính. **EN:** Stratified random split may be used only as a sanity check, not as the main reported result.
- **D-11:** **VI:** Mọi fitting step của preprocessing phải fit trên train only; validation/test chỉ transform, không refit. **EN:** All preprocessing fitting steps must be fit on train only; validation/test are transformed without refitting.

### Preprocessing Scope for Phase 1 / Phạm vi preprocessing của Phase 1

- **D-12:** **VI:** Phase 1 tạo preprocessing module reproducible, nhưng chỉ cần phiên bản model-ready đầu tiên cho baseline sau. **EN:** Phase 1 should create a reproducible preprocessing module, but only needs the first model-ready version for later baselines.
- **D-13:** **VI:** Numeric features dùng median imputation fit trên train. **EN:** Numeric features use median imputation fit on train.
- **D-14:** **VI:** Categorical features dùng explicit `"missing"` và rare-category grouping khi cần. **EN:** Categorical features use explicit `"missing"` values and rare-category grouping when needed.
- **D-15:** **VI:** `TransactionID` và `isFraud` không bao giờ đưa vào model features. **EN:** `TransactionID` and `isFraud` must never be included as model features.
- **D-16:** **VI:** Giữ `TransactionAmt` và `TransactionDT` trong processed outputs cho cost metrics và time analysis sau. **EN:** Preserve `TransactionAmt` and `TransactionDT` in processed outputs for later cost metrics and time-based analysis.
- **D-17:** **VI:** Pipeline phải lưu metadata đủ để chứng minh không fit imputer, encoder, scaler, PCA, hoặc threshold logic trên test. **EN:** The pipeline should save enough metadata to prove that no imputer, encoder, scaler, PCA, or threshold logic was fit on test data.

### EDA Outputs / Đầu ra EDA

- **D-18:** **VI:** EDA gồm dataset shape, joined shape, fraud count, legitimate count, fraud rate tính trực tiếp từ data. **EN:** EDA must include dataset shape, joined shape, fraud count, legitimate count, and fraud rate calculated directly from loaded data.
- **D-19:** **VI:** EDA gồm missing-value summaries, đặc biệt top missing columns sau identity join. **EN:** EDA must include missing-value summaries, especially top missing columns after the identity join.
- **D-20:** **VI:** EDA gồm class distribution và `TransactionAmt` distribution theo fraud vs legitimate. **EN:** EDA must include class distribution and `TransactionAmt` distribution by fraud vs legitimate.
- **D-21:** **VI:** Accuracy không phải success metric của Phase 1; Phase 1 chuẩn bị bằng chứng giải thích vì sao accuracy yếu với imbalanced fraud data. **EN:** Accuracy should not be a Phase 1 success metric; Phase 1 should prepare evidence explaining why accuracy is weak for imbalanced fraud data.

### Output Organization / Tổ chức đầu ra

- **D-22:** **VI:** Script outputs ưu tiên lưu processed split files dưới `data/processed/`. **EN:** Preferred script outputs are processed split files under `data/processed/`.
- **D-23:** **VI:** EDA outputs ưu tiên lưu tables dưới `results/` và figures dưới `reports/figures/`. **EN:** Preferred EDA outputs are tables under `results/` and figures under `reports/figures/`.
- **D-24:** **VI:** Notebook chỉ dùng để inspection/reporting; data steps reproducible phải nằm trong scripts/modules. **EN:** Use notebooks only for inspection and reporting; reproducible data steps should live in scripts/modules.

### Local-first and Google Colab-ready / Local-first và chạy được trên Google Colab

- **D-25:** **VI:** Tạo file ở local trước, nhưng code/notebook phải chạy được khi upload cả project folder lên Google Colab. **EN:** Create files locally first, but code/notebooks must run when the whole project folder is uploaded to Google Colab.
- **D-26:** **VI:** Không hardcode đường dẫn Windows như `C:\Users\...`; mọi script phải dùng relative paths từ project root hoặc nhận CLI arguments như `--raw-dir`, `--processed-dir`, `--reports-dir`. **EN:** Do not hardcode Windows paths such as `C:\Users\...`; all scripts must use paths relative to the project root or accept CLI arguments such as `--raw-dir`, `--processed-dir`, and `--reports-dir`.
- **D-27:** **VI:** Colab flow mặc định: upload/clone project folder, upload hai CSV vào `data/raw/`, install dependencies từ `requirements.txt`, rồi chạy notebook/script. **EN:** Default Colab flow: upload/clone the project folder, upload the two CSV files into `data/raw/`, install dependencies from `requirements.txt`, then run the notebook/scripts.
- **D-28:** **VI:** Notebook Phase 1 phải có cell kiểm tra môi trường Colab/local và in rõ file nào còn thiếu nếu dataset chưa có trong `data/raw/`. **EN:** The Phase 1 notebook must include a cell that detects Colab/local execution and clearly prints which files are missing if the dataset is not in `data/raw/`.
- **D-29:** **VI:** Ưu tiên tạo một notebook Colab-friendly, ví dụ `notebooks/01_data_check_colab.ipynb` hoặc `notebooks/01_data_check.ipynb` có phần Colab setup rõ ràng. **EN:** Prefer creating a Colab-friendly notebook, for example `notebooks/01_data_check_colab.ipynb` or `notebooks/01_data_check.ipynb` with a clear Colab setup section.

### the agent's Discretion / Phần agent được tự quyết

- **VI:** File format cho processed splits linh hoạt; Parquet được ưu tiên nếu dependencies có sẵn, CSV fallback được chấp nhận.  
  **EN:** The processed split file format is flexible; Parquet is preferred if dependencies are available, CSV fallback is acceptable.
- **VI:** Nếu Parquet gây lỗi dependency trên Colab, dùng CSV hoặc joblib/pickle cho artifacts trung gian là chấp nhận được.  
  **EN:** If Parquet causes dependency issues on Colab, CSV or joblib/pickle for intermediate artifacts is acceptable.
- **VI:** Memory optimization strategy linh hoạt miễn là row counts, labels, và core columns đúng.  
  **EN:** The memory optimization strategy is flexible as long as row counts, labels, and core columns remain correct.
- **VI:** Plotting style linh hoạt; figures phải đơn giản, dễ đọc, dùng được trong báo cáo.  
  **EN:** Plotting style is flexible; figures should be simple, readable, and report-ready.
- **VI:** Rare-category threshold có thể chọn khi triển khai và document trong code/config.  
  **EN:** The rare-category threshold can be chosen during implementation and documented in code/config.

</decisions>

<specifics>

## Specific Ideas / Ý tưởng cụ thể

- **VI:** Giữ Phase 1 thật ổn định và đáng tin: mục tiêu là làm các so sánh ML/RL sau này có nền tảng đúng.  
  **EN:** Keep Phase 1 deliberately stable and reliable: the goal is to make later ML/RL comparisons trustworthy.
- **VI:** Mọi fraud-rate number trong báo cáo phải tính từ dataset đã load, không lấy từ trí nhớ hoặc nguồn ngoài.  
  **EN:** Every reported fraud-rate number must be computed from the loaded dataset, not copied from memory or outside sources.
- **VI:** Khi viết README/notebook, dataset phải được nêu rõ: IEEE-CIS Fraud Detection, gồm `train_transaction.csv` và `train_identity.csv`.  
  **EN:** README/notebook instructions must explicitly name the dataset: IEEE-CIS Fraud Detection, consisting of `train_transaction.csv` and `train_identity.csv`.
- **VI:** Join step phải chứng minh rõ không mất labeled transaction nào.  
  **EN:** The join step should visibly prove that no labeled transaction was lost.
- **VI:** Split step phải dễ giải thích trong final report: `TransactionDT` order approximates transaction chronology.  
  **EN:** The split step should be easy to explain in the final report: `TransactionDT` order approximates transaction chronology.

</specifics>

<canonical_refs>

## Canonical References / Tài liệu tham chiếu chuẩn

**Downstream agents MUST read these before planning or implementing.**

### Project Scope / Phạm vi dự án

- `.planning/PROJECT.md` — **VI:** Core value, dataset constraints, LLM/RL guardrails, out-of-scope items. **EN:** Core value, dataset constraints, LLM/RL guardrails, and out-of-scope items.
- `.planning/REQUIREMENTS.md` — **VI:** Phase 1 requirements `DATA-01` đến `DATA-05` và `PREP-01` đến `PREP-04`. **EN:** Phase 1 requirements `DATA-01` through `DATA-05` and `PREP-01` through `PREP-04`.
- `.planning/ROADMAP.md` — **VI:** Phase 1 boundary, success criteria, suggested files, later-phase separation. **EN:** Phase 1 boundary, success criteria, suggested files, and later-phase separation.
- `KE_HOACH_DU_AN.md` — **VI:** Kế hoạch song ngữ đầy đủ, gồm dataset description, missing-value handling, split policy, và first 7 days. **EN:** Full bilingual project plan, including dataset description, missing-value handling, split policy, and first 7 days.

### Dataset / Bộ dữ liệu

- IEEE-CIS Fraud Detection — **VI:** Dataset chính bắt buộc của đề tài; Phase 1 dùng `train_transaction.csv` và `train_identity.csv`. **EN:** Required main dataset for the project; Phase 1 uses `train_transaction.csv` and `train_identity.csv`.

</canonical_refs>

<code_context>

## Existing Code Insights / Hiểu biết về code hiện có

### Reusable Assets / Tài nguyên tái sử dụng

- **VI:** Chưa có source code, notebook, hoặc data pipeline module trong repository tại thời điểm discuss.  
  **EN:** No existing source code, notebooks, or data pipeline modules were found in the repository at discussion time.

### Established Patterns / Pattern đã có

- **VI:** Planning files đang dùng `.planning/` cho GSD workflow state.  
  **EN:** Planning files already use `.planning/` for GSD workflow state.
- **VI:** Project documentation ưu tiên Markdown song ngữ, giữ technical terms bằng English khi hữu ích.  
  **EN:** Project documentation is Markdown-first and bilingual, preserving technical terms in English when useful.

### Integration Points / Điểm tích hợp

- `src/data/make_dataset.py` — **VI:** Loading, joining, integrity checks, splitting. **EN:** Loading, joining, integrity checks, and splitting.
- `src/features/preprocess.py` — **VI:** First-pass preprocessing cho baselines sau. **EN:** First-pass preprocessing for later baselines.
- `notebooks/01_data_check.ipynb` hoặc `notebooks/01_data_check_colab.ipynb` — **VI:** Notebook kiểm tra dữ liệu, có hướng dẫn chạy local/Colab. **EN:** Data-check notebook with local/Colab instructions.
- `requirements.txt` — **VI:** Dependencies tối thiểu để chạy local và Google Colab. **EN:** Minimal dependencies for both local and Google Colab execution.
- `results/` và `reports/figures/` — **VI:** EDA/report artifacts. **EN:** EDA/report artifacts.

</code_context>

<deferred>

## Deferred Ideas / Ý tưởng để sau

**VI:** Không có — discussion vẫn nằm trong Phase 1 scope.  
**EN:** None — discussion stayed within Phase 1 scope.

</deferred>

---

*Phase: 01-data-pipeline-and-eda*  
*Context gathered: 2026-05-24*  
*Bilingual conversion: 2026-05-24*
