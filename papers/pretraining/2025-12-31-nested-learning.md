# Nested Learning：深度学习架构的"幻觉"——论文详解

> 来源文件：`2025-12-31-nested-learning.pdf`  
> 论文题目：**Nested Learning: The Illusion of Deep Learning Architecture**  
> 作者：Ali Behrouz, Meisam Razaviyayn, Peilin Zhong, Vahab Mirrokni  
> 机构：Google, Columbia University  
> 版本信息：NeurIPS 2025；arXiv:2512.24695v1，2025-12-31  
> 原文：[arXiv 摘要页](https://arxiv.org/abs/2512.24695) · [PDF](https://arxiv.org/pdf/2512.24695)

> "We cannot solve our problems with the same thinking we used when we created them!" — 爱因斯坦

---

## 写在最前面：给新手的导读

这篇论文不是一篇"提一个新模型"的工作，而是提出了一个新的**视角（paradigm）**：把"模型架构"和"优化算法"统一起来看，认为它们都是同一个东西——**嵌套的优化问题**。

为了让你能看懂后面的内容，先把全文会反复出现的术语都解释一遍。

> **什么是 LLM（大语言模型）？**
> Large Language Model 的缩写。指的是经过海量文本预训练、参数量巨大（几亿到上千亿）的神经网络，比如 GPT-4、Llama-3、Gemini。

> **什么是 Transformer？**
> 当前几乎所有 LLM 的主干架构。核心是"自注意力（self-attention）"机制 + 前馈网络（MLP）的堆叠。本文的一个核心论点是：Transformer 其实只是 Nested Learning 框架下一个"两级（two-level）"的特例。

> **什么是 MLP（多层感知机）？**
> Multi-Layer Perceptron，最简单的前馈神经网络：输入向量 → 线性层 → 非线性激活 → 线性层 → 输出。Transformer 中夹在 attention 之间的那一块就是 MLP。

> **什么是 RNN（循环神经网络）？**
> Recurrent Neural Network。处理序列时按时间步迭代更新隐藏状态 $h_t = f(h_{t-1}, x_t)$。本文里"现代 RNN"指 Linear Attention、DeltaNet、RWKV、Titans 等线性循环模型。

> **什么是注意力机制（Attention）？**
> 给定 query（查询）、key（键）、value（值）三组向量，输出 = "用 query 和所有 key 算相似度，再对 value 加权求和"。Softmax attention 是用 softmax 归一化的标准版本。

> **什么是预训练（Pre-training）？**
> 在海量无标签或弱标签数据上用一个通用目标（比如"预测下一个 token"）把模型训练好。训练完之后再在具体任务上做微调或直接用。

> **什么是 In-Context Learning（ICL，上下文学习）？**
> 不更新模型参数，仅在 prompt（输入上下文）里给几个例子，模型就能"现场"学会做新任务。本文将其重新定义为：**任何模型从其当前 context 中适应/学习的能力**。

> **什么是 Continual Learning（持续学习）？**
> 模型在训练完之后还能不断从新数据中学到新知识，且不会"遗忘"之前学过的。和 LLM 当前的"训练完就冻结"形成鲜明对比。

> **什么是 Catastrophic Forgetting（灾难性遗忘）？**
> 模型在新任务上训练时，会迅速忘掉之前任务上的能力。是持续学习最大的拦路虎。

> **什么是 Associative Memory（关联记忆）？**
> 一个把"键 key"映射到"值 value"的算子 $\mathcal{M}(\cdot)$。比如电话簿就是从"姓名"映射到"号码"。本文最核心的视角是：**所有学习——无论是训练 MLP、更新 momentum、还是 attention——都是在做关联记忆**。

> **什么是 Gradient Descent（梯度下降）？**
> 最常见的优化算法：$W_{t+1} = W_t - \eta \nabla L(W_t)$。沿着梯度反方向走一小步，把损失变小。

> **什么是 Momentum（动量）？**
> 在 SGD 上加一个"过去梯度的指数滑动平均（EMA）"，让更新方向更平滑、收敛更快。本文的一个重要洞察是：**momentum 本身就是一个关联记忆模块**，它压缩了过去所有梯度的信息。

> **什么是 Adam？**
> 最广泛使用的优化器之一。在 momentum 基础上加了"梯度方差的滑动平均"做归一化（自适应学习率）。本文证明 Adam 是"$L_2$-回归目标"下最优的元素级关联记忆。

> **什么是 Muon？**
> 2024 年提出的优化器（Jordan et al.）。在 momentum 之上用 Newton-Schulz 迭代把梯度"正交化"，再去更新参数。在大模型预训练中表现优异。

> **什么是 Meta-Learning（元学习 / 学会学习）？**
> "学会怎么学"——通过外层优化过程学一个能让内层优化（具体任务的学习）变快变好的初始化或学习规则。MAML 是经典代表。

> **什么是 Hypernetwork（超网络）？**
> 一个网络的输出是另一个网络的权重。NL 把它解释为"低频网络生成高频网络的参数"。

> **什么是 Backpropagation（反向传播）？**
> 训练神经网络的标准算法：通过链式法则把"输出处的损失梯度"逐层往回传，得到每一层参数的梯度，再用梯度下降更新。

> **什么是 Self-Referential（自指 / 自参照）？**
> 一个系统的更新依赖于它自己当前的状态。Schmidhuber 1993 年提出过"自指权重矩阵"。本文 Hope 架构里的 Self-Modifying Titans 就是让模型生成自己的 keys/values/学习率。

> **什么是 Frequency / 频率（在本文里）？**
> 论文给的定义：**单位时间内某个组件被更新多少次**。Attention 每个 token 都更新一次（频率最高，∞），而 MLP 的权重在预训练完之后就不再更新（频率为 0）。本文用频率来给所有组件排序，从而构成"嵌套层级"。

> **什么是 NSAM（Nested System of Associative Memories，嵌套关联记忆系统）？**
> 本文的核心数学对象。一个由多层（不同频率）关联记忆组成的系统，每一层都在用梯度下降优化自己的"上下文流（context flow）"。

> **什么是 CMS（Continuum Memory System，连续介质记忆系统）？**
> 本文提出的内存设计：把传统的"短期记忆 + 长期记忆"二分推广为一个频率连续谱，每个频率档位对应一个 MLP 块。

> **什么是 Hope？**
> 本文最终落地的架构。结合了 Self-Modifying Titans（自修改的序列模型）和 CMS（连续介质记忆）。在多个基准上做实验，是论文的"概念验证"。

读完导读，下面进入正题。


## 1. 背景

### 1.1 我们正处在一个什么时代

过去几十年的 AI 进展，本质上是在四件事上反复迭代：
1. 设计**更具表达力的架构**（CNN、LSTM、Transformer、现代 RNN）；
2. 设计**更好的训练目标**（监督、自监督、对比学习、GAN、扩散）；
3. 设计**更高效的优化算法**（Adam、AdaGrad、Shampoo、Muon）；
4. **scale up**——把"对的架构 + 对的目标 + 对的优化器"做大、做长、灌更多数据。

LLM 是这四股力量交汇出的产物。但作者指出，再这么"堆更深、训更久"已经无法解决一些根本问题：

- **计算深度不随层数线性增长**（Merrill 2022）。Transformer 加再多层，能模拟的算法类别也是有上限的。
- **某些参数空间的容量随深度只有边际提升**（Kaplan 2020）。
- **优化器选错就到不了好解**——Transformer 用 SGD 训出来的解和用 Adam 训出来的解可以截然不同。
- **不能持续学习**——预训练完之后参数就冻住了，新知识只能通过 in-context（短时记忆）注入，无法沉淀进 MLP（长时记忆）。

### 1.2 一个生动的类比：顺行性遗忘症（Anterograde Amnesia）

作者把 LLM 比作**顺行性遗忘症患者**：海马体受损后，老记忆都在，但**再也无法形成新的长期记忆**。LLM 的窘境如出一辙——预训练之前学到的东西都在 MLP 里，新信息只在 context window 这个"短期缓冲"里活几千个 token，就消失了。

> **什么是顺行性遗忘症？**
> 一种神经学症状：海马体受损后，旧记忆完整保留，但**无法把短期记忆转化为长期记忆**。每天醒来，外界对患者来说都像是"今天才第一次发生"。

### 1.3 大脑的两个启示

作者从神经科学借了两个核心观察：

**(1) 大脑是多时间尺度（multi-timescale）系统。**
脑电波从快到慢分四档：
- Gamma 波（30–150 Hz）：感觉信息处理；
- Beta 波（13–30 Hz）：主动思考、工作记忆；
- Theta 波（4–8 Hz）：记忆编码；
- Delta 波（0.5–4 Hz）：深睡时的记忆巩固。

**不同频率服务不同认知功能。** 而当前深度学习模型里：所有层的权重在预训练阶段都用**同一个学习率**更新，测试时全部冻结——这是一个极其僵化的"双频率"设计：attention 频率 = ∞（每个 token 都重算），MLP 频率 = 0（永不更新）。

**(2) 大脑的结构是"统一可复用"的。**
通过半脑切除（hemispherectomy）这类极端案例可以看到，大脑神经元并不是"专门为某个功能设计"的，而是**同质、可复用、可重新分配**的。反观深度学习架构——它由 attention、MLP、卷积、门控、归一化等异构组件拼成，看起来高度异质化。**这是真异质，还是我们看问题的视角太浅了？**

### 1.4 论文要回答的核心问题

> 当前 LLM 不能持续学习。这究竟是"再加几层"就能解决的问题，还是需要一个全新的学习范式？

作者的答案是：**需要一个新范式——Nested Learning（NL，嵌套学习）。**

---

## 2. 核心问题

把作者想攻击的痛点拆开来看：

### 2.1 静态性问题

LLM 预训练完之后，唯一"可塑"的部分就是 in-context（注意力 + KV cache）。一旦 prompt 结束，所有"现场学到的"知识都灰飞烟灭。这导致：
- 没法增量学新语言、新领域；
- 没法记住几小时前用户说过的事（除非塞回 prompt）；
- 任何"长期记忆"必须靠重新训练或外挂检索（RAG）。

### 2.2 计算深度被假性堆叠掩盖

加层 ≠ 增加计算深度。一个 100 层 Transformer 在并行计算复杂度上仍属于 $\mathsf{TC}^0$ 这个相对弱的类（Merrill et al. 2024），无法解决某些状态追踪问题。

### 2.3 优化器选择缺乏理论指导

实践中，Transformer + SGD 学不出 Transformer + Adam 的解。但"哪种架构该配哪种优化器"几乎完全靠经验。作者认为这是因为我们**没有把架构和优化器统一来看**。

### 2.4 灾难性遗忘的根源被忽视

经典动量优化器在持续学习的"正交任务"场景下，会让动量缓慢地从旧任务方向漂移到新任务方向（实证：用 $\beta=0.9$，最近 6 个梯度贡献了 50%，最近 43 个贡献了 99%）——也就是说**优化器本身就是"短期记忆"，无法保住旧任务的方向**，必然遗忘。

> **什么是 EMA（指数滑动平均）？**
> $m_t = \beta m_{t-1} + (1-\beta) g_t$。一个低通滤波器，让最近的信号权重大、远处的衰减。$\beta=0.9$ 时半衰期约 7 步。

### 2.5 总结：核心问题一句话

**深度学习框架把"架构"和"优化"当成两件互不相干的事来设计；本文要论证：它们是同一件事在不同频率层级上的体现，且只有把它们放在一个统一的"嵌套优化"视角里，才能解决持续学习、计算深度、优化器选择这一连串问题。**


## 3. 近年来各种方法的对比与优缺点

为了方便理解 NL 在大图景里的位置，先把"赋予 LLM 持续学习/长上下文/自适应能力"的几大路线梳理出来。

### 3.1 按"扩展能力"划分的三大类方法

| 类别 | 代表方法 | 核心思路 | 优点 | 缺点 |
|---|---|---|---|---|
| **(A) 显式架构改进**（更大 KV、更长 attention） | Sliding Window Attention、Longformer、DuoAttention、Cartridges | 直接把 attention 的"短期记忆"拉长，或加压缩 | 实现简单，原生兼容 Transformer | 内存/计算随上下文线性甚至二次增长，仍无法持续 |
| **(B) 递归/线性 RNN（线性记忆）** | Linear Attention、RetNet、RWKV、Mamba、DeltaNet | 用 $O(d^2)$ 大小的矩阵状态 $\mathcal{M}_t$ 压缩历史，丢掉 KV cache | 推理 $O(L)$、内存常数 | 容量小，长上下文 recall 弱于 attention |
| **(C) 深层记忆 / Test-Time Training** | TTT、Titans、Atlas、Miras、Memora、DLA | 在推理时用梯度下降把 context 压进一个 MLP 模块的参数里 | 表达能力强、可处理 1M context | 推理需要梯度，计算开销大 |
| **(D) 持续学习 / 防遗忘** | EWC、Orthogonal Gradient Descent、InCA、Cartridges | 显式正则、外挂学习器或经验回放 | 防遗忘有效 | 需要外部组件，泛化性受限 |
| **(E) 元学习 / 学会学习** | MAML、Hypernetwork、Learned Optimizer | 外层学一个能让内层学得快的初始化或更新规则 | 快速适应新任务 | 训练成本高、不稳定 |
| **(F) 优化器改进** | Adam、AdaGrad、Shampoo、Soap、Muon、AdaMuon | 提升梯度压缩/预处理的能力 | 训练更快、更稳 | 各自的设计哲学割裂，缺统一视角 |

### 3.2 现代 RNN 内部按"学习规则（learning rule）"细分

| 学习规则 | 内部目标 | 更新公式骨架 | 容量 | 代表 |
|---|---|---|---|---|
| **Hebbian 规则** | 点积相似度 $-\langle\mathcal{M}k,v\rangle$ | $\mathcal{M}_t = \alpha \mathcal{M}_{t-1} + v_t k_t^\top$ | 受限（只能"加加加"） | Linear Attention、RetNet、RWKV |
| **Delta 规则** | $L_2$ 回归 $\|\mathcal{M}k-v\|_2^2$ | $\mathcal{M}_t = (I-\eta k_t k_t^\top)\mathcal{M}_{t-1} + \eta v_t k_t^\top$ | 较好（能"擦写") | DeltaNet、RWKV-7、Longhorn、Comba |
| **Oja 规则** | 加单位范数约束 | 在 Hebbian 上减一个 $\mathcal{M}^\top v$ 项 | 比 Hebbian 稳，比 Delta 弱 | OjaNet |
| **$L_p$ 回归 / Omega 规则** | $\|\mathcal{M}k-v\|_p^p$ 或带历史窗口 | 更复杂的内层 GD | 强 | Miras、Atlas（Behrouz 2025a/b） |

### 3.3 NL 视角下的统一观察

> **作者的核心论点**：上面的所有路线（A）-（F），从 NL 视角看，**都是"嵌套关联记忆系统（NSAM）"的不同设计点**——它们的差异只是：
>
> 1. **层级数量**（几层频率）；
> 2. **每一级的内目标 $\tilde{\mathcal{L}}$**（点积？$L_2$？$L_p$？）；
> 3. **每一级的更新规则**（梯度下降？带动量？带预处理？）；
> 4. **层级间如何传递知识**（直接条件、反传、初始化、生成权重）。

把它们拉到统一框架后，NL 提议的"新设计点"就是：**多于两层的、跨频率的嵌套系统**——这是当前所有方法都没探索的方向。

---

## 4. 方法详解

### 4.1 Notation 速查表（论文里用到的核心符号）

| 符号 | 含义 |
|---|---|
| $x \in \mathbb{R}^{N\times d_\text{in}}$ | 输入序列 |
| $\mathcal{M}_t$ | 记忆模块在时间 $t$ 的状态 |
| $K, V, Q$ | keys / values / queries 矩阵；$k_t, v_t, q_t$ 为对应单 token 向量 |
| $W^{(\ell)}$ 或 $W^{(f_\ell)}$ | 第 $\ell$ 层（频率 $f_\ell$）的参数 |
| $\eta_t$ | 学习率 |
| $\alpha_t$ | 遗忘门 / 权重衰减 |
| $\delta_\ell$ | 第 $\ell$ 层的"局部错误信号"（local surprise） |
| $L(\cdot)$、$\tilde{L}(\cdot)$ | 外层任务目标 / 内层关联记忆目标 |
| $C_i^{(k)}$ | 第 $k$ 层第 $i$ 个优化问题的"上下文流" |

### 4.2 核心定义：从关联记忆出发

#### 定义 1：关联记忆

给定 keys $\mathcal{K}\subseteq\mathbb{R}^{d_k}$ 和 values $\mathcal{V}\subseteq\mathbb{R}^{d_v}$，关联记忆是一个算子 $\mathcal{M}(\cdot)$，它通过最小化某个目标 $\tilde{L}$ 来学会从 $\mathcal{K}$ 到 $\mathcal{V}$ 的映射：

$$\mathcal{M}^* = \arg\min_{\mathcal{M}} \tilde{L}(\mathcal{M}(\mathcal{K}), \mathcal{V}). \tag{6}$$

> **为什么要这样定义？**
> 因为论文要论证：**所有"训练 / 学习"的过程**——无论是用反向传播训 MLP、用 momentum 更新权重、还是用 attention 处理上下文——**本质都是在解一个像 (6) 这样的关联记忆问题**。

#### 关键概念区分

> **学习 vs 记忆（来自神经心理学）**
> - **记忆（memory）**：因输入而引起的一次神经更新；
> - **学习（learning）**：获取"有效记忆"的过程。
>
> 在这个定义下，**梯度下降的每一步都是一次"记忆"**，整个训练过程就是"学习"。

### 4.3 第一个例子：训练单层 MLP 的本质是关联记忆

考虑用梯度下降训练 1 层 MLP（参数 $W$）：

$$W_{t+1} = W_t - \eta_{t+1}\nabla_W L(W_t; x_{t+1})$$
$$= W_t - \eta_{t+1} \underbrace{\nabla_{y_{t+1}} L(W_t; x_{t+1})}_{\text{Surprise}_y} \otimes\ x_{t+1}. \tag{8}$$

把 $u_{t+1} = \nabla_{y_{t+1}} L(W_t; x_{t+1})$（输出处的"局部 surprise"）拿出来，等价地写成：

$$W_{t+1} = \arg\min_W \langle Wx_{t+1}, u_{t+1}\rangle + \frac{1}{2\eta_{t+1}}\|W-W_t\|_2^2. \tag{9}$$

> **怎么读这个公式？**
> 第一项要求 $W$ 把 $x_{t+1}$ 映射到一个和 $u_{t+1}$ 内积大的方向（即"记住" key→value 的对应关系）；第二项是 proximal term，约束新 $W$ 不能离当前 $W_t$ 太远（保留旧记忆）。这就是一个标准的关联记忆优化问题。

> **重要takeaway**：**用反向传播训神经网络 = 用一个 surprise-based 关联记忆把每个数据样本映射到它的预测错误**。

### 4.4 加上动量：训练 MLP 变成"两层"嵌套

带动量的 SGD 更新规则：
$$W_{t+1} = W_t - m_{t+1}, \quad m_{t+1} = \alpha m_t + \eta_{t+1}\nabla L. \tag{11}$$

定义 $u_{t+1} = \nabla_W L$，把动量项的更新重写成关联记忆形式：

$$m_{t+1} = \arg\min_m -\langle m, u_{t+1}\rangle + \frac{1}{2\eta}\|m-m_t\|_2^2. \tag{13}$$

这个式子的解释力在于——**动量本身就是一个关联记忆**，它把"过去所有梯度"压缩进自己的参数里。所以：

> **关键观察**：
> "用带动量的 SGD 训练 MLP" = **两层嵌套优化**：
> - **外层**（低频）：用 $m_t$ 更新 $W_t$；
> - **内层**（高频）：用梯度下降优化 $m_t$，让它"记住"过去梯度。

### 4.5 Frequency 与 Nested System 的形式化

#### 定义 2：更新频率 $f_A$
组件 $A$ 的"频率"= 单位时间（一个数据点）内被更新的次数。

#### 定义 3：嵌套系统
一个嵌套系统由 $K$ 个**有序层级**组成。第 $k$ 层包含 $N_k$ 个优化问题 $\{(\mathcal{L}_i^{(k)}, \mathcal{C}_i^{(k)}, \boldsymbol{\Theta}_i^{(k)})\}$，每个问题都用梯度下降优化：

$$\theta_{i,t+1}^{(k)} = \arg\min_{\Phi_i^{(k)}} \langle \Phi_i^{(k)} x_{t+1}, -\nabla L_i^{(k)}(\theta_{i,t}^{(k)}; x_{t+1})\rangle + \frac{1}{2\eta}\|\Phi_i^{(k)}-\theta_{i,t}^{(k)}\|_2^2 \tag{19}$$

层级间的排序规则：**频率高的在前**（频率相同但计算依赖另一个的，排在被依赖者之后）。

#### 定义 4：NSAM（嵌套关联记忆系统）
在嵌套系统的基础上，要求每个优化问题都是关联记忆问题（每个 $\mathcal{C}_i^{(k)}$ 是 key-value 对集合，目标 $L_i^{(k)}$ 衡量 key→value 映射质量）。

> **直觉理解**：现代神经网络（包括 Transformer、RNN、优化器）在 NL 视角下都是 NSAM 的实例。差别只是**层数、每层的目标、每层的更新规则**。

### 4.6 一个让人豁然开朗的例子：Adaptive Transformer

把标准 Transformer block 里的 MLP 块**换成"初始状态可元学习的线性 attention"**（作者称之为 AdaTransformer）。两者的输出几乎一样：

$$y_\text{block}^\text{Transformer} = y_\text{attn}\, W_\text{MLP}, \qquad y_\text{block}^\text{AdaTrans} = y_\text{attn}\, W_\text{LinAttn}.$$

差别在哪？

- **Transformer**：$W_\text{MLP}$ 在第 1 层（最低频，预训练时更新一次，推理时冻住）；
- **AdaTransformer**：$W_\text{LinAttn}$ 既在第 1 层（其初始值由预训练学），又在第 2 层（每个 token 内部用 $\mathcal{M}_t = \mathcal{M}_{t-1} + v_t k_t^\top$ 更新）。

> **Takeaway**：所谓"hybrid 架构"（attention + 现代 RNN 混合），从 NL 视角看，**就是 Transformer block 的某些 MLP 上又加了一个"in-context 学习"的高频层**。这说明 attention 和现代 RNN **并不是异质架构**，而是同一个统一架构在不同频率上的体现。

### 4.7 反向传播 = 自指（self-referential）的关联记忆

对一个 $L$ 层 MLP，反向传播给出每层的梯度：

$$\frac{\partial L}{\partial W_\ell} = \delta_\ell\, \hat{x}_{\ell-1}^\top, \quad \delta_\ell = J_{\phi_\ell}(z_\ell)^\top\, W_{\ell+1}^\top\, \delta_{\ell+1}. \tag{29}$$

更新规则：$W_{\ell, t+1} = W_{\ell,t} - \eta_{\ell,t+1}\delta_\ell\hat{x}_{\ell-1}^\top$。

**为什么这不是简单的"对梯度做线性 attention"？**
因为 $\delta_\ell$ 本身依赖于 $W_\ell$（自己当前的状态），而不是被预先给定的。这是 **self-referential**——记忆生成自己的 value。Schmidhuber 1993 年提出过这个概念。

> **什么是 Self-Referential（自指）？**
> 系统的更新规则中，被更新对象自身参与生成"更新信号"。区别于线性 RNN——线性 RNN 的 key/value 来自外部输入，与状态无关，因此可以并行；自指系统则强制串行，但表达能力更强。

### 4.8 从 NL 看 Adam / Muon / AdaGrad

#### Adam
作者证明（Appendix B）：**Adam = "在 element-wise $L_2$-回归目标下"最优的关联记忆**。它的两个 EMA 项 $m_t$ 和 $v_t$ 分别压缩"梯度的一阶矩"和"梯度方差"。

#### Muon
$$W_{t+1} = W_t + \text{NewtonSchulz}_k(m_{t+1}). \tag{42}$$

NewtonSchulz 把动量映射到正交空间。从 NL 视角，这是一个**内层优化**：用最小化 $\|P^\top P - I\|_F^2$ 去**学一个把梯度映射到正交基的线性变换**，更新公式正是 Muon 用的 3 次多项式：
$$O_{i+1} = O_i - \zeta(O_i - g_t + 2 O_i O_i^\top O_i - I). \tag{44}$$

> **所以 Muon 不是"凭空设计"的优化器，而是一个内层 1-2 步梯度下降的近似解**。

### 4.9 提出新优化器：Delta Gradient Descent（DGD）和 Delta Momentum

经典 SGD 的内层目标是点积相似度 $\langle Wx_t, u_t\rangle$，这相当于**默认数据样本之间相互独立**。但**序列里的 token 高度相关**——这种"独立性假设"会让动量更新错过相关性结构。

替换为 $L_2$ 回归 $\frac{1}{2}\|Wx_t - u_t\|_2^2 + \frac{1}{2\eta}\|W-W_t\|_2^2$，并用 Sherman-Morrison 引理求解，得到：

$$W_{t+1} = W_t(I - \eta'_t x_t x_t^\top) - \eta'_t \nabla_W L(W_t; x_t). \tag{57}$$

这就是 **Delta Gradient Descent（DGD）**。比起标准 SGD：
- 多了一个 $W_t (I - \eta'_t x_t x_t^\top)$ 项——**根据当前样本自适应地"擦除"旧权重的部分内容**；
- 实质是把序列建模里的 Delta 规则（DeltaNet 的核心）搬到了优化器层面。

类似地把 momentum 的内目标从点积换成 $L_2$，得到 **Delta Momentum**：
$$m_{i+1} = \alpha_{i+1} m_i - \nabla L(W_i; x_i)\nabla L(W_i; x_i)^\top - \eta_t P_i \nabla L(W_i; x_i). \tag{49}$$

> **直觉**：经典动量是 EMA，相当于"最近 43 步"低通滤波；Delta Momentum 加了"基于梯度自适应衰减"的能力，遇到相关样本会主动忘掉无关方向，更善于在 loss landscape 上做长程规划。

### 4.10 Continuum Memory System（CMS，连续介质记忆）

#### 动机
传统观点把记忆切成"短期（attention，频率∞）"和"长期（MLP，频率0）"两块。CMS 把它推广为一**串频率连续的 MLP 块**：

$$y_t = \text{MLP}^{(f_k)}(\text{MLP}^{(f_{k-1})}(\cdots\text{MLP}^{(f_1)}(x_t))). \tag{70}$$

第 $\ell$ 层的更新频率是 $C^{(\ell)}$ 步：

$$\theta^{(f_\ell)}_{i+1} = \theta^{(f_\ell)}_i - \begin{cases}\sum_t \eta^{(\ell)}_t f(\theta_t^{(f_\ell)}; x_t) & \text{若 } i \equiv 0 \pmod{C^{(\ell)}}\\ 0 & \text{否则}\end{cases} \tag{71}$$

> **为什么这么设计？**
> 当某一频率档的块要被更新时，潜在被覆盖的旧知识在"更低频率"的另一块里还存着；下一次反传又能把它循环回来——形成"记忆环"，**对抗灾难性遗忘**。

#### 三种 CMS 变体

| 变体 | 知识传递方式 | 特点 |
|---|---|---|
| **Nested CMS** | 第 $s+1$ 层的初始状态由第 $s$ 层元学习 | 高阶 in-context 学习 |
| **Sequential CMS** | 各层串行连接；初始状态都由最低频 backprop 学 | 共享最持久的 context 压缩 |
| **Independent (Head-wise) CMS** | 各层并行接收输入，输出 Agg 聚合 | 训练快，最常用 |

> **效率？**
> 任意时刻只更新"到了更新档期"的块，平均仅 $O(\frac{1}{\hat{f}}\cdot \frac{L_\text{layer}}{5}\cdot d_\text{in}^2)$ 个参数；并允许 chunk-wise 并行。

### 4.11 M3 优化器：CMS 在优化器里的实例

把 CMS 思想用在优化器上，作者提出 **Multi-scale Momentum/Memory Muon (M3)**。骨架（算法 1）：

```
slow memory  M^(2)_t = M^(2)_{t-1} + β3 · sum(g over [k-1)f, kf])
O^(2)_t      = NewtonSchulz_T(M^(2)_t)
for t = kf+1, ..., (k+1)f:
    g_t        = ∇L(Θ_t)
    M^(1)_t    = M^(1)_{t-1} + β1 g_t          # fast momentum
    V_t        = V_{t-1} + β2 g_t^2
    O^(1)_t    = NewtonSchulz_T(M^(1)_t)
    Θ_t       ← Θ_{t-1} - η · (O^(1)_t + α O^(2)_t)/(√V_t + ε)
```

形象地说：**M3 = Adam + Muon + CMS**。两个动量项分别压缩"近期"和"长期"梯度，再用 NewtonSchulz 正交化、Adam 风格归一化。

### 4.12 Hope 架构：Self-Modifying Titans + CMS

Hope 是论文最终的"概念验证"架构，由两部分组成：

#### (a) Self-Modifying Titans（自修改的序列模型）

让 $W_k, W_v, W_q, W_\eta, W_\alpha$ 全部都是**可在 context 里更新的关联记忆**，而且 value 也是**自己生成的**：

$$\hat{v}_{\square,t} = \mathcal{M}_{\square,t-1}(v_t),$$
$$\mathcal{M}_{\square,t} = \mathcal{M}_{\square,t-1}(\alpha_t I - \eta_t k_t k_t^\top) - \eta_t \nabla L(\mathcal{M}_{\square,t-1}; k_t, \hat{v}_{\square,t}),$$
$$\square \in \{k, v, q, \eta, \alpha, \text{memory}\}. \tag{86}$$

> **直观地说**：模型不仅在 in-context 中**适应自己的记忆**，还在适应过程中**生成自己的学习目标和学习率**。这就是 Schmidhuber 1993 提出的"自指"思想的现代实现。

每个记忆 $\mathcal{M}_\square$ 是一个 2 层 MLP：$\mathcal{M}_\square(\cdot) = (\cdot) + W_{\square,1} \sigma(W_{\square,2}(\cdot))$。

#### (b) CMS 后端

序列模型之后接一个 CMS（多频率 MLP 链），承担"持久记忆 + 防遗忘"的角色。

#### (c) 训练并行化

用 chunk-wise 训练（Sun 2024、Behrouz 2025c）：把序列切成 $\lceil L/C\rceil$ 段，每段开头用上一段末尾的状态生成所有 keys/values/学习率，**段内可全并行**。

#### (d) Hope-Attention 变体

把 self-modifying Titans 替换成全局 softmax attention，得到一个比 Transformer 更强的"注意力 + CMS"模型。


## 5. 为什么有效

把 §2 的"核心问题"和 §4 的方法放在一起，逐条对应：

### 5.1 解决"静态性 / 不能持续学习"
- **CMS 提供"持久记忆 + 短期记忆"的频率谱**：低频块负责长期知识，高频块负责快速适应；当某块被更新而丢失旧知识时，更低频块还保留着，反传时知识能"循环回来"。
- **Self-Modifying Titans 让所有投影都能在 context 里适配**：标准 Transformer 的 $W_k, W_v, W_q$ 一旦预训练完就死了，Hope 里它们都活着。

### 5.2 解决"计算深度被假堆叠掩盖"
- **多层级嵌套带来真实的内部计算深度**：每个 token 走过 self-modifying 内部优化、CMS 多层 MLP 内部优化，相当于在每一步做了好几个嵌套的优化问题。Muon 的 NewtonSchulz 内迭代就是一个 $k$ 步的内部优化，已经在实践中证明加深"每神经元的计算量"能显著提升性能。

### 5.3 解决"优化器选择缺乏理论"
- **优化器 = 关联记忆**：选 Adam 等价于"用 element-wise $L_2$ 回归压缩梯度"；选 Muon 等价于"加一个正交化的内层映射"。设计优化器变成"选择内层目标 + 内层学习规则"，理论上有清晰的设计空间。
- **架构生成优化器的 context**：不同架构的梯度分布不同，所以**优化器应该针对架构定制**。NL 给出了"架构特定优化器"的概念依据。

### 5.4 解决"灾难性遗忘的根源"
- **传统动量是 6 步 EMA**——长上下文记忆容量太小。Delta Momentum、DGD、M3 用更复杂的内层目标和更大容量的动量记忆，让优化器自己有"长期梯度子空间"的全局视野。
- **CMS 把"哪些方向不能动"分布在多个频率档上**，最低频的那一档几乎永远保留预训练对齐的方向。

### 5.5 一句话总结"为什么有效"
**所有现有方法都在挖一口浅井——只在两个频率（pre-training 频率 0 和 attention 频率 ∞）之间做文章。NL 把井挖深，引入连续的频率谱，每一档都能压缩自己的 context。这条新维度上的设计自由度，本身就是性能增益的来源。**

---

## 6. 理论分析

论文里没有传统意义上的"定理 + 证明"，但有几个**结构性论断**值得拎出来。

### 6.1 论断 1：Adam 是 element-wise $L_2$ 回归下最优的关联记忆

> **意思是**：在所有"对每个梯度坐标独立做关联记忆"的设计里，Adam 的 $m_t$ + $v_t$ 二阶矩归一化是 Bayes 最优解。这给了 Adam 一个清晰的 NL 解释。

### 6.2 论断 2：Softmax Attention 是 $L_2$ 回归的非参数解

Softmax attention 等价于：
$$\mathcal{M}^* = \arg\min_{\mathcal{M}} \sum_{i=1}^L s(k_i, q)\|v_i - \mathcal{M}\|_2^2 = \frac{\sum_i s(k_i, q) v_i}{\sum_j s(k_j, q)}. \tag{62}$$

这是 Nadaraya-Watson 估计——用 $s(k_i, q)$ 做核加权的非参数回归。

> **影响**：当模型规模和数据规模够大时，**参数化的现代 RNN（用一个矩阵记忆）不可能在同样目标下击败 attention（非参数解）**。要超越 attention，必须**改目标或改容量**。

### 6.3 论断 3：所有现代 RNN 的学习规则都是 NSAM 的实例

| 模型 | 内目标 $\tilde{L}$ | 优化器 | 等价更新 |
|---|---|---|---|
| Linear Attention / RWKV / RetNet | $-\langle\mathcal{M}k, v\rangle$（点积） | GD | $\mathcal{M}_t = \alpha\mathcal{M}_{t-1} + v_t k_t^\top$ |
| DeltaNet / Longhorn / RWKV-7 | $\|\mathcal{M}k - v\|_2^2$ | GD | $\mathcal{M}_t = (I-\eta k_t k_t^\top)\mathcal{M}_{t-1} + \eta v_t k_t^\top$ |
| OjaNet | $-2\langle\mathcal{M}k, v\rangle + \|\mathcal{M}^\top v\|_2^2$ | GD | Hebbian + 单位范数惩罚 |
| Miras / Atlas | $\|\mathcal{M}k - v\|_p^p$ | GD | $L_p$-回归 |

> **意义**：**架构=目标+优化器**这个公式让设计新架构变成了"选目标 + 选优化器"两个独立的工程任务，大大降低了设计空间的复杂度。

### 6.4 论断 4：Pre-training 是 ICL 的极限情况

> Pre-training 就是"上下文 = 整个预训练数据"的 in-context learning。  
> 训练 / 测试的二分，是因为我们**人为切断了"高频层 → 低频层"的知识传递路径**。

### 6.5 论断 5：In-Context Learning 不是涌现能力

只要模型有多层 NL 表示（多个频率档），ICL 就**自然存在**。它在大模型上看起来"涌现"，是因为低频层（pre-training 学的那部分）训得足够好后，高频层有足够锚定来快速适应。

---

## 7. 实验设计

### 7.1 评测的六大维度

论文实验覆盖：
1. **Continual learning + ICL**（学新语言、类增量、新语料 QA）
2. **长上下文理解**（Needle-in-a-Haystack、BABILong）
3. **语言建模 + 常识推理**（Wikitext、LMB、PIQA、HellaSwag、Winogrande、ARC、SIQA、BoolQ）
4. **In-context recall**（SWDE、NQ、DROP、FDA、SQUAD、TQA + MAD 合成基准）
5. **形式语言识别**（Parity、$a^nb^n$、$a^nb^nc^n$、Shuffle-2）
6. **优化器对比**（ImageNet ViT、140M / 1.3B 语言模型训练）

### 7.2 各 benchmark 的具体例子

#### (1) CLINC（类增量意图分类）
- **能力**：在 150 个意图类别上做增量学习，不能遗忘旧类别。
- **示例**：
  - **Step 1**：模型先学 50 类，输入 "What's the weather like tomorrow?" → 预测 "weather"
  - **Step 2**：再学 50 类（不接触旧数据），输入 "Reset my card PIN" → 预测 "card_PIN_reset"
  - **Step 3**：测试旧任务，输入 "Tomorrow's forecast?" → **应仍预测 "weather"**（防遗忘）

#### (2) BABILong（长上下文推理）
- **能力**：在长达 10M token 的 haystack 中找到分布式线索做推理。
- **示例**：
  - **Input**：一篇 100K 字的混合文档，里面藏着 "John picked up the apple" 和 "John dropped it in the kitchen"。
  - **Question**：Where is the apple?
  - **Expected**：kitchen.
  - 论文里 Hope 在 10M context 上仍然能保持竞争力，而 GPT-4 在 256K 后就失败。

#### (3) Needle-in-a-Haystack (NIAH) - S-NIAH-1（pass-key 检索）
- **能力**：从一段长文本中提取一个特定的"针"。
- **示例**：
  - **Context**：80K token 的 Paul Graham 散文，中间塞了 "The pass key is 38291. Remember it."
  - **Query**：What is the pass key?
  - **Expected**：38291.
  - Hope 在 4K/8K/16K 上都拿到 100/100/100，Transformer 是 88.6/76.4/79.8。

#### (4) MK-NIAH（多键 NIAH）
- **能力**：检索多个分散的关键事实并组合。
- **示例**：
  - **Context**：长文里散布着 "key A maps to 7", "key B maps to 12", "key C maps to 5"。
  - **Query**：What does key B map to?
  - **Expected**：12.

#### (5) LongHealth（长上下文医疗 QA）
- **能力**：基于 5–7K 字的患者档案做多选 QA。
- **示例**：
  - **Patient Record**：完整病历（用药史、化验、影像报告，5K 字）。
  - **Q**：Based on the record, which medication caused the rash on day 3?
  - **A**：（4 选 1）amoxicillin。

#### (6) QASPER（NLP 论文问答）
- **能力**：以一篇完整 NLP 论文为 context，回答里面的细节。
- **示例**：
  - **Paper**：BERT 论文全文。
  - **Q**：What was the dataset size for the masked LM pretraining?
  - **A**：BookCorpus (800M words) + English Wikipedia (2,500M words).

#### (7) MTOB + Manchu（CTNL：上下文学习新语言）
- **能力**：模型从 prompt 里给的语法书 + 词典学翻译。Hope 测试两个低资源语言：Kalamang（巴布亚），Manchu（满语）。
- **示例**：
  - **Context**：Kalamang grammar book + 100 句翻译对照。
  - **Q**：Translate "I will go fishing tomorrow" to Kalamang.
  - **Expected**：相应满语/Kalamang 表达。
  - **关键设定**：先学 Manchu，再学 Kalamang，再回过头测 Manchu→英 ——验证防遗忘。

#### (8) Parity（奇偶校验，形式语言）
- **能力**：判断输入串中 1 的个数是奇是偶。Transformer **理论上做不到**（Hahn 2020）。
- **示例**：
  - **Input**：1010110
  - **Output**：even（4 个 1）。
  - Hope 拿到 100，Transformer 在 Bin1 上是 0。

#### (9) MAD（合成 ICR / 压缩基准）
- 子任务包括 Compression、In-Context Recall、Fuzzy ICR、Selective Copying、Memorization。

#### (10) Wikitext / LMB / PIQA 等
- 标准的语言建模 / 常识推理 benchmark，用 perplexity 和 accuracy 评估。

### 7.3 Baseline 分组

| 类别 | 代表 baseline |
|---|---|
| **Transformer 类** | Transformer++、Samba（混合架构） |
| **纯 Hebbian RNN** | RetNet、RWKV |
| **Delta-rule RNN** | DeltaNet、RWKV-7、Comba |
| **深层记忆 (Test-Time Training)** | TTT、Miras (Memora)、DLA、Titans |
| **持续学习专用** | EWC、InCA、ICL、DuoAttention |
| **大模型 / 检索增强** | GPT-4、GPT-4o-mini、Llama-8B + RAG |
| **小模型 SOTA** | RMT、ARMT、Titans |
| **优化器** | AdamW、Muon、AdaMuon |

### 7.4 实现细节

- **预训练数据**：约 50B token（NIAH 和 BABILong）、30B / 100B token（语言建模），来自 FineWeb-Edu + 长文档；vocab size 32K。
- **模型规模**：760M / 1.3B 参数；持续学习实验里 Hope-Attention 的 backbone 用 Llama3-8B 和 Llama-3B + 15B 持续预训练 token（**不从头训，而是用 §7.3 的"用预训练 MLP 初始化 CMS 各档"技巧**）。
- **优化器**：默认 AdamW，每个模型 tune 学习率；M3 实验另算。
- **Hope 配置**：Self-modifying Titans 的所有 $\mathcal{M}_\square$ 用 2-layer MLP；keys/queries 做 $L_2$ 归一化；窗口 4 的局部卷积；CMS 的 chunk size 512–2K（可调）。


## 8. 实验结果

### 8.1 类增量学习（CLINC / Banking / DBpedia）

(论文 Figure 6)

Hope 在三个数据集上**全部超过**所有 baseline，包括用外部学习器的 InCA 和经典的 EWC。
关键对比：Hope-Attention（仅替换 MLP 为 CMS 的 Llama 持续预训练）vs ICL（同样持续预训练但保持 MLP 不变）——前者明显更优。

> **结论**：CMS 设计本身（让 MLP 多档频率更新）比"加正则防遗忘"或"加外部学习器"更有效。

### 8.2 多频率层数对 ICL 的影响（Figure 7）

在 MK-NIAH、LongHealth、QASPER 三个长上下文 QA 基准上：
- **Hope 任意层数 / 任意最低频率 都击败了 ICL 和 DuoAttention**；
- **更多层级 → 更好 ICL 表现**；
- **最低频率越低（即模型有更持久的记忆）→ 表现越好**；
- "Lowest Frequency = 2K" 是性能/效率最优点（接近 512 的最佳但显著更省算力）。

### 8.3 Continual Translation of Novel Language（CTNL，Figure 8）

学 Manchu → 英 + Kalamang → 英：
- **设定 1**（独立学每种语言）：Hope 各变体与 ICL 持平或更好。
- **设定 2**（依次学两种语言后回测）：**ICL 灾难性遗忘**，性能跌到接近预训练基线；**Hope-3** 几乎恢复到设定 1 的水平。

> **强力证据**：CMS + 多层次设计是防遗忘的关键。

### 8.4 Needle-in-a-Haystack（Table 1）

S-NIAH-1（pass-key 检索，4K / 8K / 16K context）：
- **Transformer**：88.6 / 76.4 / 79.8
- **Hope-Attention**：100 / 100 / 100
- **Titans**：100 / 100 / 100
- **Hope**：100 / 100 / 100
- **RWKV-7 / Comba**：99–100 / 99–100 / 99–100
- **DLA**：96.4 / 71.2 / 44.0

S-NIAH-3（uuid 检索，最难）：
- **Transformer**：78.0 / 69.2 / 40.8
- **Hope-Attention**：76.8 / 68.8 / 42.4
- **Titans**：74.2 / 42.8 / 21.2
- **Hope**：73.2 / 46.2 / 24.8 — **在所有 attention-free 模型中最强**

MK-NIAH-1（多键检索，16K）：
- **Transformer**：61.4
- **Hope-Attention**：60.8
- **Hope**：14.8（仍是 attention-free 最佳）

> **结论**：长上下文 recall 的标准故事——attention-based 强于 recurrent；但 Hope 和 Hope-Attention **既缩小了与 Transformer 的差距**，又**进一步超过了 Transformer**（Hope-Attention 在多数项上微胜）。

### 8.5 BABILong（Figure 9）

10M context 长度上：
- **GPT-4 / GPT-4o-mini**：在 256K 后就崩溃；
- **Llama-8B + RAG**：256K 后能保持稳定但水平不高；
- **Titans / ARMT**：1M 之前还行，1M 后掉得快；
- **Hope**：**保持竞争力直到 10M**——CMS 的多频率压缩生效。

### 8.6 语言建模 + 常识推理（Table 2）

**760M params / 30B tokens** 平均分（数值越高越好；ppl 越低越好）：
| 模型 | Wiki ppl ↓ | LMB ppl ↓ | LMB acc ↑ | Avg ↑ |
|---|---|---|---|---|
| Transformer++ | 24.18 | 24.27 | 37.1 | 50.11 |
| Samba | 21.07 | 22.85 | 39.2 | 51.46 |
| RetNet | 25.77 | 24.19 | 34.5 | 48.19 |
| DeltaNet | 24.52 | 24.38 | 36.8 | 49.63 |
| RWKV-7 | 23.75 | 23.08 | 37.1 | 50.55 |
| Comba | 22.41 | 22.19 | 37.5 | 50.89 |
| Titans | 20.08 | 21.52 | 38.1 | 51.68 |
| **Hope** | **18.68** | **20.07** | **38.8** | **52.28** |

**1.3B params / 100B tokens**：
| 模型 | Wiki ppl ↓ | LMB ppl ↓ | Avg ↑ |
|---|---|---|---|
| Transformer++ | 17.92 | 17.73 | 53.38 |
| Samba | 16.15 | 13.21 | 54.46 |
| Titans | 15.60 | 11.41 | 56.82 |
| **Hope** | **14.39** | **10.08** | **58.04** |

> **结论**：
> - Hope 在 760M 已超 Titans / Samba 等强 baseline；
> - **Scale 起来后差距更大**（1.3B 上 Hope 平均提升 1.2 分）——说明 NL 这条新轴和参数量是**正相关**的。

### 8.7 In-Context Recall（Table 3）

短 ICR 任务（SWDE / NQ / DROP / FDA / SQUAD / TQA）：
- **Transformer 仍是最强**（71.4 / 22.0 / 23.9 / 67.3 / 39.4 / 59.1）；
- **Hope** 是 attention-free 最强（65.9 / 21.2 / 22.8 / 41.9 / 33.0 / 57.7），**显著缩小了与 Transformer 的差距**。

### 8.8 MAD 合成基准（Table 4）

| 模型 | Compression | ICR | Fuzzy ICR | Selective Copying | Memory |
|---|---|---|---|---|---|
| Transformers | 49.4 | 100 | 47.9 | 96.2 | 83.7 |
| Titans | 49.8 | 100 | 50.0 | 99.4 | 83.4 |
| **Hope** | **51.2** | **100** | **52.1** | **99.7** | **85.2** |

> **Hope 全面胜过 Transformer**，包括"Compression"和"Memory"这两个理论上 Transformer 应该更强的子任务。

### 8.9 形式语言识别（Table 5）

Parity / $a^nb^n$ / $a^nb^nc^n$ / Shuffle-2：
- **Transformer**：在 Parity Bin1 上 0%；多数 Bin0 / Bin1 上 0%；
- **LSTM / SRWM**（非线性循环但**不可并行**）：100% / 100%；
- **Linear / DeltaNet**：除了 Shuffle-2 都失败；
- **Hope**：**全部 100%，且训练并行**——同时拿下"表达力"和"训练效率"两个目标。

### 8.10 M3 优化器（Figure 11、Figure 12）

ImageNet-21K 上训 ViT（24M / 86M）：
- M3 的 train/test loss **均优于 AdamW 和 Muon**。

140M / 1.3B 语言模型训练时间对比：
- M3 比 Muon 慢一些（多两个 momentum 项），与 AdaMuon 相当——**作为概念验证可以接受，规模化部署还要优化**。

---

## 9. 消融实验

(论文 Table 6，760M 模型)

| 变体 | Language Modeling ppl ↓ | Reasoning acc ↑ |
|---|---|---|
| **Hope（完整）** | **12.24** | **58.1** |
| w/o DGD（用普通 GD） | 13.41 | 56.5 |
| w/o Momentum | 13.58 | 56.9 |
| w/o weight decay | 13.71 | 57.2 |
| w/o CMS | 13.04 | 57.3 |
| w/o inner-projection $k$ | 13.77 | 56.9 |
| w/o inner-projection $v$ | 13.90 | 55.1 |
| w/o inner-projection $q$ | 12.19 | 57.4 |

> **每个组件都正向贡献**：
> - **DGD vs GD**：换回普通 SGD 让 ppl 涨了 1.17 → DGD 的"自适应擦除"对优化非常关键；
> - **去 momentum / 去 weight decay**：都让 ppl 升 1+ → 经典的"动量 + 衰减"在自指 Titans 里同样必要；
> - **去 CMS**：ppl 升 0.8 → CMS 即便对小模型也有可见收益；
> - **去 inner-projection $v$ 影响最大**：value 投影内层化是**最有价值的"自修改"组件**。
> - **去 $q$ 投影微微变好**（ppl 12.19 vs 12.24）但准确率下降——说明 query 不必都自修改，有些位置保留外层投影更稳。

> **作者已用 Figure 7 / Table 8 间接消融了"层数"和"最低频率"两个超参数**：层数越多越好，最低频率越低（更多持久记忆）越好；2K 是性价比拐点。

---

## 10. 鲁棒性 / 泛化分析

### 10.1 上下文长度泛化（Figure 10）
"Perplexity vs Context Usage" 曲线：
- **Hope 的 ppl 随 context 增长持续下降**——说明它**真的在用长上下文**；
- **Transformer 早早趋于平坦**——长上下文里有效信号被稀释。

### 10.2 跨语言 / 持续学习鲁棒性（CTNL）
Hope-3 在依次学两种新语言后，**回测前一种语言几乎不掉点**。

### 10.3 计算深度的真实增长
在 Parity 等需要状态追踪的任务上，Transformer 即便 100 层也做不到，Hope 1 层（2 个频率档）就 100% 满分——**真实计算深度增长**而非堆 layer。

### 10.4 灾难性遗忘"是否解决"
作者给出诚实回答：
> Hope 和 CMS **缓解**了灾难性遗忘，但**没有"解决"它**。从 NL 视角看，遗忘是"有限容量被迫舍弃旧信息"的自然结果——只能通过"分布式存储 + 多频率" 把它推迟、稀释，而不能根除。

---

## 11. 总结

### 11.1 论文的主要贡献

1. **Nested Learning（NL）范式**：把"架构"和"优化"统一成嵌套的多层级关联记忆系统（NSAM）；通过更新频率给所有组件排序，揭示 Transformer 只是"频率 0 / ∞ 极端两档"的特例。

2. **优化器即关联记忆**：证明 momentum、Adam、Muon、AdaGrad 都是关联记忆模块；据此设计 **DGD（Delta Gradient Descent）**、**Delta Momentum**、**M3（Multi-scale Momentum Muon）** 等新优化器。

3. **Continuum Memory System（CMS）**：把"短期 / 长期记忆"二分推广为**频率连续谱**。每档频率对应一个 MLP 块，独立更新，互相通过反传或元学习共享知识。

4. **Self-Modifying Titans**：让 $W_k, W_v, W_q$ 全部能在 context 中更新，且 value 自生成——Schmidhuber 1993 自指思想的现代实现。

5. **Hope 架构**：Self-Modifying Titans + CMS。在 Wikitext / LMB / NIAH / BABILong / MAD / 形式语言 / 持续学习 / CTNL 等多个 benchmark 上**全面达到或超过最强 baseline**。

6. **大量结构性论断**：
   - Pre-training = ICL 的极限情况；
   - Test-time training / memorization = parametric ICL 的别名；
   - In-context learning **不是** 涌现能力，而是多层级 NL 表示的直接结果；
   - "Hybrid 架构"的本质是"在某些 MLP 上加了一个高频层"，所以现代深度学习架构其实**结构是统一的**——异质性是"看不到 NL 这个轴"造成的"幻觉（illusion）"，**这正是论文标题的含义**。

### 11.2 给新手的关键 takeaway

1. **学习 = 压缩**。神经网络的训练过程，本质上是在让一个关联记忆把"输入 → 它对应的预测错误"压缩进自己的参数里。
2. **优化器 = 短期记忆**。动量项就是一个把"过去几十步梯度"做指数滑动平均的小记忆。理解这一点，你就能重新设计优化器。
3. **频率视角**。每个组件都有"多久更新一次"这个属性。Transformer 的 attention 是无穷高频，MLP 是 0 频。**这是过度极端的设计**，中间档完全没人用过。
4. **Self-referential ≠ Linear Recurrence**。反向传播不是简单的"对梯度做线性 attention"——因为 value 由记忆自己生成，必须串行、表达力更强。
5. **持续学习 = 多频率压缩 + 知识传递**。CMS 给出的方案是"分布式、多时间尺度"，对照大脑的脑波理论。
6. **看待 Transformer 的新眼光**：它不是"终极架构"，而是 NL 这个空间里的一个**特例**——只有两档频率（∞ 和 0）。NL 的整个新维度还没被探索。

### 11.3 局限性

作者自己列出的 limitations：
- **灾难性遗忘没"解决"**——NL 只是缓解；
- **M3 优化器有计算开销**，大规模仍待优化；
- **NL 不是"终点而是路线图"**：未来工作应该探索更多层级的设计、不同 benchmark 上的更深消融、以及和强化学习等设定的结合。

### 11.4 NL 给整个领域的暗示

> "未来在持续学习、长上下文、自修改模型上的进展，将来自**更好地利用'层级'这条额外的设计维度**，而不是把静态网络越堆越深。"

这段话可以当成 LLM 领域往前走的一面镜子——回看 Transformer 之于 RNN，它打开的是"注意力 vs 顺序"这个轴；NL 提议的"频率 + 嵌套"轴还在等人去填。

---

> **最后一句**：如果你只能记住这篇论文里的一句话，记住这句——
>
> **"现代深度学习架构看似异质，其实只是同一个嵌套优化系统在不同频率上的解。把这条频率轴展开，就能解锁连续学习。"**
