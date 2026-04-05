# Vitest 前端测试入门指南

> 面向从未写过测试的前端工程师，结合 aiMiniChat 项目的实际配置。

---

## 一、测试是什么，为什么要写

### 最简单的理解

```
你写了一个函数：
  function add(a, b) { return a + b }

你怎么确认它是对的？
  手动验证：打开浏览器，console.log(add(1, 2))，看到 3，觉得没问题
  自动测试：写一段代码，让机器帮你验证

测试代码：
  expect(add(1, 2)).toBe(3)    // 1+2 应该等于 3
  expect(add(-1, 1)).toBe(0)   // -1+1 应该等于 0
  expect(add(0, 0)).toBe(0)    // 0+0 应该等于 0

好处：
  以后改了 add 函数，跑一下测试就知道有没有改坏
  不需要每次都手动验证
```

### 在 Harness Engineering 里的角色

```
Agent 修改了代码
    ↓
Agent 运行测试
    ↓
测试通过 → 代码大概率没问题
测试失败 → Agent 读到失败信息，自己修复
    ↓
不需要人介入
```

**测试是 Agent 自我验证的核心工具。** 没有测试，Agent 写完代码不知道对不对。

---

## 二、Vitest 是什么

Vitest 是一个测试框架，专门为 Vite 项目设计。

```
为什么选 Vitest 而不是 Jest：
  - 和 Vite 共享配置（不需要单独配置 babel/webpack）
  - 原生支持 TypeScript 和 ESM
  - 速度更快
  - API 和 Jest 几乎一样（学了 Vitest 也等于学了 Jest）
```

---

## 三、项目里已有的配置

你的项目已经配置好了 Vitest，不需要额外安装。

### vite.config.ts 里的测试配置

```typescript
test: {
  environment: 'jsdom',
  // jsdom 模拟浏览器环境（DOM、window、document 等）
  // 因为测试在 Node.js 里跑，但前端代码需要浏览器 API

  globals: true,
  // 全局注入 describe、it、expect 等函数
  // 不需要每个测试文件都 import { describe, it, expect } from 'vitest'

  setupFiles: ['./src/test/setup.ts'],
  // 在所有测试运行前执行的文件
  // 目前只有 import '@testing-library/jest-dom'
  // 这行代码给 expect 添加了 DOM 相关的匹配器（toBeInTheDocument 等）

  passWithNoTests: true,
  // 没有测试文件时不报错（正常退出）

  coverage: {
    provider: 'v8',
    // 使用 V8 引擎的覆盖率统计（速度快）

    reporter: ['text', 'json-summary'],
    // text：在终端输出覆盖率表格
    // json-summary：生成 JSON 文件供脚本读取

    thresholds: {
      lines: 70,
      // 代码行覆盖率必须 >= 70%，否则报错
    },
  },
},
```

### package.json 里的命令

```json
"test": "vitest",           // 启动 watch 模式（文件变化自动重跑）
"test:run": "vitest run",   // 单次运行（跑完就退出，CI 用这个）
"test:coverage": "vitest run --coverage"  // 单次运行 + 覆盖率报告
```

### src/test/setup.ts

```typescript
import '@testing-library/jest-dom'
// 这行代码给 expect 添加了这些 DOM 匹配器：
//   expect(element).toBeInTheDocument()
//   expect(element).toBeVisible()
//   expect(element).toHaveTextContent('xxx')
//   等等
```

---

## 四、测试文件放在哪里

Vitest 默认查找这些文件：

```
**/*.{test,spec}.{ts,tsx}

也就是说，文件名包含 .test. 或 .spec. 的文件都会被当作测试文件。
```

### 推荐的放置方式：和被测文件同目录

```
src/hooks/
  useAIChat.ts           ← 源代码
  useAIChat.test.ts      ← 测试文件（紧挨着源代码）

src/store/
  chatStore.ts
  chatStore.test.ts

src/components/
  ChatInput.tsx
  ChatInput.test.tsx
```

好处：
- 一眼就知道哪个文件有测试、哪个没有
- 改了源代码，测试文件就在旁边，顺手就改了

---

## 五、测试文件的基本结构

```typescript
// src/store/chatStore.test.ts

import { useChatStore } from './chatStore'

// describe：一组相关的测试
describe('chatStore', () => {

  // beforeEach：每个测试运行前执行（重置状态）
  beforeEach(() => {
    useChatStore.setState({ messages: [], isLoading: false })
  })

  // it（或 test）：一个具体的测试用例
  it('addMessage 应该把消息加入列表', () => {
    const { addMessage } = useChatStore.getState()

    addMessage({
      id: '1',
      role: 'user',
      content: '你好',
      timestamp: Date.now(),
    })

    const { messages } = useChatStore.getState()
    expect(messages).toHaveLength(1)
    expect(messages[0].content).toBe('你好')
  })

  it('clearMessages 应该清空消息列表', () => {
    const { addMessage, clearMessages } = useChatStore.getState()

    addMessage({ id: '1', role: 'user', content: '你好', timestamp: Date.now() })
    clearMessages()

    const { messages } = useChatStore.getState()
    expect(messages).toHaveLength(0)
  })
})
```

---

## 六、核心 API 速查

### describe / it / test

```typescript
describe('分组名称', () => {
  it('测试描述', () => {
    // 测试代码
  })
})

// it 和 test 完全等价，只是写法不同
// it('should do something') 读起来像英语句子
// test('做某件事') 更直白
```

### expect（断言）

```typescript
// 基础断言
expect(1 + 1).toBe(2)              // 严格相等（===）
expect({ a: 1 }).toEqual({ a: 1 }) // 深度相等（对象/数组用这个）

// 真假判断
expect(true).toBeTruthy()
expect(false).toBeFalsy()
expect(null).toBeNull()
expect(undefined).toBeUndefined()

// 数组
expect([1, 2, 3]).toHaveLength(3)
expect([1, 2, 3]).toContain(2)

// 字符串
expect('hello world').toContain('hello')
expect('hello').toMatch(/^hel/)

// 取反
expect(1).not.toBe(2)

// DOM 断言（来自 @testing-library/jest-dom）
expect(element).toBeInTheDocument()
expect(element).toHaveTextContent('你好')
expect(element).toBeVisible()
expect(element).toBeDisabled()
```

### beforeEach / afterEach

```typescript
describe('chatStore', () => {
  beforeEach(() => {
    // 每个 it 运行前执行
    // 通常用来重置状态，保证测试之间互不影响
  })

  afterEach(() => {
    // 每个 it 运行后执行
    // 通常用来清理副作用
  })
})
```

---

## 七、三种测试类型

### 类型一：纯函数测试（最简单）

测试 `src/types/` 或 `src/utils/` 里的纯函数。

```typescript
// src/utils/formatTime.ts
export function formatTime(timestamp: number): string {
  return new Date(timestamp).toLocaleTimeString()
}

// src/utils/formatTime.test.ts
import { formatTime } from './formatTime'

describe('formatTime', () => {
  it('应该返回格式化的时间字符串', () => {
    const result = formatTime(1712300000000)
    expect(typeof result).toBe('string')
    expect(result.length).toBeGreaterThan(0)
  })
})
```

### 类型二：Store 测试

测试 Zustand Store 的状态变化。

```typescript
// src/store/chatStore.test.ts
import { useChatStore } from './chatStore'

describe('chatStore', () => {
  beforeEach(() => {
    // 重置 Store 状态
    useChatStore.setState({ messages: [], isLoading: false })
  })

  it('addMessage 应该添加消息', () => {
    const { addMessage } = useChatStore.getState()
    addMessage({ id: '1', role: 'user', content: '你好', timestamp: Date.now() })

    expect(useChatStore.getState().messages).toHaveLength(1)
  })

  it('setLoading 应该更新 loading 状态', () => {
    const { setLoading } = useChatStore.getState()
    setLoading(true)

    expect(useChatStore.getState().isLoading).toBe(true)
  })
})
```

### 类型三：React 组件测试

使用 React Testing Library 测试组件的渲染和交互。

```typescript
// src/pages/LoginPage.test.tsx
import { render, screen } from '@testing-library/react'
import LoginPage from './LoginPage'

describe('LoginPage', () => {
  it('应该渲染登录标题', () => {
    render(<LoginPage />)

    // screen.getByText 查找页面上包含指定文字的元素
    expect(screen.getByText('登录')).toBeInTheDocument()
  })
})
```

组件测试需要注意：如果组件用了 React Router（比如 `useNavigate`），需要用 `BrowserRouter` 包裹：

```typescript
import { render, screen } from '@testing-library/react'
import { BrowserRouter } from 'react-router'
import HomePage from './HomePage'

describe('HomePage', () => {
  it('应该渲染标题', () => {
    render(
      <BrowserRouter>
        <HomePage />
      </BrowserRouter>
    )

    expect(screen.getByText('AI Mini Chat')).toBeInTheDocument()
  })
})
```

---

## 八、运行测试的方式

```bash
# 单次运行所有测试（CI 和 Agent 用这个）
pnpm test:run

# 单次运行指定文件
pnpm test:run src/store/chatStore.test.ts

# watch 模式（开发时用，文件变化自动重跑）
pnpm test

# 运行并生成覆盖率报告
pnpm test:coverage
```

### 测试输出怎么看

```
✓ src/store/chatStore.test.ts (2 tests) 3ms
  ✓ chatStore > addMessage 应该添加消息
  ✓ chatStore > clearMessages 应该清空消息列表

Test Files  1 passed (1)
Tests       2 passed (2)
Duration    0.15s
```

```
✗ src/store/chatStore.test.ts (2 tests) 5ms
  ✓ chatStore > addMessage 应该添加消息
  ✗ chatStore > clearMessages 应该清空消息列表

    AssertionError: expected 1 to be 0
    → 说明 clearMessages 没有正确清空，messages 里还有 1 条

Test Files  1 failed (1)
Tests       1 passed | 1 failed (2)
```

Agent 读到失败信息，就知道哪个测试失败了、期望值是什么、实际值是什么，可以直接修复。

---

## 九、测试的命名规范

```typescript
// 好的命名：描述"做了什么"+"应该怎样"
it('addMessage 应该把消息加入列表', () => {})
it('clearMessages 应该清空消息列表', () => {})
it('sendMessage 在 content 为空时不应该发送', () => {})

// 差的命名：太模糊
it('test addMessage', () => {})
it('works', () => {})
it('测试', () => {})
```

好的命名让测试失败时，光看名字就知道哪里出了问题。

---

## 十、你在 10.4 需要做的事

写以下测试文件：

```
1. src/store/chatStore.test.ts
   测试 chatStore 的所有方法：
   - addMessage
   - updateLastMessage
   - clearMessages
   - setLoading

2. src/pages/LoginPage.test.tsx（可选，练手）
   测试页面是否正确渲染
```

写完后运行 `pnpm test:run`，确认全部通过，然后发给我 review。

---

## 十一、常见问题

### Q：测试文件里能用 `@/` 路径别名吗？

可以，Vitest 共享 Vite 的配置，`@/` 别名在测试文件里也能用。

### Q：测试文件需要 import React 吗？

不需要，`tsconfig.json` 里配置了 `"jsx": "react-jsx"`，自动注入。

### Q：Zustand Store 测试时状态会互相影响吗？

会，所以每个测试前要用 `beforeEach` 重置状态：
```typescript
beforeEach(() => {
  useChatStore.setState({ messages: [], isLoading: false })
})
```

### Q：组件用了 useNavigate 但测试报错？

需要用 `BrowserRouter` 包裹组件，因为 `useNavigate` 依赖 Router Context。
