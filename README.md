# repo2skill

[English](README_EN.md) | 中文

将本地代码仓库或远程 GitHub/GitLab/Gitee 仓库转换为 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) Skill。

## 解决什么问题

Claude Code 每次新对话都不了解你的项目。repo2skill 把项目的关键信息预先提取成 skill 文档，让 Claude Code 在后续编程中自动具备项目上下文——知道代码在哪、架构怎么分层、API 怎么调、数据模型是什么。

## 功能特性

- **本地 + 远程**：支持本地项目路径和 GitHub/GitLab/Gitee 远程仓库 URL
- **多语言支持**：Java/Spring Boot、Node.js/TypeScript、Python/Django/FastAPI、Go、Rust 等
- **增量更新**：基于 `git diff` 检测变更，仅重写受影响的文档，节省 token
- **符合规范**：生成的 SKILL.md 严格遵循 skill-creator 规范（<200 行，渐进式披露）
- **批量转换**：一次处理多个项目
- **远程镜像**：GitHub API 多镜像自动切换，指数退避重试

## 安装

将 `repo2skill/` 目录复制到 Claude Code skills 目录：

```bash
# 全局安装（所有项目可用）
cp -r repo2skill/ ~/.claude/skills/repo2skill/

# 或项目本地安装
cp -r repo2skill/ your-project/.claude/skills/repo2skill/
```

无需安装任何依赖。

## 使用方式

在 Claude Code 中直接用自然语言：

```
# 本地项目转 skill
帮我把这个项目转成 skill：/path/to/my-project

# 远程仓库转 skill
帮我把这个仓库转成 skill：https://github.com/owner/repo

# 增量更新（默认）
更新一下这个 skill：my-project

# 全量重新生成
重新生成这个 skill：my-project

# 批量转换
帮我转换这几个项目:
- /path/to/project-a
- https://github.com/owner/repo-b
```

## 生成内容

### 必须生成

| 文件 | 内容 |
|------|------|
| `SKILL.md` | 主文件，项目概览、模块结构、搜索命令、入口文件（<200 行） |
| `references/PROJECT_INDEX.md` | 项目结构、依赖列表 |
| `references/ARCHITECTURE.md` | 架构设计、分层、调用链 |
| `references/API_REFERENCE.md` | API 端点、请求/响应参数、调用链 |
| `references/DATA_MODEL.md` | 数据模型、表结构、字段、关联关系 |
| `references/DOMAIN_GUIDE.md` | 业务领域概念、流程、术语 |
| `references/DEVELOPMENT.md` | 开发环境、构建、测试、配置、部署 |

### 按需生成

- `ENUMS_CONSTANTS.md` — 枚举和常量
- `BUSINESS_RULES.md` — 业务规则和约束
- `TROUBLESHOOTING.md` — 常见问题排查
- `CODE_CONVENTIONS.md` — 编码规范

## 增量更新机制

首次生成时自动创建 `.metadata` 文件，记录 commit hash。更新时：

1. 读取 `.metadata` 获取上次 commit hash
2. 执行 `git diff --name-only {old}..HEAD` 获取变更文件
3. 按文件类型映射到对应文档，仅重写受影响的部分
4. 更新 `.metadata`

**变更映射规则：**

| 变更文件 | 更新文档 |
|----------|----------|
| Controller / Router / Provider | API_REFERENCE.md |
| Entity / Model / Mapper | DATA_MODEL.md |
| Service / Domain | ARCHITECTURE.md + DOMAIN_GUIDE.md |
| pom.xml / package.json / go.mod | PROJECT_INDEX.md + DEVELOPMENT.md |
| Config / YAML / .env | DEVELOPMENT.md |
| Enum / Constants | ENUMS_CONSTANTS.md |

变更文件超过 30 个时自动回退为全量重新生成。

## 项目结构

```
repo2skill/
├── SKILL.md                              # 主技能文件 (170 行)
└── references/
    ├── project-analysis.md               # 项目分析指南 (183 行)
    ├── remote-fetch.md                   # 远程仓库获取 (122 行)
    ├── skill-template.md                 # 生成模板 (105 行)
    └── incremental-update.md             # 增量更新指南 (161 行)
```

## 对 Claude Code 编程的帮助

生成 skill 后，Claude Code 在后续对话中会自动加载项目上下文：

- **快速定位代码**：skill 中的搜索命令表让 Claude 直接知道去哪找代码
- **理解架构**：分层关系和调用链让 Claude 把代码写到正确的层
- **精确修改 API**：请求/响应参数文档让 Claude 按已有风格新增或修改接口
- **正确操作数据**：字段类型和约束让 Claude 写出正确的数据库操作
- **理解业务术语**：领域指南让 Claude 不会误解需求中的业务概念
- **执行正确命令**：开发指南让 Claude 知道该用什么构建和测试命令

## 远程仓库支持

| 平台 | API 镜像 | 认证 |
|------|----------|------|
| GitHub | 4 个 API 镜像 + 2 个 Raw 镜像，自动切换 | 可选 token（提升限额到 5000/h） |
| GitLab | 官方 API | 可选 PRIVATE-TOKEN |
| Gitee | 官方 API v5 | 可选 token |

## 参考

- [Claude Code Skills 文档](https://docs.anthropic.com/en/docs/claude-code/skills)
- 灵感来源：[zhangyanxs/repo2skill](https://github.com/zhangyanxs/repo2skill)

## License

MIT
