# Báo cáo nhóm — Lab 8 LogiX-VN

## 1. Thông tin nhóm

- Nhóm: Dota, lớp L3B.
- Thành viên và phân công: xem `TEAM.md`.
- Phiên bản nguồn được đánh giá: `4f2060dc4729fa13d235ef41d4ed88480331b4f8`, có đủ workflow, single và orchestrator. Commit nộp bài ghi ở dòng `Commit hash` của ADR.
- Nhánh nộp: `main`. Ba nhánh thành viên đã merge vào `main` qua PR #1 (single), PR #2 (orchestrator), PR #3 (evaluation, `2a8fa53`). Từ `4f2060d` đến `2a8fa53` không đổi code shape, `benchmark.json` hay `crash.json`.

## 2. Mục tiêu và cách tiếp cận

Cùng xử lý claim hoàn phí vận chuyển LogiX-VN bằng bốn kiến trúc để chọn mức tự chủ thấp nhất vẫn đáp ứng yêu cầu. Nhóm triển khai workflow, single và orchestrator; swarm có sẵn làm đối chứng. Tiền do code tính, host kiểm approval, trần theo order, idempotency và ghi ledger; model chỉ tham gia đọc/soạn theo cơ chế MockLLM của lab.

Trà phụ trách workflow/integration; Khánh phụ trách single; Tuấn Anh phụ trách orchestrator/handoff; Thành phụ trách recovery, evaluation, ADR và research card. Thành kiểm tra từng shape sau khi merge, rồi chạy lại đủ bốn shape cùng corpus, seed, pricing và evaluator. Mỗi trial dùng SQLite mới; riêng test retry/crash dùng lại database để kiểm hiệu ứng đã ghi.

## 3. Giải pháp

Workflow có trình tự Python cố định: đọc claim view → kiểm order/ownership → tính quote từ order và policy → từ chối nếu quote 0, hoặc gọi host với full quote → trả RunResult và trace. Ticket không điều khiển số tiền. Một lời gọi MockLLM draft không quyết định hoặc ghi refund.

Single chạy `run_react` tối đa 8 bước, chọn các tool đọc để lấy claim, order và quote; sau vòng lặp mới gọi host ghi tiền. Trong bản lab, logic chọn action nằm trong code và output MockLLM không quyết định số tiền. Agent không được phép tự ghi ledger.

Orchestrator tạo handoff bằng `make_handoff`, chỉ cấp tool đọc cho investigator, thu order/policy/history/quote, rồi gọi host với full quote. Investigator không có registry ghi. Handoff và trace qua allowlist; không chuyển parent secret hoặc raw headers. LLM summary không được dùng làm bộ lọc bí mật.

Cả ba chuyển tiếp `crash_at`, ghi trace khi InjectedCrash rồi raise tiếp, đóng kết nối ở finally. Host đảm bảo ledger và audit committed trong cùng transaction; checkpoint hỗ trợ tiếp tục nhưng không thay thế ledger. C-014 truyền quote 150000 để host tự trừ 30000 đã hoàn và chỉ ghi thêm 120000.

## 4. Kết quả

Ngày chạy 2026-11-17. Số liệu từ `src/reports/benchmark.json`, 14 claim × 5 seed (11,22,33,44,55) × 4 shape = 280 trial.

| Shape | Thành công | Pass rate | Pass^5 | Seed min–max | Variance | Cost/success (VND mô phỏng) | Mean/p95 steps |
| --- | --- | ---: | ---: | --- | ---: | ---: | --- |
| workflow | 70/70 | 1 | 1 | 1–1 | 0 | 28.862857 | 4.428571 / 5 |
| single | 70/70 | 1 | 1 | 1–1 | 0 | 29.798571 | 4.428571 / 5 |
| orchestrator | 70/70 | 1 | 1 | 1–1 | 0 | 57.258571 | 4.5 / 5 |
| swarm | 69/70 | 0.985714 | 0.928571 | 0.928571–1 | 0.000816327 | 49.566957 | 2.8 / 3 |

Mean model/tool calls lần lượt: workflow 1/2.785714; single 3.785714/2.785714; orchestrator 1.928571/5.571429; swarm 3/4.657143. Cost/success tính cả cost trial fail. Steps là event trace, không phải phép đo latency hoặc thước đo thời gian so sánh trực tiếp giữa shape.

Swarm fail C-002 seed 55 (`R-swarm-C-002-s55`) vì ưu tiên ngày trễ trong ticket. Đây là điểm yếu đối chứng của lab; student test có assertion chấp nhận lỗi này, nên 248 student test xanh không có nghĩa cả bốn shape đều pass 100% benchmark.

Bộ mặc định **195 passed**, student **248 passed**. Crash suite chính đạt **20/20** trong `src/reports/crash.json` (host và một scenario swarm). Ngoài ra Thành bổ sung **24 test recovery/approval** cho cả ba shape: ba crash point, mở lại kết nối, resume và retry nhiều lần, hoàn chênh lệch C-014 và từ chối approval thiếu/sai claim. Workflow 70/70 trial qua evaluator đối chiếu ledger/audit/trace và chuỗi cấm; chưa quan sát leak hoặc approval bypass trong phạm vi đó. Chưa có kết quả hidden.

Quyết định ADR: **workflow**, vì mức tự chủ thấp nhất, cùng chất lượng public với single/orchestrator và cost/success thấp hơn. Chỉ xem lại nếu dữ liệu có nhãn mới chứng minh lợi ích của kiến trúc khác đủ bù chi phí, vẫn qua các cổng an toàn. Xem `report/team/decision_adr.md`.

Trên `main` sau khi merge đủ ba nhánh (`2a8fa53`), `check_submission` đạt **0 FAIL, 0 WARN**. Trà chạy lại benchmark và crash vào thư mục tạm để kiểm tái tạo: pass rate, pass^5, seed min–max, variance trùng khớp file đã commit; cost chỉ lệch ở chữ số thứ 12 do cộng số thực; crash `20/20` trùng khớp. Checker xanh không có nghĩa là đạt ngưỡng hidden.

Research ablation R4: bật/tắt duy nhất draft của workflow, 70/70 trial pass mỗi variant, cost/success 28.862857 → 27.857143; tổng giảm 70.40 VND mô phỏng (3.48%). Cùng tools, corpus, seeds và evaluator. Kết quả tại `src/reports/research_ablation.json`, được giữ từ commit `175d770` vì workflow/helper không đổi. Thí nghiệm không đo chất lượng văn bản và không cô lập topology; xem research card.

## 5. Khó khăn và cách xử lý

Môi trường của Thành ban đầu dùng Python 3.15: smoke báo thiếu PyYAML, pytest báo thiếu module, cài PyYAML thất bại với lỗi cần Microsoft Visual C++ 14.0. Tạo lại .venv bằng Python 3.11.9 và cài requirements đã xử lý lỗi, không thay constraints hoặc host.

Hai lần tích hợp phát sinh khác biệt giữa benchmark chưa đủ shape và benchmark mới. Merge orchestrator xung đột TEAM/benchmark: giữ đóng góp cả Thành/Tuấn Anh trong TEAM, tạo lại JSON bằng benchmark chính thức thay vì chọn số liệu hoặc sửa tay. Merge đã được lưu tại `4f2060d`. Để tránh xung đột lặp lại, mỗi thành viên làm trên nhánh riêng tạo từ `main` và vào `main` qua pull request; Trà merge lần lượt PR #1–#3 và chạy lại cổng host, ma trận student, `check_submission` sau lần merge cuối.

Tên paper R4 trong mẫu chưa xác minh được; research card ghi rõ nguồn thay thế đã đọc là *AI Agents That Matter* (2024), không gán nội dung cho paper EMNLP 2026 chưa xác minh. Nếu lớp bắt buộc đúng paper trong slide thì cần đối chiếu với coach; không bịa nguồn.

## 6. Tổng hợp đóng góp

Trà: workflow và 22 test bổ sung (`f0fb63f`); integration (merge PR #1–#3, kiểm tái tạo benchmark/crash). Khánh: single (`13f5978`) và báo cáo cá nhân. Tuấn Anh: orchestrator, handoff và 23 test bổ sung (`2346e37`). Thành: test recovery, helper/72 test ablation, báo cáo benchmark/crash, ADR và research card (`175d770`, `db22c3b`, `9daa602`).

Bảng đóng góp, lịch sử checkpoint và báo cáo cá nhân nằm trong TEAM và `report/team/`. Mỗi thành viên tự xác nhận đóng góp, việc sử dụng AI và hiểu kết quả trước khi nộp; không khai commit của người khác thành commit của mình.

## 7. Hạn chế và hướng cải thiện

MockLLM và dữ liệu giả giới hạn khả năng suy rộng; giá mô phỏng, latency `unavailable`. Chưa có student_drop/nhãn hidden để kết luận đạt ngưỡng 0.75 trên 200 hidden trial. Public có thể chưa bao phủ lỗi ngoài phân phối.

Các shape khác cả cách gọi model/tool và trace nên chênh lệch benchmark không chứng minh chỉ topology gây ra. Draft workflow hiện không được đưa vào RunResult, vì vậy ablation không đánh giá chất lượng giao tiếp. Cần đánh giá có nhãn mới và đo chất lượng nghiệp vụ lẫn chi phí trước khi tăng mức tự chủ.

## 8. Tài liệu tham khảo

- Quy tắc và contract của lab: [business rules](../../docs/business_rules.md), [contracts](../../docs/contracts.md), [metrics](../../docs/metrics.md).
- Bằng chứng sinh tự động và lệnh tái tạo: [reports README](../../src/reports/README.md).
- Kapoor và cộng sự (2024), [AI Agents That Matter, arXiv:2407.01502v1](https://arxiv.org/html/2407.01502v1), nguồn thay thế cho research card R4.
