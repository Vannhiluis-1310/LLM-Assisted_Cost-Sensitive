# Bản trình bày dự án

**Đề tài:** Ứng dụng LLM hỗ trợ học tăng cường nhạy cảm chi phí trong phát hiện gian lận giao dịch thương mại điện tử trên dữ liệu mất cân bằng  
**Tên tiếng Anh:** LLM-Assisted Cost-Sensitive Reinforcement Learning for E-commerce Transaction Fraud Detection on Imbalanced Data  
**Môn:** Bảo mật Thương mại Điện tử  
**Trạng thái hiện tại:** Dự án đã có kế hoạch, yêu cầu, roadmap và khung Phase 1 cho data pipeline, EDA, preprocessing. Các phần baseline ML, embedding, RL và đánh giá chi phí là phần triển khai tiếp theo.

---

## Slide 1. Tên đề tài

### Nội dung slide

- Ứng dụng LLM hỗ trợ học tăng cường nhạy cảm chi phí trong phát hiện gian lận giao dịch thương mại điện tử.
- Dataset chính: IEEE-CIS Fraud Detection.
- Trọng tâm: tối ưu chi phí phát hiện gian lận, không tối ưu Accuracy đơn thuần.
- Hướng tiếp cận: baseline ML, local embedding, contextual bandit.

### Lời trình bày

Đề tài của nhóm tập trung vào bài toán phát hiện gian lận giao dịch thương mại điện tử trên dữ liệu mất cân bằng. Điểm chính của bài không phải là đạt Accuracy cao, mà là giảm chi phí kinh tế do quyết định sai, đặc biệt là trường hợp bỏ sót giao dịch gian lận.

---

## Slide 2. Bối cảnh và vấn đề

### Nội dung slide

- Giao dịch gian lận thường chiếm tỷ lệ nhỏ hơn rất nhiều so với giao dịch hợp lệ.
- Nếu chỉ tối ưu Accuracy, mô hình có thể dự đoán đa số là hợp lệ và vẫn đạt điểm cao.
- Trong thực tế, bỏ sót fraud thường gây thiệt hại lớn hơn việc chặn nhầm giao dịch hợp lệ.
- Vì vậy, cần đánh giá theo chi phí FN/FP và khả năng bắt fraud.

### Lời trình bày

Dữ liệu fraud thường bị mất cân bằng mạnh. Một mô hình approve gần như tất cả giao dịch có thể có Accuracy cao, nhưng lại không hữu ích vì bỏ sót các giao dịch fraud có chi phí lớn. Vì vậy, nhóm đặt trọng tâm vào các metric như Recall fraud, PR-AUC, Total Cost và Cost Saving.

---

## Slide 3. Mục tiêu nghiên cứu

### Nội dung slide

- Xây dựng pipeline thực nghiệm có thể chạy lại trên IEEE-CIS Fraud Detection.
- So sánh baseline ML với mô hình ra quyết định nhạy cảm chi phí.
- Kiểm tra contextual bandit trong bài toán approve/block một bước.
- Đánh giá vai trò của local LLM embedding từ mô tả table-to-text trung lập.
- Phân tích kết quả theo chi phí, không chỉ theo phân loại đúng/sai.

### Lời trình bày

Mục tiêu của đề tài là xây dựng một pipeline đầy đủ từ dữ liệu đến đánh giá. Nhóm sẽ có baseline ML để làm mốc so sánh, sau đó triển khai contextual bandit để học chính sách approve hoặc flag/block. Thành phần LLM được giới hạn ở embedding local, không fine-tune và không dùng API quy mô lớn.

---

## Slide 4. Dataset và phạm vi dữ liệu

### Nội dung slide

- Dataset chính: IEEE-CIS Fraud Detection.
- File sử dụng:
  - `train_transaction.csv`
  - `train_identity.csv`
- Join bằng `TransactionID`.
- Cách join: left join từ transaction sang identity để giữ toàn bộ giao dịch có label.
- Label: `isFraud`, với `1 = fraud`, `0 = legitimate`.

### Lời trình bày

Trong dự án này, nhóm chỉ dùng IEEE-CIS Fraud Detection làm dataset chính. Hai file bắt buộc là `train_transaction.csv` và `train_identity.csv`. Do không phải giao dịch nào cũng có thông tin identity, nhóm dùng left join từ bảng transaction sang bảng identity để không làm mất giao dịch có nhãn.

---

## Slide 5. Vì sao không dùng Accuracy làm metric chính?

### Nội dung slide

- Dữ liệu fraud bị mất cân bằng.
- Accuracy dễ bị chi phối bởi lớp legitimate.
- Bỏ sót fraud tạo FN Cost lớn.
- Chặn nhầm legitimate tạo FP Cost nhưng thường nhỏ hơn FN.
- Metric chính:
  - PR-AUC
  - Recall fraud
  - Precision fraud
  - F1 fraud
  - FN Cost, FP Cost, Total Cost
  - Cost Saving

### Lời trình bày

Accuracy không phản ánh đúng mục tiêu kinh doanh của fraud detection. Trong bài toán này, chi phí của false negative thường cao hơn false positive, nên một mô hình tốt phải giảm được Total Cost và cải thiện Cost Saving so với approve-all hoặc baseline ML.

---

## Slide 6. Pipeline tổng thể

### Nội dung slide

1. Load hai file train của IEEE-CIS.
2. Left join theo `TransactionID`.
3. Tính fraud rate, missing ratio và EDA cơ bản.
4. Split theo thời gian dựa trên `TransactionDT`: 70% train, 15% validation, 15% test.
5. Preprocess train-only.
6. Train baseline ML và tune threshold theo cost.
7. Tạo table-to-text và MiniLM embedding.
8. Train contextual bandit.
9. Đánh giá bằng fraud metrics và cost metrics.

### Lời trình bày

Pipeline bắt đầu từ dữ liệu gốc, sau đó join, kiểm tra phân phối fraud, chia dữ liệu theo thời gian để giảm leakage, và tiền xử lý theo nguyên tắc fit trên train only. Các phase sau sẽ dùng dữ liệu này để train baseline, tạo embedding và huấn luyện contextual bandit.

---

## Slide 7. Phần đã có trong repo hiện tại

### Nội dung slide

- Kế hoạch dự án:
  - `.planning/PROJECT.md`
  - `.planning/REQUIREMENTS.md`
  - `.planning/ROADMAP.md`
  - `KE_HOACH_DU_AN.md`
- Data pipeline:
  - kiểm tra file IEEE-CIS bắt buộc
  - left join theo `TransactionID`
  - kiểm tra row count sau join
  - dataset summary, missing summary, split summary
  - biểu đồ class distribution, amount distribution, missing values
- Preprocessing:
  - numeric median imputation
  - categorical missing fill và one-hot encoding
  - fit transformer trên train only
  - lưu metadata và artifacts

### Lời trình bày

Ở trạng thái hiện tại, repo đã có nền tảng Phase 1. Script `make_dataset.py` chịu trách nhiệm load, join, kiểm tra integrity, split theo thời gian và tạo EDA outputs. Script `preprocess.py` tạo preprocessing đầu tiên cho baseline sau, với nguyên tắc fit trên train và chỉ transform validation/test.

---

## Slide 8. Thiết kế cost-sensitive

### Nội dung slide

Quyết định của mô hình:

- `0 = approve`
- `1 = flag/block`

Hàm chi phí:

```text
C_FN = TransactionAmt * beta
C_FP = TransactionAmt * alpha
Total Cost = FN Cost + FP Cost
```

Trong đó:

- FN: approve nhầm giao dịch fraud.
- FP: block nhầm giao dịch hợp lệ.
- `beta > alpha` vì bỏ sót fraud nghiêm trọng hơn.

### Lời trình bày

Mỗi giao dịch được xem là một quyết định approve hoặc flag/block. Nếu approve nhầm fraud, chi phí false negative được tính theo số tiền giao dịch nhân với hệ số beta. Nếu block nhầm giao dịch hợp lệ, chi phí false positive được tính theo alpha. Thiết kế này giúp mô hình tối ưu theo tác động kinh tế.

---

## Slide 9. Baseline ML dự kiến

### Nội dung slide

- Approve-all baseline để làm mốc cost.
- Logistic Regression với `class_weight`.
- Random Forest fallback hoặc LightGBM/XGBoost nếu môi trường hỗ trợ.
- Threshold tuning trên validation để minimize Total Cost.
- Không tune threshold trên test.

### Lời trình bày

Trước khi dùng RL, nhóm cần baseline ML rõ ràng. Approve-all dùng để đo chi phí tham chiếu, còn Logistic Regression và Random Forest hoặc LightGBM/XGBoost giúp tạo mốc so sánh thực tế. Điểm quan trọng là threshold không cố định ở 0.5 mà được chọn trên validation theo Total Cost.

---

## Slide 10. Vai trò của LLM/local embedding

### Nội dung slide

- Không fine-tune LLM.
- Không dùng API LLM trên toàn bộ dataset.
- Tạo mô tả table-to-text trung lập từ feature thật.
- Không đưa label, score hoặc từ ngữ rủi ro vào text.
- Embedding bằng `sentence-transformers/all-MiniLM-L6-v2`.
- So sánh RL không embedding với RL có embedding.

### Lời trình bày

LLM trong đề tài chỉ đóng vai trò hỗ trợ biểu diễn dữ liệu. Nhóm sẽ chuyển một phần thông tin giao dịch thành câu mô tả trung lập, sau đó dùng MiniLM để tạo embedding local. Quan trọng là text không được chứa label, prediction score hoặc từ ngữ như high risk, suspicious, fraud-like để tránh leakage.

---

## Slide 11. Thiết kế contextual bandit

### Nội dung slide

- Bài toán một bước, không có state transition dài hạn.
- State:
  - tabular features đã preprocess
  - tùy thí nghiệm có thêm MiniLM embedding
- Action:
  - approve
  - flag/block
- Reward:
  - âm của chi phí quyết định sai
- So sánh:
  - contextual bandit without embedding
  - contextual bandit with embedding

### Lời trình bày

Với mỗi giao dịch, hệ thống chỉ cần quyết định một lần là approve hay block. Vì vậy, contextual bandit phù hợp hơn các thuật toán RL phức tạp như PPO/A2C. Reward được thiết kế trực tiếp từ chi phí, nên mô hình học theo mục tiêu giảm Total Cost.

---

## Slide 12. Kế hoạch đánh giá

### Nội dung slide

- Đánh giá trên test split cuối cùng theo thời gian.
- Fraud metrics:
  - PR-AUC
  - Recall fraud
  - Precision fraud
  - F1 fraud
- Cost metrics:
  - FN Cost
  - FP Cost
  - Total Cost
  - Cost Saving
- Sensitivity analysis với nhiều cấu hình `alpha` và `beta`.
- Error analysis cho FN/FP chi phí cao.

### Lời trình bày

Phần đánh giá sẽ dùng test split cuối cùng sau khi mọi preprocessing và threshold tuning đã được fit trên train hoặc validation đúng quy tắc. Ngoài metric phân loại, nhóm sẽ phân tích chi phí và các trường hợp lỗi có chi phí cao để hiểu mô hình sai ở đâu.

---

## Slide 13. Kết quả hiện tại và phần cần hoàn thiện

### Nội dung slide

### Đã có

- Khung kế hoạch đầy đủ.
- Yêu cầu và roadmap 4 phase.
- Script download/validate dataset từ Kaggle.
- Script load, join, EDA, split theo thời gian.
- Script preprocessing train-only.

### Cần hoàn thiện

- Chạy pipeline với dữ liệu IEEE-CIS thật.
- Train baseline ML.
- Viết cost metrics và threshold tuning.
- Tạo table-to-text và MiniLM embedding.
- Train contextual bandit.
- Tạo bảng kết quả, biểu đồ và error analysis.

### Lời trình bày

Hiện tại dự án đã có phần khung và nền tảng triển khai dữ liệu, nhưng chưa có kết quả thực nghiệm cuối cùng trong repo. Các phần tiếp theo là chạy dữ liệu thật, xây baseline, triển khai cost metrics, embedding và contextual bandit để có kết quả so sánh.

---

## Slide 14. Rủi ro và cách kiểm soát

### Nội dung slide

- Dataset lớn và thiếu nhiều giá trị:
  - dùng imputation, missing indicator hoặc xử lý categorical rõ ràng.
- Data leakage:
  - split theo thời gian, fit preprocessing trên train only.
- Claim quá mức về LLM:
  - chỉ kết luận LLM embedding có ích nếu ablation chứng minh.
- RL quá phức tạp:
  - giữ MVP ở contextual bandit.
- Sai scope:
  - không mở rộng sang phishing, IDS, malware, RAG, backend/frontend phức tạp.

### Lời trình bày

Các rủi ro chính là leakage, mở rộng quá phạm vi và claim vượt quá kết quả. Vì vậy, nhóm giữ thiết kế đơn giản, có kiểm soát: split theo thời gian, preprocessing fit trên train, LLM chỉ là embedding local và RL là contextual bandit một bước.

---

## Slide 15. Kết luận

### Nội dung slide

- Đề tài tập trung vào fraud detection nhạy cảm chi phí trên dữ liệu mất cân bằng.
- IEEE-CIS được dùng làm dataset chính với join đúng theo `TransactionID`.
- Accuracy không phải metric chính; trọng tâm là PR-AUC, fraud recall và Total Cost.
- LLM được dùng giới hạn, local, không fine-tune.
- Contextual bandit là hướng RL phù hợp với quyết định approve/block một bước.
- Repo hiện đã có nền tảng Phase 1 và sẵn sàng đi tiếp sang baseline/cost metrics.

### Lời trình bày

Tóm lại, dự án hướng đến một pipeline có thể chạy lại và đánh giá được tác động kinh tế của mô hình fraud detection. Điểm khác biệt chính là không chỉ dự đoán fraud, mà tối ưu quyết định theo chi phí. Giai đoạn hiện tại đã đặt nền tảng dữ liệu và preprocessing, tạo điều kiện để triển khai baseline ML, embedding và contextual bandit ở các phase tiếp theo.

---

## Slide 16. Câu hỏi dự kiến khi bảo vệ

### Câu 1. Vì sao không dùng Accuracy?

Vì dữ liệu fraud mất cân bằng, Accuracy có thể cao dù mô hình bỏ sót nhiều fraud. Metric phù hợp hơn là PR-AUC, Recall fraud và Total Cost.

### Câu 2. Vì sao dùng left join?

Vì `train_transaction.csv` chứa toàn bộ giao dịch có label. Không phải giao dịch nào cũng có identity, nên inner join sẽ làm mất dữ liệu có nhãn và có thể gây bias.

### Câu 3. Vì sao dùng contextual bandit thay vì RL phức tạp?

Vì mỗi giao dịch là một quyết định một bước: approve hoặc block. Không cần mô hình hóa chuỗi hành động dài hạn, nên contextual bandit đơn giản, phù hợp và dễ giải thích hơn.

### Câu 4. LLM đóng góp gì?

LLM không được fine-tune. Vai trò của nó là tạo embedding local từ mô tả table-to-text trung lập, giúp bổ sung biểu diễn cho contextual bandit. Tác dụng thực tế sẽ được kiểm tra bằng ablation.

### Câu 5. Làm sao tránh leakage?

Nhóm split theo `TransactionDT`, fit imputer/encoder/scaler trên train only, tune threshold trên validation và chỉ dùng test để đánh giá cuối cùng.

---

## Gợi ý bố cục slide khi chuyển sang PowerPoint

| Slide | Hình/đồ thị nên dùng | Ghi chú |
|---:|---|---|
| 1 | Sơ đồ đơn giản approve/block | Giới thiệu đề tài |
| 2 | Bar chart class imbalance | Sau khi chạy dataset |
| 3 | Pipeline block diagram | Nêu mục tiêu |
| 4 | Sơ đồ join 2 bảng | Nhấn mạnh `TransactionID` |
| 5 | Accuracy vs cost explanation | Dùng ví dụ approve-all |
| 6 | Pipeline end-to-end | Từ raw data đến evaluation |
| 7 | Repo status checklist | Đã có và cần làm |
| 8 | Công thức cost | FN/FP cost |
| 9 | Bảng baseline | Điền sau khi có kết quả |
| 10 | Table-to-text example | Không chứa label/risk word |
| 11 | Contextual bandit diagram | State, action, reward |
| 12 | Metrics table | PR-AUC và cost metrics |
| 13 | Roadmap remaining | Phase 2 đến Phase 4 |
| 14 | Risk matrix | Leakage, scope, overclaim |
| 15 | Key takeaways | Kết luận |
| 16 | Q&A | Chuẩn bị bảo vệ |

---

## Ghi chú cần cập nhật sau khi chạy thực nghiệm

- Điền fraud rate thật từ `dataset_summary.csv`.
- Điền số dòng/số cột sau join.
- Thêm biểu đồ class distribution và amount distribution từ `reports/figures/`.
- Điền bảng baseline metrics sau Phase 2.
- Điền bảng RL ablation sau Phase 3.
- Chỉ kết luận embedding giúp cải thiện nếu `RL with embedding` tốt hơn rõ ràng theo Total Cost hoặc Cost Saving.
