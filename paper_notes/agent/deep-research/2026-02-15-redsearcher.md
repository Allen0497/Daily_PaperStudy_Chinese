# REDSearcher 论文详解

> 来源文件：`2026-02-15-redsearcher.pdf`  
> 论文题目：**REDSearcher: A Scalable and Cost-Efficient Framework for Long-Horizon Search Agents**  
> 作者：REDSearcher Team  
> 版本信息：arXiv:2602.14234v1，2026-02-15  
> 原文：[arXiv 摘要页](https://arxiv.org/abs/2602.14234) · [PDF](https://arxiv.org/pdf/2602.14234)  
> 项目资源：[项目页](https://redsearchagent.github.io)  
> 说明：本文档基于当前目录中的本地 PDF 阅读整理，未联网核验项目页、榜单、代码或权重，也未独立复现实验结果。

---

## 1. 一句话总结

REDSearcher 提出了一套面向 **Long-Horizon Deep Search Agent** 的训练框架，核心不是单纯扩大模型或拉长推理链，而是把 **复杂任务合成、低成本 mid-training、真实/模拟环境 post-training、文本与多模态搜索能力** 放在一个统一体系里共同设计。它用“图拓扑复杂度 + 证据分散度”构造高难搜索任务，用低成本合成数据和本地模拟环境降低轨迹采集成本，再通过 SFT 与 RL 训练 30B-A3B 级别模型，目标是在较小激活参数下获得强长程搜索能力。

---

## 2. 论文要解决的问题

### 2.1 Deep Search 与普通 RAG 的区别

论文强调，REDSearcher 关注的是 **deep search**，不是普通 RAG。

普通 RAG 的典型流程是：

```text
用户问题 -> 一次或少量检索 -> 拼接文档 -> 模型回答
```

这种方式适合事实直接、证据集中、检索入口明确的问题，但不适合以下场景：

- 问题线索模糊，需要多次搜索改写；
- 证据散落在多个来源；
- 需要保持多个候选假设并逐步排除；
- 某些线索不能靠文本搜索直接解决，需要地图、学术检索、代码计算或图像工具；
- 需要先定位中间实体，再继续追踪后续证据。

Deep search 的核心是闭环：

```text
计划 -> 搜索/访问/计算/视觉识别 -> 观察 -> 修正计划 -> 再搜索 -> 验证 -> 回答
```

### 2.2 当前训练深度搜索智能体的瓶颈

论文认为，优化深度搜索智能体主要被两个问题卡住：

1. **高质量长程搜索轨迹极其稀缺**  
   真正能训练 agent 的数据不是简单 QA，而是包含多轮 thought/action/observation 的完整轨迹。要收集这些轨迹成本很高，而且错误很多。

2. **外部工具调用带来的 rollout 成本太高**  
   深度搜索可能需要几十到上百次搜索、网页访问、代码执行、学术检索或地图查询。每次 rollout 都依赖外部 API，带来费用、延迟、失败率和不可复现问题。

因此 REDSearcher 的目标是：**用可控合成任务和低成本模拟环境，让长程搜索 agent 的训练变得可扩展、可验证、成本可控。**

---

## 3. 核心贡献

论文的主要贡献可以概括为四点。

### 3.1 Dual-Constrained Task Synthesis

REDSearcher 将复杂搜索任务合成表述为一个受约束的问题，并从两个维度控制难度：

- **图拓扑复杂度**：通过 treewidth 等结构度量，让问题不只是线性多跳，而是包含环、菱形、交叉约束等更难结构；
- **证据分散度**：把互相关联的证据刻意分散到不同来源，减少“一页答案”捷径。

### 3.2 Proactive Tool-Augmented Queries

论文不希望模型只靠记忆或文本检索答题，因此将显式实体改写成必须通过工具解决的约束。例如：

- 把地点名改成“距离某城市约两小时车程的城市”，迫使模型使用地图或路线工具；
- 把学者名改成“引用量在某个范围内的学者”，迫使模型使用学术检索；
- 把文本线索替换成视觉线索，迫使模型用图像识别或视觉搜索。

这使工具调用成为成功解题的必要条件，而不是可有可无的附加能力。

### 3.3 Cost-Efficient Mid-Training

论文在 pre-training 与 post-training 之间加入两阶段 agentic mid-training：

- 第一阶段训练原子能力：grounding、planning；
- 第二阶段训练工具使用和长程交互。

目的是先把模型 warm up 成“会做 agent 的模型”，再进入昂贵的真实环境 SFT/RL，从而降低高质量 trajectory 的采集成本。

### 3.4 Functionally Equivalent Simulation Environment

论文构建本地模拟搜索环境，让它在接口上类似真实搜索环境，但成本低、速度快、可控、可复现。它保证所有必要证据都在本地语料中，同时加入大量干扰文档，避免环境过于简单。

这相当于给 deep search agent 建了一个“风洞”：先在低成本环境中快速迭代算法，再迁移到真实环境验证。

---

## 4. Agent 问题形式化

### 4.1 核心变量

论文把一次 web-enabled QA session 定义为 agent 与 environment 的多步交互。核心变量包括：

| 变量 | 含义 |
|---|---|
| `q` | 用户问题，可以是文本或图文混合 |
| `a_t` | 第 `t` 步工具动作，例如 search、visit、python、map、scholar |
| `o_t` | 工具返回的观察，例如搜索结果、网页摘要、图像信息、结构化字段 |
| `τ_t` | agent 内部状态，例如当前约束、候选假设、已抽取实体、阶段性结论 |
| `y` | 最终答案，应由收集到的证据支撑 |

### 4.2 ReAct 轨迹

REDSearcher 使用 ReAct 风格轨迹：

```math
H_T = (q, (\tau_0,a_0,o_0), (\tau_1,a_1,o_1), ..., (\tau_T,a_T,o_T), y)
```

其中 `τ_t` 可以理解为“状态/思考”，`a_t` 是工具调用，`o_t` 是环境反馈。最终模型基于整个搜索过程输出答案 `y`。

### 4.3 Context Management：Discard-all

论文采用一种非常直接的上下文管理策略：**Discard-all**。

当上下文接近窗口阈值时：

- 移除过去所有 `(τ_i, a_i, o_i)` 历史；
- 保留原始问题 `q` 和最小任务规格；
- 让 agent 从新上下文重新开始 rollout。

这和 MiroThinker 的“保留 thought/action，省略旧 observation”不同。REDSearcher 的策略更激进，优点是实现简单、能释放大量上下文；缺点是可能丢失前期证据和长期状态，需要模型重新探索。

---

## 5. 难度建模：图拓扑复杂度 + 证据分散度

REDSearcher 的数据合成部分非常关键。论文认为，一个 deep search 问题难不难，不能只看 hop 数，而要看 **推理结构是否纠缠**，以及 **证据是否分散**。

### 5.1 Topological Logical Complexity：Treewidth 视角

论文将问题的逻辑结构抽象成图 `G=(V,E)`：

- 节点代表实体、条件、约束或中间变量；
- 边代表依赖、关系或推理连接。

然后用 **treewidth** 衡量结构复杂度。直觉上：

- 低 treewidth：像链或树，可顺着路径一步步推；
- 高 treewidth：多个变量互相约束，需要同时维护多个候选并做全局一致性验证。

论文用近似复杂度表达式说明：

```math
C_{reasoning} \approx O(N \cdot d^{k+1})
```

其中：

- `N` 是推理步数；
- `d` 是每一步候选分支数；
- `k=tw(G)` 是 treewidth。

当 `k` 增大时，搜索空间会近似指数增长。这解释了为什么“3 跳线性问题”可能比“2 跳但互相约束的问题”更简单。

### 5.2 三类结构复杂度

论文用 Figure 2 展示三种典型结构：

| 类型 | 结构 | treewidth | 难点 |
|---|---|---:|---|
| Type I | 链 / 树 | `k=1` | 顺序传播即可，大部分传统多跳 QA 属于这一类 |
| Type II | 环 / 菱形约束 | `k=2` | 多条路径汇合，需要同时验证电影、导演、演员等变量一致 |
| Type III | 高维耦合 / 类 clique | `k≥3` | 多变量完全耦合，不能拆成独立子问题，需要全局一致性检查 |

这部分的思想很重要：REDSearcher 不满足于制造“更长的问题”，而是制造“结构更纠缠的问题”。

### 5.3 Distributional Complexity：Minimum Source Dispersion

只有结构复杂还不够。如果所有证据都在同一个网页中，即使逻辑结构复杂，模型也可能一次搜索直接找到答案。

因此论文定义 **Minimum Source Dispersion（MSD）**：解决该问题至少需要多少个不同文档来覆盖所有必要证据。

```math
D_{task}=\min |S|, \quad s.t. \ Cover(S,G)=True
```

含义是：从网页集合 `W` 中选最小文档子集 `S`，只要 `S` 中的信息足以覆盖图 `G` 的所有必要节点/边，那么 `|S|` 就是该任务的最小证据来源数。

难任务应同时满足：

```text
高 treewidth + 高 MSD
```

也就是：

- 推理结构复杂；
- 必要事实分散；
- 没有单页捷径；
- 必须多轮搜索和跨文档综合。

---

## 6. Scalable Complex Task Synthesis Pipeline

论文的数据合成 pipeline 分成两大阶段：QA Generation 与 Task Verification。

### 6.1 Seed Collection and Filtering

REDSearcher 使用英文和中文 Wikipedia 实体初始化 seed pool。为了保证 seed 适合构造搜索任务，进行多级过滤：

1. **长度过滤**：去除过短页面和过长/过热门页面；
2. **结构过滤**：去除列表、索引、术语表；
3. **元页面过滤**：去除管理性页面；
4. **概念过滤**：用 LLM 分类器区分具体实体和抽象理论；
5. **别名/重定向去重**：形成紧凑高信号实体池。

这一步确保问题不会来自无意义、过窄或过泛的实体。

### 6.2 Graph Construction and Topological Enrichment

从 seed entity 出发，论文通过两条路径扩展图：

- **Wikidata 结构关系**：抽取结构化三元组；
- **Web traversal**：沿网页和超链接发现相关文档。

关键是，论文不只是构造简单 DAG，还用 LLM-driven Graph Agent 对图进行拓扑加密，引入环和交叉约束，让搜索路径从线性检索变成联合约束满足。

### 6.3 One-Graph-Multi-Task Sampling

构造一个拓扑复杂、证据丰富的 master graph 成本很高。REDSearcher 采用 **One-Graph-Multi-Task**：

- 从同一 master graph 采样多个 connected subgraph；
- 每个 subgraph 对应一个独立 QA；
- answer node 按拓扑角色选择，例如深层叶子、高度 hub；
- 不同 answer node 引出不同推理模式。

这能把图构造成本摊薄到大量样本上。

### 6.4 Question Generation

给定采样到的知识图和目标答案，使用强 LLM 生成自然语言问题。要求：

- 忠实表达图中的约束；
- 语言自然、简洁；
- 不直接暴露答案；
- 保留多跳/多约束结构。

### 6.5 Tool-Enforced Query Evolution

这是 REDSearcher 数据构造中很有特色的一步。它用 Editor Agent 把问题中的直接事实改写成工具可解析的功能约束。例如：

| 原始线索 | 改写后线索 | 需要工具 |
|---|---|---|
| 上海 | “距离北京约 1200 km 的中国东部城市” | 地图/距离检索 |
| 某学者姓名 | “引用量约为 N 的学者” | Google Scholar |
| 某地点图片 | “图中建筑/标志对应的地点” | 图像识别/图像搜索 |

这样模型不能仅靠文本关键词匹配，而必须调用适当工具。

---

## 7. Verifier Pipeline：任务验证流程

由于合成问题会引入混淆和跨源组合，可能出现太简单、不可检索、内部矛盾、多答案等问题。论文设计了逐级升级的 verifier pipeline。

### 7.1 五级验证

1. **LLM solver pre-filter，无工具**  
   如果没有工具的 LLM 可以直接答对，说明任务太简单，删除。

2. **Retrievability check，搜索片段验证**  
   用搜索引擎查问题，如果 top-50 snippets 中没有给定答案，说明开放网络可检索性弱，删除。

3. **Hallucination / inconsistency check**  
   把构造时的证据和问题交给 LLM verifier，检查 question-graph-answer 是否矛盾。

4. **Agent rollout verification**  
   用强 tool-using agent 对同一问题进行多次 rollout，只要至少一次能找到给定答案，就保留，并记录 pass rate。

5. **Answer uniqueness check**  
   检查是否存在多个候选答案都满足约束。如果 agent 自然发散到不同有效答案，说明问题不唯一，删除。

### 7.2 数据质量研究

论文报告了两个质量观察：

- 对 500 个样本进行人工检查，超过 85% 通过逻辑一致性和 grounding 充分性验证；
- DeepSeek-V3.2 在标准 agent setting 下约 40% 准确率，说明数据并不简单；
- 人类 annotator 在 30 分钟搜索预算下能解出 47%，说明任务大体可解但具有挑战性。

这组结果支持作者的主张：合成数据不是随便生成的，而是具备一定可解性和难度。

---

## 8. 多模态任务合成

REDSearcher 将同一合成框架扩展到 image-text 场景。

### 8.1 Modality Injection

核心思想是把纯文本 DAG 变成跨模态 DAG，让某些节点必须通过图像理解解决。

两种机制：

1. **Visual Attribute Anchoring**  
   给中间节点附加图像，并生成/检索图像描述，例如物体、场景、符号、图表趋势。

2. **Cross-modal Dependency**  
   让下游节点依赖某个视觉线索，例如衣服上的徽章、背景建筑、图表趋势线。如果不看图，就无法推出后续节点。

这避免图片沦为装饰，而是成为推理链中的必要瓶颈。

### 8.2 Multimodal Question Fuzzing

论文使用图像感知的 fuzzing：

- 不直接命名图中内容，而用“图中这个标志”“该背景中的建筑”等抽象引用；
- 把视觉证据插入到任意推理位置，而不仅仅放在问题开头；
- 可在文本推理几步后设置视觉瓶颈，提高有效推理深度。

### 8.3 Multimodal Verifier Pipeline

多模态场景额外有几类坏样本：

- 不看图也能回答；
- 只看图就能猜答案；
- 图片与问题无关；
- 图片过于直接泄漏答案；
- 图像和网页证据不能形成互补链条。

因此额外加入：

- text-only solvability check；
- text-only retrievability check；
- vision-only solvability check；
- visual-search alignment；
- multimodal agent rollout。

### 8.4 多模态轨迹生成

论文使用 Qwen3VL-235B 作为 ReAct agent 生成多模态 SFT 轨迹：

- 交替生成 reasoning 和 tool call；
- 每个 episode 最多 20 轮；
- 只保留最终答案匹配 ground-truth 的轨迹。

---

## 9. Training Recipe：训练流程

REDSearcher 从开源模型出发，论文附录说明文本模型基于 **Qwen3-30B-A3B**，多模态模型基于 **Qwen3-VL-30B-A3B-Thinking**。

整体训练包括：

```text
Mid-training Stage I -> Mid-training Stage II -> Agentic SFT -> Agentic RL
```

---

## 10. Agentic Mid-Training

论文认为，pre-trained LLM 虽然有知识和语言能力，但没有足够的外部环境交互经验。直接做 agentic RL 成本高、探索成功率低。因此先做两阶段 mid-training。

### 10.1 Stage I：Intent-anchored Grounding and Hierarchical Planning（32K Context）

Stage I 训练两个原子能力。

#### Intent-anchored Grounding

在噪声网页环境中，模型需要从大量无关信息中找出当前意图缺失的信息。论文通过 reverse QA synthesis 构造数据：

1. 给定中心实体 `E` 与文档 `D`；
2. 抽取与中心实体相关的事实片段 `F`；
3. 基于这些事实生成不同 query intents；
4. 在输入中混入相关文档和无关 distractor documents；
5. 训练模型根据当前 intent 抽取关键事实，避免幻觉。

这对应真实搜索中的一个重要能力：网页很长、噪声很多，agent 必须知道“当前到底要找什么”。

#### Hierarchical Planning

深度搜索问题通常目标模糊、推理路径长，不可能从问题一开始就规划每一步。论文将目标分为：

- **Concrete goals**：当前明确要解决的子问题；
- **Ambiguous goals**：未来需要通过搜索缩小不确定性的目标。

这让 agent 能维护“已获得信息”和“仍需获取信息”，避免搜索过程中迷失。

### 10.2 Stage II：Agentic Tool Use and Long-horizon Interaction（128K Context）

Stage II 让模型接触工具调用和长程环境反馈。

#### Agentic Tool Use

论文用 LLM 合成大量工具协议：

- 工具描述；
- 接口签名；
- 工具调用链；
- 对应环境反馈。

这样不必真实调用 API，就能低成本覆盖多样工具调用模式。

#### Long-Horizon Interaction

仅靠 LLM 模拟长程环境会面临成本和一致性问题，因此论文构建本地模拟 web search 环境：

- 基于 Wikipedia 和 web crawl dumps；
- 支持 search、visit 等操作；
- 确保合成问题可在本地环境解决；
- 用大规模复杂 query 在本地环境生成轨迹。

这一步训练模型在 128K context 下保持状态、目标一致性和长链路搜索能力。

---

## 11. Agentic Post-Training

Post-training 包括 SFT 和 RL。

### 11.1 高质量真实环境轨迹合成

REDSearcher 使用五类真实工具：

| 工具 | 功能 |
|---|---|
| Search | Google 搜索，返回标题、snippet、URL |
| Visit | 访问网页，用 Jina 和 summarizer 按目标抽取信息 |
| Python | 沙箱代码执行，用于计算、数据处理、逻辑验证 |
| Google Scholar | 学术论文、引用、作者检索 |
| Google Maps | 地点、路线、距离、地理信息查询 |

轨迹合成使用 ReAct 循环，最大上下文 128K。过滤策略：

- 只保留最终答案正确的样本；
- 删除失败工具调用过多的样本；
- 每个问题只保留一条轨迹，增加样本多样性。

### 11.2 Supervised Fine-Tuning

SFT 在 mid-training checkpoint 上进行，使用高质量轨迹训练 agentic reasoning。训练时：

- 使用标准 next-token prediction；
- mask 掉 environment observation，不对工具返回计算 loss；
- 最大 context length 为 128K。

这和大多数 agent SFT 一致：模型学的是 thought/action/final answer，不学工具输出本身。

### 11.3 Agentic Reinforcement Learning

REDSearcher 使用 RLVR 和 GRPO。

#### 奖励

最终 reward 是 `{0,1}`：

- 答案正确：1；
- 答案错误：0。

论文明确说不使用 format reward，因为 SFT 阶段已经让模型掌握了输出格式。

#### GRPO

对每个问题采样一组 rollouts，计算组内归一化 advantage。核心是：同一个问题下，比组平均更好的轨迹概率上升，比组平均更差的下降。

#### 模拟环境

真实搜索 API 存在不稳定和高成本问题，因此论文构建 functionally equivalent simulation environment：

- 包含数千万本地文档；
- 基于 finewiki dumps、缓存搜索/访问结果；
- 支持 search、visit、python；
- 用 URL obfuscation 避免模型利用 Wikipedia URL 模式作弊；
- 加入大量干扰文档模拟真实网络噪声。

#### RL Query Curation

论文过滤过简单和过难样本，因为它们不能提供有效学习信号。还用 **Agent-as-Verifier** 检查 QA 是否多答案或 ground-truth 不一致。论文称该流程将 RL query set 错误率降低到原来的 10%。

#### RL 训练框架

由于 deep search rollout 很长，论文使用异步 rollout：

- 基于 Slime；
- rollout 可能达到 128K tokens；
- 设计两级负载均衡以提高 prefix cache 命中率；
- 同一 rollout 维持 inference engine affinity；
- 跨 engine 用 round-robin + least-access；
- environment server 封装工具接口并提供 fallback。

---

## 12. 实验设置

### 12.1 文本 benchmark

论文评估：

- Humanity's Last Exam；
- BrowseComp；
- BrowseComp-ZH；
- GAIA。

### 12.2 多模态 benchmark

论文评估：

- MM-BrowseComp；
- BrowseComp-VL；
- MMSearch-Plus；
- MMSearch；
- LiveVQA。

还把 REDSearcher-MM 放到部分文本 benchmark 上测试迁移能力。

### 12.3 Baselines

文本对比包括：

- Proprietary Deep Research Agents：Seed1.8、Gemini、GPT-5、Claude、OpenAI-o3 等；
- Open-source Deep Research Agents：Kimi-K2.5-Agent、GLM-4.7、DeepSeek-V3.2、LongCat 等；
- Open-source 30B-A3B Agents：WebResearcher、WebSailorV2、Tongyi DeepResearch、GLM-4.7-Flash。

多模态对比包括：

- Gemini、Seed、GPT-5；
- Qwen2.5-VL、Qwen3-VL Thinking；
- MMSearch-R1、WebWatcher、DeepEyesV2、Vision-DeepResearch。

### 12.4 推理与训练细节

附录给出关键实现细节：

| 项目 | 设置 |
|---|---|
| 文本 backbone | Qwen3-30B-A3B |
| mid-training Stage I batch size | 512 |
| mid-training Stage II batch size | 256 |
| SFT batch size | 128 |
| mid/SFT learning rate | 5e-5 到 1e-6 |
| RL mini-step | 32 queries × 16 rollouts = 512 samples |
| RL learning rate | 1e-6 |
| RL clip high | 0.28 |
| entropy / KL loss | 不使用 |
| inference temperature | 0.85 |
| inference top-p | 0.95 |
| maximum length | 128K |
| visit summarizer | Qwen3-30B-A3B-Instruct-2507 |
| LLM-as-Judge | GPT-OSS-120B |
| 多模态 backbone | Qwen3-VL-30B-A3B-Thinking |
| 多模态 SFT LR | 1e-5 |
| 多模态 RL batch | 32 prompts × 8 rollouts |
| 多模态最大响应长度 | 32,768 tokens |
| 多模态 RL 工具调用上限 | 20 calls |

---

## 13. 文本实验结果

论文表 1 中 REDSearcher 的核心结果如下。

### 13.1 主要文本结果

| 模型 | Size | BrowseComp | BrowseComp-ZH | GAIA | HLE | Overall |
|---|---:|---:|---:|---:|---:|---:|
| WebResearcher-30B | 30B-A3B | 37.3 | 45.2 | - | 28.8 | - |
| WebSailorV2-30B | 30B-A3B | 35.3 | 44.1 | 74.1 | 30.6 | 46.0 |
| Tongyi DeepResearch-30B | 30B-A3B | 43.4 | 46.7 | 70.9 | 32.9 | 48.5 |
| REDSearcher | 30B-A3B | 42.1 / 57.4* | 49.8 / 58.2* | 80.1 | 34.3 | 51.6 |

其中 `*` 表示使用 context management technique 的结果。

### 13.2 结果解读

论文强调：

- REDSearcher 在同规模 30B-A3B agent 中 overall 最强；
- GAIA 达到 80.1，高于 Tongyi DeepResearch-30B 的 70.9 和 WebSailorV2-30B 的 74.1；
- BrowseComp-ZH 49.8 / 58.2* 高于 Tongyi DeepResearch 的 46.7；
- HLE 34.3 高于 Tongyi DeepResearch 的 32.9；
- BrowseComp 原始 42.1 略低于 Tongyi 43.4，但 context management 后达到 57.4。

需要谨慎的是，论文表中部分模型使用 context management 前后结果差异巨大，例如 BrowseComp 42.1 → 57.4，这说明推理框架和上下文策略对最终分数影响很大，不能简单把结果归因于模型权重本身。

---

## 14. Mid-Training 消融实验

论文表 2 展示 progressive mid-training stages 对 downstream SFT 的影响。

| Benchmark | Base | Stage I Grounding | Stage I Planning | Stage II Agentic |
|---|---:|---:|---:|---:|
| BrowseComp | 34.74 | 36.61 | 36.97 | 40.44 |
| BrowseComp-ZH | 26.82 | 27.34 | 29.84 | 38.75 |
| HLE | 32.25 | 32.00 | 31.37 | 31.25 |
| GAIA | 77.43 | 76.70 | 80.83 | 79.13 |
| Average | 42.81 | 43.16 | 44.75 | 47.39 |

### 14.1 解读

- Grounding 对 BrowseComp 有正向作用，说明从噪声文档抽取意图相关证据很重要；
- Planning 对 GAIA 提升明显，说明 GAIA 更依赖任务拆解和规划；
- Stage II Agentic 对 BrowseComp-ZH 提升最大，26.82 → 38.75，表明长程环境交互对中文复杂搜索尤为重要；
- HLE 在这些阶段没有提升，甚至略降，说明 HLE 可能更多依赖领域知识或特殊推理，不一定受益于该 mid-training 配方。

这组消融证明 mid-training 不是装饰项，而是 REDSearcher 性能的重要来源。

---

## 15. RL 结果与搜索效率

论文 Figure 6 显示 RL 后继续提升。

关键数字：

- SFT 平均评估 reward：47.4；
- RL 后平均 reward：51.3，提升 +3.9；
- BrowseComp：39.4 → 42.1，提升 +2.7；
- 平均工具调用次数：100.6 → 90.1，下降 10.4%。

### 15.1 重要发现

REDSearcher 的 RL 不只是让模型“多搜”，反而让模型 **更少工具调用、更高 reward**。这与一些 interactive scaling 论文强调“更深交互带来更好效果”略有不同：REDSearcher 说明，当模型学会更好策略后，可能用更短路径完成任务。

这很有实践价值，因为真实产品不仅关心准确率，也关心：

- 工具调用成本；
- 响应延迟；
- 搜索 API 稳定性；
- 用户等待体验。

---

## 16. Tool-free vs Tool-enabled 分析

论文指出，最终准确率可能混合两种能力：

1. 模型靠参数记忆直接答对；
2. 模型靠工具搜索和证据综合答对。

因此它比较 tool-free 和 tool-enabled 设置。观察是：

- REDSearcher 在 tool-free 下分数较低；
- 开工具后提升显著；
- 一些强 baseline tool-free 也能答对不少题，可能来自预训练覆盖或 benchmark overlap。

这支持作者观点：评价 deep search agent 时，不能只看 final accuracy，也应看 **工具带来的增益**。如果一个模型不开工具也会很多答案，它的分数可能不完全代表 agentic search 能力。

---

## 17. 多模态实验结果

论文表 3 展示 REDSearcher-MM 的结果。

| 模型 | MM-BrowseComp | BrowseComp-VL | MMSearch-Plus | MMSearch | LiveVQA | HLE(text) | HLE-VL | BrowseComp | BrowseComp-ZH |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Qwen3-VL Thinking 30B | 10.7 | 37.1 | 11.0 | 59.7 | 64.8 | 8.8 | 8.7 | 0.2 | 7.2 |
| Qwen3-VL Thinking 235B | 12.1 | 43.1 | 17.4 | 63.3 | 70.2 | 14.5 | 14.1 | 0.3 | 18.6 |
| Vision-DeepResearch 30B | - | 53.7 | 28.5 | 69.6 | 77.6 | - | - | - | - |
| REDSearcher-MM-SFT | 25.3 | 55.3 | 20.2 | 70.3 | 78.5 | 24.4 | 24.2 | 30.1 | 43.1 |
| REDSearcher-MM-RL | 23.5 | 57.2 | 26.6 | 72.9 | 79.3 | 25.3 | 25.6 | 31.2 | 44.5 |

### 17.1 解读

- REDSearcher-MM 明显强于原始 Qwen3-VL agent baseline，说明多模态 agentic 训练有效；
- RL 后在 BrowseComp-VL、MMSearch-Plus、MMSearch、LiveVQA、文本迁移等多数指标提升；
- MM-BrowseComp 上 SFT 25.3 高于 RL 23.5，说明 RL 不是所有任务都稳定提升；
- 与 Vision-DeepResearch 相比，REDSearcher-MM 在 BrowseComp-VL、MMSearch、LiveVQA 有竞争力，但 MMSearch-Plus 低于 Vision-DeepResearch；
- 多模态训练还迁移到文本 benchmark，BrowseComp 31.2、BrowseComp-ZH 44.5，说明搜索/推理能力部分跨模态共享。

---

## 18. 多模态行为分析

论文进一步分析 REDSearcher-MM 的工具轮数、工具类型和 thinking pattern。

### 18.1 Turns Distribution

论文将 benchmark 分为简单和困难两类，观察到：

- 简单任务通常少量工具调用即可获得足够证据；
- 困难任务需要更多搜索轮次；
- 模型有时已经遇到正确证据，但因为置信度不足继续搜索；
- 在困难任务上 over-searching 更明显，很多样本接近 30-turn cutoff；
- RL 后工具调用轮数下降，尤其在较简单 benchmark 上更明显。

这说明多模态搜索 agent 也面临“何时停止”的核心难题。

### 18.2 Tool Category Distribution

不同 benchmark 激活不同工具模式：

- MMSearch 更偏 web search 和 webpage browsing；
- MM-BrowseComp 需要更多 text-search steps，因为证据更长程；
- MMSearch-Plus 更强调细粒度视觉感知，因此 zoom-in 和 image search 更多。

这说明多模态 deep search 不是简单“加一个图像输入”，而是需要根据任务动态组合视觉搜索、局部放大、网页访问和文本检索。

### 18.3 Thinking Patterns

论文把 thinking pattern 分成三类：

1. **Decomposition**：将复杂问题拆成可操作子问题；
2. **Reflection**：回看中间结论，识别缺失证据或不确定性，调整搜索计划；
3. **Verification**：提交最终答案前用额外证据交叉验证候选。

复杂 benchmark 中 decomposition、reflection、verification 更频繁，多模态任务中模型也更可能显式把视觉信息纳入 reasoning。

---

## 19. 与 MiroThinker / Tongyi 的关系

从当前目录里的几篇论文看，REDSearcher 与 MiroThinker、Tongyi DeepResearch 关注点不同。

| 论文 | 核心关键词 | 主要差异 |
|---|---|---|
| Tongyi DeepResearch | agentic mid-training + post-training + context management | 强调开源 30B-A3B deep research agent 的完整训练范式 |
| MiroThinker v1.0 | model/context/interactive scaling | 强调交互深度是第三个 scaling 维度 |
| MiroThinker-1.7 & H1 | effective interaction scaling + verification | 强调每步交互质量与 local/global verifier |
| REDSearcher | dual-constrained synthesis + cost-efficient training + simulation | 强调复杂任务可控合成和低成本训练环境 |
| S1-DeepResearch | beyond search, real-world deliverables | 强调从 closed QA 扩展到报告、文件、技能、指令跟随 |

REDSearcher 的独特性在于它把任务难度控制讲得更形式化：treewidth、MSD、工具强制约束、模拟环境，都是为了解决“如何规模化制造真正有训练价值的问题”。

---

## 20. 论文亮点

### 20.1 把问题难度从 hop 数推进到图结构

传统多跳 QA 常常只说 2-hop、3-hop，但 hop 数不能描述多变量耦合。REDSearcher 用 treewidth 解释为什么环、菱形、类 clique 问题更难，这比“多跳”更精确。

### 20.2 把证据分散作为显式目标

很多合成 QA 的问题是：虽然逻辑多跳，但答案可以通过单页搜索直接找到。REDSearcher 的 MSD 约束正面处理了这个问题。

### 20.3 强制工具使用，而不是期待模型自己探索

如果工具使用只靠 RL 稀疏奖励，样本效率很低。REDSearcher 通过 tool-enforced query evolution，让问题本身要求使用地图、学术检索、视觉等工具，显著增加工具调用学习信号。

### 20.4 Sim-to-real 的工程路线清晰

论文没有只依赖昂贵真实环境，而是构造本地模拟环境进行快速算法迭代。这对 agentic RL 非常重要，因为真实 web 环境不稳定、慢、贵、不可复现。

### 20.5 RL 让搜索更高效

许多 agent 论文展示 RL 后工具调用变多、交互更深；REDSearcher 的有趣点是 RL 后平均工具调用下降但性能上升，说明 RL 学到的是更优搜索策略，而不是单纯加长轨迹。

---

## 21. 局限与批判性阅读

论文没有单独列出 Limitations，但从内容可以总结几个风险。

### 21.1 Discard-all 可能丢失长期证据

当上下文过长时直接丢弃所有历史，虽然释放上下文，但可能导致：

- 已验证证据丢失；
- 候选排除过程丢失；
- 多源对齐进度丢失；
- agent 重新搜索导致成本增加；
- 对时间线、财务分析、法律对照等任务不够稳健。

更强的方案可能需要结构化 evidence memory 或 citation store。

### 21.2 合成问题仍可能与真实用户任务有分布差距

REDSearcher 的合成任务非常复杂，但很多是“谜题式”或“线索式”搜索 QA。真实用户的 deep research 可能更开放，例如市场分析、法律总结、实验方案设计。这与 S1-DeepResearch 论文提出的“beyond search”问题形成对照。

### 21.3 Benchmark 数字受推理策略影响很大

BrowseComp 使用 context management 前后差异巨大，说明分数不仅来自模型，还来自：

- 最大 context；
- 工具调用上限；
- 是否 restart / discard；
- 搜索 API 和 visit summarizer；
- judge 模型；
- 系统 prompt。

横向比较时需要确认这些条件是否一致。

### 21.4 多模态能力不是全面领先

REDSearcher-MM 很强，但并非所有多模态指标第一。例如 MMSearch-Plus 仍低于 Vision-DeepResearch，MM-BrowseComp RL 低于 SFT。这说明多模态 RL 和视觉工具组合仍有不稳定性。

### 21.5 工具强制可能带来“过度工具依赖”

Tool-enforced query 能教会模型主动用工具，但也可能让模型形成“必须调用工具”的偏好，即使某些问题可以直接回答或更少工具解决。论文中 RL 后工具调用减少是好迹象，但仍需要成本约束和停止策略。

---

## 22. 对实践的启示

如果要构建自己的长程搜索 agent，REDSearcher 提供了很实用的工程路线。

### 22.1 先定义任务难度，而不是盲目合成 QA

可以从以下维度控制数据：

- 推理图是否线性；
- 是否存在环/交叉约束；
- 必要证据需要几个来源；
- 是否存在单页 shortcut；
- 是否必须使用非文本工具；
- 是否唯一答案。

### 22.2 用本地模拟环境做 RL 风洞

真实 web RL 成本高且不稳定。可先构建：

- 本地搜索索引；
- 缓存网页库；
- 本地 visit summarizer；
- 可控 QA；
- 干扰文档；
- 与真实工具同接口的模拟 API。

先在模拟环境验证算法，再迁移到真实环境。

### 22.3 中间能力训练比直接 RL 更现实

直接让模型在真实环境中 RL，失败率高、成本大。更实际的路径是：

```text
grounding/planning 数据 -> 工具协议数据 -> 模拟长轨迹 -> 真实环境 SFT -> 真实/模拟 RL
```

### 22.4 评估 agent 要看成本和工具收益

建议同时报告：

- final accuracy；
- tool-free accuracy；
- tool-enabled gain；
- 平均工具调用次数；
- 平均 token；
- 平均延迟；
- 搜索失败率；
- 是否使用缓存或上下文重启。

---

## 23. 可复现/继续研究方向

1. **treewidth 与准确率关系实验**  
   按 `k=1/2/3+` 分组，观察模型准确率和工具调用数。

2. **MSD 消融**  
   控制图结构相同，仅改变证据来源数量，看多源分散对性能影响。

3. **Tool-enforced query 消融**  
   对比普通文本混淆、地图约束、学术约束、视觉约束分别带来的工具学习收益。

4. **Discard-all vs Recency Retention vs Evidence Memory**  
   比较不同上下文管理策略在长任务上的准确率、成本、重复搜索率。

5. **模拟环境到真实环境迁移**  
   系统评估 sim-to-real gap，查看哪些工具调用策略在真实 web 上失效。

6. **RL 成本约束**  
   在 reward 中加入工具调用惩罚，训练更短、更准的搜索策略。

7. **多模态停止策略**  
   针对 over-searching，训练模型识别何时视觉证据足够、何时继续 zoom/search。

---

## 24. 术语速查

| 术语 | 解释 |
|---|---|
| Deep Search | 多轮搜索、访问、验证、综合的长程信息查找任务 |
| ReAct | Thought/Action/Observation 交替的 agent 范式 |
| Treewidth | 衡量图结构纠缠程度的指标，高 treewidth 表示多变量耦合更强 |
| MSD | Minimum Source Dispersion，解决任务所需最少证据来源数 |
| Tool-Enforced Query | 把线索改写成必须通过工具解决的约束 |
| Intent-anchored Grounding | 根据当前搜索意图从噪声文档中抽取关键信息 |
| Hierarchical Planning | 将长程任务拆成当前明确目标和未来待消歧目标 |
| Functionally Equivalent Simulation | 接口类似真实工具、但本地低成本可控的模拟环境 |
| RLVR | Reinforcement Learning with Verifiable Rewards |
| GRPO | 组内相对优势策略优化方法 |
| Discard-all | 上下文过长时丢弃历史工具交互，从问题重新开始 |

---

## 25. 最终理解

REDSearcher 的核心价值在于，它把 long-horizon search agent 的训练问题拆成了三个可操作问题：

```text
如何制造足够难的问题？
如何低成本获得足够多的长程轨迹？
如何让模型在工具环境中更高效地搜索和验证？
```

它的答案是：用 **treewidth + MSD** 控制问题复杂度，用 **tool-enforced query** 强制工具学习，用 **mid-training** 提前训练 grounding/planning/tool-use，用 **本地模拟环境** 降低 RL 迭代成本，再用 **真实环境 SFT/RL** 提升最终表现。

这篇论文特别适合作为“如何规模化训练搜索型 agent”的工程蓝图。它没有把重点放在更复杂的 agent workflow，而是强调数据、环境和训练 recipe 的共同设计。相比只展示榜单分数的系统，REDSearcher 更值得借鉴的是它对任务难度、工具必要性和训练成本的系统化处理。
