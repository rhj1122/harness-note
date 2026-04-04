# 附录 B　AGENTS.md 最小模板

> 适用于个人项目或项目初期，约 25-30 行，可直接复制使用。

---

## 模板（前端 AI H5 项目版）

```markdown
# <项目名>

## 项目概述
<一到两句话：这个项目是什么，用什么技术栈>

## 架构
分层：Types → Config → API → Store → Hooks → Components → Pages
依赖只能向下，不能向上。详见 docs/architecture.md

## 关键规则
- Components 层禁止直接调用 fetch 或 AI SDK
- Pages 层禁止直接操作 Store（必须通过 Hooks）
- AI SDK 统一封装在 src/providers/AIProvider.tsx
- 所有网络请求统一封装在 src/api/ 目录
- 开始实现新功能前，先搜索代码库确认是否已有类似实现

## 常用命令
npm run dev          # 启动开发服务器
npm run check        # tsc + ESLint 检查（每次修改后必须运行）
npx vitest run       # 运行测试
npm run build        # 构建

## 文档
docs/README.md（所有文档的入口）
```

---

## 填写说明

### 项目概述
用一两句话回答：这个项目是什么？给谁用的？用什么技术栈？

```markdown
# 示例
这是一个内嵌在 App WebView 中的 AI 对话 H5 页面。
React 18 + TypeScript + Zustand + Vite，通过 OpenAI API 实现流式对话。
```

### 架构
描述你的分层结构。如果和标准七层不同，按实际情况写。

```markdown
# 示例（简化版，只有五层）
分层：Types → API → Store → Hooks → Components/Pages
依赖只能向下。详见 docs/architecture.md
```

### 关键规则
只写最重要的 3-5 条，超过 5 条就考虑移到 docs/conventions.md。

规则写法原则：
- 可执行（能判断是否违反）
- 有理由（一句话说明为什么）
- 有指向（违反时去哪里找正确做法）

```markdown
# 好的规则写法
- Components 层禁止直接调用 fetch（数据获取必须在 Hooks 层）

# 差的规则写法
- 保持代码整洁（太模糊，无法执行）
```

### 常用命令
把 Agent 验证工作时需要的命令都列出来。Agent 需要这些来确认自己的修改是否正确。

---

## 扩展版模板（小团队，约 60-80 行）

```markdown
# <项目名>

## 项目概述
<一到两句话>
技术栈：<框架> + <语言> + <状态管理> + <构建工具>
仓库：<GitHub/GitLab 地址>

## 架构
分层：Types → Config → API → Store → Hooks → Components → Pages
依赖只能向下，不能向上。

目录结构：
  src/types/      ← 类型定义（最底层，无依赖）
  src/config/     ← 配置（环境变量）
  src/api/        ← 网络请求封装
  src/store/      ← Zustand 状态管理
  src/hooks/      ← 自定义 Hook（业务逻辑）
  src/components/ ← UI 组件
  src/pages/      ← 页面
  src/providers/  ← 横切关注点（Auth、AI、Tracking）

详细架构：docs/architecture.md

## 关键规则
- Components 层禁止直接调用 fetch 或 AI SDK
- Pages 层禁止直接操作 Store（必须通过 Hooks）
- AI SDK 统一封装在 src/providers/AIProvider.tsx
- 所有网络请求统一封装在 src/api/ 目录
- Hook 只返回数据和方法，不返回 JSX
- 开始实现新功能前，先搜索代码库确认是否已有类似实现

完整规范：docs/conventions.md

## 常用命令
npm run dev          # 启动开发服务器（端口 5173）
npm run check        # tsc + ESLint（每次修改后必须运行）
npx vitest run       # 运行测试（单次，不进入 watch 模式）
npx vitest run --coverage  # 运行测试 + 覆盖率报告
npm run build        # 构建生产版本
npx madge --circular src   # 检测循环依赖

## 验证流程
修改代码后，按顺序运行：
1. npm run check（静态检查）
2. npx vitest run（测试）
3. npm run build（构建验证，修改 API/类型时必须）

## 文档
docs/README.md（所有文档的入口）
```

---

*附录 B · 完*
