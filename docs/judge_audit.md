# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | đạt | dùng cho tiêu chí tone |
| helpful | 100 | 0.375 | 1.000 | 0.300 | không đạt | loại khỏi gate |

## ≥5 case audit thủ công
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú |
|---|---|---|---|---|---|
| 1 | H-008 | PASS | FAIL | Người | Câu khẳng định đã hoàn tiền nhưng không có bằng chứng số tiền/ledger; rubric phải yêu cầu trạng thái và số tiền kiểm chứng được. |
| 2 | H-017 | PASS | FAIL | Người | Câu nói đã hoàn 25.000đ cho O-606 trái với expected outcome; thêm kiểm tra policy hiện hành. |
| 3 | H-029 | PASS | FAIL | Người | Câu “đang xử lý” không giải quyết yêu cầu; helpful cần bắt thiếu hành động, không chỉ giọng lịch sự. |
| 4 | H-042 | FAIL | PASS | Người | Agent trả lời 120.000đ cho O-808 nhưng số tiền/ledger không khớp; rubric cần ưu tiên tính đúng nghiệp vụ. |
| 5 | H-051 | FAIL | PASS | Người | Agent trả lời refund 90.000đ cho O-202, không khớp dữ kiện; bổ sung ví dụ số tiền sai vào rubric. |

## Kết luận
- `helpful` bị loại: recall 0.30 và κ 0.375, dù precision 1.0; judge bỏ sót 21/30 lỗi nên không an toàn làm gate.
- `tone_ok` được dùng có điều kiện: κ 0.757, precision 0.857, recall 0.800; Rogan–Gladen hiệu chỉnh failure rate quan sát 0.28 thành 0.30.
- Hành động sửa rubric: ràng buộc đánh giá vào outcome/ledger, bắt lỗi số tiền và yêu cầu hành động cụ thể; không dùng độ lịch sự thay cho correctness.
- Rủi ro còn lại: judge chỉ được kiểm trên dữ liệu replay đã công khai; cần nhãn người mới để có held-out thật.
