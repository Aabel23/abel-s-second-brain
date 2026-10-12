---
id: project-project1-app-map
type: project-map
scope: project:project1_app
status: source-observed
checked: 2026-10-11
anchor_commit: 16dab0e
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

## Trạng thái một dòng (11/10/2026)

HEAD `7bd7f52`: code app/server và docs cũ **đã bị xoá** (`28febfb`, `7bd7f52`) để thiết kế lại hệ IoT. Repo chỉ còn `documentation/` và `version1.1/`. Đang ở tầng 1 hiến pháp; quyết định đã chốt ở [facts/project1-app-v11-iot-design-confirmed.md](../../facts/project1-app-v11-iot-design-confirmed.md). Các note tầng 2 (ARCHITECTURE, TESTING, MIDDLEWARE) mô tả code cũ, chỉ dùng làm tham khảo lịch sử.

2026-10-11 (chưa commit): đã **xóa hẳn `version1.1/admin_gui/`** và `check_permissions.py` theo lệnh user. `version1.1/database/` tổ chức lại thành `core/` (config, connection, helpers), `schema/` (schema.sql, seed.sql, apply.py), `services/` (inventory, order_ticket, error_log, recipe_export); bỏ mọi CLI của database (`database.main`, CLI của order_ticket/error_log). Hai hàm kho mà `machine/ingredient_sync` cần (`list_ingredients`, `refill`) nằm ở `database/services/inventory.py`. store_gui không còn mở API quản trị; `database.sql` drop `admin_user` và `role_permission`; `install.sh` bỏ phase account. Từ nay mọi chức năng quản trị phải đi qua app + server mới.

## Kiểm độ tươi trước khi tin note

`git -C /home/abel/project/internproj/project1_app log --oneline -1`. Nếu khác `16dab0e`, chỉ đọc diff từ commit đó (cách làm trong STATUS), cập nhật note rồi trả lời.

## Quy ước cứng của repo (đọc nguồn khi sửa code)

`AGENTS.md` (thiết kế và quy trình đã khoá), `MODULE_PATTERN.md` (tên `đối_tượng_thành_phần_hành_động`, route tập trung ở `server/configuration/routing.py` (kiểm 12/10/2026; chuẩn cũ ghi `server/config/`), path tiếng Anh `/<app|machine>/<đối_tượng>/<mục_tiêu>/<hành_động>` theo [facts/routing-naming-standard.md](../../facts/routing-naming-standard.md), `MODULES` import trực tiếp, không registry). Test luôn đặt trong `tests/`. Đổi hợp đồng gói tin thì phải sửa app, server và máy cùng lúc.
