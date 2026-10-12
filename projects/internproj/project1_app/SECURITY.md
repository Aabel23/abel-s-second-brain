---
id: project1-app-security
type: project-analysis
scope: project:project1_app
status: source-observed
checked: 2026-10-09
anchor_commit: 58a0b01
---

# Bảo mật `project1_app`: lỗ hổng đã biết và thiết kế đã chốt

## Lỗ hổng (nguồn: `log/SECURITY_NOTES.md` trong repo, viết ngày 28/09)

File gốc nhắc vài route **đã bị bỏ** (`/app/gui-lenh`, `/machine/tra-dong-bo`). Cột "Hiện tại" là kết quả đối chiếu với code ngày 09/10.

| Mã | Mức | Vấn đề | Hiện tại |
|---|---|---|---|
| SEC-01 | Cao | HTTP không mã hoá: mật khẩu, token, product key đi dạng rõ | Còn. Sẽ xử lý bằng D5-A (TLS) + bảo vệ gói tin |
| SEC-02 | Cao | Ai có product key trên tem QR cũng giả được máy, cướp lệnh, trả kết quả giả | Còn. Sẽ xử lý bằng D3-B |
| SEC-03 | TB | Bảng IP rate limit đầy 1000 thì chặn mọi IP mới | Còn (`http_rate_limit.py`) |
| SEC-04 | TB | Lấp đầy 1000 phiên đăng nhập chờ | **Hết hiệu lực**: login giờ là 1 bước, không còn phiên chờ |
| SEC-05 | TB | Đăng ký làm lộ email/username đã tồn tại | Còn: `user_register_validate.py:15,17` trả "Tên đăng nhập/Email đã tồn tại" |
| SEC-06 | TB | Bom gzip | Route gzip đã bỏ |
| SEC-08 | Thấp | `GET /app/machine/status/get` không cần đăng nhập | Còn. D8-C: sẽ bỏ route này |
| SEC-09 | Cao | Máy ở chế độ pair Bluetooth trao key cho bất kỳ thiết bị nào | Còn (mã máy) |
| SEC-10 | TB | Chia sẻ Bluetooth chọn người nhận theo tên thiết bị | Còn |
| SEC-11 | TB | Nhân viên có quyền như chủ (đổi giá, nạp kho) | Còn: `QUYEN_MENU` và `QUYEN_NAP_KHO` đều gồm `manager` |
| SEC-12 | TB | Không giới hạn số thread/kết nối | Còn (`ThreadingHTTPServer`). Middleware G3 sẽ xử lý |
| SEC-13 | TB | APK release ký bằng debug key | Còn |
| SEC-14 | Thấp | Username trùng nhau chỉ khác hoa/thường | Có test X1 |
| SEC-15 | Thấp | Phiên 30 ngày, không có "đăng xuất mọi thiết bị" | Còn |
| SEC-16 | TB | Dán đè tem QR giả | Còn |
| SEC-17 | ? | DB MySQL của máy không có trong repo | Không kiểm được |
| SEC-18 | Thấp | Server nghe trên `0.0.0.0` | Còn |
| SEC-19 | TB | Bom zlib trong gói Menu | Còn |

## Thiết kế bảo vệ gói tin đã chốt (07/10/2026, user chốt)

Nguồn: `agent_workspace/tasks/middleware/internal/packet-security/design.md` §0 và §13. Thiết kế đã chốt **không** có nghĩa thư viện đã build được hay vector đã đạt.

| Mã | Quyết định |
|---|---|
| D1-B | Root ký manifest là file khoá có passphrase, nằm trên máy ký air-gapped, backup ở 2 nơi |
| C2-B | ECDSA P-256/SHA-256 cho root, app và máy, mỗi vai một khoá. Wire r‖s 64 byte, low-s |
| C4-B | Khoá app bắt buộc nằm trong Android Keystore mức TEE trở lên, không đạt thì từ chối enroll. Máy và server giữ key file quyền 0600 |
| C1-A | HPKE DHKEM(X25519, HKDF-SHA256) / HKDF-SHA256 / AES-128-GCM |
| C3-A | App dùng Bouncy Castle HPKE qua Kotlin MethodChannel. Server và Pi dùng PyHPKE (`cryptography`). Dart không tự làm crypto |
| D2-C | App kiểm đồng hồ hệ điều hành bằng thời gian do server ký, qua một kênh riêng |
| D3-B | Máy được provision khi lắp: sinh cặp khoá, admin đăng ký public key. Chủ máy nhận claim code niêm phong, app yêu cầu máy ký nonce |
| D4-B | Recovery code dùng 1 lần, lưu hash, có rate limit |
| D5-A | Server Python tự kết thúc TLS (`ssl` stdlib), cert tự cấp được pin trong `network_security_config` |
| D6-A | Chuyển toàn bộ sang profile bảo vệ trong một đợt bảo trì, không chạy song song giao thức cũ. Lý do: hệ thống chưa có người dùng thật |
| D8-C | Bỏ `GET /app/machine/status/get`, gộp `online`/`last_seen` vào danh sách máy |

(D7 không có trong bộ đã chốt. Nếu cần, xem `archive/middleware-operator-2026-10/decision-options-2026-10-07.md`.)

Cần nghiên cứu sâu (các phát hiện F01–F11, cải tiến I01–I12, P0 gồm journal mode/durability, trusted time, bootstrap) thì mở [archive/security-research-2026-10-07/FINDINGS.md](archive/security-research-2026-10-07/FINDINGS.md) và `IMPROVEMENTS.md`. Chỉ mở khi task chạm tới phần này.
