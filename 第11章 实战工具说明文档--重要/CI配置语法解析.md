# CI 配置文件语法解析

---

## 一、GitHub Actions（`.github/workflows/harness-ci.yml`）

### 整体结构

```yaml
name: Harness CI          # 这条流水线在 GitHub 界面显示的名称

on:                       # 触发条件（什么时候跑）
  pull_request: ...
  push: ...

jobs:                     # 要执行的任务列表
  quick-check: ...
  arch-check: ...
```

---

### 触发条件（on）

```yaml
on:
  pull_request:           # 提交 PR 时触发
    branches:
      - main              # 目标分支是 main 的 PR
      - master
      - 'release-**'      # 目标分支以 release- 开头的 PR（** 是通配符）

  push:                   # 直接 push 代码时触发
    branches:
      - main              # push 到 main 分支
      - 'dev-**'          # push 到 dev-xxx 分支
      - 'fix-**'
      - 'release-**'
```

---

### 单个 job 的结构

```yaml
quick-check:              # job 的 ID（唯一标识）
  name: 快速检查           # 在 GitHub 界面显示的名称
  runs-on: ubuntu-latest  # 在什么系统上运行（GitHub 提供的虚拟机）

  steps:                  # 这个 job 的执行步骤（按顺序执行）

    - uses: actions/checkout@v4
    # uses 表示使用一个现成的 Action（别人写好的步骤）
    # actions/checkout@v4 = 把代码仓库 checkout 到虚拟机上
    # 没有这步，后面的命令找不到你的代码

    - uses: pnpm/action-setup@v4
      with:
        version: 10
    # 安装 pnpm 包管理器
    # with 是传给这个 Action 的参数

    - uses: actions/setup-node@v4
      with:
        node-version: '24'
        cache: 'pnpm'
    # 安装 Node.js v24
    # cache: 'pnpm' 表示缓存 pnpm 的依赖，下次跑更快

    - name: 安装依赖        # name 是这个步骤的显示名称
      run: pnpm install --frozen-lockfile
    # run 表示直接执行 shell 命令
    # --frozen-lockfile 表示严格按照 pnpm-lock.yaml 安装，不允许更新锁文件

    - name: 格式检查
      run: pnpm format:check
    # 执行 package.json 里定义的 format:check 脚本
```

---

### job 之间的依赖（needs）

```yaml
arch-check:
  needs: quick-check      # 必须等 quick-check 通过后才开始
                          # 如果 quick-check 失败，arch-check 不会运行
```

```
执行顺序：
  quick-check
      ↓（通过后）
  arch-check
      ↓（通过后）
  test
      ↓（通过后）
  build
```

---

## 二、GitLab CI（`.gitlab-ci.yml`）

### 整体结构

```yaml
stages:           # 定义阶段顺序
  - quick-check
  - arch-check
  - test
  - build

.base:            # 公共配置模板（以 . 开头表示不是真正的 job，只是模板）
  ...

workflow:         # 触发条件
  ...

quick-check:      # 真正的 job
  extends: .base  # 继承 .base 的配置
  stage: quick-check
  script: ...
```

---

### 公共配置模板（.base）

```yaml
.base:
  image: node:24
  # 使用 Docker 镜像 node:24 作为运行环境
  # 相当于 GitHub Actions 的 runs-on + setup-node 的组合

  before_script:
    # before_script 在每个 job 的 script 之前自动执行
    - corepack enable
    # 启用 Node.js 内置的 corepack 工具（用于管理 pnpm/yarn 版本）

    - corepack prepare pnpm@10 --activate
    # 安装并激活 pnpm v10
    # 相当于 GitHub Actions 里的 pnpm/action-setup@v4

    - pnpm install --frozen-lockfile
    # 安装依赖

  cache:
    key:
      files:
        - pnpm-lock.yaml
    # 缓存的 key 基于 pnpm-lock.yaml 文件的内容
    # pnpm-lock.yaml 没变 → 用缓存；变了 → 重新安装
    paths:
      - node_modules/
    # 缓存 node_modules 目录
```

---

### workflow（触发条件）—— 重点

`workflow` 是 GitLab CI 特有的全局触发条件配置，决定整条流水线是否运行。

```yaml
workflow:
  rules:
    # MR 触发
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    # push 触发（指定分支）
    - if: $CI_PIPELINE_SOURCE == "push" && $CI_COMMIT_BRANCH == "main"
    - if: $CI_PIPELINE_SOURCE == "push" && $CI_COMMIT_BRANCH == "master"
    - if: $CI_PIPELINE_SOURCE == "push" && $CI_COMMIT_BRANCH =~ /^dev-/
    - if: $CI_PIPELINE_SOURCE == "push" && $CI_COMMIT_BRANCH =~ /^fix-/
    - if: $CI_PIPELINE_SOURCE == "push" && $CI_COMMIT_BRANCH =~ /^release-/
```

`rules` 是一个列表，**从上到下依次判断，第一个匹配的规则生效**。

默认行为：匹配到规则 → 运行流水线；没有匹配到任何规则 → 不运行。

**注意：** 必须同时判断 `$CI_PIPELINE_SOURCE` 和 `$CI_COMMIT_BRANCH`，原因：
- MR 触发时，`$CI_COMMIT_BRANCH` 是空的，只能用 `$CI_PIPELINE_SOURCE` 判断
- push 触发时，两个变量都有值，加上 `$CI_PIPELINE_SOURCE == "push"` 让意图更清晰，避免误触发

---

#### `==` 和 `=~` 的区别

| 运算符 | 含义 | 示例 |
|--------|------|------|
| `==` | 精确相等 | `$VAR == "main"` → 变量值必须完全等于 "main" |
| `=~` | 正则匹配 | `$VAR =~ /^dev-/` → 变量值必须匹配正则表达式 |

**`==` 精确匹配示例：**
```yaml
- if: $CI_COMMIT_BRANCH == "main"
# 只有分支名完全等于 "main" 才匹配
# "main-v2" 不匹配
# "Main" 不匹配（大小写敏感）
```

**`=~` 正则匹配示例：**
```yaml
- if: $CI_COMMIT_BRANCH =~ /^dev-/
# ^ 表示字符串开头
# 匹配：dev-feature、dev-login、dev-anything
# 不匹配：my-dev-branch（不是以 dev- 开头）
```

**正则语法说明：**
```
/^dev-/    → 以 dev- 开头
/^fix-/    → 以 fix- 开头
/^release-/→ 以 release- 开头

^ = 字符串开头
- = 普通字符（连字符）
/ / = 正则表达式的定界符（GitLab 用斜杠包裹正则）
```

---

#### GitLab 内置变量

```yaml
$CI_PIPELINE_SOURCE   # 流水线的触发来源
  "merge_request_event" = 由 MR 触发
  "push"               = 由 push 触发
  "schedule"           = 由定时任务触发
  "web"                = 在 GitLab 界面手动触发

$CI_COMMIT_BRANCH     # 当前提交所在的分支名
  例："main"、"dev-login"、"fix-bug-123"
```

---

### job 的结构

```yaml
quick-check:
  extends: .base      # 继承 .base 的所有配置（image、before_script、cache）
  stage: quick-check  # 属于哪个阶段（对应 stages 列表里的名称）
  script:             # 要执行的命令（在 before_script 之后执行）
    - pnpm format:check
    - pnpm type-check
```

---

### GitLab 的阶段依赖

GitLab CI 不需要像 GitHub Actions 那样写 `needs`，**阶段顺序由 `stages` 列表决定**：

```yaml
stages:
  - quick-check   # 第一个跑
  - arch-check    # quick-check 全部通过后跑
  - test          # arch-check 全部通过后跑
  - build         # test 全部通过后跑
```

同一个 stage 里的 job 并行执行，不同 stage 按顺序执行。

---

### 构建产物（artifacts）

```yaml
build:
  ...
  artifacts:
    paths:
      - dist/         # 保存 dist/ 目录
    expire_in: 3 days # 3 天后自动删除
    when: on_success  # 只在 job 成功时保存
```

相当于 GitHub Actions 的 `upload-artifact`，让你可以在 GitLab 界面下载构建产物。

---

## 三、两者对比总结

| 概念 | GitHub Actions | GitLab CI |
|------|---------------|-----------|
| 配置文件位置 | `.github/workflows/*.yml` | `.gitlab-ci.yml`（根目录）|
| 触发条件 | `on:` | `workflow: rules:` |
| 运行环境 | `runs-on` + `setup-node` | `image: node:24` |
| 安装 pnpm | `pnpm/action-setup@v4` | `corepack enable` |
| 阶段依赖 | `needs: job-name` | `stages:` 列表顺序 |
| 公共配置复用 | 无内置（用 composite action）| `.base` 模板 + `extends` |
| 构建产物 | `upload-artifact` | `artifacts:` |
| 内置变量 | `github.ref`、`github.event_name` | `$CI_COMMIT_BRANCH`、`$CI_PIPELINE_SOURCE` |
