---
id: project1-app-testing
type: project-analysis
scope: project:project1_app
status: source-observed
checked: 2026-10-09
anchor_commit: 58a0b01
---

# Test `project1_app`: cách chạy và kết quả gần nhất

## Lệnh trên máy Linux này

```sh
cd /home/abel/project/internproj/project1_app
PYTHONDONTWRITEBYTECODE=1 /usr/bin/python3 -m unittest discover -s tests/python -t .   # ~12s, SQLite tạm
cd app/flutter_app && flutter analyze && flutter test ../../tests/flutter              # flutter ở ~/development/flutter
python tests/e2e/run_e2e.py [--skip-build]                                             # cần điện thoại USB, adb ~/Android/Sdk
```

- **Không dùng `python3` trên PATH.** Nó trỏ vào venv của `project1_server`.
- `tests/run_tests.py` và `test.ps1` (Windows) ghi báo cáo vào `log/`. Báo cáo cũ đã chuyển sang [archive/test-runs-2026-09](archive/test-runs-2026-09/). Runner vẫn ghi vào `log/`. Nếu cần giữ lại thì chuyển sang archive theo [archive/README.md](archive/README.md).
- Tiêu chí từng luồng S/X/A/F/E nằm ở `log/TIEU_CHI_TEST.md` trong repo.

## Kết quả 10/10/2026: 70 test (thêm S11, S12), vẫn 25 error, 1 fail

Cùng nguyên nhân như bảng dưới. Sửa 4 dòng (bước HB-01 trong `agent_workspace/tasks/heartbeat-long-poll/TASK.md`: bỏ `LOGIN_STATES` ở setUp security, `MACHINE_DIR` → `version1.1/machine` ở relay và `machine_sim`, bỏ route `session/verify` khỏi `test_services_share_one_port`) thì còn 7 lỗi: `test_machine_bluetooth`, `test_cleanup_callbacks`, 5 test `test_user_login`. SEC-04 khi đó "expected failure" vì `AttributeError`, không phải vì lỗ hổng.

## Kết quả 09/10/2026: 66 test, 25 error, 1 fail

| Nhóm | Số | Nguyên nhân gốc | Cách sửa |
|---|---|---|---|
| `test_server_security` (17), `test_user_login` (5) | 22 | `setUp` gọi `user_login_process.LOGIN_STATES`, nhưng login đã thành 1 bước và bỏ trạng thái này (37a58c0) | Viết lại test theo login 1 bước. Các ca SEC-xx vẫn cần giữ ý nghĩa |
| `test_server.test_services_share_one_port` | 1 fail | Mong đợi `/app/user/session/verify` trả 400, server trả 404 vì route đã bỏ | Bỏ route này khỏi danh sách test |
| `test_server.test_cleanup_callbacks` | 1 | Patch `user_login_main.tick`, nhưng `tick` không còn tồn tại | Cập nhật test |
| `test_machine_relay` | 1 | Cần `machine/main.py` ở gốc repo, mà file đã xoá | Tuỳ quyết định về `machine/`, xem STATUS |
| `test_machine_bluetooth` | 1 | Không import được `machine.pairing...` | Như trên |

Cả 26 lỗi đều do **test và code lệch nhau sau refactor hoặc di chuyển**. Chưa thấy lỗi nghiệp vụ mới. DB thật không bị đụng (đã so sha256 trước và sau khi chạy).

## Lịch sử (bằng chứng ở archive)
- 28/09: 29 luồng đều ĐẠT, X1 đạt với 6 lỗ hổng đã biết (`expectedFailure`).
- 29/09: E2E E0–E12 ĐẠT trên Samsung SM_A256E. Claude round 3 lỗi hàng loạt, sau đó chạy lại đạt (`archive/test-reports-2026-09-29/`).
- Từ 05/10 không có lượt chạy nào được ghi lại, cho tới lượt 09/10 ở trên.
