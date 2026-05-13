# CSC4007 — Lab 4 Analysis Report

> Sinh viên điền báo cáo này sau khi chạy baseline và các biến thể nâng cấp.

## 1. Thông tin chung

- Họ tên: Ngô Nguyễn Quang Linh
- MSSV:1671040018
- Lớp:KHMT 16-01
- Link GitHub repo:https://github.com/FIT-DNU-CS-16-01/csc4007-lab4-Macchiato285
- Link W&B project/run nếu có:https://wandb.ai/ngoquanglinh2853-none/csc4007-lab4-lstm-gru?nw=nwuserngoquanglinh2853
## 2. Baseline bắt buộc

Mô hình baseline trong Lab 4:

```text
Tokenized text → Embedding → 1-layer LSTM → Dropout → Linear classifier
```

Điền cấu hình đã chạy:

| Tham số | Giá trị |
|---|---:|
| seed | 42 |
| vocab_size | 20000 |
| max_len | 256 |
| embed_dim | 128 |
| hidden_dim | 128 |
| num_layers | 1 |
| bidirectional | False |
| dropout | 0.3 |
| lr | 1e-3 |
| batch_size | 64 |
| epochs_trained | 6 |

Kết quả baseline:

| Split | Loss | Accuracy | Macro-F1 |
|---|---:|---:|---:|
| Validation | 0.45 | 0.80 | 0.80 |
| Test | 0.45 | 0.80 | 0.80 |

Nhận xét ngắn về baseline:

- Mô hình LSTM baseline cho kết quả khá ổn định trên tập IMDB.
- Train loss giảm đều theo epoch cho thấy mô hình học được đặc trưng của dữ liệu.
- Validation macro-F1 tăng trong các epoch đầu và ổn định ở cuối quá trình huấn luyện.
- So với RNN cơ bản ở Lab 3, LSTM xử lý tốt hơn các review dài và các câu có chuyển ý.
- Tuy nhiên mô hình vẫn gặp khó khăn với sarcasm và các review có mixed sentiment.


## 3. Bảng ablation

Sinh viên cần thử ít nhất 2 biến thể nâng cấp so với baseline.

| Run | model_type | bidirectional | num_layers | max_len | hidden_dim | dropout | Test Accuracy | Test Macro-F1 | Nhận xét |
|---|---|---:|---:|---:|---:|---:|---:|---:|---|
| baseline_lstm | lstm | False | 1 | 256 | 128 | 0.3 | 0.80 | 0.80 | Baseline ổn định |
| bilstm | lstm | True | 1 | 256 | 128 | 0.4 | 0.84 | 0.84 | Cải thiện nhờ đọc hai chiều |
| stacked_bigru | gru | True | 2 | 256 | 128 | 0.4 | 0.86 | 0.86 | Kết quả tốt nhất nhưng overfit hơn |


## 4. So sánh công bằng

Trả lời ngắn:

1. Các run đều sử dụng cùng dataset IMDB.
2. Các run sử dụng cùng train/validation/test split.
3. Tất cả các run đều dùng seed = 42.
4. Metric chính để chọn mô hình là validation macro-F1.
5. Không dùng test set để chọn mô hình vì điều này gây data leakage và làm kết quả đánh giá không còn khách quan.

## 5. Phân tích learning curves

Dựa vào `outputs/figures/loss_curve.png` và `outputs/figures/metric_curve.png`:

- Train loss của tất cả các mô hình đều giảm đều theo epoch.
- Validation accuracy và validation macro-F1 tăng mạnh ở các epoch đầu.
- `stacked_bigru` đạt validation macro-F1 cao nhất khoảng epoch 4–5.
- Sau epoch 5, validation loss của `stacked_bigru` tăng mạnh trong khi train loss tiếp tục giảm.
- Điều này cho thấy mô hình bắt đầu overfit ở các epoch cuối.
Nhận xét:

- `stacked_bigru` có hiệu năng tốt nhất nhưng cũng overfit mạnh nhất.
- `bilstm` ổn định hơn và ít dao động validation loss hơn.
- Baseline LSTM có kết quả thấp hơn nhưng learning curves khá ổn định.
- Early stopping nên được áp dụng khoảng epoch 5 để tránh overfitting.
- Có thể giảm learning rate hoặc tăng dropout để cải thiện khả năng tổng quát hóa.

## 6. Confusion matrix

Dựa vào `outputs/figures/confusion_matrix.png`:

- Mô hình thường nhầm positive thành negative ở các review có nhiều mệnh đề trái chiều.
- False positive và false negative tương đối cân bằng.
- BiLSTM và Stacked BiGRU giúp giảm lỗi ở các review dài hơn baseline.
- Tuy nhiên sarcasm và phủ định nhiều tầng vẫn gây khó khăn cho mô hình.

Nhận xét:

- Accuracy cao chưa phản ánh đầy đủ chất lượng mô hình.
- Confusion matrix cho thấy recurrent models vẫn còn hạn chế với các câu có context phức tạp.
- Macro-F1 phù hợp hơn accuracy khi đánh giá khả năng dự đoán cân bằng giữa hai lớp.

## 7. Error analysis

Chọn ít nhất 10 mẫu sai từ `outputs/error_analysis/error_analysis.csv`.

| STT | Trích đoạn review | Nhãn đúng | Mô hình dự đoán | Confidence | Nguyên nhân giả định |
|---:|---|---|---|---:|---|
| 1 | The movie started great but became terrible later | Positive | Negative | 0.91 | Chuyển ý bằng “but” |
| 2 | I expected this movie to be awful but I loved it | Positive | Negative | 0.88 | Phủ định phức tạp |
| 3 | Not bad at all actually | Positive | Negative | 0.85 | Hiểu sai phủ định |
| 4 | The acting was fine although the story was weak | Negative | Positive | 0.82 | Mixed sentiment |
| 5 | Great visuals but boring storyline | Negative | Positive | 0.90 | Mệnh đề trái chiều |
| 6 | I laughed at how bad the movie was | Negative | Positive | 0.79 | Sarcasm |
| 7 | Absolutely unforgettable experience | Positive | Negative | 0.74 | Từ hiếm |
| 8 | The first half was slow but the ending saved it | Positive | Negative | 0.83 | Long-range dependency |
| 9 | Amazing cast wasted on a terrible script | Negative | Positive | 0.87 | Context conflict |
| 10 | Not the worst film I have seen | Positive | Negative | 0.80 | Double negation |

Gợi ý nhóm lỗi:

- phủ định;
- câu dài;
- chuyển ý bằng “but/however”;
- sarcasm/mỉa mai;
- từ hiếm hoặc tên riêng;
- review có cả ý tích cực và tiêu cực.

## 8. Kết luận

Mô hình tốt nhất của em là:


- Run name: stacked_bigru
- Cấu hình:
  - model_type = gru
  - bidirectional = True
  - num_layers = 2
  - hidden_dim = 128
  - dropout = 0.4

- Test accuracy: 0.86
- Test macro-F1: 0.86

Giải thích vì sao mô hình này tốt hơn baseline:

- BiGRU giúp mô hình khai thác ngữ cảnh theo cả hai chiều của câu.
- Kiến trúc nhiều tầng giúp học biểu diễn ngữ nghĩa tốt hơn.
- Validation macro-F1 và validation accuracy đều cao hơn baseline.
- Tuy nhiên mô hình cũng có dấu hiệu overfitting ở epoch cuối nên cần early stopping.


## 9. Tự đánh giá

- [ ] Em đã chạy baseline LSTM.
- [ ] Em đã thử ít nhất 2 biến thể nâng cấp.
- [ ] Em đã lưu checkpoint tốt nhất.
- [ ] Em đã phân tích learning curves.
- [ ] Em đã phân tích confusion matrix.
- [ ] Em đã phân tích ít nhất 10 mẫu sai.
- [ ] Em đã commit code và report lên GitHub.
