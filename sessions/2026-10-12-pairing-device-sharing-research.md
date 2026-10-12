---
id: session-20261012-pairing-device-sharing-research
type: session-learning
scope: project:project1_app
status: extracted
date: 2026-10-12
source: current-conversation
---

# Mục tiêu và chuẩn đạt
User yêu cầu "research giao thức pair máy và share device" cho hệ IoT máy pha v1.1 (`project1_app`). Agent hiểu: pair = kích hoạt lần đầu (mã niêm phong → BLE → wifi → nhận Owner); share = Owner/Manager bàn giao máy cho tài khoản nhân viên bằng QR hoặc Bluetooth. User không sửa cách hiểu này. Đạt khi có bản tổng hợp kèm câu hỏi để user quyết, rồi HTML research theo chuẩn FexMix trong `documentation/research/`.

## Những lần chỉnh hướng
- Không có correction trực tiếp. User nhận 8 câu hỏi dạng (a)/(b) kèm khuyến nghị và trả lời ngắn từng số. Lần sau: giữ dạng này.
- User **bác khuyến nghị bỏ Bluetooth share** (agent khuyên chỉ QR vì iOS không dùng SPP, BLE nền bị hạn chế). User giữ quyết định cũ "cả QR lẫn BT ngay bản đầu". Lần sau: không đề xuất lại bỏ BT share nếu không có lý do mới. Cùng kiểu với lần user giữ email dù research khuyên SĐT.
- User muốn mọi thông số thời hạn, ngưỡng là biến cấu hình lâu dài, gom vào một file config, dev chỉnh được; yêu cầu ghi vào doc. Agent chỉ ghi vào doc, chưa tạo file trong code (theo hiến pháp, chưa sang tầng kỹ thuật).

## Quyết định và giới hạn
- [user-confirmed] 1b: share qua Bluetooth có ở MVP, làm bằng BLE, hai điện thoại hiện mã 6 số để so.
- [user-confirmed] 2a: server ký giấy phép share; người share chỉ xác nhận (vân tay/PIN), không giữ khóa ký.
- [user-confirmed] 3a: vé dùng cho LAN/BT gắn khóa riêng của điện thoại (proof-of-possession) ngay từ MVP.
- [user-confirmed] 4a: mã niêm phong là chuỗi ngẫu nhiên dài (~100 bit) in QR; không dùng PIN ngắn và PAKE.
- [user-confirmed] 5b: máy tự sinh cặp khóa ở lần bật đầu, gắn với mã niêm phong (không yêu cầu xưởng nạp khóa/chứng chỉ ở MVP).
- [user-confirmed] 6: các thông số (hạn vé, hạn bản sao quyền, hạn mã mời, khung chờ claim…) là biến config lâu dài, gom một file config, dev chỉnh được.
- [user-confirmed] 7a: giữ mã niêm phong cố định suốt đời máy; chấp nhận rủi ro chủ cũ biết mã khi bán lại.
- Câu 8 (Manager có gỡ được Employee không) user chưa trả lời.
- [agent-proposed, chưa chốt] BT share chỉ chở mã mời giống QR, sau đó đi tiếp như QR; `share/sign` đổi nghĩa thành "xác nhận"; cần thêm route app xin vé; giá trị mặc định: vé 24 giờ, bản sao 7 ngày, làm mới 6 giờ, mã mời 15 phút, khung claim 15 phút, phiên BT 10 phút, tối đa 30 người/máy, 5 mã mời chờ/máy, thử sai 1 lần/giây.
- [agent-proposed] Server chỉ nhận khóa công khai của máy một lần, ở bước kích hoạt, khi app nộp mã niêm phong + mã một lần mà máy đưa qua BLE.
- [source-observed, agent đã tự mở file kiểm] `version1.1/machine/pairing/machine_bluetooth_pair.py:19-26,52-57`: bước `identify` không xác thực, gửi `product_key` dạng rõ cho bất kỳ ai nối được. `machine_bluetooth_serve.py:11,30-31,112`: dùng SPP, tự chấp nhận mọi yêu cầu ghép kiểu Just Works. `server_connection/machine_server_request.py:12`: xưng danh bằng `product_key` cố định, không ký. Import lệch giữa `machine.config.env` và `config.env`. **Chưa sửa.**
- [agent báo, chưa tự kiểm] `machine/main.py` chạy lệnh từ server không kiểm chữ ký; URL server mẫu `http://`.

## Sản phẩm hiện hành
- HTML research: `project1_app/documentation/research/pairing-and-device-sharing.html` (7 thành phần, 8 sơ đồ Mermaid, bảng thông số config, 35 chú thích). Đã chụp kiểm 1400 px: 0 sơ đồ rơi về mã nguồn, không tràn ngang ở màn hẹp.
- Kết quả 3 agent researcher (Sonnet medium): [archive/pairing-sharing-research-2026-10-12.md](../projects/internproj/project1_app/archive/pairing-sharing-research-2026-10-12.md).

## Phần còn mở
- Manager có gỡ được Employee không (agent khuyên: chỉ người do chính Manager đó thêm vào).
- Bản sao quyền hết hạn mà máy vẫn offline thì LAN/BT làm gì (agent khuyên: chỉ xem).
- Owner đứng cạnh máy tạm khóa máy qua BT khi offline (agent khuyên: cho, vì chỉ thu hẹp quyền).
- Nhật ký theo người nằm ở đâu (agent gợi ý: máy ký nhật ký, đẩy qua `MACHINE_EVENT_SEND`).
- Hộp bị mở trước khi tới tay chủ; quy trình tranh chấp; Owner xóa tài khoản chưa chuyển chủ.
- BLE phone-to-phone giữa iOS và Android cần thử trên máy thật.
- Doc vẫn ở `research/`; chưa chuyển sang `architect/`. Chưa tạo file config trong code.
