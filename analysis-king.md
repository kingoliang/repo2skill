# repo2skill 深度分析 — King (study)

## 本质定位

repo2skill 是一个 **Claude Code Agent Skill**——它不是程序、不是工具、不是库。它是一套精心设计的"元指令"，教会 Claude Code 如何把任意代码仓库转化为结构化的项目知识文档。

更本质地说，repo2skill 解决的核心问题是 **LLM 的会话健忘症**：Claude Code 每次新对话都不了解你的项目。传统做法是开发者手写文档或每次对话手动复制上下文，repo2skill 把这个过程自动化了——让 AI 自己为 AI 生成上下文。

这是一个 **"AI 为 AI 写文档"** 的范式。

## 设计思路与创新点

### 1. 零代码架构

整个项目 741 行，全是 Markdown。没有 `main.py`，没有 `index.js`，没有可执行文件。这是一个大胆的设计决策：**用自然语言而不是代码来编程**。

创新在于：它证明了在 Agent Skill 的语境下，结构化的 Markdown 指令和代码一样有"执行力"——只要运行时是 LLM agent 而不是 CPU。

### 2. 三层渐进式信息架构

```
.metadata (机器读) → SKILL.md (入口, <200行) → references/ (按需加载)
```

这不是随意的文件组织。它精确匹配了 Claude Code 的 Skill 加载机制：
- **SKILL.md 作为"索引"被自动加载**——所以必须短小（<200行），只放导航信息
- **references/ 按需读取**——只在需要时才消耗 token，避免上下文窗口浪费
- **.metadata 记录状态**——让增量更新成为可能

这个设计的本质是 **token 经济学**：在 LLM 有限的上下文窗口中，最大化信息密度。

### 3. 增量更新 = 差分编程

`git diff → 文件分类 → 映射到文档 → 局部重写` 这个流程，本质上是把软件工程的增量编译思想应用到了文档生成上。

`incremental-update.md` 里的文件-文档映射表是整个项目最精妙的部分——它建立了"代码变更类型"和"知识文档区域"之间的因果关系，让 LLM 知道改了 Entity 就该更新 DATA_MODEL，改了 Controller 就该更新 API_REFERENCE。

### 4. 分析模式的工程化

`project-analysis.md` 把"人类开发者理解一个新项目"的隐性知识显性化了：先看 build 文件判断技术栈 → 看目录结构理解分层 → 找入口文件 → 追 API → 追数据模型 → 找配置。这本质上是把 **代码考古学（Code Archaeology）** 编码成了可执行指令。

## 适用场景

**谁会用：** 使用 Claude Code 做日常开发的程序员/团队。

**什么时候用：**
1. **接手新项目时** — 快速生成项目知识库，不用花 2 天读代码
2. **团队统一上下文** — 新人加入后，Claude Code 自动具备项目知识
3. **长期维护** — 增量更新让文档和代码保持同步，解决"文档过时"的经典问题
4. **多项目切换** — 每个项目一个 skill，Claude Code 自动切换上下文

**不适用：**
- 不用 Claude Code 的团队（离开 Claude Code 这就是一堆 Markdown）
- 极大型 monorepo（GitHub API 限额 + 30 文件阈值）
- 非代码项目（数据分析、ML 模型等）

## 局限性与改进空间

### 根本局限
1. **平台锁定** — 100% 绑定 Claude Code。如果 Anthropic 改了 Skill 规范，整个项目需要重写
2. **质量不可控** — 生成的文档质量完全取决于 LLM 当次的表现，没有自动化验证（比如检查 SKILL.md 是否真的 <200 行、references 里的文件路径是否存在）
3. **单向信息流** — repo → skill 是单向的。skill 生成后如果人工修改了文档，下次增量更新会覆盖掉

### 可改进
1. **加 validation 步骤** — 生成后自动检查：行数限制、文件引用完整性、模板字段覆盖率
2. **提高增量阈值** — 30 文件太低，改到 50-80 更合理，或者按变更比例（>50% 文件变更才全量）
3. **支持 .gitignore 过滤** — 分析时应跳过 build artifacts、node_modules 等
4. **多 agent 支持** — 目前只考虑 Claude Code，但 Cursor、Copilot Workspace 等也有类似 skill 机制

## 对我们团队 (AKQ) 的价值

### 直接价值
- **akq-payment-rca** 可以立即跑一遍，生成 skill 后我们三个人的 Claude Code CLI 都能自动理解项目代码
- 以后新项目 `akq-*` 都先跑 repo2skill，省去互相讲解项目结构的时间

### 间接价值
- 这个项目本身就是 **skill 工程的教科书** — 它展示了怎么设计一个好的 Agent Skill：清晰的入口、按需加载的详细文档、状态管理
- 我们可以参考它的模式为自己的工具写 skill

### 战略价值
- 如果 Kingo 有更多项目要我们维护，repo2skill 是让我们快速建立项目认知的加速器
- "AI 为 AI 写文档"这个范式值得深入探索，可能是未来 AI 协作开发的基础设施

---

*分析人：King (study) · Claude Opus 4.6 · 2026-02-19*
