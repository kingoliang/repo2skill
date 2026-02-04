# 增量更新指南

## .metadata 文件格式

首次生成 skill 时，在 skill 目录下创建 `.metadata` 文件：

```json
{
  "source_path": "/absolute/path/to/project",
  "source_type": "local",
  "commit_hash": "abc1234def5678",
  "generated_at": "2025-01-15T10:30:00Z",
  "skill_name": "my-project",
  "tech_stack": "Java + Spring Boot + MyBatis",
  "generated_files": [
    "SKILL.md",
    "references/PROJECT_INDEX.md",
    "references/ARCHITECTURE.md",
    "references/API_REFERENCE.md",
    "references/DATA_MODEL.md",
    "references/DOMAIN_GUIDE.md",
    "references/DEVELOPMENT.md"
  ]
}
```

远程仓库额外字段：
```json
{
  "source_type": "remote",
  "source_url": "https://github.com/owner/repo",
  "platform": "github",
  "branch": "main"
}
```

---

## 增量更新流程

### 1. 读取 .metadata

```bash
Read {skill-dir}/.metadata
```

如果 `.metadata` 不存在 → 提示用户该 skill 不支持增量更新，建议全量重新生成。

### 2. 检查项目状态

```bash
cd {source_path}
git rev-parse HEAD              # 当前 commit
git rev-parse {last_commit}     # 验证旧 commit 是否还在历史中
```

如果旧 commit 不在历史中（例如 rebase/force push）→ 回退到全量重新生成。

### 3. 获取变更文件列表

```bash
git diff --name-only {last_commit}..HEAD
```

如果无变更 → 告知用户 skill 已是最新，无需更新。

### 4. 分类变更文件

对 diff 输出的每个文件路径，按以下规则分类：

| 变更文件匹配模式 | 需要更新的文档 |
|------------------|----------------|
| `**/controller/**`, `**/routes/**`, `**/views/**`, `**/*Provider.*`, `**/*Handler.*` | API_REFERENCE.md |
| `**/entity/**`, `**/model/**`, `**/models/**`, `**/schema/**`, `**/*DO.*`, `**/*Mapper.xml` | DATA_MODEL.md |
| `**/service/**`, `**/services/**`, `**/domain/**`, `**/usecases/**` | ARCHITECTURE.md, DOMAIN_GUIDE.md |
| `**/enum/**`, `**/enums/**`, `**/constants/**`, `**/types/**` | ENUMS_CONSTANTS.md |
| `**/config/**`, `**/configuration/**`, `**/*.yml`, `**/*.yaml`, `.env*` | DEVELOPMENT.md |
| `pom.xml`, `package.json`, `go.mod`, `Cargo.toml`, `requirements.txt` | PROJECT_INDEX.md, DEVELOPMENT.md |
| `Makefile`, `Dockerfile`, `docker-compose.*`, `.github/workflows/**` | DEVELOPMENT.md |
| `README.md`, `CONTRIBUTING.md` | DEVELOPMENT.md |

#### 特殊情况

- **新增/删除目录** → 额外更新 SKILL.md 的模块结构表
- **构建文件变更** (pom.xml/package.json) → 额外检查是否有新增依赖影响技术栈描述
- **大量文件变更** (>30个文件) → 建议全量重新生成，因为增量更新可能遗漏关联变化

### 5. 重新分析受影响的代码

仅读取变更文件和相关文件（同模块/同目录），重新提取信息。

对于删除的文件：从对应文档中移除相关条目。
对于新增的文件：在对应文档中添加新条目。
对于修改的文件：重新分析并替换对应文档中的条目。

### 6. 重写受影响的文档

仅重写步骤 4 中标记需要更新的文档，其余文档保持不变。

### 7. 更新 .metadata

```json
{
  "commit_hash": "{new_HEAD_commit}",
  "generated_at": "{current_timestamp}",
  "last_update_type": "incremental",
  "updated_files": ["references/API_REFERENCE.md", "references/DATA_MODEL.md"]
}
```

---

## 远程仓库增量更新

远程仓库无法直接 git diff，使用以下替代方案：

1. 通过 API 获取 compare 接口：
   ```bash
   # GitHub
   curl -s "https://api.github.com/repos/{owner}/{repo}/compare/{last_commit}...HEAD"
   ```
2. 从 response 的 `files` 数组提取变更文件列表
3. 后续流程与本地相同

如果 compare API 失败（commit 太旧、仓库 rebase 等）→ 回退到全量重新生成。

---

## 回退到全量重新生成的条件

满足以下任一条件时，自动切换为全量重新生成：

1. `.metadata` 文件不存在
2. 旧 commit hash 不在 git 历史中
3. 变更文件数 >30
4. 构建文件变更导致技术栈发生根本变化（如从 Maven 切到 Gradle）
5. 用户明确要求（"重新生成"、"全量更新"、"--force"）

回退时告知用户原因：
```
检测到 XX 个文件变更（超过阈值 30），将执行全量重新生成以确保完整性。
```

---

## 更新摘要格式

```
Skill 增量更新完成

项目: {source_path}
变更范围: {last_commit_short}..{new_commit_short} ({N} 个 commit)

变更文件: {M} 个
更新的文档:
  ~ references/API_REFERENCE.md  (新增 2 个 API, 修改 1 个)
  ~ references/DATA_MODEL.md     (修改 1 个模型)
  - 其余文档无变化

.metadata 已更新: commit {new_commit_short}
```
