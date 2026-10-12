---
id: session-20261008-plan-live-checklist
type: session-learning
scope: project:project1_server
status: extracted
date: 2026-10-08
source: current-conversation (Claude Code, VS Code, phiên 08/10/2026)
---

# Mục tiêu và chuẩn đạt

Trong `project1_server` (server mẹ FlexMix), user cần **theo dõi plan đang thực thi theo thời gian thực**:
một trang plan HTML stream ở cổng **6767**, có **checklist** từng bước với ba trạng thái
`chưa làm` / `đang làm` / `đã làm`, quy tắc đánh dấu **ghi thành Markdown** để agent làm theo, và một
**bảng nhỏ cho thấy agent nào đang hoạt động, đang làm gì**. Đạt khi mở trình duyệt thấy trang tự cập nhật
mỗi lần lưu file.

## Những lần chỉnh hướng

- Bước đầu user chỉ gõ "check" đường dẫn repo → tôi hiểu là kiểm sức khỏe repo (đọc README, chạy test, báo lỗi) và **không tự sửa**, chỉ đề xuất. User không phản đối; lần sau "check" = kiểm và báo, hỏi trước khi sửa.
- Tôi để server chạy nền trên 6767 khi test → user chạy lại bị `Address already in use`. **Lần sau: không chiếm cổng user định dùng; dọn tiến trình nền trước khi bàn giao, hoặc nói rõ nó đang chạy.**
- Khi thêm bảng agent, user nói "thôi khỏi" rồi ngay sau đó "làm tiếp đi :))". Tôi đã gỡ rồi thêm lại. **Lần sau: với lệnh huỷ giữa chừng của user, dừng lại thay vì vội gỡ code; chờ một nhịp/hỏi ngắn nếu việc gỡ tốn công.** (suy luận của agent)

## Quyết định và giới hạn

- [user-confirmed] Cổng 6767; checklist ba trạng thái; quy tắc viết vào md; có bảng agent hoạt động.
- [agent-decided, user không phản đối] Nguồn chuẩn là Markdown: `internal/plan/checklist.md` (quy tắc ở đầu file + bảng sinh bằng `--sync`), `internal/plan/agent_hoat_dong.md` (agent báo bằng lệnh, không sửa tay). Trạng thái ban đầu seed từ `internal/plan/bang_chung/*.md`, user nên duyệt lại.
- [agent-decided] Mặc định bind 127.0.0.1; `--host 0.0.0.0` cho LAN. Server chỉ phục vụ `docs/` và `internal/`.
- [source-observed] Quy ước dự án: sửa plan ở Markdown, không sửa tay `docs/server_plan.html`; dựng bằng `internal/plan/cong_cu/dung_trang_plan.js`. Một khối = một agent. Xem `project1_server/AGENTS.md`.
- [source-observed] Máy dev Linux không có `node` trên PATH, không có `python3-venv`/pip; có `uv`, Node đi kèm VS Code ở `~/.vscode-server/cli/servers/*/server/node`, có google-chrome để chụp màn hình headless. README dự án viết lệnh kiểu Windows (`.venv/Scripts`).

## Sản phẩm hiện hành

- `internal/plan/cong_cu/theo_doi_plan.py` — server live (stdlib), `--check`, `--sync`, `--agent/--step/--task/--idle`.
- `internal/plan/checklist.md` — quy tắc + 166 mục (16 đã làm, 15 đang làm, 135 chưa làm lúc seed).
- `tests/plan/test_checklist.py` — 17 test đạt.
- Sửa `AGENTS.md` (bắt buộc dùng checklist + báo hoạt động), `README.md`, `internal/plan/index.md`.
- **Chưa commit.**

## Phần còn mở

- Test `tests/s_net/test_net.py::test_tls_isolation_http_and_safe_logs` đỏ trên Linux (bind lại cổng ephemeral bị TIME-WAIT phía server sau ca TLS 1.2). Lỗi của test, không phải server; đã đề xuất đổi sang kiểm `ConnectionRefusedError`, **user chưa trả lời**.
- README ghi "chưa commit" nhưng đã commit; README chỉ có lệnh Windows — đề xuất sửa, chưa làm.
- User chưa duyệt trạng thái seed trong checklist.
