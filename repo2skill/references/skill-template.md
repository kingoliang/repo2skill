# Skill 文档生成模板

## SKILL.md 模板

SKILL.md 必须 <200 行。使用祈使句/动词优先风格。

```markdown
---
name: {skill-name}
description: {丰富的描述，包含使用场景和触发关键词，80-150字}
---

# {项目名} 开发指南

## 项目概览

| 属性 | 值 |
|------|-----|
| **项目类型** | {类型} |
| **技术栈** | {语言} + {框架} + {关键库} |
| **基础包/入口** | `{base-package-or-entry}` |

## 模块结构

| 模块 | 路径 | 说明 |
|------|------|------|
| {模块1} | `{path-pattern}` | {说明} |
| {模块2} | `{path-pattern}` | {说明} |

## 按功能搜索

| 功能 | 搜索命令 | 说明 |
|------|----------|------|
| {功能1} | `Glob "{pattern}"` | {说明} |
| {功能2} | `Grep "{keyword}" --glob "{pattern}"` | {说明} |

## 核心入口文件

| 文件 | 路径 | 说明 |
|------|------|------|
| {文件1} | `{path}` | {说明} |

## 详细文档

| 需要了解 | 参考文档 |
|----------|----------|
| 项目结构和依赖 | [PROJECT_INDEX.md](references/PROJECT_INDEX.md) |
| 架构设计和分层 | [ARCHITECTURE.md](references/ARCHITECTURE.md) |
| API 接口详情 | [API_REFERENCE.md](references/API_REFERENCE.md) |
| 数据模型/表结构 | [DATA_MODEL.md](references/DATA_MODEL.md) |
| 业务领域概念 | [DOMAIN_GUIDE.md](references/DOMAIN_GUIDE.md) |
| 开发、构建、测试 | [DEVELOPMENT.md](references/DEVELOPMENT.md) |

## 常用命令

```bash
# 安装依赖
{install-command}

# 启动开发
{dev-command}

# 运行测试
{test-command}

# 构建
{build-command}
```
```

---

## References 文档模板

### PROJECT_INDEX.md

包含: 基本信息表(项目名/版本/语言/框架) → 目录结构(深度3-4层) → 核心依赖表(依赖/版本/用途) → 开发依赖表。每节标注 `> 数据来源: {file}`。

### ARCHITECTURE.md

包含: 分层架构图(API→Service→Repository→DB) → 模块依赖关系 → 核心组件表(组件/位置/职责)。

### API_REFERENCE.md

包含: API 索引表(方法/路径/说明/认证) → 每个 API 详情块:
- 基本信息(文件路径)
- 请求参数表(字段/类型/必填/说明)
- 响应参数表(字段/类型/说明)
- 调用链(Controller→Service→Repository)

Java 项目: 从 `@NotBlank`/`@NotNull` 注解判断必填。

### DATA_MODEL.md

包含: 模型列表表(模型/文件/对应表) → 每个模型详情:
- 字段表(字段/类型/约束/说明)
- 关联关系

### DOMAIN_GUIDE.md

包含: 核心概念表 → 业务流程(步骤图) → 术语表(术语/含义)。

### DEVELOPMENT.md

包含: 环境要求表(工具/版本) → 快速开始命令 → 构建命令表 → 测试命令 → 配置说明表(配置项/说明/默认值) → 部署信息。
