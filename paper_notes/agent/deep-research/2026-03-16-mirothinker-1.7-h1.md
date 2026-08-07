# MiroThinker-1.7 & H1 论文详解

> 来源文件：`2026-03-16-mirothinker-1.7-h1.pdf`  
> 论文题目：**MiroThinker-1.7 & H1: Towards Heavy-Duty Research Agents via Verification**  
> 作者：MiroMind Team  
> 版本信息：arXiv:2603.15726v1，2026-03-16  
> 原文：[arXiv 摘要页](https://arxiv.org/abs/2603.15726) · [PDF](https://arxiv.org/pdf/2603.15726)  
> 项目资源：[在线服务](https://dr.miromind.ai) · [MiroThinker GitHub](https://github.com/MiroMindAI/MiroThinker) · [MiroFlow GitHub](https://github.com/MiroMindAI/MiroFlow) · [Hugging Face](https://huggingface.co/miromind-ai/MiroThinker-1.7)  
> 说明：本文档基于当前目录中的本地 PDF 阅读整理，未联网核验外部链接、榜单、模型权重或服务状态，也未独立复现实验结果。

---

## 1. 一句话总结

这篇论文是 MiroThinker v1.0 之后的升级报告，提出 **MiroThinker-1.7** 和旗舰系统 **MiroThinker-H1**。MiroThinker-1.7 的重点是通过 agentic mid-training 提升每一步交互的“原子能力”，让模型用更少、更有效的交互完成复杂任务；MiroThinker-H1 则在 1.7 基础上加入 **Local Verifier** 和 **Global Verifier**，把验证机制嵌入长程推理过程，形成面向高难任务的 heavy-duty reasoning mode。

---

## 2. 与 MiroThinker v1.0 的关系

当前专题中已有 `2025-11-18-mirothinker.pdf`，那篇 v1.0 技术报告强调：

```text
Model Scaling + Context Scaling + Interactive Scaling
```

即模型大小、上下文长度、交互深度共同提升研究智能体能力。

这篇 `MiroThinker-1.7 & H1` 则进一步修正和深化了 interactive scaling 的概念：

- v1.0 更强调“更多、更深的 agent-environment interaction 会提升效果”；
- 1.7/H1 强调“交互次数本身不是目标，关键是每一步是否有效”；
- H1 进一步强调“复杂任务中必须引入验证，否则长链路会累积错误”。

可以把演进理解为：

```text
MiroThinker v1.0：交互深度是第三个 scaling 维度
MiroThinker-1.7：提升每一步交互质量，实现 effective interaction scaling
MiroThinker-H1：通过 local/global verification 把有效交互升级为 heavy-duty reasoning
```

---

## 3. 论文要解决的问题

### 3.1 长程推理不是越长越好

论文指出，很多真实任务，例如科学分析、金融研究、开放网页研究，需要：

- 长链推理；
- 迭代信息收集；
- 多源证据验证；
- 中间结论检查；
- 最终答案的证据支撑。

但如果模型每一步规划、检索、总结、工具调用都不可靠，那么增加交互次数可能带来反效果：

- 错误中间结论被后续步骤放大；
- 噪声网页被不断纳入推理；
- 搜索路径越来越偏；
- 工具调用失败积累；
- 长推理链重复且不可读。

因此论文提出：**要扩展的是有效交互，而不是机械延长轨迹。**

### 3.2 有效交互依赖两个条件

论文把有效长程推理拆成两个核心条件：

1. **强 step-level atomic capabilities**  
   每一步都要会规划、会推理、会总结、会正确调用工具。

2. **可验证机制**  
   推理过程中要能检查中间决策，最终答案前要能审计整条证据链。

MiroThinker-1.7 主要解决第一点；MiroThinker-H1 主要解决第二点。

---

## 4. 核心贡献

### 4.1 MiroThinker-1.7：强化每一步 agentic atomic capability

论文引入 agentic mid-training，专门增强：

- planning；
- reasoning；
- tool use；
- answer summarization。

它不是只训练完整轨迹模仿，而是单独训练冷启动规划、上下文条件下的单步 reasoning、阶段性 summarization 等原子动作。

### 4.2 MiroThinker-H1：verification-centric heavy-duty reasoning

H1 在 1.7 上加入：

- **Local Verifier**：在中间步骤检查规划、工具调用、假设更新等局部决策；
- **Global Verifier**：在最终回答前审计完整证据链，选择最有证据支撑的候选答案。

### 4.3 双 pipeline QA 构造

论文使用两条互补 QA 合成路线：

- Corpus-based Pipeline：高吞吐、大覆盖，用文档子图合成多跳 QA；
- WebHop Pipeline：基于开放网页扩展、结构化多跳图和分层验证，构造更可控、更难、更贴近真实 web 的任务。

### 4.4 四阶段训练流程

MiroThinker-1.7 训练分为：

```text
Agentic Mid-training -> Agentic SFT -> Agentic Preference Optimization -> Agentic RL
```

### 4.5 强实验表现

论文报告 H1 在 BrowseComp、BrowseComp-ZH、GAIA、SEAL-0、FrontierSci-Olympiad、FinSearchComp 等任务上达到强结果，尤其在搜索密集和专业领域任务上表现突出。

---

## 5. Agentic Workflow：双循环 ReAct 架构

MiroThinker-1.7 仍基于 ReAct，但增加了上下文管理、工具调用修正和 episode restart。

### 5.1 Step Loop

在 episode `e` 的第 `t` 步，轨迹为：

```math
H_t^{(e)} = \{(T_1,A_1,O_1), ..., (T_{t-1},A_{t-1},O_{t-1})\}
```

其中：

- `T_i`：thought；
- `A_i`：action/tool call；
- `O_i`：observation/tool result。

与 v1.0 类似，模型不直接读取完整原始轨迹，而是通过 context operator `Φ_t` 转成可放入上下文窗口的 effective context。

### 5.2 Sliding-window observation retention

论文定义最近 `K` 个 step 的索引集合：

```math
S_t(K)=\{i \in \{1,...,t-1\} \mid i \ge t-K\}
```

对 observation 使用截断/省略：

```math
Φ_t(O_i)=
\begin{cases}
Trunc_L(O_i), & i \in S_t(K) \\
\emptyset, & otherwise
\end{cases}
```

有效上下文保留：

- 完整 thought trace；
- 完整 action trace；
- 最近 `K` 个 observation；
- 每个 observation 最多 `L` tokens；
- 旧 observation 省略。

实验中 `K=5`。

### 5.3 Result Truncation

对 `run_command`、`run_python_code` 等可能产生长输出的工具，按 `L` 截断，并加入 `[Result truncated]` 标记。

意义：

- 防止单个工具输出撑爆上下文；
- 允许模型知道结果被截断；
- 模型可在下一步发起更精确的 follow-up。

### 5.4 Episode Loop 与 Restart Policy

MiroThinker-1.7 不仅有 step loop，还有 episode loop。

如果一个 episode 达到最大轮数 `Tmax` 仍没有有效 final answer，或者 final-answer format 错误，则：

- 丢弃该 episode 历史；
- 从原始 query 重新开始；
- 最多重试 `Rmax` 次。

论文实验中 `Rmax=5`。

最后一轮不再无限推迟答案，如果再次达到 `Tmax`，会尝试给出答案，并回退到轨迹中的 best intermediate answer。

### 5.5 为什么要 episode restart？

长轨迹可能出现：

- 上下文越来越脏；
- 搜索路径跑偏；
- 旧错误假设污染后续决策；
- 工具失败或格式问题无法恢复。

清空上下文重新开始，相当于让模型用不同路径再试一次。它和 H1 的 verifier 结合后，可以在候选路径之间选择更可靠答案。

---

## 6. Tool Interface：工具接口

MiroThinker-1.7 的工具分三类。

### 6.1 Information Retrieval

| 工具 | 功能 |
|---|---|
| `google_search` | 基于 Google 后端返回标题、URL、snippet 等结构化搜索结果 |
| `scrape_and_extract_info` | 访问指定 URL，并按 agent 给定目标抽取任务相关信息 |

`scrape_and_extract_info` 具有多级 fallback pipeline，Jina 是主要 scraping backend。抓到页面后，轻量 LLM 会把原始网页压缩成任务相关证据，避免主模型直接处理超长网页。

这本质上是工具端 context compression。

### 6.2 Code Execution

使用 E2B Linux sandbox：

- `create_sandbox`；
- `run_command`；
- `run_python_code`。

支持：

- 文件 I/O；
- 数值计算；
- 数据处理；
- 程序验证；
- 命令行操作。

### 6.3 File and Data Transfer

| 工具 | 功能 |
|---|---|
| `upload_file_from_local_to_sandbox` | 本地上传到 sandbox |
| `download_file_from_sandbox_to_local` | sandbox 下载到本地 |
| `download_file_from_internet_to_sandbox` | 直接从 URL 下载远程文件到 sandbox |

这些工具让 agent 能处理上传文件、远程资料、数据集和生成物。

### 6.4 Tool Call Robustness

论文指出，模型会出现：

- 错误 server routing；
- 幻觉工具名；
- 参数名不匹配；
- 工具参数格式错误。

MiroThinker-1.7 在框架层自动拦截并修正部分错误，以避免长程任务因为一次 malformed tool call 被毁掉。

### 6.5 Benchmark Contamination Prevention

论文强调，在评测中会屏蔽可能泄漏 benchmark 的来源，例如 HuggingFace dataset pages。发现新泄漏域名后，会加入 blocklist，并统一应用于所有工具。

---

## 7. High-Quality QA Construction

论文使用两条互补 pipeline：Corpus-based Pipeline 和 WebHop Pipeline。

### 7.1 Corpus-based Pipeline

这条 pipeline 沿用 MiroThinker 1.0 思路：

1. 从 Wikipedia、OpenAlex 等高互联语料构建文档库；
2. 保留 hyperlink topology；
3. 从 seed document 出发采样 connected subgraph；
4. 抽取跨文档事实；
5. 用强 LLM 合成多跳 QA；
6. 通过 prompt diversification 和 obfuscation 提高问题多样性。

优点：

- 吞吐高；
- 覆盖广；
- 适合构建基础推理能力。

缺点：

- 难度控制隐式；
- reasoning depth 没有结构性保证；
- 可能存在信息泄漏或 shortcut。

### 7.2 WebHop Pipeline

WebHop 用于生成更难、更贴近开放网页、更可控的问题。核心包括三个机制。

#### Structured Multi-hop Graphs

构造以答案实体为 root 的 directed reasoning tree：

- 每条边是可验证语义关系；
- tree depth 控制推理 hop 数；
- 只允许使用 parent-child edges 上的事实，防止绕过预定路径。

#### Web-based Semantic Expansion

root entity 来自知识库以保证答案可验证，child nodes 通过 live web search 扩展。论文排除百科类来源，以引入更真实、多样的网页知识。

这让训练任务更接近 inference-time open web 条件。

#### Hierarchical Solvability Verification

论文对每层关系进行验证：

- 给定 child entities，搜索 agent 应能在有限候选中定位 parent；
- 对 root entity 使用更严格标准：只靠 first-hop neighbors 应能唯一识别 root；
- 用 anonymized fact table 让 LLM 推断隐藏 root；
- 失败样本在昂贵后续步骤前剔除。

这相当于保证每一跳都是“可解但不平凡”。

### 7.3 Adaptive Leaf Obfuscation

叶子实体如果太容易泄漏答案，会被替换成功能描述。例如：

```text
Louvre Pyramid -> a royal residence in southern England
```

若 LLM 能直接从描述识别原实体，则描述被拒绝并重新生成。

### 7.4 Difficulty-Adaptive Filtering

论文用不同能力水平的搜索 agent 进行后验过滤：

- 弱 agent 可解的问题放入早期训练，例如 SFT；
- 强 agent 仍难以解决的问题保留给后期训练，例如 RL；
- 形成 curriculum-style 的难度分层语料。

这比统一混合所有问题更合理，因为不同训练阶段需要不同难度的数据。

---

## 8. Training Pipeline：四阶段训练

MiroThinker-1.7 基于开源 Qwen3 MoE checkpoints 初始化。训练流程为：

```text
1. Agentic Mid-training
2. Agentic Supervised Fine-tuning
3. Agentic Preference Optimization
4. Agentic Reinforcement Learning
```

---

## 9. 阶段一：Agentic Mid-training

Mid-training 是 1.7 相对 v1.0 的关键增强。目标不是让模型完整模仿一条专家轨迹，而是增强单步 agentic atomic capabilities。

### 9.1 覆盖的数据类型

论文使用大规模 agentic supervision，覆盖：

- single-turn planning；
- context-conditioned reasoning；
- intermediate summarization；
- answer aggregation under partial observations；
- general instruction following；
- knowledge-intensive data。

### 9.2 Agentic Planning Boosting

训练模型在只看到用户 query 时生成：

- structured plan；
- first tool call。

数据来自：

- synthetic multi-hop QA；
- open-domain task data；
- 多领域样本。

质量控制使用 taxonomy-aware planner-judge filtering：

1. LLM judge 先分类问题，例如 logic/math、puzzle-style retrieval、direct retrieval；
2. 对不同类别应用不同过滤标准；
3. 拒绝常见坏计划：
   - 直接复制用户问题作为 search query；
   - search query 过度约束；
   - 过早猜实体；
   - 检索覆盖不足；
4. 对知识型规划，judge 检查计划是否能检索核心事实；
5. 不合格样本最多重采样 `K` 次，仍失败就丢弃。

### 9.3 Agentic Reasoning and Summarization Sculpting

这部分不是训练整条轨迹，而是：

1. 从成功多轮轨迹中选取第 `k` 步；
2. 保留此前完整上下文；
3. 将该步 rewrite 成更高质量 target；
4. 只对这一 turn 施加监督。

被 rewrite 的 turn 可以是：

- evidence consolidation；
- tool-use decision making；
- intermediate summarization；
- partial observations aggregation。

论文还随机应用 context summarization 策略，让模型学会在上下文不完整或已压缩情况下继续 reasoning。

这种训练比完整轨迹 SFT 更细粒度，能降低整条轨迹噪声对模型的影响。

### 9.4 Mid-training Objective

对单个目标 assistant turn `y_k` 做 next-token prediction：

```math
L_{mid}(\theta)=-E_{(C_{<k},y_k)\sim D_{mid}}[\log \pi_\theta(y_k|C_{<k})]
```

其中：

- `k=1` 时，`C_<1` 就是用户问题，训练 cold-start plan；
- `k>1` 时，`C_<k` 包含此前轨迹，训练上下文条件下的 reasoning/summarization。

### 9.5 为什么 mid-training 重要？

论文后续实验显示，MiroThinker-1.7-mini 相比 MiroThinker-1.5-30B 在五个 agentic benchmark 上：

- 平均性能提升 16.7%；
- 平均交互轮数减少 43.0%。

这说明 mid-training 不只是让模型“会做更多步”，而是让每一步更有信息增益。

---

## 10. 阶段二：Agentic SFT

SFT 让模型学习完整 structured agentic interaction behavior。

### 10.1 数据形式

训练集：

```math
D_{SFT}=\{(x_i,H_i)\}_{i=1}^{N}
```

其中 `x_i` 是任务指令，`H_i` 是专家轨迹：

```math
H_i=\{(T_{i,t},A_{i,t},O_{i,t})\}_{t=1}^{T_i}
```

### 10.2 数据清洗

原始轨迹即使由强 LLM 生成，也会有噪声：

- response 内重复；
- 跨 response 重复；
- malformed tool invocation；
- 错误工具名；
- 参数无法解析；
- 调用未定义工具；
- 工具报错后不重试。

论文用 rule-based filtering 和 data-cleaning pipeline 保证 SFT 语料一致性。

### 10.3 训练方式

每条轨迹被格式化为 multi-turn conversation：

- user：初始任务 + 各步 tool observation；
- assistant：thought + action。

训练时不真实执行工具，observation 是预先收集的。

SFT loss：

```math
L_{SFT}(\theta)=-E_{(x,H)}\left[\sum_{t=1}^{T_H}\log \pi_\theta(T_t,A_t|x,H_{<t})\right]
```

---

## 11. 阶段三：Agentic Preference Optimization

使用 DPO 进一步优化 decision-making。

### 11.1 偏好数据

```math
D_{PO}=\{(x_i,H_i^+,H_i^-)\}_{i=1}^{M}
```

其中 `H^+` 是 preferred trajectory，`H^-` 是 dispreferred trajectory。

### 11.2 偏好标准

论文强调：偏好主要基于 **最终答案正确性**，不强制固定结构。

不使用硬规则，例如：

- 固定 planning length；
- 固定 step counts；
- 固定 reasoning template。

原因是这类规则会导致偏差，并降低跨任务泛化。

### 11.3 质量过滤

chosen trajectory 必须：

- reasoning 连贯；
- 有显式 planning；
- final answer 正确。

rejected trajectory 也必须：

- 有有效 final answer；
- 没有严重重复；
- 没有截断；
- 没有 malformed output。

这避免模型只学会区分“好格式”和“坏格式”，而是真正学习决策质量差异。

### 11.4 Preference Distillation

对 MiroThinker-1.7-mini，论文使用 preference distillation，将更强模型的偏好信号迁移给 mini 模型。这样 mini 不只从 chosen/rejected pair 学，也从强模型偏好中获得额外 guidance。

---

## 12. 阶段四：Agentic RL

最终阶段使用 GRPO，让模型在 live environment 中 trial-and-error。

### 12.1 基础设施

需要同时运行大量 agent sessions，因此论文构建分布式环境，支持：

- multi-source web retrieval；
- page-level extraction；
- summarization；
- LLM-based answer verification；
- 低延迟 judge。

### 12.2 Streaming Rollout with Priority Scheduling

MiroThinker v1.0 已经使用 streaming rollout，即 worker 从队列取任务，完成后放入 buffer，buffer 满就训练。

1.7 进一步加入 priority scheduling：

- 长尾 rollout 更早被提升处理；
- 防止困难样本长时间不进入训练；
- 避免训练分布被短任务主导。

### 12.3 Entropy Control

论文指出 RL 中 policy entropy 过早坍塌会影响探索。为此引入 targeted entropy control：

- 对 negative rollouts 中低 log probability tokens 加辅助 KL penalty；
- 防止模型持续压低这些 token 概率；
- 保持探索能力和训练稳定性。

### 12.4 Reward 与 GRPO Objective

奖励函数：

```math
R(x,H)=\alpha_c R_{correct}(H)-\alpha_f R_{format}(H)
```

GRPO 对同一 prompt 采样 `G` 条轨迹，计算相对组均值的 advantage：

```math
\hat A_i = R(x,H_i)-\frac{1}{G}\sum_j R(x,H_j)
```

最终目标加入 token-level KL regularization，并使用动态 `β_KL(t,H)` 对负样本中低概率 token 施加额外约束。

---

## 13. Heavy-duty Reasoning Mode：H1 的验证机制

这是本文区别于 v1.0/1.7 的最大亮点。

### 13.1 为什么需要 verification？

论文的基本判断是：

```text
生成一个正确推理路径很难，但验证一个路径是否可靠通常更容易。
```

在长程搜索中，模型常会沿着最高概率路径走。但最高概率路径不一定正确，尤其在难题上可能是习惯性错误模式。Verification 的目标是：

- 阻止模型在错误路径上越走越远；
- 鼓励探索 alternative actions；
- 检查证据是否充分；
- 不让模型过早给出答案。

### 13.2 Local Verifier

Local verification 作用在中间步骤，检查：

- 当前 plan 是否合理；
- 是否应该换搜索关键词；
- 工具调用是否有用；
- 当前 observation 是否真的支持假设；
- 是否存在未验证候选；
- 是否需要继续收集证据。

它的效果是让模型在局部决策处更谨慎，并避免只是确认自己的先验偏好。

### 13.3 Global Verifier

Global verification 作用在完整轨迹末尾：

- 组织所有证据链；
- 检查最终答案是否由完整证据支持；
- 如果证据不足，要求 agent resample 或补全 reasoning chain；
- 在可控 compute budget 下选择证据最完整、最可靠的答案。

### 13.4 Local 与 Global 的分工

| 机制 | 作用位置 | 解决问题 |
|---|---|---|
| Local Verifier | 中间步骤 | 纠正局部错误、减少错误路径、提升每步有效性 |
| Global Verifier | 最终答案前 | 审计全局证据、比较候选路径、避免过早回答 |

H1 的本质是把 search agent 从“生成式探索器”升级成“生成 + 验证闭环系统”。

---

## 14. 实验设置

### 14.1 Benchmark 类别

论文评估两大类 benchmark。

#### Agentic benchmarks

- Humanity's Last Exam；
- BrowseComp；
- BrowseComp-ZH；
- GAIA；
- DeepSearchQA；
- WebWalkerQA；
- FRAMES；
- SEAL-0。

#### 专业领域 benchmarks

- FrontierSci-Olympiad：科学推理；
- SUPERChem：化学推理；
- FinSearchComp：金融搜索与分析；
- MedBrowseComp：医疗浏览与综合。

### 14.2 评测子集

论文说明：

- HLE 使用 2,158 个 text-only 子集；
- SUPERChem 使用 text-only 子集；
- FinSearchComp 使用 T2/T3 subset；
- 其他 benchmark 使用完整测试集。

### 14.3 推理参数

| 参数 | 值 |
|---|---|
| agent style | ReAct-style agent |
| temperature | 1.0 |
| top-p | 0.95 |
| context length | 256K tokens |
| maximum output length | 16,384 tokens |
| `Tmax` | 大多数 benchmark 200；BrowseComp/BrowseComp-ZH/DeepSearchQA 为 300 |
| `Rmax` | 5 |
| retention budget `K` | 5 |

### 14.4 Avg@k 与 Judge

- BrowseComp、BrowseComp-ZH、HLE、DeepSearchQA：avg@3；
- GAIA、xbench-DeepSearch-2510、SEAL-0：avg@8；
- GAIA、WebWalkerQA、DeepSearchQA、BrowseComp、BrowseComp-ZH：`gpt-4.1-2025-04-14` judge；
- HLE：官方协议中的 `o3-mini-2025-01-31`。

---

## 15. Agentic Benchmark 结果

论文表 1 的 MiroThinker 系列结果如下：

| 模型 | BrowseComp | BrowseComp-ZH | HLE | GAIA | xbench-DeepSearch-2510 | SEAL-0 | DeepSearchQA |
|---|---:|---:|---:|---:|---:|---:|---:|
| MiroThinker-1.7-mini | 67.9 | 72.3 | 36.4 | 80.3 | 57.2 | 48.2 | 67.9 |
| MiroThinker-1.7 | 74.0 | 75.3 | 42.9 | 82.7 | 62.0 | 53.0 | 72.1 |
| MiroThinker-H1 | 88.2 | 84.4 | 47.7 | 88.5 | 72.0 | 61.3 | 80.6 |

### 15.1 结果解读

- H1 在 BrowseComp 达到 88.2，高于 Gemini-3.1-Pro 的 85.9 和 Claude-4.6-Opus 的 84.0；
- H1 在 BrowseComp-ZH 达到 84.4，高于 Seed-2.0-Pro 的 82.4；
- H1 在 GAIA 达到 88.5，相比表中 OpenAI-GPT-5 的 76.4 高 12.1 分；
- H1 在 SEAL-0 达到 61.3，是表中最强；
- H1 在 DeepSearchQA 达到 80.6，低于 Claude-4.6-Opus 的 91.3，但仍较强；
- 1.7-mini 虽然小，但 BrowseComp-ZH 72.3、GAIA 80.3，说明训练 recipe 对小激活参数模型也有效。

### 15.2 mini、1.7、H1 的递进

| 版本 | 主要能力来源 | 典型变化 |
|---|---|---|
| 1.7-mini | 小模型 + agentic training | 高效率，少激活参数，具备较强 agent 能力 |
| 1.7 | 更大模型 + 同类训练 | 整体能力高于 mini |
| H1 | 1.7 + local/global verification + 更多 compute | 搜索密集和难推理任务显著提升 |

H1 的提升不是单纯模型变大，而是 reasoning mode 改变。

---

## 16. 专业领域结果

论文表 2：

| 模型 | FrontierSci-Olympiad | SUPERChem text-only | FinSearchComp T2/T3 | MedBrowseComp |
|---|---:|---:|---:|---:|
| Qwen3.5-397B | 60.6 | 49.6 | 60.8 | 47.9 |
| Seed-2.0-Pro | 74.0 | 53.0 | 70.2 | - |
| GPT-5.2-high | 77.1 | 58.0 | 73.8 | - |
| Claude-4.5-Opus | 71.4 | 43.2 | 66.2 | - |
| Gemini-3-Pro | 76.1 | 63.2 | 52.7 | - |
| MiroThinker-1.7-mini | 67.9 | 36.8 | 62.6 | 48.2 |
| MiroThinker-1.7 | 71.5 | 42.1 | 67.9 | 54.2 |
| MiroThinker-H1 | 79.0 | 51.3 | 73.9 | 56.5 |

### 16.1 解读

- H1 在 FrontierSci-Olympiad 为 79.0，高于 GPT-5.2-high 77.1 和 Gemini-3-Pro 76.1；
- H1 在 FinSearchComp 为 73.9，略高于 GPT-5.2-high 73.8；
- H1 在 MedBrowseComp 为 56.5，是表中最高；
- SUPERChem 上 H1 为 51.3，低于 Gemini-3-Pro 63.2 和 GPT-5.2-high 58.0。

这说明 H1 对科学/金融/医学深度搜索很强，但化学专业推理仍有差距，可能因为 SUPERChem 更依赖领域知识、化学计算或特定题型。

---

## 17. Long Report Evaluation

论文使用 DeepResearchEval query generation framework 自动生成 50 个 deep research queries，并评估 report quality 与 factuality。

表 3 中 MiroThinker 系列结果：

| 模型 | Report | Factuality | Overall |
|---|---:|---:|---:|
| MiroThinker-1.7-mini | 75.4 | 78.4 | 76.9 |
| MiroThinker-1.7 | 76.5 | 78.5 | 77.5 |
| MiroThinker-H1 | 76.8 | 79.1 | 78.0 |
| ChatGPT-5.4 Deep Research | 76.4 | 85.5 | 81.0 |

### 17.1 解读

- H1 的 Report 维度 76.8 略高于 ChatGPT-5.4 Deep Research 的 76.4；
- 但 Factuality 79.1 低于 ChatGPT-5.4 的 85.5；
- Overall 78.0 低于 ChatGPT-5.4 的 81.0；
- MiroThinker 系列整体报告质量强，但事实性仍有提升空间。

论文称 H1 在 report quality 上 SOTA，但如果看 overall，ChatGPT-5.4 仍更高。因此阅读时要区分具体维度。

---

## 18. Effective Interaction Scaling 实验

论文重点强调：更有效的交互不是更多交互。

### 18.1 1.7-mini vs 1.5-30B

在相同 30B parameter budget 下，MiroThinker-1.7-mini 对比 MiroThinker-1.5-30B：

- 五个 agentic benchmarks 平均性能提升 16.7%；
- 平均交互轮数减少 43.0%；
- HLE 提升尤其明显：性能 +17.4%，轮数 -61.6%。

### 18.2 结论

这支持论文的核心假设：

```text
有效交互扩展 = 提高每一步的 planning/reasoning/summarization/tool-use 质量
而不是无脑增加轮数
```

如果每一步都更准确、更聚焦、更能抽取关键信息，agent 反而可以更短路径完成任务。

---

## 19. Verification-Centric Heavy-Duty Reasoning 结果

### 19.1 Local Verifier hard subset 实验

论文选取 BrowseComp 中 MiroThinker-1.7 经常失败的 295 个 hard questions。

| 模型 | Pass@1 | Steps |
|---|---:|---:|
| MiroThinker-1.7 | 32.1 | 1185.2 |
| MiroThinker-H1 w/ Local Verifier Only | 58.5 | 210.8 |

### 19.2 解读

Local Verifier Only 带来：

- Pass@1 +26.4；
- steps 从 1185.2 降到 210.8，减少 974.4，约为原来的六分之一。

这很重要，因为 Local Verifier 没有显式目标是减少步数，但它通过纠正错误路径、避免无效搜索，天然提高了交互效率。

### 19.3 Global Verifier 与 compute scaling

论文称 Global Verifier 在所有 benchmark 上带来一致提升，特别是：

- BrowseComp：+14.2；
- SEAL-0：+8.3；
- FrontierScience-Olympiad：+7.5；
- HLE：+4.8。

Figure 7 显示 H1 在 BrowseComp 上有 token scaling curve：

- 16× compute 时达到 85.9；
- 64× compute 时达到 88.2。

这说明 H1 的 heavy-duty mode 可以用 test-time compute 换准确率，但收益会涉及更高成本和延迟。

---

## 20. 论文的关键思想

### 20.1 从 Interactive Scaling 到 Effective Interaction Scaling

v1.0 强调交互深度；1.7/H1 强调交互质量。

更准确的公式可能是：

```text
研究能力 ≈ 模型能力 × 每步交互质量 × 有效交互次数 × 验证可靠性
```

如果每步质量低，增加次数可能负收益；如果每步质量高，甚至可以减少次数并提升性能。

### 20.2 原子能力训练很关键

很多 agent 失败不是因为最终答案生成能力差，而是中间某一步错了：

- 第一个 search query 不好；
- 没有识别 observation 中的关键证据；
- 过早猜答案；
- summary 漏掉重要约束；
- 工具调用参数错；
- 没有在证据冲突时继续验证。

MiroThinker-1.7 的 mid-training 正是针对这些单步行为。

### 20.3 Verification 是长程推理的保险机制

长程 agent 的错误很难靠最后一次生成自动修复。H1 的思路是：

- 局部错误要在局部修；
- 全局答案要由完整证据链审计；
- 生成与验证分工，利用“验证比生成容易”的不对称性。

---

## 21. 局限与批判性阅读

论文没有集中列出 limitations，但可以从方法和实验中看到若干问题。

### 21.1 H1 很可能成本较高

H1 使用 local/global verifier 和 compute scaling。BrowseComp 从 16× 到 64× compute 分数 85.9 → 88.2，虽然有提升，但成本也显著增加。实际产品要权衡：

- 更高准确率；
- 更高 token 成本；
- 更多工具调用；
- 更长等待时间；
- 更复杂系统工程。

### 21.2 Verification 细节不够完全展开

论文描述 Local Verifier 和 Global Verifier 的思想，但没有完全公开 verifier prompt、候选生成策略、采样数、budget 分配、失败恢复细节。复现 H1 heavy-duty mode 可能比复现普通 ReAct 更困难。

### 21.3 部分对比来自公开报告而非统一复跑

表 1/2 中不少竞品结果来自技术报告或 model card。不同系统可能使用：

- 不同工具集；
- 不同上下文长度；
- 不同最大轮数；
- 不同 judge；
- 不同 benchmark 子集；
- 不同日期的 web 环境。

因此表格适合方向性参考，但不能完全等同严格控制变量实验。

### 21.4 LLM-as-a-Judge 仍有偏差

多项 benchmark 使用 gpt-4.1 或 o3-mini judge。对于开放式回答，LLM judge 有用，但也可能受：

- 答案格式；
- 语言表达；
- 引用形式；
- 模型偏好；
- prompt 细节影响。

### 21.5 Long report factuality 仍落后最强商业系统

虽然 H1 report quality 高，但 factuality 低于 ChatGPT-5.4 Deep Research。这说明：

- 生成结构化长报告不等于事实完全可靠；
- citation grounding 仍是 deep research 的难点；
- global verifier 可能还需要更严格证据引用检查。

### 21.6 Restart 可能带来重复搜索

Episode restart 能清理污染上下文，但也可能导致：

- 重复搜索相同路径；
- 浪费工具调用；
- 丢失已排除候选；
- 对需要长期证据累积的任务不友好。

更强方案可能需要跨 episode 的结构化 memory，而不是完全 clean slate。

---

## 22. 对实践的启示

### 22.1 不要只看工具调用次数

Agent 产品评估应同时看：

- 成功率；
- 平均工具调用；
- 平均 token；
- 每步是否引入新证据；
- 重复搜索率；
- 局部纠错率；
- 最终证据完整性。

### 22.2 单步训练数据很有价值

相比只收集完整成功轨迹，可以构造：

- 给 query 生成 first plan；
- 给历史上下文生成下一步 search；
- 给 observation 生成证据摘要；
- 给候选答案生成验证计划；
- 给不完整证据生成“还缺什么”。

这类原子能力数据可能更低噪、更容易扩展。

### 22.3 在高难任务上使用 verification mode

可以设计分层推理模式：

| 场景 | 推荐模式 |
|---|---|
| 简单事实查询 | 普通回答或少量搜索 |
| 中等多跳 QA | ReAct + recency retention |
| 高风险/高难研究 | ReAct + local verifier + global verifier |
| 长报告/专业分析 | evidence store + citation verifier + final audit |

### 22.4 Verification 可以降本而不只是增成本

Local Verifier hard subset 实验显示，验证机制可能减少无效步骤。也就是说，虽然每步多一个 verifier 调用看似增加成本，但如果它显著减少错误路径和重试，总成本可能下降。

---

## 23. 可继续研究的问题

1. **Local Verifier 消融**  
   比较不同 local verification 频率：每步验证、每 N 步验证、只在低置信度时验证。

2. **Global Verifier 机制公开与复现**  
   需要明确候选答案如何采样、证据链如何评分、预算如何分配。

3. **Verifier 与 Memory 结合**  
   用结构化 evidence memory 记录跨 episode 证据，避免 clean-slate restart 丢失历史。

4. **成本感知 H1**  
   在 verifier 决策中加入 token/tool cost，训练动态选择 compute budget。

5. **Factuality 优化**  
   对 long report 加入 citation-reference alignment、claim-level verification、来源可靠性评分。

6. **中文和专业领域验证**  
   针对中文网页、金融公告、医学文献、科学论文分别设计 verifier。

7. **验证器自身可靠性评估**  
   验证器也可能错，需要评估 false positive/false negative。

---

## 24. 术语速查

| 术语 | 解释 |
|---|---|
| Effective Interaction Scaling | 提高每一步交互质量，而不是单纯增加交互长度 |
| Heavy-duty Reasoning | 面向高难任务的强推理模式，通常使用更多 compute 和验证 |
| Local Verifier | 对中间计划、工具调用、假设更新进行局部审查 |
| Global Verifier | 对完整证据链和候选答案进行最终审计 |
| Sliding-window Filtering | 只保留最近 K 个 observation，保留完整 thought/action trace |
| Episode Restart | 单次 episode 失败或耗尽轮数后，从原问题重新开始 |
| Agentic Mid-training | 介于 pre-training 与 post-training 之间，增强 agent 原子能力 |
| Preference Distillation | 用强模型偏好信号指导小模型偏好优化 |
| GRPO | 基于同组相对奖励的策略优化方法 |
| LLM-as-a-Judge | 用大模型判定答案或报告质量 |

---

## 25. 最终理解

这篇论文的核心信息是：**长程研究智能体不应该只追求更多轮搜索和更长思考，而应该追求更有效的交互，以及对交互过程的持续验证。**

MiroThinker-1.7 通过 agentic mid-training 提升每一步的 planning、reasoning、tool-use、summarization 能力，使模型能用更少轮数完成更难任务；MiroThinker-H1 进一步把 verification 嵌入推理过程，通过 Local Verifier 修正局部错误，通过 Global Verifier 审计最终证据链，在高难搜索与专业推理任务上获得显著提升。

如果说 MiroThinker v1.0 的关键词是 **Interactive Scaling**，那么这篇论文的关键词就是：

```text
Effective Interaction Scaling + Verification-Centric Heavy-Duty Reasoning
```

它为研究型 agent 的下一步发展给出了清晰方向：不只是更大模型、更长上下文、更多工具调用，而是让每一步更可靠，让最终答案更可验证。
