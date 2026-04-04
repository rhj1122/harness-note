# 附录 C　分层架构约束规则示例

> 前端七层架构的完整约束规则，可直接用于 ESLint 自定义规则和 docs/architecture.md。

---

## 一、前端七层架构定义

```
┌─────────────────────────────────────────┐
│                  Pages                  │  页面层（最顶层）
├─────────────────────────────────────────┤
│               Components               │  组件层
├─────────────────────────────────────────┤
│            Hooks / Composables          │  业务逻辑层
├─────────────────────────────────────────┤
│              Store / State              │  状态管理层
├─────────────────────────────────────────┤
│                   API                   │  接口层
├─────────────────────────────────────────┤
│                  Config                 │  配置层
├─────────────────────────────────────────┤
│                  Types                  │  类型定义层（最底层）
└─────────────────────────────────────────┘

依赖方向：只能向下，不能向上
```

---

## 二、各层职责与约束

### Types 层
```
目录：src/types/
职责：纯 TypeScript 类型定义
约束：
  ✅ 只包含 interface、type、enum
  ❌ 禁止 import 任何其他层的模块
  ❌ 禁止包含业务逻辑或副作用
```

### Config 层
```
目录：src/config/
职责：读取和管理配置（环境变量等）
约束：
  ✅ 只依赖 Types 层
  ❌ 禁止包含业务逻辑
  ❌ 禁止直接调用 API
```

### API 层
```
目录：src/api/
职责：封装所有网络请求和 AI SDK 调用
约束：
  ✅ 只依赖 Types、Config 层
  ✅ 只做请求/响应转换，不包含业务判断
  ❌ 禁止包含 UI 逻辑（不能 import React 组件）
  ❌ 禁止直接操作 Store
```

### Store 层
```
目录：src/store/
职责：管理全局共享状态
约束：
  ✅ 只依赖 API、Types 层
  ❌ 禁止包含 UI 逻辑
  ❌ 禁止直接调用 AI SDK（通过 API 层）
```

### Hooks 层
```
目录：src/hooks/
职责：封装业务逻辑，连接 Store 和 API
约束：
  ✅ 只依赖 Store、API、Types 层
  ✅ 只返回数据和方法，不返回 JSX
  ❌ 禁止直接渲染 UI（不能返回 JSX 元素）
  ❌ 禁止直接调用 AI SDK（通过 API 层）
```

### Components 层
```
目录：src/components/
职责：可复用 UI 组件
约束：
  ✅ 只依赖 Hooks、Types 层
  ❌ 禁止直接调用 fetch 或 AI SDK
  ❌ 禁止直接操作 Store（通过 Hooks）
  ❌ 禁止包含业务逻辑（业务逻辑在 Hooks 层）
```

### Pages 层
```
目录：src/pages/
职责：页面组合，路由入口
约束：
  ✅ 只依赖 Components、Hooks 层
  ❌ 禁止直接操作 Store（通过 Hooks）
  ❌ 禁止直接调用 API（通过 Hooks）
  ❌ 禁止包含业务逻辑
```

---

## 三、Providers 层（横切关注点）

```
目录：src/providers/
职责：封装横切关注点，可被任何层使用

标准 Providers：
  AuthProvider.tsx    ← Token 管理、登录态
  AIProvider.tsx      ← AI SDK 封装，统一调用入口
  RequestProvider.tsx ← HTTP 请求拦截器、统一错误处理
  TrackingProvider.tsx← 埋点/监控
  ThemeProvider.tsx   ← 主题、国际化

约束：
  ✅ 每个 Provider 只做一件事（单一职责）
  ✅ 通过 React Context 或依赖注入使用
  ❌ 禁止在 Provider 层以外直接初始化 AI SDK
  ❌ 禁止在 Provider 层以外直接调用埋点 SDK
```

---

## 四、对应的 ESLint 规则清单

| 规则名 | 检测内容 | 级别 |
|--------|---------|------|
| `no-api-in-components` | Components 层直接 import API 层 | error |
| `no-store-in-pages` | Pages 层直接 import Store 层 | error |
| `no-ai-sdk-in-components` | Components 层直接 import AI SDK | error |
| `no-ai-sdk-in-hooks` | Hooks 层直接 import AI SDK（应通过 AIProvider）| error |
| `no-direct-fetch` | Components/Pages 层直接使用 fetch | error |
| `no-jsx-in-hooks` | Hooks 层返回 JSX 元素 | warn |
| `no-business-logic-in-api` | API 层包含业务判断逻辑 | warn |

---

## 五、docs/architecture.md 模板

```markdown
# 系统架构

## 分层结构

分层：Types → Config → API → Store → Hooks → Components → Pages

依赖规则：每一层只能依赖它下面的层，不能依赖上面的层。

## 各层职责

| 层级 | 目录 | 职责 |
|------|------|------|
| Types | src/types/ | 纯类型定义，无副作用 |
| Config | src/config/ | 配置加载（环境变量）|
| API | src/api/ | 网络请求和 AI SDK 封装 |
| Store | src/store/ | 全局状态管理（Zustand）|
| Hooks | src/hooks/ | 业务逻辑，连接 Store 和 API |
| Components | src/components/ | 可复用 UI 组件 |
| Pages | src/pages/ | 页面组合，路由入口 |

## Providers（横切关注点）

目录：src/providers/

| Provider | 职责 |
|----------|------|
| AIProvider | AI SDK 统一封装 |
| AuthProvider | Token 管理、登录态 |
| RequestProvider | HTTP 请求拦截器 |
| TrackingProvider | 埋点/监控 |

## 违规示例

❌ Components 层直接调用 AI SDK → 应通过 useAIProvider()
❌ Pages 层直接操作 Store → 应通过 Hooks 层
❌ Hooks 层返回 JSX → Hooks 只返回数据和方法
```

---

*附录 C · 完*
