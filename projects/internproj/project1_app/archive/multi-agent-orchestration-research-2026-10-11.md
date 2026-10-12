---
id: multi-agent-orchestration-research-20261011
type: research-archive
scope: global
status: source-observed
date: 2026-10-11
source: sessions/2026-10-11-multi-agent-orchestration-research.md
---

# Research: điều phối nhiều agent LLM (AWS, LangGraph, AutoGen, blackboard, debate, các hệ lớn)

Đây là kết quả cuối của 4 agent research (Sonnet, effort medium), do operator gộp lại. File này là **kho định hướng**, không phải quyết định của user. Chưa viết HTML. Chủ đề không gắn với code repo, nhưng mục đích dùng còn chờ user chốt: (a) đối chiếu với luồng agent làm việc của user, hoặc (b) tham khảo chung. Tạm để trong archive của project1_app theo quy ước internproj.

Mức kiểm nguồn:
- **mở**: agent đã fetch trang gốc;
- **tìm**: chỉ thấy qua snippet hoặc nguồn thứ cấp;
- **nhớ**: kiến thức nền của agent, chưa tra lại.

## Kết luận chung (4 phần đồng thuận)

| Kết luận | Bằng chứng |
|---|---|
| Bắt đầu bằng một agent hoặc luồng cố định. Chỉ tách agent khi đo được lợi ích. | Anthropic "Building effective agents", OpenAI guide, LangChain, Microsoft Agent Framework ("viết được hàm thì đừng dùng agent"), AWS, Cognition |
| Đọc thì song song, ghi thì một luồng. Nghiên cứu, review, phản biện chạy song song. Ghi file, code, quyết định do một agent làm. | Anthropic research system (nhiều agent đọc, 1 agent viết). Cognition: quyết định ngầm xung đột. Cả hai nói code kém hợp multi-agent. |
| Subagent chủ yếu để cô lập context. Trả về bản cô đọng hoặc tham chiếu file, không chia sẻ toàn bộ lịch sử. | Claude Code subagents. Bedrock bật/tắt chia sẻ lịch sử và truyền dữ liệu lớn bằng tham chiếu. Anthropic ghi kết quả ra filesystem. |
| Thứ tự đã biết thì dùng luồng cố định (đồ thị). Chỉ để LLM chọn ở điểm rẽ nhánh thật. | MAF thêm Workflows, CrewAI có Flows, Strands có Graph, ADK có Sequential/Parallel/Loop. Đây là xu hướng hội tụ. |
| Debate không phải mặc định để tăng độ đúng. Nó không thắng ổn định self-consistency/vote, tốn hơn, và có sycophancy (đúng đổi thành sai). | Smit et al. ICML 2024, 2502.08788, 2509.05396 |
| Debate có ích khi đối kháng và lệch thông tin (debater biết điều judge không biết). Dùng cho giám sát và phơi lập luận. | Khan et al. 2402.06782: judge yếu đạt 76% so với 48% |
| Phản biện phải độc lập: khác model người sinh, có rubric, có test. | Ye et al. ICLR 2025: 12 bias của LLM-as-judge |
| Lỗi chủ yếu nằm ở thiết kế và kiểm chứng, không phải ở model. | MAST (Cemri 2025): 14 kiểu lỗi, tỉ lệ lỗi 41–87% trên 7 framework |
| Multi-agent tốn khoảng 15 lần token chat. Chỉ đáng khi việc rộng và có giá trị cao. | Anthropic (eval nội bộ) |

## Luồng tối thiểu đề xuất (agent suy ra, chưa chốt)

1. Một agent điều phối giữ ngữ cảnh và quyền quyết định. Không có kênh agent-agent ngang hàng.
2. Prompt giao việc cho subagent ghi rõ mục tiêu, phạm vi, định dạng đầu ra và ranh giới.
3. Nghiên cứu, review và phản biện là subagent chỉ đọc, chạy song song, trả về bản tóm tắt.
4. Có một vùng trạng thái chung có cấu trúc (file plan và tiến độ). Đây là dạng blackboard nhẹ, giống ledger của Magentic-One.
5. Ghi và code chỉ một luồng. Plan phải được người duyệt trước.
6. Một vòng critic độc lập (khác model nếu có), có rubric và test, giới hạn số vòng.
7. Quyết định cuối do người hoặc test, không do đa số agent.
8. Mọi vòng lặp có giới hạn số bước, số agent và token.

Để sau: swarm/handoff ngang hàng, A2A, hierarchy nhiều tầng, debate nhiều vòng, framework có checkpoint và time-travel.

## Phần 1 · AWS

- **Agent Squad** (trước là Multi-Agent Orchestrator):
  - Classifier chọn 1 agent theo mô tả và lịch sử. Câu ngắn ("ok") quay về agent đang chờ. JevClassifier không dùng LLM, có nhánh `unknown`.
  - SupervisorAgent theo kiểu agent-as-tools, chạy song song, có thể lồng thành "team of teams".
  - Agent báo repo đã chuyển từ `awslabs` sang `2fastlabs/agent-squad`. **Chưa xác minh.** [mở] https://github.com/awslabs/agent-squad
- **Bedrock multi-agent collaboration**:
  - Có hai chế độ: Supervisor, và Supervisor with routing (việc đơn giản đi thẳng tới agent, việc mơ hồ mới lên supervisor).
  - Soft limit 3 tầng. Bật/tắt chia sẻ lịch sử. Payload referencing. GA ngày 10/3/2025.
  - Bedrock Agents "Classic" đã ngừng nhận khách mới, AWS chuyển sang AgentCore.
  - Nguồn [mở]: https://docs.aws.amazon.com/bedrock/latest/userguide/agents-multi-agent-collaboration.html · https://aws.amazon.com/blogs/aws/introducing-multi-agent-collaboration-capability-for-amazon-bedrock/ · https://aws.amazon.com/blogs/machine-learning/unlocking-complex-problem-solving-with-multi-agent-collaboration-on-amazon-bedrock
- **Strands Agents** [tìm; trang patterns trả 404]:
  - Bốn pattern: agents-as-tools, swarm (handoff tự động, dễ lặp vô hạn), graph, workflow (DAG).
  - v1.0 ra 7/2025, có A2A. https://aws.amazon.com/blogs/opensource/introducing-strands-agents-an-open-source-ai-agents-sdk/
- **Benchmark AWS (vendor)**: multi-agent đạt khoảng 90% so với single 53–80% trên 3 miền.
- **Khách hàng được nêu tên, không có số liệu**: Moody's, PwC, Robin AI [tìm]. Nguồn: https://www.businesswire.com/news/home/20241203470182/en · https://venturebeat.com/ai/aws-brings-multi-agent-orchestration-to-bedrock/ . Northwestern Mutual [tìm]: https://www.infoq.com/news/2025/01/aws-bedrock-multi-agent-ai
- **Khuyến nghị AWS**: router hoặc agent đơn trước; supervisor 1 tầng khi cần gom kết quả; graph khi quy trình cố định; chưa dùng swarm.

## Phần 2 · LangGraph, AutoGen, CrewAI

- **LangGraph** [nhớ]:
  - State graph: node, edge có điều kiện, state schema dùng chung có reducer, checkpoint theo thread, interrupt cho human-in-the-loop, store dài hạn.
  - Các kiểu multi-agent: network, supervisor, hierarchical, supervisor-as-tools, swarm/handoff.
- **LangChain blog** [mở]:
  - Bài: https://www.langchain.com/blog/choosing-the-right-multi-agent-architecture · https://www.langchain.com/blog/how-and-when-to-build-multi-agent-systems
  - Bốn pattern: Subagents, Skills, Handoffs, Router.
  - Bắt đầu bằng một agent. Tách khi vướng giới hạn context hoặc khi nhiều team cùng làm.
  - Truy vấn đa miền: Subagents và Router khoảng 9K token, Skills khoảng 15K, Handoffs từ 7 lần gọi trở lên.
  - Bài có số tự mâu thuẫn: "67%" so với khoảng 40%.
  - Bài không khuyến khích multi-agent cho code.
- **AutoGen** [mở]:
  - README ghi đã vào maintenance mode, người mới nên dùng **Microsoft Agent Framework** (kế thừa AutoGen và Semantic Kernel; .NET, Python, Go preview; gồm Agents, Harness Agent, Workflows đồ thị, middleware). https://github.com/microsoft/autogen · https://learn.microsoft.com/en-us/agent-framework/overview/
  - GA khoảng 4/2026 [tìm].
  - v0.2: GroupChat với speaker selection. v0.4: actor model, SelectorGroupChat, Swarm, GraphFlow [nhớ].
- **Magentic-One**:
  - Orchestrator giữ Task Ledger (sự kiện, phỏng đoán, kế hoạch) và Progress Ledger. Khi kẹt thì sửa ledger và lập lại kế hoạch.
  - GAIA 38%, WebArena 32,8% [tìm]. https://arxiv.org/abs/2411.04468
- **CrewAI** [nhớ]:
  - Agent có role, goal, backstory. Crew chạy sequential hoặc hierarchical (manager LLM).
  - Flows là lớp xác định bên ngoài. Không có case study độc lập.
- **Case study LangGraph** (nguồn LangChain, vendor, chưa kiểm độc lập):
  - LinkedIn (recruiter hierarchical, SQL Bot), Uber (sinh unit test cho migration), Replit, Klarna (85 triệu người dùng, -80% thời gian), Elastic.
  - Nguồn: https://langchain.com/built-with-langgraph · https://www.langchain.com/blog/top-5-langgraph-agents-in-production-2024 · https://blog.langchain.com/is-langgraph-used-in-production

## Phần 3 · Blackboard, debate, phản biện

- **Blackboard** [nhớ]:
  - Gốc ở Hearsay-II, BB1, Nii 1986. Có ba thành phần: board, knowledge sources, control. Các agent không gọi nhau trực tiếp.
  - Khác pub-sub ở chỗ giữ trạng thái có cấu trúc bền.
- **Bản LLM**:
  - Salemi et al. 2025: agent phụ tự quyết có đáp hay không. Cải thiện 13–57% end-to-end trên KramaBench, DS-Bench, DA-Code [tìm, từ abstract]. https://arxiv.org/abs/2510.01285
  - Terrarium là testbed an ninh cho blackboard (agent độc hại, đầu độc board), không phải kết quả hiệu năng. https://arxiv.org/abs/2510.14312
- **Bằng chứng về debate**:

| Paper | Kết quả | Mức kiểm |
|---|---|---|
| Du et al. 2023 https://arxiv.org/abs/2305.14325 | Dương: arithmetic 81,8 so với 67,0; GSM8K 85 so với 77 | tìm (thứ cấp) |
| Smit et al. ICML 2024 https://arxiv.org/abs/2311.17371 | Âm: không vượt ổn định self-consistency; nhạy với "mức đồng ý" | tìm |
| https://arxiv.org/abs/2502.08788 | Âm: MAD không vượt CoT/SC kể cả khi thêm compute | tìm |
| Talk Isn't Always Cheap https://arxiv.org/abs/2509.05396 | Âm: accuracy giảm qua vòng, sycophancy | tìm |
| Khan et al. 2024 https://arxiv.org/abs/2402.06782 | Dương (đối kháng, lệch thông tin): 76% so với 48% | tìm |
| Ye et al. ICLR 2025 https://arxiv.org/abs/2410.02736 | 12 bias của LLM-as-judge | tìm |
| MoA https://arxiv.org/abs/2406.04692 | AlpacaEval 65,1% so với 57,5% | tìm (thứ cấp) |

- **Thiết kế vòng debate** (agent suy ra): 2–3 agent, 1–2 vòng. Trộn model. Có một vai cố định phản biện. Dừng khi đáp án không đổi hoặc hết ngân sách. Quyết định cuối: test hoặc tool → judge khác model → người.
- **Thiếu**: case study hệ thật (review code, fact-check nhiều agent) có số liệu.

## Phần 4 · Hệ lớn và phần xuyên suốt

- **Anthropic research system** [mở] https://www.anthropic.com/engineering/multi-agent-research-system
  - Lead (Opus 4) điều phối subagent (Sonnet 4) chạy song song, lưu plan ra bộ nhớ ngoài, có CitationAgent.
  - Kết quả: hơn single 90,2%, khoảng 15 lần token. Token dùng giải thích 80% phương sai trên BrowseComp.
  - Quy tắc scale: 1 agent cho việc đơn giản, 2–4 cho so sánh, hơn 10 cho việc phức tạp. Lead chạy đồng bộ.
  - Lỗi đã gặp: từng sinh 50 subagent cho câu hỏi đơn giản; ưa trang SEO rác. Eval khoảng 20 query là đủ bắt đầu.
- **Building effective agents** [mở] https://www.anthropic.com/research/building-effective-agents
  - Phân biệt workflow với agent.
  - Năm pattern: chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer.
- **Claude Code subagents** [mở] https://code.claude.com/docs/en/sub-agents
  - Context riêng, không thấy lịch sử cha. Giới hạn tool bằng allowlist.
  - Lồng tối đa 3 tầng, tối đa 20 subagent đồng thời (mặc định). Resume được, có fork.
- **OpenAI Agents SDK** [mở] https://openai.github.io/openai-agents-python/handoffs/
  - Handoff chuyển cả lịch sử, as_tool chỉ chuyển input.
  - Input guardrail chỉ áp cho agent đầu, output guardrail chỉ cho agent cuối, nên handoff giữa chừng không được bảo vệ.
  - Guide phân biệt manager và decentralized [tìm]: https://www.maginative.com/article/how-to-build-ai-agents-a-detailed-practical-guide-from-openai/
- **Google ADK**: Sequential/Parallel/Loop cộng LLM delegation. Docs đã dời tới https://adk.dev/workflows/ , chi tiết [nhớ].
- **A2A và MCP**: MCP nối agent với tool, A2A nối agent với agent. Hai giao thức bổ sung nhau. Mức trưởng thành của A2A: nguồn mâu thuẫn [tìm]. https://redis.io/blog/mcp-vs-a2a-which-protocol-do-you-need
- **MetaGPT** https://arxiv.org/abs/2308.00352
  - SOP theo vai, message pool pub-sub.
  - HumanEval 85,9%. Executability 3,75 so với ChatDev 2,25. Sửa tay 0,83 so với 2,5. Chi phí $0,92–1,39.
- **ChatDev**: chat chain theo pha. https://arxiv.org/abs/2307.07924
- **Agent code đơn agent**:
  - OpenHands CodeAct 2.1 đạt 53% SWE-bench Verified [mở]. https://www.openhands.dev/blog/openhands-codeact-21-an-open-state-of-the-art-software-development-agent
  - Devin: merge PR 67%, song song ở mức nhiều bản độc lập. https://cognition.com/blog/devin-annual-performance-review-2025
- **Cognition "Don't Build Multi-Agents"** (6/2025) [mở] https://cognition.com/blog/dont-build-multi-agents
  - Chia sẻ toàn trace, chạy single-thread.
  - Ví dụ Flappy Bird: các agent lệch phong cách nhau.
  - Nhận định của operator: ít mâu thuẫn với Anthropic (đọc thì song song, ghi thì một luồng).
- **MAST** https://arxiv.org/abs/2503.13657
  - Dữ liệu: hơn 1.600 trace, kappa 0,88.
  - Thiết kế hệ thống chiếm khoảng 44%: lặp bước 15,7%, không biết điều kiện dừng 12,4%, sai đặc tả 11,8%.
  - Lệch pha giữa agent khoảng 32%.
  - Kiểm chứng khoảng 24%: kiểm sai 9,1%, không kiểm 8,2%.
  - Can thiệp trên ChatDev: thêm bước kiểm mục tiêu cao cho +15,6%.

## Chưa xác minh hoặc có thể lỗi thời
- Repo Agent Squad đổi chủ; AutoGen ở maintenance mode và ngày GA của MAF; Bedrock Classic ngừng nhận khách mới. Operator chưa tự kiểm.
- Số liệu Anthropic, AWS và LangChain là vendor tự báo, eval nội bộ, model cũ.
- Mô tả LangGraph, CrewAI, AutoGen v0.4 và ADK dựa nhiều vào [nhớ]. API đổi nhanh.
