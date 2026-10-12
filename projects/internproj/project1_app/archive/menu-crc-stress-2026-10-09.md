# Workflow `menu-crc-stress-tool` — 09/10/2026

Run `wf_50bfe1dd-704`. 7 agent Opus effort low: Build ×3 (harness/oracle/scenario song song theo hợp đồng chốt sẵn) → Integrate → Review ×2 (lens "không gian lận" và correctness) → Fix. Nhánh `feature/menu-image-cache`, chưa commit.

## Kết quả
- Integrate: không cần sửa, 300 vòng seed 42 PASS (1,76s), seed 7 ×100 PASS, thêm `tests/python/test_menu_crc_stress.py` + FLOWS S12.
- Review tìm 3 lỗi trong tool (không có bug sản phẩm):
  - high: không kiểm update thật sự ghi DB — sabotage `update_drink` no-op vẫn PASS → thêm check `update_applied`.
  - low: reply ok/conflict thiếu `packet` bị bỏ qua → giờ tính fail `packet_valid`.
  - low: vòng lặp máy (`main.run`) không dừng được → harness thay `poll`/heartbeat bằng wrapper ném `_Stop`, `stop()` join thread.
- Sabotage khác đều bị bắt: version +1, bỏ cột, JSON ASCII, CRC một phần cột, lọt món xóa mềm.
- Agent kiểm lại sau cùng: oracle tự kiểm OK; 500 vòng seed 2026 PASS 3,06s, 354 menu khác nhau, 0 va chạm; unittest S11+S12 OK.

## Ghi chú harness
- `POLL_WAIT_SECONDS` = 1.5 (không phải 0.3) vì `main.poll()` ngủ 1s khi server trả nhanh hơn 1s.
- Rate limit vô hiệu bằng patch `http_rate_limit.MAX_REQUESTS = 10**9`.
- Đổi tên món ghi thẳng SQLite máy (`db_execute`) vì API chỉ cho sửa `price`/`available`.
- Log server/máy rất ồn; báo cáo nằm sau dòng `=== BÁO CÁO`.
