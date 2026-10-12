---
id: liveness-review-2026-10-10
type: agent-report
scope: project:project1_app
date: 2026-10-10
source: 3 subagent (Opus, effort medium), chỉ đánh giá, không sửa code
---

# Đánh giá cơ chế online của máy: kết quả cuối của 3 agent

Bản cô đọng do agent chính ghi lại. Không lưu transcript. Tài liệu tổng hợp nằm ở repo: `docs/heartbeat-vs-long-poll.html` (và `.md`).

## Agent 1: bỏ hẳn heartbeat (đọc code)
- `is_online()` là **cổng chặn của `send()`** (`machine_transport.py:51-52`, trả 503). Cả 5 lệnh đều qua đây: `nhan_kho`, `nap_kho`, `nhan_menu`, `cap_nhat_menu`, `nhan_anh`.
- Thread heartbeat độc lập với vòng poll (`version1.1/machine/main.py:56`). Vòng treo vẫn báo online. Heartbeat chứng minh "process sống", không chứng minh "nhận được lệnh".
- Máy cũ gặp 404 không crash: `HTTPError` là lớp con của `URLError`, bị bắt ở `machine_server_heartbeat.py:29`. Ngược lại, máy mới gặp server cũ thì mọi lệnh 503, nên **phải nâng server trước**.
- Tải lúc rảnh: 0,325 → 0,125 req/s mỗi máy (−62%). Mỗi heartbeat tốn 1 thread, 1 SHA-256 và 1 truy vấn SQLite.
- Test vỡ:
  - S1: `test_server` dòng 78-79, 96, 120, 143, 182, 195.
  - X1: dòng 262 (route heartbeat 404), 163-178.
  - **SEC-02 (186) và SEC-08 (290) `expectedFailure` sẽ thành unexpected success**, tức pass giả.
  - S11: `test_machine_menu_image.py:56`.
  - S12: `harness.py:175` (`self.machine.heartbeat`).
  - S8: relay chờ 5 s.
  - E2E và Flutter không vỡ.
- Docs: khoảng 18 file nhắc heartbeat, gồm `FlexMix_System_Manual.html` (11 dòng), `machine_link/README.md`, các `*_FLOW.html`.
- Kết luận: **nên bỏ, có điều kiện**:
  - Định nghĩa lại online: poll đang mở, hoặc poll/result cuối dưới W, hoặc lệnh đã giao chưa quá `COMMAND_TIMEOUT`.
  - Sửa `send()` để máy bận vẫn được coi là online.
  - Triển khai 2-3 pha.
  - Sửa test trước.

## Agent 2: gộp, A = poll đánh dấu sống, B = online vào list (đọc code)
- `DANG_CHO = {lenh_id: (machine_id, queue)}`, nên không tra theo máy được. `send()` ghi vào đó **trước khi** máy lấy lệnh (`:55-58`). Vì vậy dùng `DANG_CHO` làm điều kiện online thì máy chết vẫn "online", và request chờ 20 s thay vì 503. Bắt buộc chỉ tính lệnh **đã take**, kèm chỉ mục theo `machine_id`.
- `mark_seen` đặt lúc poll **nhận** request, trước `take()`, không đặt bên trong `CO_LENH` (KHOA không reentrant). Không mark lúc poll trả về.
- App **không refresh định kỳ**. `refreshStatuses()` (`dashboard_controller.dart:106-129`) chỉ chạy khi mở dashboard, chọn máy hoặc kéo làm mới. Lỗi status bị đổi thành offline.
- Chi phí ước lượng:
  - A: khoảng 15 dòng code và 50 dòng test.
  - B: khoảng 15 dòng server, 50 dòng app và 150 dòng test.
- B gồm hai bước:
  - B1: thêm trường `online`, không vỡ app cũ.
  - B2: đóng route. App cũ sẽ hiện mọi máy Offline mà không báo lỗi.
- e2e (`run_e2e.py:157,163`), harness stress và relay test đang dùng route status công khai.
- Thứ tự khuyến nghị:
  1. Commit nhánh ảnh, vì đụng `routing.py`/`routing.dart`.
  2. B1.
  3. B2.
  4. A tối giản.

  Không gộp A và B vào một commit.

## Agent 3: tờ giấy trắng (web, không đọc code)
- MQTT keepalive + LWT, Socket.IO ping/pong, Pusher và ThingsBoard đều coi **mọi lưu lượng** là dấu hiệu sống. Heartbeat riêng (Azure heartbeat pattern) chỉ dùng khi kênh không báo sống đủ nhanh.
- Agent 3 khuyên B+C: giữ heartbeat, `last_seen = max(heartbeat, poll, result)`, thêm trạng thái **busy**, tính online lúc đọc.
  - Lưu `last_seen` vào DB tối đa 60 s một lần (giống cách GitLab làm với `last_activity_at`).
  - Trả `unknown` trong 15 s đầu sau khi server restart.
  - Chống nhấp nháy: chờ trạng thái ổn định 5-10 s rồi mới phát sự kiện.
  - Gộp online vào API list có token. Endpoint không token là BOLA (OWASP API1).
- Nâng cấp lên Redis khi chạy nhiều process. Chuyển sang MQTT broker khi có hàng nghìn máy hoặc cần realtime.

## Tổng hợp sau khi user phản biện
Agent 3 và agent 2 ban đầu khuyên giữ heartbeat. User hỏi "có long-poll liên tục thì khác gì heartbeat", và kết luận chuyển thành: **bỏ heartbeat (B), làm theo pha**. Lý do:
- Khoảng trống lúc máy chạy lệnh được lấp bằng điều kiện "lệnh đã take dưới 20 s".
- Lệnh dài hơn 20 s đã là 502 với app.

Case study bổ sung, do agent chính tự tìm: Step Functions (`GetActivityTask` + `SendTaskHeartbeat` chỉ khi đang làm task), Temporal (`RecordHeartbeat` từ trong activity) và GitHub Actions runner (long-poll + gia hạn job lock khi đang chạy job). Cả ba ủng hộ mẫu "rảnh thì dùng poll, bận thì heartbeat theo task".
