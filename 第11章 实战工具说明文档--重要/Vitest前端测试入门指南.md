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

### 类型四：React Hook 测试

Hook 不能直接调用（必须在 React 组件内部使用），所以需要用 `renderHook` 来测试。

`renderHook` 来自 `@testing-library/react`，它会创建一个临时组件来运行你的 Hook。

#### 基础用法

```typescript
// src/hooks/useCounter.ts
import { useState, useCallback } from 'react'

export function useCounter(initial = 0) {
  const [count, setCount] = useState(initial)
  const increment = useCallback(() => setCount(c => c + 1), [])
  const decrement = useCallback(() => setCount(c => c - 1), [])
  return { count, increment, decrement }
}

// src/hooks/useCounter.test.ts
import { renderHook, act } from '@testing-library/react'
import { useCounter } from './useCounter'

describe('useCounter', () => {
  it('初始值应该为 0', () => {
    const { result } = renderHook(() => useCounter())
    //      ^^^^^^ result.current 就是 Hook 的返回值
    expect(result.current.count).toBe(0)
  })

  it('increment 应该加 1', () => {
    const { result } = renderHook(() => useCounter())

    // act() 包裹所有会触发状态更新的操作
    act(() => {
      result.current.increment()
    })

    expect(result.current.count).toBe(1)
  })

  it('应该支持自定义初始值', () => {
    const { result } = renderHook(() => useCounter(10))
    expect(result.current.count).toBe(10)
  })
})
```

#### 关键概念

```
renderHook(() => useXxx())
  → 返回 { result }
  → result.current 就是 Hook 当前的返回值
  → 每次状态更新后，result.current 会自动更新
```

#### act() 详解

**act 是什么：** React 要求所有导致状态更新的操作都必须包裹在 `act()` 里。这是 React 的规则，不是 Vitest 的。

**为什么需要 act：** React 的状态更新是异步批处理的。如果不用 `act()` 包裹，你调用了 `increment()`，但 `result.current.count` 可能还没更新，断言就会失败。

```typescript
// ❌ 不用 act：状态可能还没更新，断言可能失败
result.current.increment()
expect(result.current.count).toBe(1)  // 可能还是 0！

// ✅ 用 act：确保状态更新完成后再断言
act(() => {
  result.current.increment()
})
expect(result.current.count).toBe(1)  // 一定是 1
```

**什么时候需要 act：**

```typescript
// 需要 act 的情况：调用了会触发 setState 的操作
act(() => {
  result.current.increment()     // Hook 内部调用了 setState
})

act(() => {
  result.current.sendMessage('你好')  // Hook 内部调用了 setState
})

act(() => {
  fireEvent.click(button)        // 点击按钮触发了组件的 setState
})

// 不需要 act 的情况：只是读取值，没有触发状态更新
const { result } = renderHook(() => useCounter())
expect(result.current.count).toBe(0)  // 只是读取初始值，不需要 act
```

**act 和 async 的组合：** 如果操作是异步的（比如 API 调用），用 `async act`：

```typescript
// 同步操作：普通 act
act(() => {
  result.current.increment()
})

// 异步操作：async act
await act(async () => {
  await result.current.sendMessage('你好')
  // sendMessage 内部有 await fetch(...)
  // 需要等异步操作完成后，状态才会更新
})
```

**完整的 act 示例：**

```typescript
import { renderHook, act } from '@testing-library/react'
import { useCounter } from './useCounter'

describe('useCounter', () => {
  it('连续调用 increment 三次应该加 3', () => {
    const { result } = renderHook(() => useCounter())

    // 可以在一个 act 里做多次操作
    act(() => {
      result.current.increment()
      result.current.increment()
      result.current.increment()
    })

    // act 结束后，所有状态更新都已完成
    expect(result.current.count).toBe(3)
  })

  it('先加后减应该回到原值', () => {
    const { result } = renderHook(() => useCounter(5))

    act(() => {
      result.current.increment()  // 5 → 6
    })
    expect(result.current.count).toBe(6)

    act(() => {
      result.current.decrement()  // 6 → 5
    })
    expect(result.current.count).toBe(5)
  })
})
```

---

#### Mock（模拟）详解

**Mock 代码写在哪里：** 全部写在 `xxx.test.ts` 测试文件里，不会影响源代码。

**vi 是什么，需要 import 吗：**

`vi` 是 Vitest 提供的全局工具对象（类似 Jest 的 `jest`）。因为 `vite.config.ts` 里配置了 `globals: true`，所以 `vi` 和 `describe`、`it`、`expect` 一样，是全局可用的，**不需要 import**。

```typescript
// vite.config.ts 里的这行配置：
// globals: true
// 让以下变量全局可用，不需要 import：
//   describe, it, test, expect, vi, beforeEach, afterEach

// 所以测试文件里可以直接用：
vi.fn()
vi.mock(...)
vi.mocked(...)

// 如果没有配置 globals: true，就需要手动 import：
// import { vi, describe, it, expect } from 'vitest'
```

**一个完整的测试文件长什么样：**

```typescript
// src/hooks/useAIChat.test.ts

// ① import 测试工具（renderHook、act 来自 testing-library，需要 import）
import { renderHook, act } from '@testing-library/react'

// ② import 被测试的 Hook
import { useAIChat } from './useAIChat'

// ③ import 需要断言的 mock 函数（用于验证是否被调用）
import { streamMessage } from '@/api/aiApi'

// ④ Mock 声明（vi 是全局的，不需要 import）
//    虽然写在 import 后面，但 Vitest 会自动提升到最前面执行
vi.mock('@/api/aiApi', () => ({
  streamMessage: vi.fn(),
  sendMessage: vi.fn(),
}))

// ⑤ 测试用例
describe('useAIChat', () => {
  // ⑥ 每个测试前重置（清除上一个测试的 mock 调用记录）
  beforeEach(() => {
    vi.clearAllMocks()
  })

  it('初始状态应该没有消息', () => {
    // ⑦ 测试代码
  })
})
```

**总结：**
- `vi`、`describe`、`it`、`expect`、`beforeEach` → 全局可用，不需要 import
- `renderHook`、`act`、`waitFor` → 来自 `@testing-library/react`，需要 import
- `render`、`screen`、`fireEvent` → 来自 `@testing-library/react`，需要 import
- Mock 代码只存在于 `.test.ts` 文件里，不影响源代码

---

**Mock 解决什么问题：**

```
你的 useAIChat Hook 内部调用了 streamMessage（发 WebSocket 请求）。
测试时如果真的发请求：
  - 需要启动 VoltAgent 服务器（麻烦）
  - 网络不稳定时测试会随机失败（不可靠）
  - 每次跑测试都要等网络响应（慢）

Mock 的做法：
  用一个"假的 streamMessage"替代真的
  这个假函数不发请求，直接返回你指定的数据
  测试只验证 Hook 的逻辑，不验证网络请求
```

**vi.fn()：创建假函数**

```typescript
// 创建一个假函数
const mockFn = vi.fn()

// 调用它
mockFn('hello')
mockFn('world')

// 断言它被调用过
expect(mockFn).toHaveBeenCalled()           // ✅ 被调用过
expect(mockFn).toHaveBeenCalledTimes(2)     // ✅ 被调用了 2 次
expect(mockFn).toHaveBeenCalledWith('hello') // ✅ 第一次调用时传了 'hello'
```

**vi.fn() 控制返回值：**

```typescript
// 返回固定值
const mockAdd = vi.fn().mockReturnValue(42)
mockAdd(1, 2)  // 返回 42（不管传什么参数）

// 返回 Promise（模拟异步函数）
const mockFetch = vi.fn().mockResolvedValue({ name: '张三' })
await mockFetch()  // 返回 Promise.resolve({ name: '张三' })

// 返回 rejected Promise（模拟请求失败）
const mockFetchError = vi.fn().mockRejectedValue(new Error('网络错误'))
await mockFetchError()  // 抛出 Error('网络错误')

// 自定义实现（完全控制行为）
const mockCalc = vi.fn().mockImplementation((a, b) => a * b)
mockCalc(3, 4)  // 返回 12
```

**vi.mock()：替换整个模块**

这是最常用的 Mock 方式——把一个模块里的所有导出替换成假的。

```typescript
// 假设 src/api/aiApi.ts 导出了两个函数：
// export async function sendMessage(messages) { ... }
// export async function* streamMessage(messages) { ... }

// 在测试文件顶部，用 vi.mock 替换整个模块
vi.mock('@/api/aiApi', () => ({
  sendMessage: vi.fn().mockResolvedValue({
    id: 'msg-1',
    role: 'agent',
    content: '你好！',
    timestamp: Date.now(),
  }),
  streamMessage: vi.fn(),
}))

// 现在测试文件里 import 的 sendMessage 和 streamMessage 都是假的
// useAIChat 内部调用 streamMessage 时，不会真的发 WebSocket 请求
```

**vi.mock 的执行时机：**

```typescript
// vi.mock 会被 Vitest 自动提升到文件顶部执行
// 所以不管你写在哪里，它都会在所有 import 之前生效

import { useAIChat } from './useAIChat'  // 这里 import 时，aiApi 已经被 mock 了

vi.mock('@/api/aiApi', () => ({          // 虽然写在 import 后面
  streamMessage: vi.fn(),                // 但实际上在 import 之前就执行了
}))
```

**在测试中获取和操作 mock 函数：**

```typescript
import { streamMessage } from '@/api/aiApi'

vi.mock('@/api/aiApi', () => ({
  streamMessage: vi.fn(),
}))

describe('useAIChat', () => {
  beforeEach(() => {
    // 每个测试前重置 mock（清除调用记录和返回值）
    vi.clearAllMocks()
  })

  it('sendMessage 应该调用 streamMessage', async () => {
    // 为这个测试指定 mock 的行为
    // vi.mocked() 把 streamMessage 的类型转为 Mock 类型，方便调用 mock 方法
    vi.mocked(streamMessage).mockImplementation(async function* () {
      yield '你'
      yield '好'
      yield '！'
    })

    const { result } = renderHook(() => useAIChat())

    await act(async () => {
      await result.current.sendMessage('你好')
    })

    // 验证 streamMessage 被调用了
    expect(streamMessage).toHaveBeenCalled()

    // 验证消息列表更新了
    expect(result.current.messages.length).toBeGreaterThan(0)
  })

  it('streamMessage 失败时应该显示错误消息', async () => {
    // 模拟请求失败
    vi.mocked(streamMessage).mockImplementation(async function* () {
      throw new Error('连接失败')
    })

    const { result } = renderHook(() => useAIChat())

    await act(async () => {
      await result.current.sendMessage('你好')
    })

    // 验证错误被处理了（不是直接崩溃）
    const lastMessage = result.current.messages[result.current.messages.length - 1]
    expect(lastMessage.content).toContain('错误')
  })
})
```

**Mock 的常用操作速查：**

```typescript
// 创建
vi.fn()                                    // 创建假函数
vi.mock('模块路径', () => ({ ... }))        // 替换整个模块
vi.mocked(realFn)                          // 获取已 mock 的函数（带类型）

// 控制返回值
.mockReturnValue(value)                    // 同步返回
.mockResolvedValue(value)                  // 异步返回（Promise.resolve）
.mockRejectedValue(error)                 // 异步失败（Promise.reject）
.mockImplementation(fn)                    // 自定义实现

// 断言
expect(mockFn).toHaveBeenCalled()          // 被调用过
expect(mockFn).toHaveBeenCalledTimes(n)    // 被调用了 n 次
expect(mockFn).toHaveBeenCalledWith(arg)   // 被调用时传了 arg

// 重置
vi.clearAllMocks()                         // 清除所有 mock 的调用记录
vi.resetAllMocks()                         // 清除调用记录 + 重置返回值
vi.restoreAllMocks()                       // 恢复原始实现
```

#### 测试需要 Provider 的 Hook

如果 Hook 内部用了 Context（比如 `useAIProvider()`），需要用 `wrapper` 提供 Provider：

```typescript
import { renderHook } from '@testing-library/react'
import { AIProvider } from '@/providers/AIProvider'
import { useAIChat } from './useAIChat'

describe('useAIChat', () => {
  it('初始状态应该没有消息', () => {
    const { result } = renderHook(() => useAIChat(), {
      wrapper: AIProvider,
      // wrapper 会把 Hook 包裹在 <AIProvider> 里面
      // 这样 Hook 内部的 useAIProvider() 就能正常工作
    })

    expect(result.current.messages).toHaveLength(0)
    expect(result.current.isStreaming).toBe(false)
  })
})
```

如果需要多个 Provider，可以组合：

```typescript
function AllProviders({ children }: { children: React.ReactNode }) {
  return (
    <BrowserRouter>
      <AIProvider>
        {children}
      </AIProvider>
    </BrowserRouter>
  )
}

const { result } = renderHook(() => useAIChat(), {
  wrapper: AllProviders,
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
