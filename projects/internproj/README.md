---
id: project-internproj-map
type: project-map
scope: project:internproj
status: source-observed
checked: 2026-10-09
source: /home/abel/project/internproj
---

# Bản đồ `internproj`

Điểm vào nhanh cho các phiên Codex/Claude Code. Đây là ảnh chụp cấu trúc đã kiểm ngày 2026-10-09, không thay thế code và chỉ dẫn trong từng repo. Trước khi sửa, đọc `AGENTS.md`/`CLAUDE.md` của repo liên quan và kiểm working tree.

```text
/home/abel/project/internproj/
├── project1_app/       repo Git riêng: app Flutter + service/app/machine
├── project1_server/    repo Git riêng: server mẹ quản lý nhiều máy
└── version1.1/         repo Git riêng: phần mềm chạy trên máy pha chế
```

`version1.1/` còn xuất hiện *bên trong* `project1_app/` và `project1_server/`. Ba thư mục đó thuộc ba repo khác nhau; không giả định chúng luôn đồng bộ.

## Chọn note cần đọc

| Task | Note | Nguồn hiện hành cần đối chiếu |
|---|---|---|
| App điện thoại, service kết nối app/máy (**repo đang tập trung**, theo `internproj/AGENT.md`) | [project1_app.md](project1_app.md), rồi một file trong `project1_app/` | `git log -1` so với `anchor_commit`; code chỉ đọc theo diff |
| Server mẹ FlexMix | [project1_server.md](project1_server.md) | `project1_server/AGENTS.md`, `README.md`, `internal/plan/` |
| Phần mềm điều khiển máy pha | [version1.1.md](version1.1.md) | `version1.1/AGENTS.md`, `README.md`, code liên quan |

## Cách dùng

Đọc note tầng 1 của đúng repo, rồi chỉ mở file tầng 2 cần cho task ([facts/tiered-project-context.md](../../facts/tiered-project-context.md)). Repo đã có `<repo>/STATUS.md` thì trả lời câu hỏi tiến trình từ đó, không quét code. Sau review hoặc cập nhật có thay đổi cấu trúc, vai trò module hay trạng thái, sửa đúng note, ngày `checked` và `anchor_commit`. Báo cáo agent mới ghi vào `<repo>/archive/`. Hiện chỉ `project1_app` có đủ 3 tầng. Yêu cầu hiện tại và bằng chứng code luôn ưu tiên hơn bản đồ này.
