# Research: tài khoản, đăng nhập, danh tính offline (11/10/2026)

3 agent Sonnet med (prompt không chứa nội dung vault) + 1 lượt kiểm nguồn bằng web. Đối chiếu code `project1_app` @ `c34d55e`. HTML: `documentation/research/account-login-identity.html`.

## Kết luận đồng thuận

- Hai lớp danh tính: app↔server dùng email + mật khẩu + token phiên; app↔máy dùng khoá điện thoại (Keystore) và bản quyền do server ký. Máy không thấy mật khẩu hay token.
- Token phiên opaque lưu băm ở server, không JWT (cần thu hồi ngay; server nhỏ). Refresh xoay vòng, dùng lại thì thu hồi cả phiên.
- OTP email chỉ để xác minh email và khôi phục (NIST cho phép), không dùng đăng nhập hằng ngày.
- Tạo tài khoản "chưa xác minh" trước; đăng ký lại cùng email thì ghi đè; dọn sau 24 giờ.
- OTP 6 số CSPRNG, 10 phút, một lần, lưu HMAC, gắn mục đích; 5 sai/mã, 10 sai/giờ/email; gửi lại 60s, 5 mã/giờ/email, 20/giờ/IP.
- Argon2id m=19MiB t=2 p=1 (OWASP). bcrypt giới hạn 72 byte.
- Bản quyền: {mã máy, phiên bản tăng dần, giờ cấp, hạn, [người, điện thoại, khoá công khai, vai]}, ký ECDSA P-256. Máy chống phát lại bằng nonce. Cửa sổ rủi ro gỡ người khi máy offline = min(thời gian mất server, hạn).
- Không làm ở MVP: OAuth/OIDC, DPoP, MFA bắt buộc, CAPTCHA, passkey, đổi email, PKI kiểu Matter, giấy phép share qua BLE khi offline.

## Nguồn (mức kiểm)

| Nguồn | Nội dung dùng | Mức |
|---|---|---|
| https://pages.nist.gov/800-63-4/sp800-63b.html | Mật khẩu 15 (đơn yếu tố)/8 (MFA); OTP ≥6 số ≤10 phút; email chỉ cho xác minh/khôi phục; AAL1 phiên ≤30 ngày, AAL2 ≤24h/nghỉ ≤1h | mở (đọc ~1/3 trang) |
| https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html | Argon2id tham số | mở |
| https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html | Phản hồi giống nhau, thu hồi phiên, không tự đăng nhập | mở |
| https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html | Câu trung tính khi đăng ký, đếm sai theo tài khoản | mở |
| https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html | Entropy token, thu hồi ở server; timeout của nó viết cho web | mở |
| https://www.rfc-editor.org/rfc/rfc9700.html | Refresh của public client phải xoay vòng hoặc sender-constrained (2.2.2) | mở |
| https://firebase.google.com/docs/auth/admin/manage-sessions | ID token 1h, checkRevoked | mở |
| https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation | Reuse detection; "Rotation Overlap Period" mặc định tắt | mở |
| https://supabase.com/docs/guides/auth/sessions | Reuse interval 10s, access 1h | mở |
| https://docs.cloud.google.com/identity-platform/docs/admin/email-enumeration-protection | Đăng ký trùng vẫn trả EMAIL_EXISTS | mở |
| https://developer.tesla.com/docs/fleet-api/virtual-keys/developer-guide · https://www.ndss-symposium.org/wp-content/uploads/2023-82-paper.pdf | Khoá điện thoại đăng ký vào xe; MITM BLE | tìm |
| https://nrfconnectdocs.nordicsemi.com/ncs/latest/matter/access-control-guide.html | ACL trên thiết bị, root cài lúc kích hoạt | tìm |
| https://docs.nuki.io/guide/overview/concepts/ | Quyền lưu trên khoá; API không tạo quyền khi khoá offline | tìm |
| https://support.apple.com/guide/security/sec75f6d5040/web | Khoá trong Secure Element | tìm |
| Tuya, Xiaomi, Apple ID, app ngân hàng | — | không kiểm được, không dùng làm căn cứ |

## Đối chiếu code

- `version1.1/machine/pairing/machine_bluetooth_serve.py:30` RequestConfirmation không kiểm; `machine_bluetooth_pair.py:52-57` gửi `product_key` sau khi ghép; `server_connection/machine_server_request.py:12` `product_key` là danh tính máy → nghiêm trọng, chưa sửa.
- `flutter_app/lib/feature/user_auth/ui/auth_page.dart`: có họ tên, "Tên đăng nhập" (204-216), email chỉ khi đăng ký, mật khẩu ≥8 (264). `otp_page.dart`: 6 số, cooldown 60s chỉ ở client.
- Agent đăng ký báo "chưa có ô họ tên" — sai, đã kiểm lại: có (`auth_page.dart:189-199`).
