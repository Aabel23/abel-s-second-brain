---
id: project1-app-contracts-skeleton-research-2026-10-12
type: research-archive
scope: project:project1_app
status: source-observed
checked: 2026-10-12
source: two Sol-low researcher reports for C-02 and C-03
---

# Research hợp đồng skeleton · nhận máy và trạng thái trực tiếp

Bốn agent Sol low nghiên cứu theo `documentation/plan/tasks/C-02-claim-research.md`, `C-03-relay-status-research.md` và hai câu hỏi bổ sung về bootstrap/bảo vệ kênh, tính khả thi thư viện/transport. Chỉ đọc repo và nguồn công khai; không sửa code. Đây là kết quả nghiên cứu, **không phải quyết định của user**. Operator so với fact đã chốt trước khi chuyển thành hợp đồng.

## C-02 · Danh tính máy và nhận chủ

| Phương án | Điều kiện | Rủi ro / giới hạn |
|---|---|---|
| A. Xưởng cấp danh tính, PAKE, phiếu nhận chủ | Nạp khóa/chứng chỉ tại xưởng, giữ mã hộp bí mật | Người mở hộp trước vẫn có thể nhận chủ; khóa lưu file có thể bị chép. Đây là phương án researcher khuyên, **trái quyết định user 12/10: máy tự sinh khóa lần đầu, không đòi xưởng nạp chứng chỉ, mã dài không PAKE**. |
| B. A cộng quyền giao hàng | Hãng/đại lý quản lý hồ sơ cấp quyền | Chống nhận chủ trước tốt hơn nhưng thêm quy trình bán/đổi trả. |
| C. Trusted-user provisioning | Người có quyền cấp credential tạm | Không tự chứng minh máy chính hãng nếu không có gốc tin cậy khác. |
| D. Serial và khóa tự sinh khi bật đầu | Server tin đăng ký đầu hoặc có allowlist/gốc tin cậy khác | Serial/khóa tự sinh không chứng minh xuất xứ; giả máy có thể đăng ký trước. Gần quyết định user hơn, còn lỗ hổng cần bàn. |

Các ca cần chốt: hộp bị mở trước; bán lại dùng mã cố định mà chủ cũ đã biết; reset/thay bo có thể làm mất khóa; giả serial CPU; nghe lén Bluetooth; server thấy khóa lần đầu. Hai HTML `documentation/architect/cryptography.html` và `features/machine-activation.html` đang giả định chứng chỉ xưởng/PAKE; `machine_bluetooth_pair.py` hiện gửi `product_key` rõ, `machine_server_request.py` đặt key vào body HTTP. Route chỉ là hằng, chưa có handler.

| Nguồn case study / chuẩn | Liên hệ | Mức kiểm |
|---|---|---|
| [RFC 9383 · SPAKE2+](https://www.rfc-editor.org/rfc/rfc9383.html) | PAKE bảo vệ trao đổi mã ngắn, không cấp xuất xứ máy hay quyền Owner | Researcher dẫn nguồn trực tiếp; Operator chưa kiểm lại nội dung nguồn |
| [RFC 8995 · BRSKI](https://www.rfc-editor.org/rfc/rfc8995.html) | Tách danh tính xưởng khỏi quyền gia nhập; không phù hợp bê nguyên quy trình vào bán lẻ | Researcher dẫn nguồn trực tiếp; Operator chưa kiểm lại nội dung nguồn |
| [AWS IoT · provisioning by trusted user](https://docs.aws.amazon.com/iot/latest/developerguide/provision-wo-cert.html) | Credential tạm do người tin cậy cấp; vẫn cần giải bài toán xuất xứ | Researcher dẫn nguồn trực tiếp; Operator chưa kiểm lại nội dung nguồn |

## C-03 · Relay và trạng thái máy

`MACHINE_STATUS_GET` trong `features/command-relay.html` trả online/last-seen do server suy từ poll. Skeleton cần một câu hỏi **mới tới máy qua route relay chung**, có ID tương quan và câu trả lời từ máy. Đề xuất: app sinh ID mới mỗi lần bấm refresh; server kiểm phiên/quyền trên đúng máy, giữ pending trong RAM với deadline; máy nhận qua poll, đo trạng thái lúc xử lý; server chỉ ghép kết quả đúng ID và máy trước deadline. Offline trả ngay; kết quả về muộn không đổi timeout hay trôi sang lượt hỏi mới. Gửi lại cùng ID là cùng phép đo, không có nghĩa đo lại; cần đánh dấu replay/độ mới nếu trả kết quả cũ. Đây là đề xuất kỹ thuật, chưa chốt gói hay timeout.

Code hiện tại: `server/src/` trống; `version1.1/machine/main.py` mới có handler menu/kho, chưa có trạng thái; `machine_server_request.py` timeout 10 giây chung; `flutter_app/lib/feature/dashboard/ui/machines_tab.dart` mới có bool online. Phần lưu bền ID→kết quả cho lệnh ghi chưa cần cho câu hỏi chỉ đọc ở skeleton.

| Nguồn case study / chuẩn | Liên hệ | Mức kiểm |
|---|---|---|
| [Azure IoT Hub · Direct methods](https://learn.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-direct-methods) | Hỏi đồng bộ máy đang kết nối, có timeout; tham chiếu semantics, không buộc đổi transport | Researcher dẫn nguồn trực tiếp; Operator chưa kiểm lại nội dung nguồn |
| [Azure IoT Hub · Device twins](https://learn.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-device-twins) | Bản lưu trạng thái cloud khác câu trả lời mới từ máy | Researcher dẫn nguồn trực tiếp; Operator chưa kiểm lại nội dung nguồn |
| [RFC 6202 · HTTP long polling](https://www.rfc-editor.org/rfc/rfc6202) | Máy sau router chủ động mở request chờ lệnh, cần deadline và poll lại | Researcher dẫn nguồn trực tiếp; Operator chưa kiểm lại nội dung nguồn |

## Bổ sung · mã niêm phong dài, khóa tự sinh, kênh bảo vệ

Researcher xét đúng hướng user chốt sau C-04: serial và bản kiểm mã `S` đã đăng ký trên server; cùng mã dài trong máy và QR; máy tự sinh khóa lần đầu; không có chứng chỉ/khóa xưởng hoặc PAKE.

- BLE cấp wifi: cần kênh có xác thực bằng mã ngẫu nhiên mạnh và AEAD trước khi chuyển wifi. Không gửi `S` rõ trên BLE; BlueZ bond hoặc gói `identify` hiện tại không đủ. TLS 1.3 external PSK + DHE là một chuẩn có thể xét nếu triển khai được trên BLE; tránh tự ghép HMAC/ECDH/AES thành giao thức mới. Mã phải có entropy từ CSPRNG, không chỉ dài về ký tự.
- Bootstrap cloud: máy gửi `S` trong HTTPS đến tiến trình server, hoặc thêm phong bì HPKE nếu cần che khỏi proxy; server so với bản kiểm mã đã đăng ký, kiểm challenge một lần và chữ ký khóa máy mới trên serial, khóa công khai, challenge và context; ghi `serial → public_key` nguyên tử. Đăng ký khóa và nhận Owner là hai bước khác nhau.
- Server chỉ giữ `hash(S)` thì **không** kiểm được `HMAC_S(challenge)`; dùng `hash(S)` làm khóa HMAC biến bản kiểm mã thành credential khi DB lộ. Nếu gửi `S` trong TLS để so hash, server sẽ nhìn thấy `S`; tài liệu cũ nói server không biết mã không còn đúng.
- Sau bootstrap, HTTPS + chữ ký ứng dụng bằng khóa máy đã đăng ký là phương án nhẹ hơn mTLS; mTLS vẫn có thể dùng nhưng cần cấp cert vận hành và quản lý hạn/thu hồi sau bootstrap. Cả hai cần ràng method/path/body/nonce và chống replay. Chữ ký ứng dụng cho backend tự kiểm dù TLS kết thúc ở proxy.
- Giữ phong bì HPKE bên trong TLS là khả thi với danh tính máy mới nhưng phải đặc tả riêng xác thực phản hồi, freshness và replay. Bỏ lớp này đổi ranh giới tin cậy vì proxy có thể đọc token/nội dung. **Đây là lựa chọn thiết kế user cần quyết**, không phải Planner tự tối ưu.

Rủi ro còn: mã bị lộ trước bootstrap cho phép đua đăng ký khóa; mã cố định không chống clone khi kẻ xấu biết mã và có máy; reset wifi không được xóa khóa/Owner; mất khóa hoặc thay bo không được tự gắn lại bằng mã cũ nếu chưa có quy trình phục hồi.

| Nguồn case study / chuẩn | Liên hệ | Mức kiểm |
|---|---|---|
| [RFC 9257 · TLS external PSK](https://www.rfc-editor.org/rfc/rfc9257.html), [RFC 9258 · TLS PSK importers](https://www.rfc-editor.org/rfc/rfc9258.html) | Kênh PSK mạnh cho onboarding, vẫn phải kiểm thư viện/transport BLE | Researcher dẫn nguồn trực tiếp; Operator chưa kiểm lại nội dung nguồn |
| [Espressif provisioning](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/provisioning/provisioning.html) | Proof of possession và kênh mã hóa cấp wifi; Security 2 dùng SRP/PAKE nên không phù hợp quyết định hiện tại | Researcher dẫn nguồn trực tiếp; Operator chưa kiểm lại nội dung nguồn |
| [RFC 9421 · HTTP Message Signatures](https://www.rfc-editor.org/rfc/rfc9421.html), [RFC 8446 · TLS 1.3](https://www.rfc-editor.org/rfc/rfc8446.html), [RFC 9180 · HPKE](https://www.rfc-editor.org/rfc/rfc9180.html) | Chữ ký request sau bootstrap, TLS và phong bì lớp trong; mỗi chuẩn không tự giải chống replay nghiệp vụ | Researcher dẫn nguồn trực tiếp; Operator chưa kiểm lại nội dung nguồn |

## Tình trạng

C-02/C-03 và C-04 xong; user chốt đăng ký mã/serial trước khi bán và **giữ TLS + phong bì mã hóa bên trong ngay skeleton**. Planner C-05 đang làm hợp đồng. Chưa phát phiếu coder. Trạng thái trực tiếp cần hợp đồng tách online/cache khỏi kết quả máy.

## Bổ sung · khả thi thư viện và transport (agent Sol low thứ tư)

Researcher kiểm repo và upstream ngày 12/10/2026, chưa chạy PoC: Flutter chỉ có `mobile_scanner`, `qr_flutter`; chưa có HPKE/BLE. Android target Java/Kotlin 17, máy hiện BlueZ Bluetooth Classic SPP/RFCOMM với `NoInputNoOutput`, `version1.1/requirements.txt` có `cryptography` chưa pin; server repo mới chỉ có routing. Phải thay transport nhận máy thành BLE GATT và thêm adapter crypto, không coi là đã có.

| Phần | Hướng khả thi theo API upstream | Mức kiểm / giới hạn |
|---|---|---|
| Python HPKE | [PyHPKE 0.6.5](https://github.com/dajiaji/pyhpke), [API](https://pyhpke.readthedocs.io/en/latest/api.html): Base/PSK/Auth/AuthPSK, suite RFC 9180 | Researcher xem upstream; chưa kiểm liên thông với Android |
| Android HPKE | [rust-hpke 0.14.1](https://github.com/rozbb/rust-hpke) qua native/FFI; BouncyCastle Kotlin còn đang kiểm | Có API sender/receiver; cần build ABI/wrapper, chưa là dependency sẵn |
| BLE bảo vệ | [snow 0.10.0](https://github.com/mcginty/snow), [PSK API](https://docs.rs/snow/latest/snow/struct.Builder.html), Noise `NNpsk0_25519_ChaChaPoly_SHA256` | Message API khớp GATT; chưa kiểm Android↔Pi; không tự ghép primitive |
| Linux BLE | [BlueZ GATT](https://bluez.readthedocs.io/en/latest/gatt-api/), [Bless](https://github.com/kevincar/bless); Python [noiseprotocol 0.3.1](https://github.com/plizonczyk/noiseprotocol) hoặc binding snow | Noise Python cũ/revision khác, cần vector liên thông; Bless chưa chứng minh Python 3.14 của máy |
| Android BLE | [BluetoothGatt](https://developer.android.com/reference/android/bluetooth/BluetoothGatt) qua native hoặc plugin | Cần phân mảnh/ráp frame và thử MTU thấp; không dùng BlueZ bond làm xác thực |

TLS 1.3 external PSK trên GATT có chuẩn nhưng Dart `SecureSocket` không có PSK/custom BLE transport, Python SSL PSK chỉ từ 3.13 và cần OpenSSL/BIO adapter; không là đường triển khai sẵn. Nguồn: [Python SSL](https://docs.python.org/3.14/library/ssl.html), [Dart SecureSocket](https://api.dart.dev/dart-io/SecureSocket/secureServer.html), [OpenSSL PSK callbacks](https://docs.openssl.org/3.0/man3/SSL_CTX_set_psk_client_callback/). Pure Dart `turnkey_crypto` có profile/wire riêng, chưa chứng minh generic RFC 9180 ([source](https://github.com/tkhq/dart-sdk/blob/main/packages/crypto/lib/src/hpke.dart)).

Ca kiểm trước coder/hoặc gate đầu phiếu: RFC 9180 vectors; Python↔Android cùng suite/mode cả hai chiều, tamper/wrong key; Noise wrong PSK/prologue/replay; Android↔Pi thật với MTU 23, frame đứt, reconnect; reset giữ khóa máy. Kết luận **khả thi theo API công khai, chưa xác minh tích hợp/hardware**. Researcher đang kiểm riêng API BouncyCastle Kotlin vì Planner đề xuất; bổ sung kết quả khi về.

### Kiểm tiếp BouncyCastle 1.86

Researcher đọc source tag `r1rv86` và Javadoc upstream: [HPKE.java](https://github.com/bcgit/bc-java/blob/r1rv86/core/src/main/java/org/bouncycastle/crypto/hpke/HPKE.java), [HPKE API](https://downloads.bouncycastle.org/java/docs/bcprov-jdk18on-javadoc/org/bouncycastle/crypto/hpke/HPKE.html), [context API](https://downloads.bouncycastle.org/java/docs/bcprov-jdk18on-javadoc/org/bouncycastle/crypto/hpke/HPKEContext.html), [download](https://www.bouncycastle.org/download/bouncy-castle-java/), [license MIT](https://github.com/bcgit/bc-java/blob/r1rv86/LICENSE.html). BC có Base/PSK/Auth/AuthPSK và exporter; method PSK sender là `SetupPSKS`, receiver `setupPSKR`. Kotlin MethodChannel truyền byte sang BC Java lightweight API, không cần Rust/NDK; PyHPKE phía Python có context tương ứng. Đây là lựa chọn cloud có ít lớp tích hợp hơn rust-hpke FFI, nhưng chưa chạy vector/Android API-level thực.

**Giới hạn:** HPKE PSK + exporter không tự tạo handshake hai chiều, key confirmation, freshness/replay hay transport sequencing cho BLE. Nếu lấy exporter rồi tự dựng AES-GCM channel, đó là thiết kế giao thức mới. Researcher khuyến nghị BC↔PyHPKE cho **phong bì cloud**, Noise PSK implementation có handshake + transport cho **kênh BLE**. Mọi tuyên bố liên thông vẫn phải qua vector và Android↔Pi thật.
