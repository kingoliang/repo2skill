# repo2skill 深度分析 — Ace 🂡

## 本质定位

repo2skill **不是工具，不是程序，是一个"元 Skill"（Meta-Skill）**。

它是一个 Claude Code Agent Skill，功能是**生成其他 Agent Skill**。用 prompt 编程 agent 行为，让 Claude Code 按照预设的分析流程和模板，把任意代码仓库转化为结构化的 Skill 文档。

换句话说：**它是 Skill 的工厂。**

## 核心问题和解法

**问题**：Claude Code 每次新对话都是白纸，不了解你的项目。手动写 Skill 文档费时费力，而且容易遗漏。

**解法**：把"分析代码、生成文档"这个过程本身变成一个 Skill，让 Claude Code 自己来做。这是一种**自举（bootstrapping）**思路 — 用 agent 来为 agent 创建知识。

## 设计思路深析

### 1. 三层渐进式架构
- **metadata**（.metadata JSON）→ 机器读的元数据
- **SKILL.md**（<200行）→ 快速概览，每次对话自动加载
- **references/**（按需加载）→ 深度信息，用到时才读

这个设计解决了一个关键矛盾：**上下文窗口有限 vs 项目信息量大**。200 行的 SKILL.md 消耗极少 token，但包含了最关键的入口信息和搜索命令。

### 2. Prompt as Code
整个项目 0 行可执行代码，1017 行 Markdown。这些 Markdown 不是文档，是**指令**。project-analysis.md 里的每个 Glob/Grep 命令都是给 Claude Code 执行的。skill-template.md 里的模板是输出规范。

这是 prompt engineering 的高级形态：**用自然语言写"程序"，agent 是"运行时"。**

### 3. 增量更新 = 状态管理
.metadata 文件记录 commit hash，git diff 检测变更，变更文件按类型映射到对应文档。这其实是一套**简易的状态管理和增量编译系统**，只不过"编译"的产物是文档而不是二进制。

## 适用场景

1. **开发者个人**：快速为自己的项目建立 Claude Code 知识库
2. **团队协作**：新成员加入时，Skill 就是项目的"活文档"
3. **多项目管理**：同时维护多个项目的 Claude Code 上下文
4. **开源项目贡献者**：快速理解陌生项目的结构

**不适用**：
- 非 Claude Code 用户（强绑定 Claude Code 生态）
- 极小项目（几个文件的项目不需要这么重的文档）
- 非代码仓库（纯文档/数据仓库）

## 局限性

1. **生态锁定**：只能在 Claude Code 里用。OpenClaw 的 skill 体系虽然类似但格式不同
2. **质量不可控**：生成的文档质量完全取决于 LLM 当次的表现，没有任何校验机制
3. **语言覆盖有限**：project-analysis.md 里只写了 Java/Node/Python/Go/Rust 的 patterns，其他语言需要自己扩展
4. **远程获取脆弱**：GitHub API 60次/小时限制，分析大 repo 可能中途断掉
5. **增量更新的 30 文件阈值偏保守**：活跃项目一次 merge 就可能超过

## 对我们团队的价值

### 直接价值
- 把 `akq-payment-rca` 转成 Skill，以后用 Claude Code 维护时自动具备项目上下文

### 间接价值
- **设计模式可借鉴**：三层渐进式架构、增量更新机制、变更文件映射规则，这些思路可以移植到 OpenClaw 的 skill 体系
- **prompt engineering 范例**：展示了如何用纯自然语言构建一个有完整工作流的 agent 能力

### 需要注意
- 这是 Claude Code 生态的 Skill，不是 OpenClaw 的 Skill。如果要在 OpenClaw 体系下用类似功能，需要改造适配
