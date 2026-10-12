# Bản thử kiểm chứng plan heartbeat → long-poll (10/10/2026)

Operator dựng thử trên worktree tách rời từ `91c8cdd` (đã xoá) để lấy số liệu cho
`project1_app/agent_workspace/tasks/heartbeat-long-poll/TASK.md`. **Chỉ operator dùng** để đối chiếu khi review;
không đưa cho coder/tester, để bản của agent được làm độc lập.

| File | Nội dung |
|---|---|
| `prototype-phase1.diff` | HB-01 + HB-02 + HB-03 + S13 (bản đồng hồ giả, 12 test) |
| `prototype-all-phases.diff` | Bản đầu cả 3 pha (S13 còn bản dùng `sleep`, đã thay) |
| `test_machine_online.reference.py` | S13 cuối: 12 test, ~9,4 s, 8/8 sabotage đỏ |

Số liệu chính (cũng có trong TASK.md, mục "Bằng chứng nghiên cứu"):
- `91c8cdd`: 70 test, failures=1, errors=25. Thêm HB-01 (4 dòng): errors=7, expected failures=6.
- Thêm Pha 1 + test: 82 test, errors=7 (có sẵn), expected failures=6, lặp ổn định.
- Pha 2: S8, S12, S13 OK; stress 200 vòng seed 42 PASS. Pha 3: chỉ còn 7 lỗi có sẵn.
- Test cũ đổi sang poll trên server cũ: failures=4, errors=10, unexpected successes=2 (lane check HB-03).
