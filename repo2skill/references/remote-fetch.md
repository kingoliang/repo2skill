# Remote Repository Data Fetching

## URL Parsing

### Platform Detection

| Platform | URL Pattern |
|----------|-------------|
| GitHub | `github.com/{owner}/{repo}` |
| GitLab | `gitlab.com/{owner}/{repo}` |
| Gitee | `gitee.com/{owner}/{repo}` |

Extract: platform, owner, repo name

---

## GitHub

### API Mirrors (by priority)

1. `https://api.github.com`
2. `https://gh.api.888888888.xyz`
3. `https://gh-proxy.com/api/github`
4. `https://api.fastgit.org`

### Raw File Mirrors

1. `https://raw.githubusercontent.com`
2. `https://raw.fastgit.org`

### Key Endpoints

```bash
# Repository info
curl -s -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/{owner}/{repo}

# README (base64 encoded)
curl -s https://api.github.com/repos/{owner}/{repo}/readme

# File tree (recursive)
curl -s "https://api.github.com/repos/{owner}/{repo}/git/trees/main?recursive=1"

# Get file content
curl -s https://api.github.com/repos/{owner}/{repo}/contents/{path}
```

### Rate Limits
- Unauthenticated: 60/hour
- Authenticated: 5000/hour
- Check header: `X-RateLimit-Remaining`

---

## GitLab

### Endpoint

`https://gitlab.com/api/v4`

### Key APIs

```bash
# Project info (path must be URL encoded)
curl -s "https://gitlab.com/api/v4/projects/{owner}%2F{repo}"

# File tree
curl -s "https://gitlab.com/api/v4/projects/{owner}%2F{repo}/repository/tree?recursive=1"

# Read file
curl -s "https://gitlab.com/api/v4/projects/{owner}%2F{repo}/repository/files/{path}/raw?ref=main"
```

### Rate Limits
- Unauthenticated: ~60/minute

---

## Gitee

### Endpoint

`https://gitee.com/api/v5`

### Key APIs

```bash
# Repository info
curl -s https://gitee.com/api/v5/repos/{owner}/{repo}

# README
curl -s "https://gitee.com/api/v5/repos/{owner}/{repo}/readme"

# File tree
curl -s "https://gitee.com/api/v5/repos/{owner}/{repo}/git/trees/master?recursive=1"
```

---

## Retry Strategy

1. Try primary mirror
2. On failure (403/429/timeout) → Switch to next mirror
3. Exponential backoff: 1s → 2s → 4s → 8s
4. Max 3 retries per mirror
5. All failed → Inform user to check network or VPN

## Key Files to Fetch

The following files must be fetched from remote repositories for analysis:

1. README.md
2. Build files (package.json / pom.xml / go.mod, etc.)
3. Directory tree structure
4. Core source files (filtered from directory tree, fetched individually)
5. Configuration files

**Note**: Large repo file trees may be truncated; prioritize files under src/.

## Branch Detection

Try in order: `main` → `master` → `develop` → `dev`
