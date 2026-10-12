---
id: session-20261009-menu-crc-image-cache
type: session-learning
scope: project:project1_app
status: extracted
date: 2026-10-09
source: current-conversation
---

# Mục tiêu và chuẩn đạt
User muốn hiểu CRC trong hệ thống, cách đồng bộ menu, rồi:
1. Thiết kế giao thức đưa ảnh món từ máy sang app.
2. Thử phương án B trên một nhánh riêng.
3. Có tool stress-test luồng CRC menu để dùng lại về sau.

Chuẩn đạt mà user nêu cho tool:
- Thao tác ở quyền quản lý (manager), đổi menu liên tục: giá, tên ngẫu nhiên, trạng thái bán.
- Một bên chạy luồng CRC bằng **code hiện tại, không gian lận**. Bên cuối tự tính bằng hàm riêng.
- Lặp vài trăm menu, trả về menu gốc, thống kê pass/fail.

## Những lần chỉnh hướng
- Agent kết luận "không có CRC32, chỉ có CRC-16" vì chỉ grep `internproj/version1.1/`. Thực tế CRC32 nằm ở bản `project1_app/version1.1/machine/menu_sync/` và `project1_app/server/.../menu_sync`. **Lần sau:** `internproj` có nhiều bản `version1.1`, phải grep cả ba repo trước khi kết luận "không có".
- User nghĩ CRC là mã định danh duy nhất của menu. Agent đã giải thích: CRC dùng để phát hiện lỗi hoặc thay đổi, không phải để định danh (CRC-16 chỉ có 65.536 giá trị).
- User đề xuất chỉ so các trường quan trọng (tên, giá, ảnh) và bỏ trường logic. Agent phản biện: trường logic như `drinkId`, `orderable`, `options` lại quan trọng nhất. Chỉ nên dọn khỏi output các trường trang web không đọc. User không chốt phương án nào, chuyển sang chủ đề ảnh.
- Agent đề xuất làm cả phía máy. User sửa: "version1.1 là phía machine, tôi không quan tâm, cái tôi quan tâm là giao thức để cập nhật gói tin" → [facts/project1-app-protocol-focus.md](../facts/project1-app-protocol-focus.md).
- User muốn lấy dữ liệu "trực tiếp từ db hiện tại" nhưng MySQL không chạy và DB SQLite của máy rỗng (0 byte). Agent dùng seed `database.sql` chạy qua hàm thật và ghi rõ giới hạn đó.
- Agent định giao việc bằng Agent tool. User từ chối và chọn "ultracode": chạy Workflow nhiều agent Opus effort low.

## Quyết định và giới hạn
- [user-confirmed] Chỉ tập trung giao thức/gói tin app ↔ server ↔ máy. Không triển khai phía máy khi chưa được yêu cầu.
- [user-confirmed] Tool test luồng phải chạy code thật, oracle độc lập, có thống kê pass/fail và khôi phục menu gốc → [facts/test-tool-no-cheating.md](../facts/test-tool-no-cheating.md).
- [user-confirmed] Thử phương án B (lệnh máy riêng + cache ảnh trong app theo hash) trên nhánh riêng.
- [source-observed] CRC trong hệ thống:
  - CRC-16/CCITT-FALSE, in ra 5 chữ số thập phân, bảo vệ payload QR ở `version1.1/qrproto/crc.py`. Có hai lớp: OCRC và ICRC.
  - CRC32 dùng làm `menu_version` app ↔ máy: `zlib.crc32` trên JSON gọn của mọi cột `FIELDS` (`project1_app/version1.1/machine/menu_sync/machine_menu_pack.py`).
  - `store_gui/sync_menu.py` không dùng CRC: so JSON menu đã bỏ `generatedAt` (`menu_body`), chỉ ghi file khi khác.
- [source-observed] Seed `database.sql` có công thức (recipe) dùng `drink_id` 1–5, trong khi món có id 1001–1005. Có thể lệch sau `migrate_id_ranges`, chưa kiểm.

## Sản phẩm hiện hành
Nhánh `feature/menu-image-cache` của `project1_app`, tách từ `refactor/server-modules` @ `07a54d0`, **chưa commit**.

Giao thức ảnh (phương án B):
- Gói `menu_sync` thêm cột `image_hash` (CRC32 byte ảnh, 0 = không ảnh). Không tăng `v`, vì app đọc gói theo tên cột.
- Route mới `POST /app/machine/image/get` (`MACHINE_IMAGE_GET` / `machineImageGet`), body `{token, machine_id, anh:[{drink_id,image_hash}] ≤20}`, gửi lệnh máy `nhan_anh`.
- Máy trả `{status, anh:[{drink_id,image_hash,mime,data}|{drink_id,loi}], con_lai}`, mỗi lượt khoảng ≤700 KB ảnh gốc (giới hạn body result 1 MB).

Thay đổi theo phía:
- Server: `server/service/dashboard_sync/menu_sync/machine_menu_image.py`.
- App:
  - `lib/feature/machine_menu/machine_menu_image.dart`: `MenuImageCache` lưu ở `systemTemp/menu_images/<machine>/<hash>.img`, tự kiểm CRC32 trước khi lưu, xử lý `con_lai`.
  - `Drink.imageHash`.
  - Tab hiển thị `Image.file`, lỗi thì quay về `InitialsTile`.
- Phía máy **chưa làm**: máy chưa gửi `image_hash` và chưa có lệnh `nhan_anh`.

Test:
- `tests/python/test_machine_menu_image.py` (S11) và `tests/flutter/machine_menu_image_test.dart` (chưa đăng ký trong `run_tests`).
- Tool `tests/tools/menu_crc_stress/` (`harness.py`, `oracle.py`, `scenario.py`, `__main__.py`) và wrapper `tests/python/test_menu_crc_stress.py` (S12).
  - Lệnh chạy: `python3 -m tests.tools.menu_crc_stress --iterations N --seed S [--json]`.
  - Kết quả: 300 vòng seed 42 và 500 vòng seed 2026 đều PASS, 354 menu khác nhau, 0 va chạm, khôi phục đúng.

Bài tính mẫu với 5 ảnh seed:
- Tổng ảnh gốc 158.080 B. Một lượt `image/get` có body 211.212 B, bằng 21% của 1 MB.
- `menu_version` đổi từ 2573580648 sang 3773534334 khi thêm cột `image_hash`.

Báo cáo của workflow: [archive/menu-crc-stress-2026-10-09.md](../projects/internproj/project1_app/archive/menu-crc-stress-2026-10-09.md).

## Phần còn mở
- Commit nhánh, nếu user muốn.
- Làm phía máy cho `image_hash` và `nhan_anh`, nếu user muốn.
- Có gộp vào main không.
- Đăng ký test Flutter ảnh vào `run_tests`.
- Tool stress chưa kiểm `image_hash`.
- Dọn file sabotage ở `/tmp/claude-1000/sab/`.
- Chưa chốt có dọn trường thừa (`available`/`inStock` cấp món) trong `store_gui/sync_menu.py` hay không.
