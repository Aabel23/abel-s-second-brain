---
id: project1-app-architecture
type: project-analysis
scope: project:project1_app
status: source-observed
checked: 2026-10-12
anchor_commit: 0ec3a56
source: /home/abel/project/internproj/project1_app
---

# Kiến trúc `project1_app`: phân tích toàn repo

Đọc file này khi cần ngữ cảnh code: sửa module, tìm file, hiểu hợp đồng gói tin. Tiến trình xem [STATUS.md](STATUS.md).
Mục "Skeleton v2" dưới đây đối chiếu HEAD `0ec3a56` ngày 12/10/2026. Các mục đánh số bên dưới là ảnh chụp **code cũ** tại commit `16dab0e`, chỉ dùng để hiểu lịch sử, không áp vào skeleton mới. Khi HEAD/working tree đổi, kiểm diff phần liên quan.

## Skeleton v2 · hiện hành tại `0ec3a56`

| Vai trò | Code và hợp đồng chính |
|---|---|
| Server tài khoản và cloud | `server/src/account.py`, `cloud_protection.py`, `http.py`, `server/main.py`; schema `server/migrations/001_accounts.sql` và `002_devices_sessions.sql` |
| Server máy và relay | `server/src/devices.py`, `relay.py`; máy được đăng ký qua `factory_register.py`, claim lưu trong MySQL, relay chờ trong RAM |
| App Flutter | `flutter_app/lib/shared/cloud/`, `feature/user_auth/`, `feature/machine_register/`, `feature/machines/`, `feature/machine_menu/`, `feature/machine_ingredient/` |
| Android native | `CloudCrypto.kt`, `SessionStore.kt`, `ClaimProtocol.kt`, `ClaimBle.kt`, `ClaimChannel.kt` dưới `flutter_app/android/app/src/main/kotlin/com/example/simple_app/` |
| Máy | `version1.1/machine/config/`, `pairing/`, `server_connection/`, `main.py`; dùng nghiệp vụ menu/kho cũ qua dispatcher mới |
| Hợp đồng | `contracts/messages/v2-decisions.md` ưu tiên khi mâu thuẫn với các hợp đồng cũ; lộ trình và trạng thái ở `documentation/plan/mvp-fast-path.md` |

Chưa nghiệm thu trên Android/Pi thật, HTTPS domain/cert và Gmail thật. Review 12/10/2026 phát hiện server vẫn nhận mật khẩu 8 ký tự (D9 yêu cầu 15), cho phép refresh token đã dùng trong 30 giây mà không thu hồi phiên (D10 yêu cầu thu hồi), và phần lớn đường tài khoản gọi `transaction()` trực tiếp nên deadlock/lock timeout trả 503 ngay thay vì retry tối đa 3 lần theo v2. Báo user, chưa sửa code.

## Ảnh chụp trung gian · tài khoản và cloud (12/10/2026, trước v2)

| Vai trò | Code hiện hành | Hợp đồng |
|---|---|---|
| Server HTTPS/HPKE/dispatch | `server/src/http.py`, `cloud_protection.py`, `crypto_keys.py`, `types.py`; `server/main.py`, `server/configuration/skeleton.py` | `contracts/messages/cloud-protection.md`, `contracts/COMPATIBILITY.md` |
| Tài khoản/SMTP/MySQL | `server/src/account.py`, `smtp_mail.py`, `database.py`; `server/migrations/001_accounts.sql` | `contracts/messages/auth.md`, phiếu `documentation/plan/tasks/S-01-server-account.md` |
| App cloud/account | `flutter_app/lib/shared/cloud/`, `feature/user_auth/data/`, UI auth/OTP, `app_navigation.dart`, `main.dart` | Phiếu `documentation/plan/tasks/A-01-app-account.md` |
| Android native crypto/storage | `flutter_app/android/app/src/main/kotlin/com/example/simple_app/{CloudCrypto,SessionStore,MainActivity}.kt` | HPKE BouncyCastle, AndroidKeyStore; asset public pin hiện `{}` cố ý fail startup |
| Test/fixture | `tests/server/`, `tests/fixtures/crypto/`, `flutter_app/test/user_auth/`, AndroidTest `CloudCryptoTest.kt` | 32 server tests/MySQL cô lập, 20 Flutter tests; AndroidTest mới build, chưa chạy thiết bị |

Account/cloud đã qua review nhỏ. Review lớn có 2 finding user chọn ghi nhận chưa sửa (quota login cạnh tranh và ô xác nhận mật khẩu khi quay lại OTP); xem `documentation/plan/account-module-review-2026-10-12.md`. Claim/relay/Bluetooth chưa có code mới; profile Bluetooth HPKE PSK ở `contracts/messages/bt-claim.md` còn BLOCKED. Không diễn giải bộ code cũ phía dưới là đã tích hợp với skeleton.

## 1. Hệ thống là gì

FlexMix là máy pha đồ uống. Repo này có 3 phần:

| Phần | Ở đâu | Ngôn ngữ | Quy mô |
|---|---|---|---|
| App quản lý cho chủ quán/nhân viên | `app/flutter_app/` | Flutter (Android) + Kotlin cho Bluetooth | ~3.9k dòng Dart |
| Server trung gian app ↔ máy | `server/` | Python stdlib (`http.server`, `sqlite3`) | ~2.2k dòng |
| Client chạy trên máy (Raspberry Pi) | **`version1.1/machine/`** (đã tracked trong repo này); `machine/` ở gốc repo đã xoá | Python | nhỏ |

Máy nằm sau router của quán nên server không gọi xuống máy được. Máy **long-poll** server để lấy lệnh. App gọi server qua HTTP JSON **không mã hoá**.

```text
App ──HTTP JSON {token,...}──▶ server :8000 ──hộp thư RAM──▶ máy long-poll /machine/command/poll
                                   ▲                              │
                                   └──── /machine/result/send ◀───┘ (send() trả kết quả trên request app ban đầu)
App ◀─Bluetooth (MethodChannel Kotlin)─▶ máy: pair lấy product key; app↔app: chia sẻ mã mời
```

## 2. Server

### Khởi động và dispatch
- `python -m server.main [--host --port]`, mặc định `0.0.0.0:8000` (`server/config/config.py`).
- `server/main.py` có tuple `MODULES` gồm 8 module import trực tiếp. **Quy ước đã khoá:** không có registry động.
- `server/lib/http/http_server.py`:
  - `ModuleServer(ThreadingHTTPServer)` kiểm trùng route khi lắp, gọi `setup()`, rồi gọi `tick()` khoảng mỗi 0.5s.
  - `do_POST` lần lượt hỏi từng `module.handle(request)`. Không module nào nhận thì trả 404 `{"valid":false,"message":"Không có endpoint này"}`.
  - `do_GET` hỏi `handle_get`, không có thì `send_error(404)`.
  - **Chưa có lớp middleware.** Nếu thêm, điểm chèn tự nhiên là `make_handler()`.
- `server/lib/http/http_json.py: handle_routes(request, routes, max_body, error, limited)` là khung chung của mọi module: tra route → rate limit theo IP (nếu `limited`) → `read_json` (giới hạn body, chặn JSON lồng sâu/surrogate) → gọi flow → `sqlite3.Error` thì trả 503 → `send_json`.

### Hai dạng response (hợp đồng, đổi phải đổi cả app)
- Nhóm tài khoản/máy: `{valid, message, ...}`. Status suy từ `valid` (200 / 400 / 429 khi có `retry_after`), qua `with_valid_status`.
- Nhóm Menu/Kho/cổng máy: lỗi dạng `{loi: ...}`, flow trả `(body, status)`. 503 khi máy offline, 502 khi máy lỗi hoặc timeout.

### Bảng route (`server/config/routing.py`)
Path tiếng Anh theo `/<app|machine>/<đối_tượng>/<mục_tiêu>/<hành_động>` từ 09/10/2026, chuẩn ở [facts/routing-naming-standard.md](../../../facts/routing-naming-standard.md).

| Route | Module (`server/service/…`) | Xác thực | Ghi chú |
|---|---|---|---|
| `/app/user/account/register`, `/app/user/otp/send`, `/app/user/otp/verify` | `user_register/` | rate limit IP | Trạng thái đăng ký và OTP nằm trong RAM. OTP 5 lần/giờ/email, chờ 60s giữa các lần, tối đa 5 lần nhập sai |
| `/app/user/session/login` | `user_login/` | rate limit IP | **1 bước** từ commit 37a58c0 (05/10): trả `token` ngay. Băm giả khi không có user để không lộ qua thời gian phản hồi |
| `/app/user/session/logout` | `user_login/` | token | Xoá phiên, không rate limit |
| `/app/user/machine/register` | `machine_register/` | token | Người quét tem đầu tiên thành chủ. `BEGIN IMMEDIATE`. Gửi lại cùng key thì trả cùng ID |
| `/app/machine/share/create`, `/app/machine/share/accept`, `/app/machine/staff/list`, `/app/machine/staff/revoke` | `machine_share/` | token, chủ | Mã mời 24 byte, TTL 5 phút, dùng 1 lần, lưu hash |
| `/app/user/machine/list`, `/app/machine/name/update`, `/app/user/machine/remove` | `dashboard_sync/machinelist_sync/` | token | Gỡ máy: chủ thì xoá máy, nhân viên thì bỏ quyền của mình |
| `GET /app/machine/status/get?machine_id=` | `machinelist_sync` | **không** | SEC-08. Quyết định D8-C: sẽ bỏ, gộp `online`/`last_seen` vào `/app/user/machine/list` |
| `/app/machine/menu/get`, `/app/machine/menu/update` | `dashboard_sync/menu_sync/` | token, owner/manager | Lệnh máy `nhan_menu`, `cap_nhat_menu`. Version CRC32, tối đa 200 thay đổi, chỉ sửa `available` và `price` |
| `/app/machine/image/get` | `dashboard_sync/menu_sync/machine_menu_image.py` | token, owner/manager | App xin tối đa 20 ảnh/lượt theo `drink_id` và `image_hash`; server chuyển lệnh `nhan_anh` cho máy |
| `/app/machine/ingredient/get`, `/app/machine/ingredient/refill` | `dashboard_sync/ingredient_sync/` | token, owner/manager | Lệnh máy `nhan_kho`, `nap_kho`. `target` = id hoặc `"all"`, `value` = gram hoặc `"full"` |
| `/machine/command/poll`, `/machine/result/send` | `machine_link/` | `product_key` của máy | Poll và result ghi "đã thấy" (`mark_seen`) vào RAM; poll giữ tối đa 8s. `/machine/heartbeat/send` đã bỏ (404) từ `a662e34` |

### Tài nguyên dùng chung (`server/lib/`, không import `server.service`)
- `security/user_session.py`: token `secrets.token_urlsafe(32)`. DB chỉ lưu SHA-256. TTL 30 ngày. Token gửi **trong body JSON** (`token`), không dùng header.
- `security/user_password.py`: `pbkdf2_sha256$200000$salt$digest`.
- `machine/machine_access.py: check_access(data, roles)` đi theo chuỗi token → user → có quản lý máy → vai trò.
- `machine/machine_transport.py`: hộp thư lệnh, kết quả chờ, trạng thái online (`LAN_THAY_CUOI`, `DANG_LAM`), **tất cả trong RAM**, mất khi restart.
  - `send()` chờ tối đa 20s.
  - Online = poll/result cuối < 15s (`MACHINE_SEEN_TIMEOUT_SECONDS`) **hoặc** có lệnh máy đã lấy mà `send()` còn chờ, < 20s. Lệnh xếp hàng chưa lấy không tính. Vòng máy treo → offline sau khi `send()` hết giờ, lệnh sau 503 ngay.
  - Timeout mà máy chưa lấy lệnh thì huỷ lệnh. Máy đã lấy rồi thì báo "hãy tải lại".
  - Kết quả về muộn bị bỏ, nhưng máy vẫn nhận `da_nhan:true`.
- `http/http_rate_limit.py`: 30 request/phút/IP, bảng tối đa 1000 IP. Bảng đầy thì chặn IP mới (SEC-03).
- `validation/`: kiểm định dạng `request_id`/`machine_id`, dọn trạng thái hết hạn.

### Database (SQLite `server/database/database.db`, file thật, git-ignore `*.db`)
- `user/schema.sql`: bảng `users` (username UNIQUE, role mặc định `staff`) và `app_sessions` (`token_hash`, `expires_at`).
- `machine/schema.sql`: bảng `machines` (`machine_id`, `name`, `product_key_hash` UNIQUE, ...) và `machine_managers` (`role` owner|manager, tối đa 1 owner mỗi máy).
- `machine/machine_share_schema.sql`: bảng `machine_invites`.
- `init_db()` tạo bảng nếu chưa có và tự thêm cột `product_key_hash` cho DB cũ. `connection.py` chỉ bật `foreign_keys`.

### Trạng thái chỉ có trong RAM (mất khi restart, chỉ chạy được 1 tiến trình)
Phiên đăng ký và OTP (`user_register`), hộp thư lệnh, trạng thái online, bảng rate limit. Phiên đăng nhập thì nằm trong DB nên còn sau restart.

## 3. App Flutter (`app/flutter_app/lib/`)
- `main.dart` → `app/app_navigation.dart` lắp ghép màn hình. Auth và dashboard không import nhau, giao tiếp qua callback.
- `config/app_config.dart`:
  - URL mặc định cứng là `http://192.168.1.158:8000`, ghi đè bằng `--dart-define=SERVER_URL`.
  - Có ghi chú IP Tailscale `100.127.250.88`.
- `config/routing.dart`: route đăng nhập một bước hiện khớp server; app đã bỏ `userSessionVerify` từ commit `bb82f53`.
- `core/server_client.dart`:
  - Có `account()` cho dạng `{valid}` và `machineData()` cho dạng `{status}`.
  - Gặp `login_required` thì gọi callback để quay về màn hình đăng nhập.
  - Timeout kết nối 5s.
- `core/http_json.dart: postJson` dùng cho login/OTP. `core/machine_packet.dart` định nghĩa gói QR/Bluetooth.
- Các feature trong `feature/`:
  - `user_auth` (đăng nhập, đăng ký, OTP).
  - `machine_register` (QR hoặc Bluetooth).
  - `machine_share` (QR hoặc Bluetooth app↔app).
  - `machine_list`, `machine_menu`, `machine_ingredient`.
  - `orders`: **chỉ dữ liệu demo** (`demo_orders.dart`), chưa có API.
  - `dashboard`: lắp ghép các tab overview / machines / menu / kho / orders.
- `feature/machine_menu/machine_menu_image.dart`: cache ảnh theo máy và CRC32 byte ảnh, tải phần thiếu qua `/app/machine/image/get`, xác minh CRC32 trước khi ghi file.
- Bluetooth: `MethodChannel('flexmix/bluetooth_pairing')` gọi `android/.../BluetoothPairing.kt`.
- Thư viện: `mobile_scanner`, `qr_flutter`. **Chưa có thư viện crypto.** Theo quyết định C3-A, crypto sẽ dùng Bouncy Castle qua Kotlin.

## 4. Client máy (`version1.1/machine/`)
- `main.py`: vòng lặp hỏi lệnh `{id, instruction, data}`, tra bảng `COMMANDS` (gồm `MENU_COMMANDS` và `INGREDIENT_COMMANDS`), chạy lệnh rồi gửi kết quả.
- Các module:
  - `menu_sync/` (request/sync/validate/store/pack).
  - `ingredient_sync/`.
  - `pairing/` (BlueZ Bluetooth pair/serve).
  - `server_connection/` (request: poll, gửi kết quả). Không còn thread heartbeat từ `a662e34`.
  - `config/` (env: tên máy, product key, URL server từ `machine.env`).
- Tầng DB MySQL của máy nằm ở mã máy pha gốc, không thuộc repo này (SEC-17).

## 5. Test và công cụ (`tests/`)
Chi tiết lệnh và kết quả: [TESTING.md](TESTING.md).
- `python/`: unittest dùng SQLite tạm, không chạm DB thật.
- `flutter/`: widget test.
- `e2e/run_e2e.py`: điều khiển điện thoại qua ADB.
- `relay/machine_sim.py`, `bluetooth_pair/`, `server_module/`: công cụ giả máy và giả app thứ hai.
- `run_tests.py`, `test.ps1` (PowerShell, viết cho Windows): chạy theo luồng và ghi `log/FLOWS.md`, `log/HISTORY.md` và báo cáo theo thời điểm vào `log/`.
- `tools/check_middleware_d6_docs.py`: chỉ kiểm tài liệu HTML.
- `sandbox/` chỉ còn `__pycache__`, không có mã.

## 6. Tài liệu trong repo (mở khi cần, không đọc mặc định)
| File | Nội dung |
|---|---|
| `AGENTS.md` | Thiết kế và quy trình **đã khoá**: quy ước module, quy trình team AI |
| `MODULE_PATTERN.md`, `MODULE_REWORK.md` | Quy ước đặt tên, ranh giới module, quy trình rework |
| `server/service/**/README.md`, `*_FLOW.html` | Luồng từng module (HTML có sơ đồ) |
| `log/SECURITY_NOTES.md`, `log/TIEU_CHI_TEST.md` | Danh sách lỗ hổng SEC-xx và tiêu chí từng luồng test. Test có tham chiếu tới hai file này |
| `agent_workspace/` | Đội agent: `TEAM.md`, `agents/*.md` (6 vai + `hacker`), `PHASE_FORMAT.md`, `CODE_STYLE.md`, `sync_agents.py` |
| `docs/` (sắp lại 10/10/2026, chưa commit) | Mục lục `docs/README.md`. `manual/system-manual.html` (manual tổng, trước là `FlexMix_System_Manual.html`); `flows/` mỗi luồng một trang: `account-auth`, `machine-register`, `machine-share-manage`, `machine-link`, `menu-image` (chuẩn FexMix, viết 10/10/2026 theo `agent_workspace/tasks/docs-flows/BRIEF.md`) và `menu-ingredient-sync` (trước là `dong-bo-du-lieu.html`, còn CSS cũ); template chuẩn chép ở `docs/assets/template.html`; `design/heartbeat-vs-long-poll.{md,html}` (code/test dẫn chiếu bản .md); `assets/` CSS + font offline dùng chung |
| `agent_workspace/tasks/middleware/` | Kế hoạch middleware: HTML cho người dùng, `TASK.md`, spec tại `internal/`. Tóm tắt ở [MIDDLEWARE.md](MIDDLEWARE.md) |

## 7. Bẫy cần nhớ
- Đổi URL, tên lệnh hoặc trường gói tin thì phải sửa **cùng lúc** app (`routing.dart`), server (`routing.py`) và máy, kèm test.
- `python3` trên PATH đang trỏ venv của `project1_server`. Chạy test app bằng `/usr/bin/python3` (3.14).
- `tests/test.ps1` và hướng dẫn trong repo dùng đường dẫn Windows (`.venv/Scripts`, `D:/...`). Máy hiện tại chạy Linux.
- `menu_sync` của máy đọc SQLite `version1.1/machine/database/database.db` (file không có trong tree), còn `ingredient_sync` đọc MySQL qua package database của máy pha; `sqlite_reference/machine.sql` ghi runtime vẫn dùng MySQL. Chưa chốt menu dùng DB nào (kiểm 09/10).
- `version1.1/` có 3 bản ở 3 chỗ khác nhau, không giả định chúng đồng bộ (xem `../README.md`).
