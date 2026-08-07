# DeepResearch 论文汇总对比

> 本文件用于把当前目录中的 7 篇 DeepResearch / Search Agent / Agentic RL 相关论文进行统一梳理、横向对比和实践映射，方便后续整理、复盘，以及为“业务型预测 Agent”项目选型。  
> 覆盖论文：MiroThinker、REDSearcher、MiroThinker-1.7 & H1、S1-DeepResearch、Tongyi DeepResearch、RubricEM、Quest。
> 原文入口：[MiroThinker](https://arxiv.org/abs/2511.11793) · [REDSearcher](https://arxiv.org/abs/2602.14234) · [MiroThinker-1.7 & H1](https://arxiv.org/abs/2603.15726) · [S1-DeepResearch](https://arxiv.org/abs/2606.15367) · [Tongyi DeepResearch](https://arxiv.org/abs/2510.24701) · [RubricEM](https://arxiv.org/abs/2605.10899) · [Quest](https://arxiv.org/abs/2605.24218)

---

## 0. 总览结论

这几篇论文其实是在回答同一个大问题：

```text
如何把一个通用大模型训练成真正能做长程研究的 Agent？
```

它们的共同答案不是“把上下文拉长”或“接一个搜索工具”这么简单，而是：

```text
高质量任务构造
+ 工具交互环境
+ 长程轨迹合成
+ 上下文/记忆管理
+ SFT 冷启动
+ 偏好优化或过程验证
+ RL 环境交互
+ Verifier / Rubric / Reward 体系
+ 真实或模拟评测闭环
```

如果要服务你的目标——构建一个**业务型预测 DeepResearch Agent**，最核心的落地路线应该是：

```text
Quest / REDSearcher / S1 负责“怎么构造复杂任务和评价标准”
Tongyi / MiroThinker 负责“怎么组织完整训练管线”
MiroThinker-1.7 & H1 / RubricEM 负责“怎么提升长程交互质量、验证和 RL 信用分配”
再叠加业务预测特有的 Point-in-Time、真实 outcome 回流和概率校准
```

---

## 1. 论文清单

| 序号 | 论文 | 本地详解文件 | 核心关键词 |
|---:|---|---|---|
| 1 | MiroThinker: Pushing the Performance of Open Deep Research Models through Interactive Scaling | [2025-11-18-mirothinker.md](./2025-11-18-mirothinker.md) | Interactive Scaling、Agentic SFT、DPO、GRPO、工具交互 |
| 2 | REDSearcher: A Scalable and Cost-Efficient Deep Search Framework | [2026-02-15-redsearcher.md](./2026-02-15-redsearcher.md) | 复杂任务合成、图拓扑复杂度、证据分散度、模拟环境、RL Query Curation |
| 3 | MiroThinker-1.7 & H1 | [2026-03-16-mirothinker-1.7-h1.md](./2026-03-16-mirothinker-1.7-h1.md) | Atomic Capability、Step Loop、Episode Restart、Local/Global Verifier、Heavy Mode |
| 4 | S1-DeepResearch | [2026-06-13-s1-deep-research.md](./2026-06-13-s1-deep-research.md) | Graph-grounded task、AgentLoop、Trajectory Refinement、多维轨迹验证 |
| 5 | Tongyi DeepResearch Technical Report | [2025-11-04-tongyi-deep-research.md](./2025-11-04-tongyi-deep-research.md) | Agentic Mid-training、SFT、RL、环境交互、Heavy Mode、Model Merging |
| 6 | RubricEM: Meta-RL with Rubric-guided Policy Decomposition beyond Verifiable Rewards | [2026-05-11-rubricem.md](./2026-05-11-rubricem.md) | Rubric-guided scaffold、Stage-Structured GRPO、Reflection Meta-Policy、Rubric Bank |
| 7 | Quest: Training Frontier Deep Research Agents with Fully Synthetic Tasks | [2026-05-22-quest.md](./2026-05-22-quest.md) | Fully Synthetic Tasks、Rubric Tree、Context State、MT/SFT/RL、异步 RL |

---

## 2. 一句话横向理解

| 论文 | 一句话理解 |
|---|---|
| MiroThinker | 强 DeepResearch Agent 的能力来自和工具环境的有效交互扩展，而不是只靠模型参数或长上下文。 |
| REDSearcher | 训练 Deep Search Agent 的关键是可控、可扩展、低成本地合成复杂搜索任务，并控制任务难度和证据分散度。 |
| MiroThinker-1.7 & H1 | 长程交互不是越长越好，要提升每一步工具/推理质量，并用 Local/Global Verifier 做 Heavy-duty reasoning。 |
| S1-DeepResearch | DeepResearch 不只是搜索答案，而是图驱动任务构造、长程 AgentLoop 执行和多维轨迹验证。 |
| Tongyi DeepResearch | 一个强 DeepResearch Agent 需要从 Agentic Mid-training 到 SFT、RL、环境、Heavy Mode 的端到端训练体系。 |
| RubricEM | 对没有标准答案的长报告任务，rubric 应贯穿 Agent 执行、Judge 反馈、阶段化 RL 信用分配和经验记忆。 |
| Quest | 用完全合成的 Rubric Tree 任务、结构化 Context State 和 MT/SFT/RL，可以训练覆盖事实检索、引用 grounding、报告综合的通用 DeepResearch Agent。 |

---

## 3. 核心问题对比

| 论文 | 主要解决的问题 | 关注重点 |
|---|---|---|
| MiroThinker | 如何通过更多、更有效的工具交互提升开源 DeepResearch 模型能力 | Interactive Scaling、训练流程、工具 RL |
| REDSearcher | 如何低成本构造可控复杂度的 Deep Search 训练数据 | 数据合成、任务复杂度、模拟环境 |
| MiroThinker-1.7 & H1 | 如何避免长程 Agent 交互低效、错误累积和缺少验证 | 阶段能力、Verifier、Heavy Mode |
| S1-DeepResearch | 如何构造覆盖真实研究任务多能力的数据和轨迹 | 图驱动任务、AgentLoop、轨迹验证 |
| Tongyi DeepResearch | 如何系统训练一个强 DeepResearch Agent | 全流程 recipe、环境、RL、Heavy Mode |
| RubricEM | 开放长报告没有可验证答案时，RL 如何分配信用并复用经验 | Rubric、Stagewise RL、Meta-policy、Memory |
| Quest | 如何用合成任务训练泛化到多类 DeepResearch benchmark 的 Agent | Rubric Tree、Context Management、合成数据、MT/SFT/RL |

---

## 4. 技术路线总对比

| 维度 | MiroThinker | REDSearcher | MiroThinker-1.7/H1 | S1 | Tongyi | RubricEM | Quest |
|---|---|---|---|---|---|---|---|
| 主要范式 | Interactive Scaling | Scalable Deep Search | Verification-centric Agent | Agentic Data Construction | End-to-end Agentic Training | Rubric-guided Meta-RL | Synthetic Rubric-tree Training |
| 数据来源 | MultiDocQA、轨迹合成、开源数据 | 图结构任务合成、WebHop、多模态 | Corpus/WebHop 双 pipeline | 图驱动任务、AgentLoop | 大规模合成 + 环境交互 | DR Tulu 类数据 + Teacher SFT | 完全合成 Quest-8K |
| 任务类型 | 深度搜索/研究问答 | 深度搜索、复杂检索 | 长程研究、专业推理 | 多场景 DeepResearch | 通用 DeepResearch | 开放式长报告 | objective + open-ended |
| 训练阶段 | SFT -> DPO -> GRPO | Mid-training -> SFT -> RL | Mid-training -> SFT -> Preference -> RL | SFT 为主 | Mid-training -> SFT -> RL -> Merge | Structured SFT -> SS-GRPO + Meta-RL | MT -> SFT -> GRPO RL |
| RL 重点 | 工具交互策略 | 搜索效率、模拟环境 | Agentic RL、验证 | 较少强调 RL | 环境交互 RL | Stagewise credit + reflection | Rubric-tree reward + fact-checking |
| 上下文管理 | Recency retention、truncation | Discard-all baseline | Sliding-window、result truncation | 轨迹模式和任务特化 | Context management paradigm | Stage scaffold + rubric memory | Context State / Context Condenser |
| Verifier/Rubric | 基础 reward 和过滤 | 多级 verifier | Local/Global Verifier | 多维 trajectory verification | Judge / Heavy Mode | Rubric 全流程核心 | Rubric Tree 核心 |
| 工程重点 | rollout 加速、工具环境 | 低成本合成和模拟 | 稳定多轮交互 | 数据系统 | 训练环境与异步 rollout | 异步 reflection 分支 | 双缓存、异步 evaluation |
| 对业务预测价值 | 高 | 高 | 很高 | 高 | 很高 | 很高 | 很高 |

---

## 5. 按模块拆解对比

### 5.1 数据构造

| 论文 | 数据构造方式 | 关键优点 | 潜在问题 |
|---|---|---|---|
| MiroThinker | MultiDocQA synthesis + Agentic trajectory synthesis + open-source data | 覆盖多文档、多工具轨迹 | 任务难度控制相对简单 |
| REDSearcher | Seed -> graph construction -> topology enrichment -> one-graph-multi-task -> tool-enforced query evolution | 能显式控制图复杂度和证据分散度 | 合成分布可能与真实业务有差距 |
| MiroThinker-1.7/H1 | Corpus-based + WebHop 双 pipeline，adaptive leaf obfuscation，difficulty filtering | 更重视任务可解性、难度过滤 | 流程复杂，依赖强 teacher/verifier |
| S1 | Seed entity pool -> subgraph -> constraint injection -> query/QA generation -> semantic evolution | 真实研究任务覆盖广，图结构清晰 | 更偏通用任务，业务标签需改造 |
| Tongyi | 大规模多风格问题合成、uncertainty injection、PhD-level research questions | 规模化强，覆盖多风格 | 细节闭源较多，不易完全复现 |
| RubricEM | 主要沿用 DR Tulu 训练数据，重点不在数据合成 | 专注算法机制 | 对数据构造贡献有限 |
| Quest | Trending keywords -> autonomous web exploration -> rubric tree -> evaluation protocol -> strict filtering | 数据、评价、reward 统一，可复现性强 | 强依赖闭源模型自动合成与评测 |

**整理结论：**

```text
如果要构造你自己的业务预测数据集，最值得结合的是：
REDSearcher 的复杂度/证据分散度控制
+ S1 的业务实体图/事件图思想
+ Quest 的 Rubric Tree / Evaluation Protocol
+ 业务预测自己的 Point-in-Time / Outcome 标签
```

---

### 5.2 任务复杂度控制

| 论文 | 复杂度刻画方式 | 对业务预测的启发 |
|---|---|---|
| REDSearcher | Topological Logical Complexity + Minimum Source Dispersion | 预测任务也应按推理链复杂度和证据分散度分层 |
| S1 | Information Flow、Feedback Dependency、Width、Depth | 可用于定义业务预测任务难度，如多实体、多因果链、多反馈关系 |
| Quest | Rubric Tree 的 Breadth × Depth，C1-C9 | 可用 Forecast Rubric Tree 的宽度/深度定义任务难度 |
| MiroThinker-1.7/H1 | Difficulty-adaptive filtering | 训练集应过滤太简单/太难任务，RL 用中等难度任务 |
| Tongyi | Uncertainty injection、PhD-level questions | 业务任务可注入不确定因素和复杂约束 |
| RubricEM | Stagewise rubrics 的区分度 | 任务难度可通过 stage score variance 衡量 |
| MiroThinker | 交互轮数和工具必要性 | 复杂任务应需要多轮工具调用，而不是模型记忆即可答 |

---

### 5.3 工具环境

| 论文 | 工具设计 | 重点 |
|---|---|---|
| MiroThinker | 信息检索、代码执行、文件管理 | 工具接口是 Agent 能力放大的基础 |
| REDSearcher | 搜索工具、模拟环境、多模态工具 | 强调 functionally equivalent simulation environment 降成本 |
| MiroThinker-1.7/H1 | 信息检索、代码执行、文件/数据传输、tool robustness | 强调工具调用鲁棒性和 benchmark contamination prevention |
| S1 | Search、file、multimodal、skill tools | 强调动态技能使用和任务特化工具 |
| Tongyi | Prior world / simulated / real-world environments | 不同训练阶段使用不同环境 |
| RubricEM | Google Search + Semantic Scholar | 工具不是重点，重点是围绕工具轨迹做 stagewise RL |
| Quest | Google Search、Visit、Python、Google Scholar + cache | 工程上最强调工具缓存、异步 evaluation、fallback |

**对业务预测 Agent 的工具启发：**

```text
必须建设工具层，而不是只用联网搜索。
业务预测至少需要：搜索、网页访问、内部数据库、时间序列、Python、baseline model、证据库、outcome lookup。
```

---

### 5.4 上下文与记忆管理

| 论文 | 方法 | 适合借鉴程度 |
|---|---|---|
| MiroThinker | Recency-based retention + result truncation | 简单实用，适合 MVP |
| REDSearcher | Discard-all baseline | 可作为对照，不建议主方案 |
| MiroThinker-1.7/H1 | Sliding-window observation retention + result truncation | 适合长任务和 Heavy Mode |
| S1 | 按任务场景 refinement，保留任务相关轨迹 | 适合多场景 Agent |
| Tongyi | Context management paradigm | 适合全流程系统设计 |
| RubricEM | Rubric bank，跨 episode 和同 episode reflection reuse | 适合经验复用和失败复盘 |
| Quest | Context State：trusted/untrusted/uncertain | 最适合业务预测证据管理 |

**推荐给业务预测 Agent 的组合：**

```text
短任务：MiroThinker 的 recency retention + result truncation
长任务：Quest 的 Context State
高价值任务：Quest Context State + RubricEM Reflection Bank + H1 Global Verifier
```

---

### 5.5 训练流程

| 论文 | 推荐/实际训练流程 | 特点 |
|---|---|---|
| MiroThinker | Agentic SFT -> DPO -> GRPO | 三阶段简洁清晰，适合基础路线 |
| REDSearcher | Agentic Mid-training -> SFT -> RL | 强调 cost-efficient mid-training 和模拟环境 |
| MiroThinker-1.7/H1 | Agentic Mid-training -> SFT -> Preference Optimization -> RL | 更完整，强调 atomic capability 和验证 |
| S1 | 主要 SFT | 重数据构造和轨迹验证，训练路线相对简单 |
| Tongyi | Agentic Mid-training -> SFT -> RL -> Model Merging | 产业级完整 recipe |
| RubricEM | Structured SFT -> SS-GRPO + Reflection Meta-Policy | 对 RL 阶段创新最大 |
| Quest | MT -> SFT -> GRPO RL | 结合 context MT、session-level SFT、异步 RL |

**对你的 Qwen3.6-35B 项目建议：**

```text
MVP：工具环境 + 历史数据 + Forecast Rubric Tree + Agentic SFT
V1：加入 Context State、DPO/IPO、Verifier、真实 outcome 回流
V2：加入 Quest 式 MT、RubricEM 式 Stagewise RL、Reflection Bank、Heavy Mode
```

---

### 5.6 SFT 数据与轨迹

| 论文 | SFT 的核心对象 | 重要技巧 |
|---|---|---|
| MiroThinker | Agentic trajectory | 工具调用轨迹、答案、开源数据混合 |
| REDSearcher | 真实环境轨迹和工具增强轨迹 | query curation 和 trajectory filtering |
| MiroThinker-1.7/H1 | stage-aware atomic capability 轨迹 | planning、reasoning、summarization 分能力训练 |
| S1 | AgentLoop trajectory | 统一轨迹格式，loss 只训模型动作 |
| Tongyi | ReAct mode + Context management mode | 两阶段 context length 训练 |
| RubricEM | stage-structured trajectory | tool output masking，XML scaffold |
| Quest | session-level trajectory | context condensation 边界切 session，适配超长任务 |

**最值得吸收的 SFT 设计：**

```text
SFT 不应只训练最终答案，应该混合：
任务解析、计划、工具调用、证据摘要、状态更新、审查、最终预测、反思。
```

---

### 5.7 DPO / Preference Optimization

| 论文 | DPO/Preference 结论 | 启发 |
|---|---|---|
| MiroThinker | Agentic DPO 有效，用 chosen/rejected 轨迹优化 | 适合工具轨迹质量偏好 |
| MiroThinker-1.7/H1 | Preference Optimization 是四阶段之一 | 可以增强每一步质量 |
| Quest | DPO for long-form report comparison 未成功 | 长报告偏好对噪声大，不能盲目做 DPO |
| RubricEM | 不重点用 DPO，而是 stagewise RL + reflection reward | 对开放任务可能比 DPO 更细粒度 |
| Tongyi | 重点在 SFT/RL，DPO 不是主线 | 工业系统更看重环境 RL |
| REDSearcher | 偏好优化不是重点 | 更重视合成和 RL |
| S1 | 主要 SFT | 可用验证结果构造偏好，但论文不是重点 |

**业务预测建议：**

```text
DPO 可以做，但不要用微弱分差构造 pair。
优先选择：
1. 高置信错误 vs 校准正确
2. 有证据链 vs 无证据链
3. 处理反证 vs 忽略反证
4. 无泄漏 vs 有泄漏
5. 真实 outcome 更好且解释合理 vs outcome 差且过度自信
```

---

### 5.8 RL 与 Reward

| 论文 | RL 方法 | Reward 来源 | 主要创新 |
|---|---|---|---|
| MiroThinker | GRPO | 答案/轨迹 reward、工具环境反馈 | Agentic RL 和 streaming rollout |
| REDSearcher | GRPO | verifiable reward + 搜索效率 | 模拟环境、RL query curation |
| MiroThinker-1.7/H1 | Agentic RL | verifiers / outcome / stage 能力 | entropy control、priority scheduling |
| S1 | 不以 RL 为重点 | - | 强调 SFT 数据质量 |
| Tongyi | GRPO 改造 | real-world + simulated reward | 异步 rollout、automatic data curation |
| RubricEM | SS-GRPO | stagewise rubric judge + reflection judge | 阶段化信用分配和 meta-policy |
| Quest | GRPO | rubric-tree reward + fact-checking reward | rubric tree 评价协议、异步 evaluation |

**业务预测 RL 推荐：**

```text
不要只用 accuracy。
应使用：
Brier / Log Loss / MAE / NDCG
+ evidence reward
+ calibration reward
+ leakage penalty
+ cost penalty
+ stagewise process reward
```

---

### 5.9 Verifier / Judge / Rubric

| 论文 | 验证机制 | 特点 |
|---|---|---|
| REDSearcher | 多级 verifier pipeline | 验证任务可解性、证据充分性、答案可靠性 |
| MiroThinker-1.7/H1 | Local Verifier + Global Verifier | 每步审查 + 最终审查，适合 Heavy Mode |
| S1 | Multi-dimensional trajectory verification | 按任务类型验证轨迹和交付物 |
| Tongyi | Judge + Heavy Mode synthesis | 多代理研究和汇总审查 |
| RubricEM | Stagewise evolving rubric judge | 每个阶段生成和维护可区分 rubric |
| Quest | Rubric Tree + Python / LLM evaluation protocol | 数据、评测、RL 统一 |
| MiroThinker | reward / judge / 轨迹过滤 | 相对基础，但训练闭环完整 |

**业务预测最佳组合：**

```text
Quest 的 Rubric Tree：定义任务评价标准
H1 的 Local/Global Verifier：检查过程和最终结论
RubricEM 的 stagewise rubric judge：给 RL 阶段分配过程信用
业务 outcome evaluator：用真实结果判断最终预测
```

---

## 6. 按项目阶段推荐参考哪篇

| 项目阶段 | 最该参考的论文 | 原因 |
|---|---|---|
| 业务任务定义 | Quest、REDSearcher、S1 | 任务结构、复杂度、rubric tree、图驱动定义 |
| 数据集构造 | Quest、REDSearcher、S1、Tongyi | 合成任务、证据分散、图构造、大规模 pipeline |
| 标签和评价协议 | Quest、RubricEM | rubric tree、stagewise judge、开放任务评价 |
| 工具环境 | MiroThinker、Tongyi、Quest | 工具接口、环境交互、缓存、异步 |
| 上下文管理 | Quest、MiroThinker-1.7/H1、Tongyi | Context State、sliding window、truncation |
| SFT | MiroThinker、S1、Tongyi、RubricEM、Quest | 轨迹格式、stage scaffold、session-level training |
| DPO | MiroThinker、MiroThinker-1.7/H1、Quest | 有效案例 + 失败教训 |
| RL | Tongyi、MiroThinker、RubricEM、Quest、REDSearcher | GRPO、SS-GRPO、异步 rollout、reward 设计 |
| Verifier | MiroThinker-1.7/H1、RubricEM、Quest、S1 | Local/Global、stagewise、rubric tree、多维验证 |
| Heavy Mode | Tongyi、MiroThinker-1.7/H1 | 并行研究、综合审查、高价值任务 |
| 部署与监控 | Quest、Tongyi | 缓存、fallback、异步工程、环境稳定性 |

---

## 7. 对业务预测 Agent 的直接映射

你的目标是：

```text
针对业务问题，用 DeepResearch 思想检索和调用大量工具获取信息，
基于信息进行结果预测，最后判断预测是否和真实结果一致。
```

这和论文的对应关系如下。

### 7.1 业务预测任务构造

| 需求 | 可借鉴论文 | 落地动作 |
|---|---|---|
| 构造真实预测问题 | S1、REDSearcher、Quest | 从业务实体-事件-指标图生成历史预测任务 |
| 控制难度 | REDSearcher、Quest、S1 | 用证据分散度、rubric tree 宽深、推理链长度分层 |
| 避免简单记忆 | REDSearcher、S1 | 工具必要性和 parametric knowledge filtering |
| 多类型预测 | Quest | 用 rubric tree 表示 binary/regression/ranking/open-ended report |
| 生成评价协议 | Quest | 每个预测任务绑定 outcome 计算规则和评分函数 |

### 7.2 预测 Agent 工作流

| 需求 | 可借鉴论文 | 落地动作 |
|---|---|---|
| 规划和工具调用 | MiroThinker、RubricEM | Forecast Plan -> Evidence Research -> Forecast Review -> Prediction |
| 多轮搜索和查库 | MiroThinker、Tongyi | ReAct/AgentLoop 工具交互 |
| 状态管理 | Quest | trusted/untrusted/uncertain Context State |
| 长任务不丢证据 | MiroThinker-1.7/H1、Quest | sliding window + result truncation + context condenser |
| 高价值任务增强 | Tongyi、H1 | Heavy Mode：多代理并行研究 + 全局综合 |

### 7.3 训练路线

| 需求 | 可借鉴论文 | 落地动作 |
|---|---|---|
| 冷启动 | MiroThinker、Tongyi、Quest | Agentic SFT |
| 工具格式学习 | S1、RubricEM | tool output masking、轨迹 loss masking |
| 上下文能力 | Quest | Context summarization / information extraction MT |
| 偏好优化 | MiroThinker、Quest | 谨慎构造高质量 pair，避免微弱分差 DPO |
| RL | RubricEM、Quest、Tongyi | outcome reward + stagewise process reward + evidence reward |
| 经验复用 | RubricEM | outcome-aware reflection bank |

### 7.4 评测与上线

| 需求 | 可借鉴论文 | 落地动作 |
|---|---|---|
| 多维评测 | Quest、S1、RubricEM | outcome + evidence + calibration + process + cost |
| Verifier | H1、RubricEM | Local/Global/Stagewise Verifier |
| 真实结果回流 | 论文较少直接覆盖，需要业务新增 | Outcome Tracker |
| 防未来泄漏 | REDSearcher、H1 的 contamination 思想 + 业务新增 | Point-in-Time evidence snapshot |
| 成本控制 | MiroThinker、Quest、Tongyi | 工具预算、缓存、异步 rollout、cost reward |

---

## 8. 最推荐融合方案

如果把 7 篇论文融合成一个适合业务预测 Agent 的方案，可以这样设计。

### 8.1 数据层

```text
S1 的业务实体-事件-指标图
+ REDSearcher 的复杂度/证据分散度控制
+ Quest 的 Forecast Rubric Tree
+ 业务自己的 Point-in-Time 和 outcome label
```

每个样本应该包含：

```json
{
  "task_id": "...",
  "forecast_time": "...",
  "deadline": "...",
  "question": "...",
  "forecast_rubric_tree": {},
  "available_tools": [],
  "evidence_snapshot": [],
  "outcome_definition": {},
  "outcome": {},
  "scoring_rule": "brier|log_loss|mae|ndcg"
}
```

### 8.2 Agent 层

```text
RubricEM 的阶段化 scaffold
+ MiroThinker 的 ReAct 交互
+ Quest 的 Context State
+ H1 的 Local/Global Verifier
+ Tongyi 的 Heavy Mode
```

推荐阶段：

```text
Forecast Plan
-> Evidence Research
-> Evidence State Update
-> Forecast Review
-> Prediction Output
-> Outcome Reflection
```

### 8.3 训练层

```text
Quest 的 MT：Context Summarization + Information Extraction
MiroThinker / S1 的 Agentic SFT：完整轨迹训练
MiroThinker 的 DPO：只对高质量偏好对做
RubricEM 的 SS-GRPO：阶段化过程 reward
业务新增：真实 outcome reward + 校准 reward
```

推荐训练路线：

```text
Stage 0: Prompt + 工具协议冻结
Stage 1: Forecast Rubric Tree 数据构造
Stage 2: Agentic MT，可选但推荐 V1 做
Stage 3: Stage-aware SFT
Stage 4: DPO/IPO/ORPO，谨慎小规模
Stage 5: Historical Replay RLVR / GRPO
Stage 6: SS-GRPO + Outcome-aware Reflection Bank
Stage 7: 影子上线和持续回流
```

### 8.4 Reward 层

```text
Quest 的 rubric-tree reward
+ Quest 的 fact-checking reward
+ RubricEM 的 stagewise reward
+ 业务预测的 proper scoring reward
```

推荐公式：

```text
R_total = 0.45 * outcome_score
        + 0.15 * calibration_score
        + 0.15 * evidence_grounding_score
        + 0.10 * process_rubric_score
        + 0.05 * verifier_score
        + 0.05 * format_score
        - 0.10 * cost_penalty
        - 1.00 * leakage_penalty
```

其中：

- `outcome_score`：Brier、Log Loss、MAE、NDCG 等；
- `calibration_score`：概率桶校准；
- `evidence_grounding_score`：证据和结论是否匹配；
- `process_rubric_score`：计划、检索、审查是否合格；
- `verifier_score`：Local/Global Verifier 通过情况；
- `format_score`：JSON/schema/citation 是否合规；
- `cost_penalty`：工具调用和 token 成本；
- `leakage_penalty`：未来泄漏强惩罚。

---

## 9. 哪些方法应优先做，哪些以后做

### 9.1 MVP 必做

| 模块 | 来源论文 | 原因 |
|---|---|---|
| Point-in-Time 数据 | 业务预测新增 | 预测任务根基 |
| Forecast Rubric Tree | Quest | 统一任务、评测、训练标准 |
| 工具环境 | MiroThinker、Tongyi | DeepResearch Agent 核心能力来源 |
| Agentic SFT | MiroThinker、S1、Tongyi | 冷启动必须做 |
| Evidence Store | Quest、H1 | 可追溯和防泄漏 |
| 基础 Verifier | H1、RubricEM | 防止格式错、证据错、未来泄漏 |
| 离线回测 | 业务预测新增 | 证明真实预测能力 |

### 9.2 V1 推荐做

| 模块 | 来源论文 | 原因 |
|---|---|---|
| Context State | Quest | 长程证据管理 |
| Context Summarization MT | Quest | 提升上下文压缩能力 |
| Preference Optimization | MiroThinker、H1 | 提升工具/证据/输出偏好 |
| Reflection Bank | RubricEM | 复用历史预测错误经验 |
| Heavy Mode 原型 | Tongyi、H1 | 高价值复杂预测 |
| Historical Replay RL | Tongyi、REDSearcher、Quest | 可控成本下做 RL |

### 9.3 V2 再做

| 模块 | 来源论文 | 原因 |
|---|---|---|
| Stage-Structured GRPO | RubricEM | 强但复杂，要求 judge 稳定 |
| Evolving Rubric Judge | RubricEM | 适合大规模 RL 后期 |
| 大规模自动合成 | Quest、REDSearcher、S1 | 需要数据工程和质检成熟 |
| 全参 MT/SFT | Tongyi、Quest | 算力成本高，先用 LoRA/QLoRA 验证 |
| 多代理 Heavy Mode | Tongyi、H1 | 成本高，先限高价值任务 |
| 异步 rollout/evaluation 平台 | Quest、RubricEM、Tongyi | RL 规模化后必需 |

---

## 10. 关键差异：Rubric Tree、Rubric、Verifier、Reward

这几篇论文中有几个概念容易混淆。

| 概念 | 主要论文 | 是什么 | 用在哪里 |
|---|---|---|---|
| Rubric Tree | Quest | 层级评价树，叶子节点可验证，root 聚合总分 | 数据合成、评价协议、RL reward |
| Self-generated Rubric | RubricEM | Agent 在 Plan 阶段自己生成的任务标准 | 指导计划、搜索、审查、回答 |
| Stagewise Rubric | RubricEM | Judge 为每个阶段生成/维护的评价标准 | SS-GRPO 阶段奖励 |
| Verifier | H1、S1、REDSearcher | 检查任务、证据、过程、答案是否可靠 | 过滤、审查、Heavy Mode |
| Reward | MiroThinker、Tongyi、Quest、RubricEM | 用于 RL 的优化信号 | 训练策略 |
| Reflection | RubricEM、Quest retry | 从错误或历史尝试中总结经验 | retry、memory、meta-policy |

对业务预测 Agent 来说，建议这样分工：

```text
Forecast Rubric Tree：任务定义和评分结构
Self-generated Rubric：Agent 每次预测前生成的研究计划标准
Verifier：规则/模型检查过程是否合规
Reward：真实结果 + 过程质量 + 证据质量
Reflection：真实 outcome 出现后总结经验
```

---

## 11. 每篇论文最值得摘出来的实践点

### 11.1 MiroThinker

最值得借鉴：

- Interactive Scaling；
- 不只追求长上下文，还要追求有效工具交互；
- Agentic SFT -> DPO -> GRPO 的基础训练路线；
- 工具环境和 rollout 加速；
- 成本纳入训练目标。

对业务预测的直接应用：

```text
训练模型学会主动查内部数据、外部信息、运行代码和调用 baseline，而不是凭记忆预测。
```

### 11.2 REDSearcher

最值得借鉴：

- 任务复杂度不是 hop 数，而是图拓扑复杂度；
- 证据分散度是 Deep Search 难度核心；
- 强制工具使用的 query evolution；
- 模拟环境降低 RL 成本；
- RL 任务池要筛选中等难度样本。

对业务预测的直接应用：

```text
构造历史预测样本时，要控制信息来源分散度，比如内部销售表 + 库存 + 新闻 + 竞品 + 行业指标。
```

### 11.3 MiroThinker-1.7 & H1

最值得借鉴：

- 长程交互不是越多越好；
- 每一步 atomic capability 要强；
- Step Loop 和 Episode Restart；
- Sliding-window observation retention；
- Local Verifier 和 Global Verifier；
- Heavy Mode 的验证中心设计。

对业务预测的直接应用：

```text
高价值预测任务应加入 Local/Global Verifier，防止查错、算错、证据不足、概率过度自信。
```

### 11.4 S1-DeepResearch

最值得借鉴：

- Graph-grounded task formulation；
- Complexity evolution；
- AgentLoop execution；
- Trajectory refinement；
- Multi-dimensional trajectory verification；
- Skill-aware rollout。

对业务预测的直接应用：

```text
用业务实体-指标-事件-证据图来生成预测问题，而不是人工随便写问题。
```

### 11.5 Tongyi DeepResearch

最值得借鉴：

- Agentic Mid-training、SFT、RL 的整体范式；
- Prior World / Simulated / Real-world Environment；
- SFT 和 RL 分工；
- Heavy Mode：parallel research + integrative synthesis；
- 异步 rollout 和 automatic data curation；
- Model merging。

对业务预测的直接应用：

```text
业务预测 Agent 应区分历史回放环境、模拟环境、真实影子环境，不能直接在线试错。
```

### 11.6 RubricEM

最值得借鉴：

- Rubric 不是只用于最终打分，而是贯穿执行、评分、记忆；
- Plan/Research/Review/Answer scaffold；
- Stage-Structured GRPO；
- Evolving rubric buffer；
- Reflection Meta-Policy；
- Rubric Bank；
- Cross-episode transfer 和 within-episode refinement。

对业务预测的直接应用：

```text
把历史预测错误总结成 outcome-aware reflection bank，让模型在相似预测任务中复用经验。
```

### 11.7 Quest

最值得借鉴：

- Rubric Tree 合成任务；
- Objective + Open-ended 统一训练；
- Context State：trusted/untrusted/uncertain；
- Context Summarization 和 Relevant Information Extraction MT；
- Session-level training；
- Rubric-tree reward + fact-checking reward；
- 搜索/访问双缓存；
- 异步 evaluation。

对业务预测的直接应用：

```text
为每个预测任务构造 Forecast Rubric Tree，并用 Context State 管理已验证、被否定、待验证的证据。
```

---

## 12. 重要共识

这些论文虽然方法不同，但形成了几个共识。

### 12.1 数据和环境比模型更重要

几乎所有论文都说明：

```text
Agent 能力不是只靠 base model，而是来自数据、工具、环境和训练闭环。
```

对业务预测来说，Qwen3.6-35B 是推理核心，但预测质量更依赖：

- 历史数据；
- 业务标签；
- 工具覆盖；
- 防泄漏；
- 结果回流；
- 评测闭环。

### 12.2 SFT 是冷启动，RL 是策略优化

SFT 让模型学会格式、工具、轨迹。

RL 让模型学会：

- 什么时候搜索；
- 搜什么；
- 什么时候停止；
- 如何权衡证据；
- 如何降低成本；
- 如何提升最终结果。

不能跳过 SFT 直接大规模 RL。

### 12.3 Reward 必须和任务结构一致

如果任务是长报告，只用 exact match 不够。

如果任务是业务预测，只用 LLM Judge 不够。

必须让 reward 反映真实目标：

```text
业务预测 = 真实 outcome + 概率校准 + 证据质量 + 工具成本 + 防泄漏
```

### 12.4 上下文管理不是简单截断

长程研究 Agent 需要记住：

- 已经查过什么；
- 哪些事实可信；
- 哪些事实被否定；
- 哪些还要继续查；
- 当前计划是否需要调整。

Quest 的 Context State 是最适合业务预测的方案。

### 12.5 复杂任务需要 Verifier

Agent 长程交互越多，错误累积风险越大。

因此需要：

- Local Verifier：每步检查；
- Evidence Verifier：证据检查；
- Leakage Verifier：未来泄漏检查；
- Global Verifier：最终预测前审查；
- Calibration Verifier：概率校准检查。

---

## 13. 重要分歧和取舍

### 13.1 是否要做大规模 Mid-training

| 观点 | 代表论文 | 说明 |
|---|---|---|
| 需要 | Tongyi、Quest、REDSearcher、H1 | 可提升 agentic 基础能力和上下文能力 |
| 不一定 | MiroThinker、S1 | 高质量 SFT 也能起步 |

业务预测建议：

```text
MVP 不一定做全参 MT；
可以先 SFT；
V1 再做轻量 Agentic MT，例如 Context State 更新和信息抽取。
```

### 13.2 DPO 是否有效

| 观点 | 代表论文 | 说明 |
|---|---|---|
| 有效 | MiroThinker、H1 | 对工具轨迹和 agentic 行为偏好有帮助 |
| 不稳定 | Quest | 长报告 pairwise DPO 噪声大、过拟合 |

业务预测建议：

```text
DPO 要做强偏好 pair，不做微弱分差 pair。
```

### 13.3 RL 用最终 reward 还是阶段 reward

| 方案 | 代表论文 | 优点 | 缺点 |
|---|---|---|---|
| 最终 reward | MiroThinker、Quest、Tongyi | 简单，易实现 | 信用分配粗 |
| 阶段 reward | RubricEM、H1 思想 | 信号密，能优化过程 | judge 复杂，噪声更多 |

业务预测建议：

```text
先最终 outcome reward + 规则过程 reward；
后续再引入 RubricEM 式 stagewise reward。
```

### 13.4 上下文保留还是压缩

| 方法 | 代表论文 | 优缺点 |
|---|---|---|
| 保留最近内容 | MiroThinker | 简单，但可能丢关键早期证据 |
| Discard-all | REDSearcher 某些设置 | 低成本，但会重复搜索 |
| Sliding-window | H1 | 工程可控 |
| Context State | Quest | 最适合长程研究，但需要 condenser 质量 |
| Reflection Bank | RubricEM | 跨任务经验复用，但有错误传播风险 |

---

## 14. 对你的项目的最终建议

### 14.1 最适合你的整体组合

你的业务预测 Agent 最好采用以下组合：

```text
数据构造：S1 + REDSearcher + Quest
Agent 工作流：MiroThinker + RubricEM + H1
上下文管理：Quest + H1
训练路线：Tongyi + MiroThinker + Quest
RL 设计：RubricEM + Quest + 业务真实 outcome
Verifier：H1 + RubricEM + 业务防泄漏
上线闭环：Quest 异步工程 + 业务 Outcome Tracker
```

### 14.2 推荐优先级

最优先做：

1. Forecast Rubric Tree；
2. Point-in-Time evidence snapshot；
3. Agentic SFT；
4. Context State；
5. Outcome 回流和 Brier/Log Loss 评测；
6. Local/Global Verifier；
7. Reflection Bank；
8. Historical Replay RL；
9. Stagewise GRPO。

### 14.3 不建议一开始做

- 从零预训练；
- 大规模 full-parameter MT；
- 没有稳定 reward 的在线 RL；
- 只靠 LLM Judge 判断预测对错；
- 用当前互联网直接回测历史任务；
- 无防泄漏的数据合成；
- 未经过质检的大规模合成轨迹；
- 对长报告微弱分差做 DPO。

---

## 15. 推荐阅读顺序

如果目的是快速整理方法脉络，建议这样读：

```text
1. Tongyi DeepResearch：先建立完整训练管线大图
2. MiroThinker：理解 Interactive Scaling 和 SFT/DPO/RL 基础路线
3. REDSearcher：理解复杂任务合成和证据分散度
4. S1-DeepResearch：理解图驱动任务构造和多维轨迹验证
5. Quest：理解 Rubric Tree、Context State、合成数据和异步工程
6. MiroThinker-1.7 & H1：理解 Verifier、Heavy Mode 和阶段质量控制
7. RubricEM：理解 stagewise RL、rubric memory 和 meta-policy
```

如果目的是服务业务预测 Agent，建议这样读：

```text
1. Quest：先学 Forecast Rubric Tree 和 Context State
2. REDSearcher / S1：学业务任务和证据图怎么构造
3. MiroThinker / Tongyi：学完整训练路线
4. H1：学 Verifier 和 Heavy Mode
5. RubricEM：学阶段化 RL 和经验复用
```

---

## 16. 一张最终对比表

| 论文 | 最核心资产 | 最适合解决 | 对业务预测价值 | 实施难度 |
|---|---|---|---|---|
| MiroThinker | Interactive Scaling + SFT/DPO/GRPO | 让模型学会工具交互 | 高 | 中 |
| REDSearcher | 复杂任务合成 + 证据分散度 | 构造难度可控的数据 | 高 | 中高 |
| MiroThinker-1.7/H1 | Verifier + Heavy Mode | 提升长程任务可靠性 | 很高 | 高 |
| S1 | 图驱动任务 + 轨迹验证 | 构造真实研究任务 | 高 | 中高 |
| Tongyi | 完整工业级训练范式 | 端到端训练 DeepResearch Agent | 很高 | 高 |
| RubricEM | Stagewise GRPO + Reflection Bank | 开放任务 RL 信用分配和经验复用 | 很高 | 很高 |
| Quest | Rubric Tree + Context State + MT/SFT/RL | 合成可评价任务和长程上下文管理 | 很高 | 高 |

---

## 17. 最终总结

这 7 篇论文可以被看作 DeepResearch Agent 训练体系的 7 个拼图：

```text
MiroThinker：证明交互扩展重要
REDSearcher：告诉你复杂搜索任务怎么低成本合成
S1：告诉你真实研究任务怎么图驱动构造和验证
Tongyi：给出工业级端到端训练大框架
H1：告诉你长程交互必须验证和做 Heavy Mode
Quest：告诉你任务、评价、上下文和训练如何开放复现
RubricEM：告诉你开放长报告 RL 如何做阶段信用分配和经验复用
```

对你的业务预测 Agent，最重要的不是照搬某一篇，而是融合成一个新的范式：

```text
业务预测 DeepResearch Agent
= Point-in-Time 预测任务
+ Forecast Rubric Tree
+ 多源工具检索
+ Context State 证据管理
+ Stage-aware Agent 工作流
+ Agentic SFT
+ 谨慎 DPO
+ Outcome-aware RL
+ Stagewise Verifier
+ Reflection Bank
+ 真实结果回流闭环
```

最终判断一个方案是否正确，不看论文 benchmark 分数，而看：

```text
历史回测是否无泄漏；
概率是否校准；
真实 outcome 上是否优于业务 baseline；
证据链是否可审计；
成本是否可控；
上线后是否能持续从错误中学习。
```
