---
id: session-20261013-linux-server-and-diagram-standard
type: session-learning
scope: project:project1_app
status: extracted
date: 2026-10-13
source: current-conversation
---

# Mục tiêu và chuẩn đạt

User hỏi việc còn lại của `project1_app`, cách chạy server trên Linux, rồi yêu cầu có tài liệu Linux và chỉnh sơ đồ tài liệu cho dễ đọc. Cuối phiên user yêu cầu lưu phiên bằng `session-secondbrain`.

## Những lần chỉnh hướng

- User chỉ ra repo chưa có hướng dẫn Linux. Agent tạo `server/SETUP_LINUX.md` và liên kết từ `server/README.md` thay cho câu trả lời chỉ có lệnh trong chat.
- User chỉ ra Mermaid tự ngắt `MACHINE_SHARE_CREATE` giữa chữ và dồn nhiều việc trong một nhãn. Agent tắt tự bẻ dòng của sơ đồ trình tự; việc nào có nhiều ý thì xuống dòng dạng list có chủ ý.
- User chỉ ra xử lý nội bộ của Server không cần mũi tên tự quay. Agent đổi phần hiển thị thành một nút số trên trục, giữ thứ tự số bước.
- User chốt nút số lớn 1,4 lần, rồi yêu cầu đưa quy tắc vào chuẩn trong second brain và áp dụng toàn bộ tài liệu liên quan.

## Quyết định và giới hạn

- [user-confirmed] Trong sơ đồ tài liệu, không ngắt giữa từ hoặc mã; nội dung gồm nhiều việc hiển thị từng dòng như list; xử lý nội bộ chỉ có nút số, không có mũi tên vòng; vòng tròn và chữ số của nút lớn 1,4 lần. Phạm vi áp dụng: chuẩn tài liệu gốc và các tài liệu hiện có với sơ đồ phù hợp.
- [source-observed] [Chuẩn active](../document-framework/manual/STANDARD.md), [template](../document-framework/manual/template.html) và [fact](../facts/document-standard-flexmix-manual.md) đã được cập nhật. `reference-FexMix_Munual.html` là bản chụp lịch sử của mẫu 09/10/2026, không dùng làm template active.
- [source-observed] Repo ở HEAD `0ec3a56` trong nhánh `clean-slate`; các thay đổi tài liệu của phiên chưa commit. File `documentation/serve_architect.py` đã bị sửa từ trước khi agent bắt đầu chỉnh tài liệu và không thuộc phần việc của phiên.

## Sản phẩm hiện hành

- [Hướng dẫn Linux](/home/abel/project/internproj/project1_app/server/SETUP_LINUX.md): MySQL, Python, chuyển khóa ứng dụng khi đổi server, Tailscale HTTPS, `.env`, chạy và kiểm kết nối. [README server](/home/abel/project/internproj/project1_app/server/README.md) có liên kết mới. Hướng dẫn dựa trên cấu hình/code repo; chưa chạy trên máy Linux đích.
- [Asset xử lý sơ đồ](/home/abel/project/internproj/project1_app/documentation/architect/assets/sequence-layout.js) áp dụng kích thước nút số và bỏ hình mũi tên tự quay. `doc.js` cùng 17 trang HTML có sơ đồ trình tự dùng quy tắc `wrap:false`; 31 nhãn xử lý nội bộ nhiều việc được đổi thành các dòng list. Một nhãn có dấu `;` khiến Mermaid không parse trong `features/contracts.html` đã được tách dòng.
- Kiểm bằng trình duyệt: cả 17 trang có sơ đồ trình tự đều render, không còn fallback về mã Mermaid; các nút số có bán kính 8,4 px; không còn vòng tự quay trong SVG. Ở viewport 420 px, sơ đồ cuộn trong khung và trang không tràn ngang. `node --check` cho JS active và JS trong template đạt; `git diff --check` sạch.

## Phần còn mở

- Chưa nghiệm thu server trên máy Linux thật, Android/Pi và Gmail thật; I-01 của project vẫn mở theo `projects/internproj/project1_app/STATUS.md` và `documentation/plan/mvp-fast-path.md`.
- Các thay đổi của phiên chưa commit. Không nhận file `documentation/serve_architect.py` là sản phẩm của phiên.
