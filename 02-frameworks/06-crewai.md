---
status: draft
baseline: 2026-11-20
sources: 9
---

# CrewAI

## 一句话结论

CrewAI 用"角色团队"的隐喻包装多 agent：`role` / `goal` / `backstory` 定义成员，`Task` 定义工作，`Process` 决定谁先做。1.x 之后它其实是两套东西——Crews 负责"自主协作"，Flows 负责"确定性编排"。适合演示、内容生产、流程相对固定的多角色任务；不适合需要精确状态机、可恢复执行或严格成本控制的场景。

## 先看一个任务

某咨询团队每周要出一篇行业简报：研究员找三条可核实的事实，写手压成六百字，审稿专门挑"没有出处的断言"。三个人都在，卡住的不是写，而是"谁先做、做到什么算完、退稿怎么退"——用一个长 prompt 把它们串起来，模型经常在同一轮里把研究和写作混着做。

在这个框架里，这件事先被拆成角色，再被拆成工作：

| 拆法 | 对应对象 | 这个任务里是什么 |
|---|---|---|
| 谁来做 | `Agent` | 研究员、写手、审稿，各带 `role` / `goal` / `backstory` |
| 做什么 | `Task` | "找三条事实""成文一篇""挑出无出处断言"，每条带 `expected_output` |
| 按什么顺序 | `Crew` + `Process.sequential` | 三个 task 依次跑，后一个用 `context` 接前一个的输出 |
| 退稿怎么办 | `Flow` | 审稿不达标就回到写作步骤，由代码决定，不由模型决定 |

三个抽象记住就够进门：**`Agent` 是岗位说明书**（`role` 是职位名，`goal` 是 KPI，`backstory` 是"这个人为什么可信"，它会进每一轮 prompt）；**`Task` 是工单**（`expected_output` 是验收标准，不写就等于放弃验收）；**`Crew` 是项目组，`Flow` 是流水线**——前者把"谁做什么"交给模型，后者把"何时做什么"收回代码。三者的骨架可以压成一行：`Agent` 装人，`Task` 装活，`Crew(process)` 或 `@start / @listen / @router` 决定顺序。

所以读下面的十维时，先问一句：你手里的事更像"组一个项目组"，还是更像"排一条流水线"？

## 定位与谱系

- 出品方与许可：crewAI, Inc.（创始人 João Moura），MIT 许可（仓库 LICENSE，2026-11-20 核对）。
- 当前版本：`crewai` 1.15.44，2026-11-19 发布；要求 Python ≥ 3.10 且 < 3.14（PyPI，2026-11-20 核对）。
- 前身与合并：无前身，2024 年起从零自建，官方反复强调"不依赖 LangChain"。1.x 是一次大版本重整：Crews 与 Flows 成为并列的一等抽象。
- 在七层模型中的位置：**编排**。它不管模型、不管托管 runtime（那部分交给商业产品 CrewAI AMP），核心资产是 agent/task/flow 这三层抽象与配套的 memory、knowledge、guardrails。
- 生态位：介于"轻量 SDK"（OpenAI Agents SDK）与"低层图框架"（LangGraph）之间——比前者多一套角色与流程模型，比后者少一份显式状态图。

1.x 的几个变化值得单独记，因为它们改变了老代码的行为：Crews 与 Flows 从"一个特性"升为"两套并列抽象"；记忆从四类（短期/长期/实体/外部）合并成统一的 `Memory`；`crewai create crew` 的默认脚手架从 Python/YAML 换成 JSON-first。这三点加起来，意味着 2024–2025 年的大部分教程在当前版本上只能当思路参考。

### 它解决什么、不解决什么

- 解决：让"多个角色分工完成一件事"在几十行内跑起来，并且默认带上 memory、knowledge、guardrails、可观测集成。
- 不解决：可恢复的长流程（没有 LangGraph 那种 checkpoint + 时间旅行）、细粒度控制流（Flows 有事件，但不是状态机）、跨进程 agent 协议（A2A 未见官方一等支持，截至 2026-11）。
- 划界：Crews 是"自治团队"，Flows 是"事件驱动流程"，两者可以嵌套但语义不同；很多团队把该用 Flows 的场景写成了 Crews，然后抱怨不可控。委托本身的机制问题见 [多智能体协作](../01-foundations/05-multi-agent.md)。
- 与相邻框架的坐标：相比 LangGraph，它没有显式状态图与 checkpoint；相比 OpenAI Agents SDK，它多了角色/流程模型但少了官方 tracing 的一等支持；相比 AutoGen 血缘的 Microsoft Agent Framework，它更偏"业务流程"而非"研究型对话"。

## 核心概念

```mermaid
flowchart TB
    subgraph Crews[自主协作]
    A1[Agent: role/goal/backstory] --> T1[Task]
    A2[Agent] --> T2[Task]
    T1 --> P[Process: sequential / hierarchical]
    T2 --> P
    end
    subgraph Flows[确定性编排]
    S[@start] --> L[@listen]
    L --> R{@router}
    R -->|分支A| L2[@listen]
    R -->|分支B| L3[@listen]
    end
    P -.作为一步.-> L
```

| 概念 | 职责 | 容易误解的点 |
|---|---|---|
| `Agent` | 角色定义：`role`、`goal`、`backstory`，外加工具、LLM、迭代上限 | backstory 是 prompt 的一部分，越长越贵，不是"文档注释" |
| `Task` | 一件可验收的工作：`description`、`expected_output`，可带 `context` 依赖 | `expected_output` 是给模型看的验收标准，不写就等于放弃验收 |
| `Crew` | agent + task 的集合，加一个 `Process` | 建了 Crew 不等于会协作，`allow_delegation` 默认是 False |
| `Process` | `sequential` 按序执行；`hierarchical` 由 manager 分配与校验 | hierarchical 必须给 `manager_llm` 或 `manager_agent` |
| `Flow` | 事件驱动编排：`@start` / `@listen` / `@router`，状态用 dict 或 Pydantic 模型 | Flow 的步骤是普通 Python，不调模型也能跑 |
| `Memory` | 1.15 起的统一记忆：`remember` / `recall` / `forget`，LLM 推断 scope 树 | 已取代旧的 short-term / long-term / entity / external 四类记忆 |

两套抽象并存不是历史包袱，而是对两类不确定性的分工：Crews 把"谁来做什么"的不确定性交给模型，适合问题本身没被完全定义的时候；Flows 把"何时做什么"的不确定性收回代码，适合步骤已知、只有局部需要判断的时候。选错的代价不对称——该用 Flow 却用了 Crew，得到的是不可复现；该用 Crew 却用了 Flow，得到的是过度编排。

**Crews 与 Flows 的选择表**（这是本篇最该记住的一张表）：

| 判断项 | 选 Crews | 选 Flows |
|---|---|---|
| 下一步谁做 | 模型临场决定（委托/分派） | 代码决定（事件与路由） |
| 流程形态 | 探索型、角色互补 | 固定骨架、少量判断点 |
| 状态 | 隐式，主要在对话里 | 显式 `state`，可持久化、可 fork |
| 失败恢复 | 基本没有 | 有：按步骤落盘，可从快照重来 |
| 可复现性 | 差（同输入不同路径） | 好（分支由代码决定） |
| 典型场景 | 调研、多视角评审、内容创作 | 审批流、ETL + LLM 步骤、带人工卡点的流水线 |

## 执行模型

**Crews 是"委托"，Flows 是"事件"**，两条执行路径的区别决定了一切。

Crew 的 sequential 过程很直白：按 task 列表顺序跑，前一个 task 的输出通过 `context` 传给后一个。hierarchical 则多一层 manager：任务不预分配给具体 agent，由 manager 根据能力描述派活、审结果、判断是否完成——控制权在模型手里，不在代码里。

Flows 相反：`@start` 标记入口，`@listen` 监听某个方法的输出，`@router` 返回分支名决定走哪条边，`or_` / `and_` 组合触发条件。状态是显式的（`self.state`，dict 或 Pydantic 模型），每个 Flow 实例自带一个 UUID 作为 state id。持久化分两级：类级持久化（每次运行都落盘）与方法级持久化（只存指定步骤的结果），还支持从某个快照 fork 出新 state 而保留原历史。

一个 Crew 任务在内部其实是这样跑的：agent 拿到 task 描述与 `expected_output` → 召回相关 memory → 组 prompt（含 backstory、工具定义、上下文）→ 模型决定调工具还是直接作答 → 若 `allow_delegation=True` 且模型选择委托，则插入一次子任务 → 循环到 `max_iter` 或给出答案 → 输出交给下一个 task 或写回 Flow。这一段解释了为什么"加一个 agent"往往同时增加了 token、延迟和不确定性：它增加的是循环，不只是角色。

Flow 侧还有两个生产向能力：**人工反馈**（在步骤里暂停等人输入，再继续）与**用量指标**（每次 Flow 运行给出 token 统计）。人工卡点在 Crews 里只能做到 task 级的 `human_input`，粒度不同；用量指标则是成本归因的起点。

选择规则可以压缩成一句话：**流程能画成固定的图，就用 Flows；流程需要"谁来做"由模型临场决定，才用 Crews。**

> 判断：CrewAI 的 Crews 解决的是"叙事"，Flows 解决的才是"生产"。1.x 把 Flows 提到一等地位，本质上是承认了纯角色协作在生产环境不够用。

## 十维对比

| 维度 | 说明 |
|---|---|
| 抽象层次 | 高层：角色 + 任务 + 流程；比图框架高，比纯 prompt 模板低 |
| 执行模型 | Crews 顺序/层级委托；Flows 事件驱动（start → listen → router），单进程异步 |
| 状态与持久化 | Flow state 显式且可持久化（类级/方法级，可 fork）；Crew 内部状态基本不落盘 |
| 多 agent 支持 | 一等公民：角色分工 + 委托；hierarchical 由 manager agent 调度，但拓扑是隐式的 |
| 工具与 MCP | 自带工具库；MCP 支持 stdio / SSE / streamable-http、多 server 并存 |
| 可观测 | 商业版 AMP 提供控制平面 tracing；开源侧靠 Langfuse、Datadog、Arize Phoenix、Braintrust 等集成 |
| 评测集成 | 无内建评测器；有 guardrails（task/agent 级）与官方"评估用例"指南，正式评测靠外部工具 |
| 部署形态 | 自托管（容器/函数）；商业版 AMP 提供云与 on-prem 部署 |
| 语言与生态 | Python 主线；CLI 脚手架默认 JSON-first（`agents/*.jsonc`、`crew.jsonc`），老项目用 `--classic` 回到 Python/YAML |
| 许可证与治理 | MIT；公司主导，商业功能（AMP、Studio）闭源，开源与商业边界清晰 |

在多家框架之间做取舍时，建议先用 [选型矩阵](00-selection-matrix.md) 的四问过滤一遍，再看 Crews/Flows 这套叙事是否真的匹配你的流程形态。

## 最小示例

```python
from crewai import Agent, Task, Crew, Process

researcher = Agent(role="研究员", goal="找出三条可核实的事实",
                   backstory="你只相信一手来源。", allow_delegation=False, max_iter=8)
writer = Agent(role="撰稿", goal="写成 200 字结论",
               backstory="你擅长把事实压成短句。")

t1 = Task(description="调研 {topic}", expected_output="三条要点", agent=researcher)
t2 = Task(description="成文", expected_output="200 字结论", agent=writer, context=[t1])

crew = Crew(agents=[researcher, writer], tasks=[t1, t2], process=Process.sequential)
crew.kickoff(inputs={"topic": "agent 框架"})
```

两个细节值得注意：`allow_delegation=False` 是显式关掉的（默认也是 False，但很多人以为开着 Crew 就会自动协作）；`context=[t1]` 才是把上游输出喂给下游的正确方式，靠 prompt 里写"参考上一步"并不可靠。

同一件事用 Flow 表达，控制点就回到代码里：

```python
from crewai.flow.flow import Flow, start, listen, router
from pydantic import BaseModel

class S(BaseModel):
    score: float = 0.0

class Review(Flow[S]):
    @start()
    def draft(self):
        return run_writer_crew()          # 把 Crew 当成一个步骤

    @listen(draft)
    def judge(self, text):
        self.state.score = score_text(text)
        return self.state.score

    @router(judge)
    def route(self):
        return "ok" if self.state.score > 0.8 else "retry"

    @listen("retry")
    def again(self):
        return run_writer_crew(strict=True)
```

同一个业务，Crew 版本省代码、Flow 版本省事故。生产上常见的组合是"外层 Flow + 内层 Crew"：骨架、状态、重试由 Flow 管，需要多角色碰撞的那一步交给 Crew。

## 适用与不适用

- **适合**：内容生产流水线（调研 → 撰写 → 审校）、需要多个"视角"互相检查的分析任务、把现有 Python 脚本串成带 LLM 步骤的流程（这正是 Flows 的强项）。
- **适合**：团队里产品/业务同学要读得懂编排——role/goal/backstory 的心智负担远低于状态图。
- **适合**：把已有的 Python 业务代码"局部 agent 化"——Flow 的步骤就是普通函数，可以在任意位置插入一次 LLM 调用，而不必把整个系统改写成 agent。
- **适合**：把"人审"作为一个正式步骤的流程——Flow 的原生人工反馈比自己在 Crew 外面包一层队列干净得多。
- **适合**：教学与内部推广。角色叙事让非工程同学第一次能读懂"系统里发生了什么"，这是很多框架做不到的。
- **不适合**：长流程、需要断点续跑与精确重试。Crew 没有 checkpoint，崩了就从头发车；这块应换 [LangGraph](01-langgraph.md) 一类带持久化的图框架。
- **不适合**：成本敏感的高频调用。角色 prompt + task context + memory 召回会层层叠加，见 [成本与延迟](../04-engineering/05-cost-latency.md)。
- **不适合**：需要严格可复现的流程（合规、计费）。hierarchical 的调度由模型决定，同样输入不保证同样路径。
- **不适合**：需要跨进程、跨组织的 agent 互操作。A2A 未见官方一等支持（截至 2026-11），需要自己在工具层接。

## 常见坑

- **把该用 Flows 的场景写成 Crews**：典型症状是"跑三次三种任务分解"，然后不断加 prompt 约束。触发条件：流程其实固定、但团队想保留"自主性"。后果：不可复现、调试靠猜。规避：先用 Flows 画出确定的骨架，只在"需要判断"的那一步嵌一个 Crew 或单 agent。
- **hierarchical 的 token 账单翻倍**：manager 的规划、派活、审阅都是额外模型调用，且每次都要带全部 agent 的能力描述。规避：只在任务确实需要动态分派时用；固定流程用 sequential + 明确 `agent`。
- **backstory 写成小作文**：它是每轮 prompt 的一部分，一个 800 字的 backstory 乘以任务数就是纯成本。规避：控制在两三句，把领域知识放 knowledge 源而不是 backstory。
- **1.15 升级后 memory 配置语义变了**：旧的 short-term / long-term / entity / external 四类记忆被统一 `Memory` 取代，老代码里的记忆参数不再对应原来的行为，表现为"记忆像是没了"或"召回变慢变贵"。规避：升级前读 memory 迁移说明，按 scope 重新设计。
- **`crewai create crew` 生成的目录和教程不一致**：现在默认 JSON-first（`agents/*.jsonc` + `crew.jsonc`），网上大量 YAML/Python 教程对应的是旧脚手架，照抄会找不到文件。规避：新项目按当前脚手架走，需要旧结构加 `--classic`。
- **memory 的 LLM 推断本身要花钱**：`remember()` 会调模型推断 scope、类别、重要性，批量写入时这是一笔隐性开销。规避：批量导入走显式 `scope=`，别让它自由推断。
- **hierarchical 的 manager 用弱模型会放大问题**：manager 负责拆任务与验收，它判断失误时下游 agent 会把错的任务做得很完整。触发条件：为了省钱把 `manager_llm` 设成小模型。规避：manager 与执行者同级或更强，并且给 `expected_output` 写可检验的验收条件。
- **verbose 日志被当成可观测**：控制台输出不是 trace，没有 span、没有耗时分解，出问题仍然定位不到是哪一步。规避：接 Langfuse/OpenTelemetry 一类，或直接用带 trace 的托管方案。
- **`expected_output` 写成一句形容词**："高质量的报告"不是验收标准，模型会按自己的理解收尾，guardrails 也没法校验。规避：写成可检查的约束（结构、字段、字数、必须包含的证据）。
- **Flow 的状态里塞大对象**：持久化会把 state 整体序列化，塞进原始文档或大列表后，每次落盘都变慢，fork 还要复制一份。规避：state 只放指针与摘要，原文放外部存储。

## 开放问题

- Crews 的隐式拓扑能否被"导出成图"？目前只能靠日志还原，缺少可评审的静态结构。
- 统一 Memory 依赖 LLM 做 scope 推断，规模上去后的延迟与一致性如何，官方未给基准。
- 开源版与 AMP 的能力边界（tracing、部署、RBAC）会不会继续拉大，直接影响"先用开源再上商业"的路径假设。
- Guardrails 目前偏"输出校验"，缺少与流程级重试、降级的联动语义。
- JSON-first 脚手架让"配置即代码"更彻底，但也让 diff、review、CI 校验的实践需要重建——官方尚未给出配套的校验工具链。
- Crews 里的"委托"缺少作用域与预算约束：一个 agent 可以在自己的迭代里反复派活，成本上限只能靠 `max_iter` 与外部超时兜。
- 官方基准多为自己发布的对比（如"比 LangGraph 快 5.76 倍"这类），缺少第三方同条件复现，选型时不宜直接引用。

## 参考来源

- crewai 1.15.44 · PyPI · 2026-11-19 · https://pypi.org/project/crewai/
- crewAIInc/crewAI（MIT，含 Crews/Flows 定位与 JSON-first 脚手架）· GitHub · 2026-11-20 · https://github.com/crewAIInc/crewAI
- Agents（`role`/`goal`/`backstory` 与 `allow_delegation`、`max_iter`）· docs.crewai.com v1.15.44 · 2026-11 · https://docs.crewai.com/v1.15.44/en/concepts/agents
- Processes（sequential / hierarchical 与 manager）· docs.crewai.com v1.15.44 · 2026-11 · https://docs.crewai.com/v1.15.44/en/concepts/processes
- Flows（`@start`/`@listen`/`@router`、state、持久化与 fork）· docs.crewai.com v1.15.44 · 2026-11 · https://docs.crewai.com/v1.15.44/en/concepts/flows
- Memory（统一记忆、scope 树、composite 评分）· docs.crewai.com v1.15.44 · 2026-11 · https://docs.crewai.com/v1.15.44/en/concepts/memory
- Crews（Crew 与 Task 的组装方式）· docs.crewai.com v1.15.44 · 2026-11 · https://docs.crewai.com/v1.15.44/en/concepts/crews
- MCP 集成（stdio / SSE / streamable-http）· docs.crewai.com v1.15.44 · 2026-11 · https://docs.crewai.com/v1.15.44/en/mcp/overview
- 可观测集成（Langfuse 等）· docs.crewai.com v1.15.44 · 2026-11 · https://docs.crewai.com/v1.15.44/en/observability/langfuse
