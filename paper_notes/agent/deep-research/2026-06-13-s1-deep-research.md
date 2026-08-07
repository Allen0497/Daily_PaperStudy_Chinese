# S1-DeepResearch 论文详解

> 来源文件：`2026-06-13-s1-deep-research.pdf`  
> 论文题目：**S1-DeepResearch: Beyond Search, Toward Real-World Long-Horizon Research Agents**  
> 机构：XScience Lab，Wenge AI  
> 版本信息：arXiv:2606.15367v1，2026-06-13  
> 原文：[arXiv 摘要页](https://arxiv.org/abs/2606.15367) · [PDF](https://arxiv.org/pdf/2606.15367)  
> 项目资源：[GitHub](https://github.com/ScienceOne-AI/S1-DeepResearch) · [Hugging Face](https://huggingface.co/ScienceOne-AI/S1-DeepResearch-32B) · [数据集](https://huggingface.co/datasets/ScienceOne-AI/S1-DeepResearch-15k) · [ModelScope](https://modelscope.cn/models/ScienceOne-AI/S1-DeepResearch-32B)  
> 说明：本文档基于当前目录中的本地 PDF 阅读整理，未联网核验模型、数据集、代码或评测结果，也未独立复现实验。

---

## 1. 一句话总结

S1-DeepResearch 认为当前很多 deep research agent 其实仍然偏 **deep search**：会搜索、浏览、定位答案，但训练数据主要是 closed-ended QA 和信息定位，缺少真实研究任务中的报告写作、文件理解与生成、复杂指令约束、技能调用和开放式探索。论文提出一个统一的 agentic trajectory construction paradigm，将 **闭合答案 QA 的可验证性** 与 **开放式研究探索的真实性** 结合，构造覆盖五大能力维度的 S1-DeepResearch-15K 轨迹数据，并用它 SFT 出 S1-DeepResearch-32B。

---

## 2. 这篇论文的定位

当前目录中几篇论文都围绕 Deep Research Agent，但关注点不同：

| 论文 | 核心问题 |
|---|---|
| Tongyi DeepResearch | 如何通过 mid-training + SFT + RL 训练开源 deep research agent |
| MiroThinker v1.0 | 交互深度是模型/上下文之外的第三个 scaling 维度 |
| REDSearcher | 如何可控、低成本合成长程搜索任务与轨迹 |
| MiroThinker-1.7 & H1 | 如何让交互更有效，并用 verification 支撑 heavy-duty reasoning |
| S1-DeepResearch | 如何从 search-centric agent 扩展到真实 deep research workflow |

S1-DeepResearch 的关键词是：

```text
Beyond Search
```

它不是只追求 BrowseComp/GAIA 这类搜索 QA 分数，而是把真实研究场景拆成：

- 长链复杂推理；
- 深度研究指令跟随；
- 长篇研究报告生成；
- 文件理解与生成；
- 动态技能使用。

---

## 3. 论文要解决的问题

### 3.1 Deep Research 不等于 Deep Search

论文明确区分：

- **Deep Search**：侧重定位和验证信息，通常目标答案比较确定；
- **Deep Research**：不仅要找信息，还要构建分析框架、整合证据、处理冲突、形成报告或交付物。

Deep research 任务常见特征：

- 目标开放，可能有多个合理输出；
- 证据不完整或互相冲突；
- 需要引用、论证、组织和综合；
- 用户有复杂约束，例如来源、格式、字数、分析角度、假设条件；
- 可能包含上传文件、图像、视频、表格、代码、技能文档；
- 最终输出可能是报告、表格、文件、网页、PDF、图表，而不只是短答案。

### 3.2 现有训练数据的局限

论文认为现有 agentic trajectory 数据多偏 search-centric：

- 主要训练信息检索；
- 主要是 closed-ended QA；
- 最终答案短；
- 可以通过 answer correctness 过滤；
- 对报告写作、文件交付、复杂指令跟随和技能使用覆盖不足。

这类数据适合训练：

- query reformulation；
- web browsing；
- evidence localization；
- answer verification。

但不足以训练真实 deep research 所需的：

- evidence integration；
- knowledge synthesis；
- planning and decision making；
- structured report generation；
- artifact generation；
- skill orchestration。

### 3.3 核心瓶颈

论文把瓶颈定义为：**缺少既可规模化、又能忠实模拟真实研究流程的高质量 agentic trajectories。**

闭合 QA 的优点：

- 答案明确；
- 容易验证；
- 容易大规模过滤。

开放研究任务的优点：

- 更真实；
- 更接近用户需求；
- 需要报告、综合、决策和交付。

但开放任务难以：

- 自动合成；
- 自动验证；
- 控制难度；
- 判断唯一正确性。

因此 S1 的路线是：**把 closed-ended QA 和 open-ended exploration 合成进一个统一轨迹构造范式。**

---

## 4. 核心贡献

论文宣称三项贡献。

### 4.1 发布模型与数据

- **S1-DeepResearch-32B**：基于 Qwen3-32B 的 agentic model；
- **S1-DeepResearch-15K**：15K 高质量 agentic trajectories。

需要注意，论文表 2/3 中展示的构造后总体轨迹统计为 49,299 条，但摘要强调对外 release 的是 15K 高质量轨迹集合。可理解为完整构造池与公开发布子集之间存在筛选关系。

### 4.2 统一轨迹构造范式

提出三阶段数据构造系统：

```text
Graph-grounded task formulation
-> Agentic trajectory rollout
-> Multi-dimensional trajectory verification
```

该系统同时建模：

- information acquisition；
- knowledge synthesis；
- planning-oriented agent behaviors。

### 4.3 系统评测

在 20 个 benchmark、五大能力维度上评估，S1-DeepResearch-32B 在同规模开源模型中达到强结果，并在部分任务上接近 proprietary frontier models。

---

## 5. 与现有模型的能力覆盖对比

论文表 1 比较了不同 deep research model 是否明确覆盖多个能力。

能力列包括：

| 缩写 | 含义 |
|---|---|
| LHR-Text | 文本长程复杂推理 |
| LHR-MM | 多模态长程复杂推理 |
| Instr. | 深度研究指令跟随 |
| Report | 报告生成 |
| Doc. | 文档/文件理解与生成 |
| Skill | 动态技能调用 |

S1-DeepResearch 是表中唯一明确覆盖全部六类能力的模型：

- LHR-Text：✓；
- LHR-MM：✓；
- Instruction following：✓；
- Report generation：✓；
- Document understanding/generation：✓；
- Skill orchestration：✓。

训练 recipe 列显示 S1 当前主要使用 SFT，没有使用 mid-training 或 RL。这一点很关键：论文想证明 **高质量多能力轨迹数据本身就能显著提升 32B 模型**。

---

## 6. Agentic Data Construction System 总览

S1 的数据构造系统模拟人类研究者解决复杂问题的过程：

1. 构造复杂任务；
2. 让 agent 在工具环境中执行；
3. 对轨迹进行多维验证；
4. 只保留满足质量、约束和可验证性的样本。

整体分三阶段。

### 6.1 Phase I：Graph-Grounded Task Formulation and Complexity Evolution

用知识图谱/连接子图作为任务骨架，生成复杂 query。

关键操作：

- seed entity pool construction；
- graph construction；
- query generation；
- constraint injection；
- query evolution；
- topology-aware difficulty filtering。

### 6.2 Phase II：AgentLoop Execution and Trajectory Refinement

将任务放进 sandbox，通过 AgentLoop 执行真实工具交互，生成完整 trajectory。

工具包括九类：

- web search；
- web visit；
- image search；
- academic search；
- file parsing；
- code execution；
- bash；
- image QA；
- video QA。

然后对不同场景做 refinement，例如：

- report-oriented refinement；
- file-oriented refinement；
- multimodal reasoning refinement；
- skill-oriented refinement。

### 6.3 Phase III：Multi-Dimensional Trajectory Verification

对五类能力分别设计 verifier，检查：

- closed-form answer 是否正确；
- 报告引用是否支持 claim；
- 复杂指令约束是否满足；
- 生成文件是否可执行且语义符合；
- 技能是否正确调用且有效。

---

## 7. Phase I 详解：图驱动任务构造

### 7.1 Seed Entity Pool Construction

从 Wikipedia Entities 初始化 entity pool，通过 hybrid filtering 选择高质量 seed。

过滤流程：

1. **Topology and popularity filtering**  
   用 Sitelinks 数量和 2-hop neighbors 规模过滤掉太冷门或太泛的实体。

2. **Low-information-density filtering**  
   删除时间实体、纯数值实体等信息密度低的节点。

3. **Searchability verification**  
   要求候选实体能检索出足够有效网页，确保后续任务可在外部信息空间中解决。

4. **Safety filtering**  
   排除有害、敏感、高争议内容，降低偏见和不稳定检索风险。

目标是得到：

- 知识密度高；
- 语义清楚；
- 外部可检索；
- 可验证；
- 适合跨文档探索的 seed。

### 7.2 Subgraph Construction

对每个 seed entity，使用双路径扩展 DAG：

- **Wikidata path**：多跳 relation traversal，建立结构化实体关系；
- **Open-domain search path**：用搜索引擎补充开放网页知识，扩展语义连接。

论文还引入 multimodal information parsing：

- 文本层面：entity recognition、relation extraction、event summarization；
- 视觉层面：从网页图片、图表、场景中解析视觉实体，并与文本语义对齐。

然后使用 community detection 把大而稀疏的全局图切成语义紧密的 connected subgraphs，作为后续任务生成的结构基础。

### 7.3 Constraint Injection

开放研究任务如果没有约束，容易变成泛泛而谈。S1 在任务生成前注入九维约束。

九个约束维度来自论文 Appendix B：

| 约束 | 含义 |
|---|---|
| Source Constraints | 限定信息来源、学科领域、文献范围，保证证据质量和领域一致性 |
| Argumentation Constraints | 规定证据组织、多视角验证、结论完整性，提高论证严谨度 |
| Reasoning Constraints | 指定演绎、归纳、因果、比较等推理模式 |
| Objective Constraints | 定义核心研究目标、决策标准、评价要求，防止目标漂移 |
| Hypothetical Constraints | 引入假设、物理边界或逻辑条件，限定分析空间 |
| Output Format Constraints | 指定最终回答结构、格式、标记规范 |
| Output Scale Constraints | 控制覆盖范围、分析深度、生成规模 |
| Execution Constraints | 约束工具使用策略、行动顺序和资源预算 |
| Contextual Constraints | 引入时间范围、角色视角、应用场景，提高真实性 |

这一步是 S1 区别于 search QA 的关键：它让任务更像真实用户委托的研究项目，而不是一句谜题式问题。

### 7.4 Query/QA Generation

根据 connected subgraph 和约束，生成两类任务。

#### Open-Ended Query Generation

生成研究导向 query，要求：

- 多阶段分析；
- 使用子图中的实体、关系、视觉语义；
- 自然融入约束；
- 有明确研究边界；
- 有可执行目标；
- 有可追溯事实基础。

#### Closed-Form QA Generation

生成答案确定的 QA：

- 答案事实必须可追溯到节点、边或属性；
- 问题与答案要和图结构一致；
- 保持明确验证标准。

### 7.5 Semantics-Driven Query Evolution

对 closed-form QA，论文引入 iterative query evolution，增加复杂度但保持答案可达。

每轮随机选择 query 中的 semantic units，并做两类改写：

1. **Entity Semantic Rewriting**  
   将显式实体替换成上下文描述、功能角色、历史行为、关系表达。例如不直接说实体名，而说“曾在某事件中担任某角色的人”。

2. **Condition Semantic Generalization**  
   将精确时间、地点、数字、事件改成更抽象表达，例如时间段、空间区域、事件类型。

这降低直接关键词搜索成功率，迫使模型进行多跳推理。

### 7.6 Parametric Knowledge Filtering

为了避免模型不用工具、只靠参数知识答题，论文在 tool-free 设置下测试生成任务：

- 禁用搜索、解析等外部工具；
- 直接让 base model 回答；
- 如果能解决，认为样本复杂度低，删除。

这类似 REDSearcher 的 no-tool pre-filter，但 S1 用它服务于更广泛的 deep research 数据构造。

### 7.7 Topology-Aware Difficulty Estimation

Query evolution 后，原始子图可能不再准确表示最终问题的推理结构。因此 S1 为每个 evolved QA 构造 task-specific reasoning graph：

```math
G=(V,E)
```

节点代表：

- key entities；
- constraints；
- intermediate conclusions；
- final answers。

边代表：

- dependency；
- reasoning relations。

结构类型包括：

- sequential；
- convergent；
- divergent；
- comparative；
- graph-based reasoning。

然后从四个拓扑维度计算难度。

#### Information Flow Complexity

衡量中间节点的信息聚合与传播：

```math
C_{info}(G)=\sum_{v\in V\setminus\{v_s,v_t\}}(d^-(v)d^+(v))^2
```

入度和出度都高的节点表示跨路径依赖强。

#### Feedback Dependency Complexity

近似计算 feedback edges 数量，衡量循环依赖和非线性结构：

```math
C_{fb}(G)=|\hat E_{fb}|
```

反馈边越多，模型越需要在多条推理路径之间保持一致性。

#### Width Complexity

在去除反馈边后的 DAG 上，用 BFS 分层，取最大层宽：

```math
C_{width}(\tilde G)=\max_k |L_k|
```

宽度越大，代表并行分支越多。

#### Reasoning Depth Complexity

最长依赖链长度：

```math
C_{depth}(\tilde G)=\max_{\pi\in P(v_s,v_t)}|\pi|
```

深度越大，长链状态保持越难。

#### 综合难度分数

对四个指标做 Z-score normalization，再加权求和：

```math
D=\sum_{m\in M} w_m z_m
```

最后每个 batch 只保留 top 30% 最复杂样本进入 Phase II。

---

## 8. Phase II 详解：AgentLoop 执行与轨迹细化

### 8.1 Tool Environment

AgentLoop 提供九类 atomic tools。

| 工具 | 作用 |
|---|---|
| Web Search | 开放网页发现，识别来源、实体、术语、观点、后续方向 |
| Web Visit | 目标导向阅读网页，提取日期、事实、出处、立场等信息 |
| Academic Search | 学术出版物检索，支持文献综述、方法对比、科学主张验证 |
| File Parsing | 解析 PDF、slides、spreadsheets、docs、archives、transcripts 等 |
| Code Execution | 确定性计算、数据清洗、统计、绘图、验证 |
| Bash | 文件系统操作、命令行流程、脚本运行、技能执行 |
| Image Search | 检索图片、图表、截图、地图、视觉例子 |
| Image QA | 目标导向解读图像，例如读图表、识别对象、抽取证据 |
| Video QA | 解析视频中的动作、事件、过程和动态信息 |

这些工具被设计成通用 atomic actions，而不是固定 pipeline，让 agent 自己组合。

### 8.2 Tool-Interactive Trajectory Construction

给定任务 `x`，AgentLoop 将模型放入 sandbox 执行：

```math
\tau=(x,a_1,o_1,a_2,o_2,...,a_T,o_T,y)
```

其中：

- `a_t` 是模型 action 或 tool call；
- `o_t` 是环境 observation；
- `y` 是最终回答。

训练时：

- assistant-side reasoning/action/final answer 可作为监督信号；
- tool observations 被视为外部环境输出，不计算 loss。

### 8.3 Skill-Aware Rollout

S1 特别加入 skill-aware trajectory construction。

在这个设置中：

- sandbox 挂载目标 skill 和多个 distractor skills；
- agent 只看到轻量 metadata 和 `SKILL.md` 路径；
- agent 必须用 bash 等工具阅读相关文档；
- 根据文档调用底层脚本；
- 完成任务并总结结果。

这和真实工具/插件使用非常接近：模型不能凭空知道技能如何用，必须读说明、选择正确技能、避免干扰技能、按步骤执行。

### 8.4 Scenario-Specific Trajectory Refinement

AgentLoop 生成的是初始轨迹，S1 还根据任务类型细化。

#### Report-Oriented Refinement

用于深度研究报告：

- 评估已收集证据相关性；
- 从 query、约束、证据中推导报告 outline；
- 明确评价标准；
- 重写最终报告，提升覆盖、组织、逻辑一致性、格式合规；
- 刷新最后一个 Think step，让最终 reasoning state 与报告一致。

#### File-Oriented Refinement

用于文件理解与生成：

- 在原 query 中加入目标格式、schema、layout、内容组织要求；
- 将文本答案转成 `execute_code` 生成文件的步骤；
- 最终 response 总结生成文件和完成状态。

这让模型学会“交付文件”，而不是只写一段文本。

#### Multimodal Reasoning Refinement

用于多模态任务：

- 如果原轨迹依赖关键图片，就把图片插入 query；
- 如果依赖关键实体，就检索代表图片并组合到 query；
- 把文本问题重写成 native multimodal input。

#### Skill-Oriented Refinement

用于技能任务：

- 把 inline code/config/JSON/CSV/YAML/scripts 等内容转成附件；
- query 改写为引用上传文件；
- agent 需要识别文件类型和任务意图；
- 阅读 skill documentation；
- 执行相关工具；
- 总结结果。

---

## 9. Phase III 详解：多维轨迹验证

S1 的验证不只看最终答案，而是按任务类型检查轨迹质量。

### 9.1 Closed-form Complex Reasoning Verification

使用 LLM-as-a-Judge + reference answer alignment，检查：

- 最终答案是否与 reference 对齐；
- 推理轨迹逻辑是否正确；
- 中间推导是否完整；
- 是否存在错误传播。

比单纯 exact match 更细。

### 9.2 Deep Research Report Verification

报告任务重点检查 factual consistency 和 academic standards：

- in-text citation 是否与 reference entries 对应；
- citation chain 是否完整；
- cited evidence 是否语义支持对应 claim；
- 是否存在“引用看起来合法，但其实不支持论断”的 citation hallucination。

### 9.3 Instruction Following Verification

使用九维 constraint checker，检查轨迹是否满足：

- source；
- argumentation；
- reasoning；
- objective；
- hypothetical；
- output format；
- output scale；
- execution；
- contextual constraints。

任何约束违反都可能导致样本被过滤。

### 9.4 File Understanding and Generation Verification

重点检查：

- 多步生成 workflow 中，最后阶段是否包含 Python execution；
- 是否真的生成了文件；
- 文件结构、信息覆盖、呈现格式是否符合任务要求；
- 生成文件与原始需求语义是否一致。

### 9.5 Skill-use Verification

检查：

- target skill 是否被正确激活；
- 是否错误调用 distractor skill；
- skill module 对最终轨迹是否有贡献；
- 外部知识和可执行模块使用是否有效；
- 是否存在冗余、无效或错误调用。

---

## 10. 数据集统计与能力分布

### 10.1 与其他 agent trajectory datasets 对比

论文表 2 对比了 S1 与 REDSearcher、OpenSeeker、OpenResearcher 等数据集。

| Dataset | Samples | Tool Calls | Total Len. | Traj Think | Step Think | Answer Len. | Final Think | Tool Types | Tool Pool |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| REDSearcher Text | 10001 | 64.1 | 59890 | 10149 | 156 | 234 | 349 | 2.12 | 5 |
| REDSearcher MM | 5816 | 12.2 | 12830 | 3582 | 272 | 236 | 753 | 3.27 | 6 |
| OpenSeeker All | 11677 | 46.1 | 73835 | 19181 | 408 | 349 | 579 | 1.94 | 2 |
| OpenResearcher | 97630 | 52.6 | 55090 | 5628 | 105 | 214 | 617 | 2.80 | 4 |
| S1-DeepResearch | 49299 | 9.7 | 20431 | 2524 | 273 | 1739 | 552 | 1.92 | 9 |

### 10.2 关键差异

S1 与搜索型数据集相比：

- 平均工具调用更少；
- 最终答案更长；
- 工具池更大；
- 输出更偏“研究交付物”；
- 不追求无限搜索，而追求完成真实任务。

论文强调：OpenSeeker 的错误轨迹甚至比正确轨迹工具调用更多、推理链更长，说明 **更长搜索不一定更好**。S1 要训练的是“何时停止搜索、如何组织证据、如何满足约束、如何生成可用结果”。

### 10.3 S1 内部六类数据统计

论文表 3：

| Category | Samples | Tool Calls | Total Len. | Traj Think | Step Think | Answer Len. | Final Think | Tool Types | Tool Pool |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Long-Horizon Reasoning Text | 18246 | 10.7 | 18830 | 2916 | 249 | 879 | 398 | 1.93 | 4 |
| Long-Horizon Reasoning MM | 5203 | 8.8 | 21258 | 1167 | 127 | 363 | 214 | 2.96 | 8 |
| Deep Research Report Generation | 5258 | 23.5 | 51625 | 3904 | 178 | 8921 | 1241 | 1.29 | 4 |
| Deep Research Instruction Following | 7877 | 3.9 | 16213 | 2050 | 422 | 1626 | 786 | 1.42 | 6 |
| Skill Using | 4208 | 3.7 | 12706 | 2255 | 450 | 1402 | 646 | 1.94 | 8 |
| File Understanding & Generation | 8507 | 8.0 | 11803 | 2234 | 245 | 257 | 401 | 2.10 | 6 |
| Overall | 49299 | 9.7 | 20431 | 2524 | 273 | 1739 | 552 | 1.92 | 9 |

### 10.4 三类轨迹模式

论文总结出三种模式：

1. **Output-intensive trajectories**  
   报告生成工具调用最多、总长度最长、答案最长，说明它结合了外部证据和长文输出构造。

2. **Decision-intensive trajectories**  
   指令跟随和技能使用工具调用少，但 step-level thinking 长，说明重点在约束理解、动作选择和执行控制。

3. **Tool-heterogeneous trajectories**  
   多模态任务工具类型最多；文件任务最终文本短，但大量工作发生在解析、执行和文件生成中。

---

## 11. Training：SFT 训练

S1-DeepResearch 从 **Qwen3-32B** 初始化。训练只使用 supervised fine-tuning。

### 11.1 轨迹表示

```math
\tau=(x,a_1,o_1,a_2,o_2,...,a_T,y)
```

其中：

- `x`：用户任务；
- `a_t`：模型生成的 reasoning step 或 tool action；
- `o_t`：环境 observation；
- `y`：最终 response。

### 11.2 Loss

训练监督 assistant-side actions 和 final answer，不对 observation 计算 loss：

```math
L=-\sum_{t=1}^{T}\log p_\theta(a_t|x,a_{<t},o_{<t}) - \log p_\theta(y|x,a_{\le T},o_{\le T})
```

这鼓励模型学习：

- 如何基于累积 observation 选择动作；
- 何时继续工具调用；
- 如何最终综合证据生成回答或交付物。

### 11.3 与其他论文训练 recipe 的对比

S1 当前没有使用 mid-training 或 RL。这既是局限，也是论文想强调的数据价值：

- 若只靠 SFT 就能大幅提升，说明多能力高质量轨迹很有用；
- 但在 BrowseComp、HLE 等高难 closed-form deep search 上，论文也承认还需要 RL。

---

## 12. 实验设置

### 12.1 五大能力维度

S1 评估覆盖五类 deep research 能力。

#### Long-horizon complex reasoning

文本：

- BrowseComp；
- BrowseComp-ZH；
- GAIA text-only；
- Humanity's Last Exam text-only；
- xBench-DeepSearch。

多模态：

- LiveVQA；
- MM-Search；
- BrowseComp-VL；
- RealXBench；
- MM-BrowseComp；
- HLE-VL。

#### Deep research report generation

- DeepResearch Bench；
- DeepResearch Bench II；
- ResearchRubrics。

#### Deep research instruction following

- ComplexBench；
- DeepResearchIF（自建）。

#### File understanding and generation

- GAIA attachment subset；
- GTA；
- FileSys（自建）。

#### Dynamic skill utilization

- SkillsUse（自建）。

### 12.2 自建 benchmark

#### FileSys

454 个样本，评估模型从自然语言请求生成可用文件，覆盖：

- DOCX；
- PDF；
- HTML；
- XLSX；
- SVG diagrams；
- data visualizations；
- structured/visual artifacts。

指标：

- CodeExc：代码执行和文件创建是否成功；
- FileAns：生成 artifact 是否语义满足任务。

#### DeepResearchIF

900 个样本，覆盖 general、scientific、industrial 场景。评估是否满足研究型复杂约束，包括：

- task scope；
- source selection；
- evidence usage；
- analytical methods；
- assumptions；
- reasoning procedures；
- long-form report generation。

指标：

- strict sample-level accuracy；
- 9 类顶层约束、26 个细粒度约束的 macro-average。

#### SkillsUse

400 个 queries，分 No-attachment 与 Attachment。评估：

- 是否发现相关 skill；
- 是否阅读 skill documentation；
- 是否避开 distractors；
- 是否遵循 skill-specific procedure；
- 是否完成 skill-guided execution。

每条轨迹按 Result、Execution、Skill Usage 三个维度、12 个细粒度指标 judge。

### 12.3 Evaluation Settings

论文称尽可能用统一 agent environment。外部交互任务提供相同 atomic tools：

- web search；
- web visit；
- image search；
- academic search；
- file parsing；
- code execution；
- bash；
- image QA；
- video QA。

统一推理设置：

| 参数 | 值 |
|---|---|
| temperature | 0.85 |
| top-p | 0.95 |
| repetition penalty | 1.1 |
| max tool calls | 150 |
| max context length | 128K tokens |

---

## 13. 文本长程复杂推理结果

论文表 4 中 S1-DeepResearch：

| 模型 | Size | Train | GAIA Text | BrowseComp | BrowseComp-ZH | xBench-DeepSearch | HLE Text | Overall |
|---|---:|---|---:|---:|---:|---:|---:|---:|
| Qwen3-32B | 32B | - | 30.2 | 3.2 | 7.3 | 39.0 | 9.3 | 17.8 |
| Qwen3-235B | 235B | - | 33.0 | 11.5 | 13.5 | 45.0 | 11.8 | 23.0 |
| Tongyi-DeepResearch | 30B | Mid/SFT/RL | 70.9 | 43.4 | 46.7 | 75.0 | 32.9 | 53.8 |
| MiroThinker-1.7-mini | 30B | Mid/SFT/RL | - | 67.9 | 72.3 | - | 57.2 | - |
| S1-DeepResearch | 32B | SFT | 72.8 | 36.7 | 48.4 | 79.3 | 30.3 | 53.5 |

### 13.1 解读

S1 相比 Qwen3-32B 提升巨大：

- GAIA：30.2 → 72.8；
- BrowseComp：3.2 → 36.7；
- BrowseComp-ZH：7.3 → 48.4；
- xBench-DeepSearch：39.0 → 79.3；
- HLE：9.3 → 30.3。

这说明 SFT 多能力 agentic trajectories 明显提升搜索、工具、推理能力。

但 S1 在 BrowseComp 和 HLE 上低于 MiroThinker-1.7-mini，也低于部分使用 RL 的专门 search agents。论文也承认：这些高难 closed-form search benchmark 可能需要进一步 RL。

---

## 14. 长篇研究报告结果

论文表 5：

| 模型 | DeepResearchBench | DeepResearchBench II | ResearchRubrics | Overall |
|---|---:|---:|---:|---:|
| Qwen3-32B | 36.0 | 27.8 | 41.7 | 35.2 |
| Qwen3-235B | 38.4 | 29.7 | 43.9 | 37.3 |
| Qwen3.5-397B | 45.7 | 44.2 | 58.6 | 49.5 |
| GLM-5 | 46.9 | 44.5 | 63.4 | 51.6 |
| UniScientist | 46.0 | 48.0 | 59.9 | 51.3 |
| S1-DeepResearch | 46.5 | 41.7 | 58.7 | 48.7 |

### 14.1 解读

- S1 大幅超过 Qwen3-32B 和 Qwen3-235B；
- 与 397B/744B/1T 级别模型接近；
- 低于 GLM-5 overall 51.6 和 UniScientist 51.3；
- 说明 32B 模型通过报告轨迹 SFT 可以获得不错的 report workflow 能力，但不是全局最佳。

论文强调，研究报告写作能力不完全由模型规模决定，还取决于是否学过“从证据收集到报告构造”的完整流程。

---

## 15. 深度研究指令跟随结果

论文表 6：

| 模型 | DeepResearchIF Query Acc | DeepResearchIF Constraint Macro | ComplexBench | Overall |
|---|---:|---:|---:|---:|
| Qwen3-32B | 4.1 | 28.9 | 77.0 | 40.6 |
| Qwen3-235B | 7.4 | 54.9 | 79.8 | 43.6 |
| DeepSeek-V3.2 | 25.8 | 76.0 | 83.9 | 54.8 |
| GPT-5.2 | 56.4 | 89.2 | 86.7 | 71.6 |
| S1-DeepResearch | 25.2 | 74.3 | 83.1 | 54.2 |

### 15.1 解读

相比 Qwen3-32B：

- Query-Level Accuracy：4.1 → 25.2；
- Constraint-Level Macro：28.9 → 74.3。

这说明九维 constraint injection 与 instruction-oriented trajectory 确实提升了长程约束保持能力。

但与 GPT-5.2 仍有较大差距，尤其 query-level accuracy 25.2 vs 56.4。复杂指令跟随仍是难点。

---

## 16. 文件理解与生成结果

论文表 7：

| 模型 | GAIA File | GTA | FileSys | Overall |
|---|---:|---:|---:|---:|
| Qwen3-32B | 24.2 | 70.3 | 44.7 | 46.4 |
| Qwen3.5-397B | 67.7 | 73.3 | 53.3 | 64.8 |
| GLM-5 | 70.7 | 72.7 | 64.3 | 69.2 |
| Doubao-2.0-Pro | 71.0 | 75.0 | 77.5 | 74.5 |
| S1-DeepResearch | 62.9 | 68.0 | 69.3 | 66.7 |

### 16.1 解读

- S1 大幅超过 Qwen3-32B；
- FileSys 69.3 高于 GLM-5 的 64.3 和 Qwen3.5-397B 的 53.3；
- GAIA File 和 GTA 不是最强；
- 说明 S1 的 file-oriented refinement 对生成 artifact 很有效。

FileSys 强说明 S1 不只是读取文件，也学会“生成可执行交付物”。

---

## 17. Dynamic Skill Utilization 结果

论文表 8：

| 模型 | w/ attachment | w/o attachment | Overall |
|---|---:|---:|---:|
| Qwen3-32B | 45.6 | 44.3 | 44.9 |
| Qwen3.5-397B | 63.6 | 68.5 | 66.0 |
| DeepSeek-V3.2 | 70.1 | 71.6 | 70.8 |
| Gemini-3.1-Pro | 70.9 | 78.1 | 74.5 |
| S1-DeepResearch | 69.7 | 71.7 | 70.1 |

### 17.1 解读

- S1 从 Qwen3-32B 的 44.9 提升到 70.1；
- 接近 DeepSeek-V3.2 的 70.8；
- 低于 Gemini-3.1-Pro 的 74.5；
- 表明 skill-oriented rollout 对“阅读说明、选择技能、避免 distractor、执行流程”很有效。

---

## 18. Test-Time Scaling 分析

论文 Figure 7 比较 Pass@1 与 Pass@3。

关键值：

| Benchmark | Pass@1 | Pass@3 |
|---|---:|---:|
| GAIA Text | 72.8 | 85.4 |
| BrowseComp | 34.9 | 47.6 |
| BrowseComp-ZH | 48.4 | 69.2 |
| HLE Text | 30.3 | 40.1 |
| xBench-DeepSearch | 79.3 | 93.0 |
| Overall | 53.3 | 67.1 |

### 18.1 解读

S1 有较强 test-time scaling：多次采样/多条 rollout 能显著提升成功率。

原因可能是：

- 不同 rollout 探索不同检索入口；
- 某次失败可能来自搜索路径不好，而不是模型完全不会；
- Pass@3 可以从多个候选中覆盖更完整证据；
- closed-ended deep search 很适合用多路径探索提高召回。

但 Pass@3 意味着成本约 3 倍，实际产品需要看 cost-quality tradeoff。

---

## 19. Parameter Efficiency 分析

论文 Figure 8 强调：S1 用 32B backbone 达到与数百 B 到 1T 参数模型相近的 deep research performance region，约用 30× 更少参数。

论文将其归因于：

- 多维 agentic trajectories；
- 覆盖真实 deep research workflow；
- 从 search-centric data 转向 research-oriented task completion；
- 不只训练搜索，也训练报告、文件、技能、约束。

这说明在 agent 场景中，**数据结构和任务覆盖** 对性能影响可能不亚于参数规模。

---

## 20. 多模态评估讨论

S1 不是原生多模态模型，而是 text-centric LLM + 外部视觉工具。

流程是：

```text
图像/视频 -> 外部视觉理解工具 -> 文本 observation -> LLM 推理和搜索
```

论文表 9：

| 模型 | LiveVQA | MM-Search | BrowseComp-VL | RealXBench | MM-BrowseComp | HLE-VL |
|---|---:|---:|---:|---:|---:|---:|
| Qwen3-32B | 50.3 | 35.1 | 26.8 | 28.4 | 7.1 | 11.4 |
| Qwen3.5-397B | 75.7 | 55.6 | 45.4 | 32.0 | 27.2 | 18.7 |
| Vision-DeepResearch | 77.6 | 69.6 | 53.7 | - | - | - |
| REDSearcher-MM-SFT | 78.5 | 70.3 | 55.3 | - | 25.3 | 24.2 |
| S1-DeepResearch | 67.7 | 54.4 | 39.1 | 31.4 | 19.2 | 15.2 |

### 20.1 解读

S1 可以处理多模态任务，但与原生或专门训练的多模态研究系统仍有差距。原因：

- 视觉信息先被转成文本 observation，细粒度视觉 grounding 损失；
- 空间关系、图像局部线索、跨模态对齐能力有限；
- tool-augmented perception 不等同于 end-to-end multimodal representation learning。

论文也将“Gap to Native Multimodal Reasoning”列为局限。

---

## 21. Case Studies

论文附录/案例展示覆盖五类能力。

### 21.1 报告生成

SME forensic accounting 案例中，模型把开放式专业请求组织成：

- 财务报表诊断；
- 早期预警指标；
- 现金流预测；
- 干预策略；
- 后续会计控制。

说明模型能把多领域知识整理成专业分析框架。

### 21.2 动态技能使用

TiO2 surface slab construction 案例中，模型遵循材料建模流程：

- surface selection；
- slab generation；
- thickness/vacuum configuration；
- structural verification。

说明它能结合科学知识和 executable skill。

### 21.3 文件理解与生成

几何题案例中，模型先推导圆半径，再生成 HTML 页面和 PDF，包含：

- step-by-step explanation；
- labeled diagrams；
- highlighted final answer。

说明它能从 reasoning 转化为 polished artifact。

### 21.4 指令跟随

可持续建筑书单与 IT 支持培训计划案例展示模型能维护：

- 主题约束；
- 年份约束；
- 固定数量；
- metadata 字段；
- Markdown 表格；
- 必需章节；
- 长文结构；
- 训练/练习/评估/资源计划。

### 21.5 长程复杂推理

案例包括：

- 叶酸发现与结晶时间线；
- 铁路历史与鸟类迁徙路线；
- 多模态地理定位与附近推荐。

展示模型能跨时间、领域和模态整合线索。

---

## 22. 论文局限

论文明确列出三类限制。

### 22.1 Limited Coverage of Coding-Oriented Tasks

S1 面向 deep research，不重点训练 coding agent 能力，例如：

- iterative program synthesis；
- execution-based debugging；
- repository-level code understanding；
- software engineering task completion。

这些任务需要更强代码环境反馈、程序验证、repo-scale context。未来可扩展到 coding-centric agent。

### 22.2 Gap to Native Multimodal Reasoning

S1 通过外部视觉工具将图像转成文本 observation，再交给 LLM 推理。这种方式在：

- 细粒度视觉感知；
- 空间理解；
- 直接跨模态 reasoning；
- visual geo-localization；
- subtle visual cues。

方面不如原生多模态模型。

### 22.3 Training Recipe 仍可增强

S1 当前主要用 SFT，没有 mid-training、RL、online preference optimization。论文承认未来要探索：

- agentic reinforcement learning；
- online preference optimization；
- long-horizon feedback；
- adaptive tool-use training。

这也解释了它在 BrowseComp/HLE 等 search-intensive closed-form benchmark 上仍落后于使用 RL 的模型。

---

## 23. 额外批判性阅读

### 23.1 “15K” 与表格“49,299”需区分

摘要说 release S1-DeepResearch-15K，但表 2/3 分析了 49,299 trajectories。阅读时应区分：

- 完整构造/统计数据池；
- 公开发布的 15K 高质量子集；
- 实际训练使用数据是否完全等同公开数据。

论文没有在摘要附近完全展开二者关系，复现时需要看数据发布说明。

### 23.2 只用 SFT 的优势与限制

S1 的优点是证明数据质量很强；限制是缺少 RL 后的在线探索优化。对于 closed-form deep search：

- SFT 学到“专家轨迹分布”；
- 但搜索失败后自适应策略、停止策略、工具错误恢复可能仍需 RL。

### 23.3 自建 benchmark 的外部可比性

FileSys、DeepResearchIF、SkillsUse 很有价值，但自建 benchmark 也需要关注：

- 是否公开完整数据和评测脚本；
- judge 是否稳定；
- 是否存在对 S1 数据构造偏好的匹配；
- 是否有人工复核；
- 其他模型是否同工具同环境评测。

### 23.4 Report 评价不等同实际事实可靠性

研究报告评估通常依赖 LLM judge 或 rubrics。即使分数高，也要单独考察：

- claim-level factuality；
- citation support；
- source reliability；
- hallucinated references；
- missing counter-evidence。

### 23.5 多模态工具链可能成为瓶颈

S1 的多模态能力取决于外部 image/video QA tools。如果这些工具误读图像，LLM 后续推理无法恢复。原生多模态训练仍是未来方向。

---

## 24. 对实践的启示

### 24.1 Deep research 产品不能只训练搜索 QA

真实用户常要的是：

- 一份报告；
- 一个表格；
- 一个 PDF；
- 一个数据分析文件；
- 一套执行方案；
- 一个技能驱动的技术产物。

因此训练数据要覆盖“交付物生成”，而不是只覆盖“找到答案”。

### 24.2 约束注入是构造真实任务的关键

如果要合成深度研究任务，应系统注入：

- 来源约束；
- 论证约束；
- 推理约束；
- 输出格式；
- 输出规模；
- 执行预算；
- 角色/场景。

否则模型会学到泛泛总结，而不是严格满足任务要求。

### 24.3 训练 skill use 要模拟真实插件使用

简单告诉模型“调用某工具”不够。更真实的轨迹应包含：

- 多个 distractor skills；
- 只暴露 metadata；
- 让模型读 `SKILL.md`；
- 根据说明执行脚本；
- 处理附件；
- 检查结果。

### 24.4 文件生成要验证 artifact，而不是只看文本

FileSys 的思路很重要：

- 先看代码是否执行；
- 再看文件是否生成；
- 再看文件语义是否符合；
- 最后看呈现质量。

这比让模型口头说“我已生成文件”可靠得多。

---

## 25. 可继续研究方向

1. **SFT + RL 对比**  
   在同一 S1 数据上加入 agentic RL，观察 BrowseComp/HLE 和 report/file/skill 是否继续提升。

2. **开放式任务 verifier 可靠性**  
   对 report、instruction、file、skill 的 verifier 做人工校准，估计误判率。

3. **多模态原生模型训练**  
   将 S1 的 multimodal trajectory construction 用到 VLM/MLLM，而不是 text LLM + visual tools。

4. **成本敏感数据构造**  
   在 trajectory 中加入 token/tool/time cost，训练模型更高效完成研究任务。

5. **Report claim-level evidence store**  
   训练模型将每个 claim 显式链接到 evidence snippets，减少 citation hallucination。

6. **Skill library scaling**  
   扩大技能数量，研究 skill discovery、skill routing、skill composition、distractor robustness。

7. **真实企业/科研任务评估**  
   用真实用户报告、文件生成、数据分析、文献综述任务检验 S1 是否能从 benchmark 泛化。

---

## 26. 术语速查

| 术语 | 解释 |
|---|---|
| Deep Search | 长程检索与答案定位，通常答案较确定 |
| Deep Research | 包含搜索、分析、综合、报告、文件、技能和交付物的完整研究流程 |
| Agentic Trajectory | 用户任务、模型动作、工具观察、最终回答组成的轨迹 |
| Graph-Grounded Task Formulation | 以知识图/子图为任务骨架生成复杂任务 |
| Constraint Injection | 在任务生成前注入来源、论证、格式、执行等约束 |
| Query Evolution | 通过实体改写和条件泛化增加 closed-form QA 难度 |
| AgentLoop | 在 sandbox 中驱动工具交互轨迹生成的执行框架 |
| Scenario-Specific Refinement | 针对报告、文件、多模态、技能任务进行轨迹重写和增强 |
| Multi-Dimensional Verification | 按任务类型检查答案、引用、约束、文件和技能使用 |
| FileSys | 自建文件理解与生成 benchmark |
| DeepResearchIF | 自建深度研究指令跟随 benchmark |
| SkillsUse | 自建动态技能调用 benchmark |

---

## 27. 最终理解

S1-DeepResearch 的核心价值在于，它把 deep research agent 从“会搜索答案”推进到“会完成研究任务”。论文最重要的判断是：

```text
真实 deep research 的终点不是找到一个短答案，而是交付可信、结构化、符合约束、可执行的研究结果。
```

因此 S1 的数据构造不只追求更多工具调用或更长搜索轨迹，而是围绕五类真实能力构造可验证轨迹：长链推理、报告写作、复杂指令、文件生成、技能使用。

这篇论文适合用来提醒所有 deep research 系统开发者：如果训练数据只包含 BrowseComp 式 closed QA，模型可能会变成强 search agent，但不一定能成为真正的 research agent。真正的 deep research 需要把“找信息”与“组织证据、满足约束、生成交付物、调用技能”放到同一个训练和评测框架中。
