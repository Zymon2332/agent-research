# 主流 Agent 技术体系（2026-11 基线）

面向技术选型与架构评审的结构化文档集：**36 篇，一篇一主题**，按"总览 → 基础 → 框架 → 协议 → 工程化 → 产品 → 展望"分层组织。每篇自带结论、对比表、踩坑与来源，可独立阅读。

- 时间基线：**2026-11-20**。所有版本号与规范状态标注核实日期，生态变化快，引用前请先看篇内的"基线"。
- 写作规范：[STYLE.md](STYLE.md)；目录契约：[OUTLINE.md](OUTLINE.md)。
- 全量来源汇总：[references.md](references.md)。

## 怎么读

| 目的 | 路径 | 预计 |
|---|---|---|
| 快速建立坐标系 | [技术全景](00-overview/01-landscape.md) → [协议分层](03-protocols/00-protocol-stack.md) → [选型矩阵](02-frameworks/00-selection-matrix.md) | 40 分钟 |
| 做框架选型 | [选型矩阵](02-frameworks/00-selection-matrix.md) → 对应框架篇 → [工程化](04-engineering/01-observability.md) | 2 小时 |
| 从零搭一个能上线的 agent | [Agent 循环](01-foundations/01-agent-loop.md) → [上下文工程](01-foundations/02-context-engineering.md) → [MCP](03-protocols/01-mcp.md) → [可靠性](04-engineering/04-reliability-and-hitl.md) → [部署](04-engineering/06-deployment.md) | 3 小时 |
| 新人 onboarding | 按卷顺序通读 | 1–2 天 |

## 目录

### 卷 0 · 总览

| 篇目 | 主题 | 状态 |
|---|---|---|
| [01-landscape](00-overview/01-landscape.md) | 2026 技术全景与七层模型 | 已完稿 |
| [02-evolution](00-overview/02-evolution.md) | 四代演进与关键时间线 | 已完稿 |
| [03-glossary](00-overview/03-glossary.md) | 术语表与易混词辨析 | 已完稿 |

### 卷 1 · 基础理论与范式

| 篇目 | 主题 | 状态 |
|---|---|---|
| [01-agent-loop](01-foundations/01-agent-loop.md) | 感知-思考-行动循环与控制结构 | 已完稿 |
| [02-context-engineering](01-foundations/02-context-engineering.md) | 上下文预算、压缩与卸载 | 已完稿 |
| [03-memory](01-foundations/03-memory.md) | 记忆分类、实现谱系与写入策略 | 已完稿 |
| [04-tool-use-and-sandbox](01-foundations/04-tool-use-and-sandbox.md) | 工具 schema 与沙箱执行 | 已完稿 |
| [05-multi-agent](01-foundations/05-multi-agent.md) | 多智能体拓扑与失败模式 | 已完稿 |

### 卷 2 · 主流框架

| 篇目 | 主题 | 状态 |
|---|---|---|
| [00-selection-matrix](02-frameworks/00-selection-matrix.md) | 十维对比与决策树 | 已完稿 |
| [01-langgraph](02-frameworks/01-langgraph.md) | 图编排与 checkpoint | 已完稿 |
| [02-microsoft-agent-framework](02-frameworks/02-microsoft-agent-framework.md) | AutoGen + Semantic Kernel 合并体 | 已完稿 |
| [03-openai-agents-sdk](02-frameworks/03-openai-agents-sdk.md) | 轻量 agents/handoffs/guardrails | 已完稿 |
| [04-google-adk](02-frameworks/04-google-adk.md) | Gemini 生态与 A2A 一等公民 | 已完稿 |
| [05-claude-agent-sdk](02-frameworks/05-claude-agent-sdk.md) | 产品化 harness | 已完稿 |
| [06-crewai](02-frameworks/06-crewai.md) | 角色协作叙事 | 已完稿 |
| [07-llamaindex](02-frameworks/07-llamaindex.md) | 数据层优先的编排 | 已完稿 |
| [08-pydantic-ai](02-frameworks/08-pydantic-ai.md) | 类型安全与结构化输出 | 已完稿 |
| [09-strands-agents](02-frameworks/09-strands-agents.md) | AWS 的 model-driven SDK | 已完稿 |
| [10-agno-and-lightweight](02-frameworks/10-agno-and-lightweight.md) | 轻量 runtime 与 CodeAgent | 已完稿 |
| [11-typescript-stack](02-frameworks/11-typescript-stack.md) | TS/JS 生态选型 | 已完稿 |

### 卷 3 · 互操作协议与标准

| 篇目 | 主题 | 状态 |
|---|---|---|
| [00-protocol-stack](03-protocols/00-protocol-stack.md) | 四层协议全景与邻近规范 | 已完稿 |
| [01-mcp](03-protocols/01-mcp.md) | Model Context Protocol 与 2026-07-28 规范 | 已完稿 |
| [02-a2a](03-protocols/02-a2a.md) | Agent2Agent 与任务生命周期 | 已完稿 |
| [03-acp](03-protocols/03-acp.md) | 编辑器-Agent 解耦 | 已完稿 |
| [04-agents-md](03-protocols/04-agents-md.md) | 仓库级上下文约定 | 已完稿 |

### 卷 4 · 工程化

| 篇目 | 主题 | 状态 |
|---|---|---|
| [01-observability](04-engineering/01-observability.md) | trace、语义约定与回放 | 已完稿 |
| [02-evaluation](04-engineering/02-evaluation.md) | 三层评测与 benchmark 边界 | 已完稿 |
| [03-security](04-engineering/03-security.md) | 注入、权限与供应链 | 已完稿 |
| [04-reliability-and-hitl](04-engineering/04-reliability-and-hitl.md) | 持久化执行与人工介入 | 已完稿 |
| [05-cost-latency](04-engineering/05-cost-latency.md) | 成本结构、缓存与路由 | 已完稿 |
| [06-deployment](04-engineering/06-deployment.md) | 托管 runtime 与自建架构 | 已完稿 |

### 卷 5 · 产品形态与落地

| 篇目 | 主题 | 状态 |
|---|---|---|
| [01-coding-agents](05-products/01-coding-agents.md) | 编码 Agent 的架构共性 | 已完稿 |
| [02-computer-use](05-products/02-computer-use.md) | 浏览器 / GUI Agent | 已完稿 |
| [03-deep-research](05-products/03-deep-research.md) | 深度研究 Agent | 已完稿 |
| [04-enterprise-platforms](05-products/04-enterprise-platforms.md) | 企业平台与低代码 | 已完稿 |

### 卷 6 · 展望

| 篇目 | 主题 | 状态 |
|---|---|---|
| [01-open-problems](06-outlook/01-open-problems.md) | 开放问题、风险与趋势 | 已完稿 |

## 文档约定

- **事实**：带来源链接，标注机构与日期；无法核实的写"截至 2026-11 未见权威数据"。
- **判断**：用 `> 判断：` 引用块标出，与事实分离。
- **术语**：首次出现给英文，之后用英文术语。
- **图**：Mermaid 内联，可 diff。
- **篇幅**：每篇 150–300 行。

## 更新日志

- 2026-11-20：初始化目录、规范与模板，开始按批次写作。
- 2026-11-20：36 篇全部完稿，共约 7,700 行；汇总去重来源 300 条，跨篇审校完成。
