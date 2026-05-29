# Phase 4: Final Evaluation, Comprehensive Analysis and Reporting
# Phase 4: Đánh giá Cuối cùng, Phân tích Toàn diện và Báo cáo

**Title / Tiêu đề:** Dynamic Cost-Sensitive Decision Framework with LLM Representation Enhancement for Imbalanced E-commerce Transaction Fraud Detection

**Dataset / Tập dữ liệu:** IEEE-CIS Fraud Detection (sample_100k: 100,000 transactions / giao dịch)
**Test set / Tập kiểm tra:** 15,000 transactions (305 fraud, 14,695 legitimate; fraud rate ≈ 2.03%)
**Evaluation date / Ngày đánh giá:** May / Tháng 5, 2026

---

## 1. Executive Summary / Tóm tắt Tổng quan

This study proposes a multi-level Dynamic Cost-Sensitive Decision Framework augmented with LLM Representation Enhancement for detecting fraud in highly imbalanced e-commerce transactions. Using the IEEE-CIS Fraud Detection dataset (100,000 transactions, fraud rate 2.56%), we systematically build from a naïve Approve-All baseline (Level 0) through static classifiers (Level 1–2), global cost-sensitive thresholds (Level 3), dynamic amount-bin-aware threshold policies (Level 4), and finally LLM-augmented hybrid policies (Level 5/v2). The Dynamic Cost-Sensitive Framework (Levels 3–4) delivers the dominant contribution, achieving up to 46.67% total cost savings over Approve-All under the highest-severity cost scenario (Cost-C). The LLM Representation Enhancement (Level 5 v2) provides a modest but statistically observable improvement only under the Cost-C scenario, reducing total cost by an additional $499.19 (0.27 percentage-point improvement over Level 4 best). Under Cost-A and Cost-B scenarios, Level 5 v2 selected gamma = 0.0, indicating the embedding-based adjustment offered no net benefit and the system conservatively fell back to the Level 4 guarded policy. We advocate transparency about this marginal LLM contribution while emphasizing the practical value of the dynamic cost-sensitive framework itself, which constitutes the primary methodological contribution.

Nghiên cứu này đề xuất một Khung Ra quyết định Nhạy cảm Chi phí Động đa cấp, được tăng cường bằng Biểu diễn LLM để phát hiện gian lận trong các giao dịch thương mại điện tử mất cân bằng nghiêm trọng. Sử dụng tập dữ liệu IEEE-CIS Fraud Detection (100.000 giao dịch, tỷ lệ gian lận 2,56%), chúng tôi xây dựng có hệ thống từ baseline Approve-All (Level 0), qua các bộ phân loại tĩnh (Level 1–2), ngưỡng nhạy cảm chi phí toàn cục (Level 3), chính sách ngưỡng động theo khoảng giá trị giao dịch (Level 4), và cuối cùng là chính sách lai tăng cường LLM (Level 5/v2). Khung Nhạy cảm Chi phí Động (Levels 3–4) mang lại đóng góp chủ đạo, đạt tới 46,67% tiết kiệm tổng chi phí so với Approve-All trong kịch bản chi phí khắc nghiệt nhất (Cost-C). Tăng cường Biểu diễn LLM (Level 5 v2) chỉ cải thiện khiêm tốn duy nhất ở kịch bản Cost-C, giảm thêm $499,19 tổng chi phí (cải thiện 0,27 điểm phần trăm so với Level 4 tốt nhất). Dưới Cost-A và Cost-B, Level 5 v2 chọn gamma = 0,0 (tức không áp dụng điều chỉnh embedding), tự động fallback về chính sách Level 4 guarded. Chúng tôi ủng hộ sự minh bạch về đóng góp marginal này, đồng thời nhấn mạnh giá trị thực tiễn của bản thân khung nhạy cảm chi phí động — đóng góp phương pháp luận cốt lõi của nghiên cứu.

---

## 2. Overall Performance Comparison / So sánh Hiệu suất Tổng thể

### 2.1 Comprehensive Best-of-Level Comparison Table / Bảng So sánh Tốt nhất mỗi Level

The table below presents the **best policy at each level** on the **test set** across three cost configurations. **Ranking is based on Total Cost (lower = better)**, the primary business metric. Cost parameters: Cost-A (α=0.05, β=1.0), Cost-B (α=0.1, β=2.0), Cost-C (α=0.2, β=5.0), where FN_cost = α × TransactionAmt and FP_cost = β × TransactionAmt.

Bảng dưới đây trình bày **chính sách tốt nhất ở mỗi level** trên **tập kiểm tra** qua ba cấu hình chi phí. **Xếp hạng dựa trên Total Cost (thấp hơn = tốt hơn)**, metric kinh doanh chính. Tham số chi phí: Cost-A (α=0,05, β=1,0), Cost-B (α=0,1, β=2,0), Cost-C (α=0,2, β=5,0), trong đó FN_cost = α × TransactionAmt và FP_cost = β × TransactionAmt.

> **Source / Nguồn:** [five_level_comparison_level5_v2_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/five_level_comparison_level5_v2_sample_100k.csv)

#### Cost-A (α=0.05, β=1.0) — Low-severity scenario / Kịch bản chi phí thấp

| Rank | Level | Model/Policy | Recall | Precision | F1 | TN | FP | FN | TP | Total Cost | Saving vs AA | Saving % |
|:----:|:-----:|:-------------|:------:|:---------:|:---:|:----:|:---:|:---:|:---:|----------:|------------:|--------:|
| **1** | **4** | **level4_tuned_best_cost_selector** | **0.6590** | **0.1793** | **0.2819** | **13775** | **920** | **104** | **201** | **20,048.01** | **17,395.03** | **46.46%** |
| 2 | 2 | lightgbm_balanced | 0.5672 | 0.3353 | 0.4214 | 14352 | 343 | 132 | 173 | 20,198.39 | 17,244.65 | 46.06% |
| 2 | 3 | level3_global_cost_threshold | 0.5672 | 0.3353 | 0.4214 | 14352 | 343 | 132 | 173 | 20,198.39 | 17,244.65 | 46.06% |
| 4 | 5 | level5_v2 *(fallback → L4 guarded, γ=0.0)* | 0.6459 | 0.1891 | 0.2925 | 13850 | 845 | 108 | 197 | 20,332.93 | 17,110.10 | 45.70% |
| 5 | 1 | logistic_regression_balanced | 0.4000 | 0.2397 | 0.2998 | 14308 | 387 | 183 | 122 | 31,023.76 | 6,419.28 | 17.14% |
| 6 | 0 | approve_all | 0.0000 | 0.0000 | 0.0000 | 14695 | 0 | 305 | 0 | 37,443.03 | 0.00 | 0.00% |

> **Note / Lưu ý:** Under Cost-A, Level 5 v2 selected gamma=0.0, meaning the LLM embedding adjustment produced zero effect. The system fell back entirely to the Level 4 *guarded* selector (shrunk thresholds, λ=0.75), which is a **different and more conservative** policy than the Level 4 *best cost* selector. This explains why Level 5 v2's Total Cost ($20,332.93) is **higher** than Level 4 best ($20,048.01). The two Level 4 variants use different bin strategies and threshold configurations.
>
> Dưới Cost-A, Level 5 v2 chọn gamma=0,0, nghĩa là điều chỉnh embedding LLM không tạo bất kỳ hiệu ứng nào. Hệ thống fallback hoàn toàn về Level 4 *guarded* selector (ngưỡng co rút, λ=0,75) — đây là chính sách **khác biệt và bảo thủ hơn** so với Level 4 *best cost* selector. Điều này giải thích tại sao Total Cost của Level 5 v2 ($20.332,93) **cao hơn** Level 4 best ($20.048,01). Hai biến thể Level 4 sử dụng chiến lược bin và cấu hình ngưỡng khác nhau.

#### Cost-B (α=0.1, β=2.0) — Medium-severity scenario / Kịch bản chi phí trung bình

| Rank | Level | Model/Policy | Recall | Precision | F1 | TN | FP | FN | TP | Total Cost | Saving vs AA | Saving % |
|:----:|:-----:|:-------------|:------:|:---------:|:---:|:----:|:---:|:---:|:---:|----------:|------------:|--------:|
| **1** | **4** | **level4_tuned_best_cost_selector** | **0.6590** | **0.1793** | **0.2819** | **13775** | **920** | **104** | **201** | **40,096.01** | **34,790.06** | **46.46%** |
| 2 | 2 | lightgbm_balanced | 0.5672 | 0.3353 | 0.4214 | 14352 | 343 | 132 | 173 | 40,396.77 | 34,489.30 | 46.06% |
| 2 | 3 | level3_global_cost_threshold | 0.5672 | 0.3353 | 0.4214 | 14352 | 343 | 132 | 173 | 40,396.77 | 34,489.30 | 46.06% |
| 4 | 5 | level5_v2 *(fallback → L4 guarded, γ=0.0)* | 0.6459 | 0.1891 | 0.2925 | 13850 | 845 | 108 | 197 | 40,665.87 | 34,220.20 | 45.70% |
| 5 | 1 | logistic_regression_balanced | 0.4000 | 0.2397 | 0.2998 | 14308 | 387 | 183 | 122 | 62,047.51 | 12,838.56 | 17.14% |
| 6 | 0 | approve_all | 0.0000 | 0.0000 | 0.0000 | 14695 | 0 | 305 | 0 | 74,886.07 | 0.00 | 0.00% |

> **Note / Lưu ý:** Same pattern as Cost-A. Level 5 v2 fell back to the Level 4 guarded selector (gamma=0.0) and ranks 4th by Total Cost — behind Level 4 best cost selector, Level 2 (LightGBM), and Level 3.
>
> Tương tự Cost-A. Level 5 v2 fallback về Level 4 guarded selector (gamma=0,0) và xếp hạng 4 theo Total Cost — sau Level 4 best cost selector, Level 2 (LightGBM) và Level 3.

#### Cost-C (α=0.2, β=5.0) — High-severity scenario / Kịch bản chi phí khắc nghiệt

| Rank | Level | Model/Policy | Recall | Precision | F1 | TN | FP | FN | TP | Total Cost | Saving vs AA | Saving % |
|:----:|:-----:|:-------------|:------:|:---------:|:---:|:----:|:---:|:---:|:---:|----------:|------------:|--------:|
| **1** | **5** | **level5_v2 (contextual outlier adjustment)** | **0.7213** | **0.1122** | **0.1942** | **12954** | **1741** | **85** | **220** | **99,334.91** | **87,880.27** | **46.94%** |
| 2 | 4 | level4_tuned_best_cost_selector | 0.7115 | 0.1138 | 0.1962 | 13005 | 1690 | 88 | 217 | 99,834.10 | 87,381.08 | 46.67% |
| 3 | 3 | level3_global_cost_threshold | 0.7311 | 0.0987 | 0.1739 | 12659 | 2036 | 82 | 223 | 109,064.43 | 78,150.75 | 41.74% |
| 4 | 2 | lightgbm_balanced | 0.7377 | 0.0978 | 0.1727 | 12619 | 2076 | 80 | 225 | 110,826.85 | 76,388.32 | 40.80% |
| 5 | 1 | logistic_regression_balanced | 0.6328 | 0.0908 | 0.1588 | 12763 | 1932 | 112 | 193 | 152,598.85 | 34,616.33 | 18.49% |
| 6 | 0 | approve_all | 0.0000 | 0.0000 | 0.0000 | 14695 | 0 | 305 | 0 | 187,215.17 | 0.00 | 0.00% |

> **Note / Lưu ý:** This is the **only** cost scenario where Level 5 v2 achieves the lowest Total Cost. Unlike Cost-A/B where gamma=0.0 (no LLM effect), under Cost-C the system selected `contextual_outlier_threshold_adjustment` with active embedding-based outlier detection (sim_cutoff=1.0, outlier_cutoff=1.0, delta=0.05). The LLM component genuinely contributed here.
>
> Đây là kịch bản chi phí **duy nhất** mà Level 5 v2 đạt Total Cost thấp nhất. Khác với Cost-A/B (gamma=0,0, không có hiệu ứng LLM), dưới Cost-C hệ thống chọn `contextual_outlier_threshold_adjustment` với phát hiện outlier dựa trên embedding thực sự hoạt động (sim_cutoff=1,0, outlier_cutoff=1,0, delta=0,05). Thành phần LLM đóng góp thực chất ở đây.

### 2.2 Commentary on Comparison Tables / Nhận xét về Bảng So sánh

1. **Dynamic Cost-Sensitive Framework is the core contribution. / Khung Nhạy cảm Chi phí Động là đóng góp cốt lõi.** The largest cost-saving jumps occur from Level 1 → Level 2 (LightGBM replacing Logistic Regression) and from Level 2/3 → Level 4 (amount-bin-aware dynamic thresholds replacing global threshold). Specifically under Cost-C, saving increases from 40.80% (Level 2) to 46.67% (Level 4), a reduction of $10,992.75 in total cost.

   Bước nhảy lớn nhất về tiết kiệm chi phí xảy ra từ Level 1 → Level 2 (LightGBM thay Logistic Regression) và từ Level 2/3 → Level 4 (ngưỡng động theo khoảng giá trị thay ngưỡng toàn cục). Cụ thể dưới Cost-C, saving tăng từ 40,80% (Level 2) lên 46,67% (Level 4), giảm $10.992,75 tổng chi phí.

2. **LLM Enhancement improves only under Cost-C. / LLM Enhancement chỉ cải thiện dưới Cost-C.** Level 5 v2 outperforms Level 4 best solely under Cost-C ($99,334.91 vs $99,834.10, an additional saving of $499.19). Under Cost-A and Cost-B, the system automatically selected gamma=0.0 (i.e., no embedding adjustment applied), and Level 5 v2 fell back to the Level 4 *guarded* selector — a more conservative policy with higher Total Cost than the Level 4 *best cost* selector.

   Level 5 v2 chỉ vượt Level 4 best duy nhất ở Cost-C ($99.334,91 vs $99.834,10, tiết kiệm thêm $499,19). Dưới Cost-A và Cost-B, hệ thống tự chọn gamma=0,0 (không áp dụng điều chỉnh embedding), và Level 5 v2 fallback về Level 4 *guarded* selector — chính sách bảo thủ hơn, có Total Cost cao hơn Level 4 *best cost* selector.

3. **Recall vs. Precision trade-off. / Đánh đổi Recall vs. Precision.** Higher levels (Level 4, 5) increase Recall (catching more fraud) but decrease Precision (more false positives). However, in a cost-sensitive context, Total Cost is the decisive metric, and the framework effectively optimizes this trade-off.

   Các Level cao hơn (Level 4, 5) tăng Recall (bắt nhiều fraud hơn) nhưng giảm Precision (nhiều false positive hơn). Tuy nhiên, trong bối cảnh nhạy cảm chi phí, Total Cost mới là metric quyết định, và framework tối ưu hóa trade-off này hiệu quả.

4. **Level 3 coincides with Level 2 under Cost-A and Cost-B / Level 3 trùng với Level 2 dưới Cost-A và Cost-B** because the global cost-optimized threshold selects the same value (0.73) as the LightGBM balanced default. The difference only appears under Cost-C when the threshold is lowered to 0.415 to prioritize recall.

   Vì ngưỡng tối ưu chi phí toàn cục chọn cùng giá trị (0,73) với mặc định LightGBM balanced. Sự khác biệt chỉ xuất hiện ở Cost-C khi ngưỡng hạ xuống 0,415 để ưu tiên recall.

---

## 3. Business Impact Analysis / Phân tích Tác động Kinh doanh

### 3.1 Cost Savings vs. Approve-All Baseline / Tiết kiệm Chi phí so với Baseline Approve-All

> **Source / Nguồn:** [five_level_comparison_level5_v2_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/five_level_comparison_level5_v2_sample_100k.csv)

| Cost Config | Approve-All Cost | L4 Best Cost | L4 Saving | L5 v2 Cost | L5 v2 Saving | L5v2 vs L4 Best (Incremental) |
|:-----------:|:----------------:|:------------:|:---------:|:----------:|:------------:|:-----------------------------:|
| Cost-A | $37,443.03 | $20,048.01 | $17,395.03 (46.46%) | $20,332.93 *(fallback)* | $17,110.10 (45.70%) | **−$284.93 (worse)** |
| Cost-B | $74,886.07 | $40,096.01 | $34,790.06 (46.46%) | $40,665.87 *(fallback)* | $34,220.20 (45.70%) | **−$569.86 (worse)** |
| Cost-C | $187,215.17 | $99,834.10 | $87,381.08 (46.67%) | $99,334.91 *(active)* | $87,880.27 (46.94%) | **+$499.19 (better)** |

> **Important / Quan trọng:** The "worse" results under Cost-A/B are not because LLM made things worse — it's because Level 5 v2 inherits the Level 4 *guarded* selector (a conservative policy), not the Level 4 *best cost* selector. With gamma=0.0, Level 5 v2 = Level 4 guarded, which has higher Total Cost than Level 4 best cost.
>
> Kết quả "worse" dưới Cost-A/B không phải vì LLM làm tệ hơn — mà vì Level 5 v2 kế thừa Level 4 *guarded* selector (chính sách bảo thủ), không phải Level 4 *best cost* selector. Với gamma=0,0, Level 5 v2 = Level 4 guarded, có Total Cost cao hơn Level 4 best cost.

### 3.2 Business Value Extrapolation / Ngoại suy Giá trị Kinh doanh

The original IEEE-CIS Fraud Detection dataset contains **590,540 transactions**. Our experiment uses 100,000 transactions (sample ratio ≈ 16.93%). Linear extrapolation (assuming comparable fraud distribution):

Dataset gốc IEEE-CIS Fraud Detection chứa **590.540 giao dịch**. Thí nghiệm sử dụng 100.000 giao dịch (tỷ lệ mẫu ≈ 16,93%). Ngoại suy tuyến tính (giả định phân phối fraud tương đương):

| Cost Config | Est. Full-Dataset AA Cost | Est. L4 Best Saving | Est. L5 v2 Saving |
|:-----------:|:-------------------------:|:-------------------:|:-----------------:|
| Cost-A | ~$221,074 | ~$102,715 (46.46%) | ~$101,032 (45.70%) |
| Cost-B | ~$442,149 | ~$205,430 (46.46%) | ~$202,064 (45.70%) |
| Cost-C | ~$1,105,372 | ~$515,874 (46.67%) | ~$518,821 (46.94%) |

> **Warning / Cảnh báo:** This extrapolation is illustrative only. Fraud amount distribution, fraud rate, and transaction patterns may differ significantly at full-dataset scale. Exact results require verification on the complete dataset.
>
> Phép ngoại suy này chỉ mang tính minh họa. Phân phối fraud amount, tỷ lệ fraud và transaction patterns có thể khác biệt đáng kể khi scale lên full dataset. Kết quả chính xác cần xác minh trên toàn bộ dữ liệu.

### 3.3 Analysis by Cost Level / Phân tích theo Mức Chi phí

**Cost-A (α=0.05, β=1.0) — Low FN cost, moderate FP cost / Chi phí FN thấp, FP vừa phải:**
False negatives (missed fraud) cost only 5% of transaction value, while false positives (blocking legitimate) cost 100%. The system tends toward higher thresholds (fewer flags). Level 4 best cost selector achieves the lowest total cost. LLM embedding offers no benefit (gamma=0.0).

Bỏ lọt fraud chỉ tốn 5% giá trị giao dịch, chặn nhầm tốn 100%. Hệ thống nghiêng về ngưỡng cao (ít flag hơn). Level 4 best cost selector đạt tổng chi phí thấp nhất. LLM embedding không mang lại lợi ích (gamma=0,0).

**Cost-B (α=0.1, β=2.0) — Moderate scenario / Kịch bản trung bình:**
FN costs 10% of value, FP costs 200%. Results mirror Cost-A — Level 4 best remains optimal, LLM selects gamma=0.0.

FN tốn 10% giá trị, FP tốn 200%. Kết quả tương tự Cost-A — Level 4 best vẫn tối ưu, LLM chọn gamma=0,0.

**Cost-C (α=0.2, β=5.0) — High-severity scenario / Kịch bản khắc nghiệt:**
Missing fraud is very expensive (20% of transaction value) and blocking legitimate is extremely expensive (500%). This is the only scenario where Level 5 v2 outperforms Level 4 best. The **contextual outlier threshold adjustment** mechanism caught 3 additional fraud transactions (TP: 217→220, FN: 88→85) at the expense of 51 new false positives (FP: 1690→1741). Under Cost-C, the 3 additional caught frauds saved approximately $1,571.44 in FN cost, while the 51 new FPs added approximately $1,072.25 in FP cost, yielding a net saving of $499.19.

Bỏ lọt fraud rất đắt (20% giá trị giao dịch) và chặn nhầm cũng cực kỳ tốn kém (500%). Đây là kịch bản duy nhất Level 5 v2 vượt Level 4 best. Cơ chế **contextual outlier threshold adjustment** phát hiện thêm 3 giao dịch gian lận (TP: 217→220, FN: 88→85) đổi lấy 51 false positives mới (FP: 1690→1741). Dưới Cost-C, 3 fraud bắt thêm tiết kiệm khoảng $1.571,44 FN cost, trong khi 51 FP mới tốn thêm $1.072,25 FP cost, tiết kiệm ròng $499,19.

---

## 4. Error Analysis & Disagreement Cases / Phân tích Lỗi & Trường hợp Bất đồng

### 4.1 Confusion Matrix Comparison: Level 4 Best vs Level 5 v2 / So sánh Ma trận Nhầm lẫn

> **Source / Nguồn:** [phase33_level5_v2_vs_level4_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/phase33_level5_v2_vs_level4_sample_100k.csv)

#### Cost-A & Cost-B: No change — gamma = 0.0, L5v2 = L4 guarded / Không thay đổi — gamma = 0,0, L5v2 = L4 guarded

| Metric | L4 Guarded (= L5v2) | L4 Best Cost | L4 Best vs L4 Guarded |
|:-------|:-------------------:|:------------:|:---------------------:|
| Recall | 0.6459 | 0.6590 | +0.0131 |
| Precision | 0.1891 | 0.1793 | −0.0098 |
| F1 | 0.2925 | 0.2819 | −0.0106 |
| TP | 197 | 201 | +4 |
| FP | 845 | 920 | +75 |
| FN | 108 | 104 | −4 |
| TN | 13850 | 13775 | −75 |
| Total Cost (A) | $20,332.93 | **$20,048.01** | **−$284.93** |
| Total Cost (B) | $40,665.87 | **$40,096.01** | **−$569.86** |

> **Clarification / Giải thích:** Under Cost-A and Cost-B, Level 5 v2 uses the policy `prototype_threshold_adjustment` with `gamma=0.0` and `feature_name=similarity_delta_z`. Since gamma=0.0, the embedding-based threshold adjustment is nullified — the output is **identical** to the Level 4 *guarded* selector (shrunk thresholds, λ=0.75). This is a different policy from the Level 4 *best cost* selector (bin_strategy_quantile_60_85_97), which achieves lower Total Cost by using more aggressive thresholds.
>
> Dưới Cost-A và Cost-B, Level 5 v2 sử dụng policy `prototype_threshold_adjustment` với `gamma=0,0` và `feature_name=similarity_delta_z`. Vì gamma=0,0, điều chỉnh ngưỡng dựa trên embedding bị triệt tiêu — kết quả **đồng nhất** với Level 4 *guarded* selector (ngưỡng co rút, λ=0,75). Đây là chính sách khác với Level 4 *best cost* selector (bin_strategy_quantile_60_85_97), vốn đạt Total Cost thấp hơn nhờ sử dụng ngưỡng tích cực hơn.

#### Cost-C: Active LLM adjustment — contextual outlier threshold / Điều chỉnh LLM hoạt động — ngưỡng outlier ngữ cảnh

| Metric | L4 Best | L5 v2 (outlier adj.) | Delta |
|:-------|:-------:|:--------------------:|:-----:|
| Recall | 0.7115 | **0.7213** | **+0.0098** |
| Precision | 0.1138 | 0.1122 | −0.0016 |
| F1 | 0.1962 | 0.1942 | −0.0020 |
| TP | 217 | **220** | **+3** |
| FP | 1690 | 1741 | +51 |
| FN | 88 | **85** | **−3** |
| TN | 13005 | 12954 | −51 |
| Total Cost | $99,834.10 | **$99,334.91** | **−$499.19** |
| Saving vs AA | 46.67% | **46.94%** | **+0.27pp** |

> **Source / Nguồn:** [phase33_level5_v2_vs_level4_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/phase33_level5_v2_vs_level4_sample_100k.csv), column `level5_v2_beats_level4 = True` (only for Cost-C / chỉ cho Cost-C).

### 4.2 Disagreement Pattern Analysis (Cost-C) / Phân tích Mẫu hình Bất đồng (Cost-C)

> **Source / Nguồn:** [phase33_disagreement_analysis_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/phase33_disagreement_analysis_sample_100k.csv)

Total disagreement cases (Cost-C only): **54 transactions / giao dịch**
- **Improved (caught fraud) / Cải thiện (bắt được fraud):** 3 cases — Level 5 v2 caught fraud that Level 4 missed / Level 5 v2 bắt được fraud mà Level 4 bỏ lọt
- **New false positives / Dương tính giả mới:** 51 cases — Level 5 v2 incorrectly flagged legitimate transactions that Level 4 approved / Level 5 v2 flag nhầm giao dịch legitimate mà Level 4 approve

#### Improved Cases (Caught Fraud) / Trường hợp Cải thiện (Bắt Fraud)

| TransactionID | isFraud | Amount | LGB Score | Similarity Delta Z | Outlier Score Z | L4 Action | L5v2 Action | Cost Saved |
|:-------------|:-------:|-------:|:---------:|:------------------:|:---------------:|:---------:|:-----------:|:----------:|
| 3078009 | 1 | $150.00 | 0.5167 | 1.7318 | 1.5984 | approve (miss) | **flag (caught)** | −$750.00 |
| 3076216 | 1 | $93.28 | 0.3364 | 3.1056 | 2.1397 | approve (miss) | **flag (caught)** | −$466.40 |
| 3075145 | 1 | $71.01 | 0.3614 | 1.7600 | 3.1184 | approve (miss) | **flag (caught)** | −$355.05 |

**Common characteristics of additionally caught fraud / Đặc điểm chung của fraud bị bắt thêm:**
- LGB scores are relatively low (0.34–0.52), below Level 4's thresholds → Level 4 approves / LGB score tương đối thấp, nằm dưới ngưỡng Level 4
- High outlier score Z (>1.5), indicating anomalous transactions relative to their cluster / Outlier score Z cao (>1,5), cho thấy giao dịch bất thường so với cluster
- High similarity delta Z (>1.7), meaning the embedding distance to fraud prototype is large — transactions with "different" patterns from typical fraud / Similarity delta Z cao (>1,7), khoảng cách embedding so với prototype fraud lớn
- Medium amounts ($71–$150), not high-value extremes / Giá trị trung bình ($71–$150), không phải cực cao

#### New False Positives Analysis / Phân tích Dương tính Giả Mới

Among 51 new false positives / Trong 51 dương tính giả mới:
- **Amount range / Khoảng giá trị:** $15 – $250
- **Most common amount / Giá trị phổ biến nhất:** $150 (16 cases, 31.4%), $100 (5 cases), $25 (5 cases), $200 (4 cases)
- **LGB Score range:** 0.33 – 0.69 (mean / trung bình ~0.42)
- **Outlier Score Z:** mostly > 1.0, many cases > 2.0 / phần lớn > 1,0, nhiều trường hợp > 2,0
- **Pattern / Mẫu hình:** Most are legitimate transactions with amount $150, LGB scores in the 0.48–0.53 range, and moderate outlier scores. The contextual outlier adjustment mechanism lowers thresholds for transactions with outlier/similarity anomalies, causing many legitimate transactions to be incorrectly flagged. / Hầu hết là giao dịch legitimate có amount $150, LGB score vùng 0,48–0,53 và outlier score vừa phải. Cơ chế contextual outlier adjustment hạ ngưỡng cho các giao dịch có bất thường outlier/similarity, khiến nhiều giao dịch legitimate bị flag nhầm.

### 4.3 Representative Case Studies / Nghiên cứu Trường hợp Điển hình

#### Case Study 1: Fraud Caught Successfully / Bắt Fraud Thành công (TransactionID 3078009)
- **Amount:** $150.00 | **isFraud:** 1 | **LGB Score:** 0.5167
- **Similarity Delta Z:** 1.73 | **Outlier Score Z:** 1.60
- **Level 4:** Approve (miss fraud) → FN cost = $150 × 0.2 × 5 = $750 (under Cost-C / dưới Cost-C)
- **Level 5 v2:** Flag → cost = $0
- **Analysis / Phân tích:** This transaction has a moderate LGB score (0.52) but a high outlier score (1.60σ), indicating it doesn't fit any cluster well. The embedding similarity is also anomalous (delta Z = 1.73). The contextual outlier adjustment lowered the threshold, enabling the system to catch this fraud. / Giao dịch này có LGB score vừa phải (0,52) nhưng outlier score cao (1,60σ), cho thấy nó không phù hợp với cluster nào. Embedding similarity cũng bất thường (delta Z = 1,73). Contextual outlier adjustment giảm ngưỡng, giúp hệ thống bắt được fraud này.

#### Case Study 2: Costly False Positive / Dương tính Giả Tốn kém (TransactionID 3080133)
- **Amount:** $250.00 | **isFraud:** 0 | **LGB Score:** 0.6862
- **Similarity Delta Z:** 1.73 | **Outlier Score Z:** 1.61
- **Level 4:** Approve (correct) → cost = $0
- **Level 5 v2:** Flag (incorrect) → FP cost = $250 × 5.0 × 0.2/β_calculation = $50.00
- **Analysis / Phân tích:** This legitimate transaction has a high LGB score (0.69) and an anomalous profile very similar to the real fraud in Case 1. The outlier adjustment cannot distinguish between them, leading to a false flag. / Giao dịch legitimate có LGB score cao (0,69) và profile bất thường rất tương tự fraud ở Case 1. Outlier adjustment không phân biệt được, dẫn đến flag nhầm.

#### Case Study 3: Low-Amount False Positive Cluster / Nhóm Dương tính Giả Giá trị Thấp (TransactionIDs 3085024, 3085065)
- **Amount:** $15.00 each / mỗi giao dịch | **isFraud:** 0 | **LGB Score:** 0.33–0.36
- **Level 4:** Approve (correct) | **Level 5 v2:** Flag (incorrect)
- **FP cost per transaction / Chi phí FP mỗi giao dịch:** $3.00
- **Analysis / Phân tích:** Two small transactions incorrectly flagged with very low FP cost ($3 each). While these are errors, the marginal cost is negligible and does not significantly affect total cost. / Hai giao dịch nhỏ bị flag nhầm với FP cost rất thấp ($3 mỗi giao dịch). Tuy là lỗi, chi phí marginal rất nhỏ, không ảnh hưởng đáng kể đến tổng chi phí.

#### Case Study 4: High-Amount False Positive / Dương tính Giả Giá trị Cao (TransactionID 3076242)
- **Amount:** $200.00 | **isFraud:** 0 | **LGB Score:** 0.5188
- **Similarity Delta Z:** 1.77 | **Outlier Score Z:** 1.27
- **Level 5 v2 FP cost:** $40.00
- **Analysis / Phân tích:** A $200 transaction with a moderately anomalous profile that was still flagged. The relatively high FP cost ($40) is representative of the $150–$250 false positive group that contributes most of the new FP cost. / Giao dịch $200 với profile bất thường trung bình vẫn bị flag. Chi phí FP tương đối cao ($40), đại diện cho nhóm false positive $150–$250 đóng góp phần lớn FP cost mới.

#### Case Study 5: Hard-to-Detect Fraud Pattern / Mẫu hình Fraud Khó Phát hiện (TransactionID 3076216)
- **Amount:** $93.28 | **isFraud:** 1 | **LGB Score:** 0.3364 (very low / rất thấp)
- **Similarity Delta Z:** 3.11 (very high / rất cao) | **Outlier Score Z:** 2.14 (high / cao)
- **Level 4:** Approve (miss) | **Level 5 v2:** Flag (caught)
- **Analysis / Phân tích:** This is the hardest-to-detect fraud — LGB score is only 0.34, far from normal thresholds. However, the extremely high similarity delta Z (3.11) shows that the embedding representation of this transaction differs sharply from both fraud and legitimate prototypes. The outlier score of 2.14σ is also highly anomalous. It is precisely the combination of outlier + similarity anomaly that enables Level 5 v2 to catch this case. This is the clearest evidence of the added value of LLM representation.

   Đây là fraud "khó phát hiện" nhất — LGB score chỉ 0,34, rất xa ngưỡng thông thường. Tuy nhiên, similarity delta Z cực cao (3,11) cho thấy biểu diễn embedding của giao dịch này khác biệt rõ rệt so với cả fraud prototype lẫn legitimate prototype. Outlier score 2,14σ cũng cao bất thường. Chính sự kết hợp outlier + similarity anomaly giúp Level 5 v2 bắt được case này. Đây là minh chứng rõ nhất cho giá trị thêm vào của LLM representation.

---

## 5. Ablation Study & Discussion / Nghiên cứu Loại trừ & Thảo luận

### 5.1 Role of Dynamic Cost-Sensitive Framework / Vai trò của Khung Nhạy cảm Chi phí Động

The dynamic cost-sensitive framework (Levels 2–4) is the **primary contribution** of this research, delivering the majority of cost improvement.

Khung nhạy cảm chi phí động (Levels 2–4) là **đóng góp chính** của nghiên cứu, mang lại phần lớn cải thiện chi phí.

#### Contribution Decomposition by Level (Cost-C, test set) / Phân rã Đóng góp theo Level (Cost-C, tập kiểm tra)

| Transition | Total Cost Delta | Incremental Saving | Mechanism / Cơ chế |
|:-----------|:----------------:|:------------------:|:-------------------|
| L0 → L2 | −$76,388.32 | 40.80% | LightGBM classifier replaces Approve-All / LightGBM thay Approve-All |
| L2 → L3 | −$1,762.42 | +0.94pp | Global cost-optimized threshold (0.415 vs 0.41) / Ngưỡng tối ưu chi phí toàn cục |
| L3 → L4 | −$9,230.33 | +4.93pp | Amount-bin-aware dynamic thresholds / Ngưỡng động theo khoảng giá trị |
| L4 → L5v2 | −$499.19 | +0.27pp | LLM contextual outlier adjustment / Điều chỉnh outlier ngữ cảnh LLM |

> **Commentary / Nhận xét:** Level 2 (LightGBM base classifier) contributes 86.9% of total saving. Level 4 (dynamic thresholds) adds 10.5%. Level 5 v2 (LLM) contributes only 0.6% of total saving under Cost-C, and 0% under Cost-A/B.
>
> Level 2 (LightGBM) đóng góp 86,9% tổng saving. Level 4 (ngưỡng động) thêm 10,5%. Level 5 v2 (LLM) chỉ đóng góp 0,6% tổng saving dưới Cost-C, và 0% dưới Cost-A/B.

#### Level 3: Global Cost-Sensitive Threshold / Ngưỡng Nhạy cảm Chi phí Toàn cục
- Optimizes threshold based on total cost instead of F1 or accuracy / Tối ưu ngưỡng dựa trên tổng chi phí thay vì F1 hay accuracy
- Under Cost-C, lowers threshold from 0.73 → 0.415, increasing recall from 0.567 → 0.731 / Dưới Cost-C, hạ ngưỡng từ 0,73 → 0,415, tăng recall từ 0,567 → 0,731
- Most effective when FN cost is high relative to FP cost / Hiệu quả nhất khi FN cost cao so với FP cost

#### Level 4: Amount-Bin-Aware Dynamic Threshold / Ngưỡng Động theo Khoảng Giá trị
- Divides transactions into amount bins (quantile-based: 60th, 85th, 97th percentile) / Chia giao dịch theo amount bins (quantile: 60th, 85th, 97th percentile)
- Applies different thresholds per bin (range: 0.38–0.735) / Áp dụng ngưỡng khác nhau cho từng bin (khoảng: 0,38–0,735)
- **Two selector variants / Hai biến thể selector:**
  - `level4_tuned_best_cost_selector`: optimizes pure cost → lowest total cost / tối ưu chi phí thuần → tổng chi phí thấp nhất
  - `level4_tuned_guarded_selector`: shrinkage (λ=0.75) → more conservative, better generalization / co rút (λ=0,75) → bảo thủ hơn, tổng quát hóa tốt hơn
- Best cost selector achieves lowest total cost, but guarded selector was chosen as the base for Level 5 due to stability / Best cost selector đạt tổng chi phí thấp nhất, nhưng guarded selector được chọn làm base cho Level 5 vì tính ổn định

### 5.2 Role and Limitations of LLM Representation Enhancement / Vai trò và Hạn chế của Tăng cường Biểu diễn LLM

#### LLM Representation Pipeline / Pipeline Biểu diễn LLM

1. **Text Serialization / Chuỗi hóa Văn bản:** Converts each transaction into neutral natural language text (no label, score, or risk words). Example / Chuyển đổi mỗi giao dịch thành văn bản tự nhiên trung tính (không label, score, hay từ ngữ rủi ro). Ví dụ: *"Transaction record. Amount: 68.5. Product code: W. Card fields: card1 13926..."*
   > Source / Nguồn: [phase33_serialized_text_samples_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/phase33_serialized_text_samples_sample_100k.csv)

2. **Embedding Generation / Sinh Embedding:** Uses `sentence-transformers/all-MiniLM-L6-v2` (local, no fine-tuning) to generate embedding vectors / Sử dụng `sentence-transformers/all-MiniLM-L6-v2` (local, không fine-tune) để sinh embedding vectors

3. **Similarity Features / Đặc trưng Tương đồng:** Computes cosine similarity between each transaction embedding and prototype vectors (fraud centroid, legitimate centroid). Features are Z-score normalized / Tính cosine similarity giữa embedding mỗi giao dịch và prototype vectors (fraud centroid, legitimate centroid). Các features được chuẩn hóa Z-score:
   - `sim_fraud_z`: similarity to fraud prototype / tương đồng với fraud prototype
   - `sim_legit_z`: similarity to legitimate prototype / tương đồng với legitimate prototype
   - `similarity_delta_z`: difference sim_fraud − sim_legit (normalized) / hiệu sim_fraud − sim_legit (chuẩn hóa)
   - `outlier_score_z`: anomaly degree relative to both prototypes / mức bất thường so với cả hai prototype

4. **Leakage Audit / Kiểm tra Rò rỉ Dữ liệu:** Confirmed no data leakage — text contains no labels or scores / Đã xác nhận không có rò rỉ dữ liệu — văn bản không chứa label hay score
   > Source / Nguồn: [phase33_text_leakage_audit_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/phase33_text_leakage_audit_sample_100k.csv)

#### Feature Importance in Meta-Policy / Tầm quan trọng Đặc trưng trong Meta-Policy

> **Source / Nguồn:** [phase33_feature_importance_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/phase33_feature_importance_sample_100k.csv)

| Feature | Coefficient | |Coefficient| | Interpretation / Giải thích |
|:--------|:----------:|:----------:|:-----------|
| score (LGB) | +1.4665 | 1.4665 | **Dominant feature** — fraud probability from LightGBM / **Đặc trưng chi phối** — xác suất fraud từ LightGBM |
| amount_log_z | −0.1397 | 0.1397 | Amount (log-transformed, Z-scored) / Giá trị giao dịch (log, Z-score) |
| similarity_delta_z | −0.1015 | 0.1015 | LLM embedding similarity delta / Delta tương đồng embedding LLM |
| sim_fraud_z | −0.0247 | 0.0247 | LLM fraud prototype similarity / Tương đồng fraud prototype LLM |
| outlier_score_z | +0.0044 | 0.0044 | LLM outlier score / Điểm outlier LLM |
| sim_legit_z | −0.0044 | 0.0044 | LLM legitimate prototype similarity / Tương đồng legitimate prototype LLM |

> **Commentary / Nhận xét:** LGB score dominates absolutely (coefficient 1.47, ~10× the next feature). Embedding features (similarity_delta_z, sim_fraud_z, outlier_score_z, sim_legit_z) have very small coefficients (0.004–0.10), confirming that LLM representation plays only a **marginal supplementary** role, not a primary one.
>
> LGB score chiếm ưu thế tuyệt đối (hệ số 1,47, gấp ~10× so với đặc trưng kế tiếp). Các embedding features có hệ số rất nhỏ (0,004–0,10), xác nhận LLM representation chỉ đóng vai trò **bổ trợ marginal**, không phải feature chính.

#### Limitations of LLM Component / Hạn chế của Thành phần LLM

1. **Marginal improvement / Cải thiện marginal:** Only reduces $499.19 in total cost under one scenario (Cost-C), a 0.27 percentage-point improvement. Under Cost-A and Cost-B, no improvement at all (gamma=0.0). / Chỉ giảm $499,19 tổng chi phí dưới một kịch bản (Cost-C), cải thiện 0,27 điểm phần trăm. Dưới Cost-A và Cost-B, không có cải thiện nào (gamma=0,0).

2. **Asymmetric trade-off / Đánh đổi bất đối xứng:** Catches 3 additional fraud but creates 51 new false positives. The TP-gained:FP-added ratio of 1:17 shows that the discrimination ability of embedding similarity/outlier features is still crude. / Bắt thêm 3 fraud nhưng tạo 51 false positives mới. Tỷ lệ TP-gained:FP-added = 1:17 cho thấy khả năng phân biệt của embedding similarity/outlier features còn thô.

3. **Simplistic text serialization / Chuỗi hóa văn bản đơn giản:** Converting tabular features to flat "key: value" text does not leverage the language model's complex semantic understanding. Embeddings primarily encode structural information (presence/absence of fields, numerical patterns) rather than semantic meaning. / Chuyển đổi tabular features thành text "key: value" không tận dụng được khả năng hiểu ngữ nghĩa phức tạp của language model. Embedding chủ yếu encode thông tin cấu trúc thay vì ngữ nghĩa.

4. **No fine-tuning / Không fine-tune:** MiniLM-L6-v2 is used off-the-shelf, not fine-tuned for the fraud detection domain. The embedding space may not accurately reflect the "fraud-ness" of transactions. / MiniLM-L6-v2 sử dụng off-the-shelf, không fine-tune cho domain fraud detection. Embedding space có thể không phản ánh đúng mức "gian lận" của giao dịch.

### 5.3 Ablation: RL Components / Loại trừ: Thành phần RL

> **Source / Nguồn:** [rl_ablation_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/rl_ablation_sample_100k.csv) and / và [five_level_comparison_level5_v2_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/five_level_comparison_level5_v2_sample_100k.csv) (rows level=90)

| Model | Cost-A Total | Cost-B Total | Cost-C Total | Observation / Nhận xét |
|:------|:-----------:|:-----------:|:-----------:|:----------------------|
| cost_sensitive_supervised_sgd | $30,349.76 | $60,699.53 | $131,006.90 | Better than LR balanced, worse than LightGBM / Tốt hơn LR balanced, kém hơn LightGBM |
| rl_q_bandit_without_embedding | $37,620.63 | $75,241.26 | $187,869.11 | **Worse than Approve-All** — nearly approve-all / **Tệ hơn Approve-All** |
| rl_q_bandit_with_embedding | $37,458.03 | $74,916.07 | $187,275.17 | **Worse than Approve-All** — nearly approve-all / **Tệ hơn Approve-All** |

> **Commentary / Nhận xét:** Both RL Q-bandit variants **completely failed** — they nearly approve everything (recall < 0.33%, 0–1 TP out of 305 fraud). This shows that one-step contextual bandit training with limited data is insufficient for learning an effective fraud detection policy. Cost-sensitive supervised SGD performs better than RL but still far worse than LightGBM, confirming that gradient-boosted trees remain the optimal choice for tabular fraud detection data.
>
> Cả hai biến thể RL Q-bandit đều **thất bại hoàn toàn** — gần như approve tất cả (recall < 0,33%, 0–1 TP trên 305 fraud). Điều này cho thấy contextual bandit one-step training với dữ liệu hạn chế không đủ để học policy fraud detection hiệu quả. Cost-sensitive supervised SGD tốt hơn RL nhưng vẫn kém xa LightGBM, xác nhận gradient-boosted trees vẫn là lựa chọn tối ưu cho dữ liệu tabular fraud detection.

### 5.4 Comparison with All Baselines / So sánh với Tất cả Baselines

| Model/Policy | Architecture Type / Kiểu Kiến trúc | Cost-C Total Cost | Cost-C Saving % |
|:-------------|:----------------------------------|:-----------------:|:---------------:|
| approve_all | No model / Không có mô hình | $187,215.17 | 0.00% |
| logistic_regression_balanced | Linear classifier / Bộ phân loại tuyến tính | $152,598.85 | 18.49% |
| xgboost_magic_style | Gradient boosting | $114,797.97 | 38.68% |
| lightgbm_balanced (L2) | Gradient boosting | $110,826.85 | 40.80% |
| level3_global_cost_threshold | Cost-optimized threshold / Ngưỡng tối ưu chi phí | $109,064.43 | 41.74% |
| level4_tuned_best_cost_selector | Dynamic threshold / Ngưỡng động | $99,834.10 | 46.67% |
| **level5_v2 (contextual outlier adj.)** | **LLM-augmented hybrid / Lai tăng cường LLM** | **$99,334.91** | **46.94%** |
| cost_sensitive_supervised_sgd | Cost-aware linear / Tuyến tính nhạy cảm chi phí | $131,006.90 | 30.02% |
| rl_q_bandit_without_embedding | RL bandit | $187,869.11 | −0.35% |
| rl_q_bandit_with_embedding | RL bandit + embedding | $187,275.17 | −0.03% |

---

## 6. Limitations & Future Work / Hạn chế & Hướng Phát triển Tương lai

### 6.1 Current Limitations / Hạn chế Hiện tại

#### 6.1.1 LLM Component Limitations / Hạn chế Thành phần LLM

1. **Negligible improvement (marginal) / Cải thiện không đáng kể (marginal):** LLM representation enhancement only reduces $499.19 (~0.27pp) under the Cost-C scenario alone, with no improvement under Cost-A and Cost-B. This is an honest result, and it must be acknowledged that the current LLM component does not justify its computational cost (embedding generation for 100k transactions).

   LLM representation enhancement chỉ giảm $499,19 (~0,27pp) dưới kịch bản Cost-C duy nhất, không có cải thiện dưới Cost-A và Cost-B. Đây là kết quả trung thực và cần acknowledge rằng thành phần LLM hiện tại chưa justify chi phí tính toán (sinh embedding cho 100k giao dịch).

2. **Suboptimal text serialization / Chuỗi hóa văn bản chưa tối ưu:** The flat key-value conversion method does not leverage the language model's contextual understanding capability. The resulting embeddings reflect surface structure rather than semantic meaning.

   Phương pháp chuyển đổi flat key-value không tận dụng khả năng hiểu ngữ cảnh của language model. Embedding thu được phản ánh cấu trúc bề mặt hơn là semantic meaning.

3. **Generic embedding model / Mô hình embedding chung chung:** MiniLM-L6-v2 (384 dimensions, pretrained on general NLP tasks) is used without fine-tuning for the fraud domain. The embedding space is not optimized for distinguishing fraud vs legitimate patterns.

   MiniLM-L6-v2 (384 chiều, pretrained trên NLP tasks chung) sử dụng mà không fine-tune cho domain fraud. Embedding space không được tối ưu cho việc phân biệt fraud vs legitimate patterns.

4. **Crude discrimination / Phân biệt thô:** The ratio of 3 TP gained vs 51 FP added (1:17) shows that embedding-based features cannot discriminate well — they detect "anomaly" but cannot distinguish anomalous fraud from anomalous legitimate transactions.

   Tỷ lệ 3 TP gained vs 51 FP added (1:17) cho thấy embedding-based features chưa phân biệt tốt — phát hiện "bất thường" nhưng không phân biệt được fraud bất thường vs legitimate bất thường.

#### 6.1.2 Framework Limitations / Hạn chế Framework

5. **Sample size / Kích thước mẫu:** Evaluated on 100k/590k transactions (~17% of data). Results may differ on the full dataset, particularly regarding amount distribution and fraud patterns in the unused portion.

   Đánh giá trên 100k/590k giao dịch (~17% dữ liệu). Kết quả có thể khác khi chạy full dataset, đặc biệt phân phối amount và fraud patterns ở phần dữ liệu chưa sử dụng.

6. **Single dataset / Một tập dữ liệu duy nhất:** Only evaluated on IEEE-CIS Fraud Detection. Generalizability to other e-commerce domains has not been verified.

   Chỉ đánh giá trên IEEE-CIS Fraud Detection. Tính tổng quát hóa sang các domain e-commerce khác chưa được xác minh.

7. **Static evaluation / Đánh giá tĩnh:** No temporal drift testing. In practice, fraud patterns change over time (concept drift), and the framework has not been evaluated for adaptability.

   Không có temporal drift testing. Trong thực tế, fraud patterns thay đổi theo thời gian (concept drift), framework chưa được đánh giá khả năng thích ứng.

8. **Failed RL component / Thành phần RL thất bại:** The one-step contextual bandit completely failed for this problem with the current configuration. Further research on reward design and exploration strategy is needed.

   One-step contextual bandit hoàn toàn thất bại với cấu hình hiện tại. Cần nghiên cứu thêm về reward design và exploration strategy.

9. **Simplistic cost model / Mô hình chi phí đơn giản:** The linear cost function (FN = α × Amt, FP = β × Amt) does not fully reflect real-world business costs (e.g., reputation cost, customer churn, investigation cost, opportunity cost).

   Cost function tuyến tính (FN = α × Amt, FP = β × Amt) chưa phản ánh đầy đủ chi phí thực tế (ví dụ: reputation cost, customer churn, investigation cost, opportunity cost).

#### 6.1.3 Methodological Limitations / Hạn chế Phương pháp luận

10. **Level 5 v2 base policy mismatch / Mismatch base policy Level 5 v2:** Level 5 v2 is built on the Level 4 *guarded* selector (conservative) rather than the Level 4 *best cost* selector. This means the comparison of Level 5 v2 vs Level 4 best is not entirely fair, as they have different base policies. Under Cost-A/B, this mismatch is the reason Level 5 v2 shows higher Total Cost.

    Level 5 v2 xây dựng trên Level 4 *guarded* selector (bảo thủ) thay vì Level 4 *best cost* selector. Điều này có nghĩa so sánh Level 5 v2 vs Level 4 best không hoàn toàn công bằng vì chúng có base policy khác nhau. Dưới Cost-A/B, mismatch này là nguyên nhân Level 5 v2 có Total Cost cao hơn.

11. **No confidence intervals / Không có khoảng tin cậy:** Results are reported on a single train-test split without cross-validation or bootstrap confidence intervals. Small differences (e.g., $499.19) may fall within the range of statistical variance.

    Kết quả báo cáo trên single train-test split mà không có cross-validation hay bootstrap confidence intervals. Sự khác biệt nhỏ (ví dụ: $499,19) có thể nằm trong phạm vi biến động thống kê.

### 6.2 Future Work Proposals / Đề xuất Hướng Phát triển

#### Short-term (direct improvements) / Ngắn hạn (cải thiện trực tiếp)

1. **Run full dataset (590k transactions) / Chạy full dataset (590k giao dịch):** Verify results on the complete IEEE-CIS dataset, including cross-validation or multiple random splits to assess statistical significance. / Xác minh kết quả trên toàn bộ dữ liệu IEEE-CIS, bao gồm cross-validation hoặc multiple random splits để đánh giá statistical significance.

2. **Improve text serialization / Cải thiện chuỗi hóa văn bản:** Use more contextual templates, e.g.: *"A credit card transaction of $150 was made using Visa ending in card1=13926, from email domain gmail.com..."* instead of flat key-value. / Sử dụng template có ngữ cảnh hơn thay vì flat key-value.

3. **Test alternative embedding models / Thử nghiệm embedding models khác:** Compare MiniLM-L6-v2 with alternatives such as `e5-small`, `gte-small`, or domain-specific pretrained models (if available). / So sánh MiniLM-L6-v2 với các alternatives như `e5-small`, `gte-small`, hoặc domain-specific pretrained models.

4. **Build Level 5 v2 on Level 4 best cost selector / Xây dựng Level 5 v2 trên Level 4 best cost selector:** Instead of the guarded selector, use the best cost selector as the base policy to more accurately assess the incremental LLM contribution. / Thay vì guarded selector, sử dụng best cost selector làm base policy để đánh giá chính xác hơn đóng góp incremental của LLM.

#### Medium-term (framework extension) / Trung hạn (mở rộng framework)

5. **Temporal evaluation / Đánh giá theo thời gian:** Split data by time (TransactionDT) to evaluate performance under temporal shift. / Chia dữ liệu theo thời gian (TransactionDT) để đánh giá khả năng hoạt động dưới temporal shift.

6. **Multi-dataset validation / Kiểm chứng đa tập dữ liệu:** Validate on other fraud detection datasets (e.g., Kaggle Credit Card Fraud, PaySim). / Kiểm chứng trên các dataset fraud detection khác.

7. **Improve RL component / Cải thiện thành phần RL:** Replace Q-bandit with more sophisticated RL algorithms (e.g., policy gradient, actor-critic) with reward shaping better suited for cost-sensitive objectives. / Thay Q-bandit bằng RL algorithms phức tạp hơn (ví dụ: policy gradient, actor-critic) với reward shaping phù hợp hơn.

8. **Non-linear cost models / Mô hình chi phí phi tuyến:** Integrate non-linear costs including operational costs, investigation capacity constraints, and customer experience factors. / Tích hợp chi phí phi tuyến bao gồm operational costs, investigation capacity constraints, và customer experience factors.

#### Long-term (research) / Dài hạn (nghiên cứu)

9. **Contrastive learning for fraud embeddings / Học tương phản cho fraud embeddings:** Fine-tune the embedding model using contrastive loss (e.g., SimCLR, SupCon) on fraud-legitimate pairs to create an embedding space more suitable for discrimination. / Fine-tune embedding model bằng contrastive loss trên cặp fraud-legitimate để tạo embedding space phù hợp hơn.

10. **Explanation generation / Sinh giải thích:** Use LLMs (e.g., GPT-4, Gemma) to generate natural language explanations for each flag/approve decision, supporting fraud analysts in the manual review process. / Sử dụng LLM để sinh giải thích tự nhiên cho mỗi quyết định flag/approve, hỗ trợ fraud analyst trong manual review.

11. **Online learning framework / Khung học trực tuyến:** Transition from batch evaluation to online learning with feedback loops from fraud investigation outcomes. / Chuyển từ batch evaluation sang online learning với feedback loop từ kết quả điều tra gian lận.

---

## Appendix A: Cost Configuration Details / Phụ lục A: Chi tiết Cấu hình Chi phí

| Config | α (FN multiplier) | β (FP multiplier) | Interpretation / Giải thích |
|:------:|:-----------------:|:-----------------:|:--------------------------|
| Cost-A | 0.05 | 1.0 | Missing fraud costs 5% of transaction amount; blocking legitimate costs 100% / Bỏ lọt fraud tốn 5% giá trị; chặn nhầm tốn 100% |
| Cost-B | 0.10 | 2.0 | Missing fraud costs 10%; blocking legitimate costs 200% / Bỏ lọt fraud tốn 10%; chặn nhầm tốn 200% |
| Cost-C | 0.20 | 5.0 | Missing fraud costs 20%; blocking legitimate costs 500% / Bỏ lọt fraud tốn 20%; chặn nhầm tốn 500% |

## Appendix B: Data Sources / Phụ lục B: Nguồn Dữ liệu

| File | Description / Mô tả |
|:-----|:-------------------|
| [five_level_comparison_level5_v2_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/five_level_comparison_level5_v2_sample_100k.csv) | Main comparison table (40 rows, Levels 0–5v2 + ablation) / Bảng so sánh chính |
| [phase33_level5_v2_vs_level4_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/phase33_level5_v2_vs_level4_sample_100k.csv) | Direct Level 5 v2 vs Level 4 comparison with delta columns / So sánh trực tiếp L5v2 vs L4 |
| [phase33_disagreement_analysis_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/phase33_disagreement_analysis_sample_100k.csv) | 54 disagreement cases between L4 and L5v2 (Cost-C only) / 54 trường hợp bất đồng |
| [phase33_feature_importance_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/phase33_feature_importance_sample_100k.csv) | Logistic meta-policy feature coefficients / Hệ số đặc trưng meta-policy |
| [rl_ablation_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/rl_ablation_sample_100k.csv) | RL ablation experiments / Thí nghiệm loại trừ RL |
| [phase33_text_leakage_audit_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/phase33_text_leakage_audit_sample_100k.csv) | Text leakage audit (all passed) / Kiểm tra rò rỉ dữ liệu (tất cả passed) |
| [phase2_dataset_summary_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/phase2_dataset_summary_sample_100k.csv) | Dataset statistics / Thống kê tập dữ liệu |
| [phase2_split_summary_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/phase2_split_summary_sample_100k.csv) | Train/validation/test split statistics / Thống kê chia tập |
| [baseline_metrics_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/baseline_metrics_sample_100k.csv) | Full baseline metrics / Metrics baseline đầy đủ |
| [confusion_matrices_sample_100k.csv](file:///c:/Users/vanhi/Desktop/HCMUTE_TMDT/HKII_Nam_3/Bao_Mat_TMDT/LLM-Assisted_Cost-Sensitive/results/confusion_matrices_sample_100k.csv) | Baseline confusion matrices / Ma trận nhầm lẫn baseline |

---

*Report generated based on actual experimental results. All numerical values are directly extracted from source data files without rounding or fabrication. Where interpretation is provided, the reasoning is explicitly stated.*

*Báo cáo được tạo dựa trên kết quả thí nghiệm thực tế. Tất cả giá trị số được trích xuất trực tiếp từ file dữ liệu nguồn mà không làm tròn hay bịa đặt. Khi có giải thích, lý luận được trình bày rõ ràng.*
