# Incremental Update Guide

## .metadata File Format

On first skill generation, create a `.metadata` file in the skill directory:

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

Additional fields for remote repositories:
```json
{
  "source_type": "remote",
  "source_url": "https://github.com/owner/repo",
  "platform": "github",
  "branch": "main"
}
```

---

## Incremental Update Flow

### 1. Read .metadata

```bash
Read {skill-dir}/.metadata
```

If `.metadata` does not exist → Inform user this skill does not support incremental update, suggest full regeneration.

### 2. Check Project State

```bash
cd {source_path}
git rev-parse HEAD              # Current commit
git rev-parse {last_commit}     # Verify old commit is still in history
```

If old commit is not in history (e.g., rebase/force push) → Fall back to full regeneration.

### 3. Get Changed File List

```bash
git diff --name-only {last_commit}..HEAD
```

If no changes → Inform user skill is up to date, no update needed.

### 4. Classify Changed Files

For each file path in the diff output, classify by the following rules:

| Changed File Pattern | Document to Update |
|---------------------|--------------------|
| `**/controller/**`, `**/routes/**`, `**/views/**`, `**/*Provider.*`, `**/*Handler.*` | API_REFERENCE.md |
| `**/entity/**`, `**/model/**`, `**/models/**`, `**/schema/**`, `**/*DO.*`, `**/*Mapper.xml` | DATA_MODEL.md |
| `**/service/**`, `**/services/**`, `**/domain/**`, `**/usecases/**` | ARCHITECTURE.md, DOMAIN_GUIDE.md |
| `**/enum/**`, `**/enums/**`, `**/constants/**`, `**/types/**` | ENUMS_CONSTANTS.md |
| `**/config/**`, `**/configuration/**`, `**/*.yml`, `**/*.yaml`, `.env*` | DEVELOPMENT.md |
| `pom.xml`, `package.json`, `go.mod`, `Cargo.toml`, `requirements.txt` | PROJECT_INDEX.md, DEVELOPMENT.md |
| `Makefile`, `Dockerfile`, `docker-compose.*`, `.github/workflows/**` | DEVELOPMENT.md |
| `README.md`, `CONTRIBUTING.md` | DEVELOPMENT.md |

#### Special Cases

- **Added/deleted directories** → Also update the module structure table in SKILL.md
- **Build file changes** (pom.xml/package.json) → Also check if new dependencies affect tech stack description
- **Large number of changes** (>30 files) → Recommend full regeneration, as incremental update may miss correlated changes

### 5. Re-analyze Affected Code

Only read changed files and related files (same module/directory), re-extract information.

For deleted files: Remove corresponding entries from the document.
For added files: Add new entries to the corresponding document.
For modified files: Re-analyze and replace corresponding entries in the document.

### 6. Rewrite Affected Documents

Only rewrite documents marked in step 4; leave remaining documents unchanged.

### 7. Update .metadata

```json
{
  "commit_hash": "{new_HEAD_commit}",
  "generated_at": "{current_timestamp}",
  "last_update_type": "incremental",
  "updated_files": ["references/API_REFERENCE.md", "references/DATA_MODEL.md"]
}
```

---

## Remote Repository Incremental Update

Remote repositories cannot use git diff directly; use the following alternative:

1. Fetch via the compare API:
   ```bash
   # GitHub
   curl -s "https://api.github.com/repos/{owner}/{repo}/compare/{last_commit}...HEAD"
   ```
2. Extract changed file list from the `files` array in the response
3. Subsequent flow is the same as local

If compare API fails (commit too old, repo rebased, etc.) → Fall back to full regeneration.

---

## Conditions for Falling Back to Full Regeneration

Automatically switch to full regeneration when any of the following conditions are met:

1. `.metadata` file does not exist
2. Old commit hash is not in git history
3. Changed file count >30
4. Build file changes cause fundamental tech stack change (e.g., Maven to Gradle)
5. User explicitly requests ("regenerate", "full update", "--force")

Inform user of the reason when falling back:
```
Detected XX file changes (exceeds threshold of 30), performing full regeneration to ensure completeness.
```

---

## Update Summary Format

```
Skill incremental update complete

Project: {source_path}
Change range: {last_commit_short}..{new_commit_short} ({N} commits)

Changed files: {M}
Updated documents:
  ~ references/API_REFERENCE.md  (added 2 APIs, modified 1)
  ~ references/DATA_MODEL.md     (modified 1 model)
  - Remaining documents unchanged

.metadata updated: commit {new_commit_short}
```
