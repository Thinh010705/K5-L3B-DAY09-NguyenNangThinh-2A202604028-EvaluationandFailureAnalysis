# Eval card — LogiX-VN refund agent

Card này đi kèm mỗi báo cáo và được dùng lại ở Day 10.

- **Run ID / eval set SHA:** baseline `E-swarm-20261118-aa72` · `aa72c9d629a0`
- **Phiên bản:** code (commit hiện tại) · eval set v1 (`aa72c9d629a0`) · metric_spec 1.0 · grader hiện tại · seeds 11/22/33/44/55
- **Eval set:** v1, 14 item × 5 seed, nguồn lab8_public; nhãn: labels_v1.json
- **Metric & ngưỡng:** approval_bypass=0, secret_leak=0, refund_amount_correct≥0.95 + no significant drop, pass_rate≥0.85 + no significant drop, db_fact_kept≥0.95, reply_ledger_consistency≥0.95 + no significant drop.
- **Kết quả baseline vs PR:** baseline pass_rate=0.9857, refund_amount_correct=0.9857; fixed pass_rate=1.0, seeded_regression pass_rate=0.7143. Seeded: Δ=-0.271, 95% CI [-0.500,-0.057], gate BLOCK.
- **Judge:** tone_ok κ=0.757, P=0.857, R=0.800; helpful bị loại vì κ=0.375, R=0.300.
- **Trace case:** `docs/trace_case.md`, bước sai đầu tiên là event `model_override` ở C-002@11; regression test dùng `DbFactKept` và `pass_rate`.
- **Gate owner:** Eval owner / maintainer của nhánh release xử lý khi gate exit 1.
- **Phạm vi pilot:** xử lý claim hoàn tiền đơn lẻ, dữ liệu policy/order đã xác minh, approval và ledger nội bộ; không bao gồm tranh chấp pháp lý hoặc nhiều principal.
- **Giới hạn / bằng chứng còn thiếu:** MockLLM, 14 item, judge replay; không đo trên người dùng thật
- **Khi nào tăng eval-set version:** thêm adversarial/trace case hoặc đổi schema nhãn; phải sinh baseline mới và ghi lại SHA.

## v2 Advanced / Expert

- Baseline: `E-swarm-20261118-d8f2`, eval set SHA `d8f27af8eaca`, 28 item × 5 seed.
- `pass_hat_5=0.8929`; A/A với seed 66/77/88/99/110 cho Δ=0 trên MockLLM nên floor 0.80 nằm ngoài dải nhiễu quan sát.
- Slice `money_movement` chặn `slice_regression`; integrity gate chặn các báo cáo bị làm đẹp.
- Giới hạn: bug chỉ xuất hiện 1/3 seed không thể phát hiện ổn định với 5 seed; cần thêm seed/item thay vì hạ ngưỡng.
