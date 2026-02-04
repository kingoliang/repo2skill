# repo2skill

English | [中文](README.md)

Convert local code repositories or remote GitHub/GitLab/Gitee repositories into [Claude Code](https://docs.anthropic.com/en/docs/claude-code) Skills.

## Problem It Solves

Claude Code starts every new conversation without any knowledge of your project. repo2skill pre-extracts key project information into skill documents, giving Claude Code automatic project context in subsequent sessions — where code lives, how architecture is layered, how APIs work, what data models look like.

## Features

- **Local + Remote**: Supports local project paths and GitHub/GitLab/Gitee remote repository URLs
- **Multi-language**: Java/Spring Boot, Node.js/TypeScript, Python/Django/FastAPI, Go, Rust, and more
- **Incremental Updates**: Detects changes via `git diff`, only rewrites affected documents, saving tokens
- **Standards Compliant**: Generated SKILL.md strictly follows skill-creator spec (<200 lines, progressive disclosure)
- **Batch Conversion**: Process multiple projects at once
- **Remote Mirrors**: GitHub API multi-mirror auto-switching with exponential backoff retry

## Installation

Copy the `repo2skill/` directory to your Claude Code skills directory:

```bash
# Global install (available for all projects)
cp -r repo2skill/ ~/.claude/skills/repo2skill/

# Or project-local install
cp -r repo2skill/ your-project/.claude/skills/repo2skill/
```

No dependencies required.

## Usage

Use natural language in Claude Code:

```
# Convert local project to skill
Convert this project to a skill: /path/to/my-project

# Convert remote repo to skill
Convert this repo to a skill: https://github.com/owner/repo

# Incremental update (default)
Update this skill: my-project

# Full regeneration
Regenerate this skill: my-project

# Batch conversion
Convert these projects:
- /path/to/project-a
- https://github.com/owner/repo-b
```

## Generated Content

### Always Generated

| File | Content |
|------|---------|
| `SKILL.md` | Main file: project overview, module structure, search commands, entry files (<200 lines) |
| `references/PROJECT_INDEX.md` | Project structure, dependency list |
| `references/ARCHITECTURE.md` | Architecture design, layering, call chains |
| `references/API_REFERENCE.md` | API endpoints, request/response parameters, call chains |
| `references/DATA_MODEL.md` | Data models, table structures, fields, relationships |
| `references/DOMAIN_GUIDE.md` | Business domain concepts, processes, terminology |
| `references/DEVELOPMENT.md` | Dev environment, build, test, config, deployment |

### Generated When Applicable

- `ENUMS_CONSTANTS.md` — Enums and constants
- `BUSINESS_RULES.md` — Business rules and constraints
- `TROUBLESHOOTING.md` — Common issue troubleshooting
- `CODE_CONVENTIONS.md` — Coding conventions

## Incremental Update Mechanism

A `.metadata` file is automatically created on first generation, recording the commit hash. On update:

1. Read `.metadata` to get last commit hash
2. Run `git diff --name-only {old}..HEAD` to get changed files
3. Map file types to corresponding documents, only rewrite affected sections
4. Update `.metadata`

**Change Mapping Rules:**

| Changed Files | Updated Document |
|---------------|-----------------|
| Controller / Router / Provider | API_REFERENCE.md |
| Entity / Model / Mapper | DATA_MODEL.md |
| Service / Domain | ARCHITECTURE.md + DOMAIN_GUIDE.md |
| pom.xml / package.json / go.mod | PROJECT_INDEX.md + DEVELOPMENT.md |
| Config / YAML / .env | DEVELOPMENT.md |
| Enum / Constants | ENUMS_CONSTANTS.md |

Automatically falls back to full regeneration when >30 files changed.

## Project Structure

```
repo2skill/
├── SKILL.md                              # Main skill file (170 lines)
└── references/
    ├── project-analysis.md               # Project analysis guide (183 lines)
    ├── remote-fetch.md                   # Remote repo fetching (122 lines)
    ├── skill-template.md                 # Generation templates (105 lines)
    └── incremental-update.md             # Incremental update guide (161 lines)
```

## How It Helps Claude Code Programming

Once a skill is generated, Claude Code automatically loads project context in subsequent conversations:

- **Locate code fast**: Search command tables in the skill tell Claude exactly where to find code
- **Understand architecture**: Layering and call chains help Claude write code in the correct layer
- **Modify APIs precisely**: Request/response parameter docs let Claude follow existing patterns
- **Handle data correctly**: Field types and constraints help Claude write correct DB operations
- **Understand business terms**: Domain guide prevents Claude from misinterpreting business concepts
- **Run correct commands**: Dev guide tells Claude which build and test commands to use

## Remote Repository Support

| Platform | API Mirrors | Auth |
|----------|-------------|------|
| GitHub | 4 API mirrors + 2 Raw mirrors, auto-switching | Optional token (raises limit to 5000/h) |
| GitLab | Official API | Optional PRIVATE-TOKEN |
| Gitee | Official API v5 | Optional token |

## References

- [Claude Code Skills Documentation](https://docs.anthropic.com/en/docs/claude-code/skills)
- Inspired by: [zhangyanxs/repo2skill](https://github.com/zhangyanxs/repo2skill)

## License

MIT
