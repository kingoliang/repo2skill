# 远程仓库数据获取

## URL 解析

### 平台检测

| 平台 | URL 模式 |
|------|----------|
| GitHub | `github.com/{owner}/{repo}` |
| GitLab | `gitlab.com/{owner}/{repo}` |
| Gitee | `gitee.com/{owner}/{repo}` |

提取: 平台、owner、repo 名称

---

## GitHub

### API 镜像（按优先级）

1. `https://api.github.com`
2. `https://gh.api.888888888.xyz`
3. `https://gh-proxy.com/api/github`
4. `https://api.fastgit.org`

### Raw 文件镜像

1. `https://raw.githubusercontent.com`
2. `https://raw.fastgit.org`

### 关键端点

```bash
# 仓库信息
curl -s -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/{owner}/{repo}

# README（base64 编码）
curl -s https://api.github.com/repos/{owner}/{repo}/readme

# 文件树（递归）
curl -s "https://api.github.com/repos/{owner}/{repo}/git/trees/main?recursive=1"

# 获取文件内容
curl -s https://api.github.com/repos/{owner}/{repo}/contents/{path}
```

### 速率限制
- 未认证: 60次/小时
- 认证: 5000次/小时
- 检查 header: `X-RateLimit-Remaining`

---

## GitLab

### 端点

`https://gitlab.com/api/v4`

### 关键 API

```bash
# 项目信息（路径需 URL 编码）
curl -s "https://gitlab.com/api/v4/projects/{owner}%2F{repo}"

# 文件树
curl -s "https://gitlab.com/api/v4/projects/{owner}%2F{repo}/repository/tree?recursive=1"

# 读取文件
curl -s "https://gitlab.com/api/v4/projects/{owner}%2F{repo}/repository/files/{path}/raw?ref=main"
```

### 速率限制
- 未认证: ~60次/分钟

---

## Gitee

### 端点

`https://gitee.com/api/v5`

### 关键 API

```bash
# 仓库信息
curl -s https://gitee.com/api/v5/repos/{owner}/{repo}

# README
curl -s "https://gitee.com/api/v5/repos/{owner}/{repo}/readme"

# 文件树
curl -s "https://gitee.com/api/v5/repos/{owner}/{repo}/git/trees/master?recursive=1"
```

---

## 重试策略

1. 尝试主镜像
2. 失败（403/429/超时）→ 切换下一个镜像
3. 指数退避: 1s → 2s → 4s → 8s
4. 每个镜像最多重试 3 次
5. 全部失败 → 告知用户检查网络或 VPN

## 获取关键文件

远程仓库需获取以下文件用于分析：

1. README.md
2. 构建文件（package.json / pom.xml / go.mod 等）
3. 目录树结构
4. 核心源码文件（通过目录树筛选后逐个获取）
5. 配置文件

**注意**: 大型仓库文件树可能被截断，优先获取 src/ 目录下的文件。

## 分支检测

依次尝试: `main` → `master` → `develop` → `dev`
