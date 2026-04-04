# 附录 A　核心术语中英对照表

> 按字母顺序排列，方便查阅。

---

## 核心概念

| 中文 | English | 说明 |
|------|---------|------|
| 驾具 / 约束系统 | Harness | 约束 AI Agent 可靠工作的整套环境系统，包括文档、规则、工具、反馈回路 |
| 智能体 | Agent | 能自主执行任务的 AI 程序，可以调用工具、读写文件、循环直到完成任务 |
| 驾驭工程 | Harness Engineering | 设计和维护 Harness 的工程学科，2026年2月由 OpenAI 正式命名 |
| 大语言模型 | LLM（Large Language Model）| 底层 AI 模型，被动响应输入 |
| 氛围编码 | Vibe Coding | 无系统约束地用 AI 写代码，适合可丢弃的原型项目 |
| 智能体工程 | Agentic Engineering | AI Agent 自主完成任务的工程方式，有基本约束但不系统 |

---

## 上下文工程

| 中文 | English | 说明 |
|------|---------|------|
| 上下文 | Context | Agent 执行任务时能"看到"的所有信息总和 |
| 上下文窗口 | Context Window | 模型单次能处理的最大信息量，超出则"遗忘"早期内容 |
| 上下文工程 | Context Engineering | 管理 Agent 在任务过程中看到的信息的工程方法 |
| 提示词工程 | Prompt Engineering | 优化单次指令措辞的技术 |
| 技能文档 | Skills | 告诉 Agent 何时用哪个工具、为什么用的文档 |
| 渐进式工具发现 | Progressive Disclosure | 按需加载工具定义，而不是一次性全部提供 |

---

## 架构约束

| 中文 | English | 说明 |
|------|---------|------|
| 代码检查器 | Linter | 自动扫描代码、发现规范违规的工具（前端常用 ESLint）|
| 持续集成 | CI（Continuous Integration）| 提交代码时自动运行的检查流水线 |
| 持续交付 | CD（Continuous Delivery）| 自动化部署流程 |
| 横切关注点 | Cross-cutting Concerns | 被多个模块共同使用的功能（鉴权、日志、监控等）|
| 提供者模式 | Providers Pattern | 用于封装横切关注点的设计模式 |
| 依赖方向 | Dependency Direction | 分层架构中，层与层之间允许的依赖流向 |
| 循环依赖 | Circular Dependency | A 依赖 B，B 又依赖 A 的问题结构 |

---

## 熵管理

| 中文 | English | 说明 |
|------|---------|------|
| 代码熵 | Code Entropy | 代码库混乱程度的度量，熵越高越难维护 |
| AI 垃圾代码 | AI Slop | AI 生成的低质量、难维护代码（重复、风格混乱、过度工程化）|
| 黄金原则 | Golden Principles | 编码进仓库的最佳实践规则集，定义"好代码"的标准 |
| 技术债 | Technical Debt | 为了短期速度牺牲长期可维护性积累的问题 |
| 死代码 | Dead Code | 存在于代码库但从未被调用的代码 |
| 重复率 | Duplication Rate | 代码库中重复代码块占总代码量的比例 |

---

## 可观测性

| 中文 | English | 说明 |
|------|---------|------|
| 可观测性 | Observability | 通过工具让 Agent 能感知代码运行结果的能力 |
| 反馈回路 | Feedback Loop | Agent 做了一件事到知道结果是否正确的完整循环 |
| 并行工作树 | Git Worktree | 同一 Git 仓库的多个独立工作目录，支持多 Agent 并行 |
| 首 Token 延迟 | TTFT（Time to First Token）| 从发送 AI 请求到收到第一个响应 token 的时间 |
| Web 核心指标 | Web Vitals | 衡量网页用户体验的关键性能指标（LCP、FID、CLS 等）|

---

## 工作流

| 中文 | English | 说明 |
|------|---------|------|
| 拉取请求 | PR（Pull Request）| GitHub 上的代码合并请求 |
| 合并请求 | MR（Merge Request）| GitLab 上的代码合并请求，等同于 PR |
| 设计决策记录 | ADR（Architecture Decision Record）| 记录重要架构决策及其原因的文档 |
| 自动合并 | Auto-merge | 满足条件时 PR 自动合并，无需人工操作 |
| 快速失败 | Fail Fast | CI 流水线中，发现问题立即停止，不继续后续步骤 |

---

*附录 A · 完*
