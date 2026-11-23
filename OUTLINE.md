# 主流 Agent 技术体系调研文档 · 大纲（v0.1，待确认）

> 状态：已确认（标准档 B，2026-11-20）。此文件为文档集的"目录契约"，实际内容已按卷分文件落盘。
> 时间基线：2026-11-20（Asia/Shanghai）。生态变化快，所有版本号 / 规范状态均标注"事实核对日期"。

## 0. 目标与边界

**要解决的问题**：把 2026 年当前主流的 Agent 技术体系，拆成一套可独立阅读、可交叉引用、可持续更新的结构化文档；支撑技术选型、架构评审、新人 onboarding。

**边界**：
- 覆盖：概念与范式、主流框架、互操作协议、工程化（可观测/评测/安全/可靠性/部署）、代表性产品形态、开放问题。
- 不覆盖：模型训练与微调细节、具体业务场景的 prompt 合集、厂商商业分析（仅在必要处作为背景）。
- 不做"全家桶单文件"，一篇一主题，单篇 150–300 行。

## 1. 组织原则

1. **分层组织**：模型层 → Agent 运行时 → 编排框架 → 互操作协议 → 工程化 → 产品形态 → 展望。读者可自下而上，也可按需跳读。
2. **一篇一主题**：每个框架 / 协议 / 工程问题独立成篇，文件内自包含，文件间用相对链接交叉引用。
3. **事实与判断分离**：事实性描述附来源链接（带发布/更新日期）；"选型建议""我的判断"单独成节并标注。
4. **统一对比维度**：框架篇统一 10 个维度（见 §6），协议篇统一"解决哪一层 + 传输 + 治理方 + 成熟度"。
5. **中文为主，术语保留英文**：首次出现给英文原名与缩写，后续直接用英文术语（agent loop、harness、handoff…）。
6. **图表内联**：架构图/时序图用 Mermaid，不引入二进制图片，保证可 diff、可 review。

## 2. 目录结构

```
agent-research/
├── README.md                          # 总索引：全景图、阅读路径、文档约定、更新日志
├── OUTLINE.md                         # 本大纲（确认后锁定为目录契约）
├── references.md                      # 全量来源汇总（按卷分组，含访问日期）
│
├── 00-overview/                       # 卷 0：总览（先读）
│   ├── 01-landscape.md                # 2026 Agent 技术全景与分层模型
│   ├── 02-evolution.md                # 三代演进脉络与关键时间线
│   └── 03-glossary.md                 # 术语表（中英对照，40–60 条）
│
├── 01-foundations/                    # 卷 1：基础理论与范式
│   ├── 01-agent-loop.md               # Agent 的本质：感知-思考-行动循环
│   ├── 02-context-engineering.md      # 上下文工程
│   ├── 03-memory.md                   # 记忆体系
│   ├── 04-tool-use-and-sandbox.md     # 工具调用与沙箱执行
│   ├── 05-multi-agent.md              # 多智能体协作
│   └── 06-agent-architectures.md      # 六种架构方案与 Super-Agent
│
├── 02-frameworks/                     # 卷 2：主流框架（一篇一框架）
│   ├── 00-selection-matrix.md         # 选型矩阵与决策树
│   ├── 01-langgraph.md
│   ├── 02-microsoft-agent-framework.md
│   ├── 03-openai-agents-sdk.md
│   ├── 04-google-adk.md
│   ├── 05-claude-agent-sdk.md
│   ├── 06-crewai.md
│   ├── 07-llamaindex.md
│   ├── 08-pydantic-ai.md
│   ├── 09-strands-agents.md
│   ├── 10-agno-and-lightweight.md     # Agno / smolagents / CAMEL
│   └── 11-typescript-stack.md         # Mastra / Vercel AI SDK / Agents SDK (JS)
│
├── 03-protocols/                      # 卷 3：互操作协议与标准
│   ├── 00-protocol-stack.md           # 协议分层全景 + 邻近规范
│   ├── 01-mcp.md                      # Model Context Protocol
│   ├── 02-a2a.md                      # Agent2Agent
│   ├── 03-acp.md                      # Agent Client Protocol
│   └── 04-agents-md.md                # AGENTS.md 与上下文约定
│
├── 04-engineering/                    # 卷 4：工程化
│   ├── 01-observability.md            # 可观测性与 trace
│   ├── 02-evaluation.md               # 评测体系与 benchmark
│   ├── 03-security.md                 # 安全与权限
│   ├── 04-reliability-and-hitl.md     # 可靠性、失败模式与 HITL
│   ├── 05-cost-latency.md             # 成本与延迟优化
│   └── 06-deployment.md               # 部署形态与托管 runtime
│
├── 05-products/                       # 卷 5：产品形态与落地
│   ├── 01-coding-agents.md            # 编码 Agent
│   ├── 02-computer-use.md             # 浏览器 / GUI Agent
│   ├── 03-deep-research.md            # 深度研究 Agent
│   ├── 04-enterprise-platforms.md     # 企业平台与低代码
│   └── 05-vendor-agent-architectures.md  # 大厂参考架构
│
├── 06-outlook/                        # 卷 6：展望
│   └── 01-open-problems.md            # 开放问题、风险与趋势
│
└── templates/                         # 写作模板
    ├── TOPIC-TEMPLATE.md
    └── FRAMEWORK-TEMPLATE.md
```

## 3. 分卷大纲

### 卷 0 · 总览（先读，建立坐标系）

**00/01-landscape.md｜2026 Agent 技术全景**
- 七层模型：模型层 → 运行时（harness/sandbox）→ 编排框架 → 互操作协议 → 工具与数据 → 可观测/评测 → 应用形态。
- 每层的职责、层间边界、典型玩家；一张 Mermaid 分层图。
- 2026 年的关键变化（4–6 条）：MCP 走向无状态核心 + 扩展化（2026-07-28 规范）；A2A 发布 1.0 并被 Linux 基金会托管；AGENTS.md 收敛为跨工具事实标准；AutoGen + Semantic Kernel 合并为 Microsoft Agent Framework；"Agent SDK + 托管 runtime"成为主流交付形态；computer use / 长时任务的工程化。
- 常见误区分层：框架 ≠ 协议 ≠ runtime。

**00/02-evolution.md｜演进脉络**
- 三代划分：Prompt 工程（2022–2023）→ 工具调用与单 Agent 循环（2023–2024）→ 协议化与多 Agent 编排（2024–2025）→ 产品化与工程化（2025–2026）。
- 时间线（表格）：CoT/ReAct、function calling、MCP（2024-11）、A2A（2025-04）、A2A 1.0（2026-03）、MCP 2026-07-28、AAIF 成立、AGENTS.md 采用率。
- 为什么"框架潮"会收敛到"协议 + runtime"：从 demo 到生产的约束转移。
- 被高估与被低估的方向（判断，单独标注）。

**00/03-glossary.md｜术语表**
- 40–60 条中英对照：agent、agentic、harness、scaffold、agent loop、ReAct、tool call、trajectory、context rot、compaction、memory、handoff、supervisor、subagent、guardrail、HITL、durable execution、sandbox、Agent Card、MCP/A2A/ACP、elicitation、sampling、extension、eval、LLM-as-judge 等。
- 易混词辨析：Agent vs Workflow、Framework vs SDK vs Runtime、Protocol vs API、Orchestration vs Choreography。

### 卷 1 · 基础理论与范式（与框架无关的"不变式"）

**01/01-agent-loop.md｜Agent 的本质**
- 形式化：Observation → Thought → Action → Result 循环；与 workflow/DAG 的区别（谁决定下一步）。
- 四种主流控制结构：ReAct、Plan-and-Execute、Reflection/Critic、Tree-of-Thought / 搜索式。
- 停止条件、预算控制、错误恢复。
- 何时不该用 agent（确定性任务用 workflow）。
- 单篇含一张控制流对比图。

**01/02-context-engineering.md｜上下文工程**
- 上下文即工作内存：system prompt / tools / history / retrieved / scratchpad 的分区与预算。
- context rot、lost-in-the-middle、长上下文的真实收益边界。
- 压缩与卸载：summarization、compaction、structured note-taking、sub-agent 隔离、文件系统即外部记忆。
- KV cache 友好性、prompt 稳定性对成本/延迟的影响。
- 实践清单：上下文预算表模板。

**01/03-memory.md｜记忆体系**
- 分类：工作记忆 / 情节记忆 / 语义记忆 / 程序性记忆。
- 实现谱系：全文+向量检索、知识图谱、文件式记忆（memory.md / notepad）、参数化记忆（微调）、sleep-time compute / 后台整理。
- 写入策略（何时写、写什么、去重与冲突）、遗忘与过期、隐私。
- 与 RAG 的关系与差异。

**01/04-tool-use-and-sandbox.md｜工具调用与沙箱**
- 工具 schema 设计：命名、描述、粒度、参数校验、错误信息可读性。
- 调用模式：并行、串行依赖、流式、长时任务、human approval。
- 执行环境：本地进程 / 容器 / microVM（Firecracker、gVisor）/ 云沙箱；文件系统、网络出口、资源限额。
- 代码解释器与 computer use 的工具面差异。
- 工具数量爆炸的治理（工具检索、分层命名空间、MCP 网关）。

**01/05-multi-agent.md｜多智能体**
- 拓扑：supervisor/orchestrator、handoff、group chat、debate、pipeline、blackboard。
- 上下文隔离是收益来源，也是调试成本来源。
- 通信：消息、共享状态、文件、协议（A2A）。
- 失败模式：错误放大、无限循环、责任稀释、成本爆炸。
- 判断：什么时候多 agent 真的比单 agent + 好工具更强。

**01/06-agent-architectures.md｜Agent 架构方案**
- 五面解剖：控制 / 执行 / 记忆 / 治理 / 观测。
- 六种方案：单体、路由、主管、层级、群体、流水线；三种判据。
- Super-Agent：Router/Worker 双层、事件驱动、上下文隔离、与 supervisor 的区别、失败模式。

### 卷 2 · 主流框架（一篇一框架，统一 10 维对比）

**02/00-selection-matrix.md｜选型矩阵**
- 统一维度：抽象层次（代码优先/图/对话）、执行模型、状态与持久化、多 agent 支持、MCP/工具生态、可观测、评测集成、部署形态、语言、许可证与治理。
- 决策树：从"要不要持久化/要不要人工介入/团队语言/是否上云"四个问题出发。
- 反选建议：什么情况下不要用框架。

**02/01-langgraph.md**
- 定位：低层图编排 + 持久化 checkpoint；LangChain 生态与 LangSmith。
- 核心：StateGraph、node/edge、conditional edge、interrupt、checkpointer、time travel。
- 适合：需要精确控制与可恢复长流程；不适合：想要开箱即用的角色协作。

**02/02-microsoft-agent-framework.md**
- 定位：AutoGen + Semantic Kernel 合并后的统一框架；Python/.NET（Go 预览）。
- 核心：Agent、Workflow、middleware、OTel、DevUI；GroupChat 血缘。
- 版本陷阱：0.2→0.4 重写导致教程失真；迁移路径。

**02/03-openai-agents-sdk.md**
- 定位：轻量 agents/tools/handoffs/guardrails 抽象 + tracing；Responses API 生态。
- 核心：Agent、Runner、handoff、guardrail、session、tracing。
- 与 AgentKit（Agent Builder / ChatKit）的关系；锁定风险。

**02/04-google-adk.md**
- 定位：Gemini 生态的 agent 开发套件；ADK + Vertex Agent Engine + A2A 一等公民。
- 核心：LlmAgent、WorkflowAgent（sequential/parallel/loop）、tools、callbacks、eval。
- 多云可用性、与 MCP/A2A 的衔接。

**02/05-claude-agent-sdk.md**
- 定位：把 Claude Code 的 harness 产品化（agent loop + 内置工具 + 权限 + 子 agent + hooks + skills）。
- 核心：query/stream、tool 权限模型、subagent、hooks、MCP 接入、session 管理。
- 与"自建 loop + 原生 API"的取舍；托管 runtime 的锁定度。

**02/06-crewai.md**
- 定位：role/goal/task 抽象，面向"团队协作"叙事；Crews + Flows。
- 核心：Agent(role, goal, backstory)、Task、Process(sequential/hierarchical)、Flows 事件驱动。
- 何时够用、何时成为天花板。

**02/07-llamaindex.md**
- 定位：RAG/数据连接器起家，AgentWorkflow 作为上层编排。
- 核心：160+ connectors、index/retriever、query engine、AgentWorkflow、eval。
- 真正卖点是数据层而非 agent loop。

**02/08-pydantic-ai.md**
- 定位：类型安全、Python 原生、结构化输出优先； Logfire 可观测。
- 核心：Agent、typed deps、output_type、tools、graph、evals。
- 适合强类型后端团队。

**02/09-strands-agents.md**
- 定位：AWS 的 model-driven 轻量 SDK；Bedrock AgentCore 集成。
- 核心：Agent + tools 装饰器、multi-agent（swarm/graph）、session、A2A/MCP。
- 云绑定与可移植性。

**02/10-agno-and-lightweight.md**
- Agno：高性能轻量 runtime + 内置 memory/knowledge/team。
- smolagents：CodeAgent（代码即动作）范式。
- CAMEL：角色扮演式多 agent 研究框架。
- 共同点：低抽象、快启动、适合原型与评测。

**02/11-typescript-stack.md**
- Mastra：TS 原生 agent/workflow/RAG/evals。
- Vercel AI SDK：流式 UI + tool calling 的 Web 侧标准件。
- OpenAI Agents SDK (JS)：与 Python 版对齐度。
- 前端/全栈团队的选型要点。

### 卷 3 · 互操作协议与标准

**03/00-protocol-stack.md｜协议分层全景**
- 四个层次：工具/上下文（MCP）、Agent 间（A2A）、编辑器-Agent（ACP）、仓库级约定（AGENTS.md）。
- 一张"谁和谁说话"的定位图；与普通 REST/OpenAPI、gRPC 的关系。
- 邻近规范速览（不作为主线，只做定位与取舍）：agents.json、OpenAI Apps SDK、AGNTCY、ANP、llms.txt。
- 判断：协议已收敛到什么程度、还缺什么（身份、计费、审计）。

**03/01-mcp.md**
- 架构：host / client / server；原语：tools、resources、prompts、sampling、elicitation、roots。
- 传输：stdio、Streamable HTTP；2026-07-28 规范要点：无状态核心、Multi Round-Trip Requests、header-based routing、cacheable list results、授权加固、扩展框架（Tasks、Apps 作为版本化扩展）。
- 安全：tool poisoning、prompt injection 经由工具结果、过度授权、供应链。
- 工程实践：server 设计、网关与注册中心、版本兼容、灰度。
- 生态现状与采用度（标注来源与日期）。

**03/02-a2a.md**
- 定位：跨厂商 / 跨组织 agent 协作；与 MCP 的分工（工具 vs 委托）。
- 核心对象：Agent Card（能力声明、JWS 签名）、Task 生命周期（8 种状态）、Artifact、Message。
- 三种 binding：JSON-RPC 2.0、gRPC、HTTP+JSON；流式与推送。
- 企业治理：发现、认证、审计、SLA；1.0 之后的变化与采用情况。

**03/03-acp.md**
- 定位：编辑器/IDE 与 agent 进程解耦（Zed 发起），JSON-RPC over stdio。
- 核心：session、prompt turn、工具权限请求、文件系统访问、terminal。
- ACP Registry 的分发意义；与 MCP（agent 内部工具）的分层关系。
- 采用面：编辑器与 CLI agent。

**03/04-agents-md.md**
- 起源与治理：AMP/agents.md 之争 → AGENTS.md → AAIF 托管。
- 格式与最佳实践：分层目录、就近原则、可执行命令、禁区内核。
- 工具支持矩阵（Codex、Copilot、Gemini、Claude Code 等）与 CLAUDE.md 兼容问题。
- 与 rules 文件、skills、hooks 的关系；踩坑（上下文膨胀、指令冲突）。

### 卷 4 · 工程化（从能跑到可上线）

**04/01-observability.md**
- 为什么 agent 的 trace 与普通服务不同：一次请求 = 多轮 LLM + 多次工具调用 + 分支。
- OpenTelemetry GenAI 语义约定；span 结构；记录 prompt/response/工具参数的成本与隐私。
- 工具面：LangSmith、Langfuse、Phoenix/Arize、Braintrust、OTel Collector。
- 回放、数据集沉淀、在线评估闭环。

**04/02-evaluation.md**
- 三层评测：单步（工具选择/参数）→ 轨迹（trajectory）→ 端到端任务成功率。
- 主流 benchmark：τ-bench/τ²-bench、SWE-bench 系、GAIA、WebArena/WebVoyager、OSWorld、AgentBench、Terminal-Bench；各自测什么、不能测什么。
- LLM-as-judge 的可靠性与校准；pass@k / pass^k；成本-方差权衡。
- 回归体系：黄金集、轨迹快照、A/B、CI 集成。

**04/03-security.md**
- 威胁模型：prompt injection（直接/间接）、tool poisoning、confused deputy、数据外泄、过度代理、供应链（MCP server / 包）。
- 权限设计：最小权限、capability、按工具/按路径授权、审批门、沙箱边界。
- 身份：agent 代表谁行动（OAuth、token 交换、on-behalf-of）、审计。
- 防御纵深：输入隔离、输出校验、egress 控制、双模型审查。
- 合规视角：日志留存、数据驻留、可解释性。

**04/04-reliability-and-hitl.md**
- 失败分类学：幻觉动作、工具误用、循环、超时、部分失败、状态不一致。
- 持久化执行：checkpoint、resume、幂等、补偿；Temporal/Restate/自建队列的取舍。
- 重试与降级：模型 fallback、工具 fallback、预算熔断。
- HITL 设计：审批点选择、批处理、异步人工、超时策略；"人在环上"而非"人在环中"。
- 长时任务（小时/天级）的工程约束。

**04/05-cost-latency.md**
- 成本结构：输入 token（含工具定义与历史）× 轮数；缓存命中率；工具调用开销。
- 优化手段：prompt caching、上下文压缩、模型路由（小模型做分类/抽取）、并行工具、投机执行、批处理、early exit。
- 延迟拆解：TTFT、解码、工具 RTT、串行轮次；流式与乐观 UI。
- 度量：每任务成本、每成功任务成本（cost per success）才是真指标。

**04/06-deployment.md**
- 形态谱系：库/SDK 嵌入 → 自建服务 → 托管 agent runtime（Bedrock AgentCore、Azure AI Foundry Agent Service、Vertex Agent Engine、Cloudflare Agents）→ 低代码平台。
- 关键差异：会话状态、沙箱、身份、网络、扩缩容、可观测接入。
- 自建参考架构：API 网关 + 编排服务 + 沙箱池 + 状态存储 + 队列 + 观测。
- 多租户与隔离；灰度与版本化（prompt/工具/模型三层版本）。

### 卷 5 · 产品形态与落地

**05/01-coding-agents.md**
- 形态：CLI（Claude Code、Codex CLI、Aider）、IDE（Cursor、Copilot）、云端异步（Devin、Codex cloud）、开源自托管（OpenHands）。
- 架构共性：repo 上下文构建、工具集（读/写/搜/跑测试）、沙箱、子 agent、hooks、AGENTS.md、权限审批。
- 评测：SWE-bench Verified、Terminal-Bench；真实采用中的瓶颈（review 成本、上下文、测试质量）。
- 以 DSH 作为"harness + 插件 + 技能"的案例拆解（如需要）。

**05/02-computer-use.md**
- 两种路线：DOM/无障碍树驱动 vs 纯视觉坐标点击。
- 代表：OpenAI Operator、Claude computer use、Browser Use、Gemini 浏览器 agent。
- 工程难点：页面漂移、反自动化、长流程恢复、凭据与支付、审计。
- 评测：WebArena、WebVoyager、OSWorld。

**05/03-deep-research.md**
- 任务特征：开放式问题、多源检索、长时、需要引用。
- 架构：planner → 并行 searcher → reader → 交叉验证 → 写作 → 引用核对。
- 代表产品与开源实现；质量评估（事实性、覆盖度、引用准确率）。
- 与普通 RAG 问答的边界。

**05/04-enterprise-platforms.md**
- 平台谱系：Copilot Studio、Foundry、Vertex、Dify、n8n、Zapier Agents。
- 能力对比：可视化编排、连接器、治理、多租户、计费。
- 企业落地的真实约束：IT 审批、数据边界、身份集成、SLA。
- "平台 vs 代码框架"的决策建议。

**05/05-vendor-agent-architectures.md｜大厂参考架构**
- 六家对照：Anthropic / OpenAI / Google / Microsoft / AWS / DeepSeek。
- 三个共同点（harness 化、MCP、子 agent）与四个分歧。
- 按处境选参考架构；锁定度看沙箱 / 记忆 / 身份三层。

### 卷 6 · 展望

**06/01-open-problems.md**
- 可靠性：能力已不是瓶颈，组合后的可靠性才是（引用 2026 综述观点）。
- 评测：缺长时任务、缺成本归一、缺可复现轨迹。
- 记忆与持续学习：跨会话、跨任务的稳定改进。
- 安全与治理：agent 身份、责任归属、审计标准。
- 标准化：协议收敛后的空白（计费、发现、信任）。
- 趋势判断（标注为主观）：runtime 商品化、harness 成为护城河、评测即基础设施。

## 4. 统一模板

**通用（TOPIC-TEMPLATE）**
```
---
status: draft | reviewed
baseline: 2026-11-20
sources: n
---
# 标题
## 一句话结论
## 问题与边界（解决什么 / 不解决什么）
## 核心概念与架构（Mermaid）
## 关键机制拆解
## 对比表
## 实践建议与常见坑
## 开放问题
## 参考来源（标题 · 机构 · 日期 · 链接）
```

**框架篇（FRAMEWORK-TEMPLATE）** 额外固定 10 维：
抽象层次 / 执行模型 / 状态与持久化 / 多 agent / 工具与 MCP / 可观测 / 评测 / 部署 / 语言与生态 / 许可证与治理，外加"最小示例（可选）""适用与不适用"。

## 5. 规模与批次

| 档位 | 篇数 | 体量 | 说明 |
|---|---|---|---|
| A 精简 | ~18 | 3–4 万字 | 框架合并为 4 篇、工程化合并为 3 篇，只讲结论与对比 |
| B 标准（推荐） | 38 | 8–10 万字 | 上述结构，每篇 150–300 行，框架一篇一框架 |
| C 深度 | 38+ | 12 万字+ | 在 B 基础上加可运行代码示例、benchmark 复现记录、更多一手来源 |

**交付批次**（每批完成后可 review，不阻塞后续）：
1. P0 骨架：README 骨架 + 模板 + 空目录 + 本大纲定稿
2. P1 总览：landscape / evolution / glossary
3. P2 基础：agent-loop / context / memory / tool / multi-agent
4. P3 协议：protocol-stack / MCP / A2A / ACP / AGENTS.md
5. P4 工程化：6 篇
6. P5 框架：选型矩阵 + 11 篇
7. P6 收尾：产品 4 篇 + outlook + references + README 定稿

## 6. 待确认

1. **规模**：A / B / C 哪一档？
2. **框架名单**：是否需要增删（如 AutoGen 单列、OpenHands、Dify、Mastra 单列；DSH 是否作为案例）？
3. **代码示例**：框架篇是否附最小可运行示例（显著增加篇幅与维护成本）？
4. **产品与商业**：卷 5 保留产品形态，但不做商业/市场份额分析，是否可以？
5. **时间与版本**：每篇是否强制标注"适用版本 + 事实核对日期"（推荐是）？
