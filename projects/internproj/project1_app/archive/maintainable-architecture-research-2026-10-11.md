---
id: project1-app-maintainable-architecture-research-20261011
type: research-archive
scope: project:project1_app
status: source-observed
date: 2026-10-11
source: sessions/2026-10-11-maintainable-architecture-research.md
---

# Research: kiến trúc và quy trình để sửa từng phần mà không sửa cả hệ

Kết quả cuối của các agent research (Sonnet, effort medium), gộp và kiểm nguồn bởi operator. Đây là **kho định hướng**: nguyên lý, case study, link nguồn. Không phải quyết định của user. Bản HTML cho người đọc: `project1_app/documentation/research/maintainable-large-systems.html` (repo, commit đối chiếu `c34d55e`).

- **Vòng 1** (xong): 3 agent: ranh giới code, thay đổi an toàn trên production, debug và quan sát.
- **Vòng 2** (xong): 3 agent: bản đồ 12–15 kiểu kiến trúc cho hệ app + cloud gateway + thiết bị offline; giao tiếp và vòng đời thiết bị (hợp đồng có phiên bản, đồng bộ offline, OTA, fleet); quy trình làm việc và bảo trì (ADR/RFC/C4, nhánh, release, SemVer, deprecate, test, trực sự cố, DORA). Kết quả sẽ thêm vào mục "Vòng 2" cuối file.

## Vòng 1 · Kết luận chung

Hệ lớn chồng **bốn lớp**, mỗi lớp giải một nửa bài toán:

| Lớp | Nguyên tắc cốt lõi |
|---|---|
| 1. Ranh giới | Chia theo **nghiệp vụ**, không theo tầng kỹ thuật. Mỗi khối **sở hữu dữ liệu** của mình, khối khác chỉ đi qua API. Ranh giới phải **máy kiểm** trong CI (Packwerk, ArchUnit, Bazel visibility, Nx, import-linter). Mỗi khối có chủ (Conway, CODEOWNERS). |
| 2. Hợp đồng | API nhỏ, ổn định, chỉ thêm không đổi nghĩa; phá vỡ thì mở `/v2` song song. Tương thích cả hai chiều để rollback được. Contract test (Pact) khi có nhiều bên gọi. |
| 3. Thay đổi an toàn | Tách deploy khỏi release bằng feature flag. Canary so với nhóm đối chứng cùng lúc, không so trước/sau. Thay tính năng bằng strangler fig / branch by abstraction / parallel run. Đổi dữ liệu bằng expand/contract, mỗi bước một deploy. Cô lập lỗi: timeout + retry backoff, fallback, circuit breaker, bulkhead, rate limit, cell-based, shuffle sharding. Dọn flag ngay khi 100%. |
| 4. Quan sát | Request ID xuyên mọi khối; log có cấu trúc, metric (golden signals, RED/USE), trace (OpenTelemetry). Alert theo triệu chứng người dùng (SLO). Hỏi "vừa đổi gì?" trước. Postmortem không đổ lỗi. |

**Microservices không phải mặc định.** Modular monolith có ranh giới máy kiểm là điểm bắt đầu cho đội nhỏ; tách service chỉ khi (1) mô tả được khối trong một câu "giữ dữ liệu gì, mở API gì", (2) có lý do cụ thể (scale lệch, hai đội vướng release, cần cô lập lỗi), (3) đã có nền tảng vận hành (log có request ID, deploy tự động, rollback một lệnh).

**Không làm sớm:** microservices từ đầu; tách service theo bảng (dẫn tới distributed monolith); Kafka/event sourcing/CQRS khi chưa cần; cell-based, chaos engineering; tự xây nền tảng flag/quan sát; thư viện chung phình to.

### Bộ tối thiểu theo quy mô (nhỏ 1–3 đội / vừa 4–15 / lớn >15)

| | Nhỏ | Vừa | Lớn |
|---|---|---|---|
| Ranh giới | Modular monolith, thư mục theo nghiệp vụ, một cửa công khai mỗi khối | + lint ranh giới, CODEOWNERS | Microservices chọn lọc, gom theo domain (DOMA) |
| Hợp đồng | Chỉ thêm trường | Versioning, contract test API quan trọng | Contract test khắp nơi |
| Release | CI chặn merge, deploy/rollback một lệnh, flag từ config | Dịch vụ flag, canary một máy xem bằng mắt | Canary tự động, rollout theo vùng |
| Dữ liệu | Expand/contract mọi thay đổi schema | + gh-ost/pt-osc bảng lớn | Migration tự động |
| Cô lập lỗi | Timeout, retry backoff, một fallback | Circuit breaker, rate limit | Cell-based, shuffle sharding |
| Quan sát | Log JSON có request ID, Sentry, golden signals | Tracing OpenTelemetry, SLO | Wide events, profiling liên tục, chaos |

### Luồng mẫu "viết lại một tính năng khi hệ đang chạy"
Commit nhỏ sau flag → CI (test, ranh giới, contract) → deploy flag tắt → parallel run so kết quả → bật nội bộ → 1% → quan sát so nhóm đối chứng → lỗi thì tắt flag (vài giây) → vá → 10/50/100% → xoá flag và code cũ.

### Luồng mẫu "từ cảnh báo tới dòng code"
Alert theo SLO → dashboard golden signals khoanh service → trace theo request ID → "vừa deploy/bật flag?" thì tắt/rollback trước → log + stack trace cùng ID → phát lại để tái hiện → sửa, test hồi quy, postmortem.

## Vòng 1 · Đối chiếu project1_app (commit `c34d55e`)

| Lớp | Hiện có | Nhận xét (agent, chưa phải quyết định) |
|---|---|---|
| Ranh giới | `flutter_app/lib/feature/<tính năng>/` (user_auth, machine_register, machine_share, machine_menu, machine_ingredient, orders); `version1.1/machine/` (menu_sync, ingredient_sync, pairing, server_connection); `version1.1/database/services/`; server mới có `server/configuration/routing.py` | Đúng hướng |
| Hợp đồng | Quy ước repo: đổi gói tin thì sửa app, server, máy cùng lúc | **Rủi ro lớn nhất**: khi máy đã ở quán và app đã trên điện thoại thì không cập nhật cùng lúc được → dạng distributed monolith. Đề xuất hợp đồng có phiên bản, tương thích ngược |
| Thay đổi an toàn | Thiết kế chưa có OTA phần mềm máy | Còn mở: OTA theo nhóm máy, tự quay về bản cũ |
| Quan sát | Thiết kế có nhật ký theo người không xoá được | Còn mở: tách nhật ký kiểm toán khỏi log debug |
| Cô lập lỗi | Dữ liệu trên máy; app tự chuyển LAN → BT | Đúng hướng: mỗi máy là một "cell" tự nhiên, LAN/BT là fallback |

Đề xuất operator (chưa chốt, user muốn research rộng hơn trước): ba bên đều modular monolith; app → server hỏi–đáp; server ↔ máy bằng sự kiện có phiên bản, máy gửi bù khi có mạng; không EDA đầy đủ (broker lớn, event sourcing, CQRS).

## Case study và nguồn

Cột "Kiểm" ghi: **mở** = agent hoặc operator đã mở được trang; **tìm** = xác nhận qua kết quả tìm kiếm/bài thứ cấp; **nhớ** = link từ kiến thức sẵn có, chưa mở lại.

| Ai / nguồn | Ý chính | Link | Kiểm |
|---|---|---|---|
| Shopify | Modular monolith Rails rất lớn, component theo nghiệp vụ, Packwerk ép ranh giới | https://shopify.engineering/deconstructing-monolith-designing-software-maximizes-developer-productivity · https://shopify.engineering/shopify-monolith | mở |
| Uber DOMA | Gom hàng nghìn microservice thành domain, mỗi domain một gateway | https://www.uber.com/blog/microservice-architecture/ | mở |
| Segment | Gộp >50 repo destination về một; 3 kỹ sư chỉ giữ hệ chạy; thư viện chung lệch phiên bản | https://twilio.com/en-us/blog/developers/best-practices/goodbye-microservices | mở |
| Amazon Prime Video (2023) | Dịch vụ giám sát chất lượng video: Step Functions + Lambda + S3 → tiến trình gộp trên EC2/ECS, dữ liệu trong bộ nhớ, giảm ~90% chi phí. InfoQ: vẫn nhiều task ECS, không hẳn "monolith". Chỉ một dịch vụ | https://www.infoq.com/news/2023/05/prime-ec2-ecs-saves-costs/ | tìm (bài gốc chưa đọc) |
| Google monorepo | ~2 tỉ dòng, 86TB, Piper, CitC, trunk-based | https://research.google/pubs/why-google-stores-billions-of-lines-of-code-in-a-single-repository/ | tìm |
| Meta rapid release | Từ cherry-pick sang push từ master (4/2016), rollout nhân viên → % traffic; 100% web từ master 4/2017 | https://engineering.fb.com/2017/08/31/web/rapid-release-at-massive-scale/ | tìm |
| Meta Configerator | Quản lý config/flag toàn hạ tầng | https://research.facebook.com/publications/holistic-configuration-management-at-facebook/ | nhớ |
| Kayenta (Netflix + Google) | Canary tự động bằng thống kê, tích hợp Spinnaker | https://cloud.google.com/blog/products/gcp/introducing-kayenta-an-open-automated-canary-analysis-tool-from-google-and-netflix | tìm |
| Spinnaker | CD đa cloud của Netflix | https://spinnaker.io/ | nhớ |
| Netflix Hystrix | Circuit breaker, bulkhead, fallback (đã bảo trì, ý tưởng sang Resilience4j) | https://github.com/Netflix/Hystrix | nhớ |
| Google SRE | Canarying, error budget, golden signals, postmortem | https://sre.google/workbook/canarying-releases/ · https://sre.google/sre-book/embracing-risk/ · https://sre.google/sre-book/monitoring-distributed-systems/ · https://sre.google/sre-book/postmortem-culture/ · https://sre.google/sre-book/release-engineering/ | mở (canary, monitoring, postmortem) |
| AWS Builders' Library | Deploy an toàn theo wave/one-box/bake time; timeout-retry-jitter; shuffle sharding | https://aws.amazon.com/builders-library/automating-safe-hands-off-deployments/ · https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/ · https://aws.amazon.com/builders-library/workload-isolation-using-shuffle-sharding/ | nhớ (bị chuyển hướng khi mở) |
| AWS cell-based | Giảm phạm vi ảnh hưởng bằng cell | https://docs.aws.amazon.com/wellarchitected/latest/reducing-scope-of-impact-with-cell-based-architecture/reducing-scope-of-impact-with-cell-based-architecture.html | nhớ |
| GitHub Scientist / gh-ost | Parallel run khi thay code lõi; migration MySQL online qua binlog | https://github.blog/2016-02-03-scientist/ · https://github.com/github/gh-ost | nhớ |
| Percona pt-osc | Migration online bằng trigger | https://docs.percona.com/percona-toolkit/pt-online-schema-change.html | nhớ |
| Martin Fowler | Feature Toggles (4 loại), Strangler Fig, Branch by Abstraction, Parallel Change | https://martinfowler.com/articles/feature-toggles.html · https://martinfowler.com/bliki/StranglerFigApplication.html · https://martinfowler.com/bliki/BranchByAbstraction.html · https://martinfowler.com/bliki/ParallelChange.html | mở (Feature Toggles) |
| Trunk-based development | Nền cho flag và branch by abstraction | https://trunkbaseddevelopment.com/ | nhớ |
| Knight Capital (1/8/2012) | Dùng lại flag cũ của Power Peg (code chết từ 2003); 1/8 máy chưa cập nhật; lỗ ~461,1 triệu USD trong 45 phút | https://en.wikipedia.org/wiki/Knight_Capital_Group · https://www.sec.gov/Archives/edgar/data/0001060749/000119312512513607/d457296dex21.htm | tìm |
| Google Dapper | Tracing xuyên service, sampling, instrument trong thư viện chung | https://research.google/pubs/dapper-a-large-scale-distributed-systems-tracing-infrastructure/ | mở |
| Netflix Edgar | Dựng lại phiên xem từ trace + log theo mã phiên | https://www.infoq.com/articles/observability-edgar-netflix/ | mở |
| Netflix Simian Army | Chaos Monkey | https://netflixtechblog.com/the-netflix-simian-army-16e57fbab116 | nhớ |
| LinkedIn / Jay Kreps "The Log" (16/12/2013) | Kafka: thay pipeline điểm-điểm bằng log chung | https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying | tìm |
| Meta Scuba (VLDB 2013) | Truy vấn ad hoc dưới 1 giây để debug | https://www.cs.uic.edu/~brents/cs494-cdcs/papers/scuba.pdf | mở |
| Spotify Backstage | Danh mục service, chủ sở hữu cạnh code | https://backstage.io/docs/features/software-catalog/ | mở |
| Honeycomb | Observability 2.0, wide events | https://www.honeycomb.io/blog/time-to-version-observability-signs-point-to-yes | mở |
| Uber Jaeger / Twitter Zipkin | Tracing mã nguồn mở | https://www.uber.com/blog/distributed-tracing/ · https://blog.x.com/engineering/en_us/a/2012/distributed-systems-tracing-with-zipkin | nhớ (404/chuyển hướng) |

## Vòng 2 · Kết luận chung (3 agent, xong 11/10/2026)

**"Modular monolith hay EDA" là so sánh khập khiễng.** Kiến trúc nằm trên 4 trục: chia code (C), giao tiếp (G), dữ liệu (D), triển khai (T). Modular monolith thuộc trục C/T, EDA thuộc trục G; chọn được cả hai.

Cả ba agent hội tụ về: **ba bên đều modular monolith; máy là nguồn sự thật (edge-first); cloud mỏng có "shadow nhẹ" (desired/reported/version) cho lệnh, quyền, cấu hình; một envelope thông điệp chung có phiên bản cho cloud/LAN/BT; sự kiện chỉ dùng cho thông báo và đồng bộ bù, không EDA đầy đủ.** Đây là đề xuất của agent, chưa phải quyết định của user.

### A · Bản đồ kiến trúc (16 kiểu)

| Kiểu | Trục | Đội nhỏ |
|---|---|---|
| Layered / n-tier | C | Có (nền) |
| Hexagonal / clean / onion | C | Có, chỉ ở biên phần cứng và biên truyền tải |
| Modular monolith | C, T | Rất hợp |
| Microservices | C, G, D, T | Không |
| SOA (ESB) | G, T | Không |
| Event-driven (broker / mediator) | G | Chỉ bus trong tiến trình |
| Microkernel / plugin | C | Hợp phía máy (driver phần cứng) |
| Actor model | C, G | Không bắt buộc |
| CQRS | D | Không ở bản đầu |
| Event sourcing | D | Không; bảng log append-only là đủ cho audit |
| Space-based | D, T | Không |
| Serverless / FaaS | T | Chỉ việc phụ (thông báo, cron) |
| Cell-based | T | Chưa cần |
| Edge / fog | T, D | Bản chất của hệ này |
| Device shadow / digital twin | D, G | Bản nhẹ |
| Local-first (CRDT) | D | Ý tưởng có, không dùng CRDT cho đơn/kho |

Ánh xạ đề xuất:
- **Máy:** modular monolith (thư mục theo miền như `version1.1/` hiện có) + microkernel cho driver phần cứng (bơm, loadcell, máy in, quét QR) + hexagonal nhẹ (lõi recipe/order/kho không import thư viện phần cứng) + event bus trong tiến trình kiểu Home Assistant + edge-first.
- **Server:** modular monolith nhỏ (tài khoản, danh sách máy, quyền, chuyển tiếp) + shadow nhẹ (online, phiên bản quyền, hàng đợi lệnh).
- **App:** feature-first + layered gọn (UI / state / repository); repository là một "cổng" truyền tải với ba adapter cloud/LAN/BT.

Ma trận (agent A, 1–5): A = modular monolith + shadow nhẹ; B = A + bus nội bộ + plugin driver; C = EDA đầy đủ (broker, microservices cloud).

| Tiêu chí | A | B | C |
|---|---|---|---|
| Sửa cục bộ | 4 | 5 | 4 |
| Debug tại quán | 5 | 4 | 2 |
| Chi phí đội nhỏ | 5 | 4 | 1 |
| Chạy offline | 5 | 5 | 3 |
| Lên hàng nghìn máy | 3 | 3 | 5 |
| Bảo mật/quyền | 4 | 4 | 3 |
| Dễ test | 4 | 4 | 3 |
| Hợp code hiện có | 5 | 4 | 2 |

Agent A khuyến nghị **B**, giới hạn: lệnh là gọi hàm, sự kiện chỉ để thông báo (tránh event spaghetti). Tổ hợp IoT phổ biến: thiết bị microkernel + cloud modular monolith + shadow (đội nhỏ); hub cục bộ là gốc + cloud để truy cập từ xa (Hue, Home Assistant, Matter); broker MQTT + microservices + CQRS/ES (đội lớn).

### B · Giao tiếp và vòng đời thiết bị

| Đường | Kiểu |
|---|---|
| App → server | Hỏi–đáp HTTPS cho command/query; có thể thêm kênh đẩy cho thông báo |
| Server ↔ máy | Máy mở kết nối ra cloud (qua NAT) và giữ mở; server đẩy lệnh xuống, máy gửi kết quả/event lên. Poll HTTP là mức MVP |
| App ↔ máy LAN | Cùng định dạng lệnh, tìm máy bằng mDNS |
| App ↔ máy BT | Cùng định dạng lệnh, đóng khung (độ dài + id), phân mảnh |

Phân loại thông điệp: command (có id, ack, idempotent), query (không tác dụng phụ), event (đã xảy ra, không từ chối).

Hợp đồng có phiên bản: (1) một envelope chung `{v, type, id, ts, from, body}` cho mọi đường; (2) một nguồn schema trong `contracts/`, sinh kiểm tra cho Dart và Python; (3) bắt tay báo `proto_min/proto_max/ver/capabilities`; (4) chỉ thêm, bên nhận bỏ qua trường lạ, phá vỡ thì tăng major và giữ song song major cũ N tháng; (5) kiểm capabilities thay vì so số phiên bản; (6) thứ tự cập nhật server → máy → app, server là lớp dịch giữa các phiên bản; (7) golden messages (fixture) dùng chung cho 3 bên, Pact để sau; (8) test ma trận 2–3 phiên bản gần nhất.

Đồng bộ offline: máy là nguồn sự thật; `command_id` do app sinh, máy lưu bảng đã xử lý để không chạy lại; outbox cho event cùng transaction với thay đổi nghiệp vụ, gửi khi có mạng, server khử trùng theo `event_id`; thứ tự theo `seq` tăng dần mỗi máy, không theo đồng hồ; khoá lạc quan theo `version`; shadow desired/reported, máy kéo toàn bộ khi nối lại (Azure không giữ thông báo cho máy offline); bản sao quyền quá hạn thì từ chối lệnh ghi.

OTA MVP (vài chục máy): bản phát hành có số phiên bản + SHA-256 + chữ ký hãng; `releases/<ver>/` + symlink `current`; chỉ cập nhật khi máy rảnh; health check sau khởi động, không đạt thì tự quay về bản cũ; migration DB tương thích ngược một bản, sao lưu trước khi migrate; server ghi phiên bản máy, rollout thủ công. Sau MVP: A/B partition (Mender/RAUC), rollout theo nhóm tự dừng, delta update, chuỗi khoá và thu hồi.

Fleet MVP: heartbeat (phiên bản, uptime, đĩa, nhiệt độ, số event chờ trong outbox, lỗi gần nhất); log vòng cục bộ, gửi tóm tắt trước; mã lỗi chuẩn hoá.

Đối chiếu code máy (operator đã kiểm lại 11/10/2026, commit `c34d55e`):
- Giữ được: `version1.1/machine/menu_sync/machine_menu_sync.py` có khoá lạc quan (`menu_version` khác → `conflict` + gói mới; trùng → `up_to_date`); máy chủ động gọi ra server (`server_connection/machine_server_request.py`: `poll_command`, `send_result`) nên qua NAT; BT ở `pairing/`.
- Thiếu: không có phiên bản giao thức hay bắt tay; không lưu `lenh_id` đã xử lý (gửi lại có thể chạy hai lần, nguy hiểm với trừ kho); không có outbox; mỗi lần gọi mở HTTP riêng `timeout=10`, không retry/backoff; `product_key` nằm trong body mỗi request (`machine_server_request.py:12`), nếu `SERVER_URL` là http thì lộ; chưa có OTA, chữ ký, báo phiên bản; chưa có hợp đồng chung giữa 3 bên.

### C · Quy trình làm việc và bảo trì

Vòng đời một thay đổi: issue có nhãn (`app`/`server`/`machine`/`contract`, `bug`/`feature`/`debt`) → định cỡ "appetite" (Shape Up) → mini design doc 1–3 trang khi mơ hồ hoặc chạm nhiều bên, ADR cho quyết định quan trọng → sửa hợp đồng trước trong PR riêng → plan trước khi code (nhất là với AI agent) → nhánh ngắn ≤1–2 ngày, flag cho phần dở → CI (lint, unit, contract bằng fixture, path filter) → 1 reviewer người + CODEOWNERS → main luôn releasable → tag riêng `server-vX`/`app-vX`/`machine-vX` + changelog → triển khai mỗi bên một nhịp → quan sát, runbook, postmortem → bảo trì định kỳ (nợ kỹ thuật 10–20% hoặc cool-down, nâng thư viện hàng tháng, dọn flag, gỡ deprecated).

Điểm chính:
- **ADR (Nygard):** 5 phần (Title, Context, Decision, Status, Consequences), đánh số, không xoá, đổi thì superseded. Bắt buộc, nhẹ, trong repo.
- **C4:** chỉ vẽ context và container (container khớp 3 sản phẩm). arc42 chỉ mượn mục quyết định và rủi ro.
- **Nhánh:** GitHub flow / trunk-based; `release/machine-x.y` chỉ khi cần vá máy cũ. Monorepo giữ được thay đổi atomic qua 3 bên.
- **SemVer:** public API của hệ là **giao thức**; `protocol_version` tách khỏi phiên bản từng sản phẩm; mỗi sản phẩm SemVer riêng; bảng tương thích trong repo; server hỗ trợ N-1 hoặc N-2 major giao thức, gợi ý giữ major cũ 6–12 tháng (cần user quyết).
- **Deprecate (Kubernetes):** GA không gỡ trong cùng major; beta gỡ sau 9 tháng hoặc 3 minor; áp dụng nhãn `experimental`/`stable` cho trường/endpoint.
- **Test:** pyramid; contract bằng fixture JSON chung; fake driver phần cứng trong CI; hardware-in-the-loop để sau.
- **DORA:** hiện 5 chỉ số (lead time, deployment frequency, failed deployment recovery time, change fail rate, deployment rework rate); đội nhỏ theo dõi tay lead time và change fail rate.
- **AI agent:** plan trước, phạm vi thư mục rõ, CODEOWNERS + CI làm hàng rào, agent không sửa `contracts/` khi không có ADR/PR riêng, PR do AI viết vẫn có người review, bug fix kèm test.

Gói tối thiểu bản đầu: `contracts/` (schema, `protocol_version`, fixture); `docs/adr/`; 2 sơ đồ C4; mẫu mini design doc; GitHub flow + CODEOWNERS; CI có path filter; SemVer + tag riêng + changelog + bảng tương thích; file hướng dẫn cho AI agent; runbook + mẫu postmortem; issue có nhãn.

Không làm sớm: GitFlow; RFC nhiều tầng duyệt; arc42 đầy đủ, C4 cấp component/code; Backstage, dashboard DORA, trực 24/7; Pact đầy đủ, HIL; tách repo từng sản phẩm; gom 3 sản phẩm một version chung; ADR cho mọi chuyện nhỏ; cho AI sửa hợp đồng hay merge không qua người.

### Nguồn vòng 2

| Nguồn | Ý chính | Link | Kiểm |
|---|---|---|---|
| AWS IoT Device Shadow | desired/reported/delta, `version` chống tin cũ, app ghi desired, máy ghi reported | https://docs.aws.amazon.com/iot/latest/developerguide/iot-device-shadows.html | mở |
| AWS IoT Jobs | Rollout có đích, trạng thái từng máy | https://docs.aws.amazon.com/iot/latest/developerguide/iot-jobs.html | nhớ |
| Azure device twin | tags/desired/reported, `$version`/etag, giới hạn 8KB/32KB, không giữ thông báo cho máy offline | https://learn.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-device-twins | mở |
| Azure direct methods | Lệnh đồng bộ, cần máy online | https://learn.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-direct-methods | nhớ |
| Android A/B | Hai slot, bootloader quay về slot cũ, `update_verifier` | https://source.android.com/docs/core/ota/ab | mở |
| Mender / balena | OTA A/B, rollback, nhóm triển khai, update lock khi đang làm việc | https://docs.mender.io · https://docs.balena.io | nhớ |
| Tesla vehicle-command | Server cho qua chưa đủ, xe tự kiểm chữ ký lệnh bằng khoá đã đăng ký | https://github.com/teslamotors/vehicle-command | tìm |
| Home Assistant | Core nhỏ: event bus, state machine, service registry; integration cắm vào; local-first | https://developers.home-assistant.io/docs/architecture_index · https://developers.home-assistant.io/docs/dev_101_index | mở (mỏng) |
| Matter | Local-first, fabric với root CA riêng, nhiều admin, OTA trong chuẩn | https://csa-iot.org/all-solutions/matter/ · https://www.home-assistant.io/integrations/matter/ | tìm |
| Philips Hue | Bridge cục bộ là gốc, cloud là đường phụ | developers.meethue.com | nhớ (chỉ trang login) |
| Square / Toast offline | Lưu đơn cục bộ, đồng bộ sau, có giới hạn thời gian/số tiền | — | nhớ (Square 404) |
| Fowler Monolith First | Bắt đầu monolith mô-đun vì ranh giới chưa rõ | https://martinfowler.com/bliki/MonolithFirst.html | mở |
| Richards & Ford, Fundamentals of Software Architecture | Danh mục style; service-based là điểm giữa | sách | nhớ |
| Google design docs (Malte Ubl) | Chỉ viết khi mơ hồ; goals/non-goals/alternatives | https://www.industrialempathy.com/posts/design-docs-at-google/ | mở |
| Google SWE Book ch.9 Code Review | PR nhỏ, 1 reviewer, owner, tự động hoá | https://abseil.io/resources/swe-book/html/ch09.html | mở |
| Nygard ADR | 5 phần, đánh số, superseded | https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions | mở |
| C4 model | context, container, component, code | https://c4model.com | mở |
| Shape Up | appetite, cycle 6 tuần, cool-down | https://basecamp.com/shapeup | mở |
| DORA | 5 chỉ số | https://dora.dev/guides/dora-metrics-four-keys/ | mở |
| SemVer | public API, deprecate trước khi gỡ ở major | https://semver.org | mở |
| Kubernetes deprecation policy | Thời hạn theo mức ổn định | https://kubernetes.io/docs/reference/using-api/deprecation-policy/ | mở |
| Uber RFC, Spotify squad/Backstage, ThoughtWorks Radar (lightweight ADR), Accelerate, Pact, arc42 | — | — | nhớ |
