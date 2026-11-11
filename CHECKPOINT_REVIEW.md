# Đối soát checkpoints · 11/11/2026

Kết luận: sản phẩm cục bộ của CP2–CP6 đáp ứng contract và nhất quán với code, tests, trace, kết quả eval cùng NOTES. CP1 đã xác minh môi trường, tools, cấu hình chạy được và repo public; chưa xác minh nguồn fork hoặc thao tác push thật. CP7 chưa hoàn thành. Các nhận xét về việc học viên hiểu/giải thích code cần được kiểm tra trực tiếp.

## Trạng thái theo checkpoint

| Checkpoint | Bằng chứng đã kiểm tra | Trạng thái / việc còn lại |
|---|---|---|
| CP1 — Setup và khám phá tools | Python 3.12.16; hai tools snapshot đã chạy; cấu hình OpenRouter/model trong `.env` khớp eval; các smoke test thật đã hoàn tất; repo đúng tên, GitHub metadata xác nhận public và quyền push | Phần kỹ thuật đã xác minh. Chưa xác minh cờ fork hoặc đã push thử qua Git/SSH; quyền push không thay thế bằng chứng push thành công |
| CP2 — Agent loop | `agent.py`; tests tool-call IDs, nhiều calls, trajectory hai tools; `artifacts/smoke_snapshot.json` completed | Đạt contract loop; kết quả nội dung vẫn cần đối chiếu bằng chứng |
| CP3 — Lỗi và trace | Source baseline `artifacts/agent_before.py`; kiểm tra baseline 35/37 pass, chỉ hai tests nguồn fail; trace có event và đủ trường; lỗi JSON/tool/provider và finish được xử lý | Đạt. Bản cuối có thêm validation nguồn nên hiện pass cả 37 tests |
| CP4 — Eval trước | `eval_before.json`: 22/22 case, 76,36/100; NOTES dẫn answer/trace A7 và các lỗi khác | Đạt; baseline giữ nguyên |
| CP5 — Phân tích và cải tiến | NOTES mục 1–3 được ghi trước run sau đã chỉnh phạm vi; case A7 thiếu nguồn; source cuối chỉ thêm validation nguồn so với baseline, ngoài docstring | Đạt; chỉ bắt có ít nhất một URL incident đã quan sát, không xác minh mọi claim |
| CP6 — Eval sau và so sánh | `eval_after.json`: 22/22 case, 81,36/100; config bằng baseline; NOTES mục 4–5 có bảng từng case, calls, token, chi phí, suy giảm và giới hạn | Đạt. Một lần mỗi phiên bản không chứng minh độ ổn định hoặc nguyên nhân tăng điểm |
| CP7 — Demo, chốt và nộp | Đã có code, NOTES, eval và trace phục vụ demo; các thay đổi còn trong working tree, nhánh hiện tại `feat/status-investigation-agent` | Chưa thực hiện demo trực tiếp, commit/push thay đổi mới hoặc nộp LMS. Chưa merge vào main |

## Bộ kết quả chính

| Chỉ số | Trước | Sau |
|---|---:|---:|
| Điểm tổng | 76,36 | 81,36 |
| Case được chấm | 22/22 | 22/22 |
| Model calls agent | 50 | 51 |
| Token agent | 41.358 | 41.124 |
| Chi phí agent (USD) | 0,01035870 | 0,01020750 |
| Token judge | 40.157 | 39.592 |
| Chi phí judge (USD) | 0,00740595 | 0,00728430 |
| Điểm A7 | 75 | 75 |

Agent và judge cùng dùng OpenRouter / `GPT-4o-mini`, dataset core, snapshot gốc, max_steps=8; workers mặc định 1. `.env` hiện khớp provider/model trong hai file eval. Cấu hình cùng nhau không chứng minh model/judge luôn cho kết quả giống nhau.

- `eval_results_e2.json` đạt 95 cho riêng E2; đã lưu bản sao nguyên vẹn tại `artifacts/eval_e2.json` vì bản gốc bị ignore theo `.gitignore`. E2 trong bộ trước đạt 80, trong bộ sau đạt 100: đây là các lần chạy khác nhau, không phải số liệu mâu thuẫn.
- Điểm 78,18 và source/prompt mở rộng thuộc lần thử cũ, được giữ nguyên tại `artifacts/previous_attempt/`; không trộn vào bộ so sánh hiện tại.
- Code hiện tại khớp byte-for-byte với `artifacts/agent_after.py`. Source baseline có bản lưu riêng. Hash và số liệu kiểm chứng nằm trong `artifacts/verification_manifest.json`.
- Bảng 22 case, trích answer A7 và JSON trace trong NOTES khớp dữ liệu gốc. Trích answer chỉ bỏ khoảng trắng cuối dòng để trình bày; các file eval không bị chỉnh tay.

## Kiểm tra tính toàn vẹn

- `python3 -m unittest discover -s tests -v`: 37/37 pass trong lượt đối soát này. Các tests bonus tools trong suite không chứng minh agent đã implement bonus.
- Tính lại summary, mechanical checks, số model events/usage và replay observations của các báo cáo đều khớp. Bộ trước có 46 tool observations, bộ sau có 44; E2 riêng có 4 và lần thử cũ có 34.
- `git diff --check` pass. Fixtures, dataset, scenarios, grading, tests, CLI, provider adapter và tài liệu đề/rubric giữ nguyên so với HEAD.
- `.env` không được theo dõi bởi Git. Không tìm thấy giá trị API key từ `.env` trong các tệp được Git theo dõi hoặc tệp mới không bị ignore tại thời điểm rà soát. Không lưu key vào báo cáo.
- Thông tin người dùng sửa trong NOTES (`LẠI TRỌNG ĐỨC`) được giữ nguyên. Thư mục `.vscode/` không bị sửa trong lượt đối soát.

## Giới hạn và việc còn lại

- Model vẫn nhầm snapshot với hiện tại ở E1; H1/H6 chưa đọc incident liên quan và bị cap 60. Smoke test live vẫn kết luận quá mức từ danh sách chưa lọc. Những lỗi này được ghi trong NOTES, không tuyên bố đã sửa hết.
- Không có event validation_missing_source trong eval sau thật; unit tests chứng minh đường yêu cầu sửa nguồn, nhưng không đủ để quy mức tăng 5 điểm cho thay đổi này.
- Demo có thể dùng case A6 trong eval_after: đọc incident gốc rồi incident liên quan ở tool step tiếp theo. Dùng E1 để trình bày lỗi diễn đạt thời điểm snapshot. Đây mới là gợi ý chuẩn bị, chưa phải demo đã thực hiện.
- Trước khi nộp: học viên đọc/giải thích code và NOTES, demo, kiểm tra các tệp cần đưa vào Git, commit/push, quyết định merge theo thỏa thuận, và nộp link LMS trước hạn theo ca học. Không xác nhận được việc nộp LMS từ các tệp cục bộ.
- `WORK_LOG.md` là nhật ký riêng cho người dùng, không thuộc hồ sơ nộp. Không stage, commit hoặc push tệp này. `NOTES.md` là ghi chú phân tích bắt buộc cần nộp theo đề.
- Có lỗi diễn đạt sẵn trong tài liệu gốc: CHECKPOINTS mở đầu ghi “1 giờ nghỉ”, nhưng bảng của chính file và GUIDE đều dành 10 phút nghỉ. Giữ nguyên đề; việc đối soát dựa trên tiêu chí CP1–CP7 và lịch chi tiết, không sửa tài liệu gốc.

Lượt này chỉ đối soát và chỉnh hồ sơ; không sửa agent, không chạy lại LLM/eval và không commit/push/merge hoặc nộp LMS.

## Đối chiếu RUBRIC.md

| Nhóm tiêu chí bài chính | Điểm tối đa | Bằng chứng hiện có | Điểm cần lưu ý |
|---|---:|---|---|
| Agent loop | 30 | agent.py giữ assistant message nguyên vẹn, xử lý mọi call, đúng tool_call_id và điều kiện dừng; tests và trace nhiều bước | Không dùng framework. Học viên cần giải thích được code, không suy ra điểm đạt chỉ từ tests |
| Robustness và trace | 15 | max_steps, INVALID_ARGUMENTS, lỗi tool/provider, finish bất thường; trace đủ event/step/name/arguments/result | Lỗi tool có observation để model xử lý; lỗi provider kết thúc run. Hiện không phát hiện lỗi contract bắt buộc |
| Eval và phân tích | 20 | Hai file đủ 22/22 case, điểm khác null, cùng config; NOTES có case A7, answer/trace thật, bảng so sánh và giới hạn judge | Không dùng E2 riêng làm điểm cả bộ; không trộn kết quả cũ 78,18 vào bộ chính 76,36 → 81,36 |
| Cải tiến và trình bày | 20 | Một cải tiến gate nguồn, tests nguồn pass, so sánh trước/sau và lỗi còn lại được giải thích | Demo trực tiếp và khả năng giải thích của học viên chưa xác minh. Đã chuẩn bị DEMO.md, không coi kịch bản là demo đã hoàn thành |

Đây là đối chiếu bằng chứng, không tự chấm 85/85. Điểm eval 81,36/100 khác điểm bài chính tối đa 85. Bonus 15 điểm là tùy chọn và chưa thực hiện.

Các điều kiện mất điểm trực tiếp trong rubric đã kiểm tra: không thấy framework, thiếu tool_call_id, bỏ sót tool calls, crash ở các tình huống lỗi tool được kiểm tra, completed khi length, trace thiếu event, tổng điểm null, khác config, thiếu evidence hoặc thiếu so sánh. Tệp dataset/fixtures/grading và rubric không bị sửa. Các hạn chế của model vẫn tồn tại và đã ghi trung thực, không đảm bảo mọi câu trả lời đều đúng.

- H1/H6 vẫn bị cap 60 theo mechanical checks vì thiếu lượt điều tra tiếp; không che hoặc sửa tay các điểm này. Cần giải thích được khi demo.
- Đã thêm `/WORK_LOG.md` vào .gitignore và xác nhận bằng git check-ignore để tránh commit nhầm nhật ký riêng. Tài liệu artifacts đã tham chiếu NOTES thay vì phụ thuộc WORK_LOG.
- Đã kiểm tra không thấy giá trị API key hiện tại trong các working files được rà soát và 76 Git blobs reachable. Đây không phải cam kết phát hiện mọi loại secret hoặc các key cũ không còn trong cấu hình.
- Tên repo đúng mẫu, metadata GitHub trong lần rà soát trước là public. Chưa đưa thay đổi hiện tại lên remote; chỉ repo public chưa có nghĩa giảng viên đã thấy bài làm mới.
- Chưa xác nhận ca học/ngày lab, hạn riêng do coach thông báo, thời điểm nộp hoặc link đã dán lên LMS. Không tự đánh dấu nộp đúng hạn.

Việc còn lại để tránh mất điểm thủ tục: học viên đọc/giải thích code và NOTES; demo trace nhiều bước và một lỗi thật theo DEMO.md; kiểm tra file được stage, commit/push và mở repo để xác nhận đúng phiên bản; nộp đúng link LMS trước hạn của ca học. Không stage/commit/push WORK_LOG.md hoặc .env. Merge main chỉ khi người dùng yêu cầu.
