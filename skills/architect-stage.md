---
name: architect-stage
description: Stage architect. Dùng khi user bảo tổng hợp research thành kiến trúc (cốt lõi), "chuyển sang architect", "chốt kiến trúc". Operator kiểm đủ tiêu chí, hỏi user theo lượt tới khi hết xung đột và chỗ thiếu; user nói "đủ rồi, tự quyết đi" thì tự chốt, tự bổ sung thành bản kiến trúc hoàn chỉnh; rồi đóng băng và giao planner kèm luật không hỏi lại thiết kế.
source: user chốt 12/10/2026 (quy tắc gốc ở HIEN_PHAP.md, mục Stage architect và Planner không hỏi lại thiết kế)
status: confirmed
---

# Stage architect

Quy tắc gốc nằm ở [HIEN_PHAP.md](../HIEN_PHAP.md), mục *Stage architect* và *Planner không hỏi lại thiết kế*. File này là cách chạy. Bộ tiêu chí, mẫu câu hỏi và mẫu giao việc cho planner do agent đề xuất ngày 12/10/2026; user muốn đổi thì sửa ở đây.

Lời user ngày 12/10/2026: "khi tôi kêu tổng hợp research thành một kiến trúc cốt lõi, agent nên hỏi tôi nhiều hơn để có thể hoàn thiện các tiêu chí cần thiết, chốt lại các phần cần thiết, giải quyết các xung đột và chốt phương án tốt nhất, nếu tôi đủ rồi, tự quyết đi thì tự động chốt lại bản kiến trúc hoàn chỉnh, tự bổ sung các phần thiếu. đến stage plan, không được phép hỏi hay chỉnh sửa gì nữa, chỉ dựa vào kiến trúc và chia việc thôi, hiện tại planner hỏi rất nhiều và làm lệch hướng architect, điều này rất nguy hiểm. chỉ khi logic quan trọng thì mới được hỏi"

## Ai làm gì

- **Operator** chạy stage. Đây là agent duy nhất hỏi user và viết bản kiến trúc cuối.
- **Architect phụ** (tùy chọn): đọc research trong repo và code, trả về bản nháp cùng danh sách xung đột và chỗ thiếu. Không đọc vault; prompt không chứa nội dung vault (theo [RULES.md](../RULES.md)).
- Operator tự đối chiếu thêm với quyết định đã chốt trong vault (`facts/`, note phiên). Xung đột hay nằm ở chỗ này.

## Bước 1 · Bảng kiểm tiêu chí

Mỗi tiêu chí một dòng, gán một trạng thái: ✓ đã chốt · ◐ research đề xuất, user chưa chốt · ✗ xung đột · ○ thiếu.

| # | Tiêu chí | Phải trả lời được |
|---|---|---|
| T1 | Mục tiêu và chuẩn đạt | Giải bài toán gì; nhìn thế nào là đạt |
| T2 | Phạm vi bản này | Làm gì, không làm gì, ranh giới MVP |
| T3 | Ràng buộc | Quyết định cũ phải giữ; nền tảng, phần cứng, mạng, con người |
| T4 | Khối và trách nhiệm | Mỗi khối làm gì, không làm gì |
| T5 | Nguồn sự thật | Dữ liệu nào nằm ở đâu; ai được ghi; bản sao nào, hết hạn ra sao |
| T6 | Luồng chính | Luồng đầu-cuối: ai nói với ai, theo thứ tự nào |
| T7 | Giao tiếp giữa khối | Khối A đưa gì cho khối B, cam kết gì |
| T8 | Quyền và tin cậy | Ai được làm gì; xác minh danh tính thế nào; cần bảo vệ gì |
| T9 | Lỗi và trạng thái bất thường | Mất mạng, quá hạn, gửi lại, xung đột, khôi phục |
| T10 | Thông số cấu hình | Ngưỡng, thời hạn, giá trị mặc định |
| T11 | Đánh đổi và rủi ro chấp nhận | Bỏ gì để được gì; rủi ro còn lại |
| T12 | Cách nghiệm thu | Kiểm thế nào để biết kiến trúc đạt |

- Tầng 1 trả lời bằng ngôn ngữ khối và luồng, không nêu thư viện hay định dạng gói tin. Tầng 2 (một khối) mới đi vào phương pháp, giao thức, thuật toán.
- Tiêu chí không áp dụng thì ghi "không áp dụng" kèm lý do, không bỏ trống.
- Xung đột cần dò: research ↔ research; research ↔ quyết định đã chốt; quyết định ↔ quyết định; thiết kế ↔ code thật.

Báo user trong chat: số mục mỗi trạng thái và danh sách mục chưa ✓.

## Bước 2 · Hỏi theo lượt

- Mỗi lượt khoảng 8 câu. Phiên 12/10 user trả lời tốt dạng 8 câu (a)/(b) có khuyến nghị.
- Thứ tự: ✗ xung đột → ○ và ◐ ảnh hưởng nhiều khối → phần còn lại.
- Mẫu một câu:

```text
3. [T5 · xung đột] Bản sao quyền trên máy hết hạn khi đang offline thì sao?
   Bối cảnh: research A khuyên …; quyết định 11/10 nói …
   (a) … : được …, mất …
   (b) … : được …, mất …
   Khuyến nghị: (a), vì …
```

- User trả lời ngắn ("1a 2b 3: …") là đủ. Câu user bỏ qua thì dời sang lượt sau, không coi là đồng ý.
- Lựa chọn mới đụng một quyết định khác thì phản biện: nêu đúng hai điểm đụng nhau và hỏi giữ bên nào.
- Không đề xuất lại phương án user đã bác nếu không có lý do mới.
- Mục ◐ không được ngầm coi là đã chốt chỉ vì research nghiêng về một phía. Thà hỏi thêm còn hơn đoán.
- Cuối mỗi lượt báo: "Còn N mục mở: T…, T…". Hết mục mở thì tóm kiến trúc trong vài câu và hỏi chốt.

## Bước 3 · Ủy quyền "tự quyết"

Nhận diện: "đủ rồi", "tự quyết đi", "chốt đi", "khỏi hỏi nữa" hoặc ý tương đương. Nếu user chỉ nói một phần ("phần offline tự quyết") thì chỉ áp cho phần đó và vẫn hỏi phần còn lại.

Mỗi mục còn mở chọn theo thứ tự:
1. khớp mọi quyết định đã chốt;
2. bằng chứng research, case study mạnh nhất;
3. cách đơn giản nhất đủ đạt T1 (không overengineer);
4. hòa nhau thì chọn cách dễ đổi về sau.

Tiêu chí ○ thiếu: tự viết nội dung tối thiểu đủ dùng, theo cùng thứ tự trên.

Ghi nhãn mỗi mục `[agent tự chốt theo ủy quyền DD/MM]` kèm một dòng lý do. Bước này không hỏi thêm câu nào. Báo trong chat danh sách mục tự chốt (mỗi mục một dòng) và đường dẫn bản kiến trúc.

## Bước 4 · Đóng băng bản kiến trúc

File `<repo>/documentation/architect/<chủ-đề>.html`, theo [chuẩn FexMix](../document-framework/manual/STANDARD.md), có chú thích thuật ngữ. Bắt buộc có:
- dòng trạng thái "Đã chốt DD/MM/YYYY · phiên bản vN";
- sơ đồ khối và luồng chính, đúng mức của tầng;
- bảng quyết định: tiêu chí → quyết định → nguồn (user DD/MM · agent tự chốt theo ủy quyền · đã chốt trước đó);
- mục "Ngoài phạm vi bản này".

Không được còn "câu hỏi mở", "TBD", "chờ quyết" hay hai phương án song song chưa chọn. Kiểm trước khi giao: T1–T12 đều ✓ hoặc "không áp dụng"; tìm trong doc không còn các cụm trên.

Sau đó cập nhật `STATUS.md` của repo (một dòng nhật ký) và fact quyết định nếu project có note, theo [RULES.md](../RULES.md).

## Bước 5 · Giao planner

Prompt cho planner chỉ gồm: đường dẫn bản kiến trúc đã chốt kèm phiên bản, phạm vi cần lập lộ trình, đường dẫn code, và khối luật dưới đây. Không kèm research.

```text
Luật planner (HIEN_PHAP, 12/10/2026):
- Chỉ dựa vào bản kiến trúc <đường dẫn> vN và code hiện có. Không đọc research để chọn lại phương án.
- Không hỏi user về thiết kế. Không đề xuất kiến trúc, khối, luồng hay tính năng khác. Không sửa bản kiến trúc. Không mở lại quyết định đã chốt.
- Tự quyết chi tiết thực thi trong khung kiến trúc: file, tên, cây thư mục, thứ tự task, xử lý lỗi và ca biên trong một khối, cách kiểm. Chỗ nhỏ chưa rõ thì chọn cách đơn giản nhất khớp kiến trúc và ghi vào mục "Giả định của planner".
- Chỉ dừng và báo khi gặp logic quan trọng:
  (1) kiến trúc thiếu hoặc mâu thuẫn ở điểm làm đổi hành vi giữa các khối, quyền và bảo mật, hoặc tính đúng của dữ liệu, và không suy ra được từ chính bản kiến trúc;
  (2) code hoặc nền tảng cho thấy kiến trúc không chạy được như đã viết.
  Khi đó gom thành một lượt; mỗi mục nêu chỗ trong kiến trúc, vì sao chặn, các hướng. Không tự chọn. Phần không bị ảnh hưởng vẫn làm tiếp.
```

Operator nhận câu hỏi của planner:
- thuộc (1) hoặc (2): đưa user, cập nhật bản kiến trúc lên vN+1, giao planner làm tiếp;
- không thuộc: trả planner "tự quyết theo khung, ghi giả định", không chuyển cho user.

## Bản chạy được

Chép hoặc symlink file này thành `~/.claude/skills/architect-stage/SKILL.md` (cùng cách với research-stage) để Claude Code tự nhận khi user bảo tổng hợp research thành kiến trúc.
