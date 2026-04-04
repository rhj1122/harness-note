# 附录 E　Harness 设计自检清单

> 用于验证 Harness 是否完整、有效。建议在项目初始化完成后和每次 Harness Review 时使用。

---

## 一、上下文工程自检

### AGENTS.md
- [ ] AGENTS.md 存在于项目根目录
- [ ] 行数控制在 100 行以内
- [ ] 包含项目概述（1-2句话）
- [ ] 包含架构概览（分层结构 + 依赖方向）
- [ ] 包含关键规则（3-5条，直接写，不跳转）
- [ ] 包含常用命令（dev / check / test / build）
- [ ] 包含文档入口（指向 docs/README.md）
- [ ] 内容与当前代码库实际情况一致（无过时信息）

### docs/ 目录
- [ ] docs/README.md 存在，列出了所有文档的分类和链接
- [ ] docs/architecture.md 存在，描述了分层结构和依赖规则
- [ ] docs/conventions.md 存在，描述了编码规范
- [ ] docs/tech-choices.md 存在，列出了技术选型和禁用清单
- [ ] docs/golden-principles.md 存在，包含黄金原则
- [ ] 所有文档中的文件路径引用都是有效的
- [ ] 所有文档中的链接都是有效的（无死链）

---

## 二、架构约束自检

### ESLint 配置
- [ ] ESLint 已配置（eslint.config.js 存在）
- [ ] TypeScript 类型检查已配置（@typescript-eslint）
- [ ] 至少有 3 条自定义 Harness 架构规则
- [ ] 每条规则的错误信息包含：违反了什么 + 为什么 + 怎么修
- [ ] 关键规则设为 error（不是 warn）
- [ ] npm run check 命令能正常运行（tsc + ESLint）

### 架构边界
- [ ] 分层架构已在 docs/architecture.md 中文档化
- [ ] 各层的目录结构与文档描述一致
- [ ] Providers 层已建立（AIProvider、AuthProvider 等）
- [ ] 运行 `npx madge --circular src` 无循环依赖

---

## 三、熵管理自检

### 黄金原则
- [ ] 黄金原则文档存在（docs/golden-principles.md）
- [ ] 每条原则包含：规则 + 为什么 + 反例
- [ ] 原则针对真实出现过的问题（不是凭空想象）
- [ ] 原则与当前技术栈一致（无过时原则）

### 质量度量
- [ ] 测试覆盖率 > 70%（Hooks 层 > 80%）
- [ ] 运行 `npx ts-prune` 无大量未使用导出
- [ ] 运行 `npx jscpd src` 重复率 < 5%
- [ ] 无超过 150 行的组件文件
- [ ] 无超过 100 行的 Hook 文件

---

## 四、可观测性自检

### 本地验证工具
- [ ] `npm run check` 能正常运行（tsc + ESLint）
- [ ] `npx vitest run` 能正常运行
- [ ] `npm run build` 能正常运行
- [ ] 以上三个命令全部通过（无错误）

### CI 流水线
- [ ] CI 配置文件存在（.github/workflows/ 或 .gitlab-ci.yml）
- [ ] CI 包含：格式检查 + 类型检查 + ESLint + 测试 + 构建
- [ ] CI 在 PR 提交时自动触发
- [ ] CI 失败时 PR 无法合并（分支保护规则已配置）

---

## 五、Agent 工作验证

### 给 Agent 一个测试任务，验证 Harness 是否有效

```
测试任务：
  "在 src/components/ 下创建一个 HelloWorld 组件，
   显示文字'Hello Harness'，
   运行 npm run check 和 npx vitest run 确认通过。"

验证点：
  □ Agent 是否读取了 AGENTS.md？
  □ Agent 是否把文件放在了正确的目录？
  □ Agent 是否主动运行了 npm run check？
  □ 生成的代码是否符合架构规则（ESLint 通过）？
  □ 生成的代码风格是否与项目一致？
```

---

## 六、Harness Review 自检（每 2 周）

- [ ] AGENTS.md 是否还准确反映当前项目状态？
- [ ] 有没有 ESLint 规则检测的违规模式已经不存在了？
- [ ] 有没有新的违规模式需要新增 ESLint 规则？
- [ ] 黄金原则是否还适用于当前技术栈？
- [ ] docs/ 中有没有过时的文档需要更新？
- [ ] 质量指标趋势是否健康（稳定或改善）？

---

## 七、评分参考

```
满分：所有项目都勾选
  → Harness 完整，可以放心使用 AI Agent

60-80分（大部分勾选）：
  → Harness 基本可用，但有改进空间
  → 优先补全未勾选的高优先级项

40-60分（约一半勾选）：
  → Harness 还不完整，Agent 犯错率会较高
  → 建议先补全基础项（AGENTS.md + ESLint + CI）

40分以下：
  → 建议按 8.3 节的渐进式迁移方案，逐步建立 Harness
```

---

*附录 E · 完*
