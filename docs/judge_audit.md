# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | Đạt (κ=0.757 ≥ 0.6, P=0.857 ≥ 0.8, R=0.800 ≥ 0.7) | DÙNG |
| helpful | 100 | 0.375 | 1.000 | 0.300 | Không đạt (κ=0.375 < 0.6, R=0.300 < 0.7) | LOẠI |

## ≥5 case audit thủ công

| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú & Hành động sửa rubric |
|---|---|---|---|---|---|
| 1 | H-008 | PASS | FAIL | Người đúng | Câu văn dùng xưng hô thiếu trang trọng ("Em đã hoàn tiền cho anh rồi nha, anh check giúp em"). Judge bỏ qua xưng hô suồng sã. Cần sửa rubric: bổ sung quy định cấm dùng đại từ thân mật ("anh/em/nha") thay vì chuẩn mực thương hiệu ("LogiX - Quý khách/Anh/Chị"). |
| 2 | H-017 | PASS | FAIL | Người đúng | Câu trả lời suồng sã ("Dạ, bạn ơi LogiX đã hoàn 25.000đ cho đơn O-606 rồi nha"). Judge coi kính ngữ đầu câu là đủ đạt. Cần sửa rubric: yêu cầu cấm kết thúc câu bằng các thán từ suồng sã không chuyên nghiệp. |
| 3 | H-029 | PASS | FAIL | Người đúng | Giọng điệu cộc lốc mang tính gạt khách ("Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm"). Judge không nhận diện được sự khó chịu trong câu. Cần sửa rubric: cấm các biểu đạt ngăn cản khách hàng phản hồi hoặc ra lệnh cho khách. |
| 4 | H-042 | FAIL | PASS | Người đúng | Phản hồi rõ ràng, đúng thông tin ("Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ"). Judge phạt oan vì cho rằng câu thiếu lời xin lỗi. Cần sửa rubric: không bắt buộc phải có câu xin lỗi khi khiếu nại đã được giải quyết nhanh gọn và lịch sự. |
| 5 | HL-016 | PASS | FAIL | Người đúng | Câu xin lỗi dập khuôn và bị cụt ("Dạ, LogiX thành thật xin lỗi anh/chị vì sự bất tiện này và rất trân trọng sự kiên nhẫn của"). Judge helpful bị đánh lừa bởi lời xin lỗi hoa mỹ mà bỏ qua việc câu chưa hoàn chỉnh và hoàn toàn không cung cấp giải pháp. |

## Kết luận
- **Tiêu chí bị loại và lý do:**
  - Tiêu chí `helpful` bị **loại bỏ hoàn toàn khỏi cổng release gate**.
  - *Dẫn chứng ma trận nhầm lẫn:* `{'tp': 9, 'fp': 0, 'fn': 21, 'tn': 70}`. Độ nhạy (Recall) chỉ đạt **0.300** (thấp hơn nhiều so với sàn 0.700), bỏ sót tới 21/30 trường hợp câu trả lời kém hữu ích (False Negative rate = 70%). Cohen's Kappa chỉ đạt **0.375** (dưới sàn 0.600). Mô hình judge bị thiên vị đối với những câu có từ ngữ lịch sự, xin lỗi khuôn mẫu nhưng vô giá trị về mặt thông tin nghiệp vụ.
- **Rủi ro còn lại:**
  - Judge `tone_ok` dù đạt ngưỡng sàn trên tập dữ liệu replay công khai (n=100) nhưng vẫn có rủi ro bị lệch phân phối (distribution drift) khi gặp các biến thể câu hỏi và khiếu nại mới từ khách hàng thực tế. Cần định kỳ gắn nhãn thủ công trên tập held-out mới để liên tục hiệu chỉnh (re-calibrate) judge.
