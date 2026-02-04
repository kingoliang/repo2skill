---
name: repo2skill
description: 将本地代码仓库或远程 GitHub/GitLab/Gitee 仓库转换为 Claude Code Skill，支持基于 git diff 的增量更新。分析项目结构、技术栈、核心模块、API、数据库，自动生成符合 skill-creator 规范的 SKILL.md 和 references 文档。适用于 Java/Spring Boot/Node.js/Python/Go/Rust 等项目。使用方式：提供本地项目路径或远程仓库 URL。更新 skill 时自动检测 git 变更，仅重写受影响的文档。
---

# repo2skill - 代码仓库转 Skill 生成器

## 功能

分析本地代码仓库或远程仓库，生成完整的 Claude Code Skill，包括：
- `SKILL.md` - 主技能文件（<200行，符合 skill-creator 规范）
- `references/` - 详细参考文档（按需加载）
- `scripts/` - 可复用脚本（如有必要）

## 使用方式

```
# 首次生成
帮我把这个项目转成 skill：/path/to/local/project
帮我把这个仓库转成 skill：https://github.com/owner/repo

# 增量更新（默认，基于 git diff）
更新一下这个 skill：{skill-name}

# 全量重新生成
重新生成这个 skill：{skill-name}
```

**参数**:
- 本地路径或远程 URL（GitHub/GitLab/Gitee）
- 可选指定 skill 名称和保存位置

---

## 执行流程

### 第一步：判断来源类型

- 本地路径 → 直接分析文件系统
- 远程 URL → 参考 [remote-fetch.md](references/remote-fetch.md) 获取数据后分析

### 第二步：项目分析

参考 [project-analysis.md](references/project-analysis.md) 执行：

1. 识别技术栈（语言、框架、构建工具）
2. 分析目录结构和模块划分
3. 识别核心入口文件
4. 分析 API 层、业务层、数据层
5. 提取配置、枚举、常量

### 第三步：生成 Skill 文档

参考 [skill-template.md](references/skill-template.md) 生成：

#### 必须生成的文件

```
{skill-name}/
├── SKILL.md                          # 主文件（<200行）
└── references/
    ├── PROJECT_INDEX.md              # 项目概览、目录结构、依赖
    ├── ARCHITECTURE.md               # 架构设计、分层、调用链
    ├── API_REFERENCE.md              # API 端点/函数索引及详情
    ├── DATA_MODEL.md                 # 数据模型/数据库表结构
    ├── DOMAIN_GUIDE.md               # 业务领域、核心概念
    └── DEVELOPMENT.md                # 开发指南、构建、测试、配置
```

#### 按需生成（项目存在相关内容时）

- `ENUMS_CONSTANTS.md` - 枚举和常量
- `BUSINESS_RULES.md` - 业务规则和约束
- `TROUBLESHOOTING.md` - 常见问题和排查
- `CODE_CONVENTIONS.md` - 编码规范

### 第四步：选择保存位置

询问用户保存位置：

1. **项目本地**: `.claude/skills/{skill-name}/`
2. **全局用户**: `~/.claude/skills/{skill-name}/`

### 第五步：写入文件并验证

1. 创建目录结构
2. 写入所有文件
3. 生成 `.metadata` 文件记录本次生成快照（参考 [incremental-update.md](references/incremental-update.md)）
4. 验证完整性（SKILL.md <200行、所有引用文件存在）
5. 输出生成摘要

---

## 更新已有 Skill

默认使用 **git diff 增量更新**，用户明确要求时执行全量重新生成。

详细流程参考 [incremental-update.md](references/incremental-update.md)。

### 增量更新流程

1. 读取 skill 目录下的 `.metadata` 获取上次生成的 commit hash 和项目路径
2. 在项目目录执行 `git diff --name-only {last_commit}..HEAD` 获取变更文件列表
3. 根据变更文件类型映射到需要更新的 reference 文档
4. 仅重新分析变更相关的代码，重写受影响的文档
5. 更新 `.metadata` 中的 commit hash

### 全量重新生成

用户明确要求时（"重新生成"、"全量更新"），跳过 diff 检测，按首次生成流程完整重新执行。

---

## 生成原则

1. **SKILL.md 必须 <200行** - 详细内容放 references/
2. **渐进式披露** - metadata → SKILL.md → references 三层结构
3. **实用导向** - 包含真实的搜索模式、Glob/Grep 命令、代码入口
4. **标注数据来源** - 每个文档标注信息提取自哪些文件
5. **不假设不猜测** - 仅记录代码中实际存在的内容
6. **description 要丰富** - 包含足够的触发关键词和使用场景

---

## 批量转换

支持同时转换多个项目：

```
帮我转换这几个项目:
- /path/to/project-a
- /path/to/project-b
- https://github.com/owner/repo-c
```

逐个执行分析和生成，最后统一报告结果。

---

## 输出摘要格式

```
Skill 生成完成

生成位置: ~/.claude/skills/{skill-name}/

生成的文件:
  SKILL.md (主文件, XX行)
  references/PROJECT_INDEX.md
  references/ARCHITECTURE.md
  references/API_REFERENCE.md
  references/DATA_MODEL.md
  references/DOMAIN_GUIDE.md
  references/DEVELOPMENT.md

项目分析:
  技术栈: {tech-stack}
  模块数: X 个
  API 数: X 个
  数据表: X 个
```

---

## 错误处理

- 路径不存在 → 提示并要求重新提供
- 远程仓库不可访问 → 参考 [remote-fetch.md](references/remote-fetch.md) 的镜像和重试策略
- 项目为空 → 告知用户并停止
- 分析过程中出错 → 记录错误、跳过该部分、继续生成其余文档
