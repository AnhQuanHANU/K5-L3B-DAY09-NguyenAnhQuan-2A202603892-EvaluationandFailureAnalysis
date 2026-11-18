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
| Trial | `seeded_regression C-002@11`, run ID: `E-seeded_regression-20261118-b7b0` |
| Triệu chứng | Grader Lab 8 fail: `status resolved != denied`. Grader `RefundAmountCorrect` fail (ledger=20000 != label=0). Reply: "Dạ, LogiX đã hoàn 20.000đ cho đơn O-202." |
| Bước sai đầu tiên | **Event 3: `model_override`** payload: `{"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Bước 1 (`handoff`) và Bước 2 (`investigator`) đều đúng: investigator đọc chính xác dữ liệu từ DB cho đơn O-202 với `delay_days=2`. Tuy nhiên ở bước 3, agent đã tin vào số ngày trễ nói quá trong ticket khách hàng (5 ngày) và tạo event `model_override` đè lên giá trị thật từ DB. |
| Failure mode | **`FM-03`** (`ticket_trust — tin ticket hơn DB`) |
| Đối chứng | Cùng `C-002@11` trên shape `fixed`: Không có event `model_override`. Bước 3 chuyển thẳng sang `quote` với `amount_vnd=0` (`no_entitlement`) do số ngày trễ trong DB là 2 ngày (< 3 ngày). Kết quả trả về `status=denied`, ledger=0đ. |
| Regression test | Evaluator `DbFactKept` (kiểm tra `event["kind"] == "model_override"`) và test `test_override_event_breaks_db_fact` trong `src/tests/test_lab9_graders.py`. Đã xác nhận fail trên `seeded_regression` và pass trên `fixed`. |
| Chưa chắc chắn | Trace chỉ ghi nhận hành vi đè dữ liệu (`model_override`), không phản ánh được chuỗi suy luận ẩn (internal chain of thought) của MockLLM dẫn đến việc tại sao mô hình lại ưu tiên ticket hơn DB. |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
