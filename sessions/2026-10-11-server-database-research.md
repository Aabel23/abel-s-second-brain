---
id: session-20261011-server-database-research
type: session-learning
scope: project:project1_app
status: extracted
date: 2026-10-11
source: current-conversation
---

# Mục tiêu và chuẩn đạt

User yêu cầu "research thiết kế database cho server" (server hãng trong `project1_app`). Đạt khi: có mô hình dữ liệu ở mức khái niệm (chưa chọn engine), có case study, và có HTML research đọc dễ, dùng **sơ đồ chuẩn của ngành dữ liệu**.

# Những lần chỉnh hướng

- Agent định viết doc bằng kiểu sơ đồ khối/luồng của chuẩn FexMix → user: "phải dùng sơ đồ data của ngành khoa học dữ liệu, nếu dùng doc của ta sẽ khá khó nhìn" → doc về dữ liệu dùng ERD ký hiệu chân chim (khái niệm + logic), state/sequence UML. Fact: [doc-data-diagrams-erd](../facts/doc-data-diagrams-erd.md).
- Agent viết HTML **chưa đối chiếu fact đã chốt trong vault** (skill research-stage bước 5.1 có yêu cầu) → doc lệch 3 quyết định (xem Phần còn mở). Lần sau: đọc các fact `project1-app-*-confirmed` trước khi viết doc của project1_app.

# Quyết định và giới hạn

- [user-confirmed, trả lời "ok" cho 4 khuyến nghị] xem [project1-app-server-database-confirmed](../facts/project1-app-server-database-confirmed.md): server giữ bản sao nhật ký theo người; quyền lưu cả lịch sử; máy offline thì server từ chối lệnh thường, chỉ giữ thông điệp quản trị; lượt research này chỉ ở mức mô hình dữ liệu khái niệm, chưa chọn engine.
- [source-observed] `version1.1/database/schema/schema.sql` của máy có 11 bảng (món, công thức, đơn, `error_log`), ghi "không có migration"; chưa có bảng người/quyền, mã lệnh đã chạy, hộp thư đi, seq. `machine_server_request.py` gọi server từng lần, timeout 10 s, không retry. `mysql_migration/` ở gốc internproj đã trống trong phiên.
- [agent-proposed, chưa chốt] ~14 thực thể: account, session, otp_challenge, machine, machine_state, membership, invite, grant, permission_event, admin_outbox, audit_event (khóa machine+epoch+seq, không FK, chỉ ghi thêm), release, update_run; online và lệnh đang chờ để trong bộ nhớ; chống chạy hai lần ở máy (bảng mã lệnh→kết quả ~24 h); admin_outbox gộp bản (một dòng/loại/máy); xóa tài khoản = ẩn danh hóa giữ dòng account; machine_id UUIDv7, serial là UK.
- Researcher: 3 agent Sonnet medium (A danh tính+quyền, B trạng thái máy+lệnh+cập nhật, C nhật ký+xuyên suốt), không đọc vault, không đọc `project1_server/`. Kết quả: [archive](../projects/internproj/project1_app/archive/server-database-research-2026-10-11.md).

# Sản phẩm hiện hành

- `project1_app/documentation/research/server-database.html` (untracked): 5 mục + chú thích, 12 sơ đồ Mermaid (7 ERD, 4 state, 1 sequence, 1 flow), 31 chú thích; check_render: 0 fallback, không tràn ngang.
- Lệch template chuẩn (agent tự làm, chờ user duyệt): Mermaid 11.4.1 → **11.6.0** (11.4.1 không hiểu `direction LR` trong erDiagram, vẽ thành thực thể); thêm CSS cho `code` dài xuống dòng (figcaption, note, td, steps) để không tràn ngang ở màn hẹp; thêm themeVariables `attributeBackgroundColorOdd/Even` và `er:{useMaxWidth:false}`. Mermaid 11.6 hiểu nhãn bắt đầu bằng `>` là blockquote markdown → tránh.

# Phần còn mở

- **Doc lệch fact đã chốt, chưa sửa:** (1) 3.1 ghi "không giữ mật khẩu, đăng nhập email + OTP" — sai, đã chốt mật khẩu ≥8 + nhập lại mật khẩu cho việc nguy hiểm ([auth fact](../facts/project1-app-auth-identity-confirmed.md)) → `account` cần `password_hash`; (2) hạn bản sao quyền trên máy để là câu hỏi mở — đã chốt 7 ngày; (3) "chuyển chủ: xuống Manager hay rời hẳn" ghi chưa có đề xuất — đã chốt Owner cũ xuống Manager (còn mở: có cho rời hẳn không). Khóa công khai điện thoại (Keystore) nên gắn theo thiết bị/phiên thay vì một cột trên account — cần xem lại.
- Câu hỏi chờ user (ưu tiên 1–5): sự kiện nào là nhật ký; epoch khi máy cài lại; Owner xóa tài khoản; xóa dữ liệu máy khi gỡ/đổi chủ; hạn lưu nhật ký + nơi đặt server (pháp lý chưa xác minh: NĐ 13/2023 hay luật BVDLCN mới); ẩn danh hóa; chuỗi băm MVP hay sau; machine_id vs serial; ngưỡng offline + hạn outbox 30 ngày; gỡ người offline qua BT.
- Phía máy còn thiếu 4 thứ mô hình cần (bảng mã lệnh, hộp thư đi seq/epoch, nhật ký theo người, bản sao quyền) — ghi trong doc, chưa thiết kế.
- Phát hiện lúc lưu phiên (STATUS, phiên khác ngày 12/10/2026): user đã chốt **mọi database dùng MySQL, bỏ SQLite**. Doc `server-database.html` còn ghi "chưa chọn engine" và tiêu chí chọn engine ở 4.6 — cần cập nhật theo quyết định đó.
