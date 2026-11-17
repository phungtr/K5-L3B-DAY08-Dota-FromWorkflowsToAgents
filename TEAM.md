# Team

## Thông tin nhóm

- Tên nhóm: Dota
- Lớp: L3B
- Repo nhóm: `K5-L3B-DAY08-Dota-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt: https://github.com/phungtr/K5-L3B-DAY08-Dota-FromWorkflowsToAgents · nhánh `main` · commit chốt: [`4f2060d`](https://github.com/phungtr/K5-L3B-DAY08-Dota-FromWorkflowsToAgents/commit/4f2060dc4729fa13d235ef41d4ed88480331b4f8) (trùng `Commit hash` trong `report/team/decision_adr.md`)

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Phùng Quang Trà |2A202604094|traquangphung@gmail.com|phungtr| Nhóm trưởng · workflow (A) + integration |
|   2 | Vũ Nam Khánh |2A202604112|khanhvnm03@gmail.com|khanhvnm| single agent (B) |
|   3 | Nguyễn Tuấn Anh|2A202604089|capntryisreal@gmail.com|TuanAnhNg131| orchestrator (C) + handoff |
|   4 | Phạm Duy Thành |2A202604075|thanhteyv2@gmail.com|thanhtey1234| ledger/crash + evaluation (benchmark, ADR) |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Phùng Quang Trà | workflow (A) + integration | `f0fb63f` | `src/tests/test_workflow_shape.py` (22 test) |
| Vũ Nam Khánh | single agent (B) | `13f5978` | Ma trận student Single Agent (62 test: T01–T14, retry, crash) |
| Nguyễn Tuấn Anh | orchestrator (C) + handoff | `2346e37` | `src/tests/test_orchestrator_shape.py` (23 test) |
| Phạm Duy Thành | ledger/crash + evaluation | `175d770` (giai đoạn 1–2); `db22c3b` (single/evaluation); `9daa602` (hoàn thiện evaluation, ADR) | `src/tests/test_team_crash_recovery.py` (24 test đủ ba shape), `src/tests/test_research_draft_ablation.py` (72 test) |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Mỗi thành viên điền mục của mình dưới đây (đổi `MSSV_HoTen` thành MSSV và họ tên không dấu).

## Thành viên 1 — 2A202604094_PhungQuangTra

### Phần việc phụ trách

- Shape A `workflow` (`src/lab8/shapes/workflow.py`) + integration

### Đóng góp cụ thể

- Viết shape `workflow` theo 7 bước cố định: đọc claim qua `get_claim_view` → thiếu `order_id` thì `needs_info` → `lookup_order` (không thấy/không sở hữu thì `escalated`) → quote bằng `compute_compensation` → quote 0 thì `denied` → quote > 0 thì gọi `durability.issue_refund` với quote đầy đủ → `resolved` / `denied` / `failed` (conflict).
- Ranh giới code/model: tiền do code tính; MockLLM chỉ được gọi 1 lần để soạn câu trả lời, prompt chỉ có `status` và số tiền, không chứa ticket. Số ngày trễ ghi trong ticket bị bỏ qua.
- Durability: truyền `crash_at` vào `issue_refund`; khi `InjectedCrash` thì ghi trace rồi raise tiếp, không nuốt lỗi.
- Trace: chỉ ghi field đã qua allowlist; không ghi ticket, `parent_secret`, header.
- Thêm 22 test trong `src/tests/test_workflow_shape.py`: ticket chèn lệnh không làm tăng tiền (5 seed), output của model không quyết số tiền, claim không tồn tại trả `failed` có trace hợp lệ, gọi lại với số tiền khác thì `amount_conflict` và giữ dòng cũ, trace của 14 claim public không lộ secret/header/ticket.

### Bằng chứng

- Commit [`f0fb63f`](https://github.com/phungtr/K5-L3B-DAY08-Dota-FromWorkflowsToAgents/commit/f0fb63f) (nhánh `Phungtr-04094`).
- `python -m pytest -q -m student -o addopts= -k workflow` → 62 passed.
- `python -m pytest -q` → 76 passed (cổng host xanh, gồm 22 test mới).
- `python -m lab8.check_submission` → `[ OK ] shape workflow: format đúng, 12/12 claim public pass ở seed 11`.
- Báo cáo cá nhân: [report/team/2A202604094_PhungQuangTra.md](report/team/2A202604094_PhungQuangTra.md).

**Nhật ký integration**

| Lần | Ngày | Merge vào `main` | Cổng host `pytest -q` | Ma trận student | `check_submission` |
| --- | --- | --- | --- | --- | --- |
| 1 | 2026-11-17 | `Phungtr-04094` → `main`, fast-forward tới `f0fb63f`, không conflict | 76 passed | 124 passed / 124 failed (workflow 62/62, swarm 62/62; 124 fail là của `single`, `orchestrator` chưa merge) | 12 FAIL (2 shape chưa có; chưa có benchmark, crash, ADR, research card) |
| 2 | 2026-11-17 | PR #1 `khanhvnm-04112` (`9571721`), PR #2 `nta` (`21cd806`), PR #3 `thanh-ledger-evaluation` (`2a8fa53`). Conflict TEAM/benchmark đã được Thành giải trên nhánh của mình tại `4f2060d` trước khi mở PR | 195 passed | 248 passed, 0 failed | 0 FAIL, 0 WARN |

Kiểm tra tái tạo sau lần 2 (trên `main` tại `2a8fa53`, chạy lại vào thư mục tạm, không ghi đè `src/reports/`):

- `lab8.benchmark` 4 shape × 14 claim × 5 seed: pass rate, pass^5, seed min–max, variance trùng khớp với `src/reports/benchmark.json`; cost chỉ lệch ở chữ số thứ 12 trở đi do cộng số thực (ví dụ workflow 28.862857142857138 → 28.862857142857145), không ảnh hưởng số 6 chữ số trong ADR.
- `lab8.crash_test --count 20`: `20/20`, trùng khớp `src/reports/crash.json`.
- `git diff 4f2060d 2a8fa53` không đổi `src/lab8/shapes/`, `src/reports/benchmark.json`, `src/reports/crash.json`.


## Thành viên 2 — 2A202604112_VuNamKhanh

### Phần việc phụ trách

- Shape B `single` (`src/lab8/shapes/single.py`).

### Đóng góp cụ thể

- Triển khai vòng ReAct bằng `run_react` với giới hạn tối đa 8 bước.
- Cho agent chỉ sử dụng các tool đọc: `get_claim_view`, `lookup_order` và
  `compute_compensation`; agent không được gọi `issue_refund`.
- Giữ flow nghiệp vụ chung: claim không tồn tại → `failed`, thiếu `order_id` →
  `needs_info`, order không hợp lệ → `escalated`, quote 0 → `denied`.
- Dùng `compute_compensation` để tính tiền từ order/policy; không dùng ticket text
  hoặc output model để quyết định số tiền.
- Truyền full quote vào `durability.issue_refund`; host chịu trách nhiệm approval,
  cap theo order, idempotency và ghi ledger.
- Xử lý host refusal thành `denied`, amount conflict thành `failed`, thành công
  thành `resolved`.
- Truyền `crash_at`, ghi trace trước khi để `InjectedCrash` propagate và dùng
  `finish()` cho mọi nhánh return.
- Trace chỉ ghi action/status/reason an toàn, không ghi ticket, secret, header hoặc
  dữ liệu PII.

### Bằng chứng

- Thay đổi chính: [src/lab8/shapes/single.py](src/lab8/shapes/single.py).
- `python -m pytest -q -m student -o addopts= -k "not orchestrator and not workflow and not swarm"`
  → `62 passed`.
- `python -m pytest -q -m "not student"` → `76 passed` (cổng host xanh).
- Full student matrix còn fail ở orchestrator vì shape C vẫn là stub; không phải lỗi
  của Single Agent.
- Commit [`13f5978`](https://github.com/phungtr/K5-L3B-DAY08-Dota-FromWorkflowsToAgents/commit/13f5978)
  — `feat(single): implement run_claim and _run functions with decision logic`.
- Báo cáo cá nhân: [report/team/2A202604112_VuNamKhanh.md](report/team/2A202604112_VuNamKhanh.md).

## Thành viên 3 — 2A202604089_NguyenTuanAnh

### Phần việc phụ trách

- Shape C `orchestrator` + handoff (`src/lab8/shapes/orchestrator.py`)

### Đóng góp cụ thể

- Triển khai Shape C `orchestrator` (`src/lab8/shapes/orchestrator.py`): đọc claim qua `get_claim_view` → kiểm tra sự tồn tại của claim và `order_id` → dựng gói bàn giao có kiểu (typed handoff) bằng `lab8.handoff.make_handoff` với các công cụ chỉ đọc (`lookup_order`, `lookup_policy`, `lookup_refund_history`, `compute_compensation`) và `facts` rỗng → ghi nhận `handoff` đã làm sạch (sanitized) vào `trace`.
- Chạy `run_readonly_investigators` để tác tử điều tra chỉ đọc thu thập thông tin đơn hàng, chính sách, lịch sử hoàn tiền và tính báo giá đền bù (`quote`).
- Giữ chặt ranh giới quyền hạn (least privilege): tác tử điều tra không có quyền ghi; chỉ `orchestrator` mới gọi `durability.issue_refund` với **quote đầy đủ** để host tự trừ phần đã hoàn ở các claim trước (T13, T14) và kiểm tra phê duyệt.
- Truyền `crash_at` vào `durability.issue_refund`; khi gặp `InjectedCrash` thì ghi sự kiện `crash` và flush file trace trước khi `raise` tiếp.
- Thêm 23 test trong `src/tests/test_orchestrator_shape.py`: kiểm tra chống prompt injection từ ticket trên 5 seed, kiểm tra `make_handoff` từ chối công cụ ghi và lọc bỏ các trường ngoài allowlist (`parent_secret`, `raw_headers`, `ticket`), kiểm tra `sanitize` chặn `CANARY_SECRET_`, kiểm tra hoàn tiền một phần trên C-014 (T14), xung đột số tiền khi retry (`amount_conflict`), và quét trace của 14 claim public đảm bảo không lộ secret/header/ticket.

### Bằng chứng

- Thay đổi chính: [src/lab8/shapes/orchestrator.py](src/lab8/shapes/orchestrator.py), [src/tests/test_orchestrator_shape.py](src/tests/test_orchestrator_shape.py).
- `python -m pytest -q -m student -o addopts= -k orchestrator` → `62 passed`.
- `python -m pytest -q -m student -o addopts=` → `248 passed` (cả 4 shape đều xanh 100%).
- `python -m pytest -q` → `99 passed` (cổng host xanh, gồm 23 test mới).
- `python -m lab8.check_submission` → `[ OK ] shape orchestrator: format đúng, 12/12 claim public pass ở seed 11`.
- Báo cáo cá nhân: [report/team/2A202604089_NguyenTuanAnh.md](report/team/2A202604089_NguyenTuanAnh.md).

## Thành viên 4 — 2A202604075_PhamDuyThanh

### Phần việc phụ trách

- Ledger/crash + evaluation (`reports/crash.json`, `reports/benchmark.json`, ADR, research card)

### Đóng góp cụ thể

- Giai đoạn 1–2 (2026-11-17): bổ sung 8 test crash/retry/approval cho workflow, kiểm đủ ba crash point, mở lại kết nối, resume và retry nhiều lần; C-014 chỉ trả thêm 120.000 đồng.
- Thêm helper ablation `workflow_no_draft` và 72 test về kết quả nghiệp vụ, accounting, trace và khôi phục patch. Helper dùng cho research card bắt buộc, không đăng ký bonus.
- Tạo baseline 280 trial bằng benchmark chính thức, crash report 20/20 và ablation 140 trial; điền research card R4 cùng phần kết quả ban đầu của báo cáo nhóm.
- Sau merge `1f7d89d`: mở rộng tám kịch bản recovery/approval sang single thành 16 test; cập nhật này đã commit tại `db22c3b`.
- Sau merge orchestrator tại `4f2060d`: mở rộng thành 24 test dùng chung cho đủ workflow/single/orchestrator, tạo lại benchmark/crash, hoàn thiện ADR chọn workflow và cập nhật báo cáo theo đủ bốn shape. Phần hoàn thiện đã commit tại `9daa602` và merge vào `main` qua PR #3 (`2a8fa53`).
- Không sửa code shape của thành viên khác hoặc thước đo của lab. Chưa có kết quả hidden, không suy ra đạt ngưỡng hidden từ public.
- Code và tài liệu giai đoạn này được soạn với trợ lý Codex theo yêu cầu; thành viên cần tự đọc, chạy lại và bổ sung tự đánh giá trước khi nộp.

### Bằng chứng

- Checkpoint giai đoạn 1–2 tại `175d770`: 80 test mới pass, 156 test mặc định pass, 124 student test workflow/swarm pass.
- Checkpoint sau merge single: 16 test recovery/approval dùng chung workflow/single, tổng 164 test mặc định pass; 186 student test không gồm orchestrator pass. Benchmark mới: workflow/single 70/70, swarm 69/70; chỉ orchestrator còn stub.
- Checkpoint hiện tại: 195 test mặc định pass, 248 student test pass; 24 test recovery/approval riêng của Thành kiểm đủ ba shape. Workflow/single/orchestrator 70/70 trial mỗi shape; swarm 69/70 (lỗi tham chiếu C-002 seed 55).
- `src/reports/crash.json`: 20/20. Đây là crash suite host/swarm; 24 test riêng bổ sung bằng chứng cho ba shape nhóm viết.
- `src/reports/research_ablation.json`: mỗi variant 70/70 pass; cost/success 28.862857 → 27.857143 VND mô phỏng.
- Báo cáo cá nhân: [report/team/2A202604075_PhamDuyThanh.md](report/team/2A202604075_PhamDuyThanh.md).
- Commit kỹ thuật: `175d770`, `db22c3b` và `9daa602`; `4f2060d` là merge commit có đủ code nguồn được đánh giá. Phần hoàn thiện test/tài liệu (`9daa602`) đã merge vào `main` qua PR #3 (`2a8fa53`).
