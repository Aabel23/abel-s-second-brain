---
id: session-20261011-maintainable-architecture-research
type: session-learning
scope: project:project1_app
status: extracted
date: 2026-10-11
source: current-conversation
---

# Mục tiêu và chuẩn đạt
User hỏi: hệ lớn realtime (Netflix, YouTube, Google, Facebook) làm sao sửa, fix bug, bảo trì, debug, tháo một tính năng ra refactor/nâng cấp mà không sửa cả codebase. Muốn deep research, rồi HTML "mô tả trực quan để tôi học, tôi chưa hình dung được". Sau đó đặt câu hỏi cho hệ máy pha: modular monolith hay EDA.

## Những lần chỉnh hướng
- Agent đặt HTML ở `internproj/documentation/research/` và coi là kiến thức chung → user: "không phải tôi nói đây là công việc của project app sao" (`internproj/AGENT.md`: tập trung `project1_app`) → chuyển file vào `project1_app/documentation/research/`, thêm mục đối chiếu project1_app. Lần sau: đọc `AGENT.md` và mặc định câu hỏi kiến trúc trong `internproj` là cho `project1_app`.
- Agent đề xuất ngay "ba bên modular monolith + sự kiện server↔máy" và hỏi chốt → user: "nên tìm nhiều kiến trúc hệ thống và quy trình làm việc, bảo trì hơn trước khi chốt" → chạy vòng 2 research (3 agent). Lần sau: trước khi đề nghị chốt kiến trúc, quét rộng nhiều phương án và cả quy trình.

## Quyết định và giới hạn
- [user-confirmed] Lưu research, nguồn, case study vào vault để định hướng (xem [facts/save-research-sources-to-vault.md](../facts/save-research-sources-to-vault.md)).
- [user-confirmed] Chưa chốt modular monolith hay EDA; research rộng hơn trước.
- [source-observed] `project1_app` HEAD `c34d55e`: có lại `flutter_app/` (giao diện theo `lib/feature/<tính năng>/`, ~2.3k dòng), `server/` (mới `configuration/routing.py`), `version1.1/machine/` (menu_sync, ingredient_sync, pairing, server_connection).

## Sản phẩm hiện hành
- HTML: `project1_app/documentation/research/maintainable-large-systems.html` (untracked): 4 lớp, luồng viết lại tính năng, 8 thành phần, chọn theo quy mô, 15 case study, mục 7 đối chiếu project1_app, 71 chú thích. Kiểm render: 12 sơ đồ, 0 lỗi, không tràn ngang.
- Kiến trúc (bản nháp chờ duyệt): `project1_app/documentation/architect/system-architecture.html`. User yêu cầu "viết architect cho project, có cả repo đề xuất, phân chia"; agent viết theo hướng hội tụ của vòng 2, mọi nguyên tắc trừ "máy là nguồn sự thật" gắn nhãn đề xuất. Kiểm render: 8 sơ đồ, 0 lỗi, không tràn ngang.
- Research và nguồn: [archive/maintainable-architecture-research-2026-10-11.md](../projects/internproj/project1_app/archive/maintainable-architecture-research-2026-10-11.md).

## Phần còn mở
- Vòng 2 research xong, đã ghi vào archive; chưa đưa vào HTML. Đề xuất hội tụ (chưa chốt): ba bên modular monolith, máy là nguồn sự thật, cloud mỏng có shadow nhẹ, envelope chung có phiên bản cho cloud/LAN/BT, sự kiện chỉ để thông báo và đồng bộ bù.
- Câu hỏi chờ user: hợp đồng app/server/máy có phiên bản (đổi quy ước "sửa cả ba cùng lúc"); OTA phần mềm máy vào thiết kế tầng 1; tách nhật ký kiểm toán khỏi log debug; modular monolith / EDA / tổ hợp.
