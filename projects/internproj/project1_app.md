---
id: project-project1-app-map
type: project-map
scope: project:project1_app
status: source-observed
checked: 2026-10-13
anchor_commit: 0ec3a56
source: /home/abel/project/internproj/project1_app
---

# `project1_app`: điểm vào (tầng 1)

App Flutter (Android) và server Python trung gian để quản lý máy pha FlexMix: tài khoản, đăng ký/chia sẻ máy qua QR hoặc Bluetooth, menu, kho. Repo Git riêng ở `/home/abel/project/internproj/project1_app`. `internproj/AGENT.md` yêu cầu tập trung repo này, **không đụng `project1_server` khi chưa được phép**.

## Đọc gì cho câu hỏi nào (chỉ mở đúng một file)

| Câu hỏi hoặc task | Mở |
|---|---|
| "Tới đâu rồi?", "Đã làm X chưa?", "Làm gì tiếp?" | [STATUS.md](project1_app/STATUS.md). **Không quét code** |
| Sửa hoặc thêm tính năng, cần biết file, route, gói tin, DB | [ARCHITECTURE.md](project1_app/ARCHITECTURE.md) |
| Chạy test, test đỏ vì sao | [TESTING.md](project1_app/TESTING.md) |
| Lỗ hổng, thiết kế crypto đã chốt | [SECURITY.md](project1_app/SECURITY.md) |
| Giả định tấn công khi tự chuyển online → LAN → Bluetooth (v1.1, chưa chốt) | [OFFLINE_THREATS.md](project1_app/OFFLINE_THREATS.md) |
| Middleware, plan G0–G7, quy trình team AI | [MIDDLEWARE.md](project1_app/MIDDLEWARE.md) |
| Cần nguyên văn báo cáo, transcript, log cũ | [archive/README.md](project1_app/archive/README.md), rồi `rg` đúng file |

Tài liệu HTML trong `documentation/architect/`, `documentation/research/` và `documentation/plan/` dùng chung `documentation/assets/`: `doc.css`, `doc.js`, `sequence-layout.js`, `fonts.css` và `fonts/` cục bộ (kiểm nguồn 13/10/2026, working tree chưa commit tại HEAD `0ec3a56`). `documentation/serve_architect.py` phục vụ gốc `documentation/` để cả ba nhánh cùng truy cập asset. Hướng dẫn chạy server trên Linux ở `server/SETUP_LINUX.md`.

## Trạng thái một dòng (12/10/2026)

HEAD `0ec3a56`, working tree sạch trước review 12/10/2026. Skeleton v2 đã có tài khoản, claim BLE, relay, menu/kho qua máy ở `server/`, `flutter_app/`, `version1.1/machine/`; nghiệm thu Android/Pi/HTTPS/Gmail thật còn mở. Xem [STATUS.md](project1_app/STATUS.md) để lấy tiến độ; [ARCHITECTURE.md](project1_app/ARCHITECTURE.md) có bản đồ code hiện hành, phần code cũ giữ làm lịch sử.

2026-10-11 (chưa commit): đã **xóa hẳn `version1.1/admin_gui/`** và `check_permissions.py` theo lệnh user. `version1.1/database/` tổ chức lại thành `core/` (config, connection, helpers), `schema/` (schema.sql, seed.sql, apply.py), `services/` (inventory, order_ticket, error_log, recipe_export); bỏ mọi CLI của database (`database.main`, CLI của order_ticket/error_log). Hai hàm kho mà `machine/ingredient_sync` cần (`list_ingredients`, `refill`) nằm ở `database/services/inventory.py`. store_gui không còn mở API quản trị; `database.sql` drop `admin_user` và `role_permission`; `install.sh` bỏ phase account. Từ nay mọi chức năng quản trị phải đi qua app + server mới.

## Kiểm độ tươi trước khi tin note

`git -C /home/abel/project/internproj/project1_app log --oneline -1`. Nếu khác `0ec3a56`, đọc diff từ commit neo rồi đối chiếu phần liên quan.

## Quy ước cứng của repo (đọc nguồn khi sửa code)

Repo hiện không có `AGENTS.md` hoặc `MODULE_PATTERN.md` (kiểm 12/10/2026). Route tập trung ở `server/configuration/routing.py`; quy ước cũ về `MODULES` thuộc code lịch sử, không áp vào skeleton mới. Test Python ở `tests/`, Flutter ở `flutter_app/test/`. Đổi hợp đồng gói tin phải kiểm app, server và máy cùng lúc.
