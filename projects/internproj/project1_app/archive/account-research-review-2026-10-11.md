---
id: archive-account-research-review-20261011
type: agent-report
scope: project:project1_app
status: extracted
date: 2026-10-11
source: 3 agent general-purpose, Opus 5.5 effort medium, không đọc vault
---

# Kết quả cuối: 2 research tài khoản + 1 review thiết kế (v1.1)

Phiên: [session](../../../../sessions/2026-10-11-iot-system-design-tier1.md). Chỉ ghi kết quả cuối.

## R6. App POS/F&B Việt Nam (KiotViet, Sapo FnB, MISA CukCuk, Ocha, iPOS/Fabi, GrabMerchant)
- **SĐT là danh tính chính**, email chỉ tùy chọn. Form đăng ký có 4–5 trường (họ tên, SĐT, tên quán, tỉnh/phường, ngành). Không hỏi mã số thuế; thông tin pháp lý thu sau.
- Đăng ký xong là **có cửa hàng ngay**, không duyệt (Grab là ngoại lệ vì là sàn).
- **Chủ tạo tài khoản cho nhân viên** (tên đăng nhập + mật khẩu). Nhân viên không tự đăng ký; cả 6 app đều vậy.
- Vai trò mẫu có thể tick sửa. Quản lý không quản nhân viên (Ocha, Grab). Khoá tài khoản khi nghỉ việc là tính năng mặc định.
- Phương án: A kiểu KiotViet / B hãng cấp tài khoản theo đơn bán / C mời theo SĐT / D tài khoản chủ + PIN nhân viên.
- Agent khuyên: B chính, A dự phòng. **Mâu thuẫn với lựa chọn của user** (tự đăng ký, doanh nghiệp tùy chọn), chỉ ghi để tham khảo.
- Lời khuyên dữ liệu: tách *Người dùng* khỏi *Thành viên doanh nghiệp*.

## R7. Danh tính khác lối thường
- OTP qua Zalo ZNS, SMS dự phòng. Chưa có giá chính thức; cần báo giá eSMS/VietGuys. NĐ 147/2024 đã khiến người dùng quen xác thực bằng SĐT.
- POS quốc tế (Square, Toast, Loyverse, Lightspeed) tách hai tầng: **chủ có tài khoản thật, nhân viên là hồ sơ + PIN** trên thiết bị cửa hàng.
- Phương án: A Square/Toast PIN / B mời qua SĐT (Square Team) / C kiểu "nhà" Tuya, Google Home / D quyền theo ca có hạn.
- Agent khuyên A, chừa chỗ cho B.
- Tránh: passkey, đăng nhập Zalo (chưa rõ có lấy được SĐT không), quyền tùy biến từng thao tác.

## V1. Review thiết kế
- **Cao:**
  - C1: chưa có luật Super Admin gỡ Super Admin (console: `outranks` + luôn ≥1 owner).
  - C2: khoá kích hoạt thuộc tài khoản hay doanh nghiệp chưa định nghĩa.
  - C3: đại lý hoặc thợ lắp kích hoạt trước.
  - C4: serial CPU giả được, thay bo thì mất khoá.
  - C5: mất email là mất máy; chiếm email là chiếm máy.
- **TB:**
  - T1: chế độ offline và khoá.
  - T2: ranh giới cá nhân ↔ doanh nghiệp (dữ liệu, giải thể, người né tạo doanh nghiệp bằng cách chia mật khẩu).
  - T3: một người nhiều doanh nghiệp.
  - T4: vé bàn giao chưa có hạn, chưa ràng buộc người nhận.
  - T5: phải mang luật bất biến sang, "chỉ cấp quyền mình có".
  - T6: dữ liệu doanh nghiệp (đồng ý, ai xem, lưu bao lâu).
- **Thấp:** L1 mã xoay; L2 dữ liệu chủ cũ khi bán lại; L3 không cho xoá audit.
- **Mâu thuẫn:**
  - "không có một chủ" ↔ "khoá gắn một tài khoản";
  - console cấm owner gỡ owner ↔ nhiều SA ngang nhau;
  - tài khoản cá nhân có 2 nghĩa;
  - serial CPU ↔ "reset không gỡ khoá" (khi thay bo).
- **10 luồng thiếu:**
  1. chuyển máy;
  2. rời, xoá, giải thể;
  3. khôi phục, đổi email;
  4. hãng gỡ khoá;
  5. đại lý;
  6. thay bo;
  7. offline;
  8. thu hồi và vé bàn giao;
  9. đồng ý dữ liệu;
  10. chọn ngữ cảnh.
- **Tránh overengineer:**
  - cây chi nhánh;
  - quyền từng thao tác;
  - làm cả QR lẫn BT ở bản đầu (chọn QR);
  - thu nhiều thông tin bắt buộc;
  - hệ thống đại lý đầy đủ.
