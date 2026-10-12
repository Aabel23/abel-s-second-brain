# Research database server hãng — 2026-10-11

Ba researcher Sonnet medium, không đọc vault, không đọc `project1_server/`. Đối chiếu code `project1_app` · `c34d55e`. Doc: `project1_app/documentation/research/server-database.html`. Chỉ để operator đọc.

## Kết luận chung (cả 3 đồng thuận)

- Server nhỏ, ~14 thực thể; không event sourcing, không Zanzibar, không shadow đầy đủ, không broker.
- Bất biến đếm/duy nhất đặt ở dữ liệu (partial unique index "một Owner", PK (máy, người), UPDATE có điều kiện cho mã mời); bất biến "ai được làm gì" ở code.
- Lịch sử quyền (server sinh, giữ vĩnh viễn) tách khỏi nhật ký thao tác (máy sinh, có hạn lưu); đối chiếu qua command_id.
- Chống chạy hai lần ở máy (bảng mã lệnh → kết quả, ~24 h kiểu Stripe), server không cần bảng idempotency.
- Chỉ admin_outbox phải bền ở khâu chuyển tiếp; gộp bản (một dòng/loại/máy).

## A · Danh tính và quyền

- Thực thể: Account, OTP_Challenge, Session (băm refresh token), Machine (serial, khóa công khai, lifecycle, acl_version), Membership (serial, account, vai), Invite (băm mã), Grant (giấy phép ký, mang acl_version), Permission_Event (chỉ thêm), Seal_Credential (băm có muối; operator gộp thành cột `seal_hash`).
- Lịch sử: chọn bảng hiện trạng + bảng sự kiện (temporal để sau, event sourcing không).
- acl_version: optimistic concurrency `UPDATE … WHERE acl_version = v`; máy tải lại toàn bộ danh sách khi cũ.
- Owner xóa tài khoản: chặn tới khi chuyển chủ/gỡ máy. Tranh chấp: thủ công + "gỡ cưỡng bức" nội bộ.
- Không tìm thấy `permissions.py` trong `project1_app`.

## B · Trạng thái máy, lệnh, cập nhật

- machine_state một dòng/máy: reported_sw_version, reported/desired_perm_version, last_event_seq, last_seen_at (ghi thưa), state_version. Online suy từ last_seen trong bộ nhớ.
- Lệnh thường trong bộ nhớ; server restart → app gửi lại cùng mã.
- admin_outbox: pending → delivered → acked / expired (đề xuất 30 ngày); delivered quay về pending nếu không ack.
- Seq: = last+1 ghi; ≤ last bỏ, vẫn ack; > last+1 yêu cầu gửi lại. Cần boot_epoch khi máy cài lại.
- Cập nhật: release + update_run (mỗi máy một dòng), giao đề nghị qua admin_outbox.
- `routing.py` server đã có MACHINE_STATUS_GET, MACHINE_COMMAND_POLL/SEND; menu_sync dùng menu_version — dùng lại cho quyền.

## C · Nhật ký và xuyên suốt

- audit_event khóa (machine_id, seq) [operator thêm epoch], actor_id + actor_role, action, target, detail, machine_time + tz_offset, received_at (server đặt), channel, contract_version, raw_payload. Không FK (học từ chú thích `error_log`).
- MVP chống sửa: quyền INSERT/SELECT, gap detection, bản gốc trên máy. Hash chain sau MVP; ký từ máy, WORM, Merkle không cần.
- Dung lượng ~1 KB/sự kiện: 100 máy ≈ 20–70 GB/năm; quyết định lớn nhất là sự kiện nào tính là nhật ký.
- Pháp lý: NĐ 13/2023 Điều 16 quyền xóa có ngoại lệ "luật có quy định khác"; có thể đã có Luật BVDLCN hiệu lực 1/1/2026 — **chưa xác minh**. GDPR Điều 17(3). Đề xuất ẩn danh hóa (operator đơn giản hóa: giữ dòng account, xóa PII).
- Lưu thô + cột chuẩn hóa; sự kiện phiên bản lạ vẫn lưu thô. Expand/contract. UUIDv7/ULID, UTC. Sao lưu ngày + PITR, thử khôi phục mỗi quý.

## Nguồn (mức kiểm: mở = agent đã tải trang; tìm = qua WebSearch; nhớ = chưa mở lại)

| Nguồn | Mức |
|---|---|
| Matter security model — https://academy.nordicsemi.com/courses/matter-fundamentals/lessons/lesson-1-matter-introduction/topic/matter-security-model/ ; https://developers.home.google.com/matter/primer/fabric ; https://developer.espressif.com/blog/matter-multi-admin-identifiers-and-fabrics/ | mở (A) |
| Apple Home roles — https://support.apple.com/102386 | mở (A) |
| Google Home granular access (nguồn thứ ba) — https://www.cepro.com/news/google-adds-granular-home-access-controls-in-latest-google-home-app-update/619879 | mở, cần kiểm lại |
| Tuya home roles — https://support.tuya.com/en/help/_detail/K9hutpxd9ov32 | mở (A) |
| Nuki — https://developer.nuki.io/t/web-api-authentication/52 ; https://docs.nuki.io/guide/overview/how-to/ | mở (A) |
| Apple Activation Lock — https://support.apple.com/en-mk/108934 | mở (A) |
| AWS IoT fleet provisioning — https://docs.aws.amazon.com/iot/latest/developerguide/provision-wo-cert.html | mở (A) |
| Zanzibar — https://research.google/pubs/zanzibar-googles-consistent-global-authorization-system/ | nhớ |
| AWS Device Shadow — https://docs.aws.amazon.com/iot/latest/developerguide/device-shadow-document.html | mở (B) |
| AWS IoT Jobs lifecycle — https://docs.aws.amazon.com/iot/latest/developerguide/iot-jobs-lifecycle.html | mở (B) |
| Azure C2D — https://learn.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-messages-c2d | mở (B) |
| Stripe idempotency — https://docs.stripe.com/api/idempotent_requests | mở (B) |
| Google Cloud IoT Core đóng 16/8/2023 — https://www.mobileworldlive.com/featured-content/top-three/google-to-retire-cloud-iot-service/ | mở (B) |
| Mender deployment — https://docs.mender.io/overview/deployment | mở (B) |
| Tuya, Particle, balena, Matter/Thread (phần B) | không tải được |
| AWS CloudTrail log integrity — https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-log-file-validation-intro.html | tìm (C) |
| NĐ 13/2023 — https://thuvienphapluat.vn/van-ban/Cong-nghe-thong-tin/Nghi-dinh-13-2023-ND-CP-bao-ve-du-lieu-ca-nhan-465185.aspx | tìm (C) |
| GitHub audit log, Google Cloud Audit Logs, Stripe Events, Stripe online migrations, gh-ost, RFC 6962/Trillian | nhớ (C) |
