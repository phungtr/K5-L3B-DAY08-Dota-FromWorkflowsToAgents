# Decision record

- Shape chọn: workflow
- Commit hash: 4f2060dc4729fa13d235ef41d4ed88480331b4f8
- Run id dẫn chứng: R-workflow-C-001-s11; R-workflow-C-014-s55; xem src/reports/benchmark.json (70 trial workflow).
- Pass rate / pass^5 / seed min-max / variance: 1.000000 / 1.000000 / 1.000000–1.000000 / 0.000000 trên 14 public claim × 5 seed.
- Cost per success (mô phỏng, VND): 28.862857 (tổng 2020.40 VND / 70 thành công).
- Crash safety: 20/20 trong src/reports/crash.json; thêm 24 test recovery/approval dùng chung workflow, single, orchestrator.
- Secret leak / approval bypass: 0 / 0 quan sát được trên 70 trial public của workflow qua evaluator; các test thiếu approval và approval sai claim cũng không tạo refund.

## Phiên bản được đánh giá

Đánh giá ngày 2026-11-17 trên nhánh `thanh-ledger-evaluation` tại mốc nguồn `4f2060d`, sau khi merge đủ code ba thành viên. Nhánh này đã vào `main` qua PR #3 (`2a8fa53`). Từ `4f2060d` đến `2a8fa53` chỉ đổi test và tài liệu, không đổi shape, host, config, data, `benchmark.json` hay `crash.json`. Trên `main`, chạy lại benchmark và crash cho cùng pass rate, pass^5, seed min–max, variance và `20/20`; cost chỉ lệch ở chữ số thứ 12 do cộng số thực.

## Vì sao shape này

Chọn workflow làm kiến trúc nộp/chấm hidden: bài toán có luồng tuần tự và quy tắc xác định, chưa có lợi ích chất lượng quan sát được từ quyền tự chủ cao hơn. Trên cùng 14 claim, seed 11/22/33/44/55, MockLLM, pricing và evaluator, workflow/single/orchestrator đều pass 70/70 trial và pass^5 = 1. Workflow có cost/success thấp nhất trong bốn shape chính thức và mức tự chủ thấp nhất. Kết luận này dựa trên public, chưa khẳng định đạt ngưỡng hidden.

| Shape | Thành công | Pass rate / pass^5 | Seed min–max | Variance | Cost/success (VND mô phỏng) | Mean model/tool calls | Mean/p95 trace steps |
| --- | --- | --- | --- | ---: | ---: | --- | --- |
| workflow | 70/70 | 1 / 1 | 1–1 | 0 | 28.862857 | 1 / 2.785714 | 4.428571 / 5 |
| single | 70/70 | 1 / 1 | 1–1 | 0 | 29.798571 | 3.785714 / 2.785714 | 4.428571 / 5 |
| orchestrator | 70/70 | 1 / 1 | 1–1 | 0 | 57.258571 | 1.928571 / 5.571429 | 4.5 / 5 |
| swarm | 69/70 | 0.985714 / 0.928571 | 0.928571–1 | 0.000816327 | 49.566957 | 3 / 4.657143 | 2.8 / 3 |

Nguồn bảng: `src/reports/benchmark.json` do công cụ sinh, 280 trial, SQLite mới cho mỗi trial. Cost/success tính cả cost của trial fail. `steps` là số event trace; không dùng nó để suy ra tốc độ hoặc so trực tiếp hiệu suất khi các shape trace khác nhau.

## Có đáng đổi lấy multi-agent không

Hiện giữ workflow. Single đắt hơn 0.935714 VND/success; orchestrator đắt hơn 28.395714 VND/success mà chưa tăng pass rate trên public. Swarm có chi phí và variance cao hơn workflow, fail `R-swarm-C-002-s55` vì lấy ngày trễ trong ticket thay vì dữ liệu order. Đây là điểm yếu đối chứng được chủ ý giữ lại.

Quyết định sẽ được xem lại nếu task mới cần nhánh điều tra linh hoạt hoặc phân chia công việc, và thử nghiệm có nhãn trên cùng setup chứng minh lợi ích chất lượng đủ bù chi phí, vẫn giữ không bypass approval, không leak và không trả trùng. Không quy toàn bộ chênh lệch cho topology: prompt, số model/tool calls và cách ghi trace giữa các triển khai còn khác nhau.

## Trace và bằng chứng

Workflow trace chỉ ghi thông tin claim có tồn tại/có order, order view đã lọc, quote, kết quả host và quyết định. Không ghi ticket, parent secret, raw headers, điện thoại, địa chỉ, tên hàng hoặc ghi chú nội bộ. Single ghi action/decision; orchestrator ghi handoff đã qua allowlist và kết quả investigator đã lọc. Không dùng LLM summary thay cho sanitization.

Evaluator kiểm trace mới, JSON hợp lệ, run_id khớp; đối chiếu RunResult với ledger/audit và quét các chuỗi cấm trong kết quả cùng artifact cùng run id. Các trial workflow đều pass; không diễn giải đó thành chứng minh không leak trên mọi dữ liệu. Trace nằm trong thư mục mặc định được scanner kiểm; không chuyển ra ngoài để né scanner.

Crash suite host/swarm đạt 20/20. Bộ test của Thành có 24 test trên ba shape: hai claim × ba crash point và hai trường hợp approval/shape; đóng rồi mở lại kết nối, resume và retry ba lần. C-014 giữ full quote 150000, chỉ ghi thêm 120000, tổng hoàn theo order là 150000. Bộ mặc định 195 passed; student 248 passed. Student xanh gồm assertion chấp nhận lỗi tham chiếu swarm, không có nghĩa benchmark bốn shape đều đạt 100%.

Run id minh họa: `R-workflow-C-001-s11`, `R-single-C-001-s11`, `R-orchestrator-C-014-s55`, `R-swarm-C-002-s55`. Run id xác định theo shape/claim/seed, không phải batch ID duy nhất; trace local có thể được ghi lại khi chạy lại test. Lệnh tái tạo và lịch sử checkpoint nằm trong `src/reports/README.md`.

## Giới hạn

MockLLM không phải model sản xuất; giá mô phỏng, latency `unavailable`. Chưa có `student_drop/` hoặc nhãn hidden trên máy; không khai đạt ngưỡng hidden 0.75 chỉ từ kết quả public. Ablation draft trong research card chỉ đo tác động chi phí của thành phần draft, không đo chất lượng văn bản và không chứng minh topology nào tốt hơn nói chung. Workflow_no_draft là variant nghiên cứu, không phải shape chính thức được chọn để nộp.
