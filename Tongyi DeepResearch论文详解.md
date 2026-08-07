# Tongyi DeepResearch 技术报告详解

> 来源文件：`Tongyi DeepResearch Technical Report.pdf`  
> 论文题目：**Tongyi DeepResearch Technical Report**  
> 作者：Tongyi DeepResearch Team，Tongyi Lab，Alibaba Group  
> 版本信息：arXiv:2510.24701v2，2025-11-04；PDF 首页日期 2025-11-05  
> 论文中给出的资源：项目博客、GitHub、HuggingFace、ModelScope  
> 说明：本文档基于当前目录中的本地 PDF 阅读整理，未联网核验外部链接、模型、代码或评测结果，也未独立复现实验。

---

## 1. 一句话总结

Tongyi DeepResearch 是一份开源 deep research agent 技术报告，提出一个从 Qwen3-30B-A3B-Base 出发的端到端 agentic training pipeline：先用两阶段 **Agentic Continual Pre-training（Agentic CPT）** 建立 agentic prior，再用自动合成高难 QA 和轨迹做 **SFT cold start**，最后通过 **agentic RL / RLVR / GRPO** 在模拟和真实工具环境中优化长程搜索、规划、推理与综合能力。模型总参数 30.5B，每 token 激活 3.3B，在多个 deep research benchmark 上达到强开源结果。

---

## 2. 论文的整体定位

这篇报告是当前目录几篇论文中时间较早的一篇，很多后续工作都与它形成呼应：

| 论文 | 与 Tongyi 的关系 |
|---|---|
| MiroThinker v1.0 | 同样强调开源 research agent 和 interactive scaling，但 Miro 使用 8B/30B/72B 多尺寸 |
| REDSearcher | 在 Tongyi 的 agentic mid/post-training 基础上，更深入讨论复杂任务合成和低成本模拟环境 |
| MiroThinker-1.7 & H1 | 进一步提出 effective interaction scaling 和 verification-centric heavy-duty reasoning |
| S1-DeepResearch | 批判 search-centric 数据，强调 beyond search 的报告/文件/技能/指令能力 |

Tongyi DeepResearch 的核心价值在于：它给出了一个较完整的开源 deep research agent 训练范式，包含设计原则、数据合成、环境设计、SFT、RL、heavy mode、分析实验和开放复现资源。

---

## 3. 论文要解决的问题

### 3.1 Deep Research Agent 为什么重要？

论文将 deep research 定义为一种 agentic capability：模型能自主在互联网上进行多步推理和信息搜索，完成复杂研究任务。

相比普通问答，deep research 需要：

- 规划研究路线；
- 多轮检索；
- 访问网页；
- 阅读和抽取证据；
- 使用 Python、Scholar、File Parser 等工具；
- 跨来源验证；
- 汇总成研究报告。

论文指出，商业 deep research 系统已经能在几十分钟内完成原本需要人类数小时的研究任务，但大多闭源，中间过程不可见，社区缺少系统方法和公开模型。

### 3.2 通用基础模型为什么不够？

通用 LLM 通常通过：

```text
pre-training on web text -> instruction tuning / alignment
```

获得语言和通用问答能力，但它们缺少 agentic inductive bias：

- 不习惯多轮工具交互；
- 不会长期维护研究状态；
- 不会系统规划搜索路径；
- 不会在环境反馈中修正策略；
- 不会把大量 observation 压缩成持续更新的 research state。

如果直接对这类模型做 agentic RL，模型同时要学格式、工具、规划和任务目标，容易产生优化冲突。

### 3.3 人工标注不可扩展

Deep research 数据难以人工构造：

- 研究级问题本身难写；
- 答案需要跨来源查证；
- agentic trajectory 很长；
- 工具响应不可控；
- 人工标注成本极高。

因此 Tongyi 的路线是：**以自动合成数据为核心，配合不同训练阶段定制环境。**

---

## 4. 核心贡献

论文列出三大关键进展。

### 4.1 End-to-end agentic training paradigm

统一：

```text
agentic mid-training + agentic post-training
```

Mid-training 培养 agentic biases；post-training 用 SFT + RL 解锁深度研究能力。

### 4.2 自动化高质量数据合成 pipeline

不用昂贵人工标注，自动生成：

- 研究级问题；
- agent 行为数据；
- 高质量轨迹；
- 可验证奖励数据。

不同训练阶段使用不同数据合成策略。

### 4.3 Stage-specific customized environments

论文不把环境视为固定外部现实，而是根据训练阶段设计不同环境：

- Prior World Environment；
- Simulated Environment；
- Real-world Environment。

不同环境在成本、稳定性、真实性之间取舍。

### 4.4 强开源模型结果

Tongyi DeepResearch 基于 Qwen3-30B-A3B-Base：

- 总参数 30.5B；
- 每 token 激活 3.3B；
- 在 HLE、BrowseComp、BrowseComp-ZH、WebWalkerQA、GAIA、xBench-DeepSearch、FRAMES、xBench-DeepSearch-2510 等任务上取得强结果。

---

## 5. 设计原则一：Agent Training Pipeline

论文认为 agent training 比普通 LLM training 更复杂，需要分阶段。

### 5.1 为什么需要 mid-training？

通用模型缺少 agentic prior。如果跳过 mid-training，直接做 agentic post-training，会出现：

- SFT 同时教格式和工具行为，负担过重；
- RL 需要探索太多基础工具模式，样本效率低；
- 模型不知道何时搜索、如何规划；
- agentic 能力和 alignment 可能互相冲突。

Mid-training 的作用是：

- 从 pre-training 过渡到 agentic post-training；
- 让模型先学会基本 agent 行为；
- 为后续 SFT/RL 提供强初始化。

### 5.2 SFT 与 RL 的分工

#### SFT cold start

SFT 让模型模仿 curated demonstrations，建立稳定行为基线：

- 输出格式；
- 工具调用 schema；
- 基本 ReAct 循环；
- 问题拆解；
- 状态总结。

但 SFT 的问题是 imitation，缺少探索。

#### RL

RL 通过环境交互和 reward 信号：

- 主动探索更优策略；
- 内化目标导向规划；
- 提升执行能力；
- 优先强化高奖励行为。

论文强调：SFT 教模型“会做”，RL 推动模型“做得更好”。

---

## 6. 设计原则二：Synthetic Data Centric Scaling

论文非常强调合成数据。

### 6.1 为什么合成数据适合 deep research？

论文列出合成数据优势：

1. **研究级问题容易规模化生成**  
   相比人工标注，LLM 能高效生成复杂 QA。

2. **模式和多样性更易扩展**  
   LLM 能理解 hard problems 的结构，合成多种问题模式。

3. **可定向增强 meta-capabilities**  
   可针对 planning、information synthesis、memory management 等能力专门合成数据。

4. **更容易验证**  
   构造数据时已有答案和结构，验证比人工找答案容易。

5. **形成 data flywheel**  
   一轮训练后的 agent 能生成更强数据，数据再推动下一轮模型提升。

### 6.2 三步合成框架

所有阶段的 agentic 数据都围绕三步：

```text
1. Synthesizing research-level questions
2. Generating agentic behavior data
3. Utilizing agentic data in training pipeline
```

这奠定了后面 mid-training 与 post-training 数据合成的统一思路。

---

## 7. 设计原则三：Learning Through Environmental Interaction

论文指出，环境交互对 agent intelligence emergence 很关键，但完全依赖真实环境有问题：

- 非平稳：网页、搜索排序、API 返回会变；
- 成本高：每次 API 调用都要钱；
- 不稳定：延迟、失败、访问限制；
- 难诊断：模型失败可能是策略错，也可能是环境错。

因此论文将环境划分为三类。

### 7.1 Prior World Environment

特点：

- 提供任务元素、工具定义、状态定义；
- 不返回真实环境响应；
- 基于模型预训练知识挖掘交互轨迹；
- 稳定、零成本、无限扩展；
- 但缺少真实反馈。

适合：早期 mid-training，大规模合成规划/推理/决策数据。

### 7.2 Simulated Environment

特点：

- 本地构造真实交互的可控副本；
- 稳定、快速、低成本；
- 支持快速迭代和因果分析；
- 但覆盖有限，有 sim-to-real gap。

适合：算法验证、RL 早期实验、低成本 ablation。

### 7.3 Real-world Environment

特点：

- 最真实的数据分布和反馈信号；
- 是最终能力验证场；
- 但成本高、非平稳、风险大。

适合：最终 post-training、真实评测、策略迁移验证。

### 7.4 环境使用策略

论文策略：

- Mid-training：主要用 Prior World + Simulated Environment，大规模低成本 bootstrapping；
- Post-training：先在 Simulated Environment 验证算法，再部署到 Real-world Environment 最终训练。

---

## 8. Rollout 形式化

Tongyi DeepResearch 的 agent 每一步包含三个基本组件。

### 8.1 Thought `τ_t`

内部认知过程，包括：

- 分析当前上下文；
- 回忆/总结信息；
- 规划下一步；
- 自我反思；
- 调整策略。

### 8.2 Action `a_t`

外部操作，包括：

- Search；
- Visit；
- Python Interpreter；
- Google Scholar；
- File Parser；
- 最终回答。

中间 `a_t` 是工具调用，最后 `a_T` 是给用户的报告/答案。

### 8.3 Observation `o_t`

工具执行后的反馈，用来更新内部状态并决定下一步。

---

## 9. ReAct 与 Context Management

### 9.1 ReAct 轨迹

Tongyi 使用 vanilla ReAct：

```math
H_T=(\tau_0,a_0,o_0,...,\tau_i,a_i,o_i,...,\tau_T,a_T)
```

每步策略：

```math
\tau_t,a_t \sim \pi(\cdot | H_{t-1})
```

论文选择 ReAct 的原因是简单、通用、符合 Bitter Lesson：随着模型能力扩展，简单可扩展方法往往比复杂手工 workflow 更稳健。

### 9.2 Context Management Paradigm

长程任务会受上下文窗口限制。Tongyi 使用基于 **Markovian state reconstruction** 的 context management。

模型每步不再读取完整历史，而读取：

- 原问题 `q`；
- evolving report `S_t`，作为压缩记忆；
- 上一步 action `a_t`；
- 上一步 observation `o_t`。

形式化：

```math
S_t,\tau_{t+1},a_{t+1}\sim \pi(\cdot | S_{t-1},a_t,o_t)
```

### 9.3 与 MiroThinker/REDSearcher 的上下文管理对比

| 系统 | 策略 | 特点 |
|---|---|---|
| Tongyi | evolving report / Markov state reconstruction | 用摘要状态持续承载研究进展 |
| MiroThinker v1.0/1.7 | 保留 thought/action，最近 K 个 observation | 简单 recency retention，旧 observation 省略 |
| REDSearcher | Discard-all | 上下文接近阈值时直接重置历史 |

Tongyi 的策略更像人类研究者持续写研究笔记：每一步把重要信息整合进 `S_t`，避免无限堆叠原始网页。

---

## 10. Overall Training Recipe

模型初始化：

```text
Qwen3-30B-A3B-Base
```

训练流程：

```text
Pre-training base
-> Agentic CPT Stage 1, 32K
-> Agentic CPT Stage 2, 128K
-> Agentic SFT
-> Agentic RL
-> Model Merging
```

其中 Agentic CPT 是 mid-training，SFT/RL 是 post-training。

---

## 11. Agentic Mid-training：两阶段 Agentic CPT

### 11.1 训练配置

Agentic Continual Pre-training 使用标准 next-token prediction loss。

两阶段：

| 阶段 | Context | 目标 |
|---|---:|---|
| Stage 1 | 32K | 训练基础 agentic 行为和中短上下文能力 |
| Stage 2 | 128K | 引入 64K-128K 长序列 agentic behavior data，增强长程推理和行动一致性 |

训练时混入少量 general pre-training data，避免模型在学 agent 能力时遗忘通用语言能力。

### 11.2 大规模 Agent Behavior Data Synthesis

Agentic CPT 合成 agent 工作流全生命周期数据，包括四类关键步骤：

1. Question Synthesis；
2. Planning Action；
3. Reasoning Action；
4. Decision-Making Action。

#### Large-scale Multi-style Question Synthesis

构造 entity-anchored open-world memory：

- web-crawled data；
- agent interaction trajectories；
- structured entity representations；
- entity-associated knowledge。

基于实体和知识生成多样问题，例如：

- multi-hop reasoning；
- numerical computation；
- 不同 agent 行为模式要求。

#### Planning Action

Planning 包括：

- problem decomposition；
- first-step action prediction。

论文认为 planning accuracy 与 agent 是否成功强相关。它用开源模型分析、拆解问题并预测第一步动作，再用构造问题时的实体和知识做 rejection sampling，保证计划质量。

#### Reasoning Action

当外部工具返回大量非结构化内容时，agent 要能从噪声中提取关键知识并组织 reasoning path。论文用两阶段大模型生成完整 reasoning chain，并用：

- reasoning length；
- answer consistency。

做双过滤。

#### Decision-Making Action

每一步 thought/action 都是隐式决策。论文显式建模 decision-making：

- 基于已有 demonstration trajectories 探索每步可行动作空间；
- 将原始轨迹重构成多步 decision sequences；
- 保留原始选择，训练模型理解为何选某条路径。

### 11.3 Function-calling Data via Environment Scaling

论文还通过 environment scaling 合成通用 function-calling 数据：

- 每个环境可看作 read-write database；
- 自动构造异构模拟环境；
- 系统扩展 function-calling 场景；
- 数据纳入 mid-training。

这训练模型在多样工具/环境中理解接口、读写状态和执行动作。

---

## 12. Agentic Post-training：数据合成、SFT、RL

Post-training 包括三阶段：

```text
High-quality Data Synthesis -> SFT Cold Start -> Agentic RL
```

---

## 13. 高质量数据合成

论文开发端到端合成高不确定性、super-human level QA 的流程。

### 13.1 Graph Construction

通过 random walks 构建高度互联知识图，结合：

- web search；
- 真实网站中的 isomorphic tables；
- 多源事实。

### 13.2 Subgraph Sampling

从知识图和表中采样 subgraphs / subtables，生成初始 QA。

### 13.3 Uncertainty Injection

核心是提高问题不确定性和难度。论文提到用可控 atomic operations 对实体关系做操作，例如：

- 合并属性相似实体；
- 扩大候选集合；
- 增加歧义；
- 减少直接线索；
- 抑制 shortcut。

还基于集合论形式化 information-seeking problem，以更可控地扩展问题并减少结构冗余。

### 13.4 PhD-level Research Questions

论文还构建自动数据引擎，从多学科知识库出发生成 seed QA，再通过 question-crafting agent 和工具逐步扩大范围、提高抽象度、迭代升级复杂度。

附录给出例子，例如军事历史、18 世纪 travelogue 与专利/贵族头衔、化学键长和超共轭等高不确定性问题。

---

## 14. Supervised Fine-tuning for Cold Start

### 14.1 数据来源

从合成高质量 QA 出发，用高性能开源模型生成完整 thought process 和 tool responses，再进行 rejection sampling。

只保留：

- 质量高；
- 轨迹完整；
- 问题解决模式多样；
- 工具响应合理；
- 答案正确的样本。

### 14.2 Mixed Training Paradigm

SFT 使用两种 formulation。

#### ReAct Mode

输入历史状态 `H_t`，输出当前：

- thought `τ_i`；
- tool call `a_i`。

#### Context Management Mode

输入：

- 上一步 summary `S_{t-1}`；
- 上一步 tool call `a_{i-1}`；
- 上一步 tool response `o_{i-1}`。

输出：

- 当前 trajectory summary；
- thought `τ_i`；
- tool call `a_i`。

这能增强模型：

- 状态分析；
- strategic decision-making；
- 将复杂 observation 综合成摘要；
- 在长程任务中保持焦点。

### 14.3 两阶段 context length 训练

| 阶段 | Context | 数据 |
|---|---:|---|
| SFT Stage 1 | 40K | ReAct context < 40K + 全部 Context Management Mode 样本 |
| SFT Stage 2 | 128K | ReAct context 40K-128K + 少量 40K 数据保持稳定 |

---

## 15. Agentic Reinforcement Learning

### 15.1 基本框架

RL 使用 RLVR：模型完整尝试一个任务 rollout，如果最终答案与 ground truth 匹配则获得 reward。

模型在 simulated 或 real-world environment 中不断交互，策略更新后又生成更好的数据，形成闭环。

### 15.2 Real-world Environment

工具集包括：

- Search；
- Visit；
- Python Interpreter；
- Google Scholar；
- File Parser。

论文特别强调真实工具系统稳定性。为避免 API 失败污染训练，构建 unified sandbox 和调度层：

- 并发控制；
- QPS 限制；
- 结果缓存；
- timeout-and-retry；
- graceful degradation；
- backup data sources；
- 统一工具调用接口。

目标是让模型看到的工具接口尽量稳定、确定，避免训练信号被环境随机性污染。

### 15.3 Simulated Environment

论文先构建基于 2024 Wikipedia database 的离线环境，并开发本地 RAG 工具模拟 web。

用途：

- 低成本；
- 高效率；
- 可控；
- 快速 ablation；
- 算法验证。

论文称模拟 Wiki 环境像“wind tunnel laboratory”，用于快速验证 adapted GRPO，然后再迁移到真实环境。

### 15.4 On-Policy Asynchronous Rollout Framework

Agentic rollout 多轮工具调用，训练瓶颈严重。论文实现 step-level asynchronous RL training loop：

- 基于 rLLM framework；
- 两个异步在线服务器：一个 model inference，一个 tool invocation；
- centralized interaction handler 统一处理模型输出和工具反馈；
- 多个 agent 实例并行 rollout，各自独立完成。

这提高 RL 吞吐，避免同步等待慢任务。

### 15.5 RL Algorithm：GRPO 改造

论文使用 tailored GRPO。对每个问题采样 `G` 条轨迹，reward 是纯 0/1 correctness。

优势估计：

```math
\hat A_{i,j}=R_i-mean(\{R_i\}_{i=1}^G)
```

主要特征：

- strict on-policy，轨迹总是由最新 policy 采样；
- 不使用 format reward，因为 SFT 已经教会格式；
- 使用 token-level policy gradient loss；
- 使用 clip-higher 鼓励探索；
- 使用 leave-one-out 降低 advantage 方差；
- 过滤部分负样本，例如超长未给 final answer 的轨迹，避免训练不稳定和 policy collapse。

论文强调算法修改不是为了 novelty，而是为了工程稳定。

### 15.6 Automatic Data Curation

RL 数据动态过滤：

1. 从大数据集 `D` 开始；
2. 用 SFT model 多次 rollout；
3. 删除 always fail 和 always succeed 的问题；
4. 得到中等难度训练集 `D'`；
5. 训练过程中监控哪些问题变得太容易；
6. 后台用中间 checkpoint 在原始 `D` 中寻找新的中等难题；
7. reward plateau 或到达步数后刷新 `D'`；
8. 主 RL loop 不被数据刷新阻塞。

核心思想：让训练数据随模型能力提升而进化，持续提供有效学习信号。

### 15.7 重要结论

论文明确说：agentic RL 成功更依赖：

- 数据质量；
- 环境稳定性；
- 自动数据筛选；
- 工程基础设施。

而不只是具体 RL 算法。

---

## 16. Model Merging

训练最后使用 model merging。

如果多个模型变体来自同一预训练模型，但能力偏好不同，可以用参数加权平均：

```math
\theta_{merged}=\sum_k \alpha_k \theta^{(k)}, \quad \sum_k \alpha_k=1, \alpha_k\ge 0
```

目的：

- 保留不同变体的优势；
- 提升泛化；
- 不增加额外优化成本；
- 在复杂场景中接近各自强项模型表现。

---

## 17. 实验设置

### 17.1 Benchmark

论文评估七类公开信息搜索 benchmark：

- Humanity's Last Exam；
- BrowseComp；
- BrowseComp-ZH；
- GAIA；
- xBench-DeepSearch；
- WebWalkerQA；
- FRAMES；
- 另有 xBench-DeepSearch-2510。

### 17.2 Baselines

两大类：

1. **LLM-based ReAct agents**  
   GLM-4.5、Kimi-K2、DeepSeek-V3.1、Claude-4-Sonnet、OpenAI o3/o4-mini 等。

2. **End-to-end deep-research agents**  
   OpenAI DeepResearch、Gemini DeepResearch、Kimi Researcher 等。

### 17.3 推理参数

| 参数 | 值 |
|---|---|
| temperature | 0.85 |
| repetition penalty | 1.1 |
| top-p | 0.95 |
| max tool invocations | 128 |
| context length | 128K tokens |
| 主指标 | Avg@3 |
| 额外指标 | Pass@1、Pass@3 |

论文说明大多数结果在 2025-09-16 获得，xBench-DeepSearch-2510 在 2025-10-28 评测。

### 17.4 Judge 细节

附录说明：

| Benchmark | Judge |
|---|---|
| GAIA / WebWalkerQA | Qwen2.5-72B-Instruct |
| xBench-DeepSearch / xBench-DeepSearch-2510 | Gemini-2.0-Flash-001 |
| BrowseComp / BrowseComp-ZH | GPT-4o-2024-08-06 |
| HLE text-only | 官方协议，o3-mini evaluator |
| AIME25 / HMMT25 | 人工评估 |
| SimpleQA | 官方评测脚本 |

---

## 18. 主结果

论文表 1 中 Tongyi DeepResearch 结果：

| 模型 | HLE | BrowseComp | BrowseComp-ZH | GAIA | xBench-DeepSearch | WebWalkerQA | FRAMES |
|---|---:|---:|---:|---:|---:|---:|---:|
| GLM-4.5 | 21.2 | 26.4 | 37.5 | 66.0 | 70.0 | 65.6 | 78.9 |
| Kimi K2 | 18.1 | 14.1 | 28.8 | 57.7 | 50.0 | 63.0 | 72.0 |
| DeepSeek-V3.1 | 29.8 | 30.0 | 49.2 | 63.1 | 71.0 | 61.2 | 83.7 |
| Claude-4-Sonnet | 20.3 | 12.2 | 29.1 | 68.3 | 65.0 | 61.7 | 80.7 |
| OpenAI o3 | 24.9 | 49.7 | 58.1 | - | 67.0 | 71.7 | 84.0 |
| OpenAI DeepResearch | 26.6 | 51.5 | 42.9 | 67.4 | - | - | - |
| Kimi Researcher | 26.9 | - | - | - | 69.0 | - | 78.8 |
| Tongyi DeepResearch 30B-A3B | 32.9 | 43.4 | 46.7 | 70.9 | 75.0 | 72.2 | 90.6 |

### 18.1 解读

Tongyi 的强项：

- HLE 32.9，高于表中 DeepSeek-V3.1 29.8、OpenAI o3 24.9、OpenAI DeepResearch 26.6；
- GAIA 70.9，高于 OpenAI DeepResearch 67.4 和 Claude-4-Sonnet 68.3；
- xBench-DeepSearch 75.0，为表中最高；
- WebWalkerQA 72.2，高于 OpenAI o3 71.7；
- FRAMES 90.6，为表中最高。

Tongyi 的相对弱项：

- BrowseComp 43.4，低于 OpenAI DeepResearch 51.5 和 OpenAI o3 49.7；
- BrowseComp-ZH 46.7，低于 OpenAI o3 58.1、DeepSeek-V3.1 49.2。

因此准确表述是：Tongyi 在多个 benchmark 上达到强开源 SOTA 或接近/超过商业系统，但并非所有项目第一。

---

## 19. Heavy Mode

### 19.1 Heavy Mode 是什么？

为了进一步释放 deep research agent 潜力，论文提出 Heavy Mode，通过 **Research-Synthesis framework** 做 test-time scaling。

由于 deep research trajectory 很长，直接把多个完整轨迹拼起来交给 synthesis model 不现实。Tongyi 的 context management 会产生压缩报告 `S_T`，Heavy Mode 正是利用这个压缩表示。

### 19.2 Parallel Research Phase

部署 `n` 个并行 agent，每个 agent 独立探索不同工具和推理路径：

```math
(S_T^u, answer_u)=Agent_u(q), \quad u\in[1,n]
```

每个 `S_T^u` 是第 `u` 个 agent 的最终压缩研究报告。

### 19.3 Integrative Synthesis Phase

一个 synthesis model 汇总所有并行结果：

```math
answer_{final}=Synthesis(\{(S_T^u,answer_u)\}_{u=1}^{n})
```

优势：

- 不需要拼接完整轨迹；
- 每个 agent 的 `S_T` 保留关键推理逻辑和发现；
- synthesis model 可比较多个路径；
- 在有限上下文内实现 test-time scaling。

### 19.4 Heavy Mode 结果

论文 Figure 6 报告：

- HLE：38.3%；
- BrowseComp-ZH：58.1%；
- BrowseComp：58.3%。

相比普通 Tongyi：

- HLE：32.9 → 38.3；
- BrowseComp：43.4 → 58.3；
- BrowseComp-ZH：46.7 → 58.1。

Heavy Mode 显著提升，但代价是更多并行 agent、更多工具调用和更高推理成本。

---

## 20. Pass@1 / Pass@3 分析

论文 Figure 7 对 Avg@3、Pass@1、Pass@3 做细粒度分析。

关键值：

- BrowseComp Pass@3：59.64；
- BrowseComp-ZH Pass@3：63.67；
- HLE Pass@3：45.9。

这说明同一问题多次 rollout 能提高覆盖率和成功率，也支持 deep research 任务适合 test-time scaling。

需要注意：Pass@3 不是单次成本下的结果，它依赖多次独立运行，实际应用要考虑成本。

---

## 21. RL 训练动态分析

### 21.1 Reward 与 Entropy

Figure 8 显示：

- reward 随训练显著上升；
- policy entropy 初期上升后稳定；
- 没有 collapse 或 explosion。

论文将其归因于：

- 动态数据筛选持续提供难题；
- 环境设计稳定；
- RL 算法做了必要修改；
- 过滤坏负样本降低 policy collapse 风险。

### 21.2 Context Length of RL

论文比较 32K、48K、64K context limit 的 RL。

观察：

- 三者 reward 都能稳定上升；
- 64K 上限最高，因为数据筛选也是用 64K 模型构造；
- 48K 达到稳定平衡；
- 32K 模型平均 response length 下降。

有趣结论：当 32K 模型面对由 64K 模型筛出的复杂任务时，如果照长路径走会超上下文并得零分，因此 RL 迫使它寻找更短、更高效的解法。

### 21.3 Interaction Test-time Scaling

Figure 10a 展示 BrowseComp 上，随着 context length / interaction turns 从 8K 到 128K 增加，准确率持续提升。

论文指出，这与传统 reasoning model 的 token scaling 不同：

```text
reasoning model scaling：增加输出 tokens
DeepResearch agent scaling：增加与环境的交互次数，获得更多 observation
```

这与 MiroThinker v1.0 的 interactive scaling 思想非常接近。

### 21.4 Simulated Environment 到真实环境

Figure 10b 显示模拟环境中的 reward curve 与真实环境趋势相似。论文认为模拟 Wiki 环境可以作为 algorithm wind tunnel，提高开发效率。

---

## 22. Super-human Level Synthetic Data 分析

论文对 SFT 数据做统计：

- 超过 20% 样本超过 32K tokens；
- 超过 20% 样本涉及 10 次以上工具调用。

这说明 cold-start 数据足够长、足够复杂，为 RL 提供了强初始化。

论文强调：RL 阶段通过 automated data curation 更有效利用这些 synthetic data。

---

## 23. General Benchmarks

论文还评估 AIME25、HMMT25、SimpleQA。

Figure 11 显示 Tongyi DeepResearch 相比 Qwen3-30B-A3B-Thinking-2507 有提升，并接近/超过 Qwen3-235B-A22B-Thinking-2507 某些项目。

论文解释：

- Search 对知识密集型 SimpleQA 有帮助；
- Python Interpreter 对数学推理任务有帮助；
- agentic tool use 正在与模型训练融合。

这说明 DeepResearch 训练不一定只提升搜索 benchmark，也可能提升一般知识/数学任务，因为模型学会了使用外部工具补足自身。

---

## 24. 工具系统细节

附录 D 描述五个工具。

### 24.1 Search

- 使用 Google search；
- 输入一个或多个 query；
- 并发执行；
- 每个 query 返回 top-10；
- 包含 title、snippet、URL。

### 24.2 Visit

- 用于目标导向网页信息抽取；
- 输入 URL 和 goal；
- Jina 解析网页；
- summary model 只抽取与目标相关信息；
- 降低主模型上下文压力。

### 24.3 Python Interpreter

- 在 sandbox 执行 Python；
- 代码需放在 `<code>` 标签；
- 结果必须 print；
- 支持动态计算、数据处理、库调用、验证。

### 24.4 Google Scholar

- 检索学术论文、引用、作者资料；
- 支持多个 query；
- 用于文献综述、学术证据、研究问题。

### 24.5 File Parser

- 解析 PDF、DOCX、MP4、MP3 等本地或 URL 文件；
- 第一步转换成 plain text，必要时转写音视频；
- 第二步 summary model 直接回答用户对文件的问题。

---

## 25. 论文局限

论文明确列出五点局限。

### 25.1 128K context 仍不足

最复杂长程任务可能超过 128K，需要：

- 更长 context；
- 更强 context management；
- 更可靠 memory；
- 更高效 evidence compression。

### 25.2 暂未发布更大模型

报告只发布 30B-A3B 规模，更大模型仍在进行中。模型规模可能继续提升能力。

### 25.3 报告生成 fidelity 和用户偏好仍需优化

Deep research 最终输出常是报告，仍需提高：

- 事实忠实性；
- 引用可靠性；
- 用户偏好对齐；
- 有用性；
- 可读性。

### 25.4 RL 框架效率仍可提升

论文希望探索 partial rollouts，但这涉及 off-policy training 难题，例如 distribution shift。

### 25.5 工具和 prompt 范围有限

当前训练针对特定 prompt instructions 和 predefined tool sets。未来要扩展到更广泛 agentic tool use。

---

## 26. Model Scale 讨论

论文强调小模型 agent 训练很有价值：

- 部署更高效；
- 可运行在更多设备/场景；
- 响应更快；
- 降低使用门槛；
- 让 autonomous research agent 更可及。

Tongyi 的 30.5B total / 3.3B activated per token 是 MoE 参数效率路线。

---

## 27. What's Next

论文未来方向：

- 从 domain-specific agents 到 general-purpose agents；
- 开源具有 emergent agency 的模型；
- 发展下一代 agent foundation model；
- 统一 reasoning、memory、autonomy；
- 让 AI 系统能在更广泛领域自主规划和行动。

---

## 28. 关键亮点

### 28.1 把 mid-training 正式纳入 agent 训练范式

Tongyi 的一个重要贡献是明确提出 agentic mid-training 的必要性。后续 REDSearcher、MiroThinker-1.7 也都强化了这个思路。

### 28.2 环境三分法很实用

Prior World / Simulated / Real-world 的划分非常适合 agent 工程：

- 先便宜地学基本能力；
- 再在模拟环境快速迭代；
- 最后到真实环境提升和验证。

### 28.3 Context Management Mode 贴近研究过程

用 evolving report `S_t` 作为压缩 memory，比单纯保留所有 observation 更可扩展，也贴近人类研究者不断记笔记和更新摘要的过程。

### 28.4 动态数据筛选是 RL 成败关键

Always easy / always hard 问题都没有训练信号。Tongyi 的 data curation 持续维持“中等难度”训练集，是 agentic RL 很值得借鉴的实践。

### 28.5 Heavy Mode 提供低耦合 test-time scaling

通过多个并行 agent 的摘要 `S_T` 做 synthesis，避免拼接完整轨迹导致上下文爆炸，是一种实用的 test-time scaling 设计。

---

## 29. 批判性阅读

### 29.1 各 benchmark judge 不统一

论文对不同 benchmark 使用不同 judge：Qwen2.5-72B、Gemini-2.0-Flash、GPT-4o、o3-mini、人工、官方脚本。这样符合各自协议，但跨 benchmark overall 解读要谨慎。

### 29.2 Web 环境日期会影响结果

论文标明结果采集日期。Deep research benchmark 依赖网页、搜索、API，结果可能随时间变化。复现时需要关注：

- 搜索排序；
- 网页可访问性；
- 页面内容更新；
- API fallback；
- 缓存策略。

### 29.3 Synthetic QA 可能偏谜题式

附录中的高不确定性 QA 很复杂，但有些像“线索谜题”。这对 BrowseComp 类任务有帮助，但与真实用户开放研究报告仍有分布差异。这也是 S1-DeepResearch 后来批判 search-centric 数据的背景。

### 29.4 Heavy Mode 成本未充分量化

Heavy Mode 分数提升明显，但论文没有详细量化：

- 并行 agent 数量；
- 额外 token；
- 额外工具调用；
- 延迟；
- 成本；
- synthesis 失败率。

实际产品化需要成本曲线。

### 29.5 Context summary 可能丢信息

`S_t` 是压缩记忆，若 summary 漏掉关键证据或误写中间结论，后续步骤会被污染。需要 citation-aware memory 或可回溯 evidence store 进一步增强。

### 29.6 工具替代实现可能影响复现

论文附录提到由于内部 API 和 fallback 策略，开源 GitHub 提供替代实现，并称经过测试可以复现结果。但工具实现差异仍可能影响最终分数。

---

## 30. 与其他论文的对照理解

### 30.1 与 MiroThinker v1.0

共同点：

- 都是开源 research agent；
- 都强调工具交互；
- 都用 ReAct 类范式；
- 都讨论 interaction scaling。

差异：

- Tongyi 使用 Markovian summary state `S_t`；
- Miro v1.0 使用 recency-based retention；
- Miro v1.0 提供 8B/30B/72B 多尺寸；
- Tongyi 强调 30B-A3B 高参数效率。

### 30.2 与 REDSearcher

REDSearcher 延续 Tongyi 的 mid/post-training 思路，但更深入解决：

- 任务难度形式化；
- treewidth + MSD；
- tool-enforced query；
- functionally equivalent simulation；
- RL 后工具调用效率。

### 30.3 与 MiroThinker-1.7 & H1

Miro H1 进一步提出 verification-centric heavy-duty reasoning。Tongyi Heavy Mode 是并行研究 + synthesis；Miro H1 是 local/global verifier。二者都属于 test-time scaling，但机制不同。

### 30.4 与 S1-DeepResearch

S1 认为只做 search-centric QA 不够，进一步覆盖报告、文件、技能、指令跟随。Tongyi 也有 report 和 File Parser，但主要评测仍集中于信息搜索 benchmark。

---

## 31. 对实践的启示

### 31.1 训练 agent 要分阶段

实用路线：

```text
通用 base model
-> agentic mid-training 学规划/推理/工具格式
-> SFT cold start 学完整轨迹
-> RL 学环境反馈下的策略优化
-> model merging 或偏好整合
```

### 31.2 环境稳定性比算法花样更重要

如果工具经常失败，RL reward 会非常噪：

- 策略错和工具错无法区分；
- 模型可能学到错误行为；
- 数据污染；
- 训练不稳定。

因此需要：缓存、重试、fallback、QPS 控制、统一接口。

### 31.3 数据要持续刷新难度

RL 训练集应维持“模型有机会学到东西”的难度区间：

- 太简单：所有 rollout 都成功，advantage 低；
- 太难：所有 rollout 都失败，无法学习；
- 中等难度：有成功有失败，信号最强。

### 31.4 Context management 是产品核心

长程研究不能无限塞历史。需要持续构造：

- 当前研究摘要；
- 已验证证据；
- 待解决问题；
- 候选答案；
- 引用与来源；
- 不确定性。

Tongyi 的 `S_t` 是一个基础版本。

### 31.5 Heavy Mode 适合高价值任务

对普通问题不一定需要 heavy mode；但对高价值任务，例如：

- 投资研究；
- 法律检索；
- 医学综述；
- 科学调研；
- 复杂商业分析。

可以用多 agent 并行 + synthesis 提升可靠性。

---

## 32. 可继续研究方向

1. **Context summary 的事实保持评估**  
   检查 `S_t` 是否漏掉关键证据、是否引入幻觉。

2. **Heavy Mode 成本-准确率曲线**  
   系统测试并行 agent 数量 `n` 对准确率、延迟、成本的影响。

3. **Prior/Simulated/Real 环境迁移分析**  
   研究哪些能力可从模拟环境迁移，哪些必须真实环境训练。

4. **动态数据筛选策略消融**  
   对比 fixed dataset、easy/hard filtering、动态刷新对 RL 的影响。

5. **模型合并权重搜索**  
   研究不同能力模型 merging 如何影响 search、math、report、file 等指标。

6. **从 predefined tool set 到开放工具集**  
   训练模型读工具文档、发现新工具、动态组合工具。

7. **面向真实报告的 citation verifier**  
   将 Tongyi 的 context management 与 claim-level citation store 结合。

---

## 33. 术语速查

| 术语 | 解释 |
|---|---|
| Agentic CPT | Agentic Continual Pre-training，作为 mid-training 培养 agent prior |
| Agentic Post-training | SFT cold start + agentic RL |
| RLVR | Reinforcement Learning with Verifiable Rewards |
| GRPO | Group Relative Policy Optimization |
| ReAct | Reasoning and Acting 交错轨迹 |
| Context Management Mode | 用 evolving report `S_t` 压缩历史，结合上一步 action/observation 继续推理 |
| Prior World Environment | 无真实反馈、基于先验知识的大规模低成本环境 |
| Simulated Environment | 本地可控仿真环境，低成本快速迭代 |
| Real-world Environment | 真实工具/API 环境，真实性高但成本高且不稳定 |
| Automatic Data Curation | 根据训练动态刷新中等难度训练题 |
| Heavy Mode | 多 agent 并行研究 + synthesis 的 test-time scaling 模式 |
| Model Merging | 将同源不同能力模型参数加权平均 |

---

## 34. 最终理解

Tongyi DeepResearch 的核心不是某一个算法，而是一套完整 agent 工程方法论：

```text
合成数据是燃料
环境设计是基础设施
mid-training 提供 agent prior
SFT 提供冷启动行为
RL 提供环境反馈优化
context management 支撑长程研究
heavy mode 提供 test-time scaling
```

这篇技术报告的重要意义在于，它把 deep research agent 从“搭一个工具调用 workflow”推进到“训练一个具备 agentic prior 的开源模型”。它强调，agent 能力不是只靠 prompt 和工具堆出来的，而需要数据、环境、训练和推理模式一起设计。

从后续几篇论文看，Tongyi 提出的很多问题仍是整个领域的主线：如何合成高难数据、如何降低 rollout 成本、如何让交互更有效、如何验证证据链、如何从搜索走向真实研究交付。理解 Tongyi，有助于理解后续 REDSearcher、MiroThinker-H1 和 S1-DeepResearch 的发展脉络。
