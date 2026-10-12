---
id: session-20261011-app-endpoints-auth-research
type: session-learning
scope: project:project1_app
status: extracted
date: 2026-10-11
source: current-conversation (Claude Code, 11–12/10/2026)
---

# Mục tiêu và chuẩn đạt

User muốn: (1) biết hệ hiện có bao nhiêu endpoint, tập trung vào **app** (`project1_app`), không phải `project1_server`; (2) từ thiết kế IoT v1.1 rút ra các luồng, tính năng và danh sách endpoint cần có, rồi ghi vào routing; (3) liệt kê luồng còn chưa rõ; (4) research giao thức tạo tài khoản và đăng nhập, ra HTML research.

# Những lần chỉnh hướng

- Agent trả lời số endpoint của `project1_server` (71 khai báo, 20 đã code) → user: "không, tập trung vào app" → câu hỏi về "app" mặc định là repo `project1_app`; `project1_server` chỉ khi user nêu tên.

# Quyết định và giới hạn

- [source-observed] Lúc bắt đầu, `project1_app/server/configuration/routing.py` rỗng; Flutter chưa có lời gọi HTTP nào. File routing của server app nằm ở `server/configuration/`, **không** phải `server/config/` như chuẩn đặt tên cũ ghi.
- [user-confirmed bằng "ok viết các endpoint này vào routing"] 25 endpoint: 17 app gọi (tài khoản 5, máy 4, share/quyền 7, relay 1) + 8 máy gọi. Lệnh menu/kho/đơn đi qua **một route relay chung** `/app/machine/command/send`, không mỗi lệnh một route. Agent tự đổi `member/role/update` → `/app/machine/role/update` (giữ 3 đoạn).
- [user-confirmed] Tài khoản và đăng nhập — xem [facts/project1-app-auth-identity-confirmed.md](../facts/project1-app-auth-identity-confirmed.md).
- 6 luồng × khoảng 21 tính năng (tài khoản; kích hoạt và vòng đời máy; share và quyền; vận hành; quản trị; nền hệ thống) là phân tích của agent, user chưa chốt.

# Sản phẩm hiện hành

- `project1_app/server/configuration/routing.py`: 25 hằng đường dẫn (chưa có handler). `version1.1/machine/config/routing.py`: 8 route máy, khớp server; đã kiểm tên biến khớp path và import của máy vẫn chạy.
- `project1_app/documentation/research/account-login-identity.html` (untracked): 6 mục, 7 sơ đồ, 41 chú thích; check_render: 0 fallback, không tràn ngang.
- Research 3 agent Sonnet med (đăng ký/OTP, phiên, danh tính offline) + 1 lượt kiểm nguồn: [archive](../projects/internproj/project1_app/archive/auth-identity-research-2026-10-11.md).

# Phần còn mở

- Route mới chỉ ghi trong doc, **chưa thêm vào routing.py**: `session/refresh`, `reauth`, `list`, `revoke`, `password/forgot`, `password/reset`; login nhận thêm khoá công khai.
- Thông số agent tự đề xuất, chờ duyệt: OTP 5 lần sai/mã, 5 mã/giờ, Argon2id tham số OWASP, dọn tài khoản chưa xác minh sau 24 giờ, tối đa 5 phiên, chừa 10 giây khi refresh; quên mật khẩu (3.2) cả luồng; cách bắt tay kênh mã hoá (khoá tạm có chữ ký, máy có khoá riêng từ chứng nhận hãng).
- Câu hỏi mở: giờ máy để kiểm hạn 7 ngày; nơi cất và cách đổi khoá ký server; giao thức kênh cụ thể (Noise / TLS 1.3); dịch vụ gửi thư; Owner mất email thì máy đi đâu; chuyển chủ có xác nhận email chủ cũ không.
- Lỗi đã kiểm trong code, chưa sửa: Bluetooth máy tự chấp nhận mọi điện thoại (`version1.1/machine/pairing/machine_bluetooth_serve.py:30`) và gửi `product_key` (`machine_bluetooth_pair.py:52-57`), trong khi `product_key` là danh tính duy nhất của máy với server. Màn đăng nhập còn ô "Tên đăng nhập" (`auth_page.dart:204-216`); độ dài mật khẩu và cooldown OTP chỉ kiểm ở app.
- Danh sách khoảng 20 câu hỏi theo luồng (dữ liệu hằng ngày, nhật ký ở đâu, bảng vai đầy đủ, phạm vi admin_gui cũ, share, vòng đời máy…) đã nêu trong chat, chưa chốt ngoài phần tài khoản.
