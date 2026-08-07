# 大模型论文阅读笔记

这个仓库记录我日常阅读大模型训练、Agentic Training、Deep Research 与推理相关论文时整理的中文笔记。希望这些详细拆解能帮助自己持续学习，也能为关注相同方向的同学提供参考。

当前主要关注：

- 大语言模型预训练与后训练
- Agentic Training 与工具增强智能体
- Deep Research 与长程搜索
- RLHF、RLAIF 与 Agentic RL
- 推理、验证器与 test-time scaling

## 已收录论文

以下清单会随新论文和笔记持续更新。

### Agentic Training

| 论文 | 关键词 | 资源 |
| --- | --- | --- |
| CAST: Game Solvers as Turn-Level Teachers for LLM Agents | Turn-level credit、RLVR、Solver Teacher | [中文笔记](paper_notes/agent/2026-07-28-cast.md) · [论文 PDF](paper/agent/2026-07-28-cast.pdf) |
| Self-Distilled Agentic Reinforcement Learning | Self-Distillation、GRPO、Multi-turn Agent | [中文笔记](paper_notes/agent/2026-05-14-sdar.md) · [论文 PDF](paper/agent/2026-05-14-sdar.pdf) |

### Deep Research

> [DeepResearch 论文汇总与横向对比](paper_notes/agent/deep-research/deep-research-overview.md)

| 论文 | 关键词 | 资源 |
| --- | --- | --- |
| S1-DeepResearch: Beyond Search, Toward Real-World Long-Horizon Research Agents | Beyond Search、Trajectory Construction、AgentLoop | [中文笔记](paper_notes/agent/deep-research/2026-06-13-s1-deep-research.md) · [论文 PDF](paper/deep-research/2026-06-13-s1-deep-research.pdf) |
| Quest: Training Frontier Deep Research Agents with Fully Synthetic Tasks | Synthetic Tasks、Rubric Tree、GRPO | [中文笔记](paper_notes/agent/deep-research/2026-05-22-quest.md) · [论文 PDF](paper/deep-research/2026-05-22-quest.pdf) |
| RubricEM: Meta-RL with Rubric-guided Policy Decomposition | Rubric、Meta-RL、Credit Assignment | [中文笔记](paper_notes/agent/deep-research/2026-05-11-rubricem.md) · [论文 PDF](paper/deep-research/2026-05-11-rubricem.pdf) |
| MiroThinker-1.7 & H1: Towards Heavy-Duty Research Agents via Verification | Verification、Heavy Mode、Effective Interaction | [中文笔记](paper_notes/agent/deep-research/2026-03-16-mirothinker-1.7-h1.md) · [论文 PDF](paper/deep-research/2026-03-16-mirothinker-1.7-h1.pdf) |
| REDSearcher: A Scalable and Cost-Efficient Framework for Long-Horizon Search Agents | Task Synthesis、Deep Search、Simulated Environment | [中文笔记](paper_notes/agent/deep-research/2026-02-15-redsearcher.md) · [论文 PDF](paper/deep-research/2026-02-15-redsearcher.pdf) |
| MiroThinker: Pushing the Performance Boundaries of Open-Source Research Agents | Interactive Scaling、Agentic SFT、Agentic RL | [中文笔记](paper_notes/agent/deep-research/2025-11-18-mirothinker.md) · [论文 PDF](paper/deep-research/2025-11-18-mirothinker.pdf) |
| Tongyi DeepResearch Technical Report | Agentic CPT、SFT、RL、Heavy Mode | [中文笔记](paper_notes/agent/deep-research/2025-11-04-tongyi-deep-research.md) · [论文 PDF](paper/deep-research/2025-11-04-tongyi-deep-research.pdf) |

### Pretraining 与模型架构

| 论文 | 关键词 | 资源 |
| --- | --- | --- |
| Nested Learning: The Illusion of Deep Learning Architecture | Nested Optimization、Continual Learning、Memory | [中文笔记](paper_notes/pretraining/2025-12-31-nested-learning.md) · [论文 PDF](paper/pretraining/2025-12-31-nested-learning.pdf) |

## 仓库目录

```text
.
├── README.md
├── paper_notes/             # 中文论文笔记
│   ├── agent/
│   │   └── deep-research/
│   ├── pretraining/
│   ├── alignment/
│   ├── reasoning/
│   ├── finetuning/
│   ├── survey/
│   └── others/
├── paper/                   # 与笔记同名的原始论文 PDF
│   ├── agent/
│   ├── deep-research/
│   └── pretraining/
├── indexes/                 # 总索引与主题索引
└── assets/                  # 笔记配图等资源
```

## 交流与讨论

欢迎通过 Issue 讨论论文内容、指出笔记中的错误，也欢迎提交 PR 补充观点。如果这些内容对你有帮助，欢迎 Star 或 Watch 仓库以获取后续更新。

---

> 免责声明：笔记内容仅代表个人理解，部分内容可能存在主观解读，如有错误欢迎指正。
