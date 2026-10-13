---
id: project1-app-status
type: project-status
scope: project:project1_app
status: source-observed
checked: 2026-10-13
anchor_commit: 0ec3a56
source: /home/abel/project/internproj/project1_app
---

# Tiến trình `project1_app`

Đọc file này khi user hỏi "tới đâu rồi / đã làm X chưa / làm gì tiếp". **Không quét code để trả lời các câu đó.** Sau mỗi việc làm thay đổi trạng thái, cập nhật bảng và thêm một dòng nhật ký.

## Tình trạng hiện tại · 12/10/2026

Repo ở HEAD `0ec3a56`, working tree sạch trước review 12/10/2026. Skeleton v2 đã code tài khoản, claim BLE, relay, menu/kho qua máy; `documentation/plan/mvp-fast-path.md` là trạng thái thực thi hiện hành. Chưa nghiệm thu Android/Pi, HTTPS domain/cert và Gmail thật (I-01). Review đọc code 12/10/2026 ghi ba lệch hợp đồng ở tài khoản, chưa sửa: server chấp nhận mật khẩu 8–14 ký tự (D9), không thu hồi phiên nếu refresh token đã dùng quay lại trong 30 giây (D10), một số đường DB tài khoản không retry deadlock/lock timeout như v2. Bảng 09/10 và phần lớn nhật ký bên dưới là lịch sử code cũ.

## Kiểm độ tươi (~50 token)

```sh
git -C /home/abel/project/internproj/project1_app log --oneline -1    # 0ec3a56 → note khớp commit
git -C /home/abel/project/internproj/project1_app status --short
```

Lúc kiểm (09/10/2026), working tree có ba nhóm thay đổi (lịch sử):
- 52 file `machine/` bị xoá. Mã máy đã chuyển sang `version1.1/machine/`.
- 120 file báo cáo bị xoá vì đã di cư vào vault.
- 6 file trong `agent_workspace/tasks/middleware/` bị sửa (trỏ link sang vault).
- `agent_workspace/TEAM.md` và `agent_workspace/README.md` thêm quy tắc ghi báo cáo vào vault.

Nếu HEAD khác `0ec3a56`, đọc diff từ commit neo để cập nhật note.

## Một câu

Tính năng cơ bản (tài khoản, đăng ký/chia sẻ máy, menu, kho) đã có ở cả app, server và máy, chạy qua HTTP không mã hoá. **Middleware và bảo vệ gói tin mới ở mức thiết kế/kế hoạch, chưa có dòng code nào.** Đăng nhập app đã khớp server ở commit `bb82f53`; test Python cũ vẫn cần sửa.

## Bảng trạng thái — 09/10/2026

| Hạng mục | TT | Chi tiết, bằng chứng |
|---|---|---|
| Server 8 module | ✓ | Code chạy được, xem [ARCHITECTURE.md](ARCHITECTURE.md) |
| Đăng nhập app ↔ server | ✓ code | 10/10: app chuyển sang 1 bước (`bb82f53` trên `refactor/heartbeat-long-poll`), bỏ `userSessionVerify`; Flutter 32/32 pass. Chưa chạy trên điện thoại. Test Python còn gọi `/session/verify` (`test_user_login.py`, `test_server_security.py`) |
| Test Python | ✗ | 09/10: 66 test, 25 error, 1 fail. Nguyên nhân ở [TESTING.md](TESTING.md) |
| Flutter analyze/test | ? | Chưa chạy từ 29/09 |
| E2E điện thoại | ✓ cũ | Đạt E0–E12 ngày 29/09 trên Samsung SM_A256E, trước khi đổi login và chuyển `machine/` |
| Client máy | ⚠ | `machine/` bị xoá khỏi repo (chưa commit). Bản dùng được ở `version1.1/machine/` (untracked). Test relay và bluetooth lỗi vì còn trỏ `machine/` |
| Ảnh món (phương án B) + tool stress CRC menu | → | Nhánh `feature/menu-image-cache` (từ `refactor/server-modules` @ `07a54d0`), **chưa commit**: route `/app/machine/image/get`, cache ảnh app theo `image_hash`, tool `tests/tools/menu_crc_stress` (S11, S12 pass). Phía máy chưa làm. Xem [session](../../../sessions/2026-10-09-menu-crc-image-cache.md) |
| Trạng thái online máy (heartbeat ↔ long-poll) | ✓ | 10/10: xong cả 3 pha trên nhánh `refactor/heartbeat-long-poll` (`3994f09`..`8f62ba7`, **chưa push**: máy chưa có credential GitHub). Online = poll/result < 15s hoặc lệnh đã lấy < 20s; máy không còn thread heartbeat; route heartbeat bỏ. 83 test: 7 lỗi có sẵn, 6 xfail. Còn mở: SEC-08 (online vào list), F1/F2 (xem TASK.md), merge vào nhánh chính. [Báo cáo](archive/heartbeat-refactor-2026-10-10/README.md) |
| Orders | ○ | Chỉ có dữ liệu demo, chưa có API |
| Middleware G0–G7 | ○ | Kế hoạch có 46 task, chưa triển khai task nào, xem [MIDDLEWARE.md](MIDDLEWARE.md) |
| Bảo vệ gói tin (crypto) | ⏸ | 11 quyết định đã chốt 07/10. Chưa có code, thư viện hay vector. Các cổng D6-G1…G7 chưa chạy |
| Lỗ hổng SEC-xx | ✗ | Còn mở SEC-01/02/09/10/12/13/16/19..., xem [SECURITY.md](SECURITY.md) |

## Việc tiếp theo hợp lý (agent đề xuất, user chưa chốt)

1. ~~Sửa login app sang 1 bước~~ (xong `bb82f53`). Còn: sửa test Python còn dùng `LOGIN_STATES`/`/session/verify`.
2. User quyết `machine/`: commit việc xoá và sửa test trỏ sang `version1.1/machine`, hoặc khôi phục `machine/`.
3. Làm G0 của middleware (B01–B04): snapshot hợp đồng, sửa fixture, test tái hiện, đo môi trường. Bước 1 trùng với một phần B02.

## Nhật ký (mới nhất ở trên, mỗi dòng 1 sự kiện)
- 2026-10-13: Thêm hướng dẫn Linux server `server/SETUP_LINUX.md` và link từ README; cập nhật chuẩn sơ đồ trong second brain, template, asset `documentation/architect/assets/sequence-layout.js` và 17 trang có sơ đồ trình tự (không gãy mã/từ, nội bộ chỉ nút số lớn 1,4 lần, nhiều việc thành list). Browser render 17/17, JS syntax và `git diff --check` đạt. Chưa chạy server trên Linux thật; thay đổi chưa commit tại HEAD `0ec3a56`. `documentation/serve_architect.py` đã có sửa đổi từ trước phiên này, không thuộc công việc trên.
- 2026-10-12: S-01/A-01 coder Sol low hoàn thành code tài khoản cloud TLS+HPKE, SMTP/OTP/phiên và Flutter/Kotlin HPKE+Keystore; review nhỏ vòng2 PASS (server32 tests MySQL cô lập; app20 Flutter tests, APK build). Review lớn module một lần phát hiện quota login race và ô xác nhận mật khẩu còn giá trị; user chốt ghi nhận chưa sửa, giữ mapping INTERNAL_ERROR hiện tại. BT HPKE PSK được user chọn; profile review giữ B-03 BLOCKED, official vector PSK + JVM↔Python primitive interop đạt, chưa Android/Pi thật. User chọn kết nối thiết bị để sau, chưa có HTTPS runtime. Nguồn `documentation/plan/assignments.md`, `documentation/plan/account-module-review-2026-10-12.md`, `documentation/research/ble-noise-feasibility.md`.
- 2026-10-12: User yêu cầu giao agent Sol low sau khi chuẩn bị mail. C-01 đối chiếu quyết định và code; C-02/C-03 research song song, thêm research bootstrap/mã hóa (nguồn/case study ở `archive/contracts-skeleton-research-2026-10-12.md`); C-04 tạo `documentation/architect/features/contracts.html`, Operator sửa lệch quyết định cũ; user chốt đăng ký serial và bản kiểm mã niêm phong trước khi bán, khóa máy tự sinh, giữ TLS + phong bì mã hóa bên trong ngay skeleton. Giao C-05 cho Planner Sol low; chưa code sản phẩm.
- 2026-10-12: User chốt mọi database dùng MySQL, bỏ SQLite. `version1.1/machine/menu_sync/machine_menu_store.py` chuyển sang `database/core/connection.py` (placeholder `%s`, DECIMAL→số qua `mysql_number`); `git rm` file rỗng `machine/database/database.db`. Kiểm bằng kết nối giả (nhan_menu, cap_nhat_menu OK); chưa chạy MySQL thật vì không có `BEVERAGE_DB_PASSWORD`. Chưa commit.
- 2026-10-11: Research database server hãng (3 agent), HTML `documentation/research/server-database.html` (ERD, untracked); chốt nhật ký bản sao trên server, lịch sử quyền, offline từ chối lệnh. Doc còn lệch 3 fact đã chốt, chưa sửa (`archive/server-database-research-2026-10-11.md`).
- 2026-10-11: Ghi 25 endpoint vào `server/configuration/routing.py` (17 app gọi, 8 máy gọi, relay lệnh chung `/app/machine/command/send`) và 8 route máy vào `version1.1/machine/config/routing.py`; chỉ hằng, chưa handler, chưa commit. Research tài khoản/đăng nhập/danh tính (3 agent Sonnet med): thêm `documentation/research/account-login-identity.html` (untracked); quyết định ở [fact](../../../facts/project1-app-auth-identity-confirmed.md). Lỗi chưa sửa: BT máy tự nhận mọi điện thoại và trả `product_key`; màn đăng nhập còn "Tên đăng nhập". Route refresh/reauth/list/revoke/password chưa thêm. Xem [session](../../../sessions/2026-10-11-app-endpoints-auth-research.md).
- 2026-10-12: Research giao thức đồng bộ menu/kho/máy (6 agent Sonnet med, 2 vòng). User chốt: menu đồng bộ bằng CRC, khác thì máy gửi cả gói; gói mang công thức; bỏ tuỳ chọn size/đá/đường; giá số nguyên đồng; máy dùng MySQL; trần gói 256 KB; server được giữ gói trong RAM để phát cho nhiều app; tạm chỉ Android (giữ SPP). Thêm `documentation/research/menu-inventory-machine-sync.html` (untracked). Lỗi tìm thấy, chưa sửa: `menu_sync` đọc SQLite rỗng thay vì MySQL, `reply` không bắt ValueError/không gửi lại, BT giới hạn 4096 byte. Kho và trạng thái máy còn là đề xuất, 9 câu hỏi mở trong doc.
- 2026-10-11: Viết bản nháp kiến trúc `documentation/architect/system-architecture.html` (untracked, chờ user duyệt): 6 nguyên tắc, luồng tắt món, 6 khối (hợp đồng chung, máy, server, app, đồng bộ offline, OTA), repo đề xuất (thêm `contracts/`, `documentation/adr/`, giữ tên thư mục cũ), quy trình làm việc. Chưa chốt.
- 2026-10-11: HEAD `c34d55e`. Research kiến trúc dễ bảo trì (2 vòng, 6 agent Sonnet med): thêm `documentation/research/maintainable-large-systems.html` (untracked). Đề xuất của agent (chưa chốt): ba bên modular monolith, máy là nguồn sự thật, shadow nhẹ, envelope có phiên bản. Phát hiện ở máy (chưa sửa): không lưu `lenh_id` đã xử lý, `product_key` trong body, không retry. Xem [archive](archive/maintainable-architecture-research-2026-10-11.md).
- 2026-10-11: HEAD `7bd7f52`: hai commit `28febfb`, `7bd7f52` xoá code app/server và docs cũ để thiết kế lại; repo còn `documentation/`, `version1.1/`. Bắt đầu tầng 1 hệ IoT mới (chỉ bàn qua chat, chưa file, chưa code), 5 agent research. Xem [session](../../../sessions/2026-10-11-iot-system-design-tier1.md).
- 2026-10-10: Sửa lỗi 404 `/app/user/session/verify` khi đăng nhập: app đổi sang login 1 bước khớp server (`bb82f53`, 3 file: `user_login_request.dart`, `config/routing.dart`, `tests/flutter/auth_page_test.dart`). `flutter analyze` sạch, Flutter 32/32 pass. Push thất bại: không có credential GitHub (HTTPS không có helper, SSH key bị từ chối).
- 2026-10-10: Thực thi plan heartbeat (user chốt U1 sửa máy, U2 xoá route ngay, U3 commit + push, U4 bỏ bộ đếm poll) bằng 9 agent Sonnet 5.5 med. Commit `3994f09` plan, `cfc0821` sửa nền test (26 → 7 lỗi có sẵn), `7765b66` Pha 1, `a662e34` Pha 2–3, `8f62ba7` tài liệu. Review bảo mật tìm N1 (rò `DANG_LAM` khi kết quả tới trước take), đã sửa. Push thất bại: SSH key chưa gắn GitHub, không có credential HTTPS.
- 2026-10-10: Commit `feature/menu-image-cache`: `0ff86ce` (giao thức ảnh món + tool stress S11/S12), `57d927e` (docs heartbeat vs long-poll), `91c8cdd` (xoá stub `docs/CRYPTOGRAPHY_MIDDLEWARE_DESIGN.html`; `agent_workspace/tasks/middleware/crypto-primer.html:482` còn link tới file này). Lập plan bỏ heartbeat, dựng thử 3 pha trên worktree tạm (đã xoá): baseline 70 test 1 fail/25 error; sửa 4 dòng test còn 7 lỗi có sẵn; sau Pha 1 82 test, 8/8 sabotage bị bắt; Pha 2 stress 200 vòng PASS. Test relay S8 và `tests/relay/machine_sim.py` trỏ `machine/` (chỉ còn `__pycache__`), sửa ở HB-01. Anchor chưa đổi (chưa rà `07a54d0`).
- 2026-10-10: Đánh giá cơ chế online máy bằng 3 agent (Opus med) cộng case study; kết luận đề xuất bỏ heartbeat theo 3 pha (server → máy → dọn route), chưa code. Thêm `docs/heartbeat-vs-long-poll.html` và `.md` (untracked). Phát hiện working tree xoá `docs/CRYPTOGRAPHY_MIDDLEWARE_DESIGN.html`, chưa rõ lý do. HEAD vẫn `07a54d0` trên `feature/menu-image-cache`. Xem [session](../../../sessions/2026-10-10-heartbeat-long-poll.md).
- 2026-10-09: Tạo nhánh `feature/menu-image-cache`: giao thức ảnh món (B: `image_hash` trong gói menu + lệnh `nhan_anh` + cache app theo hash) ở server/app, và tool stress CRC menu (luồng thật vs oracle độc lập, khôi phục menu gốc, 500 vòng PASS). Chưa commit; phía máy chưa làm. Lưu ý: HEAD repo hiện là `07a54d0` trên `refactor/server-modules`, khác `anchor_commit` 58a0b01 của note này.
- 2026-10-09: Đổi toàn bộ path routing sang tiếng Anh có cấu trúc (22 route server + `userSessionVerify` ở app), sửa server/app/máy/test/tài liệu cùng lúc; chưa commit. Python giữ nguyên 25 error + 1 fail như trước, Flutter 29/29 pass, smoke test 22 route không 404. Chuẩn: `facts/routing-naming-standard.md`. Sau đó sửa nốt nhãn route rút gọn trong sơ đồ (`docs/dong-bo-du-lieu.html`, `DASHBOARD_SYNC_FLOW.html`) và `agent_workspace/CODE_STYLE.md`.
- 2026-10-09: Phân tích toàn repo, lập note vault. Di cư 120 file báo cáo agent sang `archive/`, các link trong repo trỏ sang vault. Chạy test Python: 25 error, 1 fail. Phát hiện lệch login.
- 2026-10-09: User hỏi middleware đã gắn vào app chưa. Kết luận: chưa, chỉ có tài liệu.
- 2026-10-07: Chốt 11 quyết định bảo vệ gói tin (D1-B…D8-C) và cách rollout D6-A. Nghiên cứu R5 Codex↔Claude đồng thuận 10/10 (`archive/security-research-2026-10-07`).
- 2026-10-06: Plan middleware gồm 8 phase, 32 phase con, 46 task. Chốt quy trình team AI (AGENTS.md).
- 2026-10-05: Login server đổi sang 1 bước (37a58c0). App và test chưa cập nhật. Dựng hệ multiagent (50c7817).
- 2026-09-29: Refactor Flutter theo feature, rework `dashboard_sync`, chạy E2E đạt. Claude test qua đêm (`archive/test-reports-2026-09-29`).
- 2026-09-28: Sửa lỗi B1–B5, chuyển máy sang long-poll (O1), thêm bộ kịch bản tấn công X1, chuẩn hoá module (`archive/test-runs-2026-09/BUG_LOG.md`).
- 2026-09-24…26: Thêm Bluetooth pair/share, xoá/đổi tên máy, nạp kho, đồng bộ menu.
