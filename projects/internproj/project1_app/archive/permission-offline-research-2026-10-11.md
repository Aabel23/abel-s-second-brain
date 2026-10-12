---
id: archive-permission-offline-research-20261011
type: agent-report
scope: project:project1_app
status: extracted
date: 2026-10-11
source: 3 agent general-purpose, Opus 5.5 effort medium, không đọc vault, prompt không chứa thiết kế hiện tại
---

# Kết quả cuối: 3 research phân quyền, thành viên máy, offline (v1.1)

Phiên: [session](../../../../sessions/2026-10-11-iot-system-design-tier1.md). User yêu cầu "tham khảo thế giới, quên context thiết kế hiện tại", rồi gộp thành 4 option. Chỉ ghi kết quả cuối.

## 1. Phân quyền khi thiết bị offline
- Salto KS: cloud gốc, khoá giữ bản sao, master key chủ đồng bộ trước. Homelok: key gia hạn 3 ngày mỗi lần quẹt, tối đa 30 ngày. https://support.saltosystems.com/ks/general/offline-access/
- Nuki: quyền lưu trên khoá, sửa qua Bluetooth tại chỗ khi không mạng (nguồn forum dev). Glue lock (KTH): khách bị thu hồi vẫn mở qua BT khi điện thoại tắt mạng.
- Tesla: xe giữ keychain; thêm key cần key card có mặt; lệnh từ xa qua server rồi xe tự kiểm chữ ký.
- Matter: ACL trên thiết bị, người ghép đầu là admin, nhiều fabric song song.
- Square: offline có hạn 72 giờ. Toast: mất mạng đầu ca thì không đăng nhập được. Không sửa quyền offline.
- AWS Greengrass: thời gian tin bản cache là tham số (mặc định 1 phút).
- Mẫu: cloud gốc + cache; thiết bị gốc + cloud gương; quyền tự hết hạn; credential mang dữ liệu (vé ký); cấp quyền tại chỗ cần người có quyền có mặt. **Không case nào cho sửa hai nơi rồi gộp.**

## 2. Ai thuộc máy nào
- Per-device: Ring (không chuyển chủ trực tiếp, người mua lại chờ ~15 ngày), Tesla (chủ + 5 driver, chuyển chủ xác nhận email 2 bên, ≤48h), August (Owner/Guest, nhiều Owner), DJI.
- Vật chứa: Google Home (Admin/Member, mời phải chấp nhận, người cuối mới xoá được Home; Nest phải ép chuyển tài khoản), Apple Home, Hue (link mời 24h; vai Lighting quá rộng), Meraki (claim vào Tổ chức bằng mã in trên máy), UniFi (chuyển owner cho admin có sẵn), Square (device code gắn máy vào location).
- Apple Business Manager sinh ra vì máy công ty dính khoá tài khoản cá nhân nhân viên cũ.
- Lỗi hay gặp: quyền ở cá nhân thay vì tổ chức; nhiều hệ tài khoản song song; vai thành viên quá rộng; người cũ giữ quyền (NYT 2018); mã chung không truy vết; mã claim đoán được; cache offline không hạn; mất điện thoại cần thu hồi phiên từng máy.
- Agent gợi ý: "cửa hàng" chứa máy + PIN offline, chừa đường lên Tổ chức.

## 3. Vai trò nhân viên
- Square: permission set 3 mức, ngân hàng chỉ owner, chuyển sở hữu người nhận xác minh 24h. Toast: quyền theo job, trao quyền thì phải có quyền đó. Shopify: Manager approval nhập PIN tại chỗ. Lightspeed: Admin/Manager/Cashier, Cashier khoá cứng phần thiết lập. Clover tương tự. KiotViet: vai theo chi nhánh + tích quyền riêng từng người, Xem/Tạo/Sửa/Xoá, nghỉ việc thì ngừng chứ không xoá. Sapo: doanh thu phân theo nhân viên/chi nhánh.
- Máy pha/bán hàng: WMF CoffeeConnect quyền theo từng máy; Nayax có vai cho người châm hàng; PayRange tài xế tuyến chỉ thấy máy của tuyến.
- Google Home Member + 2 công tắc (Activity, Settings). Ring Limited/Standard/Advanced, Installer hết hạn 48h.
- Mẫu: luôn có mức nền cố định; vai mẫu + tuỳ chỉnh là chuẩn POS; tách xem/làm; tiền hai tầng (báo cáo giao được, ngân hàng chỉ owner); quyền theo nơi; không trao quá mình có; thao tác nhạy cảm hỏi lại PIN/mật khẩu hoặc duyệt.

## Gộp thành 4 option (operator, chưa chốt)
Xem chat phiên 2026-10-11: (1) kiểu khoá thông minh, (2) kiểu POS cửa hàng, (3) máy tự quyết, (4) vé ký mang theo.
