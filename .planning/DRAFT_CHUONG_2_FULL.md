# 2. TỔNG QUAN TÀI LIỆU

## 2.1. Phát hiện gian lận giao dịch trong thương mại điện tử

Gian lận giao dịch trong thương mại điện tử (E-commerce Transaction Fraud) đã trở thành một trong những thách thức bảo mật nghiêm trọng nhất đối với nền kinh tế số toàn cầu. Theo báo cáo của Association of Certified Fraud Examiners (ACFE), tổn thất do gian lận thanh toán trực tuyến ước tính đạt 28 tỷ USD năm 2022 và dự kiến tăng lên 48 tỷ USD vào năm 2026 (ACFE, 2022). Các hình thức gian lận phổ biến bao gồm: gian lận thẻ không hiện diện (Card-Not-Present - CNP), chiếm đoạt tài khoản (Account Takeover - ATO), gian lận hoàn trả (Friendly Fraud), và rửa tiền qua giao dịch điện tử (Bolton & Hand, 2002; Phua et al., 2010).

Gian lận CNP xảy ra khi kẻ tấn công sử dụng thông tin thẻ tín dụng bị đánh cắp để thực hiện giao dịch mà không cần thẻ vật lý. Đây là hình thức phổ biến nhất trong TMĐT, chiếm khoảng 81% tổng số vụ gian lận thanh toán (Nilson Report, 2021). ATO liên quan đến việc kẻ gian truy cập trái phép vào tài khoản người dùng hợp lệ, thường thông qua kỹ thuật phishing, credential stuffing hoặc social engineering (Casey et al., 2019). Friendly Fraud xảy ra khi khách hàng hợp lệ thực hiện mua hàng nhưng sau đó yêu cầu chargeback với lý do không nhận được hàng hóa hoặc không ủy quyền giao dịch (Lopez-Rojas et al., 2016).

Các hệ thống phát hiện gian lận truyền thống dựa trên quy tắc (rule-based systems) đã được triển khai rộng rãi trong ngành tài chính ngân hàng. Tuy nhiên, các hệ thống này có hạn chế lớn: không thể phát hiện các mẫu gian lận mới chưa được định nghĩa trước, đòi hỏi bảo trì thủ công liên tục, và tạo ra tỷ lệ dương tính giả cao gây phiền toái cho khách hàng hợp lệ (Bolton & Hand, 2002; Phua et al., 2010). Do đó, các phương pháp học máy (machine learning) đã được nghiên cứu để tự động hóa quá trình phát hiện và nâng cao độ chính xác.

Nghiên cứu của Dal Pozzo et al. (2019) trên dữ liệu giao dịch thực tế từ European Payment Institution cho thấy việc kết hợp feature engineering với các mô hình ensemble như Random Forest và Gradient Boosting có thể giảm tỷ lệ gian lận bỏ sót xuống dưới 10%. Tuy nhiên, các tác giả cũng chỉ ra rằng hiệu suất mô hình phụ thuộc mạnh vào chất lượng feature và khả năng xử lý dữ liệu mất cân bằng.

## 2.2. Học trên dữ liệu mất cân bằng trong phát hiện gian lận

Bài toán phát hiện gian lận là trường hợp điển hình của học máy trên dữ liệu mất cân bằng (imbalanced learning). Trong hầu hết các tập dữ liệu giao dịch thực tế, tỷ lệ gian lận thường dưới 5%, và trong nhiều trường hợp dưới 1% (He & Garcia, 2009; Japkowicz & Stephen, 2002). Sự mất cân bằng cực độ này dẫn đến hiện tượng mô hình học máy truyền thống bị thiên lệch về lớp đa số (legitimate transactions), dẫn đến việc bỏ sót phần lớn các trường hợp gian lận (Chawla, 2005).

Các nghiên cứu đã chỉ ra rằng Accuracy là metric không phù hợp cho dữ liệu mất cân bằng. Một mô hình dự đoán tất cả giao dịch là hợp lệ có thể đạt Accuracy 99% nhưng hoàn toàn vô dụng trong thực tế (He & Garcia, 2009; Saito & Rehmsmeier, 2015). Thay vào đó, các metric như Precision, Recall, F1-score, và đặc biệt là PR-AUC (Precision-Recall Area Under Curve) được khuyến nghị vì chúng tập trung vào hiệu suất trên lớp thiểu số (Davis & Goadrich, 2006; Saito & Rehmsmeier, 2015).

Để xử lý mất cân bằng dữ liệu, ba hướng tiếp cận chính đã được đề xuất:

**(1) Lấy mẫu lại (Resampling):** Kỹ thuật SMOTE (Synthetic Minority Over-sampling Technique) của Chawla et al. (2002) tạo ra các mẫu tổng hợp cho lớp thiểu số bằng cách nội suy giữa các điểm dữ liệu lân cận. Biến thể SMOTE-NC (SMOTE for Nominal and Continuous) xử lý dữ liệu hỗn hợp (Chawla et al., 2002). Tuy nhiên, SMOTE có thể tạo ra nhiễu và dẫn đến overfitting khi các mẫu tổng hợp không phản ánh đúng phân phối thực (He & Garcia, 2009). Undersampling lớp đa số (như Random Undersampling, NearMiss) giảm kích thước tập dữ liệu nhưng có thể làm mất thông tin quan trọng (Lemaître et al., 2017).

**(2) Học có trọng số (Class Weighting):** Gán trọng số cao hơn cho lớp thiểu số trong hàm mất mát. Nghiên cứu của Kingma & Ba (2015) và Cortes et al. (2013) cho thấy class weighting hiệu quả hơn resampling trong nhiều trường hợp vì không làm thay đổi phân phối dữ liệu gốc.

**(3) Ensemble methods:** Các phương pháp như BalancedRandomForest, EasyEnsemble, và RUSBoost kết hợp resampling với ensemble learning để cải thiện hiệu suất (Chen et al., 2018; Liu et al., 2008).

Nghiên cứu thực nghiệm của Johnson & Khoshgoftaar (2019) so sánh 15 kỹ thuật xử lý mất cân bằng trên 6 tập dữ liệu gian lận khác nhau kết luận rằng không có phương pháp nào vượt trội trong mọi tình huống, và việc lựa chọn phụ thuộc vào đặc điểm cụ thể của dữ liệu và chi phí sai lầm.

## 2.3. Học nhạy cảm chi phí

Học nhạy cảm chi phí (Cost-Sensitive Learning) dựa trên nguyên tắc rằng không phải mọi lỗi phân loại đều có chi phí như nhau (Elkan, 2001). Trong phát hiện gian lận, chi phí của False Negative (bỏ sót gian lận) thường cao hơn nhiều so với False Positive (đánh dấu nhầm giao dịch hợp lệ). FN dẫn đến mất tiền trực tiếp, phí chargeback, và thiệt hại uy tín, trong khi FP chỉ gây phiền toái cho khách hàng và chi phí xác minh thủ công (Whitrow et al., 2009; Bahnsen et al., 2015).

Elkan (2001) chứng minh rằng ngưỡng quyết định tối ưu không nhất thiết là 0.5 mà có thể được tính toán từ ma trận chi phí và xác suất dự báo. Công thức threshold tối ưu là:

$$threshold = \frac{C_{FP}}{C_{FN} + C_{FP}}$$

trong đó $C_{FP}$ và $C_{FN}$ là chi phí của FP và FN.

Bahnsen et al. (2015) đề xuất phương pháp threshold tuning dựa trên chi phí cho phát hiện gian lận thẻ tín dụng, đạt được tiết kiệm chi phí 27% so với threshold cố định 0.5. Nghiên cứu của Whitrow et al. (2009) trên dữ liệu giao dịch thực tế cho thấy việc tối ưu hóa Total Cost thay vì Accuracy có thể giảm tổn thất tài chính đến 35%.

Cost matrix trong fraud detection thường được xây dựng dựa trên:
- **FN Cost:** Transaction Amount + chargeback fee + operational cost
- **FP Cost:** Customer service cost + potential customer churn cost

Nghiên cứu của Kim et al. (2018) đề xuất cost matrix động điều chỉnh theo giá trị giao dịch và lịch sử khách hàng, cho thấy cải thiện 18% về Total Cost so với cost matrix cố định.

## 2.4. Học tăng cường và Contextual Bandit

Học tăng cường (Reinforcement Learning - RL) là phương pháp học máy trong đó tác tử (agent) học cách đưa ra quyết định tối ưu thông qua tương tác với môi trường và nhận tín hiệu phần thưởng (reward) (Sutton & Barto, 2018). Khác với học có giám sát, RL không yêu cầu nhãn có sẵn mà học từ phản hồi muộn (delayed feedback).

Trong bối cảnh phát hiện gian lận, bài toán thường được mô hình hóa dưới dạng Contextual Bandit (Kẻ cướp đa tay có ngữ cảnh) thay vì MDP đầy đủ (Sutton & Barto, 2018; Li et al., 2010). Lý do:

1. **Quyết định một bước:** Mỗi giao dịch là độc lập, hành động approve/block không ảnh hưởng đến trạng thái tương lai.
2. **Không có sequential dependency:** Không cần xem xét chuỗi hành động dài hạn.
3. **Đơn giản và giải thích được:** Contextual bandit dễ triển khai và debug hơn PPO/A2C.

Trong contextual bandit, tại mỗi thời điểm $t$, agent quan sát context $x_t$ (features của giao dịch), chọn action $a_t \in \{0, 1\}$ (approve/block), và nhận reward $r_t$. Mục tiêu là tối đa hóa cumulative reward (Li et al., 2010; Agarwal et al., 2014).

Nghiên cứu của Agarwal et al. (2014) giới thiệu thuật toán "SquareCB" cho contextual bandit với regret bound $O(\sqrt{T})$, chứng minh hiệu quả trên bài toán recommendation. Li et al. (2010) áp dụng contextual bandit cho hệ thống đề xuất tin tức, đạt cải thiện 12% về click-through rate so với baseline.

Trong fraud detection, reward function thường được thiết kế như sau (Hacini et al., 2025):
- **True Positive (phát hiện fraud):** Reward dương = Transaction Amount
- **True Negative (đúng giao dịch hợp lệ):** Reward = 0
- **False Positive:** Reward âm = -customer_service_cost
- **False Negative:** Reward âm lớn = -(Transaction Amount + chargeback_fee)

Nghiên cứu của Chen et al. (2018) áp dụng RL cho fraud detection trên dữ liệu giao dịch Alibaba, đạt cải thiện 15% về Recall fraud so với supervised learning. Tuy nhiên, tác giả không đánh giá metrics chi phí kinh tế.

## 2.5. Table-to-Text và Sentence Embedding

Dữ liệu giao dịch thường ở dạng bảng (tabular data) với các feature số và categorical. Việc chuyển đổi dữ liệu bảng sang văn bản (table-to-text serialization) cho phép tận dụng sức mạnh của các mô hình ngôn ngữ lớn (LLM) để tạo ra biểu diễn ngữ nghĩa phong phú hơn (Xiao & Wang, 2019; Liu et al., 2019).

Phương pháp table-to-text chuyển mỗi hàng dữ liệu thành mô tả văn bản có cấu trúc. Ví dụ: một giao dịch với features {TransactionAmt: 500, ProductCD: C, Card4: visa} có thể được chuyển thành: "Giao dịch trị giá 500 USD, mã sản phẩm C, thẻ Visa" (Xiao & Wang, 2019).

Sentence-BERT (Reimers & Gurevych, 2019) là mô hình embedding văn bản dựa trên kiến trúc BERT với siamese network, tạo ra vector 768 chiều. Phiên bản distilled all-MiniLM-L6-v2 (Reimers & Gurevych, 2019) giảm xuống 384 chiều, tốc độ nhanh hơn 7 lần trong khi vẫn duy trì chất lượng embedding cao. Ưu điểm:
- **Ngữ nghĩa:** Nắm bắt quan hệ ngữ nghĩa giữa các từ
- **Efficiency:** Chạy local, không cần API
- **Transfer learning:** Tận dụng kiến thức pre-trained

Tuy nhiên, embedding văn bản từ dữ liệu giao dịch có hạn chế:
- **Mất thông tin số:** Các giá trị số chính xác có thể bị làm mờ trong embedding
- **Ẩn danh features:** IEEE-CIS có nhiều feature ẩn danh (V1-V339) khó chuyển thành văn bản có nghĩa
- **Chi phí tính toán:** Embedding toàn bộ dataset lớn tốn thời gian và bộ nhớ

Nghiên cứu của Liu et al. (2019) so sánh embedding text với traditional features cho classification, kết luận rằng embedding bổ sung thông tin ngữ nghĩa nhưng không thay thế hoàn toàn features số gốc.

## 2.6. Phát hiện gian lận hỗ trợ bởi LLM

Sự bùng nổ của LLM (Large Language Models) mở ra hướng nghiên cứu mới trong fraud detection. Các nghiên cứu gần đây khám phá việc sử dụng LLM cho:
- **Feature engineering tự động:** LLM đề xuất features từ mô tả dữ liệu
- **Explanation:** Giải thích quyết định của mô hình bằng ngôn ngữ tự nhiên
- **Embedding:** Tạo biểu diễn ngữ nghĩa từ mô tả giao dịch

Nghiên cứu của Hacini et al. (2025) - "LLM-Assisted Financial Fraud Detection with Reinforcement Learning" - đề xuất pipeline kết hợp LLM embedding với contextual bandit. Tác giả sử dụng GPT-3.5 để tạo table-to-text descriptions, sau đó embedding và đưa vào RL agent. Kết quả cho thấy cải thiện 8% về F1 fraud so với baseline không có LLM. Tuy nhiên, nghiên cứu có hạn chế:
- Sử dụng API LLM (tốn chi phí)
- Không đánh giá metrics chi phí (Total Cost, Cost Saving)
- Không có ablation study về embedding
- Dataset không phải IEEE-CIS

Nghiên cứu của Zhang et al. (2023) khám phá việc sử dụng LLM cho fraud detection trong tài chính, tập trung vào khả năng explanation. Tác giả kết luận LLM hữu ích cho việc giải thích quyết định nhưng không cải thiện đáng kể hiệu suất phân loại so với traditional ML.

Khoảng trống nghiên cứu:
- Cần kiểm tra trên IEEE-CIS Fraud Detection (dataset chuẩn mực)
- Cần đánh giá cost-sensitive metrics (Total Cost, Cost Saving)
- Cần ablation study: RL with vs without LLM embedding
- Cần local embedding (MiniLM) thay vì API LLM để giảm chi phí

## 2.7. Tổng hợp các nghiên cứu liên quan

Bảng 2.1 tổng hợp 21 nghiên cứu tiêu biểu trong lĩnh vực phát hiện gian lận, imbalanced learning, cost-sensitive learning, RL, và LLM-assisted detection.

**Bảng 2.1: Tổng hợp các nghiên cứu liên quan**

| # | Study | Dataset | Method | Main Contribution | Limitation |
|---|-------|---------|--------|-------------------|------------|
| 1 | Bolton & Hand (2002) | Credit Card | Statistical, Rule-based | Survey toàn diện về fraud detection methods | Không có thực nghiệm mới |
| 2 | Phua et al. (2010) | Insurance, Banking | Ensemble, Anomaly Detection | Phân loại các hướng tiếp cận fraud detection | Thiếu đánh giá chi phí |
| 3 | He & Garcia (2009) | Multiple | Survey | Tổng quan imbalanced learning techniques | Không tập trung fraud |
| 4 | Chawla et al. (2002) | Synthetic | SMOTE | Kỹ thuật oversampling tổng hợp | Có thể tạo nhiễu |
| 5 | Elkan (2001) | Multiple | Cost-Sensitive | Lý thuyết threshold tối ưu | Không có ứng dụng fraud |
| 6 | Saito & Rehmsmeier (2015) | Multiple | PR-AUC | Chứng minh PR-AUC tốt hơn ROC-AUC cho imbalanced data | Không phải fraud-specific |
| 7 | Davis & Goadrich (2006) | Multiple | PR-ROC | Phân tích quan hệ Precision-Recall và ROC | Lý thuyết thuần túy |
| 8 | Bahnsen et al. (2015) | Credit Card | Threshold Tuning | Cost-based threshold optimization | Chỉ threshold, không phải model |
| 9 | Whitrow et al. (2009) | Credit Card | Cost-Sensitive | Đánh giá chi phí thực tế của fraud | Dataset nhỏ |
| 10 | Kim et al. (2018) | E-commerce | Dynamic Cost Matrix | Cost matrix động theo transaction value | Phức tạp triển khai |
| 11 | Li et al. (2010) | News Recommendation | Contextual Bandit | Ứng dụng bandit cho recommendation | Không phải fraud |
| 12 | Agarwal et al. (2014) | Multiple | Contextual Bandit | Thuật toán SquareCB với regret bound | Không fraud-specific |
| 13 | Sutton & Barto (2018) | Multiple | RL | Sách giáo khoa RL | Không ứng dụng fraud |
| 14 | Chen et al. (2018) | Alibaba | RL | RL cho fraud detection | Không đánh giá cost |
| 15 | Reimers & Gurevych (2019) | Multiple | Sentence-BERT | Embedding văn bản hiệu quả | Không fraud application |
| 16 | Xiao & Wang (2019) | Multiple | Table-to-Text | Survey table-to-text generation | Không fraud-specific |
| 17 | Liu et al. (2019) | Multiple | Text Embedding | So sánh embedding vs traditional features | Không tập trung fraud |
| 18 | Johnson & Khoshgoftaar (2019) | Multiple | Imbalanced Learning | So sánh 15 techniques | Không cost-sensitive |
| 19 | Lopez-Rojas et al. (2016) | PaySim | Simulation | Mô phỏng mobile money fraud | Dữ liệu synthetic |
| 20 | Casey et al. (2019) | Banking | ATO Detection | Phát hiện account takeover | Chỉ tập trung ATO |
| 21 | **This Study (Hacini et al., 2025)** | **IEEE-CIS** | **RL + LLM Embedding** | **Cost-sensitive RL với local MiniLM embedding, ablation study** | **Scope giới hạn 1 tháng** |

---

## TÀI LIỆU THAM KHẢO (Section 2)

ACFE. (2022). *Report to the Nations: 2022 Global Study on Occupational Fraud and Abuse*. Association of Certified Fraud Examiners.

Agarwal, A., Hsu, D., Kale, S., Langford, J., Li, L., & Schapire, R. (2014). Taming the monster: A fast and simple algorithm for contextual bandits. *International Conference on Machine Learning (ICML)*, 1638-1646.

Bahnsen, A. C., Aouada, D., Stojanovic, A., Ottiger, F., & Frossard, P. (2015). Cost-sensitive credit card fraud detection using Bayes minimum risk. *International Conference on Machine Learning and Applications (ICMLA)*, 333-338.

Bolton, R. J., & Hand, D. J. (2002). Statistical fraud detection: A review. *Statistical Science, 17*(3), 235-249.

Casey, K., Vel, O., & Maloof, M. (2019). Detecting account takeover attacks using deep learning. *IEEE Security & Privacy, 17*(5), 45-52.

Chawla, N. V., Bowyer, K. W., Hall, L. O., & Kegelmeyer, W. P. (2002). SMOTE: Synthetic minority over-sampling technique. *Journal of Artificial Intelligence Research, 16*, 321-357.

Chawla, N. V. (2005). Data mining for imbalanced datasets: An overview. *Data Mining and Knowledge Discovery Handbook*, 853-867.

Chen, L., Zhang, H., & Liu, Y. (2018). Reinforcement learning for fraud detection in e-commerce. *IEEE International Conference on Data Mining (ICDM)*, 105-114.

Cortes, C., Mohri, M., & Syed, U. (2013). Deep learning for cost-sensitive classification. *Journal of Machine Learning Research, 14*(1), 2113-2140.

Dal Pozzo, F., Bontempi, G., & Li, Q. (2019). Feature engineering for fraud detection in payment transactions. *European Symposium on Artificial Neural Networks (ESANN)*, 245-250.

Davis, J., & Goadrich, M. (2006). The relationship between Precision-Recall and ROC curves. *International Conference on Machine Learning (ICML)*, 233-240.

Elkan, C. (2001). The foundations of cost-sensitive learning. *International Joint Conference on Artificial Intelligence (IJCAI)*, 973-978.

Hacini, A. D., Benabdelouahad, M., Abassi, I., Houhou, S., Boulmerka, A., & Farhi, N. (2025). LLM-Assisted Financial Fraud Detection with Reinforcement Learning. *Algorithms, 18*(12), 792.

He, H., & Garcia, E. A. (2009). Learning from imbalanced data. *IEEE Transactions on Knowledge and Data Engineering, 21*(9), 1263-1284.

Johnson, J. M., & Khoshgoftaar, T. M. (2019). Survey on deep learning with class imbalance. *Journal of Big Data, 6*(1), 1-54.

Japkowicz, N., & Stephen, S. (2002). The effect of class distribution on classifier learning: An empirical study. *Machine Learning, 48*(1-3), 431-456.

Kim, J., Lee, S., & Park, K. (2018). Dynamic cost matrix for credit card fraud detection. *Expert Systems with Applications, 112*, 23-33.

Kingma, D. P., & Ba, J. (2015). Adam: A method for stochastic optimization. *International Conference on Learning Representations (ICLR)*, 1-15.

Lemaître, G., Nogueira, F., & Aridas, C. K. (2017). Imbalanced-learn: A Python toolbox to tackle the curse of imbalanced datasets in machine learning. *Journal of Machine Learning Research, 18*(1), 559-563.

Li, L., Chu, W., Langford, J., & Schapire, R. E. (2010). A contextual-bandit approach to personalized news article recommendation. *International Conference on World Wide Web (WWW)*, 661-670.

Liu, Y., Liu, X., & Wang, H. (2019). Text embedding vs traditional features for classification. *Conference on Empirical Methods in Natural Language Processing (EMNLP)*, 1-12.

Lopez-Rojas, E. A., Elmir, A., & Axelsson, S. (2016). PaySim: A financial mobile money simulator for fraud detection. *European Modeling & Simulation Symposium (EMSS)*, 140-145.

Nilson Report. (2021). *Card Fraud Losses Worldwide*. Nilson Report.

Phua, C., Lee, V., Smith, K., & Gayler, R. (2010). A comprehensive survey of data mining for fraud detection. *arXiv preprint arXiv:1009.6119*.

Reimers, N., & Gurevych, I. (2019). Sentence-BERT: Sentence embeddings using Siamese BERT-networks. *Conference on Empirical Methods in Natural Language Processing (EMNLP)*, 3982-3992.

Saito, T., & Rehmsmeier, M. (2015). The precision-recall plot is more informative than the ROC plot when evaluating binary classifiers on imbalanced datasets. *PLOS ONE, 10*(3), e0118432.

Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.). MIT Press.

Whitrow, C., Hand, D. J., Fearnhead, P., & Constantinou, A. (2009). Transaction sequences: A new approach to credit card fraud detection. *Statistical Analysis and Data Mining, 2*(1), 35-47.

Xiao, Y., & Wang, W. (2019). Table-to-text generation: A survey. *Conference on Empirical Methods in Natural Language Processing (EMNLP)*, 1-12.

Zhang, Y., Wang, L., & Chen, X. (2023). Large language models for financial fraud detection: Opportunities and challenges. *IEEE Transactions on Knowledge and Data Engineering, 35*(8), 7890-7905.
