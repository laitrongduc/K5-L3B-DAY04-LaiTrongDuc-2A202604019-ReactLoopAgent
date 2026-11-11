# Chuẩn bị demo bài chính

Kịch bản để học viên tự trình bày; chưa phải bằng chứng đã hoàn thành buổi demo. Dùng trace thật trong `eval_after.json`, không sửa kết quả và không cần gọi API thêm.

## 1. Giải thích agent loop

Mở `agent.py` và giải thích theo luồng:

1. `messages` bắt đầu bằng system prompt và câu hỏi; `trace` ghi sự kiện, `usage` lưu token/chi phí do provider trả.
2. Mỗi step gọi `provider.complete(messages, SCHEMAS)` một lần. Một response có thể có nhiều tool calls.
3. Append nguyên vẹn assistant message trước mọi tool result để giữ danh sách calls và các trường provider.
4. Parse JSON arguments, thực thi toàn bộ calls, rồi append từng observation có `role=tool`, đúng `tool_call_id` và content là JSON string. Model nhận các observation ở lần complete tiếp theo.
5. Tool lỗi được đưa lại model; provider lỗi kết thúc run. Output bị cắt hoặc finish bất thường trả error. Hết số lần gọi model trả max_steps.
6. Chỉ completed khi finish=stop, không còn calls và validation nguồn đạt. Không có incident thành công thì không bắt URL. Thiếu mọi URL incident đã quan sát thì yêu cầu sửa trong budget.

## 2. Demo trace nhiều bước: A6

Đọc trace đã lưu, không chạy lại model:

```bash
python3 - <<'PY'
import json
from pathlib import Path
report = json.loads(Path('eval_after.json').read_text())
case = next(c for c in report['cases'] if c['id'] == 'A6')
print('Câu hỏi:', case['question'])
for event in case['result']['trace']:
    if event['event'] == 'model':
        print('Model step', event['step'], 'finish:', event['finish'])
    elif event['event'] == 'tool':
        print('Tool step', event['step'], event['name'], event['arguments'])
        print('URL nguồn:', event['result'].get('url'))
print('Câu trả lời:', case['result']['answer'])
print('Điểm:', case['grade']['score'])
PY
```

Diễn biến thực tế: step 1 đọc incident GitHub `h02n9572mx9y`; observation trỏ đến incident gộp; step 2 đọc `xr3myzj7ms51`; step 3 trả final. Điểm A6 là 70, không dùng điểm đó để khẳng định mọi nhận định đều đúng. Trace này minh họa model dùng kết quả tool để quyết định bước tiếp theo.

## 3. Trình bày một lỗi thật: E1

```bash
python3 - <<'PY'
import json
from pathlib import Path
report = json.loads(Path('eval_after.json').read_text())
case = next(c for c in report['cases'] if c['id'] == 'E1')
for event in case['result']['trace']:
    if event['event'] == 'tool':
        print('captured_at:', event['result'].get('captured_at'))
print(case['result']['answer'])
print('Nhận xét judge:', case['grade']['reason'])
PY
```

Tool dùng snapshot ngày 09/11/2026, nhưng final dùng từ “hiện tại”. Cách kết luận phù hợp phải giới hạn ở thời điểm snapshot; trace không xác minh trạng thái ngoài đời lúc hỏi. Đây là lỗi nội dung dù run completed.

Nêu thêm H1/H6 vẫn bị cap 60 vì chỉ gọi tools ở một model step, chưa theo liên kết đến incident gộp. Hai calls trong cùng một step không đáp ứng việc điều tra tiếp dựa trên observation mới.

## 4. Giải thích cải tiến và kết quả

- Chọn lỗi A7: baseline đọc được URL incident nhưng final không dẫn nguồn; evidence nằm trong NOTES mục 2.
- Một cải tiến là gate URL trong loop. Nó kiểm tra có ít nhất một URL incident đã quan sát, không chứng minh mọi claim đúng hoặc mọi provider đều có nguồn.
- Tests có sẵn kiểm tra thiếu nguồn thì model được yêu cầu sửa, hết budget không báo completed và tool thất bại không bắt URL. Có thể trình bày bằng `python3 -m unittest discover -s tests -p test_validation.py -v`.
- Hai bộ eval chính đều đủ 22 case và cùng config; điểm 76,36 → 81,36. A7 vẫn 75 → 75, nhưng final sau có URL. Eval sau không có event yêu cầu sửa nguồn; không đủ bằng chứng quy mức tăng tổng cho gate. Judge/model có dao động.
- Số calls agent 50 → 51; token 41.358 → 41.124. Chi phí agent 0,01035870 → 0,01020750 USD. Số liệu không gồm judge, E2 riêng hoặc các smoke tests; chi phí judge có bảng riêng trong NOTES.
- Một lần chạy mỗi phiên bản không chứng minh độ ổn định. Điểm eval 81,36/100 không phải điểm bài lab 81,36/85.

## 5. Tự kiểm tra trước khi trình bày

Học viên cần tự giải thích được vì sao giữ assistant message, vì sao gắn đúng call ID, khác biệt giữa model step và tool call, lỗi nào cho model quan sát, điều kiện completed/max_steps/error và giới hạn của gate nguồn. Không khẳng định đã hiểu chỉ vì tests pass.

Sau khi demo thật, mới cập nhật trạng thái CP7. Commit/push và nộp link LMS là các việc riêng chưa được thực hiện bởi kịch bản này.
