# 附录 F　推荐阅读（原始资料）

> 按重要程度排序，标注了每篇文章的核心价值。

---

## 一、必读：Harness Engineering 起源文章

### OpenAI 官方博客（两篇）

**[Harness engineering: leveraging Codex in an agent-first world](https://openai.com/index/harness-engineering/)**
- 作者：Ryan Lopopolo（OpenAI 工程师）
- 发布：2026年2月
- 核心价值：Harness Engineering 概念的起源文章，描述了 3人×100万行代码实验的完整经过
- 必读理由：这是这门学科的"创世文档"，所有概念都从这里来

**[Unlocking the Codex harness: how we built the App Server](https://openai.com/index/unlocking-the-codex-harness/)**
- 作者：OpenAI 工程团队
- 发布：2026年2月
- 核心价值：第一篇的技术深度补充，详细描述了 App Server 的构建过程和具体的 Harness 设计
- 必读理由：有大量具体的技术实现细节，是第一篇的实践版

---

## 二、重要：深度解析文章

**[Harness Engineering — Martin Fowler](https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html)**
- 作者：Martin Fowler（软件工程领域权威）
- 核心价值：从独立视角对 Harness Engineering 的深度解析，与 OpenAI 原文互补
- 推荐理由：Martin Fowler 的文章以清晰和深度著称，这篇是理解 Harness 的最佳第三方解读

**[The Emerging "Harness Engineering" Playbook](https://www.ignorance.ai/p/the-emerging-harness-engineering)**
- 核心价值：总结了 Mitchell Hashimoto 的六阶段模型和 Greg Brockman 的 AGENTS.md 建议
- 推荐理由：把多个关键人物的观点整合在一起，适合快速了解全貌

---

## 三、学术论文

**[Building AI Coding Agents for the Terminal: Scaffolding, Harness, Context Engineering](https://arxiv.org/abs/2603.05344)**
- 作者：Nghi D. Q. Bui 等
- 发布：2026年3月（arXiv 2603.05344）
- 核心价值：目前已知最早的 Harness Engineering 学术论文，介绍了开源系统 OPENDEV
- 推荐理由：学术视角，有严格的定义和实验数据

---

## 四、工具和框架文档

**[LangChain Blog — The Anatomy of an Agent Harness](https://blog.langchain.com/the-anatomy-of-an-agent-harness/)**
- 核心价值：从 LangChain 生态视角解析 Harness 的组成结构
- 推荐理由：如果你在使用 LangChain 相关工具，这篇文章很有参考价值

**[HumanLayer Blog — Skill Issue: Harness Engineering for Coding Agents](https://www.humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents/)**
- 核心价值：专注于编码 Agent 的 Harness 实践，有具体的实现建议
- 推荐理由：实践导向，适合动手实施时参考

---

## 五、背景阅读（理解上下文）

**Andrej Karpathy — Vibe Coding（2025年初）**
- 搜索关键词：`Andrej Karpathy vibe coding`
- 核心价值：理解 Harness Engineering 要解决的问题的起点
- 推荐理由：Karpathy 命名了"vibe coding"，是 Harness Engineering 出现的直接背景

**Mitchell Hashimoto — AI 采用六阶段（2025年下半年）**
- 搜索关键词：`Mitchell Hashimoto AI adoption stages`
- 核心价值：理解 Harness Engineering 在 AI 编码演进中的位置
- 推荐理由：六阶段模型是理解"为什么需要 Harness"的最佳框架

---

## 六、工具文档（按需查阅）

| 工具 | 文档地址 | 用途 |
|------|---------|------|
| ESLint | https://eslint.org/docs/ | 自定义规则开发 |
| Vitest | https://vitest.dev/ | 测试配置 |
| Playwright | https://playwright.dev/ | 浏览器自动化 |
| madge | https://github.com/pahen/madge | 循环依赖检测 |
| ts-prune | https://github.com/nadeesha/ts-prune | 死代码检测 |
| jscpd | https://github.com/kucherenko/jscpd | 重复代码检测 |
| GitHub Actions | https://docs.github.com/actions | CI 配置 |
| GitLab CI | https://docs.gitlab.com/ee/ci/ | CI 配置（公司环境）|

---

## 七、阅读顺序建议

```
如果你只有 1 小时：
  → 读 OpenAI 第一篇博客（harness-engineering）

如果你有半天：
  → OpenAI 两篇博客 + Martin Fowler 的解析

如果你想深入研究：
  → 以上所有 + arXiv 论文 + LangChain 博客

如果你想了解背景：
  → 先读 Karpathy 的 vibe coding
  → 再读 Hashimoto 的六阶段
  → 最后读 OpenAI 的 Harness Engineering
```

---

*附录 F · 完*
