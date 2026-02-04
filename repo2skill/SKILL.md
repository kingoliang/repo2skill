---
name: repo2skill
description: Convert local code repositories or remote GitHub/GitLab/Gitee repositories into Claude Code Skills with git diff based incremental updates. Analyzes project structure, tech stack, core modules, APIs, databases, and auto-generates SKILL.md and references documents following skill-creator spec. Supports Java/Spring Boot/Node.js/Python/Go/Rust projects. Usage: provide a local project path or remote repository URL. Updates detect git changes and only rewrite affected documents.
---

# repo2skill - Repository to Skill Generator

## Features

Analyze local code repositories or remote repositories to generate complete Claude Code Skills, including:
- `SKILL.md` - Main skill file (<200 lines, follows skill-creator spec)
- `references/` - Detailed reference documents (loaded on demand)
- `scripts/` - Reusable scripts (if necessary)

## Usage

```
# First-time generation
Convert this project to a skill: /path/to/local/project
Convert this repo to a skill: https://github.com/owner/repo

# Incremental update (default, git diff based)
Update this skill: {skill-name}

# Full regeneration
Regenerate this skill: {skill-name}
```

**Parameters**:
- Local path or remote URL (GitHub/GitLab/Gitee)
- Optionally specify skill name and save location

---

## Workflow

### Step 1: Determine Source Type

- Local path → Analyze filesystem directly
- Remote URL → Fetch data per [remote-fetch.md](references/remote-fetch.md) then analyze

### Step 2: Project Analysis

Follow [project-analysis.md](references/project-analysis.md) to:

1. Identify tech stack (language, framework, build tools)
2. Analyze directory structure and module layout
3. Identify core entry files
4. Analyze API layer, business layer, data layer
5. Extract configuration, enums, constants

### Step 3: Generate Skill Documents

Follow [skill-template.md](references/skill-template.md) to generate:

#### Required Files

```
{skill-name}/
├── SKILL.md                          # Main file (<200 lines)
└── references/
    ├── PROJECT_INDEX.md              # Project overview, directory structure, dependencies
    ├── ARCHITECTURE.md               # Architecture design, layering, call chains
    ├── API_REFERENCE.md              # API endpoints/function index and details
    ├── DATA_MODEL.md                 # Data models/database table structures
    ├── DOMAIN_GUIDE.md               # Business domain, core concepts
    └── DEVELOPMENT.md                # Dev guide, build, test, configuration
```

#### Optional Files (generated when project has relevant content)

- `ENUMS_CONSTANTS.md` - Enums and constants
- `BUSINESS_RULES.md` - Business rules and constraints
- `TROUBLESHOOTING.md` - Common issues and troubleshooting
- `CODE_CONVENTIONS.md` - Coding conventions

### Step 4: Choose Save Location

Ask user where to save:

1. **Project local**: `.claude/skills/{skill-name}/`
2. **Global user**: `~/.claude/skills/{skill-name}/`

### Step 5: Write Files and Verify

1. Create directory structure
2. Write all files
3. Generate `.metadata` file to record generation snapshot (see [incremental-update.md](references/incremental-update.md))
4. Verify completeness (SKILL.md <200 lines, all referenced files exist)
5. Output generation summary

---

## Updating Existing Skills

Defaults to **git diff incremental update**; full regeneration when user explicitly requests it.

See [incremental-update.md](references/incremental-update.md) for detailed workflow.

### Incremental Update Flow

1. Read `.metadata` from skill directory to get last commit hash and project path
2. Run `git diff --name-only {last_commit}..HEAD` in project directory to get changed files
3. Map changed file types to reference documents that need updating
4. Only re-analyze changed code and rewrite affected documents
5. Update commit hash in `.metadata`

### Full Regeneration

When user explicitly requests ("regenerate", "full update"), skip diff detection and execute the full first-time generation workflow.

---

## Generation Principles

1. **SKILL.md must be <200 lines** - Detailed content goes in references/
2. **Progressive disclosure** - metadata → SKILL.md → references three-layer structure
3. **Practical focus** - Include real search patterns, Glob/Grep commands, code entry points
4. **Cite data sources** - Each document notes which files information was extracted from
5. **No assumptions** - Only document what actually exists in the code
6. **Rich description** - Include sufficient trigger keywords and use cases

---

## Batch Conversion

Supports converting multiple projects at once:

```
Convert these projects:
- /path/to/project-a
- /path/to/project-b
- https://github.com/owner/repo-c
```

Process each sequentially, report all results at the end.

---

## Output Summary Format

```
Skill generation complete

Location: ~/.claude/skills/{skill-name}/

Generated files:
  SKILL.md (main file, XX lines)
  references/PROJECT_INDEX.md
  references/ARCHITECTURE.md
  references/API_REFERENCE.md
  references/DATA_MODEL.md
  references/DOMAIN_GUIDE.md
  references/DEVELOPMENT.md

Project analysis:
  Tech stack: {tech-stack}
  Modules: X
  APIs: X
  Data tables: X
```

---

## Error Handling

- Path does not exist → Prompt user and ask for correct path
- Remote repo inaccessible → Use mirror and retry strategy from [remote-fetch.md](references/remote-fetch.md)
- Project is empty → Inform user and stop
- Error during analysis → Log error, skip that section, continue generating remaining documents
