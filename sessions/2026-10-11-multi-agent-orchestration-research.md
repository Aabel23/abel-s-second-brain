---
id: session-20261011-multi-agent-orchestration-research
type: session-learning
scope: global
status: extracted
date: 2026-10-11
source: current-conversation
---

# Mục tiêu và chuẩn đạt
User yêu cầu: "thực hiện research về aws multiagent orchestrator, langchain, autogen, blackboard pattern, debate/adversarial pattern, các hệ thống nổi tiếng và lớn khác". User không nêu hệ thống cụ thể nào để áp dụng.

## Những lần chỉnh hướng
- Không có. User không phản hồi giữa phiên, chỉ yêu cầu chốt phiên.

## Quyết định và giới hạn
- [agent-run] Chạy skill research-stage. Chia 4 researcher (Sonnet, effort medium, chạy nền), không có nội dung vault trong prompt:
  - AWS (Agent Squad, Bedrock, Strands);
  - LangGraph, AutoGen, CrewAI;
  - blackboard, debate, phản biện;
  - các hệ lớn (Anthropic, OpenAI, Google, MetaGPT, Devin) cùng MAST và Cognition.
- [chưa chốt] User chưa trả lời câu hỏi nào. Đây không phải quyết định của user.

## Sản phẩm hiện hành
- Archive research và nguồn: [archive/multi-agent-orchestration-research-2026-10-11.md](../projects/internproj/project1_app/archive/multi-agent-orchestration-research-2026-10-11.md). Gồm kết luận đồng thuận, luồng tối thiểu 8 bước (agent đề xuất), bảng nguồn có mức kiểm.
- Bản tổng hợp chỉ có trong chat. **Chưa viết HTML research.**

## Phần còn mở
Các câu hỏi chờ user:
1. Mục đích của research:
   - (a) đối chiếu với luồng agent làm việc của user (hiến pháp: operator, researcher, planner, coder, reviewer, debate). Agent khuyên chọn (a).
   - (b) tài liệu tham khảo chung.
2. Bước "debate" trong hiến pháp nên là gì. Research cho thấy debate nhiều vòng giữa các agent cùng model ít lợi. Agent khuyên: một critic độc lập chạy 1 vòng, sau đó người quyết.
3. Có cần framework không. Agent khuyên chưa cần; dùng subagent Claude Code cộng file trạng thái chung là đủ.
4. Nơi đặt HTML: `project1_app/documentation/research/` hay `docs/` ở gốc internproj.

Việc kiểm còn thiếu: chưa tự xác minh các mục sau:
- Agent Squad đổi chủ repo;
- AutoGen ở maintenance mode;
- Bedrock Classic ngừng nhận khách mới.
