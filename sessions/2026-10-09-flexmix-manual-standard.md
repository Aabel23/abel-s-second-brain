---
id: session-20261009-flexmix-manual-standard
type: session-learning
scope: project:internproj
status: extracted
date: 2026-10-09
source: current-conversation
---

# Mục tiêu và chuẩn đạt
User chỉnh các sơ đồ Mermaid trong `/home/abel/project/internproj/FexMix_Munual.html` (manual máy pha FlexMix), rồi chốt thiết kế của trang này làm chuẩn cho mọi document sau. Đạt = sơ đồ gọn, dễ đọc; chi tiết không nhồi vào khối; mũi tên gọn; màu có nghĩa.

## Những lần chỉnh hướng
- Agent (model Haiku) đọc sai chữ trong ảnh chụp sơ đồ và gợi ý tên lệch nội dung → khi nhận ảnh, đối chiếu với file nguồn trước khi gợi ý.
- Agent đổi tên khối thành "Khởi động & phục hồi", "Giao đơn cho runner" và in đậm → user: vấn đề là **nội dung bên trong** khối, không phải tên; tách chi tiết thành bảng nổi bên cạnh.
- Sau khi có bảng, user: **giữ tên gốc** ("Khởi động", "Dựng công thức (4.3)").
- User gọi sơ đồ trạng thái 4.7 là "bảng 4.7"; yêu cầu giữ luồng và thiết kế, chỉ restyle đẹp hơn.
- Mũi tên "có" trong sơ đồ 4.2 quá xéo → user muốn bớt xéo.

## Quyết định và giới hạn
- [user-confirmed] Thiết kế FexMix_Munual là chuẩn; mọi document sau phải cùng chất lượng → [facts/document-standard-flexmix-manual.md](../facts/document-standard-flexmix-manual.md).
- [user-confirmed] Khối sơ đồ chỉ ghi tên gốc + số ①②; chi tiết ở bảng `dg-side` cạnh sơ đồ.
- [source-observed] Mục 4.4 (`#c-flow`): `figure.dg.dg-split`, bảng sticky ≥1000 px, xuống dưới ở màn hẹp. CSS `.dg-split/.dg-main/.dg-side` thêm cạnh `.dg-tools`.
- [source-observed] Mục 4.7 (`#c-life`): stateDiagram tô màu `unused ui · in_progress proc · used ok · noqr_err bad · expired muted`, nhãn 2 dòng, đổi thứ tự cạnh để hết cắt nhau. Renderer giờ chèn `classDef` cho mọi sơ đồ có dòng `class`, thêm class `muted`.
- [source-observed] Mục 4.2 (`#c-ticket`): khai báo nhánh "có" trước, thêm `H ~~~ R` để kéo `raw_qr.json` gần hàng ô quyết định. Thử `--->` cho nhánh "không" làm sơ đồ xấu hơn, đã bỏ. Mũi tên vẫn hơi chéo.
- [source-observed] Kiểm bằng Chrome headless 1400 px và 420 px; cách chụp ghi trong STANDARD.md.

## Sản phẩm hiện hành
- `/home/abel/project/internproj/FexMix_Munual.html` (thư mục không phải git). Bản trước các sửa đổi ở scratchpad phiên, không bền.
- [document-framework/manual/STANDARD.md](../document-framework/manual/STANDARD.md), [template.html](../document-framework/manual/template.html) (đã render thử đủ component), [reference-FexMix_Munual.html](../document-framework/manual/reference-FexMix_Munual.html) (bản chụp 09/10/2026).

## Phần còn mở
- Mũi tên "có" ở 4.2 muốn nằm ngang hẳn thì phải đổi bố cục (409 rẽ xuống, hoặc "scanner ghi" đi vòng); user chưa chọn.
- Khối `Z` ("reset phần cứng · resume watcher · xoá raw_qr.json") ở 4.4 vẫn ghi chi tiết trong khối; chưa tách theo quy tắc mới.
