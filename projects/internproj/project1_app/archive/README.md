---
id: project1-app-archive
type: archive-catalog
scope: project:project1_app
status: source-observed
checked: 2026-10-09
---

# Kho báo cáo agent của `project1_app` (tầng 3)

**Không đọc mặc định.** Chỉ mở khi cần trích nguyên văn, bằng chứng hoặc lý do của một quyết định cũ. Nội dung đã được tóm tắt ở các note tầng 2: [STATUS](../STATUS.md), [TESTING](../TESTING.md), [SECURITY](../SECURITY.md), [MIDDLEWARE](../MIDDLEWARE.md). Tìm theo từ khoá bằng `rg <từ> <thư mục>` thay vì đọc cả file. Transcript `.jsonl` và `.log` rất dài, đừng mở toàn bộ.

Ngày 09/10/2026, các file dưới đây được chuyển từ repo sang vault. Bản trong vault và bản gốc giống nhau từng byte (đã `diff -r`/`cmp`). Bản gốc vẫn có trong lịch sử git của repo (`git show 58a0b01:<đường dẫn cũ>`). Link tương đối bên trong các file archive vẫn trỏ theo đường dẫn cũ trong repo, nên một số link có thể gãy.

| Thư mục | Đường dẫn cũ trong repo | Nội dung | Mở khi |
|---|---|---|---|
| [test-runs-2026-09/](test-runs-2026-09/) | `log/` (trừ 4 file runner/spec còn dùng) | Báo cáo từng lượt `run_tests.py` 28–29/09 (`YYYYMMDD-HHMMSS.md`, `-e2e.txt`), `HISTORY.md` (bảng ĐẠT/LỖI theo lượt), `BUG_LOG.md` (B1–B5, O1, T1–T3), `refactor-*.txt` | Cần biết vì sao một luồng hỏng/đạt trước đây |
| [test-reports-2026-09-29/](test-reports-2026-09-29/) | `tests/reports/` | `refactor-20260929.md` (Codex refactor và kiểm), `claude-20260929.md` (Claude test qua đêm 02:50–05:00) | Truy lại việc refactor Flutter/server ngày 29/09 |
| [security-research-2026-10-07/](security-research-2026-10-07/) | `doc/security-research/2026-10-07/` | Lượt nghiên cứu R5: `FINDINGS.md` F01–F11, `IMPROVEMENTS.md` I01–I12, `CLAUDE_REVIEW.md`, `consensus-loop/CONSENSUS.md` (đồng thuận 10/10), handoff, log scheduler, nguồn | Đụng tới durability, trusted time, bootstrap, revoke hoặc thư viện HPKE |
| [middleware-operator-2026-10/](middleware-operator-2026-10/) | `agent_workspace/tasks/middleware/internal/operator/` | Design review r1–r5 và feedback, `decision-options` (D) và `crypto-options` (C), `system-fit` (đối chiếu với code), `d6-rollout`, `decision-debate/` (transcript và critique), planner-intake, visual-review | Cần lý do hoặc phương án bị loại của các quyết định D/C |
| [cpp-setup-2026-10/](cpp-setup-2026-10/) | `agent_workspace/tasks/cpp-setup/` | Cài môi trường C++ trong VS Code (g++ 15.2, GDB 17.1), đã xong | Hầu như không cần |
| [liveness-review-2026-10-10.md](liveness-review-2026-10-10.md) | — (mới) | Kết quả cuối của 3 agent đánh giá heartbeat và long-poll: bỏ heartbeat, gộp online vào list, nghiên cứu ngoài | Đụng tới `is_online`, heartbeat, route status hoặc SEC-08 |
| [heartbeat-plan-2026-10-10/](heartbeat-plan-2026-10-10/) | — (mới) | Bản thử kiểm chứng plan bỏ heartbeat (diff 3 pha, test S13 tham chiếu, số liệu cổng). Chỉ operator dùng khi review | Thực thi hoặc review `agent_workspace/tasks/heartbeat-long-poll/TASK.md` |
| [heartbeat-refactor-2026-10-10/](heartbeat-refactor-2026-10-10/README.md) | — (mới) | Kết quả cuối lượt thực thi plan bỏ heartbeat: commit, agent, cổng, bài học vận hành, việc còn mở | Đụng tới online máy, `machine_transport`, hoặc chạy lại team agent |
| [iot-design-research-2026-10-11.md](iot-design-research-2026-10-11.md) | — (mới) | Kết quả cuối 5 agent research tầng 1 hệ IoT v1.1: claim máy, dữ liệu desired/reported, phân quyền, định danh chủ, gắn thiết bị qua kênh bán | Thiết kế nhận máy, phân quyền hoặc đồng bộ dữ liệu v1.1 |
| [account-research-review-2026-10-11.md](account-research-review-2026-10-11.md) | — (mới) | 2 research tài khoản (POS Việt Nam, danh tính SĐT/PIN) + 1 review thiết kế tài khoản/kích hoạt: lỗi C1–C5, mâu thuẫn, 10 luồng thiếu | Thiết kế tài khoản, doanh nghiệp, nhân viên v1.1 |
| [permission-offline-research-2026-10-11.md](permission-offline-research-2026-10-11.md) | — (mới) | 3 research thế giới (prompt không chứa thiết kế hiện tại): phân quyền khi offline, ai thuộc máy nào, vai trò nhân viên; gộp 4 option | Phân quyền, offline, chế độ LAN/BT v1.1 |
| [maintainable-architecture-research-2026-10-11.md](maintainable-architecture-research-2026-10-11.md) | — (mới) | 6 research (2 vòng): 4 lớp bảo trì (ranh giới, hợp đồng, thay đổi an toàn, quan sát); 16 kiểu kiến trúc + ma trận cho app/server/máy; hợp đồng có phiên bản, đồng bộ offline, OTA, fleet; quy trình ADR/C4/nhánh/SemVer/deprecate/DORA; bảng nguồn có mức kiểm; đối chiếu code `c34d55e` | Chọn kiến trúc, giao thức 3 bên, OTA, quy trình làm việc/bảo trì |
| [multi-agent-orchestration-research-2026-10-11.md](multi-agent-orchestration-research-2026-10-11.md) | — (mới) | 4 research điều phối nhiều agent LLM: AWS (Agent Squad, Bedrock, Strands), LangGraph/AutoGen→MAF/CrewAI, blackboard + debate (kết quả âm tính), hệ lớn (Anthropic, OpenAI, ADK/A2A/MCP, MetaGPT, Devin) + MAST/Cognition; bảng nguồn có mức kiểm; chưa có HTML | Thiết kế luồng agent làm việc, bước debate |
| [auth-identity-research-2026-10-11.md](auth-identity-research-2026-10-11.md) | — (mới) | 3 research + kiểm nguồn: đăng ký/OTP email, phiên app↔server, danh tính điện thoại offline (Keystore + bản quyền ký); bảng nguồn có mức kiểm; lỗi Bluetooth trả `product_key` | Tài khoản, đăng nhập, kênh app↔máy v1.1 |
| [server-database-research-2026-10-11.md](server-database-research-2026-10-11.md) | — (mới) | 3 research mô hình dữ liệu server hãng: danh tính+quyền, trạng thái máy+lệnh+cập nhật, nhật ký+pháp lý+vận hành; bảng nguồn có mức kiểm; đối chiếu schema máy `c34d55e` | Thiết kế database server, nhật ký, hàng chờ quản trị |
| [contracts-skeleton-research-2026-10-12.md](contracts-skeleton-research-2026-10-12.md) | — (mới) | 4 research: danh tính/nhận máy, relay/trạng thái trực tiếp, bootstrap mã dài và ranh giới TLS/HPKE, khả thi thư viện/BLE; phương án, điểm nghẽn, case study có link và mức kiểm; chưa là quyết định user | Đặc tả contracts skeleton trước coder |

## Quy tắc cho báo cáo mới

Tranh luận giữa agent chỉ để lại **kết quả cuối** (user chốt 09/10/2026). Chạy bằng `agent_workspace/debate.py` trong repo, ghi kết quả thành một file `archive/debate-<chủ-đề>-<YYYY-MM-DD>.md`. Không lưu transcript, các lượt trao đổi, critique hay handoff. Báo cáo test hoặc lượt chạy thì ghi theo dạng `archive/<chủ-đề>-<YYYY-MM-DD>/`. Mỗi thứ mới đều thêm một dòng vào bảng trên. **Không** để trong repo. Sau đó cập nhật kết luận ngắn vào note tầng 2 tương ứng, và thêm một dòng nhật ký vào [STATUS](../STATUS.md).
Riêng các file runner tự sinh trong repo (`log/FLOWS.md`, `log/flows_state.json`, `log/HISTORY.md`, `log/<thời điểm>.md`) thì cứ để runner ghi. Khi cần giữ lại thì chuyển sang `test-runs-<tháng>/`.
