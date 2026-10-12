---
id: project1-app-offline-threats
type: assumption
scope: project:project1_app
status: partly-confirmed
date: 2026-10-11
source: current-conversation (session 2026-10-11-iot-system-design-tier1)
---

# Giả định tấn công khi tự chuyển chế độ online → LAN → Bluetooth

> Trạng thái: **giả định, chưa chốt** (ghi 2026-10-11). Đây là đầu vào cho tầng 2 của khối bảo mật. Các "hướng chặn" chỉ là gợi ý ở mức hệ thống; chọn cơ chế cụ thể là việc của tầng 2.

## Bối cảnh

- Server của hãng giữ bảng user và bảng máy (mỗi dòng là một người trên một máy). Server là **bản chuẩn**.
- Máy giữ **bản sao danh sách quyền có hạn dùng**.
- Mất Internet thì app và máy tự chuyển sang nói chuyện trực tiếp trong **wifi quán (chế độ LAN)**. Mất cả wifi thì dùng **Bluetooth**.
- Việc chuyển chế độ diễn ra **âm thầm, tự động**: user không phải quyết định gì.
- Ở chế độ LAN và Bluetooth, máy kiểm quyền bằng bản sao. Được bán hàng, quản trị theo vai; **không** được share, gỡ người, đổi vai, chuyển chủ hay đăng ký tài khoản mới.

Hệ quả: **kẻ tấn công cũng kích hoạt được việc chuyển chế độ**, chỉ cần cắt Internet hoặc phá wifi.

## Nguyên tắc vàng (user chốt 2026-10-11)

**Chế độ LAN và Bluetooth không bao giờ được mạnh hơn chế độ online.** Ba chế độ dùng chung một cách xác minh danh tính và một mức mã hoá; chỉ đường đi của tin là khác nhau. Ép máy xuống chế độ thấp không đem lại thêm quyền gì.

## Các tình huống

| # | Tình huống | Vì sao xảy ra được | Hướng chặn gợi ý | Câu cho tầng 2 |
|---|---|---|---|---|
| 1 | **Cố tình cắt mạng để dùng quyền cũ.** Kane bị chủ gỡ quyền, liền rút dây Internet của quán, nên lệnh gỡ không tới được máy. | Máy chỉ biết thay đổi quyền khi đồng bộ được với server. | Bản sao có hạn dùng. Nút gỡ người tại chỗ (chỉ gỡ, không thêm). Khi đồng bộ lại, lệnh gỡ luôn thắng. | Hạn dùng dài bao lâu? Hết hạn thì Owner có được ngoại lệ không? |
| 2 | **Máy giả trong wifi quán.** Một thiết bị lạ tự nhận là "máy 1" để lừa app gửi mật khẩu hoặc vé. | Ở chế độ LAN, app nói chuyện trực tiếp với "máy" mà không có server đứng giữa xác nhận. | Máy trình "giấy khai sinh" có dấu của hãng, cấp lúc xuất xưởng. App không bao giờ gửi mật khẩu cho máy. | Hãng cấp và giữ con dấu thế nào? Thay bo mạch thì giấy khai sinh ra sao? |
| 3 | **Điện thoại giả.** Ai đó trong wifi quán, hoặc đứng gần máy qua Bluetooth, tự nhận là Kane. | Không có server thì máy không hỏi được "đây có đúng là Kane không". | Máy kiểm "vé" có dấu của server (ý tưởng Option 4): ghi ai, máy nào, vai gì, hết hạn khi nào. | Vé ngắn hạn tới đâu? Điện thoại đổi vé mới lúc nào? |
| 4 | **Nghe lén wifi quán.** Wifi quán thường dùng chung với khách. | Dễ ngầm coi "trong nhà là an toàn". | Mã hoá mọi kết nối, kể cả trong LAN; không có ngoại lệ. | Có nên tách máy sang mạng riêng của quán không? |
| 5 | **Phát lại tin cũ.** Kẻ tấn công ghi lại lệnh "mở khoá máy" rồi gửi lại. | Tin được mã hoá vẫn có thể bị chép nguyên rồi gửi lại. | Mỗi lệnh chỉ dùng được một lần. | Nhận diện lệnh đã dùng bằng cách nào khi máy offline lâu? |
| 6 | **Chỉnh đồng hồ máy** để vé đã hết hạn thành còn hạn. | Máy kiểm hạn vé theo đồng hồ của chính nó. | Máy không chấp nhận giờ bị lùi so với lần đồng bộ cuối. | Mất điện lâu, đồng hồ máy chạy lại từ đâu? |
| 7 | **Mất điện thoại.** Điện thoại của Kane bị lấy khi vé còn hạn. | Vé nằm trên điện thoại; máy offline không biết điện thoại đã mất. | Vé ngắn hạn. Có danh sách vé bị huỷ do điện thoại người khác mang tới máy. Thu hồi đăng nhập theo từng điện thoại. | Ai được báo mất, và báo từ đâu khi quán offline? |
| 8 | **Ép xuống Bluetooth để tấn công ghép đôi.** Kẻ tấn công phá wifi rồi kết nối Bluetooth với máy. | Bluetooth là đường dự phòng cuối, dễ bị coi nhẹ. | Bluetooth dùng chung vé và mã hoá như LAN; ghép đôi không tự cấp quyền gì. | Xem mục "Hiện trạng" bên dưới. |

## Dấu hiệu chế độ

[user-confirmed 2026-10-11] **Ẩn**: app không hiện dấu hiệu chế độ. Câu cho tầng 2: khi user bấm việc bị khoá lúc offline thì app báo gì.

## Hiện trạng đã thấy trong code cũ (`version1.1`, agent báo, chưa tự kiểm lại)

- `machine/pairing/` gửi một `product_key` cố định cho bất kỳ ai hỏi qua Bluetooth.
- Agent BlueZ chấp nhận mọi yêu cầu ghép đôi (Just Works).
- `machine/server_connection/` gửi khoá đó trong mọi request.

Ba điểm này đi ngược trực tiếp với tình huống 3, 8 và nguyên tắc vàng. Đã ghi lại để bàn, **chưa sửa**.

## Nhân viên phá trước khi nghỉ (bàn 2026-10-11)

- Hai kiểu: (1) phá sau khi bị gỡ quyền, bằng cách cắt mạng để lệnh gỡ không tới máy; (2) phá khi vẫn còn là nhân viên.
- Agent đưa 4 cơ chế: A hạn ngắn theo vai, tự gia hạn; B việc nguy hiểm chỉ làm khi online; C nhật ký theo người, báo chủ việc bất thường; D khôi phục một chạm. Agent khuyên B + C.
- User lúc đầu nói "chỉ phân quyền hợp lý, còn lại là việc của pháp luật", rồi chốt lại ngay trong lượt sau:
- [user-confirmed] **Chọn B + C.**
  - **B:** offline chỉ cho việc vận hành (xem đơn, xem kho, nạp kho). Sửa menu, sửa giá, xoá nguyên liệu, khoá máy bắt buộc online, server kiểm quyền ngay lúc đó.
  - **C:** nhật ký theo từng người, không xoá được, kể cả thao tác offline (máy đẩy lên khi có mạng lại). Thao tác lớn hoặc máy mất mạng bất thường thì báo chủ. Nhật ký cũng là bằng chứng khi cần nhờ pháp luật.
- Không làm A (hạn theo vai, chỉ giữ một hạn dùng chung) và D (khôi phục một chạm) ở bản đầu.
- Câu cho tầng 2: thế nào là "thao tác lớn" và "mất mạng bất thường"; nhật ký giữ bao lâu, ai xem được.
