---
id: project-version11-map
type: project-map
scope: project:version1.1
status: source-observed
checked: 2026-10-09
source: /home/abel/project/internproj/version1.1
---

# `version1.1` — phần mềm máy pha FlexMix

Repo Git riêng ở `/home/abel/project/internproj/version1.1`. Bản đồ dựa trên `README.md`, `AGENTS.md` và cây tệp ngày 2026-10-09. Không đánh đồng với hai thư mục cùng tên nằm trong repo app/server.

```text
version1.1/
├── main.py                  điểm khởi động runtime
├── configuration/           cấu hình máy, mạng, đường dẫn phục vụ
├── store_gui/               giao diện bán hàng và web server
├── bartender_gui/           giao diện pha chế
├── admin_gui/               giao diện quản trị
├── test_gui/                giao diện kiểm phần cứng
├── order/                   luồng quét QR → công thức → pha chế
├── scan/, qrproto/          quét và giao thức QR
├── recipe/                  công thức và media
├── pump_control/            điều khiển bơm
├── loadcell/                cân/đo khối lượng
├── panel_control/           nút và panel
├── printer/                 in nhãn/QR
├── database/                lưu trữ và migration
├── deploy/                  cài đặt, kiosk, service
├── agent/                   tiến trình agent nối server mẹ (HTTPS CA ghim, FM1), chưa bật service; untracked
└── database/migrations/0003_agent.sql  sổ trạng thái/ledger cho agent (untracked)
```

README mô tả `main.py` chạy các luồng phục vụ POS, đồng bộ menu và xử lý đơn. Giao diện mặc định qua port 8080; `order/process_runner.py` dùng loopback 8000 cho luồng pha chế. Kiểm README và code hiện hành trước khi dựa vào cổng hay luồng này.

## Điểm vào khi làm việc

- Quy tắc: `AGENTS.md`, `CLAUDE.md`; tài liệu vận hành: `README.md`.
- Runtime: `main.py`, `order/run_flow.py`, `store_gui/serve.py`.
- Triển khai kiosk: `deploy/kiosk/README.md`.
- Agent server mẹ: `agent/README.md`; test `python3 -m unittest discover -s agent/tests`. `AGENT_WORKSPACE/` đã bị xoá trong working tree (chưa commit).
- Kiểm phần mềm: test nằm rải trong các module; `requirements-dev.txt` cho môi trường phát triển.

`AGENTS.md` cảnh báo repo gắn phần cứng thật: không chạy `main.py` hoặc gọi GPIO, bơm, panel, loadcell, máy in trong kiểm thử thông thường. Đọc quy tắc hiện hành trước mọi thao tác với máy.
