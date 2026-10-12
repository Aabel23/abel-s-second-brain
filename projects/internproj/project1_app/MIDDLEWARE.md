---
id: project1-app-middleware
type: project-analysis
scope: project:project1_app
status: source-observed
checked: 2026-10-09
anchor_commit: 58a0b01
---

# Middleware: kế hoạch và trạng thái

**Trạng thái 09/10/2026: chưa triển khai.** Chưa có code middleware trong `server/` hay app. Tất cả G0–G7 đều ở trạng thái "Đề xuất, chưa triển khai". Đã kiểm bằng grep và đọc `http_server.py`.

## Kế hoạch (nguồn trong repo: `agent_workspace/tasks/middleware/`)

| Phase | Nội dung | Task | Điều kiện vào |
|---|---|---|---|
| G0 | Nền kiểm chứng: snapshot hợp đồng, sửa fixture/test lỗi thời, test tái hiện, đo môi trường | B01–B04 | Code hiện tại |
| G1 | Vòng đời request: context, response một lần, lỗi an toàn, log allowlist, correlation | L01–L04, D01, D04, O01 | G0 |
| G2 | Biên HTTP: framing, đọc đủ byte, deadline, media type/header | H01–H04 | G1 |
| G3 | Sức chứa: giới hạn kết nối trước khi tạo thread, tách luồng việc, giới hạn queue lệnh máy, deadline/huỷ | C01–C05, D03, O02–O03 | G2 + B04 |
| G4 | Danh tính/quyền: ma trận route-auth, thu hồi phiên, limiter với NAT, quyền GET status | S01–S05, D02 | G2 |
| G5 | Nghiệm thu: contract suite, đo tải, fault injection, thiết bị đích | V01–V05, O04 | G3/G4 |
| G6 | Vận hành: config/boot, shutdown/restart, rollout/rollback, TLS | P01–P05 | G5 |
| G7 | Mở rộng có điều kiện: runtime HTTP, device credential, idempotency, command bền, nhiều worker, DB | E01–E06 | G6 |

- Nơi gắn middleware đầu tiên: `server/lib/http/http_server.py: make_handler()` (G1). Gói chung `handle_routes` cũng nằm trong khung đó.
- Spec từng task (inputs/steps/files/acceptance/rollback): `internal/phases/<nhóm>/*.md`. Bản đồ phase: `phase-map.json`. Lộ trình: `internal/roadmap.md`.
- HTML cho người dùng đọc: `index.html`, `architecture.html`, `packet-security.html`, `crypto-primer.html`.
- Bảo vệ gói tin là nhánh thiết kế riêng nằm trong `internal/packet-security/`. Quyết định đã chốt xem [SECURITY.md](SECURITY.md).

## Quy trình team AI (AGENTS.md, đã chốt 06/10)
1. 2 architect phản biện nhau.
2. Operator hợp nhất, làm HTML để user chốt.
3. Planner chia phase.
4. 1 coder làm từng bước, 1 reviewer độc lập kiểm.
5. Cập nhật ✓/✗ sau mỗi lượt.

Vai và quyền ghi trong `agent_workspace/TEAM.md`, file vai trong `agents/`. Chạy `sync_agents.py` để sinh `.claude/agents/`.

## Hồ sơ nghiên cứu và thảo luận (archive, chỉ mở khi cần nguyên văn)
- `archive/middleware-operator-2026-10/`: design review r1–r5, decision/crypto options, transcript debate, system-fit, d6-rollout (khoảng 1 MB).
- `archive/security-research-2026-10-07/`: lượt nghiên cứu R5 Codex→Claude, FINDINGS/IMPROVEMENTS/CONSENSUS.
