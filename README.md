# 📚 论文阅读笔记

## 关于本仓库

大家好！我是一名计算机专业在读 PhD 学生，目前在某厂基础模型组实习。这个仓库用于记录我日常阅读大模型训练、Agentic 训练相关论文的详细笔记，希望能够督促自己保持持续学习的习惯，同时也欢迎感兴趣的朋友一起交流讨论。

## 📖 关注方向

- 大语言模型（LLM）预训练 / 后训练方法
- RLHF / RLAIF 等对齐技术
- Agent 相关训练方法（Agentic Training）
- 推理能力增强（Reasoning / CoT）
- 模型微调与优化技巧
- 其他前沿大模型相关工作

## 📂 仓库结构

```
.
├── README.md
├── notes-template.md        # 单篇论文笔记模板
├── notes_undealing/       # 待处理论文与笔记（不提交 Git）
├── indexes/                 # 总索引与主题索引
│   ├── papers.md            # 所有论文阅读记录
│   └── topics.md            # 主题分类与关键词
├── papers/                  # 按主题分类的论文笔记
│   ├── pretraining/         # 预训练相关
│   ├── alignment/           # 对齐/RLHF/RLAIF相关
│   ├── agent/               # Agent训练与Agentic workflow相关
│   │   └── deep-research/ # DeepResearch 专题笔记
│   ├── reasoning/           # 推理能力、CoT、test-time scaling相关
│   ├── finetuning/          # SFT、PEFT、LoRA、模型压缩等
│   ├── survey/              # 综述、教程、系统性总结
│   └── others/              # 暂未归类内容
├── assets/
│   ├── figures/             # 笔记配图
│   └── pdfs/                # 可选：论文PDF，不想同步大文件可留空
│       └── deep-research/   # DeepResearch 原始 PDF
└── scripts/                 # 可选：辅助脚本
```

## ✍️ 笔记格式

每篇笔记大致包含以下内容：

- **论文标题 & 链接**
- **核心问题**：论文试图解决什么问题
- **方法概述**：核心思路和技术方案
- **实验结果**：关键实验和结论
- **个人思考**：优缺点分析、启发与延伸思考

建议文件命名：

```text
papers/<topic>/<YYYY-MM-DD>-<short-paper-name>.md
```

例如：

```text
papers/reasoning/2026-08-07-chain-of-thought.md
```

原始 PDF 使用与笔记相同的文件名主体，仅扩展名不同，并放入 `assets/pdfs/<topic>/`。

## 🎯 更新计划

会尽量保持每周更新，记录近期阅读和思考的论文，欢迎 star 和 watch 来持续关注更新～

## 🤝 交流讨论

如果你也对大模型训练、Agent 相关方向感兴趣，欢迎：

- 提 Issue 讨论论文内容
- 提 PR 补充笔记或修正错误
- Fork 本仓库，建立自己的阅读笔记

一起学习，一起进步！🚀

---

> 免责声明：笔记内容仅代表个人理解，如有错误欢迎指正；部分内容可能存在主观解读，仅供参考学习交流使用。
