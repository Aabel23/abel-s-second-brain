---
id: session-20261011-iot-system-design-tier1
type: session-learning
scope: project:project1_app
status: extracted
date: 2026-10-11
source: current-conversation
---

# Mục tiêu và chuẩn đạt
User muốn xây lại hệ IoT quản lý máy pha FlexMix (`project1_app/version1.1`): máy ↔ server trung gian ↔ app. App cần đăng nhập và phân quyền. Nhân viên xem đơn, nạp nguyên liệu, nhận máy được giao. Chủ/quản lý xem thống kê, nhận máy mới, chia sẻ máy bằng QR hoặc Bluetooth. Phiên này chạy **tầng 1 của hiến pháp** (bàn qua chat, chưa kỹ thuật, chưa file, chưa code). User muốn mỗi câu chốt có **4 option** và research song song bằng agent.

Repo lúc này: `project1_app` HEAD `7bd7f52`. Hai commit `28febfb` và `7bd7f52` đã xoá code app/server và tài liệu cũ để thiết kế lại; chỉ còn `documentation/` và `version1.1/`.

## Những lần chỉnh hướng
- Agent đề xuất tầng "cửa hàng" và mô hình chùm chìa khoá rộng. User thu gọn: chủ nhận máy trước, bàn giao bằng QR/BT, sau đó **không cần QR/BT trong công việc hằng ngày**. Lần sau: bắt đầu từ mô hình tối thiểu user nêu, đừng mở rộng trước.
- Agent đề xuất claim bằng mã xoay trên màn hình. User chỉ lỗ hổng: người đứng trước máy trước vẫn cướp được. Agent research định danh chủ doanh nghiệp và gắn thiết bị qua kênh bán (Toast, Meraki, Apple ABM), rồi đề xuất gán chủ lúc bán. User chỉ tiếp: **bán qua đại lý thì không hợp**. Agent đề xuất chuỗi bàn giao hãng → đại lý → chủ.
- User chọn **mô hình kiểu iCloud / Activation Lock** (kích hoạt lần đầu tại máy thì khoá vào tài khoản, khoá nằm ở server), giữ luồng nhận máy đã bàn, và yêu cầu dò lỗ hổng. Agent liệt kê 8 lỗ hổng.
- Agent nói danh tính máy dễ xoá khi cài lại. User sửa: **dùng serial CPU làm định danh**. Agent nhận sai phần "xoá danh tính", nhưng giữ điểm: serial là tên, chưa phải bằng chứng (vẫn giả danh được).
- Agent giả định luật "mỗi máy đúng một chủ". User sửa: **không có luật đó; mô hình là Super Admin / Admin / Staff**. Lần sau: không áp luật sở hữu đơn nhất khi user chưa chốt.

## Quyết định và giới hạn
- [user-confirmed] Ba khối máy / server / app. App và máy giao tiếp qua server trung gian.
- [user-confirmed] Đăng ký trong app: thông tin, mật khẩu, email → OTP qua mail để xác nhận. Nhân viên đăng ký giống chủ.
- [user-confirmed] Chủ nhận máy mới trước, sau đó bàn giao cho nhân viên bằng QR hoặc Bluetooth. Đã nhận máy rồi thì làm việc không cần QR/BT.
- [user-confirmed] Một máy có nhiều nhân viên. Ai được share là do chủ chỉnh trong chức năng quản lý user.
- [user-confirmed] App được tắt món, sửa menu, khoá máy, nhưng phải quy định rõ trong phân quyền.
- [user-confirmed] User thích phân quyền chặt vì có thông tin và quyết định cần quyền cao. **Phiên bản này thay thế hoàn toàn `admin_gui`.** Mô hình nhóm quyền của `version1.1/admin_gui/permissions.py` là tham khảo.
- [user-confirmed] Nhận máy theo mô hình iCloud/Activation Lock, giữ luồng nhận máy tại máy đã bàn.
- [user-confirmed] Định danh máy = **serial CPU**.
- [user-confirmed] Vai: **Super Admin / Admin / Staff**. Không có luật mỗi máy một chủ.
- [source-observed, agent báo, chưa tự kiểm] `version1.1/machine/pairing/` gửi `product_key` tĩnh cho bất kỳ ai hỏi qua Bluetooth; agent BlueZ chấp nhận mọi ghép đôi (Just Works); `machine/server_connection/` gửi khoá đó trong mọi request. Đã ghi để bàn, **chưa sửa**.
- [source-observed, agent báo] `version1.1/agent/routing.py` đã có hình hài tách dữ liệu (menu + ack đi xuống; orders, stock, errors đi lên). `order_ticket` lưu giá/tên lúc bán.

## Sản phẩm hiện hành
- Chưa có file thiết kế. Toàn bộ nằm trong chat.
- Memory Claude: `~/.claude/projects/-home-abel-project-internproj/memory/v11-app-replaces-admin-gui.md`.
- Kết quả 5 agent research (Opus 5.5 medium): [archive/iot-design-research-2026-10-11.md](../projects/internproj/project1_app/archive/iot-design-research-2026-10-11.md).

## Phần còn mở (chờ user chốt)
- Câu 1, đăng nhập khi app thay admin_gui: A chỉ app / **B app + đăng nhập tại máy bằng QR (agent khuyên)** / C PIN offline / D chìa khẩn cấp. Agent đề xuất thống nhất một chiều "máy hiện mã, app quét" và xác nhận lại (step-up) cho thao tác quyền cao.
- Câu 3, dữ liệu: agent khuyên **D tách theo loại** (server giữ cấu hình mong muốn, máy báo đơn/tồn kho/nạp kho, sổ sự kiện chỉ ghi thêm). Phụ: 3a máy offline không khoá ngay được, có tự khoá sau N ngày không; 3b nạp kho ghi tại máy.
- Phân quyền: theo từng máy (agent khuyên A1, sẵn đường lên A3 cửa hàng/doanh nghiệp) hay theo tổ chức. Ánh xạ Super Admin = chủ hay = hãng. Super Admin đá nhau: a ngang vai không đụng nhau (khuyên, giống `outranks`) / b người sáng lập / c xoá có thời gian chờ.
- Lỗ hổng mô hình iCloud: ① kích hoạt trước (chế độ vận chuyển + mã niêm phong + ân hạn); ② máy offline vẫn bán (tự khoá sau N ngày); ③ serial CPU chỉ là tên → bí mật đi kèm a lần đầu / **b hãng cấp lúc xuất xưởng (khuyên)** / c chip bảo mật; thay bo mạch do hãng chuyển; ④ sổ xuất kho serial → đại lý; ⑤–⑧ mất tài khoản, máy cũ dính khoá, lừa chuyển máy, nhân viên lạm quyền.
- Kênh bán: hãng bán thẳng, qua đại lý, hay cả hai. Đại lý sau khi bán còn quyền bảo trì không.
- Sau khi chốt: bước 2–3 hiến pháp (operator đặc tả khối, HTML sơ đồ theo chuẩn FexMix_Munual).

## Phần 2: tạo tài khoản chủ (sau khi user kéo về trọng tâm)
- [user-confirmed] User nói phiên đang lạc đề, yêu cầu tập trung vào "làm sao tạo tài khoản cho chủ". Lần sau: giữ một câu hỏi trọng tâm, đừng mở nhánh bảo mật hay kênh bán khi user chưa hỏi.
- Agent đưa 4 cách: 1 kích hoạt máy là chủ / 2 tạo doanh nghiệp là chủ (kiểu Slack, Shopify) / 3 hãng tạo sẵn (kiểu Toast) / 4 ai cũng là chủ không gian riêng. [user-confirmed] **Chọn 2**, kèm thu thập thông tin doanh nghiệp lúc đăng ký để hỗ trợ.
- [user-confirmed] **Tạo doanh nghiệp là tùy chọn**: chỉ cần khi muốn quản lý nhân sự, v.v. Tài khoản cá nhân vẫn tự kích hoạt và dùng máy.
- [agent-proposed, chưa chốt]
  - Danh sách thông tin: bắt buộc = tên, loại hình, SĐT, tỉnh/thành; nên có = địa chỉ, người đại diện, mã số thuế, mô hình kinh doanh; theo máy = địa chỉ đặt máy.
  - Bảng chia chức năng: cá nhân không có bàn giao, phân quyền, nhật ký. Hai ô còn bỏ ngỏ: thống kê gộp, hãng hỗ trợ.
  - Lúc tạo doanh nghiệp, hỏi "chuyển máy cá nhân nào vào".
  - Khối mới: màn hình nội bộ của hãng, chỉ xem.
  - Bước đồng ý xử lý dữ liệu cá nhân (NĐ 13/2023, Luật BVDLCN).
  - Rủi ro "mượn tài khoản" nếu tạo doanh nghiệp nặng nề.
  - Một tài khoản nhiều vai, cần chuyển ngữ cảnh.
- Câu mở: một tài khoản tạo được nhiều doanh nghiệp không; các ô "?" trong bảng chức năng; có thêm khối màn hình hãng không.
- Đã cho chạy 2 agent research hướng khác (POS/F&B Việt Nam; danh tính theo SĐT, không mật khẩu, nhân viên không cần tài khoản đầy đủ) và 1 agent review thiết kế. Kết quả: [archive/account-research-review-2026-10-11.md](../projects/internproj/project1_app/archive/account-research-review-2026-10-11.md).

## Phần 3: chốt sau research và review
- [user-confirmed]
  - Danh tính đăng ký **giữ email** (bác SĐT).
  - Nhân viên **tự đăng ký + bàn giao** (bác chủ tạo hộ hay PIN).
  - Đồng ý **doanh nghiệp ngầm**.
  - Bàn giao **cả QR lẫn Bluetooth** ở bản đầu (bác lời khuyên chỉ làm QR).
  - Vai **Owner / Manager / Employee**.
- Lần sau: user giữ lựa chọn email dù research khuyên SĐT. Đừng đề xuất lại SĐT nếu không có lý do mới.

### Ghi chú triển khai "doanh nghiệp ngầm" (user yêu cầu ghi để tầng 2 rõ hơn; chưa chốt chi tiết)
- Tạo doanh nghiệp ngầm ngay khi tài khoản xác nhận OTP xong. Người dùng không thấy chữ "doanh nghiệp".
- Doanh nghiệp có hai trạng thái: **cá nhân** (1 người, không có quản lý nhân sự) và **đã nâng cấp** (có tên, thông tin, mở khoá bàn giao, phân quyền, nhật ký).
- Kích hoạt máy luôn gắn máy với doanh nghiệp của người đang kích hoạt (ngữ cảnh hiện tại), không gắn với cá nhân. Sửa được lỗi C2 của review.
- Nâng cấp không chuyển máy nào, chỉ đổi trạng thái và thêm thông tin. Dữ liệu máy đi theo máy.
- Khi làm việc ở doanh nghiệp người khác: tài khoản là *thành viên* với vai Manager/Employee. App có chỗ chọn ngữ cảnh (doanh nghiệp của tôi / doanh nghiệp X).
- Câu tầng 2 phải trả lời:
  - xoá tài khoản thì doanh nghiệp ngầm và máy đi đâu;
  - doanh nghiệp đã nâng cấp có được hạ về cá nhân không;
  - một tài khoản được nâng cấp thêm doanh nghiệp thứ hai không;
  - tên hiển thị mặc định của doanh nghiệp ngầm;
  - luật Owner gỡ Owner (C1, còn mở);
  - giữ "luôn ≥1 Owner".

## Phần 4: đổi kiến trúc sở hữu cho MVP
- [user-confirmed] Bỏ bước "tạo doanh nghiệp". **Người kích hoạt máy đầu tiên mặc định là Owner.** Share máy lần đầu thì bật phân quyền.
- [user-confirmed] Phạm vi **theo từng máy (A)** cho MVP. Đã bác: B (doanh nghiệp ngầm ẩn, agent khuyên), C (nút share tất cả), D (nhóm máy).
- [user-confirmed] **Bỏ thu thập thông tin doanh nghiệp** khỏi bản đầu (d).
- [user-confirmed] Người được share **mặc định là Employee**.
- [user-confirmed] Đổi vai theo **b**: Owner nâng tới Manager; Manager chỉ share thêm Employee, không nâng vai. Bác a (chỉ Owner, nâng được cả lên Owner), c (không đổi vai), d (nâng tới vai thấp hơn mình).
- [user-confirmed] **Mỗi máy 1 Owner** + **chuyển quyền sở hữu ngay trong MVP**: Owner cũ giao máy rồi xuống Manager. Bác: 1 Owner không có chuyển (agent khuyên), nâng thêm Owner, hoãn sang bản sau.
- [user-confirmed] Người được share/phân vai **không cần đồng ý, không cần thao tác gì**: được phân là có hiệu lực ngay. (Agent hiểu áp dụng cả khi nhận chuyển quyền Owner; user chưa nói riêng.)
- Câu tầng 2 phát sinh: có cần xác nhận lại (OTP/mật khẩu) khi chuyển không; Owner xoá tài khoản mà chưa chuyển thì máy ra sao.
- Mục "Ghi chú triển khai doanh nghiệp ngầm" ở Phần 3 hết hiệu lực cho MVP. Câu còn mở chuyển sang theo máy: luật Owner gỡ Owner trên một máy, giữ "luôn ≥1 Owner" mỗi máy, xoá tài khoản thì máy mình làm Owner đi đâu.
- Lần sau: user ưu tiên phương án tối giản cho MVP dù agent khuyên phương án mở rộng hơn.

## Phần 5: đăng nhập tại máy và dữ liệu
- Agent đưa 8 cơ chế (chỉ app; QR mở màn hình máy; QR chứng minh có mặt rồi thao tác trên app; Bluetooth gần máy; PIN; mã một lần; chìa vật lý; xin phép từ xa). Agent khuyên 1+3 và chấp nhận mất mạng thì không quản trị.
- [user-confirmed] **Bác** việc chấp nhận mất mạng. **Dữ liệu món, nguyên liệu local trên máy**, vì thiết bị IoT đã làm vậy từ lâu. Không có wifi thì dùng **Bluetooth** (user đề xuất).
- Lần sau: user coi vận hành offline là yêu cầu gốc của IoT; đừng đề xuất bỏ offline để đơn giản hoá.
- Agent khuyên "việc cần có mặt luôn qua Bluetooth" (B). [user-confirmed] **Bác vì cồng kềnh**: Bluetooth chỉ là **tùy chọn** khi mất mạng.
- [user-confirmed] **Server = gateway giữa máy và app, không chứa gì.** Agent nêu xung đột: tài khoản/OTP và khoá kích hoạt kiểu iCloud (đã chốt "khoá nằm ở server") cần chỗ lưu; máy offline thì app không xem được gì; tin nhắn tới máy offline không có chỗ chờ. [user-confirmed] Server **chỉ giữ đăng nhập và danh sách máy**. Còn mở: "danh sách máy" gồm những gì (chủ máy/khoá kích hoạt, ai được share, vai?).
- Hệ quả agent nêu, chờ chốt: máy phải tự giữ danh sách người và vai để xác thực khi offline; thu hồi quyền chỉ tới máy khi máy có mạng hoặc có người của máy kết nối.


## Phần 6: phân quyền và offline, chế độ online/LAN/BT, kích hoạt kiểu Matter
Phạm vi: phần hội thoại agent thấy được trong phiên Claude Code này (sau phần 5).

### Những lần chỉnh hướng
- Agent hỏi "bản nào là bản chuẩn" của danh sách quyền. User định nghĩa: bảng user + bảng máy, mỗi dòng một người trên một máy. Rồi đề xuất mức nền ai cũng có (xem hàng chờ, món đang chạy, kho, nạp kho).
- User yêu cầu **cử agent tham khảo thế giới, quên context thiết kế hiện tại**, và đưa 4 option. 3 agent research (Opus 5.5 medium, prompt chỉ có bài toán chung): [archive/permission-offline-research-2026-10-11.md](../projects/internproj/project1_app/archive/permission-offline-research-2026-10-11.md).
- User đổi ý nhiều vòng: Option 1 → "phụ thuộc online, máy không giữ bản sao" → mô hình LAN, server ở quán → quay lại Option 1 "vì an toàn hơn" → hỏi đặt máy làm server LAN → agent khuyên không (mất Activation Lock, thành Option 3) → chốt server hãng + tự chuyển chế độ.
  Lần sau: khi user đổi hướng, nêu ngay hệ quả với các quyết định đã chốt (agent làm vậy và giúp user tự quay lại). Ghi vault ngay sau mỗi lần chốt và đánh dấu bản bị thay.
- Chống phá trước khi nghỉ: user nói "chỉ phân quyền, còn lại là pháp luật", agent nêu cần nhật ký làm bằng chứng, user chốt B + C.
- Kích hoạt: user muốn lần ghép đầu dùng Bluetooth cấp luôn wifi, **yêu cầu đề xuất từ case study** → agent đưa Matter / Espressif-Tuya / SoftAP / dây mạng (viết từ hiểu biết sẵn có, chưa agent kiểm). Mã niêm phong: user chốt "một lần" rồi đổi thành "cố định, coi là mã định danh"; agent tách định danh (serial, công khai) khỏi mã (bí mật), user chọn A.
- User hỏi lại "1 hả" khi câu hỏi viết tắt. Lần sau: mỗi câu hỏi kèm một tình huống ví dụ cụ thể.
- User muốn 4 option mỗi câu, ngắn; trả lời một chữ cái.

### Quyết định [user-confirmed]
Ghi đầy đủ trong [facts/project1-app-v11-iot-design-confirmed.md](../facts/project1-app-v11-iot-design-confirmed.md). Tóm tắt:
- Option 1: server bản chuẩn (bảng user + bảng máy), máy giữ bản sao có hạn dùng, vai cố định, không chỉnh quyền từng người.
- Server của hãng (cloud). Mất Internet tự chuyển LAN, mất wifi chuyển BT; âm thầm, không hiện dấu hiệu. Máy không làm server.
- Nguyên tắc vàng: LAN/BT không bao giờ mạnh hơn online.
- B + C: offline chỉ việc vận hành; nhật ký theo người, không xoá được, báo chủ việc bất thường.
- Tắt/bật món là việc vận hành (user "ok", nên xác nhận lại).
- Kích hoạt: mã niêm phong theo hộp (B), luồng kiểu Matter (A); mã cố định, bí mật, chỉ tác dụng khi máy chưa có chủ; định danh vẫn serial CPU; lần đầu bắt buộc online.
- Đổi wifi: Owner/Manager qua BT, làm được offline.
- Bán lại: chủ cũ gỡ máy → người mua dùng lại mã niêm phong.

### Sản phẩm
- [projects/internproj/project1_app/OFFLINE_THREATS.md](../projects/internproj/project1_app/OFFLINE_THREATS.md): 8 tình huống tấn công khi tự chuyển chế độ, nguyên tắc vàng, quyết định B+C, ẩn chế độ. User yêu cầu đặt trong vault (bản trong repo đã xoá).
- HTML tầng 1 theo chuẩn FexMix: `project1_app/documentation/design/iot-v11-system-design.html` (toàn cục, kích hoạt kiểu Matter, 3 chế độ, vai và quyền, vòng đời máy, nguyên tắc bảo mật, còn mở). Không chứa 8 tình huống tấn công (giữ trong vault theo ý user). Đã chụp kiểm 1400/420 px.
- [source-observed] `project1_app/version1.1/admin_gui/` đã không còn (khớp tầng 1 note project).

### Phần còn mở
- Kích hoạt: đường dự phòng hãng xử lý tranh chấp (hoá đơn + chờ); chuyển chủ có cho chủ cũ rời hẳn (agent khuyên có cả hai).
- Owner mất/xoá tài khoản thì máy đi đâu.
- Gỡ người tại chỗ qua BT: vướng nguyên tắc vàng, cần user quyết.
- Hỏi lại mật khẩu cho việc nguy hiểm; độ dài hạn dùng bản sao.
- Vé ký (Option 4) làm lớp xác minh BT/LAN và danh tính máy: agent đề xuất, chưa chốt.
- Nhật ký (C) nằm ở đâu khi "server không chứa dữ liệu".
- Chưa bàn: luồng dữ liệu hằng ngày, luồng bàn giao QR/BT.
- Tầng 1 chưa qua cổng: chưa có đặc tả khối và HTML sơ đồ (bước 2–3 hiến pháp).
