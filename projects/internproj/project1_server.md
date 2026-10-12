---
id: project-project1-server-map
type: project-map
scope: project:project1_server
status: source-observed
checked: 2026-10-09
source: /home/abel/project/internproj/project1_server
---

# `project1_server` — server mẹ FlexMix

Repo Git riêng ở `/home/abel/project/internproj/project1_server`. Bản đồ dựa trên `README.md`, `AGENTS.md` và cây tệp ngày 2026-10-09. Plan và tiến độ có thể thay đổi nhanh; đọc nguồn hiện hành.

```text
project1_server/
├── AGENTS.md                 điểm vào quy trình dự án
├── README.md                 kiến trúc/tình hình do repo ghi
├── server/
│   ├── main.py, manage.py    chạy server và lệnh quản trị
│   ├── wiring.py             ghép các khối
│   ├── config/              cấu hình, routing
│   ├── contracts/           interface nội bộ
│   ├── core/db/             migration, cơ sở dữ liệu
│   ├── core/net/            mạng, TLS, wake
│   ├── security/            fm1, seca
│   ├── modules/             accounts, catalogue, machines
│   └── static/shell/        giao diện khung
├── internal/
│   ├── contracts/           hợp đồng giữa các khối
│   └── plan/                kế hoạch, checklist, công cụ, bằng chứng
├── docs/                     tài liệu thiết kế, trang HTML offline
├── tests/                    test theo khối; UI shell
├── spike/                    mã thử và vector, không phải runtime
├── agents/                   phối hợp agent và bối cảnh dự án
├── deploy/server/            triển khai server
└── version1.1/               bản sao mã máy, untracked (không thuộc Git repo này)
```

README xác định đây là server quản lý nhiều máy FlexMix trong LAN, thay `admin_gui` trên từng máy. `server/modules/` hiện có code cho `accounts`, `catalogue`, `machines`; không dùng mô tả tiến độ cũ trong README để khẳng định tất cả module đã hoàn tất.

## Điểm vào khi làm việc

- Quy trình và bối cảnh: `AGENTS.md`, `agents/memory/cach_phoi_hop.md`, `agents/memory/hieu_biet_server_me.md`.
- Plan và trạng thái: `internal/plan/index.md`, `internal/plan/checklist.md`, `internal/plan/giao_viec/README.md`, bằng chứng trong `internal/plan/bang_chung/`.
- Thiết kế: `docs/server_architect.html`, `internal/contracts/`.
- Runtime: `server/main.py`, `server/wiring.py`, các module tương ứng.
- Test: `tests/`, `spike/` cho thử nghiệm riêng. Xem lệnh trong `README.md` và môi trường thực tế trước khi chạy.

Checklist và báo hoạt động có quy tắc riêng trong `AGENTS.md`; khi thực hiện task trong repo, đọc bản hiện hành ở đó. Không sao chép trạng thái checklist vào note này.
