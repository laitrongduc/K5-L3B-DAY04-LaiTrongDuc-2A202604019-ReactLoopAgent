# Bằng chứng thực hành và rà soát

- `agent_before.py`: source baseline tương ứng `../eval_before.json`, đã hoàn thành loop và xử lý lỗi Bài 1–2, chưa validation nguồn.
- `agent_after.py`: bản sao source cuối tương ứng `../eval_after.json`; chỉ thêm validation nguồn so với baseline (ngoài docstring).
- `smoke_snapshot.json`, `smoke_live.json`: kết quả chạy thử thật trên source cuối. `completed` không bảo đảm câu trả lời đúng; xem lỗi còn lại trong `../NOTES.md`.
- `eval_e2.json`: bản sao nguyên vẹn của `../eval_results_e2.json` (95/100 cho riêng E2). Bản gốc bị ignore bởi quy tắc `eval_results*.json`; bản sao này có thể đưa vào Git. Không dùng điểm E2 riêng thay cho điểm bộ 22 case.
- `previous_attempt/`: giữ nguyên source, NOTES và kết quả eval sau cải tiến của lần thử nhiều thay đổi trước khi rà soát. Đây là lịch sử, không phải bộ kết quả chính hiện tại.

Hai file dùng so sánh chính là `../eval_before.json` và `../eval_after.json`. Không chỉnh tay file kết quả eval. Việc chạy lại sau rà soát được giải thích trong mục 3–4 của `../NOTES.md`; hồ sơ bằng chứng không phụ thuộc nhật ký riêng.

`verification_manifest.json` lưu hash source/fixtures/báo cáo và số liệu được tính lại trong lượt đối soát. Trạng thái CP1–CP7 và giới hạn xem tại `../CHECKPOINT_REVIEW.md`. Manifest phản ánh thời điểm kiểm tra; nếu thay đổi source hoặc dữ liệu, cần kiểm tra lại.
