# repo2skill 深度分析 — Queen (Launchpad)

## 1. 本质定位

**repo2skill 是一个 Claude Code Agent Skill，更准确地说是一个"元 Skill"（Meta-Skill）— Skill 的工厂。**

它不是独立工具，是一套 prompt 指令集，安装到 `~/.claude/skills/` 后被 Claude Code 自动加载执行。

**核心问题**：Claude Code 每次新会话都没有项目上下文，需要反复解释项目结构。repo2skill 通过预生成项目知识库，让后续对话自动"认识"项目。

---

## 2. 设计思路

### 零代码，纯 Prompt Engineering

整个项目是 1 个 SKILL.md + 4 个 reference docs，用自然语言描述工作流程和分析规则。没有解析器，没有 AST，完全依赖 LLM 的理解能力。

### 三层渐进式结构

```
.metadata      → 快照信息（commit hash、tech stack）
SKILL.md       → 主入口（<200 行），项目概览 + 搜索命令表
references/    → 详细文档，按需加载
```

这是 Claude Code skill-creator 规范的核心设计：**主文件精简，详情按需加载**。

### 增量更新机制

`.metadata` 记录上次 commit hash，`git diff` 检测变更，按文件类型映射到对应文档：

| 变更文件 | 更新文档 |
|----------|----------|
| Controller | API_REFERENCE.md |
| Entity/Model | DATA_MODEL.md |
| Service | ARCHITECTURE.md + DOMAIN_GUIDE.md |
| Config/YAML | DEVELOPMENT.md |

只重写受影响的部分，节省 token。

---

## 3. 创新点

1. **用 prompt 编程 agent 行为** — 不是写解析器，是写"指令手册"教 LLM 怎么分析代码
2. **变更文件 → 文档映射** — 精确定位需要更新的文档，避免全量重跑
3. **GitHub API 多镜像** — 国内友好，4 个 API 镜像 + 2 个 Raw 镜像，自动切换 + 指数退避重试
4. **Agent 自举（bootstrapping）** — 一个 Skill 生成其他 Skill

---

## 4. 适用场景

- 需要用 Claude Code 做开发的团队
- 有多个需要长期维护的项目
- 项目代码量较大，需要快速建立上下文
- 需要增量更新（而不是每次全量重跑）

---

## 5. 局限性

| 问题 | 影响 |
|------|------|
| 强依赖 Claude Code | 离开就没法用 |
| 无验证机制 | 生成质量完全依赖 LLM 自觉，没有 schema 校验 |
| GitHub API 限额 | 未认证 60/h，大 repo 可能不够 |
| 30 文件阈值偏低 | 活跃项目日常 PR 可能就超了 |
| 增量更新边界条件 | rebase/force push 会回退全量 |

---

## 6. 对 AKQ 团队的价值

1. **对 akq-payment-rca**：可以生成 skill，以后维护更容易
2. **对新项目**：快速建立知识库，降低上手成本
3. **对 Queen (我)**：我负责文档，这个工具和我的职责高度相关

---

## 7. 改进建议

1. **增加 JSON schema 验证** — 校验生成的文档结构
2. **提高 30 文件阈值** — 或改为"变更文件占比"
3. **支持更多语言** — 目前主 Java/Node/Python/Go/Rust
4. **增加中文模板选项** — 当前模板是英文

---

## 8. 与其他 Skill 的关系

repo2skill 生成的 SKILL.md 会被其他 Skill 引用：
- `coding-agent` skill 可以调用生成的 skill 来理解项目
- `skill-creator` skill 的规范被 repo2skill 严格遵循

这是一个 **Skill 生态系统** 的关键基础设施。
