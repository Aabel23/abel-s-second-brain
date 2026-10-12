# Research pair máy và share device (12/10/2026)

Tóm tắt kết quả 3 agent researcher (Sonnet medium, chạy song song). Prompt chỉ gồm câu hỏi, đường dẫn code và file thiết kế `documentation/research/iot-v11-system-design.html` trong repo; không có nội dung vault. Đây là bản rút gọn do operator viết, không phải nguyên văn. Session: [sessions/2026-10-12-pairing-device-sharing-research.md](../../../../sessions/2026-10-12-pairing-device-sharing-research.md).

## Agent 1 · Pair lần đầu
- Mã niêm phong ngẫu nhiên ~100 bit (base32 ~20 ký tự) trong QR, nên không cần PAKE; NIST SP 800-63B §5.1.3.2 chỉ đòi giới hạn thử khi dưới 64 bit. Chỉ làm chậm (~1 lần/giây), không khóa cứng (tránh DoS).
- Kênh: BLE GATT + bắt tay X25519 có PSK là mã niêm phong (kiểu Noise NNpsk0), HKDF, AEAD. Không dựa vào bảo mật link BLE (Just Works không chống MITM; OOB/Numeric Comparison không khả dụng trên Flutter/iOS).
- Gửi wifi qua kênh BLE kiểu Espressif/Matter: thử mạng, báo lỗi, hoàn tác. Bỏ SoftAP, DPP.
- Claim cần 3 bằng chứng: máy ký nonce server bằng khóa máy; app nộp mã niêm phong; mã một lần N do máy sinh, gửi cả server và app qua BLE. `UPDATE ... WHERE owner IS NULL` chống race. Claim idempotent theo N.
- Trạng thái UNCLAIMED → SESSION → CLAIMED. Mã chỉ tác dụng khi chưa có chủ.
- Rủi ro: hộp bị mở trước khi bán; khóa trong file đọc được nếu có root; bán lại thì chủ cũ biết mã.
- Case: Matter (Silicon Labs), ESP-IDF provisioning, Tuya, AWS fleet provisioning, Azure DPS, Apple Activation Lock, RFC 9383.

## Agent 2 · Share device
- QR mời (Owner/Manager hiện, người mới quét) + bước người share xác nhận tên/email người nhận. Token CSPRNG ≥128 bit, lưu băm, dùng một lần, hạn 10–15 phút (OWASP). Không đặt token vào query string.
- Khuyên server ký giấy phép thay vì điện thoại người share ký (kiểu CCC/Nuki) để khỏi quản lý khóa ký ở điện thoại.
- Khuyên bỏ BT share ở MVP: SPP không chạy trên iOS, phone-to-phone chỉ BLE và iOS hạn chế nền, đứng gần không chứng minh đúng người. Nếu giữ: BLE + so 6 số, BT chỉ chở mã mời. **User giữ BT share (1b).**
- Server: bảng invite + grant; kiểm quyền tại thời điểm gọi; vai luôn Employee; giới hạn số invite chờ (5), số người/máy (20–50), rate limit; báo Owner khi có người mới.
- Máy kéo bản quyền (sau NAT), long-poll báo phiên bản mới; bản sao ký, `version` chống rollback, hạn 7–30 ngày.
- Vé LAN/BT: khuyên bearer ngắn hạn ở MVP, gắn khóa sau. **User chọn gắn khóa ngay (3a).**
- Code: `routing.py:29-31` có share create/accept/sign; `machine_share_page.dart:49,95` gọi `notAvailableYet`; trang BT có nút rỗng.

## Agent 3 · Nền tin cậy xuyên suốt
- Server là issuer duy nhất (khóa trong KMS); máy pin khóa công khai server. Serial chỉ là nhãn; bằng chứng là khóa máy (lý tưởng: chứng chỉ xuất xưởng do root hãng ký; nếu xưởng chưa làm được thì sinh lần boot đầu gắn mã niêm phong).
- Một cách xác minh cho 3 chế độ: vé ký (CWT/COSE hoặc JWT) chứa người, máy, vai, khóa công khai app, exp, perm_ver; app ký challenge của máy. Kênh (Noise/TLS raw key) chỉ mã hóa; quyền đến từ vé.
- Đồng hồ: nonce chống replay; hạn so với `last_server_time` + đồng hồ đơn điệu.
- Bản sao quyền ký, perm_ver tăng dần, hạn 7 ngày; vé 24 giờ. Thu hồi chỉ online; cửa sổ rủi ro = min(hạn vé, hạn bản sao). Epoch tăng khi chuyển chủ/gỡ máy. Không cần CRL/OCSP.
- Không cho gỡ người offline; cho Owner tạm khóa máy qua BT (chỉ thu hẹp quyền).
- Gỡ/bán lại: xóa khóa vận hành và bản sao, giữ danh tính xuất xưởng; reset tại chỗ không đủ để nhận chủ lại. Tranh chấp: hãng xử lý thủ công.
- Nhật ký: máy ký bằng khóa máy, đẩy lên khi online (chưa nghiên cứu sâu).
- Ghi chú nguồn: Apple chỉ có link diễn đàn; Nest, August, ETSI, NIST IR 8259, RFC 8392/9052 chưa mở lại.
