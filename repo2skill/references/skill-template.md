# Skill Document Generation Templates

## SKILL.md Template

SKILL.md must be <200 lines. Use imperative/verb-first style.

```markdown
---
name: {skill-name}
description: {Rich description with use cases and trigger keywords, 80-150 words}
---

# {Project Name} Development Guide

## Project Overview

| Property | Value |
|----------|-------|
| **Project Type** | {type} |
| **Tech Stack** | {language} + {framework} + {key-libs} |
| **Base Package/Entry** | `{base-package-or-entry}` |

## Module Structure

| Module | Path | Description |
|--------|------|-------------|
| {module1} | `{path-pattern}` | {description} |
| {module2} | `{path-pattern}` | {description} |

## Search by Feature

| Feature | Search Command | Description |
|---------|---------------|-------------|
| {feature1} | `Glob "{pattern}"` | {description} |
| {feature2} | `Grep "{keyword}" --glob "{pattern}"` | {description} |

## Core Entry Files

| File | Path | Description |
|------|------|-------------|
| {file1} | `{path}` | {description} |

## Detailed Documentation

| Need to Know | Reference Document |
|--------------|--------------------|
| Project structure and dependencies | [PROJECT_INDEX.md](references/PROJECT_INDEX.md) |
| Architecture design and layering | [ARCHITECTURE.md](references/ARCHITECTURE.md) |
| API interface details | [API_REFERENCE.md](references/API_REFERENCE.md) |
| Data models/table structures | [DATA_MODEL.md](references/DATA_MODEL.md) |
| Business domain concepts | [DOMAIN_GUIDE.md](references/DOMAIN_GUIDE.md) |
| Development, build, testing | [DEVELOPMENT.md](references/DEVELOPMENT.md) |

## Common Commands

```bash
# Install dependencies
{install-command}

# Start development
{dev-command}

# Run tests
{test-command}

# Build
{build-command}
```
```

---

## References Document Templates

### PROJECT_INDEX.md

Contains: Basic info table (name/version/language/framework) → Directory structure (depth 3-4) → Core dependencies table (dependency/version/purpose) → Dev dependencies table. Each section annotated with `> Data source: {file}`.

### ARCHITECTURE.md

Contains: Layered architecture diagram (API→Service→Repository→DB) → Module dependency relationships → Core components table (component/location/responsibility).

### API_REFERENCE.md

Contains: API index table (method/path/description/auth) → Each API detail block:
- Basic info (file path)
- Request parameters table (field/type/required/description)
- Response parameters table (field/type/description)
- Call chain (Controller→Service→Repository)

Java projects: Determine required fields from `@NotBlank`/`@NotNull` annotations.

### DATA_MODEL.md

Contains: Model list table (model/file/corresponding table) → Each model detail:
- Field table (field/type/constraints/description)
- Relationships

### DOMAIN_GUIDE.md

Contains: Core concepts table → Business processes (step diagrams) → Glossary (term/meaning).

### DEVELOPMENT.md

Contains: Environment requirements table (tool/version) → Quick start commands → Build commands table → Test commands → Configuration table (config/description/default) → Deployment info.
