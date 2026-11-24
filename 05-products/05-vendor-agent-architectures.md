---
status: draft
baseline: 2026-11-23
sources: 10
---

# 大厂 Agent 参考架构：六家的骨架与分歧

## 一句话结论

Anthropic、OpenAI、Google、Microsoft、AWS 与 DeepSeek 在 2026 年已经收敛到同一个骨架：**一个 harness 管住循环与上下文，一层 MCP 接工具，一组子 agent 做隔离，一道审批闸门管副作用，一套 OTel 约定管观测**。真正的分歧只有三处——单 agent 还是多 agent、会话归产品还是归平台、工具面开放还是闭环。

## 先看同一句需求

同一句需求——"每周一自动汇总上周的客户反馈，生成周报发到群里"——拿去问六家的默认做法，会得到六套不同的方案：Anthropic 让你写 skill + 子 agent + hook，OpenAI 给单 agent + handoff + sessions，Google 给你一棵层级 agent 树，Microsoft 给 agents + workflows，AWS 让你拼 Runtime + Memory + Gateway，DeepSeek 给插件化 harness 加审批 seam。需求一个字没变，默认组装方式全变了。

| 需求片段 | 六家的默认差异 | 对应概念 |
|---|---|---|
| "每周一自动跑" | 调度归产品、归 runtime，还是归你自己的 cron | loop 归属 |
| "读三个系统的反馈" | MCP 原生，还是自有 connector | 工具面开放度 |
| "汇总成一份周报" | 单 agent 长上下文，还是多个子 agent 分片 | 单/多 agent 分歧 |
| "发到群里" | hooks、guardrail，还是审批闸门 | 强制力放在哪一层 |

**参考架构是什么？** 人话不是白皮书，而是"你什么都不说时，它默认怎么组装"。类比点外卖的默认套餐：同样是"一荤一素"，每家配的例汤和主食不同；例子是六家都默认"harness + MCP + 子 agent + 审批 + OTel"，差别只在默认值。

**harness 是什么？** 人话是包在模型外面的那圈循环、上下文、工具与权限。类比汽车的座舱与仪表盘：发动机是模型，但方向盘、刹车、油表决定了这辆车能不能上路；例子是六家都在 2025–2026 年把 loop 从框架代码搬进了产品与 runtime。

同一句需求长出六套方案，差异不在能力清单，而在几个默认选择；下面把六家的骨架摊开对照。

## 问题与边界

- **解决什么**：把六家的"参考架构"放在同一张表里对比，回答"它们的 agent 到底怎么组装、哪些是共性、哪些是选择"。
- **不解决什么**：不做产品功能评测，不比较模型能力，不预测市场份额；框架 API 细节见[卷 2](../02-frameworks/00-selection-matrix.md)，协议见[卷 3](../03-protocols/00-protocol-stack.md)。
- **证据口径**：以官方文档与工程博客为主；厂商自述标注"厂商声称"，二手解读标注来源性质。
- **术语**：这里的"参考架构"指**产品与 SDK 体现出的默认组装方式**，不是某份架构白皮书。

## 六家对照

| 维度 | Anthropic | OpenAI | Google | Microsoft | AWS | DeepSeek |
|---|---|---|---|---|---|---|
| 入口产品 | Claude Code / Agent SDK | ChatGPT agent / Deep Research / Agents SDK | Gemini Enterprise / ADK | MAF / Foundry Hosted Agents | Strands / Bedrock AgentCore | DSH / Super-agent（探索） |
| loop 归属 | harness（Agent SDK 即产品） | 平台托管 harness + SDK | ADK 运行时 + 托管 runtime | MAF + Agent Harness | AgentCore Runtime + Strands | DSH 插件化 harness |
| 编排原语 | 子 agent、skills、hooks | handoff、guardrails、agents-as-tools | 层级 agent 树、sequential/parallel/loop | agents + workflows + middleware | swarm / graph | Router/Worker（提案） |
| 工具面 | MCP 原生 | MCP + hosted tools | MCP + A2A | MCP + 函数工具 | MCP + Gateway | MCP + 自有工具集 |
| 子 agent | 有，隔离上下文 | handoff 为主 | 有，树形转移 | 有，workflow 编排 | 有，swarm/graph | Worker + Subagent |
| 上下文策略 | compaction + 文件 + 渐进披露 | 平台管理 + 会话 | 会话 + 记忆库 | 压缩 + 文件记忆 + todo | Memory 服务 | 跨会话 + 任务隔离 |
| 沙箱 | 客户端执行 / 容器 | 原生 sandbox + 第三方 provider | 托管 runtime | Foundry 托管 | AgentCore 隔离 | 可替换 provider |
| 治理 | 权限模式 + hooks | guardrails + 审批 | IAM + 策略 | Entra Agent ID + 策略 | Identity + Policy | 审批 seam + 策略 |
| 观测 | OTel | tracing（内置） | Cloud Trace / OTel | OTel 内置 | CloudWatch / OTel | trace 事件 |

## 各家拆解

### Anthropic：harness 即产品

Claude Code 的扩展层是理解它的钥匙：`CLAUDE.md`（常驻约定）、**Skills**（按需加载的 Markdown 知识/流程，渐进披露）、**Subagents**（隔离上下文、只回摘要）、**Dynamic workflow**（Claude 自己写脚本并行跑多个 subagent）、**Hooks**（生命周期上执行确定性脚本，唯一能"强制"的层）、**MCP**（外部工具）（[Extend Claude Code](https://code.claude.com/docs/en/features-overview)）。

它的架构主张是"给模型一台电脑 + 一套可组合的扩展点"，而不是"给模型一张流程图"。多 agent 只在一处被明确采用：研究系统用 lead + subagents + CitationAgent，官方称比单 agent 高 90.2%，代价是约 15 倍 token，且承认同步等待是瓶颈（[How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)）。

> 判断：Anthropic 的差异化不在 loop，而在**"skills 的渐进披露 + hooks 的强制力"**这对组合——前者管上下文预算，后者管"说了必须做到"。

### OpenAI：平台托管 harness，SDK 保持轻

OpenAI 有两条线。产品线是"单 agent + 强工具"：Deep Research 用 o3 系模型 + 浏览 + Python，跑 5–30 分钟产出报告；ChatGPT agent 给模型一台虚拟电脑（浏览器、终端、连接器），端到端执行。SDK 线保持轻量：agents、tools、handoffs、guardrails、sessions、tracing，编排靠 handoff 而不是图。

2026-04 的架构调整值得注意：官方把 **harness（控制面）与 sandbox（计算面）拆开**，SDK 提供模型原生的 harness，sandbox 支持 Blaxel、Cloudflare、Daytona、E2B、Modal 等第三方 provider。这等于承认"循环"和"执行环境"是两件可以分开采购的事。

另一条信号是收敛：AgentKit 的 Agent Builder 与 Evals 于 2026-11-30 下线，官方建议回到 Agents SDK。可视化编排层不稳定，SDK + 协议 + runtime 才是沉淀下来的部分。

### Google：树形 agent + 协议先行

ADK 的默认形态是一棵**层级 agent 树**：root agent 依据子 agent 的 `description` 决定把对话转给谁，子 agent 还能再往下转；编排用 `SequentialAgent`、`ParallelAgent`、`LoopAgent` 这类工作流 agent 组合。它把"可预测的转移路径"当成卖点——树结构限制了谁能转给谁，调试因此更容易。

Google 的另一半是协议：A2A 解决跨 agent 委托，MCP 解决工具接入，两者在 2026 年都进入基金会治理。部署侧是 Gemini Enterprise Agent Platform（原 Vertex），Agent Runtime（原 Agent Engine）。

### Microsoft：合并后的统一底座

MAF 把 AutoGen 的对话式多 agent 与 Semantic Kernel 的企业能力合到一套 API：agents、workflows、middleware、OTel，Python 与 .NET 对齐。2026-06 的 Build 上补了 **Agent Harness**：函数调用、历史持久化、上下文压缩、todo 列表、文件记忆、skills、web search、工具审批、内置 OTel——基本是把"生产级 loop 该有什么"列了一遍。托管侧是 Foundry Hosted Agents，治理侧接 Entra Agent Identity。

它的优势场景很明确：.NET 栈、Azure 生态、需要企业身份与合规的团队。

### AWS：把 agent 拆成可采购的原语

AWS 的路线是"不赌框架，卖原语"。Strands 是轻量 SDK（model-driven、装饰器式工具、swarm/graph 多 agent），AgentCore 则把运行时拆成十余项服务：Runtime、Memory、Gateway、Identity、Observability、Code Interpreter、Browser、Policy、Evaluations、Optimization、Registry、Harness 等。好处是每一块都能单独替换，坏处是拼装成本与云绑定。

### DeepSeek：插件化 harness + Super-agent 探索

DSH 的 harness 是"一切皆插件"：模型适配器、工具、会话日志、loop 本身都可替换，组装单位是 profile + bundle + patch；扩展点包括 skills、subagent provider（进程内 spawn、fork、Claude Code、Codex、ACP）、审批 seam 与可替换沙箱。

在此之上，团队 2026-10-31 公开了 **Super-agent** 的架构探索（明确说明是思路探讨、非现有产品功能）：Router 常驻、多渠道接入、事件驱动、Worker 独立上下文并行、必要时再派 Subagent，目标是让人从"调度员"回到"目标设定与验收"（[报道](https://finance.sina.com.cn/roll/2026-11-01/doc-inixeyiz3146996.shtml)）。

## 三个共同点

1. **harness 化**：loop、上下文、工具、权限从"框架里的代码"变成"产品/运行时的一部分"。六家都在 2025–2026 年做了这件事。
2. **MCP 成为默认工具面**：不是"支持 MCP"，而是"工具就该通过 MCP 接"。差异只在托管网关与治理能力。
3. **子 agent 成为标准件**：无论叫 subagent、worker 还是 handoff，隔离上下文 + 只回摘要的模式已是共识。

## 四个分歧

| 分歧 | 一方 | 另一方 | 影响 |
|---|---|---|---|
| 单 agent vs 多 agent | OpenAI 产品偏单 agent + 强工具 | Anthropic 研究系统、DeepSeek Super-agent 偏多单元 | 成本、可调试性、上下文策略完全不同 |
| 会话归谁 | 产品托管（ChatGPT、Claude Code） | SDK/自建（Agents SDK、ADK、Strands） | 锁定度与数据边界 |
| 上下文策略 | compaction 为主（Claude Code、MAF） | 检索/记忆服务为主（Google、AWS） | 长任务稳定性与成本结构 |
| 工具面开放度 | MCP/A2A/ACP 全开放 | 自有 connector 生态 | 集成成本与生态红利 |

## 为什么是 2026 年收敛

四个条件同时成熟，才让"harness 化"成为共同选择，而不是某家的偏好：

1. **原生工具调用成熟**。function calling 不再是弱点，框架不必自己解析动作文本，编排层的价值随之下降。
2. **上下文成为主要成本**。压缩、缓存、渐进披露直接决定毛利，这些能力必须放在 loop 旁边，而不是散在业务代码里。
3. **安全事件倒逼**。间接注入与工具投毒让"权限、沙箱、审批"从可选项变成默认件。
4. **协议标准化**。MCP 无状态化之后，工具面可以像普通 HTTP 服务一样采购与治理。

结果是同一句话：**框架变薄，runtime 变厚**。

## 三个可复用的设计决策

跨厂商看，真正值得抄的不多，这三个是共识：

- **控制面与执行面分离**。OpenAI 把 harness 与 sandbox 拆开、DSH 把沙箱做成可替换 provider、AWS 把 Runtime 与 Code Interpreter 分成不同服务——都在承认"循环"和"跑代码的地方"是两件事。
- **上下文分层**。常驻约定（CLAUDE.md / AGENTS.md）、会话历史、检索内容、产物文件，四层各有各的预算与淘汰策略。六家的实现都能映射到这套分区。
- **强制力放在 hooks 与审批，不放在 prompt**。Claude Code 的 hooks、MAF 的工具审批、DSH 的 approval seam 都在做同一件事：能拦住的才叫规则，写在提示词里的只是建议。

## 常见误读

| 误读 | 实际 | 后果 |
|---|---|---|
| "用了某家 SDK 就有参考架构" | SDK 只提供积木，runtime、沙箱、治理要自己补 | 上线时才发现缺一半 |
| "多 agent 更先进" | 产品线主流仍是单 agent + 强工具，多 agent 集中在研究与长任务 | 成本翻倍、成功率反降 |
| "支持 MCP 就等于工具体系可用" | 工具描述质量、权限与网关才是瓶颈 | 工具越多，选错率越高 |
| "harness 可以随时替换" | 没有标准接口，迁移约等于重写 | 锁定被低估 |

## 对选型的启示

| 你的处境 | 更接近的参考架构 | 理由 |
|---|---|---|
| 做编码 agent | Anthropic / DeepSeek 的 harness 形态 | 工具密集、需要 skills 与审批、上下文靠文件 |
| 做通用任务产品 | OpenAI 的"单 agent + 虚拟电脑" | 工具面广、验收靠结果而非轨迹 |
| 已有 GCP 栈 | Google ADK + Agent Runtime | 树形编排与 A2A 原生 |
| 已有 Azure/.NET | MAF + Foundry | 身份、托管、OTel 一体 |
| 要自建可控 runtime | AWS AgentCore 或 DSH 式插件化 | 原语可替换，避免整体锁定 |
| 跨组织协作 | MCP + A2A | 协议已收敛，别再自造 |

判断锁定度可以只看三层：**沙箱、记忆、身份**。这三层如果只能用厂商的，迁移成本就高；如果都能替换，框架与模型都可以换。

## 开放问题

- **多 agent 的收益归因**：Anthropic 的 90.2% 与"15 倍 token"是同一份内部评测，缺独立复现；OpenAI 的产品路线则回避了多 agent。
- **harness 的可移植性**：同一模型换 harness 有十几个百分点的差距，但 harness 本身没有标准接口，迁移等于重写。
- **记忆的归属**：记忆放在产品、平台还是应用？三者的删除权与合规责任不同。
- **主动执行的治理**：Super-agent 类系统在无人 @ 时行动，审批与审计模型还没有共识。
- **评测口径**：各家的"任务成功率"定义不同，跨厂商数字不可比。
- **厂商与框架的边界**：当模型厂商同时卖 harness 与 runtime，第三方框架的生态位在哪里？

## 参考来源

- Extend Claude Code（CLAUDE.md / Skills / Subagents / Hooks / MCP） · Claude Code Docs · 2026-11 · https://code.claude.com/docs/en/features-overview
- How we built our multi-agent research system · Anthropic · 2025-06 · https://www.anthropic.com/engineering/multi-agent-research-system
- The next evolution of the Agents SDK（harness 与 sandbox 拆分） · OpenAI · 2026-04 · https://openai.com/index/the-next-evolution-of-the-agents-sdk/
- Introducing Deep Research · OpenAI · 2025-02 · https://openai.com/index/introducing-deep-research/
- Agent Builder 与 Evals 弃用时间线 · OpenAI · 2026-06 · https://developers.openai.com/api/docs/deprecations
- Build multi-agent systems with ADK（层级 agent 树与转移） · Google · 2026 · https://codelabs.developers.google.com/codelabs/production-ready-ai-with-gc/3-developing-agents/build-a-multi-agent-system-with-adk
- Microsoft Agent Framework at BUILD 2026（Agent Harness / Hosted Agents） · Microsoft DevBlogs · 2026-06 · https://devblogs.microsoft.com/agent-framework/microsoft-agent-framework-at-build-2026-announce/
- Amazon Bedrock AgentCore（Runtime / Memory / Gateway / Identity / Policy） · AWS · 2026 · https://docs.aws.amazon.com/bedrock-agentcore/
- Super-agent 设计探索：迈向更自主的 Agent 系统 · DeepSeek Harness 团队 · 2026-10-31 · https://finance.sina.com.cn/roll/2026-11-01/doc-inixeyiz3146996.shtml
- Agent2Agent 与 AAIF 治理 · Linux Foundation · 2026-08 · https://a2a-protocol.org/latest/blog/2026/08/27/a-new-chapter-for-a2a-joining-the-agentic-ai-foundation/
