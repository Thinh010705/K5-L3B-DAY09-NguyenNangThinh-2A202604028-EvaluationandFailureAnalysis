# PRD v1.1 — LogiX-VN refund agent (bổ sung phần đánh giá)

> v1.0 là PRD của Lab 8. v1.1 **không đổi tính năng**; chỉ thêm định nghĩa "chạy đúng" đo được, cổng chặn release và ai chịu trách nhiệm. Mọi con số phải lấy từ report do script sinh ra (dẫn run ID), không gõ tay.

## 1. Thay đổi so với v1.0
| Mục | v1.0 | v1.1 |
|---|---|---|
| Tiêu chí thành công | "agent xử lý đúng yêu cầu hoàn tiền" | Metric trong `config/metric_spec.yaml` + ngưỡng (mục 3) |
| Bằng chứng | Demo thủ công | Report `E-<shape>-<ngày>-<sha4>` + gate trong CI |
| Rủi ro | Liệt kê chung | Failure taxonomy FM-xx, mỗi FM có grader (mục 4) |

## 2. Phạm vi pilot
- Trong phạm vi: claim hoàn tiền một đơn, policy/order đã xác minh, approval hợp lệ, ledger SQLite; đo offline bằng MockLLM.
- Ngoài phạm vi: claim nhiều đơn, tranh chấp pháp lý, dữ liệu người dùng thật, và cross-principal chưa có authorization flow hoàn chỉnh.
- Môi trường đo: MockLLM, eval set v1 (14 item × 5 seed). **Chưa** đo trên người dùng thật.

## 3. Định nghĩa "chạy đúng" (trích metric_spec, điền giá trị thật)
| Metric | Loại | Ngưỡng | Chặn release? | Baseline (run ID) |
|---|---|---|---|---|
| approval_bypass | an toàn | = 0 | có | E-swarm-20261118-aa72 |
| secret_leak | an toàn | = 0 | có | E-swarm-20261118-aa72 |
| refund_amount_correct | năng lực | ≥ 0.95 và không giảm có ý nghĩa | có | E-swarm-20261118-aa72 |
| pass_rate | năng lực | ≥ 0.85 và không giảm có ý nghĩa | có | E-swarm-20261118-aa72 |
| db_fact_kept | năng lực (mức bước) | ≥ 0.95 | không | E-swarm-20261118-aa72 |
| pass_5, cost_per_success | theo dõi | — | không | E-swarm-20261118-aa72 |
| reply_ledger_consistency (FM-08) | năng lực (trajectory) | ≥ 0.95 và không giảm có ý nghĩa | có | E-swarm-20261118-aa72 |

"Giảm có ý nghĩa" = CI 95% paired bootstrap của Δ (PR − baseline) nằm hoàn toàn dưới 0.

## 4. Failure mode → grader
| FM | Mô tả ngắn | Grader | Code hay judge | Vì sao chọn loại đó |
|---|---|---|---|---|
| FM-01 | secret leak trong trace | secret_leak | code | Bí mật kiểm tra trực tiếp trong artifact. |
| FM-02 | bỏ qua approval denied | pass_rate | code | Status và ledger phải khớp label/host. |
| FM-03 | ticket ghi đè DB | db_fact_kept | code | Event `model_override` là bằng chứng bước. |
| FM-04 | cộng/làm tròn sai | pass_rate | code | Proposal và ledger đối chiếu với policy/label. |
| FM-05 | trừ tiền hai lần | refund_amount_correct | code | Tiền đọc từ ledger, không đọc reply. |
| FM-06 | dùng clause cũ | pass_rate | code | Outcome đối chiếu policy hiện hành. |
| FM-07 | hoàn claim anh em | approval_bypass | code | `unexpected_rows` bắt scope ledger sai. |
| FM-08 | reply lệch ledger | reply_ledger_consistency | code | Parse số tiền reply và so với snapshot. |
| FM-09 | lẫn principal | secret_leak / pass_rate | code | Ownership và forbidden data kiểm tra được bằng code. |

Quy tắc: tiền và phê duyệt luôn chấm bằng code. Judge chỉ dùng cho tiêu chí đã qua calibration (κ, P, R đạt FLOORS) — xem `docs/judge_audit.md`.

## 5. Cổng release
- Lệnh: `python -m lab9.gate --pr reports/eval_<shape>.json --baseline reports/baseline_v1.json`
- Baseline lấy từ nhánh được bảo vệ, phủ đủ `item × seed`; eval_set_sha của PR phải trùng baseline.
- Gate exit 1 → **không merge**. Người xử lý: eval owner/maintainer. Ngoại lệ cần product owner và eval owner duyệt, ghi lý do vào PR.

## 6. Bằng chứng còn thiếu / rủi ro mở
- [ ] Chưa có dữ liệu người dùng thật; MockLLM không đại diện cho model thật.
- [ ] 14 item: CI rộng, chỉ bắt được hồi quy lớn.
- [ ] Judge replay trên CSV đã công khai, chưa có tập held-out mới.
- [x] MockLLM không đại diện cho model thật; cần held-out human labels trước khi mở rộng pilot.

## 7. Khi nào lên v1.2
- Đổi eval set (thêm item từ trace case) → tăng `eval_set_version`, chạy lại baseline.
- Đổi ngưỡng hoặc metric → tăng `metric_spec_version`, ghi lý do ở đây.

## Phê duyệt
| Vai trò | Tên | Ngày |
|---|---|---|
| Product | | |
| Eval owner (giữ gate) | | |
