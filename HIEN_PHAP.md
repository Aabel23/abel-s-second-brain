---
id: workflow-constitution
type: constitution
scope: global
status: confirmed
source: current-conversation, user chốt 2026-10-10
valid_from: 2026-10-10
updated: 2026-10-12
supersedes: ""
---

# Hiến pháp làm việc: từ ý tưởng tới triển khai

> **Văn bản quan trọng nhất của second brain** (user chốt 10/10/2026). Mọi note, fact, session, skill, project map và archive khác trong vault chỉ là **kho lưu trữ kèm theo**: dùng để tra cứu, không được đi ngược hiến pháp. Chỉ yêu cầu hiện tại do chính user nói rõ mới đứng trên hiến pháp.

User chốt ngày 10/10/2026: đây là **luồng làm việc chính, khóa cứng**, áp dụng cho mọi ý tưởng, hệ thống hay tính năng mới cần thiết kế. Chỉ bỏ hoặc đổi một bước khi user nói rõ trong yêu cầu hiện tại. Agent không được tự rút ngắn quy trình vì thấy việc "đơn giản".

Nguyên tắc gốc: **hiểu hệ thống trước, kỹ thuật sau.** Đưa chi tiết kỹ thuật vào quá sớm sẽ làm rối luồng tư duy của user.

## Tầng 1 · System design (chưa bàn kỹ thuật)

Ngôn ngữ ở tầng này: khối, trách nhiệm của khối, ai nói chuyện với ai, dữ liệu chảy theo hướng nào, đánh đổi. **Không** nêu thư viện, tên hàm, tham số, định dạng gói tin, thuật toán cụ thể, trừ khi user hỏi.

1. **Nghe và bàn qua chat.** User nêu ý tưởng; agent hỏi, giải thích và phản biện ở mức system design. Chưa chạy agent khác, chưa viết file, chưa code. Bước này xong khi agent đã tóm phương án trong vài câu và **user xác nhận đã hiểu và chốt phương án**.
2. **Operator đặc tả sơ bộ.** Operator (agent điều phối chính) viết đặc tả cấu trúc dự án, rồi chia hệ thống thành các khối thành phần nhỏ, mỗi khối có trách nhiệm rõ.
3. **HTML architect để user đọc.** Chỉ gồm: sơ đồ khối toàn hệ thống (không chi tiết) và các luồng làm việc đề xuất. Theo chuẩn thiết kế [document-standard-flexmix-manual](facts/document-standard-flexmix-manual.md). Không liệt kê "cái gì là cái gì" ở mức chi tiết.
4. **User đọc, hỏi, sửa.** Operator sửa HTML theo phản hồi và lặp tới khi user chốt sơ đồ.
5. **Deep research song song.** Mỗi khối trong thiết kế đã chốt giao cho **một agent researcher riêng, chạy song song, model Sonnet effort medium** (user đổi từ Opus 5.5 ngày 11/10/2026). Mỗi agent có hai mục tiêu:
   - tìm giải pháp tốt nhất cho phần kiến trúc được giao;
   - nghiên cứu case study: các công ty lớn làm thế nào, phương pháp tiêu chuẩn là gì.
   Researcher và architect **không được truy cập vault** (xem [RULES.md](RULES.md)). Prompt giao việc chỉ gồm câu hỏi và phạm vi code, không chép nội dung hay kết luận từ vault.
6. **Tổng hợp và đề xuất cải tiến.** Operator gộp kết quả, đánh giá, đề xuất cải tiến hệ thống, rồi cập nhật HTML (vẫn ở mức system design). User phản biện. **Lặp bước 4–6 bao nhiêu vòng cũng được** cho tới khi chốt thiết kế chính. Agent đồng thuận với nhau không thay cho user chốt. Khi user bảo tổng hợp research thành kiến trúc, chạy theo mục *Stage architect* bên dưới.
7. **Cổng sang tầng 2.** Chỉ đi tiếp khi đủ ba điều: thiết kế chính đã chốt; tài liệu system design đơn giản, dễ hiểu; user xác nhận đã hiểu hệ thống.

## Tầng 2 · Kỹ thuật thực thi, đi sâu từng khối

Đến đây mới bàn kỹ thuật: cần phương pháp, thuật toán, giao thức hay công cụ nào cho từng bài toán. Mỗi khối chạy lại cùng một chu trình:

**research → case study → architect → document → debate → improvement → final decision → execution**

- *Debate*: các agent phản biện nhau, rồi user phản biện. Chỉ user ra *final decision*.
- *Architect → final decision* chạy theo mục *Stage architect* bên dưới.
- *Model agent* (user chốt 11/10/2026): researcher và agent viết code (coder) chạy **Sonnet effort medium**. **Planner chạy Opus effort high**, vì planner phải viết đặc tả và hướng dẫn chi tiết trước khi coder bắt tay vào code.
- *Ranh giới planner / coder* (user chốt 11/10/2026):
  - **Planner** viết plan thực thi đầy đủ: đặc tả, file nào, hàm nào, hợp đồng dữ liệu, thứ tự bước, ca kiểm và cách kiểm. Plan phải chi tiết tới mức một coder Sonnet làm theo sát được mà không phải tự quyết thiết kế. Mọi lựa chọn thực thi (cấu trúc, tên, xử lý lỗi, trường hợp biên) là việc của planner, nhưng chỉ trong khung kiến trúc đã chốt; planner không đổi kiến trúc (xem *Planner không hỏi lại thiết kế*).
  - **Coder** chỉ giải đúng và tối thiểu bài toán planner đặt ra. Không thêm tính năng, lớp trừu tượng, cấu hình, "chừa sẵn cho sau này", không sửa ngoài phạm vi plan. Plan thiếu, mơ hồ hoặc sai thì dừng và báo lại planner, không tự lấp chỗ trống.
- *Execution* đi theo **chuỗi thực thi sau architect** ở mục dưới. Quy trình riêng của dự án (ví dụ `project1_app`: `AGENTS.md`, `agent_workspace/TEAM.md`) chỉ bổ sung chi tiết, không bỏ bước nào của chuỗi này.
- Lỗi lúc triển khai mà đòi đổi thiết kế thì quay về bước architect/document của khối đó, đưa user chốt lại.

## Stage architect · chốt kiến trúc (user chốt 12/10/2026)

Chạy khi user bảo tổng hợp research thành kiến trúc (tầng 1 bước 6, hoặc bước architect → final decision của một khối ở tầng 2). Đây là **stage cuối cùng được hỏi user về thiết kế**: sau stage này kiến trúc đóng băng, planner chỉ chia việc. Vì vậy agent phải **hỏi nhiều và kỹ hơn** ở đây, không để chỗ trống cho planner tự lấp.

Operator chạy stage và là agent duy nhất hỏi user. Architect phụ (nếu cử) chỉ dựng bản nháp và danh sách xung đột, chỗ thiếu từ research trong repo và code; vẫn không đọc vault. Cách chạy, bộ tiêu chí và mẫu câu hỏi: [skills/architect-stage.md](skills/architect-stage.md).

1. **Kiểm đủ tiêu chí.** Đối chiếu research, quyết định đã chốt và code với bộ tiêu chí của một kiến trúc hoàn chỉnh. Mỗi tiêu chí mang một trong bốn trạng thái: đã chốt; research đề xuất nhưng user chưa chốt; xung đột; thiếu.
2. **Hỏi theo lượt tới khi hết mục mở.** Mỗi lượt gom câu hỏi đánh số, phương án (a)/(b) kèm đánh đổi và khuyến nghị. Hỏi xung đột trước, rồi tới chỗ thiếu ảnh hưởng nhiều khối. Phản biện khi lựa chọn của user đụng một quyết định khác. Không tự lấp chỗ trống, không ngầm chọn hộ. Sau mỗi lượt báo còn bao nhiêu mục mở.
3. **User nói "đủ rồi, tự quyết đi" (hoặc ý tương đương) thì dừng hỏi.** Agent tự chốt mọi mục còn mở và tự bổ sung phần thiếu, chọn theo thứ tự: khớp quyết định đã chốt → bằng chứng research, case study mạnh nhất → cách đơn giản nhất đủ đạt mục tiêu (không overengineer) → dễ đổi về sau. Mỗi mục ghi nhãn "agent tự chốt theo ủy quyền" kèm một dòng lý do. Ủy quyền này **là** user chốt: bản kiến trúc thành bản đã chốt, không hỏi lại. User ủy quyền một phần ("phần X tự quyết") thì chỉ áp cho phần đó.
4. **Đóng băng.** Bản kiến trúc hoàn chỉnh nằm trong `documentation/architect/`: không còn câu hỏi mở hay "TBD"; có bảng quyết định (tiêu chí → quyết định → nguồn: user, hoặc agent tự chốt theo ủy quyền); việc không làm ghi ở mục "Ngoài phạm vi". Ghi ngày chốt và phiên bản.
5. **Cổng sang planner:** mọi tiêu chí đã chốt, trực tiếp hoặc bằng ủy quyền. Còn mục mở thì không gọi planner.

Muốn đổi kiến trúc sau khi đóng băng: mở lại stage architect cho đúng điểm đó, cập nhật bản kiến trúc, tăng phiên bản. Planner làm tiếp theo bản mới.

## Chuỗi thực thi sau architect (user chốt 11/10/2026)

Vai trò: **research** tìm phương pháp và nghiên cứu ý tưởng. **Architect** tổng hợp research thành thiết kế hệ thống ở mức vận hành cao nhất. **Planner** "compile" hệ thống lý tưởng đó thành chi tiết vận hành bên trong, rồi vạch lộ trình phát triển. Thứ tự gồm 5 stage, mỗi stage xong mới sang stage sau.

Cấp bậc công việc: **Phase ⊃ Module (khối) ⊃ Task** (một phiếu phân công).

### Planner không hỏi lại thiết kế (user chốt 12/10/2026)

Planner chỉ dựa vào bản kiến trúc đã đóng băng để chia việc. Lý do user nêu: planner hỏi nhiều làm lệch hướng architect, rất nguy hiểm.

- **Input:** bản kiến trúc đã chốt và code hiện có. Research không phải input của planner; không đọc lại research để chọn lại phương án.
- **Cấm:** hỏi user về thiết kế; đề xuất kiến trúc, khối, luồng hay tính năng khác; sửa bản kiến trúc; mở lại quyết định đã chốt.
- **Tự quyết, không hỏi:** chi tiết thực thi trong khung kiến trúc (file, tên, cây thư mục, thứ tự task, xử lý lỗi và ca biên bên trong một khối, cách kiểm). Chỗ nhỏ chưa rõ thì chọn cách đơn giản nhất khớp kiến trúc và ghi vào mục "Giả định của planner" trong doc lộ trình.
- **Chỉ được hỏi khi logic quan trọng**, tức một trong hai trường hợp:
  1. bản kiến trúc thiếu hoặc tự mâu thuẫn ở điểm làm đổi hành vi giữa các khối, phân quyền và bảo mật, hoặc tính đúng của dữ liệu, và không suy ra được từ chính bản kiến trúc;
  2. code hoặc nền tảng thật cho thấy kiến trúc không chạy được như đã viết.

  Khi đó planner dừng phần bị ảnh hưởng (phần khác vẫn làm), gom thành **một** lượt hỏi; mỗi câu chỉ đúng chỗ trong bản kiến trúc, nói vì sao chặn và nêu các hướng. Planner không tự chọn. User trả lời thì operator cập nhật bản kiến trúc (stage architect cho đúng điểm đó), rồi planner đi tiếp.
- **Operator lọc câu hỏi của planner:** câu không thuộc hai trường hợp trên thì trả về cho planner tự quyết theo khung, không chuyển cho user.
- Cổng duyệt lộ trình ở stage 1 vẫn giữ. Đó là user duyệt, không phải planner hỏi. User bác vì kiến trúc thì quay lại stage architect; bác vì cách chia việc thì planner sửa lộ trình.

### Stage 1 · Planner giải phẫu architect và lập lộ trình

1. **Tách khối:** đọc bản kiến trúc đã chốt, xác định khối nào tách ra và phát triển độc lập được trước, khối nào phụ thuộc khối nào.
2. **Lộ trình tổng quát:** sắp các khối thành phase theo thứ tự phụ thuộc.
3. **Đi sâu từng khối:** quy logic và luồng đã chốt của khối ra cách thực thi; codespace (thư mục, file); các file liên kết với nhau thế nào. Không chọn lại luồng: luồng là quyết định của stage architect.
4. **Doc lộ trình:** dùng lại skill/chuẩn viết doc (FexMix_Munual, chú thích thuật ngữ) để mô tả lộ trình chia nhỏ theo phase.
5. **Cổng:** user đọc và **duyệt lộ trình**. Chưa duyệt thì không sang stage 2.

### Stage 2 · Phân công

Planner viết phiếu phân công cho từng task. Mỗi phiếu gồm: đặc tả ban đầu; input và bối cảnh cần đọc; file được tạo hoặc sửa; hợp đồng dữ liệu; output kỳ vọng; ca kiểm và cách kiểm; phạm vi cấm động vào. Phiếu phải đủ chặt để coder không phải tự quyết thiết kế (xem *Ranh giới planner / coder*).

### Stage 3 · Coder thực thi

Gọi coder (Sonnet medium) làm **siêu sát** theo phiếu. Coder chỉ viết, không sáng tạo, không tạo kiến trúc phụ. Phiếu thiếu, mơ hồ hoặc sai thì coder dừng và báo, không tự lấp.

### Stage 4 · Review nhỏ (mỗi task)

Review nhẹ: **chỉ so logic code với phiếu phân công**. Kiểm cả việc coder có làm thừa ngoài phiếu không. Không đánh giá kiến trúc ở bước này.

### Stage 5 · Review lớn (mỗi module, một lần)

Chỉ chạy **khi đã code xong một module** (mọi task của module qua review nhỏ). So sánh lần lượt với:
1. đặc tả;
2. phân công;
3. mục tiêu của phase;
4. hệ thống lớn (thiết kế architect).

**Không tự ý review đi review lại**: mỗi module chỉ một lần review lớn, trừ khi user yêu cầu chạy lại.

### Điểm agent đề xuất, user chưa chốt riêng

Các điểm dưới đây áp dụng tạm. User có thể đổi bất kỳ lúc nào.
- Phiếu phân công do planner tự phát sau khi lộ trình đã duyệt, không cần user duyệt từng phiếu.
- Review nhỏ báo sai: code lệch phiếu thì trả coder sửa, tối đa 2 vòng, quá 2 vòng thì báo user. Phiếu sai hoặc thiếu thì trả về planner. Reviewer nhỏ là agent độc lập với coder, Sonnet medium.
- Review lớn do agent Opus high làm. Kết quả chỉ ghi lại phát hiện (vấn đề, bằng chứng, hướng xử lý) rồi đưa user quyết, không tự kích hoạt sửa (theo *Kỷ luật thực thi*).
- Doc lộ trình là HTML trong `<repo>/documentation/plan/`. Phiếu phân công là file `.md` cho coder đọc.
- Bộ tiêu chí kiến trúc và mẫu câu hỏi trong [skills/architect-stage.md](skills/architect-stage.md), cùng cách hiểu "logic quan trọng" ở mục *Planner không hỏi lại thiết kế*, do agent cụ thể hoá từ yêu cầu của user ngày 12/10/2026.

## Kỷ luật thực thi (user chốt 10/10/2026, tuyệt đối)

- **Cấm overengineer ở những bước đầu của hệ thống.** Chỉ làm đúng và đủ những gì thiết kế đã chốt. Không thêm lớp trừu tượng, tính năng, cấu hình hay "chuẩn bị cho sau này" khi thiết kế không yêu cầu.
- **Bám sát logic đã bàn trong thiết kế.** Nếu code thật cho thấy logic thiết kế không chạy được hoặc có cách tốt hơn, dừng lại và báo user. Không tự chọn cách khác.
- **Mọi vấn đề phát sinh về sau phải được bàn và ghi chú, không tự sửa code.** Lỗi, rủi ro, ý tưởng cải tiến hay chỗ lệch thiết kế mà agent phát hiện: ghi lại (vấn đề, bằng chứng, các hướng xử lý) trong tài liệu của khối đó hoặc báo trong chat, rồi chờ user bàn và quyết. Chỉ sửa code khi user đã đồng ý cách sửa. Việc này áp dụng cho cả việc nằm ngoài bước đang được giao.

## Điều cấm

- Overengineer; làm lệch logic thiết kế đã chốt; tự sửa code khi vấn đề chưa được bàn với user.
- Nhảy sang code, plan hay chọn công nghệ khi tầng 1 chưa chốt.
- Phân công hoặc gọi coder khi user chưa duyệt lộ trình; review lớn trước khi module xong, hoặc review lớn lặp lại khi user không yêu cầu.
- Nhồi chi tiết kỹ thuật vào tài liệu tầng 1.
- Gọi planner khi bản kiến trúc còn mục mở; agent tự lấp chỗ trống trong kiến trúc khi user chưa ủy quyền "tự quyết".
- Planner hỏi user về thiết kế, sửa bản kiến trúc hay mở lại quyết định đã chốt, trừ khi gặp logic quan trọng (xem *Planner không hỏi lại thiết kế*).
- Coi kết luận của nhiều agent là quyết định của user. Ngoại lệ duy nhất: user ủy quyền "tự quyết" ở stage architect.
- Để architect hoặc researcher đọc vault, hoặc đưa nội dung vault vào prompt của chúng.

## Quan hệ với quy trình riêng của dự án

Quy trình 06/10/2026 trong `project1_app/AGENTS.md` cho hai architect đề xuất trước rồi mới đưa HTML cho user. Ở **giai đoạn thiết kế**, hiến pháp này thay thế phần đó, vì đây là quyết định mới hơn của user: bàn qua chat với user trước, research sau khi đã chốt sơ đồ khối. Ở **giai đoạn triển khai**, quy trình planner/coder/reviewer của dự án vẫn dùng được, nhưng phải chứa đủ 5 stage của *Chuỗi thực thi sau architect* (11/10/2026): cổng duyệt lộ trình, phiếu phân công, review nhỏ theo task, review lớn một lần mỗi module.

## Ánh xạ vào thư mục docs (khung học từ `project1_app/docs`, 10/10/2026)

User nói bộ docs này chưa hoàn thiện; chỉ lấy cấu trúc sơ cấp của nó. Phần ánh xạ dưới đây là **gợi ý của agent, user chưa chốt**:

| Thư mục | Vai trò hiện tại | Dùng trong hiến pháp |
|---|---|---|
| `index.html` | Mục lục tra cứu, có tìm kiếm | Thêm thẻ cho mỗi tài liệu mới |
| `manual/` | Một manual tổng quan: bức tranh toàn cục, luồng chính, thành phần, API, vận hành | Đích cuối của tầng 1: khi đã chốt, phần sơ đồ khối và luồng chính nhập vào đây |
| `design/` | So sánh phương án, quyết định, nghiên cứu; bản HTML kèm bản `.md` | HTML architect tầng 1 trong các vòng lặp; kết quả research, case study và final decision từng vòng |
| `flows/` | Mỗi file giải thích sâu một luồng | Tài liệu tầng 2 của từng khối hoặc từng luồng |
| `assets/` | CSS, font và template dùng chung, chạy offline | HTML mới chép từ `assets/template.html` |

Quy ước tên file theo `docs/README.md`: tiếng Anh, chữ thường, nối bằng `-`.
