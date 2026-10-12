---
id: heartbeat-refactor-2026-10-10
type: agent-report
scope: project:project1_app
date: 2026-10-10
source: operator (Opus) + 9 agent Sonnet 5.5 effort medium; chỉ kết quả cuối, không transcript
---

# Thực thi plan bỏ heartbeat: kết quả cuối

Plan: `project1_app/agent_workspace/tasks/heartbeat-long-poll/TASK.md` (bảng tiến độ có bằng chứng từng bước).
Nhánh `refactor/heartbeat-long-poll`, chưa push (máy chưa có credential GitHub).

## Commit

| Commit | Nội dung |
|---|---|
| `3994f09` | Plan đa agent |
| `cfc0821` | HB-01 sửa nền test: 70 test từ `failures=1, errors=25` còn `errors=7` có sẵn |
| `7765b66` | Pha 1: online suy từ poll/result/lệnh đã lấy; S13 13 test |
| `a662e34` | Pha 2–3: bỏ thread heartbeat phía máy, bỏ route `/machine/heartbeat/send` |
| `8f62ba7` | Tài liệu hiện trạng (20 file) + tiến độ plan |

## Agent và kết quả

| Agent | Bước | Kết quả |
|---|---|---|
| T1 tester | HB-01, HB-03 | Đạt; lane check khớp đúng số đo dự kiến |
| C1 coder | HB-02, sửa N1 | Đạt; `machine_transport.py` trùng từng byte bản thử của operator |
| T2 tester | HB-05, HB-10b | 13 test S13, 9/9 sabotage đỏ đúng test. ⚠ Bản đầu chép file tham chiếu của operator để lộ trong scratchpad |
| R1 reviewer | HB-06 | Không Chặn/Nên sửa; 3 gợi ý (1 đã làm: assert `last_seen` None) |
| K1 cybersecurity | HB-06 | Không Chặn; N1 Nên sửa (rò `DANG_LAM` khi kết quả tới trước take) đã sửa; F1, F2 có sẵn |
| C2 coder | HB-07 | Đạt; stress 300 vòng PASS |
| C3 coder | HB-10a | Đạt; làm 2 lần do operator chép nhầm lane |
| D1 coder tài liệu | HB-08 | 20 file; sửa 1 Nên sửa + 2 gợi ý của R2 |
| R2 reviewer | HB-09 | Không Chặn; HTML cân bằng thẻ |

Cổng cuối G2: 83 test `errors=7, expected failures=6`; stress 300 vòng seed 42 PASS; grep code chỉ còn 2 assertion 404.

Đo tải lúc rảnh (T3, HB-11; một máy, localhost, 120 s, lặp 3 lần giống nhau): `91c8cdd` 0,325 req/s (poll 0,125 + heartbeat 0,200), `8f62ba7` 0,125 req/s, giảm 61,5%. Commit ghi số đo: xem `git log` nhánh.

## Bài học vận hành

- Bash của subagent trong cây chính đôi khi bị auto-mode chặn ("Modify Shared Resources"); Edit vẫn chạy. Operator tự chạy kiểm.
- Không để file tham chiếu của operator ở thư mục mà prompt của agent trỏ tới.
- Khi chép file từ nhiều lane, chỉ định rõ lane nguồn cho từng file; `cp a || cp b` sai khi cả hai lane có file.
- Lưu patch lane trước khi gỡ worktree.

## Còn mở

- Push nhánh (cần credential của user).
- SEC-08: đưa `online` vào `/app/user/machine/list`, đóng route status.
- F1: `/machine/result/send` với id không băm được cắt kết nối. F2: kết quả tới trước take thì máy vẫn chạy lệnh cũ.
- 7 lỗi test có sẵn (login 1 bước, bluetooth, `test_cleanup_callbacks`); SEC-04 xfail vì AttributeError.
