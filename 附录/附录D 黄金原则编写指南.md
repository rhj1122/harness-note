# 附录 D　黄金原则编写指南

> 黄金原则是定义"好代码长什么样"的标准，引导 Agent 生成高质量代码。

---

## 一、黄金原则的三要素

每条黄金原则必须包含：

```
1. 规则本身（一句话，可执行，可验证）
2. 为什么（原因，让 Agent 理解背后的逻辑）
3. 反例（违反时的样子，帮助 Agent 识别问题）
```

---

## 二、编写格式模板

```markdown
### GP-[类别][编号]：[规则标题]

**规则：** [一句话，可执行的规则]

**为什么：** [原因，1-2句话]

**反例：**
```typescript
// ❌ 错误
[违反规则的代码示例]

// ✅ 正确
[符合规则的代码示例]
```
```

---

## 三、前端 AI H5 项目黄金原则示例集

### 组件类（GP-C）

```markdown
### GP-C01：组件文件不超过 150 行

**规则：** 单个组件文件（.tsx）不超过 150 行。

**为什么：** 超过 150 行通常意味着组件承担了过多职责，
应该拆分为更小的子组件或将逻辑提取到 Hook。

**反例：**
// ❌ 错误：ChatPage.tsx 包含了消息列表、输入框、工具栏的全部逻辑，共 300 行
// ✅ 正确：拆分为 ChatPage.tsx（50行）+ MessageList.tsx + ChatInput.tsx
```

```markdown
### GP-C02：组件 props 超过 5 个时，用 interface 定义并考虑拆分

**规则：** 组件 props 超过 5 个时，必须用 interface 定义，并评估是否需要拆分组件。

**为什么：** 过多 props 是组件职责不清晰的信号，通常意味着组件在做太多事情。

**反例：**
// ❌ 错误
<ChatMessage id={} content={} role={} timestamp={} isStreaming={} onRetry={} onCopy={} />

// ✅ 正确
interface ChatMessageProps { message: Message; onAction: (action: MessageAction) => void }
<ChatMessage message={msg} onAction={handleAction} />
```

```markdown
### GP-C03：组件内不写业务逻辑，业务逻辑放在 Hooks 层

**规则：** 组件的事件处理函数只做两件事：调用 Hook 方法，或更新局部 UI 状态。

**为什么：** 业务逻辑在 Hook 里才能被复用和单独测试。

**反例：**
// ❌ 错误：在 onClick 里直接写 API 调用
const handleSend = async () => {
  const response = await fetch('/api/chat', { method: 'POST', body: content })
  setMessages([...messages, await response.json()])
}

// ✅ 正确：调用 Hook 方法
const { sendMessage } = useAIChat()
const handleSend = () => sendMessage(content)
```

### Hook 类（GP-H）

```markdown
### GP-H01：Hook 只返回数据和方法，不返回 JSX

**规则：** 自定义 Hook 的返回值只能是数据（状态）和方法（函数），不能包含 JSX 元素。

**为什么：** Hook 是逻辑层，JSX 是视图层，混在一起破坏分层架构，
导致逻辑无法在不同 UI 场景下复用。

**反例：**
// ❌ 错误
function useAIChat() {
  const MessageList = () => <ul>{messages.map(...)}</ul>  // 返回了 JSX
  return { messages, MessageList }
}

// ✅ 正确
function useAIChat() {
  return { messages, sendMessage, isStreaming }  // 只返回数据和方法
}
```

```markdown
### GP-H02：Hook 的返回值用对象解构，不用数组解构

**规则：** 除了 useState 风格的配对（[value, setValue]），
自定义 Hook 的返回值应使用对象而非数组。

**为什么：** 对象解构允许调用方按需取值，数组解构强制按顺序，
扩展性差，调用方需要记住顺序。

**反例：**
// ❌ 错误
return [messages, sendMessage, isLoading, error]  // 调用方必须记住顺序

// ✅ 正确
return { messages, sendMessage, isLoading, error }  // 按需取值
```

```markdown
### GP-H03：一个 Hook 只做一件事

**规则：** 每个 Hook 的职责应该可以用一句话描述，名称能清晰反映它做什么。

**为什么：** 职责单一的 Hook 更容易测试、复用和维护。

**反例：**
// ❌ 错误：useChat 既管消息，又管用户信息，又控制 UI 状态
function useChat() { /* 200行，做了很多事 */ }

// ✅ 正确：职责分离
function useAIChat() { /* 只管 AI 对话逻辑 */ }
function useUserProfile() { /* 只管用户信息 */ }
function useChatUI() { /* 只管 UI 状态（滚动、展开等）*/ }
```

### API 层类（GP-A）

```markdown
### GP-A01：API 函数只做请求/响应转换，不包含业务判断

**规则：** src/api/ 下的函数只负责发请求和转换响应格式，
业务判断（权限、校验、流程控制）放在 Hooks 层。

**为什么：** API 层保持纯粹，便于 mock 测试和替换实现。

**反例：**
// ❌ 错误：API 层包含权限判断
export async function sendMessage(messages: Message[]) {
  if (!isVipUser()) throw new Error('需要 VIP')  // 业务判断！
  return await aiClient.chat(messages)
}

// ✅ 正确：API 层只做请求
export async function sendMessage(messages: Message[]) {
  return await aiClient.chat(messages)
}
```

```markdown
### GP-A02：所有 API 函数必须有明确的 TypeScript 返回类型

**规则：** API 函数的返回类型必须明确声明，禁止使用 any 或省略返回类型。

**为什么：** 类型安全是前端工程质量的基础，any 是 Slop 的温床。

**反例：**
// ❌ 错误
async function getMessages() { ... }  // 省略返回类型
async function getMessages(): Promise<any> { ... }  // 使用 any

// ✅ 正确
async function getMessages(): Promise<Message[]> { ... }
```

### Store 类（GP-S）

```markdown
### GP-S01：Store 只存储需要跨组件共享的状态

**规则：** 只有需要在多个组件间共享的状态才放进 Store，
局部状态（只在一个组件内使用）用 useState。

**为什么：** 过度使用全局 Store 会让状态流向难以追踪，增加调试难度。

**反例：**
// ❌ 错误：把只在 ChatInput 里用的 inputValue 放进 Store
const useChatStore = create(() => ({
  messages: [],
  inputValue: '',  // 只有 ChatInput 用，不需要全局
}))

// ✅ 正确：局部状态用 useState
function ChatInput() {
  const [inputValue, setInputValue] = useState('')  // 局部状态
  const { sendMessage } = useAIChat()  // 全局逻辑通过 Hook
}
```

---

## 四、黄金原则的维护规则

```
触发新增的时机：
  发现 AI Slop 的新模式 → 写一条原则预防
  Code Review 反复出现同类问题 → 写一条原则
  重构时发现某种模式特别难维护 → 写一条原则

触发更新的时机：
  技术栈迁移（换了状态管理库等）→ 更新相关原则
  某条原则已被 ESLint 规则覆盖 → 可以从黄金原则中移除

判断是否需要写成 ESLint 规则：
  能被机械检测（检查 import 路径、文件名等）→ 写成 ESLint 规则
  需要语义理解（判断逻辑是否合理）→ 保留为黄金原则
```

---

*附录 D · 完*
