---
id: archive-iot-design-research-20261011
type: agent-report
scope: project:project1_app
status: extracted
date: 2026-10-11
source: 5 agent general-purpose, Opus 5.5 effort medium, không đọc vault
---

# Kết quả cuối 5 agent research, tầng 1 hệ IoT v1.1

Tóm tắt kết quả cuối, không phải transcript. Phiên: [session](../../../../sessions/2026-10-11-iot-system-design-tier1.md).

## R1. Máy mới lên mạng và claim
(Agent báo không truy cập web được; case study dựa trên kiến thức sẵn có.)
- A **mã xoay trên màn hình, app quét, xác nhận trên máy** (kiểu device-code RFC 8628). Agent khuyên dùng.
- B BLE từ app (Ring/Matter/Tuya).
- C app hiện QR, máy quét (Wyze).
- D nạp sẵn ở nhà máy, kèm mã sở hữu.
- Đã có chủ thì từ chối; factory reset không xoá quyền sở hữu.
- Tránh làm sớm: PKI, Matter, SoftAP, LTE.
- Phát hiện: `product_key` tĩnh lộ qua BT; pairing Just Works.

## R2. Dữ liệu đơn, kho, menu
- A relay thuần. B máy gốc, server bản sao. C server gốc, máy cache.
- **D tách theo loại (desired/reported + sổ sự kiện idempotent)**. Agent khuyên dùng.
- Case study: AWS Device Shadow, Azure Device Twin, Square/Toast offline, Coca-Cola Freestyle, Nayax.
- Khoá máy hay tắt món là *trạng thái mong muốn*, không phải lệnh một lần.
- Tránh: event sourcing toàn hệ thống, CRDT, broker, stream realtime.

## R3. Phân quyền và chuyển nhượng
- A1 vai theo máy. A2 theo cửa hàng. **A3 cửa hàng, thu hẹp xuống máy**. A4 IAM đầy đủ.
- Khuyên: A1 trước, có cửa hàng ngầm định, sau nâng lên A3.
- Luật: không cấp vượt quyền mình; step-up cho thao tác rủi ro; thu hồi có hiệu lực ngay; audit log chỉ ghi thêm.
- Chuyển nhượng: B1 hai bên xác nhận, B2 release/claim, **B3 B1 + có mặt tại máy**, B4 qua support.
- Ghi chú: agent giả định "đúng 1 Owner"; **user sau đó bác luật này** (Super Admin / Admin / Staff).
- Đối chiếu code `admin_gui/auth.py`: `outranks`, luôn ≥1 owner hoạt động.

## R4. Định danh chủ doanh nghiệp
- Ba mẫu xác minh:
  - nắm vật/kênh (GBP, UniFi);
  - pháp lý KYB (Grab, Zalo OA, MoMo, Stripe);
  - **kênh bán hàng (Toast hợp đồng, Meraki mã đơn, VNPay)**.
- Khuyên: gán trước chủ theo đơn bán, kèm lối thoát thủ công của hãng.

## R5. Gắn thiết bị vào bên mua qua kênh bán
- Case study: Apple ABM/ADE, Android Zero-touch, Samsung KME, Autopilot, Meraki Order Claim Key, Aruba, Nayax, Tesla, Activation Lock/FRP.
- Mẫu chung:
  - kênh bán xác lập quyền;
  - bí mật đi ngoài máy;
  - mỗi thiết bị một tổ chức;
  - hãng gỡ khoá khi có hoá đơn hợp lệ.
- Khuyên: gán trước serial lúc bán, cộng ân hạn và khiếu nại bằng hoá đơn; claim key (Meraki) làm dự phòng.

## Kết cục trong phiên
User chọn **mô hình iCloud** (kích hoạt lần đầu tại máy), định danh bằng **serial CPU**, vai Super Admin / Admin / Staff. Các khuyến nghị trên còn chờ chốt; xem "Phần còn mở" trong session.
