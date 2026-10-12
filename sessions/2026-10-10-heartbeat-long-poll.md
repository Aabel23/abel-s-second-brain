---
id: session-20261010-heartbeat-long-poll
type: session-learning
scope: project:project1_app
status: extracted
date: 2026-10-10
source: current-conversation
---

# Mục tiêu và chuẩn đạt
User hỏi tiến độ, rồi xét lại cách server biết máy online. Câu hỏi gốc:
- Machine/user có nằm trong DB server không?
- Sao không đồng bộ trạng thái online cùng luồng máy?
- Có nên đưa phiên đăng ký/OTP vào DB không?
- "Có long polling rồi thì đâu cần heartbeat nữa?"

Sản phẩm user yêu cầu:
1. Ba agent ngầm, chỉ đánh giá, không viết code, **cùng dùng Opus effort medium**:
   - Một agent tính chi phí nếu bỏ hẳn heartbeat.
   - Một agent tính chi phí nếu gộp heartbeat vào long-poll và đưa online vào list.
   - Một agent "như tờ giấy trắng", không đọc code, research cách người ta làm.
2. Một trang HTML so sánh hai phương thức: điểm mạnh, điểm yếu, khuyên dùng, có case study nếu tìm được.

## Những lần chỉnh hướng
- Agent (tôi) ban đầu nói heartbeat "còn lý do tồn tại" vì máy chạy lệnh tuần tự, nên không poll khi đang chạy lệnh. Hai agent con cũng kết luận giữ heartbeat. User hỏi lại: "nếu có long polling liên tục thì khác gì heartbeat mà không nên xoá?". Kết luận đảo lại:
  - Khoảng trống duy nhất là lúc máy đang chạy lệnh. Khoảng này lấp được bằng điều kiện "lệnh máy đã take, chưa quá 20 s".
  - "Không nên xoá ngay" chỉ là ràng buộc về thứ tự triển khai, không phải nhu cầu thật.
  - **Lần sau:** tách rõ "ràng buộc rollout" với "nhu cầu bản chất", đừng để cái trước thành lý do giữ một cơ chế.
- Agent đề xuất coi "có lệnh trong `DANG_CHO`" là online. Agent con chỉ ra lỗi: `send()` ghi vào `DANG_CHO` **trước khi** máy lấy lệnh, nên máy chết vẫn "online". Phải dùng "lệnh đã take".
- "lưu md" của user có nghĩa là chạy `/session-secondbrain`, không phải lưu bản .md của tài liệu. Agent đã hiểu sai và tạo thêm file `docs/heartbeat-vs-long-poll.md` → [facts/luu-md-session-secondbrain.md](../facts/luu-md-session-secondbrain.md).

## Quyết định và giới hạn
- [user-confirmed] Chỉ đánh giá và viết tài liệu, chưa sửa code. User **chưa** chốt triển khai phương án B.
- [source-observed] Trong `project1_app`, user/máy nằm trong SQLite `server/database/database.db`:
  - Bảng `users` và `app_sessions`.
  - Bảng `machines` và `machine_managers` (role owner|manager).
  - `/app/user/machine/list` join `machine_managers` với `machines`, trả `{machine_id,name,role}` (`machinelist_sync/machine_list_get.py`).
- [source-observed] Những thứ chỉ nằm trong RAM: online (heartbeat), hộp thư lệnh (`HOP_THU`, `DANG_CHO`), phiên đăng ký/OTP (`REGISTRATIONS`, TTL 900 s).
- [source-observed] Lỗ SEC-08: app gọi `GET /app/machine/status/get` **không token** cho từng máy, N+1 request.
- [source-observed] Thread heartbeat (`version1.1/machine/main.py:55-56`) chạy độc lập với vòng lệnh. Vòng lệnh treo thì máy vẫn "online", và `send()` (dùng `is_online` làm cổng chặn, `machine_transport.py:51`) xếp lệnh, chờ 20 s rồi trả 502.
- Kết luận của agent (chưa được user chốt): chọn B, suy online từ long-poll.
  - Online khi có một trong ba: có poll đang mở; poll/result cuối dưới 15 s; có lệnh đã take dưới 20 s.
  - Triển khai 3 pha: server trước (giữ route heartbeat), rồi máy, rồi dọn route.
  - Đưa `online`/`last_seen` vào machine list rồi đóng route status.
  - Giữ phiên đăng ký/OTP trong RAM, chỉ đưa vào DB khi chạy nhiều process.
- Case study ủng hộ B:
  - Step Functions, Temporal và GitHub Actions runner đều là worker long-poll. Lúc rảnh, poll là tín hiệu sống. Lúc bận, heartbeat gắn với task, không có thread heartbeat mù.

## Sản phẩm hiện hành
- `project1_app/docs/heartbeat-vs-long-poll.html`: dựng theo template FexMix_Munual, đã chụp kiểm ở 1400 px và 420 px. Đã sửa các lỗi:
  - Sơ đồ state vỡ do `&gt;`: Mermaid coi `;` là dấu tách câu.
  - Sơ đồ quyết định LR rối, đổi sang TD.
  - `.pc-box`/`.dg-main` cần `align-content:start`.
- `project1_app/docs/heartbeat-vs-long-poll.md`: bản Markdown, tạo do hiểu nhầm "lưu md". Giữ lại hay xoá do user quyết.
- Kết quả cuối của 3 agent: [archive/liveness-review-2026-10-10.md](../projects/internproj/project1_app/archive/liveness-review-2026-10-10.md).
- Cả hai file docs đều untracked, trên nhánh `feature/menu-image-cache`.

## Phần còn mở
- User chốt có triển khai B không, và có làm pha 2 phía máy (`version1.1`) không.
- Thứ tự đề xuất:
  1. Commit nhánh ảnh. Nhánh này đụng cùng `routing.py`/`routing.dart`.
  2. B1: đưa online vào list.
  3. Pha 1 server.
  4. B2: đóng route status, cẩn thận SEC-02/08 pass giả.
- Working tree đang xoá `docs/CRYPTOGRAPHY_MIDDLEWARE_DESIGN.html` mà không note nào ghi lại. Cần hỏi user.
- Máy chưa có lệnh `nhan_anh` (đã biết từ phiên 09/10).
