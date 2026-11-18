# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Mẫu điền (mỗi bạn 1 case, lấy từ trial đang fail trong report)

| Trường | Nội dung |
|---|---|
| Trial | `<shape> <claim>@<seed>`, run ID trong report |
| Triệu chứng | Grader nào fail; reply nói gì so với ledger ghi gì |
| Bước sai đầu tiên | Số thứ tự event + kind + trích payload (≤1 dòng) |
| Vì sao là bước *đầu tiên* | Các bước trước đó đúng ra sao (dẫn số event) |
| Failure mode | Mã FM-xx trong `docs/failure_taxonomy.md` |
| Đối chứng | Cùng claim@seed trên shape `fixed`: bước đó khác thế nào |
| Regression test | Tên test + grader/assert nào bắt được; đã thấy fail trên variant lỗi và pass trên `fixed` chưa |
| Chưa chắc chắn | Điều trace không chứng minh được |

## Case đã phân tích: `seeded_regression C-002@11`

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, report `E-seeded_regression-20261118-aa72` |
| Triệu chứng | Lab 8 grader FAIL: `status resolved != denied`; ledger ghi 20.000đ thay vì 0đ. |
| Bước sai đầu tiên | Event 3, `model_override`, payload `{"delay_days": 5}`. DB đã xác minh đơn O-202 trễ 2 ngày. |
| Vì sao là bước đầu tiên | Events 1–2 chỉ tạo handoff và đọc order/policy; sai lệch bắt đầu khi ticket ghi đè fact DB, trước quote và refund. |
| Failure mode | FM-03 `ticket_trust` |
| Đối chứng | `fixed C-002@11` giữ `delay_days=2`, proposal 0 và không ghi refund. |
| Regression test | `DbFactKept` phải PASS ở fixed và FAIL ở seeded; `pass_rate` cũng chặn variant qua paired CI. |
| Chưa chắc chắn | Trace chứng minh lỗi của claim/seed này, nhưng không chứng minh bug xảy ra trên mọi seed; vì vậy eval chạy 5 seed. |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
