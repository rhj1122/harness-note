# Zustand 状态管理入门指南

> 面向从未使用过 Zustand 的前端工程师，结合 aiMiniChat 项目的实际代码。

---

## 一、Zustand 是什么

Zustand 是一个轻量级的 React 状态管理库。

```
对比其他方案：
  Redux：概念多（action、reducer、dispatch、middleware），样板代码多
  Context：简单场景够用，但性能差（任何状态变化都会导致所有消费者重渲染）
  Zustand：API 极简，性能好，不需要 Provider 包裹
```

Zustand 的核心理念：**一个函数创建 Store，一个 Hook 消费 Store，没了。**

---

## 二、基本用法

### 创建 Store

```typescript
import { create } from 'zustand'

// create<类型>() 创建一个 Store
// 参数是一个函数，接收 set，返回初始状态和方法
const useCounterStore = create<{
  count: number
  increment: () => void
  decrement: () => void
}>()(set => ({
  // 初始状态
  count: 0,

  // 方法（通过 set 更新状态）
  increment: () => set(state => ({ count: state.count + 1 })),
  decrement: () => set(state => ({ count: state.count - 1 })),
}))
```

### 在组件中使用

```typescript
function Counter() {
  // 直接调用 useCounterStore，就像普通的 Hook
  const { count, increment, decrement } = useCounterStore()

  return (
    <div>
      <p>{count}</p>
      <button onClick={increment}>+1</button>
      <button onClick={decrement}>-1</button>
    </div>
  )
}
```

**不需要 Provider 包裹。** 这是 Zustand 和 Context/Redux 最大的区别——任何组件直接调用 Hook 就能用。

---

## 三、set 函数详解

`set` 是 Zustand 提供的状态更新函数，有两种用法：

### 用法一：传对象（直接覆盖）

```typescript
// 直接传一个对象，Zustand 会浅合并到当前状态
set({ count: 10 })

// 等价于：state = { ...state, count: 10 }
// 其他字段不受影响
```

### 用法二：传函数（基于当前状态计算）

```typescript
// 传一个函数，参数是当前状态，返回要更新的部分
set(state => ({ count: state.count + 1 }))

// 需要读取当前状态时用这种写法
// 比如 increment 需要知道当前 count 是多少
```

### 项目中的实际例子

```typescript
// src/store/configStore.ts
setConfig: config => set({ config, isLoaded: true }),
// 直接传对象：同时更新 config 和 isLoaded

// src/store/chatStore.ts
addMessage: message =>
  set(state => ({
    messages: [...state.messages, message],
  })),
// 传函数：需要读取当前 messages 数组，在末尾追加新消息
```

---

## 四、项目中的两个 Store

### configStore（简单 Store）

```typescript
// src/store/configStore.ts

import { create } from 'zustand'

interface ConfigStore {
  config: Record<string, unknown> | null  // 全局配置数据
  isLoaded: boolean                       // 是否已加载
  setConfig: (config: Record<string, unknown>) => void  // 设置配置
}

export const useConfigStore = create<ConfigStore>()(set => ({
  config: null,
  isLoaded: false,
  setConfig: config => set({ config, isLoaded: true }),
}))
```

使用场景：
```typescript
// 在 Hook 中写入（useAppInit.ts）
const { setConfig } = useConfigStore()
setConfig(data)

// 在页面中读取（ChatPage.tsx）
const { config, isLoaded } = useConfigStore()
```

### chatStore（复杂 Store）

```typescript
// src/store/chatStore.ts

import { create } from 'zustand'
import type { Message } from '@/types'

interface ChatStore {
  messages: Message[]
  isLoading: boolean
  addMessage: (message: Message) => void
  updateLastMessage: (content: string) => void
  clearMessages: () => void
  setLoading: (loading: boolean) => void
}

export const useChatStore = create<ChatStore>()(set => ({
  messages: [],
  isLoading: false,

  addMessage: message =>
    set(state => ({
      messages: [...state.messages, message],
    })),

  updateLastMessage: content =>
    set(state => {
      const messages = [...state.messages]
      const last = messages[messages.length - 1]
      if (last) {
        messages[messages.length - 1] = { ...last, content }
      }
      return { messages }
    }),

  clearMessages: () => set({ messages: [] }),

  setLoading: loading => set({ isLoading: loading }),
}))
```

---

## 五、性能优化：选择性订阅

### 问题

```typescript
// 这样写，组件会在 Store 的任何字段变化时都重渲染
const { messages, isLoading } = useChatStore()
// messages 变了 → 重渲染 ✅
// isLoading 变了 → 重渲染 ✅（即使这个组件不关心 isLoading）
```

### 解法：用选择器只订阅需要的字段

```typescript
// 只订阅 messages，isLoading 变化时不会重渲染
const messages = useChatStore(state => state.messages)

// 只订阅 isLoading
const isLoading = useChatStore(state => state.isLoading)

// 订阅多个字段（用对象返回）
const { messages, isLoading } = useChatStore(state => ({
  messages: state.messages,
  isLoading: state.isLoading,
}))
```

### 什么时候需要选择器

```
小组件 / 字段少的 Store：
  直接解构就行，不需要选择器
  const { config, isLoaded } = useConfigStore()

大组件 / 字段多的 Store / 频繁更新的 Store：
  用选择器避免不必要的重渲染
  const messages = useChatStore(state => state.messages)
```

---

## 六、在组件外部使用 Store

Zustand 的 Store 不仅能在 React 组件里用，也能在普通函数里用：

```typescript
// 在组件外部读取状态
const currentMessages = useChatStore.getState().messages

// 在组件外部更新状态
useChatStore.getState().addMessage(newMessage)

// 直接设置状态（跳过 action）
useChatStore.setState({ isLoading: true })
```

### 什么时候用 getState

```
在 React 组件/Hook 里：
  用 useChatStore()（Hook 方式，自动响应状态变化）

在非 React 环境里（工具函数、测试、API 回调等）：
  用 useChatStore.getState()（直接读取，不会触发重渲染）
```

### 测试中的用法

```typescript
// 测试文件里经常用 getState 和 setState
beforeEach(() => {
  // 重置状态
  useChatStore.setState({ messages: [], isLoading: false })
})

test('addMessage 应该添加消息', () => {
  // 调用方法
  useChatStore.getState().addMessage({ ... })

  // 读取状态验证
  expect(useChatStore.getState().messages).toHaveLength(1)
})
```

---

## 七、TypeScript 类型定义

### 基本模式

```typescript
// 1. 先定义 interface
interface MyStore {
  // 状态
  data: string
  count: number

  // 方法
  setData: (data: string) => void
  increment: () => void
}

// 2. 传给 create<MyStore>()
export const useMyStore = create<MyStore>()(set => ({
  data: '',
  count: 0,
  setData: data => set({ data }),
  increment: () => set(state => ({ count: state.count + 1 })),
}))
```

### 为什么有两个 ()

```typescript
create<MyStore>()(set => ...)
//             ^^ 这两个括号

// 第一个 () 是泛型参数的调用
// 第二个 () 是传入 Store 定义函数
// 这是 TypeScript 的类型推断限制，固定写法，记住就行
```

---

## 八、常见模式

### 模式一：异步操作

```typescript
interface UserStore {
  user: User | null
  isLoading: boolean
  fetchUser: (id: string) => Promise<void>
}

export const useUserStore = create<UserStore>()(set => ({
  user: null,
  isLoading: false,

  fetchUser: async (id) => {
    set({ isLoading: true })
    try {
      const user = await api.getUser(id)
      set({ user, isLoading: false })
    } catch {
      set({ isLoading: false })
    }
  },
}))
```

**注意：** 在 aiMiniChat 项目中，API 调用应该放在 Hooks 层，不放在 Store 层。Store 只存储状态和简单的同步操作。

### 模式二：计算属性（派生状态）

```typescript
// Zustand 没有内置的计算属性，用选择器实现
const unreadCount = useChatStore(state =>
  state.messages.filter(m => !m.isRead).length
)
```

### 模式三：重置 Store

```typescript
interface ChatStore {
  messages: Message[]
  isLoading: boolean
  // ... 其他字段
  reset: () => void
}

const initialState = {
  messages: [],
  isLoading: false,
}

export const useChatStore = create<ChatStore>()(set => ({
  ...initialState,
  // ... 方法
  reset: () => set(initialState),
}))
```

---

## 九、Zustand vs useState 的选择

```
用 useState：
  ✅ 状态只在一个组件内使用
  ✅ 状态不需要跨组件共享
  例：输入框的 value、弹窗的 open/close

用 Zustand Store：
  ✅ 状态需要在多个组件间共享
  ✅ 状态需要在 Hook 或非 React 环境中访问
  例：用户登录态、消息列表、全局配置
```

项目中的黄金原则（docs/golden-principles.md）也提到了这一点：
> Store 只存储需要跨组件共享的状态，局部状态用 useState。

---

## 十、常见问题

### Q：Store 的状态修改后组件没有更新？

检查是否用了 `getState()` 而不是 Hook：
```typescript
// ❌ getState 不会触发重渲染
const messages = useChatStore.getState().messages

// ✅ Hook 方式会自动响应变化
const messages = useChatStore(state => state.messages)
```

### Q：set 更新后立即读取还是旧值？

`set` 是同步的，但 React 的重渲染是异步的：
```typescript
set({ count: 10 })
// 这里 getState().count 已经是 10 了
// 但组件还没重渲染，组件里的 count 还是旧值
// 下一次渲染时组件才会拿到新值
```

### Q：多个 set 调用会触发多次渲染吗？

React 18+ 会自动批处理，多个 set 只触发一次渲染：
```typescript
// 只触发一次重渲染
set({ isLoading: true })
set({ messages: [...messages, newMsg] })
```

但如果想确保原子更新，可以在一个 set 里同时更新：
```typescript
set({ isLoading: true, messages: [...messages, newMsg] })
```
