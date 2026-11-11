# Ghi chú phân tích · Day 04

Họ và tên: LẠI TRỌNG ĐỨC
MSSV: 2A202604019

## 1. Case được chọn

- ID case: A7 — xác minh liệu incident Cloudflare Spectrum tại Phoenix có gây lỗi tại Việt Nam không.
- Điểm lần chạy trước cải tiến: 75/100.
- Lỗi được chọn: câu trả lời đã đọc chi tiết incident nhưng không dẫn URL nguồn.
- Câu trả lời thực tế trong `eval_before.json` (chỉ bỏ khoảng trắng cuối dòng khi trình bày; file eval giữ nguyên):

```text
Hiện tại, dịch vụ Cloudflare Spectrum đang gặp sự cố với trạng thái "degraded performance". Cụ thể, có một sự cố mang tên "Spectrum Availability Issue in Phoenix" đang được theo dõi. Sự cố này bắt đầu vào ngày 07 tháng 11 năm 2026 và liên quan đến khả năng vận hành của các ứng dụng Spectrum tại Phoenix.

Tuy nhiên, không có thông tin nào chỉ ra rằng sự cố này ảnh hưởng trực tiếp đến dịch vụ ở Việt Nam. Sự cố tại Phoenix có thể ảnh hưởng đến một số khu vực nhất định, nhưng không thể xác định được sự liên quan đến lỗi mà bạn đang gặp phải ở Việt Nam mà không có bằng chứng cụ thể hơn.

### Bước kiểm tra tiếp theo đề nghị:
1. **Kiểm tra trạng thái kết nối từ Việt Nam đến Cloudflare**: Sử dụng một công cụ nào đó để xác thực lại khả năng truy cập và hiệu suất dịch vụ Cloudflare từ địa điểm của bạn.
2. **Theo dõi cập nhật của sự cố**: Tiếp tục theo dõi trang status của Cloudflare để xem có thông tin mới về sự cố này hay không.

Nếu bạn gặp vấn đề kéo dài và có lý do để tin rằng nó có liên quan, hãy xem xét liên hệ với bộ phận hỗ trợ của Cloudflare để có thêm thông tin hoặc báo cáo cụ thể về vấn đề.
```

## 2. Bằng chứng trong trace

Tool result thực tế của A7:

```json
{
  "event": "tool",
  "step": 2,
  "name": "get_incident_details",
  "arguments": {
    "provider": "cloudflare",
    "incident_id": "c60k3g4vsoc1"
  },
  "result": {
    "ok": true,
    "provider": "cloudflare",
    "captured_at": "2026-11-09T03:50:47.638054+00:00",
    "id": "c60k3g4vsoc1",
    "name": "Spectrum Availability Issue in Phoenix",
    "status": "monitoring",
    "created_at": "2026-11-07T00:59:31.548Z",
    "updated_at": "2026-11-07T02:51:59.805Z",
    "components": [
      "Spectrum"
    ],
    "updates": [
      {
        "status": "monitoring",
        "created_at": "2026-11-07T02:51:59.805Z",
        "body": "We are continuing to monitor for any further issues.",
        "truncated": false
      },
      {
        "status": "investigating",
        "created_at": "2026-11-07T00:59:31.548Z",
        "body": "Cloudflare is investigating issues with Spectrum in Phoenix. This may be impacting the availability of existing Spectrum applications. We are working to understand the full impact and mitigate this problem. More updates to follow shortly.",
        "truncated": false
      }
    ],
    "updates_total": 2,
    "url": "https://www.cloudflarestatus.com/incidents/c60k3g4vsoc1"
  }
}
```

URL incident đã có trong tool result thành công, nhưng không xuất hiện trong final answer. Judge cũng ghi “thiếu nguồn cụ thể”. Nguyên nhân ở agent loop: baseline chấp nhận final ngay khi finish=stop và hết calls, không kiểm tra nguồn. Không khẳng định rằng các phần còn lại của câu trả lời đều đúng.

Baseline `eval_before.json` được giữ nguyên, đủ 22 case. Source baseline có SHA256 `a82b30063cc8846acdd77b8b157dfcf47b8d379119d802adabd05ee9edd1f6ab`; bản sao source chỉ lưu riêng trên máy, không thuộc hồ sơ nộp.

## 3. Thay đổi đã thực hiện

- Một thay đổi hành vi so với baseline: thêm validation URL nguồn trong `agent.py`. Giữ nguyên system prompt gốc và các xử lý lỗi đã hoàn thành ở Bài 2.
- Lưu URL từ get_incident_details thành công. Khi final không chứa bất kỳ URL incident đã quan sát nào, ghi event validation_missing_source và yêu cầu model sửa trong max_steps. Nếu hết lượt mà chưa sửa thì trả max_steps; không bắt URL khi chưa có incident thành công.
- Chọn thay đổi này vì giải quyết trực tiếp lỗi thiếu nguồn trong trace A7 và đúng ví dụ/contract ở Bài 4, tests/test_validation.py. Không hard-code câu trả lời hoặc ID incident.
- Phần prompt mở rộng của lần thử trước đã được hoàn tác để phép so sánh chỉ có một thay đổi. Source, ghi chú và kết quả cũ được lưu riêng trên máy; không chỉnh tay kết quả eval.
- Mục 1–3 được ghi trước lần eval sau cải tiến đã chỉnh phạm vi; kết quả thực tế đã được điền vào mục 4 sau khi chạy.

## 4. So sánh hai lần chạy

Hai file cùng config: provider/judge `openrouter`, model agent/judge `GPT-4o-mini`, dataset `core`, max_steps=8; dùng snapshot gốc và workers mặc định 1. Baseline được giữ nguyên; chỉ eval_after được runner chạy lại sau khi thu hẹp thay đổi và ghi mục 1–3. Đây là lần sửa phạm vi thí nghiệm, không chọn kết quả cao nhất trong nhiều lần chạy. Kết quả lần thử trước (78,18) được lưu nguyên riêng trên máy, không trộn vào bộ so sánh được nộp.

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---:|---:|
| Case được chấm | 22/22 | 22/22 |
| Điểm tổng | 76.36 | 81.36 |
| Điểm A7 | 75 | 75 |
| Số model calls agent | 50 | 51 |
| Token agent | 41358 | 41124 |
| Chi phí agent (USD) | 0.01035870 | 0.01020750 |
| Token judge | 40157 | 39592 |
| Chi phí judge (USD) | 0.00740595 | 0.00728430 |

Chi phí từ usage của provider, không gồm E2 riêng, smoke tests hoặc lần thử trước. Điểm tổng tăng 5,00; không chứng minh validation là nguyên nhân tăng điểm.

| Case | Trước | Sau |
|---|---:|---:|
| E1 | 20 | 70 |
| E2 | 80 | 100 |
| E3 | 80 | 75 |
| E4 | 75 | 100 |
| E5 | 65 | 70 |
| E6 | 75 | 75 |
| A1 | 90 | 75 |
| A2 | 95 | 100 |
| A3 | 75 | 80 |
| A4 | 100 | 100 |
| A5 | 55 | 60 |
| A6 | 80 | 70 |
| A7 | 75 | 75 |
| A8 | 75 | 90 |
| H1 | 60 | 60 |
| H2 | 100 | 100 |
| H3 | 100 | 100 |
| H4 | 100 | 80 |
| H5 | 95 | 100 |
| H6 | 60 | 60 |
| H7 | 50 | 75 |
| H8 | 75 | 75 |

- Kết quả nguồn của case đã chọn: A7 trước không dẫn nguồn; sau có URL `https://www.cloudflarestatus.com/incidents/c60k3g4vsoc1`. Điểm A7 không đổi. Final answer sau được giữ trong eval_after.json, không sửa tay.
- Số event validation_missing_source trong eval mới: 0. Model tự dẫn URL trong các case đã đọc incident thành công, nên chưa quan sát được gate sửa câu trả lời ở lần eval này. Hai unit tests kiểm tra yêu cầu sửa và hết budget đã pass; test incident thất bại không bắt URL cũng pass.
- Source baseline chạy lại offline toàn bộ 37 tests: 35 pass, chỉ 2 tests validation nguồn fail, đúng checkpoint CP3. Source cuối: 37/37 tests pass. Các bản sao source được lưu riêng trên máy để đối chiếu; source cuối của hồ sơ nộp là agent.py.
- Lỗi còn lại: E1 vẫn dùng từ hiện tại cho snapshot; H1/H6 vẫn bỏ qua incident gộp, đều bị cap 60 do chỉ một tool step. H7 lần này không bịa operational cho Cloudflare timeout nhưng vẫn gọi trạng thái GitHub từ snapshot là hiện tại; không kết luận lỗi này đã ổn định vì không có thay đổi kiểm soát timeout mới.
- Suy giảm cần ghi nhận: A1 90 → 75, A6 80 → 70, H4 100 → 80. A5 tăng 55 → 60 nhưng vẫn thấp. H4 không gọi tool và vẫn được judge xác nhận làm đúng yêu cầu thơ, minh họa biến động/giới hạn judge; không quy mọi thay đổi điểm cho code.
- Smoke tests bản cuối: cả snapshot và live đều completed, mỗi run 2 model calls/1 tool call. Snapshot vẫn kết luận hiện tại không đúng giới hạn dữ liệu; live vẫn kết luận không có cache purge dù chỉ lấy 5 incident chưa lọc. Kết quả chạy thử được lưu riêng trên máy để phân tích, không coi completed là câu trả lời đúng.

## 5. Giới hạn

- Judge là phép đo xấp xỉ và dùng cùng model với agent; có thể thiên lệch, bỏ sót lỗi hoặc giải thích sai. Completed chỉ là kết thúc bình thường. Điểm eval không tự động là điểm lab.
- Snapshot bài chính lưu ngày 09/11/2026, không xác nhận trạng thái hiện tại. Tìm theo chuỗi trong 50 incident, trả tối đa 5; chi tiết chỉ giữ hai update mới nhất và update đầu tiên, tối đa 900 ký tự mỗi nội dung.
- Một lần chạy mỗi phiên bản không chứng minh độ ổn định hoặc quan hệ nhân quả với điểm số.
- Validation chỉ kiểm tra sự hiện diện của ít nhất một URL incident đã quan sát, không chứng minh mọi claim đúng, mọi provider có nguồn hoặc mọi URL khác đều hợp lệ. Không sửa các lỗi snapshot, timeout hoặc bỏ sót follow-up bằng thay đổi này.
